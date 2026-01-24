# Derived State 최소화

## 개요

"계산으로 얻을 수 있는 값"은 state로 들고 있지 말고 selector나 useMemo로 파생시키는 패턴입니다. 동기화 버그의 80%가 여기서 발생합니다.

## 특징

- **계산 기반 파생**: 저장된 상태에서 계산 가능한 값은 별도 상태로 관리하지 않음
- **동기화 버그 방지**: 상태 간 불일치 문제 해결
- **성능 최적화**: useMemo를 통한 불필요한 재계산 방지

## 예시

```tsx
// ❌ 나쁜 예: 파생 상태를 별도로 관리
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [completedCount, setCompletedCount] = useState(0); // 파생 상태
  
  useEffect(() => {
    setCompletedCount(todos.filter(t => t.completed).length);
  }, [todos]); // 동기화 필요 - 버그 위험!
}

// ✅ 좋은 예: useMemo로 파생
function TodoList() {
  const [todos, setTodos] = useState([]);
  
  const completedCount = useMemo(
    () => todos.filter(t => t.completed).length,
    [todos]
  );
  
  const activeTodos = useMemo(
    () => todos.filter(t => !t.completed),
    [todos]
  );
  
  return (
    <div>
      <p>Completed: {completedCount}</p>
      {activeTodos.map(todo => <TodoItem key={todo.id} todo={todo} />)}
    </div>
  );
}
```

## 구현 방법

1. **파생 가능 여부 확인**: 현재 상태에서 계산 가능한지 판단
2. **useMemo 활용**: 계산 비용이 있는 경우 useMemo로 메모이제이션
3. **Selector 패턴**: Redux/Zustand 사용 시 selector로 파생 값 계산
4. **상태 제거**: 파생 상태를 별도 state로 관리하지 않음

## 장점

- 동기화 버그 근본적 해결
- 코드 간결성 향상
- 상태 관리 복잡도 감소
- 성능 최적화 가능 (useMemo 활용)

## 단점

- 초기 설계 시 파생 가능 여부 판단 필요
- useMemo 의존성 배열 관리 주의 필요
- 복잡한 계산의 경우 성능 고려 필요
