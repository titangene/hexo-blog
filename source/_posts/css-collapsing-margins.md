---
title: 重新認識 CSS - Collapsing margins
date: 2019-10-02 19:50:42
author: Titangene
tags:
  - IT 鐵人賽
categories:
  - CSS
cover_image: /images/cover/css.png
---

![](../images/cover/css.png)

兩個 box 之間的 margin 相鄰 (adjoining) 時，可能會讓 margin 發生合併，這個現象就被稱為 collapsing margin，而合併的 margin 就被稱為 collapsed margin。

<!-- more -->

## 前言

> 「重新認識 CSS」這個系列名稱的由來就如其名，我想要重新認識它。雖然以前就有學過 CSS，但這次想從 CSS Spec 中學到最原始的定義和內容，更加了解 CSS 的原理，讓我在切版的時候可以更加確定自己在做什麼，我踩到的雷只是因為我不夠了解它才會炸開。
> 
> 在這 30 天的內容中，會將 Spec 內看到的資料整理成這個系列，也希望正在學 CSS 的各位可以更加了解它。另外我也會同時將文章發至我的 Blog，如果想直接看文內的程式碼 Demo 畫面，可以到我的 Blog 來看 😃。
> 
> 本文同步發表於 iT 邦幫忙：[重新認識 CSS - Collapsing margins](https://ithelp.ithome.com.tw/articles/10223906)
> 
> 「重新認識 CSS」系列文章發文於：
> - [iT 邦幫忙](https://ithelp.ithome.com.tw/users/20117586/ironman/2617)
> - [Titangene Blog](https://titangene.github.io/tags/it-%E9%90%B5%E4%BA%BA%E8%B3%BD/)

兩個 box 之間的 margin 相鄰 (adjoining) 時，可能會讓 margin 發生合併，這個現象就被稱為 collapsing margin，而合併的 margin 就被稱為 collapsed margin。

在下列這些情況會發生 collapsing margin：
- 兩個 block 都是 in-flow [block-level box](https://www.w3.org/TR/CSS22/visuren.html#block-boxes)，且參與相同的 [BFC](https://www.w3.org/TR/CSS22/visuren.html#block-formatting)
- 兩個 block 沒有 line box、clearance、padding 或沒有 border 將它們的 margin 隔開
- 垂直相鄰的 box edge：
  - box 的 `margin-top` 和第一個 in-flow 子元素的 `margin-top`
  - box 的 `margin-bottom` 和下一個 in-flow 之後的 sibling 的 `margin-top`
  - 最後一個 in-flow 子元素的 `margin-bottom` 和其父元素的 `margin-bottom`，且父元素的高度為 `auto`
  - 未建立新的 BFC，且 `min-height` 為 0、`height` 為 `auto`，且沒有 in-flow children 的 box 的 `margin-top` 和 `margin-bottom`

而下面這些情況不會發生 collapsing margin：
- root 元素的 box 的 margin
- 有 [clearance](https://www.w3.org/TR/CSS22/visuren.html#clearance)
- 水平的 margin
- 建立新的 BFC (block formatting context) 的元素的 margin 不會與 in-flow 的子元素 collapse，例如：float 的元素、`overflow` 屬性值不為 `visible` 的元素
- `position: absolute` 的元素的 margin
- inline-block box 的 margin

資料來源：
- [CSS 2.2 - 8.3.1 Collapsing margins](https://www.w3.org/TR/CSS22/box.html#collapsing-margins)