# Design Tokens (색/간격/타이포의 단일 진실)

## 개요

Design Tokens는 색, 폰트, spacing, radius, shadow를 토큰으로 관리하고 컴포넌트는 토큰만 소비하는 패턴입니다. 브랜드 스킨, 다크모드, 멀티 테마에 강한 구조를 제공합니다. 모든 디자인 값의 "단일 진실의 원천(Single Source of Truth)" 역할을 합니다.

## 특징

- **중앙 집중식 관리**: 모든 디자인 값이 한 곳에서 관리됨
- **일관성 보장**: 같은 토큰을 사용하여 일관된 디자인 유지
- **테마 지원**: 다크모드, 멀티 브랜드 등 쉽게 지원
- **유지보수성**: 디자인 변경 시 토큰만 수정하면 전체 반영

## 예시

```tsx
// Design Tokens 정의
export const tokens = {
  colors: {
    primary: {
      50: '#f0f9ff',
      100: '#e0f2fe',
      500: '#0ea5e9',
      600: '#0284c7',
      700: '#0369a1',
    },
    neutral: {
      50: '#fafafa',
      100: '#f5f5f5',
      500: '#737373',
      900: '#171717',
    },
    semantic: {
      success: '#10b981',
      error: '#ef4444',
      warning: '#f59e0b',
      info: '#3b82f6',
    },
  },
  spacing: {
    xs: '0.25rem',   // 4px
    sm: '0.5rem',    // 8px
    md: '1rem',      // 16px
    lg: '1.5rem',    // 24px
    xl: '2rem',      // 32px
  },
  typography: {
    fontFamily: {
      sans: ['Inter', 'system-ui', 'sans-serif'],
      mono: ['Fira Code', 'monospace'],
    },
    fontSize: {
      xs: '0.75rem',   // 12px
      sm: '0.875rem',  // 14px
      base: '1rem',    // 16px
      lg: '1.125rem',  // 18px
      xl: '1.25rem',   // 20px
    },
    fontWeight: {
      normal: 400,
      medium: 500,
      semibold: 600,
      bold: 700,
    },
  },
  borderRadius: {
    none: '0',
    sm: '0.125rem',   // 2px
    md: '0.375rem',   // 6px
    lg: '0.5rem',     // 8px
    full: '9999px',
  },
  shadow: {
    sm: '0 1px 2px 0 rgb(0 0 0 / 0.05)',
    md: '0 4px 6px -1px rgb(0 0 0 / 0.1)',
    lg: '0 10px 15px -3px rgb(0 0 0 / 0.1)',
  },
} as const;

// 다크모드 토큰
export const darkTokens = {
  ...tokens,
  colors: {
    ...tokens.colors,
    neutral: {
      50: '#171717',
      100: '#262626',
      500: '#a3a3a3',
      900: '#fafafa',
    },
  },
};

// Theme Context
export const ThemeContext = createContext(tokens);

export const ThemeProvider = ({ 
  children, 
  theme = 'light' 
}: {
  children: React.ReactNode;
  theme?: 'light' | 'dark';
}) => {
  const currentTokens = theme === 'dark' ? darkTokens : tokens;

  return (
    <ThemeContext.Provider value={currentTokens}>
      {children}
    </ThemeContext.Provider>
  );
};

// 토큰 사용 훅
export const useTokens = () => {
  return useContext(ThemeContext);
};

// 컴포넌트에서 토큰 사용
export const Button = ({ children, variant = 'primary' }) => {
  const tokens = useTokens();

  return (
    <button
      style={{
        backgroundColor: tokens.colors.primary[500],
        color: tokens.colors.neutral[50],
        padding: `${tokens.spacing.sm} ${tokens.spacing.md}`,
        borderRadius: tokens.borderRadius.md,
        fontSize: tokens.typography.fontSize.base,
        fontWeight: tokens.typography.fontWeight.medium,
        boxShadow: tokens.shadow.sm,
      }}
    >
      {children}
    </button>
  );
};

// CSS 변수로 토큰 노출 (더 나은 방법)
export const TokenProvider = ({ children, theme = 'light' }) => {
  const currentTokens = theme === 'dark' ? darkTokens : tokens;

  const cssVariables = {
    '--color-primary-500': currentTokens.colors.primary[500],
    '--color-primary-600': currentTokens.colors.primary[600],
    '--color-neutral-50': currentTokens.colors.neutral[50],
    '--color-neutral-900': currentTokens.colors.neutral[900],
    '--spacing-xs': currentTokens.spacing.xs,
    '--spacing-sm': currentTokens.spacing.sm,
    '--spacing-md': currentTokens.spacing.md,
    '--spacing-lg': currentTokens.spacing.lg,
    '--font-size-base': currentTokens.typography.fontSize.base,
    '--font-size-lg': currentTokens.typography.fontSize.lg,
    '--border-radius-md': currentTokens.borderRadius.md,
    '--shadow-sm': currentTokens.shadow.sm,
  };

  return (
    <div style={cssVariables as React.CSSProperties}>
      {children}
    </div>
  );
};

// CSS에서 토큰 사용
const ButtonStyled = styled.button`
  background-color: var(--color-primary-500);
  color: var(--color-neutral-50);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--border-radius-md);
  font-size: var(--font-size-base);
  box-shadow: var(--shadow-sm);

  &:hover {
    background-color: var(--color-primary-600);
  }
`;

// TypeScript 타입 안전성
type TokenPath = 
  | `colors.${keyof typeof tokens.colors}.${number}`
  | `spacing.${keyof typeof tokens.spacing}`
  | `typography.fontSize.${keyof typeof tokens.typography.fontSize}`;

export const getToken = (path: TokenPath) => {
  // 토큰 경로에서 값 가져오기
  const parts = path.split('.');
  let value: any = tokens;
  for (const part of parts) {
    value = value[part];
  }
  return value;
};
```

## 구현 방법

1. **토큰 구조 정의**: 색상, 간격, 타이포그래피 등 계층적 구조로 정의
2. **타입 안전성**: TypeScript로 토큰 타입 정의
3. **Context 제공**: Theme Context로 토큰 전역 제공
4. **CSS 변수 변환**: 런타임에 CSS 변수로 변환하여 스타일시트에서 사용
5. **테마 확장**: 다크모드, 멀티 브랜드 등 테마 확장 지원
6. **빌드 타임 최적화**: 정적 토큰은 빌드 타임에 CSS 변수로 변환

## 장점

- 디자인 일관성 보장
- 테마 전환 용이
- 유지보수성 향상
- 브랜드별 스킨 쉽게 적용
- 디자이너와 개발자 간 소통 용이

## 단점

- 초기 설정 복잡도
- 토큰 구조 설계 필요
- 모든 값을 토큰화하기 어려울 수 있음
- 런타임 성능 고려 필요 (CSS 변수 사용 시)
