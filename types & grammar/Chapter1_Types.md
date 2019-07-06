# Chapter 1: Types
## 1.1 자바스크립트 내장 타입
- null
- undefined
- boolean
- number
- string
- object
- symbol (ES6에서 추가)
> object를 제외한 이들을 원시 타입(primitives)이라 한다.

## 1.2 object 하위 타입
- function
- array

## 1.3 각 타입에 대한 typeof 결과 값
```js
typeof undefined     === "undefined"; // true
typeof true          === "boolean";   // true
typeof 42            === "number";    // true
typeof "42"          === "string";    // true
typeof { life: 42 }  === "object";    // true
typeof Symbol()      === "symbol";    // true (added in ES6!)

// null 값의 typeof 반환 값은 "null"이 아닌 "object"이다.
// 이는 아주 오래된 버그이고 앞으로도 고쳐지지 않을 버그이다.
typeof null === "object"; // true

typeof function a(){ /* .. */ } === "function"; // true
typeof [1,2,3] === "object"; // true
```

## 1.4 `undefined` vs "undeclared"
 - undefined (정의되지 않은)
   - 접근 가능한 스코프에 변수가 선언되었으나 현재 아무런 값도 할당되지 않은 상태
 - undeclared (선언되지 않은)
   - 접근 가능한 스코프에 변수 자체가 선언조차 되지 않은 상태
 - 다만 두 경우 모두 typeof 함수에서 "undefined"로 반환되기 때문에 어떤 경우인지 구분을 잘 해야한다.

> 코드 예시
> ```js
> // a 변수는 undefined, b 변수는 undeclared
> var a;
> 
> typeof a; // "undefined"
> typeof b; // "undefined"
>
> a; // undefined
> b; // ReferenceError: b is not defined
> ```

## 1.5 typeof 안전 가드
선언되었는지 확인할 수 없는 값을 사용할 때 ReferenceError가 발생하지 않도록 확인하는 역할로 typeof를 사용할 수 있다.
