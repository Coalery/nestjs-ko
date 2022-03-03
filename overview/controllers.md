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

```typescript
// cats.controller.ts
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

### Request 객체

핸들러는 가끔 클라이언트의 **요청**에 대한 세부 사항에 접근해야할 때가 있습니다. Nest에서는 기반 플랫폼(기본적으로는 Express)의 [request 객체](https://expressjs.com/en/api.html#req)에 접근할 수 있습니다. 주입 받을 핸들러의 시그니처에 `@Req()` 데코레이터를 넣으면, Nest가 request 객체를 주입해주게 됩니다.

```typescript
// cats.controller.ts
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}
```

> **팁**
> 
> 위의 `request: Request` 파라미터처럼 `express`의 타입 정보를 가져오려면, `@types/express` 패키지를 설치해주세요.

Request 객체는 HTTP 요청을 나타내고, 쿼리 문자열이나 파라미터, HTTP 헤더, body 등을 갖습니다. (더 알아보고 싶다면 [여기](https://expressjs.com/en/api.html#req)를 참고하세요.) 대부분의 경우, 해당 속성들을 직접 가져올 필요는 없습니다. 대신, `@Body()`나 `@Query()` 등 각각 전용 데코레이터가 있고, 이를 사용하면 됩니다. 아래는 Nest에서 제공하는 데코레이터와 특정 플랫폼에 대한 객체들을 연결한 표입니다.

|데코레이터|객체|
|:---|:---|
|`@Request()`, `@Req()`|`req`|
|`@Response()`, `@Res()` <strong style='color: red'>*</strong>|`res`|
|`@Next()`|`next`|
|`@Session()`|`req.session`|
|`@Param(key?: string)`|`req.params` / `req.params[key]`|
|`@Body(key?: string)`|`req.body` / `req.body[key]`|
|`@Query(key?: string)`|`req.query` / `req.query[key]`|
|`@Headers(name?: string)`|`req.headers` / `req.headers[name]`|
|`@Ip()`|`req.ip`|
|`@HostParam()`|`req.hosts`|

<strong style='color: red'>*</strong> Express나 Fastify 등의 기반 HTTP 플랫폼 간 타입 호환성을 위해, Nest는 `@Res()`와 `@Response()` 데코레이터를 제공합니다. `@Res()`는 `@Response()`의 별명일 뿐, 다른 의미는 없습니다. 둘 모두 기반 네이티브 플랫폼의 `response` 객체의 인터페이스를 직접 노출시킵니다. 이걸 사용할 땐 `@types/express`처럼 기반 라이브러리의 타입도 임포트하는게 좋습니다. 어쨌든, `@Res()`나 `@Response()`를 메서드 핸들러에 사용하는 경우, Nest는 해당 핸들러를 **Library-specific 모드**로 인식합니다. 이 경우, 해당 핸들러의 응답을 개발자가 직접 처리해야 합니다. 즉, `res.json(...)`이나 `res.send(...)` 등의 방법으로 응답 객체를 호출하여 직접 응답을 만들어내야 하며, 하지 않으면 HTTP 서버가 중단됩니다.

> **팁**
> 
> 자신만의 데코레이터를 만들어보고 싶다면, [여기](https://docs.nestjs.com/custom-decorators)를 참고하세요.

### 자원

앞에서 우리는 `cats` 자원을 가져오는 엔드포인트를 **GET**으로 정의했습니다. 일반적으로는, 새로운 데이터를 만들기 위한 엔드포인트도 제공하고 싶어질 겁니다. 그러면, **POST** 핸들러를 만들어봅시다.

```typescript
// cats.controller.ts
import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  create(): string {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

간단하죠? Nest는 모든 표준 HTTP 메서드에 대한 데코레이터를 제공합니다: `@Get()`, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`, `@Options()`, `@Head()`. 또, 모든 메서드에 대한 엔드포인트를 정의하고 싶을 때에는 `@All()`을 쓰면 됩니다.

### 라우트 와일드카드

패턴 기반 라우팅도 지원합니다. 예를 들면, 애스터리크(별표, *)는 모든 문자 조합과 매치되는 와일드카드로 사용됩니다.

```typescript
@Get('ab*cd')
findAll() {
  return 'This route uses a wildcard';
}
```

`'ab*cd'` 라우트 경로는 `abcd`, `ab_cd`, `abecd` 등에 매치됩니다. 라우트 경로에는 `?`, `+`, `*`, `()`를 쓸 수 있으며, 각각은 정규표현식의 같은 문자에 대응되는 부분집합입니다.

### 상태 코드

위에서 말했던 것처럼, POST 요청이 **201**인 것을 제외하면 모든 응답의 **상태 코드**는 기본적으로 **200**입니다. 응답의 상태 코드를 바꾸려면 핸들러에 `@HttpCode(...)`를 붙이면 됩니다.

```typescript
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
```

> **팁**
> 
> `HttpCode`는 `@nestjs/common` 패키지에서 임포트하세요.

가끔 상태 코드가 정적이지 않고, 여러 요소에 따라 바뀌어야 할 때가 있습니다. 이 경우, `@Res()`를 통해 주입 받은 library-specific **응답** 객체를 사용하시면 됩니다. 만약 오류가 발생해서 상태 코드를 바꿔야할 경우, 예외를 발생시키면 됩니다.

### 헤더

직접 응답 헤더를 지정하려면, `@Header()` 데코레이터를 사용하거나 library-specific 응답 객체의 `res.header()`를 직접적으로 호출하면 됩니다.

```typescript
@Post()
@Header('Cache-Control', 'none')
create() {
  return 'This action adds a new cat';
}
```

> **팁**
> 
> `Header`는 `@nestjs/common` 패키지에서 임포트하세요.

### 리다이렉션

응답을 특정 URL으로 리다이렉트 하려면, `@Redirect()` 데코레이터를 사용하거나 library-specific 응답 객체의 `res.redirect()`를 직접적으로 호출하면 됩니다.

`@Redirect()`는 `url`과 `statusCode` 두 선택 인수를 받습니다. 이때, `statusCode`의 기본 값은 `302`(`Found`)입니다.

```typescript
@Get()
@Redirect('https://nestjs.com', 301)
```

가끔, HTTP 상태 코드나 리다이렉트 URL을 동적으로 결정하고 싶을 때가 있을 겁니다. 그때는 아래의 형식을 가진 객체를 라우트 핸들러에서 반환하면 됩니다.

```typescript
{
  "url": string,
  "statusCode": number
}
```

반환된 값은 `@Redirect()` 데코레이터의 인수를 덮어씌웁니다. 예를 들면 아래와 같습니다.

```typescript
@Get('docs')
@Redirect('https://docs.nestjs.com', 302)
getDocs(@Query('version') version) {
  if (version && version === '5') {
    return { url: 'https://docs.nestjs.com/v5/' };
  }
}
```

### 라우트 파라미터

`GET /cats/1`로 아이디가 `1`인 고양이를 가져오고 싶을 때처럼 요청의 일부로 **동적인 데이터**를 가져올 필요가 있을 때, 정적인 경로로 설정된 라우트는 제대로 동작하지 않을 것입니다. 파라미터와 함께 라우트를 정의하려면, 요청 URL에서 가져올 동적 데이터가 있는 위치에 라우트 파라미터 **토큰**을 추가하면 됩니다. 아래에서, `@Get()` 데코레이터에 라우트 파라미터 토큰을 사용한 예를 보여줍니다. 이렇게 선언된 라우트 파라미터는 메서드 시그니처에 붙일 수 있는 `@Param()` 데코레이터를 통해 접근할 수 있습니다.

```typescript
@Get(':id')
findOne(@Param() params): string {
  console.log(params.id);
  return `This action returns a #${params.id} cat`;
}
```

`@Param()`은 위 예시에서의 `params`처럼 메서드 파라미터에 붙여, 메서드 안에서 **라우트** 파라미터를 데코레이터를 붙인 변수의 속성으로 사용할 수 있게 합니다. 위 코드에서는, `params.id`를 참조하여 `id` 파라미터에 접근하였습니다. 데코레이터에 특정 파라미터의 토큰을 넘겨주면, 해당 이름을 가진 라우트 파라미터를 바로 참조할 수 있습니다.

> **팁**
> 
> `Param`은 `@nestjs/common` 패키지에서 임포트하세요.

```typescript
@Get(':id')
findOne(@Param('id') id: string): string {
  return `This action returns a #${id} cat`;
}
```

### 서브 도메인 라우팅

`@Controller()` 데코레이터에 `host` 옵션을 주면, 들어오는 요청의 HTTP host가 설정 값과 일치한 요청만 받도록 설정할 수 있습니다.

```typescript
@Controller({ host: 'admin.example.com' })
export class AdminController {
  @Get()
  index(): string {
    return 'Admin page';
  }
}
```

> **주의**
> 
> **Fastify**의 중첩 라우트 지원이 부족하기 때문에, 서브 도메인 라우팅을 사용할 때는 Express 어댑터가 대신 사용됩니다.

위의 라우트 파라미터와 비슷하게, `hosts` 옵션은 호스트 네임의 특정 위치에 있는 동적 갑을 가져오기 위해 토큰을 사용할 수 있습니다. 아래에서, `@Controller()` 데코레이터에 호스트 파라미터 토큰을 사용한 예를 보여줍니다. 이렇게 정의된 호스트 파라미터는 메서드 시그니처에 추가할 수 있는 `@HostParam()` 데코레이터를 통해 접근할 수 있습니다.

```typescript
@Controller({ host: ':account.example.com' })
export class AccountController {
  @Get()
  getInfo(@HostParam('account') account: string) {
    return account;
  }
}
```

### 스코프

다른 프로그래밍 언어 배경을 가진 사람들은 Nest에 들어오는 요청들 간의 거의 모든 것이 공유된다는 것을 받아들이기 힘들 수 있습니다. Nest는 데이터베이스에 대한 연결 풀, 전역 상태의 싱글톤 서비스 등, 여러 공유되는 리소스를 갖고 있습니다. 왜 이렇게 설계되었는지를 이해하려면, 먼저 Node.js는 각각의 요청을 분리된 쓰레드로 처리하는 무상태 요청/응답 멀티 쓰레드 모델을 따르지 않는다는 것을 알아야 합니다. Nest는 Node.js 위에서 동작하기 때문에, 싱글톤 인스턴스를 사용하는 것이 우리의 환경에서는 가장 **안전**합니다.

그러나, GraphQL 어플리케이션에서 각 요청에 대한 캐싱을 하거나, 요청 트래킹, 멀티테넌시(multi-tenancy) 등 요청 기반 수명을 갖는 컨트롤러가 필요한 경우 같은 예외 상황도 존재합니다. 그 때에는 스코프를 다르게 설정해야 하는데, [여기](https://docs.nestjs.com/fundamentals/injection-scopes)를 참고하세요.

### 비동기

우리는 모던 자바스크립트를 좋아하고, 여기서 데이터 추출은 대게 **비동기**로 이루어짐을 알고 있습니다. 그렇기 때문에, Nest는 비동기 함수를 잘 지원하고 잘 작동시킵니다.

> **팁**
> 
> `async / await` 기능에 대해 더 알아보려면 [여기](https://kamilmysliwiec.com/typescript-2-1-introduction-async-await)를 참고하세요.

모든 비동기 함수는 `Promise`를 반환해야 합니다. 이는 개발자가 지연된 값을 반환하면, Nest가 스스로 `resolve`한다는 것을 뜻합니다. 아래 예시를 봅시다.

```typescript
// cats.controller.ts
@Get()
async findAll(): Promise<any[]> {
  return [];
}
```

위 코드는 잘 작동됩니다. 게다가 Nest의 라우트 핸들러는 RxJS의 [Observable 스트림](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html) 또한 반환할 수 있기 때문에 더욱 강력합니다. Nest는 자동으로 스트림을 subscribe 하고, 스트림이 한 번 완료되면 마지막에 발생한 값을 가져옵니다.

```typescript
// cats.controller.ts
@Get()
findAll(): Observable<any[]> {
  return of([]);
}
```

위의 두 방법은 모두 잘 작동하며, 필요에 따라 사용하면 됩니다.

### 요청의 바디 데이터

이전의 POST 라우트 핸들러 예시는 클라이언트의 매개변수를 받지 않았습니다. 이를 `@Body()` 데코레이터를 추가하여 고쳐봅시다.

그 전에, 타입스크립트를 사용한다면 먼저 **DTO**(Data Transfer Object) 스키마를 정의해야 합니다. DTO는 네트워크를 통해서 어떻게 데이터가 보내질 것인가를 정의한 객체입니다. DTO 스키마는 **타입스크립트**의 인터페이스나 간단하게 클래스를 사용하여 정의할 수 있습니다. 여기에는 클래스를 쓸 것을 추천드립니다. 그 이유는, 클래스는 자바스크립트 ES6 표준의 한 부분이므로 자바스크립트로 컴파일 될 때, 사라지지 않고 실제 요소로 보존되기 때문입니다. 또, 타입스크립트의 인터페이스는 트랜스파일(transpile) 과정에서 사라지기 때문에, Nest가 런타임에 사용할 수가 없게 됩니다. 이는 **Pipes** 같은 기능에서 런타임에 변수의 메타타입에 접근할 수도 있기 때문에 중요합니다.

자, 이제 `CreateCatDto` 클래스를 만들어봅시다.

```typescript
// create-cat.dto.ts
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

위 클래스는 세 가지의 기본적인 속성을 갖고 있습니다. 이제 `CatsController` 안에서 새로 만든 DTO를 아래와 같이 사용할 수 있습니다.

```typescript
// cats.controller.ts
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  return 'This action adds a new cat';
}
```

> **팁**
> 
> 메서드 헨들러에 들어오면 안되는 속성들을 `ValidationPipe`를 통해서 걸러낼 수 있습니다. 이 경우, 허용되는 속성을 설정하고(whitelist) 허용되지 않는 속성은 자동적으로 결과 객체에서 뺄 수 있습니다. 위의 `CreateCatDto` 예시의 경우, 우리가 허용한 속성은 `name`, `age`, `breed`가 됩니다. 더 알아보시려면, [여기](https://docs.nestjs.com/techniques/validation#stripping-properties)를 참고하세요.

### 에러 처리

예외를 활용하는 법 등, 에러를 처리하는 방법은 [여기](https://docs.nestjs.com/exception-filters)로 나눠졌습니다.

### 활용 예시

아래는 몇몇 데코레이터를 사용한 기본적인 컨트롤러의 예시입니다. 이 컨트롤러는 내부 데이터에 접근하고, 조작할 수 있는 여러 메서드를 제공합니다.

```typescript
// cats.controller.ts
import { Controller, Get, Query, Post, Body, Put, Param, Delete } from '@nestjs/common';
import { CreateCatDto, UpdateCatDto, ListAllEntities } from './dto';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return `This action removes a #${id} cat`;
  }
}
```

> **팁**
> 
> Nest CLI는 위의 모든 일을 할 필요가 없게 **모든 기반 코드**를 자동으로 생성하여 개발자 경험이 더 간단하게 만드는 생성기를 제공합니다. 이 기능에 대해서 더 알아보고 싶다면, [여기](https://docs.nestjs.com/recipes/crud-generator)를 참고하세요.

### 시작 및 실행

위의 컨트롤러가 모두 정의되어도, Nest는 `CatsController`가 존재한다는 사실을 모르며, 이 때문에 클래스의 인스턴스가 생성되지 않습니다.

컨트롤러는 항상 모듈에 속해아 하므로, `@Module()` 데코레이터 내의 `controllers` 배열에 추가해주어야 합니다. 아직 `AppModule`을 제외하고는 아무 모듈도 정의하지 않았으므로, 이 모듈을 이용해서 Nest에게 `CatsController`를 알려줍시다.

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
  controllers: [CatsController],
})
export class AppModule {}
```

`@Module()` 데코레이터를 달아서, 모듈 클래스에 메타데이터를 설정했습니다. 이제, Nest는 어떤 컨트롤러를 마운트해야 하는지 쉽게 알 수 있습니다.

### 특정 라이브러리에 대한 방법

지금까지 Nest가 응답을 다루는 표준 방법에 대해서 이야기 했습니다. 응답을 다루는 두 번째 방법은, library-specific [응답 객체](https://expressjs.com/en/api.html#res)를 사용하는 것입니다. 특정 응답 객체를 가져오기 위해서는, `@Res()` 데코레이터를 사용해야 합니다. 차이점을 보기 위해, 아래와 같이 `CatsController`를 다시 써보았습니다.

```typescript
// cats.controller.ts
import { Controller, Get, Post, Res, HttpStatus } from '@nestjs/common';
import { Response } from 'express';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Res() res: Response) {
    res.status(HttpStatus.CREATED).send();
  }

  @Get()
  findAll(@Res() res: Response) {
     res.status(HttpStatus.OK).json([]);
  }
}
```

위 방법이 잘 작동하고, 응답 객체에 대한 완전한 조작(헤더, 특정 라이브러리에 대한 기능 등)을 제공하기 때문에 더 유연한 것은 맞지만, 주의를 가지고 사용해야 합니다. 일반적으로, 이 방법은 덜 명확해보이기도 하고 몇몇의 단점도 있습니다. 가장 큰 단점은 코드가 특정 플랫폼에 의존하게 된다는 것과, 테스트하기 힘들어진다는 것(가짜 응답 객체를 만들어야 함), 두 가지를 들 수 있습니다.

또한 위 예시의 경우, 인터셉터나 `@HttpCode()`, `Header()` 데코레이터 등 Nest의 표준 응답 처리 방법에 의존한 Nest의 기능들을 사용할 수 없게 됩니다. 이를 고치려면, 아래와 같이 `passthrough` 옵션을 `true`로 주면 됩니다.

```typescript
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.status(HttpStatus.OK);
  return [];
}
```

이제 어떠한 조건에 따라 쿠키나 헤더를 설정하는 등, 네이티브 객체와 상호작용 가능하면서 동시에 나머지는 프레임워크가 처리할 수 있도록 할 수 있습니다.