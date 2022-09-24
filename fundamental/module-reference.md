# Module reference

원문 : [https://docs.nestjs.com/fundamentals/module-ref](https://docs.nestjs.com/fundamentals/module-ref)

Nest는 내부 프로바이더 리스트를 탐색하고, 주입 토큰을 조회 키로 사용하여 프로바이더의 레퍼런스를 가져올 수 있는 `ModuleRef` 클래스를 제공합니다. 또한, `ModuleRef` 클래스는 정적인 프로바이더와 스코프를 갖는 프로바이더를 동적으로 인스턴스화 할 수 있는 방법을 제공합니다. `ModuleRef`는 아래와 같이 일반적인 방법으로 클래스에 주입 해줄 수 있습니다.

```typescript
// cats.service.ts
@Injectable()
export class CatsService {
  constructor(private moduleRef: ModuleRef) {}
}
```

> **팁**
>
> `ModuleRef` 클래스는 `@nestjs/core` 패키지에서 가져올 수 있습니다.

### 인스턴스 가져오기

`ModuleRef` 인스턴스(이하 **모듈 레퍼런스**)는 `get()` 메서드를 갖고 있습니다. 이 메서드는 주입 토큰이나 클래스 이름을 사용하여 **현재** 모듈 안에 존재하는 프로바이더, 컨트롤러, 또는 가드, 인터셉터 등을 가져올 수 있습니다.

```typescript
// cats.service.ts
@Injectable()
export class CatsService implements OnModuleInit {
  private service: Service;
  constructor(private moduleRef: ModuleRef) {}

  onModuleInit() {
    this.service = this.moduleRef.get(Service);
  }
}
```

> **주의**
>
> Transient나 Request-scoped 프로바이더의 경우, `get()` 메서드로 갖고 올 수 없습니다. 대신에, [아래](https://docs.nestjs.com/fundamentals/module-ref#resolving-scoped-providers)에서 설명하는 기술을 사용해야 합니다. 스코프를 제어하는 방법에 대해서 배우시려면 [여기](https://docs.nestjs.com/fundamentals/injection-scopes)를 참고해주세요.

다른 모듈에 주입된 프로바이더처럼, 전역 컨텍스트로부터 프로바이더를 가져오고 싶다면, `get()`의 두 번째 매개변수로 `{ strict: false }` 옵션을 넘겨주면 됩니다.

```typescript
this.moduleRef.get(Service, { strict: false });
```

### 스코프를 갖는 프로바이더 만들기

스코프(Transient, request-scoped)를 갖는 프로바이더를 동적으로 만들려면, `resolve()` 메서드의 인수로 해당 프로바이더의 주입 토큰을 넘겨주면 됩니다.

```typescript
// cats.service.ts
@Injectable()
export class CatsService implements OnModuleInit {
  private transientService: TransientService;
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    this.transientService = await this.moduleRef.resolve(TransientService);
  }
}
```

`resolve()` 메서드는 자신의 **DI 컨테이너 서브트리**로부터, 해당 프로바이더의 유일한 인스턴스를 반환합니다. 각 서브트리는 유일한 컨텍스트 식별자를 갖습니다. 따라서, 만약 `resolve()` 메서드를 한 번 이상 호출한 뒤에 각 인스턴스 레퍼런스를 비교해보면, 모두 다른 레퍼런스를 갖는다는 걸 알 수 있습니다.

```typescript
// cats.service.ts
@Injectable()
export class CatsService implements OnModuleInit {
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    const transientServices = await Promise.all([
      this.moduleRef.resolve(TransientService),
      this.moduleRef.resolve(TransientService),
    ]);
    console.log(transientServices[0] === transientServices[1]); // false
  }
}
```

`resolve()` 메서드를 여러번 호출해도 같은 인스턴스를 생성하며, 각각의 호출이 같은 DI 컨테이너 서브트리를 공유하는 걸 보장하게 하고 싶다면, `resolve()` 메서드에 컨텍스트 식별자를 넘겨주면 됩니다. 새로운 컨텍스트 식별자를 생성하려면, `ContextIdFactory` 클래스를 사용하면 됩니다. 해당 클래스는 적절한 유일 식별자를 반환하는 `create()` 메서드를 제공합니다.

```typescript
// cats.service.ts
@Injectable()
export class CatsService implements OnModuleInit {
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    const contextId = ContextIdFactory.create();
    const transientServices = await Promise.all([
      this.moduleRef.resolve(TransientService, contextId),
      this.moduleRef.resolve(TransientService, contextId),
    ]);
    console.log(transientServices[0] === transientServices[1]); // true
  }
}
```

> **팁**
>
> `ContextIdFactory` 클래스는 `@nestjs/core` 패키지에서 가져올 수 있습니다.

### `REQUEST` 프로바이더 등록하기

`ContextIdFactory.create()`를 사용해서 수동으로 만들어진 컨텍스트 식별자의 DI 서브트리에서, `REQUEST` 프로바이더는 `undefined`이 됩니다. `REQUEST` 프로바이더가 인스턴스화 되지 않고, Nest의 의존성 주입 시스템에 의해 관리되지 않기 때문인데요.

수동으로 만들어진 DI 서브 트리에 커스텀 `REQUEST` 객체를 등록하려면, 아래와 같이 `ModuleRef#registerRequestByContextId()` 메서드를 사용하면 됩니다.

```typescript
const contextId = ContextIdFactory.create();
this.moduleRef.registerRequestByContextId(/* YOUR_REQUEST_OBJECT */, contextId);
```

### 현재 서브 트리 가져오기

가끔, **요청 컨텍스트**에서 `REQUEST` 스코프를 갖는 프로바이더의 인스턴스를 만들고 싶을 때가 있을 겁니다. 예를 들면, `CatsService`가 `REQUEST` 스코르를 갖고, 여기에다가 `REQUEST` 스코프의 `CatsRepository` 인스턴스를 만들어내야 한다고 생각해봅시다. 같은 DI 컨테이너 서브 트리를 공유하려면, 새로운 컨텍스트 식별자를 만들지 않고, 현재의 컨텍스트 식별자를 가져와야 합니다. 현재 컨텍스트 식별자를 얻으려면, `@Inject()` 데코레이터를 사용해서 요청 객체를 주입 받으면 됩니다.

```typescript
// cats.service.ts
@Injectable()
export class CatsService {
  constructor(@Inject(REQUEST) private request: Record<string, unknown>) {}
}
```

> **팁**
>
> `REQUEST` 프로바이더에 대해서 더 알아보려면 [여기](https://docs.nestjs.com/fundamentals/injection-scopes#request-provider)를 참고해주세요.

이제, `ContextIdFactory` 클래스의 `getByRequest()` 메서드를 사용해서 요청 객체 기반의 컨텍스트 식별자를 생성하고, `resolve()`에 인수로 넣어주면 됩니다.

```typescript
const contextId = ContextIdFactory.getByRequest(this.request);
const catsRepository = await this.moduleRef.resolve(CatsRepository, contextId);
```

### 동적으로 커스텀 클래스 인스턴스화하기

**이전에 프로바이더로 등록되지 않은** 클래스를 동적으로 인스턴스화 하려면, 모듈 레퍼런스의 `create()` 메서드를 사용하면 됩니다.

```typescript
// cats.service.ts
@Injectable()
export class CatsService implements OnModuleInit {
  private catsFactory: CatsFactory;
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    this.catsFactory = await this.moduleRef.create(CatsFactory);
  }
}
```

이 기술은 프레임워크 컨테이너 밖에서 조건적으로 다른 클래스를 인스턴스화할 수 있게 해줍니다.

### 문서 기여자

- [러리](https://github.com/Coalery)
