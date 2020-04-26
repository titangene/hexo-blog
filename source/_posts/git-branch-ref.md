---
title: 深入 Git：分支 refs
date: 2020-03-22 18:49:50
author: Titangene
tags:
  - 深入 Git
  - w3HexSchool
categories:
  - Git
cover_image: /images/cover/git.jpg
---

![](../images/cover/git.jpg)

本篇將深入探討 Git 分支到底是什麼？建立分支時到底建立了什麼？如何紀錄分支要指向哪個 commit？

<!-- more -->

Git 參考 (references 或 refs) 都存在 `.git/refs` 目錄內，目錄結構如下：

```shell
$ tree .git/refs/
.git/refs/
├── heads
└── tags

2 directories, 0 files
```

因為「[深入 Git](https://titangene.github.io/tags/深入-git/)」此文章系列都是探討如何用 Git 的底層指令來模擬 Git 高階指令的操作。在之前建立 [blob](https://titangene.github.io/article/git--blob-object.html)、[tree](https://titangene.github.io/article/git-tree-object.html) 和 [commit](https://titangene.github.io/article/git-commit-object.html) 物件時，都還沒建立 Git refs，所以 `.git/refs` 目錄內的資料夾中，才沒有其他紀錄指標的檔案。

若用平常使用的高階指令就會幫你建立很多 Git refs，目錄可能像下面這樣：

```shell
$ tree .git/refs/
.git/refs/
├── heads
│   ├── develop
│   └── master
├── original
│   └── refs
│       └── heads
│           └── master
├── remotes
│   └── origin
│       ├── develop
│       └── master
└── tags
    └── v1.0
```

那我們就開始試著利用底層指令來手動建立 Git refs 吧！

## 建立分支 ref

若沒有 Git refs，就要像這樣指定 commit 的 SHA-1 值才能看到歷史紀錄：

```shell
$ git log --oneline --decorate 200c91
200c91c second commit
9079eb3 first commit
```

若有一個指標指向該 commit 的 SHA-1 值就不用再記住了，省很多麻煩。

而 `master` 就是最常見的指標，要建立分支的指標其實很簡單，只要在 `.git/refs/heads` 目錄內，建立該分支名稱的檔案，而檔案內容就是你要指定的 commit SHA-1 值，例如：我要將 `master` 分支指向 `200c91` 這個 commit 上，可執行以下指令建立分支 ref：

```shell
$ echo 200c91c4acb82e529fb8205ea786a17cc10008b6 > .git/refs/heads/master
```

建立分支後，就可以使用剛剛建立的 head ref (分支頂端的 commit 的 named reference，通常儲存在 `.git/refs/heads` 目錄中) [^head-ref_git-glossary] 來察看歷史紀錄：

```shell
$ git log --oneline --decorate master
200c91c (HEAD -> master) second commit
9079eb3 first commit
```

[^head-ref_git-glossary]: [Git - gitglossary Documentation](https://git-scm.com/docs/gitglossary#Documentation/gitglossary.txt-aiddefheadahead)

不過，不建議你自己編輯 ref 檔案。Git 提供 `git update-ref` 這個底層指令可讓你更新 ref (可不用輸入完整的 commit SHA-1 值，因為 Git 會自動找到對應的 Git 物件)：

```shell
$ git update-ref refs/heads/master 200c91
```

剛剛建立的 head ref 就是分支。分支只是一個指標 (不是像樹的分岔樹枝)，用來紀錄正在開發的分支的最新 commit (或 head) 的 SHA-1 值：

```shell
$ cat .git/refs/heads/master
200c91c4acb82e529fb8205ea786a17cc10008b6
```

要察看此 repo 有哪些 Git ref，可用 Git 提供的 `git show-ref` 這個底層指令來察看：

```shell
$ git show-ref
200c91c4acb82e529fb8205ea786a17cc10008b6 refs/heads/master
```

所以你在執行 `git branch <branch>` 指令時，Git 基本上會執行 `git update-ref` 指令，將你當前所在分支的最後一個 commit 的 SHA-1 新增至你想建立的新 reference (也就是分支 ref)。

資料來源：
- [Git - Git References | Pro Git, 2/e](https://git-scm.com/book/en/v2/Git-Internals-Git-References)