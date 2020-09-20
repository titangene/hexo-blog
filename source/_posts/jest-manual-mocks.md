---
title: Jest：Manual Mocks
date: 2020-07-26 23:59:04
author: Titangene
tags:
  - Jest
  - JavaScript
  - Unit Testing
  - mock
  - w3HexSchool
categories:
  - Testing
cover_image: /images/cover/jest.jpg
---

![](../images/cover/jest.jpg)

manual mock 是用於透過 mock 資料來對功能進行 stub out。例如：若你不想存取網站或 DB 之類的遠端資源，可能需要使用 fake data 來 manual mock 這些功能，以確保可以快速測試且不會出錯。

<!-- more -->

> 其他 Jest 相關文章可參閱 [Jest 系列文章](https://titangene.github.io/tags/jest/)。

# Mocking user modules

manual mock 是透過將模組寫在相鄰該模組的 `__mocks__/` 子目錄內來定義的。例如：要在 `models` 目錄 mock 一個名為 `user` 的模組，請建立名為 `user.js` 的檔案，並將該檔案放在 `models/__mocks__` 目錄中。

```
.
├── models
│   ├── __mocks__
│   │   └── user.js
│   └── user.js
└── main.js
```

:::info
`__mocks__` 資料夾會區分大小寫 (case-sensitive)，所以在某些 OS 上命名成 `__MOCKS__` 會失效。
:::

## 範例

完整目錄如下：

```shell
.
├── src
│   ├── controllers
│   │   ├── __mocks__
│   │   │   └── user.js
│   │   └── user.js
│   ├── models
│   │   ├── index.js
│   │   └── user.js
│   ├── setup.js
│   └── main.js
└── __tests__
    └── userMocked.test.js
```

在 `src/controllers/user.js` 內提供取得第一個使用者資訊的 `getFirstUser()` (從 DB 拿出來的資料)：

```javascript
// src/controllers/user.js
const User = require('../models').User;

module.exports = {
  async getFirstUser() {
    try {
      const userId = 1;
      const user = await models.user.findByPk(userId);
      return user;
    } catch (error) {
      console.log(error.message);
    }
  }
};
```

接著在 `src/controllers` 目錄內新增 `__mocks__` 子目錄，並在裡面建立名為 `user.js` 的檔案，內容如下，manual mock 的內容就是回傳固定的 fake data，讓每次執行測試不用真的去讀取資料庫的資料：

```javascript
// src/controllers/__mocks__/user.js
const User = jest.createMockFromModule('../user');

User.getFirstUser = async () => ({
  name: 'Mock name',
  age: 87
});

module.exports = User;
```

測試要引入 manual mock 的模組時，記得要呼叫 `jest.mock('./moduleName')`，其餘測試寫法就跟平時一樣：

```javascript
// __tests__/userMocked.test.js
import User from '../src/controllers/user';

jest.mock('../src/controllers/user');

it('if user model is mocked', async () => {
  const expected = {name: 'Mock name', age: 87};

  const user = await User.getFirstUser();

  expect(user).toMatchObject(expected);
});
```

# Mocking Node modules

若要 mock 的模組是 Node 模組 (例如：`lodash` )，則 mock 應放在與 `node_modules` 相鄰的 `__mocks__` 目錄中 (除非你將 [`roots`](https://jestjs.io/docs/en/configuration#roots-arraystring) 配置為指向專案 root 目錄以外的資料夾)，並且會被自動 mock。無需明確呼叫 `jest.mock('./moduleName')`：

```
.
├── __mocks__
│   └── lodash.js
├── node_modules
│   └── lodash
│       └── lodash.js
└── main.js
```

可以透過在目錄結構中建立一個與 scoped module 名稱 match 的檔案來 mock scoped module。例如：若要 mock 名為 `@scope/project-name` 的 scoped module，請建立名為 `__mocks__/@scope/project-name.js` 的檔案：

```
.
├── __mocks__
│   └── @scope
│       └── project-name.js
├── node_modules
│   └── @scope
│       └── project-name
└── main.js
```

:::warning
警告：若要 mock Node 的核心模組 (例如：`fs` 或 `path` )，必須要明確呼叫 (例如：`jest.mock('path')` )，因為預設不會 mock 核心 Node 模組。
:::

# 例如

```
.
├── config
├── __mocks__
│   └── fs.js
├── models
│   ├── __mocks__
│   │   └── user.js
│   └── user.js
├── node_modules
└── views
```

當給定模組有 manual mock 時，Jest 的模組系統會在明確呼叫 `jest.mock('moduleName')` 時使用該模組。

但是，當配置的 `automock` 設為 `true` 時，即使未呼叫 `jest.mock('moduleName')`，也會用 manual mock implementation 來取代自動建立的 mock。

若要取消 mock，需要在應該使用實際模組 implementation 的測試中顯式明確呼叫 [`jest.unmock('moduleName')`](https://jestjs.io/docs/en/jest-object#jestunmockmodulename)。

:::info
為了正確的 mock，Jest 需要 `jest.mock('moduleName')` 與 `require` / `import` 陳述句在同一個 scope 內。
:::

例如：有一個模組可提供給定目錄中所有檔案的摘要。這裡使用核心 (內建) `fs` 模組：

```javascript
// fileSummarizer.js
'use strict';

const fs = require('fs');

function summarizeFilesInDirectorySync(directory) {
  return fs.readdirSync(directory).map(fileName => ({
    directory,
    fileName,
  }));
}

exports.summarizeFilesInDirectorySync = summarizeFilesInDirectorySync;
```

由於我們希望測試避免實際對硬碟進行存取 (因為會很慢且脆弱)，所以會透過擴充自動 mock 來為 `fs` 模組建立 manual mock。我們的 manual mock 會實作可用於測試的 `fs` API 的自訂版：

```javascript
// __mocks__/fs.js
'use strict';

const path = require('path');

const fs = jest.createMockFromModule('fs');

// 這是自訂函數，測試可在 setup 過程中使用此函數
// 來指定使用任何 `fs` API 時 mock filesystem 上的檔案應該為何
let mockFiles = Object.create(null);
function __setMockFiles(newMockFiles) {
  mockFiles = Object.create(null);
  for (const file in newMockFiles) {
    const dir = path.dirname(file);

    if (!mockFiles[dir]) {
      mockFiles[dir] = [];
    }
    mockFiles[dir].push(path.basename(file));
  }
}

// 自訂版的 `readdirSync` 透過
// 從`__setMockFiles` 設定的指定 mocked 檔案列表中讀取
function readdirSync(directoryPath) {
  return mockFiles[directoryPath] || [];
}

fs.__setMockFiles = __setMockFiles;
fs.readdirSync = readdirSync;

module.exports = fs;
```

現在我們來寫測試。請注意，由於它是核心 Node 模組，所以需要明確告知我們要 mock `fs` 模組 (也就是需明確呼叫 `jest.mock('fs')` )：

```javascript
// __tests__/FileSummarizer-test.js
'use strict';

jest.mock('fs');

describe('listFilesInDirectorySync', () => {
  const MOCK_FILE_INFO = {
    '/path/to/file1.js': 'console.log("file1 contents");',
    '/path/to/file2.txt': 'file2 contents',
  };

  beforeEach(() => {
    // 在每次測試之前 setup 一些 mocked out 檔案資訊
    require('fs').__setMockFiles(MOCK_FILE_INFO);
  });

  test('includes all files in the directory in the summary', () => {
    const FileSummarizer = require('../FileSummarizer');
    const fileSummary = FileSummarizer.summarizeFilesInDirectorySync(
      '/path/to',
    );

    expect(fileSummary.length).toBe(2);
  });
});
```

此範例的 mock 是用 [`jest.createMockFromModule`](https://jestjs.io/docs/en/jest-object#jestcreatemockfrommodulemodulename) 來生成自動 mock，並覆蓋預設的行為 (也就是大部份都是真的，但少部份是假的)。推薦此方法，但不強迫。若不想使用自動 mock，則可從 mock 檔案中 export 自己的函數。完全 manual mock 的一個缺點是它們是手動的，代表必須在模組 mocking changes 時隨時手動更新它們。所以最好在滿足你的需求時使用或擴充自動 mock。

為了確保 manual mock 和實際實作保持同步，在 export manual mock 模組之前，請在 manual mock 中使用 [`jest.requireActual(moduleName)`](https://jestjs.io/docs/en/jest-object#jestrequireactualmodulename) 使用真實的模組，並用 mock 函數對其進行變更，這可能會很有用。

> 範例程式碼：[examples/manual-mocks](https://github.com/facebook/jest/tree/master/examples/manual-mocks)。

# 與 ES 模組引入一起使用

若你正在使用 ES 模組 imports，通常會傾向將 `import` 陳述句放在測試檔案的最上面。但通常你需要指示 Jest 在模組使用 mock 之前使用它。所以 Jest 會自動將 `jest.mock` 呼叫 hoist 至模組的最上面 (在 import 之前)。

> 詳情可參閱 [kentcdodds/how-jest-mocking-works](https://github.com/kentcdodds/how-jest-mocking-works)。

# JSDOM 中未實作的 mock 方法

若某些程式碼使用的方法尚未實作 JSDOM (Jest 使用的 DOM implementation)，則不易於測試。

例如：`Window.matchMedia()` 的情況。Jest 會回傳 `TypeError: window.matchMedia is not a function`，不能正確執行測試。

在這種情況下，在測試檔案中 mock `matchMedia` 應該可以解決此問題：

```javascript
// __tests__/matchMedia.mock.js
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),     // deprecated
    removeListener: jest.fn(),  // deprecated
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

:::info
`matchMedia.mock.js` 這個檔案放哪都可以，但記得配置要設成 `testEnvironment: "jsdom"`。
:::

- 若在測試中 invoked 的函數 (或方法) 中使用 `window.matchMedia()`，則此方法可以運作
- 若在測試檔案中直接執行 `window.matchMedia()`，Jest 會出現相同的錯誤

所以解決方案是將 manual mock 放在獨立的檔案中，並在測試之前將其包含在測試檔案中：

```javascript
// src/useMatchMedia.js
export default function useMatchMedia() {
  return window.matchMedia('xx').matches;
}
```

```javascript
// __tests__/userMocked.test.js
import "./matchMedia.mock"; // 必須在測試檔案之前 import
import useMatchMedia from "../src/useMatchMedia";

it('use matchMedia()', () => {
  // 在這裡測試 method...
  expect(useMatchMedia()).toBeFalsy();
});
```

資料來源：
- [Manual Mocks · Jest](https://jestjs.io/docs/en/manual-mocks)