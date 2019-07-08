# Chapter 2 Value 값
## 2.1 배열
- 어떤 타입도 담을수 있다. / 길이도 정하지 않아도 된다.
- 원소를 다 제거해도 length는 남는다.
- js에서는 배열도 객체, 
- 숫자가 키인 객체 따라서 string 키도 삽인 가능.
  > string 숫자인경우 number처럼 동작
```js
var a = [];
var b = [];
a[3] = 1;
b["3"] = 1;
a.length; // 4
b.length; // 4
```
- 중간에 구멍난 배열도 선언 가능
```js
var a = [];
a[0] = 1;
a[2] = 3;
a[1]; // undefined
a.length; // 3
```
### 2.1.1 유사배열
- 진짜 배열로 바꾸고 싶을때 slice()를 사용
- ES6의 경우 Array.from()

## 2.2 문자열
- 배열 같지만 배열과 같지 않다.
```js
var a = "foo";
var b = ["f","o","o"];
a.length === b.length; // 3
a.indexOf("o") === b.indexOf("o"); // 1
a === b // fasle
```
- 문자열은 불변값으로 특정 index에 접근은 가능하나 수정은 불가.
  > 따라서 문자열 매서드는 항상 새로운 문자열을 생성한다. (ex. toUpperCase)
- 배열의 메서드중 불변 배열 메서드만 사용 가능.
  > join, map등은 가능 reverse는 불가
- 따라서 split해서 배열화 한후 수정후 join하는 방식으로 사용

## 2.3 숫자
- number only / int, double같은 타입은 따로 없음
  > [IEEE 754](https://ko.wikipedia.org/wiki/IEEE_754)

### 2.3.1 숫자 구문
- 10진수 리터럴로 표시
```js
.42 === 0.42 // true
42.0 === 42 // true
2e2 === 200 // true
2e+2  === 200 // true (toExponential)
```
- Number.prototype 사용가능 [Number.rototype](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Number/prototype)
- 다른 진법 사용 가능
```js
0xf3 === 243 // true 16진법
0363 === 243 // true?! 8진법 strict에서 사용불가.
0o363 === 243 // true 8진법
0b11110011 === 243 // true 2진법
```

### 2.3.2 작은 소수값
```js
0.1 + 0.2 === 0.3 // false
```
- [WHY?](https://nybounce.wordpress.com/2016/06/30/ieee-754-0-1-0-2-0-30000000000000004-0-1-0-2-%E2%89%A0-0-3/)
- Number.EPSILON 사용해 폴리필 구현
```js
return Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON // true
```

### 2.3.3 안정한 정수 범위
- Number.MIN_SAFE_INTEGER ~ Number.MAX_SAFE_INTEGER

### 2.3.4 정수인지 확인
- Number.isInteger() or Number.isSafeInteger // from ES6
- 폴리필
```js
return typeof num === "number" && num % 1 === 0;
return typeof num === "number" && Math.abs(num) <= Number.MAX_SAFE_INTEGER;
```

### 2.3.5 32비트 (부호있는) 정수
```js
a | 0 // '| 0' 로 32비트 정수로 강제변환 
```

## 2.4 특수값
### 2.4.1 값이 아닌값
- undefined / null 타입과 값이 같다.
- 어떤 값이 있다가 비게 된 값인지 값이 할당되기 전상태를 나타내는지 정의해서 쓰는게 좋다.

### 2.4.2 undefined
```js
undefined = 2; // 가능하나 좋지 않음.
```
  > void연산자 : 결과값이 없음을 나타냄, 항상 undefined로 결과값을 만듬
```js
var a = 2;
void a; // undefined
a; // 2
```

### 2.4.3 특수 숫자
- NaN : 유효하지 않은 숫자, 실패한 숫자
```js
var a = 2 / "foo" ; // NaN
typeof a; // number
var b = 3 / "foo" ; // NaN
a === b // false
```
  > NaN !== NaN 반사성이 없다.
- isNaN()ㅇ로 판단할수 있지만 숫자가 아닌경우도 true이다.
- 따라서 Number.isNaN() from ES6
- 폴리필은 반사성이 없을 통해 구현한다.
```js
Number.isNaN = function(n) {
  return n !== n;
}
```
 - 무한대 (Number.POSITIVE_INFINITY / Number.NEGATIVE_INFINITY)
 - 영 (0 / -0)
```js
var a = 3 / Number.POSITIVE_INFINITY // 0
var b = -3 / Number.POSITIVE_INFINITY // -0
String(b); // "0"
a === b; // true
```

### 2.4.4 특이한 동등 비교
- Object.is() from ES6
```js
var a = 3 / Number.POSITIVE_INFINITY // 0
var b = -3 / Number.POSITIVE_INFINITY // -0
var c = 2 / "foo"; // NaN
Object.is(a, b); // false
Object.is(c, NaN); // true
```
- 효율은 ==, === 연산이 좋다.

### 2.5 값과 레퍼런스
- js는 포인터라는 개념이 없다. 참조하는 개념이 없다.
- 값을 복사 하거나 레퍼런스를 복사하는 개념만 존재
- 값복사 - null, undefined, string, number, booleam, symbol
  > 서로 동시에 값을 변경할 방법은 없다.
```js
var a = 1;
var b = a;
b++; // a = 1, b =2
a = 3; // a = 3 b = 2 
```
- 레퍼런스 복사 - 합성값(객체 배열)
```js
var a = [1,3];
var b = a;
b.push(4); // a =[1,3,4]; b=[1,3,4];
a.push(5); // a =[1,3,4,5]; b=[1,3,4,5];
b = [1,2]; //  a =[1,3,4,5]; b=[1,2];

function foo(x) { // x = a
  x.push(3); // x=[1,2,3] a=[1,2,3]
  x = [4,5]; // x=[4,5] a=[1,2,3]
  x.push(6); // x=[4,5,6] a=[1,2,3]
}

var a = [1,2];
foo(a); // foo(a.slice()); (레퍼런스 복사 -> 값복사)

a; // a=[1,2,3]
```
- 값복사냐 레퍼런스 복사냐를 개발자가 선택할수 없다!
- 레퍼런스 복사 -> 값복사 : slice()등을 활용
- 값복사 -> 레퍼런스 복사 : 원시값을 합성값으로 감싸야한다.
```js
var a = 2;
foo(a); // foo({ a }); (값복사 -> 레퍼런스 복사 )
```