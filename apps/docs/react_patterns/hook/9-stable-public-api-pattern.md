# Stable Public API Pattern (Hook 반환값 설계)

## 개요

Hook이 반환하는 형태를 일관되게 설계하여 호출하는 쪽에서 사용법이 통일되도록 하는 패턴입니다. 팀 생산성을 높입니다.

## 핵심 원칙

- **일관된 구조**: 모든 Hook이 유사한 반환값 구조를 가짐
- **예측 가능성**: Hook 이름만 봐도 반환값 구조를 예측 가능
- **타입 안정성**: TypeScript로 인터페이스 명확히 정의

## 표준 반환값 구조

### 데이터 Fetching Hook

```tsx
// 표준 구조: { data, status, error, actions }
type FetchHookResult<T> = {
  data: T | null;
  status: 'idle' | 'loading' | 'success' | 'error';
  error: Error | null;
  refetch: () => void;
};

function useUser(userId: string): FetchHookResult<User> {
  const [data, setData] = useState<User | null>(null);
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
  const [error, setError] = useState<Error | null>(null);
  
  const refetch = useCallback(async () => {
    setStatus('loading');
    try {
      const user = await fetchUser(userId);
      setData(user);
      setStatus('success');
    } catch (err) {
      setError(err as Error);
      setStatus('error');
    }
  }, [userId]);
  
  useEffect(() => {
    refetch();
  }, [refetch]);
  
  return { data, status, error, refetch };
}
```

### Mutation Hook

```tsx
// 표준 구조: { mutate, status, error, reset }
type MutationHookResult<TData, TVariables> = {
  mutate: (variables: TVariables) => Promise<TData>;
  status: 'idle' | 'loading' | 'success' | 'error';
  error: Error | null;
  reset: () => void;
};

function useCreateUser(): MutationHookResult<User, CreateUserInput> {
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
  const [error, setError] = useState<Error | null>(null);
  
  const mutate = useCallback(async (input: CreateUserInput) => {
    setStatus('loading');
    setError(null);
    
    try {
      const user = await createUser(input);
      setStatus('success');
      return user;
    } catch (err) {
      const error = err as Error;
      setError(error);
      setStatus('error');
      throw error;
    }
  }, []);
  
  const reset = useCallback(() => {
    setStatus('idle');
    setError(null);
  }, []);
  
  return { mutate, status, error, reset };
}
```

### 상태 관리 Hook

```tsx
// 표준 구조: { value, setValue, actions }
type StateHookResult<T> = {
  value: T;
  setValue: (value: T | ((prev: T) => T)) => void;
  reset: () => void;
};

function useToggle(initialValue = false): StateHookResult<boolean> {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => {
    setValue(prev => !prev);
  }, []);
  
  const reset = useCallback(() => {
    setValue(initialValue);
  }, [initialValue]);
  
  return {
    value,
    setValue,
    toggle,
    reset,
  };
}
```

## 일관된 네이밍 컨벤션

### 1. 상태 필드

```tsx
// ✅ 일관된 네이밍
const { data, loading, error } = useQuery();
const { data, loading, error } = useMutation();
const { data, loading, error } = useFetch();

// ❌ 혼란스러운 네이밍
const { result, isLoading, err } = useQuery();
const { value, pending, failure } = useMutation();
```

## 실전 예시: 통일된 API 설계

### Query 계열 Hook

```tsx
// 모든 Query Hook이 동일한 구조
function useUsers(): FetchHookResult<User[]> {
  // ...
  return { data, status, error, refetch };
}

function useUser(id: string): FetchHookResult<User> {
  // ...
  return { data, status, error, refetch };
}

function useProducts(): FetchHookResult<Product[]> {
  // ...
  return { data, status, error, refetch };
}
```

### Mutation 계열 Hook

```tsx
// 모든 Mutation Hook이 동일한 구조
function useCreateUser(): MutationHookResult<User, CreateUserInput> {
  // ...
  return { mutate, status, error, reset };
}

function useUpdateUser(): MutationHookResult<User, UpdateUserInput> {
  // ...
  return { mutate, status, error, reset };
}

function useDeleteUser(): MutationHookResult<void, string> {
  // ...
  return { mutate, status, error, reset };
}
```

## 고급 패턴: Generic Hook Factory

```tsx
// 재사용 가능한 Hook 팩토리
function createQueryHook<T>(
  fetcher: () => Promise<T>
): () => FetchHookResult<T> {
  return function useQuery() {
    const [data, setData] = useState<T | null>(null);
    const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
    const [error, setError] = useState<Error | null>(null);
    
    const refetch = useCallback(async () => {
      setStatus('loading');
      try {
        const result = await fetcher();
        setData(result);
        setStatus('success');
      } catch (err) {
        setError(err as Error);
        setStatus('error');
      }
    }, []);
    
    useEffect(() => {
      refetch();
    }, [refetch]);
    
    return { data, status, error, refetch };
  };
}

// 사용
const useUsers = createQueryHook(() => fetchUsers());
const useProducts = createQueryHook(() => fetchProducts());
```

## 컴포넌트에서의 사용

```tsx
// 모든 Hook이 동일한 구조라서 사용법이 일관됨
function UserDashboard() {
  const { data: users, status, error, refetch } = useUsers();
  const { mutate: createUser, status: createStatus } = useCreateUser();
  
  if (status === 'loading') return <Spinner />;
  if (status === 'error') return <Error message={error.message} />;
  
  const handleCreate = async () => {
    try {
      await createUser({ name: 'John' });
      refetch(); // 동일한 패턴으로 새로고침
    } catch (err) {
      // 에러는 mutate에서 throw되므로 여기서 처리
    }
  };
  
  return (
    <div>
      <button onClick={handleCreate} disabled={createStatus === 'loading'}>
        Create User
      </button>
      {users?.map(user => <UserCard key={user.id} user={user} />)}
    </div>
  );
}
```

## 타입 정의로 강제하기

```tsx
// 공통 타입 정의
export type HookResult<T> = {
  data: T | null;
  status: 'idle' | 'loading' | 'success' | 'error';
  error: Error | null;
};

export type QueryHook<T> = () => HookResult<T> & {
  refetch: () => void;
};

export type MutationHook<TData, TVariables> = () => {
  mutate: (variables: TVariables) => Promise<TData>;
} & Omit<HookResult<TData>, 'data'> & {
  reset: () => void;
};

// 사용
const useUsers: QueryHook<User[]> = () => {
  // 반환값이 QueryHook 타입과 일치해야 함
  return { data, status, error, refetch };
};
```

## 장점

1. **학습 곡선 감소**: 한 Hook을 배우면 다른 Hook도 쉽게 사용
2. **코드 일관성**: 팀 전체가 동일한 패턴 사용
3. **리팩토링 용이**: 구조가 일관되어 변경이 쉬움
4. **타입 안정성**: TypeScript로 구조 강제 가능
5. **생산성 향상**: 새로운 Hook 사용 시 문서 참조 최소화

## 주의사항

- 모든 Hook에 무조건 적용할 필요는 없음 (과도한 추상화 방지)
- 팀 내에서 컨벤션을 문서화하고 공유
- 기존 Hook과의 호환성 고려
