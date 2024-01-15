## 설치

```zsh
npm i @tanstack/react-query@5 
npm i -D @tanstack/react-query-devtools@5
```

- 최신 버전(v5)과 이전 버전 차이가 꽤 나기 때문에 주의
- RQ의 장점인 개발도구를 함께 설치

## 세팅

```tsx
// RQProvider.tsx
import { useState } from "react";  
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";  
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";  
  
interface Props {  
  children: React.ReactNode;  
}  
  
export default function RQProvider({ children }: Props) {  
  const [client, setClient] = useState(  
    new QueryClient({  
      defaultOptions: {  
        queries: {  
          refetchOnWindowFocus: false,  
          retry: false,  
        },  
      },  
    }),  
  );  
  
  return (  
    <QueryClientProvider client={client}>  
      {children}  
      <ReactQueryDevtools  
        initialIsOpen={process.env.NEXT_PUBLIC_MODE === "local"}  
      />  
    </QueryClientProvider>  
  );  
}
```

- 리액트 쿼리 클라이언트를 전달하는 Provider로 감쌀 수 있도록 컴포넌트 작성
- devtools는 개발 단계에서만 사용하므로 환경 변수로 표시할 수 있도록 설정