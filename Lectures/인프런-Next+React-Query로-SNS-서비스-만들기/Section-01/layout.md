- layout.tsx 에 작성
## root layout

기본적으로 `src/layout.tsx`에 작성된 레이아웃을 공통으로 사용함

## sub layout

- 특정 라우팅에서의 레이아웃을 사용하기 위해서는 해당 폴더에 `layout.tsx` 작성
- 검색할 때 해당 위치에서 root로 빠져나가면서 `layout.tsx`를 검색
- 계층 구조의 layout을 사용함

## template.tsx

- 레이아웃은 페이지가 넘나들 때 리렌더링 되지 않음
- 만약 페이지를 넘나들 때 매번 새롭게 마운트해야 하는 경우 사용
- `layout.tsx`와 `template.tsx` 중 하나만 사용
- 잘 사용하지 않아도 된다.
