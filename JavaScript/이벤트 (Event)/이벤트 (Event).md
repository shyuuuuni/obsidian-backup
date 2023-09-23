`Event`는 DOM 엘리먼트에서 발생하는 이벤트를 나타내는 일종의 자바스크립트 인터페이스다. `Event` 라는 최상위 인터페이스에서 파생되어 `InputEvent`, `KeyboardEvent` 등 특정 이벤트를 나타내는 인터페이스가 존재한다.

이벤트는 마우스 클릭, 키보드 입력과 같은 사용자와의 상호작용을 통해 발생할 수 있다. 또는 비동기 호출 등 API 의 작업에 의해 생성될 수 있다.

다른 방법으로, 동적으로 이벤트를 발생시키기 위해 `HTMLElement.click()` 과 같은 이벤트를 발생시키는 메소드를 호출하거나, `EventTarget.dispatchEvent()`와 같이 이벤트를 송신할 수 있다.

이러한 이벤트는 일반적으로 객체의 형태로 이벤트 핸들러의 매개변수로 전달되어 이벤트 핸들러에서 로직을 처리하는데 사용할 수 있다.

```js
// 예를 들어,
function eventHandler(event) {
  if (event.type == "fullscreenchange") {
    /* 전체화면 여부 변화 처리 */
  } /* fullscreenerror */ else {
    /* 전체화면 오류 처리 */
  }
}
```

# 이벤트 타겟

이벤트 객체의 프로퍼티 중 `Event.target`과 `Event.currentTarget` 이라는 프로퍼티를 통해 이벤트와 관련된 엘리먼트를 확인할 수 있다.

## 1. Event.target

`Event.target`은 이벤트가 실제로 발생하는 대상 엘리먼트를 가리킨다. `Event.target`은 [[이벤트 버블링 (Event Bubbling)]] 단계에서 실제로 이벤트 핸들러가 등록된 엘리먼트와 다를 수 있다.

예를 들어, 아래와 같이 이벤트 위임을 구현할 수 있다.

```js
// 목록 생성
const ul = document.createElement("ul");
document.body.appendChild(ul);

const li1 = document.createElement("li");
const li2 = document.createElement("li");
ul.appendChild(li1);
ul.appendChild(li2);

function hide(evt) {
  // e.target은 사용자가 클릭한 <li> 요소를 가리킴
  // 여기서 e.currentTarget은 부모인 <ul>을 가리킬 것
  evt.target.style.visibility = "hidden";
}

// 목록에 수신기 부착
// 각각의 <li>를 클릭할 때 호출됨
ul.addEventListener("click", hide, false);
```

이 경우 `ul` 엘리먼트 하나에 이벤트 핸들러를 등록한다. 그리고 이벤트 핸들러인 `hide` 함수는 내부적으로 `evt.target` 이라는 프로퍼티 접근을 통해, 실제로 클릭 이벤트가 발생한 `ul` 엘리먼트 하위의 `li` 엘리먼트를 식별하여 스타일을 변경한다.

## 2. Event.currentTarget

반면 `Event.currentTarget`은 발생한 이번트의 이벤트 핸들러가 연결된 엘리먼트를 참조하는 읽기 전용 프로퍼티이다.

즉, 이벤트가 바인딩 된 DOM 엘리먼트를 의미한다.

> 참고: 함수 선언식으로 작성한 이벤트 핸들러의 경우 `Event.currentTarget`과 `this` 값이 동일한 값을 가진다.

# 이벤트 기본 동작 방지

## Event.preventDefault()

HTML 엘리먼트 중 특정 이벤트가 발생했을 때 처리하는 로직을 가지고 있는 경우가 있다. 예를 들어, `form` 태그 내에 있는 `input` 값을 `submit` 하고 새로고침 하는 등의 동작이 있다.

하지만 이벤트 핸들러를 작성하는 경우 이러한 동작이 불필요할 수 있다.

이를 방지하기 위해 `Event.preventDefault` 메소드를 사용할 수 있다. 현재 발생한 이벤트의 기본 동작을 취소할 수 있다.

## 주의해야 할 점

`Event.cancelable`이 `true` 값을 가지고 있는 경우에만 취소할 수 있다. 기본적으로 대부분의 사용자 상호작용으로 인한 페이지 이벤트는 취소가 가능하다.

따라서 `Event.preventDefault` 메소드를 통해 이벤트 기본 동작을 취소하고 싶은 경우 아래와 같이 취소 가능 여부를 확인하는 것을 권장하고 있다.

```js
function preventScrollWheel(event) {
  if (typeof event.cancelable !== "boolean" || event.cancelable) {
    // 이벤트를 취소할 수 있으므로 취소함
    event.preventDefault();
  } else {
    // 이벤트를 취소할 수 없음
    // preventDefault() 호출이 안전하지 않음
    console.warn(`The following event couldn't be canceled:`);
    console.dir(event);
  }
}

document.addEventListener("wheel", preventScrollWheel);
```
# 이벤트 전파 막기

## 1. Event.stopPropagation()

`Event.stopPropagation` 메소드를 사용하면, 이벤트 전파 과정에서 이벤트가 다음 흐름으로 넘어가지 않도록 한다.

즉, [[이벤트 버블링 (Event Bubbling)]]과 [[이벤트 캡처링 (Event Capturing)]]단계에서 더이상 이벤트가 전파되지 않도록 하는 것을 의미한다.

예를 들어,

```html
<ul onclick="ClickUl()">
	<li>
        <div onclick="ClickDiv()"></div>
    </li>
</ul>
```

위와 같이 구성된 HTML에서 `div` 엘리먼트를 클릭 할 경우 `ClickDiv` 이벤트 핸들러가 실행된 후, 이벤트 버블링에 의해 `ClickUl` 이벤트 핸들러가 실행될 것이다.

하지만 `ClickDiv` 이벤트 핸들러 내에서 `Event.stopPropagation` 메소드를 호출한다면 이벤트 버블링 과정을 방지하여 `ClickUl` 이벤트 핸들러 호출을 막을 수 있다.

## 2. Event.stopImmediatePropagation()

자바스크립트의 #addEventListener 메소드를 사용하면 하나의 엘리먼트에 여러개의 이벤트 핸들러를 등록할 수 있다. 그 중에서 하나의 엘리먼트에 동일한 타입(예를 들어, `click`)의 이벤트를 여러개 부착할 수도 있다.

`Event.stopPropagation` 메소드를 사용하면 다음 흐름으로의 이벤트 전파를 막을 뿐, 현재 엘리먼트에 등록되어 있는 이벤트 핸들러 실행을 막을 수 없다.

이 경우 `Event.stopImmediatePropagation` 메소드를 사용하면 다음 흐름으로의 전파 뿐만 아니라 현재 엘리먼트에 등록되어 있는 동일한 이벤트 호출 또한 막을 수 있다.

예를 들어,

```js
const div = document.getElementById('div')
const span = document.getElementById('span')
const button = document.getElementById('button')

div.addEventListener("click", () => console.log("div click"))
span.addEventListener("click", () => console.log("span click"))
button.addEventListener('click', (e) => {
  e.stopImmediatePropagation() // stopImmediatePropagation()으로 이벤트 전파 방지
  console.log('button click 1')
})
button.addEventListener("click", () => console.log("button click 2"))
```

위와 같이 `div-span-button` 순으로 DOM 엘리먼트가 구성되어 있고, 이벤트 핸들러가 등록되어 있는 경우를 살펴보자.

`Event.stopImmediatePropagation`이 없다고 가정하면, `button`을 클릭했을 경우 [[이벤트 버블링 (Event Bubbling)]]에 의해 `button-span-div`의 순서대로 이벤트 핸들러가 실행되었을 것이다.

하지만 `Event.stopImmediatePropagation` 메소드가 호출되어 `button`이 클릭되더라도 최종적으로 `button click 1` 이라는 콘솔 결과만 출력되고 이후 모든 이벤트 핸들러 실행을 막을 수 있다.

만약 `Event.stopImmediatePropagation`이 아닌 `Event.stopPropagation`을 사용했다면 `button click 1` 뿐만 아니라 `button click 2` 라는 텍스트도 함께  출력되었을 것이다