- Next, Svelte, Solid 모두 지원
- Provider -> Naver, Kakao 등 소셜 로그인 쉽게 가능
- 기본적인 ID/PW (CredentialsProvider) 제공

---

## 설치

```zsh
npm install next-auth @auth/core
```

## 미들웨어

> request가 완료되기 전에 실행되는 코드.
> @see: https://nextjs.org/docs/app/building-your-application/routing/middleware

- 특정 상황(로그인 여부)을 만족해야만 어디에 접근할 수 있게 하는 등의 기능 작성 가능
- app router에서만 사용 가능

```ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
 
// This function can be marked `async` if using `await` inside
export function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url))
}
 
// See "Matching Paths" below to learn more
export const config = {
  matcher: '/about/:path*',
}
```

- middleware 함수 : matcher에 걸렸을 때 실행할 함수
- config - matcher : URL 패턴

## 로그인 미들웨어

```ts
// auth.ts
import NextAuth from "next-auth";

export const {
  handlers: { GET, POST },
  auth,
  signIn,
} = NextAuth({});

```

```ts
// middleware.ts
export { auth as middleware } from "./auth";

// 인증이 필요한 라우팅
export const config = {
  matcher: ["/compose/tweet", "/home", "/explore", "/messages", "/search"],
};

```

- middleware.ts에서 middleware 함수를 NextAuth()를 호출한 auth.ts의 함수를 가져와서 사용함