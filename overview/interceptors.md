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