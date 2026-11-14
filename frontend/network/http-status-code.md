# HTTP Status Code

_2026.11.14._

> 클라이언트의 요청이 어떻게 처리되었는지 알려주는 서버가 응답의 첫 줄에 담아 보내는 세 자리 숫자.

- 인터넷 표준 기구(IETF)에 의해 정해진 것으로, 어떤 서버·클라이언트에서든 항상 같은 의미로 사용된다.
- 응답에 어떤 상태 코드를 담을지 결정하는 주체는 크게 세 가지가 있다.
  - 백엔드 개발자(코드): 비즈니스 규칙에 따라 직접 지정 _EX) 409, 403_
  - 백엔드 프레임워크: 자동 지정 _EX) 404, 500_
  - 중간 서버(게이트웨이·프록시): 백엔드로부터 정상적인 응답을 받지 못했을 때 대신 지정 _EX) 502, 504_

```http
HTTP/1.1 409 Conflict                  ← status line
Content-Type: application/json         ← header

{"message":"이미 가입된 이메일입니다"}  ← body(optional)
```

> 단순화한 HTTP 응답 메시지 예시.

## 2xx - 성공

### 200 OK

- 요청이 정상 처리됨.
- 일반적으로 응답 바디에 해당 요청의 결과를 담아 반환

### 201 Created

- 요청이 정상 처리되었고, 그 결과로 새로운 리소스가 생성됨
- 주로 `POST` 요청의 응답에 사용

### 204 No Content

- 요청이 정상 처리되었고 응답 바디가 없음
- 주로 `DELETE` 요청의 응답에 사용

## 3xx - 리다이렉션

- 요청을 완료하기 위해 클라이언트의 추가 동작이 필요하다는 의미.
- 대부분 브라우저가 자동으로 처리하기 때문에 개발자가 직접 처리해야 하는 경우는 잘 없다.

### 301 Moved Permanently

- 리소스의 주소가 영구적으로 변경되었음
- 브라우저가 이를 저장한 이후에는 바로 새 주소로 요청한다.

### 302 Found

- 리소스의 주소가 일시적으로 변경되었음
- 별도 저장하지 않고 이후에도 기존 주소로 요청한다.

### 304 Not Modified

- 클라이언트가 가진 캐시가 여전히 유효하다.
- 응답 바디가 없으며 클라이언트는 보유한 캐시를 그대로 사용

## 4xx - 클라이언트 에러

- 클라이언트의 요청에 문제가 있어 서버가 처리할 수 없다는 의미.
- 클라이언트 측에서 요청을 수정해야 한다.

### 400 Bad Request

- 요청의 형식이 올바르지 않음. _EX) 필수 값 누락, 타입 불일치 등_

### 401 Unauthorized

- 토큰이 없거나 만료되어 요청자를 식별할 수 없음
- 같은 401이라도 API에 따라 구체적인 원인이 다를 수 있으므로 적절히 처리한다.

#### 일반 API - 액세스 토큰 만료

```tsx
/* 로그인 API는 별도 처리 */
const skip = url.includes("/user/login");

if (status === 401 && !config._retry && !skip) {
  config._retry = true;
  await reissueTokens();
  return apiClient(config);
}
```

- 리프레시 토큰으로 액세스 토큰을 재발급받은 뒤 기존 요청을 재시도한다.
- 자동으로 처리되지 않으므로 클라이언트 단에서 직접 구현해야 함

> **액세스 토큰 vs 리프레시 토큰**  
> 액세스 토큰
>
> - 매 API 요청 헤더에 실어 보내는 토큰.
> - 자주 전송되어 탈취 위험이 크므로 수명이 짧다.
>
> 리프레시 토큰
>
> - 액세스 토큰 재발급 시에 사용되는 토큰.
> - 재발급 요청 시에만 전송되므로 탈취 위험이 적고, 따라서 수명이 길다.

#### 로그인 API - 이메일·비밀번호 불일치

```tsx
if (error.status === 401) {
  setSubmitError("이메일 또는 비밀번호가 올바르지 않아요.");
  return;
}
```

- 로그인 실패를 의미하므로 사용자에게 이를 안내한다.

### 403 Forbidden

- 요청자는 식별되었으나 해당 요청에 대한 권한이 없음  
  _EX) 일반 사용자가 관리자 페이지 API 호출, 다른 사람 게시글 수정·삭제 등_

```tsx
if (error.status === 403) {
  toast.error("이 작업을 수행할 권한이 없어요.");
  return;
}
```

### 404 Not Found

- 요청한 URL에 해당하는 리소스가 존재하지 않음
- 원인은 크게 두 가지
  - 정의되지 않은 엔드포인트 _EX) 경로 오타, 미배포 API_
  - 엔드포인트는 존재하나 해당 식별자의 데이터가 없음 _EX) `/stores/999`인데 999번 가게 데이터가 DB에 존재 X_

### 409 Conflict

- 요청은 유효하나 서버의 현재 상태와 충돌함.

```tsx
async function checkNicknameAvailable(nickname: string): Promise<boolean> {
  try {
    await apiClient.get(API_ENDPOINTS.user.checkNickname, {
      params: { nickname },
    });
    return true;
  } catch (error) {
    if (error instanceof ApiHttpError && error.status === 409) {
      return false;
    }
    throw error;
  }
}
```

- 닉네임 중복 확인: 409를 에러가 아닌 **사용 불가**라는 정상 결과(`false`)로 별도 처리한다.

### 422 Unprocessable Entity

- 요청 형식은 올바르나 그 내용이 유효성 규칙을 만족하지 않는다.
- 400과 경계가 모호해 둘 중 하나만 사용하는 경우가 많음

### 429 Too Many Requests

- 일정 시간 내 허용된 요청 횟수를 초과함

## 5xx - 서버 에러

- 요청은 유효하나 서버가 처리에 실패했다는 의미.

### 500 Internal Server Error

- 서버 내부에서 예상하지 못한 에러가 발생함

### 502 Bad Gateway

- 중간 서버가 백엔드로부터 유효하지 않은 응답을 받음

### 503 Service Unavailable

- 서버가 일시적으로 요청을 처리할 수 없음. _EX) 점검, 과부하 등_

### 504 Gateway Timeout

- 중간 서버가 백엔드의 응답을 제한 시간 내에 받지 못함

## 비고

- 예시 코드는 과거 프로젝트에서 실제로 사용한 코드를 일부 생략하여 작성하였다.
