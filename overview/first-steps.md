# First steps

원문 : https://docs.nestjs.com/first-steps

이 글에서는 Nest의 핵심을 배우게 됩니다. 기본적인 CRUD 어플리케이션을 만들어보며 Nest의 필수 구성 요소들과 친해져보겠습니다!

### 언어

우리는 타입스크립트도 좋아하지만, 그 이전에 Node.js를 좋아합니다. 이 때문에, Nest는 타입스크립트와 순수 자바스크립트 둘 다 사용할 수 있도록 만들어졌습니다. Nest는 언어의 최신 기능을 사용하므로, 만약 순수 자바스크립트를 사용하려 하신다면 [Babel](https://babeljs.io/) 컴파일러를 먼저 준비해주세요.

여기서 제공하는 대부분의 예시는 모두 타입스크립트를 사용하지만, 언제나 우측 상단에 있는 버튼을 눌러서 바닐라 자바스크립트로 이루어진 코드를 볼 수 있습니다.

> **역주** : 위 기능의 경우 본 페이지에서는 제공하지 않습니다. 원문을 참고하시길 바랍니다.

### 사전 준비

v13을 제외한 10.13.0 버전 이상의 [Node.js](https://nodejs.org/en/)가 필요합니다. 본인의 컴퓨터에 설치되어 있는지 확인해주세요.

### 설치

[Nest CLI](https://docs.nestjs.com/cli/overview)를 사용하면 쉽게 새로운 프로젝트를 만들 수 있습니다. [npm](https://www.npmjs.com/)이 설치되어 있다면, 아래 명령어를 실행시켜서 대망의 새 Nest 프로젝트를 만들어보세요!

```shell
$ npm i -g @nestjs/cli
$ nest new project-name
```

`project-name` 디렉토리가 만들어지고, 필요한 노드 모듈들과 기본적인 파일들이 설치되며, `src/` 디렉토리에는 몇몇 핵심 파일들이 생성됩니다.

```
src
├─ app.controller.spec.ts
├─ app.controller.ts
├─ app.module.ts
├─ app.service.ts
└─ main.ts
```

위 핵심 파일들의 간단하게 설명하면 아래와 같습니다.

|파일명|설명|
|:---:|:---|
|`app.controller.ts`|라우트 하나만 있는 기본적인 컨트롤러가 있는 파일|
|`app.controller.spec.ts`|`app.controller.ts`의 유닛 테스트를 작성하는 파일|
|`app.module.ts`|어플리케이션의 루트 모듈이 있는 파일|
|`app.service.ts`|메서드 하나만 있는 기본적인 서비스가 있는 파일|
|`main.ts`|시작 파일(entry file). 핵심 함수인 `NestFactory`를 사용하여 Nest 어플리케이션 인스턴스를 만듭니다.|

> **역주** : 위에서 서술한 컨트롤러, 서비스 등은 지금은 이해 못하셔도 괜찮습니다. 이후에 나올 챕터들에서 자세한 설명이 나옵니다!

간단하게 `main.ts` 파일부터 보겠습니다. 해당 파일에는 어플리케이션을 시작해주는 비동기 함수(`bootstrap`)가 있습니다.

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

Nest 어플리케이션 인스턴스를 만들 땐 `NestFactory` 클래스를 사용합니다. 해당 클래스는 어플리케이션 인스턴스를 만들 때 사용하는 몇몇의 정적 메서드를 제공하는데, 그 중 위에서 쓰인 `create` 메서드는 `INestApplication` 인터페이스를 만족하는 어플리케이션 객체를 반환합니다.

위 예시의 코드를 통해 인바운드 HTTP 요청을 기다리는 리스너를 구동시킬 수 있습니다.

이처럼 Nest CLI를 통해 만들어진 프로젝트는 **각각의 모듈이 자신의 전용 디렉토리를 갖는** 구조로 만들어집니다.

### 플랫폼

Nest는 플랫폼에 구애 받지 않는 프레임워크를 목표로 합니다. 플랫폼에 독립적이면, 개발자들은 여러 프로젝트에 재사용할 수 있는 코드를 만들 수 있기 때문입니다.

Nest는 어댑터만 있다면 어느 노드 HTTP 프레임워크와도 같이 사용할 수 있습니다. 기본적으로는 [express](https://expressjs.com/)와 [fastify](https://www.fastify.io/), 두 HTTP 플랫폼을 기본적으로 지원합니다. 필요에 따라 알맞게 선택해서 사용하면 됩니다.

|이름|설명|
|:---:|:---|
|`platform-express`|[Express](https://expressjs.com/)는 유명하고 가벼운 노드 웹 프레임워크입니다. Nest에서는 기본적으로 `@nestjs/platform-express` 패키지를 사용합니다.|
|`platform-fastify`|[Fastify](https://www.fastify.io/)는 최대의 효율과 속도를 제공하는 것에 집중하여, 높은 성능과 낮은 오버헤드를 갖는 프레임워크입니다. Fastify를 사용하려면 [여기](https://docs.nestjs.com/techniques/performance)를 참고하세요.|

어느 플랫폼을 사용하던 자체 어플리케이션 인터페이스를 노출할 수 있으며, Express는 `NestExpressApplication`이고, Fastify는 `NestFastifyApplication`입니다.

아래 예시처럼 `NestFactory.create()` 메서드의 제네릭에 타입을 넣으면, `app` 객체는 특정 플랫폼에서만 사용할 수 있는 메서드를 갖게 됩니다. 만약 기본 플랫폼 API에 접근할 필요가 없는 경우, 굳이 타입을 지정할 필요는 없습니다.

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);
```

### 어플리케이션 실행하기

설치가 모두 완료되었다면, 아래 명령어를 통해 어플리케이션을 시작할 수 있습니다.

```sh
$ npm run start
```

위 명령어는 `src/main.ts` 파일에 정의되어 있는 포트로 HTTP 서버가 요청을 기다리게 합니다. 어플리케이션이 잘 켜졌다면, 브라우저를 열어서 `http://localhost:3000/`으로 가보세요. `Hello World!` 메세지를 볼 수 있을 거에요!

### 문서 기여자

- [러리](https://github.com/Coalery)
- [cpprhtn](https://github.com/cpprhtn)
