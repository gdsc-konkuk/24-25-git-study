> 생소한 명령어들 git book 7.8 ~ 8.4 (2024.11.11)
### 7.8 [Advanced Merging](https://git-scm.com/book/en/v2/Git-Tools-Advanced-Merging)
- **병합 중단** (병합 도중 문제가 발생했을 때):
  ```bash
  git merge --abort
  ```
- **병합 전략**
  - 병합 전략은 `-s` 옵션으로 지정할 수 있다.
  - recursive: 3-way. 지정하지 않아도 깃이 알아서 수행.
  - ours와 theirs: 특정 브랜치의 변경 사항만을 유지하고자 할 때 사용됨.
    - **충돌 발생 시** 현재 브랜치(our branch)의 변경 사항을 우선하여 병합
      ```bash
      git merge -Xours <branch-name>
      ```
    - **충돌 여부와 상관없이** 현재 브랜치의 내용을 그대로 유지하고, 병합이 완료된 것처럼 기록
      ```bash
      git merge -s ours <branch-name>
      ```
- **서브 트리 병합**
  - 다른 레포지토리를 현재 프로젝트의 특정 디렉토리에 통합. 외부 저장소를 가져와 특정 폴더에 넣고, 커밋 기록도 유지.

### 7.9 [Rerere](https://git-scm.com/book/en/v2/Git-Tools-Rerere)
“reuse recorded resolution”이라고 해서 **한 번 해결한 병합 충돌을 기억하고 재사용**하여, 동일한 충돌 발생 시 자동으로 해결한다.
- **Rerere 활성화**
  ```bash
  git config --global rerere.enabled true
  ```
- **충돌 기록 확인**
  ```bash
  git rerere status
  ```
- **해결 상태 확인**
  ```bash
  git rerere diff
  ```
- 충돌 해결 후 커밋 -> `Recorded resolution for <file-name>.` -> merge 되돌리고 rebase한 후에 다시 merge하면 conflict 없이 알아서 해결!

### 7.11 [Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
레포지토리 안에 다른 레포지토리를 넣어서 관리할 수 있게 해준다. 하나의 메인 프로젝트에 여러 외부 프로젝트(라이브러리, 모듈 등)를 종속적으로 관리할 수 있다.
- **서브모듈 추가**
  ```bash
  git submodule add <repository-url>
  ```
  repository-url은 절대경로, 상대경로 모두 가능. `.gitmodules` 파일(서브디렉토리와 하위 프로젝트 URL의 매핑 정보를 담은 설정파일)이 만들어진다.
  서브모듈은 
- **서브모듈 초기화 및 클론**
  ```bash
  git clone --recurse-submodules <repository-url>
  ```
  이 옵션 없이 clone시 서브모듈 디렉토리는 비어 있다.
- **서브모듈 업데이트**
  ```bash
  git submodule update --remote
  ```
  프로젝트의 루트 디렉토리에서 `git pull`을 하는 경우 서브모듈들에 변경사항이 적용되지 않는다.
- **서브모듈 변경 사항 커밋**
  ```bash
  cd <path-to-submodule>
  git checkout <commit-hash>
  cd ..
  git add <path-to-submodule>
  git commit -m "Update submodule"
  ```
- **서브모듈 삭제**
  ```bash
  git submodule deinit -f <path>
  rm -rf .git/modules/<path>
  git rm -f <path>
  ```
- **서브모듈을 일괄적으로 작업**
  ```bash
  git submodule foreach '<command>'
  ```
  `<command>`에 원하는 명령을 입력하여 모든 서브모듈에서 실행할 수 있다.

### 7.12 [Bundling](https://git-scm.com/book/en/v2/Git-Tools-Bundling)
레포지토리의 특정 커밋이나 브랜치들을 하나의 파일로 압축하여 저장하고, 이를 통해 네트워크 없이도 레포지토리를 이동하거나 공유할 수 있게 해주는 기능
- **번들 생성**
  ```bash
  git bundle create <file> <branch>
  ```
- **번들 파일의 내용 확인**
  ```bash
  git bundle list-heads <bundle-file>
  ```

### 7.13 [Replace](https://git-scm.com/book/en/v2/Git-Tools-Replace)
### 8.2 [Git Attributes](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes)
### 8.4 [An Example Git-Enforced Policy](https://git-scm.com/book/en/v2/Customizing-Git-An-Example-Git-Enforced-Policy)
