---
title: 深入 Git：Git 物件儲存 - blob 物件
date: 2020-02-16 23:56:20
author: Titangene
tags:
  - 深入 Git
  - w3HexSchool
categories:
  - Git
cover_image: /images/cover/git.jpg
---
![](../images/cover/git.jpg)

本篇將深入探討 Git 如何運作，在執行 `git add` 將檔案加入 index 時，Git 會如何建立和儲存 blob 物件。

<!-- more -->

Git 是一個 content-addressable (按內容定址，按檔案內容定位) 的檔案系統，這代表 Git 的核心是一個 key-value data store (資料儲存) [^Git-internals_pro-git]。我們只要提供檔案內容，Git 就會透過一些演算法計算成 key，未來就可透過該 key 來檢索出對應的內容。

[^Git-internals_pro-git]: [Git Internals - Git Objects | Pro Git 2/e](https://git-scm.com/book/zh-tw/v2/Git-Internals-Git-Objects)

Git 有四種 type (類型) 的物件：blob、tree、commit 和 tag。

下面會使用 `git hash-object` 這個底層指令來介紹物件名稱是如何產生的。

## 手動建立 blob 物件

首先，先執行 `git init` 初始化 repo：

```shell
$ git init
Initialized empty Git repository in /home/titan/project/git-demo/.git/
```

初始化新的 repo 時，Git 會在 `.git` 目錄中初始化 `objects` 目錄，並在裡面建立 `pack` 和 `info` 子層的空目錄：

```shell
$ tree .git/objects
.git/objects
├── info
└── pack
```

使用底層指令 `hash-object` 將資料儲存至 `.git` 目錄中，並獲得對應的 key (也就是物件名稱，或稱 SHA-1 值)。下面是 `hash-object` 的 option：
- `-w`：
  - 將物件寫入至物件資料庫 (也就是 `.git/objects` 目錄內)，並輸出該物件的 key (也就是 SHA-1 checksum)
  - 若不用此 option，就只會輸出 key，不會儲存物件
- `--stdin`：
  - 從 stdin (standard input，標準輸入) 讀取內容
  - 若不用此 option，`hash-object` 指令預設會從檔案中讀取，所以必須在 `hash-object` 指令之後加上指定的檔案路徑
    - 例如：`git hash-object README.md`

```shell
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
```

`hash-object` 指令會輸出 40 個字元的 checksum hash，這是個 SHA-1 hash (後面會介紹 [SHA-1](#SHA-1))，是由儲存的內容和 header 資訊所計算出來的 checksum。

在 Git 的儲存方式是一份內容就存成一個檔案，都放在 `.git/objects` 目錄內，子目錄為 SHA-1 的前 2 個字元，檔名為剩餘的 38 個字元。

```shell
$ tree .git/objects
.git/objects
├── d6
│   └── 70460b4b4aece5915caf5c68d12f560a9fe3e4
├── info
└── pack
```

## 使用 `cat-file` 指令察看物件資訊

使用 `cat-file` 指令取得物件的內容，可用於檢查物件。使用 `-p` option 找出內容的 type 並輸出該內容：

```shell
$ git cat-file -p d67046
test content
```

使用 `-t` option 可獲得該物件的 type：

```shell
$ git cat-file -t d67046
blob
```

剛剛建立的 `d67046` 就是 Git 的 blob 物件。

## 變更檔案內容後再次建立 blob 物件

那如果變更檔案內容後，再建立 blob 物件又會如何？看下面範例：

先建立一個全新的專案，並使用 `git init` 初始化 Git repo：

```shell
$ mkdir git-demo-2
$ cd git-demo-2
$ git init
Initialized empty Git repository in /home/titan/project/git-demo-2/.git/
```

接著建立一個名為 `test.txt` 的檔案，內容為 `v1`，使用 `git hash-object` 建立的 blob 物件為 `626799`，後來將內容修改為 `v2`，再次建立的 blob 物件為 `8c1384`：

```shell
$ echo 'v1' > test.txt
$ git hash-object -w test.txt
626799f0f85326a8c1fc522db584e86cdfccd51f

$ git cat-file -t 626799
blob

$ echo 'v2' > test.txt
$ git hash-object -w test.txt
8c1384d825dbbe41309b7dc18ee7991a9085c46e

$ git cat-file -t 8c1384
blob
```

可以看到當檔案內容不同時，就可以產生不同的 blob 物件：

```shell
$ tree .git/objects
.git/objects
├── 62
│   └── 6799f0f85326a8c1fc522db584e86cdfccd51f
├── 8c
│   └── 1384d825dbbe41309b7dc18ee7991a9085c46e
├── info
└── pack
```

## 建立內容相同，但檔名不同的檔案

當你建立檔案內容相同，但檔名不同的檔案時，在 repo 內也只會存一份，因為物件的名稱是由檔案內容來決定的，而不是依檔名決定。

不過，`git hash-object` 指令還是會重新產生同一個 blob 物件 (因為該物件的檔案時間被更新了)。範例如下：

在建立 `other-test.txt` 檔案之前，`.git/objects/8c/6799f...` 的檔案時間是 `23:38`：

```shell
$ tree .git/objects
.git/objects
├── 62
│   └── 6799f0f85326a8c1fc522db584e86cdfccd51f
├── 8c
│   └── 1384d825dbbe41309b7dc18ee7991a9085c46e
├── info
└── pack

$ ls -lR .git/objects
.git/objects:
總計 16
drwxr-xr-x 2 titan titan 4096  2月 16 23:39 62
drwxr-xr-x 2 titan titan 4096  2月 16 23:40 8c
drwxr-xr-x 2 titan titan 4096  2月 16 23:39 info
drwxr-xr-x 2 titan titan 4096  2月 16 23:39 pack

.git/objects/62:
總計 4
-r--r--r-- 1 titan titan 18  2月 16 23:38 6799f0f85326a8c1fc522db584e86cdfccd51f

.git/objects/8c:
總計 4
-r--r--r-- 1 titan titan 18  2月 16 23:39 1384d825dbbe41309b7dc18ee7991a9085c46e

...
```

在建立 `other-test.txt` 檔案之後，`.git/objects/8c/6799f...` 的檔案時間變成 `23:40`：

```shell
$ echo 'v2' > other-test.txt
$ git hash-object -w other-test.txt
8c1384d825dbbe41309b7dc18ee7991a9085c46e

$ tree .git/objects
.git/objects
├── 62
│   └── 6799f0f85326a8c1fc522db584e86cdfccd51f
├── 8c
│   └── 1384d825dbbe41309b7dc18ee7991a9085c46e
├── info
└── pack

$ ls -lR .git/objects
.git/objects:
總計 16
drwxr-xr-x 2 titan titan 4096  2月 16 23:39 62
drwxr-xr-x 2 titan titan 4096  2月 16 23:40 8c
drwxr-xr-x 2 titan titan 4096  2月 16 23:39 info
drwxr-xr-x 2 titan titan 4096  2月 16 23:39 pack

.git/objects/62:
總計 4
-r--r--r-- 1 titan titan 18  2月 16 23:40 6799f0f85326a8c1fc522db584e86cdfccd51f

.git/objects/8c:
總計 4
-r--r--r-- 1 titan titan 18  2月 16 23:39 1384d825dbbe41309b7dc18ee7991a9085c46e

...
```

## `hash-object` 計算物件名稱的演算法

那 Git 的物件名稱 (也就是 SHA-1) 是如何計算的？不同的 Git 物件有不同的計算方式，這邊先說明 blob 物件的部份。

Git 物件都是使用 [zlib](https://www.zlib.net/) 壓縮，物件名稱的 SHA-1 hash 值就是 header 加上檔案內容，演算法如下 [^object-storage-format_Git-user-manual]：

[^object-storage-format_Git-user-manual]: [Git - user-manual Documentation](https://git-scm.com/docs/user-manual#object-details)

```
<type> <content_length>\0<content>
```

前面的 `<type> <content_length>\0` 代表 header，而 `<content>` 代表檔案內容。下面舉個例子：

建立名為 `hello.txt` 的檔案，檔案內容為 `hi`：

```shell
$ echo "hi" >  hello.txt
```

使用 `cat` 指令並加上 `-A` 選項察看檔案內容：
- `-A`：相當於 `-vET` 整合選項
- `-E`：將結尾的斷行字元以 `$` 顯示
- `-T`：將 TAB 字元以 `^I` 顯示
- `-v`：除了換行及 TAB 字元外，使用 `^` 及 `M-` 表示法顯示字元 (列出一些看不出來的特殊字元)

```shell
$ cat -A hello      
hi%
```

檔案內容的結尾 `%` 代表 LF (line Feed，`\n` )，我是在 Linux 建立 `hello.txt` 此檔案的，而 Linux 的換行字元是使用 LF 字元，所以在每一行的結尾才會多了 LF 這個字元。

所以依照上面建立物件名稱的演算法就會像這樣，下面分別代表：
- `blob`：要建立的物件是 blob 物件
- `6`：檔案內容長度
- `\0`：Null 結束字元
- `hello\n`：檔案內容

```shell
blob 6\0hello\n
```

所以如果使用以下指令就能算出此 blob 物件的 SHA-1 hash 值：
- `echo` 的 `-n` 選項：不輸出結尾的換行字元 (trailing newline)
- `sha1sum`：計算 SHA-1 值

```shell
$ echo -n "blob 6\0hello\n" | sha1sum 
ce013625030ba8dba906f756967f9e9ca394464a
```

## 實作 Git 物件名稱演算法
如果用 Python 實作此演算法：

```python
import os
import zlib
from hashlib import sha1

content = 'hello\n'
header = 'blob {}\0'.format(len(content))
store = header + content
hash = sha1(store.encode('utf-8')).hexdigest()

zlib_content = zlib.compress(store.encode('utf-8'))

path = '.git/objects/{}/{}'.format(hash[:2], hash[2:])
os.makedirs(os.path.dirname(path), exist_ok=True)
with open(path, 'wb+') as file:
    file.write(zlib_content)
```

驗證剛剛實作的 blob 物件是否有效：

```shell
$ git cat-file -p ce0136
hello
```

看到檔案內容就代表實作成功 😃。

## 驗證 Git 物件
Git 物件名稱可以用來檢查物件的 type 和物件內容是否一致，如果 Git 物件的檔案名稱或內容被惡意修改，就可以很容易的發現有錯誤。

Git 可用 `git fsck` 指令來如何驗證資料庫中的物件是否有效，如果驗證成功就會像這樣輸出：

```shell
$ git fsck
Checking object directories: 100% (256/256), done.
```

假設我惡意修改剛剛建立的 blob 物件，使用 `git fsck` 指令就會告知該物件有問題：

```shell
$ vim .git/objects/ce/013625030ba8dba906f756967f9e9ca394464a
$ git fsck                                                  
error: inflate: data stream error (incorrect header check)
error: unable to unpack header of .git/objects/ce/013625030ba8dba906f756967f9e9ca394464a
error: ce013625030ba8dba906f756967f9e9ca394464a: object corrupt or missing: .git/objects/ce/013625030ba8dba906f756967f9e9ca394464a
Checking object directories: 100% (256/256), done.
missing blob ce013625030ba8dba906f756967f9e9ca394464a
```

## SHA-1

SHA-1 (Secure Hash Algorithm 1) 是一種 hash (雜湊) 演算法，它可生成 160 bit (20 byte) 的 hash 值，通常以 40 個十六進位 (hex) 的數字表示 [^SHA-1_wiki]。

[^SHA-1_wiki]: [SHA-1 - Wiki](https://zh.wikipedia.org/wiki/SHA-1)

hash 演算法的特性 [^hash-function_wiki]：
- 給定訊息很容易計算出 hash 值
- 很難用已知的 hash 值推算出原始訊息
- 很難修改訊息而 hash 值不變
- 很難讓不同的訊息有相同的 hash 值

[^hash-function_wiki]: [密碼雜湊函式 - Wiki](https://zh.wikipedia.org/wiki/密碼雜湊函數)

所以 Git 有以下優點 [^object-database_git-user-manual]：
- Git 可透過比較物件名稱 (也就是 SHA-1) 來快速確定兩個物件是否相同
- 在每個 repo 中都以相同的方式計算物件名稱，所以儲存在不同 repo 內的相同內容會以相同的名稱來儲存
- 可以透過檢查物件名稱來確認其內容的 SHA-1 hash 值，Git 可在讀取物件時檢查錯誤

[^object-database_git-user-manual]: [Git - user-manual Documentation](https://git-scm.com/docs/user-manual#the-object-database)