# Context 리렌더 폭발 방지

## 개요

Context를 사용할 때 발생하는 불필요한 리렌더링을 방지하는 패턴입니다. Context의 값이 변경되면 해당 Context를 구독하는 모든 컴포넌트가 리렌더링되는데, 이를 최소화하는 방법입니다.

## 핵심 원칙

- **Context 쪼개기**: 하나의 Provider에 모든 상태를 넣지 않고, 관련된 상태별로 분리
- **Selector/Subscription 기반 Context**: 필요한 값만 구독하도록 구현
- **값 분리**: 자주 바뀌는 값과 자주 바뀌지 않는 값을 분리

## 해결 방법

### 1. Context 쪼개기

```tsx
// ❌ 나쁜 예: 모든 상태를 하나의 Context에
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);
  
  // user가 바뀌면 theme, notifications를 사용하는 컴포넌트도 리렌더링됨
  return (
    <AppContext.Provider value={{ user, theme, notifications }}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ 좋은 예: Context 분리
const UserContext = createContext();
const ThemeContext = createContext();
const NotificationContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);
  
  return (
    <UserContext.Provider value={user}>
      <ThemeContext.Provider value={theme}>
        <NotificationContext.Provider value={notifications}>
          {children}
        </NotificationContext.Provider>
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}
```

### 2. Selector 패턴

```tsx
// ✅ Selector를 사용한 Context
function createContextWithSelector() {
  const Context = createContext();
  
  const Provider = ({ value, children }) => {
    const [state, setState] = useState(value);
    
    return (
      <Context.Provider value={{ state, setState }}>
        {children}
      </Context.Provider>
    );
  };
  
  const useContextSelector = (selector) => {
    const { state } = useContext(Context);
    return selector(state);
  };
  
  return { Provider, useContextSelector };
}

// 사용 예시
const { Provider, useContextSelector } = createContextWithSelector();

function UserName() {
  // user.name만 구독
  const name = useContextSelector(state => state.user?.name);
  return <div>{name}</div>;
}
```

### 3. 값 분리

```tsx
// ✅ 자주 바뀌는 값과 자주 바뀌지 않는 값 분리
const StaticContext = createContext();
const DynamicContext = createContext();

function AppProvider({ children }) {
  const staticValue = useMemo(() => ({
    config: { ... },
    constants: { ... }
  }), []);
  
  const [dynamicValue, setDynamicValue] = useState({ ... });
  
  return (
    <StaticContext.Provider value={staticValue}>
      <DynamicContext.Provider value={dynamicValue}>
        {children}
      </DynamicContext.Provider>
    </StaticContext.Provider>
  );
}
```

## 주의사항

- Context가 너무 많아지면 관리가 복잡해질 수 있음
- 적절한 균형을 찾아야 함
- 라이브러리(예: Zustand, Jotai)를 사용하는 것도 고려
