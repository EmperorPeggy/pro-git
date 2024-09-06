

## All about `git diff`

[Source](https://stackoverflow.com/questions/9903541/finding-diff-between-current-and-last-version)
- As of Git 1.8.5, @ is an alias for HEAD

- Check the diff from `<commit-1>` to `<commit-2>`: `git diff <commit-1> <commit-2>`
- `git diff @~3 @~2`: check the diff from `@~~~` (`@~3`) to `@~~` (`@~2`)


## `git rm`

[Source](https://stackoverflow.com/questions/1274057/how-do-i-make-git-forget-about-a-file-that-was-tracked-but-is-now-in-gitignore)

- `git rm --cached <file>`: To stop tracking a file, we must remove it from the index


## `git rebase`

[split a commit into two](https://www.internalpointers.com/post/split-commit-into-smaller-ones-git)
1. Search for the commit you want to split: `<commit-to-be-splitted>`
2. `git rebase -i <comit-to-be-splitted>~`; don't forget the `~` at the end
3. In the text editor, put `e` before the `<commit-to-be-splitted>`
4. (you are now editing commit `<commit-to-be-splitted>`), run `git reset HEAD~1`
5. You are now free to create smaller commits. Commit now in the usual way.
6. When you are done with your surgery, invoke `git rebase --continue` to conclude the rebase.
