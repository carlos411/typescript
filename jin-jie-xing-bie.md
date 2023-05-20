# 進階型別

## 子型別與超型別

**子型別(Subtype)**、**超型別(Supertype)**。

例：Object 是 Array 的**超型別**；反過來則是 Array 是 Object 的**子型別**。可參考「型別」的圖。



## 變異性

協變性(covariance)只是四種變異性(variance)中的一種：

* Invariance(不變性)：想要一個特定型別 T。
* Covariance(協變性)：想要一個特定型別 T 或其子型別。
* Contravariance(抗變性)：想要一個特定型別 T 或其超型別。
* Bivariance(雙變性)：想要一個特定型別 T 或其子型別或超型別都可。

不用背，留意編輯器中是否有錯誤提示即可。



## 可指定性

在 TypeScript 中，**可指定性**指的是 TypeScript 用來判斷你是否能在要求型別 B 的地方，使用型別 A 的規則。



**型別 A 可以指定給型別 B 嗎**？對於 **非 enum 型別**來說，有以下兩個規則，符合其中一個即可：

1. 型別 A 是型別 B 的子型別，就可以。
2. 是上例規則1的例子，也就是若型別 A 是 any，那也可以。



對於使用 **enum** 或 **const** 關鍵字創建的列舉型別(enum types)，**型別 A 可以指定給一個 enum B 的話**，需符合下列兩個規則其中一個：

1. A 是 enum B 的一個成員。
2. B 至少有一個成員是 number，而 A 是一個 number。



## 型別擴展

**型別擴展(type widening)** 是了解 TypeScript 的型別推論(type inference)如何運作的關鍵。



當宣告一個變數，它是可在之後變動的(也就是用 var 或 let 做宣告的變數)，其型別會是：

{% code lineNumbers="true" %}
```typescript
let a = "x"      // string
let b = 3        // number
var c = true     // boolean
const d = {x: 3} // {x: number}

enum E {X, Y, Z}
let e = E.X      // E
```
{% endcode %}



然而，對於不可變的宣告(即使用 const 做宣告的變數)：

{% code lineNumbers="true" %}
```typescript
const a = "x"  // "x"
const b = 3    // 3
const c = true // true

enum E {X, Y, Z}
const e = E.X  // E.X
```
{% endcode %}

也可以使用明確的型別注釋，來防止型別被擴展：

{% code lineNumbers="true" %}
```typescript
let a: "x" = "x"         // "x"
let b: 3 = 3             // 3
var c: true = true       // true
const d: {x: 3} = {x: 3} // {x: 3}
```
{% endcode %}



當使用 var 或 let 重新指定一個未擴展的型別，TypeScript 會為你擴展它。要告訴 TypeScript 讓它保持狹義，就為你原本的宣告加上一個明確的型別注釋：

{% code lineNumbers="true" %}
```typescript
const a = "x"      // "x"
let b = a          // string

const c: "x" = "x" // "x"
let d = c          // "x"
```
{% endcode %}



初始化為 null 或 undefined 的變數，會被擴展為 any：

{% code lineNumbers="true" %}
```typescript
let a = null // any
a = 3        // any
a = "b"      // any
```
{% endcode %}

但是當被初始化為 null 或 undefined 的一個變數離開了它在其中宣告的範疇(scope)，TypeScript 就會指定一個確切型別給它：

{% code lineNumbers="true" %}
```typescript
function x(){
  let a = null // any
  a = 3        // any
  a = "b"      // any
  return a
}

let c = x()    // string
```
{% endcode %}



### const 型別

TypeScript 有一種特殊的 **const 型別**，可用它來選擇停用型別擴展：

{% code lineNumbers="true" %}
```typescript
let a = {x: 3}          // {x: number}
let b: {x: 3}           // {x: 3}
let c = {x: 3} as const // {readonly x: 3}
```
{% endcode %}

const 讓你的型別不會擴展，並且會遞迴地將其成員標示為 readonly：

{% code lineNumbers="true" %}
```typescript
let d = [1, {x: 2}]          // (number | {x: number})[]
let e = [1, {x: 2}] as const // readonly [1, {readonly x: 2}]
```
{% endcode %}



### 超量特性檢查(Excess property checking)

TypeScript 檢查**一個物件型別**是否可以指定給**另一個物件型別**時，**型別擴展**也會出現。



一個**新創的物件字面值型別(fresh object literal type)** 就是 TypeScript 從一個物件字面值(object literal)推論出的型別。如果那個物件字面值使用了一個**型別斷言**，或是**被指定給一個變數**，那麼新創的物件字面值型別會被**擴展為一個常見的物件型別**。



例：

{% code lineNumbers="true" %}
```typescript
type Options = {
  baseURL: string
  cacheSize?: number
  tier?: "prod" | "dev"
}

class API {
  constructor(private options: Options) {

  }
}

let badOptions = { // 指定給變數
  baseURL: "https://api.example.com",
  badTier: "prod"
}

new API(badOptions) // 不會報錯
```
{% endcode %}



## 精煉細分(Refinement)

TypeScript 進行以**流程**為基礎的型別推論，那是一種**符號式**的執行。

例：

{% code lineNumbers="true" %}
```typescript
type Unit = "cm" | "px" | "%"

let units: Unit[] = ["cm", "px", "%"]

function parseUnit(value: string): Unit | null {
  for(let i = 0; i < units.length; i++) {
    if(value.endsWith(units[i])){
      return units[i]
    }
  }
  return null
}

type Width = {
  unit: Unit
  value: number
}

function parseWidth(width: number | string | null | undefined): Width | null {
  // TypeScript 夠聰明，知道在 JavaScript 中對 null 進行寬鬆的相等性(兩個等號)檢查時，
  // 不管是 null 或 undefined 都會回傳 true。
  // 我們說型別從 number | string | null | undefined 細分成了 number | string。
  if(width == null){ // 如果 width 是 null 或 undefined，就提早回傳
    return null
  }

  if(typeof width === "number"){ // 如果 width 是一個數字，就預設為像素
    return {unit: "px", value: width}
  }

  
  let unit = parseUnit(width)
  if(unit){
    return {unit, value: parseFloat(width)}
  }

  return null
}
```
{% endcode %}



## 總體性(Totality)

**總體性(totality)**也叫做**周延檢查(exhaustiveness checking)**，它讓型別檢查器有辦法確保你**涵蓋了所有情況**。



例：

{% code lineNumbers="true" %}
```typescript
type Weekday = "Mon" | "Tue" | "Wed" | "Thu" | "Fri"
type Day = Weekday | "Sat" | "Sun"

// 會報錯：Function lacks ending return statement and return type does not include 'undefined'.
function getNextDay(w: Weekday): Day{
  switch(w){
    case "Mon": return "Tue"
  }
}
```
{% endcode %}

在 `tsconfig.json` 檔案中，建議開啟 **noImplicitReturns** 設定。它可以讓任何函式若沒有明確的 return，就會報錯。



## 進階的物件型別

聯集( **`|`** )和交集( **`&`** )這兩個是型別運算子，還有其它的。



### keying-in 運算子

例：

{% code lineNumbers="true" %}
```typescript
type APIResponse = {
  user: {
    userId: string
    friendList: {
      count: number
      friends: {
        firstName: string
        lastName: string
      }[]
    }
  }
}

function getAPIResponse(): Promise<APIResponse> {
  // ...
}

function renderFriendList(friendList: unknown) { // 這裡先是 uknown
  // ...
}

let response = await getAPIResponse()
renderFriendList(response.user.friendList)
```
{% endcode %}

上述的第 18 行，friendList 目前是 uknown，可以定出它的型別，然後用它重新實作 APIResponse 型別，變成這樣：

{% code lineNumbers="true" %}
```typescript
type FriendList = { // 定出 FriendList 的形狀
  count: number
  friends: {
    firstName: string
    lastName: string
  }[]
}

type APIResponse = {
  user: {
    userId: string
    friendList: FriendList // 這裡使用
  }
}

function renderFriendList(friendList: FriendList) { // 這裡也使用
  // ...
}
```
{% endcode %}

但如此一來，就得為每個頂層型別想出名稱，這並不是每次都想做的事情。

取而代之，可以 **`key in(鍵入)`** 你的型別：

{% code lineNumbers="true" %}
```typescript
type APIResponse = {
  user: {
    userId: string
    friendList: {
      count: number
      friends: {
        firstName: string
        lastName: string
      }[]
    }
  }
}

type FriendList = APIResponse["user"]["friendList"] // 改成用 key in 這個方式

function renderFriendList(friendList: FriendList) {
  // ...
}
```
{% endcode %}

可以 key in 任何形狀，以及任何陣列，例如，要取得一個個別朋友的型別，可以這樣寫：

{% code lineNumbers="true" %}
```typescript
type Friend = FriendList["friends"][number]
// number 是 key in 陣列型別的一種方式；
// 至於位元組(tuples)，就使用 0、1 或其他數字字面值型別來表示想要 key in 的索引
```
{% endcode %}

**keying in** 的語法是刻意設計的跟一般 JavaScript 物件中的欄位查找類似，如同在一個物件中查找一個值的時候那樣，**也可以在一個形狀中查找一個型別**。**注意：key in 的語法必須使用中括號的記法(bracket notation)，而非點號記法(dot notation)**。



### keyof 運算子

使用 **`keyof`** 運算子，來取得一個物件的**所有鍵值**作為**字串字面值**型別的一個**聯集**，以前面的 APIResponse 為例：

{% code lineNumbers="true" %}
```typescript
type APIResponse = {
  user: {
    userId: string
    friendList: {
      count: number
      friends: {
        firstName: string
        lastName: string
      }[]
    }
  }
}

type ResponseKeys = keyof APIResponse // "user"
type UserKeys = keyof APIResponse["user"] // "userId" | "friendList"
type FriendListKeys = keyof APIResponse["user"]["friendList"] // "count" | "friends"
```
{% endcode %}



結合 **`keying-in`** 和 **`keyof 運算子`**，可以實作一個具有型別安全性的 getter 函式，以查找一個物件中位於給定鍵值的值：

{% code lineNumbers="true" %}
```typescript
function get<O extends object, K extends keyof O>(o: O, k: K): O[K] {
  return o[k]
}

type ActivityLog = {
  lastEvent: Date
  events: {
    id: string
    timestamp: Date
    type: "Read" | "Write"
  }
}

let activityLog: ActivityLog = ....
let lastEvent = get(activityLog, "lastEvent") // 在編譯時期，驗證 lastEvent 的型別為 Date
```
{% endcode %}



### Record 型別

TypeScript 內建的 **Record 型別**，是將一個物件描述為**從某樣東西到某樣東西的一種映射(map)的方式**。

例如以下的例子，使用 **Record 型別**來建置一個 map，也就是 **Weekday 的每個星期幾，要 映射(map) 到 Day 的某個星期幾**：

{% code lineNumbers="true" %}
```typescript
type Weekday = "Mon" | "Tue" | "Wed" | "Thu" | "Fri"
type Day = Weekday | "Sat" | "Sun"

let nextDay: Record<Weekday, Day> = {
  Mon: "Tue",
  Tue: "Wed",
  Wed: "Thu",
  Thu: "Fri",
  Fri: "Sat"
}
```
{% endcode %}



### 映射型別

**映射型別(mapped types)**：讓我們用映射型別表達 nextDay 是**每個 Weekday 都有一個鍵值的一種物件，其值為一個 Day**：

{% code lineNumbers="true" %}
```typescript
type Weekday = "Mon" | "Tue" | "Wed" | "Thu" | "Fri"
type Day = Weekday | "Sat" | "Sun"

let nextDay: {[K in Weekday]: Day} = {
  Mon: "Tue",
  Tue: "Wed",
  Wed: "Thu",
  Thu: "Fri",
  Fri: "Sat"
}
```
{% endcode %}

映射型別有自己的特殊語法，而每個物件最多能有一個映射型別：

{% code lineNumbers="true" %}
```typescript
type MyMappedType = {
  [Key in UnionType]: ValueType
}
```
{% endcode %}

看一下可以用映射型別做到的一些事：

{% code lineNumbers="true" %}
```typescript
type Account = {
  id: number
  isEmployee: boolean
  notes: string[]
}

// 映射 Account 來創建一個新的物件型別 OptionalAccount，過程中把每個欄位標示為選擇性的
type OptionalAccount = {
  [K in keyof Account]?: Account[K]
}

// 映射 Account 來創建一個新的物件型別 NullableAccount，過程中把 null 加入作為每個欄位的一個可能的值
type NullableAccount = {
  [K in keyof Account]: Account[K] | null
}

// 映射 Account 來創建一個新的物件型別 ReadonlyAccount，並把它的每個欄位都變成唯讀
type ReadOnlyAccount = {
  readonly [K in keyof Account]: Account[K]
}

// 我們可以把欄位標示為選擇性(?)或 readonly 的，而我們也可以藉由減號( - )運算子來取消標示。
// 這裡的減號，僅能用於映射型別，可以用來取消 問號(?) 或 readonly 。
// 下例的結果，等同於 Account，也就是創建了一個新的物件型別 Account2，它等同於 Account 型別，
// 方法是映射 ReadonlyAccount 並以減號運算子移除 readonly 修飾詞
type Account2 = {
  -readonly [K in keyof ReadOnlyAccount]: Account[K]
}

// 創建了一個新的物件型別 Account3，它等同於原本的 Account 型別，
// 方法是映射 OptionalAccount 並以減號( - )運算子移除選擇性運算子( ? )。
type Account3 = {
  [K in keyof OptionalAccount]-?: Account[K]
}
```
{% endcode %}

內建的映射型別：

* `Record<Keys, Values>`：鍵值的型別為 Keys，值為 Values。
* `Partial<Object>`：將 Object 中的每個欄位標示為選擇性的。
* `Required<Object>`：將 Object 中的每個欄位標示為非選擇性的。
* `Readonly<Object>`：將 Object 中的每個欄位標示成唯讀的。
* `Pick<Object, Keys>`：回傳 Object 的一個子型別，僅帶有給定的 Keys。



### 伴隨物件模式

可用來把**一個型別**和**一個物件**配成對。

{% code lineNumbers="true" %}
```typescript
type Unit = 'EUR' | 'GBP' | 'JPY' | 'USD'

type Currency = { // 型別
  unit: Unit
  value: number
}

let Currency = { // 物件
  from(value: number, unit: Unit): Currency {
    return {
      unit: unit,
      value
    }
  }
}
```
{% endcode %}

可以讓相同的名稱(上例中的 Currency)繫結到一個型別以及一個值。藉由這種伴隨物件模式，利用了命名空間分離的事實，來宣告一個名稱兩次：首先作為一個型別，然後是一個值。

這種模式能讓你把語意上皆屬於單一名稱的**型別**和**值**之資訊包在一起。它也能讓使用者一次將兩者都匯入：

{% code lineNumbers="true" %}
```typescript
import {Currency} from "./Currency"

// 使用 Currency 作為一個型別
let amountDue: Currency = {
  unit: "JPY",
  value: 83733.10
}

// 使用 Currency 作為一個值
let otherAmountDue = Currency.from(330, "EUR")
```
{% endcode %}

所以，若**一個型別**和**一個物件**在語意上是相關的，就使用**伴隨物件模式(companion object pattern)**，讓該物件提供方法在那個型別上作業。



## 進階函式型別



### 改善 Tuple 的型別推論

當在 TypeScript 中宣告一個元組(tuple)，TypeScript 對於那個元組之型別的推論會放寬處理。

它會依據你所給的推論出最廣義的可能型別，忽略你元組的長度、以及哪個位置持有哪個型別。例：

{% code lineNumbers="true" %}
```typescript
let a = [1, true] // (number | boolean)[]
```
{% endcode %}

但有的時候，會想要較為嚴苛的推論，將 a 視為一個長度固定的元組，而非一個陣列。例：

{% code lineNumbers="true" %}
```typescript
function tuple<T extends unknown[]>(...ts: T): T {
  return ts
}
```
{% endcode %}

若程式碼用到很多的元組型別，就可運用這種技巧來邀免型別斷言。



### 使用者定義的型別防護

對於某些要回傳 boolean 的函式，單純指出你的函式回傳一個 boolean 可能並不足夠。舉例來說，寫一個會告訴你傳入給它的是否為 string 的函式：

{% code lineNumbers="true" %}
```typescript
function isString(a: unknown): boolean {
  return typeof a === "string"
}

isString("a") // true
isString([7]) // false
```
{% endcode %}

上例的情況沒什麼問題，但如果用在以下程式，就會有問題：

{% code lineNumbers="true" %}
```typescript
function isString(a: unknown): boolean {
  return typeof a === "string"
}

function parseInt(input: string | number) {
  let formattedInput: string
  if(isString(input)){
    formattedInput = input.toUpperCase() // Property 'toUpperCase' does not exist on type 'string | number'.
                                         // Property 'toUpperCase' does not exist on type 'number'.
  }
}
```
{% endcode %}

主要的問題在於型別細分(type refinement)，它只能細分所在的範疇中的某個變數的型別。若離開了那個範疇，這個細分並無法被帶到所在的任何新範疇。



然而，我們可以改成這樣：「讓 isString 不僅會回傳一個 boolean，而是每當那個 boolean 是 true，我們傳入給 isString 的引數就會是以一個 string 來做回傳」。也就是下方程式中的「**`a is string`**」，這就是所謂的 **`使用者定義的型別防護(user-defined type guard)`**：

{% code lineNumbers="true" %}
```typescript
function isString(a: unknown): a is string {
  return typeof a === "string"
}

function parseInt(input: string | number) {
  let formattedInput: string
  if(isString(input)){
    formattedInput = input.toUpperCase()
  }
}
```
{% endcode %}

型別防護是 TypeScript 內建的一個功能，是讓你以 typeof 和 instanceof 細分型別的東西。不過有的時候，需要自行宣告型別防護的能力，那正是 **`is 運算子`**的用途所在：**當有一個函式會細分它參數的型別並回傳一個 boolean，就能使用一個使用者定義的型別防護來確保你使用該函式的時候，細分效果都會繼續流動下去**。

使用者定義的型別防護僅限單一參數。



## 條件式型別

**條件式型別(conditional types)**是滿特別的一項功能。程式碼看起來像這樣：

{% code lineNumbers="true" %}
```typescript
// 宣告了一個新的條件式型別 IsString，它接受一個泛型 T。
// 這個條件式型別的「條件」部份是 T extends string，
// 也就是「T 是 string 的一個子型別嗎？」
// 如果 T 是 string 的子型別，就解析為型別 true；
// 否則，就解析為型別 false。
type IsString<T> = T extends string ? true : false

type A = IsString<string> // true
type B = IsString<number> // false
```
{% endcode %}



### 分配式條件型別

條件式型別的分配，下表左右兩欄意思是一樣的：

| 左欄                                              | 等同於左欄的寫法                                                                                |
| ----------------------------------------------- | --------------------------------------------------------------------------------------- |
| string extends T ? A : B                        | string extends T ? A : B                                                                |
| (string \| number) extends T ? A : B            | (string extends T ? A : B) \| (number extends T ? A : B)                                |
| (string \| number \| boolean) extends T ? A : B | (string extends T ? A : B) \| (number extends T ? A : B) \| (boolean extends T ? A : B) |



以下，接受型別為 T 的某個變數，並將之拉升為型別 T\[] 的一個陣列：

{% code lineNumbers="true" %}
```typescript
type ToArray<T> = T[]
type A = ToArray<number> // number[]
type B = ToArray<number | string> // (number | string)[]
```
{% endcode %}

現在，加入一個條件式型別，會變成如何呢？

{% code lineNumbers="true" %}
```typescript
type ToArray2<T> = T extends unknown ? T[] : T[]
type A = ToArray2<number> // number[]
type B = ToArray2<number | string> // number[] | string[]
```
{% endcode %}

當有使用一個條件式型別，TypeScript 會將聯集型別分配到條件式的分支。這就像拿那個條件式來對聯集中的每個元素集行映射(就是**分配**)。



這樣有什麼重要呢？可以安全地表達一些常見的運算。例：

讓我們建置 **`Without<T, U>`**，它會計算出屬於 T 但不屬於 U 的型別：

{% code lineNumbers="true" %}
```typescript
type Without<T, U> = T extends U ? never : T
```
{% endcode %}

可能會像樣使用 Without：

{% code lineNumbers="true" %}
```typescript
// 以下 A 和 B 的估算結果都是一樣的，即 never | number | string，
// 簡化為 number | string
type A = Without<boolean | number | string, boolean>
type B = (boolean extends boolean ? never : boolean) | (number extends boolean ? never : number) | (string extends boolean ? never : string)
```
{% endcode %}



### infer 關鍵字

宣告**泛用型別參數**的一種方式：使用角括號(**`<>`**)。

然而，**條件式型別**有自己的語法用以在**行內宣告泛用型別**，即使用 **infer 關鍵字**。

下例，宣告一個條件式型別 ElementType，它會取得一個陣列之元素的型別：

{% code lineNumbers="true" %}
```typescript
type ElementType<T> = T extends unknown[] ? T[number] : T
type A = ElementType<number[]> // number
```
{% endcode %}

現在，使用 infer 改寫它：

{% code lineNumbers="true" %}
```typescript
type ElementType2<T> = T extends (infer U)[] ? U : T
type B = ElementType2<number[]> // number
```
{% endcode %}

在上例中，ElementType 等同於 ElementType2。留意 infer 子句如何宣告一個**新的型別變數 U**：TypeScript 會從情境(context)推論出 U 的型別，依據你傳入 ElementType2 的是什麼 T。



### 內建的條件式型別

條件式型別能在型別層次表達一些非常強大的運算。這就是 TypeScript 原本就內建幾個可全域取用的條件式型別之原因所在：



**`Exclude<T, U>`**：如同前面的 Without，計算屬於 T，但不屬於 U 的那些型別：

{% code lineNumbers="true" %}
```typescript
type A = number | string
type B = string
type C = Exclude<A, B> // number
```
{% endcode %}

**`Extract<T, U>`**：計算 T 中可以指定給 U 的型別：

{% code lineNumbers="true" %}
```typescript
type A = number | string
type B = string
type C = Extract<A, B> // string
```
{% endcode %}

**`NonNullable<T>`**：計算出排除了 null 和 undefined 的 T：

{% code lineNumbers="true" %}
```typescript
type A = {a?: number | null}
type B = NonNullable<A["a"]> // number
```
{% endcode %}

**`ReturnType<F>`**：計算一個函式的回傳型別(注意：若用於泛型和重載函式時，可能會不如預期)：

{% code lineNumbers="true" %}
```typescript
type F = (a: number) => string
type R = ReturnType<F> // string
```
{% endcode %}

**`InstanceType<T>`**：計算一個類別建構器的實體型別：

{% code lineNumbers="true" %}
```typescript
type A = {new(): B}
type B = {b: number}
type I = InstanceType<A> // {b: number}
```
{% endcode %}



## 逃生管道

有的時候，沒有時間完美地定型某樣東西，只是單純的希望 TypeScript 相信你所做的是安全的。所以 TypeScript 提供了幾個**逃生管道(escape hatches)**，用在我們只是想要做某些事情但沒時間向 TypeScript 證明那是安全的時候。

注：應盡可能少用，因為若是發現你必須仰賴它們，就代表你可能有事情做錯了。



### 型別斷言

假設有一個型別 B，而且 A 是 B 的子型別，然後 B 是 C 的子型別，那麼就能斷言 B 是一個 A 或 C。

值得留意的是，只能斷言一個型別是自己的超型別或子型別，也就是說，不能斷言一個 number 是一個 string，因為那些型別不相關。

TypeScript 提供兩種方法來做型別斷言：

{% code lineNumbers="true" %}
```typescript
function formatInput(input: string){

}

function getUserInput(): string | number{
  return "aaa"
}

let input = getUserInput()

// 斷言 input 是一個 string
formatInput(input as string) // 說明 1
// 等同於下面這行
// formatInput(<string>input) // 說明 2
```
{% endcode %}

說明：

1. 使用一個**`型別斷言(as)`**來告訴 TypeScript 說，input 是一個 string，而不是那些型別要我們相信的一個 string | number 。例，如果想要快速地測試 formatInput 函式，而很確定 getUserInput 會回一個 string，可能就會想這樣做。
2. 型別斷言的正統語法是使用角括號，兩種寫法都可。
3. 建議優先使用 as 語法，因為角括號的語法可能會與 TSX 語法有衝突。



### Nonnull 斷言

對於可 null 型別的特殊情況，也就是 **`T | null`** 或 **`T | null | undefined`** 的那種型別，TypeScript 有語法用以斷言一個該種型別的值是一個 T 而非 null 或 undefined。



例：

{% code lineNumbers="true" %}
```typescript
type Dialog = {
  id?: string
}

function closeDialog(dialog: Dialog){
  if(!dialog.id){
    return
  }
  setTimeout(() => {
    removeFromDOM(dialog, document.getElementById(dialog.id!)!) // 用到「非 null 斷言運算子驚嘆號」
  }, 0)
}

function removeFromDOM(dialog: Dialog, element: Element){
  element.parentNode!.removeChild(element) // 用到「非 null 斷言運算子驚嘆號」
  delete dialog.id
}
```
{% endcode %}

上述程式碼中，**`非 null 斷言運算子(即驚嘆號)`**，會告訴 TypeScript 我們確定 `dialog.id` 、`document.getElementById` 呼叫的結果，以及 `element.parentNode` 都有定義。

當一個 **`nonnull 斷言`**跟在可能是 `null` 或 `undefined` 的一個型別之後，TypeScript 會假設該型別有定義，從 **`T | null | undefined`** 變成了一個 **`T`**，**`number | string | null`** 變成了一個 **`number | string`**，以此類推。

如果發現自己大量使用 nonnull 斷言，這通常就是應該重構程式碼的跡象了。例如上例可改成：

{% code lineNumbers="true" %}
```typescript
type VisibleDialog = {id?: string}
type DestroyedDialog = {}
type Dialog = VisibleDialog | DestroyedDialog

function closeDialog(dialog: Dialog){
  if(!("id" in dialog)){
    return
  }
  setTimeout(() => {
    removeFromDOM(dialog, document.getElementById(dialog.id)!)
  }, 0)
}

function removeFromDOM(dialog: VisibleDialog, element: Element){
  element.parentNode!.removeChild(element)
  delete dialog.id
}
```
{% endcode %}



### 確切指定斷言

確切指定斷言(**`definite assignment checks`**，作為提醒，確切指定檢查是 TypeScript 用來確保你看到某個變數的時候，該變數已經被指定了一個值的一種方式)。

例：

{% code lineNumbers="true" %}
```typescript
let userId: string
userId.toUpperCase() // 會報錯，因為要先給定資料，才能執行 userId.toUpperCase()
```
{% endcode %}



但如果我們的程式碼看起來像這樣：

{% code lineNumbers="true" %}
```typescript
let userId: string
fetchUser()

userId.toUpperCase() // 一樣會報錯，跟上面的錯誤訊息一樣

function fetchUser(){
  userId = globalCache.get("userId")
}
```
{% endcode %}

在對 fetchUser 的呼叫之後，userId 保證會有定義。但 TypeScript 無法靜態地偵測到那個，所以它依然會報錯。我們可以使用一個「**確定指定 斷言**」來告知 TypeScript 說 userId 在我們讀取它時，肯定已被指定(就是以下的驚嘆號)：

{% code lineNumbers="true" %}
```typescript
let userId!: string // 這裡加上驚嘆號，就是「確切指定 斷言」
fetchUser()

userId.toUpperCase() // 這裡就不會報錯

function fetchUser(){
  userId = globalCache.get("userId")
}
```
{% endcode %}



