# State Colocation + Render Boundary

## 개요

상태를 가능한 한 사용하는 곳 가까이에 배치하고, 자주 바뀌는 영역과 자주 바뀌지 않는 영역을 컴포넌트 경계로 분리하여 불필요한 리렌더링을 방지하는 패턴입니다.

## 핵심 원칙

- **State Colocation**: 상태를 사용하는 컴포넌트에 최대한 가까이 배치
- **Render Boundary**: 자주 바뀌는 영역과 안 바뀌는 영역을 컴포넌트로 분리
- **트리 구조 최적화**: 위쪽 트리는 고정시키고, 아래쪽에만 변경이 발생하도록

## 사용 방법

### State Colocation

```tsx
// ❌ 나쁜 예: 상태를 상위에 배치
function App() {
  const [searchQuery, setSearchQuery] = useState('');
  const [filter, setFilter] = useState('all');
  
  // searchQuery가 바뀔 때마다 전체 App 리렌더링
  return (
    <div>
      <Header />
      <Sidebar />
      <SearchBar value={searchQuery} onChange={setSearchQuery} />
      <Content filter={filter} />
    </div>
  );
}

// ✅ 좋은 예: 상태를 사용하는 곳에 배치
function App() {
  return (
    <div>
      <Header />
      <Sidebar />
      <SearchSection /> {/* SearchSection 내부에서 상태 관리 */}
      <Content />
    </div>
  );
}

function SearchSection() {
  const [searchQuery, setSearchQuery] = useState('');
  // SearchSection만 리렌더링됨
  return <SearchBar value={searchQuery} onChange={setSearchQuery} />;
}
```

### Render Boundary

```tsx
// ❌ 나쁜 예: 변경이 자주 발생하는 영역이 전체를 리렌더링
function Dashboard() {
  const [notifications, setNotifications] = useState([]);
  
  return (
    <div>
      <Header /> {/* notifications 변경 시 리렌더링 */}
      <Sidebar /> {/* notifications 변경 시 리렌더링 */}
      <MainContent /> {/* notifications 변경 시 리렌더링 */}
      <NotificationPanel notifications={notifications} />
    </div>
  );
}

// ✅ 좋은 예: 변경 영역을 분리
function Dashboard() {
  return (
    <div>
      <Header />
      <Sidebar />
      <MainContent />
      <NotificationSection /> {/* 자체적으로 상태 관리 */}
    </div>
  );
}

function NotificationSection() {
  const [notifications, setNotifications] = useState([]);
  // NotificationSection만 리렌더링됨
  return <NotificationPanel notifications={notifications} />;
}
```

### 트리 구조 최적화

```tsx
// ✅ 자주 바뀌는 영역을 아래쪽에 배치
function App() {
  // 고정된 상위 컴포넌트
  return (
    <Layout>
      <Header /> {/* 고정 */}
      <Sidebar /> {/* 고정 */}
      <MainArea>
        <DynamicContent /> {/* 자주 바뀌는 영역 */}
      </MainArea>
    </Layout>
  );
}
```

## 주의사항

- 상태를 너무 많이 분산시키면 관리가 어려워질 수 있음
- 전역 상태가 필요한 경우도 있으므로 균형을 맞춰야 함
- 컴포넌트 구조를 신중하게 설계해야 함
