# Dynamic modules

원문 : [https://docs.nestjs.com/fundamentals/dynamic-modules](https://docs.nestjs.com/fundamentals/dynamic-modules)

[모듈 챕터](https://docs.nestjs.com/modules)에서는 Nest 모듈의 기본에 대해서 다뤄보았고, [동적 모듈](https://docs.nestjs.com/modules#dynamic-modules)에 대해서도 간단하게 소개했었습니다. 이번 챕터에서는, 앞서 간단히 설명했든 동적 모듈에 대해 더 자세하게 설명합니다. 이 문서를 다 읽으시면, 동적 모듈이 무엇인지, 언제 어떻게 사용해야 하는지 이해할 수 있을 것입니다.

### 소개

**OVERVIEW** 섹션에 있는 대부분의 예시 코드들은 규칙적이거나 정적인 모듈을 다뤘습니다. 모듈은 전체 어플리케이션의 부분이 되는 [프로바이더](https://docs.nestjs.com/providers)와 [컨트롤러](https://docs.nestjs.com/controllers) 같은 컴포넌트들의 그룹을 정의합니다. 모듈들은 실행 컨텍스트나 스코프를 이러한 컴포넌트들에게 제공합니다. 예를 들어, 모듈에 정의된 프로바이더는 굳이 내보내지 않아도 같은 모듈에 있는 다른 요소에서 바로 사용할 수 있습니다. 프로바이더를 모듈 밖에서 사용하려면, 해당 프로바이더가 속하는 모듈에서 내보낸 뒤에, 사용하는 모듈에서 가져와야 했습니다.

더 친근한 예제로 한 번 봅시다!

먼저 `UsersModule`를 정의하고, `UsersService`를 모듈에 넣어준 뒤 내보냅니다. 이렇게 하면, `UsersModule`는 `UsersService`의 **호스트** 모듈이 됩니다.

```typescript
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';

@Module({
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

다음으로, `UsersModule`를 가져오는 `AuthModule`을 정의합니다. 이를 통해 `UsersModule`에서 내보내진 프로바이더들을 `AuthModule`에서 사용할 수 있게 됩니다.

```typescript
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [UsersModule],
  providers: [AuthService],
  exports: [AuthService],
})
export class AuthModule {}
```

이러한 구성을 통해, `AuthModule` 안에 속해있는 `AuthService`에 `UsersService`를 주입할 수 있게 됩니다.

```typescript
import { Injectable } from '@nestjs/common';
import { UsersService } from '../users/users.service';

@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}
  /*
    Implementation that makes use of this.usersService
  */
}
```

이를 **정적** 모듈 바인딩이라고 부릅니다. Nest가 모듈을 연결할 때 필요한 모든 정보들은 이미 호스트 모듈과 가져온 모듈에 정의되어 있습니다. 이 과정에서는 무슨 일이 발생하는지 알아봅시다. Nest는 아래의 과정을 통해 `UsersService`를 `AuthModule` 안에서 사용할 수 있게 만듭니다.

1. `UsersModule`을 인스턴스화하고, `UsersModule`이 사용하는 다른 모듈을 가져오고, 의존성을 처리합니다. ([참고](https://docs.nestjs.com/fundamentals/custom-providers))
2. `AuthModule`을 인스턴스화하고, `AuthModule`에 정의된 컴포넌트 내에서 `UsersModule`에서 내보내진 프로바이더들을 사용할 수 있게 합니다.
3. `UsersService`의 인스턴스를 `AuthService`에 주입합니다.

### 동적 모듈의 사용 예시

정적 모듈 바인딩을 사용하면, 모듈을 사용하는 곳에서 호스트 모듈의 프로바이더가 구성되는지에 **영향을** 줄 수 없습니다. 즉, 호스트 모듈에서 내보내는 프로바이더에 어떤 특별한 설정을 주고 싶어도 그게 불가능하다는 뜻입니다. 이것이 왜 문제가 될까요? 어떤 보편적인 목적의 모듈을 만들었는데, 이 모듈이 어디서 사용하는지에 따라 다르게 동작해야하는 경우를 생각해봅시다. 이는 사용하기 전에 설정이 필요한, 많은 시스템의 "플러그인"이라는 개념과 상당히 유사하다고도 볼 수 있습니다.

Nest 안에서 동적 모듈의 좋은 예시를 찾아보자면, **설정 모듈(configuration module)** 이 있습니다. 많은 어플리케이션에서 설정 모듈을 사용해서 액세스 키, 시크릿 키 등등을 외부로 빼내는 걸 볼 수 있습니다. 이를 통해, 서로 다른 배포 환경에서 어플리케이션 설정 정보를 동적으로 바꿀 수 있게 됩니다. 예를 들면, 개발 환경, 스테이지 환경 등이 있겠네요. 설정 모듈이 설정 값들을 관리하게 하면, 어플리케이션의 소스 코드는 설정 값에 대해 독립적으로 존재할 수 있습니다. 즉, 설정 값을 바꾼다고 해도 소스 코드를 바꿀 필요 없다는 거죠.

이걸 가능하게 하려면, 설정 모듈을 사용하는 모듈에서 자체적으로 설정 모듈 내의 내용을 변경할 수 있게 해야합니다. 바로 이때, *동적 모듈*을 사용하면 됩니다. 동적 모듈 기능을 사용해서, 설정 모듈을 동적으로 만들면 설정 모듈을 사용하는 모듈에서 가져올 때, API를 통해 설정 모듈이 어떻게 동작할 것인지를 제어할 수 있습니다.

다시 말해, 동적 모듈은 앞에서 봤던 정적 모듈 바인딩과는 반대로, 어떤 모듈을 다른 모듈로 가져오고, 모듈을 가져올 때 해당 모듈의 속성이나 동작 방식을 커스텀하게 제어할 수 있는 일종의 API를 제공합니다.

### 설정 모듈 예시

이 챕터에서는 [설정 챕터](https://docs.nestjs.com/techniques/configuration#service) 예제 코드의 기본적인 버전을 사용해보겠습니다. 이 챕터를 따라가면 만들어지는 코드 전체가 궁금하시다면 [여기](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules)를 참고해주세요.

아래의 코드의 목적은 `ConfigModule`이 `options` 객체를 받아, 해당 객체에 따라 커스터마이징 되는 것입니다. 기본 샘플 코드는 `.env` 파일의 위치인 프로젝트 루트 폴더의 경로를 하드 코딩한 것입니다. 이걸 이제 설정 가능하게 만들어서, `.env` 파일이 어느 폴더에 있던 관리할 수 있게 만들 겁니다. 예를 들면, 여러 `.env` 파일들이 있는 `config` 폴더를 프로젝트 루트 폴더 아래에 두는 경우가 있을 수 있겠네요. `ConfigModule`을 다른 프로젝트에서 사용할 때에도 다른 폴더를 선택할 수 있습니다. 값만 살짝 바꿔주면 되죠!

동적 모듈은 모듈을 임포트 할 때 모듈에 매개변수를 넘길 수 있게 만들어 주어서, 해당 모듈의 동작을 바꿀 수 있게 해줍니다. 이게 어떻게 작동하는지 봅시다. 동적 모듈을 사용하는 모듈의 관점에서 이것이 어떻게 보일지, 즉 최종 목표에서부터 시작해서 거꾸로 생각해보는 게 도움이 됩니다. 먼저, 정적으로 `ConfigModule`을 가져오는 예시부터 봅시다. 즉, 가져올 때 모듈에 영향을 줄 수 없는 경우죠. `@Module()` 데코레이터의 `imports` 배열에 집중해주세요!

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from './config/config.module';

@Module({
  imports: [ConfigModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

한 번 *동적 모듈*을 가져오는 부분은 어떻게 생겼는지 살펴보겠습니다. `imports` 배열의 차이를 위의 예시와 비교해보세요!

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from './config/config.module';

@Module({
  imports: [ConfigModule.register({ folder: './config' })],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

위의 동적 모듈 예시에서 무엇이 일어났는지 살펴봅시다. 무엇이 바뀌었나요?

1. `ConfigModule`은 평범한 클래스입니다. 따라서 `register()`라는 **정적 메서드**를 가질 것이라 추론할 수 있습니다. 클래스의 **인스턴스**가 아니라, `ConfigModule` 클래스를 통해 호출했으므로, 해당 메서드가 정적이라는 건 알 수 있습니다. (참고: 곧 만들어볼 `register()` 같은 메서드는 이름을 마음대로 설정할 수 있습니다. 하지만 보통 `forRoot()`나 `register()`를 사용하는 것이 일반적입니다.)
2. `register()` 메서드는 우리가 직접 정의하는 것이기 때문에, 원하는 인수들을 집어넣을 수 있도록 만들 수 있습니다. 위의 경우에서는, 적절한 속성을 갖는 간단한 `options` 객체를 받도록 만들 것입니다.
3. `register()` 메서드의 반환 값이 보통 모듈들이 들어가는 `imports` 배열에 있기 때문에, 해당 메서드는 어떤 `module`과 비슷한 무언가를 반환한다고 예상할 수 있습니다.

실제로, `register()` 메서드가 반환하는 것은 `DynamicModule`입니다. 동적 모듈은 런타임에 만들어지는 모듈 그 이상 그 이하도 아니며, 정적 모듈과 정확히 같은 속성과 추가적으로 `module`이라는 속성을 하나 더 가집니다. 이제 데코레이터에 넘겨진 모듈 옵션에 주의하며, 정적 모듈 정의를 다시 살펴보겠습니다.

```typescript
@Module({
  imports: [DogsModule],
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
```

동적 모듈은 정확히 같은 인터페이스에 추가적으로 `module`이라는 속성을 갖는 객체를 반환해야 합니다. `module` 속성의 값은 모듈의 이름으로 사용되며, 아래의 예제와 같이 모듈의 클래스 이름과 같아야 합니다.

> **팁**
> 
> 동적 모듈에서는, `module`을 **제외하고는** 모듈 옵션 객체의 모든 속성이 옵셔널 합니다.

`register()` 정적 메서드로 다시 돌아와봅시다. 이제 우리는 해당 메서드가 하는 일이 `DynamicModule` 인터페이스를 갖는 객체를 반환한다는 걸 알 수 있습니다. 정적 모듈들의 클래스 이름을 나열하는 것과 비슷하게, `register()` 메서드를 호출하면 `imports` 배열에 모듈을 제공하게 됩니다. 다시 말해, 동적 모듈 API는 단지 모듈을 반환하는 것이지만, `@Module()` 데코레이터에서 속성을 고정하는 것 대신에, 동적(programmatically)으로 명시할 수 있습니다.

완전히 이해하는 데에 도움이 될 만 한 두 사항이 있습니다.

1. 이제 `@Module()` 데코레이터의 `imports` 속성은 모듈의 클래스 이름을 받을 수 있을 뿐만 아니라, 동적 모듈을 **반환하는** 함수도 받을 수 있다는 걸 알 수 있습니다.
2. 동적 모듈은 자기 스스로 다른 모듈을 가져올 수 있습니다. 이걸 위의 예시에서 다루지는 않았지만, 만약 동적 모듈이 다른 모듈의 프로바이더를 의존한다면, `imports` 속성을 통해 해당 모듈들을 가져올 수 있습니다. 즉, 이 방식은 `@Module()` 데코레이터를 사용해서 정적 모듈의 메타데이터를 정의할 때와 완전히 비슷합니다.

위에서 했던 이해를 바탕으로, `ConfigModule`의 동적 정의가 어떻게 생겼는지 이해할 수 있게 되었습니다. 실제로 봅시다!

```typescript
import { DynamicModule, Module } from '@nestjs/common';
import { ConfigService } from './config.service';

@Module({})
export class ConfigModule {
  static register(): DynamicModule {
    return {
      module: ConfigModule,
      providers: [ConfigService],
      exports: [ConfigService],
    };
  }
}
```

이제 조각들이 어떻게 서로 연결되는지 분명하게 알 수 있습니다! `ConfigModule.register(...)`을 호출하면, 이때까지 `@Module()` 데코레이터를 통해 넘겨주었던 속성들과 동일한 속성들을 갖는 `DynamicModule` 객체를 반환합니다.

> **팁**
> 
> `DynamicModule`은 `@nestjs/common`에서 가져올 수 있습니다.

지금까지 만든 동적 모듈은 아직 그렇게 흥미롭진 않습니다. 하지만, 아직 원하는 대로 **설정**할 수 있는 기능을 사용하지 않았습니다! 이 기능을 이제부터 알아볼 겁니다.

### 문서 기여자

- [러리](https://github.com/Coalery)