---
description: "원문 : https://docs.nestjs.com/providers"
---

# Providers

프로바이더는 Nest의 기본 개념입니다. 서비스, 리포지토리, 팩토리, 헬퍼 등, 많은 Nest의 기본 클래스들은 프로바이더입니다. 프로바이더의 주요 아이디어는 의존성 **주입**이 가능하다는 것입니다. 이는 객체들이 서로 다양한 관계를 형성할 수 있도록 하고, Nest에게 인스턴스 생성을 대부분 위임할 수 있게 합니다.

![Components_1.png](https://docs.nestjs.com/assets/Components_1.png)

이전 챕터에서는 간단한 `CatsController`를 만들어 보았습니다. 컨트롤러는 HTTP 요청을 처리해야 하고, 더 복잡한 작업은 **프로바이더**에게 넘겨주어야 합니다. 프로바이더는 [모듈](https://docs.nestjs.com/modules)의 `providers`로 정의된 일반 자바스크립트 클래스입니다.

> **팁**
> 
> Nest는 의존성을 객체 지향적으로 설계하고 구성할 수 있게 하기 때문에, [SOLID](https://docs.nestjs.com/modules) 원칙을 따르는 걸 추천드립니다.

### 서비스

간단한 `CatsService`를 만들어보며 시작해봅시다. 이 서비스는 데이터를 저장하고 조회하는 역할을 맡았고, `CatsController`에서 사용되도록 설계되었습니다. 따라서 이는 프로바이더로 정의하는 것이 좋습니다.

```ts
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

```ts
// interfaces/cat.interface.ts
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

이제 고양이를 조회할 서비스 클래스를 만들었으니, 이를 `CatsController`에서 써봅시다.

```ts
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