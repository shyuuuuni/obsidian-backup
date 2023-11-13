# typeof

- 기본적으로 피연산자의 데이터 타입을 나타내는 문자열 반환
- 문자열의 결과는 기본 데이터 타입 7가지와 호스트 객체, `Function`, `object`
- 타입스크립트에서는 `typeof` 의 결과를 값에서 사용하느냐, 타입에서 사용하느냐에 따라 다르다.

```ts
interface Person {
  first: string;
  last: string;
}

const person: Person = { first: "zig", last: "song" };

function email(options: { person: Person; subject: string; body: string }) {}

const v1 = typeof person; // 값은 ‘object’ const
v2 = typeof email; // 값은 ‘function’

type T1 = typeof person; // 타입은 Person
type T2 = typeof email; // 타입은 (options: { person: Person; subject: string; body:string; }) = > void
```

## 클래스에서 typeof

- 타입스크립트 클래스는 값(함수)이면서 타입의 성격을 갖는다.
- 값에서의 typeof : `function`
- 타입에서의 typeof : 해당 클래스 명 타입