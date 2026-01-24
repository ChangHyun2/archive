# Domain Hook (도메인 전용 Hook)

## 개요

기술적 Hook이 아닌 "업무 개념"으로 Hook을 만드는 패턴입니다. 시니어들이 코드를 읽기 쉽게 만드는 핵심 습관입니다.

## 핵심 원칙

- **도메인 중심**: 기술적 구현이 아닌 비즈니스 개념으로 Hook 명명
- **의도 명확화**: Hook 이름만 봐도 무엇을 하는지 명확히 이해 가능
- **가독성**: 코드를 읽는 사람이 비즈니스 로직을 쉽게 이해

## 예시

### 기술적 Hook vs 도메인 Hook

```tsx
// ❌ 기술적 Hook (구현 세부사항 노출)
function CheckoutPage() {
  const { data, loading, error, mutate } = useMutation('/api/checkout');
  const { data: cart } = useQuery('/api/cart');
  // ...
}

// ✅ 도메인 Hook (비즈니스 의도 명확)
function CheckoutPage() {
  const { checkout, isProcessing, error } = useCheckout();
  const { items, total } = useCart();
  // ...
}
```

### useCheckout Hook

```tsx
function useCheckout() {
  const { mutate, isLoading, error } = useMutation(
    (checkoutData) => fetch('/api/checkout', {
      method: 'POST',
      body: JSON.stringify(checkoutData),
    }).then(r => r.json())
  );
  
  const checkout = useCallback(async (paymentInfo) => {
    const result = await mutate({
      paymentInfo,
      timestamp: Date.now(),
    });
    
    // 주문 완료 후 장바구니 비우기
    clearCart();
    
    // 주문 확인 페이지로 이동
    router.push(`/orders/${result.orderId}`);
    
    return result;
  }, [mutate]);
  
  return {
    checkout,
    isProcessing: isLoading,
    error,
  };
}
```

### useCart Hook

```tsx
function useCart() {
  const { data: cartItems, mutate } = useSWR('/api/cart', fetcher);
  
  const addItem = useCallback(async (product) => {
    await mutate(
      [...(cartItems || []), product],
      false // optimistic update
    );
    
    // 서버에 동기화
    await fetch('/api/cart', {
      method: 'POST',
      body: JSON.stringify({ productId: product.id }),
    });
    
    mutate(); // 재검증
  }, [cartItems, mutate]);
  
  const removeItem = useCallback(async (productId) => {
    const updated = (cartItems || []).filter(
      item => item.id !== productId
    );
    
    await mutate(updated, false);
    await fetch(`/api/cart/${productId}`, { method: 'DELETE' });
    mutate();
  }, [cartItems, mutate]);
  
  const clearCart = useCallback(async () => {
    await mutate([], false);
    await fetch('/api/cart', { method: 'DELETE' });
    mutate();
  }, [mutate]);
  
  const total = useMemo(() => {
    return (cartItems || []).reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
  }, [cartItems]);
  
  return {
    items: cartItems || [],
    total,
    addItem,
    removeItem,
    clearCart,
    itemCount: (cartItems || []).length,
  };
}
```

### usePermission Hook

```tsx
function usePermission(permission) {
  const { user } = useAuth();
  const { data: permissions } = useSWR(
    user ? `/api/users/${user.id}/permissions` : null,
    fetcher
  );
  
  const hasPermission = useMemo(() => {
    if (!permissions) return false;
    return permissions.includes(permission);
  }, [permissions, permission]);
  
  const canAccess = useCallback((resource) => {
    if (!permissions) return false;
    return permissions.some(p => 
      p.resource === resource && p.action === 'access'
    );
  }, [permissions]);
  
  return {
    hasPermission,
    canAccess,
    isLoading: !permissions,
  };
}
```

### 사용 예시

```tsx
// 비즈니스 로직이 명확하게 드러남
function ProductPage({ productId }) {
  const { items, addItem } = useCart();
  const { checkout, isProcessing } = useCheckout();
  const { canAccess } = usePermission('purchase');
  
  const handlePurchase = async () => {
    if (!canAccess('products')) {
      toast.error('구매 권한이 없습니다.');
      return;
    }
    
    await addItem({ id: productId, quantity: 1 });
    await checkout({ paymentMethod: 'card' });
  };
  
  return (
    <div>
      <button 
        onClick={handlePurchase}
        disabled={isProcessing}
      >
        구매하기
      </button>
    </div>
  );
}
```

## 도메인 Hook 설계 가이드

### 1. 비즈니스 용어 사용

```tsx
// ❌ 기술적
useMutation('/api/orders')
useQuery('/api/products')

// ✅ 도메인
useCreateOrder()
useProductCatalog()
```

### 2. 관련 로직 그룹화

```tsx
// ❌ 분산된 로직
const { data: user } = useQuery('/api/user');
const { data: profile } = useQuery('/api/profile');
const updateUser = useMutation('/api/user');

// ✅ 그룹화된 도메인 Hook
const { user, profile, updateProfile } = useUserProfile();
```

### 3. 비즈니스 규칙 캡슐화

```tsx
function useDiscount() {
  const { user } = useAuth();
  const { total } = useCart();
  
  // 비즈니스 규칙: VIP 회원은 10% 할인
  const discountRate = useMemo(() => {
    if (user?.tier === 'VIP') return 0.1;
    if (total > 100000) return 0.05;
    return 0;
  }, [user, total]);
  
  const discountedTotal = useMemo(() => {
    return total * (1 - discountRate);
  }, [total, discountRate]);
  
  return {
    discountRate,
    discountedTotal,
    savings: total - discountedTotal,
  };
}
```

## 장점

1. **가독성**: 코드를 읽는 사람이 비즈니스 의도를 쉽게 이해
2. **유지보수성**: 비즈니스 로직 변경이 한 곳에 집중
3. **재사용성**: 도메인 개념 단위로 재사용 가능
4. **테스트 용이성**: 비즈니스 로직을 독립적으로 테스트

## 주의사항

- 도메인 Hook이 너무 커지면 세분화 고려
- 기술적 세부사항은 내부에 숨기고 비즈니스 인터페이스만 노출
- 팀 내에서 도메인 용어를 일관되게 사용
