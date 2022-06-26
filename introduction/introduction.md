# Introduction

원문 : https://docs.nestjs.com

### 소개

Nest는 [Node.js](https://nodejs.org/en/)를 통해 효율적이고 확장 가능한 백엔드 어플리케이션을 구축하기 위해 만들어진 프레임워크입니다. 프로그레시브 자바스크립트를 사용하고, [타입스크립트](https://www.typescriptlang.org/)로 만들어졌으며 둘 다 완전히 지원합니다. 아직 순수 자바스크립트로 개발할 수 있긴 합니다.

Nest는 객체 지향 프로그래밍(OOP), 함수형 프로그래밍(FP), 함수 반응형 프로그래밍(FRP)의 요소들이 섞여있는 프레임워크입니다.

Nest는 강력한 HTTP 서버 프레임워크인 [Express](https://expressjs.com/)를 기본적으로 사용하지만, [Fastify](https://github.com/fastify/fastify)를 사용하도록 설정할 수도 있습니다.

Nest는 위의 두 프레임워크에 대한 추상화 계층을 제공합니다. 하지만 각 프레임워크의 API들도 개발자에게 직접적으로 노출시키기 때문에, 개발자는 많은 모듈을 자유롭게 사용할 수 있습니다.

### 철학

최근 몇 년간, 자바스크립트는 Node.js 덕분에 프론트엔드와 백엔드의 공통어\* 역할을 해왔습니다. 프론트엔드에서는 자바스크립트를 이용해서 [Angular](https://angular.io/), [React](https://github.com/facebook/react), [Vue](https://github.com/vuejs/vue) 등 멋진 프로젝트들이 만들어졌고, 프론트엔드 개발자들은 이 프로젝트를 활용해서 개발을 함으로써 생산성이 향상되었고, 빠르고 테스트 가능하며 확장 가능한 프론트엔드 어플리케이션을 만들어낼 수 있었습니다.

이렇게 많은 라이브러리들, 사람들, 도구들이 나왔습니다만, 아무도 아키텍처의 주요 문제를 효율적으로 풀지는 못했습니다.

Nest는 Angular에서 영감을 받아, 개발자들이 높은 테스트 가능성, 확장 가능성을 가지며 느슨하게 결합되어 쉽게 유지 보수할 수 있는 아키텍처를 제공합니다.

\* 원문: [lingua franca](https://en.wikipedia.org/wiki/Lingua_franca)

### 설치

[Nest CLI](https://docs.nestjs.com/cli/overview)를 통해 프로젝트의 기반을 구축할 수도 있고, 스타터 프로젝트를 클론해올 수도 있습니다. 두 방법은 똑같은 결과를 만들어내니, 편한 방법을 사용하시면 됩니다.

Nest CLI를 통해 기본 프로젝트를 만들려면 아래 명령어를 실행하면 됩니다.

```sh
$ npm i -g @nestjs/cli
$ nest new project-name
```

명령어를 실행하면 새로운 프로젝트 디렉토리가 생성되며 해당 디렉토리에 필수 파일들과 모듈들이 만들어집니다. 처음 해보신 분들은 Nest CLI를 통해 프로젝트를 만드는 것을 추천드립니다. 이 과정은 [First Steps](/overview/first-steps) 챕터에서 계속됩니다!

### Git으로 설치하기

깃을 통해 타입스크립트 스타터 프로젝트를 설치할 수 있습니다.

```sh
$ git clone https://github.com/nestjs/typescript-starter.git project
$ cd project
$ npm install
$ npm run start
```

> **팁**
> 
> 깃 히스토리 없이 리포지토리를 클론하고 싶다면, [degit](https://github.com/Rich-Harris/degit)을 사용해보세요!

브라우저를 열어서 [http://localhost:3000/](http://localhost:3000/)로 들어가보세요.

자바스크립트로 된 스타터 프로젝트를 설치하고 싶다면, 위 명령어의 `typescript-starter` 부분을 `javascript-starter`로 바꿔주면 됩니다. 아래 명령어 같이 말이죠.

```sh
$ git clone https://github.com/nestjs/javascript-starter.git project
```

npm이나 yarn을 통해 바닥부터 수동으로 주요 파일들을 설치하여 프로젝트를 만들 수도 있습니다. 물론, 이 경우엔 기본 파일들을 직접 생성하셔야 합니다.

```sh
$ npm i --save @nestjs/core @nestjs/common rxjs reflect-metadata
```

### 문서 기여자

- [러리](https://github.com/Coalery)
- [cpprhtn](https://github.com/cpprhtn)
