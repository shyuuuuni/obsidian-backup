
## 서버 컴포넌트

- 기본적으로 app router에서는 모두 서버 컴포넌트
- 비동기로도 작동 가능
- 서버에서 작동
- 내부에 클라이언트 컴포넌트 import 가능

- hooks 사용 불가

## 클라이언트 컴포넌트

- hooks 사용 가능
- 가장 상단에 "use client"; 를 쓰면 됨

- 내부에 서버 컴포넌트 import 불가능 (할 경우 서버 컴포넌트가 됨.)