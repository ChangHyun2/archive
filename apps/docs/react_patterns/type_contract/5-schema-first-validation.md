# Schema-first Validation (zod/yup) + 타입 추론

## 개요

런타임 검증 스키마를 단일 진실로 두고 `z.infer<>`로 타입 자동 생성하는 패턴입니다. 서버/클라 계약 불일치 방지에 강력합니다.

## 문제점

타입과 검증 로직이 분리되어 있으면 불일치가 발생할 수 있습니다:

```typescript
// ❌ 나쁜 예
type User = {
  name: string;
  email: string;
  age: number;
};

function validateUser(data: unknown): User {
  // 타입 정의와 검증 로직이 분리되어 있음
  if (typeof data !== 'object' || data === null) {
    throw new Error('Invalid user');
  }
  const user = data as User;
  // 검증 로직이 불완전하거나 타입과 불일치할 수 있음
  return user;
}
```

## 해결 방법: Schema-first 접근

Zod를 사용한 예시:

```typescript
// ✅ 좋은 예: 스키마를 단일 진실로 사용
import { z } from 'zod';

// 1. 스키마 정의 (단일 진실)
const UserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  age: z.number().int().positive(),
});

// 2. 타입 자동 추론
type User = z.infer<typeof UserSchema>;
// 결과: { name: string; email: string; age: number; }

// 3. 런타임 검증
function validateUser(data: unknown): User {
  return UserSchema.parse(data); // 검증 실패 시 자동으로 에러 발생
}

// 4. 안전한 파싱 (에러 처리)
function safeValidateUser(data: unknown) {
  const result = UserSchema.safeParse(data);
  if (result.success) {
    return result.data; // 타입: User
  } else {
    return null; // 또는 에러 처리
  }
}
```

## API 응답 검증

```typescript
// ✅ 서버 응답 검증
const UserResponseSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  createdAt: z.string().datetime(),
});

type UserResponse = z.infer<typeof UserResponseSchema>;

async function fetchUser(id: number): Promise<UserResponse> {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  
  // 런타임 검증으로 서버/클라 계약 불일치 방지
  return UserResponseSchema.parse(data);
}
```

## 폼 검증

```typescript
// ✅ 폼 스키마
const LoginFormSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  rememberMe: z.boolean().optional(),
});

type LoginForm = z.infer<typeof LoginFormSchema>;

function LoginForm() {
  const [formData, setFormData] = useState<Partial<LoginForm>>({});
  const [errors, setErrors] = useState<Record<string, string>>({});

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    const result = LoginFormSchema.safeParse(formData);
    if (result.success) {
      // result.data는 LoginForm 타입
      submitLogin(result.data);
    } else {
      // 에러 메시지 추출
      const fieldErrors: Record<string, string> = {};
      result.error.errors.forEach((err) => {
        if (err.path[0]) {
          fieldErrors[err.path[0].toString()] = err.message;
        }
      });
      setErrors(fieldErrors);
    }
  };

  // ...
}
```

## React Hook Form과 통합

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

const FormSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});

type FormData = z.infer<typeof FormSchema>;

function MyForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormData>({
    resolver: zodResolver(FormSchema), // 스키마로 검증
  });

  const onSubmit = (data: FormData) => {
    // data는 FormData 타입으로 안전하게 추론됨
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} />
      {errors.name && <span>{errors.name.message}</span>}
      
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

## 복잡한 스키마

```typescript
// ✅ 중첩 객체, 배열, 조건부 검증
const PostSchema = z.object({
  id: z.number(),
  title: z.string().min(1).max(100),
  content: z.string(),
  author: z.object({
    id: z.number(),
    name: z.string(),
  }),
  tags: z.array(z.string()).min(1, 'At least one tag required'),
  publishedAt: z.string().datetime().nullable(),
  metadata: z.record(z.unknown()).optional(),
});

// 조건부 검증
const CreatePostSchema = PostSchema.omit({ id: true }).extend({
  draft: z.boolean(),
}).refine(
  (data) => {
    // draft가 false면 publishedAt이 필수
    if (!data.draft && !data.publishedAt) {
      return false;
    }
    return true;
  },
  { message: 'Published posts must have a publishedAt date' }
);

type Post = z.infer<typeof PostSchema>;
type CreatePost = z.infer<typeof CreatePostSchema>;
```

## 환경 변수 검증

```typescript
// ✅ 환경 변수 검증
const EnvSchema = z.object({
  API_URL: z.string().url(),
  API_KEY: z.string().min(1),
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.string().transform(Number).pipe(z.number().int().positive()),
});

type Env = z.infer<typeof EnvSchema>;

function loadEnv(): Env {
  return EnvSchema.parse(process.env);
}

// 앱 시작 시 검증
const env = loadEnv();
```

## 장점

1. **단일 진실 원천**: 스키마가 타입과 검증의 단일 소스
2. **타입 안전성**: 자동 타입 추론으로 타입-검증 불일치 방지
3. **런타임 안전성**: 실제 데이터 검증으로 런타임 에러 방지
4. **서버/클라 계약 보장**: API 응답 검증으로 계약 불일치 조기 발견
5. **개발자 경험**: 스키마만 수정하면 타입과 검증이 자동 업데이트
