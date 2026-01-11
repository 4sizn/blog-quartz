---
title: "setState를 비동기 함수라 오해하는 이유: React의 상태 업데이트 메커니즘"
description: "React setState는 비동기 함수가 아니다. 배치 업데이트와 실행 컨텍스트를 이해하자"
tags:
  - React.js
  - javascript
  - Frontend
  - State-Management
aliases:
  - "React setState"
  - "배치 업데이트"
  - "React 상태 관리"
draft: false
lang: "ko"
enableToc: true
created: "2026-01-11"
updated: "2026-01-11"
---

## TL;DR

> **setState는 비동기 함수가 아니다. 다만 React가 성능 최적화를 위해 상태 업데이트를 배치(batch)로 처리할 뿐이다.**

## 흔한 오해

많은 개발자들이 setState를 "비동기 함수"라고 말합니다.

```javascript
class Counter extends React.Component {
  state = { count: 0 };

  handleClick = () => {
    this.setState({ count: this.state.count + 1 });
    console.log(this.state.count);  // 왜 0일까? "비동기"라서?
  };
}
```

위 코드에서 `console.log`가 업데이트된 값을 출력하지 않기 때문에, "setState가 비동기 함수"라고 생각하게 됩니다.

하지만 **이것은 정확한 표현이 아닙니다.**

## setState는 비동기 함수가 아니다

### 비동기 함수의 정의

진짜 비동기 함수는 이렇습니다:

```javascript
// Promise를 반환하는 비동기 함수
async function fetchData() {
  const response = await fetch('/api/data');
  return response.json();
}

// setTimeout/setInterval 등의 비동기 작업
setTimeout(() => {
  console.log('비동기 실행');
}, 1000);
```

### setState의 실제 특성

```javascript
// setState는 Promise를 반환하지 않는다
this.setState({ count: 1 });  // Promise가 아님!

// await을 사용할 수 없다
await this.setState({ count: 1 });  // ❌ 에러는 아니지만 의미 없음

// 콜백은 있지만 이것도 비동기의 증거는 아님
this.setState({ count: 1 }, () => {
  console.log('업데이트 완료');
});
```

## 진짜 이유: 배치 업데이트 (Batching)

### React의 성능 최적화 전략

React는 성능을 위해 **여러 setState 호출을 모아서 한 번에 처리**합니다.

```javascript
handleClick = () => {
  // React 이벤트 핸들러 내부
  this.setState({ count: this.state.count + 1 });
  this.setState({ count: this.state.count + 1 });
  this.setState({ count: this.state.count + 1 });

  // 3번 호출했지만, 리렌더링은 1번만 발생!
  // count는 1 증가 (3이 아님)
};
```

### 왜 배치 업데이트를 할까?

**성능 때문입니다.**

```javascript
// 배치 없이 즉시 적용한다면?
this.setState({ a: 1 });  // 리렌더링
this.setState({ b: 2 });  // 리렌더링
this.setState({ c: 3 });  // 리렌더링
// → 3번의 불필요한 리렌더링!

// 배치 업데이트
this.setState({ a: 1 });
this.setState({ b: 2 });
this.setState({ c: 3 });
// → 모두 모아서 1번만 리렌더링!
```

## 언제 배치 업데이트가 일어날까?

### React가 제어하는 컨텍스트 (Batching O)

```javascript
// 1. 이벤트 핸들러
handleClick = () => {
  this.setState({ count: 1 });  // 배치됨
  this.setState({ count: 2 });  // 배치됨
};

// 2. 라이프사이클 메서드
componentDidMount() {
  this.setState({ count: 1 });  // 배치됨
  this.setState({ count: 2 });  // 배치됨
}

// 3. useEffect (Hooks)
useEffect(() => {
  setCount(1);  // 배치됨
  setCount(2);  // 배치됨
}, []);
```

### React가 제어하지 못하는 컨텍스트 (Batching X) - React 17 이하

```javascript
// 1. setTimeout
handleClick = () => {
  setTimeout(() => {
    this.setState({ count: 1 });  // 즉시 적용!
    console.log(this.state.count);  // 1 출력
    this.setState({ count: 2 });  // 즉시 적용!
    console.log(this.state.count);  // 2 출력
  }, 0);
};

// 2. Promise
handleClick = () => {
  fetch('/api').then(() => {
    this.setState({ count: 1 });  // 즉시 적용!
    this.setState({ count: 2 });  // 즉시 적용!
  });
};

// 3. 네이티브 이벤트 리스너
componentDidMount() {
  document.addEventListener('click', () => {
    this.setState({ count: 1 });  // 즉시 적용!
    this.setState({ count: 2 });  // 즉시 적용!
  });
}
```

## React 18의 변화: Automatic Batching

### React 18부터는 모든 곳에서 배치!

```javascript
// React 18+
handleClick = () => {
  setTimeout(() => {
    setCount(c => c + 1);  // 배치됨!
    setCount(c => c + 1);  // 배치됨!
    // 두 업데이트가 함께 처리됨
  }, 1000);
};

fetch('/api').then(() => {
  setCount(c => c + 1);  // 배치됨!
  setFlag(f => !f);      // 배치됨!
  // 함께 처리됨
});
```

### 배치를 원하지 않는다면?

```javascript
import { flushSync } from 'react-dom';

function handleClick() {
  flushSync(() => {
    setCount(c => c + 1);
  });
  // 여기서 DOM이 이미 업데이트됨

  flushSync(() => {
    setFlag(f => !f);
  });
  // 별도로 즉시 업데이트됨
}
```

## 올바른 setState 사용법

### 1. 함수형 업데이트 사용

```javascript
// ❌ 잘못된 방법 - 이전 state 직접 참조
this.setState({ count: this.state.count + 1 });
this.setState({ count: this.state.count + 1 });
this.setState({ count: this.state.count + 1 });
// count: 1 (의도와 다름)

// ✅ 올바른 방법 - 함수형 업데이트
this.setState(prevState => ({ count: prevState.count + 1 }));
this.setState(prevState => ({ count: prevState.count + 1 }));
this.setState(prevState => ({ count: prevState.count + 1 }));
// count: 3 (의도대로)
```

### 2. 콜백으로 업데이트 후 작업

```javascript
// 업데이트 완료 후 작업이 필요하다면
this.setState(
  { count: this.state.count + 1 },
  () => {
    console.log('업데이트 완료:', this.state.count);
    // 여기서는 업데이트된 값을 사용 가능
  }
);
```

### 3. Hooks에서의 패턴

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    // ✅ 함수형 업데이트
    setCount(c => c + 1);
    setCount(c => c + 1);
    setCount(c => c + 1);
    // count: 3
  };

  // 업데이트 후 작업은 useEffect로
  useEffect(() => {
    console.log('count 변경됨:', count);
  }, [count]);
}
```

## 실전 예제

### 예제 1: 카운터

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    // ❌ 잘못된 방법
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // count: 1만 증가

    // ✅ 올바른 방법
    setCount(c => c + 1);
    setCount(c => c + 1);
    setCount(c => c + 1);
    // count: 3 증가
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+3</button>
    </div>
  );
}
```

### 예제 2: 폼 입력

```javascript
function Form() {
  const [formData, setFormData] = useState({ name: '', email: '' });

  const handleChange = (e) => {
    const { name, value } = e.target;

    // ✅ 함수형 업데이트로 안전하게
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  return (
    <form>
      <input name="name" onChange={handleChange} />
      <input name="email" onChange={handleChange} />
    </form>
  );
}
```

### 예제 3: API 호출 후 상태 업데이트

```javascript
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  const fetchUser = async () => {
    setLoading(true);

    try {
      const response = await fetch('/api/user');
      const data = await response.json();

      // React 18+: 자동으로 배치됨
      setUser(data);
      setLoading(false);
    } catch (error) {
      setLoading(false);
    }
  };

  return <div>{loading ? 'Loading...' : user?.name}</div>;
}
```

## 정리

### setState는 비동기 함수가 아니다

❌ **잘못된 이해**: "setState는 비동기 함수다"
✅ **올바른 이해**: "React가 성능 최적화를 위해 배치 처리한다"

### 핵심 포인트

1. **배치 업데이트**: 여러 setState를 모아서 한 번에 처리
2. **React 17 이하**: React 이벤트 핸들러 내에서만 배치
3. **React 18+**: 모든 곳에서 자동 배치 (Automatic Batching)
4. **함수형 업데이트**: 이전 state 기반 업데이트 시 필수
5. **콜백 사용**: 업데이트 완료 후 작업이 필요할 때

### 실천 가이드

```javascript
// ✅ 항상 함수형 업데이트 사용
setState(prev => ({ ...prev, newValue }));

// ✅ 업데이트 후 작업은 useEffect
useEffect(() => {
  // state가 변경된 후 실행
}, [state]);

// ✅ 즉시 DOM 업데이트가 필요하면 flushSync (React 18+)
import { flushSync } from 'react-dom';
flushSync(() => {
  setState(newValue);
});
```

---

## 참고 자료

- [setState is not async](https://velog.io/@jay/setStateisnotasync)
- React 공식 문서: State and Lifecycle
- React 18: Automatic Batching
