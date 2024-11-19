# Session 3 정리

## Advanced Merging

---

### **Aborting a Merge**

```bash
$ git status -sb
## master
UU hello.rb

$ git merge --abort

$ git status -sb
## master
```

- 충돌 발생 시 `git merge --abort` 명령어로 Merge 이전 상태로 되돌릴 수 있음
- 만약 Merge를 완전히 초기화하려면 `git reset --hard HEAD` 명령어 사용
    - 주의: 워킹 디렉토리에서 저장되지 않은 변경 사항이 삭제됨

### **Ignoring Whitespace**

```bash
$ git merge -Xignore-space-change whitespace
Auto-merging hello.rb
Merge made by the 'recursive' strategy.
 hello.rb | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

공백 차이로 인한 충돌은 다음 옵션으로 해결 가능

- `Xignore-all-space`: 모든 공백 변경 무시
- `Xignore-space-change`: 연속된 공백을 하나로 간주

### **Manual File Re-merging**

- Git이 자동으로 충돌을 해결하지 못할 경우, 세 파일 버전(`ours`, `theirs`, `common ancestor`)을 추출하여 수동으로 수정
- 수정 후 `git merge-file` 명령으로 파일을 병합

### **Checking Out Conflicts**

- `git checkout --conflict=diff3` 명령으로 충돌 표시 스타일을 변경
    - `diff3` 스타일은 `ours`, `theirs`와 함께 `base` 버전까지 표시
- 글로벌 설정

```bash
$ git config --global merge.conflictstyle diff3
```

### **Merge Log**

- 충돌의 원인을 분석하려면 `git log --merge` 명령 사용
- 두 브랜치의 모든 커밋을 보려면 Triple Dot 문법 활용
- `merge` 대신 `-p` 를 사용하면 충돌 난 파일의 변경사항만 볼 수 있다
    
    ```bash
    $ git log --oneline --left-right --merge
    < 694971d update phrase to hola world
    > c3ffff1 changed text to hello mundo
    ```
    

### **Combined Diff Format**

- `git diff --cc` 명령을 사용해 Combined Diff 형식으로 충돌 내용을 확인
- 충돌한 각 파일의 변경 사항과 병합 결과를 상세히 보여줌

### **Undoing Merges**

- 잘못된 Merge를 되돌리는 방법
    - `git reset --hard HEAD~`로 Merge 커밋 삭제
    - `git revert -m 1 HEAD`로 새로운 Revert 커밋 생성
- 되돌린 Merge를 다시 Merge하려면 한 번 더 Revert 실행 가능

### **Reverse the commit**

- Revert로 Merge 취소
    - `git revert -m 1 HEAD`: Merge 커밋의 두 번째 부모 변경 사항 취소
- Revert 후 문제점
    - 기존 Merge 히스토리로 인해 재Merge 불가("Already up-to-date")
- 재Merge 방법
    - 이전 Revert를 다시 Revert: `git revert <Revert 커밋 해시>`
    - 이후 정상적으로 `git merge topic` 실행

## Rerere

---

- “reuse recorded resolution”의 약자로, 충돌 해결 기록을 저장하고 재사용
- 같은 충돌이 반복되면 Git이 자동으로 해결

- 활성화 방법
    
    ```bash
    $ git config --global rerere.enabled true
    ```
    
- **Merge 충돌 해결**
    - 충돌 해결 후 커밋하면 해결 기록이 저장됨.
    - 동일한 충돌 발생 시 자동으로 해결.

여러 번 Merge 하거나, Merge 커밋을 쌓지 않으면서도 토픽 브랜치를 master 브랜치의 최신 내용으로 유지하거나, Rebase를 자주 한다면 사용하는게 좋음!!

## Submodules

---

- 한 Git 저장소 안에 다른 Git 저장소를 서브디렉토리로 포함하는 기능
- 주 프로젝트와 독립적으로 관리하면서 서브모듈 프로젝트를 사용할 수 있음

**서브모듈 추가**

```bash
$ git submodule add <URL>
```

- `.gitmodules` 파일에 서브모듈 경로와 URL이 기록됨
- `git commit`으로 서브모듈을 추가하고 변경사항을 저장

**서브모듈 포함 프로젝트 클론**

- 서브모듈 포함 프로젝트를 클론하면 디렉토리는 복제되지만 서브모듈은 빈 디렉토리로 남음
- 초기화와 업데이트 필요
    
    ```bash
    $ git submodule init
    $ git submodule update
    ```
    
- 클론 시 서브모듈을 자동으로 초기화 및 업데이트
    
    ```bash
    $ git clone --recurse-submodules <URL>
    ```
    

**서브모듈 작업**

- 최신 커밋 가져오기
    
    ```bash
    $ git submodule update --remote
    ```
    
- 특정 브랜치를 추적하려면 `.gitmodules`에 브랜치를 설정
    
    ```bash
    $ git config -f .gitmodules submodule.<name>.branch <branch>
    ```
    
- 서브모듈의 모든 디렉토리에서 명령 실행
    
    ```bash
    $ git submodule foreach '<command>'
    ```
    
- 유용한 alias 설정
    
    ```bash
    $ git config alias.sdiff '!'"git diff && git submodule foreach 'git diff'"
    $ git config alias.spush 'push --recurse-submodules=on-demand'
    $ git config alias.supdate 'submodule update --remote --merge'
    ```
    
    위와 같이 설정하면 `git supdate` 명령으로 간단히 서브모듈을 업데이트할 수 있고 `git spush` 명령으로 간단히 서브모듈도 업데이트가 필요한지 확인하며 메인 프로젝트를 Push 할 수 있다.
    

## Bundling

---

- Git 데이터를 파일 하나에 묶어 저장소 히스토리를 공유할 수 있는 기능
- 네트워크가 불안정하거나 저장소 접근이 어려운 경우, 파일을 통해 데이터 전송 가능

**Bundle 생성**

```bash
$ git bundle create <파일명> <범위>
```

- `HEAD`와 브랜치를 지정해 모든 데이터를 포함하거나, 특정 커밋 범위만 포함 가능

```bash
$ git bundle create commits.bundle master ^9a466c5
```

**Bundle에서 데이터 가져오기**

- 데이터를 특정 브랜치로 가져오기:
    
    ```bash
    $ git fetch <파일명> <브랜치>:<새 브랜치>
    ```
    
- `master` 브랜치를 `other-master`로 가져오기
    
    ```bash
    $ git fetch commits.bundle master:other-master
    ```
    

## Replace

---

- Git 객체를 실제 변경하지 않고, 특정 객체를 다른 객체처럼 보이게 하는 기능
- 커밋 히스토리를 재작성 없이 연결하거나 수정할 때 유용
    - 프로젝트의 방대한 히스토리를 두 개로 나눔
        - **원래 히스토리**: 모든 커밋 유지
        - **새 히스토리**: 최근 커밋 몇 개만 유지
    - `git replace`로 두 히스토리를 연결해 표시

- **원래 히스토리 유지 브랜치 생성**
    
    ```bash
    $ git branch history <기준 커밋>
    $ git push <리모트> history:master
    ```
    
- **새 히스토리 생성**
    - 기존 히스토리 중 필요한 부분만 남기고 새 커밋 생성
        
        ```bash
        $ echo 'get history from blah blah blah' | git commit-tree <트리 해시>
        ```
        
    - 새 커밋에 Rebase
        
        ```bash
        $ git rebase --onto <새 커밋> <기준 커밋>
        ```
        
- **Replace로 히스토리 연결**
    
    ```bash
    $ git replace <새 커밋> <원래 커밋>
    ```
    

실제 객체는 변경되지 않으므로 안전하게 원래 데이터를 유지!
