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