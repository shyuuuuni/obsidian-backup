일반 함수에서 [[this]]는 호출되는 시점에, 호출 방식에 따라 결정된다.

하지만 특별한 방법으로 `this` 가 참조하는 값을 변경할 수 있는데, 이를 명시적 `this` 바인딩이라고 한다. Function 에 포함된 3가지 메소드를 사용하여 `this` 키워드가 참조할 값을 명시할 수 있다.

# 1. Function.prototype.bind()

```javascript
func.bind(thisArg[, arg1[, arg2[, ...]]])
```

- `thisArg`: 대상 함수 `func`에 지정할 `this` 값을 지정한다. 만약 생성자 함수로 호출할 경우 무시된다.
- `arg1, arg2, ...`: 대상 함수 `func`에 전달할 파라미터이다.

`bind` 메소드는 `this` 키워드와 파라미터가 지정된 새로운 함수를 반환한다.
# 2. Function.prototype.call()

```javascript
func.call(thisArg[, arg1[, arg2[, ...]]])
```

- `thisArg`: 대상 함수 `func`에 지정할 `this` 값을 지정한다. 만약 생성자 함수로 호출할 경우 무시된다.
- `arg1, arg2, ...`: 대상 함수 `func`에 전달할 파라미터이다.

`call` 메소드는 `this` 키워드와 파라미터가 지정된 함수를 호출한다.
# 3. Function.prototype.apply()

```javascript
func.apply(thisArg, [argsArray]);
```

- `thisArg`: 대상 함수 `func`에 지정할 `this` 값을 지정한다. 만약 생성자 함수로 호출할 경우 무시된다.
- `argsArray`: 대상 함수 `func`에 전달할 파라미터 배열 또는 유사 배열이다.

`apply` 메소드는 `this` 키워드와 파라미터가 지정된 함수를 호출한다. `call` 메소드와의 차이점으로는, `call` 함수는 파라미터를 여러개 받아서 전달하지만, `apply`는 하나의 파라미터 배열을 받는다는 차이점이 있다.
