## 7.1 Revision Selection (리비전 선택)

- **리비전 하나 가리키기**
  - **SHA-1 줄여 쓰기**  
    SHA-1 해시 값은 40글자나 될 정도로 길기 때문에 전부 외우기는 힘들다. 다행히 앞 몇 글자만 외워도 식별이 가능하다.
  
  - **브랜치로 가리키기**  
    어떤 커밋이 브랜치의 가장 최신 커밋이라면 간단히 브랜치 이름으로 커밋을 가리킬 수 있다. 브랜치 이름을 Git 명령에 전달하면 브랜치가 가리키는 커밋을 가리키게 된다.
  
  - **RefLog로 가리키기**  
    Git은 자동으로 브랜치와 HEAD가 지난 몇 달 동안에 가리켰었던 커밋을 모두 기록하는데 이 로그를 “Reflog” 라고 부른다.  
    `git reflog`를 실행하면 Reflog를 볼 수 있다.
  
  - **계통 관계로 가리키기**  
    계통 관계로도 커밋을 표현할 수 있다. 이름 끝에 `^` (캐럿) 기호를 붙이면 Git은 해당 커밋의 부모를 찾는다.  
    `HEAD^`는 바로 “HEAD의 부모” 를 의미하므로 바로 이전 커밋을 보여준다.  
    계통을 표현하는 방법으로 `~` 라는 것도 있다. `HEAD~`와 `HEAD^`는 똑같이 첫 번째 부모를 가리킨다. 하지만, 그 뒤에 숫자를 사용하면 달라진다. `HEAD~2`는 명령을 실행할 시점의 “첫 번째 부모의 첫 번째 부모”, 즉 “조부모” 를 가리킨다.

- **범위로 커밋 가리키기**  
  커밋을 하나씩 조회할 수도 있지만, 범위를 주고 여러 커밋을 한꺼번에 조회할 수도 있다. 범위를 사용하여 조회할 수 있으면 브랜치를 관리할 때 유용하다.

  - **Double Dot**  
    Double Dot은 어떤 커밋들이 한쪽에는 관련됐고 다른 쪽에는 관련되지 않았는지 Git에게 물어보는 것이다.  
    `A..B`는 A 브랜치에는 없고 B 브랜치에만 있는 것을 알려준다.

  - **세 개 이상의 Refs**  
    Double Dot은 간단하고 유용하지만 두 개 이상의 브랜치에는 사용할 수 없다. Git은 `^`이나 `--not` 옵션 뒤에 브랜치 이름을 넣으면 그 브랜치에 없는 커밋을 찾아준다.
    ```bash
    $ git log refA refB ^refC
    $ git log refA refB --not refC

  - **Triple Dot**
    Triple Dot은 양쪽에 있는 두 Refs 사이에서 공통으로 가지는 것을 제외하고 서로 다른 커밋만 보여준다.
    
## 7.2 interactive staging
`git add` 명령에 `-i` 나 `--interactive` 옵션을 주고 실행하면 Git은 대화형 모드로 들어간다.

```bash
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

왼쪽에는 Staged 상태인 파일들을 보여주고 오른쪽에는 Unstaged 상태인 파일들을 보여준다.

그리고 마지막 “Commands” 부분에서는 할 수 있는 일이 무엇인지 보여준다. 파일들을 Stage하고 Unstage하는 것, Untracked 상태의 파일들을 추가하는 것, Stage한 파일을 Diff할 수 있다. 게다가 수정한 파일의 일부분만 Staging Area에 추가할 수도 있다.

## 7.3 Stashing

`git stash`는 현재 작업 디렉토리에서 수정 중인 파일들을 일시적으로 저장하고, 깨끗한 작업 상태로 되돌아가기 위해 사용됩니다. `stash`는 **Modified** 상태이며 **Tracked** 상태인 파일과 **Staging Area**에 있는 파일들을 보관합니다. 브랜치를 변경해야 할 때 작업 중이던 변경 사항을 스택에 저장해두고 나중에 다시 불러와 적용할 수 있습니다.

---

#### 예시

1. 파일을 수정하던 중 브랜치를 변경하고 싶은 상황이 발생합니다.
2. 작업 중인 파일을 커밋하지 않으므로, `stash` 명령어로 임시 저장합니다.
   - `git stash` 또는 `git stash save`를 실행하여 스택에 새로운 stash를 만듭니다.
3. 이제 자유롭게 다른 브랜치로 이동할 수 있으며, 필요할 때 `git stash apply`를 사용해 stash를 다시 적용할 수 있습니다.
   - `git stash apply stash@{2}`처럼 특정 stash를 골라서 적용할 수도 있습니다. 이름을 지정하지 않으면 가장 최근 stash가 적용됩니다.
4. 단, `git stash apply`를 사용해도 이전에 **Staged** 상태였던 파일은 자동으로 **Staged** 상태로 복구되지 않습니다. 원래 상태로 돌아가려면 `--index` 옵션을 사용합니다.

---
- **`git stash push`**
  - `git stash` 또는 `git stash push` 명령어로 변경 사항을 스택에 쌓고, 현재 작업 디렉토리를 깨끗하게 만듭니다.
  - `git stash save "메시지"`처럼 메시지를 추가하여 스택을 구분할 수 있습니다.

- **`git stash apply & pop`**
  - **`git stash apply [stash@{n}]`**: 스택에 저장된 변경 사항을 다시 적용합니다. 인덱스를 지정하지 않으면 최신 stash가 적용됩니다. 스택에 남아 있어 여러 번 적용할 수 있습니다.
  - **`git stash pop [stash@{n}]`**: `apply`와 유사하지만, 적용 후 해당 stash를 스택에서 제거합니다.

- **`git stash drop [stash@{n}]`**
  - 특정 stash를 삭제합니다. `stash@{n}`을 지정하지 않으면 가장 최근 stash가 삭제됩니다.
  - 예시: `git stash drop stash@{1}`

## 7.5 Searching
### **Git Grep**

- 커밋 트리의 내용이나 워킹 디렉토리의 내용을 문자열이나 정규표현식을 이용해 쉽게 찾을 수 있다.
    - 명령을 실행할 때 `-n` 또는 `--line-number` 옵션을 추가하면 찾을 문자열이 위치한 라인 번호도 같이 출력한다.
    - 어떤 파일에서 몇 개나 찾았는지만 알고 싶다면 `-c` 또는 `--count` 옵션을 이용한다.
    - 매칭되는 라인이 있는 함수나 메서드를 찾고 싶다면 `-p` 또는 `--show-function` 옵션을 준다.
    - `--and` 옵션을 이용해서 여러 단어가 한 라인에 동시에 나타나는 줄 찾기 같은 복잡한 조합으로 검색할 수 있다.
- 매우 빠르다. 또한, 워킹 디렉토리만이 아니라 Git 히스토리 내의 어떠한 정보라도 찾아낼 수 있다.

### **Git log searching**

**`git log`**

어떤 변수가 ***어디에*** 있는지를 찾아보는 게 아니라, 히스토리에서 ***언제*** 추가되거나 변경됐는지 찾아볼 수도 있다.

- **`-S` 옵션**
    - “pickaxe(곡괭이)” 옵션이라 한다.
    - 추가된 커밋과 없어진 커밋만 검색할 수 있다.
    - **`$ git log -S** ZLIB_BUF_MAX **--oneline**`
- **`-L` 옵션**
    - 어떤 함수나 한 라인의 히스토리를 볼 수 있다.
    - 함수의 시작과 끝을 인식해서 함수에서 일어난 모든 히스토리를 함수가 처음 만들어진 때부터 Patch를 나열하여 보여준다.

### bisect

커밋 기록들을 이분탐색하며 어떤 커밋에서 처음으로 버그가 발생했는지 찾을 수 있게 해주는 명령어이다.

1. `$ git bisect start` 
2. `$ git bisect bad`: 현재 커밋을 문제 있는 커밋으로 표시한다.
3. `$ git bisect good <커밋 해시>`: 문제가 없는 커밋을 지정한다.

이후 Git은 이진 검색을 통해 중간 커밋들을 체크아웃하면서, 해당 커밋이 문제를 가지고 있는지(`git bisect bad`) 아닌지(`git bisect good`)를 표시하도록 한다. 이렇게 반복하면 문제가 처음 발생한 커밋을 빠르게 찾을 수 있다.

`git bisect reset`을 사용하면 이 과정을 종료하고 원래 브랜치로 돌아간다.

## 7.6 Rewriting History
### **마지막 커밋 수정하기**

- **커밋 메시지만 수정**
    
    `$ git commit --amend` 
    
    자동으로 텍스트 편집기를 실행시켜서 마지막 커밋 메시지를 열어준다. 여기에 메시지를 바꾸고 편집기를 닫으면 편집기는 바뀐 메시지로 마지막 커밋을 수정한다.
    
- **프로젝트 내용을 수정**
    
    파일을 수정하고 git add 명령으로 Staging Area에 넣는다. `git add <파일>`
    
    그리고 `git commit --amend` 명령으로 커밋하면 커밋 자체가 수정되면서 추가로 수정사항을 밀어넣을 수 있다.
    
    > 이때 SHA-1 값이 바뀌기 때문에 과거의 커밋을 변경할 때 주의해야 한다. Rebase와 같이 이미 Push 한 커밋은 수정하면 안 된다.
    > 
- **기존 커밋 메시지를 그대로 유지하면서 커밋 내용을 수정**
    
    `$ git commit --amend --no-edit` 
    

### **커밋 메시지를 여러 개 수정하기**

> 다시 강조하지만 이미 중앙서버에 Push 한 커밋은 절대 고치지 말아야 한다. Push 한 커밋을 Rebase 하면 결국 같은 내용을 두 번 Push 하는 것이기 때문에 다른 개발자들이 혼란스러워 할 것이다.
> 

<aside>

1. **커밋 선택**: `git rebase -i <목록>` 명령을 입력하면, 수정할 커밋 목록이 텍스트 편집기에 표시된다. 이 목록에서 텍스트 편집기로 각 커밋 앞에 있는 `pick`을 `edit`로 변경하여 수정할 커밋을 지정한다.
2. **수정 프로세스 시작**: 파일을 저장하고 편집기를 종료하면, Git은 지정한 커밋에서 중지하고 `Stopped at <commit_hash>` 메시지를 표시한다. 이 상태에서 다음 명령을 실행할 수 있다.
3. **커밋 수정**:
    - `git commit --amend`: 이 명령을 사용해 커밋 메시지를 수정하거나 커밋에 새 파일을 추가한다.
    - 커밋 메시지 또는 내용을 수정한 후, 텍스트 편집기를 닫는다.
4. **Rebase 진행**:
    - 수정이 완료되면 `git rebase --continue` 명령을 입력하여 Rebase 프로세스를 다음 커밋으로 진행한다.
5. **반복 수행**: 원하는 모든 커밋에서 `edit`를 지정했으므로, 각 커밋에 대해 위의 과정(수정 및 `rebase --continue`)을 반복한다.
6. **Rebase 완료**: 마지막 커밋까지 모든 수정이 완료되면 Rebase가 종료된다.
</aside>

### **커밋 순서 바꾸기**

`git rebase -i <목록>` 명령어 입력 후 텍스트 편집기로 원하는 순서대로 바꾸면 된다.

### 커밋 합치기

<aside>

1. 리베이스 시작 `git rebase -i <목록>`
2. **squash 설정**:
    
    텍스트 편집기에서 각 커밋의 `pick`을 `squash`로 변경하여 합치고자 하는 커밋을 설정한다.
    
3. 저장하고 나서 편집기를 종료하면 Git은 3개의 커밋 메시지를 Merge 할 수 있도록 에디터를 바로 실행해준다.
4. 이 메시지를 저장하면 3개의 커밋이 모두 합쳐진 커밋 한 개만 남는다.
</aside>

### 커밋 분리하기

<aside>

1.  `rebase -i` 스크립트에서 해당 커밋을 "edit"로 변경한다.
2. 편집기에서 `edit`로 변경한 후 파일을 저장하고 종료하면, Git은 해당 커밋에서 멈추고 콘솔 프롬프트로 돌아온다.
3. **커밋 해제 및 분리 작업:**
    
    Git 프롬프트에서 `git reset HEAD^` 명령을 실행하여 커밋을 해제한다. 이 명령은 커밋만 되돌리고, 수정된 파일을 `Unstaged` 상태로 남겨둡니다.
    
4. **변경 사항을 분리하여 커밋**:
    - 이제 파일을 나눠서 Stage하고, 각각 커밋을 만듭니다.
    - 예를 들어
        
        ```bash
        $ git add README
        $ git commit -m 'updated README formatting'
        $ git add lib/simplegit.rb
        $ git commit -m 'added blame'
        ```
        
5. 그다음에 파일을 Stage 한 후 커밋하는 일을 원하는 만큼 반복하고 나서 `git rebase --continue` 라는 명령을 실행하면 남은 Rebase 작업이 끝난다.
</aside>

### **filter-branch**

수정해야 하는 커밋이 너무 많아서 Rebase 스크립트로 수정하기 어려울 것 같으면 사용 가능.

## 7.7 Reset Demystified
| Level         | Command                      | HEAD | Index | Workdir | WD Safe? |
|---------------|------------------------------|------|-------|---------|----------|
| **Commit Level** | `reset --soft [commit]`       | REF  | NO    | NO      | YES      |
|               | `reset [commit]`               | REF  | YES   | NO      | YES      |
|               | `reset --hard [commit]`       | REF  | YES   | YES     | NO       |
|               | `checkout <commit>`            | HEAD | YES   | YES     | YES      |
| **File Level**  | `reset [commit] <paths>`       | NO   | YES   | NO      | YES      |
|               | `checkout [commit] <paths>`   | NO   | YES   | YES     | NO       |
