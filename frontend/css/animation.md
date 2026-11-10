# Animation

_2026.11.10._

> 특정 요소를 사전에 정의해 둔 순서표대로 움직이게 하는 기능

- 순서표(`@keyframes`)를 미리 만들고, 이를 재생(`animation`)한다.
- 화면에 나타나자마자 스스로 재생된다.
- 애니메이션이 끝나면 시작 자리로 돌아감. (끝 위치에 멈추게 하려면 추가 설정(`forwards`) 필요)

## `@keyframes` 예시 코드

```css
/* move-right이라는 이름의 @keyframes 정의 */
@keyframes move-right {
  /* 전체 재생 시간 중 시점을 의미. 0% -> 0초의 상태 */
  0% {
    transform: translateX(0) translateY(0px); /* 제자리 */
  }
  50% {
    transform: translateX(50px) translateY(50px); /* 오른쪽 아래로 이동(translateY는 양수 값이면 아래로 움직인다) */
  }
  100% {
    transform: translateX(100px) translateY(0px); /* 오른쪽, 원래 높이로 이동*/
  }
}
```

## `animation` 예시 코드

```css
/* box라는 클래스의 css 정의. */
.box {
  animation: move-right 2s forwards;
  /* move-right이라는 @keyframes을 2초 동안 재생한다. */
  /* forwards: 애니메이션 재생이 끝난 위치에 그대로 멈춘다. */
}
```

## 여러 애니메이션 연속적으로 실행시키기

애니메이션을 하나만 재생하면 될 때는 CSS만으로 충분하다. (스스로 재생되니)
그러나 여러 애니메이션을 순서대로 재생하고 싶을 때, 첫 애니메이션이 끝나는 순간을 감지해서 두 번째 애니메이션으로 바꿔 주는 제어가 필요하다. 이것을 JS가 맡아서 하게 된다!
(CSS만으로도 쉼표와 지연 시간으로 이어 붙일 수는 있지만, 끝나는 시점에 맞춰 바꾸려면 JS가 편하다.)

```css
/* 오른쪽으로 이동하는 애니메이션 */
@keyframes move-right {
  0% {
    transform: translateX(0) translateY(0px);
  }
  50% {
    transform: translateX(50px) translateY(50px);
  }
  100% {
    transform: translateX(100px) translateY(0px);
  }
}

/* 크기가 커지는 애니메이션 */
@keyframes scale-up {
  /* 바로 앞 애니메이션이 끝난 xy좌표를 적어줘서 애니메이션이 끊김 없이 재생되게 한다. (y좌표도 0px이 아닐 경우 적어줘야 함) */
  0% {
    transform: translateX(100px) scale(1);
  }
  100% {
    transform: translateX(100px) scale(1.5);
  }
}

.box {
  animation: move-right 2s forwards; /* box 클래스 요소 - 처음에 move-right를 재생한다. */
}
```

```tsx
useLayoutEffect(() => {
  const box = document.querySelector(".box");
  box.addEventListener("animationend", () => {
    box.style.animation = "scale-up 1s forwards";
  });
}, []);
```

> `useLayoutEffect` 설명은 [[CSS]Transition](./transition.md) 참고

### `const box = document.querySelector(".box");`

- `box` 클래스가 붙은 요소를 찾아서 box라는 이름으로 저장한다.

### `box.addEventListener("animationend", () => {...})`

- `animationend`: 애니메이션이 끝났을 때 브라우저가 알려주는 이벤트
- `box`의 애니메이션이 끝나면 안의 함수를 실행한다.

### `box.style.animation = "scale-up 1s forwards";`

- `box`의 `animation` 값을 `scale-up`으로 바꾼다. 1초 동안 재생하고, 끝난 상태로 멈춘다.

## 참고

[[Nexon]유저의 눈길을 사로잡는 웹 애니메이션](https://www.intelligencelabs.tech/310dadb5-6b2f-814c-a1a2-f02861e80cac)
