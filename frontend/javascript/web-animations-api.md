# Web Animations API

_2026.11.11._

> 브라우저가 제공하는 JS API로, CSS 없이 JS 코드만으로 요소에 애니메이션 적용 및 제어할 수 있게 해 주는 기능

- [[CSS]Animation](../css/animation.md)의 <em>&lt;여러 애니메이션 연속적으로 실행시키기&gt;</em>에서 CSS와 JS를 함께 사용하여 구현하였던 것을 JS만으로 구현한다.

## 기본 형식

```tsx
element.animate([...], {...});
```

### `element`

- 애니메이션을 적용할 DOM 요소

### `[...]`

- CSS의 `@keyframes` 역할을 하는 배열. `{...}` 형태의 원소가 들어간다.
- `{...}` 하나가 애니메이션 시점 하나를 의미한다.
  - `transform` 등 CSS 속성: 그 시점에 요소가 가질 스타일
  - `offset`: 전체 재생 시간 중 시점. _EX) 0.3 = 30%_

### `{...}`

- 옵션 객체로, 애니메이션의 재생 방식을 결정한다. (CSS의 `animation` 역할)
  - `duration`: 재생 시간 (ms) _EX) `2000` = 2s_
  - `easing`: 속도 변화 방식 _EX) `linear`, `ease-in-out`_
  - `fill`: 애니메이션이 끝난 뒤 상태 _EX) `forwards` = 끝난 상태로 멈춤_
  - `iterations`: 애니메이션 반복 횟수 _EX) `1` = 1회 재생, `Infinity` = 무한 반복_
  - `delay`: 애니메이션 재생 전 대기 시간 (ms) _EX) `2000` = 2s_

## 예시 코드

```tsx
useLayoutEffect(() => {
  const box = document.querySelector(".box") as HTMLDivElement;
  const moveAnim = box.animate(
    [
      { transform: "translateX(0px)", offset: 0 },
      /* 애니메이션 재생의 30% 시점에서 오른쪽으로 50px, 위로 50px 이동한다. */
      { transform: "translateX(50px) translateY(-50px)", offset: 0.3 },
      { transform: "translateX(100px)", offset: 0.4 },
      { transform: "translateX(100px) translateY(-100px)", offset: 0.5 },
      { transform: "translateX(100px)", offset: 0.7 },
      { transform: "translateX(200px)", offset: 1 },
    ],
    {
      /* 2초 동안 재생하며, 1회 재생하고, 재생이 끝나면 그 상태로 멈춘다. */
      duration: 2000,
      iterations: 1,
      fill: "forwards",
    },
  );

  moveAnim.onfinish = () => {
    box.animate(
      [
        { transform: "translateX(200px) scale(1)", offset: 0 },
        { transform: "translateX(200px) scale(1.5)", offset: 1 },
      ],
      {
        duration: 2000,
        iterations: 1,
        fill: "forwards",
      },
    );
  };
}, []);
```

> `useLayoutEffect` 설명은 [[CSS]Transition](../css/transition.md) 참고

### `const box = document.querySelector(".box") as HTMLDivElement;`

- `box` 클래스가 붙은 첫 번째 요소를 찾아서 box라는 이름으로 저장한다.

> **`as HTMLDivElement`** (타입 단언)  
> `document.querySelector(".box")`의 반환 타입은 `Element | null`이다. (`box` 요소를 찾은 경우 & 못 찾은 경우)  
> 이렇게 `null` 가능성이 있는 상태에서 바로 `box.animate(...)`을 호출하면 TS가 `box가 null일 수 있음` 에러를 낸다!  
> 이를 방지하기 위해 `as HTMLDivElement`로 box의 타입을 `HTMLDivElement`(`<div>` 요소 타입)로 변경해준다. (다만 TS가 보는 타입이 바뀌어 에러를 내지 않고 넘어가는 것 뿐이고, 실제 요소가 없으면 box엔 여전히 null이 들어있어 에러가 발생한다.)

### `const moveAnim = box.animate(...);`

- `box.animate(...)`: box에 애니메이션을 적용 및 즉시 재생한다.
- `const moveAnim = ...`: `animate()`가 실행되며 반환하는 애니메이션 객체를 moveAnim이라는 이름으로 저장한다.
  > 별도 변수에 저장하는 이유: 다음 줄에서 이 애니메이션이 끝났을 때를 지정해서 다음 애니메이션을 재생시키기 위함!

### `moveAnim.onfinish = () => {...}`

- `onfinish`: 애니메이션 객체의 이벤트 핸들러 속성. 여기에 넣어 둔 함수가 애니메이션이 끝났을 때 실행된다.
- 즉 `moveAnim`이 끝나면 두 번째 애니메이션(`box.animate(...)`)이 재생된다. (CSS 방식의 `animationend` 역할)

> 두 번째 애니메이션에 `translateX(200px)`를 다시 적는 이유는 [[CSS]Animation](../css/animation.md) 참고

## 참고

[[Nexon]유저의 눈길을 사로잡는 웹 애니메이션](https://www.intelligencelabs.tech/310dadb5-6b2f-814c-a1a2-f02861e80cac)
