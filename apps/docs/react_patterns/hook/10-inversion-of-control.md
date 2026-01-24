# Inversion of Control (콜백 주입)

## 개요

Hook이 모든 결정을 하지 않고, 중요한 지점은 외부 콜백으로 위임하는 패턴입니다.

## 핵심 원칙

- **유연성**: Hook이 하드코딩된 로직 대신 외부에서 주입받은 로직 사용
- **재사용성**: 다양한 시나리오에 동일한 Hook 적용 가능
- **테스트 용이성**: 콜백을 모킹하여 쉽게 테스트

## 기본 패턴

### useForm Hook

```tsx
function useForm<T>(options: {
  initialValues: T;
  validate?: (values: T) => Record<string, string>;
  onSubmit: (values: T) => void | Promise<void>;
}) {
  const { initialValues, validate, onSubmit } = options;
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = useCallback((name: string, value: any) => {
    setValues(prev => ({ ...prev, [name]: value }));
    // 에러 초기화
    if (errors[name]) {
      setErrors(prev => {
        const next = { ...prev };
        delete next[name];
        return next;
      });
    }
  }, [errors]);
  
  const handleSubmit = useCallback(async (e: React.FormEvent) => {
    e.preventDefault();
    
    // 외부에서 주입받은 검증 함수 사용
    if (validate) {
      const validationErrors = validate(values);
      if (Object.keys(validationErrors).length > 0) {
        setErrors(validationErrors);
        return;
      }
    }
    
    setIsSubmitting(true);
    try {
      // 외부에서 주입받은 제출 함수 사용
      await onSubmit(values);
    } finally {
      setIsSubmitting(false);
    }
  }, [values, validate, onSubmit]);
  
  return {
    values,
    errors,
    isSubmitting,
    handleChange,
    handleSubmit,
  };
}
```

### 사용 예시

```tsx
function LoginForm() {
  const { values, errors, isSubmitting, handleChange, handleSubmit } = useForm({
    initialValues: { email: '', password: '' },
    
    // 검증 로직 주입
    validate: (values) => {
      const errors: Record<string, string> = {};
      if (!values.email) {
        errors.email = 'Email is required';
      } else if (!/\S+@\S+\.\S+/.test(values.email)) {
        errors.email = 'Email is invalid';
      }
      if (!values.password) {
        errors.password = 'Password is required';
      } else if (values.password.length < 8) {
        errors.password = 'Password must be at least 8 characters';
      }
      return errors;
    },
    
    // 제출 로직 주입
    onSubmit: async (values) => {
      await login(values.email, values.password);
      router.push('/dashboard');
    },
  });
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={values.email}
        onChange={(e) => handleChange('email', e.target.value)}
      />
      {errors.email && <span>{errors.email}</span>}
      
      <input
        type="password"
        value={values.password}
        onChange={(e) => handleChange('password', e.target.value)}
      />
      {errors.password && <span>{errors.password}</span>}
      
      <button type="submit" disabled={isSubmitting}>
        Login
      </button>
    </form>
  );
}
```

## 고급 패턴

### useAsync with Callbacks

```tsx
function useAsync<TData, TVariables>(
  asyncFn: (variables: TVariables) => Promise<TData>,
  options: {
    onSuccess?: (data: TData) => void;
    onError?: (error: Error) => void;
    onRetry?: (error: Error, retry: () => void) => void;
  } = {}
) {
  const { onSuccess, onError, onRetry } = options;
  const [state, setState] = useState<{
    data: TData | null;
    loading: boolean;
    error: Error | null;
  }>({
    data: null,
    loading: false,
    error: null,
  });
  
  const execute = useCallback(async (variables: TVariables) => {
    setState({ data: null, loading: true, error: null });
    
    try {
      const data = await asyncFn(variables);
      setState({ data, loading: false, error: null });
      onSuccess?.(data);
      return data;
    } catch (error) {
      const err = error as Error;
      setState({ data: null, loading: false, error: err });
      
      // 재시도 로직 주입
      if (onRetry) {
        onRetry(err, () => execute(variables));
      } else {
        onError?.(err);
      }
      
      throw err;
    }
  }, [asyncFn, onSuccess, onError, onRetry]);
  
  return {
    ...state,
    execute,
  };
}
```

### 사용 예시

```tsx
function UserCreation() {
  const { data, loading, error, execute } = useAsync(createUser, {
    // 성공 시 콜백
    onSuccess: (user) => {
      toast.success(`User ${user.name} created!`);
      router.push(`/users/${user.id}`);
    },
    
    // 에러 시 콜백
    onError: (error) => {
      toast.error(`Failed to create user: ${error.message}`);
    },
    
    // 재시도 로직 주입
    onRetry: (error, retry) => {
      if (error.message.includes('network')) {
        // 네트워크 에러면 자동 재시도
        setTimeout(retry, 1000);
      } else {
        // 다른 에러면 사용자에게 확인
        if (confirm('Retry?')) {
          retry();
        }
      }
    },
  });
  
  return (
    <button onClick={() => execute({ name: 'John' })}>
      Create User
    </button>
  );
}
```

### usePagination with Callbacks

```tsx
function usePagination<T>(
  fetcher: (page: number, pageSize: number) => Promise<{ data: T[]; total: number }>,
  options: {
    initialPage?: number;
    pageSize?: number;
    onPageChange?: (page: number) => void;
    onDataChange?: (data: T[]) => void;
  } = {}
) {
  const {
    initialPage = 1,
    pageSize = 10,
    onPageChange,
    onDataChange,
  } = options;
  
  const [page, setPage] = useState(initialPage);
  const [data, setData] = useState<T[]>([]);
  const [total, setTotal] = useState(0);
  const [loading, setLoading] = useState(false);
  
  const loadPage = useCallback(async (pageNum: number) => {
    setLoading(true);
    try {
      const result = await fetcher(pageNum, pageSize);
      setData(result.data);
      setTotal(result.total);
      setPage(pageNum);
      
      // 외부 콜백 호출
      onPageChange?.(pageNum);
      onDataChange?.(result.data);
    } finally {
      setLoading(false);
    }
  }, [fetcher, pageSize, onPageChange, onDataChange]);
  
  useEffect(() => {
    loadPage(page);
  }, [loadPage, page]);
  
  const goToPage = useCallback((pageNum: number) => {
    loadPage(pageNum);
  }, [loadPage]);
  
  return {
    data,
    page,
    total,
    pageSize,
    loading,
    totalPages: Math.ceil(total / pageSize),
    goToPage,
    nextPage: () => goToPage(page + 1),
    prevPage: () => goToPage(page - 1),
  };
}
```

## 패턴: Strategy Pattern 적용

```tsx
// 다양한 정렬 전략을 주입받는 Hook
function useSortedList<T>(
  items: T[],
  sortStrategy: (a: T, b: T) => number
) {
  const sortedItems = useMemo(() => {
    return [...items].sort(sortStrategy);
  }, [items, sortStrategy]);
  
  return sortedItems;
}

// 사용
function ProductList({ products }) {
  // 정렬 전략 주입
  const sortedByPrice = useSortedList(products, (a, b) => a.price - b.price);
  const sortedByName = useSortedList(products, (a, b) => a.name.localeCompare(b.name));
  
  return (
    <div>
      <h2>By Price</h2>
      {sortedByPrice.map(p => <Product key={p.id} product={p} />)}
      
      <h2>By Name</h2>
      {sortedByName.map(p => <Product key={p.id} product={p} />)}
    </div>
  );
}
```

## 장점

1. **유연성**: 다양한 시나리오에 동일한 Hook 적용
2. **재사용성**: 비즈니스 로직을 Hook 밖에서 정의
3. **테스트 용이성**: 콜백을 모킹하여 쉽게 테스트
4. **관심사 분리**: Hook은 구조만, 로직은 외부에서

## 주의사항

- 콜백이 너무 많아지면 복잡도 증가
- 자주 사용되는 콜백 조합은 별도 Hook으로 추상화 고려
- 콜백의 타입을 명확히 정의 (TypeScript)

## 사용 시기

- Hook이 다양한 비즈니스 로직을 처리해야 할 때
- 검증, 변환, 필터링 등의 로직이 상황에 따라 달라질 때
- Hook을 범용적으로 만들고 싶을 때
