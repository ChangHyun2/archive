# Public API에 any/unknown 방치 금지

## 개요

외부 입력(JSON, querystring, localStorage)은 `unknown`으로 받고 스키마로 좁혀서 안전하게 사용하는 패턴입니다.

## 문제점

외부 데이터를 `any`로 받거나 타입 단언을 남용하면 런타임 에러가 발생할 수 있습니다:

```typescript
// ❌ 나쁜 예
// localStorage에서 데이터 읽기
const userData = JSON.parse(localStorage.getItem('user') || '{}');
// userData는 any 타입

// 타입 단언으로 위험하게 사용
const user: User = userData as User;
user.name.toUpperCase(); // userData에 name이 없으면 런타임 에러

// API 응답
const response = await fetch('/api/user');
const user = await response.json(); // any 타입
user.name.toUpperCase(); // 안전하지 않음

// Query string 파싱
const params = new URLSearchParams(window.location.search);
const userId = params.get('id'); // string | null
const user: User = await fetchUser(userId!); // ! 사용은 위험
```

## 해결 방법: unknown + 스키마 검증

```typescript
// ✅ 좋은 예
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
});

type User = z.infer<typeof UserSchema>;

// 1. localStorage에서 안전하게 읽기
function getUserFromStorage(): User | null {
  try {
    const raw = localStorage.getItem('user');
    if (!raw) return null;
    
    // unknown으로 받기
    const data: unknown = JSON.parse(raw);
    
    // 스키마로 검증
    const result = UserSchema.safeParse(data);
    if (result.success) {
      return result.data; // User 타입으로 좁혀짐
    }
    
    console.error('Invalid user data in storage', result.error);
    return null;
  } catch (error) {
    console.error('Failed to parse user data', error);
    return null;
  }
}

// 2. API 응답 안전하게 처리
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  
  // unknown으로 받기
  const data: unknown = await response.json();
  
  // 스키마로 검증
  const result = UserSchema.safeParse(data);
  if (!result.success) {
    throw new Error(`Invalid API response: ${result.error.message}`);
  }
  
  return result.data; // User 타입으로 좁혀짐
}

// 3. Query string 안전하게 파싱
function getUserIdFromQuery(): string | null {
  const params = new URLSearchParams(window.location.search);
  const id = params.get('id');
  
  if (!id) return null;
  
  // 간단한 검증
  if (typeof id !== 'string' || id.length === 0) {
    return null;
  }
  
  return id;
}
```

## localStorage 유틸리티

```typescript
// ✅ 타입 안전한 localStorage 유틸리티
function createStorage<T>(key: string, schema: z.ZodSchema<T>) {
  return {
    get(): T | null {
      try {
        const raw = localStorage.getItem(key);
        if (!raw) return null;
        
        const data: unknown = JSON.parse(raw);
        const result = schema.safeParse(data);
        
        if (result.success) {
          return result.data;
        }
        
        // 잘못된 데이터는 삭제
        localStorage.removeItem(key);
        return null;
      } catch {
        return null;
      }
    },
    
    set(value: T): void {
      localStorage.setItem(key, JSON.stringify(value));
    },
    
    remove(): void {
      localStorage.removeItem(key);
    },
  };
}

// 사용
const userStorage = createStorage('user', UserSchema);

const user = userStorage.get(); // User | null
if (user) {
  // user는 User 타입으로 안전하게 사용 가능
  console.log(user.name);
}

userStorage.set({ id: '1', name: 'John', email: 'john@example.com' });
```

## URL 파라미터 파싱

```typescript
// ✅ Query string 파라미터 타입 안전하게 파싱
const SearchParamsSchema = z.object({
  page: z.string().transform(Number).pipe(z.number().int().positive()).optional(),
  limit: z.string().transform(Number).pipe(z.number().int().positive()).optional(),
  search: z.string().optional(),
  sort: z.enum(['asc', 'desc']).optional(),
});

type SearchParams = z.infer<typeof SearchParamsSchema>;

function parseSearchParams(): SearchParams {
  const params = new URLSearchParams(window.location.search);
  
  // unknown 객체로 변환
  const data: unknown = Object.fromEntries(params.entries());
  
  // 스키마로 검증 및 변환
  const result = SearchParamsSchema.safeParse(data);
  
  if (result.success) {
    return result.data;
  }
  
  // 기본값 반환 또는 에러 처리
  return {};
}

// 사용
function SearchPage() {
  const params = parseSearchParams();
  // params는 SearchParams 타입으로 안전하게 사용 가능
  const page = params.page ?? 1;
  const limit = params.limit ?? 10;
}
```

## Event Handler에서 사용

```typescript
// ✅ 이벤트 데이터 안전하게 처리
const FormSubmitEventSchema = z.object({
  target: z.object({
    elements: z.record(z.any()),
  }),
});

function handleFormSubmit(event: unknown) {
  // event를 unknown으로 받기
  const result = FormSubmitEventSchema.safeParse(event);
  
  if (!result.success) {
    console.error('Invalid form submit event', result.error);
    return;
  }
  
  // 이제 안전하게 사용 가능
  const formData = new FormData(result.data.target as HTMLFormElement);
  // ...
}
```

## API 클라이언트에서 사용

```typescript
// ✅ API 응답을 항상 unknown으로 받고 검증
async function safeFetch<T>(
  url: string,
  schema: z.ZodSchema<T>
): Promise<T> {
  const response = await fetch(url);
  
  if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`);
  }
  
  // unknown으로 받기
  const data: unknown = await response.json();
  
  // 스키마로 검증
  const result = schema.safeParse(data);
  if (!result.success) {
    throw new Error(`Invalid response format: ${result.error.message}`);
  }
  
  return result.data; // T 타입으로 좁혀짐
}

// 사용
const user = await safeFetch('/api/user', UserSchema); // User 타입
const posts = await safeFetch('/api/posts', z.array(PostSchema)); // Post[] 타입
```

## 환경 변수 처리

```typescript
// ✅ 환경 변수도 unknown으로 받고 검증
const EnvSchema = z.object({
  API_URL: z.string().url(),
  API_KEY: z.string().min(1),
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.string().transform(Number).pipe(z.number().int().positive()),
});

type Env = z.infer<typeof EnvSchema>;

function loadEnv(): Env {
  // process.env는 unknown으로 취급
  const data: unknown = process.env;
  
  const result = EnvSchema.safeParse(data);
  if (!result.success) {
    throw new Error(`Invalid environment variables: ${result.error.message}`);
  }
  
  return result.data;
}

// 앱 시작 시 검증
const env = loadEnv();
```

## 장점

1. **런타임 안전성**: 외부 데이터를 검증하여 런타임 에러 방지
2. **타입 안전성**: unknown에서 스키마 검증을 통해 안전한 타입으로 좁힘
3. **에러 조기 발견**: 잘못된 데이터를 사용하기 전에 발견
4. **자기 문서화**: 스키마가 데이터 형식을 명확히 정의
5. **유지보수성**: 스키마만 수정하면 타입과 검증이 자동 업데이트
