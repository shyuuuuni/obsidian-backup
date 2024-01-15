## 캐싱 (tags)

```ts
async function getPostRecommends() {  
  const res = await fetch(`http://localhost:9090/api/postRecommends`, {  
    next: {  
      tags: ['posts', 'recommends']  
    },  
    cache: "default"  
  })  
  if (!res.ok) {  
    throw new Error('Failed to fetch PostRecommends Data');  
  }  
  return res.json();  
}
```

- 기본적으로 서버 컴포넌트의 fetch는 next가 자체적으로 캐싱
- 캐싱 옵션을 수정하기 위해서는 `cache` 프로퍼티를 통해 조절

### 캐싱 종류

```ts
// This request should be cached until manually invalidated.
// Similar to `getStaticProps`.
// `force-cache` is the default and can be omitted.
const staticData = await fetch(`https://...`, { cache: 'force-cache' })

// This request should be refetched on every request.
// Similar to `getServerSideProps`.
const dynamicData = await fetch(`https://...`, { cache: 'no-store' })

// This request should be cached with a lifetime of 10 seconds.
// Similar to `getStaticProps` with the `revalidate` option.
const revalidatedData = await fetch(`https://...`, {
	next: { revalidate: 10 },
})
```

1. force-cache: 수동으로 해제할 때 까지 캐시 유지 (default)
2. no-store: 캐시하지 않음 = 항상 새로 가져옴
3. 명시한 시간마다 새롭게 캐시 갱신
	- revalidate = 0 : 캐시하지 않음(no-cache)
	- revalidate = number : 해당 시간마다 재평가
	- revalidate = false : 무기한 캐시 (force-cache)

### 태그

> 참고: 비슷하게 revalidatePath 를 사용하면 해당 URL 하위에서 사용하는 모든 캐시를 만료시킴

- 해당 요청에 대한 태그를 둘 수 있음
- `revalidateTag` 함수를 통해 해당 캐시를 만료시킬 수 있음

```tsx
'use server'
 
import { revalidateTag } from 'next/cache'

// 서버에서 함수를 실행시킬 때 포스트를 추가하고 포스트를 불러오는 태그의 캐시를 만료
export default async function submit() {
  await addPost()
  revalidateTag('posts')
}
```

```tsx
import { NextRequest } from 'next/server'
import { revalidateTag } from 'next/cache'
 
export async function GET(request: NextRequest) {
  const tag = request.nextUrl.searchParams.get('tag');

  // 혹은 라우팅 과정에서 처리할 수 있음
  revalidateTag(tag)
  return Response.json({ revalidated: true, now: Date.now() })
}
```

