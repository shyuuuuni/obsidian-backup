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
- Suspense를 사용하면 각각의 컴포넌트에서 데이터 fetching 후 각자 전달 - TTFB, FCP, TTI 시간 감소

```tsx
//example
import { Suspense } from 'react'
import { PostFeed, Weather } from './Components'
 
export default function Posts() {
  return (
    <section>
      <Suspense fallback={<p>Loading feed...</p>}>
        <PostFeed />
      </Suspense>
      <Suspense fallback={<p>Loading weather...</p>}>
        <Weather />
      </Suspense>
    </section>
  )
}
```

## Next.js의 Streaming 사례

- layout.tsx로 전달되는 부분은 따로 전송이 됨
- 그 속의 페이지 데이터에서는 fetching 등이 발생할 수 있으므로 따로 청크를 나눠서 전송함

### 우선순위

1. 페이지 Loading.tsx
2. 서버 컴포넌트 Suspense
3. 클라이언트 컴포넌트 (isPending 등)

### 서버 컴포넌트 최적화

- 서버 컴포넌트 페이지에서 데이터 Fetchiing -> 렌더링 순서로 되는 경우 최적화를 진행할 수 있음
- 데이터를 사용하는 부분만 Suspense로 감싸 fetching 로직을 이관하면, 해당 부분을 제외하고는 먼저 렌더링하여 클라이언트에게 전송

```tsx
import styles from "./Home.module.css";
import TabProvider from "@/app/(afterLogin)/home/_component/TabProvider";
import Tab from "@/app/(afterLogin)/home/_component/Tab";
import PostForm from "@/app/(afterLogin)/home/_component/PostForm";
import { Suspense } from "react";
import HomeContentsSuspense from "@/app/(afterLogin)/home/_component/HomeContentsSuspense";
import Loading from "@/app/(afterLogin)/home/loading";

export default async function Home() {
  return (
    <main className={styles.main}>
      <TabProvider>
        <Tab />
        <PostForm />
        <Suspense fallback={<Loading />}>
          <HomeContentsSuspense />
        </Suspense>
      </TabProvider>
    </main>
  );
}

```

- HomeConentsSuspense에서는 데이터를 가져오고(prefetch) 그려주는 역할