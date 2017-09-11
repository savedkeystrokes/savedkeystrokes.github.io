# PowerShell: Multiline statements

It is a bit painful sometimes when you write a script and it ends up going off the screen.

```powershell
$items = Get-ChildItem -Path $drive -Recurse -Include ("*.jpg") |  Where-Object {$_.LastWriteTime.ToUniversalTime() -lt $a.ToUniversalTime() -and $_.FullName -notlike "*Wedding*" -and $_.FullName -notlike "*Dog*" }
```

And there isn't always a natural break, adding a simple ` (I call it a back-tick! Found on the top most left most button on a UK keyboard) to the end of the line you want to break, allows you to run a statement onto several lines.

This is the same character that most will be using on forums to `block code sections`, so a lot of developers should be used to working with it.

```powershell
$items = Get-ChildItem -Path $drive -Recurse -Include ("*.jpg") | `
Where-Object {$_.LastWriteTime.ToUniversalTime() -lt $a.ToUniversalTime() `
-and $_.FullName -notlike "*Wedding*" `
-and $_.FullName -notlike "*Dog*" }
```

I know the width of this theme still cuts the lines a little, I'm working on it.It is a bit painful sometimes when you write a script and it ends up going off the screen.

```powershell
$items = Get-ChildItem -Path $drive -Recurse -Include ("*.jpg") |  Where-Object {$_.LastWriteTime.ToUniversalTime() -lt $a.ToUniversalTime() -and $_.FullName -notlike "*Wedding*" -and $_.FullName -notlike "*Dog*" }
```

And there isn't always a natural break, adding a simple ` (I call it a back-tick! Found on the top most left most button on a UK keyboard) to the end of the line you want to break, allows you to run a statement onto several lines.

This is the same character that most will be using on forums to `block code sections`, so a lot of developers should be used to working with it.

```powershell
$items = Get-ChildItem -Path $drive -Recurse -Include ("*.jpg") | `
Where-Object {$_.LastWriteTime.ToUniversalTime() -lt $a.ToUniversalTime() `
-and $_.FullName -notlike "*Wedding*" `
-and $_.FullName -notlike "*Dog*" }
```

I know the width of this theme still cuts the lines a little, I'm working on it.