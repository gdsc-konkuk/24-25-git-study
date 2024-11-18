>git internals (2024. 11. 18.)
# 10 Git의 내부
- Git이 어떻게 구현돼 있고 내부적으로 어떻게 동작하는지  

>Git은 기본적으로 Content-addressable 파일 시스템이고 그 위에 VCS 사용자 인터페이스가 있는 구조다.  
- **Content-addressable 파일 시스템**
- **데이터 전송 원리**
- **저장소를 관리하는 법**

## 10.1 Plumbing and Porcelain
- Porcelain 명령어
  - 사용자가 일반적으로 사용하는 높은 수준의 명령어.
  - 사용자가 편리하게 Git을 사용할 수 있도록 제공되는 "외부 인터페이스"에 해당.
  - e.g., `git add`
- Plumbing 명령어
  - 저수준의 명령어. → Git의 내부구조에 접근할 수 있고 실제로 왜, 그렇게 작동하는지도 살펴볼 수 있다.
  - Git의 핵심 동작을 구성하는 "배관"에 해당하며, 개발자나 스크립트에서 주로 사용.

## 10.2 Git Objects
>Git은 Content-addressable 파일시스템이다. 이게 무슨 말이냐 하면 Git의 핵심은 단순한 Key-Value(역주 - 예, 파일 이름과 파일 데이터) 데이터 저장소라는 것이다. 어떤 형식의 데이터라도 집어넣을 수 있고 해당 Key로 언제든지 데이터를 다시 가져올 수 있다.

- `git hash-object` (Plumbing 명령어): data를 주면 key값(40자 길이의 체크섬 해시)을 알려준다.
  - `-w` 옵션을 주면 data를 저장하고, 없으면 key값만 알려준다.
- `git cat-file <옵션> <key>`
  - `-p`: 내용을 볼 수 있다.
  - `-t`: 해당 개체가 무슨 개체인지 확인할 수 있다.
  
Git의 개체 모델은 데이터를 관리하기 위해 4가지 기본 개체를 사용한다.

- 블롭(Blob): 파일의 내용만 저장. 이름과 권한은 저장하지 않음.
- 트리(Tree): 디렉토리 구조를 표현하며, 파일 이름과 권한을 포함.
  ![image](https://github.com/user-attachments/assets/af95ea77-10db-4a91-b65a-d2128e146a7e)
- 커밋(Commit): 스냅샷을 누가, 언제, 왜 저장했는지에 대한 정보를 저장.
  - `git commit-tree <Tree 개체의 SHA-1 값>` 명령어로 만든다.
- 태그(Tag): 특정 커밋을 가리키는 고정된 참조점.  

## 10.3 Git References
**Refs**는 브랜치나 태그처럼 커밋을 가리키는 포인터다.

- 구조: `.git/refs`에 저장되고, 브랜치는 `heads/`, 태그는 `tags/`에 위치한다.
- HEAD: 현재 체크아웃된 브랜치를 가리키는 특별한 참조다.
- 관리: 커밋하면 자동 업데이트되고, 이름 변경(git branch -m), 삭제(git branch -d)도 가능하다.

## 10.4 Packfiles
**Packfile**은 저장소를 효율적으로 관리하기 위한 압축된 데이터 파일이다.

- 역할: 객체 파일을 압축해 디스크 공간을 절약하고, 빠른 데이터 전송을 지원한다.
- 구조: 여러 Git 객체를 하나의 파일로 묶어서 저장한다.
- 생성:
  - `git gc` 명령어로 자동 생성.
  - `git repack`으로 수동 생성 가능.
- 사용: 기존 객체와의 차이를 기반으로 델타 압축을 사용해 중복을 최소화한다.

## 10.5 The Refspec
**Refspec**은 원격 저장소와 로컬 저장소 간에 데이터를 동기화할 때 브랜치나 태그를 매핑하는 규칙이다.

- 역할: `git push`와 `git fetch` 시 어떤 참조를 동기화할지 정의한다.
- 구조: `<source>:<destination>` 형식.
  - 예: `refs/heads/main:refs/remotes/origin/main`
- 사용:
  - `Push`: 로컬 브랜치를 원격 브랜치에 동기화.
  - `Fetch`: 원격 브랜치를 로컬에 가져옴.
- 명령어 옵션:
  - `+`로 강제 업데이트(+refs/heads/main:refs/remotes/origin/main).
  - 생략 시 기본 매핑이 적용됨.

## 10.7 Maintenance and Data Recovery
### 운영
```
$ git gc --auto
```

### 데이터 복구
- `git reflog` & `git log -g`: 가장 쉬운 방법
- `$ git fsck --full`: reflog도 삭제한 경우 이용 가능하다.

### 개체 삭제
>주의: 이 작업을 하다가 커밋 히스토리를 망쳐버릴 수 있다.
- `git verify-pack`: 용량이 큰 파일(삭제하고 싶은 파일)의 해시값을 찾아낼 수 있다.
- `git rev-list --objects --all | grep <해시>`로 파일 이름을 알아낼 수 있다.
- `git log --oneline --branches -- <파일 이름>`: 파일을 추가한 커밋을 찾을 수 있다.
- ```
  $ git filter-branch --index-filter \
  'git rm --ignore-unmatch --cached git.tgz' -- 7b30847^..
  ```
  - `rm file` vs `git rm --cached`

  | 명령어                | 디스크에서 삭제 | Git 인덱스에서 삭제 | 속도  | 주요 활용       |
  |-----------------------|----------------|---------------------|-------|----------------|
  | `rm file`            | O              | X                   | 느림  | 파일 완전 삭제 |
  | `git rm --cached`    | X              | O                   | 빠름  | Git 추적 해제  |

## scalar
```bash
scalar clone [--single-branch] [--branch <main-branch>] [--full-clone]
	[--[no-]src] <url> [<enlistment>]
scalar list
scalar register [<enlistment>]
scalar unregister [<enlistment>]
scalar run ( all | config | commit-graph | fetch | loose-objects | pack-files ) [<enlistment>]
scalar reconfigure [ --all | <enlistment> ]
scalar diagnose [<enlistment>]
scalar delete <enlistment>
```
큰 깃 레포지토리를 관리하기 위한 명령어.

## maintenance
```bash
git maintenance run [<options>]
git maintenance start [--scheduler=<scheduler>]
git maintenance (stop|register|unregister) [<options>]
```
깃 레포지토리를 최적화 하기 위한 명령어
