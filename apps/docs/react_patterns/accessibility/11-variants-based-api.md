# Variants 기반 API (일관된 스타일 확장)

## 개요

Variants 기반 API는 size, variant, tone 같은 제한된 변형만 허용하는 패턴입니다. 자유로운 className 덕지덕지는 디자인 시스템 붕괴의 지름길이므로, 미리 정의된 variants만 사용하도록 제한합니다. 이를 통해 일관된 디자인과 예측 가능한 API를 제공합니다.

## 특징

- **제한된 변형**: 미리 정의된 variants만 허용
- **타입 안전성**: TypeScript로 variants 타입 제한
- **일관성 보장**: 모든 컴포넌트가 같은 variants 체계 사용
- **예측 가능성**: 사용자가 어떤 variants를 사용할 수 있는지 명확함

## 예시

```tsx
// Variants 타입 정의
type ButtonSize = 'sm' | 'md' | 'lg';
type ButtonVariant = 'primary' | 'secondary' | 'outline' | 'ghost';
type ButtonTone = 'default' | 'success' | 'error' | 'warning';

type ButtonProps = {
  size?: ButtonSize;
  variant?: ButtonVariant;
  tone?: ButtonTone;
  children: React.ReactNode;
  className?: string; // 제한적으로만 허용
} & React.ButtonHTMLAttributes<HTMLButtonElement>;

// Variants 기반 스타일 매핑
const buttonStyles = {
  size: {
    sm: 'px-2 py-1 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
  },
  variant: {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-600 text-white hover:bg-gray-700',
    outline: 'border-2 border-blue-600 text-blue-600 hover:bg-blue-50',
    ghost: 'text-blue-600 hover:bg-blue-50',
  },
  tone: {
    default: '',
    success: 'bg-green-600 hover:bg-green-700',
    error: 'bg-red-600 hover:bg-red-700',
    warning: 'bg-yellow-600 hover:bg-yellow-700',
  },
};

// Variants 기반 Button 컴포넌트
export const Button = ({
  size = 'md',
  variant = 'primary',
  tone = 'default',
  children,
  className = '',
  ...props
}: ButtonProps) => {
  const baseStyles = 'font-medium rounded transition-colors';
  const sizeStyles = buttonStyles.size[size];
  const variantStyles = tone === 'default' 
    ? buttonStyles.variant[variant]
    : buttonStyles.tone[tone];
  
  // className은 제한적으로만 병합 (디자인 시스템 우선)
  const combinedClassName = `${baseStyles} ${sizeStyles} ${variantStyles} ${className}`.trim();

  return (
    <button className={combinedClassName} {...props}>
      {children}
    </button>
  );
};

// cva (class-variance-authority) 사용 예시
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  // base styles
  'font-medium rounded transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2',
  {
    variants: {
      size: {
        sm: 'px-2 py-1 text-sm',
        md: 'px-4 py-2 text-base',
        lg: 'px-6 py-3 text-lg',
      },
      variant: {
        primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
        secondary: 'bg-gray-600 text-white hover:bg-gray-700 focus:ring-gray-500',
        outline: 'border-2 border-blue-600 text-blue-600 hover:bg-blue-50 focus:ring-blue-500',
        ghost: 'text-blue-600 hover:bg-blue-50 focus:ring-blue-500',
      },
      tone: {
        default: '',
        success: 'bg-green-600 hover:bg-green-700 focus:ring-green-500',
        error: 'bg-red-600 hover:bg-red-700 focus:ring-red-500',
        warning: 'bg-yellow-600 hover:bg-yellow-700 focus:ring-yellow-500',
      },
    },
    defaultVariants: {
      size: 'md',
      variant: 'primary',
      tone: 'default',
    },
  }
);

type ButtonVariants = VariantProps<typeof buttonVariants>;

export const ButtonWithCVA = ({
  size,
  variant,
  tone,
  children,
  className,
  ...props
}: ButtonVariants & {
  children: React.ReactNode;
  className?: string;
} & React.ButtonHTMLAttributes<HTMLButtonElement>) => {
  return (
    <button
      className={buttonVariants({ size, variant, tone, className })}
      {...props}
    >
      {children}
    </button>
  );
};

// Card 컴포넌트 - Variants 적용
type CardVariant = 'default' | 'elevated' | 'outlined';
type CardSize = 'sm' | 'md' | 'lg';

const cardVariants = cva(
  'rounded-lg transition-shadow',
  {
    variants: {
      variant: {
        default: 'bg-white',
        elevated: 'bg-white shadow-lg',
        outlined: 'bg-white border-2 border-gray-200',
      },
      size: {
        sm: 'p-4',
        md: 'p-6',
        lg: 'p-8',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
);

export const Card = ({
  variant,
  size,
  children,
  className,
  ...props
}: VariantProps<typeof cardVariants> & {
  children: React.ReactNode;
  className?: string;
} & React.HTMLAttributes<HTMLDivElement>) => {
  return (
    <div className={cardVariants({ variant, size, className })} {...props}>
      {children}
    </div>
  );
};

// Badge 컴포넌트
const badgeVariants = cva(
  'inline-flex items-center rounded-full font-medium',
  {
    variants: {
      variant: {
        default: 'bg-gray-100 text-gray-800',
        primary: 'bg-blue-100 text-blue-800',
        success: 'bg-green-100 text-green-800',
        error: 'bg-red-100 text-red-800',
        warning: 'bg-yellow-100 text-yellow-800',
      },
      size: {
        sm: 'px-2 py-0.5 text-xs',
        md: 'px-2.5 py-0.5 text-sm',
        lg: 'px-3 py-1 text-base',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
);

export const Badge = ({
  variant,
  size,
  children,
  className,
  ...props
}: VariantProps<typeof badgeVariants> & {
  children: React.ReactNode;
  className?: string;
} & React.HTMLAttributes<HTMLSpanElement>) => {
  return (
    <span className={badgeVariants({ variant, size, className })} {...props}>
      {children}
    </span>
  );
};

// 사용 예시
export const Example = () => {
  return (
    <div>
      <Button size="sm" variant="primary">Small Primary</Button>
      <Button size="md" variant="secondary" tone="success">Success</Button>
      <Button size="lg" variant="outline">Large Outline</Button>
      
      <Card variant="elevated" size="lg">
        <h2>Card Title</h2>
        <p>Card content</p>
      </Card>
      
      <Badge variant="success" size="sm">New</Badge>
    </div>
  );
};
```

## 구현 방법

1. **Variants 타입 정의**: 각 컴포넌트의 variants를 타입으로 정의
2. **스타일 매핑 객체 생성**: variants 값에 따른 스타일 매핑
3. **cva 라이브러리 활용**: class-variance-authority로 variants 관리
4. **타입 안전성 보장**: TypeScript로 variants 타입 제한
5. **className 제한**: className prop은 제한적으로만 허용
6. **일관된 네이밍**: 모든 컴포넌트에서 같은 variants 이름 사용 (size, variant 등)

## 장점

- 디자인 시스템 일관성 유지
- 예측 가능한 API
- 타입 안전성
- 자동완성 지원
- 유지보수성 향상

## 단점

- 초기 variants 설계 필요
- 모든 케이스를 커버하기 어려울 수 있음
- 새로운 variant 추가 시 신중한 검토 필요
- className 제한으로 인한 유연성 감소
