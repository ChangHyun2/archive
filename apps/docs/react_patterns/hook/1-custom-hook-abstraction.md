# Custom Hook 추상화 (useX)

## 개요

컴포넌트에서 비즈니스/데이터 로직을 분리하여 `useSomething()` 형태의 Custom Hook으로 캡슐화하는 패턴입니다.

## 핵심 원칙

- **로직 분리**: 컴포넌트에서 비즈니스/데이터 로직을 Hook으로 추출
- **UI 단순화**: UI는 "값/핸들러 사용"만 하게 만들기
- **재사용성**: 동일한 로직을 여러 컴포넌트에서 재사용 가능

## 예시

### Before (로직이 컴포넌트에 섞여 있음)

```tsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
}
```

### After (Custom Hook으로 추상화)

```tsx
// useUser.ts
function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [userId]);

  return { user, loading, error };
}

// UserProfile.tsx
function UserProfile({ userId }) {
  const { user, loading, error } = useUser(userId);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
}
```

## 장점

1. **관심사 분리**: UI 로직과 비즈니스 로직이 명확히 분리됨
2. **테스트 용이성**: Hook을 독립적으로 테스트 가능
3. **재사용성**: 여러 컴포넌트에서 동일한 로직 재사용
4. **가독성**: 컴포넌트 코드가 간결해짐

## 사용 시기

- 동일한 로직이 여러 컴포넌트에서 반복될 때
- 컴포넌트가 복잡해져서 로직을 분리하고 싶을 때
- 비즈니스 로직을 독립적으로 테스트하고 싶을 때
