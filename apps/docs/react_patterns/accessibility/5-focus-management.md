# Focus Management (포커스 제어)

## 개요

Focus Management는 모달이 열릴 때 포커스를 모달로 이동하고, 닫힐 때 원래 트리거 요소로 복원하는 패턴입니다. Focus Trap(모달 내부에서 포커스가 순환)과 스크롤 잠금까지 세트로 구현하는 것이 핵심입니다.

## 특징

- **포커스 이동**: 모달/다이얼로그 열림 시 포커스를 모달 내부로 이동
- **포커스 복원**: 모달 닫힘 시 원래 포커스 위치로 복원
- **Focus Trap**: 모달 내부에서 Tab 키를 눌러도 포커스가 모달 밖으로 나가지 않음
- **스크롤 잠금**: 모달 열림 시 배경 스크롤 방지

## 예시

```tsx
// Dialog 컴포넌트 - 포커스 관리
export const Dialog = ({ isOpen, onClose, title, children, triggerRef }) => {
  const dialogRef = useRef<HTMLDivElement>(null);
  const titleRef = useRef<HTMLHeadingElement>(null);
  const previousActiveElement = useRef<HTMLElement | null>(null);

  // 포커스 트랩 구현
  const trapFocus = (e: KeyboardEvent) => {
    if (!dialogRef.current || e.key !== 'Tab') return;

    const focusableElements = dialogRef.current.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    const firstElement = focusableElements[0] as HTMLElement;
    const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;

    if (e.shiftKey) {
      // Shift + Tab
      if (document.activeElement === firstElement) {
        e.preventDefault();
        lastElement?.focus();
      }
    } else {
      // Tab
      if (document.activeElement === lastElement) {
        e.preventDefault();
        firstElement?.focus();
      }
    }
  };

  useEffect(() => {
    if (isOpen) {
      // 현재 포커스된 요소 저장
      previousActiveElement.current = document.activeElement as HTMLElement;

      // 모달 열림: 포커스를 모달로 이동
      setTimeout(() => {
        titleRef.current?.focus();
      }, 0);

      // 포커스 트랩 활성화
      document.addEventListener('keydown', trapFocus);

      // 스크롤 잠금
      document.body.style.overflow = 'hidden';

      return () => {
        document.removeEventListener('keydown', trapFocus);
        document.body.style.overflow = '';
      };
    } else {
      // 모달 닫힘: 포커스 복원
      if (previousActiveElement.current) {
        previousActiveElement.current.focus();
      } else if (triggerRef?.current) {
        triggerRef.current.focus();
      }
    }
  }, [isOpen, triggerRef]);

  if (!isOpen) return null;

  return (
    <div
      ref={dialogRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="dialog-title"
      className="dialog-overlay"
      onClick={(e) => {
        if (e.target === e.currentTarget) {
          onClose();
        }
      }}
    >
      <div className="dialog-content">
        <h2
          id="dialog-title"
          ref={titleRef}
          tabIndex={-1}
        >
          {title}
        </h2>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
};

// 사용 예시
export const App = () => {
  const [isOpen, setIsOpen] = useState(false);
  const triggerRef = useRef<HTMLButtonElement>(null);

  return (
    <>
      <button
        ref={triggerRef}
        onClick={() => setIsOpen(true)}
      >
        Open Dialog
      </button>
      <Dialog
        isOpen={isOpen}
        onClose={() => setIsOpen(false)}
        title="Example Dialog"
        triggerRef={triggerRef}
      >
        <p>Dialog content here</p>
      </Dialog>
    </>
  );
};

// 커스텀 훅으로 포커스 관리 추상화
export const useFocusManagement = (
  isOpen: boolean,
  containerRef: React.RefObject<HTMLElement>,
  initialFocusRef?: React.RefObject<HTMLElement>
) => {
  const previousActiveElement = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (isOpen) {
      previousActiveElement.current = document.activeElement as HTMLElement;

      // 초기 포커스 설정
      setTimeout(() => {
        if (initialFocusRef?.current) {
          initialFocusRef.current.focus();
        } else if (containerRef.current) {
          const firstFocusable = containerRef.current.querySelector(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
          ) as HTMLElement;
          firstFocusable?.focus();
        }
      }, 0);

      return () => {
        // 포커스 복원
        previousActiveElement.current?.focus();
      };
    }
  }, [isOpen, containerRef, initialFocusRef]);
};
```

## 구현 방법

1. **포커스 저장**: 모달 열기 전 현재 포커스된 요소 저장
2. **초기 포커스 설정**: 모달 내부의 첫 번째 포커스 가능한 요소로 포커스 이동
3. **Focus Trap 구현**: Tab/Shift+Tab 키로 포커스가 모달 밖으로 나가지 않도록 처리
4. **포커스 복원**: 모달 닫힘 시 저장된 요소로 포커스 복원
5. **스크롤 잠금**: body의 overflow를 hidden으로 설정
6. **ESC 키 처리**: ESC 키로 모달 닫기 및 포커스 복원

## 장점

- 키보드 사용자 경험 향상
- 접근성 표준 준수
- 사용자가 어디에 있는지 명확히 인지
- 모달 사용 중 실수로 배경으로 포커스 이동 방지

## 단점

- 구현 복잡도 증가
- 여러 모달 중첩 시 관리 복잡
- 포커스 복원 실패 가능성 (요소가 DOM에서 제거된 경우)
