# Roving Tabindex

## 개요

Roving Tabindex는 리스트, 탭, 메뉴 같은 "여러 아이템 중 하나만 포커스"되는 UI에서 사용하는 패턴입니다. Tab 키는 컨테이너 진입/이탈만 담당하고, 내부 이동은 Arrow 키로 처리합니다. 이를 통해 Tab 순서를 단순하게 유지하면서도 복잡한 컴포넌트 내부에서 효율적인 키보드 내비게이션을 제공합니다.

## 특징

- **단일 포커스**: 여러 아이템 중 하나만 tabIndex={0}, 나머지는 tabIndex={-1}
- **동적 tabIndex 관리**: 포커스가 이동할 때마다 tabIndex 값 업데이트
- **Tab 키 최소화**: Tab은 컨테이너 진입/이탈만, 내부는 Arrow 키로 이동
- **접근성 향상**: 복잡한 컴포넌트에서도 명확한 키보드 내비게이션

## 예시

```tsx
// Tabs 컴포넌트 - Roving Tabindex 적용
export const Tabs = ({ tabs }) => {
  const [activeTab, setActiveTab] = useState(0);
  const [focusedTab, setFocusedTab] = useState(0);

  const handleKeyDown = (e: React.KeyboardEvent, index: number) => {
    switch (e.key) {
      case 'ArrowRight':
        e.preventDefault();
        const nextIndex = index < tabs.length - 1 ? index + 1 : 0;
        setFocusedTab(nextIndex);
        break;
      case 'ArrowLeft':
        e.preventDefault();
        const prevIndex = index > 0 ? index - 1 : tabs.length - 1;
        setFocusedTab(prevIndex);
        break;
      case 'Home':
        e.preventDefault();
        setFocusedTab(0);
        break;
      case 'End':
        e.preventDefault();
        setFocusedTab(tabs.length - 1);
        break;
    }
  };

  return (
    <div role="tablist" aria-label="Main navigation">
      {tabs.map((tab, index) => (
        <button
          key={tab.id}
          role="tab"
          aria-selected={activeTab === index}
          aria-controls={`panel-${tab.id}`}
          id={`tab-${tab.id}`}
          tabIndex={focusedTab === index ? 0 : -1}
          onClick={() => {
            setActiveTab(index);
            setFocusedTab(index);
          }}
          onKeyDown={(e) => handleKeyDown(e, index)}
          onFocus={() => setFocusedTab(index)}
        >
          {tab.label}
        </button>
      ))}
    </div>
  );
};

// Menu 컴포넌트 - Roving Tabindex 적용
export const Menu = ({ items, onSelect }) => {
  const [focusedIndex, setFocusedIndex] = useState(0);

  const handleKeyDown = (e: React.KeyboardEvent, index: number) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        const nextIndex = index < items.length - 1 ? index + 1 : 0;
        setFocusedIndex(nextIndex);
        break;
      case 'ArrowUp':
        e.preventDefault();
        const prevIndex = index > 0 ? index - 1 : items.length - 1;
        setFocusedIndex(prevIndex);
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
    // 포커스된 아이템에 포커스 이동
    const item = document.querySelector(
      `[data-menu-item-index="${focusedIndex}"]`
    ) as HTMLElement;
    item?.focus();
  }, [focusedIndex]);

  return (
    <div role="menu" aria-label="Options">
      {items.map((item, index) => (
        <button
          key={item.id}
          role="menuitem"
          tabIndex={focusedIndex === index ? 0 : -1}
          data-menu-item-index={index}
          onClick={() => onSelect(item)}
          onKeyDown={(e) => handleKeyDown(e, index)}
          onFocus={() => setFocusedIndex(index)}
        >
          {item.label}
        </button>
      ))}
    </div>
  );
};

// Radio Group - Roving Tabindex 적용
export const RadioGroup = ({ options, value, onChange }) => {
  const [focusedIndex, setFocusedIndex] = useState(
    options.findIndex((opt) => opt.value === value)
  );

  const handleKeyDown = (e: React.KeyboardEvent, index: number) => {
    switch (e.key) {
      case 'ArrowDown':
      case 'ArrowRight':
        e.preventDefault();
        const nextIndex = index < options.length - 1 ? index + 1 : 0;
        setFocusedIndex(nextIndex);
        onChange(options[nextIndex].value);
        break;
      case 'ArrowUp':
      case 'ArrowLeft':
        e.preventDefault();
        const prevIndex = index > 0 ? index - 1 : options.length - 1;
        setFocusedIndex(prevIndex);
        onChange(options[prevIndex].value);
        break;
    }
  };

  return (
    <div role="radiogroup" aria-label="Options">
      {options.map((option, index) => (
        <label key={option.value}>
          <input
            type="radio"
            value={option.value}
            checked={value === option.value}
            onChange={() => {
              onChange(option.value);
              setFocusedIndex(index);
            }}
            tabIndex={focusedIndex === index ? 0 : -1}
            onKeyDown={(e) => handleKeyDown(e, index)}
            onFocus={() => setFocusedIndex(index)}
          />
          {option.label}
        </label>
      ))}
    </div>
  );
};
```

## 구현 방법

1. **초기 포커스 설정**: 첫 번째 또는 활성화된 아이템에 tabIndex={0} 설정
2. **나머지 아이템**: tabIndex={-1}로 설정하여 Tab 키로 접근 불가하게
3. **Arrow 키 처리**: Arrow 키로 포커스 이동 시 이전 아이템은 tabIndex={-1}, 새 아이템은 tabIndex={0}
4. **포커스 동기화**: focusedIndex 상태와 실제 DOM 포커스 동기화
5. **순환 내비게이션**: 첫/마지막 아이템에서 순환되도록 구현

## 장점

- Tab 순서 단순화
- 복잡한 컴포넌트에서도 효율적인 키보드 내비게이션
- 접근성 표준 준수
- 사용자 경험 향상

## 단점

- 상태 관리 복잡도 증가
- 포커스와 tabIndex 동기화 필요
- 구현 시 주의 깊은 테스트 필요
