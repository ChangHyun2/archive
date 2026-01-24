# Progressive Rendering (Skeleton/Placeholder)

## 개요

모든 데이터를 기다렸다가 한 번에 렌더링하는 대신, 데이터가 준비되는 대로 단계적으로 화면을 보여주는 패턴입니다. Skeleton UI나 Placeholder를 사용하여 사용자 경험을 개선합니다.

## 핵심 원칙

- **점진적 렌더링**: 데이터가 준비되는 대로 단계적으로 표시
- **시각적 피드백**: 로딩 중임을 명확히 알려주는 Skeleton/Placeholder
- **체감 성능 향상**: 실제 로딩 시간은 같아도 사용자는 더 빠르게 느낌

## 사용 방법

### 기본 Skeleton UI

```tsx
// ❌ 나쁜 예: 모든 데이터를 기다렸다가 한 번에 렌더링
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState(null);
  const [friends, setFriends] = useState(null);
  
  useEffect(() => {
    Promise.all([
      fetchUser(userId),
      fetchPosts(userId),
      fetchFriends(userId)
    ]).then(([userData, postsData, friendsData]) => {
      setUser(userData);
      setPosts(postsData);
      setFriends(friendsData);
    });
  }, [userId]);
  
  if (!user || !posts || !friends) {
    return <div>Loading...</div>; // 모든 데이터를 기다림
  }
  
  return (
    <div>
      <UserInfo user={user} />
      <PostsList posts={posts} />
      <FriendsList friends={friends} />
    </div>
  );
}

// ✅ 좋은 예: 점진적 렌더링
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState(null);
  const [friends, setFriends] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
    fetchPosts(userId).then(setPosts);
    fetchFriends(userId).then(setFriends);
  }, [userId]);
  
  return (
    <div>
      {user ? (
        <UserInfo user={user} />
      ) : (
        <UserInfoSkeleton />
      )}
      
      {posts ? (
        <PostsList posts={posts} />
      ) : (
        <PostsListSkeleton />
      )}
      
      {friends ? (
        <FriendsList friends={friends} />
      ) : (
        <FriendsListSkeleton />
      )}
    </div>
  );
}
```

### Skeleton 컴포넌트 구현

```tsx
// Skeleton 컴포넌트
function Skeleton({ width, height, className }) {
  return (
    <div
      className={`skeleton ${className}`}
      style={{ width, height }}
    />
  );
}

function UserInfoSkeleton() {
  return (
    <div className="user-info-skeleton">
      <Skeleton width={100} height={100} className="avatar" />
      <Skeleton width={200} height={24} className="name" />
      <Skeleton width={150} height={16} className="email" />
    </div>
  );
}

function PostsListSkeleton() {
  return (
    <div className="posts-skeleton">
      {[1, 2, 3].map(i => (
        <div key={i} className="post-skeleton">
          <Skeleton width="100%" height={200} />
          <Skeleton width={150} height={20} />
        </div>
      ))}
    </div>
  );
}
```

### CSS 애니메이션

```css
.skeleton {
  background: linear-gradient(
    90deg,
    #f0f0f0 25%,
    #e0e0e0 50%,
    #f0f0f0 75%
  );
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
}

@keyframes loading {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}
```

### Suspense와 함께 사용

```tsx
import { Suspense } from 'react';

function UserProfile({ userId }) {
  return (
    <div>
      <Suspense fallback={<UserInfoSkeleton />}>
        <UserInfo userId={userId} />
      </Suspense>
      
      <Suspense fallback={<PostsListSkeleton />}>
        <PostsList userId={userId} />
      </Suspense>
      
      <Suspense fallback={<FriendsListSkeleton />}>
        <FriendsList userId={userId} />
      </Suspense>
    </div>
  );
}
```

### 우선순위 기반 렌더링

```tsx
// ✅ 중요한 데이터를 먼저 표시
function Dashboard() {
  const [criticalData, setCriticalData] = useState(null);
  const [secondaryData, setSecondaryData] = useState(null);
  
  useEffect(() => {
    // 중요한 데이터 먼저 로드
    fetchCriticalData().then(setCriticalData);
    
    // 그 다음 덜 중요한 데이터
    fetchCriticalData().then(() => {
      fetchSecondaryData().then(setSecondaryData);
    });
  }, []);
  
  return (
    <div>
      {criticalData ? (
        <CriticalSection data={criticalData} />
      ) : (
        <CriticalSectionSkeleton />
      )}
      
      {secondaryData ? (
        <SecondarySection data={secondaryData} />
      ) : (
        <SecondarySectionSkeleton />
      )}
    </div>
  );
}
```

## Best Practices

1. **레이아웃 유지**: Skeleton이 실제 콘텐츠와 같은 크기/레이아웃 유지
2. **애니메이션**: 시각적 피드백을 위한 부드러운 로딩 애니메이션
3. **우선순위**: 중요한 콘텐츠를 먼저 표시
4. **에러 처리**: 로딩 실패 시 적절한 에러 상태 표시
5. **접근성**: 스크린 리더 사용자를 위한 적절한 aria-label

## 주의사항

- Skeleton이 실제 콘텐츠와 크기가 다르면 레이아웃 시프트 발생
- 너무 많은 Skeleton은 오히려 사용자 경험을 해칠 수 있음
- 실제 데이터 구조와 일치하도록 설계
