---
title: 重新認識 CSS - Containing block
date: 2019-10-03 21:20:42
author: Titangene
tags:
  - IT 鐵人賽
categories:
  - CSS
cover_image: /images/cover/css.png
---

![](../images/cover/css.png)

本篇將介紹 CSS 的 Containing block。

<!-- more -->

## 前言

> 「重新認識 CSS」這個系列名稱的由來就如其名，我想要重新認識它。雖然以前就有學過 CSS，但這次想從 CSS Spec 中學到最原始的定義和內容，更加了解 CSS 的原理，讓我在切版的時候可以更加確定自己在做什麼，我踩到的雷只是因為我不夠了解它才會炸開。
> 
> 在這 30 天的內容中，會將 Spec 內看到的資料整理成這個系列，也希望正在學 CSS 的各位可以更加了解它。另外我也會同時將文章發至我的 Blog，如果想直接看文內的程式碼 Demo 畫面，可以到我的 Blog 來看 😃。
> 
> 本文同步發表於 iT 邦幫忙：[重新認識 CSS - Containing block](https://ithelp.ithome.com.tw/articles/10224274)
> 
> 「重新認識 CSS」系列文章發文於：
> - [iT 邦幫忙](https://ithelp.ithome.com.tw/users/20117586/ironman/2617)
> - [Titangene Blog](https://titangene.github.io/tags/it-%E9%90%B5%E4%BA%BA%E8%B3%BD/)

## Containing block

下面是 CSS 2.2 對於 containing block 的定義：

> In CSS 2.2, many box positions and sizes are calculated with respect to the edges of a rectangular box called a containing block. In general, generated boxes act as containing blocks for descendant boxes; we say that a box "establishes" the containing block for its descendants. The phrase "a box's containing block" means "the containing block in which the box lives," not the one it generates.

很多 box 的位置和大小都是根據某種矩形框 (rectangular box) 來計算的，在 CSS 定義中，就將此 box 稱為 containing block。通常會把產生的 box 作為 descendant box 的 containing block，也就是說一個 box 會為它的 descendant 建立 containing block。當我們在說 "一個 box 的 containing block" 是代表 "box 所在的 containing block"，而不是該 box 所產生的 containing block。

元素的 containing block 定義如下：

1. root 元素 (也就是 `<html>` ) 所在的 containing block 是一個被稱為 initial containing block 的矩形。對於 continuous media 來說，containing block 的尺寸是由 viewport 的尺寸，並且被固定 (anchored) 在 canvas 的開始位置 (origin)；對於 paged media 來說，containing block 的尺寸就是 page area
2. 當其他元素的 `position` 屬性為 `relative` 或 `relative` 時，containing block 是由最近的 ancestor box 的 content edge 所構成，該 ancestor box 是 block container 或建立了 formatting context
3. 如果元素為 `position: fixed` 時，則在 continuous media 的情況下由 viewport 建立 containing block，在 paged media 的情況下由 page area 建立 containing block
4. 如果元素為 `position: absolute` 時，則 containing block 是由最近的 `position` 屬性為 `absolute`、`relative` 或 `relative` 的 ancestor 建立，方法如下：
    1. 如果該 ancestor 是 inline 元素時，containing block 是為該元素產生的第一個和最後一個 inline box 的 padding box 周圍的 bounding box (邊界框)。在 CSS 2.2 中，如果將 inline 元素分割 (split across) 為多行時，則 containing block 未定義
    2. 否則，containing block 是由該 ancestor 的 padding edge 所構成

    如果沒有這樣的 ancestor，則 containing block 就是 initial containing block

雖然每個 box 都會根據其 containing block 來決定位置，但不會被 containing block 所限制，可能會發生 overflow。當發生 overflow 時，可以使用 CSS 提供的 `overflow` 屬性來處理，這在之後會提到。

> 有關 overflow 的詳情，可參閱我之後寫的「[重新認識 CSS - overflow](https://ithelp.ithome.com.tw/articles/10227474)」。


資料來源：
- [CSS 2.2 - 9.1.2. Containing blocks](https://www.w3.org/TR/CSS22/visuren.html#containing-block)
- [CSS 2.2 - 10.1. Definition of "containing block"](https://www.w3.org/TR/CSS22/visudet.html#containing-block-details)