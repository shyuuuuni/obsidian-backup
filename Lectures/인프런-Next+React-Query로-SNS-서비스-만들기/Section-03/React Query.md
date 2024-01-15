## 설치

```zsh
npm i @tanstack/react-query@5 
npm i -D @tanstack/react-query-devtools@5
```

- 최신 버전(v5)과 이전 버전 차이가 꽤 나기 때문에 주의
- RQ의 장점인 개발도구를 함께 설치

## 세팅

```tsx
// RQProvider.tsx
"use client";

import { useState } from "react";  
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";  
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";  
  
interface Props {  
  children: React.ReactNode;  
}  
  
export default function RQProvider({ children }: Props) {  
  const [client, setClient] = useState(  
    new QueryClient({  
      defaultOptions: {  
        queries: {  
          refetchOnWindowFocus: false,  
          retry: false,  
        },  
      },  
    }),  
  );  
  
  return (  
    <QueryClientProvider client={client}>  
      {children}  
      <ReactQueryDevtools  
        initialIsOpen={process.env.NEXT_PUBLIC_MODE === "local"}  
      />  
    </QueryClientProvider>  
  );  
}
```

- 리액트 쿼리 클라이언트를 전달하는 Provider로 감쌀 수 있도록 컴포넌트 작성
- devtools는 개발 단계에서만 사용하므로 환경 변수로 표시할 수 있도록 설정
- 위 설정은 기본적으로 전역으로 사용됨
- 이를 RQ를 사용하는 곳에서 감싸줌
	- TIP: 전역 레이아웃을 감싸지 않아도 된다. 전역으로 관리하는 것은 범위가 좁을수록 좋음. 불필요하게 감쌀 필요는 없음.
- 클라이언트 컴포넌트임에 주의

## 서버 컴포넌트 - 클라이언트 컴포넌트 hydration (prefetchQuery, dehydrated)

> 참고: https://tanstack.com/query/latest/docs/react/guides/prefetching, https://tanstack.com/query/latest/docs/react/reference/hydration#dehydrate

```tsx
import styles from "./Home.module.css";
import TabProvider from "@/app/(afterLogin)/home/_component/TabProvider";
import Tab from "@/app/(afterLogin)/home/_component/Tab";
import PostForm from "@/app/(afterLogin)/home/_component/PostForm";
import Post from "@/app/(afterLogin)/_component/Post";
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from "@tanstack/react-query";

async function getPostRecommends() {
	// 데이터 가져오기 로직
}

export default async function Home() {
  const queryClient = new QueryClient(); // 1
  await queryClient.prefetchQuery({      // 2
    queryKey: ["posts", "recommends"],
    queryFn: getPostRecommends,
  });
  const dehydratedState = dehydrate(queryClient); // 3

  return (
    <main className={styles.main}>
      <HydrationBoundary state={dehydratedState}> {/* 4 */}
        {/* 다른 컴포넌트 */}
      </HydrationBoundary>
    </main>
  );
}

```

1. 서버 컴포넌트(Home)에서 새로운 쿼리 클라이언트 생성
2. `prefetchQuery` 를 통해 서버에서 초기에 보여줄 데이터를 fetching (무한 스크롤 - prefetchInfiniteQuery 사용)
3. 2번을 await를 통해 작업 완료를 기다린 후, dehydrate 함수를 통해 쿼리 클라이언트를 감싼다
	- `dehydrate`: 나중에 hydration 되거나 캐시될 수 있는 고정된 값으로 변환 (DehydratedState)
4. HydrationBoundary의 state props로 해당 상태를 전달
