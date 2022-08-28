# Circular dependency

원문 : [https://docs.nestjs.com/fundamentals/circular-dependency](https://docs.nestjs.com/fundamentals/circular-dependency)

순환 의존(순환 참조)은 두 클래스가 서로를 의존하고 있을 때 발생합니다. 예를 들어, 클래스 A가 클래스 B를 필요로 하는데, 클래스 B도 클래스 A를 필요로 하는 거죠. Nest 안에서는 모듈 간, 혹은 프로바이더 간에서 순환 의존이 발생할 수 있습니다.

가능하다면 순환 의존은 피하는게 좋지만, 항상 그럴 수는 없습니다. 이 경우, Nest는 프로바이더 간에 순환 의존을 해결할 수 있는 두 가지 방법을 제공합니다. 이 챕터에서는, **정방향 참조(Forward Referencing)** 기법을 사용하는 방법과, **ModuleRef** 클래스를 사용하여 DI 컨테이너에서 프로바이더 인스턴스를 받아오는 기법을 사용하는 방법에 대해서 설명합니다.

또한, 모듈 간의 순환 의존을 해결하는 방법도 설명합니다.

> **주의**
>
> 순환 의존은 임포트를 그룹화하기 위해 "barrel files"/index.ts를 사용할 때도 발생할 수 있습니다. 모듈/프로바이더 클래스에서는 Barrel files를 생략해야 합니다. 예를 들어, Barrel files와 동일한 디렉토리 내의 파일을 가져올 때, Barrel files를 사용하면 안 됩니다. (i.e. `cats/cats.controller`는 `cats/cats.serice`를 가져올 때 `cats`를 가져와서는 안됩니다.) 더 자세하게 알아보려면 [이 깃헙 이슈](https://github.com/nestjs/nest/issues/1181#issuecomment-430197191)를 참고해주세요.

### 정방향 참조 (Forward reference)

**정방향 참조**는 `forwardRef()` 유틸리티 함수를 사용하여 Nest가 아직 정의되지 않은 클래스를 참조할 수 있게 해줍니다. 예를 들어, `CatsService`와 `CommonService`가 서로를 의존하고 있다면, 두 프로바이더 각각에 `@Inject()`와 `forwardRef()` 유틸리티를 사용하여 순환 의존을 해결할 수 있습니다. 그렇지 않으면, 필수적인 메타데이터가 존재하지 않기 때문에 Nest는 해당 프로바이더를 인스턴스화할 수 없습니다. 예시를 한 번 봅시다!

```typescript
// cats.service.ts
@Injectable()
export class CatsService {
  constructor(
    @Inject(forwardRef(() => CommonService))
    private commonService: CommonService
  ) {}
}
```

> **팁**
>
> `forwadRef()` 함수는 `@nestjs/common` 패키지에서 가져올 수 있습니다.

한 쪽은 처리했으니, `CommonService`에도 같은 작업을 해줍시다.

```typescript
// common.service.ts
@Injectable()
export class CommonService {
  constructor(
    @Inject(forwardRef(() => CatsService))
    private catsService: CatsService
  ) {}
}
```

> **주의**
>
> 인스턴스화의 순서는 결정되지 않습니다. 따라서, 어떤 생성자가 먼저 호출된다고 가정하고 코드를 만들면 안됩니다. `Scope.REQUEST` 스코프의 프로바이더에 의존하는 순환 의존이 발생한 경우, 정의되지 않은 의존성이 발생할 수 있습니다. 더 자세한 정보는 [여기](https://github.com/nestjs/nest/issues/5778)를 참고해주세요.

### ModuleRef 클래스

`forwardRef()`를 사용하는 대신, 코드를 리팩터링하고 `ModuleRef` 클래스를 사용하여 순환 관계의 한 쪽에서 프로바이더를 가져올 수 있습니다. `ModuleRef` 유틸리티 클래스에 대해서 더 알아보려면 [여기](https://docs.nestjs.com/fundamentals/module-ref)를 참고해주세요.

### 모듈 정방향 참조

모듈 간 순환 의존을 해결하려면, `forwardRef()` 유틸리티 함수를 각각의 모듈 연결부에 사용하면 됩니다. 예를 들면 아래와 같습니다.

```typescript
// common.module.ts
@Module({
  imports: [forwardRef(() => CatsModule)],
})
export class CommonModule {}
```

### 문서 기여자

- [러리](https://github.com/Coalery)
