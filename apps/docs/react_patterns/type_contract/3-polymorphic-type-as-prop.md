# Polymorphic 타입(as prop) 안전하게 만들기

## 개요

`as="a"`면 anchor props, `as="button"`이면 button props가 자동 추론되게 하는 패턴입니다. "좁은" Public Props + 내부 타입 분리로 외부에 노출되는 Props는 최소화하고, 내부 구현 타입은 분리해 변경 내성을 높입니다.

## 문제점

일반적으로 polymorphic 컴포넌트를 만들 때 타입 안전성을 잃기 쉽습니다:

```typescript
// ❌ 나쁜 예
function Button({ as, ...props }: { as?: string; [key: string]: any }) {
  const Component = as || 'button';
  return <Component {...props} />;
}

// 타입 안전성이 없고, 어떤 props를 전달할 수 있는지 알 수 없음
```

## 해결 방법

타입 안전한 Polymorphic 컴포넌트:

```typescript
// ✅ 좋은 예
import { ComponentPropsWithoutRef, ElementType } from 'react';

type PolymorphicProps<T extends ElementType> = {
  as?: T;
  children: React.ReactNode;
} & ComponentPropsWithoutRef<T>;

type ButtonProps<T extends ElementType = 'button'> = PolymorphicProps<T> & {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
};

function Button<T extends ElementType = 'button'>({
  as,
  variant,
  size,
  ...props
}: ButtonProps<T>) {
  const Component = as || ('button' as ElementType);
  
  return (
    <Component
      {...props}
      className={`btn btn-${variant} btn-${size}`}
    />
  );
}

// 사용 예시
<Button>Default Button</Button> // button 요소
<Button as="a" href="/page">Link Button</Button> // anchor 요소, href 자동 추론
<Button as="div" onClick={() => {}}>Div Button</Button> // div 요소
```

## 더 정교한 구현

```typescript
// ✅ 개선된 예시
type PolymorphicRef<T extends ElementType> =
  ComponentPropsWithRef<T>['ref'];

type OwnButtonProps = {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
};

type PolymorphicButtonProps<T extends ElementType> = OwnButtonProps & {
  as?: T;
} & Omit<ComponentPropsWithoutRef<T>, keyof OwnButtonProps>;

type ButtonComponent = <T extends ElementType = 'button'>(
  props: PolymorphicButtonProps<T> & { ref?: PolymorphicRef<T> }
) => React.ReactElement | null;

const Button = React.forwardRef(
  <T extends ElementType = 'button'>(
    { as, variant, size, ...props }: PolymorphicButtonProps<T>,
    ref?: PolymorphicRef<T>
  ) => {
    const Component = as || ('button' as ElementType);
    
    return (
      <Component
        ref={ref}
        {...props}
        className={`btn btn-${variant} btn-${size}`}
      />
    );
  }
) as ButtonComponent;
```

## Public Props 최소화 + 내부 타입 분리

```typescript
// ✅ Public API는 최소화
export type ButtonProps<T extends ElementType = 'button'> = {
  as?: T;
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
} & Omit<ComponentPropsWithoutRef<T>, 'children'>;

// 내부 구현 타입은 분리 (export하지 않음)
type InternalButtonState = {
  isPressed: boolean;
  isHovered: boolean;
};

type ButtonInternalProps = ButtonProps & {
  // 내부에서만 사용하는 props
  internalState?: InternalButtonState;
};

// 컴포넌트는 Public Props만 받음
export function Button<T extends ElementType = 'button'>(
  props: ButtonProps<T>
) {
  // 내부 구현은 자유롭게 변경 가능
  const [internalState, setInternalState] = useState<InternalButtonState>({
    isPressed: false,
    isHovered: false,
  });

  // ...
}
```

## 실제 사용 예시

```typescript
// 다양한 요소로 사용 가능
<Button>Button</Button>
<Button as="a" href="/link">Link</Button>
<Button as="div" role="button" tabIndex={0}>Div</Button>

// 각각의 경우 올바른 props가 자동 추론됨
// as="a"일 때는 href, target 등 anchor props 사용 가능
// as="button"일 때는 onClick, disabled 등 button props 사용 가능
```

## 장점

1. **타입 안전성**: 각 요소 타입에 맞는 props 자동 추론
2. **유연성**: 하나의 컴포넌트로 다양한 HTML 요소 렌더링
3. **변경 내성**: Public Props와 내부 타입 분리로 내부 구현 변경이 외부에 영향 없음
4. **재사용성**: 다양한 상황에서 동일한 컴포넌트 사용 가능
