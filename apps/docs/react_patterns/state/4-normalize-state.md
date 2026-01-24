# Normalize State (정규화)

## 개요

리스트/엔티티를 `{ byId, allIds }` 형태로 관리하여 업데이트/참조를 안정적으로 하는 패턴입니다. 특히 댓글/상품/유저처럼 관계가 얽힌 경우 필수입니다.

## 특징

- **정규화된 구조**: 중첩된 객체 대신 평면적 구조로 저장
- **참조 기반 접근**: ID를 통한 안정적인 엔티티 접근
- **업데이트 효율성**: 특정 엔티티만 업데이트 가능

## 예시

```tsx
// ❌ 나쁜 예: 중첩된 구조
const state = {
  posts: [
    { id: 1, title: 'Post 1', author: { id: 1, name: 'John' } },
    { id: 2, title: 'Post 2', author: { id: 1, name: 'John' } },
  ]
};
// John의 이름을 바꾸려면 모든 post를 순회해야 함

// ✅ 좋은 예: 정규화된 구조
const state = {
  posts: {
    byId: {
      '1': { id: 1, title: 'Post 1', authorId: 1 },
      '2': { id: 2, title: 'Post 2', authorId: 1 },
    },
    allIds: ['1', '2']
  },
  users: {
    byId: {
      '1': { id: 1, name: 'John' }
    },
    allIds: ['1']
  }
};

// 사용 예시
function PostList() {
  const posts = useSelector(state => 
    state.posts.allIds.map(id => ({
      ...state.posts.byId[id],
      author: state.users.byId[state.posts.byId[id].authorId]
    }))
  );
  
  return (
    <div>
      {posts.map(post => (
        <Post key={post.id} post={post} />
      ))}
    </div>
  );
}
```

## 구현 방법

1. **엔티티 분리**: 각 엔티티 타입별로 별도 저장소 구성
2. **byId 객체**: ID를 키로 하는 객체로 엔티티 저장
3. **allIds 배열**: 순서가 필요한 경우 ID 배열 유지
4. **참조 연결**: 중첩 대신 ID 참조로 관계 표현

## 장점

- 특정 엔티티 업데이트가 간단하고 효율적
- 중복 데이터 제거 (예: 같은 유저 정보)
- 일관된 데이터 구조
- 관계가 복잡한 경우에도 관리 용이

## 단점

- 초기 구조 설계가 복잡할 수 있음
- 데이터 조회 시 조인 로직 필요
- 작은 규모의 앱에서는 오버엔지니어링일 수 있음
