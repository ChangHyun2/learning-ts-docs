# Typescript for JS Programmers

## Types by Inference

ts는 js 언어를 통해 타입을 추론한다.

```js
let helloWorld = 'Hello world';
// ^ = let helloWorld: string
```

## Defining Types

다이나믹 프로그래밍을 사용하는 디자인 패턴에서는 타입 추론이 어렵기 때문에 타입스크립트는 타입을 사전 정의할 수 있도록 설계되었다.

```js
const user = {
  name: 'jun',
  id: 0,
}

interface User {
  name: string; // 콤마가 아닌 세미 콜론임에 주의!
  id: number;
}
```

`: TypeName` 문법으로 타입을 사용할 수 있다.   
아래의 예시에서는 인터페이스를 사용해 객체의 shape를 정의한다. 

```js
const user: User = {
  name: 'jun',
  id: 0,
}

// 인터페이스에 매칭되지 않는 객체는 에러가 발생한다.
const user2: User = {
  username: 'jun',
  id: 1,
}
```

class 문법을 사용할 경우 다음과 같이 interface를 사용할 수 있다.
```js
class UserAccount {
  name: string;
  id: number;

  constructor(name: string, id: number){
    this.name = name;
    this.id = id;
  }
}

const user: User = new UserAccount('Murphy', 1);
```

함수의 파라미터 또는 리턴값을 명시하기 위해 인터페이스를 사용할 수 있다.
```js
function getAdminUser(); User{
}

function deleteUser(user: User){
}
```

TS는 js의 원시타입 외에도 추가적인 타입을 갖는다.
- boolean
- bigint
- null
- number
- string
- symbol
- any
- unknown 
- never

## Composing Types

타입을 조합해 사용할 수 있으며 2개의 방법이 주로 사용된다.
- Unions
- Generics

### Unions

```js
type MyBool = true | false;

type windowStates = 'open' | 'closed' | 'minimize';
type OddNumbersUnderTen = 1 | 3 | 5 | 7 | 9;

// 서로 다른 타입 또한 조합할 수 있다.
function getLength(obj: string | string[]){
  return obj.length;
}
```

타입을 파악하고자 할 경우 `typeof`를 사용한다.
- string	`typeof s` === "string"
- number	`typeof n` === "number"
- boolean	`typeof b` === "boolean"
- undefined	`typeof undefined` === "undefined"
- function	`typeof f` === "function"
- array	Array.isArray(a)

`typeof`를 활용해 type에 따라 서로 다른 로직을 적용할 수 있다.

```js
function wrapInArray(obj: string | string[]){
  if(typeof obj==='string'){
    return [obj];
  }else{
    return obj;
  }
}
```

### Generics

Generics를 이용해 타입에 변수를 전달할 수 있다.

```js
type StringArray = Array<string>;
type NumberArray = Array<number>;
type ObjectWithNameArray = Array<{name: string}>;
```

generics를 통해 커스텀 types을 만들 수 있다.

```js
interface Backpack<Type>{
  add: (obj: Type) => void;
  get: () => Type;
}

declare const backpack: Backpack<string>; // declare를 통해 상단에 const 변수를 미리 선언해둘 수 있다.

const object = backpack.get(); // Generic을 이용해 backpack의 타입(인터페이스)에 string을 전달했기 때문에 object는 string 타입이 된다.

backpack.add(23); // 에러 발생
```

### Structural Type System

TS의 타입 체킹은 값이 갖는 shape를 통해 이루어지며, 이를 `duck typing`, `structural typing`이라 한다.

```js
interface Point {
  x: number;
  y: number;
}

function logPoint(p: Point): void{
  console.log(`${p.x} ${p.y}`)
}

const point = {x: 12, y: 26};
// point에는 타입이 선언되지 않았지만,

// logPoint에서는 point 변수의 shape를 통해 타입을 체크한다.
logPoint(point);

const color = {hex: '#13232"};
logPoint(color); // shape 불일치 => error!
```

class 문법 또한 위처럼 duck typing을 통해 타입을 체크한다.