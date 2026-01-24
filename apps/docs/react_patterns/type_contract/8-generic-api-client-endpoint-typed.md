# Generic API 클라이언트(Endpoint 타입 안전)

## 개요

`client.get<Resp>('/users')` 수준이 아니라, endpoint별 요청/응답/에러를 타입으로 고정하는 패턴입니다.

## 문제점

일반적인 API 클라이언트는 타입 안전성이 부족합니다:

```typescript
// ❌ 나쁜 예
class ApiClient {
  async get<T>(url: string): Promise<T> {
    const response = await fetch(url);
    return response.json();
  }
}

// 사용 시 타입을 매번 지정해야 하고, endpoint와 타입이 분리됨
const user = await client.get<User>('/api/users/123');
const post = await client.get<Post>('/api/posts/456');

// 문제점:
// 1. endpoint 문자열과 타입이 분리되어 불일치 가능
// 2. 요청 파라미터 타입 안전성 없음
// 3. 에러 타입이 명시되지 않음
```

## 해결 방법: Endpoint 타입 정의

```typescript
// ✅ 좋은 예
// 1. Endpoint 정의
type Endpoints = {
  'GET /api/users/:id': {
    request: { id: string };
    response: User;
    error: 'USER_NOT_FOUND' | 'UNAUTHORIZED';
  };
  
  'GET /api/posts': {
    request: { page?: number; limit?: number };
    response: Post[];
    error: 'UNAUTHORIZED' | 'SERVER_ERROR';
  };
  
  'POST /api/posts': {
    request: { title: string; content: string };
    response: Post;
    error: 'VALIDATION_ERROR' | 'UNAUTHORIZED';
  };
  
  'GET /api/posts/:id': {
    request: { id: string };
    response: Post;
    error: 'POST_NOT_FOUND' | 'UNAUTHORIZED';
  };
};

// 2. 타입 안전한 API 클라이언트
type Endpoint = keyof Endpoints;

class TypedApiClient {
  async get<E extends Endpoint>(
    endpoint: E,
    params: Endpoints[E]['request']
  ): Promise<Endpoints[E]['response']> {
    // endpoint에서 실제 URL 생성
    const url = this.buildUrl(endpoint, params);
    
    const response = await fetch(url);
    
    if (!response.ok) {
      // 에러 타입도 안전하게 처리
      throw this.handleError<Endpoints[E]['error']>(response);
    }
    
    return response.json();
  }

  async post<E extends Endpoint>(
    endpoint: E,
    body: Endpoints[E]['request']
  ): Promise<Endpoints[E]['response']> {
    const url = this.buildUrl(endpoint, body);
    
    const response = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(body),
    });
    
    if (!response.ok) {
      throw this.handleError<Endpoints[E]['error']>(response);
    }
    
    return response.json();
  }

  private buildUrl(endpoint: Endpoint, params: Record<string, unknown>): string {
    let url = endpoint;
    
    // :id 같은 파라미터 치환
    Object.entries(params).forEach(([key, value]) => {
      url = url.replace(`:${key}`, String(value));
    });
    
    // 쿼리 파라미터 추가
    const queryParams = new URLSearchParams();
    Object.entries(params).forEach(([key, value]) => {
      if (!url.includes(`:${key}`) && value !== undefined) {
        queryParams.append(key, String(value));
      }
    });
    
    const query = queryParams.toString();
    return query ? `${url}?${query}` : url;
  }

  private handleError<E extends string>(response: Response): AppError {
    // 실제 구현에서는 response에서 에러 코드 추출
    return createError.serverError(response.status);
  }
}

// 3. 사용
const client = new TypedApiClient();

// ✅ 타입 안전: endpoint와 타입이 자동으로 연결됨
const user = await client.get('GET /api/users/:id', { id: '123' });
// user 타입: User

const posts = await client.get('GET /api/posts', { page: 1, limit: 10 });
// posts 타입: Post[]

const newPost = await client.post('POST /api/posts', {
  title: 'Hello',
  content: 'World',
});
// newPost 타입: Post

// ❌ 컴파일 에러: 잘못된 endpoint
await client.get('GET /api/invalid', {});

// ❌ 컴파일 에러: 잘못된 파라미터
await client.get('GET /api/users/:id', { wrongParam: '123' });
```

## 더 실용적인 구현

```typescript
// ✅ 실제 사용하기 쉬운 형태
type ApiEndpoints = {
  getUser: {
    method: 'GET';
    path: '/api/users/:id';
    params: { id: string };
    response: User;
  };
  
  getPosts: {
    method: 'GET';
    path: '/api/posts';
    query: { page?: number; limit?: number };
    response: Post[];
  };
  
  createPost: {
    method: 'POST';
    path: '/api/posts';
    body: { title: string; content: string };
    response: Post;
  };
};

type EndpointName = keyof ApiEndpoints;

class ApiClient {
  async request<E extends EndpointName>(
    endpoint: E,
    ...args: ApiEndpoints[E] extends { params: infer P }
      ? [params: P]
      : ApiEndpoints[E] extends { query: infer Q }
      ? [query?: Q]
      : ApiEndpoints[E] extends { body: infer B }
      ? [body: B]
      : []
  ): Promise<ApiEndpoints[E]['response']> {
    const config = ApiEndpoints[endpoint];
    // 실제 구현...
    return {} as ApiEndpoints[E]['response'];
  }
}
```

## 함수형 접근

```typescript
// ✅ 함수형 API 클라이언트
type EndpointConfig = {
  getUser: { params: { id: string }; response: User };
  getPosts: { query: { page?: number }; response: Post[] };
  createPost: { body: { title: string }; response: Post };
};

function createApiClient<Config extends Record<string, any>>(
  endpoints: Config
) {
  return {
    get: <K extends keyof Config>(
      endpoint: K,
      options: Config[K] extends { params: infer P } ? { params: P }
        : Config[K] extends { query: infer Q } ? { query?: Q }
        : {}
    ): Promise<Config[K] extends { response: infer R } ? R : never> => {
      // 구현...
      return {} as any;
    },
  };
}

const api = createApiClient<EndpointConfig>({
  getUser: { params: { id: '' }, response: {} as User },
  getPosts: { query: {}, response: [] as Post[] },
  createPost: { body: { title: '' }, response: {} as Post },
});

// 사용
const user = await api.get('getUser', { params: { id: '123' } });
const posts = await api.get('getPosts', { query: { page: 1 } });
```

## React Hook과 통합

```typescript
// ✅ 타입 안전한 API Hook
function useApi<E extends EndpointName>(endpoint: E) {
  const [data, setData] = useState<ApiEndpoints[E]['response'] | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<AppError | null>(null);

  const client = new TypedApiClient();

  const fetch = useCallback(
    async (...args: Parameters<typeof client.get>) => {
      setLoading(true);
      try {
        const result = await client.get(endpoint, ...args);
        setData(result);
      } catch (err) {
        setError(err as AppError);
      } finally {
        setLoading(false);
      }
    },
    [endpoint]
  );

  return { data, loading, error, fetch };
}

// 사용
function UserProfile({ userId }: { userId: string }) {
  const { data: user, loading, error } = useApi('getUser');
  
  useEffect(() => {
    fetch({ params: { id: userId } });
  }, [userId]);

  // user는 User 타입으로 자동 추론
  if (loading) return <Loading />;
  if (error) return <Error error={error} />;
  if (!user) return null;

  return <div>{user.name}</div>;
}
```

## 장점

1. **타입 안전성**: endpoint와 타입이 자동으로 연결
2. **자동완성**: IDE에서 endpoint와 파라미터 자동완성
3. **리팩토링 안전성**: endpoint 변경 시 모든 사용처에서 에러 발생
4. **문서화**: 타입 정의 자체가 API 문서 역할
5. **에러 방지**: 잘못된 endpoint나 파라미터 사용을 컴파일 타임에 방지
