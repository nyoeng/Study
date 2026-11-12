# TanStack Query

_2026.11.12._

> 서버에서 받은 데이터를 저장 및 갱신하고, 요청 상태를 관리하는 라이브러리

> **왜 사용해야 하는가?**  
> TanStack Query 없이 `useEffect + useState`로 API를 호출하게 될 경우
>
> 1. 데이터 요청의 로딩/에러 상태, 중복 요청, 재요청 시점을 직접 구현해야 함
> 2. API를 호출하는 코드마다 `useEffect`, `useState`를 매번 작성해야 함
> 3. 받은 데이터가 컴포넌트에만 일시적으로 저장되기 때문에 페이지를 잠깐 이동했다 돌아오는 경우에도 재요청 필요

## `useQuery`

- 서버 데이터를 조회하기 위한 훅.
- `queryKey`로 데이터를 구분해 저장한다.
- 데이터 및 데이터 요청 상태(로딩, 에러)를 반환한다.

```ts
import { useQuery } from "@tanstack/react-query";

export const useMyProfileQuery = () => {
  return useQuery({
    queryKey: queryKeys.user.me(),
    queryFn: getMyProfile,
  });
};
```

### `export const useMyProfileQuery = () => {};`

- 내 프로필을 조회하는 커스텀 훅을 만든다.
- 매번 `useQuery`를 직접 호출할 필요 없이 이 훅만 호출하면 되도록 한다.

### `return useQuery({...});`

- `useQuery`를 실행 후 그 결과(데이터, 요청 상태)를 반환한다.
- `{...}`: `{queryKey: 키, queryFn: 함수}` 형태의 객체

### `queryKey: queryKeys.user.me()`

- `queryKeys`: `shared/api/query-keys.ts`에 사전에 만들어 둔 객체.
- `객체.객체.함수()` 형태, 실제 값은 `["user", "me"]` 형태의 배열

> queryKey의 자세한 구조는 아래 [queryKey](#querykey) 참고

### `queryFn: getMyProfile`

- `getMyProfile`: `entities/user/api/getMyProfile.ts`에 사전에 만들어 둔 함수.
- `getMyProfile()`이 아닌 `getMyProfile`, 즉 함수 자체를 넘기므로 함수를 언제 실행할지는 TanStack Query가 결정한다.

## `useMutation`

- 서버 데이터를 생성, 수정, 삭제하기 위한 훅.
- 훅을 호출하는 시점과 서버에 요청하는 시점이 별개이다.
  - 훅 호출 시 반환되는 `mutate` 함수를 실행하는 시점에 서버에 요청이 감
- 서버 요청 상태(진행 중, 에러)를 반환한다.

```ts
import { useMutation } from "@tanstack/react-query";

export const useSignUpMutation = () => {
  return useMutation({
    mutationFn: (payload: SignUpRequest) => signUp(payload),
  });
};
```

### `export const useSignUpMutation = () => {};`

- 회원가입 요청을 보내는 커스텀 훅을 만든다.
- 매번 `mutationFn`을 작성할 필요 없이 이 훅만 호출하면 되도록 한다.

### `return useMutation({...});`

- `useMutation`을 실행 후 그 결과(`mutate`, 요청 상태)를 반환한다.
- `{...}`: `{mutationFn: 함수}` 형태의 객체

### `mutationFn: (payload: SignUpRequest) => signUp(payload)`

- `signUp`: `entities/user/api/signup.ts`에 사전에 만들어 둔 함수.
- `(payload: SignUpRequest) => signUp(payload)`: `SignUpRequest` 타입의 데이터(`payload`)를 인자로 받아 `signUp`을 실행하는 함수.
- 추후 `mutate`를 실행하면서 넘기는 데이터가 `payload`에 들어간다.

```tsx
const { mutate: signUp, isPending: isSigningUp } = useSignUpMutation();

signUp(
  { loginId: userId, passWord: password, name: userName, email: userEmail },
  {
    onSuccess: () => {
      /* 성공 시 토스트 출력 후 로그인 페이지로 이동 */
      toast.success("회원가입이 완료되었습니다. 로그인해 주세요.");
      navigate("/login");
    },
  },
);
```

### `const { mutate: signUp, isPending: isSigningUp } = useSignUpMutation();`

- 커스텀 훅을 호출해서 반환값 중 필요한 것만 `signUp`, `isSigningUp`라는 이름으로 꺼낸다.(`entities/user/api/signup.ts`에 사전에 만들어 둔 함수와는 별개의 함수임!)

### `signUp(..., {...})`

- `mutate` 함수에 인자를 넣어 실행한다.
- 첫 번째 인자: `mutationFn`의 `payload`로 들어가는 데이터
  - `payload` 타입에 따라 객체(`{...}`)일 수도, 또는 단일 값일 수도 있음
- 두 번째 `{...}`: 서버 요청 결과에 따라 실행할 함수를 담은 객체

## `invalidateQueries`

- 저장되어 있는 데이터를 유효하지 않은 상태로 만들고, 서버에서 새로 값을 가져와 갱신하는 함수.
- 현재 페이지에서 필요한 데이터만 그때그때 다시 가져온다.

```ts
import { useMutation, useQueryClient } from "@tanstack/react-query";

/* 폴더 생성 요청을 보내는 커스텀 훅 생성 */
export const useCreateFolderMutation = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (payload: CreateFolderRequest) => createFolder(payload),
    onSuccess: async () => {
      await queryClient.invalidateQueries({
        queryKey: queryKeys.folder.all,
      });
    },
  });
};
```

### `const queryClient = useQueryClient();`

- `useQueryClient()`: TanStack Query에서 데이터를 관리하는 객체를 반환하는 훅.
- 반환된 데이터 관리 객체를 `queryClient`라는 변수에 담는다.

### `onSuccess: async () => {...}`

- `async`: 함수 안에서 `await`를 사용하기 위해 붙인다.

### `await queryClient.invalidateQueries({...});`

- `queryClient` 안의 `invalidateQueries` 함수를 실행한다.
- `{...}` 안의 queryKey에 해당하는 데이터를 유효하지 않은 상태로 만들고, 서버에서 다시 가져온다.
- `await`: 함수 실행이 완료될 때까지 기다린다. 즉 폴더 목록 갱신까지 완료된 뒤에 요청이 완료된 것으로 처리된다.

### `queryKey: queryKeys.folder.all`

- `객체.객체.배열` 형태, 실제 값은 `["folder"]`
- 즉 folder로 시작하는 키를 가진 데이터(폴더 목록, 폴더 상세 등)가 전부 무효화된다.

## `queryKey`

- 서버에서 받은 데이터를 구분하는 이름으로, 배열(`[...]`) 형태이다.
- 동일한 `queryKey`를 사용하는 곳은 저장된 데이터를 함께 사용한다.

```ts
/* queryKeys라는 하나의 객체를 정의해 두고 이 안에서 모든 queryKey를 도메인별로 관리한다. (as const를 붙여 배열 형태를 고정함)*/
export const queryKeys = {
  user: {
    all: ["user"] as const,
    me: () => ["user", "me"] as const,
  },
  folder: {
    all: ["folder"] as const,
    list: (params?: FolderListParams) => ["folder", "list", params] as const,
    detail: (folderId: string) => ["folder", "detail", folderId] as const,
  },
};
```

### `list: (params?: FolderListParams) => ["folder", "list", params] as const`

- `params`라는 이름의 인자를 받아 배열을 만드는 함수. 인자가 필수는 아니다.
- `params`의 형태는 `FolderListParams`라는 사전에 정의된 타입의 객체여야 함

## 전체 흐름 정리

- `useQuery`로 서버에서 조회해온 데이터가 `queryKey`로 구분되어 저장된다.
- `useMutation`으로 데이터를 생성, 수정, 삭제한다.
- `useMutation` 성공 시 `invalidateQueries`로 관련 `queryKey`를 무효화하여 데이터를 최신 값으로 재조회한다.

## 비고

- 예시 코드는 과거 프로젝트에서 실제로 사용한 코드를 일부 생략해 가져왔다.
