---
title: 重新認識 CSS - formatting context & independent formatting context
date: 2019-10-09 21:20:42
author: Titangene
tags:
  - IT 鐵人賽
categories:
  - CSS
cover_image: /images/cover/css.png
---

![](../images/cover/css.png)

本篇將介紹 CSS 的 formatting context 和 independent formatting context。

<!-- more -->

## 前言

> 「重新認識 CSS」這個系列名稱的由來就如其名，我想要重新認識它。雖然以前就有學過 CSS，但這次想從 CSS Spec 中學到最原始的定義和內容，更加了解 CSS 的原理，讓我在切版的時候可以更加確定自己在做什麼，我踩到的雷只是因為我不夠了解它才會炸開。
> 
> 在這 30 天的內容中，會將 Spec 內看到的資料整理成這個系列，也希望正在學 CSS 的各位可以更加了解它。另外我也會同時將文章發至我的 Blog，如果想直接看文內的程式碼 Demo 畫面，可以到我的 Blog 來看 😃。
> 
> 本文同步發表於 iT 邦幫忙：[重新認識 CSS - formatting context & independent formatting context](https://ithelp.ithome.com.tw/articles/10226493)
> 
> 「重新認識 CSS」系列文章發文於：
> - [iT 邦幫忙](https://ithelp.ithome.com.tw/users/20117586/ironman/2617)
> - [Titangene Blog](https://titangene.github.io/tags/it-%E9%90%B5%E4%BA%BA%E8%B3%BD/)

## formatting context

看一下在不同 CSS level 的 spec 中是如何定義 formatting context。

在 [CSS 2.1 內的「9.4 Normal flow」](https://www.w3.org/TR/CSS21/visuren.html#normal-flow) 是這樣定義 formatting context：

> Boxes in the normal flow belong to a formatting context, which may be block or inline, but not both simultaneously. [Block-level](https://www.w3.org/TR/CSS21/visuren.html#block-level) boxes participate in a [block formatting](https://www.w3.org/TR/CSS21/visuren.html#block-formatting) context. [Inline-level boxes](https://www.w3.org/TR/CSS21/visuren.html#inline-level) participate in an [inline formatting](https://www.w3.org/TR/CSS21/visuren.html#inline-formatting) context.

也就是說，在 CSS 2.1 定義 normal flow 中的 box 屬於某種 formatting context，可是 block 或 inline，但不能同時使用。

而不同的 box 會參與不同的 formatting context：
- [block-level box](https://www.w3.org/TR/CSS22/visuren.html#block-level) 會參與 BFC (block formatting context)
- [inline-level box](https://www.w3.org/TR/CSS22/visuren.html#inline-level) 會參與 IFC (inline formatting context)

但是在 [CSS 2.2 內的同一個章節](https://www.w3.org/TR/CSS22/visuren.html#normal-flow)中，將定義改成以下內容：

> Boxes in the normal flow belong to a formatting context, which in CSS 2.2 may be table, block or inline. In future levels of CSS, other types of formatting context will be introduced. [Block-level](https://www.w3.org/TR/CSS22/visuren.html#block-level) boxes participate in a [block formatting](https://www.w3.org/TR/CSS22/visuren.html#block-formatting) context. [Inline-level boxes](https://www.w3.org/TR/CSS22/visuren.html#inline-level) participate in an [inline formatting](https://www.w3.org/TR/CSS22/visuren.html#inline-formatting) context. Table formatting contexts are described in the [chapter on tables](https://www.w3.org/TR/CSS22/tables.html).
> 
> 資料來源：[CSS 2.2 - Appendix C. Changes](https://www.w3.org/TR/CSS22/changes.html#s.9.4)

也就是說，在 CSS 2.2 定義 formatting context 可以是 table、block 或 inlline，並且說明在將來的 CSS level 會引入其他 type 的 formatting context。另外也可以看到在 CSS 2.2 刪掉了 CSS 2.1 中的 "but not both simultaneously" 這句，也就是其實 BFC 和 IFC 可以同時使用。

後來在 [CSS Display Module Level 3](https://www.w3.org/TR/css-display-3/#formatting-context) 內也有定義 formatting context：

> A [formatting context](https://www.w3.org/TR/css-display-3/#formatting-context) is the environment into which a set of related boxes are laid out. Different formatting contexts lay out their boxes according to different rules. For example, a [flex formatting context](https://www.w3.org/TR/css-flexbox-1/#flex-formatting-context) lays out boxes according to the [flex layout](https://www.w3.org/TR/css-flexbox-1/#flex-layout) rules [[CSS3-FLEXBOX]](https://www.w3.org/TR/css-display-3/#biblio-css3-flexbox), whereas a [block formatting context](https://www.w3.org/TR/css-display-3/#block-formatting-context) lays out boxes according to the block-and-inline layout rules [[CSS2]](https://www.w3.org/TR/css-display-3/#biblio-css2).

也就是說，formatting context 是一種環境，相關的 box 會在此環境中佈局，不同的 formatting context 會根據不同的規則來佈局這些的 box。

例如：
- BFC (block formatting context) 會根據 [block 和 inline 佈局](https://www.w3.org/TR/CSS22/visuren.html#normal-flow)規則來佈局 box
- FFC (flex formatting context) 會根據 [flex 佈局](https://www.w3.org/TR/css-flexbox-1/#flex-layout)規則來佈局 box
- GFC (grid formatting context) 會根據 [grid 佈局](https://www.w3.org/TR/css-grid-1/#grid-layout)規則來佈局 box

## independent formatting context

一個 box 有兩種選擇：
- 建立一個新的 independent formatting context
- 繼續參與該 box 的 containing block 的 formatting context

而 box 建立的 formatting context 的 type 是由其 [inner display type](https://www.w3.org/TR/css-display-3/#inner-display-type) 來決定。

例如：
- [grid container](https://www.w3.org/TR/css-grid-1/#grid-container) 建立新的 [GFC (grid formatting context)](https://www.w3.org/TR/css-grid-1/#grid-formatting-context)
- [ruby container](https://www.w3.org/TR/css-ruby-1/#ruby-container) 建立新的 [RFC (ruby formatting context)](https://www.w3.org/TR/css-ruby-1/#ruby-formatting-context)
- [block container](https://www.w3.org/TR/css-display-3/#block-container) 可以建立新的 BFC 和/或新的 IFC

:::danger
詳情可參閱 Spec 中的 [`display`](https://www.w3.org/TR/css-display-3/#propdef-display) 屬性表格。(安麗自己的 `display` 文章)
:::

資料來源：
- [CSS 2.1 - 9.4. Normal flow](https://www.w3.org/TR/CSS21/visuren.html#normal-flow)
- [CSS 2.2 - 9.4. Normal flow](https://www.w3.org/TR/CSS22/visuren.html#normal-flow)
- [CSS Display Module Level 3](https://www.w3.org/TR/css-display-3/)