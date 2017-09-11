# git: using git-tfs - The Gitification Project

## More to follow

I have another post planned that I am working on, that will go into a bit more detail about the fun that can be had with git-tfs, but for now, I will just provide a reference to [the git-tfs site](http://git-tfs.com/]).

## Basic Brief

I was asked to setup a transfer for some of our tfvc sources to git. I was already quite up to speed with what git-tfs would allow you to do, but this was an experience to flex and expand on what I already knew.

I began with a basic setup, which connects to the beginning of the branch and then steps through the entire history and added the desired new home.

```powershell
$env:Path = $env:Path + ";C:\tools\gittfs"

cd C:\g

$tfbranch = "$/Project/Product/Main"

$work = "Product"

$git = "http://tfs.domain/tfs/Collection/Project/_git/$work"

git tfs clone http://tfs/tfs/Collection $tfbranch $work

cd $work

git remote add origin $git
```

After this initial setup, I created the new repository and pushed the branch. I had a colleague setting up the builds so we would address any issues after comparing the sources.

But that was a one off, I needed the process to continue when a new commit was made. The behaviour I wanted was; on new commit, get the latest version and refresh the branch, push the new head to the new git repository.

## Automation of the process

The process of creating a scripted process was relatively straightforward. 

The first event was to `checkout` the current branch and perform a fetch against the tfvc source, the head of the local branch was then applied with a hard reset and the subsequent changes pushed to the new origin.

In script form I had the following, the arguments passed in were 
%1 = local branchname / git remote branchname
%2 = source local copy location
%3 = git-tfs tfs remote branchname

```shell
cd E:\gittfs\%2

SET PATH=%PATH%;E:\tools\gittfs

git checkout %1

git tfs fetch -l

git reset --hard tfs/%3

git push origin %1

git push origin --tags
```

## Some pitfalls to be aware of

1. I was initially looking at using PowerShell for the automated script, but the output from git was returning an error to the PowerShell console, even on success. so I went with command line, with the downside being, you don't get any errors reported at all!

2. When you perform `git tfs fetch` it will start at the last known point and find the subsequent steps if a branch was deleted and a new one created with the same name, it will not jump to the new start and track any changes beyond the point it was deleted. This can be fixed by checking the commit number of the next stable point you want to reference and calling `git tfs fetch -c[commit number]`

3. For some reason or other, I had a commit on one history being completely missed off, which created a different future of the git repository versus the original tfvc. I needed to replay the history from a known commit and overwrite the history up to the latest check-in. This was possible by doing `git tfs fetch -c[commit number] --force`. If you don't use force all the commits will effectively duplicate on the head of your current branch.

4. If you do want a bridge between the 2 version, and not just a 1-way sync, you do need to make sure that your branch on tfvc is a branch and not just a folder, you will get a warning to that effect.

## Summary

I am quite surprised by what I was able to do, in such a short time, something I thought was going to cause a lot more trouble wasn't half the issue I thought. There do appear to be some little gotchas that you need to be aware of, which I listed above, but other than I was able to fulfil the brief with the git-tfs tooling.