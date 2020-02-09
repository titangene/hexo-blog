---
title: 重新認識 CSS - Media query
date: 2019-09-27 19:30:15
author: Titangene
tags:
  - IT 鐵人賽
categories:
  - CSS
cover_image: /images/cover/css.png
---

![](../images/cover/css.png)

今天來介紹 CSS 的 media query 和 media feature。

<!-- more -->

## 前言

> 「重新認識 CSS」這個系列名稱的由來就如其名，我想要重新認識它。雖然以前就有學過 CSS，但這次想從 CSS Spec 中學到最原始的定義和內容，更加了解 CSS 的原理，讓我在切版的時候可以更加確定自己在做什麼，我踩到的雷只是因為我不夠了解它才會炸開。
>
> 本文同步發表於 iT 邦幫忙：[重新認識 CSS - Media query](https://ithelp.ithome.com.tw/articles/10221641)
> 
> 在這 30 天的內容中，會將 Spec 內看到的資料整理成這個系列，也希望正在學 CSS 的各位可以更加了解它。另外我也會同時將文章發至我的 Blog，如果想直接看文內的程式碼 Demo 畫面，可以到我的 Blog 來看 😃。
> 
> 「重新認識 CSS」系列文章發文於：
> - [iT 邦幫忙](https://ithelp.ithome.com.tw/users/20117586/ironman/2617)
> - [Titangene Blog](https://titangene.github.io/tags/it-%E9%90%B5%E4%BA%BA%E8%B3%BD/)

使用 media query 可以針對指定裝置來指定要應用哪個 style sheet


複習一下[昨天](https://ithelp.ithome.com.tw/articles/10221152)提到的 media type，如果要在不同 media type 指定不同的 style sheet，可以在 HTML 的 `link` 元素上的 `media` 屬性，為外部 style sheet 指定目標 media。

例如：
- 在螢幕上顯示時使用無襯線字體
- 在列印時使用襯線字體

```html
<link rel="stylesheet" type="text/css" media="screen" href="sans-serif.css">
<link rel="stylesheet" type="text/css" media="print" href="serif.css">
```

> 其他為 style sheet 指定 media 的方法可參閱在此系列中的另一篇「[重新認識 CSS - media type](https://ithelp.ithome.com.tw/articles/10221152)」。

## Media Queries

media query 是由 media type 和零個或多個表達式組成，這些表達式會檢查指定的 media feature 條件。

media query 是一個邏輯表達式，可以為 true 或 false。如果 media query 的 media type 與執行 UA 的裝置的 media type match (例如："[Applies to](https://www.w3.org/TR/CSS22/about.html#applies-to)" 那行所定義的)，並且 media query 中的所有表達式均為 true，則 media query 為 true。

### `all` 關鍵字

如果要適用所有 media type 的 media query，可以使用簡寫語法省略掉 `all` 這個關鍵字 (後面接著 `and` )。也就是說，如果沒有明確指定 media type 時，就等同於 `all` 這個關鍵字。例如：

下面兩個 media query 相同：

```css
@media all and (min-width:500px) { … }
@media (min-width:500px) { … }
```

下面兩個也相同：

```css
@media (orientation: portrait) { … }
@media all and (orientation: portrait) { … }
```

### Combining Media Queries：

可在 media query 列表中組合多個 media query，並以逗號 `,` 分隔。

如果 media query 列表的任何 component media query 為 `true`，則為 `true`；若所有 component media query 為 `false` 才會為 `false`。

例如：如果 media type 為 `screen` 且為彩色裝置，或 media type 為 `projection` 且為彩色裝置：

```css
@media screen and (color), projection and (color) { … }
```

### `and` 關鍵字

下面是在 HTML 的 `link` 元素上的 `media` 屬性使用 `and` 關鍵字：

```html
<link rel="stylesheet" media="screen and (color)" href="test.css" />
```

跟上面一樣的 media query，只是下面是在 CSS 中使用 @import-rule 寫的：

```css
@import url(color.css) screen and (color);
```

資料來源：
- [CSS 2.2 - 7. Media types](https://www.w3.org/TR/CSS22/media.html)
- [Media Queries](https://www.w3.org/TR/css3-mediaqueries/)
- [Media Queries Level 4](https://www.w3.org/TR/mediaqueries/#media-types)