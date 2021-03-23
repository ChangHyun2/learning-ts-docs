# The Basics


```js
// toLowerCase 속성 접근
// toLowerCase 속성에 할당된 함수 실행
message.toLowerCase();

// 함수 실행
message();
```

위 코드는 `message`가 어떤 shape로 정의되었는지에 따라  
런타임에서 정상 작동되거나 에러가 발생한다.  

만약 `message`에 할당된 값을 위 코드만으로 알 수 없다면 아래의 물음에 답할 수 없을 것이다.

- message가 callable한지?
- `toLowerCase` 속성을 갖는지? => 존재한다면, callable한지?
- callable하다면, 이들은 어떤 값을 리턴하는지?

## Static Type Checking

위와 같은 원시 타입은 `typeof` 연산자를 통해 type을 확인할 수 있지만  
js는 런타임에서 타입을 체크하기 때문에(동적 타이핑) 런타임 이전에 타입을 체킹할 순 없다.  

함수의 경우 동적으로 내부 로직이 실행되기 때문에 타입에 의한 이슈가 발생하기 쉽다.

```js
// fn이 실행될 때가 되어서야 비로소 flip 속성 유무와 타입을 확인할 수 있다.
function fn(x){
  return x.flip();
}
```

이러한 JS의 단점을 보완하기 위해 Static type system을 사용하고자 TS가 개발되었다.

## non Exception Failures

js에서는 객체의 잘못된 속성에 참조할 경우 undefined를 리턴한다.

```js
const user = {
  name: 'jun',
}

// in js
user.location; // returns undefined
```

한 편, ts에서는 static type system을 통해 정의되지 않은 속성에 대해 에러를 발생시킨다.

```js
// in ts
user.location; // Property 'location' does not exist on type '{ name: string; age: number; }'.
```

오타 확인이 수월해진다.

```js
announcement.toLocalLowerCase();

function flipCoin() {
  // Meant to be Math.random()
  return Math.random < 0.5;  
// Operator '<' cannot be applied to types '() => number' and 'number'.
}

const value = Math.random() < 0.5 ? "a" : "b";
if (value !== "a") {
  // ...
} else if (value === "b") {
  // This condition will always return 'false' since the types '"a"' and '"b"' have no overlap.
  // Oops, unreachable
}
```

## Types for Tooling

type checker를 통해 editor에서 코드 작성 시 자동완성, quick fix와 같은 여러 도움을 받을 수 있다.

## TSC (compiler)

type-checking은 type-checker(컴파일러)를 통해 이루어진다.  
  
tsc 설치
```js
npm i -g typescript
```

tsc로 ts파일 실행
```js
tsc hello.ts
```

ts 파일을 실행해 타입을 체킹하고 js파일을 생성한다.  
만약 에러가 발생한다면, 에러 로깅 후 js파일을 생성한다. (js => ts로 전환 시 유용)

option을 통해 에러 발생 시 파일을 emit하지 않을 수도 있다.
```js
tsc --noEmitOnError hello.ts
```

## Explicit Types

type annotation

```js
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

// annotation에 해당되지 않는 타입을 입력으로 전달받을 경우 에러를 발생시킨다.
greet('Maddison', Date()); // string => error 

greet('Maddison', new Date()); // Date
```

타입 annotation을 통해 타입을 명시하지 않을 경우 ts는 타입을 추론한다.
```js
let msg = 'hello'
// ^= let msg: string
```

## Erased Types

tsc를 통해  ts파일을 실행하면 아래와 같이 js 코드로 변환되어 js파일을 생성한다.  
런타임 환경에서는 js 언어를 사용하므로, ts 문법은 모드 제거되며 js문법 호환성을 해결하기 위해 ES3 문법으로 변환한다.

```js
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`);
}

// 
"use strict";
function greet(person, date) {
    console.log("Hello " + person + ", today is " + date.toDateString() + "!");
}
greet("Maddison", new Date());
```

js문법의 버전은 --target 옵션을 이용해 변경할 수 있다.

```js
tsc --target es2015 input.ts
```

## Strictness

타입 체킹의 strictness를 수정할 수 있다.
https://www.typescriptlang.org/docs/handbook/tsconfig-json.html

## tsconfig.json / jsconfig.json

- command line에 input file이 명시되지 않을 경우, 현 디렉토리의 tsconfig.json를 찾고 없을 경우 상위 디렉토리를 찾게된다.
- --project 옵션을 통해 tsconfig.json의 디렉토리 위치를 정의할 수 있다.

ex)

- files 속성을 통한 설정

```js
// tsconfig.json
{
"compilerOptions": {
  "module": "commonjs",
  "noImplicitAny": true,
  "removeComments": true,
  "preserveConstEnums": true,
  "sourceMap": true
},
"files": [
  "core.ts",
  "sys.ts",
  "types.ts",
  "scanner.ts",
  "parser.ts",
  "utilities.ts",
  "binder.ts",
  "checker.ts",
  "emitter.ts",
  "program.ts",
  "commandLineParser.ts",
  "tsc.ts",
  "diagnosticInformationMap.generated.ts"
]
}
```

- files 속성을 통한 설정

```js
{
"compilerOptions": {
  "module": "system",
  "noImplicitAny": true,
  "removeComments": true,
  "preserveConstEnums": true,
  "outFile": "../../built/local/tsc.js",
  "sourceMap": true
},
"include": ["src/**/*"],
"exclude": ["node_modules", "**/*.spec.ts"]
}
```