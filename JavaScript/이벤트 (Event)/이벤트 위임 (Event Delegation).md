# 이벤트 위임이란?

이벤트 위임이란 개별 엘리먼트마다 이벤트 핸들러를 부착하는 대신, 해당 엘리먼트들의 상위 엘리먼트에만 이벤트 핸들러를 부착하는 기법을 의미한다.

이벤트 위임은 [[이벤트 버블링 (Event Bubbling)]]을 활용한 방식으로, 자식 엘리먼트에서 이벤트가 발생했을 경우, 해당 이벤트의 흐름이 상위 엘리먼트로 전파되는 것을 활용한 방식이다.

예를 들어, 리스트 항목을 나타내는 `li` 엘리먼트 모두에 이벤트 핸들러를 부착하는 대신, 해당 엘리먼트들을 감싸고 있는 `ul` 엘리먼트에 하나의 이벤트 핸들러를 부착하고, 이벤트 핸들러에서 이벤트가 발생한 대상을 식별하여 조건문을 통해 로직을 처리하도록 구현할 수 있다.
# 이벤트 위임의 장점

## 1. 메모리 효율성

자바스크립트에서 이벤트 핸들러를 등록하는 것은, 이벤트 핸들러 함수라는 객체를 메모리에 적재하고 있는 것을 의미한다.

따라서 여러개의 이벤트 핸들러를 사용하는 것 보다 하나의 이벤트 핸들러를 사용하는 이벤트 위임이 더 적은 메모리를 사용하는 이점이 있다.

## 2. 확장 편의성

동적으로 이벤트를 처리해야 하는 엘리먼트가 추가되고, 삭제되는 경우 각각 이벤트 핸들러를 부착하고, 제거하는 처리가 필요하다.

이 과정에서 이벤트 위임을 사용한다면 이벤트 핸들러 추가/삭제에 대한 리소스를 절약할 수 있고, 이벤트 핸들러를 삭제하지 않아 메모리 누수가 발생하는 잠재적인 위협을 막을 수 있다.

또한 이벤트 핸들러를 한 곳에서만 관리하기 때문에 관리가 수월하고, 결과적으로 종합하여 확장 편의성을 얻을 수 있다.

# 이벤트 위임 시 주의해야 할 점

# 1. 이벤트 타겟 결정

이벤트 위임은 상위 엘리먼트 한 곳에서 이벤트 핸들러를 통해 어떤 하위 엘리먼트의 이벤트인지 판단하고 로직을 처리해야 한다.

이 경우 이벤트가 어떤 하위 엘리먼트에서 발생했는지 판단해야 한다. 예를 들어 이벤트 객체의 `event.target`과 같은 프로퍼티를 사용할 수 있다.

하지만 이러한 방식이 명확히 이벤트 타겟을 결정하고 있는지 주의해서 확인해야 한다. 만약 잘못된 이벤트 타겟을 결정하게 된다면 이벤트 핸들러는 예기치 못한 동작을 발생시킬 수 있다.


