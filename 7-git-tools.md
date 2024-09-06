## Revision Selection

TODO: HERE

## Interactive Staging

TODO: Skipped

## [X] Stashing and Cleaning

Scenario: When you've been working on part of your project, things are in a messy state and you want to switch branches for a bit to work on something else.
The problem is, you don’t want to do a commit of half-done work just so you can get back to this point later.
The answer to this issue is the `git stash` command.

Stashing takes the dirty state of your working directory — that is, your modified tracked files and staged changes — and saves it on a stack of unfinished changes that you can reapply at any time (even on a different branch).

### Stashing Your Work

To demonstrate stashing, you'll go into your project and start working on a couple of files and possibly stage one of the changes.
```bash
$ git status
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    modified: index.html

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified: lib/simplegit.rb
```

Now you want to switch branches, but you don’t want to commit what you’ve been working on yet, so you’ll stash the changes.
To push a new stash onto your stack, equivalent to `git stash` or `git stash push`:
```bash
$ git stash
Saved working directory and index state \
  "WIP on master: 049d078 Create index file"
HEAD is now at 049d078 Create index file
(To restore them type "git stash apply")
```

You can now see that your working directory is clean:
```bash
$ git status
$ git status
# On branch master
nothing to commit, working directory clean
```

At this point, you can switch branches and do work elsewhere; your changes are stored on your stack.
To see which stashes you've stored, use `git stash list`:
```bash
$ git stash list
stash@{0}: WIP on master: 049d078 Create index file
stash@{1}: WIP on master: c264051 Revert "Add file_size"
stash@{2}: WIP on master: 21d80a5 Add number to log
```
In this case, two stashes were saved previously, so you have access to 3 different stashed works.

You can reapply **the one you just stashed** by running `git stash apply`.
If you want to apply one of the older stashes, you can specify it by naming it, like this: `git stash apply stash@{2}`.
```bash
$ git stash apply
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified: index.html
    modified: lib/simplegit.rb

no changes added to commit (use "git add" and/or "git commit -a")
```

You can see that Git re-modifies the files you reverted when you saved the stash.
In this case, you had a **clean working directory** when you tried to apply the stash, and you tried to apply it **on the same branch you saved it from**.
Having a clean working directory and applying it on the same branch aren't necessary to successfully apply a stash.
- You can save a stash on one branch, switch to another branch later, and try to reapply the changes.
- You can also have modified and uncommitted files in your working directory when you apply a stash - Git gives you merge conflicts if anything no longer applies cleanly.

The changes to your files were reapplied, but the file you staged before wasn’t restaged.
To do that, you must run the `git stash apply` command with a `--index` option to tell the command to try to reapply the staged changes.

If you had run that instead, you'd have gotten back to your original position:
```bash
$ git stash apply --index
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    modified: index.html

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified: lib/simplegit.rb
```

The `apply` option only tries to apply the stashed work — you continue to have it on your stack.
To remove it, you can run `git stash drop` with the name of the stash to remove:
```bash
$ git stash list
stash@{0}: WIP on master: 049d078 Create index file
stash@{1}: WIP on master: c264051 Revert "Add file_size"
stash@{2}: WIP on master: 21d80a5 Add number to log
$ git stash drop stash@{0}
Dropped stash@{0} (364e91f3f268f0900bc3ee613f9f733e82aaed43)
```

You can also run `git stash pop` to apply the stash and then immediately drop it from your stack.

### Creative Stashing

There are a few stash variants that may also be helpful.

The first option is `--keep-index`, which tells Git to not only include all staged content in the stash being created, but simultaneously leave it in the index.
```bash
$ git status -s
M index.html
 M lib/simplegit.rb

$ git stash --keep-index
Saved working directory and index state WIP on master: 1b65b17 added the index file
HEAD is now at 1b65b17 added the index file

$ git status -s
M index.html
```

The second common thing you may want to do with stash is to stash the untracked files as well as the tracked ones.
By default, `git stash` will stash only modified and staged _tracked_ files.

If you specify `--include-untracked` or `-u`, Git will include untracked files in the stash being created.
However, including untracked files in the stash will still not include explicitly ignored files; to additionally include ignored files, use `--all` (or just `-a`).
```bash
$ git status -s
M index.html
 M lib/simplegit.rb
?? new-file.txt

$ git stash -u
Saved working directory and index state WIP on master: 1b65b17 added the index file
HEAD is now at 1b65b17 added the index file
$ git status -s
$
```

Finally, if you specify the `--patch` flag, Git will not stash everything that is modified but will instead prompt you interactively which of the changes you would like to stash
and which you would like to keep in your working directory.
```bash
$ git stash --patch
diff --git a/lib/simplegit.rb b/lib/simplegit.rb
index 66d332e..8bb5674 100644
--- a/lib/simplegit.rb
+++ b/lib/simplegit.rb
@@ -16,6 +16,10 @@ class SimpleGit
         return `#{git_cmd} 2>&1`.chomp
       end
     end
+
+    def show(treeish = 'master')
+      command("git show #{treeish}")
+    end
 end
 test
Stash this hunk [y,n,q,a,d,/,e,?]? y

Saved working directory and index state WIP on master: 1b65b17 added the index file
```

### Creating a Branch from a Stash

If you stash some work, leave it there for a while, and continue on the branch from which you stashed the work, you may have a problem reapplying the work.
If the apply tries to modify a file that you’ve since modified, you’ll get a merge conflict and will have to try to resolve it.
If you want an easier way to test the stashed changes again, you can run `git stash branch <new-branchname>`, which
1. creates a new branch for you with your selected branch name,
2. checks out the commit you were on when you stashed your work,
3. reapplies your work there, and then
4. drops the stash if it applies successfully.

This is a nice shortcut to recover stashed work easily and work on it in a new branch.
```bash
$ git stash branch testchanges # optionally with <stash>
M   index.html
M   lib/simplegit.rb
Switched to a new branch 'testchanges'
On branch testchanges
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    modified: index.html

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified: lib/simplegit.rb

Dropped refs/stash@{0} (29d385a81d163dfd45a452a2ce816487a6b8b014)
```

### Cleaning your Working Directory

Finally, you may not want to stash some work or files in your working directory, but simply get rid of them; that’s what the `git clean` command is for.

You'll want to be pretty careful with this command, since it’s designed to remove files from your working directory that are not tracked.
If you change your mind, there is often no retrieving the content of those files.
A safer option is to run `git stash --all` to remove everything but save it in a stash.

Assuming you do want to remove cruft files or clean your working directory, you can do so with `git clean`.
To remove all the untracked files in your working directory, you can run `git clean -f -d`, which
- removes any files and also any subdirectories that become empty as a result.
- the `-f` means 'force' or "really do this," and is required if the Git configuration variable `clean.requireForce` is not explicitly set to `false`.

If you ever want to see what it would do, you can run the command with the `--dry-run` (or `-n`) option.
```bash
$ git clean -d -n
Would remove test.o
Would remove tmp/
```

By default, the `git clean` command will only remove untracked files that are not ignored.
Any file that matches a pattern in your `.gitignore` or other ignore files will not be removed.
If you want to remove those files too, such as to remove all `.o` files generated from a build so you can do a fully clean build, you can add a `-x` to the `clean` command.
```bash
$ git status -s
 M lib/simplegit.rb
?? build.TMP
?? tmp/

$ git clean -n -d
Would remove build.TMP
Would remove tmp/

$ git clean -n -d -x
Would remove build.TMP
Would remove test.o
Would remove tmp/
```

Always run `git clean` with a `-n` first to double check before changing the `-n` to a `-f` and doing it for real.
The other way you can be careful about the process is to run it with the `-i` or "interactive" flag.
This way you can step through each file individually or specify patterns for deletion interactively.
```bash
$ git clean -x -i
Would remove the following items:
  build.TMP test.o
  notes/git/pro-git/README.md~
*** Commands ***
    1: clean  2: filter by pattern  3: select by numbers  4: ask each
	5: quit  6: help

What now>
```

There is a quirky situation where you might need to be extra forceful in asking Git to clean your working directory.
If you happen to be in a working directory under which you've copied or cloned other Git repositories (perhaps as submodules), even `git clean -fd` will refuse to delete those directories.
In cases like that, you need to add a second `-f` option for emphasis.

## ...

## Searching TODO[]

You’ll often need to find where a function is called or defined, or
display the history of a method. Git provides a couple of useful tools for looking through the code
and commits stored in its database quickly and easily.

### Git Grep

Git ships with a command called grep that allows you to easily search through any committed tree,
the working directory, or even the index for a string or regular expression. 

By default, `git grep` will look through the files in your working directory. As a first variation, you
can use either of the `-n` or `--line-number` options to print out the line numbers where Git has found
matches:
```bash
$ git grep -n gmtime_r
compat/gmtime.c:3:#undef gmtime_r
compat/gmtime.c:8: return git_gmtime_r(timep, &result);
compat/gmtime.c:11:struct tm *git_gmtime_r(const time_t *timep, struct tm *result)
compat/gmtime.c:16: ret = gmtime_r(timep, result);
```

`git grep` supports a plethora of other interesting options.
1. instead of printing all of the matches, you can ask git grep to summarize the output by showing you only which files contained the search string and how many matches there were in each file with the `-c` or `--count` option:
    ```bash
    $ git grep --count gmtime_r
    compat/gmtime.c:4
    compat/mingw.c:1
    ```
2. If you’re interested in the context of a search string, you can display the enclosing method or function for each matching string with either of the `-p` or `--show-function` options:
   ```bash
   $ git grep -p gmtime_r *.c
    date.c=static int match_multi_number(timestamp_t num, char c, const char *date,
    date.c: if (gmtime_r(&now, &now_tm))
    date.c=static int match_digit(const char *date, struct tm *tm, int *offset, int
    *tm_gmt)
    date.c: if (gmtime_r(&time, tm)) {
    ```
3. search for complex combinations of strings with the `--and` flag, which ensures that multiple matches must occur in the same line of text.
   For instance, let’s look for any lines that define a constant whose name contains either of the substrings “LINK” or “BUF_MAX”, specifically in an older version of the Git codebase represented by the tag `v1.8.0` (we’ll throw in the `--break` and
`--heading` options which help split up the output into a more readable format):
    ```shell
    $ git grep --break --heading \
      -n -e '#define' --and \( -e LINK -e BUF_MAX \) v1.8.0
    v1.8.0:builtin/index-pack.c
    62:#define FLAG_LINK (1u<<20)
    v1.8.0:cache.h
    73:#define S_IFGITLINK 0160000
    74:#define S_ISGITLINK(m) (((m) & S_IFMT) == S_IFGITLINK)
    v1.8.0:environment.c
    54:#define OBJECT_CREATION_MODE OBJECT_CREATION_USES_HARDLINKS
    ```

The git grep command has a few advantages over normal searching commands like grep and ack.
The first is that it’s really fast, the second is that you can search through any tree in Git, not just the
working directory. 

### Git Log Searching

Perhaps you’re looking not for where a term exists, but when it existed or was introduced. The git
log command has a number of powerful tools for finding specific commits by the content of their
messages or even the content of the diff they introduce.

If, for example, we want to find out when the `ZLIB_BUF_MAX` constant was originally introduced, we
can use the `-S` option (colloquially referred to as the Git “pickaxe” option) to tell Git to show us only
those commits that changed the number of occurrences of that string.
```bash
$ git log -S ZLIB_BUF_MAX --oneline
e01503b zlib: allow feeding more than 4GB in one go
ef49a7a zlib: zlib can only process 4GB at a time
```

#### Line Log Search

TODO: Skipped



## Rewriting History TODO[]

