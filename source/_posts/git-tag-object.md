---
title: 深入 Git：Git 物件儲存 - tag 物件
date: 2020-04-05 23:40:46
author: Titangene
tags:
  - 深入 Git
  - w3HexSchool
categories:
  - Git
cover_image: /images/cover/git.jpg
---

![](../images/cover/git.jpg)

本篇將深入探討 Git tag 是什麼？如何建立 tag？Git 是如何儲存 tag？與 commit 和分支的差別在哪？

<!-- more -->

# Git tag

Git tag 常用於標記某個版本號碼，例如：`v0.5`、`v1.0.1`、`9.1.0-rc.2`、`v2.6.0-beta.3`。

在 Git 中有兩種類型的 tag：lightweight tag (輕量標籤) 和 annotated tag (註解標籤)。

先介紹兩者的共通點：
- 提供更 friendlier 的名稱的標記
- 都會指向某個 Git 物件 (通常為 commit 物件，但也可以是其他物件，也就是 tree 物件、blob 物件，甚至是其他 tag 物件)
  - 只是 lightweight tag 是直接指向某 Git 物件
  - 而 annotated tag 則是透過 tag 物件，間接指向某 Git 物件
- 不會改變 tag 所指向的物件 (除非用 `git tag -f` 替換現有 tag)

接著會說明各自的特性、要如何建立 lightweight tag 和 annotated tag，以及在 Git 中是如何儲存這些 tag 的。

下面會用以下 Git 歷史紀錄的範例來說明：

```shell
$ git log --oneline --decorate
32aa934 (HEAD -> master) feat: c
9b6d639 feat: b
d835d6b feat: a
```

## lightweight tag (輕量標籤)

- 很像不會移動的分支，tag 只會指向一個特定的 Git 物件
- 若只作為暫時的標籤，且不想保留額外資訊，可用 lightweight tag

### 建立 lightweight tag

建立 lightweight tag 時，不要指定 `-a`、`-s` 或 `-m` option (建立 annotated tag 時才會用到，後面會說明)：

```shell
$ git tag <tagname> [<commit> | <object>]
```

例如：

```shell
$ git tag v1.0-lw
```

也可以指定某個 commit：

```shell
$ git tag v0.2-lw 9b6d639
```

### 察看 lightweight tag

若要透過 `git show` 指令察看 lightweight tag，就不會看到該標籤的額外資訊，只會顯示標籤所在的 commit 資訊：

```shell
$ git show v1.0-lw
commit 32aa934a24212ef22cc373e7838c30618952003d (HEAD -> master, tag: v1.0-lw)
Author: titangene <titangene.tw@gmail.com>
Date:   Sun Apr 5 21:36:25 2020 +0800

    feat: c

diff --git a/c b/c
new file mode 100644
index 0000000..f2ad6c7
--- /dev/null
+++ b/c
@@ -0,0 +1 @@
+c
```

### lightweight tag 存在哪？

Git 將 lightweight tag 存在 `.git/refs/tag` 目錄內：

```shell
$ tree .git/refs/tags/
.git/refs/tags/
├── v0.2-lw
└── v1.0-lw

0 directories, 2 files
```

tag ref 的內容就是該 tag 指向的 commit 物件 (也就是該 commit SHA-1 值)：

```shell
$ cat .git/refs/tags/v1.0-lw
32aa934a24212ef22cc373e7838c30618952003d
```

使用 `git cat-file` 來確定此 SHA-1 值真的是 commit 物件：

```shell
$ git cat-file -t 32aa93
commit
$ git cat-file -p 32aa93
tree 04a59185a0c5f4047e4fd3fa87b0c84e671b00ee
parent 9b6d6398e5e020a76f9a0f8d620e676a1decc78a
author titangene <titangene.tw@gmail.com> 1586093785 +0800
committer titangene <titangene.tw@gmail.com> 1586093785 +0800

feat: c
```

### 使用底層指令建立 lightweight tag

使用 `git update-ref` 底層指令建立 lightweight tag 的 ref：

```shell
$ git update-ref refs/tags/v0.1-lw d835d6bf85258f2fe93c582f60f3f148900e0d7f
```

察看該 tag ref 的確是指向剛剛指定的 SHA-1 值：

```shell
$ cat .git/refs/tags/v0.1-lw 
d835d6bf85258f2fe93c582f60f3f148900e0d7f
```

## annotated tag (註解標籤)

- 會在 Git 的資料庫中儲存成完整的物件。它們將被計算校驗碼 (checksummed)
- 包含：
  - 建立標籤的人 (tagger) 的名字、電子郵件和建立日期
  - 紀錄一個標籤訊息 (tagging message)
  - 指標 (也就是 Git 物件)
- 可以簽名 (signed) 及透過 GNU Privacy Guard (GPG) 驗證
- 若想紀錄與 tag 有關的資訊 (例如：建立 tag 的人、時間)，建議使用 annotated tag

### 建立 annotated tag

建立 annotated tag 時，可同時指定以下 option：
- `-a`，`--annotate`：製作一個 unsigned、annotated tag 物件 (未簽名且帶有註解的標籤物件)
- `-m <msg>`，`--message=<msg>`：
  - 指定標籤訊息，此訊息會和此標籤一起儲存
  - 若你沒有為標籤指定訊息，Git 就會開啟編輯器讓你輸入 (很像 `git commit` 沒有加上 `-m` option 那樣)
  - 若加上多組 `-m` option，會將這些值串聯成段落
  - 如果使用 `-m` option 但沒有加上 `-a`、`-s` 或 `-u <keyid>`，則代表是 `-a` (也就是建立 annotated tag)

```shell
$ git tag -a <tagname> -m <msg> [<commit> | <object>]
```

例如：

```shell
$ git tag -a v1.0 -m "version 1.0"
```

也可以指定某個 commit：

```shell
$ git tag -a v0.1 -m "version 0.1" d835d6b
```

### 察看 annotated tag 資訊

使用 `git show` 指令可查看標籤資訊，以及此標籤所標記的 commit 資訊：

```shell
$ git show v0.1
tag v0.1
Tagger: titangene <titangene.tw@gmail.com>
Date:   Sun Apr 5 21:39:01 2020 +0800

version 0.1

commit d835d6bf85258f2fe93c582f60f3f148900e0d7f (tag: v0.1)
Author: titangene <titangene.tw@gmail.com>
Date:   Sun Apr 5 21:36:18 2020 +0800

    feat: a

diff --git a/a b/a
new file mode 100644
index 0000000..7898192
--- /dev/null
+++ b/a
@@ -0,0 +1 @@
+a
```

以上面標籤資訊為例，包含：
- 標籤名稱：`v0.1`
- 建立標籤的人 (tagger) 的名字、電子郵件：`Tagger: titangene <titangene.tw@gmail.com>`
- 標籤建立日期：`Date:   Sun Apr 5 21:39:01 2020 +0800`
- 標籤訊息：`version 0.1`
- 指標：在 `git show` 沒有提供，下面會提到

### annotated tag 存在哪？

Git 也將 annotated tag 存在 `.git/refs/tag` 目錄內：

```shell
$ tree .git/refs/tags/
.git/refs/tags/
├── v0.1
├── v0.1-lw
├── v0.2-lw
├── v1.0
└── v1.0-lw

0 directories, 5 files
```

tag ref 的內容就是該 tag 指向的 commit 物件 (也就是該 commit SHA-1 值)：

```shell
$ cat .git/refs/tags/v0.1
530097c078c2fa4c16a532443170a40179599a69
```

使用 `git cat-file` 來確定此 SHA-1 值時，會發現跟剛剛介紹的 lightweight tag 不一樣，lightweight tag 會直接指向 commit 物件，而 annotated tag 則是指向 tag 物件：

```shell
$ git cat-file -t 530097
tag
$ git cat-file -p 530097
object d835d6bf85258f2fe93c582f60f3f148900e0d7f
type commit
tag v0.1
tagger titangene <titangene.tw@gmail.com> 1586093941 +0800

version 0.1
```

而這個 tag 物件才是真正指向 commit 物件的 ref。

再次使用 `git cat-file` 指令，就能確定 tag 物件的確是指向 `d835d6` 這個 commit 物件：

```shell
$ git cat-file -t d835d6
commit
$ git cat-file -p d835d6
tree aaff74984cccd156a469afa7d9ab10e4777beb24
author titangene <titangene.tw@gmail.com> 1586093778 +0800
committer titangene <titangene.tw@gmail.com> 1586093778 +0800

feat: a
```

### 使用底層指令建立 annotated tag

建立 annotated tag 主要有兩個步驟：
1. 建立 tag 物件
2. 建立 tag ref，此 tag ref 會指向剛剛建立的 tag 物件

我之前在 [深入 Git：Git 物件儲存 - blob 物件](https://titangene.github.io/article/git--blob-object.html#hash-object-計算物件名稱的演算法) 這篇有提到建立 Git 物件的底層指令 `git hash-object`，以及這個底層指令的原理。

但是在使用 `git hash-object` 之前，必須要先準備好這個指令上的 `--stdin` option 要從 stdin (standard input，標準輸入) 讀取的內容，內容格式如下：

```
object <object-sha1>
type <object-type>
tag <tag-name>
tagger <tagger-name> <<tagger-email>> <timestamp>

<tag-message>
```

內容範例如下 (我先隨意將以下內容儲存在一個檔案內，任意檔名都可以，因為重點是檔案的內容)：

```shell
$ vim my-tag.txt
$ cat my-tag.txt
object 61780798228d17af2d34fce4cfbdf35556832472
type blob
tag file-b
tagger titangene <titangene.tw@gmail.com> 1586097930 +0800

file b
```

因為這次要建立的是 tag 物件，所以需要在 `git hash-object` 指令上加上 `-t` option，並且指定 `tag`，代表要建立 tag 物件：

```shell
$ git hash-object -t tag -w --stdin < my-tag.txt
8785f7a44e2af5628544ecb0aa9dccf2adfa37ff
```

這邊輸出的 `8785f7` 就是剛剛建立的 tag 物件的 SHA-1 值，我們用 `git cat-file` 來驗證一下：

```shell
$ git cat-file -t 8785f7
tag
$ git cat-file -p 8785f7
object 61780798228d17af2d34fce4cfbdf35556832472
type blob
tag file-b
tagger titangene <titangene.tw@gmail.com> 1586097930 +0800

file b
```

接著處理第二步驟，使用 `git update-ref` 底層指令建立 annotated tag 的 ref：

```shell
$ git update-ref refs/tags/file-b 8785f7a44e2af5628544ecb0aa9dccf2adfa37ff
```

察看該 tag ref 的確是指向剛剛指定的 SHA-1 值 (在這邊就是 tag 物件)：

```shell
$ cat .git/refs/tags/file-b  
8785f7a44e2af5628544ecb0aa9dccf2adfa37ff
```

## 在 `git log` 察看 tag

使用 `git log` 察看剛剛建立的 tag 都貼上哪個 commit 上面 (也就是那些 tag 都指向哪個 commit)：

```shell
$ git log --oneline --decorate
32aa934 (HEAD -> master, tag: v1.0-lw, tag: v1.0) feat: c
9b6d639 (tag: v0.2-lw) feat: b
d835d6b (tag: v0.1-lw, tag: v0.1) feat: a
```

另外提一下 (雖然上面有介紹到)，不管是 lightweight tag 還是 annotated tag，如果是指向 commit 物件之外的其他 Git 物件 (也就是 blob 物件或 tree 物件，甚至是其他 tag 物件)，都無法在 `git log` 的輸出察看，只能透過 `git show` 或是透過 `cat` 和 `git cat-file` 指令來察看該 tag 是指向哪個 Git 物件。

下面以 annotated tag 為例，透過這些指令來找到名為 `file-b` 的 tag 指向 SHA-1 值為 `8785f7` 的 tag 物件，而該 tag 物件是指向 SHA-1 值為 `617807` 的 blob 物件：

```shell
$ git tag -a file-c -m "file c" f2ad6c

$ cat .git/refs/tags/file-c
febbaa9a5276f0321d72232c4bc3aa561bfc38c5

$ git cat-file -t febbaa
tag
$ git cat-file -p febbaa
object f2ad6c76f0115a6ba5b00456a849810e7ec0af20
type blob
tag file-c
tagger titangene <titangene.tw@gmail.com> 1586100520 +0800

file c
```

## 察看所有 tag

執行 `git tag` 指令可以察看所有建立的 tag：

```shell
$ git tag
file-b
file-c
v0.1
v0.1-lw
v0.2-lw
v1.0
v1.0-lw
```

# commit vs tag

- commit 物件紀錄的指標只能是 tree 物件，不能是其他 Git 物件
- tag 物件通常紀錄的指標是 commit 物件，當然也可以是其他 Git 物件，包括 tree 物件、blob 物件，甚至是其他 tag 物件

# 分支 vs tag

那分支和 tag 都可以指向 commit 物件，那他們有何差別？
- 分支：在提交新的 commit 時，會將分支指向該分支的末端，也就是新的 commit 物件
- lightweight tag 和 annotated tag 這兩種 tag 都是永遠不會改變 tag 所指向的物件，只是提供更 friendlier 的名稱的標記而已
  - lightweight tag：很像不會移動的分支，此 tag ref 只會指向一個特定的 Git 物件
  - annotated tag：此 tag ref 會指向 tag 物件，而 tag 物件會指向一個特定的 Git 物件

資料來源：
- [Git Internals - Git References | Pro Git, 2/e](https://git-scm.com/book/zh-tw/v2/Git-Internals-Git-References)
- [Git 基礎 - 標籤 | Pro Git, 2/e](https://git-scm.com/book/zh-tw/v2/Git-基礎-標籤)
- [hash - What is the format of a git tag object and how to calculate its SHA? - Stack Overflow](https://stackoverflow.com/questions/10986615/what-is-the-format-of-a-git-tag-object-and-how-to-calculate-its-sha)