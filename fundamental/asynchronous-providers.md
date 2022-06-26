# Asynchronous providers

원문 : [https://docs.nestjs.com/fundamentals/custom-providers](https://docs.nestjs.com/fundamentals/custom-providers)

가끔 하나 이상의 **비동기 작업**이 완료될 때까지 어플리케이션 시작을 지연시켜야 할 때가 있습니다. 예를 들면, 데이터베이스와의 연결이 완료되기 전까지 요청을 받고 싶지 않을 수 있죠. 이 문제는 비동기 프로바이더를 통해 해결할 수 있습니다.

바로 `useFactory`와 `async/await`를 함께 사용하면 됩니다! `useFactory`가 `Promise` 타입을 받기 때문에, 팩토리 함수에서는 `await`를 사용하여 비동기 작업을 기다릴 수 있습니다. 이렇게 하면, Nest는 비동기 작업이 완료될 때까지 해당 프로바이더에 의존하는 클래스를 인스턴스화 시키지 않고 기다리게 됩니다.

```typescript
{
  provide: 'ASYNC_CONNECTION',
  useFactory: async () => {
    const connection = await createConnection(options);
    return connection;
  },
}
```

> **팁**
> 
> 커스텀 프로바이더에 대해 더 알아보려면 [여기](https://docs.nestjs.com/fundamentals/custom-providers)를 참고해주세요.

### 주입

비동기 프로바이더도 다른 프로바이더와 마찬가지로 토큰을 통해 다른 곳에 주입할 수 있습니다. 위의 예시에서는 `@Inject('ASYNC_CONNECTION')`을 사용하면 됩니다.

### 예시

비동기 프로바이더에 대한 더 많은 예시는 [TypeORM 레시피](https://docs.nestjs.com/recipes/sql-typeorm)를 참고해주세요.

### 문서 기여자

- [러리](https://github.com/Coalery)
