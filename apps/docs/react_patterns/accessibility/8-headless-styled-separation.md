# Headless + Styled 분리 (Design System 핵심)

## 개요

Headless + Styled 분리는 접근성과 상호작용 로직(Headless)과 스타일(UI)을 분리하는 패턴입니다. 디자인 토큰이나 테마를 바꿔도 로직은 그대로 재사용할 수 있어 디자인 시스템과 궁합이 좋습니다.

## 특징

- **관심사 분리**: 로직과 스타일을 완전히 분리
- **재사용성**: 같은 로직으로 다양한 스타일 적용 가능
- **테스트 용이성**: 로직과 스타일을 독립적으로 테스트
- **유지보수성**: 스타일 변경이 로직에 영향 없음

## 예시

```tsx
// Headless 컴포넌트 - 로직만 담당
export const useHeadlessDialog = () => {
  const [isOpen, setIsOpen] = useState(false);
  const triggerRef = useRef<HTMLElement>(null);
  const dialogRef = useRef<HTMLDivElement>(null);

  const open = () => setIsOpen(true);
  const close = () => setIsOpen(false);

  // 포커스 관리
  useEffect(() => {
    if (isOpen) {
      const firstFocusable = dialogRef.current?.querySelector(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      ) as HTMLElement;
      firstFocusable?.focus();
    } else {
      triggerRef.current?.focus();
    }
  }, [isOpen]);

  // 포커스 트랩
  useEffect(() => {
    if (!isOpen) return;

    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Escape') close();
      if (e.key === 'Tab' && dialogRef.current) {
        const focusableElements = dialogRef.current.querySelectorAll(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );
        const first = focusableElements[0] as HTMLElement;
        const last = focusableElements[focusableElements.length - 1] as HTMLElement;

        if (e.shiftKey && document.activeElement === first) {
          e.preventDefault();
          last?.focus();
        } else if (!e.shiftKey && document.activeElement === last) {
          e.preventDefault();
          first?.focus();
        }
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isOpen]);

  return {
    isOpen,
    open,
    close,
    triggerProps: {
      ref: triggerRef,
      onClick: open,
      'aria-haspopup': 'dialog',
      'aria-expanded': isOpen,
    },
    dialogProps: {
      ref: dialogRef,
      role: 'dialog',
      'aria-modal': 'true',
      'aria-hidden': !isOpen,
    },
  };
};

// Styled 컴포넌트 - 스타일만 담당
export const StyledDialog = ({ 
  trigger, 
  title, 
  children 
}: {
  trigger: React.ReactNode;
  title: string;
  children: React.ReactNode;
}) => {
  const { isOpen, close, triggerProps, dialogProps } = useHeadlessDialog();

  return (
    <>
      <div {...triggerProps}>{trigger}</div>
      {isOpen && (
        <div className="dialog-overlay" onClick={close}>
          <div className="dialog-content" {...dialogProps}>
            <h2>{title}</h2>
            {children}
            <button onClick={close}>Close</button>
          </div>
        </div>
      )}
    </>
  );
};

// 다른 스타일로 재사용
export const MinimalDialog = ({ 
  trigger, 
  title, 
  children 
}: {
  trigger: React.ReactNode;
  title: string;
  children: React.ReactNode;
}) => {
  const { isOpen, close, triggerProps, dialogProps } = useHeadlessDialog();

  return (
    <>
      <button {...triggerProps} className="minimal-trigger">
        {trigger}
      </button>
      {isOpen && (
        <div className="minimal-overlay" onClick={close}>
          <div className="minimal-dialog" {...dialogProps}>
            <header>
              <h3>{title}</h3>
              <button onClick={close} aria-label="닫기">×</button>
            </header>
            <main>{children}</main>
          </div>
        </div>
      )}
    </>
  );
};

// Headless Tabs
export const useHeadlessTabs = (defaultIndex = 0) => {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);
  const [focusedIndex, setFocusedIndex] = useState(defaultIndex);

  const handleKeyDown = (index: number, e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowRight':
        e.preventDefault();
        const next = index < tabs.length - 1 ? index + 1 : 0;
        setFocusedIndex(next);
        setActiveIndex(next);
        break;
      case 'ArrowLeft':
        e.preventDefault();
        const prev = index > 0 ? index - 1 : tabs.length - 1;
        setFocusedIndex(prev);
        setActiveIndex(prev);
        break;
    }
  };

  return {
    activeIndex,
    focusedIndex,
    setActiveIndex,
    setFocusedIndex,
    handleKeyDown,
    getTabProps: (index: number) => ({
      role: 'tab',
      'aria-selected': activeIndex === index,
      'aria-controls': `panel-${index}`,
      id: `tab-${index}`,
      tabIndex: focusedIndex === index ? 0 : -1,
      onClick: () => {
        setActiveIndex(index);
        setFocusedIndex(index);
      },
      onKeyDown: (e: React.KeyboardEvent) => handleKeyDown(index, e),
    }),
    getPanelProps: (index: number) => ({
      role: 'tabpanel',
      id: `panel-${index}`,
      'aria-labelledby': `tab-${index}`,
      hidden: activeIndex !== index,
    }),
  };
};

// Styled Tabs
export const StyledTabs = ({ tabs }: { tabs: Array<{ label: string; content: React.ReactNode }> }) => {
  const { activeIndex, getTabProps, getPanelProps } = useHeadlessTabs();

  return (
    <div className="tabs-container">
      <div role="tablist" className="tabs-list">
        {tabs.map((tab, index) => (
          <button key={index} {...getTabProps(index)} className="tab-button">
            {tab.label}
          </button>
        ))}
      </div>
      {tabs.map((tab, index) => (
        <div key={index} {...getPanelProps(index)} className="tab-panel">
          {tab.content}
        </div>
      ))}
    </div>
  );
};
```

## 구현 방법

1. **Headless 훅/컴포넌트 생성**: 접근성 로직과 상태 관리만 포함
2. **Props 반환**: 스타일 컴포넌트에 전달할 props 객체 반환
3. **스타일 컴포넌트 분리**: 스타일과 마크업만 담당하는 컴포넌트 생성
4. **조합**: Headless 로직과 다양한 스타일을 자유롭게 조합
5. **디자인 토큰 활용**: 스타일 컴포넌트에서 디자인 토큰 사용

## 장점

- 로직 재사용성 극대화
- 스타일 변경이 로직에 영향 없음
- 테스트 용이성 향상
- 디자인 시스템 구축에 최적
- 다양한 테마 지원 용이

## 단점

- 초기 구조 설계 필요
- 추상화 레벨이 높아 학습 곡선 존재
- 간단한 컴포넌트에는 오버엔지니어링일 수 있음
