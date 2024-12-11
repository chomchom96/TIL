### Intro

- 서버를 업데이트하며 실제
    - query invalidate → trigger refetch
    - 서버에서 refetch data로 캐시 업데이트
    - optimistic update → mutate 성공을 전제로 캐시 업데이트, 실패한다면 롤백
        - mutationCache의 onErrorCallback
- loading indicator
    - useIsMutating이 useIsFetching과 동일

```jsx
// 캐시 적용
mutationCache: new MutationCache({
  onError: (error) => {
    const title = createTitle(error.message, "query");
    errorHandler(title);
  },
}),
// 로딩 적용
import { useIsFetching, useIsMutating } from "@tanstack/react-query";

export function Loading() {
  const isFetching = useIsFetching();
  const isMutating = useIsMutating();
  const display = isFetching || isMutating ? "inherit" : "none";

```

### mutation 적용하기

- useQuery와 유사하지만 차이점이 있음
    - 캐시에 저장하지 않음 → 한번만 작용하기 때문
    - retry x, 설정은 가능함 → useQuery는 세번
    - no refetch
    - 캐시가 없으므로 isLoading과 isFetching의 구분이 없음
    - mutate 함수를 리턴 → 실제 mutation 실행

```jsx
const { mutate } = useMutation({
    mutationFn: (appointment: Appointment) =>
    setAppointmentUser(appointment, userId),
    onSuccess: () => {
      toast({ title: "You have reserved an appointment", status: "success" });
    },
});
return mutate;
```

### mutation 후 invalidate

- mutate 후 업데이트를 위해 기존 쿼리를 무효화
- invalidateQueries
    - query → stale, trigger refetch
    - mutate success → onSuccess 시 invalidateQueries 호출 → refetch
- Query Filter는 여러 쿼리에 영향을 줌
    - removeQueries, invalidate, cancel 등
    - query filter로 쿼리를 지목
        - 쿼리키, 유형(active, inactive, all), 상태(stale, isFetching..)

```jsx
 onSuccess: () => {
    queryClient.invalidateQueries({
      queryKey: [queryKeys.appointments],
    });
```

### useMutation으로 delete

```jsx
const { mutate } = useMutation({
  mutationFn: (appointment: Appointment) =>
    removeAppointmentUser(appointment),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: [queryKeys.appointments] });
    toast({
      title: "cancel",
      status: "info",
    });
  },
});

return mutate;
```

### 응답을 통해 쿼리 캐시 업데이트

- onSuccess 시 서버의 응답을 활용해 캐시를 업데이트하기
    - setQueryData 메소드

```jsx
const { mutate: patchUser } = useMutation({
		mutationKey: [MUTATION_KEY],
    mutationFn: (newData: User) => patchUserOnServer(newData, user),
    onSuccess: (userData: User | null) => {
      toast({ title: "success", status: "success" });
      updateUser(userData);
    },
  });
  return patchUser;
```

### 종속성 배열의 토큰

- 업데이트한 사용자 정보가 다른 페이지에서 업데이트되지 않음
    - 쿼리 캐시에서 정보를 가져오는 페이지에서 업데이트가 일어나지 않았음
    - devTools 확인 시 토큰키가 매우 김 → jwt 토큰을 의존성 배열의 일부에 포함했기 때문
    - 또한 동일한 id에 대해 두개의 쿼리가 있음 → 토큰 만료가 되어 새로 발급되었음
    - 새로운 토큰의 경우 observe가 되지 않아서 업데이트가 안되는 이슈 발생
- 사용자 데이터 캐시를 업데이트 해야함

```jsx
export const generateUserKey = (userId: number, userToken: string) => {
  return [queryKeys.user, userId, userToken]; // userToken 제거
};
```

→ 사용자 토큰을 키에서 배제해서 키가 변경되더라도 id에 대해 유지됨

### 낙관적 업데이트

- 현재는 업데이트 하는동안(로딩) 기존 데이터가 표시됨
- UI를 업데이트하되 캐시를 업데이트하지 않기
    - UI와 캐시를 동시에 업데이트할 수 있지만 상당히 번거로움
    - 이전 서버에서 오는 모든 쿼리를 취소해서 이전 데이터가 현재 캐시를 덮어쓰지 않도록 해야함
    - 업데이트가 실패할 경우를 대비해 이전 데이터를 별도 저장 후 실패 시 롤백도 해야 함
    - 이러한 롤백을 명시하고 관리해야 함
- 데이터가 굉장히 많은 컴포넌트에서 동시에 표시중인 경우 이러한 과정을 거칠 수 있지만, 간단한 경우 UI만 관리
- useMutationState 훅으로 서버에서 변이 데이터를 얻을 수 있음
    - mutationKey로 명시

### 낙관적 업데이트 작성

```jsx
// 업데이트 커스텀 훅
export function usePatchUser() {
  const { user, updateUser } = useUser();
  const queryClient = useQueryClient();

  // TODO: replace with mutate function
  const { mutate: patchUser } = useMutation({
    mutationKey: [MUTATION_KEY],
    mutationFn: (newData: User) => patchUserOnServer(newData, user),
    onSuccess: (userData: User | null) => {
      toast({ title: "success", status: "success" });
      // updateUser(userData);
    },
    // onSuccess + onError 같이 처리
    // 반드시 promise를 return -> 새로운 데이터를 받아오기 전까지 'inProgres' 상태 유지
    onSettled: () => {
      return queryClient.invalidateQueries({
        queryKey: [queryKeys.user],
      });
    },
  });
  return patchUser;
}
// 정보 표시 컴포넌트
const pendingData = useMutationState({
  // filter에 부합하는 모든 mutation data 배열을 반환
  filters: { mutationKey: [MUTATION_KEY], status: "pending" },
  select: (mutation) => {
    return mutation.state.variables;
  },
});
// 사용자에 따라 mutation이 하나이므로 0번쨰 접근
const pendingUser = pendingData ? pendingData[0] : null;
...
<Heading>Information for {pendingUser? pendingUser.name : user.name}</Heading>

```
