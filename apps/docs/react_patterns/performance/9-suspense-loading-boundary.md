# Suspense 기반 로딩 경계

## 개요

React Suspense를 사용하여 로딩 상태를 컴포넌트 트리의 경계로 체계적으로 관리하는 패턴입니다. 중첩된 로딩 상태와 스트리밍 렌더링을 구현할 수 있습니다.

## 핵심 원칙

- **로딩 경계**: Suspense로 로딩 상태를 컴포넌트 트리 경계로 관리
- **중첩 로딩**: 여러 Suspense로 독립적인 로딩 상태 관리
- **스트리밍**: 데이터가 준비되는 대로 점진적으로 렌더링

## 사용 방법

### 기본 Suspense 사용

```tsx
import { Suspense, lazy } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <LazyComponent />
    </Suspense>
  );
}
```

### 중첩 Suspense

```tsx
// ✅ 여러 로딩 경계로 독립적인 로딩 상태 관리
function Dashboard() {
  return (
    <div>
      <Suspense fallback={<HeaderSkeleton />}>
        <Header />
      </Suspense>
      
      <Suspense fallback={<ContentSkeleton />}>
        <MainContent />
      </Suspense>
      
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />
      </Suspense>
    </div>
  );
}
```

### 데이터 페칭과 Suspense

```tsx
// ✅ React Query, SWR 등과 함께 사용
import { useQuery } from '@tanstack/react-query';
import { Suspense } from 'react';

function UserProfile({ userId }) {
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    suspense: true, // Suspense 모드 활성화
  });
  
  return <div>{user.name}</div>;
}

function App() {
  return (
    <Suspense fallback={<UserProfileSkeleton />}>
      <UserProfile userId={1} />
    </Suspense>
  );
}
```

### 커스텀 Suspense 경계

```tsx
// ✅ 에러와 로딩을 함께 처리하는 경계
function ErrorBoundary({ children, fallback }) {
  const [hasError, setHasError] = useState(false);
  
  useEffect(() => {
    const errorHandler = () => setHasError(true);
    window.addEventListener('error', errorHandler);
    return () => window.removeEventListener('error', errorHandler);
  }, []);
  
  if (hasError) {
    return fallback || <div>Something went wrong</div>;
  }
  
  return children;
}

function App() {
  return (
    <ErrorBoundary fallback={<ErrorFallback />}>
      <Suspense fallback={<Loading />}>
        <LazyComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

### 스트리밍 렌더링

```tsx
// ✅ 서버 컴포넌트와 함께 스트리밍
// (Next.js 13+ App Router 예시)
async function Page() {
  return (
    <div>
      <Suspense fallback={<PostsSkeleton />}>
        <Posts />
      </Suspense>
      
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments />
      </Suspense>
    </div>
  );
}

// Posts와 Comments가 독립적으로 스트리밍됨
```

### 조건부 Suspense

```tsx
// ✅ 조건에 따라 Suspense 경계 적용
function ConditionalSuspense({ condition, children, fallback }) {
  if (condition) {
    return (
      <Suspense fallback={fallback}>
        {children}
      </Suspense>
    );
  }
  
  return children;
}

function App() {
  const [showHeavyComponent, setShowHeavyComponent] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowHeavyComponent(true)}>
        Load Heavy Component
      </button>
      
      <ConditionalSuspense
        condition={showHeavyComponent}
        fallback={<Loading />}
      >
        <HeavyComponent />
      </ConditionalSuspense>
    </div>
  );
}
```

## Best Practices

1. **적절한 fallback**: 각 Suspense에 의미 있는 로딩 UI 제공
2. **경계 분리**: 독립적으로 로드 가능한 부분은 별도 Suspense
3. **에러 처리**: ErrorBoundary와 함께 사용하여 에러 처리
4. **접근성**: 로딩 상태를 스크린 리더에 알림
5. **중첩 최소화**: 너무 깊은 중첩은 피하기

## 주의사항

- Suspense는 현재 React.lazy와 일부 데이터 페칭 라이브러리에서만 지원
- 모든 비동기 작업이 Suspense를 지원하는 것은 아님
- 서버 컴포넌트 환경에서 더 강력한 기능 제공
