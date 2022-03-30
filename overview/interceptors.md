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