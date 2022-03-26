---
description: "원문 : https://docs.nestjs.com/guards"
---

# Guards

가드는 `@Injectable()` 데코레이터가 붙어있는, `CanActivate` 인터페이스를 구현하는 클래스입니다.

![Guards_1.png](https://docs.nestjs.com/assets/Guards_1.png)

가드는 **단일 책임**을 갖습니다. 가드는 런타임의 권한, 역할, ACL 등의 조건에 따라, 주어진 요청이 라우트 핸들러에 의해 처리될 것인지를 결정합니다. 이것은 <strong>인가(authorization)</strong>라고 불리기도 합니다. 일반적으로 기존 Express 어플리케이션에서 인가와, 인가의 사촌 <strong>인증(authentication)</strong>은 [미들웨어](https://docs.nestjs.com/middleware)로 처리가 됩니다. 물론 토큰 검증과 `request` 객체에 속성을 붙이는 것이 특정 라우트 컨텍스트와 메타데이터에 한정되지 않기 때문에, 미들웨어로 인증을 처리하는 것도 좋은 선택이긴 합니다.

하지만, 미들웨어는 본질적으로 바보입니다. 그 이유는, `next()` 함수를 호출한 뒤에 어떤 핸들러가 실행될지 모르기 때문입니다. 반면, **가드**는 `ExecutionContext` 인스턴스에 접근할 수 있으므로 다음에 무엇이 실행될지를 정확히 알 수 있습니다. 앞서 나왔던 예외 필터, 파이프, 그리고 추후 나올 인터셉터와 마찬가지로, 가드 또한 요청/응답 사이클의 정확한 위치에 선언적으로 처리 로직을 끼워넣을 수 있게 설계되었습니다. 이를 통해 코드의 반복을 줄이고 선언적으로 유지할 수 있게 됩니다.

> **팁**
> 
> 가드는 미들웨어 **이후**, 그리고 인터셉터와 파이프 **이전**에 실행됩니다.

### 인가 가드

언급했듯이, 어떤 특정 라우트는 요청자가 충분한 권한을 갖고 있을 때에만 실행되어야 할 때가 있습니다. 그러므로, **인가**는 가드의 좋은 예시라고 볼 수 있습니다. 지금부터 만들 `AuthGuard`는 요청 헤더에 토큰이 붙어있는, 인증된 유저를 가정합니다. 토큰을 가져와서 검증하고, 추출한 정보를 사용하여 해당 요청이 실행될 수 있는지 아닌지를 결정합니다.

```typescript
// auth.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

> **팁**
> 
> 만약 인증(Authentication) 메커니즘을 실제로 어떻게 구현하는지가 궁금하다면, [이 챕터](https://docs.nestjs.com/security/authentication)를 참고하세요. 또한, 더 복잡한 인가(Authorization) 예제는 [이 페이지](https://docs.nestjs.com/security/authorization)를 참고해주세요.

`validateRequest()` 함수 내의 로직은 필요에 따라 간단해질 수도 있고 복잡해질 수도 있습니다. 이 예제의 요점은, 가드가 어떻게 요청/응답 사이클에 들어갈 수 있느냐입니다.

모든 가드는 `canActivate()` 함수를 구현해야 합니다. 이 함수는 현재 요청을 허가할지 안 할지를 나타내는 boolean을 반환하며, 이는 동기적으로도 가능하고, `Promise`와 `Observable`를 통해 비동기적으로도 가능합니다. Nest는 반환된 값을 통해 가드 다음에 할 작업을 결정합니다.

- `true`를 반환하면, 요청이 처리됩니다.
- `false`를 반환하면, Nest가 해당 요청을 거절합니다.

### 실행 컨텍스트

`canActivate()` 함수는 단 하나의 인수로, `ArgumentsHost`를 상속 받은 `ExecutionContext` 인스턴스를 받습니다. `ArgumentsHost`는 저번에 예외 필터 챕터에서 본 적이 있을겁니다. 위 예제에서는, `Request`객체에 대한 참조를 가져오기 위해 `ArgumentsHost`의 헬퍼 메서드를 사용하고 있으며, 이는 이전에도 동일한 이유로 사용해봤습니다. 기억이 안나신다면, [예외 필터](https://docs.nestjs.com/exception-filters#arguments-host)챕터의 **Arguments host** 부분을 참고해주세요.

`ArgumentsHost`를 확장함으로써 `ExecutionContext`도 현재의 실행 과정에 대한 추가적인 정보를 제공하는 몇 개의 헬퍼 메서드를 갖고 있습니다. 이 정보를 활용해서 여러 컨트롤러, 메서드, 실행 컨텍스트 등 광범위하게 작동할 수 있는 일반적인(generic) 가드를 만들 수 있습니다. `ExecutionContext`에 대한 자세한 내용은 [여기](https://docs.nestjs.com/fundamentals/execution-context)를 참고해주세요.

### 역할 기반(Role-based) 인증

이제 좀 더 실용적으로, 특정 역할을 갖고 있는 유저에게만 접근을 허가하는 가드를 만들어봅시다. 일단 기본적인 가드 템플릿으로 시작하여, 다음 섹션부터 구현을 해보겠습니다. 지금의 가드는 모든 요청을 허락합니다.

```typescript
// roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    return true;
  }
}
```

### 가드 적용하기

파이프나 예외 필터와 마찬가지로, 가드도 **컨트롤러 수준**, 메서드 수준, 전역 수준에 적용할 수 있습니다. 아래에서는, `@UseGuards()` 데코레이터를 사용해서 컨트롤러 수준에 가드를 적용했습니다. 이 데코레이터는 단일 인수나 반점(,)으로 구분된 인수들을 받을 수 있습니다. 이를 통해 단 한 번의 선언으로 적절하게 가드들을 적용할 수 있습니다.

```typescript
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```

> **팁**
> 
> `@UseGuards()` 데코레이터는 `@nestjs/common` 패키지에서 찾을 수 있습니다.

위에서, 인스턴스 대신에 `RolesGuard` 타입을 넘겨서 프레임워크에게 인스턴스화의 역할을 맡기고, 의존성 주입을 가능하게 만들었습니다. 물론 파이프나 예외 필터와 마찬가지로, 저 위치에 인스턴스도 넘길 수 있습니다.

```typescript
@Controller('cats')
@UseGuards(new RolesGuard())
export class CatsController {}
```

위와 같이 하면, 해당 컨트롤러에 선언된 모든 핸들러에 가드를 적용하게 됩니다. 만약 딱 하나의 메서드에만 가드를 적용하고 싶다면, `@UseGuards` 데코레이터를 **메서드 수준**에 적용하면 됩니다.

전역 가드로 설정하려면, Nest 어플리케이션 인스턴스의 `useGlobalGuards()` 메서드를 사용하면 됩니다.

> **알림**
> 
> [하이브리드 어플리케이션](https://docs.nestjs.com/faq/hybrid-application)의 경우, 기본적으로 `useGlobalGuards()`로는 게이트웨이와 마이크로서비스에 가드를 등록할 수 없습니다. (이 동작을 변경하는 방법은 [하이브리드 어플리케이션](https://docs.nestjs.com/faq/hybrid-application) 챕터를 참고해주세요.)
> 
> 대신, 하이브리드가 아닌 "표준" 마이크로서비스 어플리케이션에는 `useGlobalGuards()`로 전역 파이프를 등록할 수 있습니다.

전역 가드는 전체 어플리케이션, 즉 모든 컨트롤러와 모든 라우트 핸들러에 적용됩니다. 하지만, 위의 `useGlobalGuards()`를 사용한 예시처럼 모듈 밖에서 등록된 전역 가드는 말 그대로 모듈 밖의 컨텍스트에서 완료되었기 때문에, 의존성을 주입할 수 없게 됩니다. 이 문제를 해결하려면, 아래와 같이 **모듈에 직접** 전역 수준 가드를 등록하면 됩니다.

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}
```

> **팁**
> 
> 가드가 의존성 주입이 되도록 위와 같이 만들면, 어떤 모듈에서 설정했던 가드는 전역이 됩니다. 따라서, 전역 가드를 따로 선언하는 모듈을 따로 두시는 게 좋습니다. 또한, `useClass`는 사용자 지정 프로바이더를 등록하는 유일한 방법이 아닙니다. 자세한 건 [여기](https://docs.nestjs.com/fundamentals/custom-providers)를 참고하세요.

### 핸들러마다 역할 설정하기

`RolesGuard`가 작동하기는 하는데, 아직 그리 유용하진 않습니다. 우리는 아직 가드의 가장 중요한 기능인 [실행 컨텍스트](https://docs.nestjs.com/fundamentals/execution-context)를 활용하지 않았습니다. 예를 들어, `CatsController`의 여러 라우트들에 각각 다른 권한을 설정하고 싶을 수도 있습니다. 즉, 어떤 라우트는 어드민 유저만 사용할 수 있고, 다른 어떤 라우트는 모두에게 열려있게 만드는 것입니다. 어떻게 라우트에 유연하고 재사용 가능하게 역할을 설정할 수 있을까요?

여기서 [사용자 지정 메타데이터](https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata)를 사용합니다. Nest는 사용자 지정 **메타데이터**를 라우트 핸들러에 붙일 수 있도록 `@SetMetadata()`라는 데코레이터를 제공합니다. 이 메타데이터가 가드가 결정을 내릴 때 필요한 `role` 데이터를 제공합니다. `@SetMetadata()`를 써봅시다.

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

위처럼 하면, `create()` 메서드에 `roles` 메타데이터를 붙일 수 있습니다. 이때, `roles`는 키(key)고, `['admin']`이 값(value)입니다. 저렇게 만들어도 잘 동작하긴 하지만, `@SetMetadata()`를 라우트에 직접 사용하는 것은 좋은 방법이 아닙니다. 대신, 아래와 같이 자신만의 데코레이터를 만드세요.

```typescript
// roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

이렇게 하면 더 깔끔하고 읽기 쉬우며, 타입을 강력하게(strongly typed) 설정할 수 있습니다. 이제 `create()` 메서드에 `@Roles()` 데코레이터를 붙여봅시다.

```typescript
// cats.controller.ts
@Post()
@Roles('admin')
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

### 다 합쳐봅시다.

이제 `RolesGuard`로 돌아가서 모두 묶어봅시다. 지금은 모든 경우에 대해 `true`를 반환하여, 모든 요청이 처리되도록 하고 있습니다. 하지만, 현재 유저의 역할과 현재 처리될 라우트의 역할을 비교하여 조건적으로 값을 반환하고 싶습니다. 라우트의 역할, 즉 메타데이터에 접근하려면, `Reflector`라는 헬퍼 클래스를 사용해야 합니다. 이 클래스는 프레임워크에서 기본적으로 제공하며, `@nestjs/core` 패키지에서 가져올 수 있습니다.

```typescript
// roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return matchRoles(roles, user.roles);
  }
}
```

> **팁**
> 
> node.js에서는 확인된(authorized) 유저 정보를 `request` 객체에 붙이는 것이 일반적입니다. 그래서 위의 코드에서, `request.user`가 유저 인스턴스와 해당 유저가 갖고 있는 역할을 포함한다고 가정하였습니다. 그러므로 직접 앱을 만드실 때는 `request.user`에 유저 정보를 넣어주는 사용자 지정 **인가 가드**나 미들웨어를 위의 `RolesGuard`와 엮으셔야 합니다. 자세한 내용은 [이 챕터](https://docs.nestjs.com/security/authentication)를 참고해주세요.

> **주의**
> 
> `matchRoles()` 함수 내 로직은 필요에 따라 간단할 수도 있고, 복잡할 수도 있습니다. 이 예제의 요점은 가드가 어떻게 요청/응답 사이클에 들어갈 수 있느냐입니다.

컨텍스트에 맞는 방법으로 `Reflector`를 사용하는 방법에 대해서는 **실행 컨텍스트** 챕터의 [리플렉션과 메타데이터](https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata) 섹션을 참고해주세요.

유저가 불충분한 권한으로 엔드포인트에 요청을 보내면, Nest가 자동으로 아래의 응답을 반환합니다.

```json
{
  "statusCode": 403,
  "message": "Forbidden resource",
  "error": "Forbidden"
}
```

이는 가드가 `false`를 반환하면, 프레임워크가 `ForbiddenException`을 발생시키기 때문입니다. 만약 다른 응답을 반환하고 싶다면, 원하는 예외를 발생시키면 됩니다. 예를 들면 다음과 같습니다.

```typescript
throw new UnauthorizedException();
```

가드에서 발생한 모든 예외는 [예외 레이어](https://docs.nestjs.com/exception-filters)(전역 예외 필터와 해당 컨텍스트에 적용된 예외 필터들)에서 처리됩니다.

> **팁**
> 
> 더 현실적으로 인가를 구현하는 방법을 알아보려면 [이 챕터](https://docs.nestjs.com/security/authorization)를 참고하세요.
