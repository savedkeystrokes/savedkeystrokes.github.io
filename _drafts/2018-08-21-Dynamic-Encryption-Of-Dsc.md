---
title: "PowerShell: Dynamic Dsc Encryption"
categories: "powershell"
---
# PowerShell: Dynamic Dsc Encryption

## The Pitch

When you have a credential secret there is an issue that, at rest on the release agent, you can find an unencrypted plaintext password of the account in the mof, adding a certificate allows you to encrypt the mof in both locations.

To encrypt a Dsc process you follow the usual instructions:

- connect to a server,
- generate a document encryption certificate,
- export certificate,
- transfer certificate to authoring server,
- install certificate on authoring server,
- copy certificate to central store,
- add reference to thumbprint and certificate location on Dsc Template and ConfigurationData,
- Run the job.

I have experienced this every time I need to get a new deployment up and running to a new server, with a secret involved. What I am hopefully going to show is a way to shortcut this process and get and use the certificate at run time.

## The Guidelines on Securing a mof

I generally follow the guidelines from the <a href="https://docs.microsoft.com/en-us/powershell/dsc/securemof" target="_blank" rel="noopener">microsoft documentation</a>, but found some of the steps unnecessary.

What confused me was the need for the certificate to be registered against the authoring server, but also referencing a central store for the certificate, even though it is imported into the local machine certificate store; turns out this isn't needed!

## The Process

I often break down my process of deployment into a setup and push phase. During the setup I am making sure the servers have everything installed that they need to perform the Dsc action which is handled during the push. So getting the certificate to perform the encryption fits here.

The configuration is nothing more complicated than a hastable the information can be loaded into memory and modified as needed.

For a mof to be secured, you need:

- a Certificate file for the Target Node
- a Thumbprint based on the certificate file
- a CertificateID which comes from the node's thumbprint LocalConfigurationManager

Here is an example Config:

```powershell
@{
    AllNodes = @(
    @{NodeName = "ServersFQDN";
    Role = "WebServer"
    PSDscAllowDomainUser = $true;
    CertificateFile = "ServersFQDNDscEncryptionCert.cer"
    Thumbprint = ""
    }
)
NonNodeData = @{
    }
}
```

I am defining a `CertificateFile`, with the prefix of the `NodeName`, this isnt a fully defined path, which is required, but more a pointer to what the file will be called. The reason for this is we need to get the certificate from the target node via a session, with some conventions like this applied this makes life a bit easer. There are probably a load of improvements people could make, this was still v0.5 at the time of writing this.

When the process is started you can load this config into a variable, which allows for modifications to be made.

```powershell
$config = Get-Content ".\Config.psd1" | Out-String | iex
```

_The location of the config.psd1 in relation to where you are running the script is important, so if you know or can define the full path do that._

## Getting the Certificate

You will need to instanciate some sessions to the servers that will be targeted during this process.

In your Config you define the servers you want to get to, and using the `NodeName` from the array of servers in your `AllNodes`, you can create all the sessions you need and pass this to the following function.

- **`$WorkingDirectory`** could be `$PSScriptRoot`, this the location where all the scripts are rooted.
- **`$Sessions`** an array of all the sessions created.
- **`$Suffix`** what is the name of the file, the certificate will always be prefixed with the FQDN of the server and have the extension of `cer`, the suffix relates to the `CerfificateFile`, so if you change this value it will need to be reflected there.

If you have a sever named `Server01.local` and change the suffix to be `Cert`, the Get process will generate a file called `Server01.localCert.cer`, if you havent named the `CertificateFile` property, in your Node data, the same it will not match.

```powershell
function Get-DscEncryptionCertificate {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory=$true)]
        [String]
        $WorkingDirectory,
        [Parameter(Mandatory=$true)]
        [System.Management.Automation.Runspaces.PSSession[]]
        $Sessions,
        [String]
        $Suffix="DscEncryptionCert"
    )

    begin {
    }
    process {
        Invoke-Command -Session $Sessions -ScriptBlock {
            Param($Suffix)
            $FQDN = ([System.Net.Dns]::GetHostByName($env:computerName).HostName)
            $DnsName = "$FQDN$Suffix"
            # note: These steps need to be performed in an Administrator PowerShell session
            if ($null -ne (Get-ChildItem -Path cert:\LocalMachine\My | Where-Object Subject -Like CN=$DnsName)) {
                $thumbprint = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object Subject -Like CN=$DnsName | Select-Object -ExpandProperty ThumbPrint
                $cert = "cert:\LocalMachine\My\$thumbprint"
            }
            else {
                $cert = New-SelfSignedCertificate -Type DocumentEncryptionCertLegacyCsp -DnsName $DnsName -HashAlgorithm SHA256
            }

            Export-Certificate -Cert $cert -FilePath "c:\temp\$($DnsName).cer"
        } -ArgumentList $Suffix

        foreach ($session in $Sessions) {
            $cn = $session.ComputerName
            Move-Item -FromSession $Session -Path "c:\temp\$($cn)$($Suffix).cer" -Destination $workingDirectory
        }
    }
    end {
    }
}
```

This process, moves all the certificates from the corresponding targets to the authoring server's working directory ready for the Use process.

### Using this Use

The Use process is part of the Push even, you are ready to generate your mof files so now is where you will encrypt them.

The `Use-DscEncryptionCertificate` asks for:

- **`$ConfigData'** - The ConfigurationData from your process.
- **`$WorkingDirectory'** - The Root location of where the certificates are likely to be found.

```powershell
function Use-DscEncryptionCertificate {
    [CmdletBinding()]
    param(
        [HashTable]
        [parameter(Mandatory, ValueFromPipeline)]
        $ConfigData,
        [String]
        [Parameter(Mandatory)]
        $WorkingDirectory
    )

    begin {
    }

    process {
        foreach ($node in $ConfigData.AllNodes) {
            $certPath = (Join-Path $WorkingDirectory $node.CertificateFile)
            $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 $certPath

            $node.Thumbprint = $cert.Thumbprint
            $node.CertificateFile = $certPath
        }
    }

    end {
    $ConfigData
    }
}
```

## And Finally

The final steps of encryption are done for you. The addition of the Thumbprint and Full Path to the certificate are added to the ConfigData by the `Use-DscEncryptionCertificate`. Here is an example of the full run from Getting the sessions created and performing the Get and Use functions.

```powershell
$location = $PSScriptRoot
Set-Location $location

$config = Get-Content "$location\Config.psd1" | Out-String | iex

$targets = @()
$config.AllNodes | Where-Object { $_.Role -eq "WebServer" } | ForEach-Object { $targets += $_.NodeName }

$Sessions = New-PSSession -ComputerName $targets -Credential $deployCred

# Get Certs
Get-DscEncryptionCertificate -WorkingDirectory $location -Sessions $Sessions

# Use Certs
$config = Use-DscEncryptionCertificate -ConfigData $config -WorkingDirectory $location

. $location\DscConfig.ps1

Config -ConfigurationData $config -OutputPath $location\Config -Verbose

$cs = New-CimSession -ComputerName $targets -Credential $deployCred

Write-Host "Start DSC"

Set-DscLocalConfigurationManager $location\Config -CimSession $cs  -Verbose -ErrorAction Stop

Start-DscConfiguration -Path $location\Config -CimSession $cs -Verbose -Wait -Force  -ErrorAction Stop

Remove-CimSession -CimSession $cs
```

## Summary

Thanks for getting this far. This isnt perfect, I will probably find I have shot myself in another way, but this fits how we are working. If there are any suggestions I am more than willing to discuss.