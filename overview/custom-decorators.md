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

### 데이터 넘기기

데코레이터의 동작이 어떤 조건에 따라 바뀌어야 할 때는 `data` 매개변수를 사용하여 데코레이터의 팩토리 함수에 인수를 넘겨줄 수 있습니다. 이 기능을 사용하는 예시를 하나 들어보자면, 키를 사용해서 요청 객체의 속성을 가져오는 커스텀 데코레이터가 있습니다. 예를 들어, [인증 레이어](http://docs.nestjs.com/techniques/authentication#implementing-passport-strategies)가 요청을 검증하고, 유저 엔티티를 요청 객체에 붙여준다고 가정해봅시다. 인증된 요청에 의한 유저 엔티티는 아래 객체처럼 생겼을 겁니다.

```json
{
  "id": 101,
  "firstName": "Alan",
  "lastName": "Turing",
  "email": "alan@email.com",
  "roles": ["admin"]
}
```

한 번 속성 이름을 받아서 관련된 값이 존재하면 반환해주는 데코레이터를 정의해봅시다. 만약 해당 속성 이름을 갖는 값이 없거나 `user` 객체가 없다면 `undefined`을 반환합니다.

```typescript
// user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  },
);
```

이렇게 만들면, 아래와 같이 컨트롤러에서 `@User()` 데코레이터를 통해 특정 속성에 접근할 수 있습니다.

```typescript
@Get()
async findOne(@User('firstName') firstName: string) {
  console.log(`Hello ${firstName}`);
}
```

다른 키로 다른 속성에 접근하고 싶을 때도, 같은 데코레이터를 사용할 수 있습니다. 만약 `user` 객체가 깊거나 복잡하다면, 이 방법을 통해 더 쉽고 읽기 쉬운 요청 핸들러를 구현할 수 있습니다.

> **팁**
> 
> 타입스크립트 유저에게는 `createParamDecorator<T>()`가 제너릭이라는 것을 강조하고 싶습니다. 이는 `createParamDecorator<string>((data, ctx) => ...)`처럼 타입 세이프티(Type safety)를 명시적으로 적용할 수 있다는 것을 뜻합니다. 이 방법 외에도, `createParamDecorator((data: string, ctx) => ...)`처럼 팩토리 함수의 매개변수 타입을 지정하여 적용할 수도 있습니다. 만약 두 방법 모두 사용하지 않는다면 `data`의 타입은 `any`가 됩니다.