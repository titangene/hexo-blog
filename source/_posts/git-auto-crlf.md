---
title: 處理 Git 斷行字元的問題
date: 2020-02-23 23:42:23
author: Titangene
tags:
  - w3HexSchool
categories:
  - Git
cover_image: /images/cover/git.jpg
---

在使用 Git 的過程中，若在不同作業系統編輯同一個 repo 的檔案，可能就會發生斷行字元的問題。Git 在 config 提供了 `core.autocrlf` 選項並用 `.gitAttributes` 檔案來處理斷行字元的問題。

<!-- more -->

之前我在 Windows 建立一個專案，是用來專門放用 Markdown 檔案的筆記，而且是用 Git 來作版本控制。但最近我在這台筆電安裝完雙系統 (Windows 10 和 Ubuntu 18.04) 後，有時候會在 Ubuntu 開發並將過程寫成筆記，所以這個筆記專案就會需要在不同 OS 下編輯。

也因為以上需求讓我多學習到不同系統其實預設使用的文字檔案的斷行字元是不同的，Windows 是使用 CRLF ( `\r\n`，`0x0D 0x0A` )，而 Linux 則是使用 LF 字元 ( `\n`，`0x0A` )。也因如此，Windows 上的每個檔案的每一行結尾都會比 Liunx 還要多一個字元，也就是多了很多 CR ( `\r` ) 字元。

:::info
註：
- CR 是 Carriage Return 的意思，也就是 Enter 字元
- LF 是 Line Feed 的意思，也就是真正的換行字元
- macOS 的斷行字元也是 LF 字元 ( `\n`，`0x0A` )
:::

那為何會提到斷行字元？因為當我執行 `git status` 指令後，發現竟然檔案有被修改過，但我除了切換 OS 之外，根本沒有修改筆記啊！

```shell
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   note.md

no changes added to commit (use "git add" and/or "git commit -a")
```

其實這跟剛剛提到的斷行字元有關。如果用 `git diff` 指令就會看到筆記內的每行文字的結尾都多出了 `^M` 這個東西，而且有些編輯器 (例如：VS Code) 可能會看不出來：

```diff
$ git diff note.md
diff --git a/note.md b/note.md
index e27815a..6b5682e 100644
--- a/note.md
+++ b/note.md
@@ -1,5 +1,5 @@
-Note
-===
-
-something
+Note^M
+===^M
+^M
+something^M
 This is the last line
\ No newline at end of file
```

其實 `^M` 恰好是 vim 用來顯示 `0x0D` 的方式，而 `0x0D` 就是剛剛提到 Windows 檔案內的每一行結尾，都會比 Linux 多出 CR ( `\r` ) 這個字元。所以 `^M` 其實就等同於 CR ( `\r`，`0x0D` )。

:::info
註：
- `^M` 其中的 `^` 代表 Ctrl，所以 `^M` 也就代表 `Ctrl + M` (也就是 Enter)
- `^M` 雖然看是兩個字元，但在終端機上實際只有一個字元。在大部分的終端機系統中，包括 Windows 的 cmd、Linux 和 FreeBSD 都可用 Ctrl 來表示 `^` 字元 (caret)
- `0x0D` 是以十六進位表示的 13，而 M 正好是英文字母中第 13 個字母
:::

不過，如果有多個檔案的每一行都要刪掉 `^M` 就會很麻煩，所以 Git 提供一種自動轉換斷行符號的功能：`core.autocrlf`。

那 Git 的自動轉換斷行符號功能到底做了什麼？
- 當你執行 `git add` 將檔案加入 index 時，Git 可將 CRLF 自動傳成 LF。
- 當你執行 `git checkout` 將程式碼 checkout 出檔案系統時，Git 可將 LF 自動轉成 CRLF。

如果要在全域設定可執行下面指令來設定：

```shell
$ git config --global core.autocrlf true
```

如果是要在各別的 repo 設定，就需要在 repo 的根目錄下建立一個名為 `.gitattributes` 的檔案，當這個檔案被 commit 至 repo 時，就會覆蓋 `core.autocrlf` 的設定，以確保管理此 repo 的協作者的設定一致。下面是 `.gitattributes` 這個檔案的範例模板：

```
# Set the default behavior, in case people don't have core.autocrlf set.
* text=auto

# Explicitly declare text files you want to always be normalized and converted
# to native line endings on checkout.
*.c text
*.h text

# Declare files that will always have CRLF line endings on checkout.
*.sln text eol=crlf

# Denote all files that are truly binary and should not be modified.
*.png binary
*.jpg binary
```

此檔案的設定格式：

```
pattern attr1 attr2 ...
```

`pattern` 和屬性的中間要以空格分隔。當 `pattern` 匹配相關路徑時，後面的屬性的設定會應用在那些路徑。

`text` 這個屬性是用於啟用並控制行尾的正規化。正規化文字檔後，行尾會在 repo 中轉換為 LF。如果要控制工作目錄中使用的行尾樣式，請針對該檔案設定 `eol` 屬性 ( `eol` 就 end-of-line 的縮寫)，並且對所有文字檔設定 `core.eol` 變數。如果將 `core.autocrlf` 設為 `true` 或 `input` 會覆蓋 `core.eol` 的設定。

> 詳情可參考 [Git - git-config Documentation](https://git-scm.com/docs/git-config) 文件中對於 `core.eol` 設定的定義。

`text=auto`：路徑被標記為自動轉換行尾。如果 Git 確定該內容為文字，那該檔案的行尾就會在 checkin 時轉換成 LF。當用 CRLF commit 檔案時，則不會進行轉換。

資料來源：
- [Configuring Git to handle line endings - GitHub Help](https://help.github.com/en/articles/configuring-git-to-handle-line-endings)
- [Git 在 Windows 平台處理斷行字元 (CRLF) 的注意事項 | The Will Will Web](https://blog.miniasp.com/post/2013/09/15/Git-for-Windows-Line-Ending-Conversion-Notes)
- [ASCII - Wiki](https://zh.wikipedia.org/wiki/ASCII)
- [unix - What does ^M character mean in Vim? - Stack Overflow](https://stackoverflow.com/questions/5843495/what-does-m-character-mean-in-vim)
- [Git - gitattributes Documentation](https://git-scm.com/docs/gitattributes)
- [Git - Git Attributes](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes)
- [Git - Git Configuration](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)