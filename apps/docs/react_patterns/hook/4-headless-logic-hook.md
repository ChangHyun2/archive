# Headless Logic Hook

## 개요

UI 없이 로직만 제공하는 Hook입니다. 디자인 시스템이나 다양한 UI에 재사용 가능합니다.

## 핵심 원칙

- **로직과 UI 분리**: Hook은 상태와 로직만 제공, UI는 컴포넌트에서 담당
- **재사용성**: 동일한 로직을 다양한 UI 스타일로 구현 가능
- **유연성**: 디자인 시스템과 독립적으로 로직 관리

## 예시

### useTabs Hook

```tsx
function useTabs(initialTab = 0) {
  const [activeTab, setActiveTab] = useState(initialTab);
  
  const selectTab = useCallback((index) => {
    setActiveTab(index);
  }, []);
  
  const nextTab = useCallback(() => {
    setActiveTab(prev => prev + 1);
  }, []);
  
  const prevTab = useCallback(() => {
    setActiveTab(prev => prev - 1);
  }, []);
  
  return {
    activeTab,
    selectTab,
    nextTab,
    prevTab,
  };
}
```

### 다양한 UI로 사용

```tsx
// Material Design 스타일
function MaterialTabs({ tabs }) {
  const { activeTab, selectTab } = useTabs();
  
  return (
    <div>
      <div className="tabs-header">
        {tabs.map((tab, index) => (
          <button
            key={index}
            onClick={() => selectTab(index)}
            className={activeTab === index ? 'active' : ''}
          >
            {tab.label}
          </button>
        ))}
      </div>
      <div className="tabs-content">
        {tabs[activeTab].content}
      </div>
    </div>
  );
}

// Bootstrap 스타일
function BootstrapTabs({ tabs }) {
  const { activeTab, selectTab } = useTabs();
  
  return (
    <ul className="nav nav-tabs">
      {tabs.map((tab, index) => (
        <li key={index} className="nav-item">
          <button
            className={`nav-link ${activeTab === index ? 'active' : ''}`}
            onClick={() => selectTab(index)}
          >
            {tab.label}
          </button>
        </li>
      ))}
    </ul>
  );
}

// 커스텀 스타일
function CustomTabs({ tabs }) {
  const { activeTab, selectTab, nextTab, prevTab } = useTabs();
  
  return (
    <div className="custom-tabs">
      <button onClick={prevTab}>←</button>
      {tabs[activeTab].content}
      <button onClick={nextTab}>→</button>
    </div>
  );
}
```

### useCombobox Hook

```tsx
function useCombobox(items, options = {}) {
  const {
    filterFn = (item, query) => item.label.includes(query),
    onSelect,
  } = options;
  
  const [isOpen, setIsOpen] = useState(false);
  const [inputValue, setInputValue] = useState('');
  const [selectedItem, setSelectedItem] = useState(null);
  
  const filteredItems = useMemo(() => {
    if (!inputValue) return items;
    return items.filter(item => filterFn(item, inputValue));
  }, [items, inputValue, filterFn]);
  
  const handleSelect = useCallback((item) => {
    setSelectedItem(item);
    setInputValue(item.label);
    setIsOpen(false);
    onSelect?.(item);
  }, [onSelect]);
  
  const handleInputChange = useCallback((value) => {
    setInputValue(value);
    setIsOpen(true);
  }, []);
  
  return {
    isOpen,
    setIsOpen,
    inputValue,
    setInputValue: handleInputChange,
    selectedItem,
    filteredItems,
    selectItem: handleSelect,
  };
}
```

### useDisclosure Hook

```tsx
function useDisclosure(initialState = false) {
  const [isOpen, setIsOpen] = useState(initialState);
  
  const open = useCallback(() => setIsOpen(true), []);
  const close = useCallback(() => setIsOpen(false), []);
  const toggle = useCallback(() => setIsOpen(prev => !prev), []);
  
  return {
    isOpen,
    open,
    close,
    toggle,
  };
}
```

### useDisclosure 사용 예시

```tsx
// Modal
function Modal({ children }) {
  const { isOpen, open, close } = useDisclosure();
  
  return (
    <>
      <button onClick={open}>Open Modal</button>
      {isOpen && (
        <div className="modal-overlay" onClick={close}>
          <div className="modal-content" onClick={e => e.stopPropagation()}>
            {children}
            <button onClick={close}>Close</button>
          </div>
        </div>
      )}
    </>
  );
}

// Dropdown
function Dropdown({ trigger, menu }) {
  const { isOpen, toggle, close } = useDisclosure();
  
  return (
    <div className="dropdown">
      <div onClick={toggle}>{trigger}</div>
      {isOpen && (
        <div className="dropdown-menu">
          {menu}
        </div>
      )}
    </div>
  );
}

// Accordion
function Accordion({ title, content }) {
  const { isOpen, toggle } = useDisclosure();
  
  return (
    <div className="accordion">
      <button onClick={toggle}>{title}</button>
      {isOpen && <div className="accordion-content">{content}</div>}
    </div>
  );
}
```

## 장점

1. **UI 독립성**: 로직이 특정 UI 라이브러리에 종속되지 않음
2. **재사용성**: 동일한 로직을 다양한 디자인으로 구현 가능
3. **테스트 용이성**: UI 없이 로직만 테스트 가능
4. **유지보수성**: 로직 변경이 UI에 영향을 주지 않음

## 사용 시기

- 동일한 로직을 다양한 UI 스타일로 구현해야 할 때
- 디자인 시스템과 로직을 분리하고 싶을 때
- UI 라이브러리를 교체할 가능성이 있을 때
