# Git Internals

> Git is **fundamentally a content-addressable filesystem** with a VCS user interface written on top of it.

> TIP. `git leflog`를 이용하면 `force push`나 `reset --hard`로 날려먹은 commit을 복구할 수 있을지도?

## 10.1 Plumbing and Porcelain

- **Plumbing** Commands : do low-level work and were designed to be chained together UNIX-style or called from scripts.
    - Many of these commands aren’t meant to be used manually on the command line, but rather to be used as building blocks for new tools and custom scripts.
- **Porcelain** Commands : the more user-friendly.

### When you run `git init`

```
$ ls -F1
config
description
HEAD
hooks/
info/
objects/
refs/
```

Depending on your version of Git, you may see some additional content there, but this is a fresh git init repository

- `description` : used only by the GitWeb program, so don’t worry about it.
- `config` : contains your project-specific configuration options.
- `info` : keeps a global exclude file for ignored patterns that you don’t want to track in a `.gitignore` file.
- `hooks` : contains your client- or server-side hook scripts.

This leaves four important entries: the `HEAD` and (yet to be created) `index` files, and the `objects` and `refs` directories.

- `objects` dir : stores all the content for your database
- `refs` dir : stores pointers into commit objects in that data (branches, tags, remotes and more)
- `HEAD` file : points to the branch you currently have checked out
- `index` file : where Git stores your staging area information

## 10.2 Git Objects

Git is a content-addressable filesystem. Great. What does that mean? It means that at the core of Git is a simple key-value data store. What this means is that you can insert any kind of content into a Git repository, for which Git will hand you back a unique key you can use later to retrieve that content.

- `git hash-object` : takes some data, stores it in your `.git/objects` directory (the object database), and gives you back the unique key that now refers to that data object.
    - The output from the above command is a 40-character checksum hash.
    - This is the SHA-1 hash — a checksum of the content you’re storing plus a header, which you’ll learn about in a bit.
- But remembering the SHA-1 key for each version of your file isn’t practical; plus, you aren’t storing the filename in your system — just the content.
    - This object type is called a `blob`. 

```
$ git init test
Initialized empty Git repository in /tmp/test/.git/
$ cd test
$ find .git/objects
.git/objects
.git/objects/info
.git/objects/pack
$ find .git/objects -type f

$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4

$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4

$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content

$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30

$ echo 'version 2' > test.txt
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a

$ find .git/objects -type f
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

### Tree Objects

The next type of Git object we’ll examine is the `tree`, which solves the problem of storing the filename and also allows you to store a group of files together. 
Git stores content in a manner similar to a UNIX filesystem, but a bit simplified. 
All the content is stored as `tree` and `blob` objects, with `tree`s corresponding to UNIX directory entries and `blob`s corresponding more or less to inodes or file contents. 
A single `tree` object contains one or more entries, each of which is the SHA-1 hash of a `blob` or subtree with its associated mode, type, and filename.

You can fairly easily create your own `tree`. Git normally creates a `tree` by taking the state of your staging area or `index` and writing a series of `tree` objects from it. 
So, to create a `tree` object, you first have to set up an `index` by staging some files.

```
$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859      README
100644 blob 8f94139338f9404f26296befa88755fc2598c289      Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0      lib

$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b      simplegit.rb

$ git update-index --add --cacheinfo 100644 \
  83baae61804e65cc73a7201a7252750c76066a30 test.txt

$ git write-tree
d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579
100644 blob 83baae61804e65cc73a7201a7252750c76066a30      test.txt

$ echo 'new file' > new.txt
$ git update-index --cacheinfo 100644 \
  1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
$ git update-index --add new.txt

$ git write-tree
0155eb4229851634a0f03eb265b69f5a2d56f341
$ git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a      test.txt

$ git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git write-tree
3c4e9cd789d88d8d89c1073707c3585e41b0e614
$ git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579      bak
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a      test.txt
```

### Commit Objects

If you’ve done all of the above, you now have three trees that represent the different snapshots of your project that you want to track, but the earlier problem remains: you must remember all three SHA-1 values in order to recall the snapshots. You also don’t have any information about who saved the snapshots, when they were saved, or why they were saved. This is the basic information that the commit object stores for you.

- `git commit-tree` : create a commit object

```
$ echo 'First commit' | git commit-tree d8329f
fdf4fc3344e67ab068f836878b6c4951e3b15f3d

$ git cat-file -p fdf4fc3
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
author Scott Chacon <schacon@gmail.com> 1243040974 -0700
committer Scott Chacon <schacon@gmail.com> 1243040974 -0700

First commit

$ echo 'Second commit' | git commit-tree 0155eb -p fdf4fc3
cac0cab538b970a37ea1e769cbbde608743bc96d
$ echo 'Third commit'  | git commit-tree 3c4e9c -p cac0cab
1a410efbd13591db07496601ebc7a059dd55cfe9

$ git log --stat 1a410e
commit 1a410efbd13591db07496601ebc7a059dd55cfe9
Author: Scott Chacon <schacon@gmail.com>
Date:   Fri May 22 18:15:24 2009 -0700

	Third commit

 bak/test.txt | 1 +
 1 file changed, 1 insertion(+)

commit cac0cab538b970a37ea1e769cbbde608743bc96d
Author: Scott Chacon <schacon@gmail.com>
Date:   Fri May 22 18:14:29 2009 -0700

	Second commit

 new.txt  | 1 +
 test.txt | 2 +-
 2 files changed, 2 insertions(+), 1 deletion(-)

commit fdf4fc3344e67ab068f836878b6c4951e3b15f3d
Author: Scott Chacon <schacon@gmail.com>
Date:   Fri May 22 18:09:34 2009 -0700

    First commit

 test.txt | 1 +
 1 file changed, 1 insertion(+)
```

### Summary

You’ve just done the low-level operations to build up a Git history without using any of the front end commands. 
This is essentially what Git does when you run the `git add` and `git commit` commands — 
it stores `blobs` for the files that have changed, updates the `index`, writes out `trees`, and writes `commit objects` that 
reference the top-level `tree`s and the `commit`s that came immediately before them. 
These three main Git objects — the `blob`, the `tree`, and the `commit` — are initially stored as separate files in your `.git/objects` directory.

```
$ find .git/objects -type f
.git/objects/01/55eb4229851634a0f03eb265b69f5a2d56f341 # tree 2
.git/objects/1a/410efbd13591db07496601ebc7a059dd55cfe9 # commit 3
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a # test.txt v2
.git/objects/3c/4e9cd789d88d8d89c1073707c3585e41b0e614 # tree 3
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30 # test.txt v1
.git/objects/ca/c0cab538b970a37ea1e769cbbde608743bc96d # commit 2
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4 # 'test content'
.git/objects/d8/329fc1cc938780ffdd9f94e0d364e0ea74f579 # tree 1
.git/objects/fa/49b077972391ad58037050f2a75f74e3671e92 # new.txt
.git/objects/fd/f4fc3344e67ab068f836878b6c4951e3b15f3d # commit 1
```

![image](https://github.com/user-attachments/assets/90e1ef3e-93f9-405d-b157-bf847918b157)

## References

- Readable name for SHA-1 value
- Stored in `.git/refs` directory
- `branch` names, `HEAD`, `tag`s, `remote`s

```
$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/tags
$ find .git/refs -type f
```

### Packfiles

- Object는 기본적으로 File Content를 저장 (`loose objects`)
- 아무리 Git이 `zlib`로 압축을 한다 하더라도, 만약 File의 크기가 크다면 Disk를 너무 비효율적으로 사용하는게 아닐까?
    - `Packfile`이 바로 저장 공간을 효율적으로 사용하기 위한 방법
    - 가장 최신 버전만 Object File에 Content를 전부 저장하고, 이전 버전들은 변경 사항만을 저장
- Git does this if you have too many loose objects around, if you run the `git gc` command manually, or if you `push` to a remote server.
  
