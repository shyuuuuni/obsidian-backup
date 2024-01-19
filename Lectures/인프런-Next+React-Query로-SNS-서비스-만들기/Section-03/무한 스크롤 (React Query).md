

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
}: QueryFunctionContext<QueryKey, string>): Promise<Post[]> {
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

```tsx
export default async function Home() {
  const queryClient = new QueryClient();
  await queryClient.prefetchInfiniteQuery({
    queryKey: ["posts", "recommends"],
    queryFn: getPostRecommends,
    initialPageParam: 0, // cursor 기본값
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