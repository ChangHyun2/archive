# Typed Error Contracts

## 개요

에러를 `ErrorCode` 유니온으로 표준화해서 UI가 에러 처리 UX를 안정적으로 구현하는 패턴입니다.

## 문제점

일반적으로 에러 처리가 일관되지 않고 타입 안전하지 않습니다:

```typescript
// ❌ 나쁜 예
try {
  await fetchUser(id);
} catch (error) {
  // error의 타입이 unknown이고, 어떤 에러인지 알 수 없음
  if (error instanceof Error) {
    console.error(error.message);
    // 에러 종류에 따른 다른 UX 처리가 어려움
  }
}
```

## 해결 방법: ErrorCode 유니온

```typescript
// ✅ 좋은 예
// 1. ErrorCode 정의
type ErrorCode =
  | 'USER_NOT_FOUND'
  | 'UNAUTHORIZED'
  | 'NETWORK_ERROR'
  | 'VALIDATION_ERROR'
  | 'SERVER_ERROR'
  | 'RATE_LIMIT_EXCEEDED';

// 2. Typed Error 클래스
class AppError extends Error {
  constructor(
    public code: ErrorCode,
    message: string,
    public details?: unknown
  ) {
    super(message);
    this.name = 'AppError';
  }
}

// 3. 에러 생성 헬퍼
const createError = {
  userNotFound: (userId: string) =>
    new AppError('USER_NOT_FOUND', `User ${userId} not found`, { userId }),
  
  unauthorized: () =>
    new AppError('UNAUTHORIZED', 'Authentication required'),
  
  networkError: (originalError: Error) =>
    new AppError('NETWORK_ERROR', 'Network request failed', { originalError }),
  
  validationError: (field: string, message: string) =>
    new AppError('VALIDATION_ERROR', message, { field }),
  
  serverError: (status: number) =>
    new AppError('SERVER_ERROR', `Server error: ${status}`, { status }),
  
  rateLimitExceeded: (retryAfter: number) =>
    new AppError('RATE_LIMIT_EXCEEDED', 'Rate limit exceeded', { retryAfter }),
};

// 4. API 호출에서 사용
async function fetchUser(id: string): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`);
    
    if (response.status === 404) {
      throw createError.userNotFound(id);
    }
    
    if (response.status === 401) {
      throw createError.unauthorized();
    }
    
    if (!response.ok) {
      throw createError.serverError(response.status);
    }
    
    return await response.json();
  } catch (error) {
    if (error instanceof AppError) {
      throw error; // 이미 AppError면 그대로 전달
    }
    throw createError.networkError(error as Error);
  }
}
```

## UI에서 에러 처리

```typescript
// ✅ 컴포넌트에서 타입 안전한 에러 처리
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [error, setError] = useState<AppError | null>(null);

  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .catch((err) => {
        if (err instanceof AppError) {
          setError(err);
        } else {
          setError(createError.networkError(err as Error));
        }
      });
  }, [userId]);

  if (error) {
    // ErrorCode에 따른 다른 UX 처리
    switch (error.code) {
      case 'USER_NOT_FOUND':
        return <NotFoundMessage userId={userId} />;
      
      case 'UNAUTHORIZED':
        return <RedirectToLogin />;
      
      case 'NETWORK_ERROR':
        return <NetworkErrorRetry onRetry={() => fetchUser(userId)} />;
      
      case 'RATE_LIMIT_EXCEEDED':
        return <RateLimitMessage retryAfter={error.details?.retryAfter} />;
      
      case 'VALIDATION_ERROR':
        return <ValidationError field={error.details?.field} />;
      
      case 'SERVER_ERROR':
        return <ServerError />;
      
      default:
        // Exhaustiveness Check
        const _exhaustive: never = error.code;
        return <GenericError />;
    }
  }

  if (!user) return <Loading />;
  
  return <div>{user.name}</div>;
}
```

## React Hook으로 추상화

```typescript
// ✅ 에러 처리를 Hook으로 추상화
function useAsyncData<T>(
  fetchFn: () => Promise<T>
): {
  data: T | null;
  error: AppError | null;
  loading: boolean;
  retry: () => void;
} {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<AppError | null>(null);
  const [loading, setLoading] = useState(true);

  const execute = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await fetchFn();
      setData(result);
    } catch (err) {
      if (err instanceof AppError) {
        setError(err);
      } else {
        setError(createError.networkError(err as Error));
      }
    } finally {
      setLoading(false);
    }
  }, [fetchFn]);

  useEffect(() => {
    execute();
  }, [execute]);

  return { data, error, loading, retry: execute };
}

// 사용
function UserProfile({ userId }: { userId: string }) {
  const { data: user, error, loading, retry } = useAsyncData(
    () => fetchUser(userId)
  );

  if (loading) return <Loading />;
  if (error) return <ErrorDisplay error={error} onRetry={retry} />;
  if (!user) return null;

  return <div>{user.name}</div>;
}
```

## Error Boundary와 통합

```typescript
// ✅ Error Boundary에서 AppError 처리
class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { error: AppError | null }
> {
  constructor(props: { children: React.ReactNode }) {
    super(props);
    this.state = { error: null };
  }

  static getDerivedStateFromError(error: unknown): { error: AppError | null } {
    if (error instanceof AppError) {
      return { error };
    }
    return { error: createError.serverError(500) };
  }

  render() {
    if (this.state.error) {
      // ErrorCode에 따른 다른 UI
      switch (this.state.error.code) {
        case 'UNAUTHORIZED':
          return <RedirectToLogin />;
        case 'SERVER_ERROR':
          return <ServerErrorPage />;
        default:
          return <GenericErrorPage error={this.state.error} />;
      }
    }

    return this.props.children;
  }
}
```

## 폼 검증 에러

```typescript
// ✅ 폼 검증 에러도 ErrorCode로 통일
type ValidationErrorCode = 
  | 'REQUIRED'
  | 'INVALID_EMAIL'
  | 'PASSWORD_TOO_SHORT'
  | 'PASSWORD_MISMATCH';

const createValidationError = (code: ValidationErrorCode, field: string) =>
  createError.validationError(field, code);

// 사용
function validateLoginForm(form: LoginForm): void {
  if (!form.email) {
    throw createValidationError('REQUIRED', 'email');
  }
  
  if (!form.email.includes('@')) {
    throw createValidationError('INVALID_EMAIL', 'email');
  }
  
  if (!form.password || form.password.length < 8) {
    throw createValidationError('PASSWORD_TOO_SHORT', 'password');
  }
}
```

## 장점

1. **일관성**: 모든 에러가 표준화된 형식
2. **타입 안전성**: ErrorCode 유니온으로 모든 케이스 처리 강제
3. **UX 일관성**: 같은 에러 코드는 항상 같은 UX 처리
4. **유지보수성**: 에러 처리 로직이 중앙화되어 관리 용이
5. **테스트 용이성**: 에러 케이스를 체계적으로 테스트 가능
