# Server State vs Client State 분리

## 개요

서버에서 온 데이터와 클라이언트의 UI 상태를 명확히 분리하여 관리하는 패턴입니다. 서버 상태는 React Query/SWR 같은 라이브러리로, UI/로컬 상태는 useState/Zustand/Redux 등으로 분리합니다.

## 특징

- **명확한 책임 분리**: 서버 데이터와 클라이언트 상태의 관리 방식이 구분됨
- **캐싱 및 동기화**: 서버 상태는 자동 캐싱, 재시도, 동기화 기능 활용
- **UI 상태 관리**: 토글, 필터, 모달 등 로컬 상태는 간단한 상태 관리로 처리

## 예시

```tsx
// 서버 상태 - React Query 사용
function UserProfile({ userId }) {
  const { data: user, isLoading } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  // 클라이언트 상태 - useState 사용
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [selectedFilter, setSelectedFilter] = useState('all');

  return (
    <div>
      {isLoading ? <Spinner /> : <UserInfo user={user} />}
      <button onClick={() => setIsModalOpen(true)}>Open Modal</button>
      <Filter value={selectedFilter} onChange={setSelectedFilter} />
    </div>
  );
}
```

## 구현 방법

1. **서버 상태**: React Query, SWR, Apollo Client 등 사용
   - 캐싱, 재시도, 동기화 자동 처리
   - 서버 데이터의 생명주기 관리

2. **클라이언트 상태**: useState, useReducer, Zustand, Redux 등 사용
   - UI 토글, 필터, 모달 상태 등
   - 폼 입력값, 드래그 앤 드롭 상태 등

## 장점

- 각 상태의 특성에 맞는 최적의 관리 방식 적용
- 서버 상태의 복잡한 로직(캐싱, 동기화)을 라이브러리에 위임
- 코드 가독성 및 유지보수성 향상
- 테스트 용이성 증가

## 단점

- 초기 설정이 복잡할 수 있음
- 두 가지 상태 관리 방식을 동시에 이해해야 함
- 상태 관리 라이브러리 선택에 대한 학습 필요
