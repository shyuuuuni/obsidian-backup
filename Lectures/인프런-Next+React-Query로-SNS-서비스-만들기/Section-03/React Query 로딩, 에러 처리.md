
## loading


- isPending : 캐시된 데이터가 없고 쿼리 시도가 처리되지 않은 경우
- isLoading : 데이터를 로딩하는지 (isPending && isFetching) - 즉 쿼리에 대한 첫번째 가져오기가 진행중일 때
- isRefetching : 처음 시도를 제외하고 다음 가져오기가 진행중일 때 (!isPending && isFetching)

```tsx
"use client";

import FollowRecommend from "@/app/(afterLogin)/_component/FollowRecommend";
import styles from "./FollowSection.module.css";
import { useQuery } from "@tanstack/react-query";
import { getFollowRecommends } from "@/app/(afterLogin)/_lib/getFollowRecommends";
export default function FollowSection() {
  const { data, isPending } = useQuery({
    queryFn: getFollowRecommends,
    queryKey: ["users", "followRecommends"],
    staleTime: 60 * 1000,
    gcTime: 300 * 1000,
  });

  if (isPending) {
    return <div>로딩중..</div>;
  }

  return (
    <div className={styles.container}>
      <h3>팔로우 추천</h3>
      {data?.map((user) => <FollowRecommend key={user.id} user={user} />)}
    </div>
  );
}

```

- 주의: 만약 위와 같은 클라이언트 컴포넌트 외부에 `prefetchQuery`를 통해서 가져온다면 로딩창이 뜨지 않음 - SSR로 모두 만들어서 내보내기 때문

## error

- isError : 에러 상태인지 확인

```tsx
"use client";

import FollowRecommend from "@/app/(afterLogin)/_component/FollowRecommend";
import styles from "./FollowSection.module.css";
import { useQuery } from "@tanstack/react-query";
import { getFollowRecommends } from "@/app/(afterLogin)/_lib/getFollowRecommends";
export default function FollowSection() {
  const { data, isPending, isError } = useQuery({
    queryFn: getFollowRecommends,
    queryKey: ["users", "followRecommends"],
    staleTime: 60 * 1000,
    gcTime: 300 * 1000,
  });

  if (isPending) {
    return <div>로딩중..</div>;
  }

  if (isError) {
    return <div>에러 처리해줘..</div>;
  }

  return (
    <div className={styles.container}>
      <h3>팔로우 추천</h3>
      {data?.map((user) => <FollowRecommend key={user.id} user={user} />)}
    </div>
  );
}

```