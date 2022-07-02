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

### 문서 기여자

- [러리](https://github.com/Coalery)