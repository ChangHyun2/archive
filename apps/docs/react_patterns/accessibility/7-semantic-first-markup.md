# Semantic-first Markup

## 개요

Semantic-first Markup은 div 지옥을 피하고 button, label, fieldset, nav, main 등 시맨틱 HTML 태그를 우선 사용하는 패턴입니다. 접근성뿐만 아니라 테스트 안정성과 유지보수성까지 함께 좋아집니다.

## 특징

- **의미 있는 태그 사용**: 각 요소의 목적에 맞는 HTML 태그 선택
- **접근성 향상**: 스크린 리더가 페이지 구조를 정확히 이해
- **SEO 개선**: 검색 엔진이 콘텐츠 구조를 더 잘 이해
- **유지보수성**: 코드만 봐도 구조가 명확함

## 예시

```tsx
// 나쁜 예: div 지옥
export const BadExample = () => {
  return (
    <div onClick={handleClick}>
      <div>
        <div>제목</div>
        <div>내용</div>
      </div>
    </div>
  );
};

// 좋은 예: 시맨틱 마크업
export const GoodExample = () => {
  return (
    <article>
      <header>
        <h1>제목</h1>
      </header>
      <main>
        <p>내용</p>
      </main>
      <footer>
        <button onClick={handleClick}>작업</button>
      </footer>
    </article>
  );
};

// 폼 컴포넌트 - 시맨틱 마크업
export const Form = () => {
  return (
    <form onSubmit={handleSubmit}>
      <fieldset>
        <legend>개인 정보</legend>
        <div>
          <label htmlFor="name">이름</label>
          <input id="name" type="text" required />
        </div>
        <div>
          <label htmlFor="email">이메일</label>
          <input id="email" type="email" required />
        </div>
      </fieldset>

      <fieldset>
        <legend>선택 사항</legend>
        <div>
          <label>
            <input type="checkbox" />
            뉴스레터 구독
          </label>
        </div>
      </fieldset>

      <button type="submit">제출</button>
    </form>
  );
};

// 네비게이션 - 시맨틱 마크업
export const Navigation = () => {
  return (
    <nav aria-label="메인 네비게이션">
      <ul>
        <li>
          <a href="/">홈</a>
        </li>
        <li>
          <a href="/about">소개</a>
        </li>
        <li>
          <a href="/contact">연락처</a>
        </li>
      </ul>
    </nav>
  );
};

// 카드 컴포넌트 - 시맨틱 마크업
export const Card = ({ title, content, actions }) => {
  return (
    <article className="card">
      <header>
        <h2>{title}</h2>
      </header>
      <main>
        <p>{content}</p>
      </main>
      {actions && (
        <footer>
          {actions}
        </footer>
      )}
    </article>
  );
};

// 리스트 컴포넌트 - 시맨틱 마크업
export const ProductList = ({ products }) => {
  return (
    <section aria-label="제품 목록">
      <ul>
        {products.map((product) => (
          <li key={product.id}>
            <article>
              <h3>{product.name}</h3>
              <p>{product.description}</p>
              <button>구매하기</button>
            </article>
          </li>
        ))}
      </ul>
    </section>
  );
};

// 다이얼로그 - 시맨틱 마크업
export const Dialog = ({ isOpen, onClose, title, children }) => {
  if (!isOpen) return null;

  return (
    <dialog open aria-labelledby="dialog-title">
      <header>
        <h2 id="dialog-title">{title}</h2>
        <button onClick={onClose} aria-label="닫기">×</button>
      </header>
      <main>
        {children}
      </main>
    </dialog>
  );
};
```

## 주요 시맨틱 태그

### 구조 태그
- `<header>`: 페이지 또는 섹션의 헤더
- `<nav>`: 네비게이션 링크
- `<main>`: 메인 콘텐츠
- `<article>`: 독립적인 콘텐츠
- `<section>`: 주제별 그룹
- `<aside>`: 사이드바, 부가 정보
- `<footer>`: 페이지 또는 섹션의 푸터

### 인터랙티브 태그
- `<button>`: 클릭 가능한 버튼
- `<a>`: 링크
- `<input>`, `<select>`, `<textarea>`: 폼 입력 요소
- `<label>`: 폼 레이블

### 그룹화 태그
- `<fieldset>`: 폼 필드 그룹
- `<legend>`: fieldset의 제목
- `<ul>`, `<ol>`, `<li>`: 리스트
- `<dl>`, `<dt>`, `<dd>`: 정의 리스트

## 구현 방법

1. **의도 파악**: 각 요소의 목적과 의미 파악
2. **적절한 태그 선택**: 목적에 맞는 시맨틱 태그 사용
3. **계층 구조**: 헤더, 메인, 푸터 등 논리적 구조 구성
4. **ARIA 보완**: 시맨틱 태그만으로 부족한 경우 ARIA 속성 추가
5. **스타일링**: 시맨틱 태그를 CSS로 스타일링 (div로 감쌀 필요 없음)

## 장점

- 접근성 자동 향상
- 코드 가독성 및 유지보수성 향상
- SEO 개선
- 테스트 안정성 향상 (시맨틱 선택자 사용)
- 브라우저 기본 동작 활용

## 단점

- 모든 디자인을 시맨틱 태그로 구현하기 어려울 수 있음
- 일부 복잡한 레이아웃은 여전히 div 필요
- 스타일링 시 태그 제약 고려 필요
