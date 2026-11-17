# Zustand

_2026.11.17._

> 클라이언트의 전역 상태를 관리하는 라이브러리

> **상태(state)**  
> 값이 바뀔 경우 컴포넌트를 리렌더링하는 값.  
> 그중 여러 컴포넌트가 함께 사용하는 상태를 전역 상태라고 한다.

> **왜 사용해야 하는가?**  
> `useState`로 만든 상태는 해당 컴포넌트 안에서만 사용할 수 있으므로, 여러 컴포넌트가 값을 공유하기 위해서는 공통 부모 컴포넌트에 상태를 두고 props로 값을 내려줘야 한다.
>
> ```tsx
> <App>            // user 상태를 여기에 둠
>   <Layout user={user}>
>     <Header user={user}>
>       <Profile user={user} />   // 실제로 user를 쓰는 곳은 여기뿐...
> ```
>
> **props drilling**: 중간 컴포넌트들이 자기는 사용하지 않는 props를 계속해서 받아 넘기기만 하는 현상. 컴포넌트 구조가 깊어질수록 코드가 복잡해지고 수정하기 어려워진다.  
> **Context API**: props drilling의 해결 방법 중 하나. React에 내장된 기능으로, props 없이도 여러 컴포넌트에 값을 전달할 수 있다. 그러나 전달하는 값 중 하나만 바뀌어도 그 값들을 사용하는 컴포넌트가 모두 리렌더링되는 단점이 존재한다.

> **Zustand vs [TanStack Query](./tanstack-query.md)**  
> Zustand
>
> - 서버와 관계없이 브라우저에서만 사용하는 값, 즉 클라이언트 상태를 관리한다.
>
> TanStack Query
>
> - 서버에서 받아온 데이터, 즉 서버 상태를 관리한다. _EX) 캐싱, 재조회, 재시도_

## store

- Zustand에서 전역 상태를 담는 객체.
- 하나의 객체에 전역 상태 값과, 그 상태값을 변경하는 함수(액션)를 함께 저장한다.

```tsx
import { create } from "zustand"; // store 생성 함수 create를 import

// store에 들어갈 값, 함수의 타입
interface CounterState {
  count: number;
  increase: () => void;
  reset: () => void;
}

export const useCounterStore = create<CounterState>()((set) => ({
  count: 0,
  increase: () => set((state) => ({ count: state.count + 1 })),
  reset: () => set({ count: 0 }),
}));
```

### `export const useCounterStore = create<CounterState>()(...);`

- `CounterState` 타입의 store 객체를 생성하여 내부적으로 저장하고, **해당 store에 접근하는 훅**을 `useCounterStore`라는 이름으로 저장한다.
- Zustand에서 타입을 지정할 경우 타입 뒤에 빈 괄호 `()`를 하나 더 쓴다.  
  (타입을 사용하지 않는 JS에서는 따로 괄호를 쓰지 않는다.)

```tsx
/* TS */
create<CounterState>()((set) => ({ ... }));

/* JS */
create((set) => ({ ... }));
```

> **훅(Hook)**  
> 주로 `use`로 시작하는, 컴포넌트나 다른 훅 안에서만 사용 가능한 함수. _EX) `useState`, `useEffect`_

### `(set) => ({...})`

- store의 내용을 정의하는 함수이다.
- `{...}` 객체 안에 상태 값과, 그 상태값을 변경하는 함수(액션)를 함께 넣고 그 내용이 그대로 store에 담긴다.

## set

- store의 상태를 바꾸는 함수이다.

1. store 정의 함수를 `(set) => ({...})`로 작성하면 Zustand가 이 `set`이라는 매개변수에 상태 변경 함수를 넣어준다.
2. 상태 변경 함수가 된 `set`이 객체 내부의 액션 안에서 상태를 바꾸는 데에 사용된다.

> 사실 상태 변경 함수에 붙이는 이름일 뿐이기에 `set`을 꼭 쓰지 않아도 동작은 동일하다. 그러나 관례상 주로 `set`으로 쓴다.

### 객체 사용

- 기존 필드 값에 관계없이 특정 값으로 변경을 원하는 경우, 객체를 인자로 넣어 사용한다.

```tsx
reset: () => set({ count: 0 }),
```

> 기존 count 값에 관계없이 0으로 설정한다.

### 함수 사용

- 바꿀 값 계산을 위해 현재 값이 필요할 경우, 함수 형태로 현재 값을 불러와 사용한다.

```tsx
increase: () => set((state) => ({ count: state.count + 1 })),
```

> 현재 `count` 값에 `+1`한 값으로 설정한다. (`state.count`)

### 병합(merge)

- set은 기본적으로 인자로 넘긴 상태 필드만 변경하고 나머지 필드값은 변경 없이 그대로 유지하는 특성.

```tsx
// store: { count: 5, name: "나영" }

set({ count: 0 });
// → store: { count: 0, name: "나영" }
```

> `name` 값을 별도 인자로 넘기지 않아도 기존 값이 그대로 유지된다.

- 단, 병합은 store의 **최상위 필드까지만 가능하다. 즉, 중첩 객체의 경우 병합되지 않는다.**  
  _이렇게 최상위 필드까지만 병합하는 방식을 **얕은 병합(shallow merge)** 이라고 한다._

```tsx
// store: { user: { name: "나영", email: "a@a.com" } }

set({ user: { name: "새이름" } });
// → store: { user: { name: "새이름" } }
```

> `email`은 중첩 객체 `user` 안의 필드이므로 병합되지 않는다.  
> 따라서 `user`가 통째로 바뀌면서 별도 지정이 없었던 `email`은 사라졌다.

- 중첩 객체 안의 값을 변경하지 않는 필드들도 유지하기 위해서는 `...`(스프레드 문법)으로 기존 값을 복사해 넣어주어야 한다.

```tsx
// store: { user: { name: "나영", email: "a@a.com" } }

set((state) => ({ user: { ...state.user, name: "새이름" } }));
// → store: { user: { name: "새이름", email: "a@a.com" } }
```

> 중첩 객체 `user`의 전체 필드를 복사해서 객체에 넣은 뒤 변경하고자 하는 필드 값만 업데이트한다. 결과적으로 `email` 값이 보존된다.

## selector

- store 훅에 인자로 넣어, store에서 필요한 필드 값만 꺼내는 함수.

```tsx
function Counter() {
  const count = useCounterStore((state) => state.count); // selector: (state) => state.count
  const increase = useCounterStore((state) => state.increase); // selector: (state) => state.increase

  return <button onClick={increase}>{count}</button>;
}
```

> `state`는 store 전체이고, 그 중 `count`와 `increase`만 꺼내었다.

> **구독(subscribe)**  
> 특정 값의 변화를 감시하다가, 해당 값이 바뀌면 반응하는 것.  
> 위 예시에서 `Counter`는 `count`와 `increase`를 구독하고 있다.  
> 즉, store의 다른 필드값은 신경쓰지 않고 구독하고 있는 필드값의 변경이 있을 때에만 리렌더링된다.  
> selector 없이 `useCounterStore()`를 사용하면 store 전체를 구독하게 된다. 즉, store의 어떤 값이 바뀌든 항상 리렌더링된다.

### useShallow

- store의 필드 여러 개를 객체 하나로 묶어서 꺼내고 싶을 때 사용하는 함수.
- selector가 객체를 반환할 때, 직전 객체와 필드값을 하나씩 비교해서 같으면 직전 객체를 그대로 반환한다.

> **왜 사용해야 하는가?**  
> React가 특정 컴포넌트를 화면에 렌더링할 때, 렌더링 전후로 두 번 값을 읽는다.
>
> 1. 렌더링할 때 → selector 실행  
>    React가 `Counter` 함수를 실행하고, `useCounterStore(selector)`의 selector를 실행해 하나의 값을 받는다. 해당 값으로 화면을 그린다.
> 2. 렌더링한 직후 → selector 한 번 더 실행  
>    React는 렌더링 도중 실제 값이 바뀌었을 것을 우려하여, 여전히 지금 렌더링한 값이 유효한지 확인하기 위해 한 번 더 selector를 실행하여 값을 받는다. 이를 1에서 받은 값과 `===`로 비교해서 값이 바뀌었는지(즉, 리렌더링해야 할지) 확인한다.
>
> **여기서 문제가 발생한다.**  
> 값을 객체(`{ }`)로 꺼낼 경우, selector가 실행될 때마다 매번 새로운 객체가 만들어진다. (내부 필드값은 동일하더라도 다른 객체이다.)  
> 이 때 `===`는 객체를 비교할 때 두 객체의 필드값이 아니라, 같은 객체인지를 비교한다. 즉 1과 2에서 항상 서로 다른 객체를 받게 되므로 React는 매번 렌더링 전후로 값이 바뀌었다고 생각하고, **무한히 리렌더링하게 된다.**
>
> ```tsx
> function Counter() {
>   const { count, increase } = useCounterStore((state) => ({
>     count: state.count,
>     increase: state.increase,
>   }));
>
>   return <button onClick={increase}>{count}</button>;
> }
> ```

```tsx
import { useShallow } from "zustand/react/shallow";

function Counter() {
  const { count, increase } = useCounterStore(
    useShallow((state) => ({
      count: state.count,
      increase: state.increase,
    })),
  );

  return <button onClick={increase}>{count}</button>;
}
```

> `useShallow(selector)` 형태로 사용한다.

1. selector가 새 객체를 만든다.
2. 바로 직전 selector가 만들었던 객체와 내부 필드값을 하나씩 `===`로 비교한다.
3. 필드값이 전부 동일하면 동일한 객체로 판단하고, 새 객체를 버리고 이전 객체를 그대로 반환한다.
4. 필드값이 하나라도 다르면 새 객체를 반환하고, 다음 비교를 위해 기억해 둔다.

즉 React는 계속해서 `===`로 비교하지만 `useShallow`가 내부적으로 객체 비교의 함정을 회피해주기 때문에 객체 반환값을 원할 때 문제없이 동작 가능하다.

> **얕은 비교(shallow compare)**  
> 객체 자체가 아니라 객체의 최상위 필드값만 하나씩 비교하는 방식.

## getState·setState

- store를 컴포넌트가 아닌 곳에서 사용하기 위한 함수.
- `create`가 반환한 훅에는 store를 직접 다루는 함수(store API)가 포함되어 있어, 컴포넌트 밖에서도 store를 사용할 수 있다.

```tsx
useAuthStore.getState();
useAuthStore.setState({ accessToken: null });
```

### getState

- store 상태 전체를 반환하는 함수.
- `getState()` 뒤에 `.`를 붙여 특정 필드값 또는 액션만 꺼내올 수 있다.

```tsx
import { useAuthStore } from "@/stores/authStore";

apiClient.interceptors.request.use((config) => {
  const token = useAuthStore.getState().accessToken;
  config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

> axios 인터셉터에서(컴포넌트 외부) `.accessToken`으로 토큰만 꺼내는 예시.

- 훅 호출 동작이 아니므로 **구독하지 않는다.**
- 즉, 꺼내온 값이 바뀌더라도 리렌더링하지 않는다.

> `useCounterStore` 뒤에 `()`를 붙여 호출하면 훅 호출이다.
>
> ```tsx
> useCounterStore((state) => state.count); // useCounterStore를 호출 → 훅 호출
> useCounterStore.getState(); // getState를 호출 → 훅 호출 아님
> ```

### setState

- store 상태값을 바꾸는 함수. _`set`과 동일 기능, 객체 사용·함수 사용_
- 병합 규칙이 `set`과 동일하다. _shallow merge, 스프레드 문법(`...`)도 동일_

```tsx
// store: { accessToken: "abc", user: { name: "나영", email: "a@a.com" } }

useAuthStore.setState({ accessToken: null });
// → store: { accessToken: null, user: { name: "나영", email: "a@a.com" } }

useAuthStore.setState({ user: { name: "새이름" } });
// → store: { accessToken: null, user: { name: "새이름" } }
```

## persist

- 브라우저를 새로고침해도 값이 유지되도록, localStorage에 저장하기 위해 사용하는 함수.

> **왜 사용해야 하는가?**  
> Zustand store는 일반적으로 브라우저 탭 메모리에 저장된다. 즉, 브라우저를 새로고침하면 값이 사라진다.  
> 반면 localStorage는 브라우저 저장소에 저장되므로 브라우저를 닫거나 새로고침해도 그 값이 유지된다.

```tsx
import { persist } from "zustand/middleware"; // 별도 미들웨어 경로에서 import

export const useThemeStore = create<ThemeState>()(
  persist(
    (set) => ({
      theme: "light",
      setTheme: (theme) => set({ theme }),
    }),
    { name: "theme-storage" },
  ),
);
```

> `persist((set) => ({...}), {옵션 객체})` 형태로 사용한다.

- `{ name: "theme-storage" }`: 옵션 객체로, store 상태를 localStorage에 저장할 때 쓸 키 이름.

```text
키:  theme-storage
값:  {"state":{"theme":"dark"},"version":0}
```

> `setTheme("dark")`를 실행했을 때 localStorage에 저장되는 형태.

- localStorage는 문자열만 저장할 수 있기 때문에 store 내용이 JSON 문자열로 변환되어 저장된다.
- `state`: 저장된 상태 값. **함수(액션)는 JSON으로 변환할 수 없기 때문에 저장되지 않는다.**
- `version`: 추후 store 구조가 변경될 때 과거 저장 데이터를 구분하기 위해 붙는 번호. 별도 설정하지 않으면 0이다.

> **미들웨어(middleware)**  
> store 정의 함수를 감싸서 기능을 덧붙이는 함수.

### partialize

- `persist`는 기본적으로 store의 모든 상태값을 저장하므로, 그 중 일부 값만 저장하기 위해 사용하는 옵션.

```tsx
{
  name: "theme-storage",
  partialize: (state) => ({ theme: state.theme }),
}
```

> store 전체를 받아서 저장할 필드만 선택한다. selector와 같은 형식이다.

## 전체 흐름 정리

- `create`로 상태 값과 액션을 담은 store를 만들고, 그 store에 접근하는 훅을 받는다.
- 액션 안에서 `set`으로 상태를 변경한다. 이때 최상위 필드만 병합되므로 중첩 객체는 `...`로 기존 값을 복사한다.
- 컴포넌트에서는 selector로 필요한 값만 꺼내 구독하고, 여러 값을 객체로 꺼낼 때는 `useShallow`로 감싼다.
- 컴포넌트 밖에서는 `getState`·`setState`로 store를 읽고 변경한다.
- 새로고침 후에도 유지해야 하는 값은 `persist`로 localStorage에 저장한다.
