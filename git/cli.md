# CLI

## Clone

```
git clone URL

-b                  - clone specific branch or tag
--single-repository – clone the only branch, otherwise all branches will be cloned (to check: git branch –-all)
--depth 1           - clone single commit, otherwise full commit tree (to check: git log –oneline, should see the only latest commit)
```

## Log

```
git log --graph --all --decorate --oneline
```

* Graph – shows ASCII graph on left side
* All – shows all branches (by default only active branch displayed)
* Decorate – shows branch names and tags
* Oneline – makes output shorter

## Tags

```
# list all tags
git tag -l

# checkout tag
git checkout tags/xxx 

# better to create named branch at tag checkout, otherwise it will be just detached HEAD
git checkout tags/xxx -b <new branch name>
```

## Re-base vs Reset vs Revert

Revert – create another commit reversing the requeted one

Re-base – re-create all commits from requested one to the latest

Reset – cancel all commits till the requested one, keep workspace untouched

## Stashing

The one place I do like stashing is if I discover I forgot something in my last commit and have already started working on the next one in the same branch:

```
# Assume the latest commit was already done
# start working on the next patch, and discovered I was missing something

# stash away the current mess I made
git stash save

# some changes in the working dir

# and now add them to the last commit:
git add -u
git commit --amend

# back to work!
git stash pop

```

## Cache credentials

```
git config credential.helper 'cache --timeout=10000000'

###########################################
# apply this setting globally on the host #
###########################################
git config --global credential.helper 'cache --timeout=10000000'
```

To remove config parameter

```
git config -–unset --global credential.helper
```

## Push to the same branch

```
git config --global push.default matching
```

## Get branch name by commit id

```
git branch --containg <commit id|tag>
```



## Token authentication

1. Generate new token [https://github.com/settings/tokens](https://github.com/settings/tokens)
2. Clone repository
3. Change remote-url using following git command

```
git remote set-url origin https://ivan.kuchin:ghp_xxxxxx@github.com/IvanKuchin/XXXXX.git
```

## GitHub Container Registry token authentication

```
docker login ghcr.io -u ivankuchin

ghp___GHCR_TOKEN___
```

## Specify an SSH key for git push for a given domain

{% embed url="https://stackoverflow.com/questions/7927750/specify-an-ssh-key-for-git-push-for-a-given-domain" %}

## Remove sensitive info from previous commits

Search for sensitive info

```
git log --all -S__password__
```

To remove commits found at a prev step -> run interactive rebase

```
git rebase -i commit_id_before_sensitive
```

Put a keyword `drop` against requested commit

Git will rebase all later commits and remove the affected one

## Clear git cache

Use it when:

1. You want to untrack a lot of files, or
2. You updated your _.gitignore_ file

```
git rm -r --cached .
```

* [rm](https://git-scm.com/docs/git-rm) is the remove command
* **-r** will allow recursive removal
* **–cached** will only remove files from the index. Your files will still be there.

The `rm` command can be unforgiving. If you wish to try what it does beforehand, add the `-n` or `--dry-run` flag to test things out.

```
git add .
git commit -m "commit after cache clean-up"
```
