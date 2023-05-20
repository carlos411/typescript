# 安裝

## TypeScript 的執行環境

TSC 本身是一個以 TypeScript 撰寫的命令列應用程式，這表示需要安裝 NodeJS。



在專案資料夾中，安裝 "TSC"、"TSLint" 及 "NodeJS 的型別宣告"：

{% code lineNumbers="true" %}
```
npm init
npm install --save-dev typescript tslint @types/node
```
{% endcode %}



## tsconfig.json 檔案

每個 TypeScript 專案的根目錄，都應該要有一個 **`tsconfig.json`** 檔案。此檔案定義應該要編譯什麼檔案、要編譯而哪個目錄、要發出哪個版本的 JavaScript。

編輯 `tsconfig.json` 檔案，內容如下：

{% code lineNumbers="true" %}
```json
{
  "compilerOptions": {
    "lib": ["es2015"],
    "module": "commonjs",
    "outDir": "dist",
    "sourceMap": true,
    "strict": true,
    "target": "es2015"
  },
  "include": ["src"]
}
```
{% endcode %}

說明：

| 選項      | 說明                                                                                                                                                          |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| include | TSC 應查看哪些資料夾以找出 TypeScript 檔。                                                                                                                               |
| lib     | TSC 應假設你要執行程式碼的環境有哪些 API，例如 ES5 的 Function.prototype.bind、ES2015 的 Object.assign 及 DOM 的 document.querySelector 等。寫 TypeScript 給瀏覽器用的時候，新都 「**dom**」 到 lib。 |
| module  | TSC 應將你的程式碼編譯為使用哪個模組系統，例 CommonJS、ES2015。                                                                                                                   |
| outDir  | TSC 應該把產出的 JavaScript 程式碼放在哪個資料夾。                                                                                                                           |
| strict  | 檢查無效程式碼時盡可能使用嚴格模式。                                                                                                                                          |
| target  | TSC 應將程式碼編譯為哪個 JavaScript 版本，例 ES3、ES5、ES2015 等。                                                                                                            |

其它更多選項，可參考：[https://www.typescriptlang.org/docs/handbook/compiler-options.html](https://www.typescriptlang.org/docs/handbook/compiler-options.html)

也可透過以下指令來查看可用的命令列選項：

```
./node_modules/.bin/tsc --help
```



## tslint.json

專案資料夾中，也應有一個 **`tslint.json`** 檔案，其中包括你的 TSLint 配置，規範了希望程式碼使用的編碼風格，例如要用 tab 或 space 等。

以下命令會產生帶有預設 TSLint 配置的一個 **`tslint.json`** 檔案：

```
./node_modules/.bin/tslint --init
```

產出來的 **`tslint.json`** 檔案，預設內容如下：

{% code lineNumbers="true" %}
```json
{
    "defaultSeverity": "error",
    "extends": [
        "tslint:recommended"
    ],
    "jsRules": {},
    "rules": {},
    "rulesDirectory": []
}
```
{% endcode %}

設定不用加分號(semicolon)：

{% code lineNumbers="true" %}
```json
{
    "defaultSeverity": "error",
    "extends": [
        "tslint:recommended"
    ],
    "jsRules": {},
    "rules": {
        "semicolon": false
    },
    "rulesDirectory": []
}
```
{% endcode %}

更多 TSLint 設明文件，可參考：[https://palantir.github.io/tslint/rules/](https://palantir.github.io/tslint/rules/)



## 建立第一個 ts 檔案

在專案資料夾後，建立 **`src`** 資料夾，然後在 src 資料夾中，建立 **`index.ts`** 檔案，內容如下：

```
console.log("Hello Typescript! 中文")
```



編譯：

```
./node_modules/.bin/tsc
```

此時會自動產生 **`dist`** 資料夾，然後產生一個新的 **`index.js`** 檔案和 **`index.js.map`** 檔案。



以 NodeJS 執行：

```
node ./dist/index.js
```



## 其它參考套件

ts-node：[https://www.npmjs.com/package/ts-node](https://www.npmjs.com/package/ts-node)



