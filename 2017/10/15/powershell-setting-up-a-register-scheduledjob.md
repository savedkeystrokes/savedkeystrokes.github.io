# Powershell: Setting up a Register-ScheduledJob

Let's look at something that I did, I needed to keep an eye on all the recycle bins on a server, that was a separate problem, but I wanted to schedule the job to occur once a week.

Just in case you are interested the script to do the recycle clean looks like this

```powershell
$Shell = New-Object -ComObject Shell.Application
$Global:Recycler = $Shell.NameSpace(0xa)

$Global:Recycler.Items() | Remove-Item -Recurse
```

Register-ScheduledJob requires a ScriptBlock, so this becomes

```powershell
$scriptBlock = {
$Shell = New-Object -ComObject Shell.Application
$Global:Recycler = $Shell.NameSpace(0xa)

$Global:Recycler.Items() | Remove-Item -Recurse
}
```

I then needed it to run in an elevated state, so I needed to set some options.

```powershell
$options = New-ScheduledJobOption -RunElevated
```

Then I wanted a trigger to run every Monday at 7 AM.

```powershell
$trigger = New-JobTrigger -DaysOfWeek Monday -At 07:00 -Weekly
```

I've also had to have one trigger every hour

```powershell
$trigger = New-JobTrigger -Once -At "2017-07-14" -RepetitionInterval (New-TimeSpan -Hour 1) -RepetitionDuration ([TimeSpan]::MaxValue)
```

And then you wire it all together.

```powershell
Register-ScheduledJob -Name "Recycle Cleaner" -ScriptBlock $scriptblock -Trigger $trigger -ScheduledJobOption $options
```