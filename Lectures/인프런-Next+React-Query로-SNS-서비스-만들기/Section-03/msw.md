## PathParams

- next.js의 slug와 같은 역할로, 바뀔 수 있는 자리
- 콜백 함수의 파라미터의 `params`로 접근할 수 있음

```ts
http.get("/api/search/:tag", ({ request, params }) => {
    const { tag } = params;
    return HttpResponse.json([
      {
        postId: 1,
        user: user[0],
        content: `${1} 검색결과 ${tag}`,
        images: [{ imageId: 1, link: faker.image.urlLoremFlickr() }],
        createdAt: generateDate(),
      },
      {
        postId: 2,
        user: user[0],
        content: `${2} 검색결과 ${tag}`,
        images: [{ imageId: 1, link: faker.image.urlLoremFlickr() }],
        createdAt: generateDate(),
      },
      {
        postId: 3,
        user: user[0],
        content: `${3} 검색결과 ${tag}`,
        images: [{ imageId: 1, link: faker.image.urlLoremFlickr() }],
        createdAt: generateDate(),
      },
      {
        postId: 4,
        user: user[0],
        content: `${4} 검색결과 ${tag}`,
        images: [{ imageId: 1, link: faker.image.urlLoremFlickr() }],
        createdAt: generateDate(),
      },
      {
        postId: 5,
        user: user[0],
        content: `${5} 검색결과 ${tag}`,
        images: [{ imageId: 1, link: faker.image.urlLoremFlickr() }],
        createdAt: generateDate(),
      },
    ]);
  })
```

## 에러 반환하기

- ex: 404와 같은 에러를 반환하는 방법
- httpResponse.json의 첫번째 body에는 데이터, 두번째에는 상태를 전달할 수 있음

```ts
  http.get("/api/users/:userId", ({ request, params }): StrictResponse<any> => {
    const { userId } = params;
    const found = user.find((v) => v.id === userId);
    if (found) {
      return HttpResponse.json(found);
    }
    return HttpResponse.json(
      { message: "no_such_user" },
      {
        status: 404,
      },
    );
  }),
```