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

> 캐싱

- RQ는 많은 요청에 대응하기 쉽도록 자체적으로 캐싱 기능을 제공하고 관리할 수 있게 해 줌.
	- 매번 새로운 데이터를 가져오는 것이 아님
	- ex: 포스트 - 수정되지 않는다면 매번 새롭게 가져오지 않아도 됨.
		1. 시간: 주기적으로 업데이트
		2. 액션: 수정되었을 때 캐시를 만료시키는 식으로 구현 가능
- 서버 뿐만 아니라 클라이언트(브라우저)에서도 캐시된 데이터를 가져오는 것이 더 빠른 경험을 제공할 수 있음
- 반면 Redux와 같은 상태관리 툴은 이러한 캐싱 부분에서 약점을 가지고 있음. 반면 RQ는 강력한 캐싱 기능과 함께 기존의 상태관리처럼 공유하여 사용하기 쉬운 기능을 제공함.
	- 또한 Redux는 최근에 사용하기에는 보일러플레이트도 많고 무거움 - zustand와 같은 가볍고 쉬운 라이브러리 사용

> 인터페이스 표준화

- 일반적으로 데이터를 가져오는 상태 = `가져오는 중(loading) + 성공 + 실패`
	- RQ에서는 기본적으로 위와 같은 인터페이스로 표준화해서 제공
	- 만약 다른 툴로 개발하려고 하면 해당 상태들을 따로 관리해야 하고, 개발력이 저하될 수 있음

## 캐시 상태 (fresh, stale, inactive)

- 이러한 상태들은 RQ devtools를 사용하면 굉장히 편함

### fresh

- 기본적으로 서버에서 데이터를 불러온 상태
- 최신 데이터
- 언제까지 fresh 일 지 확신할 수 없음 - 개발자가 설정해 줄 수 있음
- **RQ에서는 모든 데이터는 fresh가 아니라고 가정함**

- staleTime : fresh 상태로 유지할 시간 (기본값 = 0, 즉시 stale로 변경)
	- Infinity로 했을 경우 항상 fresh (다시는 새로 가져오지 않음)
### stale

- **기회**가 되면 데이터를 새로 불러와야 하는 상태
	- **refetchOnWindowFocus** : 탭을 전환해서 돌아올 경우 다시 불러옴
	- **refetchOnMount** : 컴포넌트 언마운트 후 새로 마운트 되는 경우 - 페이지 이동, state 변경 등등
	- **refetchOnReconnect** : 인터넷 연결이 끊겼다가 새로 접속이 되는 순간 가져옴
		- 추가 - **retry** : 데이터 가져오기에 실패했을 때 몇 번 재시도 할 지?
- RQ의 기본 데이터 상태 (데이터를 받아오면 바로 fresh-stale로 변경됨)

### inactive

- 현재 보고 있는 화면에서 사용하지 않는 데이터 상태
	- ex: 홈에서 post 캐시를 사용하다가 다른데로 이동했을 때 post가 inactive로 이동
- fresh 상태일 경우 (다시 가져올 필요가 없으니) 로딩 없이 바로 가져올 수 있음
- cacheTime(gcTime) : 사용하지 않는 데이터(캐싱)를 정리하는 시간 (기본값 = 5분)
	- 해당 쿼리에 대한 active가 없는 경우 = 즉, inactive인 상태로 변경이 되면 해당 시간 이후 메모리에서 삭제
	- **gcTime은 staleTime보다 길어야 함**
		- 만약 st=5, gc=3 일 경우, (1) inactive가 된 경우 3분 뒤 캐시가 사라짐 - (2) st는 5분 동안은 fresh임을 보장하는데, 3분 뒤에는 가져올 수 없음 - (3) 의미가 없어짐
-
### fetching

- 데이터를 가져오는 상태 (순간적)

### paused

- 데이터를 가져오다 중단된 상태
- ex: 인터넷이 잠깐 끊겼을 때

## 쿼리 조작 (refetch, invalidate, reset)

### refetch

- 새롭게 데이터를 가져오는 것
- **무조건 새로 가져옴**
- 언제 사용? : 화면에 보이지 않더라도 필요한 경우 (전역적으로 사용되는 데이터 등)

### invalidate

- 어떤 데이터를 invalidate 하면, 더 이상 해당 데이터를 사용하지 않겠다는 것을 의미함
- 이후 새롭게 데이터를 가져옴 (= refetch와 비슷한 동작)
- 즉시 데이터를 새로 가져오지는 않음
	- Observer (현재 페이지에서 해당 데이터를 사용하고 있는 수) 가 0이 되면 (= inactive 상태) 대기하다가, 다시 해당 데이터를 사용하는 곳으로 이동하면 (Observer = 1 이상) 그 때 데이터를 새롭게 가져옴
- 언제 사용? : 비교적 효율적으로 데이터를 다시 가져올 때 (즉시 가져오는 것이 아니라, 필요할 때 가져와야 할 때)

### reset



