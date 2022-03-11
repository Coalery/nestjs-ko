---
description: "원문 : https://docs.nestjs.com/exception-filters"
---

# Exception filters

Nest는 어플리케이션의 모든 처리되지 않은 예외를 처리하는 역할을 하는 <strong>예외 층(Exceptions Layer)</strong>를 갖고 있습니다. 어플리케이션 코드에서 예외가 처리되지 않았을 때, 해당 예외가 예외 층에서 잡혀서, 자동으로 적당한 유저 친화적 응답을 전송합니다.

![Filter_1.png](https://docs.nestjs.com/assets/Filter_1.png)

위 행위(예외 처리)는 내장된 **전역 예외 필터**에 의해 실행되며, `HttpException`과 그 자식 클래스에 대한 예외를 처리합니다. 만약 예외가 `HttpException`도 아니고 그 자식 클래스도 아니어서 예외를 인식할 수 없을 때에는, 내장된 예외 필터가 아래의 기본 JSON 응답을 만들어냅니다.

```json
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

> **팁**
> 
> 전역 예외 필터는 부분적으로 `http-errors` 라이브러리를 지원합니다. 기본적으로 `statusCode`와 `message` 속성을 가진 예외는 적절히 채워져서 응답으로 보내집니다. 그 외의 인식되지 않은 예외는 `InternalServerErrorException`으로 처리됩니다.

### 표준 예외 발생시키기

Nest는 `@nestjs/common` 패키지에서 `HttpException` 클래스를 제공합니다. 일반적인 HTTP REST/GraphQL API 기반 어플리케이션은, 어떤 에러가 발생했을 때 표준 HTTP 응답 객체를 전송하게 됩니다.

`CatsController`의 `findAll()` 메서드로 예를 들어보겠습니다. 이 라우트 핸들러가 어떤 이유로 인해서 아래와 같이 예외를 발생시켜야 한다고 가정해봅시다.

```typescript
// cats.controller.ts
@Get()
async findAll() {
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
}
```

> **팁**
> 
> 여기서 쓴 `HttpStatus`는 `@nestjs/common` 패키지의 헬퍼 열거형입니다.

클라이언트가 이 엔드포인트를 호출하면, 아래와 같이 응답이 오게 됩니다.

```json
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

`HttpException` 생성자는 응답을 결정하는 두 가지의 인수가 필요합니다.

- `response` 인수는 JSON 응답입니다. `string`이 될 수도 있고, 아래에서 설명할 `object`가 될 수도 있습니다.
- `status` 인수는 [HTTP 상태 코드](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)입니다.

기본적으로 JSON 응답은 두 속성을 갖습니다.

- `statusCode`: `status` 인수로 들어온 HTTP 상태 코드입니다.
- `message`: `status`에 기반한 간단한 HTTP 에러 설명입니다.

JSON 응답의 `message` 부분만 바꾸고 싶으면, `response` 인수로 문자열을 전달하면 됩니다. 반대로 전체 JSON 응답을 바꾸고 싶으면, `response` 인수로 객체를 전달하면 됩니다. 그렇게 하면, Nest가 객체를 직렬화하고 JSON 응답으로 반환하게 됩니다.

생성자의 두 번째 인수인 `status`는 유효한 HTTP 상태 코드여야 합니다. 실수를 방지하기 위해선, `@nestjs/common`의 `HttpStatus` 열거형을 사용하는 것이 좋습니다.

아래는 전체 응답을 바꾼 예시입니다.

```typescript
// cats.controller.ts
@Get()
async findAll() {
  throw new HttpException({
    status: HttpStatus.FORBIDDEN,
    error: 'This is a custom message',
  }, HttpStatus.FORBIDDEN);
}
```

위와 같이 하면, 응답은 아래와 같습니다.

```json
{
  "status": 403,
  "error": "This is a custom message"
}
```