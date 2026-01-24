# Selector Pattern

## 개요

store에서 필요한 조각만 읽는 선택자(selector)를 두고, 컴포넌트는 selector만 구독하는 패턴입니다. 리렌더 최소화와 결합도 감소를 달성합니다.

## 특징

- **선택적 구독**: 필요한 데이터만 선택적으로 구독
- **리렌더 최소화**: 관련 없는 상태 변경 시 불필요한 리렌더 방지
- **결합도 감소**: 컴포넌트가 store 구조에 직접 의존하지 않음

## 예시

```tsx
// Selector 정의
const selectUserById = (state, userId) => state.users.byId[userId];
const selectUserPosts = (state, userId) => 
  state.posts.allIds
    .map(id => state.posts.byId[id])
    .filter(post => post.authorId === userId);

// 컴포넌트에서 사용
function UserProfile({ userId }) {
  // 필요한 데이터만 선택적으로 구독
  const user = useSelector(state => selectUserById(state, userId));
  const posts = useSelector(state => selectUserPosts(state, userId));
  
  // user가 변경될 때만 리렌더
  // posts가 변경될 때만 리렌더
  
  return (
    <div>
      <h1>{user.name}</h1>
      <PostList posts={posts} />
    </div>
  );
}

// Reselect를 사용한 메모이제이션
import { createSelector } from 'reselect';

const selectPosts = state => state.posts;
const selectUserId = (state, userId) => userId;

const selectUserPosts = createSelector(
  [selectPosts, selectUserId],
  (posts, userId) => 
    posts.allIds
      .map(id => posts.byId[id])
      .filter(post => post.authorId === userId)
);
```

## 구현 방법

1. **Selector 함수 작성**: store에서 필요한 데이터를 추출하는 함수
2. **컴포넌트에서 사용**: useSelector, useStore 등에서 selector 호출
3. **메모이제이션**: Reselect 등으로 불필요한 재계산 방지
4. **재사용 가능한 selector**: 공통 로직은 재사용 가능한 selector로 분리

## 장점

- 불필요한 리렌더링 방지로 성능 향상
- store 구조 변경 시 selector만 수정하면 됨
- 테스트 용이 (selector 단위 테스트)
- 코드 재사용성 증가

## 단점

- 초기 설정이 복잡할 수 있음
- Reselect 같은 라이브러리 학습 필요
- 작은 앱에서는 오버엔지니어링일 수 있음
