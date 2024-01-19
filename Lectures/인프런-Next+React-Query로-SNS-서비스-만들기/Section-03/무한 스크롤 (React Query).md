

## msw 설정

- API 호출 시 cursor (마지막으로 호출한 번호)를 같이 전달
- cursor는 일반적으로 queryString에 전달함. -> `URL(request.url).searchParams` 사용

```ts
  http.get('/api/postRecommends', ({ request }) => {
    const url = new URL(request.url)
    const cursor = parseInt(url.searchParams.get('cursor') as string) || 0
    return HttpResponse.json(
      [
        {
          postId: cursor + 1,
          User: User[0],
          content: `${cursor + 1} Z.com is so marvelous. I'm gonna buy that.`,
          Images: [{imageId: 1, link: faker.image.urlLoremFlickr()}],
          createdAt: generateDate(),
        },
        {
          postId: cursor + 2,
          User: User[0],
          content: `${cursor + 2} Z.com is so marvelous. I'm gonna buy that.`,
          Images: [
            {imageId: 1, link: faker.image.urlLoremFlickr()},
            {imageId: 2, link: faker.image.urlLoremFlickr()},
          ],
          createdAt: generateDate(),
        },
        {
          postId: cursor + 3,
          User: User[0],
          content: `${cursor + 3} Z.com is so marvelous. I'm gonna buy that.`,
          Images: [],
          createdAt: generateDate(),
        },
        {
          postId: cursor + 4,
          User: User[0],
          content: `${cursor + 4} Z.com is so marvelous. I'm gonna buy that.`,
          Images: [
            {imageId: 1, link: faker.image.urlLoremFlickr()},
            {imageId: 2, link: faker.image.urlLoremFlickr()},
            {imageId: 3, link: faker.image.urlLoremFlickr()},
            {imageId: 4, link: faker.image.urlLoremFlickr()},
          ],
          createdAt: generateDate(),
        },
        {
          postId: cursor + 5,
          User: User[0],
          content: `${cursor + 5} Z.com is so marvelous. I'm gonna buy that.`,
          Images: [
            {imageId: 1, link: faker.image.urlLoremFlickr()},
            {imageId: 2, link: faker.image.urlLoremFlickr()},
            {imageId: 3, link: faker.image.urlLoremFlickr()},
          ],
          createdAt: generateDate(),
        },
      ]
    )
  }),
```


## 쿼리 함수 설정

- 호출하는 쿼리 함수에는 `pageParam`을 자동으로 전달해준다.
- 타입스크립트에서는 `QueryFunctionContext<쿼리키타입, pageParam타입>`을 전달한다.
- 이후 쿼리스트링으로 pageParam을 전달한다.

```ts
import { Post } from "@/models/Post";
import { QueryFunctionContext } from "@tanstack/react-query";

type QueryKey = [_1: string, _2: string];

export async function getPostRecommends({
  pageParam,
}: QueryFunctionContext<QueryKey, number>): Promise<Post[]> {
  const res = await fetch(
    `http://localhost:9090/api/postRecommends?cursor=${pageParam}`,
    {
      next: {
        tags: ["posts", "recommends"],
      },
    },
  );

  if (!res.ok) {
    throw new Error("Failed to fetch PostRecommends Data");
  }

  return res.json();
}

```

## 서버 컴포넌트 설정

- prefetchInfiniteQuery 사용
- initialPageParams를 필수로 적용해야 함
- 여기서 prefetchInfiniteQuery의 제네릭에 타입을 잘 넣어줘야 에러가 뜨지 않음
	- `데이터 타입, 에러 타입, InfiniteData<데이터타입>, 쿼리키타입, pageParam 타입`

```tsx
export default async function Home() {
  const queryClient = new QueryClient();
  await queryClient.prefetchInfiniteQuery<
    IPost[],
    Object,
    InfiniteData<IPost[]>,
    [_1: string, _2: string],
    number
  >({
    queryKey: ["posts", "recommends"],
    queryFn: getPostRecommends,
    initialPageParam: 0, // lastPage = [1~5 포스트], 그 다음 = [6~10 포스트], ...
  });
  const dehydratedState = dehydrate(queryClient);

  return (
    <main className={styles.main}>
      <HydrationBoundary state={dehydratedState}>
        <TabProvider>
          <Tab />
          <PostForm />
          <HomeContents />
        </TabProvider>
      </HydrationBoundary>
    </main>
  );
}

```

## 클라이언트 컴포넌트 설정

- 클라이언트에서는 `useInfiniteQuery`를 사용한다.
	- 제네릭은 서버와 동일
- 마찬가지로 initialPageParams를 필수로 적용해야 함
- 또한 다음 페이지를 계산하는 `getNextPageParam`도 추가해야 함.

```tsx
"use client";

import { InfiniteData, useInfiniteQuery } from "@tanstack/react-query";
import { getPostRecommends } from "@/app/(afterLogin)/home/_lib/getPostRecommends";
import Post from "@/app/(afterLogin)/_component/Post";
import { Post as IPost } from "@/models/Post";
import { useInView } from "react-intersection-observer";
import { useEffect } from "react";

export default function PostRecommends() {
  const { data, fetchNextPage, hasNextPage, isFetching } = useInfiniteQuery<
    IPost[],
    Object,
    InfiniteData<IPost[]>,
    [_1: string, _2: string],
    number
  >({
    queryKey: ["posts", "recommends"],
    queryFn: getPostRecommends,
    initialPageParam: 0, // lastPage = [1~5 포스트], 그 다음 = [6~10 포스트], ...
    getNextPageParam: (lastPage) => lastPage.at(-1)?.postId ?? 0, // 이전 페이지의 마지막 페이지 번호
    staleTime: 60 * 1000,
    gcTime: 300 * 1000,
  });
  const { ref, inView } = useInView({
    threshold: 0,
    delay: 0,
  });

  // 스크롤 바닥에 닿았을 때 작동
  useEffect(() => {
    if (inView && hasNextPage && !isFetching) {
      fetchNextPage();
    }
  }, [inView, fetchNextPage, hasNextPage, isFetching]);

  return (
    <>
      {data?.pages
        .flatMap((page) => [...page])
        .map((post) => <Post key={post.postId} post={post} />)}
      <div ref={ref} style={{ height: "50px" }} />
    </>
  );
}

```

### intersection observer

- 스크롤을 끝까지 내렸을 때 다음 페이지를 불러올 수 있도록 제공하는 라이브러리
	- threshold: 얼마나 보였을 때 발동할지 (0: 즉시, 1: 100%)
	- delay: 몇초 뒤 발동할지
- `useEffect`를 통해 `inView`상태일 때 다음 페이지를 불러오는 `fetchNextPage` 호출
