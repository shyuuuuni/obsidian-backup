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

## route.ts

- /src/api/auth/[...nextauth]/route.ts 파일 생성 (nextauth 이름은 상관 없긴 함.)
- 해당 라우트는 클라이언트 서버에 대한 API 호출 담당
- 기본 사용법 : 해당 파일에 `GET`, `POST` 함수를 export하여 사용
- nextauth와 사용법 : 기존에 만든 auth.ts의 GET, POST를 가져와서 export -> nextauth가 이러한 처리를 모두 관리를 해 줌.

	```tsx
	export { GET, POST } from "@/auth";
	```

## 환경변수

```.env
# .env
AUTH_URL=http://localhost:9090  : API URL
AUTH_SECRET=mustkeepinsecret    : 비밀번호(유출X)
```

- AUTH_URL은 실제 백엔드 서버 URL로 설정해야 함.

## Credential 인증 (ID/PW)

```ts
import CredentialsProvider from "next-auth/providers/credentials";

export const {
  handlers: { GET, POST },
  auth,
  signIn,
} = NextAuth({
  pages: {
    signIn: "/i/flow/login",
    newUser: "/i/flow/signup",
  },
  providers: [
    CredentialsProvider({
      async authorize(credentials) {
        const authResponse = await fetch(`${process.env.AUTH_URL}/api/login`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            id: credentials?.username,
            password: credentials?.password,
          }),
        });

        // if (!authResponse.ok) {
        //   return null;
        // }
        //
        // const user = await authResponse.json();
        // console.log("user", user);
        // return {
        //   email: user.id,
        //   name: user.nickname,
        //   image: user.image,
        //   ...user,
        // };
      },
      }}),
  ],
});

```

- 기본적으로 credentials로 넘겨오는 객체 안에는 username과 password가 포함되어 있음.
- 이를 백엔드 서버 URL로 요청을 보내서 유효한 ID/PW인지 확인
- 