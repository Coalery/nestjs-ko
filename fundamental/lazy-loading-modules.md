# Lazy-loading modules

원문 : [https://docs.nestjs.com/fundamentals/lazy-loading-modules](https://docs.nestjs.com/fundamentals/lazy-loading-modules)

이하 Lazy-loading은 지연 로딩이라고 지칭합니다.

일반적으로, 모듈들은 열심히 로딩됩니다. 어플리케이션이 로드되자마자 모든 모듈들이 즉시 필요하던 말던 모두 로딩된다는 뜻입니다. 대부분의 어플리케이션에선 이러한 작동 방식에 문제가 없으나, 시작할 때 발생하는 레이턴시, 즉 "콜드 스타트"가 큰 문제가 되는 **서버리스 환경**에서 동작하는 앱이나 워커들은 이 부분이 병목 지점이 될 수 있습니다.

지연 로딩은 특정 서버리스 함수 호출에 필요한 모듈만 불러오기 때문에, 부트스트랩 시간을 줄이는 데에 도움이 될 수 있습니다. 게다가, 서버리스 함수가 한 번 작동을 하면, 다른 모듈을 비동기적으로 불러와서 이후 호출에 대한 부트스트랩 시간을 훨씬 더 단축할 수 있습니다.

> **팁**
>
> 만약 **앵귤러** 프레임워크에 친숙하다면, 이전에 "지연 로딩 모듈"에 대해 본 적이 있을 겁니다. 하지만 이 기술은 단어는 같지만 Nest에서는 **기술적으로 다르므로**, 완전히 다른 기능이라고 생각해주시면 됩니다.

> **주의**
>
> 지연 로딩되는 모듈과 서비스에서는 [생명 주기 훅 메서드](https://docs.nestjs.com/fundamentals/lifecycle-events)가 호출되지 않습니다.

### 시작해봅시다.

모듈을 필요할 때 로드하기 위해, Nest는 일반적인 방법으로 클래스에 주입 받을 수 있는 `LazyModuleLoader` 클래스를 제공합니다.

```typescript
// cats.service.ts
@Injectable()
export class CatsService {
  constructor(private lazyModuleLoader: LazyModuleLoader) {}
}
```

> **팁**
>
> `LazyModuleLoader` 클래스는 `@nestjs/core` 패키지에서 가져올 수 있습니다.

아니면 아래와 같이 어플리케이션 부트스트랩 파일(`main.ts`)에서 `LazyModuleLoader` 프로바이더의 객체를 받을 수 있습니다.

```typescript
// "app" represents a Nest application instance
const lazyModuleLoader = app.get(LazyModuleLoader);
```

위와 같이 하면, 이제 어떤 모듈이던 아래와 같이 불러올 수 있습니다.

```typescript
const { LazyModule } = await import("./lazy.module");
const moduleRef = await this.lazyModuleLoader.load(() => LazyModule);
```

> **팁**
>
> "지연 로딩된" 모듈은 첫번째로 `LazyModuleLoader#load` 메서드를 호출했을 때 캐싱됩니다. 이는, 연속적인 `LazyModule` 로딩이 **아주 빨라지며**, 모듈을 다시 로딩하는 게 아니라 캐싱된 인스턴스를 반환한다는 걸 뜻합니다.
>
> ````Load "LazyModule" attempt: 1
> time: 2.379ms
> Load "LazyModule" attempt: 2
> time: 0.294ms
> Load "LazyModule" attempt: 3
> time: 0.303ms```
> ````
>
> 또한, "지연 로딩된" 모듈은 어플리케이션 부트스트랩 과정에서 로딩된 모듈들과 같은 모듈 그래프를 공유하며, 이후에 등록된 다른 지연 모듈들과도 공유됩니다.

`lazy.module.ts`는 **일반적인 Nest 모듈**을 내보내는 타입스크립트 파일이며, 추가적인 수정이 필요하지 않습니다.

`LazyModuleLoader#load` 메서드는 `LazyModule`의 [모듈 레퍼런스](https://docs.nestjs.com/fundamentals/module-ref)를 반환하며, 이를 통해 모듈 내부에 등록되어 있는 프로바이더들에 접근할 수 있고, 어떠한 프로바이더던 각각의 주입 토큰을 해당 레퍼런스를 가져올 수 있습니다.

예를 들어, `LazyModule`이 아래와 같이 정의되어 있다고 합시다.

```typescript
@Module({
  providers: [LazyService],
  exports: [LazyService],
})
export class LazyModule {}
```

> **팁**
>
> 지연 로딩된 모듈은 애초에 말이 안되기 때문에 **전역 모듈**로 등록될 수 없습니다. 모듈의 로딩이 지연되려면, 해당 모듈을 사용할 가능성이 있는 정적으로 등록된 모든 모듈들이 이미 인스턴스화 되어있어야 하기 때문입니다. 마찬가지로, 등록된 **전역 가드, 인터셉터 등** 또한 제대로 동작하지 않습니다.

이렇게 하면, `LazyService` 프로바이더의 레퍼런스를 아래와 같이 얻을 수 있습니다.

```typescript
const { LazyModule } = await import("./lazy.module");
const moduleRef = await this.lazyModuleLoader.load(() => LazyModule);

const { LazyService } = await import("./lazy.service");
const lazyService = moduleRef.get(LazyService);
```

> **주의**
>
> 만약 **Webpack**을 사용하고 있다면, `tsconfig.json`파일에서 `compilerOptions.module`을 `esnext`로, `compilerOptions.moduleResolution`을 `node`로 설정해주세요. 없다면 추가해주세요.
>
> ```json
> {
>   "compilerOptions": {
>     "module": "esnext",
>     "moduleResolution": "node",
>     ...
>   }
> }
> ```
>
> 위와 같이 옵션을 설정해주면, [Code Splitting](https://webpack.js.org/guides/code-splitting/) 기능을 사용할 수 있습니다.

### 문서 기여자

- [러리](https://github.com/Coalery)
