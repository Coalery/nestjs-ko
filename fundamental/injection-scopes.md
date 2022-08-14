# Injection scopes

원문 : [https://docs.nestjs.com/fundamentals/injection-scopes](https://docs.nestjs.com/fundamentals/injection-scopes)

다른 프로그래밍 언어 배경을 가진 사람들은 Nest에 들어오는 요청들 간의 거의 모든 것이 공유된다는 것을 받아들이기 힘들 수 있습니다. Nest는 데이터베이스에 대한 연결 풀, 전역 상태의 싱글톤 서비스 등, 여러 공유되는 리소스를 갖고 있습니다. 왜 이렇게 설계되었는지를 이해하려면, 먼저 Node.js는 각각의 요청을 분리된 쓰레드로 처리하는 무상태 요청/응답 멀티 쓰레드 모델을 따르지 않는다는 것을 알아야 합니다. Nest는 Node.js 위에서 동작하기 때문에, 싱글톤 인스턴스를 사용하는 것이 우리의 환경에서는 가장 **안전**합니다.

그러나, GraphQL 어플리케이션에서 각 요청에 대한 캐싱을 하거나, 요청 트래킹, 멀티테넌시(multi-tenancy) 등 요청 기반 수명을 갖는 컨트롤러가 필요한 경우 같은 예외 상황도 존재합니다. 주입 스코프는 원하는 프로바이더 수명 주기를 설정할 수 있는 메커니즘을 제공합니다.

### 프로바이더 스코프

프로바이더는 아래의 스코프를 가질 수 있습니다.

|   스코프    | 설명                                                                                                                                                                                                                                             |
| :---------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|  `DEFAULT`  | 프로바이더의 인스턴스 단 한 개가 전체 어플리케이션에 공유됩니다. 인스턴스의 수명은 어플리케이션의 생명주기와 직접 연결됩니다. 어플리케이션이 시작(bootstrap)되면, 모든 싱글톤 프로바이더가 인스턴스화 됩니다. 기본적으로 이 스코프를 사용합니다. |
|  `REQUEST`  | 각각의 들어오는 **요청**에 대해 독립적으로 새로운 프로바이더 인스턴스가 만들어집니다. 인스턴스는 요청 처리가 완료되었을 때 삭제됩니다(garbage-collected).                                                                                        |
| `TRANSIENT` | 일시적 프로바이더는 공유되지 않습니다. 일시적 프로바이더를 주입받는 각각의 위치에서는 새로운 전용 인스턴스를 받게 됩니다.                                                                                                                        |

> **팁**
>
> 대부분의 경우에서 싱글톤 스코프를 사용하는 걸 **추천드립니다**. 프로바이더를 사용하는 곳이나 요청에 대해 프로바이더를 공유하는 것은, 인스턴스가 어플리케이션이 시작할 때만 단 한 번 초기화된 뒤에 캐싱해서 사용할 수 있다는 걸 뜻합니다.

### 사용법

`@Injectable()` 데코레이터의 `scope` 프로퍼티로 옵션 객체를 넘겨줌으로써 주입 스코프를 명시할 수 있습니다.

```typescript
import { Injectable, Scope } from "@nestjs/common";

@Injectable({ scope: Scope.REQUEST })
export class CatsService {}
```

비슷하게 [커스텀 프로바이더](https://docs.nestjs.com/fundamentals/custom-providers)는 프로바이더 등록 시에 `scope` 프로퍼티를 주면 됩니다.

```typescript
{
  provide: 'CACHE_MANAGER',
  useClass: CacheManager,
  scope: Scope.TRANSIENT,
}
```

> **팁**
>
> `@Scope` 열거형은 `@nestjs/common` 패키지에서 가져올 수 있습니다.

싱글톤 스코프가 기본적으로 사용되며, 굳이 선언할 필요는 없습니다. 프로바이더를 싱클톤 스코프로 정의하고 싶다면, `scope` 프로퍼티에 `Scope.DEFAULT` 값을 사용하면 됩니다.

> **알림**
>
> 웹소켓 게이트웨이는 싱글톤으로써 동작해야 하므로, `REQUEST` 스코프 프로바이더를 사용하면 안됩니다. 각각의 게이트웨이는 실제 소켓을 캡슐화하며, 여러 번 인스턴스화 될 수 없습니다. 이러한 제한은 [Passport 전략](https://docs.nestjs.com/security/authentication#request-scoped-strategies)이나 Cron 컨트롤러 같은 다른 프로바이더에도 적용됩니다.

### 컨트롤러 스코프

컨트롤러도 스코프를 가질 수 있으며, 이는 해당 컨트롤러 안에 선언되어 있는 모든 요청 핸들러 메소드에 적용됩니다. 프로바이더 스코프처럼, 컨트롤러의 스코프도 자신의 생명주기를 정의합니다. `REQUEST` 스코프 컨트롤러의 경우, 들어오는 각각의 요청에 대해 새로운 인스턴스가 만들어지고, 요청 처리가 완료되면 삭제됩니다(garbage-collected).

컨트롤러 스코프는 `ControllerOptions` 객체의 `scope` 프로퍼티를 통해 선언할 수 있습니다.

```typescript
@Controller({
  path: "cats",
  scope: Scope.REQUEST,
})
export class CatsController {}
```

### 스코프 계층

`REQUEST` 스코프는 주입 체인이 확 커지게 만듭니다(bubbles up). `REQUEST` 스코프 프로바이더에 의존하는 컨트롤러 자신도 `REQUEST` 스코프가 됩니다.

`CatsController <- CatsService <- CatsRepository` 의존성 그래프를 상상해봅시다. `CatsService`가 `REQUEST` 스코프이고 다른 건 모두 기본 싱글톤이라 했을 때, `CatsController`는 주입된 서비스에 의존하고 있기 때문에 `REQUEST` 스코프가 됩니다. 아무 곳에도 의존하지 않는 `CatsRepository`는 계속 싱글톤 스코프로 남아있습니다.

`TRANSIENT` 스코프 의존성은 위의 패턴을 따르지 않습니다. 만약 싱글톤(`DEFAULT`) 스코프의 `DogsService`가 `TRANSIENT` 스코프의 `LoggerService` 프로바이더를 주입 받으면, 해당 서비스의 새 인스턴스를 받게 됩니다. 하지만, `DogsService`는 계속 싱글톤 스코프로 남게 되며, 해당 서비스를 주입 받는 어느 곳이던 `DogsService`의 새로운 인스턴스를 만들지 않습니다. 만약 `DogsService`도 주입 받을 때 새로운 인스턴스를 만들고 싶다면, 해당 서비스도 명시적으로 `TRANSIENT`를 지정해주어야 합니다.

### REQUEST 프로바이더

HTTP 서버 기반 어플리케이션(`@nestjs/platform-express`, `@nestjs/platform-fastify` 등)에서, `REQUEST` 스코프 프로바이더에서 원래의 요청 객체의 레퍼런스에 접근하고 싶을 수도 있습니다. 이 때는 `REQUEST` 객체를 주입 받으면 됩니다.

```typescript
import { Injectable, Scope, Inject } from "@nestjs/common";
import { REQUEST } from "@nestjs/core";
import { Request } from "express";

@Injectable({ scope: Scope.REQUEST })
export class CatsService {
  constructor(@Inject(REQUEST) private request: Request) {}
}
```

기반 플랫폼/프로토콜의 차이 때문에, 마이크로서비스나 GraphQL 어플리케이션의 경우 들어오는 요청에는 살짝 다르게 접근해야 합니다. [GraphQL](https://docs.nestjs.com/graphql/quick-start) 어플리케이션에는 `REQUEST` 대신에 `CONTEXT`를 주입받아야 합니다.

```typescript
import { Injectable, Scope, Inject } from "@nestjs/common";
import { CONTEXT } from "@nestjs/graphql";

@Injectable({ scope: Scope.REQUEST })
export class CatsService {
  constructor(@Inject(CONTEXT) private context) {}
}
```

그런 다음 `request`를 프로퍼티로 포함하도록 `context` 값을 `GraphQLModule`에서 구성하면 됩니다.

### INQUIRER 프로바이더

로깅이나 통계에 사용하기 위해 프로바이더가 만들어진 클래스를 가져오고 싶다면,`INQUIRER` 토큰으로 주입받으면 됩니다.

```typescript
import { Inject, Injectable, Scope } from "@nestjs/common";
import { INQUIRER } from "@nestjs/core";

@Injectable({ scope: Scope.TRANSIENT })
export class HelloService {
  constructor(@Inject(INQUIRER) private parentClass: object) {}

  sayHello(message: string) {
    console.log(`${this.parentClass?.constructor?.name}: ${message}`);
  }
}
```

그런 다음, 아래와 같이 사용하면 됩니다.

```typescript
import { Injectable } from "@nestjs/common";
import { HelloService } from "./hello.service";

@Injectable()
export class AppService {
  constructor(private helloService: HelloService) {}

  getRoot(): string {
    this.helloService.sayHello("My name is getRoot");

    return "Hello world!";
  }
}
```

위의 예시에서 `AppService#getRoot`를 호출하면, `"AppService: My name is getRoot"`가 콘솔에 나옵니다.

### 성능

`REQUEST` 스코프 프로바이더를 사용하면 어플리케이션 성능에 영향을 줄 수 있습니다. Nest가 최대한 많은 메타데이터를 캐싱하려고 하긴 하지만, 각각의 요청에 대하여 클래스의 인스턴스를 새로 만들어주어야 합니다. 따라서, 평균 응답 시간이나 전체적인 벤치마킹 결과가 느려질겁니다. `REQUEST` 스코프 프로바이더를 꼭 사용해야만 하는 상황이 아니라면, 기본 싱글톤 스코프를 사용하는 걸 강력하게 추천합니다.

> **팁**
>
> 위에서 지나치게 위협적이긴 했지만, `REQUEST` 스코프 프로바이더를 사용하더라도 잘 디자인된 어플리케이션은 최대 5% 정도로만 느려집니다.

### 문서 기여자

- [러리](https://github.com/Coalery)
