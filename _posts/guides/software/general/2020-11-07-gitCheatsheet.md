---
title: "GIT Cheatsheet"
excerpt: "Useful git commands"
permalink: /guides/software/git
toc: true
categories:
  - guide
  - software
  - software-guide
---

### Branches

Create a new branch and check it out
```
git checkout -b <new-branch-name>
```

Delete a branch, or several Branches
```
git branch -D <branch-1>
git branch -D <branch-1> <branch-2> <etc>
```

### Rebase

Git rebase a branch onto `master` to keep the history clean:

```
git checkout master
git pull
git checkout <branch-name>
git rebase master
git push -f 
```
**Note: gotta use `force` since the remote's history will disagree with the rebase - be careful with this is multiple people are developing on the same branch.**

### Remove a commit you don't want

Say you want to get rid of "Fix LED driver pin indexing bug" in:

```
% git log
commit 473c7fc9ddd219f47b620c7d6c8e577cc465dcd2 
    Integrate thermistor driver with ADC and model

commit e616121e4e6601b7fdd467ecdc8147a421955860
    Fix LED driver pin indexing bug

commit 1eee3dd25b2da82ee86f8ac62858ae92e1dfce1a
    Add steinhart-hart equation model

commit b9712ba101de52fdc69f0a1d4f35421d38d75bef
    Add scaffolding for thermistor driver
```

It can be removed by performing an interactive `git rebase` (target the branch before the one we want to remove)

```
git rebase -i 1eee3dd25b2da82ee86f8ac62858ae92e1dfce1a 
```

We are then presented with an interactive vi session. Change `pick` to `drop` for the commit we want to remove, then save and exit.

```
drop e616121 Fix LED driver pin indexing bug
pick 473c7fc Integrate thermistor driver with ADC and model

# Rebase 18847fd..473c7fc onto 18847fd (1 command)
# ... etc
```

Now `git log` shows:
```
commit 7ec9784b2427c759f1dce7ffe83057c446812572
    Integrate thermistor driver with ADC and model

commit 1eee3dd25b2da82ee86f8ac62858ae92e1dfce1a
    Add steinhart-hart equation model

commit b9712ba101de52fdc69f0a1d4f35421d38d75bef
    Add scaffolding for thermistor driver
```

### Squash a bunch of commits

Say we have the following commits, and would prefer them to all be part of one unified commit:
```
% git log
commit 7ec9784b2427c759f1dce7ffe83057c446812572
    Integrate thermistor driver with ADC and model

commit 1eee3dd25b2da82ee86f8ac62858ae92e1dfce1a
    Add steinhart-hart equation model

commit b9712ba101de52fdc69f0a1d4f35421d38d75bef
    Add scaffolding for thermistor driver
```

Again, we do an interactive `rebase`.

```
% git rebase -i ~HEAD-3
```

This time, change the last two commits from `pick` to `squash`.

```
pick b9712ba Add scaffolding for thermistor driver
squash 1eee3dd Add steinhart-hart equation model
squash 7ec9784 Integrate thermistor driver with ADC and model

# Rebase b9712ba..7ec9784 onto b9712ba (2 commands)
```

We can then choose a new commit message:

```
# This is a combination of 3 commits.
# This is the 1st commit message:

Add thermistor driver with steinhart-hart estimator

# Please enter the commit message for your changes. Lines starting...
```

Our history is now:
```
% git log
commit d303b5387dd444f35473be0cc25b286da7a76255 (HEAD -> feature/thermistor)
    Add thermistor driver with steinhart-hart estimator
```

### Submodules

Add an existing repository:

```
git submodule add <repo-url.git>
```

When cloning a repository with submodules, the submodules will be empty.

To populate the submodules, run:

```
git submodule update --init
```

