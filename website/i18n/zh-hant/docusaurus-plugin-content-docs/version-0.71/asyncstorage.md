---
id: asyncstorage
title: '🚧 AsyncStorage'
---

> **已移除。** 請改用 [社群套件](https://reactnative.directory/?search=storage) 中的方案。

`AsyncStorage` 是一個未加密、非同步、持久化且應用全域的鍵值儲存系統，應取代 LocalStorage 使用。

建議在輕量使用之外，應基於 `AsyncStorage` 封裝抽象層，而非直接操作全域性的 `AsyncStorage`。

On iOS, `AsyncStorage` is backed by native code that stores small values in a serialized dictionary and larger values in separate files. On Android, `AsyncStorage` will use either [RocksDB](http://rocksdb.org/) or SQLite based on what is available.

JavaScript 層的 `AsyncStorage` 提供清晰 API 介面、真實的 `Error` 物件與非批次操作方法，所有 API 均返回 `Promise` 物件。

Importing the `AsyncStorage` library:

```jsx
import {AsyncStorage} from 'react-native';
```

持久化資料：

```jsx
_storeData = async () => {
  try {
    await AsyncStorage.setItem(
      '@MySuperStore:key',
      'I like to save it.',
    );
  } catch (error) {
    // Error saving data
  }
};
```

讀取資料：

```jsx
_retrieveData = async () => {
  try {
    const value = await AsyncStorage.getItem('TASKS');
    if (value !== null) {
      // We have data!!
      console.log(value);
    }
  } catch (error) {
    // Error retrieving data
  }
};
```

---

# 參考文獻

## 方法

### `getItem()`

```jsx
static getItem(key: string, [callback]: ?(error: ?Error, result: ?string) => void)
```

根據 `key` 取得項目並於完成時觸發回調。返回 `Promise` 物件。

**參數：**

| Name     | Type                                        | Required | Description                                                       |
| -------- | ------------------------------------------- | -------- | ----------------------------------------------------------------- |
| key      | string                                      | Yes      | Key of the item to fetch.                                         |
| callback | `?(error: ?Error, result: ?string) => void` | No       | Function that will be called with a result if found or any error. |

---

### `setItem()`

```jsx
static setItem(key: string, value: string, [callback]: ?(error: ?Error) => void)
```

為 `key` 設定值並於完成時觸發回調。返回 `Promise` 物件。

**參數：**

| Name     | Type                       | Required | Description                                  |
| -------- | -------------------------- | -------- | -------------------------------------------- |
| key      | string                     | Yes      | Key of the item to set.                      |
| value    | string                     | Yes      | Value to set for the `key`.                  |
| callback | `?(error: ?Error) => void` | No       | Function that will be called with any error. |

---

### `removeItem()`

```jsx
static removeItem(key: string, [callback]: ?(error: ?Error) => void)
```

移除指定 `key` 的項目並於完成時觸發回調。返回 `Promise` 物件。

**參數：**

| Name     | Type                       | Required | Description                                  |
| -------- | -------------------------- | -------- | -------------------------------------------- |
| key      | string                     | Yes      | Key of the item to remove.                   |
| callback | `?(error: ?Error) => void` | No       | Function that will be called with any error. |

---

### `mergeItem()`

```jsx
static mergeItem(key: string, value: string, [callback]: ?(error: ?Error) => void)
```

合併現有 `key` 值與輸入值（假設兩者均為 JSON 字串）。返回 `Promise` 物件。

**注意：** 並非所有原生實作都支援此功能。

**參數：**

| Name     | Type                       | Required | Description                                  |
| -------- | -------------------------- | -------- | -------------------------------------------- |
| key      | string                     | Yes      | Key of the item to modify.                   |
| value    | string                     | Yes      | New value to merge for the `key`.            |
| callback | `?(error: ?Error) => void` | No       | Function that will be called with any error. |

範例：

```jsx
let UID123_object = {
  name: 'Chris',
  age: 30,
  traits: {hair: 'brown', eyes: 'brown'},
};
// You only need to define what will be added or updated
let UID123_delta = {
  age: 31,
  traits: {eyes: 'blue', shoe_size: 10},
};

AsyncStorage.setItem(
  'UID123',
  JSON.stringify(UID123_object),
  () => {
    AsyncStorage.mergeItem(
      'UID123',
      JSON.stringify(UID123_delta),
      () => {
        AsyncStorage.getItem('UID123', (err, result) => {
          console.log(result);
        });
      },
    );
  },
);

// Console log result:
// => {'name':'Chris','age':31,'traits':
//    {'shoe_size':10,'hair':'brown','eyes':'blue'}}
```

---

### `clear()`

```jsx
static clear([callback]: ?(error: ?Error) => void)
```

清除 _所有_ 客戶端、函式庫等的 `AsyncStorage` 資料。通常不應直接呼叫此方法，建議改用 `removeItem` 或 `multiRemove` 僅清除自身應用鍵值。返回 `Promise` 物件。

**參數：**

| Name     | Type                       | Required | Description                                  |
| -------- | -------------------------- | -------- | -------------------------------------------- |
| callback | `?(error: ?Error) => void` | No       | Function that will be called with any error. |

---

### `getAllKeys()`

```jsx
static getAllKeys([callback]: ?(error: ?Error, keys: ?Array<string>) => void)
```

取得當前應用 _所有_ 已知鍵值（含其他呼叫者與函式庫）。返回 `Promise` 物件。

**參數：**

| Name     | Type                                             | Required | Description                                                     |
| -------- | ------------------------------------------------ | -------- | --------------------------------------------------------------- |
| callback | `?(error: ?Error, keys: ?Array<string>) => void` | No       | Function that will be called with all keys found and any error. |

---

### `flushGetRequests()`

```jsx
static flushGetRequests(): [object Object]
```

使用單一批次呼叫刷新所有待處理的請求以獲取數據。

---

### `multiGet()`

```jsx
static multiGet(keys: Array<string>, [callback]: ?(errors: ?Array<Error>, result: ?Array<Array<string>>) => void)
```

This allows you to batch the fetching of items given an array of `key` inputs. Your callback will be invoked with an array of corresponding key-value pairs found:

```
multiGet(['k1', 'k2'], cb) -> cb([['k1', 'val1'], ['k2', 'val2']])
```

該方法返回一個 `Promise` 物件。

**參數：**

| Name     | Type                                                              | Required | Description                                                                                                         |
| -------- | ----------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------- |
| keys     | `Array<string>`                                                   | Yes      | Array of key for the items to get.                                                                                  |
| callback | `?(errors: ?Array<Error>, result: ?Array<Array<string>>) => void` | No       | Function that will be called with a key-value array of the results, plus an array of any key-specific errors found. |

範例：

```jsx
AsyncStorage.getAllKeys((err, keys) => {
  AsyncStorage.multiGet(keys, (err, stores) => {
    stores.map((result, i, store) => {
      // get at each store's key/value so you can work with it
      let key = store[i][0];
      let value = store[i][1];
    });
  });
});
```

---

### `multiSet()`

```jsx
static multiSet(keyValuePairs: Array<Array<string>>, [callback]: ?(errors: ?Array<Error>) => void)
```

使用此方法批次儲存多組鍵值對。操作完成時將收到單一回調函數並包含任何錯誤：

```
multiSet([['k1', 'val1'], ['k2', 'val2']], cb);
```

該方法返回一個 `Promise` 物件。

**參數：**

| Name          | Type                               | Required | Description                                                                  |
| ------------- | ---------------------------------- | -------- | ---------------------------------------------------------------------------- |
| keyValuePairs | `Array<Array<string>>`             | Yes      | Array of key-value array for the items to set.                               |
| callback      | `?(errors: ?Array<Error>) => void` | No       | Function that will be called with an array of any key-specific errors found. |

---

### `multiRemove()`

```jsx
static multiRemove(keys: Array<string>, [callback]: ?(errors: ?Array<Error>) => void)
```

呼叫此方法以批次刪除 `keys` 陣列中的所有鍵。返回一個 `Promise` 物件。

**參數：**

| Name     | Type                               | Required | Description                                                             |
| -------- | ---------------------------------- | -------- | ----------------------------------------------------------------------- |
| keys     | `Array<string>`                    | Yes      | Array of key for the items to delete.                                   |
| callback | `?(errors: ?Array<Error>) => void` | No       | Function that will be called an array of any key-specific errors found. |

範例：

```jsx
let keys = ['k1', 'k2'];
AsyncStorage.multiRemove(keys, err => {
  // keys k1 & k2 removed, if they existed
  // do most stuff after removal (if you want)
});
```

---

### `multiMerge()`

```jsx
static multiMerge(keyValuePairs: Array<Array<string>>, [callback]: ?(errors: ?Array<Error>) => void)
```

批次合併指定鍵集的現有值與新值（假設值為字串化的JSON）。返回一個 `Promise` 物件。

**注意**：並非所有原生實作都支援此功能。

**參數：**

| Name          | Type                               | Required | Description                                                                  |
| ------------- | ---------------------------------- | -------- | ---------------------------------------------------------------------------- |
| keyValuePairs | `Array<Array<string>>`             | Yes      | Array of key-value array for the items to merge.                             |
| callback      | `?(errors: ?Array<Error>) => void` | No       | Function that will be called with an array of any key-specific errors found. |

範例：

```jsx
// first user, initial values
let UID234_object = {
  name: 'Chris',
  age: 30,
  traits: {hair: 'brown', eyes: 'brown'},
};

// first user, delta values
let UID234_delta = {
  age: 31,
  traits: {eyes: 'blue', shoe_size: 10},
};

// second user, initial values
let UID345_object = {
  name: 'Marge',
  age: 25,
  traits: {hair: 'blonde', eyes: 'blue'},
};

// second user, delta values
let UID345_delta = {
  age: 26,
  traits: {eyes: 'green', shoe_size: 6},
};

let multi_set_pairs = [
  ['UID234', JSON.stringify(UID234_object)],
  ['UID345', JSON.stringify(UID345_object)],
];
let multi_merge_pairs = [
  ['UID234', JSON.stringify(UID234_delta)],
  ['UID345', JSON.stringify(UID345_delta)],
];

AsyncStorage.multiSet(multi_set_pairs, err => {
  AsyncStorage.multiMerge(multi_merge_pairs, err => {
    AsyncStorage.multiGet(['UID234', 'UID345'], (err, stores) => {
      stores.map((result, i, store) => {
        let key = store[i][0];
        let val = store[i][1];
        console.log(key, val);
      });
    });
  });
});

// Console log results:
// => UID234 {"name":"Chris","age":31,"traits":{"shoe_size":10,"hair":"brown","eyes":"blue"}}
// => UID345 {"name":"Marge","age":26,"traits":{"shoe_size":6,"hair":"blonde","eyes":"green"}}
```