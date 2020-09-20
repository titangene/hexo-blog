---
title: Jest：DOM 測試 (jQuery)
date: 2020-08-02 23:55:10
author: Titangene
tags:
  - Jest
  - JavaScript
  - Unit Testing
  - mock
  - DOM
  - jQuery
  - w3HexSchool
categories:
  - Testing
cover_image: /images/cover/jest.jpg
---

![](../images/cover/jest.jpg)

若用 Jest 來測試直接操作 DOM 的程式碼，最大好處是不用安裝額外的套件就可以測試，因為 Jest 附帶了 `jsdom`，它是用來模擬 DOM 環境，讓你很像在瀏覽器上呼叫 DOM API，進而觀察 DOM 的操作是否符合預期，也就代表測畫面不用真的開啟瀏覽器，不用等待畫面渲染就可以進行測試。

<!-- more -->

> 其他 Jest 相關文章可參閱 [Jest 系列文章](https://titangene.github.io/tags/jest/)。

# 寫範例程式碼

本篇會用下面程式碼作為範例：
- `fetchCurrentUser.js`：發送 request，並將收到的資料進行解析處理
- `displayUser.js`：在按鈕上註冊 `click` 事件，點擊按鈕後會發 API，並將資料顯示在畫面上

```javascript
// src/fetchCurrentUser.js
const $ = require('jquery');

function parseJSON(user) {
  return {
    fullName: user.firstName + ' ' + user.lastName,
    loggedIn: true,
  };
}

function fetchCurrentUser(callback) {
  return $.ajax({
    success: user => callback(parseJSON(user)),
    type: 'GET',
    url: 'http://example.com/currentUser',
  });
}

module.exports = fetchCurrentUser;
```

```javascript
// src/displayUser.js
const $ = require('jquery');
const fetchCurrentUser = require('./fetchCurrentUser.js');

$('#button').click(() => {
  fetchCurrentUser(user => {
    const loggedText = 'Logged ' + (user.loggedIn ? 'In' : 'Out');
    $('#username').text(user.fullName + ' - ' + loggedText);
  });
});
```

# 建立測試

被測試的函數在 `#button` DOM 元素上新增一個事件監聽器，所以需要設定 DOM 來進行測試。

Jest 附帶了 `jsdom`，它模擬一個 DOM 環境，很像瀏覽器，代表呼叫的每個 DOM API 都可像在瀏覽器中觀察的方式一樣。

mock `fetchCurrentUser.js` 可讓測試不用真的發出請求，可 reslove 成 local mock data，快速進行測試。

```javascript
// __tests__/displayUser.test.js
jest.mock('../src/fetchCurrentUser');

it('點擊按鈕後顯示使用者已登入', () => {
  // 設定 document body
  document.body.innerHTML = `
    <span id="username"></span>
    <button id="button"></button>`;

  // 此 module 有 side-effect
  require('../src/displayUser');

  const $ = require('jquery');
  const fetchCurrentUser = require('../src/fetchCurrentUser');

  // 告訴 fetchCurrentUser mock 函數自動使用一些資料來 invoke callback
  fetchCurrentUser.mockImplementation(callback => {
    callback({
      fullName: 'Titan',
      loggedIn: true,
    });
  });

  // 點擊前的 DOM
  console.log(document.body.innerHTML);

  // 使用 jQuery 模擬點擊按鈕
  $('#button').click();

  // 點擊後的 DOM
  console.log(document.body.innerHTML);

  // Assert fetchCurrentUser 函數已被呼叫，
  // 且 span#username 的 inner text 已按預期更新了
  expect(fetchCurrentUser).toBeCalled();
  expect($('#username').text()).toEqual('Titan - Logged In');
});
```

下面測試為何要用 `$.ajax.mock.calls[0][0].success()` 的方式測，而不是直接 mock `src/fetchCurrentUser.js` 檔案內的 `parseJSON`？

因為 `src/fetchCurrentUser.js` 檔案內的 `parseJSON` 沒有 export，所以不能 mock，而且 mock 就失去測試的意義了，該測試就是為了確定 `$.ajax` 發出請求拿到的使用者資料透過 `parseJSON` 處理後是否會得到正確的資料 (API 會回傳使用者的 `firstName` 和 `lastName`，而 `parseJSON` 是負責把名字組合成 `fullName` 和是否登入的狀態 `loggedIn` )。

:::info
註：Jest 官方文件提供的 [examples/jquery](https://github.com/facebook/jest/tree/master/examples/jquery/) 範例內原本沒有以下內容，因視需求而修改的：
- 為了讓將 mock 過的 `$.ajax()` 在每個測試執行前都被清乾淨 (不保留前一個測試使用個的痕跡)，所以需要在 `beforeEach()` 內加上 `$.ajax.mockClear()`
- 原本範例內的每個測試都 require 了 `jquery` 和 `fetchCurrentUser.js`，為了簡化測試檔，把這些 require 統一放在測試檔的最上面
- 可用與 `expect(callback.mock.calls[0][0]).toEqual()` 行為一致的 `expect(callback).toBeCalledWith()` 但更簡潔的
:::

```javascript
// __tests__/fetchCurrentUser.test.js
import $ from 'jquery';
const fetchCurrentUser = require('../src/fetchCurrentUser');

jest.mock('jquery');

beforeEach(() => {
  jest.resetModules();
  $.ajax.mockClear();
});

it('用正確的參數呼叫 $.ajax', () => {
  // 呼叫要測試的函數
  const dummyCallback = () => {};
  fetchCurrentUser(dummyCallback);

  // 確保在前兩行有正確的呼叫 $.ajax
  // 不在意 $.ajax 的請求結果，只驗證呼叫 $.ajax 時傳的參數是否正確
  expect($.ajax).toBeCalledWith({
    success: expect.any(Function),
    type: 'GET',
    url: 'http://example.com/currentUser',
  });
});

it('$.ajax 請求完成後呼叫 callback', () => {
  // 為 callback 建立一個 mock function
  const callback = jest.fn();
  fetchCurrentUser(callback);

  // 模擬 `$.ajax` 執行自己的 callback
  // 第一次呼叫的第一個參數
  $.ajax.mock.calls[0][0].success({
    firstName: 'Bobby',
    lastName: 'Marley',
  });

  // assert 模擬 `$.ajax` 呼叫的 callback 傳入的 arg
  // 第一次呼叫的第一個參數
  expect(callback.mock.calls[0][0]).toEqual({
    fullName: 'Bobby Marley',
    loggedIn: true,
  });
  expect(callback).toBeCalledWith({
    fullName: 'Bobby Marley',
    loggedIn: true,
  });
});
```

在 `fetchCurrentUser.test.js` 測試檔內我自己加了一個測試，用來測使用者點擊按鈕後是否正確的呼叫 `fetchCurrentUser()`，並且裡面呼叫的 `$.ajax()` 是否有正確的呼叫 `success` 內的 `callback` (即 `success: user => callback(parseJSON(user))` )。

但為了不讓 `$.ajax()` 發出真的請求，所以用 `$.ajax = jest.fn()` mock，接著再透過 `$.ajax.mock.calls[0][0].success({...})` 的方式呼叫 `success` 內的 `callback`。

不過，這個測試比較複雜，因為測試檔的最上面使用了 `jest.mock('jquery')`，讓整個測試檔都 mock 了 jQuery，但在 `require('../src/displayUser')` 要綁定按鈕點擊事件時需要用真的 jQuery，所以才需要用 `jest.unmock('jquery')` unmock jQuery。

除了綁定按鈕點擊事件要用真的 jQuery，觸發點擊事件後顯示使用者已登入的 `$('#username').text()` 也要用真的 jQuery (因本測試會用 `span#username` 的 `innerText` 來驗證測試)。

```javascript
// __tests__/fetchCurrentUser.test.js
import $ from 'jquery';
const fetchCurrentUser = require('../src/fetchCurrentUser');

jest.mock('jquery');

beforeEach(() => {
  jest.resetModules();
  $.ajax.mockClear();
});

it('點擊按鈕發出 $.ajax 請求，請求完成後顯示使用者已登入', () => {
  // 設定 document body
  document.body.innerHTML = `
    <span id="username"></span>
    <button id="button"></button>`;

  // 使用 `jest.unmock(...)` 後，require 的模組都會是真的，不是 mock 的
  // 因測試檔的最上面 mock 了 jQuery，而 `displayUser.js` 內
  // 需要跑真的 jQuery，所以需要 unmock jQuery，
  jest.unmock('jquery');

  // 此 module 有 side-effect
  require('../src/displayUser');

  // 觸發點擊事件內要執行真的 `$(...).text()`，所以需要真的 jQuery
  const $ = require('jquery');

  // 但只有 `$.ajax` 需要 mock
  $.ajax = jest.fn();
  // 模擬點擊按鈕
  document.querySelector('button').click();

  // 模擬 `$.ajax` 執行自己的 callback
  // 第一次呼叫的第一個參數
  $.ajax.mock.calls[0][0].success({
    firstName: 'Bobby',
    lastName: 'Marley',
  });

  // Assert span#username 的 inner text 已按預期更新了 (畫面顯示使用者已登入)
  expect($('#username').text()).toEqual('Bobby Marley - Logged In');
});
```

資料來源：
- [DOM Manipulation · Jest](https://jestjs.io/docs/en/tutorial-jquery)