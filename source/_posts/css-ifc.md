---
title: 重新認識 CSS - Inline formatting context (IFC)
date: 2019-10-11 21:20:42
author: Titangene
tags:
  - IT 鐵人賽
categories:
  - CSS
cover_image: /images/cover/css.png
---

![](../images/cover/css.png)

本篇將介紹 CSS 的 inline formatting context (IFC)。

<!-- more -->

## 前言

> 「重新認識 CSS」這個系列名稱的由來就如其名，我想要重新認識它。雖然以前就有學過 CSS，但這次想從 CSS Spec 中學到最原始的定義和內容，更加了解 CSS 的原理，讓我在切版的時候可以更加確定自己在做什麼，我踩到的雷只是因為我不夠了解它才會炸開。
> 
> 在這 30 天的內容中，會將 Spec 內看到的資料整理成這個系列，也希望正在學 CSS 的各位可以更加了解它。另外我也會同時將文章發至我的 Blog，如果想直接看文內的程式碼 Demo 畫面，可以到我的 Blog 來看 😃。
> 
> 本文同步發表於 iT 邦幫忙：[重新認識 CSS - Inline formatting context (IFC)](https://ithelp.ithome.com.tw/articles/10227194)
> 
> 「重新認識 CSS」系列文章發文於：
> - [iT 邦幫忙](https://ithelp.ithome.com.tw/users/20117586/ironman/2617)
> - [Titangene Blog](https://titangene.github.io/tags/it-%E9%90%B5%E4%BA%BA%E8%B3%BD/)

在 IFC 中，box 會從一個 containing block 的頂端開始一個接著一個的方式來水平排列，而 box 之間會受到水平的 margin、border 和 padding 影響，但不會被垂直的 margin、border 和 padding 所影響，如以下範例：

```html

```

box 可以以不同的方式垂直對齊：
- 它們的 bottom 或 top 可以對齊
- box 內的 text 的 baseline 可以對齊

包含形成行 (line) 的 box 的矩形區域被稱為 line box，line box 的寬度是由 containing block 和 float 來決定，line box 的高度是由 `line-height` 


UA 會將 inline-level box 排列 (flow) 至 line box 的垂直 stack 內，line box 的高度以下面方式來決定：
1. 計算 line box 中每個 inline-level box 的高度。對於 replaced 元素、inline-block 元素和 inline-table 元素，這是它們的 margin box 的高度。對於 inline box，這是它們的 `line-height`。
2. 



inline box (inner display type 為 `flow` 的 non-replaced inline-level box) 的內容會參與跟該 inline box 本身相同的 IFC

block container 在某些情境下會建立新的 IFC：

IFC 是由不包含 block-level box 的 block container 所建立的。當元素的 block container 只包含 inline-level content 時，block container 會建立新的 IFC，然後該元素還會產生一個 root inline box，該 root inline box 會 wrap 那些 inline content。

> root inline box 是一個 anonymous inline box，該 box 會自動產生可以容納 block container 的所有 inline-level content。
> 
> 有關 anonymous inline box 的詳情，可參閱我前幾天寫的「[重新認識 CSS - Visual formatting model：Box generation (inline)](https://ithelp.ithome.com.tw/articles/10225035)」，內有提供實際範例說明，在此篇就不再追述。






資料來源：
- [CSS 2.2 - 9.4.2 Inline formatting contexts](https://www.w3.org/TR/CSS22/visuren.html#inline-formatting)
- [CSS Display Module Level 3](https://www.w3.org/TR/css-display-3/)
- [CSS Inline Layout Module Level 3](https://www.w3.org/TR/css-inline-3/)