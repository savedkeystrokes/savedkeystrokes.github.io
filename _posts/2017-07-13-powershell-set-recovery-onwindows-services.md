---
title: "PowerShell: Set-Recovery on Windows Services"
---
# PowerShell: Set-Recovery on Windows Services

## Connectivity outage was stopping services

We had this issue during some intermittent network outage or slowdown; the database connection used by an nsb service would drop and cause the services to stop because an error was detected. To get around that we set all the services to have a recovery to restart after 3 minutes, but there were a lot of services and after a new deployment the setup would be lost, until it was added to the release process.

## Using sc.exe to set the recovery

I did a bit of googling and made up the following PowerShell function to call `sc.exe`, set the recovery action that would restart the service after a defined millisecond wait (as an aside I found 3 minutes did the job):

```powershell
function Set-Recovery {
param
(
[string]
[Parameter(Mandatory=$true)]
$ServiceName,
[string]
[Parameter(Mandatory=$true)]
$Server,
[int]
[Parameter(Mandatory=$true)]
$restart
)

sc.exe "\\$Server" failure $ServiceName reset= 0 actions= restart/$restart #Restart after x ms
}
```

This is then simply called by doing the following

```powershell
# Example of call
Set-Recovery "NSBService" "ServerName" 18000
```

Information is then easily provided to this call, by looping through a list of known servers and services.
