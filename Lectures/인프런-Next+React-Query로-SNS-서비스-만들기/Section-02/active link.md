
## 언제 사용?

/home 일때에는 home 로고에 bold를, /search 일때는 search 로고에 bold를 주는 등 특정 링크 상황에 따라 무언가 설정 할 때

즉 주소랑 연동하기 위해 사용
## 어떻게 사용?

- 서버 컴포넌트 불가능 - 클라이언트 컴포넌트 only
- next의 `useSelectedLayoutSegment` 훅 사용
	- 기본적으로 라우팅의 가장 바깥을 가져옴
	- 만약 /compose/tweet 과 같이 안쪽의 라우팅도 가져와야 하는 경우 `useSelectedLayoutSegments` 훅 사용

### 예시

- 라우팅: /home -> 세그먼트: home
- 라우팅: /explore -> 세그먼트: explore
- **라우팅: /compose/tweet -> 세그먼트: compose** - 주의