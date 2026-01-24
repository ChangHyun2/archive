# Component API Surface Minimization

## 개요

Component API Surface Minimization 패턴은 public props를 최소화하고, 불필요한 구현 디테일 노출을 막는 설계 패턴입니다. 이는 캡슐화, 변경 내성, 사용 난이도 측면에서 핵심적인 패턴입니다.

## 문제: 과도한 API 노출

```tsx
// 나쁜 예: 내부 구현 디테일을 모두 노출
function Dropdown({
  isOpen,
  setIsOpen,
  selectedItem,
  setSelectedItem,
  items,
  onSelect,
  onOpen,
  onClose,
  // ... 내부 상태와 로직을 모두 노출
}) {
  // 사용자가 모든 것을 직접 관리해야 함
}
```

## 해결: 최소한의 API

```tsx
// 좋은 예: 필요한 것만 노출
function Dropdown({
  items,
  onSelect,
  defaultValue,
}: {
  items: string[];
  onSelect?: (item: string) => void;
  defaultValue?: string;
}) {
  // 내부 상태와 로직은 모두 캡슐화
  const [isOpen, setIsOpen] = useState(false);
  const [selectedItem, setSelectedItem] = useState(defaultValue);
  
  // ...
}
```

## 원칙

### 1. 필요한 것만 노출

```tsx
// 나쁜 예
function Input({
  value,
  onChange,
  onFocus,
  onBlur,
  onKeyDown,
  onKeyUp,
  onKeyPress,
  // ... 모든 이벤트 핸들러 노출
}) {
  // ...
}

// 좋은 예
function Input({
  value,
  onChange,
  onFocus, // 정말 필요한 경우만
}: {
  value: string;
  onChange: (value: string) => void;
  onFocus?: () => void;
}) {
  // ...
}
```

### 2. 내부 상태 캡슐화

```tsx
// 나쁜 예: 내부 상태를 외부에 노출
function Accordion({
  isOpen,
  onToggle,
  // 사용자가 상태를 직접 관리해야 함
}) {
  // ...
}

// 좋은 예: 내부에서 상태 관리
function Accordion({
  defaultOpen = false,
  onToggle,
}: {
  defaultOpen?: boolean;
  onToggle?: (isOpen: boolean) => void;
}) {
  const [isOpen, setIsOpen] = useState(defaultOpen);
  // 내부에서 상태 관리
}
```

### 3. 구현 디테일 숨기기

```tsx
// 나쁜 예: 내부 구조를 노출
function Table({
  data,
  renderCell,
  cellRenderer,
  rowRenderer,
  headerRenderer,
  // ... 내부 렌더링 로직을 모두 노출
}) {
  // ...
}

// 좋은 예: 간단한 API
function Table({
  data,
  columns,
}: {
  data: any[];
  columns: Column[];
}) {
  // 내부 렌더링 로직은 캡슐화
}
```

## 고급 패턴: getProps 패턴

필요한 경우에만 세부 제어를 허용:

```tsx
function useToggle() {
  const [on, setOn] = useState(false);
  
  return {
    on,
    toggle: () => setOn(prev => !prev),
    getToggleProps: (props = {}) => ({
      'aria-pressed': on,
      onClick: () => setOn(prev => !prev),
      ...props,
    }),
  };
}

// 사용
function ToggleButton() {
  const { getToggleProps } = useToggle();
  
  return (
    <button {...getToggleProps({ className: 'custom-class' })}>
      Toggle
    </button>
  );
}
```

## 예시: Form 컴포넌트

### 나쁜 예: 모든 것을 노출

```tsx
function Form({
  fields,
  values,
  errors,
  touched,
  setFieldValue,
  setFieldTouched,
  validateField,
  // ... 폼 라이브러리의 모든 API 노출
}) {
  // ...
}
```

### 좋은 예: 최소한의 API

```tsx
function Form({
  onSubmit,
  children,
}: {
  onSubmit: (values: Record<string, any>) => void;
  children: React.ReactNode;
}) {
  const [values, setValues] = useState({});
  
  // 내부에서 모든 폼 로직 처리
  // 사용자는 간단한 API만 사용
}
```

## 장점

- **캡슐화**: 내부 구현 변경이 외부에 영향을 주지 않음
- **사용 편의성**: 간단하고 직관적인 API
- **변경 내성**: 내부 구현 변경이 breaking change가 아님
- **유지보수성**: 변경해야 할 곳이 명확함

## 단점

- **유연성 제한**: 특수한 케이스에 대응하기 어려울 수 있음
- **확장성**: 나중에 기능 추가가 필요할 수 있음

## 균형점 찾기

1. **80/20 원칙**: 80%의 사용 사례를 커버하는 최소한의 API
2. **점진적 노출**: 기본은 간단하게, 필요시 확장 가능
3. **명확한 의도**: 각 prop의 목적이 명확해야 함

## 예시: 점진적 노출

```tsx
// 기본 사용 (간단)
<Dropdown items={items} onSelect={handleSelect} />

// 고급 사용 (필요시)
<Dropdown
  items={items}
  onSelect={handleSelect}
  getItemProps={(item) => ({
    disabled: item.disabled,
  })}
/>
```

## 결론

Component API Surface를 최소화하면:
- 사용하기 쉬운 컴포넌트
- 변경에 강한 컴포넌트
- 유지보수하기 좋은 컴포넌트

를 만들 수 있습니다. 핵심은 "필요한 것만 노출하고, 나머지는 캡슐화"하는 것입니다.
