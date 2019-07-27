# Chapter 3: Natives
가장 많이 쓰이는 네이티브
- String()
- Number()
- Boolean()
- Array()
- Object()
- Function()
- RegExp()
- Date()
- Error()
- Symbol - ES6에서 추가
> 네이티브는 사실은 내장 함수(built-in functions)이다.

아래와 같이 네이티브는 생성자처럼 사용할 수 있다
```js
var s = new String("Hello world!");
console.log(s.toString()); // "Hello World!"
```

각 native는 생성자처럼 사용할 수 있지만 생성자의 결과물은 예상과 다를 수 있다.
```js
var s = new String("Hello world")

typeof of s;  // "object" ... "String"이 아니다

s instanceof String; // true

Object.prototype.toString.call(s); // "[object String]"
```
> 네이티브는 사실 `원시 값(primative)`을 감싼 `객체 레퍼`이다.

```js
console.log(native);
```
*Note:* 객체 래퍼에 대한 구현은 각 브라우저 개발자마다 다르게 결정했기 때문에 결과물이 다르다.


## 3.1 내부 [[Class]]
`typeof`가 `object`인 값에는 `[[Class]]`라는 내부 프로퍼티가 추가로 붙는다.
이 프로퍼티는 직접 접근할 수 없고 `Object.prototype.toString()` 메서드에 값을 넣어 호출함으로 존재를 엿볼 수 있다.

```js
Object.prototype.toString.call([1, 2, 3]); // "[object Array]"
Object.prototype.toString.call(/regex/i);  // "[object RegExp]"

// 네이티브 생성자가 없는 Null, Undefined의 경우
Object.prototype.toString.call(null);      // "[object Null]"
Object.prototype.toString.call(undefined); // "[object Undefined]"

// 네이티브 생성자가 있는 타입들의 경우 박싱(Boxing) 과정을 거친다
Object.prototype.toString.call("abc");     // "[object String]"
Object.prototype.toString.call(42);        // "[object Number]"
Object.prototype.toString.call(true);      // "[object Boolean]"
```

## 3.2 래퍼 박싱하기
원시값에는 프로퍼티나 메서드가 없으므로 `.length`나 `.toString()`을 접근하려면 객체 래퍼로 감싸줘야 되지만 알아서 박싱(래핑)을 해주므로 다음과 같은 코드가 가능하다.

```js
var a = "abc";
a.length;        // 3
a.toUpperCase(); // "ABC"
```

개발자가 직접 객체 형태로 선 최적화(pre-optimize)하면 프로그램이 느려질 수 있으므로 직접 객체 형태로 사용하지 않도록 하자.

### 3.2.1 객체 래퍼의 함정
아래 예시는 false라는 원시 값을 감싼 객체이고 객체는 `truthy`하므로 원치 않는 결과가 발생할 수 있다.
```js
var a = new Boolean(false);

if (!a) {
  console.log("Oops"); // 실행되지 않는다.
}
```

## 3.3 언박싱
객체 래퍼의 원시 값은 `valueOf()` 메서드로 추출한다.
```js
var a = new String("abc");
var b = new Number(42);
var c = new Boolean(true);

a.valueOf(); // "abc"
b.valueOf(); // 42
c.valueOf(); // true
```

## 3.4 네이티브, 나는 생성자다.
### 3.4.1 Array()
```js
var a = new Array(1, 2, 3);
var b = [1, 2, 3];

a; // [1, 2, 3]
b; // [1 ,2 ,3]
```
*Note:* Array() 생성자는 `new` 키워드를 꼭 필요로 하지 않는다.

Array 생성자에는 인자로 숫자를 하나만 받으면 배열의 크기를 미리 정하는 기능(presize)이 있다. 하지만, 매우 중요하게도 이렇게 만들 경우 배열은 빈 배열에 `length`값만 바꾼 결과를 불러 일으킨다.

*Note:* '빈 슬롯'을 한 군데 이상 가진 배열을 '구멍 난 배열`이라고 한다.

```js
var a = new Array(3);
var b = [undefined, undefined, undefined];
var c = [];
c.length = 3;

a; // 브라우저 별로 다르다.
b; // [ undefined, undefined, undefined ]
c; // 브라우저 별로 다르다.
/*
[참고] a와 c의 브라우저 별 결과

Chrome
	- v.75.0.3770.142 기준: [empty x 3]
	- 예전: [undefined x 3]
FireFox
  - v.67.0 기준: Array(3) [ <3 empty slots> ]
	- 예전: [ , , , ]
Node
  - v.12.1.0 기준: [ <3 empty items> ]

FireFox 예전 버전에서 콤마가 세 개 있어 슬록이 네 개인 배열처럼 보이는 이유는 ES5부터 후미 콤마(trailing comma)를 허용했기 때문이다.
실제로는 슬록이 세 개인 배열이 맞다.
*/
```

`a`와 `b`가 어떨 때는 같은 값을 보이다가도 그렇지 않을 때도 있다는 점이 더 나쁘다.
```js
a.join( "-" ); // "--"
b.join( "-" ); // "--"

a.map(function(v,i){ return i; }); // [ undefined x 3 ]
b.map(function(v,i){ return i; }); // [ 0, 1, 2 ]
```

빈 슬롯이 아닌 undefined로 채워진 배열을 생성하려면 아래와 같이 생성해야 된다.
```js
var a = Array.apply(null, { length: 3 });
a; // [ undefined, undefined, undefined ]
```

### 3.4.2 Object(), Function(), RegExp()
- `Object`
생성자는 사실상 사용할 일이 없다.

- `Function`
함수의 인자나 내용을 동적으로 정의해야하는, 매우 드문 경우에 한해 유용하다.

- `RegExp`
정규 표현식 패턴을 동적으로 정의할 경우 유용하다.
그렇지 않을 경우 성능상 이점이 있는 리터럴 형식으로 정의할 것을 권장한다.

### 3.4.3 Date(), Error()
- `Date`
생성자는 날짜/시각을 인자로 받는다.
UNIX timestamp를 `getTime()`을 통해 알 수도 있지만 `Date.now()`를 사용하는 방법이 더 쉽다.

*Note:* `new` 키워드 없이 `Date()`만을 호출하면 현재 날짜/시간에 해당하는 문자열을 반환한다.

- `Error`
`Array`와 같이 `new`가 있든 없든 결과가 같다.
주로 현재의 실행 스택 콘텍스트(Execution stack context)를 포착하여 객체에 담아 사용한다.  `error` 객체가 생성된 라인 번호 등 디버깅에 도움이 될 정보들을 담고 있다.

### 3.4.4 Symbol()
심벌(Symbol)은 ES6에서 처음 선보인 새로운 원시 값이다. 심벌은 충돌 염려 없이 객체 프로퍼티로 사용 가능한, 특별한 '유일 값'이다 (절대적으로 유일함이 보장되지는 않는다!).

Symbol은 앞에 `new`라는 키워드를 붙이면 에러가 나는 유일한 네이티브 생성자다.
```js
var mysym = Symbol( "my own symbol" );
mysym;				// Symbol(my own symbol)
mysym.toString();	// "Symbol(my own symbol)"
typeof mysym; 		// "symbol"

var a = { };
a[mysym] = "foobar";

Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
```

*Note:* 심벌은 객체가 아니라 단순한 스칼라 원시 값이다.

### 3.4.5 네이티브 프로토타입
내장 네이티브 생성자는 각자의 `.prototype` 객체를 가지고 해당 객체의 하위 타입별로 고유한 로직이 담겨있다.

*Note:* 문서화 관례에 따라 `String.prototype.XYZ` 는 `String#XYZ`로 줄여 쓴다.

`String.prototype` 객체에 정의되어 있는 메서드들
- `String#indexOf()`: 문자열에서 특정 문자의 위치를 검색
- `String#charAt()`: 문자열에서 특정 위치의 문자를 반환
- `String#substr()`, `String#substring()`, `String#slice()`: 문자열의 일부를 새로운 문자열로 추출
- `String#toUpperCase()`, `String#toLowerCase()` 대문자/소문자로 변환된 새로운 문자열 생성
- `String#trim()`: 앞/뒤 공란이 제거된 새로운 문자열 생성

위 메서드 중 수정(대소문자 변환이나 트리밍)이 일어나면 늘 기존 값으로부터 새로운 값을 생성한다.

```js
Array.prototype; // [] - 빈 배열

typeof Function.prototype; // "function"
Function.prototype();      // 빈 함수

RegExp.prototypes.toString();  // "/(?:)/"
"abc".match(RegExp.prototype); // [""]
```

#### 디폴트값으로서의 프로토타입
변수가 적절한 값을 가지고 있지 않은 경우 프로토타입은 모두 멋진 디폴트값이 될 수 있다.

```js
function isThisCool(vals,fn,rx) {
	vals = vals || Array.prototype;
	fn = fn || Function.prototype;
	rx = rx || RegExp.prototype;

	return rx.test(
		vals.map( fn ).join( "" )
	);
}

isThisCool();		// true

isThisCool(
	["a","b","c"],
	function(v){ return v.toUpperCase(); },
	/D/
);					// false
```

*Note:* ES6 부터는 `vals = vals || 디폴트 값`과 같은 구문트리는 더이상 필요 없다. 왜냐하면 네이티브 문법을 통해 함수 선언부에 있는 인자(parameter)에 디폴트 값들은 설정해 줄 수 있기 때문이다.

프로토타입으로 디폴트 값을 세팅하면 이미 생성되어 내장된 `.prototype`을 이용하므로 사소하지만 추가적인 이점이 있다.

또한 이후에 변경될 디폴트 값으로 `Array.prototype`을 사용하지 말자. 위 예제에서 vals 변수를 수정할 경우 결국 `Array.prototype`도 수정되게 된다.
