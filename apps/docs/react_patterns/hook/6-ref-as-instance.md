# Ref as Instance (인스턴스 보관)

## 개요

렌더와 무관하게 유지해야 하는 인스턴스는 `useRef`에 저장하는 패턴입니다.

## 핵심 원칙

- **렌더링 독립성**: 리렌더링과 무관하게 인스턴스 유지
- **값 보존**: 컴포넌트 생명주기 동안 동일한 인스턴스 유지
- **메모리 관리**: 명시적인 정리(cleanup) 필요

## 사용 사례

### 1. AbortController

```tsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const abortControllerRef = useRef(null);
  
  useEffect(() => {
    // 새로운 요청 전에 이전 요청 취소
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
    
    abortControllerRef.current = new AbortController();
    
    fetch(url, { signal: abortControllerRef.current.signal })
      .then(res => res.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err);
        }
      });
    
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [url]);
  
  return data;
}
```

### 2. Timer ID

```tsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  const timerRef = useRef(null);
  
  useEffect(() => {
    // 이전 타이머 취소
    if (timerRef.current) {
      clearTimeout(timerRef.current);
    }
    
    // 새 타이머 설정
    timerRef.current = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      if (timerRef.current) {
        clearTimeout(timerRef.current);
      }
    };
  }, [value, delay]);
  
  return debouncedValue;
}
```

### 3. WebSocket

```tsx
function useWebSocket(url) {
  const [message, setMessage] = useState(null);
  const wsRef = useRef(null);
  
  useEffect(() => {
    wsRef.current = new WebSocket(url);
    
    wsRef.current.onmessage = (event) => {
      setMessage(JSON.parse(event.data));
    };
    
    wsRef.current.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
    
    return () => {
      if (wsRef.current) {
        wsRef.current.close();
      }
    };
  }, [url]);
  
  const sendMessage = useCallback((data) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify(data));
    }
  }, []);
  
  return { message, sendMessage };
}
```

### 4. 외부 라이브러리 인스턴스

```tsx
function useChart(containerRef, data) {
  const chartInstanceRef = useRef(null);
  
  useEffect(() => {
    if (!containerRef.current) return;
    
    // Chart.js 인스턴스 생성
    chartInstanceRef.current = new Chart(containerRef.current, {
      type: 'line',
      data: {
        datasets: [{
          data: data,
        }],
      },
    });
    
    return () => {
      if (chartInstanceRef.current) {
        chartInstanceRef.current.destroy();
        chartInstanceRef.current = null;
      }
    };
  }, [containerRef.current]); // container만 의존성
  
  // 데이터 업데이트는 별도 effect로 처리
  useEffect(() => {
    if (chartInstanceRef.current) {
      chartInstanceRef.current.data.datasets[0].data = data;
      chartInstanceRef.current.update();
    }
  }, [data]);
  
  return chartInstanceRef.current;
}
```

### 5. Intersection Observer

```tsx
function useIntersectionObserver(ref, options = {}) {
  const [isIntersecting, setIsIntersecting] = useState(false);
  const observerRef = useRef(null);
  
  useEffect(() => {
    if (!ref.current) return;
    
    observerRef.current = new IntersectionObserver(
      ([entry]) => {
        setIsIntersecting(entry.isIntersecting);
      },
      options
    );
    
    observerRef.current.observe(ref.current);
    
    return () => {
      if (observerRef.current) {
        observerRef.current.disconnect();
      }
    };
  }, [ref, options]);
  
  return isIntersecting;
}
```

## useState vs useRef

### useState (렌더링 트리거)

```tsx
// ❌ 잘못된 사용: 인스턴스를 state에 저장
function Component() {
  const [controller, setController] = useState(new AbortController());
  // 문제: 리렌더링마다 새로운 인스턴스 생성
}
```

### useRef (렌더링 없이 값 보존)

```tsx
// ✅ 올바른 사용: 인스턴스를 ref에 저장
function Component() {
  const controllerRef = useRef(new AbortController());
  // 장점: 컴포넌트 생명주기 동안 동일한 인스턴스 유지
}
```

## 패턴: Lazy Initialization

```tsx
// 인스턴스 생성이 비용이 큰 경우
function useExpensiveInstance() {
  const instanceRef = useRef(null);
  
  // 첫 렌더링에만 인스턴스 생성
  if (instanceRef.current === null) {
    instanceRef.current = new ExpensiveClass();
  }
  
  return instanceRef.current;
}
```

## 주의사항

1. **정리(cleanup) 필수**: useEffect의 cleanup에서 인스턴스 정리
2. **null 체크**: 사용 전에 ref.current가 null이 아닌지 확인
3. **의존성 관리**: ref는 의존성 배열에 포함하지 않음 (항상 동일한 ref 객체)

## 장점

1. **성능**: 불필요한 리렌더링 방지
2. **안정성**: 인스턴스가 예상치 못하게 재생성되지 않음
3. **메모리 관리**: 명시적인 정리로 메모리 누수 방지
