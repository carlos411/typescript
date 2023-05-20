# 處理錯誤

## 回傳 null

例：

{% code lineNumbers="true" %}
```typescript
function ask(){
  return prompt("When is your birthday?")
}

function isValid(date: Date){
  return Object.prototype.toString.call(date) === "[object Date]" && !Number.isNaN(date.getTime())
}

function parse(birthday: string): Date | null { // 還有這裡有 null
  let date = new Date(birthday)
  if(!isValid(date)){
    return null // 這裡回傳 null
  }else{
    return date
  }
}


let date = parse(ask())
if(date){
  console.info("Date is", date.toISOString())
}else{
  console.error("Error parsing date for some reason")
}

```
{% endcode %}



## 擲出例外

當我們擲出一個例外而非回傳 null，如此我們就能處理特定的故障模式，並有失誤相關的一些資料來更輕鬆的除錯。例：

{% code lineNumbers="true" %}
```typescript
function ask(){
  return prompt("When is your birthday?")
}

function isValid(date: Date){
  return Object.prototype.toString.call(date) === "[object Date]" && !Number.isNaN(date.getTime())
}

// 自訂的錯誤型別
class InvalidDateFormatError extends RangeError {}
class DateIsInTheFutureError extends RangeError {}

function parse(birthday: string): Date {
  let date = new Date(birthday)
  if(!isValid(date)){
    // 擲出例外
    throw new InvalidDateFormatError("Enter a date in the form YYYY/MM/DD")
  }
  if(date.getTime() > Date.now()){
    // 擲出例外
    throw new DateIsInTheFutureError("Are you a timelord?")
  }
  return date
}

try {
  let date = parse(ask())
  console.info("Date is", date.toISOString())
} catch(e) {
  if(e instanceof RangeError){
    console.error(e.message)
  }else if(e instanceof DateIsInTheFutureError){
    console.info(e.message)
  }else{
    throw e
  }
  
}

```
{% endcode %}



## 回傳例外

例：

{% code lineNumbers="true" %}
```typescript
function ask(){
  return prompt("When is your birthday?")
}

function isValid(date: Date){
  return Object.prototype.toString.call(date) === "[object Date]" && !Number.isNaN(date.getTime())
}

// 自訂的錯誤型別
class InvalidDateFormatError extends RangeError {}
class DateIsInTheFutureError extends RangeError {}

function parse(birthday: string): Date | InvalidDateFormatError | DateIsInTheFutureError {
  let date = new Date(birthday)
  if(!isValid(date)){
    // 以下是回傳例外
    return new InvalidDateFormatError("Enter a date in the form YYYY/MM/DD")
  }
  if(date.getTime() > Date.now()){
    // 以下是回傳例外
    return new DateIsInTheFutureError("Are you a timelord?")
  }
  return date
}

let result = parse(ask())
if(result instanceof InvalidDateFormatError){
  console.error(result.message)
}else if(result instanceof DateIsInTheFutureError){
  console.info(result.message)
}else{
  console.info("Date is", result.toISOString())
}
```
{% endcode %}

或者可以不用個別處理每個例外，但至少得明確這樣做：

{% code lineNumbers="true" %}
```typescript
let result = parse(ask())
if(result instanceof Error){
  console.error(result.message)
}else{
  console.info("Date is", result.toISOString())
}
```
{% endcode %}



## Option 型別

Option 型別不回傳一個值，而是回傳一個容器，其中可能有，也可能沒有一個值。此容器幾乎可以是任何的資料結構，只要它能存放一個值就行了，例如，能夠使用一個陣列作為容器：

{% code lineNumbers="true" %}
```typescript
function ask(){
  return prompt("When is your birthday?")
}

function isValid(date: Date){
  return Object.prototype.toString.call(date) === "[object Date]" && !Number.isNaN(date.getTime())
}

// 自訂的錯誤型別

function parse(birthday: string): Date[] { // 指定回傳陣列
  let date = new Date(birthday)
  if(!isValid(date)){
    return []   // 回傳陣列作為容器
  }
  return [date] // 回傳陣列作為容器
}

let date = parse(ask())
date.map(_ => _.toISOString())
    .forEach(_ => console.info("Date is", _))
```
{% endcode %}



