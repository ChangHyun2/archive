# startTransition / useDeferredValue로 입력 UX 개선

## 개요

검색이나 필터링처럼 사용자 입력에 반응하는 UI에서, 입력 반응성을 우선 처리하고 무거운 렌더링은 뒤로 미루는 패턴입니다. React 18의 동시성 기능을 활용합니다.

## 핵심 원칙

- **입력 우선순위**: 사용자 입력은 즉시 반응
- **무거운 렌더링 지연**: 검색 결과 등 무거운 렌더링은 낮은 우선순위로
- **타이핑 끊김 방지**: 입력 중에도 부드러운 사용자 경험

## startTransition 사용

### 기본 사용법

```tsx
import { startTransition, useState } from 'react';

// ❌ 나쁜 예: 모든 상태 업데이트가 동기적으로 처리
function SearchBox() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    
    // 무거운 필터링이 입력을 막음
    const filtered = heavyFilter(data, value);
    setResults(filtered);
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />
      <ResultsList results={results} />
    </div>
  );
}

// ✅ 좋은 예: startTransition으로 우선순위 분리
function SearchBox() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    
    // 입력은 즉시 반영 (높은 우선순위)
    setQuery(value);
    
    // 검색 결과는 낮은 우선순위로 처리
    startTransition(() => {
      const filtered = heavyFilter(data, value);
      setResults(filtered);
    });
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />
      <ResultsList results={results} />
    </div>
  );
}
```

### useTransition Hook

```tsx
import { useTransition, useState } from 'react';

function SearchBox() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    
    startTransition(() => {
      const filtered = heavyFilter(data, value);
      setResults(filtered);
    });
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <span>Searching...</span>}
      <ResultsList results={results} />
    </div>
  );
}
```

## useDeferredValue 사용

### 기본 사용법

```tsx
import { useDeferredValue, useState, useMemo } from 'react';

// ✅ useDeferredValue로 값 업데이트 지연
function SearchBox() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  
  // deferredQuery가 변경될 때만 무거운 계산 실행
  const results = useMemo(() => {
    return heavyFilter(data, deferredQuery);
  }, [deferredQuery]);
  
  const isPending = query !== deferredQuery;
  
  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      {isPending && <span>Searching...</span>}
      <ResultsList results={results} />
    </div>
  );
}
```

### 복잡한 예시

```tsx
function AdvancedSearch() {
  const [filters, setFilters] = useState({
    query: '',
    category: 'all',
    sortBy: 'relevance'
  });
  
  const deferredFilters = useDeferredValue(filters);
  const isPending = filters !== deferredFilters;
  
  const results = useMemo(() => {
    return heavySearch(data, deferredFilters);
  }, [deferredFilters]);
  
  return (
    <div>
      <SearchInputs 
        filters={filters} 
        onChange={setFilters} 
      />
      {isPending && <SearchIndicator />}
      <SearchResults results={results} />
    </div>
  );
}
```

## 언제 사용해야 할까?

### startTransition 사용 시기

- 사용자 입력에 즉시 반응해야 하는 경우
- 무거운 상태 업데이트가 입력을 막는 경우
- 검색, 필터링, 정렬 등

### useDeferredValue 사용 시기

- 파생된 값(derived value)을 지연시키고 싶을 때
- 계산 비용이 큰 값 업데이트를 지연시키고 싶을 때
- 입력과 결과 표시를 분리하고 싶을 때

## Best Practices

1. **입력 필드**: 사용자 입력은 항상 즉시 반영
2. **무거운 작업**: 필터링, 정렬, 검색은 transition으로
3. **시각적 피드백**: isPending으로 로딩 상태 표시
4. **적절한 사용**: 모든 상태 업데이트에 사용하지 말 것
5. **에러 처리**: transition 내부의 에러도 적절히 처리

## 주의사항

- startTransition은 상태 업데이트만 지연시킴 (네트워크 요청은 아님)
- 너무 많은 transition은 오히려 성능 저하
- React 18 이상에서만 사용 가능
- 서버 컴포넌트에서는 사용 불가

## 비교

```tsx
// startTransition: 상태 업데이트를 transition으로 표시
startTransition(() => {
  setState(newValue);
});

// useDeferredValue: 값 자체를 지연
const deferredValue = useDeferredValue(value);
```
