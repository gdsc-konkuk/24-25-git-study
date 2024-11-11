# revision selection: 7.1

→ 리비전 조회하는 다양한 방법

### **SHA-1 줄여 쓰기**

`git show 1c002d`

### **브랜치로 가리키기**

`git show topic1`

### **RefLog로 가리키기**

`git reflog`

HEAD가 5번 전에 가리켰던 것

`git show HEAD@{5}`

### **계통 관계로 가리키기**

이름 끝에 `^` (캐럿) 기호를 붙이면 Git은 해당 커밋의 부모를 찾는다.

```jsx
$ git log --pretty=format:'%h %s' --graph
* 734713b fixed refs handling, added gc auto, updated tests
*   d921970 Merge commit 'phedders/rdocs'
|\
| * 35cfb2b Some rdoc changes
* | 1c002dd added some blame and merge stuff
|/
* 1c36188 ignore *.gem
* 9b29157 add open3_detach to gemspec file list
```

`HEAD^` 는 바로 “HEAD의 부모” 를 의미하므로 바로 이전 커밋을 보여준다.

### 범위로 커밋 가리키기

- **Double Dot(..)**

master에는 없지만, experiment에는 있는 커밋

```jsx
$ git log master..experiment
D
C
```

브랜치를 Merge 할 때 Merge 하기 전에 무엇이 변경됐는지 확인하기

`$ git log origin/master..HEAD`

- **Triple Dot(…)**

양쪽에 있는 두 Refs 사이에서 공통으로 가지는 것을 제외하고 서로 다른 커밋만 보여준다.

`log` 명령에 `--left-right` 옵션을 추가하면 각 커밋이 어느 브랜치에 속하는지도 보여줌

```jsx
$ git log --left-right master...experiment
< F
< E
> D
> C
```

# interactive staging: 7.2

→ 대화형 명령

### git add -i

```jsx
$ git add -i
           staged     unstaged path
  1:    unchanged        +0/-1 TODO
  2:    unchanged        +1/-1 index.html
  3:    unchanged        +5/-1 lib/simplegit.rb

*** Commands ***
  1: status     2: update      3: revert     4: add untracked
  5: patch      6: diff        7: quit       8: help
What now>
```

# stash: 7.3

→ 현재 작업 중인 내용을 임시로 저장하여 나중에 다시 적용할 수 있게 해줌(브랜치 변경시)

→ Modified이면서 Tracked 상태인 파일과 Staging Area에 있는 파일들을 보관해두는 장소

```jsx
$ git stash
Saved working directory and index state \
  "WIP on master: 049d078 added the index file"
HEAD is now at 049d078 added the index file
(To restore them type "git stash apply")
```

### **push**

`git stash push` 는 변경된 파일들을 스택에 추가한다. 단순히 git stash라고 입력해도 동일하게 작동한다.

### **apply & pop**

스택에 저장된 변경 사항을 다시 작업 영역에 적용할 때 사용.

- **apply**: git stash apply는 스택에 저장된 변경 사항을 적용하지만, 스택에서는 제거X
- **pop**: git stash pop은 스택에 저장된 변경 사항을 적용하면서 스택에서 제거

### drop

`git stash drop`은 스택에서 지정한 stash를 삭제. 특정 stash를 지정하지 않으면 가장 최근의 stash가 삭제된다.

### git clean(신중히 사용)

워킹디렉토리 청소하기 `git clean`

작업하고 있던 파일을 Stash 하지 않고 단순히 그 파일들을 치워버리고 싶을 때

어떤 일이 일어나는지 미리 알고 싶을 때 `-n` 옵션

# Git 검색 도구: 7.4

### git grep

커밋 트리의 내용이나 워킹 디렉토리의 내용을 문자열이나 정규표현식을 이용해 쉽게 찾을 수 있다.

```jsx
$ git grep -n gmtime_r

compat/gmtime.c:3:#undef gmtime_r
compat/gmtime.c:8:      return git_gmtime_r(timep, &result);
compat/gmtime.c:11:struct tm *git_gmtime_r(const time_t *timep, struct tm *result)
compat/gmtime.c:16:     ret = gmtime_r(timep, result);
compat/mingw.c:826:struct tm *gmtime_r(const time_t *timep, struct tm *result)
compat/mingw.h:206:struct tm *gmtime_r(const time_t *timep, struct tm *result);
date.c:482:             if (gmtime_r(&now, &now_tm))
date.c:545:             if (gmtime_r(&time, tm)) {
date.c:758:             /* gmtime_r() in match_digit() may have clobbered it */
git-compat-util.h:1138:struct tm *git_gmtime_r(const time_t *, struct tm *);
git-compat-util.h:1140:#define gmtime_r git_gmtime_r
```

### git log

히스토리에서 **_언제_** 추가되거나 변경됐는지

```jsx
$ git log -S ZLIB_BUF_MAX --oneline
e01503b zlib: allow feeding more than 4GB in one go
ef49a7a zlib: zlib can only process 4GB at a time

```

라인 로그 검색

```jsx
$ git log -L :git_deflate_bound:zlib.c

commit ef49a7a0126d64359c974b4b3b71d7ad42ee3bca
Author: Junio C Hamano <gitster@pobox.com>
Date:   Fri Jun 10 11:52:15 2011 -0700

    zlib: zlib can only process 4GB at a time

diff --git a/zlib.c b/zlib.c
--- a/zlib.c
+++ b/zlib.c
@@ -85,5 +130,5 @@
-unsigned long git_deflate_bound(z_streamp strm, unsigned long size)
+unsigned long git_deflate_bound(git_zstream *strm, unsigned long size)
 {
-       return deflateBound(strm, size);
+       return deflateBound(&strm->z, size);
 }

commit 225a6f1068f71723a910e8565db4e252b3ca21fa
Author: Junio C Hamano <gitster@pobox.com>
Date:   Fri Jun 10 11:18:17 2011 -0700

    zlib: wrap deflateBound() too

diff --git a/zlib.c b/zlib.c
--- a/zlib.c
+++ b/zlib.c
@@ -81,0 +85,5 @@
+unsigned long git_deflate_bound(z_streamp strm, unsigned long size)
+{
+       return deflateBound(strm, size);
+}
+
```

# bisect: 7.5

git bisect는 두 커밋 사이에서 버그가 발생한 커밋을 찾기 위해 이진 검색을 수행

1. **좋은 커밋(good)과 나쁜 커밋(bad) 사이의 중간 커밋으로 자동 이동**

Git은 두 커밋의 중간에 해당하는 커밋으로 자동 checkout합니다. 코드가 해당 커밋 시점으로 이동되므로, 개발자는 이 시점의 코드 상태를 확인합니다.

1. **문제가 발생하는지 여부 확인 후 상태 표시**
   - 문제가 발생한다면 git bisect bad로 표시하고,
   - 문제가 없으면 git bisect good으로 표시합니다.
2. **이진 검색 반복**

Git은 중간 지점으로 계속 이동하며, 개발자가 문제 발생 여부를 표시해 주는 과정을 반복합니다.

1. **문제의 커밋을 최종적으로 찾아냄**

이러한 과정을 거치면, 점점 범위가 좁아져 최종적으로 문제를 일으킨 커밋을 찾게 됩니다.

이 방식은 수십, 수백 개의 커밋이 있을 때 **효율적으로 문제의 커밋을 빠르게 찾는** 데 유용

# 히스토리 단장하기: 7.6

### git commit —amend 로컬에서만!

마지막 커밋 내용 or 커밋메세지 수정하기
`--no-edit` 옵션을 주면 커밋메세지 에디터 안 뜸

### rebase

이미 중앙 서버에 Push한 커밋을 Rebase하면 XX ‼️

- **대화형 Rebase 사용하기**

`git rebase -i` 지정한 커밋부터 HEAD까지 한 번에 Rebase할 수 있습니다.

각 커밋마다 잠시 멈춰서 메시지를 수정하거나 파일을 추가하는 등의 작업이 가능합니다.

- **마지막 커밋 메시지 수정**

git rebase -i HEAD~3과 같이 실행하여 마지막 세 개 커밋을 수정할 수 있습니다.

편집기가 열리면 각 커밋을 edit로 설정해 멈추게 하고,

`git commit --amend`로 메시지를 수정한 뒤

`git rebase --continue`로 Rebase를 이어갑니다.

- **커밋 순서 바꾸기**

Rebase 스크립트에서 커밋 순서를 조정하거나 삭제할 수 있습니다.

pick 명령어를 통해 원하는 순서대로 커밋을 재배열한 후 저장하면 됩니다.

- **커밋 합치기**

여러 커밋을 하나로 합치려면 squash 옵션을 사용합니다.

pick과 squash를 조합해 원하는 커밋을 병합할 수 있으며, 병합된 커밋의 메시지를 편집할 수 있습니다.

- **커밋 분리하기**

기존 커밋을 edit으로 설정하고 git reset HEAD^ 명령으로 해당 커밋을 되돌린 뒤, 필요한 부분만 여러 번 나눠 커밋합니다.

분리 작업 후 git rebase --continue로 Rebase를 완료합니다.

### filter-branch

→ 수정해야할 커밋이 너무 많아서 rebase 로 수정하기 어려울 때

```jsx
$ git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
Rewrite 6b9b3cf04e7c5686a9cb838c3f36a8cb6a0fc2bd (21/21)
Ref 'refs/heads/master' was rewritten
```

# reset & checkout: 7.7

### **reset**

`git reset`은 브랜치의 위치를 이동시키거나, 작업 영역에서 변경 사항을 되돌리는 데 사용

- `--soft`: 브랜치만 이동하고, staging과 working directory의 변경 사항은 유지
- `--mixed:` 브랜치와 staging 영역을 이동시키고, working directory는 유지
- `--hard`: 브랜치, staging, working directory 모두를 지정한 커밋으로 이동. 이 경우 변경 사항이 완전히 제거되니 주의가 필요!

### **checkout**

브랜치나 특정 커밋으로 이동하거나, 특정 파일의 이전 버전으로 돌아갈 때 사용.

`git checkout <branch_name>`은 해당 브랜치로 이동

`git checkout <commit_id> -- <file>`은 특정 파일의 이전 커밋 상태로 되돌린다.
