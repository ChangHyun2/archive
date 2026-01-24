# Polymorphic Component Pattern (as prop)

## 개요

Polymorphic Component 패턴은 같은 컴포넌트를 `button`, `a`, `div` 등으로 렌더링 타입만 바꿔서 재사용성을 높이는 패턴입니다.

## 특징

- **타입 안정성**: TypeScript와 함께 사용 시 타입 안정성 보장
- **재사용성**: 하나의 컴포넌트로 다양한 HTML 요소 렌더링
- **유연성**: 사용자가 렌더링할 요소를 선택 가능

## 기본 구현

```tsx
type PolymorphicComponentProps<T extends React.ElementType> = {
  as?: T;
  children: React.ReactNode;
} & React.ComponentPropsWithoutRef<T>;

function Button<T extends React.ElementType = 'button'>({
  as,
  children,
  ...props
}: PolymorphicComponentProps<T>) {
  const Component = as || 'button';
  return <Component {...props}>{children}</Component>;
}
```

## 사용 예시

```tsx
// button으로 렌더링
<Button onClick={handleClick}>Click me</Button>

// a 태그로 렌더링
<Button as="a" href="/link">Link</Button>

// div로 렌더링
<Button as="div" role="button" tabIndex={0}>Div button</Button>
```

## TypeScript 고급 구현

```tsx
type PolymorphicRef<T extends React.ElementType> =
  React.ComponentPropsWithRef<T>['ref'];

type PolymorphicComponentProps<T extends React.ElementType> = {
  as?: T;
  children: React.ReactNode;
} & Omit<React.ComponentPropsWithoutRef<T>, 'as' | 'children'>;

type PolymorphicComponent = <T extends React.ElementType = 'button'>(
  props: PolymorphicComponentProps<T> & { ref?: PolymorphicRef<T> }
) => React.ReactElement | null;

const Button = React.forwardRef(function Button<T extends React.ElementType = 'button'>(
  { as, children, ...props }: PolymorphicComponentProps<T>,
  ref: PolymorphicRef<T>
) {
  const Component = as || 'button';
  return <Component ref={ref} {...props}>{children}</Component>;
}) as PolymorphicComponent;
```

## 장점

- 하나의 컴포넌트로 다양한 HTML 요소 렌더링
- 스타일과 로직은 유지하면서 렌더링 타입만 변경
- 접근성 고려 시 유용 (button vs link)

## 주의사항

- TypeScript 타입 정의가 복잡할 수 있음
- `as` prop에 따라 전달되는 props가 달라짐
- 접근성 속성 관리 필요 (role, tabIndex 등)

## 활용 사례

- 버튼 컴포넌트를 링크처럼 사용
- 카드 컴포넌트를 클릭 가능한 요소로 변환
- 공통 스타일을 가진 다양한 HTML 요소 렌더링
