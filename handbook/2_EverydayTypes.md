# Everyday Types

js에서 자주 사용되는 타입을 ts에서 어떻게 다루는지 간략하게 알아본다.

## Arrays

아래와 같이 배열의 타입을 정의할 수 있다.

```js
[1,2,3]
// umber[] or Array<number>

['a', 'b', 'c']
// string[] or Array<string>
```

note : `[number]`는 number[]와 다른 형태의 타입으로 tuple types에서 다룬다.

## any

타입 체킹 에러를 발생시키고 싶지 않을 때 사용한다.

```js
let obj: any = { x: 0 };

// any로 선언했기에 타입 에러가 발생하지 않는다.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

만약 value에 type을 지정하지 않는다면 타입스크립트는 context에서 타입을 추론할 수 없게 되고 default로 타입을 any로 추론한다.  
이러한 implicitAny 추론 유무는 `noImplicitAny`를 옵션을 통해 수정할 수 있다.

## Type Annotations on Variables

const, var, let을 통해 변수를 선언할 경우, type annotation을 통해 변수의 타입을 명시할 수 있다.  
`int x = 0;`과 같이 변수 앞에 타입이 선언되는 타 언어와는 달리 `let name:String = 'jun'`과 같이 변수 뒤에 타입이 선언된다.

대부분의 경우에는 type을 선언할 필요가 없으며 타입스크립트가 자동적으로 타입을 추론한다.
```js
let myName = 'alice'
```

## Functions

type annotation을 통해 parameter/return value에 input/output 타입을 명시할 수 있다.

```js
// Parameter type annotation
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}

// return type annotation
function getFavoriteNumber(): number {
  return 26;
}

// parameter/return type annotation
function getKoreanAge(birthDay: Date): number{
  const yyyy = new Date().getFullYear();

  return yyyy - birthDay + 1;
}
```

위와 같이 return 타입을 명시할 수는 있지만 ts는 return의 statement에 따라 타입 추론을 수행하기 때문에 일반적으로 return type annotation을 작성할 필요는 없다. 몇몇 코드베이스에서는 문서화 목적 또는 예기치 못한 수정을 방지하기 위해 리턴 타입을 명시하기도 한다.

## Anonymous Functions

익명 함수는 type annotation이 없을 경우 any로 parameter를 추론하는 함수 선언과는 달리 익명함수가 실행되는 context에서 parameter의 타입을 추론하는데, 이를 contextual typing이라 한다.

```js
const names = ['a', 'b', 'c']

names.forEach(c => console.log(c.toUppercase())); // 타입이 string으로 추론된다.

// Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
```

## Object Types

속성을 구분하는 separator는 `;` 또는 `,`를 사용하며  
마지막 separator는 선택적으로 사용한다.

```js
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 3, y: 7 });

// {x:number, y:number,}도 가능
```

### Optional Properties

객체의 속성이 optional할 경우 `?`를 속성 이름 뒤에 추가한다.

```js
function foo(obj: {a: number, b?: number}){
  const {a,b} = obj;

  return b === undefined ? a + b : a;
}

// both ok
console.log(foo({ a: 1 }))
console.log(foo({ a: 1, b: 2 }))
```

함수의 실행에 의한 내부 로직은 런타임 환경에서 동작하므로 js코드이다. 따라서 위와 같이 undefined에 대한 조건문을 함수 로직 내에서 처리해줘야 한다.

## Union Types

ts의 타입 시스템은 확장 가능한 타입 시스템이다.  
ts의 기본 타입과 여러 연산자를 이용해 새로운 타입을 만들어낼 수 있다.

### Defining a Union Type

union type은 2개 이상의 타입들을 `|` 연산자를 이용해 결합한다.  
union type을 이루는 타입들을 union's member라 한다.

```js
function foo(bar: number | string){
  console.log('number or string : ' , bar);
}

// both ok
foo(10);
foo('hello');
```

### Working with Union Types

union을 이용해 로직을 만들 경우, union을 이루는 모든 member에 대해 로직이 유효해야만 한다.

```js
function foo(bar: number | string){
  console.log(bar.toUpperCase()); // error !
}
```

따라서 union의 타입을 구체화시켜(`typeof`, `Array.isArray()`, ...) 코드를 작성해야하며 이를 Narrowing이라 한다.

```js
function foo(id: number | string){
  if(typeof id === 'string'){
    console.log(id.toUpperCase());
  }else{
    console.log(id);
  }
}

function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // Here: 'x' is 'string[]'
    console.log("Hello, " + x.join(" and "));
  } else {
    // Here: 'x' is 'string'
    console.log("Welcome lone traveler " + x);
  }
}
```

만약 아래와 같이 union을 이루는 members가 공통적으로 로직에 대해 유효할 경우, narrowing을 수행하지 않아도 된다.

```js
function foo(bar: number[] | roo: string){
  return bar.slice();
}
```

## Type Aliases(별칭)

지금까지는 type annotation을 통해 값에 타입을 선언했다.  
매우 직관적이며 간편하게 타입을 선언할 수 있지만, 이들은 모두 재사용이 불가능하다는 단점이 있다.  
타입을 재사용하고자 할 경우 Type Aliases를 사용할 수 있다.

type alias는 `type` 문법을 통해 선언한다.

```js
type Point = {
  x: number;
  y: number;
}

function getDistance(p1:Point, p2:Point){
  // return get distance
}

type ID = number | string;
```

type alias는 어떠한 shape(타입)에 대한 별칭일 뿐이다. 따라서, 아래의 예시와 같이 변수에 선언한 type alias와 shape이 동일한 type을 재할당할 경우 타입 체킹에서 에러가 발생하지 않는다.

```js
type UserInputSanitizedString = string;

function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}

// UserInputSanitizedString 타입을 갖는다.
let userInput = sanitizeInput(getInput()); 

// string 타입을 재할당하더라도 shape이 동일하기에 에러가 발생하지 않는다.
userInput = "new input";
```

## Interfaces

interface는 type alias과 마찬가지로 객체의 shape을 정의하고 이를 재사용할 수 있도록 해준다.
  
차이점이라면, type alias는 정해진 shape을 확장할 수 없지만 interface는 `extends`를 통해 객체 속성을 추가하며 shape을 확장해나갈 수 있다.
  
**type alias(이하 type)과 interface 비교**

```js
// 1. extends

// interface
interface Animal[
  name: string,
]

interface Bear extends Animal{
  honey: boolean
}

// type (with intersection)
type Animal{
  name: string,
}

type Bear = Animal & {
  honey: Boolean,
}

const bear = getBear()
console.log(bear.name, bear.honey)

// 2. adding new field

// interface

interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

// type (error !)
type Window {
  title: string
}

type Window {   // Error: Duplicate identifier 'Window'.
  title: TypeScriptAPI
}
```

interface의 경우 `Window[property명]`으로 에러에 로깅된다.
type 또한 어떤 type명으로 shape이 선언되었는지 확인할 수 있다.

## Type Assertions

여러 web API 객체와(ex `HTMLCanvasElement`) 같이 ts의 기본 타입으로 추론할 수 없지만, 타입이 명확한 것들은 `as`를 통해 type assertion을 통해 타입을 명시할 수 있다.

```js
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

type annotation과 같이 컴파일 단계에서 제거되어 런타임에는 영향을 미치지 않는다.  

jsx syntax를 사용하는 게 아니라면, `<>`을 사용할 수도 있다.

```js
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

Reminder : type assertion은 컴파일 단계에서 제거되므로 type assertion에 대한 체킹이 런타임에서는 발생하지 않는다.  

type assertion은 type을 더 구체화 할 경우에만 사용될 수 있다.  
A as B이면 B가 A의 부분집합이어야한다.
```js
const x = 'hello' as number;
// Error! Conversion of type 'string' to type 'number' may be a mistake because neither type sufficiently overlaps with the other. If this was intentional, convert the expression to 'unknown' first.
```
# ????
Sometimes this rule can be too conservative and will disallow more complex coercions that might be valid. If this happens, you can use two assertions, first to any (or unknown, which we’ll introduce later), then to the desired type:

const a = (expr as any) as T;

## Literal Types

원시 타입인 string / number / Boolean 타입들은 어떠한 특정 값을 type으로 정할 수 있다.  
js에서는 const,let으로 상수값을 할당하기 때문에 ts에서 하나의 literal type을 사용하는 것은 무의미하지만, Union type으로 여러 원시 값을 묶어 사용할 경우 literal type이 유용하게 사용된다.  

```js
// ex 1)
const pallete = {
  primary: 'red',
  secondary: 'blue',
}

function Button({ color: 'primary' | 'secondary' }){
  return <button style={{ backgroundColor: pallete[color] }}/>
}

// ex 2)
function compare(a: string, b: string) : -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : a < b : -1;
}
```

literal type은 type 중 하나일 뿐이므로, 다른 형태의 type과 union type으로 묶을 수도 있다.  

```js
// ex 3)
interface foo { 
  bar: 'a' | 'b' | 'c',
  count: number
}

function configure(bar : Options | "auto"){
  // ...
}

// ex 4)

// boolean 타입은 2개의 boolean literal type인 true/false를 결합한 union type이다.
function toggle(state: true | false ){
  return !state;
}
```

### Literal Inference

객체를 변수에 초기화할 때, ts는 객체의 속성이 갖는 값이 변할 수 있는 것을 가정한 채로 타입을 추론한다.

```js
const obj = { counter: 0 }; // 0이라는 특정 값이 아닌, number로 타입을 추론한다.
if (someCondition) {
  obj.counter = 1;
}
```

만약 객체의 속성값을 특정 값인 literal type으로 추론하길 원한다면, type assertion을 통해 타입을 구체화한다.

```js
// Change 1: as를 이용해 "GET"이라는 literal type으로 객체의 속성값의 타입을 명시한다.
const req = { url: "https://example.com", method: "GET" as "GET" };

// Change 2
handleRequest(req.url, req.method as "GET");
```

`as const`를 사용할 경우 객체의 모든 속성값을 literal type으로 간주하게 된다.
```js
const req = { url: '/', method: 'GET'} as const
```

## null and undefined

값이 없음을 명시하는 `null`, 아직 값이 초기화되지 않은 상태를 나타내는 `undefined`는 ts에도 같은 이름으로 존재하지만, `strictNullChecks` 컴파일 옵션에 따라 다르게 타입 체킹이 이루어진다.

### strictNullchecks

`on` : null, undefined가 될 수 있는 타입일 경우, Narrowing을 통해 타입을 구체화해 null 또는 undefined에 대한 로직을 처리해야 한다.

```js
function doSomething(x: string | undefined) {
  if (x === undefined) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```

### Non-null Assertion Operator (Postfix 1)

값이 undefined 또는 null이 아니라는 것을 명시할 때 사용한다.  
런타임에서는 반영되지 않기 때문에 반드시 `null` 또는 `undefined`라는 것이 확실할 때에만 사용하도록 한다.  

예시를 통해 살펴보면
```js

// 인자로 전달되는 파라미터가 number 또는 undefined이므로 ts 에러가 로깅된다.
function liveDangerously(x?: number | undefined) {
  console.log(x.toFixed());
}

// non-null assertion 사용
// x가 null 또는 undefined가 아니라는 것을 ts에 알림으로써 ts 에러가 발생하지 않는다.
function liveDangerously(x?: number | undefined) {
  console.log(x!.toFixed());
}

liveDangerously() // ok
liveDangerously(2.3) // ok
liveDangerously(undefined) // ok
liveDangerously('hello') // type error

// 런타임에서는 `x.toFixed`가 실행될 때 에러가 발생한다.
[ERR]: Cannot read property 'toFixed' of undefined 
[ERR]: "Executed JavaScript Failed:" 
```

### Enums

Enum Reference에서 살펴본다.

### less Common Primitives

```js
// js의 BigInt
const oneHundred: bigint = BigInt(100);

// js의 Symbol

const firstName = Symbol("name");
const secondName = Symbol("name");

if (firstName === secondName) {
This condition will always return 'false' since the types 'typeof firstName' and 'typeof secondName' have no overlap.
  // Can't ever happen
}
```