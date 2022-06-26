---
description: "원문 : https://docs.nestjs.com/fundamentals/custom-providers"
---

# Custom providers

앞에서 **의존성 주입(DI)** 의 다양한 면을 살펴보았고, 이를 Nest에서 어떻게 사용하는지를 보았습니다. 그 예시 중 하나가 바로 클래스에 프로바이더를 주입할 때 사용하는 [생성자 기반](https://docs.nestjs.com/providers#dependency-injection) 의존성 주입입니다. 아마 의존성 주입이 Nest 코어에 기본적으로 내장되어 있다는 걸 들어도 딱히 놀라시진 않겠죠. 하지만, 아직 주요한 패턴 하나만을 만나봤을 뿐입니다! 어플리케이션이 점점 복잡해지면 의존성 주입 시스템의 모든 기능들이 필요해질 겁니다. 그러니, 더 자세히 알아봅시다!

### 의존성 주입 기초

의존성 주입은 의존성들의 인스턴스를 직접 생성하지 않고, IoC 컨테이너가 생성하도록 역할을 넘기는 [제어 반전(Inversion of Control, IoC)](https://en.wikipedia.org/wiki/Inversion_of_control) 기술입니다. Nest에서 IoC 컨테이너는 NestJS 런타임 시스템이 되겠죠? [프로바이더 챕터](https://docs.nestjs.com/providers)에서 나왔던 예시에서 무슨 일이 일어나는지 한 번 살펴보도록 하겠습니다.

먼저, 프로바이더를 정의합니다. `@Injectable()` 데코레이터를 사용해서 `CatsService`를 프로바이더라고 표시합니다.

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

어떻게 이렇게만 해도 의존성 주입이 동작할까요? 이는 아래의 세 가지 주요 단계를 통해 이루어집니다.

1. `cats.service.ts`에서, `@Injectable()` 데코레이터는 `CatsService` 클래스가 Nest IoC 컨테이너에 의해 관리될 수 있는 클래스라고 선언합니다.
2. `cats.controller.ts`에서, `CatsController`는 생성자 주입으로 `CatsService` 토큰에 대한 의존성을 정의합니다.
    ```typescript
    constructor(private catsService: CatsService)
    ```
3. `app.module.ts`에서, `cats.service.ts` 파일의 `CatsService` 클래스를 `CatsService` 토큰과 연결합니다. [아래에서](https://docs.nestjs.com/fundamentals/custom-providers#standard-providers) 정확히 어떻게 연결(등록)이 발생하는지 살펴볼 것입니다.

Nest IoC 컨트롤러가 `CatsController`를 인스턴스화할 때는 먼저 종속성을 찾습니다*. `CatsService` 의존성을 찾았으면, 위의 3번 단계인 등록 단계에서 설정된 `CatsService` 토큰을 찾습니다. 기본적으로 설정되는 `SINGLETON` 스코프 때, Nest는 `CatsService`의 인스턴스를 만들고 캐시한 뒤 반환하거나, 이미 캐시되어 존재한다면 해당 인스턴스를 반환하게 됩니다.

\* 이 설명은 요점에 더 집중하기 위해 더 간단하게 요약한 것입니다. 이 때문에 생략된 중요한 부분 중 하나는, 의존성 처리를 위해 코드를 분석하는 과정이 매우 정교하며 이는 어플리케이션 부트스트래핑 중에 발생한다는 것입니다. 의존성 분석을 사용하면, 위 예시에서 `CatsService`가 스스로 의존성을 갖고 있다고 해도 이를 제대로 처리할 수 있습니다. 의존성 그래프는 기본적으로 "바텀 업"의 순서로 알맞게 처리된다는 것을 보장하며, 이는 개발자는 복잡한 의존성 그래프를 직접 관리할 필요가 없게 만듭니다.

### 표준 프로바이더

이제 `@Module()` 데코레이터를 더 자세히 살펴봅시다. 아래와 같이 `app.module.ts`에서 모듈을 선언했습니다.

```typescript
@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
```

`providers` 프로퍼티는 프로바이더들의 배열을 받습니다. 지금까지 클래스 이름의 리스트를 통해 프로바이더들을 `providers` 프로퍼티로 넘겨주었습니다. 하지만, `providers: [CatsService]` 문법은 아래의 더 정확한 문법의 줄임말 같은 것입니다.

```typescript
providers: [
  {
    provide: CatsService,
    useClass: CatsService,
  },
];
```

명확한 코드를 보았으니 이제 등록 과정을 이해하실 수 있을 겁니다!
이 부분에서 `CatsService` 토큰과 `CatsService`를 분명하게 연결시키고 있습니다. 이 때는 보통 토큰과 인스턴스의 클래스가 같은 이름을 쓰기 때문에, 줄여서 간단하게 사용할 수 있도록 만들어 놓은 것입니다.

### 커스텀 프로바이더

만약 위의 표준 프로바이더가 제공하는 것 그 이상의 기능이 필요하다면 어떻게 해야할까요?
아래와 같은 예시가 있을 수 있겠죠!

- Nest가 인스턴스화/캐싱하지 않고, 직접 커스텀 인스턴스를 만들고 싶을 때
- 이미 존재하는 클래스를 두 번째 의존성에서 재사용하고 싶을 때
- 테스트를 위해 클래스를 모킹하여 오버라이딩 하고 싶을 때

Nest에서는 커스텀 프로바이더를 정의하여 위의 경우들을 처리할 수 있습니다. 커스텀 프로바이더를 정의하는 방법에는 여러가지가 있는데, 하나씩 살펴봅시다.

> **팁**
> 
> 만약 의존성 해결에서 문제가 발생했다면, `NEST_DEBUG` 환경 변수를 설정하여 어플리케이션 시작 시 추가적인 의존성 해결 관련 로그를 가져올 수 있습니다.

### 값 프로바이더: `useValue`

`useValue`는 상수값을 주입하거나, 외부 라이브러리를 Nest 컨테이너 안에 넣거나, 실제 구현 대신 가짜(mock) 객체로 대체할 때 사용합니다. 테스트를 위해 Nest가 실제 `CatsService` 대신에 가짜 객체를 사용하도록 해봅시다.

```typescript
import { CatsService } from './cats.service';

const mockCatsService = {
  /* mock implementation
  ...
  */
};

@Module({
  imports: [CatsModule],
  providers: [
    {
      provide: CatsService,
      useValue: mockCatsService,
    },
  ],
})
export class AppModule {}
```

위의 예시에서 `CatsService` 토큰은 `mockCatsService` 가짜 객체와 연결됩니다. `useValue`는 어떠한 값을 필요로 하며, 위의 경우에서 값은 `CatsService` 클래스와 같은 인터페이스를 갖는 리터럴 객체입니다. 타입스크립트의 [구조적 타이핑](https://www.typescriptlang.org/docs/handbook/type-compatibility.html) 때문에, 리터럴 객체거나 `new`로 만들어진 클래스 인스턴스거나 인터페이스와 호환만 된다면 어떤 객체든 사용할 수 있습니다.

### 비클래스 기반 프로바이더 토큰

앞에서, 우리는 `providers` 배열 내에 속해있는 프로바이더의 토큰(`provide`)으로 클래스 이름을 사용했습니다. 이렇게 클래스 이름을 토큰으로 사용하는 것은 **생성자 기반 주입**에 사용되는 표준 패턴입니다. 이 개념이 명확하게 기억나지 않으신다면 [의존성 주입 기초](https://docs.nestjs.com/fundamentals/custom-providers#di-fundamentals)를 다시 한 번 보고 와주세요!
하지만 가끔, 문자열이나 심볼(Symbol)을 토큰으로 사용하고 싶을 때가 있습니다. 다음과 같이 말이죠.

```typescript
import { connection } from './connection';

@Module({
  providers: [
    {
      provide: 'CONNECTION',
      useValue: connection,
    },
  ],
})
export class AppModule {}
```

이 예시에서는, 문자열 토큰(`'CONNECTION'`)과 외부 파일에서 가져와서 이미 존재하는 `connection` 객체를 연결하였습니다.

> **알림**
> 
> 문자열을 토큰으로 사용할 수도 있고, 자바스크립트의 [심볼](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)이나 타입스크립트의 [열거형](https://www.typescriptlang.org/docs/handbook/enums.html)을 사용할 수도 있습니다.

이전에, 어떻게 표준 [생성자 기반 주입](https://docs.nestjs.com/providers#dependency-injection) 패턴을 사용하여 프로바이더를 주입할 수 있는지를 살펴보았습니다. 해당 패턴은 의존성이 클래스 이름으로 선언되어야만 했습니다. 하지만 `'CONNECTION'` 커스텀 프로바이더는 문자열 값 토큰을 사용하고 있네요. 이런 프로바이더는 어떻게 주입을 해야할까요? 바로, `@Inject()` 데코레이터를 사용하면 됩니다. 이 데코레이터는 토큰을 인수로 받습니다.

```typescript
@Injectable()
export class CatsRepository {
  constructor(@Inject('CONNECTION') connection: Connection) {}
}
```

> **팁**
> 
> `@Inject()` 데코레이터는 `@nestjs/common` 패키지에서 가져올 수 있습니다.

위의 예시에서는 더 나은 이해를 위해 `'CONNECTION'` 문자열을 바로 썼습니다만, 클린 코드를 위해서는 `constats.ts` 같은 분리된 파일에 토큰을 정의하는 것이 좋습니다. 즉, 자체 파일에 정의해서 필요할 때마다 임포트 해오는 심볼이나 열거형처럼 사용하면 됩니다.

### 클래스 프로바이더: `useClass`

`useClass`는 토큰과 연결될 클래스를 동적으로 결정할 수 있게 해줍니다. 예를 들면 `ConfigService` 클래스가 있고, 현재 상태에 따라 Nest가 다른 설정 서비스 구현체를 제공하길 바란다고 생각해봅시다. 이를 구현한 코드가 아래와 같습니다.

```typescript
const configServiceProvider = {
  provide: ConfigService,
  useClass:
    process.env.NODE_ENV === 'development'
      ? DevelopmentConfigService
      : ProductionConfigService,
};

@Module({
  providers: [configServiceProvider],
})
export class AppModule {}
```

위 코드에서 두 가지 부분을 한 번 살펴봅시다. 먼저 `configServiceProvider` 리터럴 객체를 정의했고, 이를 모듈 데코레이터의 `providers` 프로퍼티에 넣어주었습니다. 여기까지는 기능적으로 이 챕터에서 지금까지 사용한 예시와 동일합니다.

또한, 토큰으로 `ConfigService` 클래스 이름을 사용하였습니다. `ConfigService`에 의존하는 모든 클래스에 대하여, Nest는 제공된 클래스(`DevelopmentConfigService`나 `ProductionConfigService`)의 인스턴스를 주입하며, 다른 곳에 선언된 기본 구현을 오버라이딩 합니다. 예를 들면, `@Injectable()` 데코레이터와 함께 선언된 `ConfigService`가 있겠죠!

즉, `ConfigService`에 의존하는 모든 클래스에게, 다른 곳에 `ConfigService`가 선언되어있던 안 되어있던 관계 없이 현재 환경(`NODE_ENV`)에 따라 `DevelopmentConfigService`나 `ProductionConfigService` 중 하나의 인스턴스를 주입해줍니다.

### 팩토리 프로바이더: `useFactory`

`useFactory`는 프로바이더를 **동적으로** 만들 수 있게 해줍니다. 팩토리 함수에서 반환된 값을 프로바이더로 제공합니다. 팩토리 함수는 필요에 따라 간단할 수도 있고, 복잡할 수도 있습니다. 간단한 팩토리 함수는 다른 프로바이더에 의존하지 않지만, 더 복잡한 팩토리 함수는 결과를 계산할 때 다른 프로바이더가 필요할 수도 있습니다.  이 경우, 팩토리 함수는 필요한 다른 프로바이더를 자기 스스로 주입하며, 아래와 같은 특성을 갖게 됩니다.

1. 팩토리 함수는 인수를 받을 수 있습니다.
2. `inject` 프로퍼티는, Nest가 의존성을 해결하고 인스턴스화 과정 중에 팩토리 함수에게 인수로 넘겨줄 프로바이더의 배열을 받습니다. 이 프로바이더들도 옵셔널이 될 수 있습니다. Nest는 `inject` 배열 내에 있는 프로바이더의 인스턴스를 팩토리 함수의 인수로 넘겨줍니다. 이때, 배열 내의 프로바이더 순서와 인수 순서는 동일합니다. 이를 보여주는 코드가 아래와 같습니다.

```typescript
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider, optionalProvider?: string) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider, { token: 'SomeOptionalProvider', optional: true }],
  //       \_____________/            \__________________/
  //        This provider              The provider with this
  //        is mandatory.              token can resolves to `undefined`.
};

@Module({
  providers: [
    connectionFactory,
    OptionsProvider,
    // { provide: 'SomeOptionalProvider', useValue: 'anything' },
  ],
})
export class AppModule {}
```

### 별칭 프로바이더: `useExisting`

`useExisting`은 이미 존재하는 프로바이더에 대한 별칭을 생성할 수 있게 해줍니다. 아래의 예시에서, `'AliasedLoggerService'` 문자열 기반 토큰은 `LoggerService` 클래스 기반 토큰의 별칭입니다. 이때, `AliasedLoggerService`와 `LoggerService` 두 의존성을 아래처럼 같이 넣어준다고 가정해봅시다. 만약 두 의존성이 `SINGLETON` 스코프로 명시되어 있다면, 둘은 같은 인스턴스로 처리됩니다.

```typescript
@Injectable()
class LoggerService {
  /* implementation details */
}

const loggerAliasProvider = {
  provide: 'AliasedLoggerService',
  useExisting: LoggerService,
};

@Module({
  providers: [LoggerService, loggerAliasProvider],
})
export class AppModule {}
```

### 비서비스 기반 프로바이더

프로바이더는 보통 서비스를 제공하지만, 꼭 서비스만 제공할 수 있는 것은 아닙니다. 프로바이더는 **어떤** 값이던 제공할 수 있습니다. 예를 들면, 아래와 같이 현재 환경에 따라 다른 설정 객체들의 배열을 제공할 수도 있습니다.

```typescript
const configFactory = {
  provide: 'CONFIG',
  useFactory: () => {
    return process.env.NODE_ENV === 'development' ? devConfig : prodConfig;
  },
};

@Module({
  providers: [configFactory],
})
export class AppModule {}
```

### 커스텀 프로바이더 내보내기

다른 프로바이더들과 마찬가지로 커스텀 프로바이더는 선언한 모듈 내로 범위가 제한됩니다. 이를 다른 모듈에서도 사용할 수 있게 하려면 모듈에서 내보내야 하며, 아래와 같이 해당 프로바이더의 토큰을 사용하거나 프로바이더 객체를 사용하면 됩니다.

아래는 토큰을 사용하여 내보낸 것입니다.

```typescript
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: ['CONNECTION'],
})
export class AppModule {}
```

또는, 프로바이더 객체를 사용한 것입니다.

```typescript
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: [connectionFactory],
})
export class AppModule {}
```

### 문서 기여자

- [러리](https://github.com/Coalery)
