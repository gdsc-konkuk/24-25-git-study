### 7.8 [Advanced Merging](https://git-scm.com/book/en/v2/Git-Tools-Advanced-Merging)

작업한 내용을 임시 브랜치에 커밋하거나 이전에 stash를 해두어 충돌을 최대한 피할 수 있으나, 충돌은 언제든지 발생한다. 

충돌시 해결방법

- merge 취소 (이 상황 피하기, 회피력 999999)
    - git merge --abort : 머지하기 전 되돌리기, 워킹 디렉토리에 파일이 존재하면 작동안함
    - git reset --hard HEAD: 워킹 디렉토리도 완전히 되돌리기
- 공백 무시하기
    - merge 취소하고, `git merge -Xignore-space-change <branch>`  후 다시 merge
    - -Xignore-all-space : 모든 공백 무시
    - -Xignore-space-change : 뭉쳐있는 공백을 하나로 취급
- 수동 merge
    
    ```powershell
    $ git show :1:hello.rb > hello.common.rb
    $ git show :2:hello.rb > hello.ours.rb
    $ git show :3:hello.rb > hello.theirs.rb
    $ git ls-files -u
    100755 ac51efdc3df4f4fd328d1a02ad05331d8e2c9111 1	hello.rb
    100755 36c06c8752c78d2aff89571132f3bf7841a7b5c3 2	hello.rb
    100755 e85207e04dfdd5eb0a1e9febbc67fd837c44a1cd 3	hello.rb
    $ dos2unix hello.theirs.rb // dos2unix로 변환 
    $ git merge-file -p \ 
    hello.ours.rb hello.common.rb hello.theirs.rb > hello.rb
    $ git diff -b // 워킹 디렉토리에 무엇이 바뀌었는지 알 수 있음 
    --ours: 무엇이 합쳐졌는지
    --theirs: 가져온 것과 비교해서 무엇이 바뀌었는지
    --base: 양쪽 모두 비교해서 무엇이 바뀌었는지 
    ```
    
    ignore-all-space는 DOS의 개행 문자가 남아서 한 파일에 두 형식의 개행문자가 뒤섞이기 때문에 위 방법이 더 나음
    
- 충돌 파일 checkout
    - --conflict: 충돌 표시된 부분을 교체
    - --ours
    - --theirs
- merge 로그 확인
    - `git log`: 로그에 충돌에 대한 정보가 있다.
        - --merge: 충돌이 발생한 파일이 속한 커밋만 보여줌
        - -p: 충돌난 파일의 변경사항 보여줌
    - Combined Diff 형식
        - ours와 워킹 디렉토리의 차이와 theirs와 워킹 디렉토리사이의 차이를 보여줌
        - git diff나 git log를 통해서 무엇이 어떻게 바뀌었는지 알 수 있음
- merge 되돌리기
    - `git reset --hard HEAD~` : Refs 수정
    - `git revert -m 1 HEAD` : 커밋 되돌리기
- 다른 방식의 Merge
    - Merge는 보통 recursive 전략을 사용함
    - 브랜치를 한번에 merge 하는 방법
        1. Our/Their 선택하기
            - -X: 어떤 한쪽을 선택할 떄 사용 (ex -Xours/-Xtheirs)
        2. 서브트리 merge
            - Git은 서브트리를 찾아서 메인 프로젝트로 서브프로젝트의 내용을 Merge함

### 7.9 [Rerere](https://git-scm.com/book/en/v2/Git-Tools-Rerere)

```powershell
$ git config --global rerere.enabled true //활성화
$ git merge <브랜치>
Auto-merging hello.rb
CONFLICT (content): Merge conflict in hello.rb
Recorded preimage for 'hello.rb'
Automatic merge failed; fix conflicts and then commit the result.

//확인
$ git status // 어떤 파일에서 충돌발생했는지 확인
$ git rerere status // 충돌난 파일 확인
$ git rerere diff // 해결중인 상태 확인
$ git ls-files -u // 이전/현재/대상 버전의 해시 확인

각자 변경한 부분을 적용해서 해결
$ git add hello.rb
$ git commit //Recorded resolution for FILE 메시지 확인 가능
```

Merge 되돌리고 Rebase해서 master 브랜치에 쌓기

```powershell
$ git reset --hard HEAD^
$ git checkout <브랜치>
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: <브랜치> one word
Using index info to reconstruct a base tree...
Falling back to patching base and 3-way merge...
Auto-merging hello.rb
CONFLICT (content): Merge conflict in hello.rb
Resolved 'hello.rb' using previous resolution. // 이미 충돌이 해결됨
Failed to merge in the changes.
Patch failed at 0001 i18n one word

$ git checkout --conflict=merge hello.rb // 충돌이 발생한 시점의 상태로 파일 내용을 되돌리기
$ git rerere // 충돌 ㅂ라생한 코드 자동 해결
$ git add hello.rb
$ git rebase --continue
```
### 7.11 [Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

서브모듈: git 저장소 안에 다른 git 저장소를 디렉토리로 분리해 넣는 것

외부 프로젝트를 관리하거나 하위 프로젝트를 가지는 프로젝트에 유용

```powershell
$ git submodule add <저장소 url> // 하위 디렉토리에 서브모듈 추가
$ git status 
On branch master
Your branch is up-to-date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   .gitmodules // 서브모듈 관리 파일: 이걸 보고 어떤 서브모듈 프로젝트가 있는지 확인 가능
    new file:   DbConnector
    
$ git diff --cached --submodule // 더 자세하게 설명해줌
$ git commit -am 'added DbConnector module' // 커밋
$ git push origin master // 끝
```
