# Chapter 4: 강제 변환

강제성은 유용한 기능일 수도 있고 언어 자체의 결함일 수도 있다.
이 챕터에서는 강제성의 장단점에 대해 알아볼 것이다.

## 4.1 값 변환

- 의도적 => 타입 변환
  - 컴파일 시
- 암묵적 => 강제 변환
  - 사용하는 값에 따라 결정
  - `string`, `number`, `boolean`이 사용된 경우 해당 타입 따라
  - `object`, `function` 같은 값이 사용되면 강제 변환 없음
  - 런타임에서 동적으로

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

- `object` => `[object Object]`
  - 별도로 `toString()`을 손보지 않은 경우에만 해당
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
  - `undefined` => `Nan`
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
