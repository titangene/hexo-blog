---
title: 深入 Git：HEAD refs
date: 2020-03-29 23:55:33
author: Titangene
tags:
  - 深入 Git
  - w3HexSchool
categories:
  - Git
cover_image: /images/cover/git.jpg
---

![](../images/cover/git.jpg)

本篇將深入探討 Git `HEAD` 是什麼？儲存在 repo 的哪裡？以及跟 `git branch` 和 `git checkout` 之間的關係。

<!-- more -->

上次提到的 [Git 分支 ref](https://titangene.github.io/article/git-branch-ref.html)，可用來紀錄不同的工作過程，例如最常見的 `master` 和 `develop` 分支，`master` 分支通常用來紀錄發佈的版本，而 `develop` 分支通常用來紀錄正在開發的版本。

那 Git 是如何知道你正在哪一個分支？其實是透過 `HEAD` 檔案來紀錄的。這個檔案通常會放在 `.git` 路徑底下。

通常 `HEAD` 檔案是對當前所在分支的 symbolic reference [^symref_git-glossary]，也就是紀錄你正在哪一個分支的檔案：

```shell
$ cat .git/HEAD
ref: refs/heads/master
```

[^symref_git-glossary]: [Git - gitglossary Documentation](https://git-scm.com/docs/gitglossary#def_symref)

如果執行 `git checkout` 來切換分支，Git 就會更新 `HEAD` 檔案：

```shell
$ git checkout
Switched to branch 'test'
$ cat .git/HEAD
ref: refs/heads/test
```

在執行 `git commit` 時，為何當前所在的分支會一直指向最新的 commit？這是因為 `HEAD` 會指向當前分支，而當前分支會指向該分支的末端，也就是說，當執行 `git commit` 時，Git 會建立新的 [commit 物件](https://titangene.github.io/article/git-commit-object.html)，而分支會被更新成指向該分支的末端 (也就是新的 commit 物件)。

通常 `HEAD` 會指向當前分支，若 `HEAD` 檔案的內容為 Git 物件的 SHA-1 值時，則代表 [detached HEAD](https://git-scm.com/docs/git-checkout#_detached_head) 狀態。這種情況會在你使用 `git checkout` 切換至 tag、commit 或遠端分支時發生。在此狀態下，沒有分支會與 working tree 關聯。

```shell
$ git checkout 665439e            
Note: checking out '665439e'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 665439e feat: a
```

## 手動編輯 `HEAD` 檔案模擬 checkout 分支

我們來手動編輯此 `HEAD` 檔案，驗證改變 `HEAD` 檔案的內容就等同於在 `git checkout` 至某個分支。

假設目前正在 `test` 分支，然後將 `HEAD` 檔案的內容從 `ref: refs/heads/test` 變成 `ref: refs/heads/master`：

```shell
$ git branch
* test
  master

$ cat .git/HEAD
ref: refs/heads/test

$ echo "ref: refs/heads/master" > .git/HEAD
```

然後用 `git branch` 就能驗證現在已經「`checkout`」至 `master` 分支：

```shell
$ git branch
  test
* master
```

## 使用 `symbolic-ref` 底層指令

雖然剛剛有提到可以手動編輯 `HEAD` 檔案，但建議使用 Git 提供的底層指令 [`git symbolic-ref`](https://git-scm.com/docs/git-symbolic-ref) 來編輯和讀取 `HEAD` 檔案：

讀取 `HEAD` 檔案 (察看當前所在分支)：

```shell
$ git symbolic-ref HEAD
refs/heads/master
```

加上 `--short` option 會只顯示分支名稱，不會顯示分支 ref 的路徑：

```shell
$ git symbolic-ref HEAD --short
master
```

設定 `HEAD` 檔案 (等同於 `git checkout <branch>` )：

```shell
$ git symbolic-ref HEAD refs/heads/test
$ cat .git/HEAD
ref: refs/heads/test
```

一定要用 `refs/` 為開頭的 ref 才能設定 `HEAD` 檔案，要不然會報錯：

```shell
$ git symbolic-ref HEAD test
fatal: Refusing to point HEAD outside of refs/
```

資料來源：
- [Git Internals - Git References | Pro Git, 2/e](https://git-scm.com/book/en/v2/Git-Internals-Git-References)