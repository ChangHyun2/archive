# Theme Provider + Multi-brand Theming

## 개요

Theme Provider + Multi-brand Theming은 Theme를 Context로 주입하고, 제품/브랜드별 테마를 쉽게 교체 가능하게 만드는 패턴입니다. "한 번 만든 컴포넌트로 여러 제품"이 목표일 때 필수입니다. Design Tokens와 함께 사용하여 완전한 테마 시스템을 구축합니다.

## 특징

- **중앙 집중식 테마 관리**: 모든 테마가 한 곳에서 관리됨
- **런타임 테마 전환**: 사용자가 테마를 동적으로 변경 가능
- **브랜드별 커스터마이징**: 각 브랜드/제품별 고유 테마 지원
- **컴포넌트 재사용**: 같은 컴포넌트로 다양한 테마 적용

## 예시

```tsx
// 테마 타입 정의
type Theme = {
  colors: {
    primary: string;
    secondary: string;
    background: string;
    text: string;
    border: string;
  };
  spacing: {
    xs: string;
    sm: string;
    md: string;
    lg: string;
    xl: string;
  };
  typography: {
    fontFamily: string;
    fontSize: {
      base: string;
      lg: string;
      xl: string;
    };
  };
  borderRadius: {
    sm: string;
    md: string;
    lg: string;
  };
};

// 브랜드별 테마 정의
const brandThemes: Record<string, Theme> = {
  default: {
    colors: {
      primary: '#0ea5e9',
      secondary: '#8b5cf6',
      background: '#ffffff',
      text: '#171717',
      border: '#e5e5e5',
    },
    spacing: {
      xs: '0.25rem',
      sm: '0.5rem',
      md: '1rem',
      lg: '1.5rem',
      xl: '2rem',
    },
    typography: {
      fontFamily: 'Inter, sans-serif',
      fontSize: {
        base: '1rem',
        lg: '1.125rem',
        xl: '1.25rem',
      },
    },
    borderRadius: {
      sm: '0.125rem',
      md: '0.375rem',
      lg: '0.5rem',
    },
  },
  brandA: {
    colors: {
      primary: '#ef4444',
      secondary: '#f59e0b',
      background: '#fafafa',
      text: '#1f2937',
      border: '#d1d5db',
    },
    spacing: {
      xs: '0.25rem',
      sm: '0.5rem',
      md: '1rem',
      lg: '1.5rem',
      xl: '2rem',
    },
    typography: {
      fontFamily: 'Roboto, sans-serif',
      fontSize: {
        base: '1rem',
        lg: '1.125rem',
        xl: '1.25rem',
      },
    },
    borderRadius: {
      sm: '0.25rem',
      md: '0.5rem',
      lg: '0.75rem',
    },
  },
  brandB: {
    colors: {
      primary: '#10b981',
      secondary: '#3b82f6',
      background: '#ffffff',
      text: '#111827',
      border: '#e5e7eb',
    },
    spacing: {
      xs: '0.25rem',
      sm: '0.5rem',
      md: '1rem',
      lg: '1.5rem',
      xl: '2rem',
    },
    typography: {
      fontFamily: 'Poppins, sans-serif',
      fontSize: {
        base: '1rem',
        lg: '1.125rem',
        xl: '1.25rem',
      },
    },
    borderRadius: {
      sm: '0.125rem',
      md: '0.375rem',
      lg: '0.5rem',
    },
  },
};

// 다크모드 테마 (각 브랜드별)
const darkThemes: Record<string, Theme> = {
  default: {
    ...brandThemes.default,
    colors: {
      ...brandThemes.default.colors,
      background: '#171717',
      text: '#fafafa',
      border: '#404040',
    },
  },
  brandA: {
    ...brandThemes.brandA,
    colors: {
      ...brandThemes.brandA.colors,
      background: '#1f2937',
      text: '#f9fafb',
      border: '#374151',
    },
  },
  brandB: {
    ...brandThemes.brandB,
    colors: {
      ...brandThemes.brandB.colors,
      background: '#111827',
      text: '#f9fafb',
      border: '#374151',
    },
  },
};

// Theme Context
type ThemeContextType = {
  theme: Theme;
  brand: string;
  mode: 'light' | 'dark';
  setBrand: (brand: string) => void;
  setMode: (mode: 'light' | 'dark') => void;
};

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Theme Provider
export const ThemeProvider = ({
  children,
  initialBrand = 'default',
  initialMode = 'light',
}: {
  children: React.ReactNode;
  initialBrand?: string;
  initialMode?: 'light' | 'dark';
}) => {
  const [brand, setBrand] = useState(initialBrand);
  const [mode, setMode] = useState<'light' | 'dark'>(initialMode);

  const theme = useMemo(() => {
    const themes = mode === 'dark' ? darkThemes : brandThemes;
    return themes[brand] || themes.default;
  }, [brand, mode]);

  // CSS 변수로 테마 주입
  useEffect(() => {
    const root = document.documentElement;
    root.style.setProperty('--color-primary', theme.colors.primary);
    root.style.setProperty('--color-secondary', theme.colors.secondary);
    root.style.setProperty('--color-background', theme.colors.background);
    root.style.setProperty('--color-text', theme.colors.text);
    root.style.setProperty('--color-border', theme.colors.border);
    root.style.setProperty('--spacing-xs', theme.spacing.xs);
    root.style.setProperty('--spacing-sm', theme.spacing.sm);
    root.style.setProperty('--spacing-md', theme.spacing.md);
    root.style.setProperty('--spacing-lg', theme.spacing.lg);
    root.style.setProperty('--spacing-xl', theme.spacing.xl);
    root.style.setProperty('--font-family', theme.typography.fontFamily);
    root.style.setProperty('--font-size-base', theme.typography.fontSize.base);
    root.style.setProperty('--font-size-lg', theme.typography.fontSize.lg);
    root.style.setProperty('--font-size-xl', theme.typography.fontSize.xl);
    root.style.setProperty('--border-radius-sm', theme.borderRadius.sm);
    root.style.setProperty('--border-radius-md', theme.borderRadius.md);
    root.style.setProperty('--border-radius-lg', theme.borderRadius.lg);
  }, [theme]);

  return (
    <ThemeContext.Provider value={{ theme, brand, mode, setBrand, setMode }}>
      {children}
    </ThemeContext.Provider>
  );
};

// useTheme 훅
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

// 테마를 사용하는 컴포넌트
export const Button = ({ children, variant = 'primary' }) => {
  const { theme } = useTheme();

  return (
    <button
      style={{
        backgroundColor: variant === 'primary' ? theme.colors.primary : theme.colors.secondary,
        color: theme.colors.background,
        padding: `${theme.spacing.sm} ${theme.spacing.md}`,
        borderRadius: theme.borderRadius.md,
        fontFamily: theme.typography.fontFamily,
        fontSize: theme.typography.fontSize.base,
        border: `1px solid ${theme.colors.border}`,
      }}
    >
      {children}
    </button>
  );
};

// CSS 변수 사용 (더 나은 방법)
const StyledButton = styled.button<{ variant?: 'primary' | 'secondary' }>`
  background-color: ${(props) =>
    props.variant === 'primary' ? 'var(--color-primary)' : 'var(--color-secondary)'};
  color: var(--color-background);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--border-radius-md);
  font-family: var(--font-family);
  font-size: var(--font-size-base);
  border: 1px solid var(--color-border);
`;

// 테마 전환 컴포넌트
export const ThemeSwitcher = () => {
  const { brand, mode, setBrand, setMode } = useTheme();

  return (
    <div>
      <select value={brand} onChange={(e) => setBrand(e.target.value)}>
        <option value="default">Default</option>
        <option value="brandA">Brand A</option>
        <option value="brandB">Brand B</option>
      </select>
      <button onClick={() => setMode(mode === 'light' ? 'dark' : 'light')}>
        {mode === 'light' ? '🌙' : '☀️'}
      </button>
    </div>
  );
};

// 사용 예시
export const App = () => {
  return (
    <ThemeProvider initialBrand="brandA" initialMode="light">
      <ThemeSwitcher />
      <Button variant="primary">Click me</Button>
    </ThemeProvider>
  );
};
```

## 구현 방법

1. **테마 타입 정의**: TypeScript로 테마 구조 타입 정의
2. **브랜드별 테마 객체 생성**: 각 브랜드/제품별 테마 정의
3. **Theme Context 생성**: React Context로 테마 전역 제공
4. **CSS 변수 주입**: 테마 값을 CSS 변수로 변환하여 스타일시트에서 사용
5. **테마 전환 로직**: 런타임에 테마 변경 가능하도록 구현
6. **컴포넌트에서 테마 소비**: useTheme 훅으로 테마 값 사용

## 장점

- 한 번 만든 컴포넌트로 여러 브랜드 지원
- 런타임 테마 전환 지원
- 유지보수성 향상
- 일관된 디자인 시스템 구축
- 다크모드 쉽게 지원

## 단점

- 초기 구조 설계 복잡도
- 테마 구조 변경 시 영향 범위 큼
- 성능 고려 필요 (CSS 변수 사용 시)
- 테마 간 일관성 유지 필요
