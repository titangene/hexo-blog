---
title: Jest：Describe & Test case
date: 2020-06-14 23:57:34
author: Titangene
tags:
  - Jest
  - JavaScript
  - Unit Testing
  - w3HexSchool
categories:
  - Testing
cover_image: /images/cover/jest.jpg
---

![](../images/cover/jest.jpg)

上次介紹了 [Jest 提供的 matcher](https://titangene.github.io/article/jest-matcher-assertion.html)，可讓你驗證程式碼是否符合預期，而這次來說明如何透過 `describe` 和 `test` 區塊來組織測試案例。當需求變多時，可針對需求來分類測試案例，將相關的測試放在同一個群組區塊內，此時就會用到 Jest 提供的 `describe` 和 `test` 區塊。

<!-- more -->

- `test` 區塊：即測試案例 (test case)，使用 matcher 驗證程式執行結果是否符合預期
- `describe` 區塊：將多個相關的 `test` 區塊放在一起，便於組織測試案例

> 其他 Jest 相關文章可參閱 [Jest 系列文章](https://titangene.github.io/tags/jest/)。

下面範例都會用數學運算的範例來說明：

```javascript
const addition = (x, y) => {
  if (typeof(x) === 'number' && typeof(y) === 'number')
   throw new CustomError('請輸入數字');
  return x + y;
};
const subtraction = (x, y) => {
  if (typeof(x) === 'number' && typeof(y) === 'number')
   throw new CustomError('請輸入數字');
  return x - y;
};
```

# `describe` 區塊

## `describe(name, fn)`

argument：
- `name`：描素此 `describe` 區塊的敘述
- `fn`：包含 `test` 區塊的函數

用來將多個相關的 `test` 區塊放在同一個 `describe` 區塊內，以便於將測試組織成群組。例如：

```javascript
describe('數學運算', () => {
  test('加法運算', () => {
    expect(addition(1, 2)).toBe(3);
  });
  
  test('減法運算', () => {
    expect(subtraction(3, 1)).toBe(2);
  });
});
```

但寫測試不一定要用 `describe` 區塊，也可以不用將 `test` 區塊包在 `describe` 區塊內，而是直接寫在測試檔上，但建議你使用 `describe` 區塊。

也可以巢狀使用 `describe` 區塊：

```javascript
describe('數學運算', () => {
  describe('加法運算', () => {
    describe('傳入非數字值', () => {
      test('傳入布林值應拋出 CustomError', () => {
        expect(addition(true, 2)).toThrow(CustomError);
      });

      test('傳入字串值應拋出 CustomError', () => {
        expect(addition('a', 1)).toThrow(CustomError);
      });
    });
    
    describe('傳入數字值', () => {
      test('應回傳正確的運算結果', () => {
        expect(addition(1, 2)).toBe(3);
      });
    });
  });

  describe('減法運算', () => {
    describe('傳入非數字值', () => {
      test('傳入布林值應拋出 CustomError', () => {
        expect(subtraction(true, 2)).toThrow(CustomError);
      });

      test('傳入字串值應拋出 CustomError', () => {
        expect(subtraction('a', 1)).toThrow(CustomError);
      });
    });
    
    describe('傳入數字值', () => {
      test('應回傳正確的運算結果', () => {
        expect(subtraction(3, 1)).toBe(2);
      });
    });
  });
});
```

## `describe.only(name, fn)`

alias：`fdescribe(name, fn)`

讓測試只跑使用 `describe.only()` 的 `describe` 區塊。

例如：

```javascript
describe.only('數學運算', () => {
  test('加法運算', () => {
    expect(addition(1, 2)).toBe(3);
  });
  
  test('減法運算', () => {
    expect(subtraction(3, 1)).toBe(2);
  });
});

describe('其他', () => {
  // 此區塊的測試會被略過
});
```

## `describe.skip(name, fn)`

alias：`xdescribe(name, fn)`

不想執行指定的 `describe` 區塊可用 `describe.skip()`。

例如：

```javascript
describe('數學運算', () => {
  test('加法運算', () => {
    expect(addition(1, 2)).toBe(3);
  });
  
  test('減法運算', () => {
    expect(subtraction(3, 1)).toBe(2);
  });
});

describe.skip('其他', () => {
  // 略過此區塊的測試
});
```

# `test` 區塊

## `test(name, fn, timeout)`

alias：`it(name, fn, timeout)`

argument：
- `name`：測試名稱
- `fn`：包含要測試的期望函數
- `timeout` (可選)：測試終止之前要等待的時間 (單位毫秒)，預設為 5 秒

測試檔案內一定要有 `test` 區塊才能測試程式。

剛剛有提到，不一定要 `describe` 區塊，可直接在測試檔內放 `test` 區塊即可。例如：

```javascript
test('加法運算', () => {
  expect(addition(1, 2)).toBe(3);
});
```

## `test.only(name, fn, timeout)`

alias：`it.only(name, fn, timeout)` 和 `fit(name, fn, timeout)`

在 debug 測試檔內的測試時，通常只需要執行一部分的測試，此時就可以將那些 `test` 區塊加上 `.only` 來指定要執行哪些測試，其餘的測試都會被略過。

> 註：`timeout` argument 跟 `test()` 一樣，所以不再次重複說明。

例如：

```javascript
test.only('加法運算', () => {
  expect(addition(1, 2)).toBe(3);
});

test('其他運算', () => {
  // 略過此區塊的測試
});
```

:::info
`test.only` 只會在 debug 階段使用，不建議 commit 至 Git 版本控制中，記得要在 debug 後刪掉 `.only`。
:::

## `test.skip(name, fn)`

alias：`it.skip(name, fn)` 或 `xit(name, fn)` 或 `xtest(name, fn)`

若要跳過一些 `test` 區塊的測試，可用 `test.skip()`。

例如：

```javascript
test('加法運算', () => {
  expect(addition(1, 2)).toBe(3);
});

test.skip('其他運算', () => {
  // 略過此區塊的測試
});
```

若不想執行某 `test` 區塊的測試，雖然可將該測試註解掉，但建議用 `test.skip`，因為只少會保留原本的縮排和顯示程式碼 highlight。

## `test.todo(name)`

alias：`it.todo(name)`

規劃要寫的測試 (寫在 `name` argument 上) 可用 `test.todo()`，這些測試會在摘要輸出中 highlight 顯示，以便提醒你還有多少測試還沒寫。

若提供 test callback 函數，`test.todo()` 會拋出錯誤。若你已實作了該測試，且該測試已出錯，而你不想執行此測試，請改用 `test.skip()`。

例如：

```javascript
const add = (a, b) => a + b;

test.todo('add should be associative');
```

資料來源：
- [Globals · Jest](https://jestjs.io/docs/en/api)