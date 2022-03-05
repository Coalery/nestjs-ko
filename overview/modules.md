---
description: "원문 : https://docs.nestjs.com/modules"
---

# Modules

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