# State Colocation

## 개요

상태는 "쓰는 곳"에 최대한 가깝게 두는 패턴입니다. 전역 상태부터 만들지 말고, 공유가 필요해지는 순간에만 끌어올리기(lift up)하거나 store로 승격시킵니다.

## 특징

- **로컬 우선**: 상태는 기본적으로 사용하는 컴포넌트에 가깝게 배치
- **점진적 승격**: 필요에 따라 점진적으로 상위로 이동
- **과도한 전역화 방지**: 불필요한 전역 상태 생성 방지

## 예시

```tsx
// ✅ 좋은 예: 로컬 상태로 시작
function Counter() {
  const [count, setCount] = useState(0); // 로컬 상태
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// 필요해지면 부모로 끌어올리기
function App() {
  const [count, setCount] = useState(0); // 공유가 필요해짐
  
  return (
    <div>
      <Counter count={count} onIncrement={() => setCount(count + 1)} />
      <AnotherComponent count={count} />
    </div>
  );
}

// 정말 전역이 필요할 때만 store 사용
// Zustand 예시
const useCountStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

function Counter() {
  const { count, increment } = useCountStore();
  return <button onClick={increment}>{count}</button>;
}
```

## 구현 방법

1. **로컬 상태로 시작**: useState로 컴포넌트 내부에 상태 생성
2. **공유 필요 시 lift up**: 공통 부모로 상태 이동
3. **전역 상태는 최후의 수단**: 정말 필요할 때만 store 사용
4. **점진적 리팩토링**: 필요에 따라 단계적으로 승격

## 장점

- 불필요한 전역 상태 방지
- 컴포넌트 간 결합도 감소
- 상태 관리 복잡도 최소화
- 성능 최적화 (불필요한 리렌더 방지)

## 단점

- 리팩토링이 필요할 수 있음
- 상태 위치 결정에 대한 경험 필요
- 때로는 초기 설계가 어려울 수 있음
