# Chapter 4: 강제 변환

강제변환은 유용한 기능일 수도 있고 언어 자체의 결함일 수도 있다.
이 챕터에서는 강제성의 장단점에 대해 알아볼 것이다.

## 4.1 값 변환

- 명시적 => 타입 캐스팅
- 암묵적 => 강제 변환

_알아두어야 할 것:_ JS 강제 변환의 결과는 항상 원시값(`string`, `number`, `boolean`) 중에 하나. `object`나 `function`같은 복잡한 값이 결과인 경우 강제 변환이 일어나는 것은 아님. 자세한 내용은 원시값을 일치하는 `object`로 감싸는 것을 의미하는 "boxing"을 다루는 3장에서.

사람들이 많이 혼동하므로 "암묵적 변환"과 "명시적 변환"이라고 부르겠음

```js
var a = 42;

var b = a + ''; // 암묵적 변환
// number + string = string으로 암묵적 형 변환

var c = String(a); // 명시적 변환
// a를 String으로 변환하겠다. 이건 꽤 명시적이다.
```

결과는 같지만 _어떻게_ 변환이 일어나는 지가 중요함

## 4.2 추상 값 연산

어떻게 값이 `string`, `number`, `boolean`이 _되는지_ 알아보도록 하자.

### 4.2.1 `ToString`

`string`이 **아닌** 어떤 값이 `string`이 될 때 `ToString`가 처리함

- `null` => `"null"`
- `undefined` => `"undefined"`
- `true` => `"true"`
- `number` => 해당 숫자가 문자열로
  - 아주 큰 수나 아주 작은 수는 아래처럼

```js
// `1.07` * `1000`, 이걸 일곱 번 함
var a = 1.07 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000;

// 21자
a.toString(); // "1.07e21"
```

- `object` => `[[Class]]]` === `[object 클래스이름]`
  - 별도로 `toString()`을 손보지 않은 경우에만 해당
    - 이 함수가 추상 속성 `[[Class]]`를 반환
  - object를 `string`으로 강제 변환하면 `ToPrimitive`가 처리함
    - 이거는 추후 `ToNumber`에서 더 설명하겠음
- `Array` => 모든 값을 `","`으로 붙인 문자열

```js
var a = [1, 2, 3];

a.toString(); // "1,2,3"
```

명시적으로 `toString()`을 호출할 수도 있음

#### 4.2.1.1 JSON을 문자열로

- 엄밀히 말해서 **문자열로 바꾸는 것**은 강제 타입 변환과 같은 것은 아님
- 단순한 값들은 `toString()`을 사용하는 것과 동일하게 동작
  - 결과값이 항상 `string` 타입

```js
JSON.stringify(42); // "42"
JSON.stringify('42'); // ""42"" (따옴표를 포함한 문자열임)
JSON.stringify(null); // "null"
JSON.stringify(true); // "true"
```

- *JSON-safe*에 대해
  - `undefined`, `function` (ES6+에서 `symbol`, 순환 참조를 하는 `object`)는 JSON-safe가 **아님**
    - 이런 값은 다른 언어에서 JSON으로 처리할 수 없어서
- `JSON.stringify(...)`가 `undefined`, `function`, `symbol`은 자동으로 제외하고 처리함
  - `array`에 이런게 포함되어 있으면 `null`로 치환됨, 배열 순서를 보장하려고
  - `object`에 포함 시 그냥 제외

```js
JSON.stringify(undefined); // undefined
JSON.stringify(function() {}); // undefined

JSON.stringify([1, undefined, function() {}, 4]); // "[1,null,null,4]"
JSON.stringify({ a: 2, b: function() {} }); // "{"a":2}"
```

- 순환 참조하는 `object`를 `JSON.stringify(...)`에 넣으면 에러 뱉음
- `object`에 `toJSON()` 포함 시 이를 우선적으로 호출
- 처리를 못해주는 값을 포함한 `object`를 변환하고 싶으면 `toJSON()`을 따로 정의해주고, 여기서 *JSON-safe*한 값으로 변환해주도록 하자

```js
var o = {};

var a = {
  b: 42,
  c: o,
  d: function() {},
};

// 순환 참조를 하고 있다.
o.e = a;

// 원래라면 아래를 호출 시 에러가 난다.
// JSON.stringify( a );

// 커스텀 함수를 정의해주자.
a.toJSON = function() {
  // 정상적인 값만 돌려주고 있다.
  return { b: this.b };
};

// 아주 잘 실행이 된다.
JSON.stringify(a); // "{"b":42}"
```

- `toJSON()`에서 `string` 타입이 아니라 값들을 원래의 타입 그대로 반환해야 한다는 점을 잊지 말자
  - 문자열 변환에서 처리할 수 있는 **JSON-safe**한 값을 돌려준다! 문자열이 아니라!

```js
var a = {
  val: [1, 2, 3],

  // 아주 잘 했다!
  toJSON: function() {
    return this.val.slice(1);
  },
};

var b = {
  val: [1, 2, 3],

  // 이거는 원하는 결과가 아니다!
  toJSON: function() {
    return '[' + this.val.slice(1).join() + ']';
  },
};

JSON.stringify(a); // "[2,3]"

JSON.stringify(b); // ""[2,3]""
```

- 그 외 `JSON.stringify(...)`의 유용한 것들 (1)
  - 두 번째 인자로 `array` 혹은 `function` *replacer*를 넘길 수 있음
  - 어떤 값을 필터링할지 알려줌
- `array`인 경우
  - 문자열 배열이어야 함
  - `object`를 처리할 때 해당 키가 있으면 포함
- `function`인 경우
  - `(key, value) => { ... }`의 형태
  - 특정 키를 스킵하고 싶으면 `undefined` 반환
  - 포함하고 싶으면 _값_ 반환
  - 첫 번째 `key`는 `object` 자체가 들어오므로 **필터링**을 하자!
  - 배열값은 재귀
    - `(index, value) => {...}`

```js
var a = {
  b: 42,
  c: '42',
  d: [1, 2, 3],
};

JSON.stringify(a, ['b', 'c']); // "{"b":42,"c":"42"}"

JSON.stringify(a, function(k, v) {
  if (k !== 'c') return v;
});
// "{"b":42,"d":[1,2,3]}"
```

- 그 외 `JSON.stringify(...)`의 유용한 것들 (2)
  - 세 번째 인자로 _space_ 전달 가능
  - 양의 정수: 값들 사이에 몇 칸 띄어쓰기 할지
  - `string`: 들여쓰기에 사용할 문자

```js
var a = {
  b: 42,
  c: '42',
  d: [1, 2, 3],
};

JSON.stringify(a, null, 3);
// "{
//    "b": 42,
//    "c": "42",
//    "d": [
//       1,
//       2,
//       3
//    ]
// }"

JSON.stringify(a, null, '-----');
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"
```

### 4.2.2 `ToNumber`

- `number`가 _아닌_ 값이 숫자 연산에 사용된 경우
  - `true` => `1`
  - `false` => `0`
  - `undefined` => `NaN`
  - `null` => `0`
  - `"3"` => `3`
  - 앞에 `0`이 붙은 8진수는 10진수 처럼 처리됨
  - `object` & `array`
    - 기본 자료형으로 변환 => `number`가 아닌 경우 `ToNumber` 규칙따라 처리
    - `valueOf()`에서 기본 자료형 반환 시 그 값으로 처리
    - `valueOf()` 없고 `toString()` 제공하는 경우 `toString()` 이용
    - 위 모든 경우에 해당 사항 없으면 `TypeError` 뱉음

```js
var a = {
  valueOf: function() {
    return '42';
  },
};

var b = {
  toString: function() {
    return '42';
  },
};

var c = [4, 2];
c.toString = function() {
  return this.join(''); // "42"
};

Number(a); // 42
Number(b); // 42
Number(c); // 42
Number(''); // 0
Number([]); // 0
Number(['abc']); // NaN
```

### 4.2.3 `ToBoolean`

**여기가 제일 헷갈리고 잘못 이해하는 부분**

- `true`, `false` => `boolean` 맞음
- `1`, `0` => `number` 이지만 `true`와 `false` 처럼 쓸 수도 있음
  - 그래도 `boolean`은 아님

#### 4.2.3.1 거짓인 값들

모든 JS 값은 두 부류로 나뉨:

1. `boolean`으로 형 변환 시 `false`가 되는 값
2. 그 외 나머지 다. (`true`로 바뀌는 것들)

`ToBoolean` 연산에 따르면 아래의 값들이 _거짓인_ 값임

- `undefined`
- `null`
- `false`
- `+0`, `-0`, `NaN`
- `""`

**뭐던 간에 위 리스트에 없으면 `boolean`으로 변환 시 참이 된다..!**

#### 4.2.3.2 거짓인 object

모든 object는 참이라면서 이런 챕터는 왜 있는 걸까.
거짓인 object가 *거짓인 값*을 감싼 object를 의미하는 게 *아니라는 것*을 보여주겠다.

```js
var a = new Boolean(false);
var b = new Number(0);
var c = new String('');
```

a, b, c는 거짓인 값을 감싼 object인데, 이게 `true`냐 `false`냐.

```js
var d = Boolean(a && b && c);

d; // true
// 4.2.3.1에서 나온 거짓인 값 목록 외의 모든 것은 true이다.
```

결론적으로 a, b, c 셋 다 `true`이다.

헷갈리겠지만 이건 JS 언어 자체에서 뭘 어떻게 하는 게 아니라 브라우저에서 자체적으로 만든 개념이다.

"거짓인 object"는 일반적인 object처럼 보이고 그렇게 동작하지만 `boolean`으로 형 변환했을 때 `false`가 되는 것들이다.

대표 예시: `document.all`

- JS 엔진이 아니라 *DOM*에서 제공
- array-like object
- deprecated된지 오래 됨
  - 오래된, 비표준 IE를 감지하는 용도로 사용
  - `if (document.all) { /* IE이다! */ }`

#### 4.2.3.3 참인 값들

**거짓인 값들 목록에 없으면 다 참인 값임**

```js
var a = 'false';
var b = '0';
var c = "''";

var d = Boolean(a && b && c);

d; // 정답은 `true`. `""`을 제외한 `string`은 전부 참이니까. ;
```

```js
var a = []; // 빈 배열, true
var b = {}; // 빈 object, true
var c = function() {}; // 빈 함수, true

var d = Boolean(a && b && c);

d; // true
```


## 4.3 명시적 강제변환
### 4.3.1 명시적 강제변환: 문자열 <-> 숫자
- String(), Number() 함수를 이용하여 변환
- 함수 앞에 new 키워드가 없으므로 객체 래퍼를 생성하는 것이 아니라 변환만 이뤄짐

```js
var a = 42;
var b = String( a );

var c = "3.14";
var d = Number( c );

b; // "42"
d; // 3.14
```

- String(), Number() 함수를 이용하지 않는 명시적인 타입 변환
```js
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c; // 단항 연산자와 이진 연산자를 붙여서 쓰면 혼동이 올 수 있으니 주의 할 것

b; // "42"
d; // 3.14
```

#### 4.3.1.1 날짜 -> 숫자
- `+`단항 연산자를 통한 Date객체 변환
```js
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );

+d; // 1408369986000

var timestamp = +new Date(); // 타임스탬프 값을 얻기 위한 관용적인 표현
```
- 강제변환을 하지 않는 Date객체 변환
```js
var timestamp = new Date().getTime();
// var timestamp = (new Date()).getTime();
// var timestamp = (new Date).getTime();

var timestamp = Date.now(); // ES5 문법. 되도록 이거 씁시다.
```

#### 4.3.1.2 틸드(~) 연산자
- `~` 연산자는 ToInt32 추상 연산을 한 뒤 NOT 비트와이즈 연산을 한다.
- 즉 해당 값의 2의 보수를 구하게 된다. (32비트 이내의 정수일 경우)
- 함수의 실행 결과로 -1을 실패로 처리하는 함수들을 사용할 때 유용하다.
```js
var a = "Hello World";

if (a.indexOf( "lo" ) >= 0) {	// true
	// found it!
}
if (a.indexOf( "lo" ) != -1) {	// true
	// found it
}

if (a.indexOf( "ol" ) < 0) {	// true
	// not found!
}
if (a.indexOf( "ol" ) == -1) {	// true
	// not found!
}

// ~ 연산자를 사용한 경우
~a.indexOf( "lo" );			// -4   <-- truthy!

if (~a.indexOf( "lo" )) {	// true
	// found it!
}

~a.indexOf( "ol" );			// 0    <-- falsy!
!~a.indexOf( "ol" );		// true

if (!~a.indexOf( "ol" )) {	// true
	// not found!
}
```

- 또 하나의 `~` 연산자의 용도는 비트 잘라내기 위한 `~~`(더블 틸드) 연산자를 사용하기 위함이다.
- 더블 틸드 연산자를 사용할 때 주의할 점.
  - 32비트 정수 에서만 안전하다
  - 음수에서는 Math.floor()와 결괏값이 달라진다
```js
Math.floor( -49.6 );	// -50
~~-49.6;				// -49
```
  - `x | 0` 도 같은 결과 값을 리턴하고 더 빠르다. (조금이라도)
  - 다만 연산자 우선순위 때문에 사용할 일이 있을 수도 있다.
```js
~~1E20 / 10;		// 166199296

1E20 | 0 / 10;		// 1661992960
(1E20 | 0) / 10;	// 166199296
```

### 4.3.2 명시적 강제변환: 숫자 형태의 문자열 파싱
```js
var a = "42";
var b = "42px";

Number( a );	// 42
parseInt( a );	// 42

Number( b );	// NaN
parseInt( b );	// 42
```

- Number 함수는 비 숫자형 문자를 허용하지 않는다. (NaN을 리턴한다)
- parseInt 함수는 비 숫자형 문자를 허용한다.
- parseInt 함수는 문자열에 쓰는 함수이다.
- ES5 이전에는 parseInt 함수의 두 번째 인자(기수)를 넘기지 않으면 문자열의 첫 번째 문자를 보고 마음대로 추정한다.
  - 첫 번째 문자가 x나 X이면 16진수로, 0이면 8진수로 해석함
  - 시간 등을 파싱할 때 09 와 같은 숫자형 문자열이 넘어오므로 주의해야 함
  
 #### 비 문자열 파싱
 ```js
parseInt( 1/0, 19 ); // 18
```
- 위 예제에서는 `1/0` 이라는 값이 'Infinity' 라는 문자열로 변환되었고 19진법의 18을 뜻하는 I를 숫자로 파싱하여 18이라는 값을 얻게 됨
- 기타 parseInt 예제 참고
```js
parseInt( 0.000008 );		// 0   ("0" from "0.000008")
parseInt( 0.0000008 );		// 8   ("8" from "8e-7")
parseInt( false, 16 );		// 250 ("fa" from "false")
parseInt( parseInt, 16 );	// 15  ("f" from "function..")

parseInt( "0x10" );			// 16
parseInt( "103", 2 );		// 2
```

### 4.3.3 명시적 강제변환: * -> 불리언
- Boolean() 함수를 사용할 수 있지만 주로 `!!`라는 이중 부정 연산자를 사용한다.

## 4.4 암시적 변환
### 4.4.1 암시적 이란?
- 암시적 변환은 자바스크립트에서 제거해야 하는 혹은 쓰지 말아야하는 기능이 아니라 작동원리를 잘 이해하여 코드의 장황함이나 보일러플레이트 등을 줄이는데 유용하게 쓸 수 있는 기능이다.

### 4.4.2 암시적 강제변환: 문자열 <-> 숫자
- `+` 연산자는 숫자의 덧셈, 문자열 접합 두 가지 목적으로 오버로드 된다.
- `+` 피연산자 중 하나가 객체 값이라면 ToPrimitive 추상 연산을 수행한다.
  - ToPrimitive 추상연산은 우선 valueOf() 함수로 단순 원시값이 배열 된다면 그 값을 리턴하고 단순 원시값이 리턴되지 않는다면 toString()으로 넘어간다.

```js
var a = "42";
var b = "0";

var c = 42;
var d = 0;

a + b; // "420"
c + d; // 42

var e = [1,2];
var f = [3,4];

// 배열 값은 valueOf 함수로 단순 원시값으로 변환되지 않기 때문에 toString의 값으로 + 연산이 수행된다
e + f; // "1,23,4"
```

> 추가로 알아둘 것
> ```js
> [] + {} // "[object Object]"
> {} + [] // 0
> // 아래 연산에서 {}는 객체가 아닌 빈 코드 블락으로 해석된다. 따라서 + [] 의 강제변환 값인 0이 리턴된다.
> ({}) + [] // "[object Object]"
> ```
- 숫자는 공백 문자열 `""`와 더하면 간단히 문자열로 강제변환된다.
```js
var a = 42;
var b = a + "";

b; // "42"
```
- String() 함수를 통해 명시적 변환하는 경우 ToPrimitive 추상 연산이 일어나지 않기 때문에 다른 결과가 나오는 경우가 있다.
```js
var a = {
	valueOf: function() { return 42; },
	toString: function() { return 4; }
};

a + "";			// "42"

String( a );	// "4"
```

- `-`,`*`,`/` 연산자는 숫자 연산만 가능한 연산자이므로 피연산자 값은 숫자로 강제변환 된다.
```js
var a = "3.14";
var b = a - 0;

b; // 3.14
```
```js
var a = [3];
var b = [1];

a - b; // 2
```
### 4.4.3 암시적 강제변환: 불리언 -> 숫자
- 범용적인 기법은 아니지만 상황에 따라서 `true`/truthy 값을 1로, `false`/falsy 값을 0으로 변환하여 문제를 쉽게 해결할 수 있다.
- 인자 중 하나만 truthy 값인 것을 확인하는 onlyOne() 함수 구현
```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		// skip falsy values. same as treating
		// them as 0's, but avoids NaN's.
		if (arguments[i]) {
			sum += arguments[i];
		}
	}
	return sum == 1;
}

var a = true;
var b = false;

onlyOne( b, a );		// true
onlyOne( b, a, b, b, b );	// true

onlyOne( b, b );		// false
onlyOne( b, a, b, b, b, a );	// false
```
  - 명시적 변환 버전
```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		sum += Number( !!arguments[i] );
	}
	return sum === 1;
}
```

### 4.4.4 암시적 강제변환: * -> 불리언
- 암시적으로 불리언 강제변환이 일어나는 표현식
  - if() 문의 조건 표현식
  - for(;;) 반복문에서 두 번째 조건 표현식
  - while() 및 do ... while() 루프의 조건 표현식
  - ?: 삼항 연산자의 첫 번째 조건 표현식
  - ||(논리 OR) 및 &&(논리 AND) 의 좌측 피연산자

### 4.4.5 &&와 || 연산자
- ES5 11.11에 명세
  - && 또는 || 연산자의 결괏값이 반드시 불리언 타입이어야 하는 것은 아니며 * 항상 두 피연산자 표현식 중 어느 한쪽 값으로 귀결된다. *
```js
a || b;
// 대략 다음과 같다
a ? a : b;

a && b;
// 대략 다음과 같다
a ? b : a;

// 다만 a 값이 표현식이라면 두 번 평가될 수 있다. 함수 호출 등이 두 번 될 수 있다는 의미.
```
- 자바스크립트 엔진에서 `a || b` 식의 a가 true/truthy 값인 경우 b를 평가하지 않는다.
- 자바스크립트 엔진에서 `a && b` 식의 a가 false/falsy 값인 경우 b를 평가하지 않는다.
  - 이를 이용하여 `&&`를 가드 연산자로 사용할 수 있다.
```js
function foo() {
	console.log( a );
}

var a = 42;

a && foo(); // 42
```

### 4.4.6 심벌의 강제변환
- 여러 이유 때문에 심벌 -> 문자열 명시적 강제변환은 허용되나 암시적 강제변환은 금지되며 시도만 해도 에러가 난다.
- 심벌 <-> 숫자 변환은 불가능하다.
- 심벌 <-> 불리언 변환은 명시적/암시적 모두 강제변환이 가능하다. (항상 true로 변환됨)
- 심벌 값을 강제 변환할 일은 많지 않을 것이다.
