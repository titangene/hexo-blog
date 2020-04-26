---
title: 深入 Git：Git 物件儲存 - tree 物件
date: 2020-03-01 23:58:39
author: Titangene
tags:
  - 深入 Git
  - w3HexSchool
categories:
  - Git
cover_image: /images/cover/git.jpg
---

![](../images/cover/git.jpg)

本篇將深入探討 Git 如何運作，Git 是如何建立和儲存 tree 物件。

<!-- more -->

之前講到 blob 物件是由檔案內容來產生的，那 Git 是如何知道這些檔案內容是分別存在哪個目錄內的檔案名稱中？

目錄名稱和檔案名稱就是由 tree 物件來管理。一個 tree 物件可以紀錄包含哪些 blob 物件 (也就是檔案內容)，以及該 blob 物件對應的檔案名稱，以及其他 tree 物件和其對應的目錄名稱。例如：

```shell
$ git cat-file -p 
```

Git 儲存內容的方式類似 Unix 檔案系統，所有內容在 Git 都儲存成 tree 物件和 blob 物件，tree 物件對應於 Unix directory entries (目錄項目)，而 blob 物件是對應 inode (記錄檔案的權限與相關屬性) [^file-system_linux-bird] 或檔案內容 [^git-object_pro-git]。

[^file-system_linux-bird]: [鳥哥的 Linux 私房菜 -- 第七章、Linux 磁碟與檔案系統管理](http://linux.vbird.org/linux_basic/0230filesystem.php)
[^git-object_pro-git]: [Git - Git Objects](https://git-scm.com/book/zh-tw/v2/Git-Internals-Git-Objects)

每個 tree 物件可包含一個或多個 tree entries，每個 tree entry 都包含一個指向 blob 物件或 subtree 的 SHA-1 指標 (hash 值) 以及關聯模式 (associated mode)、物件類型、檔名。例如：

先建立一個 commit：

```shell
$ git init
Initialized empty Git repository in /home/titan/project/demo/.git/

$ mkdir styles
$ touch index.html README.md styles/main.css
$ git add .
$ tree .git/objects
.git/objects
├── e6
│   └── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
├── info
└── pack

3 directories, 1 file

$ git commit -m "init"   
[master (root-commit) 158f5a2] init
 3 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README.md
 create mode 100644 index.html
 create mode 100644 styles/main.css
```

`master^{tree}` 代表由 `master` 分支上的最後一次 commit 所指向的 tree 物件：

```shell
$ tree
.
├── index.html
├── README.md
└── styles
    └── main.css

1 directory, 3 files

$ git cat-file -p master^{tree}
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	README.md
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	index.html
040000 tree c6d62294a2323c2047b43787d55a44ff8c94f565	styles
```

`styles` 子目錄是指向另一個名為 `c6d622` 的 tree 物件，而這個 tree 物件又指向另一個名為 `e69de2` 的 blob 物件：

```shell
$ git cat-file -p c6d622
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	main.css
```

## 建立 index

初始化 Git repo：

```shell
$ git init
Initialized empty Git repository in /home/titan/project/git/pro-git/.git/
$ tree .git        
.git
├── branches
├── config
├── description
├── HEAD
├── hooks
│   └── ...
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags

9 directories, 15 files
```

建立專案需要的檔案：

```shell
$ mkdir styles
$ touch index.html README.md styles/main.css
$ tree
.
├── index.html
├── README.md
└── styles
    └── main.css

1 directory, 3 files
```

現在 Git 還未追蹤這些檔案：

```shell
$ git status                      
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	README.md
	index.html
	styles/

nothing added to commit but untracked files present (use "git add" to track)
```

建立 blob 物件：

```shell
$ git hash-object -w index.html
9015a7a32ca0681be64471d3ac2f8c1f24c1040d
$ tree .git/objects            
.git/objects
├── 90
│   └── 15a7a32ca0681be64471d3ac2f8c1f24c1040d
├── info
└── pack

3 directories, 1 file
```

在將 commit 之前，Git 會根據 staging area (預存區或稱暫存區) 或 index 來建立 tree 物件，所以必須在建立 tree 物件之前，先暫存物件來建立 index。

使用 `update-index` 底層指令來為單一檔案建立 index。

因為 index 之前沒有該檔案，甚至連 index 都還沒建立 (也就是沒有 `.git/index` 檔案)，所以必須加上 `--add` 選項。另外，因為要增加的檔案還不在工作目錄中，而是在資料庫 (也就是 `.git/object` 目錄內) 中，所以需要加上 `--cacheinfo` 選項，`--cacheinfo` 選項後面必須指定相關 mode、物件 SHA-1 值、檔名。

:::info
`git update-index` 指令：將 working tree 中的檔案內容註冊至 index
- `--add`：
  - 如果指定的檔案不在 index 中，就會新增該檔案至 index
  - 預設行為 (也就是不加上 `--add` 選項時) 會忽略新檔案
- `--cacheinfo <mode> <object> <path>` 或 `--cacheinfo <mode>,<object>,<path>`：
  - 將指定的資訊直接插入 index
  - 用於註冊不在當前工作目錄中的檔案。對於最小 checkout 合併很有用
  - 為了向後相容，還可將這三個參數作為三個單獨的參數提供，但是鼓勵新的使用者使用單一參數形式
:::

```shell
$ git update-index --add --cacheinfo \
    100644 9015a7a32ca0681be64471d3ac2f8c1f24c1040d index.html

$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   index.html

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	README.md
	styles/
```

這邊指定的 `100644` 是 mode，代表是個普通的檔案。這類似 Unix 的檔案屬性 [^file-permission_linux-bird]，用來表示該檔案的類型 (檔案、目錄、link) 和權限 (可讀、可寫、可執行)，以下是常見的 (用二進位和八進位表示) [^git-file-permission_stack-overflow] [^30-git-09_doggy]：

- `0100000000000000` ( `040000` )：Directory
- `1000000110100100` ( `100644` )：Regular non-executable file
- `1000000110110100` ( `100664` )：Regular non-executable group-writeable file
- `1000000111101101` ( `100755` )：Regular executable file
- `1010000000000000` ( `120000` )：Symbolic link
- `1110000000000000` ( `160000` )：Gitlink

:::info
註：`.git/index` 的二進位表示法 [^index-format_git-repo-github]
- 前 4-bit：物件 type
  - `1000`：regular file
  - `1010`：symbolic link
  - `1110`：gitlink
- 中間 3-bit：沒有使用
- 後面 9-bit：unix 權限
  - regular file 只能用 `0755` 和 `0644`
  - symbolic link 和 gitlink 在此 field 中的值為 `0`

blob 物件只對 `100644`、`100664` 和 `100755` 這三種 mode 有效。
:::

[^file-permission_linux-bird]: [鳥哥的 Linux 私房菜 -- 第五章、Linux 的檔案權限與目錄配置](http://linux.vbird.org/linux_basic/0210filepermission.php#filepermission_perm)
[^git-file-permission_stack-overflow]: [file permissions - How to read the mode field of git-ls-tree's output - Stack Overflow](https://stackoverflow.com/questions/737673/how-to-read-the-mode-field-of-git-ls-trees-output)
[^index-format_git-repo-github]: [git/index-format.txt at master · git/git](https://github.com/git/git/blob/master/Documentation/technical/index-format.txt#L63)
[^30-git-09_doggy]: [30 天精通 Git 版本控管 第 09 天：比對檔案與版本差異 by 保哥](https://github.com/doggy8088/Learn-Git-in-30-days/blob/master/zh-tw/09.md)

接著使用 `write-tree` 指令將 index 寫至 tree 物件中。不需要使用 `-w` 選項，如果該 tree 物件還不存在，則呼叫 `write-tree` 會根據 index 的狀態自動建立 tree 物件：

```shell
$ git write-tree
2dce93ea08ed9059be0a838c6bcf62b7b5c28907
$ git cat-file -p 2dce93
100644 blob 9015a7a32ca0681be64471d3ac2f8c1f24c1040d	index.html
$ git cat-file -t 2dce93
tree
```

如果使用 `index.html` 的第二版和一個新檔案來建立新的 tree 物件：

```shell
$ echo 'text' >> index.html
$ cat index.html 
index
text
$ git update-index index.html
$ git update-index --add README.md
$ git status                           
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   README.md
	new file:   index.html

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	styles/
```

```shell
$ tree .git/objects      
.git/objects
├── 2d
│   └── ce93ea08ed9059be0a838c6bcf62b7b5c28907
├── 70
│   └── f702d2c1492a9989a64c8e68bedf7c81052898
├── 7e
│   └── 59600739c96546163833214c36459e324bad0a
├── 90
│   └── 15a7a32ca0681be64471d3ac2f8c1f24c1040d
├── info
└── pack

6 directories, 4 files

$ git cat-file -p 70f702          
index
text
$ git cat-file -p 7e5960
# README
```

建立 tree 物件 (將 staging area 的狀態或 index 紀錄到一個 tree 物件)：

```shell
$ git write-tree
436d33ce00a9bbf0a1ce763b2c36330b6faf309b
$ git cat-file -p 436d33      
100644 blob 7e59600739c96546163833214c36459e324bad0a	README.md
100644 blob 70f702d2c1492a9989a64c8e68bedf7c81052898	index.html
```