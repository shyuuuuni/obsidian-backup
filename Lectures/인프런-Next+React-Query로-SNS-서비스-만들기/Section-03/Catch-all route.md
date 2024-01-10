
> 그 사이 모든 라우트를 허용

(ex) /app/shop/[...slug]/page.tsx 라는 라우팅이 있다고 가정하면
- /app/shop/a
- /app/shop/a/b
- ...
모두 허용함

만약 옵셔널하게 사용하고 싶다면? - `[[...slug]]` 이런식으로 괄호를 두 개 사용

# 사용

아래 페이지에서 사용한다고 가정하면,

```tsx
export default function Page({ params }: { params: { slug: string } }) {
  return <h1>My Page</h1>
}
```

실제 예시 :

|Route|`params` Type Definition|
|---|---|
|`app/blog/[slug]/page.js`|`{ slug: string }`|
|`app/shop/[...slug]/page.js`|`{ slug: string[] }`|
|`app/shop/[[...slug]]/page.js`|`{ slug?: string[] }`|
|`app/[categoryId]/[itemId]/page.js`|`{ categoryId: string, itemId: string }`|