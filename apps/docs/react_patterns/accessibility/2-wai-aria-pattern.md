# WAI-ARIA Pattern 준수

## 개요

WAI-ARIA Pattern 준수는 Dialog, Menu, Combobox, Tabs 등 복잡한 UI 컴포넌트를 구현할 때 W3C에서 정의한 "정석 role/aria 상태"를 따라 구현하는 패턴입니다. 올바른 role과 aria-expanded, aria-selected, aria-controls 등의 상태 연동이 핵심입니다.

## 특징

- **표준 준수**: W3C WAI-ARIA Authoring Practices 가이드라인 준수
- **상태 연동**: aria 속성과 실제 컴포넌트 상태를 동기화
- **스크린 리더 호환성**: 보조 기술이 컴포넌트 상태를 정확히 인식
- **예측 가능성**: 표준 패턴을 따르므로 사용자 경험이 일관됨

## 예시

```tsx
// Dialog 패턴
export const Dialog = ({ isOpen, onClose, title, children }) => {
  const titleId = useId();
  const descriptionId = useId();

  useEffect(() => {
    if (isOpen) {
      // 포커스 트랩 및 초기 포커스 설정
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby={titleId}
      aria-describedby={descriptionId}
    >
      <h2 id={titleId}>{title}</h2>
      <div id={descriptionId}>{children}</div>
      <button onClick={onClose}>Close</button>
    </div>
  );
};

// Menu 패턴
export const Menu = ({ isOpen, onClose, items }) => {
  const [selectedIndex, setSelectedIndex] = useState(-1);

  return (
    <div
      role="menu"
      aria-orientation="vertical"
    >
      {items.map((item, index) => (
        <button
          key={item.id}
          role="menuitem"
          aria-selected={selectedIndex === index}
          onClick={() => {
            item.onClick();
            onClose();
          }}
        >
          {item.label}
        </button>
      ))}
    </div>
  );
};

// Tabs 패턴
export const Tabs = ({ tabs }) => {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <div role="tablist" aria-label="Main navigation">
      {tabs.map((tab, index) => (
        <button
          key={tab.id}
          role="tab"
          aria-selected={activeTab === index}
          aria-controls={`panel-${tab.id}`}
          id={`tab-${tab.id}`}
          onClick={() => setActiveTab(index)}
        >
          {tab.label}
        </button>
      ))}
      {tabs.map((tab, index) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={activeTab !== index}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
};
```

## 구현 방법

1. **WAI-ARIA Authoring Practices 참조**: 각 컴포넌트 타입에 맞는 표준 패턴 확인
2. **필수 role 적용**: 컴포넌트의 목적에 맞는 role 속성 설정
3. **상태 속성 동기화**: aria-expanded, aria-selected, aria-checked 등을 실제 상태와 연동
4. **관계 속성 설정**: aria-controls, aria-labelledby, aria-describedby로 요소 간 관계 명시
5. **키보드 내비게이션 구현**: 표준 키보드 패턴 준수 (Arrow keys, Enter, Esc 등)

## 장점

- 보조 기술과의 완벽한 호환성
- 접근성 표준 준수로 법적 요구사항 충족
- 예측 가능한 사용자 경험
- 유지보수 시 표준 문서 참조 가능

## 단점

- 초기 학습 곡선 존재
- 모든 상태를 정확히 동기화해야 함
- 복잡한 컴포넌트는 구현이 까다로울 수 있음
