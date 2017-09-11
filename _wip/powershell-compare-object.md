# PowerShell: Compare-Object

I'd recently been trying to compare if a set of files exist in another location.

So I started with best intentions.

```powershell
$list1 = Get-ChildItems "location1" -Recursive
$list2 = Get-ChildItems "location2" -Recursive

foreach($item in $list1)
{
if($list2 -contains $item.Name)
{
#do some things
}
}
```

this started to get a little tricky to manage and track as I was dealing with a collection of files and trying to compare a specific element from one file with the whole array of files I had from the other location.

So I turned to Google; In my searches, I found `Compare-Object`

Compare-Object allows you to provide 2 different lists of things, it will then tell you which  thing is missing from which list, using operators of ``

So my above reduced to:

```powershell
$items = Compare-Object -ReferenceObject $list1 -DifferenceObject $list2
```

As I needed the actual files, I added the `-PassThru` argument so that the files found were available to use.

That meant, from the list of $items I was able to find those which were missing from one side and `Copy-Item` or `Move-Item` them over.