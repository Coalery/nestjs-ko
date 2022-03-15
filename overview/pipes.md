---
description: "원문 : https://docs.nestjs.com/pipes"
---

# Pipes

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