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

### 1. 서버사이드

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

### 2. 클라이언트 사이드

```tsx
"use client";

import { useQuery } from "@tanstack/react-query";
import { getPostRecommends } from "@/app/(afterLogin)/home/_lib/getPostRecommends";
import Post from "@/app/(afterLogin)/_component/Post";

export default function PostRecommends() {
  const { data } = useQuery({
    queryKey: ["posts", "recommends"],
    queryFn: getPostRecommends,
  });

  return data?.map((post) => <Post key={post.postId} post={post} />);
}

```

- 위와 같이 useQuery를 사용하면 된다. (RQProvider로 감싸진 하위 컴포넌트 일 경우)

## React Query vs Redux(Redux-saga)

> 왜 리액트 쿼리를 사용했나요? 왜 SWR을 사용했나요? (면접 질문 유용할듯)

### 본질

- RQ의 핵심은 서버의 데이터를 fetching하는 것
- Redux의 핵심은 컴포넌트 간 데이터를 공유하는 것 (그 과정에서 데이터를 불러오는 것도 중요하지만, 기본적으로는 데이터 공유를 위한 것)

### 차이점

- RQ는 많은 요청에 대응하기 쉽도록 자체적으로 캐싱 기능을 제공하고 관리할 수 있게 해 줌.
	- 매번 새로운 데이터를 가져오는 것이 아님
	- ex: 포스트 - 수정되지 않는다면 매번 새롭게 가져오지 않아도 됨.
		1. 시간: 주기적으로 업데이트
		2. 액션: 수정되었을 때 캐시를 만료시키는 식으로 구현 가능
- 서버 뿐만 아니라 클라이언트(브라우저)에서도 캐시된 데이터를 가져오는 것이 더 빠른 경험을 제공할 수 있음

## fresh, stale, inactive

- 이러한 상태들은 RQ devtools를 사용하면 굉장히 편함

### fresh

### stale

### inactive

