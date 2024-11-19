#### **1. Plumbing과 Porcelain**  
- **Plumbing 명령어**: 내부 작동 방식을 다루는 명령어.  
- `.git` 디렉토리 구조:
  - `HEAD`: 현재 브랜치를 가리킴.
  - `index`: 스테이징 영역 데이터 저장.
  - `objects`: Git 객체 저장소.
  - `refs`: 브랜치와 태그 정보.  

#### **2. Git 객체 (Git Objects)**  
- **Blob**: 파일의 내용만 저장 (파일명은 저장하지 않음).  
  ```bash
  $ echo 'content' | git hash-object -w --stdin
  ```
- **Tree**: 파일명 포함 및 디렉토리 구조 저장.  
  ```bash
  $ git update-index --add 파일
  $ git write-tree
  ```
- **Commit**: 부모 커밋 및 커밋 메시지 포함.  
  ```bash
  $ echo 'message' | git commit-tree tree_hash
  ```

#### **3. Git References**  
- **Refs**: 사람이 읽기 쉬운 이름으로 SHA-1 해시 참조.  
  ```bash
  $ git update-ref refs/heads/branch commit_hash
  ```
- **HEAD**: 현재 작업 중인 브랜치.  
  ```bash
  $ git symbolic-ref HEAD refs/heads/new_branch
  ```

#### **4. Packfile**  
- **Packfile**: 객체들을 하나의 파일로 묶어 저장 공간 최적화.  
  ```bash
  $ git gc
  ```
- 최신 버전을 완전히 저장하고, 이전 버전은 차이만 기록.

#### **5. Refspec**  
- **형식**: `[+]<src>:<dst>`  
  - `+`: fast-forward 여부와 상관없이 업데이트.  
  ```bash
  $ git fetch origin branch:local_branch
  ```

#### **6. 데이터 복구**  
- **reflog**: 커밋 기록 추적 및 복구.  
  ```bash
  $ git reflog
  $ git reset --hard commit_hash
  ```
- **dangling objects**: 연결되지 않은 객체를 찾아 복구 가능.  
  ```bash
  $ git fsck --full
  ```

#### **7. 유지보수 및 최적화**  
- **Git Maintenance**:
  - `commit-graph` 생성.
  - `gc`로 loose objects 정리.
  - `incremental-repack`으로 저장 공간 최적화.  
  ```bash
  $ git maintenance start
  ```
