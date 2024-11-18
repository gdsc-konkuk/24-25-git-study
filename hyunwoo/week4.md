## **Plumbing & Porcelain**

---

- Git은 기본적으로 **Content-addressable 파일 시스템**으로, VCS 사용자 인터페이스를 제공하는 구조이다.
- Git은 **저수준 명령어**와 **고수준 명령어**로 구분된다.
    - **Plumbing 명령어**: Git 내부 동작과 구조를 직접 제어하는 명령어로, 주로 새로운 도구 개발이나 스크립트 작성에 사용됨.
    - **Porcelain 명령어**: 사용자 친화적으로 설계된 명령어로, checkout, branch, remote 등 일반 사용자들이 주로 사용하는 기능을 포함함.

`git init`명령으로 생성된 .git 디렉토리에는 다음과 같은 핵심 구성요소가 포함된다:

1. **HEAD 파일**: 현재 체크아웃된 브랜치를 가리킴.
2. **index 파일**: Staging Area 정보를 저장.
3. **objects 디렉토리**: 모든 데이터와 콘텐츠를 저장하는 데이터베이스.
4. **refs 디렉토리**: 커밋 객체의 포인터(브랜치, 태그, 리모트 등)를 저장.

Git 저장소의 핵심은 `.git` 디렉토리에 있으며, 이 디렉토리를 복사하기만 해도 저장소가 백업 된다!!

## Git Objects

---

**데이터 저장 (Plumbing 명령어)**

- `git hash-object` 명령어로 데이터를 저장하고 Key를 반환.

    ```bash
    $ echo 'test content' | git hash-object -w --stdin
    d670460b4b4aece5915caf5c68d12f560a9fe3e4
    ```

- `w`: 데이터 실제 저장.
- `-stdin`: 표준 입력으로 데이터 전달.

**데이터 구조**

- 데이터는 `.git/objects` 아래에 저장되며 파일 이름은 데이터와 헤더의 SHA-1 체크섬.
- 해시 첫 2글자는 디렉토리 이름, 나머지 38글자는 파일 이름으로 사용.

    ```bash
    $ find .git/objects -type f
    .git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
    ```


**데이터 조회**

- `git cat-file -p <SHA-1>`: 저장된 데이터를 출력.

    ```bash
    $ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
    test content
    ```


**버전 관리**

- 파일 내용을 여러 버전으로 저장.이후 수정 및 재저장 가능:

    ```bash
    $ echo 'version 1' > test.txt
    $ git hash-object -w test.txt
    83baae61804e65cc73a7201a7252750c76066a30
    ```

    ```bash
    $ echo 'version 2' > test.txt
    $ git hash-object -w test.txt
    1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
    ```


**파일 복원**

- 저장된 버전으로 파일을 복원.

    ```bash
    $ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
    $ cat test.txt
    version 1
    ```


**Blob Object**

- Git은 파일 내용만 저장하며, 이를 **Blob Object**라고 부름. → Tree Object에 구조 및 이름 저장
- 파일 이름은 저장되지 않음.
- `git cat-file -t <SHA-1>`로 개체 유형 확인 가능:

    ```bash
    $ git cat-file -t 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
    blob
    ```


**Tree Object**

```bash
$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859      README
100644 blob 8f94139338f9404f26296befa88755fc2598c289      Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0      lib
```

- `lib`는 디렉토리로 하위 Tree 개체를 가리킴.
- `README`와 `Rakefile`은 Blob 개체(파일 내용)를 가리킴.

**Staging Area에 파일 추가**

Git은 일반적으로 Staging Area(Index)의 상태대로 Tree 개체를 만들고 기록

- `git update-index` 명령어를 사용하여 Staging Area에 파일을 추가
- 아직 Staging Area에 없는 파일이기 때문에 `--add` 옵션을 꼭 줘야 한다

```bash
$ git update-index --add --cacheinfo 100644 \
  83baae61804e65cc73a7201a7252750c76066a30 test.txt
```

- `100644`: 파일 모드(일반 파일).

**Tree Object 생성**

- 현재 Staging Area 상태를 기반으로 Tree Object를 생성

```bash
$ git write-tree
d8329fc1cc938780ffdd9f94e0d364e0ea74f579
```

**새로운 파일 추가**

- 새로운 파일(`new.txt`)을 Staging Area에 추가하고 기존 파일(`test.txt`)을 수정

    ```bash
    $ echo 'new file' > new.txt
    $ git update-index --add --cacheinfo 100644 \
      1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
    $ git update-index --add new.txt
    ```


**Tree Object 재생성**

- 변경된 Staging Area를 기반으로 새로운 Tree Object를 생성

    ```bash
    $ git write-tree
    0155eb4229851634a0f03eb265b69f5a2d56f341
    ```

    - 생성된 Tree Object 확인:

        ```bash
        $ git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341
        100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
        100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a      test.txt
        ```


**Tree Object를 하위 디렉토리로 추가**

- 기존 Tree Object를 하위 디렉토리(`bak`)로 추가

    ```bash
    $ git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
    $ git write-tree
    3c4e9cd789d88d8d89c1073707c3585e41b0e614
    ```

    - 생성된 Tree Object 확인

        ```bash
        $ git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
        040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579      bak
        100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
        100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a      test.txt
        ```


**결과**

- new.txt
- test.txt (version 2)
- bak/
    - test.txt (version 1)

여전히 스냅샷을 불러오려면 SHA-1 값을 기억해야함!

스냅샷에 대한 정보는 commit object에 있음

**첫 번째 commit object 생성**

- `git commit-tree` 명령을 사용해 커밋 개체를 생성

    ```bash
    $ echo 'first commit' | git commit-tree d8329f
    fdf4fc3344e67ab068f836878b6c4951e3b15f3d
    ```


**생성된 commit object 확인**

- `git cat-file -p` 명령으로 커밋 개체 내용을 확인

    ```bash
    $ git cat-file -p fdf4fc3
    tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
    author Scott Chacon <schacon@gmail.com> 1243040974 -0700
    committer Scott Chacon <schacon@gmail.com> 1243040974 -0700
    
    first commit
    ```

- 정보
    - `tree`: 최상위 Tree Obhect의 SHA-1 값.
    - `author` 및 `committer`: 작성자와 커밋자 정보.
    - 커밋 메시지: `first commit`.

**Commit Object 두 개 더 생성**

- 이전 Commit Object를 가리키도록 생성

    ```bash
    $ echo 'second commit' | git commit-tree 0155eb -p fdf4fc3
    cac0cab538b970a37ea1e769cbbde608743bc96d
    $ echo 'third commit' | git commit-tree 3c4e9c -p cac0cab
    1a410efbd13591db07496601ebc7a059dd55cfe9
    ```

- `p` 옵션은 부모 커밋을 지정한다.

이러면 Git history 완성!!

git log 실행하면 잘 보인다..

**요약**

Git은 변경된 파일을 Blob Object로 저장하고 현 Index에 따라서 Tree Object를 만든다.

그리고 이전 Commit Object와 최상위 Tree Object를 참고해서 Commit Object를 만든다.

즉 Blob, Tree, Commit Object가 Git의 주요 Object이고 이 Object는 전부 `.git/objects` 디렉토리에 저장된다.

**Object 생성 방식**

- Ruby쉘로 흉내낸 방법
    - Object의 타입과 크기를 포함한 헤더를 생성 (1)
    - 헤더와 원래 내용을 합쳐서 SHA-1 체크섬을 계산 (2)
    - zlib으로 내용을 압축 (3)
    - zlib으로 압축한 내용을 Object로 저장 (4)

    ```ruby
    
    // 1
    $ irb
    >> content = "what is up, doc?"
    => "what is up, doc?"
    >> header = "blob #{content.length}\0"
    => "blob 16\u0000"
    
    // 2
    >> store = header + content
    => "blob 16\u0000what is up, doc?"
    >> require 'digest/sha1'
    => true
    >> sha1 = Digest::SHA1.hexdigest(store)
    => "bd9dbf5aae1a3862dd1526723246b20206e5fc37"
    
    // 3
    >> require 'zlib'
    => true
    >> zlib_content = Zlib::Deflate.deflate(store)
    => "x\x9CK\xCA\xC9OR04c(\xCFH,Q\xC8,V(-\xD0QH\xC9O\xB6\a\x00_\x1C\a\x9D"
    
    // 4
    >> path = '.git/objects/' + sha1[0,2] + '/' + sha1[2,38]
    => ".git/objects/bd/9dbf5aae1a3862dd1526723246b20206e5fc37"
    >> require 'fileutils'
    => true
    >> FileUtils.mkdir_p(File.dirname(path))
    => ".git/objects/bd"
    >> File.open(path, 'w') { |f| f.write zlib_content }
    => 32
    ```


## **Git Refs**

---

### Git Refs

Git Refs는 **커밋 SHA-1 해시 값을 기억하기 쉽게 저장하는 포인터** 역할을 한다. `.git/refs` 디렉토리에 저장되며, 브랜치와 태그 등이 이에 해당한다. Refs는 특정 커밋을 가리키며, 이를 통해 Git 히스토리를 더 간편하게 관리할 수 있다.

- Refs 디렉토리 구조

```bash
$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/tags
```

**Refs 생성**

- SHA-1 값을 `.git/refs/heads/<브랜치 이름>` 파일에 저장

    ```bash
    $ echo 1a410efbd13591db07496601ebc7a059dd55cfe9 > .git/refs/heads/master
    ```

- Refs를 사용하여 히스토리 확인:

    ```bash
    $ git log --pretty=oneline master
    1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
    cac0cab538b970a37ea1e769cbbde608743bc96d second commit
    fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit
    ```


내부는 단순!

**Refs 안전하게 업데이트**

- `git update-ref` 명령을 사용하여 Refs를 안전하게 업데이

    ```bash
    bash
    코드 복사
    $ git update-ref refs/heads/master 1a410efbd13591db07496601ebc7a059dd55cfe9
    
    ```


### **HEAD**

HEAD파일은 현 브랜치를 가리키는 간접 Refs

```bash
$ cat .git/HEAD
ref: refs/heads/master
```

git checkout test를 실행하면..

```bash
$ cat .git/HEAD
ref: refs/heads/test
```

`git symbolic-ref` 로 안전하게 사용할 수 있다

```bash
// 읽기
$ git symbolic-ref HEAD
refs/heads/master

// 변경
$ git symbolic-ref HEAD refs/heads/test
$ cat .git/HEAD
ref: refs/heads/test
```

### Tag

Commit Object와 유사하나 Tree Object가 아닌 Commit Object를 가리키는 것이 차이점!

- **Lightweight 태그**
    - 단순히 특정 커밋을 가리키는 참조(Refs).
    - 별도의 태그 개체를 생성하지 않음.

```bash
	$ git update-ref refs/tags/v1.0 <커밋_SHA1>
```

- **Annotated 태그**
    - 별도의 태그 개체를 생성하며, 메타데이터 포함.
    - 태그 개체는 커밋을 직접 가리키지 않고, 태그 개체 자체가 커밋을 참조.

```bash
$ git tag -a v1.1 <커밋_SHA1> -m "test tag"

// tag object 조회
$ git cat-file -p 9585191f37f7b0fb9444f35a9bf50de191beadc2
object 1a410efbd13591db07496601ebc7a059dd55cfe9 // 실제 커밋 object
type commit
tag v1.1
tagger Scott Chacon <schacon@gmail.com> Sat May 23 16:48:58 2009 -0700

test tag
```

### Remote

- 리모트 Refs는 원격 저장소(리모트)의 브랜치 정보를 로컬에 저장하는 참조(Refs).
- 서버와 마지막으로 교환한 커밋 정보를 기록하며, `.git/refs/remotes/<리모트명>/<브랜치명>`에 저장됨.
- 읽기 전용 브랜치로, **Checkout 불가**하며, **참조 및 읽기** 용도로 사용.

## Packfile

---

Git은 zlib으로 파일 내용을 압축하기 때문에 저장 공간이 많이 필요하지 않다!

22KB인 repo.rb 파일을 추가해보자

```bash
$ curl https://raw.githubusercontent.com/mojombo/grit/master/lib/grit/repo.rb > repo.rb
$ git checkout master
$ git add repo.rb
$ git commit -m 'added repo.rb'

// 확인
$ git cat-file -p master^{tree}
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob 033b4468fa6b2a9547a70d88d1bbe8bf3f9ed0d5      repo.rb
100644 blob e3f094f522629ae358806b17daf78246c27c007b      test.txt
```

파일을 수정해보자

```bash
$ echo '# testing' >> repo.rb
$ git commit -am 'modified repo a bit'

// 확
$ git cat-file -p master^{tree}
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob b042a60ef7dff760008df33cee372b945b6e884e      repo.rb
100644 blob e3f094f522629ae358806b17daf78246c27c007b      test.txt
```

한 줄만 추가했음에도 전혀 다른 Blob Object를 저장한다!! (“Loose” 개체 포맷)

처음 것과 두 번째 것 사이의 차이점만 저장할 수 없을까?

→ Pack(압축)할 수 있다

Pack 실행

1. git gc 명령
2. remote로 push할 때

```bash
$ git gc
Counting objects: 18, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (14/14), done.
Writing objects: 100% (18/18), done.
Total 18 (delta 3), reused 0 (delta 0)

// 압축 결과
$ find .git/objects -type f
.git/objects/bd/9dbf5aae1a3862dd1526723246b20206e5fc37
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
.git/objects/info/packs
.git/objects/pack/pack-978e03944f5c581011e6998cd0e9e30000905586.idx
.git/objects/pack/pack-978e03944f5c581011e6998cd0e9e30000905586.pack

```

- **압축 효과**
    - Packfile로 압축 시 디스크 사용량이 절반 이하로 감소.
    - 예: 22K Blob 두 개 -> Packfile에서 7K로 축소.
- **Delta 압축**
    - 첫 번째 개체는 변화된 부분만 저장.
    - 최신 개체는 원본 그대로 저장(더 자주 사용되기 때문).
- **Dangling 개체**
    - 어떤 커밋에서도 참조하지 않는 개체는 Packfile에 포함되지 않음.
    - 즉, “what is up, doc?” 과 “test content” 예제에서 만들었던 개체

## Refspec

---

```bash
$ git remote add origin https://github.com/schacon/simplegit-progit
```

위 명령을 실행하면 `origin` 이라는 저장소 이름, URL, Fetch 할 Refspec를 `.git/config` 파일에 추가한다.

```bash
[remote "origin"]
    url = https://github.com/schacon/simplegit-progit
    fetch = +refs/heads/*:refs/remotes/origin/*
```

- 형식 :  `+<src>:<dest>`
- `+`: Fast-forward가 아닌 업데이트도 허용.
- `<src>`: 리모트 저장소의 Refs 패턴.
- `<dst>`: 로컬 저장소의 Refs 패턴.

기본적으로 Git은 `git remote add` 명령으로 생성한 설정을 참고하여 리모트 서버에서 `refs/heads/` 에 있는 Refs를 가져다 로컬의 `refs/remotes/origin/` 에 기록한다.

로컬에서 서버에 있는 master 브랜치에 접근 (`refs/remotes/origin/master`)

아래 명령은 모두 동일하게 작동한다!

```bash
$ git log origin/master
$ git log remotes/origin/master
$ git log refs/remotes/origin/master
```

master 브랜치만 가져오게할 수도 있다

```bash
fetch = +refs/heads/master:refs/remotes/origin/master
```

이 외에도 다양한 설정이 있음..

만약 QA팀이 네임스페이스를 사용하지 않는 브랜치를 리모트에 네임스페이스를 써서 Push 하려면 어떻게 해야 할까?

```bash
$ git push origin master:refs/heads/qa/master
```

Refspec으로 서버에 있는 Refs를 삭제할 수도 있다!!

```bash
$ git push origin :topic

$ git push origin --delete topic
```

Refspec의 형식은 <src>:<dst> 이니까 <src> 를 비우고 실행하면 <dst> 를 비우라는 명령이 된다.

## Maintenance and Data Recovery

---

### Maintenance

**gc 명령**

- **`git gc`**: Garbage Collection 명령으로 저장소를 최적화
    - Loose 개체를 Packfile에 압축
    - 오래된 dangling(어떤 Refs도 가리키지 않는) 개체 삭제
- **`git gc --auto`**: Loose 개체가 많거나 Packfile이 많을 때 자동 실행
    - 조건
        - Loose 개체 ≥ 7000개
        - Packfile ≥ 50개
    - `gc.auto`와 `gc.autopacklimit`로 조정 가능

**Refs 압축**

- **`git gc`** 실행 시 `.git/refs`의 파일을 `.git/packed-refs`로 압축.
- Refs 조회 순서:
    1. `.git/refs`에서 검색.
    2. 없으면 `.git/packed-refs`에서 검색.

### Data Recovery

**Commit Recovery**

- **Reflog 사용**
    - **`git reflog`**: 잃어버린 커밋 기록 확인.
    - **`git log -g`**: Reflog를 `git log` 형태로 출력.
    - 잃어버린 커밋을 가리키는 브랜치 생성.

```bash
$ git branch recover-branch <SHA>
```

- **Reflog 없이 복구**
    - **`git fsck --full`**
        - 데이터베이스의 Integrity 검사.
        - dangling 개체(잃어버린 커밋 포함)를 표시.
    - dangling 커밋의 SHA-1로 브랜치 생성해 복구.

### **Removing Objects**

Clone할 때 모두 다 내려받는게 문제가 될 수도 있음!

누군가 매우 큰 바이너리 파일을 넣어버리면..?

삭제해서 커밋을 올려도 history에는 남아있어서 계속 포함되는 것…

특히나 Subversion이나 Perforce 저장소를 변환할 때 문제가 됨

주의할 점은 이 작업은 커밋 히스토리를 망쳐버릴 수도 있다는 거..

- **문제 개체 찾기**
    - **`git verify-pack`**: Packfile 내 개체 크기 확인.
    - **`git rev-list --objects --all | grep <SHA>`**: 개체의 파일명 확인.
- **히스토리에서 삭제**
    - **`git filter-branch --index-filter`**:
        - 특정 개체를 모든 커밋에서 삭제

    ```bash
    
    $ git filter-branch --index-filter 'git rm --ignore-unmatch --cached <파일>' -- <문제 커밋 범위>
    ```

- **남은 Refs 정리 및 최적화**
    - **`rm -Rf .git/refs/original .git/logs/`**: 남은 Refs와 로그 제거.
    - **`git gc`**: 최적화하여 공간 절약.
- **완전 삭제**
    - **`git prune --expire now`**: Loose 개체 완전 삭제.