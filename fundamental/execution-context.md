# Execution context

원문 : [https://docs.nestjs.com/fundamentals/execution-context](https://docs.nestjs.com/fundamentals/execution-context)

Nest는 HTTP 서버, 마이크로서비스, 웹소켓 어플리케이션 등, 여러 어플리케이션 컨텍스트에서 동작하는 여러 유틸리티 클래스를 제공하며, 이들은 쉽게 어플리케이션을 제작할 수 있도록 도와줍니다. 이 유틸리티들은 현재 실행 컨텍스트에 대한 정보를 제공합니다. 이를 사용하여 컨트롤러, 메서드, 또다른 실행 컨텍스트에서 광범위하게 동작할 수 있는 일반적인(generic) 가드, 필터, 인터셉터를 만들 수 있습니다.

이 챕터에서는 `ArgumentsHost`와 `ExecutionContext`, 두 가지 클래스에 대해서 다룹니다.

### ArgumentsHost 클래스

`ArgumentsHost` 클래스는 핸들러에 주어질 값들을 가져올 수 있는 메서드들을 제공합니다. 값을 가져올 적절한 컨텍스트를 선택할 수 있습니다. Nest는 일반적으로 `host`라는 매개변수로, `ArgumentsHost`의 인스턴스에 접근해야 하는 곳에 제공합니다. 예를 들면, 예외 필터의 `catch()` 메서드는 `ArgumentsHost` 인스턴스와 함께 호출됩니다.

`ArgumentsHost`는 단순히 핸들러의 인수에 대한 추상화 역할을 합니다. 예를 들어 HTTP 서버 어플리케이션의 경우(`@nestjs/platform-express`를 사용할 때), `host` 객체는 Express의 `[request, response, next]` 배열을 캡슐화합니다. 이때, `request`는 요청 객체, `response`는 응답 객체, 그리고 `next`는 어플리케이션의 요청-응답 사이클을 제어하는 함수입니다. 반면에, GraphQL 어플리케이션은 `host` 객체가 `[root, args, context, info]` 배열을 갖게 됩니다.

### 현재 어플리케이션 컨텍스트

여러 어플리케이션 컨텍스트에서 동작하는 가드, 필터, 인터셉터들을 만들 땐, 각각이 현재 실행되고 있는 어플리케이션의 유형을 알아낼 방법이 필요합니다. 이는 `ArgumentsHost`의 메서드인 `getType()`을 사용하면 됩니다.

```typescript
if (host.getType() === "http") {
  // do something that is only important in the context of regular HTTP requests (REST)
} else if (host.getType() === "rpc") {
  // do something that is only important in the context of Microservice requests
} else if (host.getType<GqlContextType>() === "graphql") {
  // do something that is only important in the context of GraphQL requests
}
```

> **팁**
>
> `GqlContextType`은 `@nestjs/graphql` 패키지에서 가져올 수 있습니다.

어플리케이션의 유형을 가져올 수 있으면, 이제 아래와 같이 더 일반적인 컴포넌트들을 만들 수 있습니다.

### 호스트 핸들러의 인수

핸들러에 전달될 인수들의 배열을 가져오는 한 방법은, `host` 객체의 `getArgs()` 메서드를 사용하는 것인데요.

```typescript
const [req, res, next] = host.getArgs();
```

`getArgByIndex()` 메서드를 이용하여, 인덱스를 통해 특정 인수를 뽑아올 수도 있습니다.

```typescript
const request = host.getArgByIndex(0);
const response = host.getArgByIndex(1);
```

위의 예시와 같이 인덱스를 통해 요청 객체와 응답 객체를 가져오는 것은, 특정 실행 컨텍스트에 어플리케이션이 종속(couple)될 수 있으므로, 일반적으로 추천하지는 않습니다. 대신에, 어플리케이션에 적절한 실행 컨텍스트로 바꿔주는 `host` 객체의 유틸리티 메서드를 사용하여 코드를 더욱 탄탄하고 재사용 가능하게 만들 수 있습니다. 컨텍스트 변경 유틸리티 메서드는 아래와 같습니다.

```typescript
/**
 * Switch context to RPC.
 */
switchToRpc(): RpcArgumentsHost;
/**
 * Switch context to HTTP.
 */
switchToHttp(): HttpArgumentsHost;
/**
 * Switch context to WebSockets.
 */
switchToWs(): WsArgumentsHost;
```

이전의 예시를 `switchToHttp()` 메서드를 사용하여 다시 써봅시다. `host.switchToHttp()`를 호출하면, HTTP 어플리케이션 컨텍스트에 맞는 `HttpArgumentsHost` 객체를 반환합니다. `HttpArgumentsHost` 객체는 원하는 객체를 가져오기 위하여 두 개의 유용한 메서드를 갖고 있습니다. 이 경우에는 네이티브 Express 타이핑이 된 객체를 반환하기 위해 Express 타입 단언(type assertion)을 사용합니다.

```typescript
const ctx = host.switchToHttp();
const request = ctx.getRequest<Request>();
const response = ctx.getResponse<Response>();
```

`WsArgumentsHost`와 `RpcArgumentsHost`도 마찬가지로 마이크로서비스와 웹소켓 컨텍스트 각각의 알맞은 객체를 반환하는 메서드를 갖고 있습니다. 아래는 `WsArgumentsHost`의 메서드입니다.

```typescript
export interface WsArgumentsHost {
  /**
   * Returns the data object.
   */
  getData<T>(): T;
  /**
   * Returns the client object.
   */
  getClient<T>(): T;
}
```

아래는 `RpcArgumentsHost`의 메서드입니다.

```typescript
export interface RpcArgumentsHost {
  /**
   * Returns the data object.
   */
  getData<T>(): T;

  /**
   * Returns the context object.
   */
  getContext<T>(): T;
}
```

### ExecutionContext 클래스

`ExecutionContext`는 현재 실행 과정에 대한 추가적인 정보를 제공해주는 `ArgumentsHost`를 상속 받습니다. `ArgumentsHost`처럼, Nest는 가드의 `canActivate()` 메서드나 인터셉터의 `intercept()` 메서드처럼 `ExecutionContext` 인스턴스가 필요한 곳에 제공해줍니다. 해당 클래스는 아래의 메서드를 제공하는데요.

```typescript
export interface ExecutionContext extends ArgumentsHost {
  /**
   * 현재 핸들러가 속한 컨트롤러 클래스의 타입을 반환합니다.
   */
  getClass<T>(): Type<T>;
  /**
   * 요청 파이프라인에서 다음에 호출될 핸들러 메서드의 레퍼런스를 반환합니다.
   */
  getHandler(): Function;
}
```

`getHandler()` 메서드는 호출될 핸들러에 대한 레퍼런스를 반환합니다. `getClass()` 메서드는 해당 핸들러가 속한 `Controller` 클래스의 타입을 반환합니다. 예를 들어 HTTP 컨텍스트에서는, 만약 현재 처리되는 요청이 `POST` 요청이며 이것이 `CatsController`의 `create()` 메서드(핸들러)에 대응된다고 했을 때, `getHandler()`는 `create()` 메서드의 레퍼런스를 반환하고, `getClass()`는 `CatsController`의 **타입**(인스턴스가 아님!)을 반환합니다.

```typescript
const methodKey = ctx.getHandler().name; // "create"
const className = ctx.getClass().name; // "CatsController"
```

현재 클래스와 핸들러 메서드의 레퍼런스에 접근할 수 있다는 것은 굉장한 유연성을 제공합니다. 가장 중요한 것은, 가드나 인터셉터에서 `@SetMetadata()` 데코레이터를 통해 설정된 메타데이터에 접근할 기회를 준다는 것입니다. 이 부분은 아래에서 더 다루겠습니다.

### Reflection과 메타데이터

Nest는 `@SetMetadata()` 데코레이터를 통해 라우트 핸들러에 **커스텀 메타데이터**를 달아주는 기능을 제공합니다. 이를 활용하여, 이 메타데이터에 접근하여 특정한 결정을 내릴 수 있습니다.

```typescript
// cats.controller.ts
@Post()
@SetMetadata('roles', ['admin'])
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

> **팁**
>
> `@SetMetadata()` 데코레이터는 `@nestjs/common` 패키지에서 가져올 수 있습니다.

위와 같이 하면, `create()` 메서드에 `roles`라는 메타데이터를 붙여줄 수 있습니다. 이때, `roles`는 메타데이터의 키이며, `['admin']`은 값입니다. 이렇게 해도 동작하기는 하지만, 핸들러에 직접적으로 `@SetMetadata()`를 사용하는 것은 좋은 방법은 아닙니다. 대신, 아래와 같이 데코레이터를 따로 만드는 것이 좋습니다.

```typescript
// roles.decorator.ts
import { SetMetadata } from "@nestjs/common";

export const Roles = (...roles: string[]) => SetMetadata("roles", roles);
```

이 방법은 더 깔끔하고, 읽기 쉬우며, 강하게 타이핑되어 있습니다. 이제 `@Roles()` 데코레이터를 `create()` 메서드에 달아봅시다.

```typescript
// cats.controller.ts
@Post()
@Roles('admin')
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

라우트의 메타데이터인 역할(`roles`)에 접근하기 위해, `Reflector` 헬퍼 클래스를 사용할 것입니다. `Reflector` 클래스는 프레임워크에서 기본적으로 제공되며, `@nestjs/core` 패키지에서 가져올 수 있습니다. `Reflector`는 일반적인 방법으로 클래스에 주입해줄 수 있습니다.

```typescript
// roles.guard.ts
@Injectable()
export class RolesGuard {
  constructor(private reflector: Reflector) {}
}
```

> **팁**
>
> `Reflector` 클래스는 `@nestjs/core` 패키지에서 가져올 수 있습니다.

이제, `get()` 메서드를 사용하여 핸들러의 메타데이터를 읽어봅시다.

```typescript
const roles = this.reflector.get<string[]>("roles", context.getHandler());
```

`Reflector#get` 메서드에 메타데이터의 **키**와 메타데이터를 가져올 **컨텍스트**(데코레이터가 달려있는 핸들러), 두 개의 인수를 넘겨주어 쉽게 메타데이터에 접근할 수 있습니다. 이 예시에서 **키**는 `'roles'`가 됩니다. 컨텍스트의 경우, 현재 처리되는 라우트 핸들러에서 메타데이터를 가져올 수 있도록 `context.getHandler()`를 호출하여 가져옵니다. `getHandler()`는 라우트 핸들러 함수의 **레퍼런스**를 반환한다는 걸 기억해주세요.

대신에, 컨트롤러 수준에서 메타데이터를 적용하여 컨트롤러 클래스 내의 모든 라우트에 메타데이터를 적용할 수 있도록 구성할 수도 있습니다.

```typescript
// cats.controller.ts
@Roles("admin")
@Controller("cats")
export class CatsController {}
```

이 경우에는, 컨트롤러 메타데이터를 가져오기 위해 `context.getHandler()` 대신에 `context.getClass()`를 두 번째 인수로 넘겨줍니다.

```typescript
// roles.gurad.ts
const roles = this.reflector.get<string[]>("roles", context.getClass());
```

여러 수준에 메타데이터를 설정한 경우, 메타데이터를 가져와서 합쳐야 할 수도 있는데요. `Reflector` 클래스는 이 경우에 사용되는 두 가지 유틸리티 메서드를 제공합니다. 이들은 컨트롤러와 메서드 메타데이터를 **둘 다** 한 번에 가져와서, 각각 다른 방법으로 합쳐줍니다.

아래와 같이 `'roles'` 메타데이터를 두 수준에서 제공하는 시나리오를 생각해봅시다.

```typescript
// cats.controller.ts
@Roles("user")
@Controller("cats")
export class CatsController {
  @Post()
  @Roles("admin")
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }
}
```

만약 `'user'`을 기본 역할로 명시하고, 특정 메서드에 대하여 선택적으로 덮어쓰고 싶다면 `getAllAndOverride()` 메서드를 사용하면 됩니다.

```typescript
const roles = this.reflector.getAllAndOverride<string[]>("roles", [
  context.getHandler(),
  context.getClass(),
]);
```

이와 같은 코드를 가진 가드에서는 위의 메타데이터와 `create()` 메서드 컨텍스트에서 동작한다고 할 때, `roles`는 `['admin']`을 갖게 됩니다.

둘 모두에서 메타데이터를 가져와서 합치고 싶다면, `getAllAndMerge()` 사용하면 됩니다. 해당 메서드는 배열이나 객체 둘 다 합쳐줍니다.

```typescript
const roles = this.reflector.getAllAndMerge<string[]>("roles", [
  context.getHandler(),
  context.getClass(),
]);
```

이렇게 하면, `roles`는 `['user', 'admin']`을 갖게 됩니다.

위의 두 메서드 모두 첫 번째 인수로 메타데이터 키를, 두 번째 인수로 `getHandler()`나 `getClass()` 메서드를 호출하여, 메타데이터를 가져올 컨텍스트들의 배열을 넘겨주면 됩니다.

### 문서 기여자

- [러리](https://github.com/Coalery)
