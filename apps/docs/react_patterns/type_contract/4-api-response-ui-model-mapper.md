# API Response → UI Model 변환 계층(Mapper) + 타입 고정

## 개요

서버 DTO 타입을 그대로 UI에서 쓰지 말고, 변환된 "앱 도메인 타입"을 기준으로 UI 개발하는 패턴입니다.

## 문제점

서버 API 응답을 그대로 UI에서 사용하면 여러 문제가 발생합니다:

```typescript
// ❌ 나쁜 예
// 서버 응답 타입을 그대로 사용
type UserResponse = {
  id: number;
  first_name: string;
  last_name: string;
  created_at: string; // ISO string
  profile_image_url: string | null;
};

function UserProfile({ user }: { user: UserResponse }) {
  // UI에서 직접 변환 로직이 섞임
  const fullName = `${user.first_name} ${user.last_name}`;
  const createdAt = new Date(user.created_at);
  const imageUrl = user.profile_image_url || '/default-avatar.png';
  
  // 문제점:
  // 1. 변환 로직이 컴포넌트에 분산
  // 2. 서버 스키마 변경 시 모든 컴포넌트 수정 필요
  // 3. UI 모델과 서버 모델이 혼재
}
```

## 해결 방법

Mapper 계층을 통한 변환:

```typescript
// ✅ 좋은 예
// 1. 서버 DTO 타입 정의 (서버 계약)
type UserDTO = {
  id: number;
  first_name: string;
  last_name: string;
  created_at: string;
  profile_image_url: string | null;
};

// 2. UI 도메인 모델 정의 (앱 내부 사용)
type User = {
  id: string; // number → string 변환
  fullName: string; // first_name + last_name 조합
  createdAt: Date; // string → Date 변환
  avatarUrl: string; // null 처리 포함
};

// 3. Mapper 함수 (변환 계층)
function mapUserDTOToUser(dto: UserDTO): User {
  return {
    id: String(dto.id),
    fullName: `${dto.first_name} ${dto.last_name}`,
    createdAt: new Date(dto.created_at),
    avatarUrl: dto.profile_image_url || '/default-avatar.png',
  };
}

// 4. UI 컴포넌트는 도메인 모델만 사용
function UserProfile({ user }: { user: User }) {
  return (
    <div>
      <img src={user.avatarUrl} alt={user.fullName} />
      <h2>{user.fullName}</h2>
      <p>Joined: {user.createdAt.toLocaleDateString()}</p>
    </div>
  );
}

// 5. API 호출 시점에 변환
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const dto: UserDTO = await response.json();
  return mapUserDTOToUser(dto);
}
```

## 복잡한 예시

```typescript
// 서버 응답
type PostDTO = {
  id: number;
  title: string;
  content: string;
  author_id: number;
  author_name: string;
  tags: string[];
  published_at: string | null;
  view_count: number;
};

// UI 모델
type Post = {
  id: string;
  title: string;
  content: string;
  author: {
    id: string;
    name: string;
  };
  tags: Tag[];
  publishedAt: Date | null;
  viewCount: number;
  isPublished: boolean;
};

type Tag = {
  id: string;
  name: string;
  slug: string;
};

// Mapper
function mapPostDTOToPost(dto: PostDTO, tags: Tag[]): Post {
  return {
    id: String(dto.id),
    title: dto.title,
    content: dto.content,
    author: {
      id: String(dto.author_id),
      name: dto.author_name,
    },
    tags: tags.filter(tag => dto.tags.includes(tag.slug)),
    publishedAt: dto.published_at ? new Date(dto.published_at) : null,
    viewCount: dto.view_count,
    isPublished: dto.published_at !== null,
  };
}
```

## Mapper 유틸리티

```typescript
// 재사용 가능한 Mapper 유틸리티
type Mapper<TSource, TTarget> = (source: TSource) => TTarget;

// 배열 변환
function mapArray<TSource, TTarget>(
  items: TSource[],
  mapper: Mapper<TSource, TTarget>
): TTarget[] {
  return items.map(mapper);
}

// 옵셔널 변환
function mapOptional<TSource, TTarget>(
  item: TSource | null | undefined,
  mapper: Mapper<TSource, TTarget>
): TTarget | null {
  return item ? mapper(item) : null;
}

// 사용 예시
const users: User[] = mapArray(userDTOs, mapUserDTOToUser);
const user: User | null = mapOptional(userDTO, mapUserDTOToUser);
```

## React Hook에서 사용

```typescript
function useUser(id: number) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(id)
      .then((user) => {
        setUser(user); // 이미 변환된 User 타입
        setLoading(false);
      })
      .catch(() => setLoading(false));
  }, [id]);

  return { user, loading };
}

// 컴포넌트에서 사용
function UserComponent({ userId }: { userId: number }) {
  const { user, loading } = useUser(userId);

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;

  // user는 항상 User 타입 (변환 완료)
  return <UserProfile user={user} />;
}
```

## 장점

1. **관심사 분리**: 서버 계약과 UI 모델 분리
2. **변경 내성**: 서버 스키마 변경 시 Mapper만 수정
3. **타입 안전성**: UI는 항상 변환된 안전한 타입 사용
4. **재사용성**: 변환 로직 중앙화로 재사용 용이
5. **테스트 용이성**: Mapper 함수를 독립적으로 테스트 가능
