# 其它

## 前端框架

內建的 DOM API 全都具備型別安全性。要從 TypeScript 使用它們，只需在專案的 `tsconfig.json` 中引入它們的型別宣告即可：

{% code lineNumbers="true" %}
```json
{
  "compilerOptions": {
    "lib": ["dom", "es2015"]
  }
}
```
{% endcode %}

以上這樣的設定，就是在告知 TypeScript 要在型別檢查你的程式碼時，引入 `lib.dom.d.ts`，也就是它內建的瀏覽器和 DOM 型別宣告。



## 動態載入

TypeScript 僅在 **`esnext`** 模組模式中支援動態載入。所以若要使用**動態載入(dynamic imports)**，就在 **`tsconfig.json`** 的 **compilerOptions** 中設定 **`{"module": "esnext"}`** 。





