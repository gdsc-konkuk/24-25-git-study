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
  - 다른 저장소를 현재 프로젝트의 특정 디렉토리에 통합. 외부 저장소를 가져와 특정 폴더에 넣고, 커밋 기록도 유지.
### 7.9 [Rerere](https://git-scm.com/book/en/v2/Git-Tools-Rerere)
Git의 Rerere 기능은 “reuse recorded resolution”이라고 해서 **한 번 해결한 병합 충돌을 기억하고 재사용**하여, 동일한 충돌 발생 시 자동으로 해결한다.
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
### 7.12 [Bundling](https://git-scm.com/book/en/v2/Git-Tools-Bundling)
### 7.13 [Replace](https://git-scm.com/book/en/v2/Git-Tools-Replace)
### 8.2 [Git Attributes](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes)
### 8.4 [An Example Git-Enforced Policy](https://git-scm.com/book/en/v2/Customizing-Git-An-Example-Git-Enforced-Policy)
