
## 리액트의 로딩, 에러 처리

- 로딩 : Suspense를 통해 컴포넌트가 대기중일 경우 fallback 컴포넌트를 보여주는 기능이 있음
- 에러 : ErrorBoundary를 통해 에러 발생 시 마찬가지로 fallback 컴포넌트를 보여주는 기능이 있음

-> Next.js에서 이를 `loading.tsx`, `error.tsx`를 통해 자동으로 처리해줌

## loading

### Suspense

- 그 안의 컴포넌트가 로딩중일 때 다른 컴포넌트를 보여줄 수 있음
- 로딩중인 상황
	1. Next.js의 데이터 fetching 상황
	2. `lazy`로 감싼 컴포넌트
	3. `use` 훅으로 감싼 Promise 반환 함수가 있을 경우



## error
