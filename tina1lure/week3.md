7.8 고급 병합 (Advanced Merging)
Git에서는 기본 병합 외에도 복잡한 충돌을 해결할 수 있는 고급 병합 옵션을 제공
ex) `ours`, `theirs`, `recursive`와 같은 병합 전략을 사용해 상황에 맞게 충돌을 해결

>>> 'theirs' 전략으로 병합하기
git merge branch_name -s recursive -X theirs
- ours: 현재 브랜치의 변경 사항을 우선시, 상대 브랜치의 변경 사항 무시
- theirs: 상대 브랜치의 변경 사항을 우선시, 현재 브랜치의 변경 사항은 무시


7.9 Rerere (Reuse Recorded Resolution)
Rerere는 이전에 해결한 충돌 기록을 저장해두고 같은 충돌이 반복될 때 자동으로 해결함. 반복적인 충돌 해결을 줄일 수 있음 ->  협업 프로젝트에서 유용할듯

>>>
# Rerere 기능 활성화
git config --global rerere.enabled true

-> Git이 충돌 해결 기록을 보관, 다음에 같은 충돌이 발생할 경우 자동으로 처리

7.11 서브모듈 (Submodules)
서브모듈은 외부 Git 저장소를 현재 프로젝트에 포함할 수 있는 기능, ex) 여러 프로젝트에 공통적으로 사용되는 라이브러리를 메인 프로젝트와 함께 관리하고 싶을 때 사용

>>>
# 서브모듈 추가하기
git submodule add https://github.com/example/example-repo.git path/to/submodule

# 서브모듈 초기화 및 업데이트
git submodule init
git submodule update

서브모듈을 통해 여러 프로젝트 통합적으로 관리 가능 및 각 서브모듈의 버전도 독립적으로 관리 가능


7.12 번들링 (Bundling)
네트워크 연결 없이도 Git 저장소를 공유할 수 있도록 모든 데이터를 하나의 파일로 압축, 오프라인으로 다른 사람에게 프로젝트를 전달할 때 유용

>>>
# 저장소 번들 생성
git bundle create repository.bundle --all

# 번들 파일에서 클론하기
git clone repository.bundle -b main

생성한 `.bundle` 파일은 Git 저장소의 모든 기록을 포함하고 원격 연결 없이도 동일한 저장소를 다른 사람과 공유 가능.


7.13 리플레이스 (Replace)
기존의 특정 커밋이나 객체를 다른 객체로 대체함, 과거 커밋을 수정하지 않고도, 실험적인 변경을 적용하고 싶은 경우 유용

>>>
# 특정 커밋을 새로운 커밋으로 대체하기
git replace <old_commit> <new_commit>

# 대체 커밋 제거하기
git replace -d <old_commit>

-> 코드 히스토리를 유지하면서도 특정 커밋을 가상으로 대체 가능, 이력이 꼬이지 않도록 실험가능

8.2 Git 속성 (Git Attributes)
Git 속성은 파일별로 특정 처리 방법을 설정할 수 있는 기능, ex) 특정 파일은 `diff`나 `merge`에서 제외하거나, 바이너리 파일로 인식하도록 설정가능

>>>
# .gitattributes 파일 예시
*.txt diff=none   # 텍스트 파일에서 diff 제외
*.jpg binary      # 이미지 파일은 바이너리로 인식

파일 형식별로 Git이 처리 방식을 달리하도록 지정할 수 있고 프로젝트 관리에 유용함


8.4 Git-Enforced Policy 예시
Git을 사용해 커밋 메시지 형식을 강제 또는 특정 파일에 대한 접근을 제한하는 정책을 만들 수 있음. ex) 커밋 메시지에 반드시 JIRA 티켓 번호가 포함되도록 할 수 있음.

>>>
# 커밋 메시지 검사 스크립트 예시
#!/bin/sh
if ! grep -q "JIRA-[0-9]\+" "$1"; then
  echo "커밋 메시지에 JIRA 티켓 번호가 필요합니다."
  exit 1
fi

위 코드를 `commit-msg` 훅에 추가하면 커밋 메시지가 특정 형식에 맞지 않을 때 커밋이 거부되도록 설정 가능함.
