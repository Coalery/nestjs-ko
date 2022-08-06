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
> 웹소켓 게이트웨이는 싱글톤으로써 동작해야 하므로, request-scoped 프로바이더를 사용하면 안됩니다. 각각의 게이트웨이는 실제 소켓을 캡슐화하며, 여러 번 인스턴스화 될 수 없습니다. 이러한 제한은 [Passport 전략](https://docs.nestjs.com/security/authentication#request-scoped-strategies)이나 Cron 컨트롤러 같은 다른 프로바이더에도 적용됩니다.

### 컨트롤러 스코프

컨트롤러도 스코프를 가질 수 있으며, 이는 해당 컨트롤러 안에 선언되어 있는 모든 요청 핸들러 메소드에 적용됩니다. 프로바이더 스코프처럼, 컨트롤러의 스코프도 자신의 생명주기를 정의합니다. request-scoped 컨트롤러의 경우, 들어오는 각각의 요청에 대해 새로운 인스턴스가 만들어지고, 요청 처리가 완료되면 삭제됩니다(garbage-collected).

컨트롤러 스코프는 `ControllerOptions` 객체의 `scope` 프로퍼티를 통해 선언할 수 있습니다.

```typescript
@Controller({
  path: "cats",
  scope: Scope.REQUEST,
})
export class CatsController {}
```

### 문서 기여자

- [러리](https://github.com/Coalery)
