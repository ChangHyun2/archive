# Keyboard-first Interaction

## 개요

Keyboard-first Interaction은 마우스 없이도 Tab, Shift+Tab, Enter, Esc, Arrow 키로 모든 기능을 사용할 수 있게 만드는 패턴입니다. 특히 Menu, Select, Autocomplete 같은 복잡한 인터랙티브 컴포넌트에서 필수적입니다.

## 특징

- **키보드 우선 설계**: 모든 인터랙션을 키보드로 먼저 구현
- **표준 키보드 패턴**: Tab, Enter, Esc, Arrow keys 등 표준 키보드 내비게이션 지원
- **포커스 관리**: 명확한 포커스 순서와 시각적 표시
- **접근성 향상**: 스크린 리더 사용자와 키보드 전용 사용자 지원

## 예시

```tsx
// Menu 컴포넌트 - 키보드 내비게이션
export const Menu = ({ items, onSelect }) => {
  const [focusedIndex, setFocusedIndex] = useState(-1);
  const menuRef = useRef<HTMLDivElement>(null);

  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex((prev) => 
          prev < items.length - 1 ? prev + 1 : 0
        );
        break;
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex((prev) => 
          prev > 0 ? prev - 1 : items.length - 1
        );
        break;
      case 'Enter':
      case ' ':
        e.preventDefault();
        if (focusedIndex >= 0) {
          onSelect(items[focusedIndex]);
        }
        break;
      case 'Escape':
        e.preventDefault();
        // 메뉴 닫기
        break;
      case 'Home':
        e.preventDefault();
        setFocusedIndex(0);
        break;
      case 'End':
        e.preventDefault();
        setFocusedIndex(items.length - 1);
        break;
    }
  };

  useEffect(() => {
    if (focusedIndex >= 0 && menuRef.current) {
      const item = menuRef.current.children[focusedIndex] as HTMLElement;
      item?.focus();
    }
  }, [focusedIndex]);

  return (
    <div
      ref={menuRef}
      role="menu"
      onKeyDown={handleKeyDown}
      tabIndex={0}
    >
      {items.map((item, index) => (
        <button
          key={item.id}
          role="menuitem"
          tabIndex={-1}
          onFocus={() => setFocusedIndex(index)}
        >
          {item.label}
        </button>
      ))}
    </div>
  );
};

// Autocomplete 컴포넌트
export const Autocomplete = ({ options, onSelect }) => {
  const [isOpen, setIsOpen] = useState(false);
  const [focusedIndex, setFocusedIndex] = useState(-1);
  const inputRef = useRef<HTMLInputElement>(null);

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (!isOpen) {
      if (e.key === 'ArrowDown' || e.key === 'Enter') {
        setIsOpen(true);
      }
      return;
    }

    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex((prev) => 
          prev < options.length - 1 ? prev + 1 : 0
        );
        break;
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex((prev) => 
          prev > 0 ? prev - 1 : options.length - 1
        );
        break;
      case 'Enter':
        e.preventDefault();
        if (focusedIndex >= 0) {
          onSelect(options[focusedIndex]);
          setIsOpen(false);
        }
        break;
      case 'Escape':
        e.preventDefault();
        setIsOpen(false);
        inputRef.current?.focus();
        break;
    }
  };

  return (
    <div>
      <input
        ref={inputRef}
        type="text"
        role="combobox"
        aria-expanded={isOpen}
        aria-autocomplete="list"
        aria-controls="autocomplete-list"
        onKeyDown={handleKeyDown}
        onFocus={() => setIsOpen(true)}
      />
      {isOpen && (
        <ul
          id="autocomplete-list"
          role="listbox"
        >
          {options.map((option, index) => (
            <li
              key={option.id}
              role="option"
              aria-selected={focusedIndex === index}
              tabIndex={-1}
            >
              {option.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};
```

## 구현 방법

1. **키보드 이벤트 핸들러 구현**: onKeyDown으로 주요 키 처리
2. **포커스 순서 정의**: Tab 순서가 논리적으로 흐르도록 설계
3. **Arrow 키 내비게이션**: 리스트/메뉴 내부 이동은 Arrow 키로 처리
4. **Enter/Space 처리**: 선택/활성화 동작 구현
5. **Esc 키 처리**: 모달 닫기, 메뉴 닫기 등 취소 동작
6. **포커스 복원**: 모달/메뉴 닫힘 후 원래 포커스 위치로 복원

## 장점

- 키보드 사용자 완전 지원
- 스크린 리더 사용자 경험 향상
- 접근성 표준 준수
- 모든 사용자에게 일관된 경험 제공

## 단점

- 구현 복잡도 증가
- 모든 케이스에 대한 키보드 동작 정의 필요
- 테스트 범위 확대
