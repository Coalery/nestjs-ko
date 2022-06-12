---
description: "원문 : https://docs.nestjs.com/fundamentals/custom-providers"
---

# Custom providers

앞에서, **의존성 주입(DI)**의 다양한 면을 살펴보았고, 이를 Nest에서 어떻게 사용하는지를 보았습니다. 그 예시 중 하나가 바로 클래스에 서비스 프로바이더 등 인스턴스들을 주입하기 위해 사용하는 [생성자 기반](https://docs.nestjs.com/providers#dependency-injection) 의존성 주입입니다. 아마 의존성 주입이 Nest 코어에 기본적으로 만들어져 있다는 걸 들어도 딱히 놀라시진 않겠죠. 하지만, 아직 주요 패턴 하나만 만나봤을 뿐입니다! 어플리케이션이 점점 복잡해지면, 의존성 주입 시스템의 모든 기능들이 필요해질 겁니다. 그러니, 더 자세히 알아봅시다!

### 의존성 주입 기초

의존성 주입은 의존성들의 인스턴스를 직접 생성하지 않고, 이를 IoC 컨테이너에게 넘기는 [제어 반전(Inversion of Control, IoC)](https://en.wikipedia.org/wiki/Inversion_of_control) 기술입니다. Nest에서 IoC 컨테이너는 NestJS 런타임 시스템이 되겠죠? [프로바이더 챕터](https://docs.nestjs.com/providers)의 예시에서 무슨 일이 일어나는지 한 번 살펴보도록 하겠습니다.

먼저, 프로바이더를 정의합니다. `@Injectable()` 데코레이터는 `CatsService`를 프로바이더라고 표시합니다.

```typescript
// cats.service.ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  findAll(): Cat[] {
    return this.cats;
  }
}
```

그런 다음, Nest에게 컨트롤러 클래스에 프로바이더 주입을 요청합니다.

```typescript
// cats.controller.ts
import { Controller, Get } from '@nestjs/common';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

마지막으로, 프로바이더를 Nest IoC 컨테이너에 등록합니다.

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

어떻게 의존성 주입이 작동할 수 있게 하는걸까요? 이는 아래 세 가지 주요 단계를 통해 이루어집니다.

1. `cats.service.ts`에서, `@Injectable()` 데코레이터는 `CatsService` 클래스가 Nest IoC 컨테이너에 의해 관리될 수 있는 클래스라고 선언합니다.
2. `cats.controller.ts`에서, `CatsController`는 생성자 주입으로 `CatsService` 토큰에 대한 의존성을 정의합니다.
  ```typescript
  constructor(private catsService: CatsService)
  ```
3. `app.module.ts`에서, `cats.service.ts` 파일의 `CatsService` 클래스를 `CatsService` 토큰과 연결합니다. [아래에서](https://docs.nestjs.com/fundamentals/custom-providers#standard-providers) 정확히 어떻게 연결(등록)이 발생하는지 살펴볼 것입니다.

Nest IoC 컨트롤러가 `CatsController`를 인스턴스화할 때, 먼저 종속성을 찾습니다*. `CatsService` 의존성을 찾았으면, 위의 3번 단계인 등록 단계에 따라 `CatsService` 클래스를 반환하는 `CatsService` 토큰을 찾습니다. 기본 동작인 `SINGLETON` 스코프 때, Nest는 `CatsService`의 인스턴스를 만들고 캐시한 뒤 반환하거나, 이미 캐시되어 존재한다면 해당 인스턴스를 반환하게 됩니다.

\* 이 설명은 요점에 더 집중하기 위해 더 간단하게 요약한 것입니다. 이 때문에 얼버무려졌던 중요한 부분 중 하나는, 의존성 처리를 위해 코드를 분석하는 과정이 매우 정교하며, 이는 어플리케이션 부트스트래핑 중에 발생한다는 것입니다. 주요 기능 중 하나로, 의존성 분석은 . 위 예시에서 만약 `CatsService`가 스스로 의존성을 갖고 있다고 해도, 이러한 의존성도 해결됩니다. 의존성 그래프는 기본적으로 "바텀 업"의 순서로 알맞게 해결된다는 것을 보장합니다. 이 메커니즘은 개발자가 복잡한 의존성 그래프를 관리해야 하는 고통을 없애줍니다.

### 표준 프로바이더

이제 `@Module()` 데코레이터를 더 자세히 살펴봅시다. `app.module.ts`에서 아래와 같이 모듈을 선언했습니다.

```typescript
@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
```

`providers` 프로퍼티는 `providers`의 배열을 받습니다. 지금까지 프로바이더들을 클래스 이름의 리스트를 통해 넘겨주었습니다. 사실, `providers: [CatsService]` 문법은 아래의 더 정확한 문법의 줄임말 같은 것입니다.

```typescript
providers: [
  {
    provide: CatsService,
    useClass: CatsService,
  },
];
```

명확한 코드를 보았으니, 이제 등록 과정을 이해하실 수 있을 겁니다. 
이 부분에서 `CatsService` 토큰과 `CatsService`를 분명하게 연결시키고 있습니다. 이 때는 보통 토큰과 인스턴스의 클래스가 같은 이름을 쓰기 때문에, 줄여서 간단하게 사용할 수 있도록 만들어 놓은 것입니다.

### 커스텀 프로바이더

만약 위의 표준 프로바이더가 제공하는 것 그 이상의 기능이 필요하다면 어떻게 해야할까요?
아래와 같은 예시가 있을 수 있죠.

- Nest가 클래스를 인스턴스화/캐싱을 하지 않고, 직접 커스텀 인스턴스를 만들고 싶을 때
- 이미 존재하는 클래스를 두 번째 의존성에서 재사용하고 싶을 때
- 테스트를 위해 클래스를 모킹하여 오버라이딩 하고 싶을 때

Nest에서는 위의 경우들을 처리하기 위해 커스텀 프로바이더를 정의할 수 있습니다. 커스텀 프로바이더를 정의할 수 있는 여러 방법이 있는데, 한 번 살펴봅시다.

> **팁**
> 
> 만약 의존성 해결에서 문제가 발생했다면, `NEST_DEBUG` 환경 변수를 설정하여 어플리케이션 시작 시 추가적인 의존성 해결 관련 로그를 가져올 수 있습니다.

