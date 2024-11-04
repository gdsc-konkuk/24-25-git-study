## Stashing

`git stash`는 현재 작업하던 일을 임시로 저장하기 위한 명령어이다.
`Stash`는 `Modified`이면서 `Tracked` 상태인 파일 과 `Staging Area`에 있는 파일들을 보관해두는 장소다. 
`git add`명령어로 `Staging Area`에 올릴 수 있다.

---

### Git Stash란?
Git Stash는 작업 중인 변경 사항을 임시로 저장해두는 공간입니다. 변경된 파일을 커밋하지 않고 다른 브랜치로 이동하거나 작업을 멈추고 싶을 때 유용합니다.

- **Modified 상태의 Tracked 파일** 및 **Staging Area에 있는 파일들**을 저장
- `git stash`나 `git stash save`를 실행해 새로운 Stash 생성
- `git stash list`로 현재 Stash 목록 확인
- `git stash apply [stash@{번호}]`로 특정 Stash 복원 (가장 최근 Stash가 기본)
- `git stash apply --index` 옵션을 사용해 Staged 상태까지 복원
- `git stash drop`으로 Stash 삭제
- `git stash pop`은 적용 후 Stash에서 바로 삭제

### Stash를 만드는 다양한 방법
- `--keep-index`: Staging Area에 있는 파일을 제외하고 Stash 생성
- `--include-untracked` 또는 `-u`: 추적 중이지 않은 파일도 포함
- `--patch`: 대화형으로 특정 변경 사항만 Stash에 포함

### Git Stash 적용 및 브랜치 생성
- `git stash branch <브랜치 이름>`: Stash를 새로운 브랜치로 적용, 이후 Stash 삭제

### Git Clean으로 워킹 디렉토리 청소하기
Git Clean은 추적되지 않은 파일들을 삭제해 워킹 디렉토리를 정리합니다. 커밋되지 않은 파일을 모두 삭제하고자 할 때 사용합니다.

- `git clean -f -d`: 추적되지 않은 모든 파일을 삭제 (하위 디렉토리 포함)
- `-n` 옵션: 삭제 전 어떤 파일이 삭제될지 미리보기
- `-x` 옵션: .gitignore에 명시된 파일도 포함하여 삭제
- `-i` 옵션: 대화형으로 선택적으로 파일 삭제

> **주의**: Clean 명령은 추적되지 않은 파일들을 영구적으로 삭제하므로 주의해서 사용, 반드시 삭제 전에 확인하려면 `-n` 옵션을 사용해야 한다.
