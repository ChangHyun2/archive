# Latest Value Pattern (stale closure 방지)

## 개요

이벤트 핸들러나 비동기 콜백에서 최신 값을 사용하기 위해 `useRef`로 최신 값을 유지하는 패턴입니다. 재사용 Hook을 만들 때 특히 자주 필요합니다.

## 문제 상황: Stale Closure

```tsx
// ❌ 문제: stale closure
function useCounter() {
  const [count, setCount] = useState(0);
  
  const increment = useCallback(() => {
    // 문제: 이 함수가 생성될 때의 count 값만 기억
    setCount(count + 1);
  }, [count]);
  
  // 비동기 작업에서 문제 발생
  const incrementAfterDelay = useCallback(() => {
    setTimeout(() => {
      // 1초 후에도 이전 count 값 사용
      setCount(count + 1);
    }, 1000);
  }, [count]);
  
  return { count, increment, incrementAfterDelay };
}
```

## 해결 방법: useRef로 최신 값 유지

```tsx
// ✅ 해결: ref로 최신 값 유지
function useCounter() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);
  
  // count가 변경될 때마다 ref 업데이트
  useEffect(() => {
    countRef.current = count;
  }, [count]);
  
  const increment = useCallback(() => {
    // 항상 최신 값 사용
    setCount(countRef.current + 1);
  }, []); // 의존성 배열이 비어있어도 안전!
  
  const incrementAfterDelay = useCallback(() => {
    setTimeout(() => {
      // 최신 count 값 사용
      setCount(countRef.current + 1);
    }, 1000);
  }, []); // 의존성 배열이 비어있어도 안전!
  
  return { count, increment, incrementAfterDelay };
}
```

## 실전 예시

### 1. 이벤트 리스너에서 최신 값 사용

```tsx
function useWindowResize(onResize) {
  const onResizeRef = useRef(onResize);
  
  // 최신 콜백 유지
  useEffect(() => {
    onResizeRef.current = onResize;
  }, [onResize]);
  
  useEffect(() => {
    const handleResize = () => {
      // 항상 최신 콜백 호출
      onResizeRef.current(window.innerWidth, window.innerHeight);
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []); // 의존성 배열이 비어있어도 안전!
}
```

### 2. 비동기 작업에서 최신 상태 사용

```tsx
function useAsyncOperation() {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  
  const isLoadingRef = useRef(isLoading);
  useEffect(() => {
    isLoadingRef.current = isLoading;
  }, [isLoading]);
  
  const performOperation = useCallback(async () => {
    if (isLoadingRef.current) {
      console.log('Already loading, skipping...');
      return;
    }
    
    setIsLoading(true);
    
    try {
      const result = await fetch('/api/data').then(r => r.json());
      
      // 비동기 작업 완료 후에도 최신 상태 확인
      if (!isLoadingRef.current) {
        // 컴포넌트가 언마운트되었거나 상태가 변경되었을 수 있음
        return;
      }
      
      setData(result);
    } finally {
      setIsLoading(false);
    }
  }, []); // 의존성 배열이 비어있어도 안전!
  
  return { data, isLoading, performOperation };
}
```

### 3. Debounce에서 최신 값 사용

```tsx
function useDebouncedCallback(callback, delay) {
  const callbackRef = useRef(callback);
  const timeoutRef = useRef(null);
  
  // 최신 콜백 유지
  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);
  
  const debouncedCallback = useCallback((...args) => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    timeoutRef.current = setTimeout(() => {
      // 최신 콜백 호출
      callbackRef.current(...args);
    }, delay);
  }, [delay]);
  
  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);
  
  return debouncedCallback;
}
```

### 4. Custom Hook에서 props 사용

```tsx
function useFormattedDate(date, format) {
  const formatRef = useRef(format);
  const [formatted, setFormatted] = useState('');
  
  useEffect(() => {
    formatRef.current = format;
  }, [format]);
  
  useEffect(() => {
    const formatDate = async () => {
      // 비동기 포맷팅 (예: locale에 따라)
      const formatted = await formatDateAsync(
        date,
        formatRef.current // 최신 format 사용
      );
      setFormatted(formatted);
    };
    
    formatDate();
  }, [date]); // format은 의존성에 없어도 됨!
  
  return formatted;
}
```

## 패턴: useLatest Hook

재사용 가능한 유틸리티 Hook으로 만들기:

```tsx
function useLatest(value) {
  const ref = useRef(value);
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref;
}

// 사용 예시
function useCounter() {
  const [count, setCount] = useState(0);
  const countRef = useLatest(count);
  
  const increment = useCallback(() => {
    setCount(countRef.current + 1);
  }, []);
  
  return { count, increment };
}
```

## 패턴: useCallbackRef

콜백 함수를 위한 특화 Hook:

```tsx
function useCallbackRef(callback) {
  const callbackRef = useRef(callback);
  
  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);
  
  const memoizedCallback = useCallback((...args) => {
    return callbackRef.current(...args);
  }, []);
  
  return memoizedCallback;
}

// 사용 예시
function Component({ onSave }) {
  const stableOnSave = useCallbackRef(onSave);
  
  useEffect(() => {
    // onSave가 변경되어도 effect 재실행 안 됨
    // 하지만 항상 최신 onSave 호출
    const timer = setInterval(() => {
      stableOnSave('auto-save');
    }, 5000);
    
    return () => clearInterval(timer);
  }, [stableOnSave]); // 안정적인 참조
}
```

## 주의사항

1. **의도 명확화**: ref를 사용하는 이유를 주석으로 설명
2. **동기화**: state 변경 시 ref도 함께 업데이트
3. **초기값**: ref의 초기값을 state의 초기값과 일치시키기

## 장점

1. **의존성 배열 단순화**: 불필요한 의존성 제거
2. **성능 최적화**: 불필요한 재생성 방지
3. **안정성**: 항상 최신 값 사용 보장

## 사용 시기

- 이벤트 리스너에서 최신 state/props 사용이 필요할 때
- 비동기 콜백에서 최신 값이 필요할 때
- 재사용 가능한 Hook을 만들 때
- 의존성 배열을 단순화하고 싶을 때
