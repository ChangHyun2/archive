# Stable Keys 전략

## 개요

리스트를 렌더링할 때 각 항목에 부여하는 key 값을 안정적이고 고유한 값으로 사용하는 패턴입니다. index를 key로 사용하면 성능 저하와 상태 관리 문제가 발생할 수 있습니다.

## 핵심 원칙

- **Stable ID 사용**: 데이터의 고유하고 안정적인 ID를 key로 사용
- **Index 지양**: 배열의 index를 key로 사용하지 않기
- **Key 변경 최소화**: key가 자주 변경되지 않도록 설계

## 문제점: Index를 Key로 사용할 때

```tsx
// ❌ 나쁜 예: index를 key로 사용
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo, index) => (
        <TodoItem key={index} todo={todo} />
      ))}
    </ul>
  );
}

// 문제점:
// 1. 항목 삭제/추가 시 불필요한 리렌더링
// 2. 컴포넌트 상태가 잘못된 항목에 매핑됨
// 3. 성능 저하 (불필요한 마운트/언마운트)
```

## 해결 방법

### 1. 고유 ID 사용

```tsx
// ✅ 좋은 예: 고유 ID 사용
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}
```

### 2. ID가 없는 경우

```tsx
// ✅ 여러 필드를 조합하여 고유한 key 생성
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        <UserItem 
          key={`${user.email}-${user.createdAt}`} 
          user={user} 
        />
      ))}
    </ul>
  );
}

// ✅ 또는 데이터에 ID 추가
function processData(data) {
  return data.map((item, index) => ({
    ...item,
    id: item.id || `generated-${index}-${Date.now()}`
  }));
}
```

### 3. 정렬/필터링 시 주의

```tsx
// ❌ 나쁜 예: 필터링 후 index 사용
function FilteredList({ items, filter }) {
  const filtered = items.filter(item => item.category === filter);
  
  return (
    <ul>
      {filtered.map((item, index) => (
        <Item key={index} item={item} />
      ))}
    </ul>
  );
}

// ✅ 좋은 예: 원본 데이터의 ID 사용
function FilteredList({ items, filter }) {
  const filtered = items.filter(item => item.category === filter);
  
  return (
    <ul>
      {filtered.map(item => (
        <Item key={item.id} item={item} />
      ))}
    </ul>
  );
}
```

## Key 변경의 영향

### 불필요한 마운트/언마운트

```tsx
// key가 변경되면 React는 완전히 새로운 컴포넌트로 인식
// 이전 컴포넌트는 언마운트되고 새 컴포넌트가 마운트됨
// → 상태 손실, 성능 저하
```

### 상태 꼬임

```tsx
function TodoItem({ todo }) {
  const [isEditing, setIsEditing] = useState(false);
  
  // key가 index인 경우:
  // [A, B, C]에서 A를 삭제하면
  // B가 A의 key를 받아서 B의 상태가 A의 상태로 보임
}
```

## Best Practices

1. **항상 고유한 ID 사용**: 데이터베이스 ID, UUID 등
2. **Key 생성 함수**: ID가 없으면 안정적인 key 생성 함수 사용
3. **Key 변경 최소화**: 데이터 구조 변경 시에도 key는 유지
4. **Key 검증**: 개발 모드에서 중복 key 경고 확인

## 예시

```tsx
// ✅ 완전한 예시
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

function TodoList({ todos }: { todos: Todo[] }) {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem 
          key={todo.id} // 안정적이고 고유한 ID
          todo={todo} 
        />
      ))}
    </ul>
  );
}
```
