# Providers

원문 : [https://docs.nestjs.com/providers](https://docs.nestjs.com/providers)

프로바이더는 Nest의 기본 개념입니다. 서비스, 리포지토리, 팩토리, 헬퍼 등, 많은 Nest의 기본 클래스들은 프로바이더입니다. 프로바이더의 주요 아이디어는 의존성 **주입**이 가능하다는 것입니다. 이는 객체들이 서로 다양한 관계를 형성할 수 있도록 하고, Nest에게 인스턴스 생성을 대부분 위임할 수 있게 합니다.

![Components_1.png](https://docs.nestjs.com/assets/Components_1.png)

이전 챕터에서는 간단한 `CatsController`를 만들어 보았습니다. 컨트롤러는 HTTP 요청을 처리해야 하고, 더 복잡한 작업은 **프로바이더**에게 넘겨주어야 합니다. 프로바이더는 [모듈](https://docs.nestjs.com/modules)의 `providers`로 정의된 일반 자바스크립트 클래스입니다.

> **팁**
> 
> Nest는 의존성을 객체 지향적으로 설계하고 구성할 수 있게 하기 때문에, [SOLID](https://docs.nestjs.com/modules) 원칙을 따르는 걸 추천드립니다.

### 서비스

간단한 `CatsService`를 만들어보며 시작해봅시다. 이 서비스는 데이터를 저장하고 조회하는 역할을 맡았고, `CatsController`에서 사용되도록 설계되었습니다. 따라서 이는 프로바이더로 정의하는 것이 좋습니다.

```typescript
// cats.service.ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

> **팁**
> 
> CLI를 사용하여 서비스를 만들려면, `$ nest g service cats` 명령어를 실행시키면 됩니다.

`CatsService`는 하나의 프로퍼티와 두 개의 메서드를 가진 기본적인 클래스입니다. 새로운 점은 `@Injectable()` 데코레이터를 사용했다는 점인데요. `@Injectable()` 데코레이터는 `CatsService`가 Nest IoC 컨테이너에 의해 관리될 수 있는 클래스라는 것을 알려주는 메타데이터를 붙여줍니다. 한편, 위 예제는 `Cat` 인터페이스도 사용하는데요. 아래와 같이 이렇게 생겼습니다.

```typescript
// interfaces/cat.interface.ts
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

이제 고양이를 조회할 서비스 클래스를 만들었으니, 이를 `CatsController`에서 써봅시다.

```typescript
// cats.controller.ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

`CatsService`는 클래스의 생성자를 통해 주입됩니다. 위 코드에서는 `private` 문법을 사용하고 있는데, 이를 통해 한 번에 `catsService` 멤버를 선언하고 초기화시킬 수 있습니다.

### 의존성 주입

Nest는 **의존성 주입**이라는 강력한 디자인 패턴을 중심으로 만들어졌습니다. 이 개념에 대해 더 알아보시려면, 공식 [Angular](https://angular.io/guide/dependency-injection) 문서의 글을 읽어보시는 것을 추천드립니다.

타입스크립트의 기능 덕분에, 의존성은 타입만으로 해결될 수 있습니다. 이 덕분에 Nest에서는 의존성을 쉽게 관리할 수 있습니다. 예를 들면, 아래의 코드에서 Nest는 `CatsService`의 인스턴스를 만들고 반환하여 `catsService`에 주입하게 됩니다. 싱글톤의 경우 다른 곳에서 요청되어 이미 존재한다면 해당 인스턴스를 반환하게 됩니다. 이 의존성은 컨트롤러의 생성자에 전달되거나, 지정된 속성에 할당됩니다.

```typescript
constructor(private catsService: CatsService) {}
```

### 스코프

프로바이더는 일반적으로 어플리케이션의 수명 주기와 연관된 수명(스코프)이 있습니다. 어플리케이션이 시작될 때에 모든 의존성들이 만들어져야 하므로, 모든 프로바이더를 인스턴스화 하게 됩니다. 유사하게, 어플리케이션이 꺼질 때에는 모든 프로바이더가 없어지게 됩니다. 하지만, 프로바이더의 수명 기준을 어플리케이션이 아니라, **요청마다** 만들어졌다 없어지게 만들 수도 있습니다(request-scoped). 이 기술에 대해서 더 알아보려면 [여기](https://docs.nestjs.com/fundamentals/injection-scopes)를 참고해주세요.

### 커스텀 프로바이더

Nest는 프로바이더들 사이의 관계를 만드는 제어 반전(Inversion of Control, IoC) 컨테이너를 갖고 있으며, 이는 위에서 설명한 의존성 주입의 기반이 됩니다. 하지만, 이 기능은 앞서 설명했던 것보다 훨씬 강력합니다. 이 덕분에 프로바이더를 선언할 때 일반 값, 클래스, 혹은 동기, 비동기 팩토리 등 여러 가지의 방법을 사용할 수 있습니다. 예시를 보려면 [여기](https://docs.nestjs.com/fundamentals/dependency-injection)를 참고하세요.

### Optional 프로바이더

가끔, 굳이 만들어질 필요가 없는 의존성이 필요할 때가 있습니다. 예를 들면, 클래스가 어떤 **설정(config) 객체**에 의존하고, 이 객체가 없을 때 기본 값을 사용하는 상황이 생길 수 있습니다. 이 경우, 설정 프로바이더가 없어서 에러가 발생하진 않으므로 의존성은 선택 사항이 됩니다.

프로바이더를 선택 사항으로 만들려면, 생성자 시그니쳐에 `@Optional()` 데코레이터를 붙이면 됩니다.

```typescript
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

위 예제에서는 `HTTP_OPTIONS`라는 사용자 지정 **토큰**을 사용하기 때문에, 커스텀 프로바이더를 사용한 것입니다. 이는 클래스의 생성자에서 의존성을 표시하는 '생성자 기반 주입'의 예시를 보여주고 있습니다. 커스텀 프로바이더와 관련 토큰에 대해 더 알아보시려면 [여기](https://docs.nestjs.com/fundamentals/custom-providers)를 참고하세요.

### 속성 기반 주입

앞에서 살펴본 기술은 생성자 메서드를 통해서 프로바이더가 주입되는 '생성자 기반 주입'입니다. 한편, 어떤 특수한 경우에는 **속성 기반 주입**이 유용합니다. 예를 들어 최상위 클래스가 하나 이상의 프로바이더에 의존하고 있다면, 이들을 서브 클래스들의 생성자에서 `super()`를 계속 호출하여 최상위 클래스로 보내는 것은 꽤 귀찮은 일입니다. 이를 피하기 위해서, 속성 단에서 `@Inject()` 데코레이터를 사용할 수도 있습니다.

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

> **주의**
> 
> 만약 클래스가 다른 프로바이더를 상속 받지 않는 경우에는, 항상 **생성자 기반 주입**을 사용하는 것이 더 낫습니다.

### 프로바이더 등록

이제 프로바이더(`CatsService`)를 선언했고, 이 서비스를 사용하는 컨트롤러(`CatsController`)도 만들었습니다. 이제, 주입이 일어날 수 있도록 Nest에 서비스를 등록해야 합니다. 모듈 파일(`app.module.ts`)에 있는 `@Module()` 데코레이터의 `providers` 배열에 서비스를 추가해봅시다.

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

Nest는 이제 `CatsController` 클래스의 의존성을 해결할 수 있게 되었습니다!

현재 디렉토리 구조는 이렇습니다.

```
src
├─ cats
│   ├─ dto
│   │   └─ create-cat.dto.ts
│   ├─ interfaces
│   │   └─ cat.interface.ts
│   ├─ cats.controller.ts
│   └─ cats.service.ts
├─ app.module.ts
└─ main.ts
```

### 수동 인스턴스 생성

이때까지, Nest가 어떻게 의존성 처리를 자동으로 처리하는지 알아보았습니다. 어떤 특정 상황에서는 내장된 의존성 주입 시스템을 사용하지 않고, 직접 프로바이더를 만들거나 가져와야 할 수도 있습니다. 이에 대해서, 잠시 두 가지 주제를 아래에서 알아보겠습니다.

이미 존재하는 인스턴스나, 프로바이더의 인스턴스를 동적으로 만들어야 한다면, [여기](https://docs.nestjs.com/fundamentals/module-ref)를 참고해주세요.

컨트롤러 없이 독립적으로 동작하는 어플리케이션이나, 켜는 동안 설정(config) 서비스를 사용해야 하는 경우 등, `bootstrap()` 함수 내에서 프로바이더를 가져와야 한다면 [여기](https://docs.nestjs.com/standalone-applications)를 참고해주세요.

### 문서 기여자

- [러리](https://github.com/Coalery)
- [cpprhtn](https://github.com/cpprhtn)
