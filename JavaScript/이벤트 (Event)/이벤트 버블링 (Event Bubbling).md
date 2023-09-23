
> [!tip] 이벤트 흐름
> HTML 문서는 각 엘리먼트(Element)가 _계층적 트리 구조_ 를 이루고 있다. 이러한 형태 때문에, 자바스크립트로 등록한 이벤트의 실행 흐름 또한 계층적 구조를 따라 연쇄적으로 발생한다.

# 이벤트 버블링

자바스크립트에서 이벤트의 기본 실행 흐름을 **이벤트 버블링** 이라고 한다. 이벤트 버블링은 ==이벤트가 실제로 발생한 타겟 엘리먼트의 이벤트 핸들러가 동작한 후, HTML 문서 상에서 최상단의 엘리먼트를 만날 때 까지 부모 엘리먼트를 찾아가며 이벤트 핸들러를 동작==하는 것을 의미한다.

![[이벤트-버블링-1.png|center]]

# 이벤트 버블링을 사용하는 방법

자바스크립트는 이벤트 핸들러를 등록할 때 기본적으로 이벤트 버블링을 사용하도록 작동한다.

하지만 이벤트 핸들러를 등록하는 `addEventHandler` 함수를 호출할 때, 명시적으로 버블링을 사용하도록 지정할 수 있다.

```javascript
addEventListener(type, listener);
addEventListener(type, listener, options);
addEventListener(type, listener, useCapture);
```

`addEventListener` 함수는 위와 같이 3가지 형태로 사용이 가능하다.

그 중 첫번째는 기본적으로 버블링을 사용하는 것을 의미한다.

```javascript
my_element.addEventListener("click", eventCallback);
```

두번째는 `options` 라는 객체를 전달해 이벤트 리스너를 등록할 때 옵션을 전달할 수 있다. 옵션 중 `capture` 라는 프로퍼티를 `false`로 설정하면 이벤트 버블링을 사용할 수 있다. 기본값이`false` 이기 때문에 따로 설정하지 않아도 된다.

```javascript
my_element.addEventListener("click", eventCallback, { capture: false });
```

마지막 세번째는 `useCapture` 라는 부울 타입의 매개변수를 전달할 수 있다. `useCapture` 값을 `false`로 설정하면 이벤트 버블링을 사용할 수 있다. 기본값이`false` 이기 때문에 따로 설정하지 않아도 된다.

```javascript
my_element.addEventListener("click", eventCallback, false);
```

