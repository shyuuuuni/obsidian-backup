전역 객체란 전역 컨텍스트에서 항상 존재하는 객체를 의미한다. 자바스크립트는 코드를 실행하기 전, 항상 최상위에 1개의 전역 객체를 생성하고, 다른 모든 객체는 전역 객체의 하위에 위치하도록 유지한다.

전역 객체는 자바스크립트를 실행하는 환경에 따라 다르다.

1. 브라우저에서는 `Window` 객체를 전역 객체로 가진다.
2. `Node.js` 에서는 `global` 객체를 전역 객체로 가진다.
3. `Worker` 에서는 `WorkerGlobalScope` 객체를 전역 객체로 가진다.

전역 객체는 항상 최상위에 존재하기 때문에, 전역 객체의 프로퍼티는 어디서든 참조할 수 있다. 예를 들어,

```javascript
const button = document.getElementById('btn');
```

위와 같은 `document` 와 같은 객체는 따로 선언되어 있지 않아도 어디서든 참조할 수 있다. 그 이유는,

```javascript
const button = window.document.getElementById('btn');
```

위와 같이 브라우저 환경에서 전역 객체인 `window` 객체의 프로퍼티로 `document` 객체가 등록되어 있고, 이를 명시하지 않아도 어디서든 참조할 수 있기 때문이다.

# 전역 변수 추가

전역 스코프에서 `var` 키워드로 변수를 선언할 경우 전역 객체의 프로퍼티로 등록된다.

```javascript
var v = 5;

console.log(window.v); // 5
```

주의해야 할 점은 `var` 키워드가 아닌 `let` 또는 `const` 키워드로 변수를 선언했을 경우에는 등록되지 않는다.

```javascript
let v = 5;

console.log(window.v); // undefined
```