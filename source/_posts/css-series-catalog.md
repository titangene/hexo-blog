---
title: 重新認識 CSS - 總結 & 系列目錄
date: 2019-10-15 21:20:42
author: Titangene
tags:
  - IT 鐵人賽
categories:
  - CSS
cover_image: /images/cover/css.png
---

![](../images/cover/css.png)

終於來到鐵人賽的最後一天！本篇對「重新認識 CSS」此系列做個總結，並整理此系列中的每篇文章可對應到哪些 CSS Spec。 

<!-- more -->

## 前言

> 「重新認識 CSS」這個系列名稱的由來就如其名，我想要重新認識它。雖然以前就有學過 CSS，但這次想從 CSS Spec 中學到最原始的定義和內容，更加了解 CSS 的原理，讓我在切版的時候可以更加確定自己在做什麼，我踩到的雷只是因為我不夠了解它才會炸開。
> 
> 在這 30 天的內容中，會將 Spec 內看到的資料整理成這個系列，也希望正在學 CSS 的各位可以更加了解它。另外我也會同時將文章發至我的 Blog，如果想直接看文內的程式碼 Demo 畫面，可以到我的 Blog 來看 😃。
>
> 本文同步發表於 iT 邦幫忙：[重新認識 CSS - 總結 & 系列目錄](https://ithelp.ithome.com.tw/articles/10228321)
> 
> 「重新認識 CSS」系列文章發文於：
> - [iT 邦幫忙](https://ithelp.ithome.com.tw/users/20117586/ironman/2617)
> - [Titangene Blog](https://titangene.github.io/tags/it-%E9%90%B5%E4%BA%BA%E8%B3%BD/)

就如前言所說，經過這次的鐵人賽，透過看 spec 來學習 CSS 的真實面貌，而不是未經驗證或自行腦補來切版，讓自己能真正的「重新認識 CSS」。

這個月以來都是當天看 spec 當天寫文，所以每天都有鐵人賽的感覺。起初看 spec 的時候，腦中天旋地轉，好多以前重來沒看過的專有名詞，還好有[好想工作室](https://ithelp.ithome.com.tw/2020ironman/signup/team/78)的夥伴跟我一起討論 spec，加快我理解的速度。

我第一次寫連續這麼多天的文章，雖然沒有像以前參加鐵人賽的前輩們說的那樣：「到後面幾天就會不知道要寫什麼了」，所以建議我在開賽前，事先規劃好這 30 天要寫什麼。不過我也沒有特別事先安排寫文的順序，因為我原本就預計會按照 [CSS 2.2](https://www.w3.org/TR/CSS22/Overview.html#minitoc) 的大綱來啃 spec，所以我不怕沒有主題可以寫，只怕自己理解的速度跟不上發文的速度 QQ，還好最後成功完賽 😄。

雖然鐵人完賽了，但這只是我學習旅程的開始，如果未來還有後續的學習紀錄，都會分享在我的 blog「[Titangene Blog](https://titangene.github.io/)」，歡迎大家來光顧。如果想知道我之前寫過哪些專案，可以到我的 [Github：titangene](https://github.com/titangene) 看看。

下面是我對於這次系列「重新認識 CSS」的整理，讓大家可以更快知道哪些內容可以對應到哪些 spec。

## 簡介

簡介 CSS，並說明如何在 HTML 使用 CSS。系列如下：

- [重新認識 CSS - CSS 簡介](https://titangene.github.io/article/css-introduction.html)

> 對應 spec 的以下幾篇：
> - [HTM 4.01 - 14. Style Sheets](https://www.w3.org/TR/html401/present/styles.html)
> - [HTML Living Standard - 4.2.4. The link element](https://html.spec.whatwg.org/multipage/semantics.html#the-link-element)

## Selector

Selector 是寫 CSS 時，必須掌握的東西，熟悉 selector 才會選到正確的元素。而此系列是依據 [Selectors Level 3](https://www.w3.org/TR/selectors-3/) 的分類來各別介紹的。系列如下：

- [重新認識 CSS - Simple Selector & Groups of selector](https://titangene.github.io/article/css-selector-1.html)
- [重新認識 CSS - Attribute selector (屬性選擇器)](https://titangene.github.io/article/css-attribute-selector.html)
- [重新認識 CSS - Pseudo-class (偽類) (1)](https://titangene.github.io/article/css-selector-pseudo-class-1.html)
- [重新認識 CSS - Pseudo-class (偽類) (2)](https://titangene.github.io/article/css-selector-pseudo-class-2.html)
- [重新認識 CSS - Pseudo-element (偽元素)](https://titangene.github.io/article/css-selector-pseudo-element.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 5. Selectors](https://www.w3.org/TR/CSS22/selector.html)
> - [CSS 2.2 - 12.1. The :before and :after pseudo-elements](https://www.w3.org/TR/CSS22/generate.html#before-after-content)
> - [Selectors Level 3](https://www.w3.org/TR/selectors-3/)
> - [Selectors Level 4](https://www.w3.org/TR/selectors-4/)
> - [CSS Pseudo-Elements Module Level 4](https://www.w3.org/TR/css-pseudo-4/)

## Values & Units

在開始認識 CSS 屬性之前，可以先認識屬性有哪些值和單位可以使用。系列如下：

- [重新認識 CSS - CSS 屬性值](https://titangene.github.io/article/css-attribute-value.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 4.3. Values](https://www.w3.org/TR/CSS22/syndata.html#values)
> - [CSS Values and Units Module Level 3](https://www.w3.org/TR/css-values-3/)
> - [CSS Values and Units Module Level 4](https://www.w3.org/TR/css-values-4/)

## Assigning property values, Cascading, and Inheritance

了解在 CSS 中，繼承和權重這些重要的觀念之後，接著就可以更深入的了解 CSS 是如何處理屬性值的。系列如下：

- [重新認識 CSS - Inheritance (繼承)](https://titangene.github.io/article/css-inheritance.html)
- [重新認識 CSS - Cascading & Specificity](https://titangene.github.io/article/css-cascading-and-specificity.html)
- [重新認識 CSS - CSS 如何處理屬性值](https://titangene.github.io/article/css-value-processing.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 6. Assigning property values, Cascading, and Inheritance](https://www.w3.org/TR/CSS22/cascade.html)
> - [CSS Cascading and Inheritance Level 3](https://www.w3.org/TR/css-cascade-3/)
> - [CSS Cascading and Inheritance Level 4](https://www.w3.org/TR/css-cascade-4/)

## `@media` & `@import` rules

引入其他 CSS 檔案或在多個裝置、解析度、螢幕尺寸下要應用哪個 style sheet (樣式表)，就需要了解如何使用 `@media` 和 `@import`。系列如下：

- [重新認識 CSS - Media type](https://titangene.github.io/article/css-media-type.html)
- [重新認識 CSS - Media query](https://titangene.github.io/article/css-media-query.html)
- [重新認識 CSS - @import](https://titangene.github.io/article/css-import.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 7. Media types](https://www.w3.org/TR/CSS22/media.html)
> - [Media Queries](https://www.w3.org/TR/css3-mediaqueries/)
> - [Media Queries Level 4](https://www.w3.org/TR/mediaqueries/)
> - [CSS 2.2 - 6.3 The @import rule](https://www.w3.org/TR/CSS22/cascade.html#at-import)

## Box model

Box model 是用來描述 document tree 中的每個元素所產生的矩形框。系列如下：

- [重新認識 CSS - Box model (前傳)](https://titangene.github.io/article/css-box-model.html)
- [重新認識 CSS - Box model：border](https://titangene.github.io/article/css-border.html)
- [重新認識 CSS - box-sizing](https://titangene.github.io/article/css-box-sizing.html)
- [重新認識 CSS - Collapsing margins](https://titangene.github.io/article/css-collapsing-margins.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 8. Box model](https://www.w3.org/TR/CSS22/box.html)
> - [CSS Box Model Module Level 3](https://www.w3.org/TR/css-box-3/)
> - [CSS Basic User Interface Module Level 3 (CSS3 UI) - 3. Box Model addition](https://www.w3.org/TR/css-ui-3/#box-model)
> - [CSS Backgrounds and Borders Module Level 3 - 4. Borders](https://www.w3.org/TR/css-backgrounds-3/#borders)

## Visual formatting model

Visual formatting model 是用來描述 UA 會如何處理視覺媒體 (visual media) 的 document tree，document tree 中的每個元素都會根據 box model 來產生 0 個或多個 box。而這些 box 的位置和大小都是相對於稱為 containing block 的矩形框的邊緣計算的。系列如下：

- [重新認識 CSS - Containing block](https://titangene.github.io/article/css-containing-block.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 9.1.2. Containing blocks](https://www.w3.org/TR/CSS22/visuren.html#containing-block)
> - [CSS 2.2 - 10.1. Definition of "containing block"](https://www.w3.org/TR/CSS22/visudet.html#containing-block-details)

### Controlling box generation

元素產生的 box 的 type，可以使用 `display` 屬性來指定，而 box 的 type 會影響 box 在 visual formatting model 中的行為。系列如下：

- [重新認識 CSS - Visual formatting model：Box generation (block)](https://titangene.github.io/article/css-box-generation-block-box.html)
- [重新認識 CSS - Visual formatting model：Box generation (inline)](https://titangene.github.io/article/css-box-generation-inline-box.html)
- [重新認識 CSS - display](https://titangene.github.io/article/css-display.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 9.2. Controlling box generation](https://www.w3.org/TR/CSS22/visuren.html#box-gen)
> - [CSS Display Module Level 3](https://www.w3.org/TR/css-display-3/)

### Formatting context & Normal flow

Normal flow 中的 box 屬於某種 formatting context，可是 block 或 inline，不同的 box 會參與不同的 formatting context。block-level box 會參與 BFC，inline-level box 會參與 IFC。系列如下：

- [重新認識 CSS - formatting context & independent formatting context](https://titangene.github.io/article/css-formatting-context.html)
- [重新認識 CSS - Block formatting context (BFC)](https://titangene.github.io/article/css-bfc.html)
- [重新認識 CSS - Inline formatting context (IFC)](https://titangene.github.io/article/css-ifc.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 9.4.1. Block formatting contexts](https://www.w3.org/TR/CSS22/visuren.html#block-formatting)
> - [CSS 2.2 - 9.4.2. Inline formatting contexts](https://www.w3.org/TR/CSS22/visuren.html#inline-formatting)
> - [CSS Display Module Level 3](https://www.w3.org/TR/css-display-3/)
> - [CSS Inline Layout Module Level 3](https://www.w3.org/TR/css-inline-3/)

### Positioning schemes

在 CSS 2.2 中，`position` 和 `float` 屬性這些定位方案都可以對 box 進行佈局。系列如下：

- [重新認識 CSS - position](https://titangene.github.io/article/css-position.html)
- [重新認識 CSS - float](https://titangene.github.io/article/css-float.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 9.3. Positioning schemes](https://www.w3.org/TR/CSS22/visuren.html#positioning-scheme)
> - [CSS Positioned Layout Module Level 3](https://www.w3.org/TR/css-position-3/)

### Layered presentation

除了水平和垂直的位置之外，如果想讓 box 可以有類似圖層的概念來排版，就能使用 `z-index` 屬性，這些 box 可以沿著 z 軸來排列，讓一個 box 可以疊在另一個 box 的上面。系列如下：

- [重新認識 CSS - z-index & stacking context](https://titangene.github.io/article/css-z-index-and-stacking-context.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 9.9. Layered presentation](https://www.w3.org/TR/CSS22/visuren.html#layers)

## Visual effects

當 block box 的內容超出 box 的 content edge 時，就可能會發生 overflow。使用 `overflow` 屬性就能指定超出的部份要如何處理 (例如：提供捲動機制來訪問被剪裁的內容)。而 `visibility` 屬性則可以指定是否要 render 由元素產生的 box，可用來顯示或隱藏 box。系列如下：

- [重新認識 CSS - overflow](https://titangene.github.io/article/css-overflow.html)
- [重新認識 CSS - visibility](https://titangene.github.io/article/css-visibility.html)

> 對應 spec 的以下幾篇：
> - [CSS 2.2 - 11. Visual effects](https://www.w3.org/TR/CSS22/visufx.html)
> - [CSS 2.2 - 17.5.5. Dynamic row and column effects](https://www.w3.org/TR/CSS22/tables.html#dynamic-effects)