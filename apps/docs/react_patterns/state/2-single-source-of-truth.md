# Single Source of Truth

## 개요

같은 의미의 상태를 여러 곳에 중복 저장하지 않고, 단일 소스에서 관리하는 패턴입니다. 폼 값, derived 값, URL 파라미터가 서로 불일치하는 사고를 방지합니다.

## 특징

- **단일 진실 공급원**: 하나의 상태만이 진실의 원천
- **일관성 보장**: 상태 불일치로 인한 버그 방지
- **동기화 최소화**: 여러 소스 간 동기화 로직 불필요

## 예시

```tsx
// ❌ 나쁜 예: 여러 곳에 상태 중복
function SearchPage() {
  const [query, setQuery] = useState(''); // 컴포넌트 상태
  const router = useRouter();
  const urlQuery = router.query.q; // URL 파라미터
  
  // 두 상태가 불일치할 수 있음
  useEffect(() => {
    setQuery(urlQuery || '');
  }, [urlQuery]);
}

// ✅ 좋은 예: URL을 단일 소스로 사용
function SearchPage() {
  const router = useRouter();
  const query = router.query.q || ''; // URL이 단일 소스
  
  const handleSearch = (newQuery) => {
    router.push({ query: { q: newQuery } });
  };
  
  return <SearchInput value={query} onChange={handleSearch} />;
}
```

## 구현 방법

1. **상태의 출처 결정**: 어떤 것이 진실의 원천인지 명확히 정의
2. **파생 상태 처리**: 다른 값들은 계산된 값(derived)으로 처리
3. **중복 제거**: 같은 의미의 상태가 여러 곳에 있는지 확인

## 장점

- 상태 불일치 버그 방지
- 코드 복잡도 감소
- 디버깅 용이
- 예측 가능한 동작

## 단점

- 초기 설계 시 신중한 고려 필요
- 기존 코드 리팩토링 시 작업량 증가
- 때로는 단일 소스로 만들기 어려운 경우 존재
