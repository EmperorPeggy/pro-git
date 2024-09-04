## Getting a Git Repository

You can obtain a Git repository in one of two ways:
1. take a local directory that is currently not under version control, and turn it into a Git repository, or
2. _clone_ an existing Git repository from elsewhere

### Initializing a Repository in an Existing Directory

`git init`:
- creates a new subdirectory named `.git` that contains all of your necessary repository files - a Git repository skeleton.
- At this point, nothing in your project is tracked yet.
- See _Git Internals_ for more information about exactly what files are contained in the `.git` directory you just created.

If you want to begin tracking existing/new files and do an initial commit
```bash
$ git add *.c
$ git add LICENSE
$ git commit -m "Initial project version"
```

### Clone an Existing Repository

The following command:
1. create a directory named `libgit2`
2. initializes a `.git` directory inside it
3. pulls down all the data for that repository and
4. checks out a working copy of the latest version
```bash
git clone https://github.com/libgit2/libgit2
```

You can also clone the repository into a directory named somthing else
```bash
git clone https://github.com/libgit2/libgit2 mylibgit
```

## Recording Changes to the Repository

A **checkout** (or a **working copy**) of all of the Git repository is in front of you after cloning.

Each file in your working directory can be in one of 2 states:
1. **tracked**, meaning they were in the last snapshot, as well as any newly staged files.
Tracked files are files that Git knows about and they can be
  - unmodified
  - modified, or
  - staged
2. **untracked**: any file in your working directory that were not in your last snapshot and are not in your staging area

### Checking the Status of Your Files

`git status`: check status of your files; shows
- which branch you are on
- untracked / modified / staged files

If you run this command directly after a clone, you will see:
```bash
$ git status
On branch master   <- The current branch you are on
your branch is up-to-date with 'origin/master'.  <- It has not diverged from the same branch on the server
nothing to commit, working tree clean  <- The status of working tree. Clean working directory: none of your tracked files are modified. Git also doesn't see any untracked files
```

### Tracking New Files & Staging Modified Files

You use the command `git add <path>` for multiple purposes.
It may be helpful to think of it more as "add precisely this content to the next commit" rather than "add this file to the project".
- if `<path>` is not tracked, `git add` will begin to track it and stage it
- if `<path>` is being tracked, `git add` will stage it
- do other things like **marking merge-conflicted files as resolved**

### Short Status

`git status -s/--short`: concise status
- first column: status of the staging area; `A`: newly tracked file, `M`: modified and staged
- second column: status of the working tree; `M`: modified in the working tree but not yet staged
- `??`: untracked files

```bash
$ git status -s
 M README            <- modified but not yet staged
MM Rakefile          <- modified and staged, then modified again
A lib/git.rb         <- newly tracked file
M lib/simplegit.rb   <- modified and staged
?? LICENSE.txt       <- untracked file
```

TODO: examine the following file status transition methods
Modify working tree / staged area; File status transition
- `git add <path>`: untracked/modified -> tracked/staged; track new files or add file to staging area
- `git restore --staged/--cached <file>`: remove `<file>` from staging area, it has no influence on the content of the working tree
- `git reset @ <file> ...`: ??
- `git restore <file>`: modified -> staged/committed; discard changes in the working dir. File content is set to the content in the staging area or last commit.
- `git checkout -- <file>`: the same as the above one

### Ignoring Files

A class of files that you don't want Git to automatically add or even show you as being untracked.
In such cases, you can create a file listing patterns to match them named `.gitignore`.
Setting up a `.gitignore` file for your new repository before you get going is generally a good idea so you don't accidentally commit files that you really don't want in your Git repository.

The rules for the patterns:
- Blank lines or lines starting with `#` are ignored
- Standard glob patterns work, and will be applied **recursively** throughout the entire working tree
  - Glob patterns are like simplified regular expressions that shells use.
    - An asterisk `*` matches 0 or more characters
    - `[abc]` matches any character inside the brackets
    - A question mark `?` matches a single character
    - Brackets enclosing characters separated by a hyphen `[0-9]` matches any character between them
    - Two asterisks match nested directories; e.g., `a/**/z` would math `a/z`, `a/b/z`, `a/b/c/z`, etc.
- You can start patterns with a forward slash `/` to avoid recursivity
- You can end patterns with a forward slash `/` to specify a directory
- You can negate a pattern by starting it with an exclamation point `!`

Github maintains a [fairly comprehensive list](https://github.com/github/gitignore) of good `.gitignore` file examples.
In the simple case, a repository might have a single `.gitignore` file in its root directory, which applies recursively to the entire repository.
However, it's also possible to have additional `.gitignore` files in subdirectories.
See `man gitignore` for details.

### Viewing Your Staged and Unstaged Changes

We use `git diff` to answer these two questions most often:
1. what have you changed but not yet staged?
	- `git diff` compares what is in your working directory with what is in your staging area.
      The result tells you the changes you've made that you haven't yet staged.
2. what have you staged that you are about to commit?
    - `git diff --staged/--cached` compares your staged changes to your last commit

`git diff --stat` shows the statistics of the diff instead of the code change.

### Committing Your Changes

`git commit`: you can see that the commit has given you some output about itself: which branch you committed to, what SHA-1 checksum the commit has, etc.
- `git commit -v`: puts the diff of your change in the editor so you can see exactly what changes you're committing
- `git commit -m "<commit-message>"`: inline commit message
- `git commit -a -m "<commit-message>"`: makes Git automatically stage every file that is already tracked before doing the commit, letting you skip the `git add` part

### Removing Files

To remove a file from Git, you need to remove it from your tracked files (more accurately, remove it from your staging area) and then commit.
`git rm <path>` does that, and also removes the file from your working directory so you don't see it as an untracked file the next time around.
So it basically equals to `rm <path>` followed by `git add/rm <path>`.

If you modified the file or had already added it to the staging area, you must force the removal with the `-f` option.

Another useful thing you may want to do is to keep the file in your working tree but remove it from your staging area.
In other words, you may want to keep the file on your disk but not have Git track it anymore.
`git rm --cached <path>`

You can pass files, directories, and file-glob patterns to the `git rm` command.
- `git rm log/\*.log`: note the backslash `\` in front of the `*`.
  - This is necessary because Git does its own filename expansion in addition to your shell's filename expansion.
  - This command removes all files that have the `.log` extension in the `log/` directory.
- `git rm \*~`: removes all files whose names end with a `~`.


### Moving Files

`git mv <file1> <file2>` basically equals to
1. `mv <file1> <file2>`
2. `git rm <file1>`
3. `git add <file2>`

## Viewing the Commit History

The `git log` command
```bash
git log #  w/o args, `git log` lists the commits made in that repository in reverse chronological order.
git log -p/--patch -2   #  additionally shows the difference (hence "patch") introduced in each commit
                        #+ You can also limit the number of log entries displayed (use `-2` to show only the last two entries)
git log --stat  #  additionaly shows a list of modified files and how many lines in those files were added/removed.
git log --pretty=oneline    #  This option changes the log output to formats other than the default
git log --pretty=format:"<format>"  #  Allows you to specify your own log output format.
                                    #+ This is especially useful when you're generating output for machine parsing
git log --pretty=oneline --graph    #  `--graph` adds a nice ASCII graph showing your branch and merge hitory
```

### Limiting Log Output

Limiting `git log` output
```bash
git log -2  #  show last two commits
git log --since=2.weeks #  e.g., "2008-01-15", "2 years 1 day 3 minites ago"
git log --author    #  filter on a specific author
git log --grep  #  lets you search for keywords in the commit message
git log --grep <patterns> --all-match   #  the default behavior for `--grep` is "or", `--all-match` change it to "and"
git log -S <function_name>  #  Only show commits adding or removing code matching the string
git log -- <path/to/file>   #  Limit the log output to commits that introduced a change to those files.
                            #+ `--` is used to separate the paths from the options
git log --no-merges #  prevent the display of merge commits
git log --oneline --decorate    #  show HEAD, branches
```

## Undoing Things

Be careful, because you can't always undo things.

When you commit too early and possibly forget to add some files, or you mess up your commit message.
If you want to redo that commit, make the additional changes you forgot, stage them, and commit again using `--amend` option
```bash
git commit --amend
```

This command taks your staging area and uses it for the commit.
You can edit the message, but it overwrites your previous commit.
You end up with a single commit - the second commit replaces the results of the first.

Remember only amend commits that are still local and have no been pushed somewhere.

### Unstaging a Staged File

Q: Say if you've staged two files but want to commit them as two separate changes, how can you unstaged one of the two?
A: `git status` says you can use `git reset HEAD <file>...` to unstage.
```bash
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
  renamed: README.md -> README
  modified: CONTRIBUTING.md

$ git reset HEAD CONTRIBUTING.md
Unstaged changes after reset:
M CONTRIBUTING.md
```

### Unmodifying a Modified File

Q: What if you don't want to keep your changes to one file? How can you unmodify it - revert it back to what it looked like when you last committed?
A: `git status` says you can run `git checkout -- <file>...` to discard changes.
```bash
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  modified: CONTRIBUTING.md
```

It’s important to understand that `git checkout -- <file>` is a dangerous command.
Any local changes you made to that file are gone — Git just replaced that file with the last staged or committed version.

If you would like to keep the changes you’ve made to that file but still need to get it out of the way for now,
we’ll go over **stashing** and branching in _Git Branching_; these are generally better ways to go.

Remember, anything that is committed in Git can almost always be recovered.
Even commits that were on branches that were deleted or commits that were overwritten with an `--amend` commit can be recovered
(see _Data Recovery_ for data recovery).
However, anything you lose that was never committed is likely never to be seen again.

### Undoing things with `git restore`

Git version 2.23.0 introduced a new command: `git restore`.
It’s basically an alternative to `git reset` which we just covered.
From Git version 2.23.0 onwards, Git will use `git restore` instead of `git reset` for many undo operations.

#### Unstaging a Staged File with `git resotre`

```bash
$ git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
  modified: CONTRIBUTING.md
  renamed: README.md -> README

$ git restore --staged CONTRIBUTING.md
```

#### Unmodifying a Modified File with `git restore`

```bash
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
  modified: CONTRIBUTING.md

$ git restore CONTRIBUTING.md
```

## Working with Remotes

Remote repositories
- they are versions of your project that are hosted on the Internet or network somewhere.
- you can have several of them, each of which generally is either read-only or read/write for you
- collaborating with others involves managing these remote repositories and pushing and pulling data to and from them when you need to share work
  - Managing remote repositories includes knowing how to
    - add remote repositories,
	- remove remotes that are no longer valid,
	- manage various remote branches and define them as being tracked or not, and more
- Remote repositories can be on your local machine

### Showing Your Remotes

See which remote servers you have configured
```bash
$ git remote  #  lists shortnames of each remote handle you've specified
              #+ `origin` is the default name Git gives to the server you cloned from (if you used `git clone`)
origin

$ git remote -v   #  shows besides the shortnames also the URLs to be used when reading and writing to that remote
pb https://github.com/paulboone/ticgit (fetch)
pb https://github.com/paulboone/ticgit (push)
origin git@github.com:mojombo/grit.git (fetch)
origin git@github.com:mojombo/grit.git (push)
```



### Adding Remote Repositories

Add a new remote Git repository
```bash
git remote add <shortname> <url>    #  e.g., `git remote add pb https://github.com/paulboone/ticgit`
```

### Fetching and Pulling from Your Remotes

`git fetch <remote>` goes out to that remote project and pulls down all the data from that remote project that you don't have yet.
After you do this, you should have references to all the branches from that remote, which you can merge in or inspect at any time.

If you clone a repository, the command automatically adds that remote repository under the name `origin`.
Note that `git fetch` only downloads the data to your local repository.
It doesn't automatically merge it with any of your work or modify what you're currently working on.
You have to merge it manually into your work when you're ready.
```bash
$ git fetch pb  # fetch all the information that paul has but that you don't yet have in your repository
remote: Counting objects: 43, done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 43 (delta 10), reused 31 (delta 5)
Unpacking objects: 100% (43/43), done.
From https://github.com/paulboone/ticgit
 * [new branch] master -> pb/master
 * [new branch] ticgit -> pb/ticgit
```

If your current branch is set up to track a remote branch, you can use `git pull` to automatically fetch and then merge that remote branch into your current branch.
By default, `git clone` automatically sets up your local `master` branch to track the remote `master` branch on the server you cloned from.

From git version 2.27, `git pull` will give a warning if `pull.rebase` variable is not set.
If you want the default behavior (fast-forward if possible, else create a merge commit): `git config --global pull.rebase "false"`
If you want to rebase when pulling: `git config --global pull.rebase "true"`.

### Pushing to Your Remotes

You can push your project upstream when you reach a point that you want to share.
```bash
git push origin master  #  push your `master` branch to your `origin` server
```

This command works only if you cloned from a server to which you have write access and if nobody has pushed in the meantime.
If you and someone else clone at the same time and they push upstream and then you push upstream, your push will rightly be rejected.
You’ll have to fetch their work first and incorporate it into yours before you’ll be allowed to push.

### Inspecting a Remote

`git remote show <remote>` gives you more information about a particular remote.
It shows
- which branch is automatically pushed to when you run `git push` while on certain branches
- which remote branches on the server you don't yet have
- which remote branches you have that have been removed from the server
- multiple local branches that are able to merge automatically with their remote-tracking branch when you run `git pull`
```bash
$ git remote show origin
* remote origin
  URL: https://github.com/my-org/complex-project
  Fetch URL: https://github.com/my-org/complex-project
  Push URL: https://github.com/my-org/complex-project
  HEAD branch: master
  Remote branches:
    master tracked
    dev-branch tracked
    markdown-strip tracked
    issue-43 new (next fetch will store in remotes/origin)
    issue-45 new (next fetch will store in remotes/origin)
    refs/remotes/origin/issue-11 stale (use 'git remote prune' to remove)
  Local branches configured for 'git pull':
    dev-branch merges with remote dev-branch
    master merges with remote master
  Local refs configured for 'git push':
    dev-branch pushes to dev-branch (up to date)
    markdown-strip pushes to markdown-strip (up to date)
    master pushes to master (up to date)
```

### Renaming and Removing Remotes

Rename & remove remotes
```bash
git remote rename <old-remote-name> <new-remote-name>   #  this changes all your remote-tracking branch names, too
git remote removal <remote-name> #  once you delete the reference to a remote this way, all remote-tracking branches and
                                 #+ configuration settings associated with that remote are also deleted
```

## Tagging

Git has the ability to tag specific points in a repository's history as being important.
Typically, ppl use this functionality to mark release points (`v1.0`, `v2.0`, and so on).

### Listing Your Tags

```bash
$ git tag # with optional -l or --list
$ git tag -l "v1.8.5*"  # You can also search for tags that match a particular pattern. Listing tag wildcards requires -l or --list option
v1.8.5
v1.8.5-rc0
v1.8.5-rc1
v1.8.5-rc2
v1.8.5-rc3
```

### Creating Tags

Git supports two types of tags: **lightweight** and **annotated**.
- a lightweight tag is very much like a branch that doesn't change
- annotated tags, however, are stored as full objects in the Git database. They're checksummed; contain the tagger name, email, and data;
  have a tagging message; and can be signed and verified with GNU Privacy Guard (GPG).

### Annotaged Tags

Specify `-a` to create an annotated tag when you run the `tag` command.

```bash
$ git tag -a v1.4 -m "my version 1.4"  # `-m` specifies a tagging message
$ git tag
v0.1
v1.4
```

You can see the tag data along with the commit that was tagged by using the `git show` command:
```bash
$ git show v1.4
tag v1.4
Tagger: Ben Straub <ben@straub.cc>
Date:   Sat May 3 20:19:12 2014 -0700

my version 1.4

commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    Change version number
```

### Lightweight Tags

This is basically the commit checksum stored in a file - no other information is kept.
To create a lightweight tag, don't supply any of the `-a`, `-s`, or `-m` options, just provide a tag name:
```bash
$ git tag v1.4
```

If you run `git show` on the tag, you don’t see the extra tag information. The command just shows the commit:
```bash
$ git show v1.4-lw
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    Change version number
```

### Tagging Later

You can also tag commits after you've moved past them.
To tag a commit, you specify the commit checksum (or part of it) at the end of the command:
```bash
$ git tag v1.2 9fceb02
```

### Sharing Tags

By default, `git push` doesn't transfer tags to remote servers.
You'll have to explicitly push tags to a shared server after you've created them.
This process is just like sharing remote branches: `git push origin <tagname>`
```bash
$ git push origin v1.5
Counting objects: 14, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (12/12), done.
Writing objects: 100% (14/14), 2.05 KiB | 0 bytes/s, done.
Total 14 (delta 3), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.5 -> v1.5
```

To transfer all of your tags to the remote server:
```bash
$ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 160 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.4 -> v1.4
 * [new tag]         v1.4-lw -> v1.4-lw
```
Now, when someone else clones or pulls from your repository, they will get all your tags as well

`git push <remote> --tags` will push both tags.
- there is no option to push only lightweight tags
- if you want to only push annotated tags: `git push <remote> --follow-tags`

### Deleting Tags

To delete a tag on your local repository:
```bash
$ git tag -d v1.4-lw
Deleted tag 'v1.4-lw' (was e7d5add)
```

There are two common variations for deleting a tag from a remote server.
- The first variation is `git push <remote> :refs/tags/<tagname>`:
  ```bash
  $ git push origin :refs/tags/v1.4-lw
  To /git@github.com:schacon/simplegit.git
   - [deleted]          v1.4-lw
  ```
  The way to interpret the above is to read it as the null value before the colon is being pushed to the remote tag name, effectively deleting it.
- The second (and more intuitive) way to delete a remote tag is with:
  ```bash
  $ git push origin --delete <tagname>
  ```

### Checking out Tags

You can do a `git checkout` of that tag if you want to view the versions of files a tag is pointing to.
Although this puts your repository in "detached HEAD" state.

In "detached HEAD" state, if you make changes and then create a commit, the tag will stay the same,
but your new commit won’t belong to any branch and will be unreachable, except by the exact commit hash.
Thus, if you need to make changes — say you’re fixing a bug on an older version, for instance — you will generally want to create a branch:
```bash
$ git checkout -b version2 v2.0.0
Switched to a new branch 'version2'
```

## Git Aliases

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
```

This technique can also be very useful in creating commands that you think should exist
```bash
git config --global alias.unstage 'reset HEAD --'   #  `git unstage fileA` == `gitreset HEAD --fileA`
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'    #  You can run an external command rather thana Git subcommand by starting the
                                            #+ command with a `!` character.

```
