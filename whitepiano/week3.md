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
  - ~~앞으론 최신화에 `rebase` 대신 이거 써야겠다~~
  - 아무리 생각해도 뭔가 이상해서 Claud.ai에게 물어봤습니다.
    > git merge 명령을 실행하면 두 가지가 발생합니다:  
    > - 병합 이력(commit history) 생성: 두 브랜치가 병합되었다는 기록이 git 히스토리에 남음  
    > - 코드 병합: 두 브랜치의 코드가 실제로 합쳐짐
    > 
    > `-s ours` 옵션을 사용하면 1번만 발생하고 2번은 발생하지 않습니다. 즉, 병합했다는 기록만 남기고 실제 코드는 현재 브랜치 상태를 그대로 유지합니다.
  - 역시 `rebase` 엔딩... 그래도 아래 정리한 `RRR`을 알게 되었으니 럭키비키인걸로~

### Subtree

-  The idea of the subtree merge is that you have two projects, and one of the projects maps to a subdirectory of the other one.
-  Not all the branches in your repository actually have to be branches of the same project. It’s not common, because it’s rarely helpful, but it’s fairly easy to have branches contain completely different histories.
- This gives us a way to have a workflow somewhat similar to the submodule workflow without using submodules.

## 7.9 RRR(Reuse Recorded Resolution)

- As the name implies, it allows you to ask Git to remember how you’ve resolved a hunk conflict so that the next time it sees the same conflict, Git can resolve it for you automatically.
- `git config --global rerere.enabled true`

### Handy Situation Example

- When you want to make sure a long-lived topic branch will ultimately merge cleanly, but you don’t want to have a bunch of intermediate merge commits cluttering up your commit history.
- With `rerere` enabled, you can attempt the occasional merge, resolve the conflicts, then back out of the merge.
- If you do this continuously, then the final merge should be easy because `rerere` can just do everything for you automatically.
- This same tactic can be used if you want to keep a branch rebased so you **don’t have to deal with the same rebasing conflicts each time** you do it.

## 8.4 Git Enforcyed Policy

- e.g., Checks for a custom commit message format
- e.g., Allows only certain users to modify certain subdirectories in a project.

### Server Side Hooks

- ACL(Access Control List)를 활용해서 사용자에 기반한 제약 가능

### Client Side Hooks

- Server Side Hook의 단점 : `push`하기 전까진 알 수 없음. 만약 실패하면, commit을 다시 하고 push하는 과정 속에서 history가 지저분해짐
