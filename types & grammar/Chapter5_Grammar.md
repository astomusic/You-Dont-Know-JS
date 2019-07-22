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

### 5.1.3 콘텍스트 큐칙
