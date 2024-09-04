# Git Branching

Unlike many other VCSs, Git encourages workflows that branch and merge often, even multiple times in a day.

## Branches in a Nutshell

Git doesn't store data as a series of changesets or differences, but instead as a series of **snapshots**.

When you make a commit, Git stores a commit object that contains a pointer to the snapshot of the content you staged.
- This object contains the author's name, email address, the commit message, pointer(s) to the commit(s) that directly came before this commit (its parent or parents): 0 parent for the initial commit, one parent for a normal commit, multiple parents for a commit that results from a merge of two or more branches.

Staging files computes a checksum for each one, stores that version of the file in the Git repository (Git refers to them as **blobs), and adds that checksum to the staging area
- When you create a commit by `git commit`, Git checksums each subdirectory and and stores them as a tree object in the Git repository.
- Git then creates a commit object that has the metadata and a pointer to the root project tree so it can re-create that snapshot when needed.
- You Git repository now contains five objects **Figure 9 - A commit and its tree**
  - three **blobs** (each representing the contents of one of the three files),
  - one **tree** that lists the contents of the directory and specifies which file names are stored as which blobs, and
  - one **commit** with the pointer to that root tree and all the commit metadta
- If you make some chagnes and commit again, the next commit stores a pointer to the commit that came immediately before it. **Figure 10 - Commits and their parents**


A branch in Git is simply a lightweight movable pointer to one of these commits.
The default branch name in Git is `master`.
If you are on `master` branch, every time you commit, the `master` branch pointer moves forward automatically.

**Figure 11 - A branch and its commit history**

### Creating a New Branch

Creating a new branch will give you a new pointer for you to move around.
`git branch <new-branch>` creates a new pointer to the same commit you're currently on.
But it doesn't switch to that branch.
```bash
git branch testing
```
**Figure 12 - Two branches pointing into the same series of commits.**

How does Git know what branch you’re currently on? It keeps a special pointer called `HEAD`.
In Git, this is a pointer to the local branch you’re currently on.
In this case, you’re still on `master`.
The `git branch` command only created a new branch — it didn’t switch to that branch.
You can easily see this by running a simple `git log` command that shows you where the branch points are pointing with `--decorate` option:
```bash
$ git log --oneline --decorate
f30ab (HEAD -> master, testing) Add feature #32 - ability to add new formats to the
central interface
34ac2 Fix bug #1328 - stack overflow under certain conditions
98ca9 Initial commit
```
**Figure 13 - HEAD pointing to a branch**

### Switching Branches

`git checkout testing`: switch to an existing branch by moving `HEAD` to point to the `testing` branch.
**Figure 14 - HEAD points to the current branch**

If you do some changes and commit now, your `testing` branch will move forward, but your `master` branch will still points to the commit you were on whe nyou ran `git checkout`
to switch branches.
**Figure 15 - The HEAD branch moves forward when a commit is made**

Let's switch back to the `master` branch:
```bash
$ git checkout master
```
**Figure 16 - HEAD moves when you checkout**

By default `git log` will only show commit history below the branch you've checked out.
To show commit history for the desired branch you need to explicitly specify it:
- `git log testing`, or
- `git log --all` to show all of the branches
- `git log --oneline --decorate --graph --all`: prints out the history of your commits, showing where your branch pointers are and how your history has diverged.
  ```bash
  $ git log --oneline --decorate --graph --all
  * c2b9e (HEAD, master) Make other changes
  | * 87ab2 (testing) Make a change
  |/
  * f30ab Add feature #32 - ability to add new formats to the central interface
  * 34ac2 Fix bug #1328 - stack overflow under certain conditions
  * 98ca9 Initial commit of my project
  ```

The command did two things:
1. it moved the `HEAD` pointer back to point to the `master` branch, and
2. it reverted the files in your working directory back to the snapshot that `master` points to.
   - It essentially rewinds the work you've done in your `testing` branch so you can go in a different direction

It's important to note that when you switch branches in Git, files in your working directory will change.
If Git cannot switch cleanly, it won't let you switch at all.

Now make some changes and commit again.
You can see your project history has diverged.
**Figure 17 - Divergent history**

A branch in Git is actually a simple file that contains the 40 character SHA-1 checksum of the commit it points to.

Create a new branch and switch to that new branch at the same time:
- `git checkout -b <new-branch-name>`: use `HEAD` by default
- `git checkout <commit/branch/tag> -b <new-branch-name>`: use a different commit and switch to a new branch there

`git switch` from Git version 2.23:
- `git switch <branch-name>`: switch to an existing branch
- `git switch -c/--create <new-branch>`: create a new branch and switch to it
- `git switch -`: return to your previously checked out branch

## Basic Branching and Merging

Let's go through a simple example of branching and merging with a workflow that you might use in the real world:
1. Do some work on a website
2. Create a branch for a new user story you're working on
3. Do some work in that branch

At this stage, you receive a call that another issue is critical and you need a hotfix
1. Switch to your production branch
2. Create a branch to add the hotfix
3. After it's tested, merge the hotfix branch, and push to production
4. Switch back to your original user story and continue working

### Basic Branching

At first, you are working on `master` and already have a couple of commits.

**Figure 18 - A simple commit history**

You've decided that you're going to work on issue #53.
You create a new branch and switch to it at the same time.
```bash
$ git checkout -b iss53
Switched to a new branch "iss53"
```
**Figure 19 - Creating a new branch pointer**

You work on your website and do some commits.
Doing so moves the `iss53` branch forward, because you've checked it out (that is, your `HEAD` is pointing to it).
```bash
$ touch index.html && git add index.html  #  You work on the issue
$ git commit -a -m "Create new footer [issue 53]"
```
**Figure 20 - The `iss53` branch has moved forward with your work**

Now you get the call that there is an issue with the website, and you need to fix it immediately.
You need to switch back to your `master` branch.

Note that if your working directory or staging area has uncommitted changes that conflict with the branch you're checking out, Git won't let you switch branches.
It's best to have a clean working state when you switch branches.
There are ways to get around this (stashing and commit amending) that we'll cover later on.
```bash
$ git checkout master
Switched to branch 'master'
```

This is an important point to remember: when you switch branches, Git resets your working directory to look like it did the last time you committed on that branch.
Next, you have a hotfix to make.
Let's create a `hotfix` branch on which to work until it's completed.
```bash
$ git checkout -b hotfix
Switched to a new branch 'hotfix'
$ touch fix.html && git add fix.html
$ git commit -a -m "Fix broken email address"
[hotfix 1fb7853] Fix broken email address
1 file changed, 2 insertions(+)
```
**Figure 21 - `hotfix` branch based on `master`**

After testing, now you want to merge the `hotfix` branch back into your `master` branch to deploy to production.
```bash
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c
Fast-forward
 index.html | 2 ++
 1 file changed, 2 insertions(+)
```
You merge the branch `hotfix` in with branch `master`.
When you try to merge one commit with a commit that can be reached by following the first commit's history,
Git simplifies things by moving the pointer forward because there is no divergent work to merge together - this is called a **fast-forward**.
Your change is now in the snapshot of the commit pointed to by the `master` branch, and you can deploy the fix.
**Figure 22 - `master` is fast-forwarded to `hotfix`**

After your super-important fix is deployed, you're ready to switch back to the work you were doing before you were interrupted.
You can first delete the `hotfix` branch, because you no longer need it.
```bash
$ git branch -d hotfix  #  Delete the `hotfix` branch
Deleted branch hotfix (3a0874c).
```

Now you can switch back to your work-in-progress branch on issue #53 and continue working on it.
```bash
$ git checkout iss53
Switched to branch "iss53"
$ echo "something" >> index.html
$ git commit -a -m "Finish the new footer [issue 53]"
[iss53 ad82d7a] Finish the new footer [issue 53]
1 file changed, 1 insertion(+)
```
**Figure 23 - Work continues on `iss53`**

If you need to pull in the hotfix in the `master` branch, you can merge `master` branch into `iss53`
Or, you can wait to integrate those changes until you decide to pull the `iss53` branch back into `master` later

### Basic Merging

Suppose you've decided that your issue #53 work is complete and ready to be merged into `master`.
**Figure 24 - Three snapshots used in a typical merge**

All you have to do is check out the branch you wish to **merge into** and then run the `git merge` command.
```bash
$ git checkout master
Switched to branch 'master'
$ git merge iss53 
Merge made by the 'recursive' strategy.
index.html |
1 +
1 file changed, 1 insertion(+)
```
Because the commit on the branch you're on isn't a direct ancestor of the branch you're merging in, Git has to do some work.
In this case, Git does a simple **three-way merge**, using the two snapshots pointed to by the branch tips and the common ancestor of the two two.
Instead of just moving the branch pointer forward, Git creates a new snapshot that results from this three-way merge and automatically creates a new commit that points to it.
This is referred to as a **merge commit**, and is special in that it has more than one parent.
**Figure 25 - A merge commit**

Now that your work is merged in, you have no further need for the `iss53` branch.
You can close the issue in your issue-tracking system, and delete the branch:
```bash
$ git branch -d iss53
```

### Basic Merge Conflicts

If you changed the same part of the same file differently in the two branches you're merging, Git won't be able to merge them cleanly.
You might get a merge conflict when you run `git merge iss53`:
```bash
 $ git merge iss53
  Auto-merging index.html
  CONFLICT (content): Merge conflict in index.html
  Automatic merge failed; fix conflicts and then commit the result.
```
In this case, Git can't automatically create a new merge commit.
It will pause the process while you resolve the conflict.

You can see which files are unmerged at any point after a merge conflict: `git status`.
```bash
$ git status
  On branch master
  You have unmerged paths.
    (fix conflicts and run "git commit")
  Unmerged paths:
    (use "git add <file>..." to mark resolution)
      both modified:      index.html
  no changes added to commit (use "git add" and/or "git commit -a")
```

You need to resolve those conflicts annotated by standard **conflict-resolution markers**.
```text
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```
After you've resolved each of these sections in each conflicted file, run `git add` on each file to mark it as resolved.
Staging the file marks it as resolved in Git.

If you want to use a graphical tool to resolve these issues, you can run `git mergetool`
If you're happy with that, you can type `git commit` to finalize the merge commit.
if you think it would be helpful to others looking at this merge in the future, you can modify this commit message with details about how you resovled the merge and explain why you did the changes you made if these are not obvious.

## Branch Management

List branches
```bash
$ git branch  #  show branches
  iss53
* master    <- The * indicates the branch that you currently have checked out (the branch HEAD points to)
  testing

$ git branch -v   #  See the last commit on each branch
  iss53   93b412c Fix javascript issue
* master  7a98805 Merge branch 'iss53'
  testing 782fd34 Add scott to the author list in the readme

# `--merged` and `--no-merged` options can filter the list to branches that you have/have not yet merged into the branch you're on.
$ git branch --merged     # Branches on this list without the `*` in front of them are generally fine to delete with `git branch -d`
$ git branch --no-merged  # It contains work that isn't merged in yet, trying to delete it with `git branch -d` will fail
  testing
$ git branch -d testing
  error: The branch 'testing' is not fully merged.
  If you are sure you want to delete it, run 'git branch -D testing'.

$ git branch --no-merged master #  You can add an additional argument to ask about the merge state w.r.t. some other branch
                                #+ w/o checking that branch out first. If there is no branch specified, `git branch --merged/
                                #+ --no-merged` will show you what is merged/not merged into your current branch
```

### Changing a branch name

- DO NOT rename branches that are still in use by other collaborators.
- DO NOT rename a branch like `master/main/mainline` without having read the section "Changing the master branch name"

Suppose you have a branch that is called `bad-branch-name` and you want to change it to `corrected-branch-name`, while keeping all history.
You also want to change the branch name on the remote (GitHub, GitLab, other server).
How do you do this?

Rename the branch locally with:
```bash
$ git branch --move <bad-branch-name> corrected-branch-name>
```

To let others see the corrected branch on the remote, push it:
```bash
$ git push --set-upstream origin corrected-branch-name
$ git branch --all
* corrected-branch-name
  main
  remotes/origin/bad-branch-name
  remotes/origin/corrected-branch-name
  remotes/origin/main
```

Note that the branch with the bad name is also still present there but you can delete it by
```bash
$ git push origin --delete <bad-branch-name>
```

#### Changing the master branch name

Changing the name of a branch like `master/main/mainline/default` will break the integrations, services, helper utilities and build/release scripts that your repository uses.
Before you do this, make sure you consult with your collaborators.

After changing the `master/main/...` branch names as described before, you have a few more tasks in front of you to complete the transition: **Read the Book**.

## Branching Workflows

We'll cover some common workflows that this lightweight branching makes possible, so you can design your own development cycle.

### Long-Running Branches

Many Git developers have a workflow that
- having only code that is entirely stable in their **master** branch — possibly only code that has been or will be released.
- They have another parallel branch named **develop** or **next** that they work from or use to test stability — it isn’t necessarily always stable, but whenever it gets to a stable state, it can be merged into master.
  - It’s used to pull in **topic branches** (short-lived branches, like your earlier `iss53` branch) when they’re ready, to make sure they pass all the tests and don’t introduce bugs.

**Figure 26 - A linear view of progressive-stability branching**

**Figure 27 - A “silo” view of progressive-stability branching**

The idea is that your branches are at various levels of stability; when they reach a more stable level, they’re merged into the branch above them.
Again, having multiple long-running branches isn’t necessary, but it’s often helpful, especially when you’re dealing with very large or complex projects.

### Topic Branches

Topic branches, are useful in projects of any size.

A topic branch is a short-lived branch that you create and use for a single particular feature or related work.
You saw this in the last section with the `iss53` and `hotfix` branches.
You did a few commits onthem and deleted them directly after merging them into your main branch.

**Figure 28 - Multiple topic branches**

**Figure 29 - History after merging `dumbidea` and `iss91v2`**

We will go into more detail about the various possible workflows for your Git project in "Distributed Git", so before you decide which branching scheme your next project will use, be sure to read that chapter.

It's important to remember when you're doing all this that these branches are completely local.
When you're branching and merging, everything is being done only in your Git repository - there is no communication with the server.

## Remote Branches

Remote references
> References (pointers) in your remote repositories, including branches, tags, and so on.

You can list remote references with
```bash
$ git ls-remote <remote> # or
$ git remote show <remote>
* remote origin
  Fetch URL: git@github.com:EmperorPeggy/linux-journey.git
  Push  URL: git@github.com:EmperorPeggy/linux-journey.git
  HEAD branch: main
  Remote branch:
    main tracked
  Local branch configured for 'git pull':
    main merges with remote main
  Local ref configured for 'git push':
    main pushes to main (up to date)
```
for remote branches as well as more info.

Remote-tracking branches
> References to the state of remote branches.
> They are local references that you cannot move.
> Git moves them for you whenever you do any network communication, to make sure they accurately represent the state of the remote repository.

Remote-tracking branch names take the form `<remote>/<branch>`.
For example `origin/master`, `origin/iss53`.

Let's look at an example.
Let's say you have a Git server on your network at `git.ourcompany.com`.
- If you clone from this, Git's `clone` command automatically names it `origin` for you, pulls down all its data, creates a pointer to where its `master` branch is, and names it `origin/master` locally.
- Git also gives you your own local `master` branch starting at the same place as origin's `master` branch.
- **Figure 30 - Server and local repositories after cloning**
- If you do some work on your local `master` branch, and, in the meantime, someone else pushes to `git.ourcompany.com` and updates its `master` branch, then your histories move forward differently.
  - As long as you stay out of contact with your `origin` server, your `origin/master` pointer doesn't move.
- **Figure 31 - Local and remote work can diverge**
- To synchronize your work with a given remote, you run a `git fetch <remote>` command. This command looks up which server "origin" is, fetches any data from it that you don't yet have, and updates
  your local database, moving your `origin/master` pointer to its new, more up-to-date position.
- **Figure 32 - `git fetch` updates your remote-tracking branches**
- To demonstrate having multiple remote servers and what remote branches for those remote projects look like, let's assume you have another internal Git server that is used only for dev by one of your
  sprint teams: `git.team1.ourcompany.com`.
  - You can add it as a new remote reference to the project: `git remote add ...`
- **Figure 33 - Adding another server as a remote**
- `git fetch teamone` will fetch everything the remote `teamone` server has that you don't have yet.
  - Git sets a remote-tracking branch called `teamone/master` to point to the commit that `teamone` has as its `master` branch.
- **Figure 34 - Remote-tracking branch for `teamone/master`**

### Pushing

When you want to share a branch with the world, you need to push it up to a remote to which you have write access.
Your local branches aren't automatically synchronized to the remotes you write to - you have to explicitly push the branches you want to share.
- You can use private branches for work you don't want to share, and push up only the topic branches you want to collaborate on.

If you have a branch named `serverfix` that you want to work on with others, you can push it up the same way you pushed your first branch:
```bash
$ git push origin serverfix
Counting objects: 24, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (24/24), 1.91 KiB | 0 bytes/s, done.
Total 24 (delta 2), reused 0 (delta 0)
To https://github.com/schacon/simplegit
 * [new branch]       serverfix -> serverfix
```

- This is a bit of a shortcut.
  Git automatically expands the `serverfix` branchname out to `refs/heads/serverfix:refs/heads/serverfix`, which means,
  - "Take my `serverfix` local branch and push it to update the remote’s `serverfix` branch."
  We’ll go over the `refs/heads/` part in detail in **Git Internals**.
  - You can also do `git push origin serverfix:serverfix`, which does the same thing — it says, "Take my serverfix and make it the remote’s serverfix."
- If you didn’t want it to be called `serverfix` on the remote, you could instead run `git push origin serverfix:awesomebranch` to push your local `serverfix` branch to the `awesomebranch` branch on the remote project.

Don’t type your password every time
> If you’re using an HTTPS URL to push over, the Git server will ask you for your username and password for authentication.
> By default it will prompt you on the terminal for this info so the server can tell if you're allowed to push.
>
> If you don't want to type it every single time you push, you can set up a "credential cache".
> `git config --global credential.helper cache`

The next time one of your collaborators fetches from the server, they will get a reference to where the server’s version of `serverfix` is under the remote branch `origin/serverfix`
```bash
$ git fetch origin
remote: Counting objects: 7, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 3 (delta 0)
Unpacking objects: 100% (3/3), done.
From https://github.com/schacon/simplegit
 * [new branch]         serverfix -> origin/serverfix
```

It’s important to note that when you do a fetch that brings down new remote-tracking branches, you don’t automatically have local, editable copies of them.
In other words, in this case, you don’t have a new `serverfix` branch — you have only an `origin/serverfix` pointer that you can’t modify.

To merge this work into your current working branch, you can run `git merge origin/serverfix`.
If you want your own `serverfix` branch that you can work on, you can **base it off your remote tracking branch**:
```bash
$ git checkout -b serverfix origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```

### Tracking Branches

Checking out a local branch from a remote-tracking branch automatically creates a **tracking branch** (and the branch it tracks is called an **upstream branch**).
Tracking branches are local branches that have a direct relationship to a remote branch.
If you're on a tracking branch and type `git pull`, Git automatically knows which server to fetch from and which branch to merge in.

When you clone a repository, it generally automatically creates a `master` branch that tracks `origin/master`.
However, you can set up other tracking branches if you wish — ones that track branches on other remotes, or don’t track the `master` branch.
The simple case is the example you just saw, running `git checkout -b <branch> <remote>/<branch>`.

This is a common enough operation that Git provides the `--track` shorthand:
```bash
$ git checkout --track origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```

In fact, this is so common that there’s even a shortcut for that shortcut.
If the branch name you're trying to checkout (a) doesn’t exist and (b) exactly matches a name on only one remote, Git will create a tracking branch for you:
```bash
$ git checkout serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```

To set up a local branch with a different name than the remote branch, you can easily use the first version with a different local branch name:
```bash
$ git checkout -b sf origin/serverfix
Branch sf set up to track remote branch serverfix from origin.
Switched to a new branch 'sf'
```

If you already have a local branch and want to set it to a remote branch you just pulled down,
or want to change the upstream branch you’re tracking, you can use the `-u` or `--set-upstream-to` option to git branch to explicitly set it at any time.
```bash
$ git branch -u origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
```

Upstream shorthand
> When you have a tracking branch set up, you can reference its upstream branch with the `@{upstream}` or `@{u}` shorthand.
> So if you’re on the `master` branch and it’s tracking `origin/master`, you can say something like `git merge @{u}` instead of `git merge origin/master` if you wish.

If you want to see what tracking branches you have set up, you can use the -vv option to git branch.
This will list out your local branches with more information including what each branch is tracking
and if your local branch is ahead, behind or both.
```bash
$ git branch -vv
  iss53 7e424c3 [origin/iss53: ahead 2] Add forgotten brackets
  master 1ae2a45 [origin/master] Deploy index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] This should do it
  testing 5ea463a Try something new
```

So here we can see that our iss53 branch is tracking origin/iss53 and is “ahead” by two, meaning
that we have two commits locally that are not pushed to the server. We can also see that our master
branch is tracking origin/master and is up to date. Next we can see that our serverfix branch is
tracking the server-fix-good branch on our teamone server and is ahead by three and behind by one,
meaning that there is one commit on the server we haven’t merged in yet and three commits
locally that we haven’t pushed. Finally we can see that our testing branch is not tracking any
remote branch.

It’s important to note that these numbers are only since the last time you fetched from each server.
If you want totally up to date ahead and behind numbers, you’ll need to fetch from
all your remotes right before running this. You could do that like this:
```bash
$ git fetch --all; git branch -vv
```

### Pulling

While the `git fetch` command will fetch all the changes on the server that you don’t have yet, it will not modify your working directory at all.
It will simply get the data for you and let you merge it yourself.
However, there is a command called `git pull` which is essentially a `git fetch` immediately followed by a `git merge` in most cases.

`git pull` will look up what server and branch your current branch is tracking, fetch from that server and then try to merge in that remote branch.

Generally it’s better to simply use the `fetch` and `merge` commands explicitly as the magic of `git pull` can often be confusing.

### Deleting Remote Branches

Suppose you're done with a remote branch — say you and your collaborators are finished with a feature and have merged it into your remote’s `master` branch (or whatever branch your stable codeline is in).
You can delete a remote branch using the `--delete` option to git push.

```bash
$ git push origin --delete serverfix
To https://github.com/schacon/simplegit
 - [deleted] serverfix
```

## Rebasing

In Git, there are two main ways to integrate changes from one branch into another: the `merge` and the `rebase`.

### The Basic Rebase

**Figure 35 - Simple divergent history**

- The `merge` command is the easiest way to integrate the branches
  - It performs a 3-way merge between the two latest branch snapshots (C3 and C4) and the most recent common ancestor of the two (C2), creating a new snapshot (and commit).
  - **Figure 36 - Merging to integrate diverged work history**
- However, with the `rebase` command, you can take all the changes that were committed on one branch and replay them on a different branch.
  - For this example, you would check out the `experiment` branch, and then rebase it onto the `master` branch as follows:
    ```bash
	$ git checkout experiment
	$ git rebase master
	First, rewinding head to replay your work on top of it...
	Applying: added staged command
	```
  - **How does `rebase` work behind the scene?**
    > This operation works by going to the common ancestor of the two branches (the one you’re on and the one you’re rebasing onto), getting the diff introduced by each commit of the branch you’re on,
    > saving those diffs to temporary files, resetting the current branch to the same commit as the branch you are rebasing onto, and finally applying each change in turn.
  - **Figure 37 - Rebasing the change introduce in `C4` onto `C3`**
  - At this point, you can go back to the `master` branch and do a fast-forward merge.
	```bash
	$ git checkout master
	$ git merge experiment
	```
  - **Figure 38 - Fast-forwarding the `master` branch**

There is no difference in the end product of the integration realized by `merge` or `rebase`, but rebasing makes for a cleaner history.
If you examine the log of a rebased branch, it looks like a linear history: it appears that all the work happened in series, even when it originally happened in parallel.

Often, you’ll do this to **make sure your commits apply cleanly on a remote branch** — perhaps in a project to which you’re trying to contribute but that you don’t maintain.
In this case, you'd do your work in a branch and then rebase your work onto origin/master when you were ready to submit your patches to the main project.
That way, the maintainer doesn’t have to do any integration work — just a fast-forward or a clean apply.

Rebasing replays changes from one line of work onto another in the order they were introduced, whereas merging takes the endpoints and merges them together.

### More Interesting Rebases

You can also have your rebase replay on something other than the rebase target branch.

For example, take a history like **Figure 39 - A history with a topic branch off another topic branch**,
- First, you branched a topic branch `server` to add some server-side functionality to your project, and made a commit.
- Then, you branched off that to make the client-side changes `client` and committed a few times.
- Finally, you went back to your `server`branch and did a few more commits.

Suppose you decide that you want to merge your client-side changes into mainline for a release, but you want to hold off on the server-side changes until it's tested further.
You can take the chagnes on `client` that aren't on `server` (`C8` and `C9`) and replay them on your `master` branch by using the `--onto` option of `git rebase`:
```bash
$ git rebase --onto master server client
```
This basically says, "take the `client` branch, figure out the patches since it diverged from the `server` branch, and replay these patches in the `client` branch as if it was based directly off the master branch instead".
- **Figure 40 - Rebasing a topic branch off another topic branch**
- Now you can fast-forward your `master` branch.
  ```bash
  $ git checkout master
  $ git merge client
  ```
- **Figure 41 - Fast-forwarding your `master` branch to include the `client` branch changes**

Let's say you decide to pull in your `server` branch as well.
You can rebase the `server` branch onto the `master` branch w/o having to check it out first by running `git rebase <base-branch> <topic-branch>`
```bash
$ git rebase master server
```
- **Figure 42 - Rebasing your `server` branch on top of your `master` branch**
- Then, run `$ git checkout master && git merge server` to fast-forward the base branch
- You can remove the `client` and `server` branches because all the work is integrated
  ``bash
  $ git branch -d client
  $ git branch -d server
  ```
- **Figure 43 - Final commit history**

### The Perils of Rebasing

Do not rebase commits that exist outside your repository and that people may have based work on.
- Suppose you have a commit history looks like **Figure 44 - Clone a repository, and base some work on it**
- Now, someone else does more work that includes a merge, and pushes that work to the central server.
  You fetch it and merge the new remote branch into your work, making your history look something like this:
  - **Figure 45 - Fetch more commits, and merge them into your work**
- Next the person who pushed the merged work decides to go back and rebase their work instead;
  they do a `git push --force` to overwrite the hisotry on the server.
  You then fetch from that server, bringing down the new commits.
  - **Figure 46 - Someone pushes rebased commits, abandoning commits you've based your work on**
- Now you're both in a pickle.
  - If you do a `git pull`, you'll create a merge commit which includes both lines of history, and your repository will look like this:
  - **Figure 47 - You merge in the same work again into a new merge commit**
  - If you run a `git log` when your history looks like this, you'll see two commits that have the same author, date, and message, which will be confusing.
  - Furthermore, if you push this history back up to the server, you'll reintroduce all those rebased commits to the central server, which can further confuse people.

### Rebase When You Rebase

If you **DO** find yourself in a situation like this, Git has some further magic that might help you out.

TODO: dig deeper into the book.

### Rebase vs. Merge

Which one is better? `rebase` or `merge`?
> One point of view on this is that your repository’s commit history is a **record of what actually happened**.
> It’s a historical document, valuable in its own right, and shouldn’t be tampered with.
> From this angle, changing the commit history is almost blasphemous; you’re lying about what actually transpired.
>
> The opposing point of view is that the commit history is the **story of how your project was made**.
> When you’re working on a project, you may need a record of all your missteps and dead-end paths,
> but when it’s time to show your work to the world, you may want to tell a more coherent story of how to get from A to B.
> People in this camp use tools like `rebase` and `filter-branch` to rewrite their commits before they’re merged into the mainline branch.
> They use tools like `rebase` and `filter-branch`, to tell the story in the way that's best for future readers.

You can get the best of both worlds: rebase local changes before pushing to clean up your work, but never rebase anything that you’ve pushed somewhere.

## Summary

We've covered basic branching and merging in Git.
You should feel comfortable
- creating and switching to new branches,
- switching between branches and
- merging local branches together.

You should also be able to
- share your branches by pushing them to a shared server, working with others on shared branches and rebasing your branches before they are shared.

Next, we'll cover what you'll need to run your own Git repository-hosting server.
