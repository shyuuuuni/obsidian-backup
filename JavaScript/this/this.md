# this란?

JavaScript에서 `this` 는 일반적으로 현재 코드를 실행하는 대상을 참조하는 키워드를 의미한다. 일반 함수에서 호출하는 `this`는 [[전역 객체 (Global Object)]]를 참조하고, 만약 `this`를 참조하는 함수가 객체의 메소드일 경우 호출한 객체를 참조한다.

> [!NOTE] 전역 객체 참조
> 만약 항상 [[전역 객체 (Global Object)]]를 참조하고 싶다면 `globalThis` 키워드를 통해 참조할 수 있다.

# this의 참조값

## 1. 전역 컨텍스트

전역 컨텍스트에서 `this`는 항상 [[전역 객체 (Global Object)]]를 참조한다.

```javascript
console.log(this === globalThis); // true
```

## 2. 일반 함수

일반 함수 컨텍스트에서 `this`는 호출 시 값이 결정된다. 만약 함수 내 `this`의 값을 설정하고 싶다면 [[this 바인딩]] 메소드를 사용할 수 있다.

### 2-1. 비엄격 모드

일반 함수에서 호출하는 `this`는 [[전역 객체 (Global Object)]]를 참조한다.

```javascript
function call() {
	console.log(this === globalThis);
}

call(); // true
```

### 2-2. 엄격 모드

[[엄격 모드 (Strict Mode)]]일 경우 비엄격 모드와 다르게 일반 함수 호출에서 `this`를 [[전역 객체 (Global Object)]]로 바인딩 하지 않는다.

ECMAScript 명세에 따르면,

1. `this` 값이 `null` 또는 `undefined` 일 경우, 비엄격 모드에서는 [[전역 객체 (Global Object)]]를 바인딩한다. 만약 [[엄격 모드 (Strict Mode)]]라면 바인딩하지 않는다.
2. `this` 값이 [[원시 자료형 (Primitive)]]일 경우, 비엄격 모드에서는 래퍼 클래스로 `autoboxing`을 수행한다. 만약 [[엄격 모드 (Strict Mode)]]라면 수행하지 않는다. (`this`가 [[원시 자료형 (Primitive)]]을 참조할 수 있다.)

1번 명세에 따라 [[this 바인딩]]된 값을 그대로 유지한다.

```javascript
function call() {
    "use strict";
    console.log(this);
}

call(); // undefined
```

## 3. 화살표 함수

[[화살표 함수 (Arrow Function)]] 컨텍스트 내부의 `this`는 **항상 생성 시점의 `this`를 참조**한다.

```javascript
const cat = {
  name: 'meow',
  foo1: function() {
	// foo2는 foo1 함수 내부에서 선언되어 있음
	// foo1은 객체 메소드임으로 this는 cat 객체를 참조함
	// foo2는 선언 시점에서 foo1 컨텍스트의 this인 cat 객체를 참조함
    const foo2 = () => {
      console.log(this.name);
    }
    foo2();
  }
};

cat.foo1();	// meow
```

## 4. 객체 메소드

