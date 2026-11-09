# Transition

_2026.11.09._

> CSS 속성 값이 바뀔 때, 정해진 시간 동안 부드럽게 바뀌도록 해 주는 기능

- 스스로 동작하지 않음. CSS 값을 변경하는 계기(호버링, 클래스 추가 등)가 있어야 한다.
- 시작 값과 끝 값, 두 가지 상태만 존재한다. 그 사이는 브라우저 단에서 알아서 채운다.

## 배경색 변경 예시 코드

```css
/* button이라는 클래스의 css 정의. css에서 클래스를 가리킬 때는 앞에 .을 붙인다. */
.button {
  background: blue; /* 기본 배경색은 파란색 */
  transition: background 0.5s; /* 배경색이 바뀔 때 0.5초 동안 서서히 바뀐다. */
}
/* button 클래스를 가진 요소에 호버링했을 때 css 정의. (호버링 뗄 때에도 동일하게 transition 적용) */
.button:hover {
  background: red; /* 호버링하면 배경색이 빨간색으로 바뀐다. */
}
```

## 위치 이동 예시 코드

```css
.box {
  transition: all 2s; /* 모든 css 속성의 값 변경에 대해 2초 동안 서서히 바뀐다. */
  transform: translateX(0px); /* 시작 위치: 제자리 */
}
/* box, tr 클래스를 둘 다 가진 요소일 때. 아래 css가 적용되며(css 값이 변경되며) transition이 일어난다. */
.box.tr {
  transform: translateX(100px); /* 기존 위치에서 X좌표로 100px만큼 이동 */
}
```

```tsx
useLayoutEffect(() => {
  setTimeout(() => {
    document.querySelector(".box").classList.add("tr");
  }, 100);
}, []);
```

### `useLayoutEffect(()=>{...},[]);`

- `() => {...}`: 실행할 내용
- `[]`: "의존성 배열" 첫 실행 후 언제 실행할 지 정의. 비어있는 경우 한 번만 실행

=> 컴포넌트 첫 렌더링 때 한 번 실행

### `setTimeout(() => {...}, 100)`

- `() => {...}`: 실행할 내용
- `100`: 몇 ms 뒤에 실행할 지 (100=0.1초)

=> 0.1초 뒤 box 클래스가 붙은 요소를 찾아서 tr 클래스를 붙이기

(렌더링 즉시 붙이면 움직임이 안 보일 수 있으니 약간의 지연을 준다.)

## 참고

[[Nexon]유저의 눈길을 사로잡는 웹 애니메이션](https://www.intelligencelabs.tech/310dadb5-6b2f-814c-a1a2-f02861e80cac)
