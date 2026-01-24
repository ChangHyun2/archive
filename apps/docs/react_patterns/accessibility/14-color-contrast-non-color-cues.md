# Color Contrast & Non-color Cues

## 개요

Color Contrast & Non-color Cues는 색상 대비(contrast) 기준을 지키고, 색상만으로 상태를 전달하지 않는 패턴입니다. 아이콘, 텍스트, 패턴 등을 추가하여 색맹 사용자나 저시력 사용자도 정보를 인지할 수 있도록 합니다. WCAG 2.1 AA 기준(일반 텍스트 4.5:1, 큰 텍스트 3:1)을 준수해야 합니다.

## 특징

- **대비 비율 준수**: WCAG 2.1 AA/AAA 기준 준수
- **다중 신호**: 색상 외에 아이콘, 텍스트, 패턴 등 추가
- **색맹 고려**: 색상만으로는 구분 불가능한 경우 대비
- **시각적 명확성**: 모든 사용자가 정보를 명확히 인지 가능

## 예시

```tsx
// 색상 대비 검증 유틸리티
export const getContrastRatio = (color1: string, color2: string): number => {
  const getLuminance = (color: string): number => {
    const rgb = hexToRgb(color);
    const [r, g, b] = rgb.map((val) => {
      val = val / 255;
      return val <= 0.03928 ? val / 12.92 : Math.pow((val + 0.055) / 1.055, 2.4);
    });
    return 0.2126 * r + 0.7152 * g + 0.0722 * b;
  };

  const l1 = getLuminance(color1);
  const l2 = getLuminance(color2);
  const lighter = Math.max(l1, l2);
  const darker = Math.min(l1, l2);
  return (lighter + 0.05) / (darker + 0.05);
};

// 대비 기준 확인
export const meetsContrastRatio = (
  foreground: string,
  background: string,
  level: 'AA' | 'AAA' = 'AA',
  isLargeText: boolean = false
): boolean => {
  const ratio = getContrastRatio(foreground, background);
  const required = level === 'AAA' 
    ? (isLargeText ? 4.5 : 7) 
    : (isLargeText ? 3 : 4.5);
  return ratio >= required;
};

// 상태 표시 - 색상 + 아이콘
export const StatusBadge = ({ status }: { status: 'success' | 'error' | 'warning' | 'info' }) => {
  const statusConfig = {
    success: {
      color: '#10b981',
      icon: '✓',
      label: '성공',
      bgColor: '#d1fae5',
    },
    error: {
      color: '#ef4444',
      icon: '✕',
      label: '오류',
      bgColor: '#fee2e2',
    },
    warning: {
      color: '#f59e0b',
      icon: '⚠',
      label: '경고',
      bgColor: '#fef3c7',
    },
    info: {
      color: '#3b82f6',
      icon: 'ℹ',
      label: '정보',
      bgColor: '#dbeafe',
    },
  };

  const config = statusConfig[status];

  return (
    <span
      style={{
        backgroundColor: config.bgColor,
        color: config.color,
        padding: '0.25rem 0.5rem',
        borderRadius: '0.25rem',
        display: 'inline-flex',
        alignItems: 'center',
        gap: '0.25rem',
      }}
      aria-label={config.label}
    >
      <span aria-hidden="true">{config.icon}</span>
      <span>{config.label}</span>
    </span>
  );
};

// 링크 - 색상 + 밑줄
export const AccessibleLink = ({ href, children }: { href: string; children: React.ReactNode }) => {
  return (
    <a
      href={href}
      style={{
        color: '#2563eb',
        textDecoration: 'underline',
        textDecorationThickness: '2px',
        textUnderlineOffset: '2px',
      }}
    >
      {children}
    </a>
  );
};

// 폼 에러 - 색상 + 아이콘 + 텍스트
export const FormError = ({ message }: { message: string }) => {
  return (
    <div
      role="alert"
      style={{
        color: '#dc2626',
        display: 'flex',
        alignItems: 'center',
        gap: '0.5rem',
        marginTop: '0.25rem',
      }}
    >
      <span aria-hidden="true">⚠</span>
      <span>{message}</span>
    </div>
  );
};

// 차트 - 색상 + 패턴
export const AccessibleChart = ({ data }: { data: Array<{ label: string; value: number; color: string }> }) => {
  const patterns = ['solid', 'diagonal', 'dots', 'horizontal'];

  return (
    <div role="img" aria-label="데이터 차트">
      {data.map((item, index) => (
        <div
          key={item.label}
          style={{
            backgroundColor: item.color,
            backgroundImage: `repeating-linear-gradient(
              45deg,
              transparent,
              transparent 10px,
              rgba(0,0,0,0.1) 10px,
              rgba(0,0,0,0.1) 20px
            )`,
          }}
          aria-label={`${item.label}: ${item.value}`}
        >
          <span>{item.label}</span>
          <span>{item.value}</span>
        </div>
      ))}
    </div>
  );
};

// 버튼 - 충분한 대비 + 명확한 텍스트
export const AccessibleButton = ({
  variant,
  children,
}: {
  variant: 'primary' | 'secondary';
  children: React.ReactNode;
}) => {
  const styles = {
    primary: {
      backgroundColor: '#0ea5e9', // 충분한 대비
      color: '#ffffff',
      border: '2px solid #0ea5e9',
    },
    secondary: {
      backgroundColor: '#ffffff',
      color: '#0ea5e9',
      border: '2px solid #0ea5e9',
    },
  };

  return (
    <button
      style={styles[variant]}
      aria-label={typeof children === 'string' ? children : undefined}
    >
      {children}
    </button>
  );
};

// 필수 필드 표시 - 색상 + 별표
export const RequiredField = ({ label, id }: { label: string; id: string }) => {
  return (
    <label htmlFor={id}>
      {label}
      <span aria-label="필수 항목" style={{ color: '#dc2626' }}>
        *
      </span>
    </label>
  );
};

// 테이블 - 색상 + 텍스트 스타일
export const AccessibleTable = ({ data }: { data: Array<{ name: string; status: string }> }) => {
  return (
    <table>
      <thead>
        <tr>
          <th>이름</th>
          <th>상태</th>
        </tr>
      </thead>
      <tbody>
        {data.map((row) => (
          <tr key={row.name}>
            <td>{row.name}</td>
            <td>
              <span
                style={{
                  color: row.status === 'active' ? '#10b981' : '#6b7280',
                  fontWeight: row.status === 'active' ? 'bold' : 'normal',
                }}
                aria-label={`상태: ${row.status === 'active' ? '활성' : '비활성'}`}
              >
                {row.status === 'active' ? '● 활성' : '○ 비활성'}
              </span>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

## 구현 방법

1. **대비 비율 계산**: 색상 대비 비율 계산 함수 구현
2. **WCAG 기준 확인**: AA/AAA 기준에 맞는지 검증
3. **다중 신호 제공**: 색상 외에 아이콘, 텍스트, 패턴 추가
4. **색상 팔레트 검증**: 디자인 시스템의 색상 조합 대비 비율 확인
5. **자동 검증 도구**: 빌드 타임에 대비 비율 검증
6. **시각적 테스트**: 색맹 시뮬레이션 도구로 테스트

## 장점

- 모든 사용자가 정보 인지 가능
- WCAG 기준 준수
- 법적 요구사항 충족
- 사용자 경험 향상
- 전문성 있는 디자인 시스템

## 단점

- 디자인 제약 증가
- 대비 비율 계산 및 검증 필요
- 색상 팔레트 선택 시 제한
- 추가 아이콘/텍스트로 인한 UI 복잡도 증가 가능
