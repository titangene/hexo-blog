---
title: Sass：@import rule
date: 2020-05-17 23:54:46
author: Titangene
tags:
  - Sass
  - CSS
  - Dart Sass
  - w3HexSchool
categories:
  - Web Dev
cover_image: /images/cover/sass.jpg
---

![](../images/cover/sass.jpg)

Sass 的 `@import` rule 可以引入 Sass 和 CSS stylesheet、提供對 mixin、function 和變數的存取，並且還能將多個 stylesheet 的 CSS 組合在一起。例如：`main.scss` 內使用 `@import` 引入 Sass 和 CSS 檔，在編譯 Sass 後，就只會產生一個 `main.css` 檔。

<!-- more -->

而 CSS 本身提供的 `@import` rule 會讓瀏覽器在呈現頁面時，有幾個 CSS 檔案是透過 `@import` 引入的，就要發出幾個 HTTP request，這與 Sass `@import` rule 的行為不同。

## 同時引入多個檔案

雖然 Sass 和 CSS 的引入語法相同，但 Sass 可以用逗號分隔，一次引入多個檔案，不需要每個檔案都各別用一行 `@import` 引入：

```sass
// _button.scss
.button { color: #aaa; }
```

```sass
// _nav.scss
.nav { color: #bbb; }
```

```sass
// main.scss
@import 'button', 'nav';

main { color: #000; }
```

輸出：

```css
.button { color: #aaa; }
.nav { color: #bbb; }
main { color: #000; }
```

當 Sass 引入檔案時，會對該檔案進行估算，很像是將內容直接抄在你使用 `@import` 的位置上。

在 `@import` 之前定義的任何 mixin、function 或變數 (甚至來自其他 `@import` 的) 都可以在你引入的 stylesheet 中使用他們。

## 不要重複引入

所以若對同一個檔案引入多次，就會造成編譯出來的 CSS 檔案內，會有重複的 stylesheet。例如：

```sass
// main.scss
@import 'button';
@import 'button';

main { color: #000; }
```

輸出：

```css
.button { color: #aaa; }
.button { color: #aaa; }
main { color: #000; }
```

除非你重複引入的檔案內，只有像是 mixin、function 或變數等不會產生樣式規則的東西，就不會產生重複的內容。

## 引入不用寫副檔名

另外，使用 Sass 的 `@import` rule 時，不用寫檔案的副檔名，因為 Sass 會自動幫你引入。

例如：當你引入 `@import "a"` 時，Sass 會自動引入 `a.scss`、`a.sass` 或 `a.css`。但 CSS 本身提供的 `@import` rule 記得要寫檔案的副檔名才能正確的引入。

## 引入路徑

- 在引入 partials (以 `_` 為開頭的檔案) 時，不用寫 `_` 就能引入，且該檔案不會被編譯
- 引入檔案時可省略 `./` 相對引入
- 在使用 Sass CLI 時，若有提供 `--load-path` 或 `-I` option，就可以簡化引入的路徑
  - 詳情可參閱我之前寫的 [Dart Sass 介紹 (使用與安裝)](https://titangene.github.io/article/dart-sass.html#load-path%EF%BC%8C-I)
  - 但引入時，Sass 會先解析相對於當前檔案的路徑，若沒有找到時，才會引入相對於 `--load-path` 提供的路徑內的檔案

例如：在 `node_modules/bootstrap/scss` 目錄內正好有名為 `_button.scss` 的檔案，而我引入了 `@import 'buttons'`：

```sass
// src/_buttons.scss
.button { color: blue; }
```

```sass
// src/main.scss
@import 'buttons';

main { color: #000; }
```

編譯 Sass (注意，我使用了 `--load-path` option，指定了 Bootstrap 內的路徑)：

```shell
$ sass --load-path=node_modules/bootstrap/scss src/main.scss
```

但輸出的結果表名，我引入的 `@import 'buttons'` 指的是 `src/_buttons.scss` 這個檔案，而不是 `node_modules/bootstrap/scss` 目錄內的 `_button.scss` 這個檔案：

```css
.button { color: blue; }
main { color: #000; }
```

因為在 `src` 目錄內，`_buttons.scss` 相對於 `main.scss`，所以不會引用到 Bootstrap 內的檔案。

## index 檔案

若在資料夾 (例如：`base` 資料夾) 內建立 `_index.scss` 或 `_index.sass` 檔案，而 index 檔案內引入了該資料夾內的其他檔案 (例如：`_code.scss` 和 `_list.scss` ) 時，則可直接透過 `@import` 該資料夾來引入那些檔案。

目錄結構：

```shell
$ tree
.
├── base
│   ├── _code.scss
│   ├── _index.scss
│   └── _list.scss
└── main.scss
```

```sass
// base/_code.scss
code { padding: 4px; }
```

```sass
// base/_list.scss
ul { padding: 0; }
```

```sass
// base/_index.scss
@import 'code', 'list';
```

```sass
// main.scss
@import 'base';

main { color: #000; }
```

輸出：

```css
code { padding: 4px; }
ul { padding: 0; }
main { color: #000; }
```

## 巢狀引入

也可以巢狀引入，但要注意的是在巢狀引入 (nested import) 中定義的 mixin、function 或變數都還是全域定義的：

```sass
// _button.scss
.button { color: #aaa; }
```

```sass
// main.scss
main {
  @import 'a';
}
```

輸出：

```css
main .button { color: #aaa; }
```

若是巢狀引入的檔案內有使用 parent selectors (也就是 `&` )，則會引用 stylesheet 巢狀在其中的選擇器 (估算的方式類似 mixin)，例如：

```sass
// _theme.scss
ul li {
  padding-left: 16px;

  [dir=rtl] & {
    padding-left: 0;
    padding-right: 16px;
  }
}
```

```sass
// style.scss
.theme-sample {
  @import 'theme';
}
```

輸出：

```css
.theme-sample ul li {
  padding-left: 16px;
}
[dir=rtl] .theme-sample ul li {
  padding-left: 0;
  padding-right: 16px;
}
```

## 引入 CSS

Sass 的 `@import` rule 除了可以引入 `.scss` 和 `.sass` 檔，也可以引入 `.css` 檔。

但要記得在引入 `.css` 檔時，不可以像 `@import 'file.css';` 這樣明確的寫你引入了 `.css` 檔，而是要忽略副檔名。因為當你明確的寫 `.css` 副檔名時，代表你想用 [CSS 原本的 `@import` rule](https://sass-lang.com/documentation/at-rules/import#plain-css-imports)。

以下這些引入方式都是 CSS 原本的 `@import` rule：
- 以 `.css` 為結尾的 URL
- 以 `http://` 或 `https://` 為開頭的 URL
- 用 `url()` 來引入 URL
- 具有 media query 的引入

```sass
@import "theme.css";
@import "http://fonts.googleapis.com/css?family=Droid+Sans";
@import url(theme);
@import "landscape" screen and (orientation: landscape);
```

## Interpolation

雖然 Sass imports 不能使用 [interpolation](https://sass-lang.com/documentation/interpolation)，但 CSS 原本的 `@import` 是可以的。這樣就可以動態生成引入的 URL，例如：

```sass
@mixin google-font($family) {
  @import url("http://fonts.googleapis.com/css?family=#{$family}");
}

@include google-font("Droid Sans");
```

輸出：

```css
@import url("http://fonts.googleapis.com/css?family=Droid Sans");
```

資料來源：[Sass: @import](https://sass-lang.com/documentation/at-rules/import)