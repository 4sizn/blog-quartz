---
title: "BroadcastChannel API: 브라우저 탭 간 실시간 통신"
description: "WebSocket 없이 브라우저 창/탭 간 메시지를 주고받는 BroadcastChannel API 활용법"
tags:
  - javascript
  - Web-API
  - Browser
  - RealTime
aliases:
  - "브라우저 탭 통신"
  - "BroadcastChannel"
  - "Web API"
draft: false
lang: "ko"
enableToc: true
created: "2026-01-11"
updated: "2026-01-11"
---

## BroadcastChannel이란?

**BroadcastChannel**은 브라우저 창, 탭 간 메시지 통신을 할 수 있는 Web API입니다.

### 핵심 특징

**동일 출처 정책 (Same-Origin Policy)**:
- 본 기능을 이용해 메시지 통신을 하기 위해서는
- Browsing Context 간 **동일한 출처**여야 한다는 엄격한 규약

**동일한 출처 조건**:
- 콘텐츠의 **스킴(Scheme)**이 일치
- **도메인(Domain)**이 일치
- **포트(Port)**가 일치

```
✅ 동일 출처
https://example.com:443/page1
https://example.com:443/page2

❌ 다른 출처
https://example.com:443      (HTTPS)
http://example.com:80        (HTTP - 스킴 다름)

https://example.com          (example.com)
https://sub.example.com      (서브도메인 - 도메인 다름)

https://example.com:443      (포트 443)
https://example.com:8080     (포트 8080 - 포트 다름)
```

## 왜 BroadcastChannel을 사용하는가?

### 기존 방법의 한계

**별도의 서버 통신 필요**:
```
탭 1 ←→ WebSocket Server ←→ 탭 2
탭 1 ←→ MQTT Broker ←→ 탭 2
탭 1 ←→ Firebase ←→ 탭 2
```

**문제점**:
- 서버 인프라 필요
- 네트워크 비용
- 지연 시간 (Latency)
- 추가 복잡성

### BroadcastChannel의 장점

**서버 없이 직접 통신**:
```
탭 1 ←→ BroadcastChannel ←→ 탭 2
      (브라우저 내부)
```

**이점**:
- ✅ 서버 불필요
- ✅ 무료
- ✅ 빠른 속도
- ✅ 간단한 구현
- ✅ 동일 기기 내 완벽한 동기화

### 사용 사례

**BroadcastChannel**을 이용한다면, **동일한 서비스의 [창/탭] 간에 동일한 데이터(Data Sync)를 보여줘야 한다면** 해결할 수 있습니다.

**실제 활용 예시**:
1. **사용자 로그인/로그아웃 동기화**
   ```
   탭 1에서 로그아웃
     → 모든 탭에서 자동 로그아웃
   ```

2. **장바구니 동기화**
   ```
   탭 1에서 상품 추가
     → 탭 2의 장바구니에도 즉시 반영
   ```

3. **알림 읽음 상태 동기화**
   ```
   탭 1에서 알림 읽음
     → 탭 2의 알림 배지 숫자 감소
   ```

4. **다크 모드 설정 동기화**
   ```
   탭 1에서 다크 모드 켜기
     → 모든 탭이 다크 모드로 전환
   ```

## 브라우저 지원 현황 (Spec)

### Can I Use

[BroadcastChannel 브라우저 지원](https://caniuse.com/broadcastchannel)

**주요 브라우저 지원**:
- ✅ Chrome 54+
- ✅ Firefox 38+
- ✅ Edge 79+
- ✅ Safari 15.4+
- ✅ Opera 41+

**미지원**:
- ❌ Internet Explorer (전체)
- ❌ Safari 15.3 이하

**모바일 지원**:
- ✅ Chrome Android
- ✅ Firefox Android
- ✅ Safari iOS 15.4+

### 폴리필 고려

구형 브라우저 지원이 필요하다면:
- LocalStorage + Storage Event 사용
- SharedWorker 활용
- 폴리필 라이브러리 사용

## 기본 사용법

### 1. 채널 생성

```javascript
// 같은 이름의 채널은 모든 탭에서 공유됩니다
const channel = new BroadcastChannel('my-channel');
```

### 2. 메시지 보내기

```javascript
// 메시지 전송 (현재 탭 제외한 모든 탭에 전달)
channel.postMessage('Hello from Tab 1!');

// 객체도 전송 가능
channel.postMessage({
  type: 'UPDATE_USER',
  payload: {
    userId: 123,
    name: 'John Doe'
  }
});

// 배열도 전송 가능
channel.postMessage(['item1', 'item2', 'item3']);
```

### 3. 메시지 받기

```javascript
// 메시지 수신
channel.onmessage = (event) => {
  console.log('받은 메시지:', event.data);

  // 메시지 타입에 따라 처리
  if (event.data.type === 'UPDATE_USER') {
    updateUserUI(event.data.payload);
  }
};

// 또는 addEventListener 사용
channel.addEventListener('message', (event) => {
  console.log('받은 메시지:', event.data);
});
```

### 4. 채널 닫기

```javascript
// 더 이상 사용하지 않을 때 채널 닫기
channel.close();
```

## 실전 예제

### 예제 1: 로그인/로그아웃 동기화

```javascript
// auth-sync.js
const authChannel = new BroadcastChannel('auth-channel');

// 로그인 처리
function login(user) {
  // 로컬 상태 업데이트
  localStorage.setItem('user', JSON.stringify(user));

  // 다른 탭에 알림
  authChannel.postMessage({
    type: 'LOGIN',
    payload: user
  });

  // UI 업데이트
  updateAuthUI(user);
}

// 로그아웃 처리
function logout() {
  // 로컬 상태 제거
  localStorage.removeItem('user');

  // 다른 탭에 알림
  authChannel.postMessage({
    type: 'LOGOUT'
  });

  // UI 업데이트
  updateAuthUI(null);
}

// 메시지 수신
authChannel.onmessage = (event) => {
  const { type, payload } = event.data;

  switch (type) {
    case 'LOGIN':
      localStorage.setItem('user', JSON.stringify(payload));
      updateAuthUI(payload);
      break;

    case 'LOGOUT':
      localStorage.removeItem('user');
      updateAuthUI(null);
      // 로그인 페이지로 리다이렉트
      window.location.href = '/login';
      break;
  }
};
```

### 예제 2: 장바구니 동기화

```javascript
// cart-sync.js
const cartChannel = new BroadcastChannel('cart-channel');

// 장바구니에 상품 추가
function addToCart(product) {
  // 현재 장바구니 가져오기
  const cart = getCart();

  // 상품 추가
  cart.push(product);

  // localStorage에 저장
  localStorage.setItem('cart', JSON.stringify(cart));

  // 다른 탭에 알림
  cartChannel.postMessage({
    type: 'ADD_ITEM',
    payload: product
  });

  // UI 업데이트
  updateCartUI(cart);
}

// 장바구니에서 상품 제거
function removeFromCart(productId) {
  const cart = getCart();
  const updatedCart = cart.filter(item => item.id !== productId);

  localStorage.setItem('cart', JSON.stringify(updatedCart));

  cartChannel.postMessage({
    type: 'REMOVE_ITEM',
    payload: productId
  });

  updateCartUI(updatedCart);
}

// 메시지 수신
cartChannel.onmessage = (event) => {
  const { type, payload } = event.data;

  switch (type) {
    case 'ADD_ITEM':
      const cart = getCart();
      cart.push(payload);
      localStorage.setItem('cart', JSON.stringify(cart));
      updateCartUI(cart);
      updateCartBadge(cart.length);
      break;

    case 'REMOVE_ITEM':
      const updatedCart = getCart().filter(item => item.id !== payload);
      localStorage.setItem('cart', JSON.stringify(updatedCart));
      updateCartUI(updatedCart);
      updateCartBadge(updatedCart.length);
      break;

    case 'CLEAR_CART':
      localStorage.removeItem('cart');
      updateCartUI([]);
      updateCartBadge(0);
      break;
  }
};

function getCart() {
  const cart = localStorage.getItem('cart');
  return cart ? JSON.parse(cart) : [];
}
```

### 예제 3: React에서 사용하기

```typescript
// useBroadcastChannel.ts
import { useEffect, useRef, useState } from 'react';

interface BroadcastMessage<T = any> {
  type: string;
  payload?: T;
}

export function useBroadcastChannel<T = any>(
  channelName: string,
  onMessage?: (message: BroadcastMessage<T>) => void
) {
  const channelRef = useRef<BroadcastChannel | null>(null);
  const [lastMessage, setLastMessage] = useState<BroadcastMessage<T> | null>(null);

  useEffect(() => {
    // BroadcastChannel 지원 확인
    if (typeof BroadcastChannel === 'undefined') {
      console.warn('BroadcastChannel is not supported');
      return;
    }

    // 채널 생성
    const channel = new BroadcastChannel(channelName);
    channelRef.current = channel;

    // 메시지 핸들러
    channel.onmessage = (event) => {
      const message = event.data as BroadcastMessage<T>;
      setLastMessage(message);
      onMessage?.(message);
    };

    // 클린업
    return () => {
      channel.close();
      channelRef.current = null;
    };
  }, [channelName, onMessage]);

  // 메시지 전송 함수
  const postMessage = (type: string, payload?: T) => {
    if (channelRef.current) {
      channelRef.current.postMessage({ type, payload });
    }
  };

  return { postMessage, lastMessage };
}

// 사용 예시
function App() {
  const { postMessage, lastMessage } = useBroadcastChannel('app-channel', (message) => {
    console.log('받은 메시지:', message);

    if (message.type === 'THEME_CHANGED') {
      setTheme(message.payload);
    }
  });

  const handleThemeChange = (newTheme: string) => {
    setTheme(newTheme);
    postMessage('THEME_CHANGED', newTheme);
  };

  return <div>...</div>;
}
```

## RemoteSeminar 서비스 적용 사례

본 기술을 이용해 **[RemoteSeminar](https://www.remoteseminar.com/)** 서비스에서 브라우저 각 탭 간 Real-Time으로 다뤄야 할 데이터에 대해 동기화를 구현했습니다.

### 구현 사례

**세미나 참여 상태 동기화**:
```javascript
// seminar-sync.js
const seminarChannel = new BroadcastChannel('seminar-channel');

// 세미나 입장
function joinSeminar(seminarId) {
  // 입장 처리
  const seminar = fetchSeminarData(seminarId);

  // 다른 탭에 알림
  seminarChannel.postMessage({
    type: 'SEMINAR_JOINED',
    payload: {
      seminarId,
      timestamp: Date.now()
    }
  });
}

// 세미나 퇴장
function leaveSeminar(seminarId) {
  // 퇴장 처리

  // 다른 탭에 알림
  seminarChannel.postMessage({
    type: 'SEMINAR_LEFT',
    payload: {
      seminarId,
      timestamp: Date.now()
    }
  });
}

// 실시간 상태 동기화
seminarChannel.onmessage = (event) => {
  const { type, payload } = event.data;

  switch (type) {
    case 'SEMINAR_JOINED':
      // 다른 탭에서 세미나에 입장했을 때
      updateParticipantList();
      break;

    case 'SEMINAR_LEFT':
      // 다른 탭에서 세미나를 나갔을 때
      updateParticipantList();
      break;

    case 'NEW_MESSAGE':
      // 새 메시지 알림
      displayNewMessage(payload);
      break;
  }
};
```

## 주의사항 및 팁

### 1. 메시지는 현재 탭을 제외한 모든 탭에 전달

```javascript
const channel = new BroadcastChannel('test');

channel.postMessage('Hello');
// 현재 탭의 onmessage는 호출되지 않음
// 다른 탭의 onmessage만 호출됨
```

**해결 방법**:
```javascript
function broadcast(message) {
  // 다른 탭에 전송
  channel.postMessage(message);

  // 현재 탭도 처리
  handleMessage(message);
}
```

### 2. 직렬화 가능한 데이터만 전송 가능

```javascript
// ✅ 가능
channel.postMessage('string');
channel.postMessage(123);
channel.postMessage({ key: 'value' });
channel.postMessage([1, 2, 3]);
channel.postMessage(null);

// ❌ 불가능
channel.postMessage(function() {});  // 함수
channel.postMessage(Symbol('test')); // Symbol
channel.postMessage(new Date());     // Date 객체는 문자열로 변환됨
```

**해결 방법**:
```javascript
// Date는 ISO 문자열로 변환
channel.postMessage({
  timestamp: new Date().toISOString()
});

// 함수는 타입만 전달하고 각 탭에서 실행
channel.postMessage({
  action: 'LOGOUT'  // 각 탭에서 logout() 함수 실행
});
```

### 3. 메모리 누수 방지

```javascript
// ❌ 나쁜 예
function initChannel() {
  const channel = new BroadcastChannel('test');
  channel.onmessage = handleMessage;
  // 채널이 닫히지 않음
}

// ✅ 좋은 예
function initChannel() {
  const channel = new BroadcastChannel('test');
  channel.onmessage = handleMessage;

  // 페이지 언로드 시 채널 닫기
  window.addEventListener('unload', () => {
    channel.close();
  });

  return channel;
}
```

### 4. 에러 처리

```javascript
const channel = new BroadcastChannel('test');

channel.onmessageerror = (event) => {
  console.error('메시지 역직렬화 오류:', event);
};

// 안전한 postMessage
function safePostMessage(data) {
  try {
    channel.postMessage(data);
  } catch (error) {
    console.error('메시지 전송 실패:', error);
  }
}
```

## BroadcastChannel vs 다른 방법

### LocalStorage + Storage Event

```javascript
// 예전 방식
localStorage.setItem('key', 'value');

window.addEventListener('storage', (event) => {
  if (event.key === 'key') {
    console.log('값 변경:', event.newValue);
  }
});
```

**단점**:
- LocalStorage에 실제로 저장해야 함
- 문자열만 가능 (객체는 JSON.stringify 필요)
- 현재 탭에서는 storage 이벤트가 발생하지 않음

### SharedWorker

```javascript
// 더 복잡한 방식
const worker = new SharedWorker('worker.js');
worker.port.postMessage('Hello');
```

**단점**:
- 별도 워커 파일 필요
- 더 복잡한 설정
- 디버깅 어려움

### BroadcastChannel이 가장 간단!

```javascript
// 간단!
const channel = new BroadcastChannel('test');
channel.postMessage({ any: 'data' });
channel.onmessage = (e) => console.log(e.data);
```

## 샘플 코드

전체 예제 코드는 [GitHub 저장소](https://github.com/4sizn/broadcastChannel-example)를 참고하세요.

## 결론

### BroadcastChannel의 장점

- ✅ **간단한 API**: 배우기 쉽고 사용하기 쉬움
- ✅ **서버 불필요**: 브라우저만으로 탭 간 통신
- ✅ **빠른 속도**: 네트워크 지연 없음
- ✅ **무료**: 추가 인프라 비용 없음
- ✅ **실시간 동기화**: 즉각적인 데이터 동기화

### 사용 시나리오

**추천**:
- 같은 사이트의 여러 탭 간 상태 동기화
- 로그인/로그아웃 동기화
- 설정 변경 동기화
- 실시간 알림 동기화

**비추천**:
- 서로 다른 도메인 간 통신 (Same-Origin 제약)
- 영구 저장이 필요한 경우
- 복잡한 메시지 큐 시스템

### 마치며

BroadcastChannel은 단순하지만 강력한 도구입니다.

적절한 상황에서 사용한다면 **복잡한 서버 인프라 없이도 탭 간 완벽한 동기화**를 구현할 수 있습니다.

---

## 참고 자료

- [MDN: BroadcastChannel API](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API)
- [Can I Use: BroadcastChannel](https://caniuse.com/broadcastchannel)
- [GitHub: broadcastChannel-example](https://github.com/4sizn/broadcastChannel-example)
- [RemoteSeminar](https://www.remoteseminar.com/)
