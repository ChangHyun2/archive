# Optimistic Update + Rollback

## 개요

UI를 먼저 바꾸고 서버 실패 시 되돌리기(롤백)하는 패턴입니다. 서버 상태 라이브러리의 mutation 패턴과 찰떡입니다.

## 특징

- **즉각적인 UX**: 서버 응답을 기다리지 않고 UI 즉시 업데이트
- **롤백 메커니즘**: 서버 요청 실패 시 이전 상태로 복원
- **낙관적 업데이트**: 사용자 경험을 우선시하는 접근

## 예시

```tsx
// React Query를 사용한 Optimistic Update
function TodoItem({ todo }) {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: updateTodo,
    // Optimistic update
    onMutate: async (newTodo) => {
      // 진행 중인 리패치 취소
      await queryClient.cancelQueries({ queryKey: ['todos'] });
      
      // 이전 값 백업
      const previousTodos = queryClient.getQueryData(['todos']);
      
      // 낙관적 업데이트
      queryClient.setQueryData(['todos'], (old) =>
        old.map(t => t.id === newTodo.id ? newTodo : t)
      );
      
      // 롤백을 위한 컨텍스트 반환
      return { previousTodos };
    },
    // 에러 발생 시 롤백
    onError: (err, newTodo, context) => {
      queryClient.setQueryData(['todos'], context.previousTodos);
      toast.error('업데이트 실패');
    },
    // 성공 시 리패치
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
  
  const handleToggle = () => {
    mutation.mutate({
      ...todo,
      completed: !todo.completed,
    });
  };
  
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={handleToggle}
        disabled={mutation.isPending}
      />
      {todo.text}
    </div>
  );
}

// 수동 구현 예시
function useOptimisticUpdate() {
  const [todos, setTodos] = useState([]);
  const [previousState, setPreviousState] = useState(null);
  
  const updateTodo = async (id, updates) => {
    // 이전 상태 백업
    setPreviousState(todos);
    
    // 낙관적 업데이트
    setTodos(prev => prev.map(t => 
      t.id === id ? { ...t, ...updates } : t
    ));
    
    try {
      await api.updateTodo(id, updates);
    } catch (error) {
      // 롤백
      setTodos(previousState);
      throw error;
    }
  };
  
  return { todos, updateTodo };
}
```

## 구현 방법

1. **이전 상태 백업**: 업데이트 전 현재 상태 저장
2. **즉시 UI 업데이트**: 서버 응답 전에 UI 변경
3. **에러 처리**: 실패 시 백업된 상태로 롤백
4. **성공 시 동기화**: 서버 응답으로 최종 상태 동기화

## 장점

- 빠른 사용자 피드백으로 UX 향상
- 느린 네트워크 환경에서도 반응성 유지
- 서버 상태 라이브러리와 자연스럽게 통합

## 단점

- 롤백 로직 구현 복잡도 증가
- 일시적 불일치 가능성 (서버와 클라이언트 상태)
- 충돌 해결 전략 필요
- 복잡한 상태의 경우 롤백이 어려울 수 있음
