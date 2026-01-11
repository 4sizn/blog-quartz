---
title: "TypeScript Omit vs Exclude: 헷갈리는 두 유틸리티 타입의 차이"
description: "객체 프로퍼티를 제외하는 Omit과 유니온 타입 멤버를 제외하는 Exclude의 차이점과 활용법"
tags:
  - Typescript
  - Programming
  - Type-System
aliases:
  - "Omit Exclude 차이"
  - "TypeScript 유틸리티 타입"
draft: false
lang: "ko"
enableToc: true
created: "2026-01-11"
updated: "2026-01-11"
---

## TL;DR

> **타입을 정의할 때 객체의 특정 프로퍼티를 빼고 싶을 땐 `Omit`, 유니온 타입의 특정 값을 빼고 싶을 땐 `Exclude`**

## 들어가며

TypeScript를 사용하다 보면 기존 타입에서 일부를 제외한 새로운 타입을 만들어야 할 때가 많습니다. 이때 자주 사용하는 두 가지 유틸리티 타입이 바로 `Omit`과 `Exclude`입니다.

이 둘은 이름도 비슷하고 역할도 비슷해 보이지만, **사용 대상**이 완전히 다릅니다.

## Omit: 객체 타입에서 프로퍼티 제외하기

### 기본 사용법

```typescript
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

// password를 제외한 나머지 프로퍼티만 가진 타입
type PublicUser = Omit<User, 'password'>;
// { id: number; name: string; email: string; }
```

### 문법

```typescript
Omit<T, K>
```

- `T`: 원본 객체 타입
- `K`: 제외할 프로퍼티 키(문자열 리터럴 타입)

### 여러 프로퍼티 제외하기

```typescript
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
};

// 여러 프로퍼티를 유니온으로 제외
type UserDTO = Omit<User, 'password' | 'createdAt' | 'updatedAt'>;
// { id: number; name: string; email: string; }
```

### 실전 예시

**API 응답과 요청 타입 분리**:
```typescript
interface Post {
  id: number;
  title: string;
  content: string;
  authorId: number;
  createdAt: Date;
  updatedAt: Date;
}

// POST 요청시 사용 (id, createdAt, updatedAt는 서버에서 생성)
type CreatePostDTO = Omit<Post, 'id' | 'createdAt' | 'updatedAt'>;
// { title: string; content: string; authorId: number; }

// PUT 요청시 사용 (id는 URL 파라미터로, 날짜는 서버에서 생성)
type UpdatePostDTO = Omit<Post, 'id' | 'authorId' | 'createdAt' | 'updatedAt'>;
// { title: string; content: string; }
```

## Exclude: 유니온 타입에서 멤버 제외하기

### 기본 사용법

```typescript
type Status = 'pending' | 'approved' | 'rejected' | 'cancelled';

// 'cancelled'를 제외한 나머지 상태들
type ActiveStatus = Exclude<Status, 'cancelled'>;
// 'pending' | 'approved' | 'rejected'
```

### 문법

```typescript
Exclude<T, U>
```

- `T`: 원본 유니온 타입
- `U`: 제외할 타입

### 여러 멤버 제외하기

```typescript
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH' | 'OPTIONS' | 'HEAD';

// 여러 멤버를 유니온으로 제외
type CommonHttpMethod = Exclude<HttpMethod, 'OPTIONS' | 'HEAD'>;
// 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH'
```

### 실전 예시

**조건부 타입과 함께 사용**:
```typescript
type Primitive = string | number | boolean | null | undefined;

// 객체와 함수만 남기기 (primitive 타입 제외)
type NonPrimitive<T> = Exclude<T, Primitive>;

type Example = NonPrimitive<string | number | object | Function>;
// object | Function
```

**이벤트 핸들러 타입**:
```typescript
type MouseEvent = 'click' | 'dblclick' | 'mousedown' | 'mouseup' | 'mousemove';
type KeyboardEvent = 'keydown' | 'keyup' | 'keypress';

type AllEvents = MouseEvent | KeyboardEvent;

// 마우스 이벤트만 제외
type OnlyKeyboardEvents = Exclude<AllEvents, MouseEvent>;
// 'keydown' | 'keyup' | 'keypress'
```

## 핵심 차이점

| 특징 | Omit | Exclude |
|---|---|---|
| **대상** | 객체 타입 | 유니온 타입 |
| **제외하는 것** | 객체의 프로퍼티 (키) | 유니온의 멤버 (타입) |
| **결과** | 새로운 객체 타입 | 새로운 유니온 타입 |
| **사용 상황** | 객체에서 일부 필드 제거 | 유니온에서 일부 옵션 제거 |

## 실전 조합 예시

### Omit과 Exclude를 함께 사용

```typescript
interface User {
  id: number;
  name: string;
  role: 'admin' | 'user' | 'guest';
  email: string;
}

// 1. Omit으로 프로퍼티 제외
type UserWithoutId = Omit<User, 'id'>;

// 2. Omit + 타입 재정의로 role 옵션 제한
type LimitedUser = Omit<User, 'role'> & {
  role: Exclude<User['role'], 'admin'>;  // 'admin'을 제외한 role
};
// { id: number; name: string; email: string; role: 'user' | 'guest'; }
```

### 실무 패턴: API 타입 관리

```typescript
// 기본 엔티티
interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  category: 'electronics' | 'clothing' | 'food' | 'books';
  status: 'active' | 'inactive' | 'deleted';
  createdAt: Date;
  updatedAt: Date;
}

// CREATE: id, 날짜 필드 제외
type CreateProductDTO = Omit<Product, 'id' | 'createdAt' | 'updatedAt'>;

// UPDATE: id, 날짜 필드 제외 + 모든 필드 optional
type UpdateProductDTO = Partial<Omit<Product, 'id' | 'createdAt' | 'updatedAt'>>;

// LIST: deleted 상태 제외
type ListProduct = Omit<Product, 'status'> & {
  status: Exclude<Product['status'], 'deleted'>;
};
```

## 내부 구현 이해하기

### Omit의 내부 구현

```typescript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

흥미롭게도 `Omit`은 내부적으로 `Exclude`를 사용합니다!
1. `keyof T`로 모든 키를 유니온 타입으로 만듦
2. `Exclude`로 제외할 키를 빼냄
3. `Pick`으로 남은 키들만 추출

### Exclude의 내부 구현

```typescript
type Exclude<T, U> = T extends U ? never : T;
```

조건부 타입을 사용하여 각 멤버를 검사하고, `U`에 속하면 `never`, 아니면 그대로 유지합니다.

## 헷갈리지 않는 팁

### 1. 명확한 기준

- **객체의 모양**을 다룬다 → `Omit`
- **가능한 값들의 집합**을 다룬다 → `Exclude`

### 2. 시각적 구분

```typescript
// Omit: 중괄호 { } 안에서
type Result1 = Omit<{ a: number; b: string }, 'b'>;

// Exclude: 파이프 | 사이에서
type Result2 = Exclude<'a' | 'b' | 'c', 'b'>;
```

### 3. 에러 메시지로 확인

잘못 사용하면 TypeScript가 알려줍니다:

```typescript
type Wrong1 = Omit<'a' | 'b', 'a'>;
// ❌ Omit은 객체 타입에만 사용

type Wrong2 = Exclude<{ a: number }, 'a'>;
// ⚠️ 의도한 대로 작동하지 않음
```

## 결론

- **Omit**: 객체 타입에서 **프로퍼티 제외**
- **Exclude**: 유니온 타입에서 **멤버 제외**

이 둘의 차이를 명확히 이해하면 TypeScript로 더 정교한 타입 시스템을 구축할 수 있습니다.

### 체크리스트

제외하고 싶은 대상이:
- ☐ 객체의 특정 필드인가? → `Omit`
- ☐ 유니온의 특정 옵션인가? → `Exclude`

---

## 참고 자료

- TypeScript 공식 문서: [Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
- 원문: https://iamshadmirza.com/difference-between-omit-and-exclude-in-typescript
