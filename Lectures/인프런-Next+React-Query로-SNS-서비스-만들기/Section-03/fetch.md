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
	- 캐싱을 할 때 이를 tags를 기준으로 캐싱함
- 캐싱 옵션을 수정하기 위해서는 `cache` 프로퍼티를 통해 조절