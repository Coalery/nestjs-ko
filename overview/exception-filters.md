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

### 사용자 지정 예외

대부분의 경우, 사용자 지정 예외를 만들 필요는 없고, 다음 섹션에서 설명하는 Nest 내장 HTTP 예외를 사용하면 됩니다. 만약 사용자 지정 예외를 만들어야만 한다면, `HttpException` 클래스를 상속받은 사용자 지정 예외들이 위치할 <strong>예외 디렉터리(Exception Hierarchy)</strong>를 만드는 것이 좋습니다. 이렇게 하면, Nest가 예외를 인식해서 자동으로 에러에 대한 응답을 처리하게 됩니다. 아래와 같은 사용자 지정 예외를 만들어봅시다.

```typescript
// forbidden.exception.ts
export class ForbiddenException extends HttpException {
  constructor() {
    super('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```

`ForbiddenException`이 `HttpException` 클래스를 상속 받았기 때문에, Nest에 내장된 예외 처리기가 원활하게 처리할 수 있습니다. 그러므로, 이 예외를 `findAll()` 메서드 내에서도 쓸 수 있게 됩니다.

```typescript
// cats.controller.ts
@Get()
async findAll() {
  throw new ForbiddenException();
}
```

### 내장된 HTTP 예외들

Nest는 `HttpException` 클래스를 상속 받은 표준 예외를 제공합니다. 이들은 `@nestjs/common` 패키지에서 찾을 수 있으며, 대부분의 일반적인 HTTP 예외들은 아래와 같이 이미 구현되어 있습니다.

- `BadRequestException`
- `UnauthorizedException`
- `NotFoundException`
- `ForbiddenException`
- `NotAcceptableException`
- `RequestTimeoutException`
- `ConflictException`
- `GoneException`
- `HttpVersionNotSupportedException`
- `PayloadTooLargeException`
- `UnsupportedMediaTypeException`
- `UnprocessableEntityException`
- `InternalServerErrorException`
- `NotImplementedException`
- `ImATeapotException`
- `MethodNotAllowedException`
- `BadGatewayException`
- `ServiceUnavailableException`
- `GatewayTimeoutException`
- `PreconditionFailedException`

### 예외 필터

많은 경우를 기본 예외 필터가 자동으로 처리하긴 하지만, 예외 층에 대한 **완전한 조작**이 필요할 때도 있습니다. 예를 들어, 로그를 남기거나, 몇 가지의 동적인 요인을 기반으로 다른 JSON 응답을 반환하고 싶을 수도 있습니다. **예외 필터**는 그러한 상황에서 쓸 목적으로 만들어졌습니다. 예외 필터를 통해 제어의 정확한 흐름과 클라이언트에게 반환될 응답의 내용을 제어할 수 있습니다.

`HttpException` 클래스의 인스턴스인 예외를 처리하는 예외 필터를 만들고, 해당 예외들에 대한 응답 로직을 구현해봅시다. 이를 위해서는, 기반 플랫폼의 `Request` 객체와 `Response` 객체에 접근할 필요가 있습니다. `Request` 객체에서는 접근하여 `url`을 가져오고, 이를 로그 정보에 포함합니다. 또, `Response` 객체에서는 `response.json()` 메서드를 이용해서 전송될 응답을 직접 처리합니다.

```typescript
// http-exception.filter.ts
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
```

> **팁**
> 
> 모든 예외 필터는 `ExceptionFilter<T>` 인터페이스를 구현해야 합니다. 그 이유는, 해당 인터페이스가 `catch(exception: T, host: ArgumentHost)` 메서드를 제공하기 때문입니다. `T`는 예외의 타입을 나타냅니다.

`@Catch(HttpException)` 데코레이터는 예외 필터에 메타데이터를 붙여서, Nest에게 해당 필터가 `HttpException` 타입의 예외만 처리한다고 알려주는 역할을 합니다. `@Catch()` 데코레이터는 단일 매개변수나 반점(,)으로 구분된 매개변수들을 받을 수 있습니다. 이를 통해 여러 예외 타입에 대한 필터를 한 번에 설정할 수 있습니다.

### ArgumentsHost

`catch()` 메서드의 매개변수를 한 번 봅시다. `exception` 매개변수는 현재 처리할 예외 객체이고, `host` 매개변수는 `ArgumentHost`의 객체입니다. `ArgumentHost`는 강력한 유틸리티 객체이며, [실행 컨텍스트 챕터](https://docs.nestjs.com/fundamentals/execution-context)*에서 더 자세히 설명합니다. 위의 예시에서는, 예외가 발생한 컨트롤러의 요청 핸들러에 들어오는 `Request` 객체와 `Response` 객체를 가져오기 위해 `ArgumentsHost`의 헬퍼 메서드를 사용했습니다. `ArgumentsHost`의 자세한 설명은 [여기](https://docs.nestjs.com/fundamentals/execution-context)를 참고하세요.

\* 이렇게 추상화를 한 이유는, `ArgumentsHost`가 모든 컨텍스트에서 동작하기 때문입니다. 위에서 나왔던 HTTP 서버에서도 잘 동작하고, 그 외에 마이크로서비스, 웹소켓에서도 동작합니다. 실행 컨텍스트 챕터에서는 **어느** 실행 컨텍스트에서든 `ArgumentsHost`와 헬퍼 함수로 적절한 [기반 인수](https://docs.nestjs.com/fundamentals/execution-context#host-methods)에 접근하는 방법을 알아볼 것입니다. 이를 통해 어떤 컨텍스트에서든 잘 동작하는 포괄적인 예외 필터를 만들 수 있습니다.

### 필터 적용하기

이제 `HttpExceptionFilter`를 `CatsController`의 `create()` 메서드에 적용시켜봅시다.

```typescript
// cats.controller.ts
@Post()
@UseFilters(new HttpExceptionFilter())
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
```

> **팁**
> 
> `@UseFilter()` 데코레이터는 `@nestjs/common` 패키지에 있습니다.

여기서는 `@UseFilters()` 데코레이터를 사용했습니다. `@Catch()` 데코레이터와 유사하게, 단일 필터 인스턴스나 반점(,)으로 구분된 필터 인스턴스들의 리스트를 넣을 수 있습니다. 위의 예시에서는 `HttpExceptionFilter`의 인스턴스를 만들어서 넣었습니다. 또는 인스턴스 대신에 클래스를 전달해서, 프레임워크에게 인스턴스화의 역할을 맡기고 **의존성 주입**을 가능하게도 만들 수 있습니다.

```typescript
// cats.controller.ts
@Post()
@UseFilters(HttpExceptionFilter)
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
```

> **팁**
> 
> 웬만하면 인스턴스 대신 클래스를 쓰는게 더 낫습니다. Nest가 같은 클래스의 인스턴스를 전체 모듈에서 재사용하여 메모리 사용량을 줄일 수 있기 때문입니다.

위의 예시에서는, `HttpExceptionFilter`를 `create()`라는 하나의 라우트 핸들러에 적용하여, 필터를 메서드 수준(method-scoped)에서 사용하였습니다. 예외 필터는 여러 다른 수준(메서드, 컨트롤러, 전역)에서 사용할 수 있습니다. 예를 들어, 컨트롤러 수준에서 필터를 적용하려면 아래와 같이 하면 됩니다.

```typescript
// cats.controller.ts
@UseFilters(new HttpExceptionFilter())
export class CatsController {}
```

위와 같이 하면, `CatsController` 내에 정의된 모든 라우트 핸들러에 `HttpExceptionFilter`를 적용할 수 있습니다.

전역 수준 필터를 적용하려면, 아래와 같이 하면 됩니다.

```typescript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new HttpExceptionFilter());
  await app.listen(3000);
}
bootstrap();
```

> **주의**
> 
> `useGlobalFilters()` 메서드는 게이트웨이나 하이브리드 어플리케이션에 대해서는 필터를 적용하지 않습니다.

전역 수준 필터는 전체 어플리케이션, 즉 모든 컨트롤러와 모든 라우트 핸들러에 적용됩니다. 하지만, 위의 `useGlobalFilters()`를 사용한 예시처럼 모듈 밖에서 등록된 전역 필터는 말 그대로 모듈 밖의 컨텍스트에서 완료되었기 때문에, 의존성을 주입할 수 없게 됩니다. 이 문제를 해결하려면, 아래와 같이 모듈에 직접 전역 수준 필터를 등록하면 됩니다.

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: HttpExceptionFilter,
    },
  ],
})
export class AppModule {}
```

> **팁**
> 
> 위와 같이 필터에 대한 의존성 주입을 시행하면, 위의 구문이 어디있던 상관 없이 필터는 전역으로 설정됩니다. 또한, `useClass`만이 사용자 지정 프로바이더를 등록하는 유일한 방법인 것은 아닙니다. 자세한 건 [여기](https://docs.nestjs.com/fundamentals/custom-providers)를 참고하세요.

위의 방법을 사용하여 필요한 만큼 필터를 추가할 수 있습니다. 각각을 `providers` 배열에 추가하기만 하면 됩니다.

### 모든 예외 처리하기

예외의 타입과 상관 없이 모든 처리되지 않은 예외를 처리하려면, `@Catch()` 데코레이터의 매개변수 리스트를 빈 상태로 두면 됩니다.

아래의 예시에서는, 클라이언트에게 응답을 전달하기 위해 [HTTP adapter](https://docs.nestjs.com/faq/http-adapter)를 사용하여 플랫폼에 독립적인 코드를 만들었으며, `Request`나 `Response`와 같이 특정 플랫폼에 대한 객체는 사용하지 않았습니다.

```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { HttpAdapterHost } from '@nestjs/core';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  constructor(private readonly httpAdapterHost: HttpAdapterHost) {}

  catch(exception: unknown, host: ArgumentsHost): void {
    // In certain situations `httpAdapter` might not be available in the
    // constructor method, thus we should resolve it here.
    const { httpAdapter } = this.httpAdapterHost;

    const ctx = host.switchToHttp();

    const httpStatus =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const responseBody = {
      statusCode: httpStatus,
      timestamp: new Date().toISOString(),
      path: httpAdapter.getRequestUrl(ctx.getRequest()),
    };

    httpAdapter.reply(ctx.getResponse(), responseBody, httpStatus);
  }
}
```