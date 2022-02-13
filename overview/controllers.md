---
description: "원문 : https://docs.nestjs.com/controllers"
---

# Controllers

컨트롤러는 들어오는 **요청**을 처리하고, **응답**을 클라이언트에게 반환하는 역할을 합니다.

![Controllers_1.png](https://docs.nestjs.com/assets/Controllers_1.png)

컨트롤러의 목적은 어플리케이션으로 들어오는 특정 요청들을 받는 것입니다. 어느 컨트롤러가 요청을 받아야하는지는 **라우팅** 매커니즘이 정하게 됩니다. 일반적으로 각각의 컨트롤러는 하나 이상의 라우트를 가지며, 각각 다른 동작을 할 수 있습니다.

기본적인 컨트롤러는 클래스와 **데코레이터**를 사용하여 만들 수 있습니다. 데코레이터는 클래스에 필요한 메타데이터를 넣어주고, Nest가 라우팅 맵을 만들 수 있게 합니다. 즉, 요청을 알맞은 컨트롤러에 연결할 수 있도록 합니다.

> **팁**
> 
> [validation](https://docs.nestjs.com/techniques/validation)이 들어있는 CRUD 컨트롤러를 빠르게 생성하고 싶다면, CLI의 [CRUD generator](https://docs.nestjs.com/recipes/crud-generator#crud-generator)를 사용해보세요: `nest g resource [name]`.

### 라우팅

아래 예제에서는 기본적인 컨트롤러를 정의할 때 필요한 `@Controller()` 데코레이터를 사용해 볼 것입니다. `@Controller()` 데코레이터에 경로를 지정하면 쉽게 관련된 라우트를 묶을 수 있고, 반복되는 코드를 최소화시킬 수 있습니다. 예를 들면, 고객 엔티티와 관련된 상호작용을 관리하는 라우트들을 `/customers` 라우트로 묶을 수도 있습니다. 이 경우, `@Controller()` 데코레이터에 `customers`라는 값을 넣어서 각각의 라우트 경로에 반복해서 넣을 필요 없이 경로를 지정할 수 있습니다.

```ts
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

> **팁**
> 
> CLI를 통해서 컨트롤러를 만들고 싶다면, `$ nest g controller cats` 명령어를 실행해보세요.

`findAll()` 메서드 위에 있는 HTTP 요청 메서드 데코레이터 `@Get()`를 통해 Nest는 HTTP 요청의 특정 엔드포엔트에 대한 핸들러를 만들 수 있습니다. 여기서 엔드포인트는 HTTP 요청 메서드(위의 경우 GET)와 라우트 경로를 말합니다. 그렇다면 라우트 경로는 어떻게 정해질까요? 라우트 경로는 컨트롤러에 정의되어 있는 경로와, 메서드의 데코레이터에 정의되어 있는 경로가 합쳐져서 정해집니다. `CatsController` 내의 모든 라우트에 `cats`라는 문자로 시작되는 경로를 사용하도록 정의하였고, 데코레이터에는 경로에 관한 아무 정보도 주지 않았습니다. 따라서 Nest는 해당 핸들러를 `GET /cats` 요청과 매핑시킵니다. 즉, 경로는 컨트롤러에 정의된 경로와 메서드 데코레이터에 정의된 경로를 포함합니다. 예를 들어, 컨트롤러에 `customers`라고 경로가 정의되어 있고 메서드에 `Get('profile')`이라 정의되어 있다면, 이는 `GET /customers/profile` 요청에 매핑됩니다.

위 예제에서는 해당하는 엔드포인트에 대해 GET 요청이 발생하는 경우, 이 요청을 Nest가 사용자 정의 메서드인 `findAll()`에 보내게 됩니다. 여기서 주의할 점은, 위 메서드의 이름은 임의로 정해졌다는 것입니다. 즉, 라우트에 연결할 메서드를 선언할 때, Nest는 메서드의 이름을 전혀 신경쓰지 않습니다.

이 메서드는 상태 코드 200과 관련된 응답(이 경우 문자열)을 반환하게 됩니다. 왜 이런 일이 일어날까요? 이를 설명하려면, 먼저 Nest가 응답을 처리하는 두가지의 **다른** 옵션을 알아야 합니다.

|옵션|설명|
|:---|:---|
|Standard<br />(recommended)|이 방법을 사용할 경우, 핸들러가 자바스크립트 객체나 배열을 반환하면 이를 자동으로 JSON으로 직렬화시킵니다. 그러나 자바스크립트 원시 타입을 반환할 경우, Nest는 해당 값을 직렬화시키지 않고 보냅니다. 즉, 값을 반환만 하면 나머지는 Nest가 알아서 하기 때문에 응답 처리가 더 간단해집니다.<br/>뿐만 아니라, 응답의 상태 코드는 201을 사용하는 POST 요청을 제외하면 기본적으로 200 입니다. 상태 코드를 바꾸고 싶다면, 핸들러에 `@HttpCode(...)` 데코레이터를 붙이면 됩니다. ([여기](https://docs.nestjs.com/controllers#status-code)를 참고하세요.)|
|Library-specific|메서드 핸들러 시그니쳐에 `@Res()` 데코레이터를 붙여서, Express 등 특정 라이브러리에 대한 응답 객체를 주입할 수 있습니다. (예: `findAll(@Res() response)`) 이렇게 하면, 해당 객체를 통해 노출된 네이티브 응답 핸들링 메서드를 사용할 수 있게 됩니다. Express의 예를 들면, 응답 코드와 응답을 할 때, `response.status(200).send()`와 같이 쓸 수 있습니다.|

> **주의**
> 
> 핸들러에서 `@Res()`나 `@Next()`를 사용하면, Nest는 당신이 library-specific 옵션을 선택했음을 감지합니다. 만약 위의 두 옵션을 모두 사용할 경우에는, 해당 라우트에 대해 Standard 옵션은 자동으로 꺼지며 예상대로 동작하지 않게 됩니다. 만약 쿠키나 헤더만 설정하고 다른 것은 프레임워크에게 맡기고 싶을 때와 같이, 두 옵션을 모두 써야할 때는 `@Res({ passthrough: true })`처럼 데코레이터의 `passthrough` 옵션을 `true`로 설정해야 합니다.

