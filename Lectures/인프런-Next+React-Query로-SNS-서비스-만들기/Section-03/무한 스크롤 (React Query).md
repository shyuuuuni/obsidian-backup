

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