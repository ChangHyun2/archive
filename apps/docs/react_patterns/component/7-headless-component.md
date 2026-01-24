# Headless Component Pattern

## 개요

Headless Component 패턴은 UI(스타일)는 없고 상태/행동/접근성 로직만 제공해서 디자인 시스템과 궁합이 좋은 패턴입니다.

## 특징

- **로직 분리**: 비즈니스 로직과 UI를 완전히 분리
- **재사용성**: 동일한 로직을 다양한 디자인으로 적용 가능
- **유연성**: 사용자가 원하는 UI를 자유롭게 구현

## 기본 구조

```tsx
function useToggle(initialValue = false) {
  const [on, setOn] = useState(initialValue);
  
  const toggle = useCallback(() => setOn(prev => !prev), []);
  const setOnValue = useCallback((value: boolean) => setOn(value), []);
  
  return {
    on,
    toggle,
    setOn: setOnValue,
    getToggleProps: (props = {}) => ({
      'aria-pressed': on,
      onClick: toggle,
      ...props,
    }),
  };
}
```

## 사용 예시

```tsx
function ToggleButton() {
  const { on, getToggleProps } = useToggle();
  
  return (
    <button {...getToggleProps()}>
      {on ? 'ON' : 'OFF'}
    </button>
  );
}

// 다른 디자인으로 재사용
function ToggleSwitch() {
  const { on, getToggleProps } = useToggle();
  
  return (
    <label className="switch">
      <input type="checkbox" {...getToggleProps()} />
      <span className="slider" />
    </label>
  );
}
```

## 복잡한 예시: Combobox

```tsx
function useCombobox({ items, onSelect }: { items: string[]; onSelect?: (item: string) => void }) {
  const [isOpen, setIsOpen] = useState(false);
  const [selectedItem, setSelectedItem] = useState<string | null>(null);
  const [inputValue, setInputValue] = useState('');
  
  const filteredItems = items.filter(item => 
    item.toLowerCase().includes(inputValue.toLowerCase())
  );
  
  const getInputProps = () => ({
    value: inputValue,
    onChange: (e: React.ChangeEvent<HTMLInputElement>) => {
      setInputValue(e.target.value);
      setIsOpen(true);
    },
    'aria-autocomplete': 'list' as const,
    'aria-expanded': isOpen,
  });
  
  const getMenuProps = () => ({
    role: 'listbox',
    'aria-label': 'Options',
  });
  
  const getItemProps = (item: string) => ({
    role: 'option',
    'aria-selected': selectedItem === item,
    onClick: () => {
      setSelectedItem(item);
      setInputValue(item);
      setIsOpen(false);
      onSelect?.(item);
    },
  });
  
  return {
    isOpen,
    selectedItem,
    filteredItems,
    getInputProps,
    getMenuProps,
    getItemProps,
  };
}
```

## 장점

- 로직과 UI를 완전히 분리
- 디자인 시스템과 독립적으로 동작
- 테스트하기 쉬움 (로직만 테스트)
- 다양한 디자인으로 재사용 가능

## 단점

- 초기 구현이 복잡할 수 있음
- 접근성 속성을 직접 관리해야 함
- 사용자가 더 많은 코드를 작성해야 함

## 활용 사례

- 디자인 시스템 라이브러리 (Radix UI, Headless UI)
- 폼 컴포넌트 (로직만 제공, 스타일은 사용자 정의)
- 복잡한 인터랙션 컴포넌트 (드롭다운, 모달, 토글)

## 유명한 라이브러리

- **Radix UI**: 완전한 Headless 컴포넌트 라이브러리
- **Headless UI**: Tailwind CSS 제작사에서 만든 Headless 컴포넌트
- **React Aria**: Adobe의 접근성 중심 Headless 컴포넌트
