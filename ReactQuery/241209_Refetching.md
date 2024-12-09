### useQuery의 select 옵션으로 데이터 필터링하기

- useQuery의 select로 받는 데이터를 변형할 수 있음
    - react query는 select문을 memoize 최적화
    - select 함수의 삼중등호 비교로 필요시, 또는 데이터와 함수 변경시에만 refetch 실행
        - 데이터와 선택 함수가 동일하면 실행하지 않음
    - 따라서 익명 함수 대신 안정적인 함수가 필요함
    - useCallback 함수와 함께 사용할 수 있음
- boolean 변수가 true일 때 데이터를 가져오는 select function 실행
    - select 옵션에 함수를 전달하면 항상 실행되기 때문에 if (true) ⇒ 문으로 전달하기 어려움
    - true일 때 실행하는 대체 함수를 작성

```jsx
 const selectFn = (data: AppointmentDateMap, showAll: boolean) => {
    if (showAll) return data;
    return getAvailableAppointments(data, userId);
  }

  const { data: appointments = fallback } = useQuery({
    queryKey: [queryKeys.appointments, monthYear.year, monthYear.month],
    queryFn: () => getAppointments(monthYear.year, monthYear.month),
    select: (data) => selectFn(data, showAll),
  });
```

→ 훅이 실행될 때마다 재실행되므로 불안정

→ useCallback 훅으로 묶음(함수 memoize)

```jsx
const selectFn = useCallback((data: AppointmentDateMap, showAll: boolean) => {
  if (showAll) return data;
  return getAvailableAppointments(data, userId);
}, [userId]);
```

- prefetch 데이터에 select 포함하지 않는 이유
    - select는 프레젠테이션 기능, 캐시에 영향 x
    - prefetch로 모든 데이터를 캐시에 저장한 뒤 useQuery 호출 시 데이터 변환 및 필터링 → select 옵션

### Refetching intro

- refetch로 stale한 쿼리를 서버에서 업데이트
    - 백그라운드에서 조건에 따라 refetch
        - 리액트 컴포넌트 마운트
        - 네트워크 재연결
        - window refocus
        - refetchInterval이 경과할 경우 → 사용자 작업이 없어도 데이터가 최신 상태가 되도록 임의 설정
- 전역 또는 특정 설정 가능
    - refetchOnMount, refetchOnWindowFocus, refetchOnReconnet → boolean
    - refetchInterval → time
    - useQuery의 refetch 함수를 설정할 수도 있음
- 기본적으로 react query의 refetch는 매우 공격적
- boolean 설정의 기본값은 모드 true
- refetching 억제하기
    - stale time 늘리기
    - 임의 boolean 설정 false
    - 자주 변경되지 않아도 큰 영향이 없는 경우에 사용

### Refetch 옵션 업데이트

```jsx
export function useTreatments(): Treatment[] {
  const fallback: Treatment[] = [];
  const { data= fallback } = useQuery({
    queryKey: [queryKeys.treatments],
    queryFn: getTreatments,
    staleTime: 600000, // 10min (default:5min)
    // staleTime이 gc보다 크면 로딩중 데이터 표시가 안될 수 있음
    gcTime: 900000,
    refetchOnMount: false,
    refetchOnWindowFocus: false,
    refetchOnReconnect: false,
  });
  return data || fallback;
}
```

→ useQuery에 적용한 설정이 prefetch에는 적용되지 않음, prefetch에도 일부 동일한 설정을 해야 설정이 적용됨

```jsx
queryClient.prefetchQuery({
  queryKey: [queryKeys.treatments],
  queryFn: getTreatments,
  staleTime: 600000,
  gcTime: 900000,
});
```

### 전역 refetch 옵션

- 최상의 queryClient에서 기본 옵션을 설정할 수 있음
    - 하위 queryClient의 설정이 오버라이딩

```jsx
// queryClient.ts
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 600000,
      gcTime: 900000,
    }
  },
  queryCache: new QueryCache({
    onError: (error) => {
      errorHandler(error.message);
    },
  }),
});
```

### Polling & Auto Refetching

- 사용자와 상호작용이 없더라도 refetch가 계속 일어나야 되는 경우가 있음
- 사용자가 페이지와 상호작용 하는 도중 서버에 변경이 일어날 수도 있음
- stalteTime, cacheTime, refetchOn 설정을 변경해서 적극적 refetch 구현
- 또는 useQuery의 refetchInterval 옵션으로 설정

```jsx
const { data: appointments = fallback } = useQuery({
  queryKey: [queryKeys.appointments, monthYear.year, monthYear.month],
  queryFn: () => getAppointments(monthYear.year, monthYear.month),
  select: (data) => selectFn(data, showAll),
  refetchOnWindowFocus: true,
  refetchInterval: 1000, // 좀 과하긴함
});
```
