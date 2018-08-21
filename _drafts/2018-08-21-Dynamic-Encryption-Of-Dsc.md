---
title: "PowerShell: Dynamic Dsc Encryption"
categories: "powershell"
---
# PowerShell: Dynamic Dsc Encryption

## The Issue

I was recently faced with the need to, yet again, encrypt a Dsc process and followed the usual instructions; connect to a server, generate a document encryption certificate, export certificate, transfer certificate to release agent server, install certificate, copy certificate to central store, add reference to thumbprint and certificate location on Dsc Template and ConfigurationData, run job.

I have experienced this every time I need to get a new deployment up and running to a new server, with a secret involved.

When you have a credential secret there is an issue that, at rest on the release agent, you can find an unencrypted plaintext password of the account in the mof, adding a certificate allows you to encrypt the mof in both locations.

## Outcome

What I am hopefully going to show is a way to shortcut this process and get and use the certificate at run time.

## The Guidelines

I was following the guidelines from the <a href="https://docs.microsoft.com/en-us/powershell/dsc/securemof" target="_blank" rel="noopener">microsoft documentation</a>, but found some of the steps unnecessary.

This behaviour suggests variations of going to the target, creating a certificate, export rom certificate store, transfer to the agent box and then transfer to a central store. When you have 12 build and release servers, I'm sure you can tell this is added pain.

What confused me was the need for the certificate to be registered against the server running the encryption, but also referencing a central store for the certificate, even though it is imported into the local machine certificate store; turned out this isn't needed!

## The Actions

I often break down my process of deployment into a setup and push. During the setup I am making sure the servers have everything installed that they need to perform the Dsc action which is handled during the push.

Created Get function for setup and Use for push.

### The Get

The action of the get connected ..

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
Copy-Item -FromSession $Session -Path "c:\temp\$($cn)$($Suffix).cer" -Destination $workingDirectory
}
}

end {
}
}
```

### And Use

Because the configuration is nothing more complicated than a hastable

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