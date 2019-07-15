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

var b = a + ""; // 암묵적 변환 
// number + string = string으로 암묵적 형 변환

var c = String( a ); // 명시적 변환
// a를 String으로 변환하겠다. 이건 꽤 명시적이다.
```

결과는 같지만 *어떻게* 변환이 일어나는 지가 중요함

## 4.2 추상 값 연산

어떻게 값이 `string`, `number`, `boolean`이 *되는지* 알아보도록 하자.

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
var a = [1,2,3];

a.toString(); // "1,2,3"
```

명시적으로 `toString()`을 호출할 수도 있음

#### 4.2.1.1 JSON을 문자열로

- 엄밀히 말해서 **문자열로 바꾸는 것**은 강제 타입 변환과 같은 것은 아님
- 단순한 값들은 `toString()`을 사용하는 것과 동일하게 동작
    - 결과값이 항상 `string` 타입

```js
JSON.stringify( 42 );	// "42"
JSON.stringify( "42" );	// ""42"" (따옴표를 포함한 문자열임)
JSON.stringify( null );	// "null"
JSON.stringify( true );	// "true"
```

- *JSON-safe*에 대해
    - `undefined`, `function` (ES6+에서 `symbol`, 순환 참조를 하는 `object`)는 JSON-safe가 **아님**
      - 이런 값은 다른 언어에서 JSON으로 처리할 수 없어서
- `JSON.stringify(...)`가 `undefined`, `function`, `symbol`은 자동으로 제외하고 처리함
    - `array`에 이런게 포함되어 있으면 `null`로 치환됨, 배열 순서를 보장하려고
    - `object`에 포함 시 그냥 제외

```js
JSON.stringify( undefined ); // undefined
JSON.stringify( function(){} );	// undefined

JSON.stringify( [1,undefined,function(){},4] );	// "[1,null,null,4]"
JSON.stringify( { a:2, b:function(){} } ); // "{"a":2}"
```

- 순환 참조하는 `object`를 `JSON.stringify(...)`에 넣으면 에러 뱉음
- `object`에 `toJSON()` 포함 시 이를 우선적으로 호출
- 처리를 못해주는 값을 포함한 `object`를 변환하고 싶으면 `toJSON()`을 따로 정의해주고, 여기서 *JSON-safe*한 값으로 변환해주도록 하자

```js
var o = { };

var a = {
	b: 42,
	c: o,
	d: function(){}
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
JSON.stringify( a ); // "{"b":42}"
```

- `toJSON()`에서 `string` 타입이 아니라 값들을 원래의 타입 그대로 반환해야 한다는 점을 잊지 말자
    - 문자열 변환에서 처리할 수 있는 **JSON-safe**한 값을 돌려준다! 문자열이 아니라!

```js
var a = {
	val: [1,2,3],

  // 아주 잘 했다!
	toJSON: function(){
		return this.val.slice( 1 );
	}
};

var b = {
	val: [1,2,3],

  // 이거는 원하는 결과가 아니다!
	toJSON: function(){
		return "[" +
			this.val.slice( 1 ).join() +
		"]";
	}
};

JSON.stringify( a ); // "[2,3]"

JSON.stringify( b ); // ""[2,3]""
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
    - 포함하고 싶으면 *값* 반환
    - 첫 번째 `key`는 `object` 자체가 들어오므로 **필터링**을 하자!
    - 배열값은 재귀
        - `(index, value) => {...}`

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
	if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```

- 그 외 `JSON.stringify(...)`의 유용한 것들 (2)
    - 세 번째 인자로 *space* 전달 가능
    - 양의 정수: 값들 사이에 몇 칸 띄어쓰기 할지
    - `string`: 들여쓰기에 사용할 문자

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, null, 3 );
// "{
//    "b": 42,
//    "c": "42",
//    "d": [
//       1,
//       2,
//       3
//    ]
// }"

JSON.stringify( a, null, "-----" );
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

Remember, `JSON.stringify(..)` is not directly a form of coercion. We covered it here, however, for two reasons that relate its behavior to `ToString` coercion:

1. `string`, `number`, `boolean`, and `null` values all stringify for JSON basically the same as how they coerce to `string` values via the rules of the `ToString` abstract operation.
2. If you pass an `object` value to `JSON.stringify(..)`, and that `object` has a `toJSON()` method on it, `toJSON()` is automatically called to (sort of) "coerce" the value to be *JSON-safe* before stringification.

### 4.2.2 `ToNumber`

If any non-`number` value is used in a way that requires it to be a `number`, such as a mathematical operation, the ES5 spec defines the `ToNumber` abstract operation in section 9.3.

For example, `true` becomes `1` and `false` becomes `0`. `undefined` becomes `NaN`, but (curiously) `null` becomes `0`.

`ToNumber` for a `string` value essentially works for the most part like the rules/syntax for numeric literals (see Chapter 3). If it fails, the result is `NaN` (instead of a syntax error as with `number` literals). One example difference is that `0`-prefixed octal numbers are not handled as octals (just as normal base-10 decimals) in this operation, though such octals are valid as `number` literals (see Chapter 2).

**Note:** The differences between `number` literal grammar and `ToNumber` on a `string` value are subtle and highly nuanced, and thus will not be covered further here. Consult section 9.3.1 of the ES5 spec for more information.

Objects (and arrays) will first be converted to their primitive value equivalent, and the resulting value (if a primitive but not already a `number`) is coerced to a `number` according to the `ToNumber` rules just mentioned.

To convert to this primitive value equivalent, the `ToPrimitive` abstract operation (ES5 spec, section 9.1) will consult the value (using the internal `DefaultValue` operation -- ES5 spec, section 8.12.8) in question to see if it has a `valueOf()` method. If `valueOf()` is available and it returns a primitive value, *that* value is used for the coercion. If not, but `toString()` is available, it will provide the value for the coercion.

If neither operation can provide a primitive value, a `TypeError` is thrown.

As of ES5, you can create such a noncoercible object -- one without `valueOf()` and `toString()` -- if it has a `null` value for its `[[Prototype]]`, typically created with `Object.create(null)`. See the *this & Object Prototypes* title of this series for more information on `[[Prototype]]`s.

**Note:** We cover how to coerce to `number`s later in this chapter in detail, but for this next code snippet, just assume the `Number(..)` function does so.

Consider:

```js
var a = {
	valueOf: function(){
		return "42";
	}
};

var b = {
	toString: function(){
		return "42";
	}
};

var c = [4,2];
c.toString = function(){
	return this.join( "" );	// "42"
};

Number( a );			// 42
Number( b );			// 42
Number( c );			// 42
Number( "" );			// 0
Number( [] );			// 0
Number( [ "abc" ] );	// NaN
```

### 4.2.3 `ToBoolean`

Next, let's have a little chat about how `boolean`s behave in JS. There's **lots of confusion and misconception** floating out there around this topic, so pay close attention!

First and foremost, JS has actual keywords `true` and `false`, and they behave exactly as you'd expect of `boolean` values. It's a common misconception that the values `1` and `0` are identical to `true`/`false`. While that may be true in other languages, in JS the `number`s are `number`s and the `boolean`s are `boolean`s. You can coerce `1` to `true` (and vice versa) or `0` to `false` (and vice versa). But they're not the same.

#### 4.2.3.1 Falsy Values

But that's not the end of the story. We need to discuss how values other than the two `boolean`s behave whenever you coerce *to* their `boolean` equivalent.

All of JavaScript's values can be divided into two categories:

1. values that will become `false` if coerced to `boolean`
2. everything else (which will obviously become `true`)

I'm not just being facetious. The JS spec defines a specific, narrow list of values that will coerce to `false` when coerced to a `boolean` value.

How do we know what the list of values is? In the ES5 spec, section 9.2 defines a `ToBoolean` abstract operation, which says exactly what happens for all the possible values when you try to coerce them "to boolean."

From that table, we get the following as the so-called "falsy" values list:

* `undefined`
* `null`
* `false`
* `+0`, `-0`, and `NaN`
* `""`

That's it. If a value is on that list, it's a "falsy" value, and it will coerce to `false` if you force a `boolean` coercion on it.

By logical conclusion, if a value is *not* on that list, it must be on *another list*, which we call the "truthy" values list. But JS doesn't really define a "truthy" list per se. It gives some examples, such as saying explicitly that all objects are truthy, but mostly the spec just implies: **anything not explicitly on the falsy list is therefore truthy.**

#### 4.2.3.2 Falsy Objects

Wait a minute, that section title even sounds contradictory. I literally *just said* the spec calls all objects truthy, right? There should be no such thing as a "falsy object."

What could that possibly even mean?

You might be tempted to think it means an object wrapper (see Chapter 3) around a falsy value (such as `""`, `0` or `false`). But don't fall into that *trap*.

**Note:** That's a subtle specification joke some of you may get.

Consider:

```js
var a = new Boolean( false );
var b = new Number( 0 );
var c = new String( "" );
```

We know all three values here are objects (see Chapter 3) wrapped around obviously falsy values. But do these objects behave as `true` or as `false`? That's easy to answer:

```js
var d = Boolean( a && b && c );

d; // true
```

So, all three behave as `true`, as that's the only way `d` could end up as `true`.

**Tip:** Notice the `Boolean( .. )` wrapped around the `a && b && c` expression -- you might wonder why that's there. We'll come back to that later in this chapter, so make a mental note of it. For a sneak-peek (trivia-wise), try for yourself what `d` will be if you just do `d = a && b && c` without the `Boolean( .. )` call!

So, if "falsy objects" are **not just objects wrapped around falsy values**, what the heck are they?

The tricky part is that they can show up in your JS program, but they're not actually part of JavaScript itself.

**What!?**

There are certain cases where browsers have created their own sort of *exotic* values behavior, namely this idea of "falsy objects," on top of regular JS semantics.

A "falsy object" is a value that looks and acts like a normal object (properties, etc.), but when you coerce it to a `boolean`, it coerces to a `false` value.

**Why!?**

The most well-known case is `document.all`: an array-like (object) provided to your JS program *by the DOM* (not the JS engine itself), which exposes elements in your page to your JS program. It *used* to behave like a normal object--it would act truthy. But not anymore.

`document.all` itself was never really "standard" and has long since been deprecated/abandoned.

"Can't they just remove it, then?" Sorry, nice try. Wish they could. But there's far too many legacy JS code bases out there that rely on using it.

So, why make it act falsy? Because coercions of `document.all` to `boolean` (like in `if` statements) were almost always used as a means of detecting old, nonstandard IE.

IE has long since come up to standards compliance, and in many cases is pushing the web forward as much or more than any other browser. But all that old `if (document.all) { /* it's IE */ }` code is still out there, and much of it is probably never going away. All this legacy code is still assuming it's running in decade-old IE, which just leads to bad browsing experience for IE users.

So, we can't remove `document.all` completely, but IE doesn't want `if (document.all) { .. }` code to work anymore, so that users in modern IE get new, standards-compliant code logic.

"What should we do?" **"I've got it! Let's bastardize the JS type system and pretend that `document.all` is falsy!"

Ugh. That sucks. It's a crazy gotcha that most JS developers don't understand. But the alternative (doing nothing about the above no-win problems) sucks *just a little bit more*.

So... that's what we've got: crazy, nonstandard "falsy objects" added to JavaScript by the browsers. Yay!

#### 4.2.3.3 Truthy Values

Back to the truthy list. What exactly are the truthy values? Remember: **a value is truthy if it's not on the falsy list.**

Consider:

```js
var a = "false";
var b = "0";
var c = "''";

var d = Boolean( a && b && c );

d;
```

What value do you expect `d` to have here? It's gotta be either `true` or `false`.

It's `true`. Why? Because despite the contents of those `string` values looking like falsy values, the `string` values themselves are all truthy, because `""` is the only `string` value on the falsy list.

What about these?

```js
var a = [];				// empty array -- truthy or falsy?
var b = {};				// empty object -- truthy or falsy?
var c = function(){};	// empty function -- truthy or falsy?

var d = Boolean( a && b && c );

d;
```

Yep, you guessed it, `d` is still `true` here. Why? Same reason as before. Despite what it may seem like, `[]`, `{}`, and `function(){}` are *not* on the falsy list, and thus are truthy values.

In other words, the truthy list is infinitely long. It's impossible to make such a list. You can only make a finite falsy list and consult *it*.

Take five minutes, write the falsy list on a post-it note for your computer monitor, or memorize it if you prefer. Either way, you'll easily be able to construct a virtual truthy list whenever you need it by simply asking if it's on the falsy list or not.

The importance of truthy and falsy is in understanding how a value will behave if you coerce it (either explicitly or implicitly) to a `boolean` value. Now that you have those two lists in mind, we can dive into coercion examples themselves.
