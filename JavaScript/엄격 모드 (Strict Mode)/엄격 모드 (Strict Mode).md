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

## 2. 