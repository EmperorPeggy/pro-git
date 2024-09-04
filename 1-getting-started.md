## What is git?

### Snapshots, Not Differences

Git thinks of its data more like a series of snapshots of a miniature filesystem.
With Git, every time you commit, or save the state of your project, Git basically takes a picture of what all your files look like at that moment and stores a reference to that snapshot.

### Git Has Integrity

Everything in Git is checksummed before it is stored and is then referred to by that checksum.
This means it's impossible to change the contents of any file or directory w/o Git knowing about it.

The mechanism that Git uses for this checksumming is called a **SHA-1 hash**.
This is a 40-character string composed of hexadecimal characters and calculated based on the contents of a file or directory structure in Git.
In fact, Git stores everything in its database not by file name but by the hash value of its contents.

### The Three States

Git has three main states that your files can reside in:
- modified: files are changed but not committed yet
- staged: you have marked a modified file in its current version to go into your next commit snapshot
- committed: the data is safely stored in your local database
- (untracked: git is not tracking it)

This leads us to the three main sections of a Git project:
- the **working tree**: single checkout of one version of the project. These files are pulled out of the compressed database in the Git directory and placed on disk for you to use
- the **staging area**: a file, generally contained in your Git directory, which stores information about what will go into your next commit; a.k.a. the **index** or **staging area**
- the **Git directory**: where Git stores the metadata and object database for your project.
It is what is copied when you _clone_ a repository from another computer.

The basic Git workflow goes something like this:
1. you modify files in your working tree
2. you selectively stage just those changes you want to be part of your next commit, which adds only those changes to the staging area
3. you do a commit, which takes the files as they are in the staging area and stores that snapshot permanently to your Git directory.

- If a particular version of a file is in the Git directory, it's considered _committed_.
- If it has been modified and was added to the staging area, it is _staged_.
- And if it was changed since it was checked out but hasn't been staged, it is _modified_.

## First-time git setup

Git comes with a tool called `git config` that lets you get and set configuration variables that control all aspects of how git looks and operates.
The vars can be stored in 3 different places. Each level overrides values in the previous level.
1. `git config --system ...`: affects `[path]/etc/gitconfig`, which contains values applied to every user on the system and all their repositories.
   It reads and wirtes from this file specifically.
2. `git config --global ...`: affects `~/.gitconfig` or `~/.config/git/config`, which contains values specific personally to you, the user.
   This affects all of the repositories you work with on your system
3. `git config [--local] ...`: affects `.git/config`, which is specific to that single repository you're currently using.
   This is the default for `git config`

You can view all of your settings and where they come from using `git config --list --show-origin`

### Your Identity, Editor, and Default branch name

The 1st thing you should do when you install Git is to set your user name and email address.
This is important because every Git commit uses this information.
```bash
$ git config --global user.name "<your-name>"
$ git config --global user.email "<your-email>"
$ git config --global core.editor <your-text-editor>  #  optional; e.g., emacs
$ git config --global init.defaultBranch <default-branch-name>    #  optional; e.g., main
```

### Checking Your Settings

```bash
$ git config --list  #  list all settings Git can find at that point
$ git config <variable> #  check what Git thinks a specific key\s value is; e.g., `user.name`
$ git config --list --show-origin   #  view all settings and where they come from
$ git config --show-origin <variable>  #  show where the `<variable>` is; e.g., `user.email`

# Example
$ git config --list
user.name=John Doe
user.email=johndoe@example.com
color.status=auto
...
$ git config user.name
John Doe
$ git config --show-origin rerere.autoUpdate
file:/home/johndoe/.gitconfig false
```

## Getting help

3 equivalent ways
```shell
$ git help <verb>    #  e.g., git help config
$ git <verb> --help  #  or `git <verb> -h` for concise help; e.g., `git config -h`
$ man git-<verb>     #  the same as `git help <verb>`; e.g., `man git-config`

# Example
$ git add -h
usage: git add [<options>] [--] <pathspec>...
  -n, --dry-run dry run
  -v, --verbose be verbose
...
```
