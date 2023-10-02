[[ES2015 (ES6)]]에서 추가된 엄격 모드를 사용하면 전체 스크립트 또는 코드 블록 내에서 몇가지 시멘틱 변경이 발생한다.

1. 조용하게 무시하던 에러를 throwing 한다.
2. 자바스크립트 엔진의 최적화 작업을 어렵게 만드는 실수를 교정한다. 즉, 비엄격 모드의 동일한 동작의 코드보다 더 빠른 실행 속도를 가지도록 한다.
3. ECMAScript의 다음 명세에서 정의될 몇가지 문법을 적용한다.

엄격 모드를 사용하기 위해서는 아래와 같이 전체 스크립트 또는 코드 블록에서 `use strict;` 라는 키워드를 작성한다.

```javascript
// 전체 스크립트 엄격 모드 구문
"use strict";
var v = "Hi!  I'm a strict mode script!";

function strict() {
  // 함수-레벨 strict mode 문법
  "use strict";
  function nested() {
    return "And so am I!";
  }
  return "Hi!  I'm a strict mode function!  " + nested();
}
```

또는 [[ES2015 (ES6)]]에서 추가된 자바스크립트 모듈을 사용하는 경우, 자동으로 엄격 모드가 적용된다.

```javascript
function strict() {
  // 모듈이기때문에 기본적으로 엄격합니다
}
export default strict;
```

# 엄격 모드 변경점

## 1. 전역 변수 생성 방지

전역 변수를 호출하듯이 키워드를 호출하는 경우, 전역 변수의 프로퍼티로 등록된다.

```javascript
function func() {
    hello = 1; // 새로운 전역 변수 hello 생성
}

func();

console.log(hello); // 1
```

만약 전역 변수를 호출하는 대신, 오타 등 실수로 새로운 전역 변수가 생성될 수 있다. 따라서 엄격 모드에서는 전역 변수를 생성하는 할당이 `Reference Error`를 발생시킨다.

```javascript
function func() {
    "use strict";
    hello = 1;
}

func(); // Uncaught ReferenceError: hello is not defined
console.log(hello); // Uncaught ReferenceError: hello is not defined
```

## 2. 쓸 수 없는(non-writable) 할당 예외

`NaN`, `undefined` 등 이미 존재하는 키워드나, 객체 프로퍼티 중 `writable` 값이 `false` 인 프로퍼티 등에 새롭게 값을 할당하는 경우, 비엄격 모드에서는 아무런 작업이 진행되지 않고 넘어간다.

```javascript
// 비엄격 모드
const obj1 = {};
Object.defineProperty(obj1, "x", { value: 42, writable: false });
obj1.x = 9; // 아무런 작업도 이루어지지 않음
```

엄격 모드에서는 이러한 할당 시도가 예외를 발생시킨다. (일반적으로 `TypeError`가 발생)

```javascript
// 엄격 모드
"use strict";
const obj1 = {};
Object.defineProperty(obj1, "x", { value: 42, writable: false });
obj1.x = 9; // Uncaught TypeError: Cannot assign to read only property 'x' of object
```

위의 예시 이외에도 `getter-only` 프로퍼티로 선언된 프로퍼티에도 접근하여 할당할 수 없다.

```javascript
"use strict";
const obj2 = { get x() { return 17; } };
obj2.x = 5; // Uncaught TypeError: Cannot set property x of #<Object> which has only a getter
```

## 3. 삭제할 수 없는 프로퍼티 삭제 예외

객체의 `configurable` 프로퍼티가 `false` 인 등, 삭제할 수 없는 프로퍼티를 삭제하는 경우 예외를 발생시킨다.

```javascript
"use strict";
delete Object.prototype; // Uncaught TypeError: Cannot delete property 'prototype' of function Object()
```

## 4. 중복된 파라미터 이름 사용 예외

함수 파라미터로 중복된 파라미터 이름을 사용할 경우 예외를 발생시킨다.

```javascript
function sum(a, a, c) {
  "use strict";
  return a + b + c; // Uncaught SyntaxError: Duplicate parameter name not allowed in this context
}
```

## 5. 8진수 구문 사용 예외

대부분의 브라우저에서는 `0644 === 420` 와 `"\045" === "%"` 와 같은 8진수 구문을 지원한다. 일반적으로 숫자 가장 앞에 `0`을 붙여 지원하는데, 이러한 방식이 개발자가 의도한데로 동작하지 않을 수 있다. 예를 들어,

```javascript
const sum =
	001 +
	010 +
	100;

console.log(sum); // 109
```

위와 같이 자릿수를 맞추기 위해 앞에 `0`을 사용할 수 있다.

개발자의 의도로는 `001`은 `1`, `010`은 `10` 을 나타내도록 작성할 수 있겠지만, 가장 앞에 0을 붙이면 8진수 구문을 따르게 된다.

```javascript
console.log(010); // 8
```

[[ES2015 (ES6)]]에서는 이를 명시적으로 `0o`라는 접두사를 붙여 8진수를 지원하지만, 여전히 위와 같은 문법을 지원하므로 실수가 발생할 수 있다.

엄격 모드에서는 이러한 8진수 문법을 사용했을 경우 구문 에러를 발생시킨다.

```javascript
"use strict";
const sum =
	001 +
	010 +
	100; // Uncaught SyntaxError: Octal literals are not allowed in strict mode.

console.log(sum);
```

## 6.  [[원시 자료형 (Primitive)]] 프로퍼티 할당 예외

비엄격 모드에서는 [[원시 자료형 (Primitive)]]에 대한 할당 작업이 발생하더라도 이를 무시했다.  엄격 모드에서는 이러한 할당 시 예외를 발생시킨다.

```javascript
"use strict";

false.true = ""; // Uncaught TypeError: Cannot create property 'true' on boolean 'false'
(14).sailing = "home"; // Uncaught TypeError: Cannot create property 'sailing' on number '14'
"with".you = "far away"; // Uncaught TypeError: Cannot create property 'you' on string 'with
```

## 7. with 문 사용 예외

엄격 모드에서는 `with` 구문을 사용할 경우 예외를 발생시킨다.

`with` 문은 다음에 나오는 객체를 스코프 체인에 추가한다. 예를 들어,

```javascript
var user = {
    name: "unikys",
    homepage: "unikys.tistory.com",
    language: "Korean"
}
with (user) {
    console.log(name === "unikys"); // 스코프 체인을 통해 user.name에 접근, true
    console.log(homepage === "unikys.tistory.com"); // 스코프 체인을 통해 user.homepage에 접근, true
    console.log(language === "Korean"); // 스코프 체인을 통해 user.language에 접근, true
    language = "javascript"; // 스코프 체인을 통해 user.language에 접근 및 할당
}
console.log(user.language === "javascript"); // true
```

위와 같이 구문 내에서 특정 객체를 쉽게 접근할 수 있도록 도와준다.

하지만 `with` 문을 사용해 변수에 접근하는 경우, 변수 접근이 모호해 질 수 있다. 예를 들어,

```javascript
var x = 17;
with (obj) {
  console.log(x); // x가 상위에 있는 변수인지, obj.x인지, 전역 변수인지 모호함
}
```

위와 같은 상황을 발생시킬 수 있다.

따라서 엄격 모드에서는 `with` 문 사용 시 구문 예외를 발생시킨다.

```javascript
"use strict";
var x = 17;
with (obj) {
  console.log(x); // Uncaught SyntaxError: Strict mode code may not include a with statement
}
```

## 8. eval 문 사용 예외

비엄격 모드의 `eval` 함수 내에서 선언한 변수를 함수 외부에서 접근할 수 있다.

```javascript
var evalX = eval("var x = 1;");
console.log(x); // 1
```

엄격 모드를 사용하는 경우 `eval` 함수 내에서 선언한 변수를 외부에서 접근할 수 없다. 또한 `eval` 함수를 호출하는 스코프 내에서 엄격 모드를 의미하는 `"use strict";` 문이 선언되어 있다면 `eval` 함수 또한 엄격 모드로 실행된다.

```javascript
"use strict";
var evalX = eval("var x = 1;");
console.log(x); // ReferenceError: x is not defined
```

## 9. 