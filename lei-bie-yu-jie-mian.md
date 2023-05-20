# 類別與介面

## 類別與繼承

以西洋棋為例：

{% code lineNumbers="true" %}
```typescript
type Colors = "Black" | "White"
type Files = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H"
type Ranks = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8

// 西洋棋局
class Game {
  private pieces = Game.makePieces()

  private static makePieces(){
    return [
      new King("White", "E", 1),
      new King("Black", "E", 8)
    ]
  }
}

// 一個棋子的一組座標
class Position {
  constructor(private file: Files, private rank: Ranks){

  }
  
  // 計算兩個棋子之間的距離
  distanceFrom(position: Position){
    return {
      rank: Math.abs(position.rank - this.rank),
      file: Math.abs(position.file.charCodeAt(0) - this.file.charCodeAt(0))
    }
  }
}

// 一個棋子
abstract class Piece {
  protected position: Position
  constructor(private readonly color: Colors, file: Files, rank: Ranks){
    this.position = new Position(file, rank)
  }
  
  // 沒有放存取修飾詞，所以預設是 public
  moveTo(position: Position){
    this.position = position
  }

  abstract canMoveTo(position: Position): boolean
}


// 棋子有六種
class King extends Piece {
  canMoveTo(position: Position){
    let distance = this.position.distanceFrom(position)
    return distance.rank < 2 && distance.file < 2
  }
}

//class Queen extends Piece {}
//class Bishop extends Piece {}
//class Knight extends Piece {}
//class Rook extends Piece {}
//class Pawn extends Piece {}
```
{% endcode %}



TypeScript 支援三種存取修飾詞，用於類別上的特性與方法：

* **public**：從任何地方都能存取。這是預設的存取層級。
* **protected**：可從此類別及其子類別的實體存取。
* **private**：僅可從此類別的實體進行存取。

關於 Piect 抽象類別：

* 告訴它的子類別說，它們必須實作一個叫做 canMoveTo 的方法，而且它必須與給定的特徵式相容。若有類別擴充了 Piece 但忘記實作抽象(abstract)的 canMoveTo 方法，那就會報錯：也就是實作一個博象類別的時候，你同時也必須實作它的抽象方法。
* 為 moveTo 準備一個預設的實作(其子類別想要的話能夠加以覆寫)。我們沒有在 moveTo 之上放置存取修飾詞，所以預設是 public，代表任何其他的程式碼都能讀取及寫入它。



### super

如果你的子類別覆寫了定義在其父類別之上的某個方法，子實體就能發出一個 super 呼叫來呼叫其父類別版本的該方法(例如： super.take)。super 呼叫有兩種：

* **方法呼叫**，像是 super.take。
* **建構器呼叫**，它有著特殊形式 **`super()`**，而且只能從建構器函式進行呼叫。



### 使用 this 作為一種回傳型別

舉例來說，模擬 ES6 Set 資料結構，支援兩種運算：新增一個數字到集合(set)，以及檢查一個給定的數字是否在集合中。會像這樣使用：

{% code lineNumbers="true" %}
```javascript
let set = new Set();
set.add(1).add(2).add(3);
set.has(2) // true
set.has(4) // false
```
{% endcode %}



定義 Set 類別，從 has 方法開始：

{% code lineNumbers="true" %}
```typescript
class Set {
  has(value: number): boolean {
    // ...
  }
}
```
{% endcode %}

那麼 add 呢？呼叫 add 的時候，會回傳 Set 的一個實體，因此這樣寫：

{% code lineNumbers="true" %}
```typescript
class Set {
  has(value: number): boolean {
    // ...
  }
  add(value: number): Set {
    // ...
  }
}
```
{% endcode %}

目前這樣都是ok的。但如果有另一個 MutableSet 類別想要繼承 Set 類別，就會這樣寫(假設有一個 delete 方法)：

{% code lineNumbers="true" %}
```typescript
class MutableSet extends Set {
  delete(value: number): boolean {
    // ...
  }
}
```
{% endcode %}

當然，Set 的 add 方法仍然會回傳一個 Set，對於我們子類別(MutableSet)中的 add 方法，必須以回傳 MutableSet 進行覆寫：

{% code lineNumbers="true" %}
```typescript
class MutableSet extends Set {
  delete(value: number): boolean {
    // ...
  }
  add(value: number): MutableSet {
    // ...
  }
}
```
{% endcode %}

處理會擴充其他類別的類別時，這可能會變得有點繁瑣，必須為會回傳 this 的每個方法覆寫特徵式。然而，可以改成使用回傳 **this**：

{% code lineNumbers="true" %}
```typescript
class Set {
  has(value: number): boolean {
    // ...
  }
  
  // 這裡的回傳改成 this
  add(value: number): this {
    // ...
  }
}

// 就可以移除 add 的覆寫，
// 因為 this 在 Set 中指向一個 Set 實體，MutableSet 中的 this 會指向一個 MutableSet 實體
class MutableSet extends Set {
  delete(value: number): boolean {
    // ...
  }
}
```
{% endcode %}



## 介面(interface)

使用類別的時候，常會發現是透過介面(interface)來使用它們。就像型別別名(type aliases)，介面是為一個型別別名，讓你不用在行內(inline)定義它的一種方式。

型別別名和介面幾乎可說是同一種東西的兩種語法，只有幾個微小的差異存在。

先談談共通之處，例：

{% code lineNumbers="true" %}
```typescript
type Sushi = {
  calories: number
  salty: boolean
  tasty: boolean
}

// 上面的這個 Sushi 型別別名，很容易就可以直接改成介面，如下：

interface Sushi {
  calories: number
  salty: boolean
  tasty: boolean
}
```
{% endcode %}

事實上，它們是完全相同的。



再看另一種情況，除了上面的 Sushi 之外，另外建立一個 Cake 型別別名：

{% code lineNumbers="true" %}
```typescript
type Cake = {
  calories: number
  sweet: boolean
  tasty: boolean
}
```
{% endcode %}

把 Food 抽出來自成其型別，所以全部變這樣：

{% code lineNumbers="true" %}
```typescript
type Food = {
  calories: number
  tasty: boolean
}

type Sushi = Food & {
  salty: boolean
}

type Cake = Food & {
  sweet: boolean
}
```
{% endcode %}

然後，幾乎等效的，也能用介面這樣做：

{% code lineNumbers="true" %}
```typescript
interface Food {
  calories: number
  tasty: boolean
}

interface Sushi extends Food {
  salty: boolean
}

interface Cake extends Food {
  sweet: boolean
}
```
{% endcode %}



那，型別(type)和介面(interface)的差異是什麼？有三種：

第一個是，型別別名比較通用，例如：以下的寫法無法改成介面的寫法：

{% code lineNumbers="true" %}
```typescript
type A = number
type B = A | string
```
{% endcode %}



第二個是關於 extends 的部份：

{% code lineNumbers="true" %}
```typescript
interface A {
  good(x: number): string
  bad(x: number): string
}

interface B extends A {
  good(x: string | number): string
  bad(x: string): string // 這個會報錯
}
```
{% endcode %}



第三個差異是，位在相同範鑄中名稱相同的多個**介面(interface)**會自動被合併；而同一個範疇中，同名的多個**型別別名(type aliases)**則會擲出一個編譯期錯誤。**介面(interface)合併**的這個部份，稱作**宣告合併(declaration merging)**。



### 宣告合併

**宣告合併(declaration merging)** 是 TypeScript 自動結合具有相同名稱的多個宣告的方式。

舉例來說，如果宣告了兩個名稱完全相同的 User 介面，那麼 TypeScript 就會自動把它們結合成單一個介面：

{% code lineNumbers="true" %}
```typescript
// User 具有單一個欄位 name
interface User {
  name: string
}

// User 現在有兩個欄位，name 和 age
interface User {
  age: number
}

let a: User = {
  name: "Ashley",
  age: 30
}
```
{% endcode %}



以下是會報錯，無法合併：

{% code lineNumbers="true" %}
```typescript
// Error: User 的所有宣告都必須有完全相同的型別參數
interface User<Age extends number> {
  age: Age
}

interface User<Age extends string> {
  age: Age
}
```
{% endcode %}



### 實作(implements)

宣告一個類別時，可以使用 **`implements`** 關鍵字來指出該類別符合某個特定的介面。例如以下的例子，Cat 類別必須實作 Animal 宣告的每個方法。

{% code lineNumbers="true" %}
```typescript
interface Animal {
  eat(food: string): void
  sleep(hours: number): void
}

// Cat 類別實作 Animal 介面
class Cat implements Animal {
  eat(food: string) {
    console.log(`I eat ${food}`)
  }
  sleep(hours: number) {
    console.log(`I sleep ${hours} hours`)
  }
}
```
{% endcode %}

介面可以宣告實體特性(屬性)，但它們不能宣告可見性修飾詞(private、protected、public)，而且它們不能使用 static 關鍵字，可以把實體特性標示為 **readonly**。例：

{% code lineNumbers="true" %}
```typescript
interface Animal {
  readonly name: string
  eat(food: string): void
  sleep(hours: number): void
}
```
{% endcode %}



類別也可以實作多個介面，例：

{% code lineNumbers="true" %}
```typescript
interface Animal {
  readonly name: string
  eat(food: string): void
  sleep(hours: number): void
}

interface Feline {
  meow(): void
}

// Cat 類別實作 Animal 介面以及 Feline 介面
class Cat implements Animal, Feline {
  name = "Whiskers"
  eat(food: string) {
    console.log(`I eat ${food}`)
  }
  sleep(hours: number) {
    console.log(`I sleep ${hours} hours`)
  }
  meow() {
    console.log("Meow")
  }
}
```
{% endcode %}



### 實作類別 vs. 擴充抽象類別

介面(interface) 是為一種形狀建立模型的方式，例如說，它代表一個物件、陣列、函式、類別或類別實體。interface 並不會發出 JavaScript 程式碼，只存在於編譯時間。

抽象類別(abstract class) 能為之建立模型的對象，就僅限一個類別。抽象類別會發出執行時期的程式碼，就是 JavaScript 類別。抽象類別可以有建構器、可提供預設實作，也可為特性和方法設定存取修飾詞，這些事情，都是介面(interface)做不到的。



## 類別以結構定型

TypeScript 是以**結構**而非名稱來比較類別。以下方程式碼為例，若有一個接受 Zebra 的一個函式，但給它它一個 Poodle，TypeScript 是可接受的，因為 Poodle 與 Zebra 的形狀是相同的。

{% code lineNumbers="true" %}
```typescript
class Zebra {
  trot(){

  }
}

class Poodle {
  trot(){
    
  }
}

function ambleAround(animal: Zebra) {
  animal.trot()
}

let zebra = new Zebra()
let poodle = new Poodle()

ambleAround(zebra)
ambleAround(poodle)
```
{% endcode %}

有一個例外是具有 private 或 protected 欄位的類別：檢查一個形狀是否可以指定給一個類別時，如果該類別有任何 private 或 protected 欄位，而且該形狀不是那個類別或該類別之子類別的一個實體，那麼該形狀就不能指定給那個類別。



## 類別會宣告值，也會宣告型別

值與型別的命名空間(namespace)在 TypeScript 中是分開的。

例：

{% code lineNumbers="true" %}
```typescript
// 值
let a = 1999
function b(){}

// 型別
type a = number
interface b {
  (): void
}

if (a + 1 > 3){ // TypeScript 從情境推論出這裡指的是「值 a」
  console.log("test")
}

let x: a = 3 // TypeScript 從情境推論出這裡指的是「型別 a」
```
{% endcode %}



類別與 enum 是特別的。它們都會在型別命名空間中產生一個型別，並在值命名空間中產生一個值。

例：

{% code lineNumbers="true" %}
```typescript
class C {}

let c: C   // 這裡的 C 指的是 C 類別的實體型別
  = new C  // 這裡的 C 指的是 C 這個值

enum E {F, G}

let e: E  // 這裡的 E 指的是 E enum 的型別
  = E.F   // 這裡的 E 指的是 E 這個值
```
{% endcode %}



## 多型

能讓一個**泛型的範疇(scope)**在你的**整個類別或介面**中使用，或者**限定在一個特定的方法**：

{% code lineNumbers="true" %}
```typescript
class MyMap<K, V> { // 此處的泛用型別，也就是 K 和 V，是 MyMap 上的每個實體方法和實體特性都能取用的
  
  // 留意：在 constructor 中，不能宣告泛用型別。
  constructor(initialKey: K, initialValue: V) {

  }
  
  // 在類別內的任何地方，都可以使用以類別為範疇的泛用型別，也就是 K 和 V。
  get(key: K): V {

  }
  set(key: K, value: V): void {

  }

  // 實體方法能存取類別層級的泛型(即 K 和 V)；也可以宣告它們自己的泛型(即 K1 和 V1)。
  merge<K1, V1>(map: MyMap<K1, V1>): MyMap<K | K1, V | V1> {
    
  }

  // 靜態(static)方法，不能存取類別層級的泛型，就像它們無法存取類別的實體變數那樣。
  // 所以，of 無法存取宣告在最上面宣告的 K 和 V，所以 of 宣告了自己的 K 和 V 泛型。
  static of<K, V>(k: K, v: V): MyMap<K, V> {

  }
}
```
{% endcode %}

也可以把**泛型**繫結到介面：

{% code lineNumbers="true" %}
```typescript
interface MyMap<K, V> {
  get(key: K): V
  set(key: K, value: V): void
}
```
{% endcode %}



