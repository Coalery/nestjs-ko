# Pipes

원문 : https://docs.nestjs.com/pipes

파이프는 `@Injectable()` 데코레이터가 붙어있는, `PipeTransform` 인터페이스를 구현하는 클래스입니다.

![Pipe_1.png](https://docs.nestjs.com/assets/Pipe_1.png)

파이프는 일반적으로 두 가지의 경우에서 사용합니다.
- **변형**: 입력 데이터를 원하는 형태로 변형합니다. (ex, 문자열을 정수로)
- **검증**: 입력 데이터를 평가해서 유효하면 변경 없이 그대로 보내주고, 아니라면 예외를 발생시킵니다.

두 경우 모두, [컨트롤러 라우트 핸들러](https://docs.nestjs.com/controllers#route-parameters)의 `arguments`에서 동작합니다. Nest는 파이프를 메서드 실행 전의 위치에 끼워 넣습니다. 그러면 파이프는 메서드에 들어오는 인수를 받아서 처리할 수 있게 됩니다. 파이프에서 변형이나 검증이 처리되면, 라우트 핸들러는 앞의 파이프에서 변형된 인수를 받아서 실행됩니다.

Nest에는 기본적으로 많은 빌트인 파이프가 제공됩니다. 물론, 사용자 지정 파이프도 만들 수 있습니다. 이 챕터에서는, 빌트인 파이프들에 대해 소개하고, 어떻게 라우트 핸들러에 적용할 수 있는지를 보여드릴 겁니다. 그런 다음, 몇 개의 사용자 지정 파이프를 만들면서 어떻게 파이프를 처음부터 만드는지 보여드릴 겁니다.

> **팁**
> 
> 파이프는 예외 지역(exceptions zone)에서 실행됩니다. 이는 파이프가 예외를 발생시키면 예외 층, 즉 전역 예외 필터와 해당 컨텍스트에 적용된 [예외 필터](https://docs.nestjs.com/exception-filters)에서 해당 예외가 처리된다는 것을 뜻합니다. 또한, 파이프에서 예외가 발생하면 컨트롤러의 메서드는 실행되지 않습니다. 이러한 규칙은 시스템 경계에 있는 외부 소스에서 어플리케이션 내로 들어오는 데이터를 쉽게 검증할 수 있는 기술입니다.

### 빌트인 파이프

Nest는 기본적으로 8개의 파이프를 제공합니다.

- `ValidationPipe`
- `ParseIntPipe`
- `ParseFloatPipe`
- `ParseBoolPipe`
- `ParseArrayPipe`
- `ParseUUIDPipe`
- `ParseEnumPipe`
- `DefaultValuePipe`

위 파이프들은 `@nestjs/common` 패키지에서 찾을 수 있습니다.

`ParseIntPipe`의 사용법에 대해 간단하게 살펴보겠습니다. 이는 **변환**의 예시입니다. 메서드 핸들러의 매개변수가 자바스크립트 정수로 바뀌는 것을 보장하며, 변환이 실패한 경우 예외를 발생시킵니다. 이후의 챕터에서, `ParseIntPipe`의 간단한 구현을 보여드릴겁니다. 이 구현 방법은 다른 빌트인 변환 파이프에서도 동일하게 적용됩니다. 변환 파이프는 `ParseBoolPipe`, `ParseFloatPipe`, `ParseEnumPipe`, `ParseArrayPipe`, `ParseUUIDPipe` 등이 있으며, 이 챕터에서는 이러한 형태의 파이프들을 묶어서 `Parse*` 파이프로 부릅니다.

### 파이프 적용하기

파이프를 사용하려면, 파이프 클래스의 인스턴스를 적절한 컨텍스트에 적용해야 합니다. `ParseIntPipe`의 예시로, 특정 라우트 핸들러에 파이프를 구성하고, 메서드가 호출되기 전에 파이프가 실행되는지를 확인해보겠습니다. 아래와 같이 메서드 매개변수 수준에 파이프를 적용할 수 있습니다.

```typescript
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

이렇게 하면, `findOne()` 메서드에서 받는 매개변수가 숫자이거나, 라우트 핸들러가 호출되기 전에 예외가 발생함을 보장합니다.

예를 들어, 아래와 같이 호출하면:

```
GET localhost:3000/abc
```

Nest는 아래와 같이 예외를 발생시킵니다.

```json
{
  "statusCode": 400,
  "message": "Validation failed (numeric string is expected)",
  "error": "Bad Request"
}
```

예외는 `findOne()` 메서드가 실행되는 것을 방지합니다.

위의 예시에서는 인스턴스가 아니라 클래스를 넘겨서, 인스턴스화의 역할을 프레임워크에게 주고 의존성 주입이 가능하게 만들었습니다. 파이프와 가드는 해당 위치에 인스턴스를 넘길 수도 있습니다. 인스턴스를 넘기는 방법은 옵션을 통해 빌트인 파이프의 동작을 변경할 때 사용합니다.

```typescript
@Get(':id')
async findOne(
  @Param('id', new ParseIntPipe({ errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE }))
  id: number,
) {
  return this.catsService.findOne(id);
}
```

다른 모든 변형 파이프도 위 예시와 비슷하게 동작합니다. 이 파이프들은 라우트 파라미터, 쿼리 스트링, 요청의 바디에 있는 값들을 검증하는 상황에서도 잘 작동합니다.

아래의 예시에서는 쿼리 스트링에 파이프를 썼습니다.

```typescript
@Get()
async findOne(@Query('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

또, 아래의 예시는 `ParseUUIDPipe`를 사용하여 라우트 파라미터를 UUID로 파싱하고, 해당 파라미터가 UUID인지를 검증합니다.

```typescript
@Get(':uuid')
async findOne(@Param('uuid', new ParseUUIDPipe()) uuid: string) {
  return this.catsService.findOne(uuid);
}
```

> **팁**
> 
> `ParseUUIDPipe()`를 사용할 때 특정 UUID 버전을 기준으로 파싱하려면, 파이프의 옵션에 버전을 넘겨주면 됩니다.

이때까지, 여러 `Parse*` 빌트인 파이프들에 대해 알아보았습니다. 검증 파이프 적용은 앞과는 살짝 다릅니다. 이에 대해서는 다음 섹션에서 알아보겠습니다.

> **팁**
> 
> 더 많은 검증 파이프의 예시는 [여기](https://docs.nestjs.com/techniques/validation)를 참고해주세요.

### 사용자 지정 파이프

위에서 언급했듯이, 자신만의 파이프도 만들 수 있습니다. Nest에서 `ParseIntPipe`나 `ValidationPipe`를 제공하긴 하지만, 각각의 간단한 버전을 만들어보면서 어떻게 처음부터 파이프를 만드는지 알아봅시다.

간단한 `ValidationPipe`로 시작해보겠습니다. 처음에는, 항등 함수처럼 입력값을 받아서 그대로 반환하는 파이프로 시작합니다.

```typescript
// validation.pipe.ts
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    return value;
  }
}
```

> **팁**
> 
> `PipeTransform<T, R>`는 모든 파이프가 구현해야 하는 제너릭 인터페이스입니다. `T`는 입력되는 `value`의 타입, `R`은 `transform()` 메서드의 반환 타입을 나타냅니다.

모든 파이프는 `PipeTransform` 인터페이스의 조건을 만족시키기 위해 `transform()` 메서드를 구현해야 합니다. 해당 메서드는 아래의 두 가지 매개변수를 가지고 있습니다.

- `value`
- `metadata`

`value`는 라우트 핸들러에 넘겨주기 전의, 현재 처리되는 메서드에 들어오는 인수(argument)입니다. 또, `metadata`는 현재 처리되는 메서드 인수의 메타데이터입니다. 메타데이터 객체는 아래의 속성들을 갖습니다.

```typescript
export interface ArgumentMetadata {
  type: 'body' | 'query' | 'param' | 'custom';
  metatype?: Type<unknown>;
  data?: string;
}
```

위 속성들은 현재 처리되는 인수들을 나타냅니다.

|속성|설명|
|:---|:---|
|`type`|인수가 `@Body()`인지, `@Query()`인지, `@Param()`인지, 혹은 사용자 지정 매개변수인지를 나타냅니다. 더 자세한 내용은 [여기](https://docs.nestjs.com/custom-decorators)를 참고해주세요.|
|`metatype`|`String`처럼 인수의 메타타입을 나타냅니다. 이 값은 라우트 메서드의 시그니처에 타입 선언이 되어있지 않거나, 바닐라 자바스크립트를 쓸 때 `undefined`이 됩니다.|
|`data`|`@Body('string')`에서 `'string'`처럼 데코레이터에 전달된 문자열을 나타냅니다. 아무 값도 안 넣어주면 `undefined`이 됩니다.|

> **주의**
> 
> 타입스크립트의 인터페이스는 트랜스파일 과정에서 사라집니다. 그러므로, 메서드 매개변수의 타입이 클래스 대신에 인터페이스로 선언되어 있다면 `metatype`의 값은 `Object`가 됩니다.

### 스키마 기반 검증

검증 파이프를 더 쓸모 있게 만들어봅시다. `CatsController`의 `create()` 메서드를 더 자세히 살펴보세요! 여기서, 서비스의 메서드를 실행하기 전에 POST의 바디 객체가 유효한지 확인하려 합니다.

```typescript
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

위 코드에서, 바디 매개변수 `createCatDto`의 타입은 `CreateCatDto` 클래스입니다. 해당 클래스는 아래와 같이 생겼습니다.

```typescript
// create-cat.dto.ts
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

`create` 메서드로 들어오는 요청이 유효한 바디를 갖는 걸 보장하게 하려면, `createCatDto` 객체의 세 멤버를 확인해봐야 합니다. 물론, 라우트 핸들러 메서드 안에서 처리할 수도 있겠지만 <strong>단일 책임 원칙(Single Responsibility Rule, SRP)</strong>을 깨뜨리기 때문에 이상적인 방법이 아닙니다.

다른 방법으로는, **검증 클래스**를 만들어서 검증 역할을 위임하는 방법이 있습니다. 하지만, 이 경우에는 각각의 메서드 시작 부분에서 계속 검증 클래스를 호출해야 하는 단점이 있습니다.

검증 미들웨어를 만드는 것은 어떨까요? 동작은 하겠지만, 전체 어플리케이션의 모든 컨텍스트에서 사용할 수 있는 **일반적인 미들웨어**로 만드는 건 불가능할 겁니다. 이는 미들웨어가 핸들러나 매개변수에 대한 정보를 가진 **실행 컨텍스트**에 접근할 수 없기 때문입니다.

눈치채셨겠지만, 이것이 파이프가 만들어진 이유입니다. 그럼, 이제 검증 파이프를 개선하러 가봅시다!

### 객체 스키마 검증

객체를 깔끔하고 [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)하게 검증하는 몇 개의 방법이 있습니다. 그 중 일반적인 방법은, **스키마 기반** 검증을 사용하는 것입니다. 이 방법을 써봅시다.

[Joi](https://github.com/sideway/joi) 라이브러리를 사용하면, 읽기 쉬운 API로 간단하게 스키마를 만들 수 있습니다. Joi 기반 스키마를 사용해서 검증 파이프를 만들어봅시다.

먼저, 필요한 패키지를 설치합니다.

```shell
$ npm install --save joi
$ npm install --save-dev @types/joi
```

아래의 예시 코드에서는, `constructor`의 인수로 스키마를 가져와서 `schema.validate()` 메서드로 주어진 스키마에 대해 들어온 인수를 검증합니다.

위에서 말했듯이, **검증 파이프**는 변하지 않은 값을 반환하거나, 예외를 발생시킵니다.

다음 섹션에서는, 어떻게 `@UsePipes()` 데코레이터로 주어진 컨트롤러 메서드의 적절한 스키마를 가져오는지 알아볼 것입니다. 이를 이용하면, 모든 컨텍스트에서 검증 파이프를 재사용할 수 있게 됩니다.

```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { ObjectSchema } from 'joi';

@Injectable()
export class JoiValidationPipe implements PipeTransform {
  constructor(private schema: ObjectSchema) {}

  transform(value: any, metadata: ArgumentMetadata) {
    const { error } = this.schema.validate(value);
    if (error) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }
}
```

### 검증 파이프 적용하기

앞서 `ParseIntPipe`나 그 외 `Parse*` 파이프 등, 변형 파이프를 어떻게 적용하는지 살펴봤습니다.

검증 파이프 적용도 간단합니다.

먼저, 메서드 수준으로 파이프를 적용해보겠습니다. 현재의 예시에서 `JoiValidationPipe`를 사용하려면 아래의 일을 해야합니다.

1. `JoiValidationPipe`의 인스턴스를 만듭니다.
2. 파이프의 생성자에 특정 컨텍스트에 대한 Joi 스키마를 넘겨줍니다.
3. 파이프를 메서드에 적용시킵니다.

아래와 같이 `@UsePipes()` 데코레이터를 사용하여 위의 일을 수행할 수 있습니다.

```typescript
@Post()
@UsePipes(new JoiValidationPipe(createCatSchema))
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

> **팁**
> 
> `@UsePipes()` 데코레이터는 `@nestjs/common` 패키지에서 가져올 수 있습니다.

### Class Validator

> **주의**
> 
> 이 섹션에서 소개하는 기술은 타입스크립트가 필요합니다. 만약 어플리케이션이 바닐라 자바스크립트로 작성되었다면 아래 기술은 사용할 수 없습니다.

앞서 소개한 검증 기술의 대체제를 살펴보겠습니다.

Nest는 [class-validator](https://github.com/typestack/class-validator)과 잘 작동합니다. 해당 라이브러리를 사용하면, 데코레이터 기반 검증을 할 수 있게 됩니다. 데코레이터 기반 검증은, 처리되는 속성의 메타타입에 접근할 수 있는 Nest의 파이프와 맞물려 동작할 때 매우 유용합니다. 시작하기 전에, 아래의 명령어를 실행하여 필요한 패키지를 설치해야합니다.

```shell
$ npm i --save class-validator class-transformer
```

설치하면, `CreateCatDto` 클래스에 몇 개의 데코레이터를 추가할 수 있게 됩니다. 이것이 이 기술의 중요한 장점인데, 따로 검증 클래스를 만들 필요 없이 `CreateCatDto` 클래스 하나만으로 POST 요청의 바디를 관리할 수 있게 됩니다.

```typescript
// create-cat.dto.ts
import { IsString, IsInt } from 'class-validator';

export class CreateCatDto {
  @IsString()
  name: string;

  @IsInt()
  age: number;

  @IsString()
  breed: string;
}
```

> **팁**
> 
> `class-validator`의 다른 데코레이터를 살펴보려면 [여기](https://github.com/typestack/class-validator#usage)를 참고하세요.

이제, 위의 데코레이터들을 활용하는 `ValidationPipe` 클래스를 만들 수 있게 되었습니다.

```typescript
// validation.pipe.ts
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToClass } from 'class-transformer';

@Injectable()
export class ValidationPipe implements PipeTransform<any> {
  async transform(value: any, { metatype }: ArgumentMetadata) {
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }
    const object = plainToClass(metatype, value);
    const errors = await validate(object);
    if (errors.length > 0) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}
```

> **알림**
> 
> 위의 예시에서 **class-transformer** 라이브러리를 사용했습니다. 해당 라이브러리는 **class-validator** 라이브러리와 같은 사람이 만들었기 때문에, 두 라이브러리는 서로 잘 맞물려서 동작합니다.

이제 위의 코드를 잘 살펴보겠습니다. 먼저, `transform()` 메서드에 `async`가 붙어있는 걸 주목해주세요. Nest는 동기 파이프와 **비동기** 파이프 둘 다 지원하기 때문에, 위와 같이 `async`를 사용한 비동기 파이프도 만들 수 있습니다. 해당 메서드를 비동기로 만든 이유는, 몇 개의 class-validator을 통한 검증이 [비동기로 처리](https://github.com/typestack/class-validator#custom-validation-classes)될 수 있기 때문입니다.

다음으로 주목해야 하는 부분은, 구조 분해(Destructuring)를 사용해서 `ArgumentMetadata`의 멤버 중 metatype 필드를 `metatype` 매개변수로 추출했다는 점입니다. 이는 `ArgumentMetadata`를 완전히 가져와서, 추가적으로 메타타입을 할당하는 과정을 줄여서 간단하게 쓴 것입니다.

그 다음, `toValidate()` 헬퍼 함수를 봐주세요. 이는, 현재 처리되는 변수가 네이티브 자바스크립트 타입일 때, 검증 과정을 넘겨버리는 역할을 합니다. 이들은 검증 데코레이터를 붙일 수 없으므로, 굳이 검증 과정을 거칠 필요가 없으므로 넘기는 것입니다.

또, 검증 과정을 거치기 위해선 기본 자바스크립트 객체를 타입이 있는 객체(typed object)로 변환해야 합니다. 이를 위해 `class-transformer` 라이브러리의 `plainToClass()` 함수를 사용하였습니다. 이렇게 하는 이유는, 네트워크 요청으로부터 바디를 역직렬화 할 때, 들어오는 POST 바디 객체에 대한 **타입 정보가 없기** 때문입니다. 하지만 `class-validator`를 통해 검증하려면 앞서 DTO에서 정의한 검증 데코레이터가 필요합니다. 즉, 들어오는 바디를 기본 바닐라 객체가 아닌 적절한 데코레이터가 붙어있는(decorated) 객체로 처리할 수 있도록 변환해주어야 바디 객체를 검증할 수 있다는 것입니다.

마지막으로, 앞서 언급했듯이 **검증 파이프**는 변하지 않은 값을 반환하거나 예외를 발생시킵니다.

이제, `ValidationPipe`를 적용시켜 봅시다. 파이프는 매개변수 수준, 메서드 수준, 컨트롤러 수준, 전역 수준에 적용시킬 수 있습니다. 앞서 나왔던 Joi 기반 검증 파이프는 메서드 수준에 파이프를 적용한 예시였습니다. 아래의 예시에서는, 파이프 인스턴스를 라우트 핸들러의 `@Body()` 데코레이터에 적용하여 POST 바디를 검증하도록 만들었습니다.

```typescript
// cats.controller.ts
@Post()
async create(
  @Body(new ValidationPipe()) createCatDto: CreateCatDto,
) {
  this.catsService.create(createCatDto);
}
```

매개변수 수준의 파이프는 특정 매개변수에 검증 로직을 적용할 때 유용합니다.

### 전역 수준 파이프

`ValidationPipe`를 최대한 일반적으로 구현하였기 때문에, 해당 파이프를 전체 어플리케이션의 모든 라우트 핸들러에 적용되는 **전역 수준** 파이프로 등록할 수 있습니다.

```typescript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

> **알림**
> 
> [하이브리드 어플리케이션](https://docs.nestjs.com/faq/hybrid-application)의 경우, `useGlobalPipes()`로는 게이트웨이와 마이크로서비스에 파이프를 등록할 수 없습니다. 대신, 하이브리드가 아닌 "표준" 마이크로서비스 어플리케이션에는 `useGlobalPipes()`로 전역 파이프를 등록할 수 있습니다.

전역 파이프는 전체 어플리케이션의 모든 컨트롤러, 모든 라우트 핸들러에 적용됩니다.

위의 예시처럼 모듈 밖에서 `useGlobalPipes()`로 전역 파이프를 등록하면, 적용이 이미 모듈 외부의 컨텍스트에서 끝났기 때문에 의존성 주입이 불가능합니다. 이 문제를 해결하려면, 전역 파이프를 아래처럼 모듈에 직접 설정하면 됩니다.

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_PIPE } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe,
    },
  ],
})
export class AppModule {}
```

> **팁**
> 
> 파이프가 의존성 주입이 되도록 위와 같이 만들면, 어떤 모듈에서 설정했던 파이프는 전역이 됩니다. 따라서, 전역 파이프를 따로 선언하는 모듈을 따로 두시는 게 좋습니다. 또한, `useClass`는 사용자 지정 프로바이더를 등록하는 유일한 방법이 아닙니다. 자세한 건 [여기](https://docs.nestjs.com/fundamentals/custom-providers)를 참고하세요.

### 빌트인 ValidationPipe

위에서 언급했듯이, Nest가 `ValidationPipe`를 기본적으로 제공하기 때문에, 굳이 일반적인 검증 파이프를 직접 구현할 필요는 없습니다. 빌트인 `ValidationPipe`는 이번 챕터에서 만든 샘플보다 더 많은 옵션를 제공하며, 만들어본 샘플은 그저 사용자 지정 파이프의 매커니즘을 설명하기 위해 만든 것일 뿐입니다. 자세한 건 [여기](https://docs.nestjs.com/techniques/validation)를 참고해주세요.

### 변형 파이프 사용 예시

검증 파이프만이 커스텀 파이프의 사용 예시인 것은 아닙니다. 이 챕터를 시작할 때, 파이프는 입력 데이터를 원하는 형태로 **변형**할 수도 있다고 언급했습니다. 이는 `transform` 함수의 반환 값이 이전의 인수 값을 완전하게 덮어쓰기 때문에 가능합니다.

이건 언제 유용할까요? 클라이언트에서 전달된 데이터가 라우트 핸들러 메서드로 들어가기 전에, 문자열을 정수로 변환하는 등의 변형이 필요한 경우를 생각해보세요! 아니면 어떤 필수 필드의 데이터가 들어오지 않았을 때 기본값을 적용하고 싶을 때도 있을 것입니다. **변형 파이프**는 클라이언트 요청과 요청 핸들러 사이에 끼어들어서 이러한 기능들을 수행합니다.

여기, 문자열을 정수값으로 변환하는 간단한 `ParseIntPipe`가 있습니다. 물론 위에서 보았듯이 Nest는 더 정교한 빌트인 `ParseIntPipe`를 갖고 있습니다. 아래의 파이프는 그저 커스텀 변형 파이프의 간단한 예시를 보여주기 위함입니다.

```typescript
// parse-int.pipe.ts
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParseIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException('Validation failed');
    }
    return val;
  }
}
```

이제, 아래와 같이 매개변수에 변형 파이프를 적용해봅시다.

```typescript
@Get(':id')
async findOne(@Param('id', new ParseIntPipe()) id) {
  return this.catsService.findOne(id);
}
```

다른 변형 파이프의 유용한 예시는, 요청으로 들어온 아이디를 통해 데이터베이스에서 **기존 유저** 엔티티를 가져오는 것입니다.

```typescript
@Get(':id')
findOne(@Param('id', UserByIdPipe) userEntity: UserEntity) {
  return userEntity;
}
```

`UserByIdPipe`의 구현은 독자의 몫으로 남겨두겠습니다. 이 파이프도 다른 모든 변형 파이프들과 마찬가지로, 입력값(`id`)을 받아서 출력값(`UserEntity`)을 반환한다는 것만 기억하시면 됩니다. 이는 보일러플레이터 코드를 핸들러에서 꺼내서 일반적인 파이프로 만듦으로써, 코드를 더 선언적이고 [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)하게 만들 수 있습니다.

### 기본값 설정

`Parse*` 파이프들은 `null`이나 `undefined` 값을 받으면 예외를 발생시기 때문에, 해당 파이프들을 사용하려면, 매개변수의 값이 정의되어 있어야 합니다. 만약 쿼리스트링 매개변수의 값이 없더라도 정상적으로 처리되게 하려면, `Parse*` 파이프가 해당 값들을 처리하기 전에 주입할 기본 값을 제공해야 합니다. 이를 위한 것이 바로 `DefaultValuePipe`입니다. 다음과 같이 관련 있는 `Parse*` 파이프 앞에 `DefaultValuePipe`를 두면 됩니다.

```typescript
@Get()
async findAll(
  @Query('activeOnly', new DefaultValuePipe(false), ParseBoolPipe) activeOnly: boolean,
  @Query('page', new DefaultValuePipe(0), ParseIntPipe) page: number,
) {
  return this.catsService.findAll({ activeOnly, page });
}
```

### 문서 기여자

- [러리](https://github.com/Coalery)
- [cpprhtn](https://github.com/cpprhtn)
