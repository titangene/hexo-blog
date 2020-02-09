---
title: 重新認識 CSS - @import
date: 2019-09-28 19:30:15
author: Titangene
tags:
  - IT 鐵人賽
categories:
  - CSS
cover_image: /images/cover/css.png
---

![](../images/cover/css.png)


CSS 的 `@import` 規則可以從其他 style sheet 中 import 樣式規則，本篇將介紹如何使用。

<!-- more -->

## 前言

> 「重新認識 CSS」這個系列名稱的由來就如其名，我想要重新認識它。雖然以前就有學過 CSS，但這次想從 CSS Spec 中學到最原始的定義和內容，更加了解 CSS 的原理，讓我在切版的時候可以更加確定自己在做什麼，我踩到的雷只是因為我不夠了解它才會炸開。
> 
> 本文同步發表於 iT 邦幫忙：[重新認識 CSS - @import](https://ithelp.ithome.com.tw/articles/10221956)
> 
> 在這 30 天的內容中，會將 Spec 內看到的資料整理成這個系列，也希望正在學 CSS 的各位可以更加了解它。另外我也會同時將文章發至我的 Blog，如果想直接看文內的程式碼 Demo 畫面，可以到我的 Blog 來看 😃。
> 
> 「重新認識 CSS」系列文章發文於：
> - [iT 邦幫忙](https://ithelp.ithome.com.tw/users/20117586/ironman/2617)
> - [Titangene Blog](https://titangene.github.io/tags/it-%E9%90%B5%E4%BA%BA%E8%B3%BD/)

## `@import` 語法
`@import` 語法有兩種寫法：
- `url()` 內包含 style sheet 的 URI
- 直接寫 style sheet 的 URI 的字串

例如：

```css
@import "mystyle.css";
@import url("mystyle.css");
```

## 解析語法錯誤處理

如果 `@import` 規則符合下面任一項，UA 就會忽略該 `@import` 規則：
- 在其他所有規則之後使用 `@import` 規則 ( `@charset` 規則除外)
- 在 block 內使用 `@import` 規則，例如：在 `@media` block 內使用

### 在其他所有規則之後使用 `@import`

例如：如果 CSS 2.2 parser 遇到以下 style sheet：

```css
@import "subs.css";
h1 { color: blue }
@import "list.css";
```

根據 CSS 2.2 規範，上面範例中的第二個 `@import` 規則 (也就是 `@import "list.css";` ) 會無效，因為 `@import` 必須寫在其他規則的前面，所以 CSS 2.2 parser 會忽略第二個 `@import` 規則，最後會將 style sheet 縮減以下這樣：

```css
@import "subs.css";
h1 { color: blue }
```

### 在 block 內使用 `@import`

例如：如果只想在 `print` media 上 import style sheet，第二個 `@import` 規則 (也就是 `@import "print-main.css";` ) 會無效，因為此 `@import` 規則不能在 `@media` block 內使用：

```css
@import "subs.css";
@media print {
  @import "print-main.css";
  body { font-size: 10pt }
}
h1 {color: blue }
```

正確的寫法是在 `@import` 規則後面加上要指定的 media type，例如：

```css
@import "subs.css";
@import "print-main.css" print;
@media print {
  body { font-size: 10pt }
}
h1 {color: blue }
```

> 如果想了解其他 media type，可參閱在此系列中的另一篇「[重新認識 CSS - media type](https://ithelp.ithome.com.tw/articles/10221152)」。

資料來源：
- [CSS 2.2 - 6.3 The @import rule](https://www.w3.org/TR/CSS2/cascade.html#at-import)
- [CSS 2.2 - 4.1.5 At-rules](https://www.w3.org/TR/CSS22/syndata.html#at-rules)
- [Media Queries](https://www.w3.org/TR/css3-mediaqueries/)
- [Media Queries Level 4](https://www.w3.org/TR/mediaqueries/#media-types)