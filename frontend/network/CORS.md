# CORS

_2026.11.15._

> **CORS(Cross-Origin Resource Sharing)**  
> 웹 페이지가 자신과 다른 출처인 서버에 요청해 가져온 리소스를 읽어도 되는지 서버가 브라우저에게 허락해 주는 규칙

## 배경

- **Same-Origin Policy**: 브라우저에서는 기본적으로 같은 출처(Origin)끼리만 리소스를 주고받을 수 있다는 보안 정책이다.
- 한 브라우저 안에서는 여러 사이트의 JS가 사용자의 로그인 상태, 네트워크 접근 권한을 공유한 채 실행된다.
- 따라서 사용자가 악성 사이트에 접속할 경우 그 사이트의 JS가 브라우저를 통해 사용자의 권한으로 다른 사이트에 요청을 보내서 개인정보를 읽어갈 수 있는 위험이 있다. _**Same-Origin Policy**는 이를 막기 위한 정책_
- 그런데 실무에서는 서로 다른 출처에서 요청을 주고받아야 하는 경우가 흔하다. 그러므로 서버가 허용한 일부 출처는 응답을 읽을 수 있도록 예외를 열어 주는 것이 필요하고, 이것이 **CORS**이다.

> **출처(Origin): 프로토콜 + 도메인 + 포트**  
> 셋 중 하나라도 다를 경우 다른 출처이다.  
> _EX) `https://a.com` ≠ `http://api.a.com` (프로토콜, 도메인 다름)_

## 동작 흐름

- **JS와 서버는 직접 통신하지 않는다**. 중간의 브라우저가 통신을 매개한다.

1. JS가 다른 출처의 서버로 요청을 보내도록 브라우저에 요청한다.
2. 브라우저가 해당 요청에 `Origin` 헤더(현재 페이지의 출처)를 붙여 전송한다.
3. 서버는 요청에 대한 응답을 돌려주고, 이때 요청의 `Origin`이 허용 출처일 경우 `Access-Control-Allow-Origin` 헤더에 해당 출처를 담아 보낸다. (아닐 경우 아예 헤더를 보내지 않는다.)
4. 브라우저는 헤더에 담겨 온 출처와 현재 페이지의 출처를 비교한다.
5. 두 출처가 일치하면 응답을 페이지의 JS에 전달하고, 일치하지 않거나 헤더가 없을 경우 응답을 JS에 전달하지 않고 CORS 에러를 발생시킨다.

_위 흐름은 Simple Request 기준이며, Preflight Request는 본 요청 전에 [사전 확인 과정](#사전-요청-흐름)이 추가된다._

> **Postman, curl은 CORS 에러가 나지 않는다**  
> 둘 다 HTTP 요청을 보내는 프로그램. 브라우저와 달리 출처 검사를 수행하지 않는다.
>
> - Postman: API 테스트용 GUI 프로그램.
> - curl: 터미널에서 사용하는 명령어. _EX) `curl https://api.b.com/users`_
>
> 브라우저는 여러 사이트의 JS가 사용자의 로그인 상태, 네트워크 접근 권한을 공유한 채 실행되기 때문에 다른 사이트의 JS가 사용자의 권한을 악용할 위험이 있다.  
> 그러나 Postman이나 curl은 사용자가 직접 실행하는 도구이기 때문에 다른 사이트의 JS가 함께 실행될 일이 없다. 따라서 별도 출처 검사를 하지 않고 CORS라는 개념 자체도 없다.

## 요청 종류

- CORS가 생기기 전에도 웹 페이지는 HTML의 `<form>` 태그로 다른 출처에 일부 형태의 요청을 보낼 수 있었다.
- 이러한 기존 요청들은 서버도 외부 출처에서 올 수 있음을 알고 내부적으로 적절히 처리하므로, 브라우저는 별도 확인 절차 없이 바로 요청을 보낸다.
- 반면 기존에는 다른 출처에서 보낼 수 없었다가 새롭게 가능하게 된 요청의 경우, 서버는 여전히 이 요청들은 자기 사이트에서만 온다고 가정하고 별도 검사 없이 처리할 수 있다. 따라서 브라우저에서 먼저 해당 요청을 보내도 될지 물어보는 절차가 추가로 필요하다.

### Simple Request

- 기존에도 외부 출처에서 올 수 있었던 요청들이다.
- 아래의 조건을 모두 만족한다.
  - 메서드가 `GET`, `POST`, `HEAD` 중 하나
  - Content-Type이 `text/plain`, `multipart/form-data`, `application/x-www-form-urlencoded` 중 하나
  - 브라우저가 자동으로 붙이는 헤더 외에 직접 추가한 헤더가 없음 (`Accept`, `Accept-Language`, `Content-Language` 등 몇 가지는 직접 추가해도 허용)

### Preflight Request

- 새롭게 외부 출처에서 올 수 있게 된 요청들이다.
- Simple Request 조건을 하나라도 벗어날 경우 해당한다.
- 자주 쓰이는 경우는 아래와 같다.
  - Content-Type: `application/json`으로 JSON 데이터를 보낼 때
  - `Authorization` 헤더로 토큰을 보낼 때
  - `PUT`, `PATCH`, `DELETE` 메서드를 쓸 때
  - 그 외 직접 만든 헤더(`X-Request-Id` 등)를 붙일 때

#### 사전 요청 흐름

- `http://localhost:3000` 페이지에서 `http://localhost:8080/users`로 JSON과 토큰을 담은 POST 요청을 보내는 예시

```http
OPTIONS /users
Origin: http://localhost:3000
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization
```

> 1. 브라우저가 OPTIONS 메서드로 자동으로 사전 요청을 보냄

- Origin: 현재 페이지의 출처
- Access-Control-Request-Method: 본 요청에서 쓰려는 메서드
- Access-Control-Request-Headers: 본 요청에 붙이려는 헤더 목록

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

> 2. 서버가 허용 범위를 응답 헤더로 알려줌

- Access-Control-Allow-Origin: 허용하는 출처
- Access-Control-Allow-Methods: 허용하는 메서드 목록
- Access-Control-Allow-Headers: 허용하는 요청 헤더 목록
- Access-Control-Max-Age: 이 결과를 브라우저가 기억해 둘 시간(초)

```http
POST /users
Origin: http://localhost:3000
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...

{"name": "kim"}
```

> 3. 사전 요청의 출처, 메서드, 헤더가 서버의 허용 범위에 모두 들어 있을 경우 본 요청을 보냄  
>    (하나라도 허용 범위를 벗어날 경우 본 요청을 보내지 않고 CORS 에러를 발생시킴)

> 4. 이후 [동작 흐름](#동작-흐름)으로 이어짐

## 쿠키 설정 (Credentialed Request)

- 브라우저는 다른 출처로 요청을 보낼 때 기본적으로 쿠키를 포함하지 않는다.
- 따라서 쿠키 기반 로그인 등에서 프론트와 백엔드의 출처가 다를 경우, 클라이언트와 서버 양쪽 모두 별도 설정이 필요하다.

### 클라이언트 설정

```ts
fetch(url, { credentials: "include" }); // fetch 사용할 때
axios.get(url, { withCredentials: true }); // axios 사용할 때
```

> 다른 출처로 보내는 요청에도 쿠키를 포함하도록 지정.  
> (사용 라이브러리에 따라 둘 중 한 줄로 설정한다)

### 서버 설정

```http
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Credentials: true
```

> 쿠키가 포함된 요청이어도 응답을 JS에 전달하도록 허용

> **와일드카드(`*`) 사용 불가능**  
> `Access-Control-Allow-Origin: *`은 모든 출처를 허용한다는 의미이다.  
> 그러나 쿠키가 포함된 요청의 응답에는 사용자의 개인정보가 담기므로, 이를 모든 출처에 허용하면 [배경](#배경)의 위험이 발생한다.  
> 따라서 Credentialed Request 설정을 하는 경우 `*` 사용이 불가능하며 허용 출처를 명확히 지정해야 한다.

> 토큰을 쿠키가 아닌 `Authorization` 헤더로 보내는 경우 credentials 설정은 필요 없다.

## 프록시

- 백엔드 CORS 설정을 바로 바꾸기 어려운 프론트엔드 개발 환경에서 사용하는 우회 방법.
- 브라우저 대신 개발 서버가 요청을 백엔드로 전달하여 브라우저와 다른 출처 간의 직접 통신을 막는다.

```ts
/* vite.config.ts - Vite 기준 코드 */
server: {
  proxy: {
    "/api": "http://localhost:8080",
  },
}
```

> `/api`로 시작하는 요청을 `http://localhost:8080`으로 전달

1. 클라이언트가 `/api/users`와 같이 별도 출처 없이 요청한다.
2. 출처가 없으므로 브라우저는 현재 페이지와 같은 출처인 프론트엔드 개발 서버(`localhost:3000`)로 요청을 보낸다. 같은 출처이므로 출처 검사가 발생하지 않는다.
3. 개발 서버가 대신 요청을 `http://localhost:8080/api/users`로 전달한다.
4. 백엔드가 개발 서버에 응답한다. 서버 간 통신이므로 CORS가 적용되지 않는다.
5. 개발 서버가 응답을 그대로 브라우저에 돌려준다.

> 개발 서버가 실행 중일 때만 동작하므로, 배포 환경에서는 서버 CORS 설정이나 nginx 등의 별도 프록시 설정이 필요하다.
