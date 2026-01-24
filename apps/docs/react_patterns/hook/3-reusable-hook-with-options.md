# Reusable Hook + "옵션/어댑터"로 확장

## 개요

Hook이 모든 케이스를 다 알게 하지 말고, options(callback, mapper 등)로 변형 가능하게 만드는 패턴입니다.

## 핵심 원칙

- **확장 가능성**: Hook이 모든 케이스를 하드코딩하지 않음
- **유연성**: options를 통해 다양한 시나리오에 대응
- **재사용성**: 하나의 Hook으로 여러 상황에 적용 가능

## 예시

### 기본 useAsync Hook

```tsx
function useAsync(asyncFn, options = {}) {
  const {
    onSuccess,
    onError,
    mapResult,
    enabled = true,
  } = options;

  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const execute = useCallback(async (...args) => {
    if (!enabled) return;
    
    setLoading(true);
    setError(null);
    
    try {
      let result = await asyncFn(...args);
      
      // 결과 변환 (mapper)
      if (mapResult) {
        result = mapResult(result);
      }
      
      setData(result);
      
      // 성공 콜백
      if (onSuccess) {
        onSuccess(result);
      }
    } catch (err) {
      setError(err);
      
      // 에러 콜백
      if (onError) {
        onError(err);
      }
    } finally {
      setLoading(false);
    }
  }, [asyncFn, onSuccess, onError, mapResult, enabled]);

  return { data, loading, error, execute };
}
```

### 다양한 사용 사례

```tsx
// 1. 기본 사용
function UserList() {
  const fetchUsers = () => fetch('/api/users').then(r => r.json());
  const { data: users, loading, execute } = useAsync(fetchUsers);
  
  useEffect(() => {
    execute();
  }, [execute]);
  
  return <div>{/* users 렌더링 */}</div>;
}

// 2. 결과 변환 (mapper)
function UserStats() {
  const fetchUsers = () => fetch('/api/users').then(r => r.json());
  const { data: stats } = useAsync(fetchUsers, {
    mapResult: (users) => ({
      total: users.length,
      active: users.filter(u => u.active).length,
    }),
  });
  
  return <div>Total: {stats?.total}</div>;
}

// 3. 성공/에러 핸들링 (callbacks)
function CreateUser() {
  const createUser = (userData) => 
    fetch('/api/users', {
      method: 'POST',
      body: JSON.stringify(userData),
    }).then(r => r.json());
  
  const { execute, loading } = useAsync(createUser, {
    onSuccess: (user) => {
      toast.success(`User ${user.name} created!`);
      router.push(`/users/${user.id}`);
    },
    onError: (error) => {
      toast.error(`Failed to create user: ${error.message}`);
    },
  });
  
  return <button onClick={() => execute({ name: 'John' })}>
    Create User
  </button>;
}

// 4. 조건부 실행 (enabled)
function UserProfile({ userId }) {
  const fetchUser = (id) => 
    fetch(`/api/users/${id}`).then(r => r.json());
  
  const { data: user } = useAsync(
    () => fetchUser(userId),
    { enabled: !!userId } // userId가 있을 때만 실행
  );
  
  return <div>{user?.name}</div>;
}
```

## 고급 패턴: 어댑터 패턴

```tsx
// 다양한 데이터 소스에 대한 어댑터
function useAsyncWithAdapter(asyncFn, adapter) {
  const { data, loading, error, execute } = useAsync(asyncFn, {
    mapResult: adapter.transform,
    onError: adapter.handleError,
  });
  
  return {
    ...adapter.format(data),
    loading,
    error,
    execute,
  };
}

// GraphQL 어댑터
const graphQLAdapter = {
  transform: (response) => response.data,
  handleError: (error) => console.error('GraphQL Error:', error),
  format: (data) => ({ result: data }),
};

// REST API 어댑터
const restAdapter = {
  transform: (response) => response,
  handleError: (error) => console.error('REST Error:', error),
  format: (data) => ({ items: data }),
};
```

## 장점

1. **유연성**: 하나의 Hook으로 다양한 시나리오 처리
2. **확장성**: 새로운 옵션을 추가하기 쉬움
3. **재사용성**: 다양한 맥락에서 동일한 Hook 사용
4. **테스트 용이성**: 각 옵션을 독립적으로 테스트 가능

## 주의사항

- 옵션이 너무 많아지면 복잡도가 증가할 수 있음
- 자주 사용되는 옵션 조합은 별도 Hook으로 추상화 고려
- 옵션의 기본값을 명확히 정의
