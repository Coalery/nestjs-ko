# Modules

원문 : [https://docs.nestjs.com/modules](https://docs.nestjs.com/modules)

모듈은 `@Module()` 데코레이터가 붙은 클래스입니다. `@Module()` 데코레이터는 **Nest**가 어플리케이션의 구조를 구성할 때 사용하는 메타데이터를 붙여주게 됩니다.

![Modules_1.png](https://docs.nestjs.com/assets/Modules_1.png)

각각의 어플리케이션은 최소 하나의 **루트 모듈**을 갖습니다. 루트 모듈은 Nest가 모듈, 프로바이더의 관계, 의존성을 만들 때 사용하는 내부 자료구조인 **어플리케이션 그래프**를 만들 때 시작점으로 사용합니다. 이론적으로 아주 작은 어플리케이션은 루트 모듈만 갖지만, 이는 일반적인 경우가 아닙니다. 모듈은 구성 요소들을 체계화하는 효율적인 방법이므로, 대부분의 어플리케이션 아키텍쳐에서는 서로 밀접한 관계에 있는 기능들을 하나의 모듈로 캡슐화하게 됩니다. 따라서, 여러 모듈을 사용하는 것이 일반적입니다.

`@Module()` 데코레이터는 모듈을 정의하는 속성들을 갖는 하나의 객체를 사용합니다.

|속성|설명|
|:---:|:---|
|`providers`|Nest Injector에 의해 인스턴스가 만들어지고, 해당 모듈에 공유될 프로바이더들|
|`controllers`|인스턴스가 만들어져야 하는, 해당 모듈에 정의된 컨트롤러들|
|`imports`|임포트 된 모듈들의 리스트. 이들은 해당 모듈에 필요한 프로바이더를 내보내는 모듈들이어야 한다.|
|`exports`|`providers`의 부분집합이며, 다른 모듈에서 이 모듈을 임포트 했을 때 사용할 수 있도록 할 프로바이더들. 프로바이더 자체나, 프로바이더의 토큰을 넣을 수 있음.|

모듈은 기본적으로 프로바이더들을 **캡슐화**합니다. 이는, 현재 모듈에 직접적으로 속해 있지 않거나, 임포트 한 모듈에서 내보내지 않는 프로바이더는 주입하는 것이 불가능하다는 것을 뜻합니다. 따라서, 모듈에서 내보내진 프로바이더는 모듈의 공개 인터페이스나 API라고 볼 수 있습니다.

### 기능 모듈

`CatsController`와 `CatsService`는 같은 어플리케이션 도메인에 속해 있습니다. 두 클래스는 서로 밀접한 관계에 있기 때문에, 기능 모듈로 묶는 것이 좋습니다. 기능 모듈은 적절하게 코드를 조직하여 코드를 체계적으로 유지하고, 기능 간에 확실한 경계를 만들 수 있습니다. 이는 특히 어플리케이션이나 팀의 규모가 성장할 때, 소프트웨어의 복잡도를 관리하고 [SOLID](https://en.wikipedia.org/wiki/SOLID) 원리를 따라서 개발할 수 있도록 도와줍니다.

`CatsModule`을 만들어봅시다.

```typescript
// cats/cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

> **팁**
> 
> CLI를 통해 모듈을 만들려면, `$ nest g module cats` 명령어를 실행시켜보세요.

`cats.module.ts` 파일 안에 `CatsModule`을 정의했고, 이 모듈에 관련된 모든 것을 `cats` 디렉토리 내로 옮겼습니다. 마지막으로, 이 모듈을 루트 모듈(`app.module.ts` 파일에 정의되어 있는 `AppModule`)에 임포트 해야 합니다.

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

현재 디렉토리 구조는 이렇습니다.

```
src
├─ cats
│   ├─ dto
│   │   └─ create-cat.dto.ts
│   ├─ interfaces
│   │   └─ cat.interface.ts
│   ├─ cats.controller.ts
│   ├─ cats.module.ts
│   └─ cats.service.ts
├─ app.module.ts
└─ main.ts
```

### 공유 모듈

Nest에서 모듈들은 기본적으로 **싱글톤**입니다. 따라서, 큰 노력 없이 여러 모듈에서 어떠한 프로바이더든 같은 인스턴스를 공유할 수 있습니다.

![Shared_Module_1.png](https://docs.nestjs.com/assets/Shared_Module_1.png)

모든 모듈은 공유 모듈입니다. 한 번 생성되면, 어떠한 모듈에서든 이를 재사용할 수 있게 됩니다. 자, 이제 `CatsService`의 인스턴스를 여러 다른 모듈에서 공유해야하는 상황을 가정해봅시다. 이를 위해선 먼저, `CatsService` 프로바이더를 모듈의 `exports` 배열에 추가하여 해당 프로바이더를 **내보내야** 합니다. 아래와 같이 말이죠.

```typescript
// cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

이제 `CatsModule`을 임포트한 모듈은 모두 `CatsService`에 접근할 수 있게 되며, 임포트한 모든 다른 모듈들은 같은 인스턴스를 공유하게 됩니다.

### 모듈 다시 내보내기

위에서 봤듯이, 모듈은 자신이 갖고 있는 프로바이더를 내보낼 수 있습니다. 게다가, 자신이 임포트한 모듈을 다시 내보낼 수도 있습니다. 아래 예시에서, `CommonModule`은 `CoreModule`에서 임포트도 되고, 내보내지기도 합니다. 이를 통해 다른 모듈에서 `CoreModule`을 임포트하면 둘 다 사용할 수 있게 됩니다.

```typescript
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}
```

### 의존성 주입

모듈 클래스는 프로바이더를 **주입**할 수도 있습니다.

```typescript
// cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```

그러나, 모듈 클래스 자체는 [순환 의존성(Circular Dependency)](https://docs.nestjs.com/fundamentals/circular-dependency) 때문에 프로바이더로 주입될 수 없습니다.

### 전역 모듈

만약 어떤 모듈을 모든 곳에서 임포트 해야 한다면, 이는 상당히 귀찮은 작업일 것입니다. Nest와는 다르게 [Angular](https://angular.io/)의 `providers`는 전역 스코프에 등록됩니다. 즉, Angular에서는 한 번 정의되면 어디서든 쓸 수 있다는 것입니다. 하지만 Nest에서는 프로바이더를 모듈 스코프에서 캡슐화하기 때문에, 먼저 캡슐화한 모듈을 임포트하지 않는다면 모듈의 프로바이더를 쓸 수 없습니다.

어떤 프로바이더(헬퍼, DB 연결 등)를 모든 곳에서 쓸 수 있게 하고 싶을 땐, `@Global()` 데코레이터를 붙여서 **전역 모듈**로 만들면 됩니다.

```typescript
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

`@Global()` 데코레이터가 모듈을 전역 스코프로 만들어줍니다. 전역 모듈은 **단 한 번** 등록되어야 하며, 일반적으로 루트 모듈이나 핵심 모듈에 등록됩니다. 위 예시에서 `CatService` 프로바이더는 어디서든 쓸 수 있게 되고, 해당 서비스를 주입 받고 싶어하는 모듈에서는 굳이 `CatsModule`을 `imports` 배열에 넣을 필요가 없어지게 됩니다.

> **팁**
> 
> 모든 것을 전역적으로 만드는 것은 좋은 디자인이 아닙니다. 전역 모듈은 어플리케이션의 기반을 만들 때 여러 곳에서 반복되는 코드를 줄이기 위해 존재합니다. 따라서, 모듈의 `imports` 배열을 사용하는 것이 일반적으로 모듈의 API를 다른 모듈에서 사용할 수 있게 하는 더 좋은 방법입니다.

### 동적 모듈

Nest의 모듈 시스템은 **동적 모듈**이라는 강력한 기능이 있습니다. 이 기능을 통해 프로바이더를 동적으로 등록하고 설정할 수 있는, 커스텀 가능한 모듈을 쉽게 만들 수 있습니다. 동적 모듈에 관해서는 [여기](https://docs.nestjs.com/fundamentals/dynamic-modules)서 더 자세히 다룹니다. 이 챕터에서는 모듈의 소개를 끝내기 위해, 간단하게 요약해서 설명합니다.

아래의 예시는 `DatabaseModule`의 동적 모듈 정의입니다.

```typescript
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

> **팁**
> 
> `forRoot()` 메서드는 동기적으로, 혹은 `Promise` 등을 통해 비동기적으로 동적 모듈을 반환합니다.

이 모듈은 `@Module()` 데코레이터를 통해 `Connection` 프로바이더를 기본적으로 정의합니다. 그러나 추가적으로, `forRoot()` 메서드의 매개변수로 들어온 `entities`와 `options` 객체에 따라 저장소(Repository) 등의 어떤 프로바이더들을 노출하게 됩니다. 동적 모듈을 통해 반환된 속성들은 `@Module()` 데코레이터에 정의된 기본 모듈 메타데이터를 **확장**합니다. (오버라이딩이 아닙니다!) 이렇게 하면, 정적으로 정의된 `Connection` 프로바이더**와** 동적으로 생성된 저장소 프로바이더 둘 다 모듈에서 내보내지게 됩니다.

만약 동적 모듈을 전역 스코프에 등록하고 싶다면, `global` 속성을 `true`로 주면 됩니다.

```typescript
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

> **주의**
> 
> 위에서 언급했듯이, 모든 것을 전역적으로 만드는 것은 좋은 디자인이 아닙니다.

`DatabaseModule`은 아래와 같이 임포트, 설정할 수 있습니다.

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

만약 동적 모듈을 다시 내보내고 싶다면, `exports` 배열에서 `forRoot()`  호출을 제외할 수 있습니다.

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
  exports: [DatabaseModule],
})
export class AppModule {}
```

[동적 모듈](https://docs.nestjs.com/fundamentals/dynamic-modules) 챕터에서는 이 주제를 더 자세히 다루고, [실제로 작동하는 예시](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules)를 포함합니다.

### 문서 기여자

- [러리](https://github.com/Coalery)
- [cpprhtn](https://github.com/cpprhtn)
