# Render Props Pattern

## 개요

Render Props 패턴은 컴포넌트가 "무엇을 렌더링할지"를 함수 children으로 받아 유연한 UI 구성을 가능하게 하는 패턴입니다.

## 특징

- **유연성**: 렌더링 로직을 완전히 사용자에게 위임
- **재사용성**: 로직과 UI를 분리하여 재사용 가능
- **커스터마이징**: 동일한 로직으로 다양한 UI 구현 가능

## 기본 구조

```tsx
function DataFetcher({ url, children }: { url: string; children: (data: any) => React.ReactNode }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);

  return <>{children({ data, loading, error })}</>;
}
```

## 사용 예시

```tsx
<DataFetcher url="/api/users">
  {({ data, loading, error }) => {
    if (loading) return <Spinner />;
    if (error) return <Error message={error} />;
    return <UserList users={data} />;
  }}
</DataFetcher>
```

## children prop 대신 render prop 사용

```tsx
function DataFetcher({ 
  url, 
  render 
}: { 
  url: string; 
  render: (state: { data: any; loading: boolean; error: any }) => React.ReactNode 
}) {
  // ... 동일한 로직
  return <>{render({ data, loading, error })}</>;
}

// 사용
<DataFetcher 
  url="/api/users" 
  render={({ data, loading, error }) => (
    loading ? <Spinner /> : <UserList users={data} />
  )} 
/>
```

## Hooks와의 비교

```tsx
// Render Props
<DataFetcher url="/api/users">
  {({ data, loading }) => <UserList users={data} />}
</DataFetcher>

// Hooks (더 현대적인 접근)
function UserListComponent() {
  const { data, loading } = useDataFetcher('/api/users');
  return <UserList users={data} />;
}
```

## 장점

- 로직과 UI를 완전히 분리
- 동일한 로직으로 다양한 UI 구현 가능
- 컴포넌트 재사용성 향상

## 단점

- 중첩이 깊어질 수 있음 (Callback Hell)
- Hooks 패턴이 더 선호되는 추세
- 성능 최적화가 어려울 수 있음

## 활용 사례

- 데이터 fetching 로직 공유
- 폼 상태 관리 로직 공유
- 마우스/키보드 이벤트 추적 로직 공유

## 현대적인 대안

현재는 Custom Hooks 패턴이 Render Props를 대체하는 추세입니다:

```tsx
// Render Props 대신 Custom Hook 사용
function useDataFetcher(url: string) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  // ... 로직
  return { data, loading, error };
}
```
