# # 1~3장 질문 정리

## 깃은 왜 사용하나요?

- 파일의 버전(수정 이력)을 관리하기 위해서
- 여러 명이 협업할 때 충돌없이 개발할 수 있음
- 과거 버전으로 쉽게 돌아갈 수 있음
- 속도가 빠름
- 비선형적인 구조
- 다른 버전관리보다 브랜치 생성/삭제가 매우 빠름

## commit, add, rm의 파일 상태 변화

- `git add`: 파일을 스테이징 영역으로 올림 (tracked 상태로 변화), 다음 커밋에 추가한다
- `git commit`: 스테이징된 파일들을 로컬 저장소에 기록함 (저장소에 저장된 상태로 변화)
- `git rm`: 파일을 스테이징 영역에서 삭제하고, 로컬 저장소에도 반영됨 (tracked 파일을 삭제)

## .gitignore 파일은 왜 사용되나요?

`.gitignore` 파일은 Git이 특정 파일이나 디렉터리를 추적하지 않도록 설정하는 데 사용. 주로 빌드 아티팩트, 로그 파일, 개인 설정 파일 등을 제외할 때 사용.

## - tags 사용법

`git tag`는 커밋을 특정 버전으로 표시하는 데 사용됩니다. 주로 소프트웨어 릴리즈 버전을 명확하게 나타낼 때 쓰며, lightweight tag와 annotated tag 두 종류가 있습니다.

- 생성: `git tag v1.0.0`
- 확인: `git tag`
- 푸시: `git push origin v1.0.0`

## HEAD와 branch는 어떤 역할을 하나요?

- **HEAD**: 현재 체크아웃된 커밋을 가리키는 포인터입니다. HEAD는 작업 중인 브랜치를 나타내기도 합니다.
- **branch**: 코드의 독립된 작업 흐름을 관리하는데 사용됩니다. 브랜치는 서로 다른 작업을 병행할 수 있도록 하며, 메인 브랜치(master/main) 외에도 기능 개발 등을 위해 새로운 브랜치를 만들 수 있습니다.

## commit 사이를 이동하는 방법 reset & checkout

- `git reset`: 커밋 기록을 되돌리고, 현재 작업 중인 파일 상태도 바꿀 수 있습니다. `-hard`, `-soft`, `-mixed` 옵션이 있습니다.
- `git checkout`: 브랜치나 커밋을 이동하여 과거의 특정 커밋 상태로 파일들을 변경할 때 사용됩니다.

## revert, restore, reset의 차이점은 무엇인가요?

- `git revert`: 특정 커밋을 되돌린 새로운 커밋을 생성. 협업 시 안전한 방법.
- `git restore`: 특정 파일을 원래 상태로 복구하거나 스테이징 영역에서 제거할 때 사용됨. 커밋 이력에는 영향 없음.
- `git reset`: 커밋 이력, 스테이징 영역, 워킹 디렉토리 상태를 변경하거나 되돌림. 커밋 기록 자체를 변경할 수 있으므로 주의가 필요함.

## rebase & merge 비교하기

- **merge**: 두 브랜치를 통합할 때 커밋 기록을 모두 유지하며, 새로운 병합 커밋이 생성됩니다.
- **rebase**: 브랜치의 커밋을 다른 브랜치의 최신 커밋 위로 재배치하여 커밋 기록을 더 깔끔하게 유지할 수 있습니다. 주의해서 사용해야 합니다.
  - **이미 공개 저장소에 Push 한 커밋을 Rebase 하지 마라**

## local branch와 remote branch

- **local branch**: 로컬 저장소에서 생성된 브랜치로, 원격 저장소와 독립적으로 작업할 수 있습니다.
- **remote branch**: 원격 저장소에 존재하는 브랜치로, `git fetch`나 `git pull`로 동기화할 수 있습니다.

## .gitconfig 파일을 사용한 유용한 기능들

- **alias**: 자주 사용하는 명령어를 짧게 설정할 수 있습니다. 예: `git config --global alias.co checkout`
- **conditional config**: 특정 디렉토리나 상황에 따라 다른 설정을 적용할 수 있습니다. 예: 작업 디렉토리에 따라 사용자 정보를 다르게 설정.
- **autoCorrect**: 명령어를 잘못 입력했을 때 자동으로 수정해주는 기능입니다. `git config --global help.autocorrect 1`

커밋하지 않고 잃어버린 것은 절대 되돌릴 수 없다ㅏ ~~
