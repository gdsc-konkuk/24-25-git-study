## 7.8 Advanced Merging

> Git’s philosophy is to be smart about determining when a merge resolution is unambiguous, but if there is a conflict, it does not try to be clever about automatically resolving it.

- `git merge --abort` (or `git reset --hard HEAD`) : cancel merging
    - The only cases where it may not be able to do this perfectly would be if you had unstashed, uncommitted changes in your working directory when you ran it, otherwise it should work fine.
- `git merge -Xignore-all-space` or `git merge -Xignore-space-change` : ignore whitespaces
    - This is a lifesaver if you have someone on your team who likes to occasionally reformat everything from spaces to tabs or vice-versa.
- `git merge -Xours` or `git merge Xtheirs` : simply choose a specific side and ignore the other side
- `git checkout --conflict` : reset conflict markers
- `git log --oneline --left-right --merge` : check conflict commit history briefly
    - If you run that with the -p option instead, you get just the diffs to the file that ended up in conflict.
- `git merge -s ours` : merge only our branch (thier branch has no knowledge of our branch)
  - **앞으론 최신화에 `rebase` 대신 이거 써야겠다**

### Subtree

-  The idea of the subtree merge is that you have two projects, and one of the projects maps to a subdirectory of the other one.
-  Not all the branches in your repository actually have to be branches of the same project. It’s not common, because it’s rarely helpful, but it’s fairly easy to have branches contain completely different histories.
- This gives us a way to have a workflow somewhat similar to the submodule workflow without using submodules.
