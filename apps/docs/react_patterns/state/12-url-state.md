# URL State (Router State)

## 개요

검색/필터/페이지네이션 같은 상태를 URL에 싣는 패턴입니다. 공유/북마크/뒤로가기/새로고침에 강해집니다.

## 특징

- **URL 동기화**: UI 상태가 URL과 동기화됨
- **공유 가능**: URL을 공유하면 같은 상태로 접근 가능
- **브라우저 히스토리**: 뒤로가기/앞으로가기 지원
- **새로고침 안정성**: 새로고침해도 상태 유지

## 예시

```tsx
// Next.js 예시
import { useRouter, useSearchParams } from 'next/navigation';

function ProductList() {
  const router = useRouter();
  const searchParams = useSearchParams();
  
  // URL에서 상태 읽기
  const page = parseInt(searchParams.get('page') || '1');
  const category = searchParams.get('category') || 'all';
  const sort = searchParams.get('sort') || 'price';
  const search = searchParams.get('q') || '';
  
  // URL 상태 업데이트
  const updateFilters = (newFilters) => {
    const params = new URLSearchParams(searchParams);
    
    Object.entries(newFilters).forEach(([key, value]) => {
      if (value === null || value === '' || value === 'all') {
        params.delete(key);
      } else {
        params.set(key, value);
      }
    });
    
    router.push(`?${params.toString()}`);
  };
  
  const handlePageChange = (newPage) => {
    updateFilters({ page: newPage.toString() });
  };
  
  const handleCategoryChange = (newCategory) => {
    updateFilters({ category: newCategory, page: '1' }); // 카테고리 변경 시 페이지 리셋
  };
  
  return (
    <div>
      <SearchInput 
        value={search}
        onChange={(value) => updateFilters({ q: value, page: '1' })}
      />
      <CategoryFilter 
        value={category}
        onChange={handleCategoryChange}
      />
      <SortSelect 
        value={sort}
        onChange={(value) => updateFilters({ sort: value })}
      />
      <ProductGrid page={page} category={category} sort={sort} search={search} />
      <Pagination page={page} onChange={handlePageChange} />
    </div>
  );
}

// React Router 예시
import { useSearchParams } from 'react-router-dom';

function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const query = searchParams.get('q') || '';
  const filters = {
    category: searchParams.get('category') || 'all',
    priceRange: searchParams.get('price') || 'all',
  };
  
  const updateQuery = (newQuery) => {
    setSearchParams(prev => {
      const next = new URLSearchParams(prev);
      if (newQuery) {
        next.set('q', newQuery);
      } else {
        next.delete('q');
      }
      return next;
    });
  };
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => updateQuery(e.target.value)}
        placeholder="Search..."
      />
      {/* 검색 결과 */}
    </div>
  );
}
```

## 구현 방법

1. **URL 파라미터 읽기**: useSearchParams, useRouter 등으로 URL 상태 읽기
2. **상태 업데이트**: URL 파라미터 업데이트로 상태 변경
3. **초기값 처리**: URL에 없을 때의 기본값 설정
4. **히스토리 관리**: push vs replace 선택 (히스토리 추가 여부)

## 장점

- URL 공유로 같은 상태 공유 가능
- 북마크 기능 자연스럽게 지원
- 브라우저 뒤로가기/앞으로가기 지원
- 새로고침해도 상태 유지
- SEO 친화적 (검색 엔진 크롤링 가능)

## 단점

- URL이 길어질 수 있음
- 복잡한 상태는 URL에 담기 어려움
- URL 파라미터 관리 복잡도 증가
- 민감한 정보는 URL에 포함하면 안 됨
