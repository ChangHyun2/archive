# Container / Presentational Pattern (Smart/Dumb)

## 개요

Container / Presentational 패턴은 데이터/비즈니스 로직과 UI를 분리해서 재사용성과 테스트 용이성을 높이는 패턴입니다. 요즘은 "느슨하게 적용"하는 편이지만, 설계 원칙으로는 여전히 유효합니다.

## 특징

- **관심사 분리**: 로직과 UI를 명확히 분리
- **재사용성**: Presentational 컴포넌트는 다양한 Container에서 재사용 가능
- **테스트 용이성**: 각각 독립적으로 테스트 가능

## Container Component (Smart Component)

데이터와 비즈니스 로직을 담당하는 컴포넌트

```tsx
function UserListContainer() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);

  const handleDelete = async (id: string) => {
    await deleteUser(id);
    setUsers(users.filter(user => user.id !== id));
  };

  return (
    <UserList 
      users={users}
      loading={loading}
      error={error}
      onDelete={handleDelete}
    />
  );
}
```

## Presentational Component (Dumb Component)

UI 렌더링만 담당하는 컴포넌트

```tsx
type UserListProps = {
  users: User[];
  loading: boolean;
  error: string | null;
  onDelete: (id: string) => void;
};

function UserList({ users, loading, error, onDelete }: UserListProps) {
  if (loading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name}
          <button onClick={() => onDelete(user.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

## 현대적인 접근: Custom Hooks

요즘은 Container 컴포넌트 대신 Custom Hooks를 사용하는 추세입니다:

```tsx
// Custom Hook (로직 분리)
function useUsers() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);

  const deleteUser = async (id: string) => {
    await deleteUser(id);
    setUsers(users.filter(user => user.id !== id));
  };

  return { users, loading, error, deleteUser };
}

// 컴포넌트 (로직 + UI)
function UserList() {
  const { users, loading, error, deleteUser } = useUsers();
  
  if (loading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name}
          <button onClick={() => deleteUser(user.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

## 장점

- **관심사 분리**: 로직과 UI가 명확히 분리됨
- **재사용성**: Presentational 컴포넌트는 다양한 곳에서 재사용 가능
- **테스트 용이성**: 각각 독립적으로 테스트 가능
- **유지보수성**: 변경 사항이 명확하게 분리됨

## 단점

- **과도한 분리**: 간단한 컴포넌트를 과도하게 분리할 필요는 없음
- **보일러플레이트**: 작은 컴포넌트에 대해서는 오버엔지니어링일 수 있음

## 언제 사용할까?

- 복잡한 비즈니스 로직이 있는 경우
- 동일한 UI를 다양한 데이터 소스와 사용하는 경우
- 테스트가 중요한 경우

## 언제 사용하지 않을까?

- 간단한 컴포넌트의 경우
- 로직이 거의 없는 경우
- 과도한 분리가 오버헤드가 되는 경우

## 결론

이 패턴은 여전히 유효한 설계 원칙이지만, 현대적인 React에서는 Custom Hooks를 통해 더 유연하게 적용할 수 있습니다. 중요한 것은 "로직과 UI를 분리한다"는 원칙을 유지하는 것입니다.
