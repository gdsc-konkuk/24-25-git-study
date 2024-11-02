> git-book 7.1 ~ 7.6 (11/2/2024)

## Commit Re-visions

- 단순 hash key와 간격 (`$ git show HEAD@{5}`) 뿐만 아니라 **시간**으로도 지정 가능 `$ git show master@{yesterday}`
- `^`를 통한 추척에는 "두 번째 부모"라는 개념이 있음. 즉 Merge에 대해 양쪽 히스토리 모두 접근이 가능함 (`~`의 경우에는 첫 번째 부모를 가르킴)
- Branch가 통합되지 않은 부분을 추출하고 싶다면 `$ git log master..experiment`와 같이 `..`를 통한 range를 설정 가능함
    - 응용 : `$ git log origin/master..HEAD` (`HEAD` 생략 가능)
    - Branch가 3개 이상일 경우
      - `$ git log refA refB ^refC`
      - `$ git log refA refB --not refC`

### Interactive Modes

- `rebase` 뿐만 아니라 `add`와 같은 명령어에도 `i` 옵션 사용 가능
- `interactive` 모드가 아니더라도 `-p` 옵션을 통해 [`hunk`](https://www.gnu.org/software/diffutils/manual/html_node/Hunks.html) 단위 커밋 가능

## Stashing

> TIP. `save` 대신 `push`를 사용할 것

> Git은 Stash에 저장할 때 수정했던 파일들을 복원해준다. 복원할 때의 워킹 디렉토리는 Stash 할 때의 그 브랜치이고 워킹 디렉토리도 깨끗한 상태였다. 하지만 꼭 깨끗한 워킹 디렉토리나 Stash 할 때와 같은 브랜치에 적용해야 하는 것은 아니다. 어떤 브랜치에서 Stash 하고 다른 브랜치로 옮기고서 거기에 Stash를 복원할 수 있다. 그리고 꼭 워킹 디렉토리가 깨끗한 상태일 필요도 없다. 워킹 디렉토리에 수정하고 커밋하지 않은 파일들이 있을 때도 Stash를 적용할 수 있다. 만약 충돌이 있으면 알려준다.
>
> Git은 Stash를 적용할 때 Staged 상태였던 파일을 자동으로 다시 Staged 상태로 만들어 주지 않는다. 그래서 git stash apply 명령을 실행할 때 --index 옵션을 주어 Staged 상태까지 적용한다. 그래야 원래 작업하던 상태로 돌아올 수 있다.

- Save : `$ git stash push`
- List : `$ git stash list`
- Apply : `$ git stash apply stash@{2}`
- Clean : `$ git stash drop stash@{0}`
- Apply + Clean : `$ git stash pop`

### Test Staches

> 보통 Stash에 저장하면 한동안 그대로 유지한 채로 그 브랜치에서 계속 새로운 일을 한다. 그러면 이제 저장한 Stash를 적용하는 것이 문제가 된다. 수정한 파일에 Stash를 적용하면 충돌이 일어날 수도 있고 그러면 또 충돌을 해결해야 한다. 필요한 것은 Stash 한 것을 쉽게 다시 테스트하는 것이다. git stash branch <브랜치> 명령을 실행하면 Stash 할 당시의 커밋을 Checkout 한 후 새로운 브랜치를 만들고 여기에 적용한다. 이 모든 것이 성공하면 Stash를 삭제한다.

```
$ git stash branch testchanges
M index.html
M lib/simplegit.rb
Switched to a new branch 'testchanges'
On branch testchanges
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

  modified:   index.html

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

  modified:   lib/simplegit.rb

Dropped refs/stash@{0} (29d385a81d163dfd45a452a2ce816487a6b8b014)
```

## Cleaning

- `$ git clean` : Working directory 안의 추적하지 않던 모든 파일 삭제
- `.gitignore`에 등록된 파일처럼 무시하는 파일은 삭제하지 않음 (`-x` 옵션으로 삭제 가능)

## Modifying History

> NOTE. SHA-1 값이 바뀌기 때문에 과거의 커밋을 변경할 때 주의해야 한다. Rebase와 같이 이미 Push 한 커밋은 수정하면 안 된다.

- `$ git commit --amend` : 바로 이전 commit을 수정할 때
- `$ git rebase -i HEAD~3` : 옛날 commit도 수정하고 싶을 때
    - `log`와는 역순으로 history를 출력
- `$ git fliter-branch` : 변경 사항이 많을 때 

### Commit 순서 바꾸기 & 삭제 Example

from

```
pick f7f3f6d changed my name a bit
pick 310154e updated README formatting and added blame
pick a5f4a0d added cat-file
```

into

```
pick 310154e updated README formatting and added blame
pick f7f3f6d changed my name a bit
```

- `added cat-file` 삭제
- `changed my name a bit` - `updatedREADME formatting and added blame` 순서 변경

### Collapse

```
pick f7f3f6d changed my name a bit
squash 310154e updated README formatting and added blame
squash a5f4a0d added cat-file
```

### Split

from

```
pick f7f3f6d changed my name a bit
edit 310154e updated README formatting and added blame
pick a5f4a0d added cat-file
```

do

```
$ git reset HEAD^
$ git add README
$ git commit -m 'updated README formatting'
$ git add lib/simplegit.rb
$ git commit -m 'added blame'
$ git rebase --continue
```

- `updated README formmatting and added blame`을 두개로 나눠 `updated README formatting` + `added blame`으로 변경

### Remove Credentials

```
$ git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
Rewrite 6b9b3cf04e7c5686a9cb838c3f36a8cb6a0fc2bd (21/21)
Ref 'refs/heads/master' was rewritten
```

### Modifying Authors

```
$ git filter-branch --commit-filter '
        if [ "$GIT_AUTHOR_EMAIL" = "schacon@localhost" ];
        then
                GIT_AUTHOR_NAME="Scott Chacon";
                GIT_AUTHOR_EMAIL="schacon@example.com";
                git commit-tree "$@";
        else
                git commit-tree "$@";
        fi' HEAD
```
