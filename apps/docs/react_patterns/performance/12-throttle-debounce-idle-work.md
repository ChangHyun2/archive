# Throttle/Debounce + Idle Work

## 개요

스크롤, 리사이즈, 입력 이벤트처럼 자주 발생하는 이벤트를 throttle/debounce로 제어하고, 가능하면 브라우저의 유휴 시간에 처리하여 성능을 최적화하는 패턴입니다.

## 핵심 원칙

- **이벤트 제어**: 자주 발생하는 이벤트를 throttle/debounce로 제어
- **유휴 시간 활용**: 중요하지 않은 작업은 브라우저 유휴 시간에 처리
- **사용자 경험**: 반응성과 성능의 균형 유지

## Throttle vs Debounce

### Throttle
- 일정 시간 간격으로 최대 한 번만 실행
- 스크롤, 리사이즈 등 연속적인 이벤트에 적합

### Debounce
- 이벤트가 멈춘 후 일정 시간이 지나면 실행
- 검색 입력, 버튼 클릭 등에 적합

## 구현 방법

### 1. Throttle Hook

```tsx
import { useCallback, useRef } from 'react';

function useThrottle(callback, delay) {
  const lastRun = useRef(Date.now());
  
  return useCallback((...args) => {
    if (Date.now() - lastRun.current >= delay) {
      callback(...args);
      lastRun.current = Date.now();
    }
  }, [callback, delay]);
}

// 사용 예시
function ScrollComponent() {
  const handleScroll = useThrottle((e) => {
    // 스크롤 처리 로직
    updateScrollPosition(e.target.scrollTop);
  }, 100); // 100ms마다 최대 한 번
  
  return <div onScroll={handleScroll}>...</div>;
}
```

### 2. Debounce Hook

```tsx
import { useCallback, useRef, useEffect } from 'react';

function useDebounce(callback, delay) {
  const timeoutRef = useRef(null);
  
  return useCallback((...args) => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    timeoutRef.current = setTimeout(() => {
      callback(...args);
    }, delay);
  }, [callback, delay]);
}

// 사용 예시
function SearchInput() {
  const [query, setQuery] = useState('');
  
  const debouncedSearch = useDebounce((value) => {
    // 검색 API 호출
    performSearch(value);
  }, 300); // 입력이 멈춘 후 300ms 후 실행
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    debouncedSearch(value);
  };
  
  return <input value={query} onChange={handleChange} />;
}
```

### 3. 리사이즈 이벤트 처리

```tsx
function useWindowResize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  const handleResize = useThrottle(() => {
    setSize({
      width: window.innerWidth,
      height: window.innerHeight
    });
  }, 150);
  
  useEffect(() => {
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, [handleResize]);
  
  return size;
}
```

### 4. 스크롤 이벤트 처리

```tsx
function useScrollPosition() {
  const [scrollY, setScrollY] = useState(0);
  
  const handleScroll = useThrottle(() => {
    setScrollY(window.scrollY);
  }, 16); // ~60fps
  
  useEffect(() => {
    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  }, [handleScroll]);
  
  return scrollY;
}
```

### 5. requestIdleCallback 활용

```tsx
function useIdleCallback(callback, options = {}) {
  const timeout = options.timeout || 2000;
  
  useEffect(() => {
    if (!window.requestIdleCallback) {
      // 폴백
      const timeoutId = setTimeout(callback, 0);
      return () => clearTimeout(timeoutId);
    }
    
    const idleCallbackId = requestIdleCallback(callback, { timeout });
    
    return () => cancelIdleCallback(idleCallbackId);
  }, [callback, timeout]);
}

// 사용 예시
function AnalyticsComponent() {
  useIdleCallback(() => {
    // 중요하지 않은 분석 작업
    sendAnalytics();
  });
  
  return <div>...</div>;
}
```

### 6. 복합 예시: 검색 + 스크롤

```tsx
function SearchAndScrollComponent() {
  const [searchQuery, setSearchQuery] = useState('');
  const [scrollPosition, setScrollPosition] = useState(0);
  
  // Debounce: 검색 입력
  const debouncedSearch = useDebounce((query) => {
    performSearch(query);
  }, 300);
  
  // Throttle: 스크롤 위치
  const throttledScroll = useThrottle((position) => {
    setScrollPosition(position);
    updateScrollIndicator(position);
  }, 16);
  
  const handleSearchChange = (e) => {
    const value = e.target.value;
    setSearchQuery(value);
    debouncedSearch(value);
  };
  
  const handleScroll = (e) => {
    throttledScroll(e.target.scrollTop);
  };
  
  return (
    <div>
      <input value={searchQuery} onChange={handleSearchChange} />
      <div onScroll={handleScroll}>
        {/* 콘텐츠 */}
      </div>
    </div>
  );
}
```

## 라이브러리 사용

### lodash 사용

```tsx
import { throttle, debounce } from 'lodash';
import { useMemo } from 'react';

function Component() {
  const throttledHandler = useMemo(
    () => throttle((value) => {
      // 처리 로직
    }, 100),
    []
  );
  
  const debouncedHandler = useMemo(
    () => debounce((value) => {
      // 처리 로직
    }, 300),
    []
  );
  
  // cleanup
  useEffect(() => {
    return () => {
      throttledHandler.cancel();
      debouncedHandler.cancel();
    };
  }, [throttledHandler, debouncedHandler]);
}
```

## Best Practices

1. **적절한 지연 시간**: 너무 짧으면 효과 없음, 너무 길면 반응성 저하
2. **이벤트 타입별 선택**: 스크롤은 throttle, 검색은 debounce
3. **리소스 정리**: 컴포넌트 언마운트 시 타이머 정리
4. **passive 이벤트**: 스크롤 이벤트는 passive 옵션 사용
5. **폴백 제공**: requestIdleCallback 미지원 환경 대비

## 주의사항

- throttle/debounce는 클로저를 사용하므로 의존성 관리 주의
- 너무 짧은 delay는 효과가 없을 수 있음
- requestIdleCallback은 모든 브라우저에서 지원되지 않음
- 메모리 누수 방지를 위해 cleanup 필수
