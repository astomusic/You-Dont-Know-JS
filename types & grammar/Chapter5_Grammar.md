# Chapter 2 문법
## 5.1 문과 표현식
- 문장 (안에) 어구들 / 어구 (안에) 구두점, 접속사, 어구
- 문 (안에) 표현식들 / 표현식 (안에) 연산자, 표현식
```js
var a = 3 * 6; // 선언문 -> 할당표현식(a = 3 * 6) -> 표현식(3 * 6)
var b = a; // 문 -> 할당표현식(b = a)
b; // 표현식문(표현식이 전부인 문)
```
### 5.1.1 문의 완료값
- 모든 문은 완료값을 가진다.
- 완료값 확인 방법 : 브라우져 개발자 콘솔 / eval() / do표현식(ES7)
```js
// 브라우져 개발자 콘솔
var a = 3 * 6; // 완료값 undefined
// eval() / 좋아 보이진 않는다.
var b, c;
b = eval( "if (true) { c = 10 + 12 }" );
b;
// do표현식 
// 크롬에서도 현재 미지원 http://kangax.github.io/compat-table/esnext/
var d, e;
d = do {
  if (true) { e = 12 + 14 }
};
d;
```
### 5.1.2 표현식의 부수 효과
```js
function foo() {
  a = a + 1;
}
var a = 1;
foo(); // 완료값 undefined, 부수효과 a가 변경됨

var c = 1;
var d = c++; // c=2, d=1 / 후위 연산에 따른 부수효과
var d = ++c; // c=2, d=2 / 전위 연산하면 발생하지 않음
var d = (c++); // c=2, d=1 / 캡슐화 해도 마찬가지
var d = (c++, c); // c=2, d=2 / 콤마연산자로 처리 가능

var obj = { item: 22 };
delete obj.item // true
obj.item // undefined

var a, b, c;
a = b = c = 42 // 부수효과를 활용한 연쇄 할당
// c = 42 -> 42 / b = 42 -> 42 / a = 42;

function vowels(str) {
  var matches;
  if (str && (matches = str.match(/[aeiou]/g))) { // 부수효과를 이용한 if문내 처리
    return matches; 
  }
}
```

### 5.1.3 콘텍스트 규칙
- 중괄호 : {}
- 객체 리터럴
```js
var a = {
  foo: bar()
}; // {}는 a의 할당될 객체 레터럴
```
- 레이블
```js
{
  foo: bar()
}; // foo는 bar()문의 레이블
```
- 블록
```js
[] + {}; // "[object Object]" / [] -> " "(강제변환) " " + {} -> 은 string으로 강제 변환 됨
{} + []; // 0 / {} 아무것도 하지않는 빈 블록 취급되고, + [] -> 0 으로 강제 변환 됨
```
- 객체분해 (es6)
```js
function getDate({ e, f }) {
  // ...
  return {
    a: 42,
    b: "foo",
  } 
}
var { a, b } = getDate();
// var res = getData();
// var a = res.a;
// var b = res.b;

getDate({ e: "hello", f: "world" }); // 객체 할당을 이용한 프로퍼티 할당
```
- else if와 선택적 블록
```js
else if(a) { // else if는 존재하지 않는다.
  // ...
}

else { // else과 if로 파싱되어 작동한다.
  if (a) {
    // ...
  }
}
```
## 5.2 연산자 우선순위
https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/%EC%97%B0%EC%82%B0%EC%9E%90_%EC%9A%B0%EC%84%A0%EC%88%9C%EC%9C%84

### 5.2.1 단락 평가
- a && b 조건에서 a가 false면 b는 연산하지 않는다.
- a || b 조건에서 a가 true면 b는 연산하지 않는다.
```js
var a = 1;
var b = () => {
  console.log('hi')
  return 'there';
};
a && b(); // hi "there"
a || b(); // 1
a = 0;
a && b(); // 0
a || b(); // hi "there"
```
### 5.2.2 끈끈한 우정
- &&, || 는 ? : 보다 우선순위가 높다. 5.2의 링크 참고.
```js
a && b || c ? a : b; // -> (a && b || c) ? a : b
```
### 5.2.3 결합성
- 좌측결합성 / && ||
```js
a && b || c // (a && b) || c
```
- 우측결합성 / ? : =
```js
a ? b : c ? d : e // a ? b : (c ? d : e)
a = b = 38; // a = (b = 38);
```
### 5.2.4 분명히 하자
- 연산자 우선순위/결합성을 활용하여 짧고 깔끔한 코드를 작성하되
- 혼동되는 부분은 ()를 통해서 명시적으로 어떤 연산이 먼저 되길 바라는지 감싸주는게 좋다.
## 5.3 세미콘론 자동 삽입
- ASI(autometic semicolon insertion)
- return의 예처럼 break, countinue, yield도 동일 
```js
return
a *= 2; 
// return 뒤에 ; 자동 삽입되어 a가 리턴 되지 않는다.
return (
  a *= 2;
);
 // a를 리턴한다.
```
### 5.3.1 에러 정정
- ASI사용에 대한 논쟁이 존재
- 작가 - ASI는 최소화하자. 필요하다고 생각되어지는 곳이라면 세미콜론 사용하다.
- 블렌단(js창시자) - 유효개행문자처럼 ASI사용하면 문제
## 5.4 에러
- 하위 에러 타입 (TypeError. ReferenceError. SyntaxError)
- js는 컴파일 도중 조기에러 존재 (코드 실행전이라 try catch로 잡을수 없다.)
- strict 모드에서는 더 많은 조기 에러 존재
```js
var a = /+foo/; // 조기에러
function foo(a, b, a) { } // 정상
function foo(a, b, a) { "use strict" } // 에러
```
### 5.4.1 너무 이른 변수 사용
- es6 임시 데드 존(TDZ) : 초기화 이전에 변수 참조 불가능한 영역
- let이 대표적,  typeof도 마찬가지다.
```js
{
  a = 2; // ReferenceError
  b = 3; // 3
  typeof a; // ReferenceError
  typeof b; // number
  let a; // 선언전에 할당으로 에러
}
```
## 5.5 함수 인자
- TDZ 관련 에러 in 디폴트 인자값
```js
function poo(a = 42, b = a + b) { } // b가 선언되지 전이라 a+b TDZ에러
function foo(a = 42, b = a + 1) {
  console.log(
    arguments.lenght,
    a,b,
    arguments[0], arguments[1],
  );
}
foo(); // 0, 42, 43, undefined, undefined
foo(10); // 1, 10, 11, 10, undefined
foo(10, undefined); // 2, 10, 11, 10, undefined
foo(10, null); // 2, 10, null, 10, null
```
- undefined를 명시적으로 넘겨주었을때와 빈체로 넓길떄 인자값의 길이가 다르다.
- null일경우 치환되지 않고 인자값 자체가 null이된다.
- 인자와 이 인자에 해당하는 arguments 슬롯을 동시에 참조하지 마라.

## 5.6 try...finally
- try 이후 catch나 finally중 하나만 필수
- finally는 반드시 실행됨 콜백함수처럼
- finally와 break는 함꼐 쓰지말자.(리턴을 취소해버림)
```js
function foo() {
  try {
    return 38; // throw 도 마찬가지
  } finally {
    console.log('hello'); 
    // throw 30; // finally부에서 예외를 던지면, try의 리턴값 사라진다. // 30
    // return; // finally부에서 return값이 우선된다. try의 리턴값 사라진다. // undefined
  }
  console.log('here');
}
foo(); // hello 38 
```
## 5.7 switch
- default 이후 에서도 break를 안쓰면 계속 실행된다.
- switch 문의 조건은 true가 아닐경우 매치실패한다.
- break가 명시적으로 없는 코드는 이유나 설명을 달아놓자.
- switch 문의 조건에서 연산은 피하자.
```js
var a = 'hi';
var b = 11
switch (true) {
  case (a || b === 11): { // 단락평가에의해 'hi'를 반환하므로 엄격히 true인 경우만 매치된다. 
    console.log("never"); // 출력되지 않음
    break;
  }
  default: {
    console.log("here"); // 출력됨 / 하단에 캐이스를 추가하면 달라짐.
  }
  // 하단에 case를 추가할경우 default 이후 에서도 break를 안쓰면 계속 실행된다.
  // case (a === 'hi'): {
  //   console.log("hi");
  //   break;
  // }
}
```
