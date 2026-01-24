# Hook Layering (계층화)

## 개요

Hook을 저수준에서 고수준으로 계층화하여 쌓아 올리는 패턴입니다. 의존성과 책임이 명확해지고 재사용이 쉬워집니다.

## 핵심 원칙

- **저수준 → 고수준**: 기본적인 Hook을 먼저 만들고, 이를 조합하여 더 구체적인 Hook 생성
- **의존성 명확화**: 각 계층의 책임과 의존성이 명확함
- **재사용성 향상**: 저수준 Hook을 다양한 고수준 Hook에서 재사용

## 예시

### 계층 구조

```
useFetch() (저수준 - 범용)
    ↓
useUserQuery() (중간 수준 - 도메인 특화)
    ↓
useCurrentUser() (고수준 - 비즈니스 로직)
```

### 구현 예시

```tsx
// 1. 저수준: 범용 fetch Hook
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch(url, options)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);

  return { data, loading, error };
}

// 2. 중간 수준: User 도메인 특화 Hook
function useUserQuery(userId) {
  const { data, loading, error } = useFetch(`/api/users/${userId}`);
  
  return {
    user: data,
    loading,
    error,
  };
}

// 3. 고수준: 현재 사용자 비즈니스 로직 Hook
function useCurrentUser() {
  const { userId } = useAuth(); // 인증 Hook에서 userId 가져오기
  const { user, loading, error } = useUserQuery(userId);
  
  return {
    currentUser: user,
    isLoading: loading,
    error,
    isAuthenticated: !!user,
  };
}
```

## 사용 예시

```tsx
// 컴포넌트에서 고수준 Hook 사용
function UserDashboard() {
  const { currentUser, isLoading, isAuthenticated } = useCurrentUser();

  if (isLoading) return <Spinner />;
  if (!isAuthenticated) return <LoginPrompt />;
  
  return <div>Welcome, {currentUser.name}!</div>;
}
```

## 장점

1. **명확한 책임 분리**: 각 계층이 명확한 역할을 가짐
2. **재사용성**: 저수준 Hook을 다양한 맥락에서 재사용 가능
3. **유지보수성**: 변경 사항이 해당 계층에만 영향
4. **테스트 용이성**: 각 계층을 독립적으로 테스트 가능

## 주의사항

- 과도한 계층화는 오히려 복잡도를 증가시킬 수 있음
- 실제 재사용이 필요한 경우에만 계층화를 고려
- 각 계층의 책임 범위를 명확히 정의
