# Git Best Practices and Recipes

## CRLF

Git has the ability to auto-convert CRLF to a single LF character. My policy is to leave these alone. When you edit a file, just accept which line-ending it already had. When adding to a project, just accept which line-ending it already had. Most editors are smart enough to keep line-endings consistent within a file.

If you're starting a new project, stick with the standard used by other Ident projects you'll importing.

## Creating branches

Branch naming scheme is as follows:

bugfix/…

feature/…

task/…

hotfix/…

To branch, do one of the following:

```sh
git branch feature/123
git checkout feature/123
```

Or the quicker:

```sh
git checkout -b feature/123
```

## Pull requests

For security practices, no direct commits to the main branch should be allowed. All commits should go thru a PR. Ideally, only tech leads should be able to bypass a PR review. But even in the case of an emergency, still create a PR with reviewers so that the code can be reviewed post-merge.

All branches should be squashed and closed on the merge of a pull request.

Ideally, the git bare repo should only show the primary branches as straight lines. The only other branches that should be on remote should be those that are awaiting PR merges.

Consequently, your local repo will become quite cluttered after a while. Take mind to clean them up.

### Cleaning stale remote branches

To clean up stale references to remote branches:

```sh
git remote prune origin
```

Or, you can prune all stale remote references automatically with your fetch or pull using the -p flag:

```sh
git fetch -p
```

Or, you can configure git globally to always prune stale remote references:

```sh
git config --global remote.origin.prune true
```

### Cleaning stale local branches

After cleaning remote branches, its a good idea to clean up your local repo to keep your tree straight. Use git branch to list all local branches.

However, after a PR, your local branch will show up as unmerged (since it was merged, squashed, and deleted by Bitbucket). Since it's a pain to root these out, I use a script to remove these stale local branches. See [codeturner/dotfiles#git](https://github.com/codeturner/dotfiles#git).

## Dangerous commands

In general, once a commit has been pushed up into the main repo, consider it to be set in stone. Just avoid force pushes altogether - unless the team decides cleanup needs to occur. Most everything can be accomplished with additional commits and that's the approach that should be taken.

## Fetch vs Pull?

Pull is a shortcut for fetching and merging on your current branch, and is one most probably use. So why would you want to opt for the fetch instead?

Downsides of a pull:

Pull only fetches the current branch you are on.

Pull makes early the decision to merge vs rebase before seeing what's upstream.

If you can live with these caveats, then just use pull cuz it's shorter. Otherwise, consider separate commands to accomplish what you want.

```sh
# fetch ALL upstream changes
$ git fetch

# what did I get?
$ git l

# hmmm, I made a few commits to my branch off of main
# and it appears that someone else committed small changes to main since then
# let's rebase
$ git rebase origin/main
```

### Fast forward fetch without checkout

Prior to pushing up changes, you should fetch the latest changes from the root branch in order to merge those in with yours. The typical way is often a pain in switching back and forth between branches:

```sh
# fetch ALL upstream changes
$ git fetch

# what did I get?
$ git l

# oh, new changes from main!
# let's switch branches and sync up
$ git co main
$ git merge

# ok, back to my working branch, and merge in upstream changes
$ git co feature/123
$ git merge main
```

However, if the fetch is just a simple fast forward, here's a shortcut to all those steps:

```sh
# fetch ALL upstream changes
$ git fetch

# what did I get?
$ git l

# oh, new changes from main!
# let's sync up just that branch without switching branches
# using the syntax: git fetch <remote> <sourceBranch>:<destinationBranch>
$ git fetch origin main:main

# ok, now merge in upstream changes to my working branch
$ git merge main
```

## Merge vs Rebase?

Both accomplish the same thing, which is to join your changes with the changes others made on the same branch. Merge is the safest, and is the default action for git pull. But rebase has power, and you should know what's up.

Let's say you're working on a new branch feature/add-stuff off of main and made a commit. Now, you're ready to push to origin, but first(!) you need to pull from origin to join with any other commits made.

```sh
# original state
$ git l
* faae761 (HEAD -> main, origin/main) Merge task/one - Greg, 2 hours ago

# commit to feature/add-stuff yields new state
$ git l
* cf21e18 (HEAD -> feature/add-stuff) file2.line2 - Greg, 4 minutes ago
* faae761 (main, origin/main) Merge task/one - Greg, 2 hours ago

# fetch upstream changes
$ git fetch
$ git l
* cf21e18 (HEAD -> feature/add-stuff) file2.line2 - Greg, 2 seconds ago
| * 7787619 (origin/main) file1:line2 - Bob, 15 minutes ago
|/
* faae761 (main) Merge task/one - Greg, 2 hours ago
```

### Option 1: Merge

Here's the results of merge in this state:

```sh
$ git merge origin/main
$ git l
*   0b6c898 (HEAD -> feature/add-stuff) Merge remote-tracking branch 'origin/main' into feature/add-stuff - Greg, 5 seconds ago
|\
| * 7787619 (origin/main) file1:line2 - Bob, 17 minutes ago
* | cf21e18 file2.line2 - Greg, 2 minutes ago
|/
* faae761 (main) Merge task/one - Greg, 2 hours ago
```

Note how it creates another commit to join those commits together. Nothing wrong with that. You can use this all day long.

### Option2: Rebase

But, what if I don't expect any merge conflicts and I just want to basically fast forward my changes to those made by Bob. That's where rebase comes in:

```sh
$ git rebase origin/main
$ git l
* 467ea70 (HEAD -> feature/add-stuff) file2.line2 - Greg, 6 minutes ago
* 7787619 (origin/main) file1:line2 - Bob, 21 minutes ago
* faae761 (main) Merge task/one - Greg, 2 hours ago
```

What happened? Notice that it basically popped your commit at cf21e18 from the stack and recreated it at a new branch point, now under the commit of 467ea70. Well, that is quite beautiful.

But is this all necessary? Remember, rebasing changes history, and that always introduces more danger. Also, considering that our PR branches are squashed and deleted upon merge, if your goal is to keep the tree clean, then this is definitely not required for that end in mind.

> **Note**: If you decide to adopt rebase into your processes, please remember that if you've pushed a commit to origin, always always always use merge instead of rebase.
>
> Rebase changes history, which if other devs have pulled a commit that you then decided to delete, they will curse your name.

## Oops, I just committed to the wrong branch

You made some changes, committed them, then realized "I forgot to branch first!" and you have commits directly on the main branch. Happens to everyone. Here's the easiest way to get this right.

```sh
# view commits
$ git l
* 876b954 (HEAD -> main) file4 - Greg, 3 minutes ago
* faae761 (origin/main) Merge task/one - Greg, 67 minutes ago

# note that the last commit should have happened on the feature/add-stuff branch
# so let's create that branch at the current spot
$ git b feature/add-stuff

# now, what happened?
$ git l
* 876b954 (HEAD -> main, feature/add-stuff) file4 - Greg, 6 minutes ago
* faae761 (origin/main) Merge task/one - Greg, 70 minutes ago

# good, changes will be saved at that new branch
# now, we need to rewind the main branch to origin/main
# note that HEAD is pointing to main (if not, you must check it out before proceeding)
$ git reset --hard origin/main

# now, what happened?
$ git l
* 876b954 (feature/add-stuff) file4 - Greg, 8 minutes ago
* faae761 (HEAD -> main, origin/main) Merge task/one - Greg, 72 minutes ago

# cool!
# now, checkout your feature branch and continue
$ git co feature/add-stuff
```

## Empty commit

Sometimes it is helpful to make a commit with no changes for the pure sake of documenting a point in time. Maybe not. But here's the recipe:

```sh
git commit --allow-empty -m "graft point"
```

## Fix last commit message

If you committed but didn't quite get the commit message right:

```sh
# to use the full editor
$ git commit --amend

# or, to just replace the full comment inline
$ git commit --amend -m "new commit message goes here"
```

Normally, amend commits should be shunned, but editing comments is pretty safe.

## Handling large diffs in PRs

Diffs on large PRs are often difficult to see in your PR review. But don't let that stop you. Bring those changes to your workstation and diff them locally.

Here's the process to pull that off.

First, you need to decide where to view these changes. You'll need a clean repo to view someone else's committed change diffs. If you have a no pending changes in your local workspace for that project, then you can just view these differences in your preferred IDE - VS Code has a decent file differ, Intellij is even better.

However, if you are in the middle of changes in your local workspace for the same project, you either need to 1) commit or stash those changes (not optimal), or 2) clone up another repo just for diffing.

Once you've found your repo to view these changes, checkout the changes that are up for PR.

Let's say a PR was pushed up for feature/add-stuff

```sh
$ git fetch
$ git l
* 1a1cea3 (origin/feature/add-stuff) file2.line3 - Greg, 2 minutes ago
* 3655920 file2.line2 - Greg, 3 minutes ago
* 7787619 (HEAD -> main, origin/main, origin/HEAD) file1:line2 - Greg, 2 hours ago
```

First, you'll need to checkout their commit, then work back from there to rewind their changes to reproduce the PR changeset:

```sh
$ git co feature/add-stuff
$ git l
* 1a1cea3 (HEAD -> feature/add-stuff, origin/feature/add-stuff) file2.line3 - Greg, 2 minutes ago
* 3655920 file2.line2 - Greg, 3 minutes ago
* 7787619 (origin/main, main) file1:line2 - Greg, 2 hours ago

# now, do a soft reset to point the index at main, but keeping changes
# from the changeset 1a1cea3 and 3655920 as staged
$ git reset --soft origin/main
* 1a1cea3 (origin/feature/add-stuff) file2.line3 - Greg, 2 minutes ago
* 3655920 file2.line2 - Greg, 3 minutes ago
* 7787619 (HEAD -> feature/add-stuff, origin/main, origin/HEAD, main) file1:line2 - Greg, 2 hours ago
```

Note that the following soft resets would have resulted in the same, since in this case, they all would point to the same commit:

```sh
$ git reset --soft origin/main
$ git reset --soft main
$ git reset --soft 7787619
$ git reset --soft HEAD~2

# or, sometimes its handy to jump back one commit at a time
# especially if you have an alias setup to jump back one
$ git reset --soft HEAD~1
$ git reset --soft HEAD~1
```

Once the code is in place, you can view the diffs using the git difftool of your choice. Intellij offers the best file differencing tool. VSCode is next best, allowing you to open this directory without any indexing overhead, which is handy if you're viewing this in a working folder just for diffing.

After you're done, undo your reset with a hard reset to the origin branch and your repo is back to normal:

```sh
git reset --hard origin/feature/add-stuff
```
