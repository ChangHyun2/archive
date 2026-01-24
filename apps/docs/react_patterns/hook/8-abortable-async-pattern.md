# Abortable Async Pattern

## 개요

비동기 작업을 취소 가능하게 만들어 레이스 컨디션이나 언마운트 후 setState를 방지하는 패턴입니다.

## 문제 상황

### 1. Race Condition

```tsx
// ❌ 문제: 빠른 연속 요청 시 이전 응답이 나중에 도착
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, [userId]);
  
  // userId가 빠르게 변경되면:
  // 1. userId=1 요청 시작
  // 2. userId=2 요청 시작
  // 3. userId=2 응답 도착 → setUser(user2)
  // 4. userId=1 응답 도착 → setUser(user1) ❌ 잘못된 상태!
}
```

### 2. 언마운트 후 setState

```tsx
// ❌ 문제: 컴포넌트 언마운트 후에도 setState 호출
function Component() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(setData); // 언마운트 후 호출되면 경고 발생
  }, []);
  
  return <div>{data}</div>;
}
```

## 해결 방법: AbortController

### 기본 패턴

```tsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const abortController = new AbortController();
    
    fetch(url, { signal: abortController.signal })
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then(setData)
      .catch(err => {
        // AbortError는 무시 (의도된 취소)
        if (err.name !== 'AbortError') {
          setError(err);
        }
      });
    
    // cleanup: 요청 취소
    return () => {
      abortController.abort();
    };
  }, [url]);
  
  return { data, error };
}
```

### 고급 패턴: 재사용 가능한 Hook

```tsx
function useAbortableFetch(url, options = {}) {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null,
  });
  
  useEffect(() => {
    const abortController = new AbortController();
    let isMounted = true;
    
    setState({ data: null, loading: true, error: null });
    
    fetch(url, {
      ...options,
      signal: abortController.signal,
    })
      .then(async res => {
        if (!res.ok) {
          throw new Error(`HTTP ${res.status}`);
        }
        return res.json();
      })
      .then(data => {
        // 컴포넌트가 마운트되어 있고 요청이 취소되지 않았을 때만 업데이트
        if (isMounted && !abortController.signal.aborted) {
          setState({ data, loading: false, error: null });
        }
      })
      .catch(err => {
        if (isMounted && err.name !== 'AbortError') {
          setState({ data: null, loading: false, error: err });
        }
      });
    
    return () => {
      isMounted = false;
      abortController.abort();
    };
  }, [url, JSON.stringify(options)]);
  
  return state;
}
```

## 다른 비동기 작업 취소 패턴

### 1. Promise 기반 작업

```tsx
function useAsyncOperation(asyncFn, deps) {
  const [result, setResult] = useState(null);
  const cancelRef = useRef(false);
  
  useEffect(() => {
    cancelRef.current = false;
    
    asyncFn()
      .then(data => {
        if (!cancelRef.current) {
          setResult(data);
        }
      })
      .catch(err => {
        if (!cancelRef.current) {
          console.error(err);
        }
      });
    
    return () => {
      cancelRef.current = true;
    };
  }, deps);
  
  return result;
}
```

### 2. axios 취소 토큰

```tsx
import axios from 'axios';

function useAxiosFetch(url) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const cancelTokenSource = axios.CancelToken.source();
    
    axios.get(url, {
      cancelToken: cancelTokenSource.token,
    })
      .then(res => setData(res.data))
      .catch(err => {
        if (!axios.isCancel(err)) {
          setError(err);
        }
      });
    
    return () => {
      cancelTokenSource.cancel('Component unmounted');
    };
  }, [url]);
  
  return { data, error };
}
```

### 3. 수동 취소 가능한 Hook

```tsx
function useCancellableAsync(asyncFn, deps) {
  const [state, setState] = useState({
    data: null,
    loading: false,
    error: null,
  });
  const cancelRef = useRef(() => {});
  
  const execute = useCallback(async (...args) => {
    let isCancelled = false;
    
    cancelRef.current = () => {
      isCancelled = true;
    };
    
    setState(prev => ({ ...prev, loading: true, error: null }));
    
    try {
      const result = await asyncFn(...args);
      
      if (!isCancelled) {
        setState({ data: result, loading: false, error: null });
      }
    } catch (err) {
      if (!isCancelled) {
        setState({ data: null, loading: false, error: err });
      }
    }
  }, deps);
  
  const cancel = useCallback(() => {
    cancelRef.current();
  }, []);
  
  useEffect(() => {
    return () => {
      cancel();
    };
  }, [cancel]);
  
  return { ...state, execute, cancel };
}
```

## 실전 예시

### 검색 자동완성

```tsx
function useSearchAutocomplete(query) {
  const [suggestions, setSuggestions] = useState([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    if (!query) {
      setSuggestions([]);
      return;
    }
    
    const abortController = new AbortController();
    setLoading(true);
    
    fetch(`/api/search?q=${query}`, {
      signal: abortController.signal,
    })
      .then(res => res.json())
      .then(data => {
        if (!abortController.signal.aborted) {
          setSuggestions(data.results);
          setLoading(false);
        }
      })
      .catch(err => {
        if (err.name !== 'AbortError' && !abortController.signal.aborted) {
          console.error(err);
          setLoading(false);
        }
      });
    
    return () => {
      abortController.abort();
    };
  }, [query]);
  
  return { suggestions, loading };
}
```

### 폴링 (Polling) 취소

```tsx
function usePolling(url, interval = 5000) {
  const [data, setData] = useState(null);
  const abortControllerRef = useRef(null);
  
  useEffect(() => {
    abortControllerRef.current = new AbortController();
    
    const poll = async () => {
      try {
        const res = await fetch(url, {
          signal: abortControllerRef.current.signal,
        });
        const data = await res.json();
        setData(data);
      } catch (err) {
        if (err.name !== 'AbortError') {
          console.error('Polling error:', err);
        }
      }
    };
    
    poll(); // 즉시 실행
    const intervalId = setInterval(poll, interval);
    
    return () => {
      clearInterval(intervalId);
      abortControllerRef.current?.abort();
    };
  }, [url, interval]);
  
  return data;
}
```

## 주의사항

1. **AbortError 처리**: 의도된 취소는 에러로 처리하지 않음
2. **상태 업데이트 전 체크**: 취소된 요청의 응답으로 상태 업데이트 방지
3. **메모리 누수**: cleanup 함수에서 반드시 취소 처리

## 장점

1. **Race Condition 방지**: 이전 요청이 나중에 도착해도 무시
2. **메모리 안전성**: 언마운트 후 setState 방지
3. **리소스 절약**: 불필요한 네트워크 요청 취소
4. **사용자 경험**: 빠른 응답으로 더 나은 UX
