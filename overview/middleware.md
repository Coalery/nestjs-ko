---
description: "원문 : https://docs.nestjs.com/middleware"
---

# Middleware

미들웨어는 라우트 핸들러 전에 호출되는 함수이며, 어플리케이션의 요청-응답 사이클의 [요청](https://expressjs.com/en/4x/api.html#req) 객체와 [응답](https://expressjs.com/en/4x/api.html#res) 객체, 그리고 `next()`라는 미들웨어 함수에 접근할 수 있습니다. **next** 함수는 일반적으로 `next`라는 이름을 가진 변수로 나타냅니다.

![Middlewarese_1.png](https://docs.nestjs.com/assets/Middlewares_1.png)

Nest의 미들웨어는 기본적으로 [express](https://expressjs.com/en/guide/using-middleware.html)의 미들웨어와 동일합니다. 아래는 express 공식 문서에서 미들웨어의 기능에 대해 설명한 글입니다.

> 미들웨어 함수는 아래의 일을 할 수 있습니다.
> - 코드 실행
> - 요청 및 응답 객체에 변화 주기
> - 요청-응답 사이클 종료
> - next 미들웨어 함수 호출
> - 현재 미들웨어에서 요청-응답 사이클을 끝낼 것이 아니라면, `next()` 함수를 호출하여 다음 미들웨어 함수에게 제어를 넘겨야 합니다. 그러지 않으면, 요청이 끝나지 않아 응답이 보내지지 않습니다.

사용자 지정 Nest 미들웨어는 함수로도 만들 수 있고, `@Injectable()` 데코레이터가 붙은 클래스로도 만들 수 있습니다. 클래스는 `NestMiddleware` 인터페이스를 구현(implement)해야 하지만, 함수는 특별히 필요한 것은 없습니다. 자, 이제 클래스 메서드를 이용하여 간단한 미들웨어 기능을 만들어봅시다.

```typescript
// logger.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}
```

### 의존성 주입

Nest 미들웨어는 의존성 주입을 지원합니다. 프로바이더나 컨트롤러와 마찬가지로, 같은 모듈 안에 있는 **의존성을 주입** 받을 수 있습니다. 늘 그래왔듯이 `constructor`를 통해서 의존성을 주입 받으면 됩니다.

### 미들웨어 적용

`@Module()` 데코레이터에는 미들웨어를 넣을 공간이 없습니다. 대신에, 모듈 클래스의 `configure()` 메서드로 미들웨어를 적용시킬 수 있습니다. 미들웨어를 포함하는 모듈은 `NestModule` 인터페이스를 구현해야 합니다. 한 번 `LoggerMiddleware`를 `AppModule`에 적용시켜볼까요?

```typescript
// app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('cats');
  }
}
```

위 예시에서는, 이전에 `CatsController` 안에 정의했었던 `/cats` 라우트 핸들러에 대해 `LoggerMiddleware`를 적용시켜 보았습니다. 만약 특정 HTTP 요청 메서드(GET, POST 등)에 대해서만 미들웨어를 적용시키고 싶다면, `forRoutes()` 메서드에 경로 `path`와 HTTP 요청 메서드 `method`를 가진 객체를 넘겨주면 됩니다.

아래 예시에서는, `RequestMethod` 열거형을 사용해서 원하는 HTTP 요청 메서드를 설정하였습니다.

```typescript
// app.module.ts
import { Module, NestModule, RequestMethod, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: 'cats', method: RequestMethod.GET });
  }
}
```

> **팁**
> 
> `configure()` 메서드는 `async/await`를 사용해서 비동기로 만들 수 있습니다. 

### 라우트 와일드카드

패턴 기반 라우트도 지원합니다. 예를 들어, 별표(asterisk, *)는 **와일드카드**로 사용할 수 있으며, 어떤 조합의 문자들과도 매치됩니다.

```typescript
forRoutes({ path: 'ab*cd', method: RequestMethod.ALL });
```

`'ab*cd'` 라우트 경로는 `abcd`, `ab_cd`, `abecd` 등에 매치됩니다. 라우트 경로에는 `?`, `+`, `*`, `()`를 쓸 수 있으며, 각각은 정규표현식의 같은 문자에 대응되는 부분집합입니다. 하이픈(`-`)과 점(`.`)은 문자 그대로 해석됩니다.

> **주의**
> 
> `fastify` 패키지는 별표(`*`) 와일드카드를 더 이상 지원하지 않는 최신 버전의 `path-to-regexp` 패키지를 사용합니다. 따라서, 파라미터를 사용해야 합니다. (ex., `(.*)`, `:splat*`)

### MiddlewareConsumer

`MiddlewareConsumer`는 헬퍼 클래스이며, 미들웨어를 관리하기 위한 몇 가지의 빌트인 메서드를 제공합니다. 해당 클래스의 모든 메서드들은 [Fluent 스타일](https://en.wikipedia.org/wiki/Fluent_interface)로 간단하게 **연결(chain)** 할 수 있습니다. `forRoutes()` 메서드는 문자열 하나, 문자열 여러 개, `RouteInfo` 객체, 컨트롤러 클래스 하나, 심지어는 컨트롤러 클래스 여러 개도 들어갈 수 있습니다. 대부분의 경우에서는 아마 컨트롤러들의 리스트를 반점(,)으로 구분하여 넘겨줄 겁니다. 아래의 예는 컨트롤러 하나를 넘겨준 예시입니다.

```typescript
// app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';
import { CatsController } from './cats/cats.controller';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes(CatsController);
  }
}
```

> **팁**
> 
> `apply()` 메서드에는 미들웨어 하나를 넘겨줄 수도 있고, [여러 미들웨어](https://docs.nestjs.com/middleware#multiple-middleware)를 넘겨줄 수도 있습니다.