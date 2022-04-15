---
description: "원문 : https://docs.nestjs.com/custom-decorators"
---

# Custom decorators

Nest는 **데코레이터**라는 언어의 기능 위에서 만들어졌습니다. 데코레이터는 많은 프로그래밍 언어들에도 있듯이 잘 알려진 개념입니다만, 자바스크립트 세상에서는 아직 상대적으로 새로운 기능에 속합니다. 데코레이터가 어떻게 동작하는지를 더 잘 이해하려면, [이 글](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841)을 읽어보는 걸 추천드립니다. 간단하게 정의해보자면 다음과 같습니다.

> ES2016 데코레이터는 타겟과 이름, 그리고 프로퍼티 설명자(property descriptor)를 인수로 갖는 함수를 반환하는 표현식입니다. 데코레이터 앞에 `@` 문자를 붙이고 데코레이팅 할 곳의 맨 위에 두면 적용됩니다. 데코레이터는 클래스, 메서드, 프로퍼티에 대해서 각각 만들 수 있습니다.

### 파라미터 데코레이터

Nest는 HTTP 라우트 핸들러에 유용하게 사용할 수 있는 **파라미터 데코레이터**를 제공합니다. 제공되는 데코레이터와 각각의 데코레이터가 나타내는 Express나 Fastify 객체들은 다음과 같습니다.

|데코레이터|객체|
|:---|:---|
|`@Request()`, `@Req()`|`req`|
|`@Response()`, `@Res()`|`res`|
|`@Next()`|`next`|
|`@Session()`|`req.session`|
|`@Param(param?: string)`|`req.params` / `req.params[param]`|
|`@Body(param?: string)`|`req.body` / `req.body[param]`|
|`@Query(param?: string)`|`req.query` / `req.query[param]`|
|`@Headers(param?: string)`|`req.headers` / `req.headers[param]`|
|`@Ip()`|`req.ip`|
|`@HostParam()`|`req.hosts`|

추가적으로, 자신만의 **커스텀 데코레이터**를 만들 수도 있습니다. 이게 왜 유용할까요?

node.js 세상에서는, 프로퍼티들을 **request** 객체에 붙이는 것이 일반적입니다. 이렇게 되면 각각의 라우트 핸들러에서 프로퍼티들을 필요할 때, 아래와 같이 수동으로 가져와야 합니다.

```typescript
const user = req.user;
```

이때 코드를 더 읽기 쉽고 투명하게 만들려면, `@User()` 데코레이터를 만들고 모든 컨트롤러에서 재사용하면 됩니다.

```typescript
// user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

이렇게 만들어두면, 필요한 곳에서 아래와 같이 깔끔하게 사용할 수 있습니다.

```typescript
@Get()
async findOne(@User() user: UserEntity) {
  console.log(user);
}
```