# 函式

## 函式的宣告

一般的寫法：

{% code lineNumbers="true" %}
```typescript
function add(a: number, b: number) {
  return a + b
}
```
{% endcode %}

至於回傳的型別，TypeScript 也會自動推論，如果想自己加上的話，如下：

{% code lineNumbers="true" %}
```typescript
function add(a: number, b: number): number {
  return a + b
}
```
{% endcode %}



以下列出常見的幾種函式宣告的寫法：

{% code lineNumbers="true" %}
```typescript
// 具名函式
function greet(name: string) {
  return "hello " + name
}

// 函式運算式
let greet2 = function(name: string) {
  return "hello " + name
}

// 箭頭函式運算式
let greet3 = (name: string) => {
  return "hello " + name
}

// 箭頭函式運算式：簡寫
let greet4 = (name: string) => "hello " + name
```
{% endcode %}

以上都遵循這種規則：**強制性的參數型別注釋，以及選擇性的回傳型別注釋**。



術語：

* **參數(parameter)**：是函式執行所需的一個資料片段，被宣告四函式宣告的一部份。也被稱為**形式參數(formal parameter)**。
* **引數(argument)**：是執行一個函式時傳入給它的一個資料片段。也 稱為**實際參數(actual parameter)**。



### 選擇性參數和預設參數

可以使用 **`問號(?)`** 來將參數標示為**選擇性**的。 宣告函式時，若有參數，**必要的參數必須先出現，後面才接著選擇性的參數**：

{% code lineNumbers="true" %}
```typescript
function log(message: string, userId?: string) {
  let time = new Date().toLocaleTimeString()
  console.log(time, message, userId || "Not singed in")
}
log("Page loaded")
log("User signed in", "da763be")
```
{% endcode %}



**預設參數**的例子：

{% code lineNumbers="true" %}
```typescript
function log(message: string, userId = "Not signed in") {
  let time = new Date().toISOString()
  console.log(time, message, userId)
}
log("User clicked on a button", "da763be")
log("User signed out")
```
{% endcode %}

提供 userId 一個預設值的時候，也移除了它的選擇性注釋，也就是問號。我們也不再需要為它定型。因為 TypeScript 會自行推論，會從預設值推論出參數的型別。



當然，也可以為預設參數加上明確的型別注釋：

{% code lineNumbers="true" %}
```typescript
type Context = {
  appId?: string
  userId?: string
}

function log(message: string, context: Context = {}) {
  let time = new Date().toISOString()
  console.log(time, message, context.userId)
}

log("User clicked on a button", {userId: "da763be"})
log("User signed out")
```
{% endcode %}



### 其餘參數(Rest Paramters)

先一個加總的例子：

{% code lineNumbers="true" %}
```typescript
function sum(numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0)
}
let all_sum = sum([1, 2, 3])

console.log(all_sum) // 6
```
{% endcode %}

改成使用**其餘參數**，也就是三個點的符號：

{% code lineNumbers="true" %}
```typescript
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0)
}
let all_sum = sum(1, 2, 3)

console.log(all_sum) // 6
```
{% endcode %}



### 認識 call、apply 與 bind

例：

{% code lineNumbers="true" %}
```typescript
function add(a: number, b: number): number {
  return a + b
}

let result1 = add(10, 20)

let result2 = add.apply(null, [10, 20])
let result3 = add.call(null, 10, 20)
let result4 = add.bind(null, 10, 20)()

console.log(result1) // 30
console.log(result2) // 30
console.log(result3) // 30
console.log(result4) // 30
```
{% endcode %}

說明：

* apply 會把一個值綁定到你函式中的 this (此例為 null)，並把它的第二個引數分聚給函式中的參數。
* call 值的事與 apply 一樣，，只是引數的部份是依序套用到函式參數。
* bind 也類似，它會把一個 this 引數和一個引數串列綁定到函式，差別在於，bind 不會執行函式，而是回傳一個新的函式，隨後能以 **`()`**、**`.call`**、**`.apply`** 來執行。



### 為 this 定型

例：

{% code lineNumbers="true" %}
```typescript
function fancyDate(this: Date) {
  return `${this.getDate()}/${this.getMonth()}/${this.getFullYear()}`
}

console.log(fancyDate.call(new Date()))
```
{% endcode %}



### Generators: 產生器函式

**產生器函式(generator function，簡稱 generator)**，是**產生一組值**的一種有利方式。

費氏數列為列：

{% code lineNumbers="true" %}
```typescript
function* createFibonacciGenerator() {
  let a = 0;
  let b = 1;
  while(true){
    yield a;
    [a, b] = [b, a + b];
  }
}
let fibonacciGenerator = createFibonacciGenerator(); // 得到一個迭代器
console.log(fibonacciGenerator.next()); // {value: 0, done: false}
console.log(fibonacciGenerator.next()); // {value: 1, done: false}
console.log(fibonacciGenerator.next()); // {value: 1, done: false}
console.log(fibonacciGenerator.next()); // {value: 2, done: false}
```
{% endcode %}

例：

{% code lineNumbers="true" %}
```typescript
function* createNumbers(): IterableIterator<number> {
  let n = 0
  while(1){
    yield n++
  }
}

let numbers = createNumbers()
console.log(numbers.next()) // { value: 0, done: false }
console.log(numbers.next()) // { value: 1, done: false }
console.log(numbers.next()) // { value: 2, done: false }
```
{% endcode %}



### Iterators: 迭代器物件

**迭代器(Iterators)** 是產生器的相反：產生器是產生一連串值的一種方式，而迭代器則是消耗那些值的一種方式。

* **Iterable(可迭代的)**：含有一個叫做 **`Symbol.iterator`** 的特性的任何物件，其值是會回傳一個迭代器的一個函式。
* **Iterator(迭代器)**：定義一個叫做 **next** 的方法的任何物件，它會回傳帶有特性 **value** 和 **done** 的一個物件。

例：

{% code lineNumbers="true" %}
```typescript
let numbers = {
  *[Symbol.iterator](){
    for(let n = 1; n <= 10; n++){
      yield n
    }
  }
}


for(let a of numbers){
  console.log(a);
}

//let allNumbers = [...numbers]
//console.log(allNumbers);

//let [one, two, ...rest] = numbers;
//console.log(one);
//console.log(two);
//console.log(rest);
```
{% endcode %}



### 型別特徵式

該如何表示函式本身的完整型別。

例：

{% code lineNumbers="true" %}
```typescript
function sum(a: number, b: number): number {
  return a + b
}
```
{% endcode %}

該如何表示 sum 函式的完整型別呢： **`(a: number, b: number) => number`** 。這就是用於函式型別的 TypeScript 語法，或稱為**`呼叫特徵式(call signature，也稱為作型別特徵式 type signature)`**。



函式呼叫特徵式只含有**型別層次(type-level)**的程式碼，也就是說，僅有型別沒有值。



型別特徵式與實作的關係：

{% code lineNumbers="true" %}
```typescript
type Log = (message: string, userId?: string) => void

let log: Log = (message, userId = "Not signed in") => {
  let time = new Date().toISOString()
  console.log(time, message, userId)
}
```
{% endcode %}



### 情境式定型(Contextual Typing)

上個範例，已經將 log 的型別宣告為 Log，所以可推論出 message 的型別必須是 string，這就是情境式定型。

另個例子：

{% code lineNumbers="true" %}
```typescript
function times(f: (index: number) => void, n: number) {
  for(let i = 0; i < n; i++){
    f(i)
  }
}

times(n => console.log(n), 4) // 這裡的 n 會被推論出其型別為 number
```
{% endcode %}



### 重載的函式型別

簡寫的型別特徵式：

{% code lineNumbers="true" %}
```typescript
type Log = (message: string, userId?: string) => void
```
{% endcode %}

如果是完整的寫法，如下：

{% code lineNumbers="true" %}
```typescript
type Log = {
  (message: string, userId?: string): void
}
```
{% endcode %}

兩者是一樣的意思，大部份情況使用簡寫的語法。



**重載的函式：具有多個型別特徵式的函式。**



例：

{% code lineNumbers="true" %}
```typescript
type Reserve = {
  (from: Date, to: Date, destination: string): void
  (from: Date, destination: string): void
}

let reserve: Reserve = (from: Date, toOrDestination: Date | string, destination?: string) => {
  if(toOrDestination instanceof Date && destination !== undefined){
    // 預訂來回的行程
  }else if(typeof toOrDestination === "string"){
    // 預訂單程行程
  }
}
```
{% endcode %}



## 多型(Polymorphism)

**泛用型別參數(generic type parameter)**：一種佔位置的型別(placeholder type)，用來在多個地方強制實行型別層次的限制。也被稱作**多型的型別參數(polymorphic type parameter)**。

**泛用型別參數**，人們經常將之簡稱為「**泛用型別(generic type)**」或「**泛型(generic)**」。



例：

{% code lineNumbers="true" %}
```typescript
type Filter = {
  <T>(array: T[], f: (item: T) => boolean): T[]
}

let filter: Filter = (array, f) => {
  let result = []
  for(let i = 0; i < array.length; i++){
    let item = array[i]
    if( f(item) ){
      result.push(item)
    }
  }
  return result
}

// T 被綁定至 number
console.log( filter([1, 2, 3], _ => _ > 2) )

// T 被綁定至 string
console.log( filter(["a", "b"], _ => _ !== "b") )

// T 被綁定至 {firstName: string}
let names = [
  {firstName: "beth"},
  {firstName: "caitlyn"},
  {firstName: "xin"}
]
console.log( filter(names, _ => _.firstName.startsWith("b")) )
```
{% endcode %}

意思是：「filter 這個函式使用一個泛型參數 T，我們事先不知道這個型別會是什麼，所以 TypeScript 如果你可以在每次我們呼叫 filter 函式的時候，推論出該型別為何，就太好了」。TypeScript 會為我們傳入給 array 的型別推論出 T。一旦 TypeScript 為給定的一個 filter 呼叫，推論出 T 是什麼，它就會將看到的每個 T 替換為該型別。T 就像一個佔位型別，等待型別檢查器依據情境進行填入。它將 Filter 的型別參數化了，這也是我們稱之為**泛用型別參數**的原因。

**`<T>`**這個符號的意思是：**宣告泛用型別參數的方式，然後設定其名稱為 T，在角括號之間，可以宣告多個，以逗號區隔的泛用型別參數**。T 只是一個型別名稱，可改用任何其他的名稱，例：A、Zebra 等，但通常使用大寫的單字母名稱，然後從字母 T 開始，接著 U、V、W 等依此類推。

**泛型**能允許更廣義的方式描述你函式所做之事的一種強大工具。思考泛型的方法是將之想成是**限制(constraint)**，使用一個泛型 T 會在 T 出現的每個地方將你繫結到 T 的任何型別限制為相同型別。



### 泛型何時繫結(綁定)

宣告一個泛用型別的位置會決定該型別的範疇，也決定了 TypeScript 何時會將一個具體型別繫結到你的泛型。

在上個範例中，因為我們宣告 **`<T>`** 作為呼叫特徵式的一部分( 也就是緊接在特徵式的左括號之前 )，TypeScript 會在我們實際呼叫型別為 Filter 的一個函式時，將一個具體型別繫結到 T。



如果是將 T 的範疇限定為型別別名 Filter，TypeScript 就會要求我們使用 Filter 的時候得明確繫結一個型別：

{% code lineNumbers="true" %}
```typescript
type Filter<T> = { // <T> 寫在型別別名
  (array: T[], f: (item: T) => boolean): T[]
}

// 使用 Filter 的時候，馬上將 number 綁定至 T
let filter: Filter<number> = (array, f) => {
  let result = []
  for(let i = 0; i < array.length; i++){
    let item = array[i]
    if( f(item) ){
      result.push(item)
    }
  }
  return result
}

// T 被綁定至 number
console.log( filter([1, 2, 3], _ => _ > 2) )
```
{% endcode %}



### 何處可以宣告泛型

第一種：一個完整的呼叫特徵式，T 的範疇限定為一個個別的特徵式。因為 T 的範疇為單一個特徵式，TypeScript 就會在你呼叫型別為 Filter 的函式時將此特徵式中的 T 繫結到一個具體型別。

{% code lineNumbers="true" %}
```typescript
type Filter = {
  <T>(array: T[], f: (item: T) => boolean): T[]
}
```
{% endcode %}

第二種：一個完整的呼叫特徵式，T 的範疇是所有的特徵式。因為 T 被宣告為 Filter 的型別的一部分，TypeScript 會在你宣告型別為 Filter 的函式時繫結 T。

{% code lineNumbers="true" %}
```typescript
type Filter<T> = {
  (array: T[], f: (item: T) => boolean): T[]
}
```
{% endcode %}

第三種：就跟第一種是一樣的，這裡是簡寫的呼叫特徵式。

{% code lineNumbers="true" %}
```typescript
type Filter = <T>(array: T[], f: (item: T) => boolean): T[]
```
{% endcode %}

第四種：就跟第二種是一樣的，這裡是簡寫的呼叫特徵式。

{% code lineNumbers="true" %}
```typescript
type Filter<T> = (array: T[], f: (item: T) => boolean): T[]
```
{% endcode %}

第五種：一個具名的函式呼叫特徵式，T 的範疇是該特徵式。TypeScript 會在你呼叫 filter 的時候繫結一個具體型別到 T，而對 filter 的每個呼叫都會得到自己的 T 繫結。

{% code lineNumbers="true" %}
```typescript
function filter<T>(array: T[], f: (item: T) => boolean): T[] {
  // ...
}
```
{% endcode %}



### 泛型推論

map 範例，T 會是 string，U 會是 boolean：

{% code lineNumbers="true" %}
```typescript
function map<T, U>(array: T[], f: (item: T) => U): U[] {
  let result = []
  for (let i = 0; i < array.length; i++){
    result[i] = f(array[i])
  }
  return result
}

let r = map(["a", "b", "c"], _ => _ === "a")
console.log(r)
```
{% endcode %}



若使用明確注釋，則變這樣：

{% code lineNumbers="true" %}
```typescript
function map<T, U>(array: T[], f: (item: T) => U): U[] {
  let result = []
  for (let i = 0; i < array.length; i++){
    result[i] = f(array[i])
  }
  return result
}

// 以下這行，明確的注釋，所以 T 是 string；U 是 boolean
let r = map<string, boolean>(["a", "b", "c"], _ => _ === "a")
console.log(r)
```
{% endcode %}



### 泛型別名

定義一個 MyEvent 型別用來描述一個 DOM 事件：

{% code lineNumbers="true" %}
```typescript
type MyEvent<T> = {
  target: T
  type: string
}
```
{% endcode %}

留意以上，是在一個型別別名中宣告**泛用型別**的唯一有效之處：也就是上面的 **`<T> 位置`**。

例如，可以像這樣來描述一個按鈕事件：

{% code lineNumbers="true" %}
```typescript
type ButtonEvent = MyEvent<HTMLButtonElement>
```
{% endcode %}



也可以使用 MyEvent 來建置另一個型別，例如

{% code lineNumbers="true" %}
```typescript
type TimedEvent<T> = {
  event: MyEvent<T>
  from: Date
  to: Date
}
```
{% endcode %}



### 有界的多型

假設二元樹：

* 一般的 TreeNode。
* LeafNode：它們是沒有子節點的 TreeNode。
* InnerNode：它們是有節點的 TreeNode。

例：留意以下的 **`T extends TreeNode`**。

{% code lineNumbers="true" %}
```typescript
type TreeNode = {
  value: string
}

type LeafNode = TreeNode & { // & 符號是交集
  isLeaf: true
}

type InnerNode = TreeNode & { // & 符號是交集
  children: [TreeNode] | [TreeNode, TreeNode]
}

function mapNode<T extends TreeNode>(node: T, f: (value: string) => string): T {
  return {
    ...node,
    value: f(node.value)
  }
}
```
{% endcode %}

藉由指出 **`T extends TreeNode`**，我們得以保留輸入節點的特定型別(TreeNode、LeafNode、InnerNode)。



### 泛型預設值

例：

{% code lineNumbers="true" %}
```typescript
type MyEvent<T extends HTMLElement = HTMLElement> = {
  target: T
  type: string
}
```
{% endcode %}

另外，具有預設值的泛用型別必須出現在沒有預設值的泛用型別之後，例：

{% code lineNumbers="true" %}
```typescript
type MyEvent2<T extends string, U extends HTMLElement = HTMLElement> = {
  target: U
  type: T
}
```
{% endcode %}



