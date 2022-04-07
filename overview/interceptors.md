---
description: "원문 : https://docs.nestjs.com/interceptors"
---

# Interceptors

인터셉터는 `@Injectable()` 데코레이터가 붙어있는, `NestInterceptor` 인터페이스를 구현하는 클래스입니다.

![Interceptors_1.png](https://docs.nestjs.com/assets/Interceptors_1.png)

인터셉터는 [관점 지향 프로그래밍(Aspect Oriented Programming, AOP)](https://en.wikipedia.org/wiki/Aspect-oriented_programming)에서 영감을 받아 만들어진 기능들을 갖고 있습니다. 인터셉터는 아래의 일을 할 수 있습니다.

- 메서드 실행 전/후에 추가적인 로직을 붙일 수 있습니다.
- 함수가 반환한 결과를 변형시킬 수 있습니다.
- 함수에서 발생한 예외를 변형시킬 수 있습니다.
- 함수의 기본적인 동작을 확장할 수 있습니다.
- 캐싱 등의 목적으로, 특정 조건에 따라 함수를 완전히 오버라이딩 할 수 있습니다.

### 기본

각각의 인터셉터는 두 인수를 받는 `intercept()` 메서드를 구현해야 합니다. 이때 첫 번째 인수는 이미 [가드](https://docs.nestjs.com/guards) 챕터에서 봤던 `ExecutionContext` 인스턴스입니다. `ExecutionContext`는 `ArgumentsHost`를 상속 받았으며, `ArgumentsHost`는 예외 필터 챕터에서 봤었습니다. 그 챕터에서 `ArgumentsHost`는 원래의 핸들러에 들어오는 인수들을 감싼 래퍼(wrapper)이며, 어플리케이션의 형태에 따라 다른 인수 배열을 갖고 있음을 보았습니다. 이에 대해 더 알아보려면 [예외 필터 챕터](https://docs.nestjs.com/exception-filters#arguments-host)를 참고해주세요.

### ExecutionContext

`ArgumentsHost`를 확장함으로써 `ExecutionContext`도 현재의 실행 과정에 대한 추가적인 정보를 제공하는 몇 개의 헬퍼 메서드를 갖고 있습니다. 이 정보를 활용해서 여러 컨트롤러, 메서드, 실행 컨텍스트 등 광범위하게 작동할 수 있는 일반적인(generic) 인터셉터를 만들 수 있습니다. `ExecutionContext`에 대해서 더 알아보려면 [여기](https://docs.nestjs.com/fundamentals/execution-context)를 참고해주세요.

### CallHandler

두 번째 인수는 `CallHandler`입니다. `CallHandler` 인터페이스는 라우트 핸들러 메서드를 인터셉터 내에서 호출할 수 있게 해주는 `handle()` 메서드를 구현합니다(implement). `intercept()` 메서드 안에서 `handle()` 메서드를 호출하지 않으면 라우트 핸들러 메서드는 절대 호출되지 않습니다.

이는 `intercept()` 메서드가 사실상 요청/응답 스트림을 **감싼다**는 뜻이 됩니다. 이를 통해, 최종 라우트 핸들러의 실행 **전/후 모두**에 로직을 구현할 수 있게 됩니다. 자, 그러면 라우트 핸들러의 실행 전에 실행할 코드는 `intercept()` 메서드에서 `handle()`을 호출하기 **전에** 작성하면 된다는 것은 자명합니다. 하지만 핸들러 실행 후에 일어나는 것에는 어떻게 영향을 줄 수 있을까요? 이는 `handle()` 메서드가 `Observable`를 반환하기 때문에, 강력한 [RxJS](https://github.com/ReactiveX/rxjs)의 연산자들을 사용하여 응답을 조작할 수 있습니다. `handle()`을 호출하는 등, 라우트 핸들러를 호출하는 것을 관점 지향 프로그래밍에서는 [Pointcut](https://en.wikipedia.org/wiki/Pointcut)이라고 부르며, 이는 추가적인 로직이 들어갈 부분임을 나타냅니다.

예를 들어, `POST /cats` 요청이 들어온다고 생각해봅시다. 이 요청은 `CatsController` 안에 정의되어 있는 `create()` 핸들러가 처리할 것입니다. 그런데 만약 중간에 `handle()` 메서드를 호출하지 않는 인터셉터가 들어가있는 경우, `create()` 메서드는 호출되지 않을 것입니다. 반대로, `handle()`이 호출되고 그 `Observable`이 반환되면, `create()` 핸들러가 호출됩니다. 그리고 `Observable`을 통해 응답 스트림을 받으면, 스트림에서 추가적인 연산이 실행되고 최종 결과가 요청자에게 반환됩니다.

### Aspect interception

처음으로 살펴볼 인터셉터 사용 예시는 유저의 상호작용에 대한 로깅입니다. 예를 들면 유저 호출 저장, 비동기적으로 이벤트 디스패치, 걸린 시간 계산 등이 있겠네요. 이를 구현한 간단한 `LoggingInterceptor` 아래의 코드에서 볼 수 있습니다.

```typescript
// logging.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');

    const now = Date.now();
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
```

> **팁**
> 
> `NestInterceptor<T, R>`의 `T`는 `Observable<T>`의 타입을 나타내며 응답 스트림의 타입이고, `R`은 `Observable<R>`로 감싸진 값의 타입입니다.

> **알림**
> 
> 인터셉터는 컨트롤러, 프로바이더, 가드 등과 마찬가지로 `constructor`를 통해 의존성을 주입 받을 수 있습니다.

`handle()`이 RxJS의 `Observable`를 반환하기 때문에, 스트림을 조작할 때 사용할 수 있는 연산자의 선택폭이 넓어집니다. 위의 예시에서는, 스트림의 정상적/비정상적 종료에 대해 익명 로깅 함수를 실행해주지만, 응답 사이클에는 간섭하지 않는 `tap()` 연산자를 사용하였습니다.

### 인터셉터 적용하기

인터셉터를 적용하려면, `@nestjs/common` 패키지의 `@UseInterceptors()` 데코레이터를 가져와 사용하면 됩니다. 인터셉터도 [파이프](https://docs.nestjs.com/pipes)와 [가드](https://docs.nestjs.com/guards)처럼 컨트롤러 수준, 메서드 수준, 전역 수준에 적용할 수 있습니다.

```typescript
// cats.controller.ts
@UseInterceptors(LoggingInterceptor)
export class CatsController {}
```

> **팁**
> 
> `@UseInterceptors()` 데코레이터는 `@nestjs/common` 패키지에서 가져올 수 있습니다.

위와 같이 하면, `CatsController`에 정의된 각각의 라우트 핸들러에 `LoggingInterceptor` 적용시킬 수 있습니다. 만약 누군가가 `GET /cats` 엔드포인트를 호출하면 표준 출력에 아래와 같이 출력되는 것을 볼 수 있습니다.

```
Before...
After... 1ms
```

인스턴스 대신에 `LoggingInterceptor` 타입을 넘겨서 프레임워크에게 인스턴스화의 역할을 맡기고, 의존성 주입을 가능하게 만든 걸 주목해주세요. 물론 파이프, 가드, 예외 필터와 마찬가지로, 저 자리에 인스턴스를 넣을 수도 있습니다.

```typescript
// cats.controller.ts
@UseInterceptors(new LoggingInterceptor())
export class CatsController {}
```

언급했듯이, 위의 구문은 해당 컨트롤러에 선언된 모든 핸들러에 인터셉터를 적용시키게 됩니다. 만약 인터셉터를 단일 메서드에만 제한하고 싶다면, 그저 데코레이터를 **메서드 수준**에 적용하면 됩니다.

전역 인터셉터를 적용하려면, Nest 어플리케이션 인스턴스의 `useGlobalInterceptors()` 메서드를 사용하면 됩니다.

```typescript
const app = await NestFactory.create(AppModule);
app.useGlobalInterceptors(new LoggingInterceptor());
```

전역 인터셉터는는 어플리케이션 전체, 즉 모든 컨트롤러와 모든 라우트 핸들러에 적용됩니다. 하지만, 위의 `useGlobalInterceptors()`를 사용한 예시처럼 모듈 밖에서 등록된 전역 인터셉터는 말 그대로 모듈 밖의 컨텍스트에서 완료되었기 때문에, 의존성을 주입할 수 없게 됩니다. 이 문제를 해결하려면, 아래와 같이 **모듈에 직접** 전역 수준 필터를 등록하면 됩니다.

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

> **팁**
> 
> 인터셉터가 의존성 주입이 되도록 위와 같이 만들면, 어떤 모듈에서 설정했던 인터셉터는 전역이 됩니다. 따라서, 전역 인터셉터를 따로 선언하는 모듈을 따로 두시는 게 좋습니다. 또한, `useClass`는 사용자 지정 프로바이더를 등록하는 유일한 방법이 아닙니다. 자세한 건 [여기](https://docs.nestjs.com/fundamentals/custom-providers)를 참고하세요.