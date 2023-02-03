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
