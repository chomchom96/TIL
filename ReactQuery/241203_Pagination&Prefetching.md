### 댓글을 위한 쿼리 생성

- useQuery로 매개변수 포함하는 쿼리 생성하기
    - 전달하는 함수에 매개변수가 필요한 경우 매개변수를 전달하는 익명 함수(화살표 함수)를 전달

```jsx
export function PostDetail({ post }) {
  const {data} = useQuery({
    queryKey: ["comments", post.id],
    queryFn: () => fetchComments(post.id),
    staleTime: 2000,
  });
```

### 쿼리 키

- comments 쿼리가 refetch되지 않음
    - 모든 comments 쿼리가 comments 키를 공유하기 때문
    - refetch trigger
        - component remount
        - window refocus
        - refetch function 수동 실행
        - automated
        - query invalidated after mutation
    1. 새로운 포스트 클릭 시마다 invalidate → 비효율적, 캐싱 데이터가 날아감
    2. 각 포스트에 대한 쿼리를 id로 라벨링
- 쿼리 키 배열에 두 번째 요소를 추가할 수 있음
    - Query function에 전달하는 value는 키에 반드시 포함

```jsx
queryKey: ["comments", post.id],
```

- 다른 포스트를 클릭 시 이전 포스트의 쿼리는 stale → gcTime에 따라 처리

### Pagination

- 현재 페이지 state로 관리 & 쿼리 키에 현재 페이지 포함

```jsx
const [currentPage, setCurrentPage] = useState(0);

const {data, isError, isLoading} = useQuery({
  queryKey: ["posts", currentPage],
  queryFn: () => fetchPosts(currentPage),
  staleTime: 2000,
});
...

<button disabled={currentPage==0} onClick={() => {setCurrentPage(currentPage => currentPage - 1)}}>
  Previous page
</button>
```

- 현재는 페이지 이동 시 로딩 표시 → prefetching으로 해결

### Pre-fetching

- 데이터를 캐시에 저장
- 기본적으로 stale → 표시할 때 refetching이 일어남
    - refetcing 전 데이터를 보여줄 수 있음

```jsx
// state 변경은 비동기므로 현재 페이지가 무엇인지 확실하게 알 수 없음
// 페이지 변경 시 useEffect로 처리
useEffect(() => {
	if (currentPage < maxPostPage) {
	  const nextPage = currentPage + 1;
	  queryClient.prefetchQuery({ 
	    queryKey: ["posts", nextPage],
	    queryFn: () => fetchPosts(nextPage),
    }
  })
}, [currentPage, queryClient])
```

### isLoading vs isFetching

- isFetching → async function not resolved
- isLoading → isFetching 하위, isFetching & 캐시 없음
- 현재는 isLoading시 로딩창이 표시되므로 fetching시 빈화면이 표시 → isFetching으로 수정시 해결

### Mutation

- 서버에서 데이터를 변경할 네트워크 호출을 mutation
- Mutation에 따른 변화 대응법
    - optimistic : 변화가 일어날 것이라 가정(에러 발생을 생각하지 않음)
    - 서버에서 받아온 데이터로 캐시 업데이트
    - invalidation : 관련 데이터 refetch
- useMutation 훅으로 처리

### useMutation으로 삭제하기

- 현재는 jsonPlaceholder API이므로 실제로 삭제되진 않지만 delete 전송 시 반환되는 useMutation 객체를 다룰 예정
    - queryKey는 없음

```jsx
const deleteMutation = useMutation({
  mutationFn: (postId) => deletePost(postId),
})

// delete시 deleteMutation.mutate로 호출
...
{selectedPost && <PostDetail post={selectedPost} deleteMutation={deleteMutation}/>}
```

### 다른 useMutation 속성

- loading & error 속성
    - useMutation은 캐시가 없기 때문에 속성이 단순함
    - refetch, stale이 존재하지 않음
    - isLoading, isFetching ⇒ isPending

```jsx
{deleteMutation.isPending && <p className="loading">Deleting,,,</p>}
{deleteMutation.isError && <p className="error">Error!</p>}
{deleteMutation.isSuccess && <p className="success">Success!</p>}
```

→ 다른 포스트 클릭시 isSuccess 상태가 변하지 않아서 표시됨 →  reset

```jsx
<li
  key={post.id}
  className="post-title"
  onClick={() => {
    deleteMutation.reset();
    setSelectedPost(post);
  }}
>
  {post.title}
</li>
```

### 요약

- QueryClient 생성, QueryClient 제공 → 캐시와 훅을 자식에 제공
- useQuery → 데이터 업데이트, fetch, loading, error 처리
- staleTime → refetch 시간 설정
- gcTime → 비활성 데이터 캐시 저장 시간 설정
- queryKey에 데이터 종속 배열 추가
- pagination & prefetching
- 일부 useMutation 기능 사용
