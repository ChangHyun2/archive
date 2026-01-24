# Testable Hook Design

## 개요

순수 로직(파서/계산/상태전이)은 Hook 밖의 pure function으로 분리하고, Hook은 "React glue"만 담당하도록 하는 패턴입니다. 테스트와 리팩토링 난이도가 낮아집니다.

## 핵심 원칙

- **순수 함수 분리**: 비즈니스 로직을 React와 독립적인 순수 함수로 추출
- **Hook은 연결만**: Hook은 React의 상태 관리와 생명주기만 담당
- **테스트 용이성**: 순수 함수는 별도로 테스트, Hook은 통합 테스트

## 문제 상황: 로직이 Hook에 섞여 있음

```tsx
// ❌ 문제: 로직이 Hook에 하드코딩되어 테스트 어려움
function usePriceCalculator(items) {
  const [total, setTotal] = useState(0);
  const [discount, setDiscount] = useState(0);
  
  useEffect(() => {
    // 비즈니스 로직이 Hook 안에 있음
    let subtotal = 0;
    for (const item of items) {
      subtotal += item.price * item.quantity;
    }
    
    let discountAmount = 0;
    if (subtotal > 1000) {
      discountAmount = subtotal * 0.1; // 10% 할인
    } else if (subtotal > 500) {
      discountAmount = subtotal * 0.05; // 5% 할인
    }
    
    setTotal(subtotal - discountAmount);
    setDiscount(discountAmount);
  }, [items]);
  
  return { total, discount };
}
```

## 해결 방법: 순수 함수로 분리

### 1. 순수 함수 추출

```tsx
// ✅ 순수 함수: React와 독립적, 쉽게 테스트 가능
function calculateSubtotal(items: CartItem[]): number {
  return items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );
}

function calculateDiscount(subtotal: number): number {
  if (subtotal > 1000) {
    return subtotal * 0.1; // 10% 할인
  }
  if (subtotal > 500) {
    return subtotal * 0.05; // 5% 할인
  }
  return 0;
}

function calculateTotal(items: CartItem[]): {
  subtotal: number;
  discount: number;
  total: number;
} {
  const subtotal = calculateSubtotal(items);
  const discount = calculateDiscount(subtotal);
  const total = subtotal - discount;
  
  return { subtotal, discount, total };
}
```

### 2. Hook은 React 연결만 담당

```tsx
// ✅ Hook: React glue만 담당
function usePriceCalculator(items: CartItem[]) {
  const [result, setResult] = useState({
    subtotal: 0,
    discount: 0,
    total: 0,
  });
  
  useEffect(() => {
    // 순수 함수 호출만
    const calculated = calculateTotal(items);
    setResult(calculated);
  }, [items]);
  
  return result;
}
```

### 3. 순수 함수 테스트

```tsx
// 순수 함수는 쉽게 테스트 가능
describe('calculateTotal', () => {
  it('should calculate subtotal correctly', () => {
    const items = [
      { price: 100, quantity: 2 },
      { price: 50, quantity: 3 },
    ];
    const result = calculateTotal(items);
    expect(result.subtotal).toBe(350);
  });
  
  it('should apply 10% discount for orders over 1000', () => {
    const items = [{ price: 600, quantity: 2 }]; // 1200
    const result = calculateTotal(items);
    expect(result.discount).toBe(120);
    expect(result.total).toBe(1080);
  });
  
  it('should apply 5% discount for orders over 500', () => {
    const items = [{ price: 300, quantity: 2 }]; // 600
    const result = calculateTotal(items);
    expect(result.discount).toBe(30);
    expect(result.total).toBe(570);
  });
});
```

## 실전 예시

### 1. 폼 검증 로직 분리

```tsx
// 순수 함수: 검증 로직
type ValidationRule<T> = (value: T) => string | null;

function validateEmail(email: string): string | null {
  if (!email) return 'Email is required';
  if (!/\S+@\S+\.\S+/.test(email)) return 'Email is invalid';
  return null;
}

function validatePassword(password: string): string | null {
  if (!password) return 'Password is required';
  if (password.length < 8) return 'Password must be at least 8 characters';
  if (!/[A-Z]/.test(password)) return 'Password must contain uppercase';
  if (!/[0-9]/.test(password)) return 'Password must contain number';
  return null;
}

function validateForm<T extends Record<string, any>>(
  values: T,
  rules: Record<keyof T, ValidationRule<any>>
): Record<string, string> {
  const errors: Record<string, string> = {};
  
  for (const [field, rule] of Object.entries(rules)) {
    const error = rule(values[field]);
    if (error) {
      errors[field] = error;
    }
  }
  
  return errors;
}

// Hook: React 연결만
function useFormValidation<T extends Record<string, any>>(
  values: T,
  rules: Record<keyof T, ValidationRule<any>>
) {
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  useEffect(() => {
    const validationErrors = validateForm(values, rules);
    setErrors(validationErrors);
  }, [values, rules]);
  
  return {
    errors,
    isValid: Object.keys(errors).length === 0,
  };
}
```

### 2. 데이터 변환 로직 분리

```tsx
// 순수 함수: 데이터 변환
function parseUserResponse(response: ApiUserResponse): User {
  return {
    id: response.user_id,
    name: response.full_name,
    email: response.email_address,
    createdAt: new Date(response.created_at),
  };
}

function transformUsersForDisplay(users: User[]): UserDisplay[] {
  return users.map(user => ({
    ...user,
    displayName: `${user.name} (${user.email})`,
    isActive: user.createdAt > new Date('2024-01-01'),
  }));
}

// Hook: React 연결만
function useUsers() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then((data: ApiUserResponse[]) => {
        // 순수 함수로 변환
        const parsed = data.map(parseUserResponse);
        setUsers(parsed);
        setLoading(false);
      });
  }, []);
  
  // 순수 함수로 변환
  const displayUsers = useMemo(
    () => transformUsersForDisplay(users),
    [users]
  );
  
  return { users: displayUsers, loading };
}
```

### 3. 상태 전이 로직 분리

```tsx
// 순수 함수: 상태 전이
type TodoStatus = 'pending' | 'in-progress' | 'completed';

function canTransition(
  from: TodoStatus,
  to: TodoStatus
): boolean {
  const transitions: Record<TodoStatus, TodoStatus[]> = {
    'pending': ['in-progress', 'completed'],
    'in-progress': ['completed', 'pending'],
    'completed': ['pending'],
  };
  
  return transitions[from]?.includes(to) ?? false;
}

function transitionTodo(
  todo: Todo,
  newStatus: TodoStatus
): Todo {
  if (!canTransition(todo.status, newStatus)) {
    throw new Error(`Cannot transition from ${todo.status} to ${newStatus}`);
  }
  
  return {
    ...todo,
    status: newStatus,
    updatedAt: new Date(),
  };
}

// Hook: React 연결만
function useTodo(todoId: string) {
  const [todo, setTodo] = useState<Todo | null>(null);
  
  const updateStatus = useCallback((newStatus: TodoStatus) => {
    if (!todo) return;
    
    try {
      // 순수 함수로 상태 전이
      const updated = transitionTodo(todo, newStatus);
      setTodo(updated);
      
      // 서버 동기화
      updateTodoOnServer(todoId, newStatus);
    } catch (error) {
      console.error('Transition failed:', error);
    }
  }, [todo, todoId]);
  
  return { todo, updateStatus };
}
```

## 테스트 전략

### 순수 함수 테스트 (단위 테스트)

```tsx
// 빠르고 간단한 단위 테스트
describe('calculateTotal', () => {
  // ...
});

describe('validateEmail', () => {
  it('should return null for valid email', () => {
    expect(validateEmail('test@example.com')).toBeNull();
  });
  
  it('should return error for invalid email', () => {
    expect(validateEmail('invalid')).toBe('Email is invalid');
  });
});
```

### Hook 테스트 (통합 테스트)

```tsx
import { renderHook, act } from '@testing-library/react-hooks';

describe('usePriceCalculator', () => {
  it('should calculate price correctly', () => {
    const items = [{ price: 100, quantity: 2 }];
    const { result } = renderHook(() => usePriceCalculator(items));
    
    expect(result.current.total).toBe(200);
  });
});
```

## 장점

1. **테스트 용이성**: 순수 함수는 의존성 없이 쉽게 테스트
2. **재사용성**: 순수 함수를 다른 맥락에서도 사용 가능
3. **리팩토링 용이성**: 로직 변경이 Hook에 영향 없음
4. **성능 최적화**: useMemo로 순수 함수 결과 캐싱 가능
5. **가독성**: 로직과 React 연결이 명확히 분리

## 주의사항

- 과도한 분리는 오히려 복잡도 증가
- 간단한 로직은 Hook 안에 두어도 됨
- 순수 함수와 Hook의 경계를 명확히 유지
