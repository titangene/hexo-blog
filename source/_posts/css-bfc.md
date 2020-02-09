---
title: 重新認識 CSS - Block formatting context (BFC)
date: 2019-10-10 21:20:42
author: Titangene
tags:
  - IT 鐵人賽
categories:
  - CSS
cover_image: /images/cover/css.png
---

![](../images/cover/css.png)

本篇將介紹 CSS 的 block formatting context (BFC)。

<!-- more -->

## 前言

> 「重新認識 CSS」這個系列名稱的由來就如其名，我想要重新認識它。雖然以前就有學過 CSS，但這次想從 CSS Spec 中學到最原始的定義和內容，更加了解 CSS 的原理，讓我在切版的時候可以更加確定自己在做什麼，我踩到的雷只是因為我不夠了解它才會炸開。
> 
> 在這 30 天的內容中，會將 Spec 內看到的資料整理成這個系列，也希望正在學 CSS 的各位可以更加了解它。另外我也會同時將文章發至我的 Blog，如果想直接看文內的程式碼 Demo 畫面，可以到我的 Blog 來看 😃。
> 
> 本文同步發表於 iT 邦幫忙：[重新認識 CSS - Block formatting context (BFC)](https://ithelp.ithome.com.tw/articles/10226848)
> 
> 「重新認識 CSS」系列文章發文於：
> - [iT 邦幫忙](https://ithelp.ithome.com.tw/users/20117586/ironman/2617)
> - [Titangene Blog](https://titangene.github.io/tags/it-%E9%90%B5%E4%BA%BA%E8%B3%BD/)


以下是 [MDN](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Block_formatting_context) 對於在何種情境下會建立 BFC 的定義：

> 以下情境會建立 BFC：
> - 文件中的 root 元素 (在 HTML 中是 `html` 元素)
> - `float` 屬性不為 `none` 的元素
> - `position` 屬性為 `absolute` 或 `fixed` 的元素
> - `display` 屬性為 `inline-block`、`flow-root`、`table-cell` 或 `table-caption` 的元素
> - 由 `display` 屬性為 `table`、`table-row`、`table-row-group`、`table-header-group`、`table-footer-group` 或 `inline-block` 的元素 implicitly 建立的 anonymous table cell
> - `overflow` 屬性不為 `visible` 的 block 元素
> - `contain` 屬性為 `layout`、`content` 或 `paint` 的元素
> - flex item ( `display` 屬性為 `flex` 或 `inline-flex` 的元素的子元素)
> - grid item ( `display` 屬性為 `grid` 或 `inline-grid` 的元素的子元素)
> - multicol container ( `column-count` 或 `column-width` 屬性不為 `auto` 的元素，包含 `column-count: 1` 的元素)
> - `column-span: all` 應該始終建立一個新的 formatting context，即使該元素沒有包含在 multicol container 中

float 的定位和 clear 規則都只會適用於同一個 BFC 中的內容：
- `float` 屬性不會影響其他 BFC 中的內容佈局
- `clear` 屬性只能 clear 同一個 BFC 中有設定 `clear` 屬性的元素之前的 float 元素，不會影響到元素本身內部或其他 BFC 中的 float 元素

當兩個 block 都在同一個 BFC 時，它們的 margin 才會發生 margin collapsing。


資料來源：
- [Block formatting context - Developer guides | MDN](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Block_formatting_context)