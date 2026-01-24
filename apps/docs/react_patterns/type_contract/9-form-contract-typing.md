# Form 계약 타입화

## 개요

폼 값 타입, 서버 요청 타입, 검증 스키마를 한 흐름으로 묶어 "한 군데만 바꾸면" 전체가 따라오게 하는 패턴입니다.

## 문제점

폼 관련 타입과 로직이 분산되어 있으면 불일치가 발생합니다:

```typescript
// ❌ 나쁜 예
// 폼 값 타입
type LoginForm = {
  email: string;
  password: string;
};

// 서버 요청 타입 (폼과 분리됨)
type LoginRequest = {
  email: string;
  password: string;
  // 실수로 필드명이 다를 수 있음
};

// 검증 로직 (타입과 분리됨)
function validateLoginForm(form: LoginForm): boolean {
  // 검증 로직이 타입과 불일치할 수 있음
  return form.email.includes('@') && form.password.length >= 8;
}

// 문제점:
// 1. 폼 타입, 요청 타입, 검증 로직이 분리되어 불일치 가능
// 2. 한 곳을 수정하면 다른 곳도 수정해야 함
// 3. 타입 안전성이 보장되지 않음
```

## 해결 방법: 단일 소스에서 타입 추론

```typescript
// ✅ 좋은 예: Zod 스키마를 단일 진실로 사용
import { z } from 'zod';

// 1. 폼 스키마 정의 (단일 진실)
const LoginFormSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

// 2. 모든 타입을 스키마에서 추론
type LoginForm = z.infer<typeof LoginFormSchema>;
type LoginRequest = z.infer<typeof LoginFormSchema>; // 동일한 타입 사용

// 3. 검증도 스키마 사용
function validateLoginForm(data: unknown): LoginForm {
  return LoginFormSchema.parse(data);
}

// 4. React Hook Form과 통합
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<LoginForm>({
    resolver: zodResolver(LoginFormSchema), // 스키마로 검증
  });

  const onSubmit = async (data: LoginForm) => {
    // data는 LoginForm 타입
    // 서버로 전송 시 LoginRequest와 동일한 타입
    await submitLogin(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input {...register('password')} type="password" />
      {errors.password && <span>{errors.password.message}</span>}
      
      <button type="submit">Login</button>
    </form>
  );
}
```

## 서버 요청 타입 변환

```typescript
// ✅ 폼 타입과 서버 요청 타입이 다른 경우
const CreatePostFormSchema = z.object({
  title: z.string().min(1),
  content: z.string(),
  tags: z.array(z.string()),
  isDraft: z.boolean(),
});

type CreatePostForm = z.infer<typeof CreatePostFormSchema>;

// 서버 요청 타입 (필드명이나 구조가 다를 수 있음)
const CreatePostRequestSchema = CreatePostFormSchema.extend({
  // 추가 필드
  authorId: z.string(),
}).transform((data) => ({
  // 서버가 기대하는 형식으로 변환
  title: data.title,
  body: data.content, // content → body 변환
  tag_list: data.tags, // tags → tag_list 변환
  draft: data.isDraft, // isDraft → draft 변환
  author_id: data.authorId,
}));

type CreatePostRequest = z.infer<typeof CreatePostRequestSchema>;

// 사용
function CreatePostForm() {
  const form = useForm<CreatePostForm>({
    resolver: zodResolver(CreatePostFormSchema),
  });

  const onSubmit = async (formData: CreatePostForm) => {
    // 폼 데이터를 서버 요청 형식으로 변환
    const request = CreatePostRequestSchema.parse({
      ...formData,
      authorId: getCurrentUserId(),
    });
    
    await createPost(request);
  };

  // ...
}
```

## 복잡한 폼 예시

```typescript
// ✅ 중첩된 폼 구조
const AddressSchema = z.object({
  street: z.string(),
  city: z.string(),
  zipCode: z.string().regex(/^\d{5}$/),
});

const UserFormSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  address: AddressSchema,
  phoneNumbers: z.array(z.string()),
});

type UserForm = z.infer<typeof UserFormSchema>;

// 서버 요청 타입 (평탄화)
const CreateUserRequestSchema = UserFormSchema.transform((data) => ({
  name: data.name,
  email: data.email,
  street: data.address.street,
  city: data.address.city,
  zip_code: data.address.zipCode,
  phone_numbers: data.phoneNumbers,
}));

type CreateUserRequest = z.infer<typeof CreateUserRequestSchema>;
```

## 조건부 검증

```typescript
// ✅ 조건부 필드 검증
const RegistrationFormSchema = z
  .object({
    email: z.string().email(),
    password: z.string().min(8),
    confirmPassword: z.string(),
    subscribe: z.boolean(),
    newsletterEmail: z.string().email().optional(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ['confirmPassword'],
  })
  .refine(
    (data) => {
      // subscribe가 true면 newsletterEmail 필수
      if (data.subscribe && !data.newsletterEmail) {
        return false;
      }
      return true;
    },
    {
      message: 'Newsletter email is required when subscribing',
      path: ['newsletterEmail'],
    }
  );

type RegistrationForm = z.infer<typeof RegistrationFormSchema>;
```

## 재사용 가능한 폼 컴포넌트

```typescript
// ✅ 제네릭 폼 컴포넌트
type FormProps<T extends z.ZodTypeAny> = {
  schema: T;
  onSubmit: (data: z.infer<T>) => Promise<void>;
  defaultValues?: Partial<z.infer<T>>;
  children: (methods: UseFormReturn<z.infer<T>>) => React.ReactNode;
};

function TypedForm<T extends z.ZodTypeAny>({
  schema,
  onSubmit,
  defaultValues,
  children,
}: FormProps<T>) {
  const methods = useForm<z.infer<T>>({
    resolver: zodResolver(schema),
    defaultValues,
  });

  return (
    <form onSubmit={methods.handleSubmit(onSubmit)}>
      {children(methods)}
    </form>
  );
}

// 사용
<TypedForm
  schema={LoginFormSchema}
  onSubmit={async (data) => {
    // data는 LoginForm 타입
    await submitLogin(data);
  }}
>
  {({ register, formState: { errors } }) => (
    <>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input {...register('password')} type="password" />
      {errors.password && <span>{errors.password.message}</span>}
      
      <button type="submit">Login</button>
    </>
  )}
</TypedForm>
```

## 장점

1. **단일 진실 원천**: 스키마 하나만 수정하면 모든 타입과 검증이 자동 업데이트
2. **타입 안전성**: 폼 값, 요청 타입, 검증이 모두 일치
3. **유지보수성**: 한 곳만 수정하면 전체가 따라옴
4. **일관성**: 폼과 서버 요청 타입이 항상 동기화
5. **개발자 경험**: 타입 추론으로 자동완성과 타입 체크 지원
