# Branded Types / Nominal Typing 흉내

## 개요

`UserId`와 `PostId` 같은 "같은 string"을 섞어 쓰는 실수를 방지하는 패턴입니다. TypeScript는 구조적 타입 시스템이지만, Branded Types를 통해 명목적 타입처럼 동작하게 만들 수 있습니다.

## 문제점

같은 기본 타입을 사용하는 값들을 실수로 섞어 쓸 수 있습니다:

```typescript
// ❌ 나쁜 예
type UserId = string;
type PostId = string;

function getUserPosts(userId: UserId): Post[] {
  // ...
}

function getPost(postId: PostId): Post {
  // ...
}

// 문제: UserId와 PostId가 모두 string이므로 서로 할당 가능
const userId: UserId = 'user-123';
const postId: PostId = userId; // ❌ 타입 에러가 발생하지 않음

getUserPosts(postId); // ❌ 실수로 PostId를 UserId로 전달해도 컴파일 통과
getPost(userId); // ❌ 실수로 UserId를 PostId로 전달해도 컴파일 통과
```

## 해결 방법: Branded Types

```typescript
// ✅ 좋은 예
// Branded Type 정의
type UserId = string & { readonly __brand: 'UserId' };
type PostId = string & { readonly __brand: 'PostId' };
type Email = string & { readonly __brand: 'Email' };

// Branded Type 생성 헬퍼
function createUserId(id: string): UserId {
  return id as UserId;
}

function createPostId(id: string): PostId {
  return id as PostId;
}

function createEmail(email: string): Email {
  if (!email.includes('@')) {
    throw new Error('Invalid email');
  }
  return email as Email;
}

// 사용 예시
const userId: UserId = createUserId('user-123');
const postId: PostId = createPostId('post-456');

getUserPosts(userId); // ✅
getUserPosts(postId); // ❌ 컴파일 에러: Type 'PostId' is not assignable to type 'UserId'

getPost(postId); // ✅
getPost(userId); // ❌ 컴파일 에러: Type 'UserId' is not assignable to type 'PostId'
```

## 더 실용적인 예시

```typescript
// ✅ API 응답에서 Branded Type 생성
type UserId = string & { readonly __brand: 'UserId' };
type PostId = string & { readonly __brand: 'PostId' };

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
});

type UserDTO = z.infer<typeof UserSchema>;

// Mapper에서 Branded Type으로 변환
function mapUserDTOToUser(dto: UserDTO): User {
  return {
    id: dto.id as UserId, // Branded Type으로 변환
    name: dto.name,
  };
}

type User = {
  id: UserId; // Branded Type 사용
  name: string;
};

// 함수 시그니처에서 Branded Type 사용
function getUserPosts(userId: UserId): Post[] {
  // userId는 항상 UserId 타입만 받음
  return fetch(`/api/users/${userId}/posts`).then(/* ... */);
}
```

## Generic Branded Type

```typescript
// ✅ 재사용 가능한 Branded Type 유틸리티
type Brand<T, B> = T & { readonly __brand: B };

type UserId = Brand<string, 'UserId'>;
type PostId = Brand<string, 'PostId'>;
type Email = Brand<string, 'Email'>;
type Timestamp = Brand<number, 'Timestamp'>;

// 사용
const userId: UserId = 'user-123' as UserId;
const timestamp: Timestamp = Date.now() as Timestamp;
```

## 실제 사용 케이스

```typescript
// ✅ API 클라이언트에서 사용
type UserId = Brand<string, 'UserId'>;
type PostId = Brand<string, 'PostId'>;

class ApiClient {
  async getUser(id: UserId): Promise<User> {
    return fetch(`/api/users/${id}`).then(/* ... */);
  }

  async getPost(id: PostId): Promise<Post> {
    return fetch(`/api/posts/${id}`).then(/* ... */);
  }

  async getUserPosts(userId: UserId): Promise<Post[]> {
    return fetch(`/api/users/${userId}/posts`).then(/* ... */);
  }
}

// 사용
const api = new ApiClient();
const userId = 'user-123' as UserId;
const postId = 'post-456' as PostId;

await api.getUser(userId); // ✅
await api.getUser(postId); // ❌ 컴파일 에러
await api.getPost(postId); // ✅
await api.getUserPosts(userId); // ✅
```

## React 컴포넌트에서 사용

```typescript
// ✅ 컴포넌트 Props에서 사용
type UserId = Brand<string, 'UserId'>;

type UserProfileProps = {
  userId: UserId; // 일반 string이 아닌 UserId만 받음
};

function UserProfile({ userId }: UserProfileProps) {
  // userId는 항상 UserId 타입
  const { user } = useUser(userId);
  
  return <div>{user.name}</div>;
}

// 사용
<UserProfile userId={'user-123' as UserId} /> // ✅
<UserProfile userId={'post-456' as PostId} /> // ❌ 컴파일 에러
```

## Zod와 통합

```typescript
import { z } from 'zod';

type UserId = Brand<string, 'UserId'>;

// Zod 스키마에서 Branded Type 생성
const UserIdSchema = z.string().transform((val): UserId => val as UserId);

const UserSchema = z.object({
  id: UserIdSchema,
  name: z.string(),
});

// 사용
const user = UserSchema.parse({ id: 'user-123', name: 'John' });
// user.id는 UserId 타입
```

## 주의사항

```typescript
// ⚠️ Branded Type은 런타임에는 일반 타입과 동일
const userId: UserId = 'user-123' as UserId;
const regularString: string = userId; // ✅ 할당 가능 (구조적으로 동일)

// 하지만 반대는 안 됨
const regularString = 'user-123';
const userId: UserId = regularString; // ❌ 컴파일 에러

// 명시적 변환 필요
const userId: UserId = regularString as UserId; // ✅
```

## 장점

1. **타입 안전성**: 같은 기본 타입을 가진 값들을 실수로 섞어 쓰는 것 방지
2. **의도 명확성**: 타입만 봐도 어떤 종류의 값인지 명확
3. **리팩토링 안전성**: ID 타입 변경 시 모든 사용처에서 에러 발생
4. **자기 문서화**: 코드만 봐도 값의 의미를 알 수 있음
