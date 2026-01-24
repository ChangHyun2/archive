# Stale-While-Revalidate (SWR)

## 개요

캐시된 데이터를 즉시 보여주고, 뒤에서 재검증/갱신하는 패턴입니다. 로딩 UX 개선과 네트워크 비용/지연을 제품 관점에서 최적화합니다.

## 특징

- **즉시 표시**: 캐시된 데이터를 먼저 보여줌
- **백그라운드 갱신**: 사용자 경험을 방해하지 않고 데이터 갱신
- **자동 재검증**: 포커스, 네트워크 재연결 등 이벤트 시 자동 갱신

## 예시

```tsx
// SWR 라이브러리 사용
import useSWR from 'swr';

function UserProfile({ userId }) {
  const { data, error, isLoading, isValidating } = useSWR(
    `/api/users/${userId}`,
    fetcher,
    {
      revalidateOnFocus: true, // 포커스 시 재검증
      revalidateOnReconnect: true, // 네트워크 재연결 시 재검증
      refreshInterval: 5000, // 5초마다 자동 갱신
    }
  );
  
  // 캐시된 데이터가 있으면 즉시 표시
  // 백그라운드에서 갱신 중이면 isValidating이 true
  if (error) return <div>Error loading user</div>;
  if (!data && isLoading) return <div>Loading...</div>;
  
  return (
    <div>
      <h1>{data.name}</h1>
      {isValidating && <span>Updating...</span>}
    </div>
  );
}

// React Query 사용
import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }) {
  const { data, isLoading, isFetching } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5000, // 5초간 fresh로 간주
    cacheTime: 10 * 60 * 1000, // 10분간 캐시 유지
    refetchOnWindowFocus: true,
    refetchOnReconnect: true,
  });
  
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <div>
      <h1>{data.name}</h1>
      {isFetching && <span>Refreshing...</span>}
    </div>
  );
}
```

## 구현 방법

1. **캐시 우선 표시**: 캐시된 데이터가 있으면 즉시 표시
2. **백그라운드 갱신**: 사용자 경험을 방해하지 않고 데이터 갱신
3. **재검증 전략 설정**: 포커스, 인터벌, 네트워크 재연결 등에 대한 재검증 설정
4. **로딩 상태 구분**: 초기 로딩과 재검증 로딩을 구분하여 표시

## 장점

- 빠른 초기 로딩 (캐시 활용)
- 자동 데이터 동기화
- 네트워크 비용 최적화
- 사용자 경험 향상 (로딩 시간 최소화)

## 단점

- 일시적으로 stale 데이터 표시 가능
- 캐시 전략 설정 복잡도
- 라이브러리 의존성 필요
- 캐시 무효화 관리 필요
