https://www.vogella.com/tutorials/GitSubmodules/article.html

## 2. Working with repos that contain submodules

### 2.1 Clone a repository including its submodules
- `git clone --recursive [--jobs/-j N] <url>`: download `N`submodules concurrently

### 2.2 Download/Fetch submodules within an already cloned repository
- `git submodule update --init [--jobs/-j N]`, or
- `git submodule update --init --recursive [--jobs/-j N]`, if there are nested submodules

### 2.3 Pulling/Fetching with submodules
Once you have set up the submodules you can update the repository with fetch/pull like you would normally do. 
- `git pull --recurse-submodules`: pull all changes in the repo including changes the submodules
- `git submodule update --remote`: pull all changes for the submodules

### 2.4 Execute a command on every submodule
- `git submodule foreach 'git reset --hard'`
- `git submodule foreach --recursive 'git reset --hard'`: including nested submodules

## 3. Creating repositories with submodules
### 3.1. Adding a submodule to a Git repository and tracking a branch
If you add a submodule, you can specify which branch should be tracked via the `-b` parameter of the `submodule add` command. 
The `git submodule init` command creates the local configuration file for the submodules, if this configuration does not exist.
- `git submodule add -b master <url>`: add submodule and set `master` as the branch you want to track
- `git submodule init`: initialize submodule configuration

### 3.2. Adding a submodule and tracking commits
Alternatively to the tracking of a branch, you can also control which commit of the submodule should be used. In this case the Git parent repository tracks the commit that should be checked out in each configured submodule. 
Performing a submodule update checks out that specific revision in the submodule’s Git repository. You commonly perform this task after you pull a change in the parent repository that updates the revision checked out in the submodule. You would then fetch the latest changes in the submodule’s Git repository and perform a submodule update to check out the current revision referenced in the parent repository. Performing a submodule update is also useful when you want to restore your submodule’s repository to the current commit tracked by the parent repository. 
- `git submodule add [URL to Git repo]`
- `git submodule init`

### 3.3. Updating which commit you are tracking
The relevant state for the submodules are defined by the main repository. If you commit in your main repository, the state of the submodule is also defined by this commit.
The `git submodule update` command sets the Git repository of the submodule to that particular commit. The submodule repository tracks its own content which is nested into the main repository. The main repository refers to a commit of the nested submodule repository.
Use the `git submodule update` command to set the submodules to the commit specified by the main repository. This means that if you pull in new changes into the submodules, you need to create a new commit in your main repository in order to track the updates of the nested submodules.

The following example shows how to update a submodule to its latest commit in its master branch.

```shell
# update submodule in the master branch
# skip this if you use --recurse-submodules
# and have the master branch checked out
cd [submodule directory]
git checkout master
git pull
# commit the change in main repo
# to use the latest commit in master of the submodule
cd ..
git add [submodule directory]
git commit -m "move submodule to latest commit in master"
# share your changes
git push
```

Another developer can get the update by pulling in the changes and running the submodules update command.

```shell
# another developer wants to get the changes
git pull
# this updates the submodule to the latest
# commit in master as set in the last example
git submodule update
```

### 3.4. Delete a submodule from a repository
To remove a submodule called mymodule you need to:
```shell
git submodule deinit -f -- mymodule
rm -rf .git/modules/mymodule
git rm -f mymodule
```
