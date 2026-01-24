# Command/Intent 기반 업데이트

## 개요

`setState(value)` 남발 대신 `addItem`/`removeItem`/`applyFilter`처럼 "의도"가 드러나는 API로 업데이트하는 패턴입니다. 규칙/검증/로깅/테스트가 쉬워집니다.

## 특징

- **의도 명확화**: 상태 변경의 의도가 함수명에 드러남
- **비즈니스 로직 캡슐화**: 상태 변경 로직을 한 곳에 집중
- **검증 및 사이드 이펙트**: 상태 변경 전후에 검증이나 로깅 추가 용이

## 예시

```tsx
// ❌ 나쁜 예: 직접적인 상태 변경
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');
  
  const handleAdd = (text) => {
    setTodos([...todos, { id: Date.now(), text, completed: false }]);
  };
  
  const handleToggle = (id) => {
    setTodos(todos.map(t => 
      t.id === id ? { ...t, completed: !t.completed } : t
    ));
  };
  
  // 의도가 불명확하고, 검증/로깅이 어려움
}

// ✅ 좋은 예: Command 패턴
function useTodoList() {
  const [todos, setTodos] = useState([]);
  
  const addTodo = useCallback((text) => {
    if (!text.trim()) {
      throw new Error('Todo text cannot be empty');
    }
    
    const newTodo = {
      id: Date.now(),
      text: text.trim(),
      completed: false,
      createdAt: new Date().toISOString(),
    };
    
    setTodos(prev => [...prev, newTodo]);
    logEvent('todo_added', { id: newTodo.id });
  }, []);
  
  const removeTodo = useCallback((id) => {
    setTodos(prev => prev.filter(t => t.id !== id));
    logEvent('todo_removed', { id });
  }, []);
  
  const toggleTodo = useCallback((id) => {
    setTodos(prev => prev.map(t => 
      t.id === id ? { ...t, completed: !t.completed } : t
    ));
    logEvent('todo_toggled', { id });
  }, []);
  
  const clearCompleted = useCallback(() => {
    setTodos(prev => prev.filter(t => !t.completed));
    logEvent('todos_cleared');
  }, []);
  
  return { todos, addTodo, removeTodo, toggleTodo, clearCompleted };
}

// 사용 예시
function TodoList() {
  const { todos, addTodo, removeTodo, toggleTodo, clearCompleted } = useTodoList();
  
  return (
    <div>
      <TodoForm onSubmit={addTodo} />
      <TodoItems 
        todos={todos}
        onToggle={toggleTodo}
        onRemove={removeTodo}
      />
      <button onClick={clearCompleted}>Clear Completed</button>
    </div>
  );
}
```

## 구현 방법

1. **의도 기반 함수 작성**: 상태 변경의 의도를 함수명에 명확히 표현
2. **비즈니스 로직 캡슐화**: 검증, 변환, 사이드 이펙트를 함수 내부에 포함
3. **일관된 API**: 유사한 작업은 유사한 패턴으로 구현
4. **Custom Hook 활용**: 복잡한 로직은 custom hook으로 분리

## 장점

- 코드 가독성 향상 (의도가 명확)
- 비즈니스 로직 중앙화
- 테스트 용이 (각 command 단위 테스트)
- 검증 및 사이드 이펙트 추가 용이
- 버그 추적 및 로깅 용이

## 단점

- 초기 구현 시간 증가
- 작은 상태 변경에도 함수가 필요할 수 있음
- 함수 네이밍에 대한 고민 필요
