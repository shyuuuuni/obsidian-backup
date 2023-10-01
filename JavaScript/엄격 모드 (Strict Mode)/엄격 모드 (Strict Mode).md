엄격 모드를 사용하면 전체 스크립트 또는 코드 블록 내에서 몇가지 시멘틱 변경이 발생한다.

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

## 3. 삭제할 수 없는 프로퍼티 삭제 예외

객체의 `configurable` 프로퍼티가 `false` 인 등, 삭제할 수 없는 프로퍼티를 삭제하는 경우 예외를 발생시킨다.

```javascript
"use strict";
delete Object.prototype; // Uncaught TypeError: Cannot delete property 'prototype' of function Object()
```

## 4. 중복된 파라미터 이름 사용 예외



