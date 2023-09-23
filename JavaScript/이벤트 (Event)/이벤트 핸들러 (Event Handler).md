> 이벤트 핸들러는 이벤트 리스너(Event Listener)라고 불리기도 한다.
# 이벤트 핸들러란?

사용자 조작이나 실행 환경에 따라 발생한 이벤트에 대해, 이를 처리하도록 등록한 함수를 이벤트 핸들러라고 한다.

대표적인 이벤트로는 사용자 조작으로 인한 마우스 클릭, 마우스 이동, 키보드 입력, 스크롤 변화 등이 있다.
# 이벤트 핸들러를 등록하는 방법

## 1. HTML 엘리먼트에 직접 등록하기

```html
<button onclick="alert('Hello, world!')">Click me!</button>
```

위와 같이 인라인 텍스트의 특정 이벤트의 어트리뷰트에 텍스트로 등록할 수 있다.

하지만 [mdn](https://developer.mozilla.org/ko/docs/Learn/JavaScript/Building_blocks/Events)에서 설명하는 것 처럼 위 방법은 추천하는 방법은 아니다. 여러개의 이벤트 핸들러를 등록해야 하는 상황에서 처리가 어렵고, HTML과 JS를 함께 작성하는 것은 코드의 가독성과 유지보수성을 떨어뜨리기 때문이다.

## 2. 이벤트 핸들러 프로퍼티 추가하기
### HTML

```html
<button onclick="alert('Hello, world!')">Click me!</button>
```
### JS

```js
const button = document.querySelector('#myButton'); button.onclick = function() { alert('Hello, world!'); };
```

위 코드처럼 HTML과 JS를 분리해서 작성할 수 있다.

하지만 위 방법은 엘리먼트 객체의 프로퍼티에 하나의 이벤트 핸들러 함수만 등록할 수 있다. 새롭게 함수를 추가하는 경우 기존 함수를 덮어쓰기 때문에 여러개의 이벤트 핸들러를 등록할 수 없다.

물론 장점도 있는데, 이벤트 핸들러 프로퍼티는 더 나은 크로스 브라우저 호환성을 가지고 있다. 예를 들어, IE8 정도로 예전의 브라우저에서도 지원되는 경우를 의미한다.

## 3. #addEventListener 메소드 사용하기 (권장)

### HTML

```html
<button onclick="alert('Hello, world!')">Click me!</button>
```
### JS

```js
const button = document.querySelector('#myButton'); button.addEventListener('click', function() { alert('Hello, world!'); });
```

HTML 엘리먼트의 `addEventListner` 메소드를 사용하여 이벤트 핸들러를 등록하면 위와 같이 HTML과 JS를 분리할 수 있다.

또한 이벤트 프로퍼티를 추가하는 방법과 다르게, `addEventListener`로 추가한 이벤트 핸들러는 중첩이 가능하다.

따라서 일반적으로 `addEventListener`를 사용해 이벤트 핸들러를 추가하는 것이 권장된다.

# 이벤트 핸들러를 제거하는 방법

## #removeEventListener 메소드 사용하기

```js
element.removeEventListener(type, listener);
element.removeEventListener(type, listener, options);
element.removeEventListener(type, listener, useCapture);
```

`removeEventListener`는 `addEventListener`로 등록한 이벤트 핸들러를 제거하는 메소드로, 이벤트 유형(type), 이벤트 핸들러 함수(listener), 그리고 옵션(options, useCapture)을 받는 메소드이다.

그 중 `options`와 `useCapture`는 `addEventListener`의 매개변수로 전달하는 그 값과 동일하다. (그 외 옵션은 무시한다.) 즉, [[이벤트 캡처링 (Event Capturing)]], [[이벤트 버블링 (Event Bubbling)]]을 지원하기 위한 옵션이 일치하는지 검사하기 위해 사용된다. 예를 들어,

```js
// 먼저 이벤트를 등록한 후,
element.addEventListener("mousedown", handleMouseDown, true);

// 이벤트를 제거해보면,
element.removeEventListener("mousedown", handleMouseDown, false); // 실패
element.removeEventListener("mousedown", handleMouseDown, true); // 성공
```

위와 같이 옵션이 동일하지 않은 경우 제거에 실패할 수 있다.

### 참고: #getEventListeners 메소드 사용하기

> 출처: [[JavaScript] 걸려 있는 클릭 이벤트 핸들러 제거하기 remove Event Listener](https://parksay-dev.tistory.com/110)

익명 함수로 이벤트 핸들러를 등록한 경우, `removeEventListener`의 매개변수 중 listener 매개변수 입력이 어려울 경우가 있다. 이 경우 `getEventListeners` 라는 함수를 사용하면 특정 객체에 있는 이벤트 핸들러를 확인할 수 있다.

하지만 `getEventListeners` 함수는 자바스크립트 공식 API가 아닌 크롬 브라우저의 개발자 도구의 콘솔을 위한 API이다. 따라서 표준 자바스크립트에서 사용할 수 없는 방식이기 때문에 사용할 수 없다.
