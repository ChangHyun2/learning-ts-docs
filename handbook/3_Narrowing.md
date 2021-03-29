# Narrowing

입력에 비해 출력에서 처리할 수 있는 타입의 종류가 적을 경우 에러가 발생한다.
```js
function padLeft(padding: number | string, input: string) {
  return new Array(padding + 1).join(" ") + input;
}
```

이러한 경우 ts 타입 시스템의 type narrowing을 적용하면 에러가 발생하지 않는다.
```js
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return new Array(padding + 1).join(" ") + input;
  }

  return padding + input;}
}
```

위 코드에서 type annotation을 제거한다면, 함수 내의 로직이 일반적인 js 코드로 보이게 되는데 ts의 타입 시스템은 의도적으로 js 친화적인 코드를 작성할 수 있도록 설계되었다.  
따라서 js 프로그래머에게는 if/else로 구성된 매우 단순한 구조로 보이지만, ts에서는 위와 같은 코드에 여러 개념들을 포함하고 있다.  

- type guard : 위의 예시 if 괄호 내에서 `typeof padding === 'number'` 표현식을 type guard라 한다.
- type narrowing : type guard를 통해 parameter에 선언된 type이 구체화되는데 이를 type narrowing이라 한다. 

```js
// type guard를 통해 type을 좁히며 이를 type narrowing이라 한다.
// 이를 통해 return 문에서 padding의 type이 바뀌게 된다.
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return new Array(padding + 1).join(" ") + input;
//                   ^ = (parameter) padding: number
  }
  return padding + input;
  //     ^ = (parameter) padding: string
}
```

## typeof type guards

js의 `typeof`는 런타임에서 값이 갖는 타입을 확인할 수 있도록 구현되었으며 여러 라이브러리에서 자주 사용되는 연산자이다. 따라서 ts 또한 이를 지원하며 `typeof`를 통해 타입을 확인할 경우 다음과 같은 string이 리턴된다.
- "string"
- "number"
- "bigint"
- "boolean"
- "symbol"
- "undefined"
- "object"
- "function"

ts에서는 `typeof`를 통해 리턴되는 값을 type guard 표현식에서 활용한다.  
한 편 아래와 같이 js와 같이 `typeof`를 통해 `null`값을 확인할 경우 리턴되는 값은 `object`가 된다.
```js
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
Object is possibly 'null'.
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```

따라서, 이러한 `null`값을 체킹하기 위해서는 Truthiness narrowing을 사용해야 한다.  

아래부터는 여러 type narrowing 방법을 알아본다.

## Truthiness narrowing

js의 falsy 값은 다음과 같다.
- 0
- NaN
- ""
- 0n (bigint version of zero)
- null
- undefined

위의 falsy 값 외에는 모두 truthy 값이 된다.

falsy/truthy 값을 조건문에서 활용할 경우 가독성을 위해 true/false로 값으로 바꾸기도 하는데, 이 때 아래와 같은 방법이 가능하며 주로 `!!`가 사용된다.

```js
Boolean('hello')
!!'world'
```

truthiness 확인을 통한 narrowing은 특히나 null, undefined를 guard할 때 주로 사용된다.

```js
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") { // truthniess narrowing에 의해 null이 제거된다.
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```

아래와 같이 truthiness narrowing을 최외곽에서 수행할 경우, '', 0과 같은 값을 처리할 때 'string', 'number' 타입 가드로 브랜칭되지 않음에 주의한다.  
따라서 아래 패턴은 가급적 지양하도록한다. (linter에서 이에 대한 linting을 지원한다.)
```js
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  DON'T DO THIS!
  //   KEEP READING
  // !!!!!!!!!!!!!!!!
  if (strs) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }

  console.log('hello');
}

printAll('') // hello
```

false를 narrowing할 경우 부정문 브랜치에서 필터링한다.

```js
function multiplyAll(
  values: number[] | undefined, 
  factor: number
){
  if(!values){
    return values;
  }else {
    return values.map(x => x * factor);
  }
}

console.log(multiplyAll(undefined, 1)) // undefined
```

## Equality narrowing

`===` `!==` `==` `!=` 연산자를 통해 narrowing하는 것을 equality narrowing이라 한다.

```js
function ex(x: string | number, y: string | boolean){
  if(x === y){
    x.toUpperCase(); // string
    y.toUpperCase(); // string
  }else{
    console.log(x); // string | number
    console.log(y); // string | boolean
  }
}
```
위 예시의 if문에서 strict equality 연산자인 `===`를 통해 x, y의 equality를 확인해 narrowing함으로써 ts는 if문 내에서 x,y가 모두 string 타입이라는 것을 알 수 있다.

한 편, 앞서 다루었던 `printAll` 예시는 아래와 같이 수정할 수 있다.
```js
function printAll(strs: string | string[] | null) {
  if (strs !== null) { 
    if (typeof strs === "object") {
      for (const s of strs) {
    // ^ = (parameter) strs: string[]
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    // ^ = (parameter) strs: string
    }
  }
}
```

loose한 항등 연산을 수행하는 equality 연산자 `==` `!=`는 아래와 같이 사용된다.
```js
interface Container {
  value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null) {
    console.log(container.value);
//                        ^ = (property) Container.value: number

    // Now we can safely multiply 'container.value'.
    container.value *= factor;
  }
}
```

## instanceof narrowing

프로토타입을 확인할 때 사용되는 `instanceof`를 통해 narrowing할 수 있다.  
자세한 내용은 ts의 class문법에서 다루도록 한다.
```js
class A{
  foo(){}
}
class B extends A {}

class C {}

const b = new B()

function foo(){
  if(b instaceof B){
    console.log('instance from b')
  }else if(b instanceof A){
    console.log('instance from a')
  }else if(b instanceof C){
    console.log('instance from c')
  }
}
```

## Assignments

변수를 선언할 때 선언문에 할당되는 표현식에 의해 변수가 가질 수 있는 타입의 범위가 결정되며, 변수에 값을 재할당할 경우 재할당되는 표현식에 따라 narrowing이 발생한다. (narrowing은 선언문의 타입 범위를 기준으로 narrowing된다.)

```js
let x = Math.random() < 0.5 ? 10 : "str"; // x: number | string 
x+1; // ts error!

x = 1; // x: number
x.toUpperCase();  // ts error !
x+1; // ok

x = "str"; // x: string
Math.floor(x) // ts error !
x+1 // ok
```

위의 예시에서 x의 선언문(`let x = ~~`에서 타입의 범위가 number | string으로 결정되었으므로 아래의 재할당에서는 에러가 발생한다.

```js
x = {
  a: 1
}
```

만약 변수를 선언만 해둔다면, 타입 범위가 any로 결정되어 어떤 타입이든 재할당할 수 있게 된다.

```js
let x;

x = 1;
x = 'str';
x = Math.random () < 0.5 ? 10 : 'str'; // narrowing은 선언문의 타입 범위를 기준으로 narrowing된다.
```

## Control flow analysis

type guard 또는 assignment에 의한 type narrowing은 근본적으로 코드의 실행 가능성(code rechability)에 의해 발생하며 code rechability는 control flow analysis에 의해 결정된다.
바꾸어 말하면 ts는 type guard 또는 assignment 코드를 마주할 때, control flow analysis를 통해 code rechability를 결정하고 code rechability에 따라 type의 범위를 좁히게 된다.

```js
function example() {
  let x: string | number | boolean;

  x = Math.random() < 0.5; // x: boolean

  if (Math.random() < 0.5) { // x: boolean
    x = "hello"; // x: string
    x.toUpperCase() // ok
  } else { 
    x = 100; // x: number
    Math.floor(x) // ok
  } // x: number | boolean
2
  x + 1 // error !
  return x; // x: string | number
}
```

## Using type predicates

지금까지는 js 코드를 이용해 type narrowing하는 방법을 살펴보았다.  
위처럼 js 코드로 type guard를 작성하다보면 재사용되는 type guard가 발생하게 되는데, 이 때 type predicates를 이용해 type guard를 함수로 모듈화한다면 더 나은 코드를 작성할 수 있다.  
type predicate(술부)는 `parameterName is Type` 문법으로 사용되며 아래 예시에서는 `pet is Fish`가 type predicate에 해당된다. 
```js
interface Fish{
  swim: () => void
}
interface Bird{
  fly: () => void
}

const getSmallPet = (): (Fish | Bird) => {
  return Math.random() > 0.5 ? 
  {
    swim: () => {}
  } : 
  {
    fly: () => {}
  }
}

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

let pet = getSmallPet();

if(isFish(pet)){
  pet.swim(); // ok
  pet.fly(); // error
}else{
  pet.fly(); // ok
  pet.swim(); // error
}
```

 
ts는 type guard에 의해 if문에서 Fish 타입으로 type을 좁히며 else문 또한 control flow analysis에 의해 Bird로 type이 좁혀진다.  

type predicates를 통해 선언된 `isFish` 함수는 아래와 같이 콜백함수로 사용될 수 있다.
```js
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];

const underWater1: Fish[] = zoo.filter(isFish);
// or, equivalently
const underWater2: Fish[] = zoo.filter<Fish>(isFish);
const underWater3: Fish[] = zoo.filter<Fish>((pet) => isFish(pet));
```

## Discriminated unions

어떠한 타입이 여러 타입으로 브랜칭될 경우 discriminated unions를 사용한다.

```js
interface Shape{
  kind: 'circle' | 'square'; 
  radius?: number; 
  length?: number;
}

// kind 속성에 예기치 않은 string literal이 입력되는 것을 방지할 수 있다.
```

Shape 타입에 discriminated union type을 이용해 type narrowing을 적용해본다.
```js
interface Shape{
  kind: 'circle' | 'square'; 
  radius?: number; 
  length?: number;
}

function getArea(shape: Shape): number {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2;
  }

  return Math.PI * length!  ** 2;
}

console.log(getArea({
  kind: 'circle',
  radius: 1
}))
console.log(getArea({
  kind: 'circle',
}))
```

위 방법처럼 non null assertion을 이용해 optional type에 대한 ts 에러를 방지할 수 있지만, `radius` 또는 `length` 속성이 정의되지 않는 경우에는 에러 처리가 불가능하다.

대안으로 `Shape` interface를 두 ioterface로 나누어본다면
```js
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;
```

위와 같이 `Shape` 타입을 선언할 수 있다.  
이 때, ts는 `Shape`의 모든 members가 동일한 속성인 `kind`를 가지며 모든 kind 속성값이 literal type인 것을 확인하고, 이를 discriminated union으로 인식하게 된다.  

`Shape` 타입을 브랜칭할 때 `Shape`의 discrimnant property인 `kind`를 이용해 브랜칭하며, switch 또는 if문이 사용될 수 있다.

```js
function getArea(shape: Shape){
  switch (shape.kind){
    case 'circle':
      return Math.PI * shape.radius ** 2;

    case 'square':
      return shape.sideLength ** 2;
  }
}
```

요약하자면 상태 변화, 메세지 변화와 같이 어떠한 literal 값에 따라 사용하는 객체의 타입이 달라질 경우 `kind` 속성을 이용해 객체의 타입을 구분한다.

## The never type

ts는 union의 모든 멤버를 type narrowing한 후에, 더 이상 발생할 수 있는 타입이 없을 경우 `never` 타입을 사용한다. 

## Exhaustiveness checking

`never` 타입은 어떠한 타입에도 할당될 수 있지만, `never` 타입에는 `never`를 제외한 다른 타입이 할당될 수 없다.  
이는 switch문의 default에서 활용될 수 있다.

```js
// Circle , Square 외의 다른 Shape을 사용할 경우 default에서 에러를 발생시킨다.

interface Triangle {
  kind: "triangle";
  sideLength: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
Type 'Triangle' is not assignable to type 'never'.
      return _exhaustiveCheck;
  }
}
```
