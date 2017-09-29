---
title: "git: Adding an alias to git for a repository refresh"
---
# git: Adding an alias to git for a repository refresh

When working with git, I find myself doing a lot of the same requests, by creating an alias I reduced my keystrokes.

## The longhand

```shell
git fetch --prune

git reset --hard origin/[branchname here]

git clean -dfX

git clean -dfx
```

## so, I created an alias

I have this added this to my `.gitconfig` file found in _Users/[loginname]_.

```shell
[alias]
refresh = !git fetch --prune && git reset --hard origin/$(git rev-parse --abbrev-ref HEAD) && git clean -dfX && git clean -dfx
```

**update**
Since writing this post I have been getting conflicts within the `.vs` folder, so that forced me to modify the refresh script.

```shell
[alias]
refresh = !git fetch --prune && git reset --hard origin/$(git rev-parse --abbrev-ref HEAD) && git clean -dfx -e '.vs/'
```

`$(git rev-parse --abbrev-ref HEAD)` works against your current branch and finds the related branch name, in theory, you have a remote branch with the same name.

the 2 git clean calls do slightly different things.

d,f,x and X are arguments on git clean, I could probably just do git clean -dfxX (not sure), but I created the script to reflect what I normally type. (**update** I checked you cannot do xX).

 
>-d
Remove untracked directories in addition to untracked files. If an untracked directory is managed by a different Git repository, it is not removed by default. Use -f option twice if you really want to remove such a directory.

>-f --force
If the Git configuration variable clean.requireForce is not set to false, git clean will refuse to delete files or directories unless given -f, -n or -i. Git will refuse to delete directories with .git sub directory or file unless a second -f is given.


>-x
Don’t use the standard ignore rules read from .gitignore (per directory) and $GIT_DIR/info/exclude, but do still use the ignore rules given with -e options. This allows removing all untracked files, including build products. This can be used (possibly in conjunction with git reset) to create a pristine working directory to test a clean build.

>-e
Exclude, this basically allows you to define an exclusion to ignore after applying -x.

>-X
Remove only files ignored by Git. This may be useful to rebuild everything from scratch, but keep manually created files.

_if I'd left the `git clean -dfX` the whole `git clean -dfx -e '.vs/'` would be pointless._

If you don't need or want the full functionality the alias can be adapted to each person on a machine suit their needs.


## Summary 

So now, all I need to do to activate the process is `git refresh` and because I added this to my global git config it is available on any git repository on my machine.

I am expecting some convention, such as you are working against a remote called origin and your remote branch has the same name as your local, but it does save me keystrokes.
