---
title: "TypeScript에서 'd.ts' 파일이란 무엇입니까: 타입 선언 파일의 모든 것"
description: "JavaScript 라이브러리에 TypeScript 타입을 추가하는 선언 파일(.d.ts)의 역할과 사용법"
tags:
  - Typescript
  - JavaScript
  - Type-System
  - Library
aliases:
  - "Declaration Files"
  - "타입 선언 파일"
  - ".d.ts 파일"
draft: false
lang: "ko"
enableToc: true
created: "2026-01-11"
updated: "2026-01-11"
---

## TL;DR

> **.d.ts 파일은 코드의 타입 추론을 돕는 타입 선언 파일(Type Declaration File)입니다.**
>
> 해당 파일을 통해 기존 JavaScript 모듈의 모양을 설명하고, TypeScript에서 타입을 검사할 수 있게 합니다.

## 들어가며

TypeScript 프로젝트를 작업하다 보면 `.d.ts` 확장자를 가진 파일들을 자주 보게 됩니다.

```
node_modules/
  @types/
    react/
      index.d.ts
      global.d.ts
    node/
      index.d.ts
  lodash/
    lodash.d.ts
```

이 파일들은 무엇이고, 왜 필요할까요?

## .d.ts 파일이란?

### Declaration File (선언 파일)

`.d.ts` 파일은 **Declaration File**의 약자로, TypeScript의 **타입 정보만을 담고 있는 파일**입니다.

```typescript
// math.d.ts - 선언 파일
export function add(a: number, b: number): number;
export function subtract(a: number, b: number): number;

// 실제 구현 코드는 없고, 타입 정보만 있음
```

### 실제 구현과의 관계

```javascript
// math.js - 실제 JavaScript 구현
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}
```

```typescript
// math.d.ts - 타입 선언
export function add(a: number, b: number): number;
export function subtract(a: number, b: number): number;
```

TypeScript 컴파일러는 `.js` 파일의 구현과 `.d.ts` 파일의 선언을 매칭하여 타입 체크를 수행합니다.

## 왜 필요한가?

### 1. JavaScript 라이브러리에 타입 추가

JavaScript로 작성된 라이브러리를 TypeScript에서 사용할 때 필요합니다.

```typescript
// lodash는 JavaScript로 작성됨
import _ from 'lodash';

// TypeScript에서 타입 체크를 받으려면?
_.chunk(['a', 'b', 'c', 'd'], 2);
//  ^ 어떤 파라미터를 받는지 알 수 있을까?
```

**해결책: @types/lodash 패키지**

```typescript
// @types/lodash/index.d.ts
declare module 'lodash' {
  export function chunk<T>(array: T[], size: number): T[][];
  // ... 수백 개의 함수 선언
}
```

이제 TypeScript가 타입을 알 수 있습니다!

### 2. 타입 체크와 자동완성

```typescript
import _ from 'lodash';

// 자동완성 제공
_.ch  // chunk, chain 등이 자동완성됨

// 타입 체크
_.chunk(['a', 'b', 'c'], '2');
//                       ^^^ ❌ Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

### 3. API 문서 역할

```typescript
// user.d.ts
interface User {
  /** 사용자 고유 ID */
  id: number;

  /** 사용자 이름 (2-50자) */
  name: string;

  /** 이메일 주소 */
  email: string;
}

/**
 * 사용자 정보를 가져옵니다
 * @param id 사용자 ID
 * @returns 사용자 객체 또는 null
 */
export function getUser(id: number): Promise<User | null>;
```

## 선언 파일의 위치

### 1. DefinitelyTyped (@types)

대부분의 유명 JavaScript 라이브러리는 `@types` 패키지로 타입 정의가 제공됩니다.

```bash
# React 타입 설치
npm install --save-dev @types/react

# Node.js 타입 설치
npm install --save-dev @types/node

# Express 타입 설치
npm install --save-dev @types/express
```

### 2. 라이브러리 자체에 포함

일부 라이브러리는 자체적으로 `.d.ts` 파일을 포함합니다.

```json
// package.json
{
  "name": "my-library",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts"  // 타입 선언 파일 위치
}
```

### 3. 프로젝트 내부 선언

```
src/
  types/
    global.d.ts
    api.d.ts
    models.d.ts
```

## Type Definition이 없는 모듈 다루기

### 문제 상황

```typescript
// untyped-library는 타입 정의가 없는 라이브러리
import something from 'untyped-library';
//                    ^^^^^^^^^^^^^^^^^^
// ❌ Could not find a declaration file for module 'untyped-library'
```

### 해결 방법 1: 모듈 선언으로 침묵시키기

```typescript
// src/types/untyped-modules.d.ts
declare module 'untyped-library';
```

이제 에러는 사라지지만, 타입은 `any`가 됩니다.

```typescript
import something from 'untyped-library';
// something의 타입은 any
```

### 해결 방법 2: 직접 타입 정의하기

```typescript
// src/types/untyped-library.d.ts
declare module 'untyped-library' {
  export interface Config {
    apiKey: string;
    timeout?: number;
  }

  export function initialize(config: Config): void;
  export function getData(): Promise<any>;

  const library: {
    version: string;
    initialize: typeof initialize;
    getData: typeof getData;
  };

  export default library;
}
```

이제 완전한 타입 체크가 가능합니다!

```typescript
import library, { initialize, getData } from 'untyped-library';

initialize({ apiKey: 'abc123' });  // ✅
initialize({ apiKey: 123 });       // ❌ Error: Type 'number' is not assignable to type 'string'
```

## 선언 파일 작성 패턴

### 함수 선언

```typescript
// 기본 함수
declare function greet(name: string): string;

// 오버로딩
declare function format(value: string): string;
declare function format(value: number): string;
declare function format(value: Date): string;

// 제네릭
declare function identity<T>(value: T): T;
```

### 클래스 선언

```typescript
declare class EventEmitter {
  constructor();

  on(event: string, listener: Function): this;
  off(event: string, listener: Function): this;
  emit(event: string, ...args: any[]): boolean;
}
```

### 인터페이스와 타입

```typescript
// 인터페이스
interface User {
  id: number;
  name: string;
  email: string;
}

// 타입 별칭
type ID = string | number;

// 유니온 타입
type Status = 'pending' | 'success' | 'error';
```

### 네임스페이스

```typescript
declare namespace MyLibrary {
  interface Config {
    debug: boolean;
  }

  function init(config: Config): void;

  namespace Utils {
    function formatDate(date: Date): string;
  }
}

// 사용
MyLibrary.init({ debug: true });
MyLibrary.Utils.formatDate(new Date());
```

### 모듈 확장 (Module Augmentation)

기존 모듈에 타입을 추가할 수 있습니다.

```typescript
// express.d.ts
import 'express';

declare module 'express' {
  interface Request {
    user?: {
      id: number;
      name: string;
    };
  }
}

// 이제 req.user를 사용할 수 있음
app.get('/profile', (req, res) => {
  console.log(req.user?.name);  // 타입 체크 O
});
```

### 전역 선언

```typescript
// global.d.ts
declare global {
  interface Window {
    myApp: {
      version: string;
      init(): void;
    };
  }

  const API_URL: string;
}

export {};  // 모듈로 만들기 위한 빈 export

// 사용
window.myApp.init();
console.log(API_URL);
```

## 실전 예제

### 예제 1: 외부 CDN 라이브러리

```html
<!-- index.html -->
<script src="https://cdn.example.com/awesome-library.js"></script>
```

```typescript
// awesome-library.d.ts
declare namespace AwesomeLibrary {
  interface Options {
    theme: 'light' | 'dark';
    language: string;
  }

  function init(options: Options): void;
  function destroy(): void;
}

// 사용
AwesomeLibrary.init({ theme: 'dark', language: 'ko' });
```

### 예제 2: Node.js 환경 변수

```typescript
// env.d.ts
declare namespace NodeJS {
  interface ProcessEnv {
    NODE_ENV: 'development' | 'production' | 'test';
    API_URL: string;
    API_KEY: string;
    PORT: string;
  }
}

// 사용
const apiUrl = process.env.API_URL;  // string 타입
const port = parseInt(process.env.PORT);
```

### 예제 3: 이미지 import

```typescript
// images.d.ts
declare module '*.png' {
  const value: string;
  export default value;
}

declare module '*.jpg' {
  const value: string;
  export default value;
}

declare module '*.svg' {
  import React from 'react';
  const SVG: React.FC<React.SVGProps<SVGSVGElement>>;
  export default SVG;
}

// 사용
import logo from './logo.png';  // string
import Icon from './icon.svg';  // React Component
```

### 예제 4: CSS Modules

```typescript
// css-modules.d.ts
declare module '*.module.css' {
  const classes: { [key: string]: string };
  export default classes;
}

declare module '*.module.scss' {
  const classes: { [key: string]: string };
  export default classes;
}

// 사용
import styles from './Button.module.css';

<button className={styles.primary}>Click</button>
```

## .d.ts 파일 자동 생성

### tsconfig.json 설정

```json
{
  "compilerOptions": {
    "declaration": true,           // .d.ts 파일 생성
    "declarationMap": true,        // .d.ts.map 파일 생성 (소스맵)
    "outDir": "./dist",
    "rootDir": "./src"
  }
}
```

### 소스 코드

```typescript
// src/math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}
```

### 자동 생성된 선언 파일

```typescript
// dist/math.d.ts
export declare function add(a: number, b: number): number;
export declare function subtract(a: number, b: number): number;
```

## 주의사항

### 1. 구현 코드 포함 불가

```typescript
// ❌ 잘못된 예
export function greet(name: string): string {
  return `Hello, ${name}`;  // 구현 코드는 .d.ts에 들어가면 안 됨
}

// ✅ 올바른 예
export declare function greet(name: string): string;
```

### 2. 타입만 export

```typescript
// types.d.ts
export interface User {
  id: number;
  name: string;
}

export type Status = 'active' | 'inactive';

// 값은 export 불가
export const MAX_USERS = 100;  // ❌ Error
```

### 3. declare 키워드 사용

```typescript
// 모듈 외부에서 사용 가능하도록
declare const API_URL: string;
declare function fetchData(): Promise<any>;
declare class HttpClient {}
```

## 결론

### 핵심 정리

1. **.d.ts 파일은 타입 정보만 담고 있는 선언 파일**
2. **JavaScript 라이브러리에 TypeScript 타입을 추가**
3. **@types 패키지로 대부분의 유명 라이브러리 커버**
4. **타입이 없는 모듈은 직접 선언 가능**
5. **tsconfig.json으로 자동 생성 가능**

### 실천 가이드

- npm 패키지 만들 때: `declaration: true` 설정으로 .d.ts 자동 생성
- 타입 없는 라이브러리 사용 시: 프로젝트 내 선언 파일 작성
- 전역 타입 추가 시: `global.d.ts` 파일 활용
- 모듈 확장 시: declare module로 기존 타입 확장

TypeScript의 타입 시스템을 최대한 활용하려면 선언 파일을 잘 이해하고 활용하는 것이 중요합니다!

---

## 참고 자료

- [What is a d.ts file in TypeScript?](https://medium.com/@ohansemmanuel/what-is-a-d-ts-file-in-typescript-2e2d90d58eca)
- TypeScript 공식 문서: [Declaration Files](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html)
- [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) - 커뮤니티 타입 정의 저장소
