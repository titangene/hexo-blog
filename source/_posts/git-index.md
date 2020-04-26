---
title: 深入 Git：index 檔案
date: 2020-03-08 23:56:40
author: Titangene
tags:
  - 深入 Git
  - w3HexSchool
categories:
  - Git
cover_image: /images/cover/git.jpg
---

![](../images/cover/git.jpg)

`git add` 會將檔案加入 index，究竟 index 到底存在哪？其實，通常會放在 `.git/index`，本篇將深入探討此檔案式如何紀錄有哪些檔案被加入 index。

<!-- more -->

index 是一個二進位檔案，通常放在 `.git/index`，其中包含路徑名稱的排序列表、每個路徑名稱的權限和 blob 物件的 SHA-1 值。而 `git ls-files` 指令可顯示 index 的內容。


## 透過 `git ls-files` 來認識 index

使用 `git ls-files` 底層指令來察看 index 檔案，但指令預設只會顯示有哪些檔名，所以加上一些選項：
- `-c`，`--cached`：預設選項，只在輸出顯示已暫存的檔案
- `-s`，`--stage`：在輸出中顯示 stage 內容的 mode bit、物件名稱和 stage number

```shell
$ git ls-files -s
100644 9015a7a32ca0681be64471d3ac2f8c1f24c1040d 0	index.html
```

從上面輸出可得知：
- `index.html` 的 file mode bits 為 `100644`，代表是 regular non-executable file
- `index.html` 的 blob 物件名稱為 `9015a7...`
- stage number 在合併衝突處理時會用到
- 檔案名稱

file mode bits (6 位數) 是檔案權限的八進位表示法，也可以是以二進位的方式讀取。這邊的 `100644` ( `0b1000000110100100` ) 代表檔案是 regular non-executable file，檔案擁有者可以讀寫，檔案群組中的其他使用者可以讀取，而其他使用者也可以讀取。

這類似 Unix 的檔案屬性 [^file-permission_linux-bird]，用來表示該檔案的類型 (檔案、目錄、link) 和權限 (可讀、可寫、可執行)，以下是常見的 (用二進位和八進位表示) [^git-file-permission_stack-overflow] [^30-git-09_doggy]：

- `0100000000000000` ( `040000` )：Directory
- `1000000110100100` ( `100644` )：Regular non-executable file
- `1000000110110100` ( `100664` )：Regular non-executable group-writeable file
- `1000000111101101` ( `100755` )：Regular executable file
- `1010000000000000` ( `120000` )：Symbolic link
- `1110000000000000` ( `160000` )：Gitlink

:::info
註：二進位表示法 [^index-format_git-repo-github]
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

在 `git add` 指令期間沒有真正使用不同的 stage number，用於處理合併衝突：
- `0`：normal，無衝突，一切正常
- `1`：base，共同 ancestor 版本
- `2`：ours，目標 ( `HEAD` ) 版本
- `3`：theris，被合併的版本

## `.git/index` 內部

對 Git index 檔案 `.git/index` 進行位元轉儲 (bits dump)：

```shell
$ xxd -b -c 4 .git/index
00000000: 01000100 01001001 01010010 01000011  DIRC
00000004: 00000000 00000000 00000000 00000010  ....
00000008: 00000000 00000000 00000000 00000001  ....
0000000c: 01011110 01011011 11000111 10010001  ^[..
00000010: 00101111 00001011 00100011 00101010  /.#*
00000014: 01011110 01011011 11000111 10010001  ^[..
00000018: 00100111 00101111 10101000 11100000  '/..
0000001c: 00000000 00000000 00001000 00000100  ....
00000020: 00000000 10111000 00001101 01001111  ...O
00000024: 00000000 00000000 10000001 10100100  ....
00000028: 00000000 00000000 00000011 11101000  ....
0000002c: 00000000 00000000 00000011 11101000  ....
00000030: 00000000 00000000 00000000 00001011  ....
00000034: 10111011 11011110 11111001 00101010  ...*
00000038: 11101111 00001111 11101100 11111000  ....
0000003c: 00100111 00000100 01110000 10100110  '.p.
00000040: 00011111 11101000 10010101 00101010  ...*
00000044: 01101010 11111101 01011011 10011010  j.[.
00000048: 00000000 00001010 01101001 01101110  ..in
0000004c: 01100100 01100101 01111000 00101110  dex.
00000050: 01101000 01110100 01101101 01101100  html
00000054: 00000000 00000000 00000000 00000000  ....
00000058: 00000000 00000000 00000000 00000000  ....
0000005c: 00101000 01000001 00101101 00100001  (A-!
00000060: 00011010 10000111 01011011 00010011  ..[.
00000064: 01101111 01010010 10010111 01110010  oR.r
00000068: 11001010 01001100 01110011 11110011  .Ls.
0000006c: 10110011 11001100 00110011 00001000  ..3.
```

或以十六進位轉儲 (hex dump) 版本：

```shell
$ xxd .git/index     
00000000: 4449 5243 0000 0002 0000 0001 5e5b c791  DIRC........^[..
00000010: 2f0b 232a 5e5b c791 272f a8e0 0000 0804  /.#*^[..'/......
00000020: 00b8 0d4f 0000 81a4 0000 03e8 0000 03e8  ...O............
00000030: 0000 000b bbde f92a ef0f ecf8 2704 70a6  .......*....'.p.
00000040: 1fe8 952a 6afd 5b9a 000a 696e 6465 782e  ...*j.[...index.
00000050: 6874 6d6c 0000 0000 0000 0000 2841 2d21  html........(A-!
00000060: 1a87 5b13 6f52 9772 ca4c 73f3 b3cc 3308  ..[.oR.r.Ls...3.
```

index 檔案包含以下資訊：
- 12-byte header
- 多個排序的 index entries
- Extensions，它們通過簽名 (signature) 來識別
- 此 checksum 和之前的 index 檔案內容的 160-bit SHA-1

### Header

index 以 12-byte 的 header 開頭：

```
hex: 4449 5243 0000 0002 0000 0001
```

由代表 "DirCache" 的 4-btye 簽名 "DIRC" ( `0x44495243` ) 組成
4-byte 是 Git index 格式的當前版本號 "2"  ( `0x00000002` )
32-bit 的 index entries "1" ( `0x00000001` )

### Index Entry

在 `index.html` 此 index entry 中包括：

#### 64-bit ctime

ctime 是 change time，也就是檔案的 metadata 在最後修改的時間
- 前面 32-bit：ctime seconds
- 後面 32-bit：ctime nanosecond fractions

```
hex: 5e5b c791 2f0b 232a
```

可用 `stat` 指令來取得 `index.html` 的最後修改時間：

```shell
$ stat -c 'ctime: %z (%Z)' index.html
ctime: 2020-03-01 22:32:49.789259050 +0800 (1583073169)
```

然後用下面指令驗證：

```shell
$ printf '%x' 1583073169
5e5bc791%
$ printf '%x' 789259050
2f0b232a%
```

#### 64-bit mtime

mtime 是 modify time，也就是檔案的資料在最後修改的時間
- 前面 32-bit：mtime seconds
- 後面 32-bit：mtime nanosecond fractions

```
hex: 5e5b c791 272f a8e0
```

可用 `stat` 指令來取得 `index.html` 的最後修改時間：

```shell
$ stat -c 'mtime: %y (%Y)' index.html
mtime: 2020-03-01 22:32:49.657434848 +0800 (1583073169)
```

然後用下面指令驗證：

```shell
$ printf '%x' 1583073169
5e5bc791%
$ printf '%x' 657434848
272fa8e0%
```

#### 32-bit dev

檔案所在的裝置：

```
hex: 0000 0804
```

可用 `stat` 指令來取得 `index.html` 檔案所在的裝置：

```shell
$ stat -c '%D' index.html
804
```

#### 32-bit ino

檔案 inode number：

```
hex: 00b8 0d4f
```

可用 `stat` 指令來取得 `index.html` 檔案所在的裝置：

```shell
$ stat -c '%i' index.html
12062031
```

然後用下面指令驗證：

```shell
$ printf '%x' 12062031
b80d4f%
```

#### 32-bit mode

檔案權限以十六進位表示

```
hex: 0000 81a4
bin: 00000000 00000000 10000001 10100100
```

可用 `stat` 指令來取得 `index.html` 檔案的權限：

```shell
$ stat -c '%f' index.html
81a4
```

#### 32-bit uid

當前使用者的 user identifier

```
hex: 0000 03e8
```

可用 `stat` 指令來取得 `index.html` 檔案的 uid：

```shell
$ stat -c '%u' index.html
1000
```

然後用下面指令驗證：

```shell
$ printf '%x' 1000
3e8%
```

#### 32-bit gid

當前使用者的 group identifier

```
hex: 0000 03e8
```

可用 `stat` 指令來取得 `index.html` 檔案的 gid：

```shell
$ stat -c '%g' index.html
1000
```

然後用下面指令驗證：

```shell
$ printf '%x' 1000
3e8%
```

#### 32-bit file size

檔案大小

```
hex: 0000 000b
```

可用 `stat` 指令來取得 `index.html` 檔案的大小：

```shell
$ stat -c '%s' index.html
11
```

然後用下面指令驗證：

```shell
$ printf '%x' 11
b%
```

#### 160-bit SHA-1 Object ID

```
hex: bbde f92a ef0f ecf8 2704
     70a6 1fe8 952a 6afd 5b9a
```

可用 `git hash-object` 指令來取得 `index.html` 檔案的 blob 物件 ID：

```shell
$ git hash-object index.html
bbdef92aef0fecf8270470a61fe8952a6afd5b9a
```

#### 16-bit 'flags' field

```
hex: 000a
```

分為：
- 1-bit：assume-valid flag
- 1-bit：extended flag (在版本 2 中必須為零)
- 2-bit：stage (合併期間)
- 如果 length 小於 0xFFF，則為 12-bit name length；否則，將 0xFFF 儲存在此 field 中

#### 16-bit field

(版本 3 或更新版) 當前面的 16-bit field 的 "extended flag" 為 1 時適用
- 1-bit：為未來保留
- 1-bit：skip-worktree flag (用於 sparse checkout)
- 1-bit：intent-to-add flag (用於 `git add -N` )
- 13-bit：未使用，必須為零

忽略此部份，因為我目前使用的 Git 的版本為 2：

```shell
$ git --version
git version 2.17.1
```

#### 檔案路徑

- 相對於 top level 目錄 (不包含斜線) 的 entry 路徑名稱 (可變長度)
- `/` 作為路徑分隔字元
- 不允許使用特殊路徑元件 `.`、`..` 和 `.git`
- 不允許使用斜線
- 確切的編碼不確定，但 `.` 和 `/` 字元會以 7-bit ASCII 編碼，並且編碼不能包含 NUL byte (UNIX 路徑名稱)

```
hex: 696e 6465 782e 6874 6d6c
```

```shell
$ printf 'index.html' | xxd
00000000: 696e 6465 782e 6874 6d6c                 index.html
```

#### 1-8 NUL byte

使用 1-8 NUL byte 將 entry 填充為 8 byte 的倍數，同時保持名稱 NUL-terminated

```
hex: 0000 0000 0000 0000
```

### Extensions

忽略此部份，因本範例的 index 無 extensions。

### SHA-1 Index Checksum

在 index 檔案內容的最後 160-bit 是 index checksum，是由這 160-bit 之前的 index 檔案內容透過 SHA-1 計算的

```
hex: 2841 2d21 1a87 5b13 6f52
     9772 ca4c 73f3 b3cc 3308
```

用下面指令驗證：
- `head -c`：列印檔案的前幾個 byte

```shell
$ cat .git/index | head -c92 | sha1sum 
28412d211a875b136f529772ca4c73f3b3cc3308  -
```

如果 index 損壞，就能透過此 SHA-1 Index Checksum 來確認，並會說明 index 檔案已損壞：

```shell
$ git status
error: bad index file sha1 signature
fatal: index file corrupt
```