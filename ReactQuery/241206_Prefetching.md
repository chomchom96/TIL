### 캐시에 데이터 추가하기

- fetch 전 캐시에 데이터를 미리 채울 수 있음
    - prefetchQuery
        - queryClient 메소드
        - 서버에서 데이터를 받아옴
    - setQueryData
        - queryClient 메소드
        - 클라이언트에서 데이터를 받아옴
        - mutation 예상 데이터로 사용할 수 있음
    - placeholderData
        - useQuery의 옵션
        - 클라이언트에서 사용
        - 캐시에 저장되지 않음
        - 고정값 또는 값읠 계산하는 함수
    - initialData
        - useQuery의 옵션
        - 캐시에 저장
        - 쿼리에 대한 유효한 데이터

### Prefetching

- 데이터가 자주 변하지 않는다고 가정
- 데이터를 default gcTime인 5분 내로 로드하지 않으면 garbage collect
- 홈 화면에서 호출, 서버 응답을 기다리기 전에 데이터 로드할 예정
- prefetchQuery
    - useQuery와 달리 클라이언트 캐시에 저장함
    - 일화성
    - useQueryClient 훅으로 사용 → queryClient를 반환
- prefetching 과정
    1. 홈페이지 로드
    2. queryClient.prefetchQuery 실행 → 데이터를 캐시에 저장(treatments)
    3. 사용자가 treatments 페이지 열기
    4. 캐시에서 데이터 가져옴
    5. stale할 경우 useQuery가 새로운 데이터를 가져옴

### Prefetching Syntax

```jsx
export function usePrefetchTreatment():void {
  const queryClient = useQueryClient();
  queryClient.prefetchQuery({
    // useQuery 키와 동일해야 해당 쿼리에 대한 prefetch임을 알려줌
    queryKey: [queryKeys.treatments],
    queryFn: getTreatments,
  })
}

// 홈화면에서 prefetch 불러움
import { usePrefetchTreatment } from "../treatments/hooks/useTreatments";

export function Home() {
  usePrefetchTreatment();
```

- 홈 화면에서 매번 prefetch하는 이유
    - 페이지가 동적이 아님 → 컴포넌트 재실행이 적음
    - 반면 treatment 페이지로 자주 이동한다고 가정하면 합리적
    - staleTime, 캐시 시간등을 설정해서 모든 단일 트리거에서 가져오지 않고 잠시 기다리도록 설정해서 방지
- useEffect로 한 번만 사용하지 않는 이유 → prefetch가 훅이므로 훅 안에 훅을 사용할 수 없음

### 커스텀 훅 useQuery

```jsx
const fallback: AppointmentDateMap = {};
const {data: appointments = fallback } = useQuery({
  queryKey: [queryKeys.appointments],
  queryFn: () => getAppointments(monthYear.year, monthYear.month),
});
```

→ 달력의 월을 넘겨도 데이터가 동일함(refetch가 일어나지 않음)

### 의존성 배열로써 쿼리 키

- useQuery를 구현하는 중 monthYear가 바뀌어도 데이터가 동일함
    - 모든 쿼리마다 키가 같기 때문
    - 새로운 데이터를 로드하기 위해 달력을 넘겨도 refetch 트리거가 없음
    - 트리거 → remount, refocus 등
    - 알려진 키에 대해서는 위의 트리거로만 refetch
- 새로운 키를 사용하면 refetch가 아니라 새롭게 가져오는 작업이 일어나므로 문제를 해결할 수 있음
    - 달마다 다른 키 사용
    - 데이터가 필요할 때 의존성 배열의 키가 달라지는지 확인
    - 첫 번째 키는 동일 → 데이터 무효화를 할 때 배열의 접두사가 똑같은 경우 한 번에 invalidate

```jsx
queryKey: [queryKeys.appointments, monthYear.year, monthYear.month],
```

### Prefetch 적용하기

- 연도는 state로 관리되므로 state가 변경될 때 useEffect 훅에서 prefetch를 처리하는 것이 효과적
    - state는 비동기 처리이므로 useEffect에서 처리할 때 state와 prefetch의 레이싱을 방지할 수 있음

```jsx
useEffect(() => {
  const nextMonthYear = getNewMonthYear(monthYear, 1);
  queryClient.prefetchQuery({
    queryKey: [
      queryKeys.appointments,
      nextMonthYear.year,
      nextMonthYear.month,
    ],
    queryFn: () => getAppointments(nextMonthYear.year, nextMonthYear.month),
  });
}, [queryClient, monthYear]);
```
