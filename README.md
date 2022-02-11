---
description: "원문 : https://docs.nestjs.com/"
---

# INTRODUCTION

### 소개

Nest(NestJS)는 효율적이고 확장 가능한 [Node.js](https://nodejs.org/en/) 서버 측 애플리케이션을 구축하기 위한 프레임워크입니다. 프로그레시브 자바스크립트를 사용하고, [타입스크립트](https://www.typescriptlang.org/)로 만들어졌으며 이를 완전히 지원합니다. (아직 순수 자바스크립트로 개발할 수는 있습니다.) Nest는 OOP(Object Oriented Programming), FP(Functional Programming), FRP(Functional Reactive Programming)의 요소들이 섞여있는 프레임워크입니다.

Nest는 기본적으로 강력한 HTTP 서버 프레임워크인 [Express](https://expressjs.com/)를 사용하며, [Fastify](https://github.com/fastify/fastify)를 사용하도록 구성할 수도 있습니다.

Nest는 위 두 프레임워크에 대한 추상화 층을 제공하지만, 이들의 API들도 직접적으로 개발자에게 노출시킵니다. 이는 개발자가 자유롭게 많은 모듈을 사용할 수 있게 합니다.

### 철학

최근 몇 년간, 자바스크립트는 Node.js 덕분에 프론트엔드와 백엔드 어플리케이션의 공통어 역할(원문: [lingua franca](https://en.wikipedia.org/wiki/Lingua_franca))을 하게 되었습니다. 이는 [Angular](https://angular.io/), [React](https://github.com/facebook/react), [Vue](https://github.com/vuejs/vue)와 같이 멋진 프론트엔드 프로젝트가 떠오르게 하였습니다. 프론트엔드 개발자들은 이러한 프로젝트들로 인해 생산성이 향상되었고, 빠르고 테스트 가능하며 확장 가능한 프론트엔드 어플리케이션을 만들어낼 수 있었습니다. 이렇게 많은 멋진 라이브러리들, 사람들, 도구들이 나왔지만, 아무도 아키텍처의 주요 문제를 효율적으로 풀지는 못했습니다.

Nest는 개발자들과 팀들이 높은 테스트 가능성, 확장 가능성을 가지며 느슨하게 결합되고 쉽게 유지 보수가 가능한 어플리케이션을 만들 수 있는 아키텍처를 제공합니다. 이 아키텍처는 Angular에서 크게 영감을 받았습니다.

### 설치

[Nest CLI](https://docs.nestjs.com/cli/overview)를 통해 프로젝트의 기반을 구축할 수도 있고, 스타터 프로젝트를 클론해올 수도 있습니다. 둘 모두 같은 결과를 만들어내니, 편한 방법을 사용하시면 됩니다.

Nest CLI를 통해 프로젝트를 시작하려면 아래 명령어를 실행하세요. 새로운 프로젝트 디렉토리가 생성되며, Nest 필수 파일들과 모듈들이 생성된 디렉토리에 채워져, 기본적인 구조를 가진 프로젝트가 만들어집니다. 처음 해보신 분들은 Nest CLI를 통해 프로젝트를 만드는 것을 추천합니다. 이는 [First Steps](/overview/first-steps)에서 계속됩니다.

```sh
$ npm i -g @nestjs/cli
$ nest new project-name
```

### Git으로 설치하기

깃을 통해 타입스크립트 스타터 프로젝트를 설치할 수 있습니다.

```sh
$ git clone https://github.com/nestjs/typescript-starter.git project
$ cd project
$ npm install
$ npm run start
```

> **팁**
> 깃 히스토리 없이 리포지토리를 클론하고 싶다면, [degit](https://github.com/Rich-Harris/degit)을 사용해보세요.

브라우저를 열고, [http://localhost:3000/](http://localhost:3000/)로 가보세요.

자바스크립트로 된 스타터 프로젝트를 설치하고 싶다면, 위 명령어에서 클론 부분을 아래와 같이 바꿔주시면 됩니다.

```sh
$ git clone https://github.com/nestjs/typescript-starter.git project
```

npm이나 yarn을 통해 주요 파일들을 설치하여 바닥부터 수동으로 프로젝트를 만들 수도 있습니다. 물론, 이 경우엔 프로젝트의 기본 파일들을 직접 생성하셔야 합니다.

```sh
$ npm i --save @nestjs/core @nestjs/common rxjs reflect-metadata
```