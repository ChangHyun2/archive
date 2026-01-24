# Event-driven State (제한적 Pub/Sub)

## 개요

토스트, 전역 모달, 트래킹 같은 "횡단 관심사"에만 이벤트로 연결하는 패턴입니다. 핵심 도메인 상태는 이벤트 지옥을 만들지 않게 조심해야 합니다.

## 특징

- **횡단 관심사 분리**: 토스트, 모달 등 전역 UI 이벤트에만 사용
- **느슨한 결합**: 이벤트 발행자와 구독자가 직접 연결되지 않음
- **제한적 사용**: 핵심 비즈니스 로직에는 사용 지양

## 예시

```tsx
// 이벤트 버스 구현
class EventBus {
  private events: Record<string, Function[]> = {};
  
  on(event: string, callback: Function) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }
  
  off(event: string, callback: Function) {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter(cb => cb !== callback);
  }
  
  emit(event: string, data?: any) {
    if (!this.events[event]) return;
    this.events[event].forEach(callback => callback(data));
  }
}

const eventBus = new EventBus();

// 토스트 알림 - 횡단 관심사
function ToastContainer() {
  const [toasts, setToasts] = useState([]);
  
  useEffect(() => {
    const handleShowToast = (message) => {
      setToasts(prev => [...prev, { id: Date.now(), message }]);
    };
    
    eventBus.on('show-toast', handleShowToast);
    return () => eventBus.off('show-toast', handleShowToast);
  }, []);
  
  return (
    <div>
      {toasts.map(toast => (
        <Toast key={toast.id} message={toast.message} />
      ))}
    </div>
  );
}

// 어디서든 토스트 표시
function LoginForm() {
  const handleLogin = async () => {
    try {
      await login();
      eventBus.emit('show-toast', { message: '로그인 성공!', type: 'success' });
    } catch (error) {
      eventBus.emit('show-toast', { message: '로그인 실패', type: 'error' });
    }
  };
  
  return <button onClick={handleLogin}>Login</button>;
}

// ❌ 나쁜 예: 핵심 도메인 상태에 이벤트 사용
// 이렇게 하면 데이터 흐름을 추적하기 어려워짐
function BadExample() {
  useEffect(() => {
    eventBus.on('user-updated', (user) => {
      // 이렇게 하면 어디서 user가 업데이트되었는지 추적 불가
      setUser(user);
    });
  }, []);
}
```

## 구현 방법

1. **이벤트 버스 생성**: 전역 이벤트 버스 또는 Context API 활용
2. **횡단 관심사 식별**: 토스트, 모달, 트래킹 등에만 사용
3. **구독/해제 관리**: useEffect에서 구독하고 cleanup에서 해제
4. **핵심 상태는 피하기**: 비즈니스 로직 상태는 이벤트 대신 명시적 상태 관리

## 장점

- 횡단 관심사 분리로 코드 구조 개선
- 컴포넌트 간 직접 의존성 제거
- 전역 UI 이벤트 처리 용이
- 유연한 이벤트 처리

## 단점

- 데이터 흐름 추적이 어려움
- 디버깅 복잡도 증가
- 과도한 사용 시 "이벤트 지옥" 발생 가능
- 타입 안정성 저하 가능
