# A11y-first Primitives

## 개요

A11y-first Primitives는 Button, Input, Dialog, Popover 같은 기본 컴포넌트를 처음부터 접근성(a11y) 규격으로 만든 뒤 전사에서 재사용하는 패턴입니다. 즉흥적으로 화면마다 aria 속성을 붙이는 방식은 결국 유지보수성과 일관성 문제를 야기합니다.

## 특징

- **초기 설계 단계부터 접근성 고려**: 컴포넌트 설계 시점부터 WAI-ARIA 가이드라인 준수
- **전사 재사용**: 한 번 만들어진 접근성 컴포넌트를 모든 프로젝트에서 재사용
- **일관성 보장**: 모든 화면에서 동일한 접근성 수준 유지
- **유지보수성 향상**: 접근성 로직이 한 곳에 집중되어 관리 용이

## 예시

```tsx
// 접근성 기본 프리미티브 컴포넌트
export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ children, disabled, ...props }, ref) => {
    return (
      <button
        ref={ref}
        disabled={disabled}
        aria-disabled={disabled}
        {...props}
      >
        {children}
      </button>
    );
  }
);

// Dialog 컴포넌트
export const Dialog = ({ isOpen, onClose, children, ...props }) => {
  return (
    <>
      {isOpen && (
        <div
          role="dialog"
          aria-modal="true"
          aria-labelledby="dialog-title"
          {...props}
        >
          {children}
        </div>
      )}
    </>
  );
};
```

## 구현 방법

1. **기본 컴포넌트 라이브러리 구축**: Button, Input, Dialog 등 핵심 컴포넌트를 접근성 기준으로 구현
2. **WAI-ARIA 가이드라인 준수**: 각 컴포넌트 타입에 맞는 role, aria 속성 적용
3. **키보드 내비게이션 지원**: 모든 인터랙티브 컴포넌트에 키보드 이벤트 처리
4. **포커스 관리**: 포커스 트랩, 포커스 복원 등 포커스 관련 로직 포함
5. **디자인 시스템 통합**: 스타일과 접근성 로직을 함께 제공

## 장점

- 접근성 문제를 사전에 예방
- 개발자가 매번 접근성을 고려할 필요 없음
- 일관된 사용자 경험 제공
- 접근성 테스트 비용 절감

## 단점

- 초기 구축 비용이 높음
- 모든 케이스를 커버하기 어려울 수 있음
- 디자인 변경 시 접근성 로직도 함께 검토 필요
