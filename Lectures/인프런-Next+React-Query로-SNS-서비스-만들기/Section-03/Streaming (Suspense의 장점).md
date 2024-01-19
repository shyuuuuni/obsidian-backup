## SSR의 한계

![[Pasted image 20240120045352.png]]

SSR을 사용하면,

1. Fetcing data on server : like 서버 컴포넌트에서 prefetch하여 데이터를 가져오는 것을 대기
2. Rendering HTML on server : SSR
3. Loading code on the client : 클라이언트에게 데이터(번들, 에셋) 전달
4. Hydrating : SSR 하이드레이션

총 4 단계로 나누어서 진행이 됨.

이 때 A가 너무 길 경우 문제가 생길 수 있음. 즉 클라이언트에게 코드를 전달해서 보여주기 전에 데이터를 너무 오래 기다려야 하는 한계가 있음

## Streaming (Suspense)

![[Pasted image 20240120045750.png]]

Streaming은 페이지를 작은 청크 단위로 나누고, 해당 청크별로 로딩 -> 로딩이 완료되면 사용자에게 전달하는 방식

- 기존에는 하나의 페이지의 모든 데이터 fetching을 기다리고 한번에 전달
- Suspense를 사용하면 각각의 컴포넌트에서 데이터 fetching 후 각자 전달 - TTFB, 
