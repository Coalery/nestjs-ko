---
description: "원문 : https://docs.nestjs.com/exception-filters"
---

# Exception filters

Nest는 어플리케이션의 모든 처리되지 않은 예외를 처리하는 역할을 하는 <strong>예외 층(Exceptions Layer)</strong>를 갖고 있습니다. 어플리케이션 코드에서 예외가 처리되지 않았을 때, 해당 예외가 예외 층에서 잡혀서, 자동으로 적당한 유저 친화적 응답을 전송합니다.

![Filter_1.png](https://docs.nestjs.com/assets/Filter_1.png)

위 행위(예외 처리)는 내장된 **전역 예외 필터**에 의해 실행되며, `HttpException`과 그 자식 클래스에 대한 예외를 처리합니다. 만약 예외가 `HttpException`도 아니고 그 자식 클래스도 아니어서 예외를 인식할 수 없을 때에는, 내장된 예외 필터가 아래의 기본 JSON 응답을 만들어냅니다.

```json
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

> **팁**
> 
> 전역 예외 필터는 부분적으로 `http-errors` 라이브러리를 지원합니다. 기본적으로, `statusCode`와 `message` 속성을 가진 예외는 적절히 채워져서 응답으로 보내집니다. 그 외의 인식되지 않은 예외는 `InternalServerErrorException`으로 처리됩니다.