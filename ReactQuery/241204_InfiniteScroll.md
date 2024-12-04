### Intro

- 페이지의 특정 위치에 도달하면 데이터 fetch
- useInfiniteQuery로 다음 쿼리를 추적
    - 다음 쿼리가 반환 객체에 포함됨
    - 다음 페이지, 이전 페이지를 이동하기 위한 쿼리
    - 전체 페이지 count 포함

### 앱 설정

- 저번 시간의 QueryClient, Devtools 설정

```
function App() {
  const queryClient = new QueryClient();

  return (
    <QueryClientProvider client={queryClient}>
      <div className="App">
        <h1>Infinite SWAPI</h1>
        <InfinitePeople />
        {/* <InfiniteSpecies /> */}
      </div>
      <ReactQueryDevtools />
    </QueryClientProvider>
  );
}
```

### useInfiniteQuery

- useQuery와 차이점
    - useQuery는 데이터를 data에 담기지만, useInfiniteQuery는 data와 pages를 포함
    - pages는 각 데이터 페이지를 나타내는 객체 배열
    - pageParams → 페이지마다 사용하는 param, 많이는 안씀
    - 각 쿼리는 pages 배열에 고유 element, 쿼리는 페이지 진행에 따라 변화
- useInfiniteQuery Syntax
    - 쿼리 함수는 객체에서 구조분해된 pageParam을 받음 → 페이지 조회에 사용
    - getNextPageParam → lastPage, allPages(다음, 전체)
- 반환 객체
    - fetchNextPage → 다음 데이터 호출 함수
    - hasNextPage → getNextPageParam에 따라 결정, boolean
    - isFetcihngNextPage → 다음 페이지를 가져오는 중인지, 일반 데이터를 가져오는 중인지와 구분됨

### useInfiniteQuery Diagram

1. component mount
    1. useInfiniteQuery 반환 객체는 undefined
2. fetch first page
    1. 쿼리 함수에 pageParam 전달 → 기본값(설정시)
    2. data.pages 설정, 첫 페이지는 data.pages[0]
3. getNextPageParam
    1. 다음 페이지와 모든 페이지를 받아 pageParam 업데이트
    2. 다음 페이지가 있는지? = pageParam이 존재하는지
4. hasNextPage 시 fetchNextPage
    1. 스크롤/버튼 클릭 등으로 트리거
    2. 구조 분해된 객체에서 fetchNextPage 호출
    3. pageParam이 뭐든 queryFn을 실행, 페이지 배열에 다음 요소로 추가
5. getNextPageParam 실행
    1. 마지막 페이지인 경우 → 다음 pageParam = undefined 반환
    2. hasNextPage = false

### useInfiniteQuery 호출 쿼리

```jsx
export function InfinitePeople() {
  const { data, hasNextPage, fetchNextPage } = useInfiniteQuery({
    queryKey: ["sw-people"],
    // pageParam이 없을 경우 initial 사용
    queryFn: ({ pageParam = initialUrl }) => fetchUrl(pageParam),
    getNextPageParam: (lastPage) => {
    // API에서 다음 페이지가 없으면 null 반환 -> undefined로 치환
      return lastPage.next || undefined;
    },
  });
  return <InfiniteScroll />;
}
```

### InfiniteScroll component

- react-infinite-scroller 패키지
    - hasMore = hasNextPage 그대로 전달
    - loadMore = () ⇒ if (!isFetching()) fetchNextPage();
        - 조건부 fetch로 중복 api 콜 방지
- 스크롤 탐지는 자동으로

```jsx
return (
  <InfiniteScroll
	  // 자동으로 페이지 로드를 실행하므로 useQuery와 중복되어 2페이지를 호출하는 이슈
	  initialLoad={false}
    loadMore={() => {
      if (!isFetching) fetchNextPage();
    }}
    hasMore={hasNextPage}
  >
    {data.pages.map((pageData) => {
      return pageData.results.map((person) => {
        <Person
          key={person.name}
          name={person.name}
          hairColor={person.hairColor}
          eyeColor={person.eyeColor}
        />;
      });
    })}
  </InfiniteScroll>
);
```

→ 비동기 처리를 하지 않아서 undefined 에러

### useInfiniteQuery Fetching & Error handling

- useQuery와 마찬가지로 isLoading을 사용

```jsx
if (isLoading) return <p>Loading..</p>;
if (isError) return <p>Error! {error.toString()}</p>;
```

- isFetching을 사용시 fetch시마다 스크롤이 원위치 → 데이터를 가져올 때마다 return하므로

### 요약

- Bi-directional Scrolling
    - 양방향 스크롤, 데이터 중간에서 시작하는 경우
    - 이후와 이전 데이터를 가져옴
    - next 메소드와 동일한 메소드를 previous에 동일한 메소드 존재
- React Query에서 pageParam은 다음 페이지 쿼리
    - getNextPageParam 옵션에서 관리
        - lastPage, allPages param을 받음
    - hasNextpage
        - boolean, pageParam=undefined면 false
        - 다음 데이터를 가져올지 판단
-
