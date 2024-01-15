- Next, Svelte, Solid 모두 지원
- Provider -> Naver, Kakao 등 소셜 로그인 쉽게 가능
- 기본적인 ID/PW (CredentialsProvider) 제공

---

## 설치

- app router에서 사용하기 위해서는 beta(5버전) 사용 필요.

```zsh
npm install next-auth@beta @auth/core
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

## 회원가입/로그인 페이지 설정

- 기본적으로 `http://localhost:3000/api/auth/signin` 와 같은 URL에 접속하면 기본으로 제공하는 로그인 창이 뜸
	![[Pasted image 20240114224431.png]]
- 수동으로 회원가입/로그인 페이지를 설정하고 싶다면 `auth.ts`의 NextAuth 옵션에 `pages` 프로퍼티를 선언하고, 그 안에 `signIn`, `newUser` 페이지를 추가

## Credential 인증 (ID/PW)

```ts
// auth.ts
import NextAuth from "next-auth";
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
            id: credentials.username,
            password: credentials.password,
          }),
        });

        if (!authResponse.ok) {
          return null;
        }

        const user = await authResponse.json();
        console.log("user", user);
        return {
          email: user.id,
          name: user.nickname,
          image: user.image,
          ...user,
        };
      },
    }),
  ],
});


```

- 기본적으로 credentials로 넘겨오는 객체 안에는 username과 password가 포함되어 있음.
- 이를 백엔드 서버 URL로 요청을 보내서 유효한 ID/PW인지 확인

반환하는 값은 User 타입으로, 아래와 같은 타입을 만족해야 함.

```ts
export interface DefaultUser {  
  id: string  
  name?: string | null  
  email?: string | null  
  image?: string | null  
}
export interface User extends DefaultUser {}

```

참고로 위 유저 타입 이외에는 값을 반환해도 참조할 수 없음.

## 로그인 방법

### 클라이언트 사이드

- `next-auth/react` 의 `signIn` 함수를 사용해야 함
- 첫번째에는 provider를 명시
- 두번째에는 옵션을 명시
	- redirect: 성공 시 페이지 이동을 의미하는데, 리액트 라우터를 통해 따로 하는것도 괜찮음

```ts
  const onSubmit: FormEventHandler<HTMLFormElement> = async (e) => {
    e.preventDefault();
    setMessage("");
    try {
      await signIn("credentials", {
        username: id,
        password,
        redirect: false,
      });
    } catch (e) {
      console.error(e);
      setMessage("로그인에 실패했습니다.");
    }
  };
```

### 서버 사이드

- 서버 사이드에서는 사용자가 만든 서버 액션 로그인 함수에서 nextauth의 기능을 사용하면 된다.

```ts
const signup = async (prevState: State, formData: FormData): Promise<State> => {  
  if (!formData.get("id") || !(formData.get("id") as string)?.trim()) {  
    return { message: "no_id" };  
  }  
  if (!formData.get("name") || !(formData.get("name") as string)?.trim()) {  
    return { message: "no_name" };  
  }  
  if (  
    !formData.get("password") ||  
    !(formData.get("password") as string)?.trim()  
  ) {  
    return { message: "no_password" };  
  }  
  if (!formData.get("image")) {  
    return { message: "no_image" };  
  }  
  let shouldRedirect = false;  
  try {  
    console.log("url: ", `${process.env.NEXT_PUBLIC_BASE_URL}/api/users`);  
    const response = await fetch(  
      `${process.env.NEXT_PUBLIC_BASE_URL}/api/users`,  
      {  
        method: "post",  
        body: formData,  
        credentials: "include",  
      },  
    );  
    console.log(response.status);  
    if (response.status === 403) {  
      return { message: "user_exists" };  
    }  
    console.log(await response.json());  
    shouldRedirect = true;  

	// 이 부분 주목: signIn은 auth.ts에서 export한 next-auth의 기능임
    // 회원 가입 성공 후 로그인까지 진행  
    await signIn("credentials", {  
      username: formData.get("id"),  
      password: formData.get("password"),  
      redirect: false,  
    });  
  } catch (err) {  
    console.error(err);  
    return { message: "error" };  
  }  
  
  if (shouldRedirect) {  
    redirect("/home"); // try/catch문 안에서 X  }  
  
  return { message: "success" };  
};
```


## 로그아웃 방법

### 클라이언트 사이드

- `next-auth/react`의 `signOut` 함수를 이용

```ts
const onLogout = async () => {  
  await signOut({ redirect: false });  
  router.replace("/");  
};
```

## 세션 가져오기

### 클라이언트 사이드 (useSession)

- `useSession`을 사용하면 클라이언트 사이드에서 세션 정보를 가져올 수 있음
- 유저 정보는 `session.data`에 저장되어 있음 (auth.ts에서 return한 값)

```tsx
"use client";  
  
import style from "./LogoutButton.module.css";  
import { signOut, useSession } from "next-auth/react";  
import { useRouter } from "next/navigation";  
  
export default function LogoutButton() {  
  const router = useRouter();  
  const { data: me } = useSession();  
  
  const onLogout = async () => {  
    await signOut({ redirect: false });  
    router.replace("/");  
  };  
  
  if (!me || !me.user) {  
    return null;  
  }  
  
  return (  
    <button className={style.logoutButton} onClick={onLogout}>  
      <div className={style.logoutUserImage}>  
        <img src={me.user?.image!} alt={me.user.id} />  
      </div>  
      <div className={style.logoutUserName}>  
        <div>{me.user.name}</div>  
        <div>@{me.user.id}</div>  
      </div>  
    </button>  
  );  
}
```

### 서버 사이드 (auth)

- `next-auth` 설정 파일의 `auth`를 가져와서 사용
- 기본적으로 promise를 반환하기 때문에 비동기 함수로 바꾸고 await로 호출

```tsx
import Main from "@/app/(beforeLogin)/_component/Main";  
import { auth } from "@/auth";  
import { redirect } from "next/navigation";  
  
export default async function Page() {  
  const session = await auth();  
  
  if (session?.user) {  
    redirect("/home"); // 리턴값이 never : 그 아래로 내려가지 않음
  }  
  
  return <Main />;  
}
```

## NextAuth와 CSRF

![[Pasted image 20240115162857.png]]

- NextAuth에서는 세션을 쿠키에 위와 같이 관리함
- 세션을 탈취하는 공격 = CSRF
- NextAuth에서는 이를 알아서 방지해줌 (csrf-token)

## 인증이 필요한 페이지 설정

- middleware 함수에서 세션을 가져와서 세션이 없다면 분기 처리 (redirect)
- config 내의 matcher에 대해서만 해당 미들웨어가 실행됨

```ts
// middleware.ts
import { auth } from "./auth";  
import { NextResponse } from "next/server";  
  
export async function middleware() {  
  const session = await auth();  
  if (!session) {  
    return NextResponse.redirect("http://localhost:3000/i/flow/login");  
  }  
}  
  
export const config = {  
  matcher: ["/compose/tweet", "/home", "/explore", "/messages", "/search"],  
};
```

- 혹은 위와 동일한 기능을 NextAuth 옵션의 callback에 추가할 수 있음