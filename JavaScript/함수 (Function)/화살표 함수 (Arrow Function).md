화살표 함수는 [[ES2015 (ES6)]]에서 추가된 새로운 함수 선언 방식으로, 전통적인 함수 표현(함수 선언식, 표현식)에 대한 대안으로 사용할 수 있다.

# 사용법

```javascript
const sum = (a, b) => {
	return a + b;
}
console.log(sum(1, 2)); // 3
```

화살표 함수는 `funciton` 키워드 없이 사용할 수 있다. 따라서 더 간결한 함수 선언이 가능하다.

매개변수가 1개라면 소괄호 `( ... )`를 생략하고 작성할 수 있다. 또한 일반 함수와 마찬가지로 나머지 매개변수와 기본 매개변수를 사용할 수 있다.

# 특징

## 1. this

### 1-1. 렉시컬 스코프 참조

일반 함수는 **함수가 어떻게 호출되는지에 따라서 `this`가 바인딩** 되었다. 예를 들어,

- 이 함수가 생성자인 경우는 새로운 객체
- [[엄격 모드 (Strict Mode)]] 함수 호출에서는 `undefined`
- 함수가 "객체 메소드"로서 호출된 경우 해당 객체
- 그 외 자세한 내용은 [[this]] 참고

하지만 이런 `this` 바인딩의 특징은 몇몇 상황에서 혼란을 발생시킬 수 있다.

```javascript
function Person() {
	// this : 생성된 Person 인스턴스 객체
	this.age = 0;

	setInterval(function growUp() {
		// this : growUp의 this = 전역 객체
		this.age++;
	}, 1000);
}
```

화살표 함수의 특징으로는 함수 자체적인 `this`를 바인딩하지 않는다는 특징을 가지고 있다. 대신 함수가 선언된 시점의  해당 지점의 렉시컬 스코프의 `this`를 사용한다.

```javascript
function Person() {
	// this : 생성된 Person 인스턴스 객체
	this.age = 0;

	setInterval(() => {
		// this : 생성된 Person 인스턴스 객체
		this.age++;
	}, 1000);
}
```

### 1-2. this 바인드 메소드

화살표 함수에 `call`, `bind`, `apply`와 같이 명시적으로 `this`를 바인딩하는 메소드를 사용하는 경우 바인딩되지 않는다.

```javascript
function normal(a, b) {
	console.log(this, a, b);
}
const arrow = (a, b) => {
	console.log(this, a, b);
}

normal.call('this', 1, 2); // String {'this'} 1 2
arrow.call('this', 1, 2); // window 1 2
```
### 1-3. [[엄격 모드 (Strict Mode)]]를 사용했을 때 주의점

엄격 모드를 사용하면 일반 함수의 경우 `this`가 `undefined`로 바인딩된다. 하지만 화살표 함수에서는 이를 무시한다.

```javascript
var f = () => {
	"use strict";
	console.log(this); // 전역 객체
}
```

## 2. arguments 키워드

화살표 함수는 `arguments` 를 자체적으로 바인딩하지 않는다.

```javascript
function normal(a, b) {
	console.log(arguments);
}
const arrow = (a, b) => {
	console.log(arguments);
}

normal(1, 2); // Arguments(2) [1, 2, callee: ƒ, Symbol(Symbol.iterator): ƒ]
arrow(1, 2); // Uncaught ReferenceError: arguments is not defined
```

## 3. 생성자 함수로 사용 불가

화살표 함수는 `new` 키워드와 함께 생성자 함수로 사용할 수 없다.

```javascript
var Foo = () => {};
var foo = new Foo(); // Uncaught TypeError: Foo is not a constructor
```

## 4. prototype 프로퍼티 사용 불가

화살표 함수는 `prototype` 프로퍼티를 가지고 있지 않는다.

```javascript
var Foo = () => {};
console.log(Foo.prototype); // undefined
```

## 5. 제너레이터로 사용 불가

화살표 함수는 제너레이터로 사용할 수 없다. 내부에 `yield` 키워드 또한 사용할 수 없다.

```javascript
const genArrowFunc = * () => {
	yield 1;
}; // SyntacError: Unexpected token '*'
```

# 사용 시 주의사항

## 1. 메소드로 사용 시

화살표 함수의 `this` 특징으로 인해 메소드로 사용 시 의도했던 동작을 하지 않을 수 있다. 일반 함수를 메소드로 호출할 경우 `this`는 호출한 객체를 참조하게 되는데, 이런 동작이 작동하지 않을 수 있다.

```javascript
const obj = {
	value: 1,
	normal() {
		console.log(this.value); // this : obj
	},
	arrow: () => {
		console.log(this.value); // this : window
	}
}

obj.normal(); // 1
obj.arrow(); // undefined
```

## 2. 객체 리터럴 반환 시

화살표 함수를 간결하게 사용하기 위해 `params => {object:literal}` 와 같이 사용 할 경우 의도하지 않게 작동할 수 있다.

객체 리터럴과 함수 본문이 차이가 발생하기 때문에, 객체 리터럴을 반환하기 위해서는 소괄호 `( ... )`로 감싸주어야 한다.

```javascript
var func = () => ({ foo: 1 });
```

## 3. 이벤트 핸들러로 사용 시

일반 함수로 [[이벤트 핸들러 (Event Handler)]]를 작성할 경우 `this`는 이벤트 핸들러가 바인딩된 `DOM` 엘리먼트를 참조한다.

하지만 화살표 함수는 `this`를 가지고 있지 않기 때문에 의도하지 않는 결과가 발생할 수 있다.

```javascript
const box = document.querySelector('.box');

box.addEventListener("click", () => {
  this.classList.toggle("opening"); // this: window
})
```

# 성능 차이

여러 아티클에서 성능을 비교했을 때 화살표 함수와 일반 함수 차이에 큰 차이가 나지 않는다고 한다. 함수를 사용하는 상황과 개발 컨벤션에 따라 선택하면 될 듯 하다.