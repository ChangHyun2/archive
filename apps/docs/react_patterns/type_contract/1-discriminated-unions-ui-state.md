# Discriminated Unions로 UI 상태 모델링

## 개요

`loading | error | success` 같은 상태를 타입으로 강제해서 누락/분기 버그를 방지하는 패턴입니다.

## 문제점

일반적으로 상태를 관리할 때 다음과 같은 문제가 발생할 수 있습니다:

```typescript
// ❌ 나쁜 예
type State = {
  loading: boolean;
  error: Error | null;
  data: Data | null;
};

// 이 경우 loading과 error가 동시에 true일 수 있고,
// data가 있는데도 loading이 true일 수 있는 등
// 불가능한 상태 조합이 타입으로 방지되지 않습니다.
```

## 해결 방법

Discriminated Union을 사용하여 상태를 명확하게 모델링합니다:

```typescript
// ✅ 좋은 예
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'error'; error: Error }
  | { status: 'success'; data: T };

// 사용 예시
function useAsyncData<T>(fetchFn: () => Promise<T>) {
  const [state, setState] = useState<AsyncState<T>>({ status: 'idle' });

  useEffect(() => {
    setState({ status: 'loading' });
    fetchFn()
      .then((data) => setState({ status: 'success', data }))
      .catch((error) => setState({ status: 'error', error }));
  }, []);

  return state;
}

// 컴포넌트에서 사용
function DataComponent() {
  const state = useAsyncData(() => fetch('/api/data'));

  // TypeScript가 모든 케이스를 체크합니다
  switch (state.status) {
    case 'idle':
      return <div>Ready to load</div>;
    case 'loading':
      return <div>Loading...</div>;
    case 'error':
      return <div>Error: {state.error.message}</div>; // error가 확실히 존재
    case 'success':
      return <div>Data: {state.data}</div>; // data가 확실히 존재
  }
}
```

## 장점

1. **타입 안전성**: 불가능한 상태 조합을 컴파일 타임에 방지
2. **명확성**: 각 상태가 어떤 데이터를 포함하는지 명확
3. **완전성**: 모든 상태 케이스를 처리하도록 강제 (Exhaustiveness Check와 함께 사용)

## 추가 예시

```typescript
// 폼 상태 모델링
type FormState<T> =
  | { status: 'editing'; values: Partial<T>; errors: Record<string, string> }
  | { status: 'submitting'; values: T }
  | { status: 'success'; result: T }
  | { status: 'error'; values: T; error: string };

// 모달 상태 모델링
type ModalState =
  | { isOpen: false }
  | { isOpen: true; type: 'confirm'; message: string; onConfirm: () => void }
  | { isOpen: true; type: 'alert'; message: string };
```
