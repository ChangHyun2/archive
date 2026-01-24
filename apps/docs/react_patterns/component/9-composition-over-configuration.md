# Composition over Configuration

## 개요

Composition over Configuration 패턴은 옵션 props로 모든 걸 해결하려 하지 말고, 조합(Composition)으로 확장 가능하게 설계하는 패턴입니다. 이는 props 폭발을 방지하는 시니어 필살기입니다.

## 문제: Props 폭발 (Props Explosion)

```tsx
// 나쁜 예: 모든 것을 props로 처리
function Button({
  variant,
  size,
  icon,
  iconPosition,
  loading,
  disabled,
  fullWidth,
  rounded,
  shadow,
  // ... 수십 개의 props
}) {
  // ...
}
```

## 해결: Composition

```tsx
// 좋은 예: 조합으로 해결
function Button({ children, ...props }: ButtonProps) {
  return <button {...props}>{children}</button>;
}

function IconButton({ icon, children, ...props }: IconButtonProps) {
  return (
    <Button {...props}>
      <Icon>{icon}</Icon>
      {children}
    </Button>
  );
}

// 사용
<IconButton icon="plus" onClick={handleClick}>
  Add Item
</IconButton>
```

## 예시: Modal 컴포넌트

### Configuration 방식 (나쁜 예)

```tsx
function Modal({
  showHeader,
  headerTitle,
  showFooter,
  footerButtons,
  closeButton,
  closeButtonText,
  // ... 수십 개의 props
}) {
  return (
    <div>
      {showHeader && <div>{headerTitle}</div>}
      <div>{children}</div>
      {showFooter && <div>{footerButtons}</div>}
    </div>
  );
}
```

### Composition 방식 (좋은 예)

```tsx
function Modal({ children }: { children: React.ReactNode }) {
  return <div className="modal">{children}</div>;
}

Modal.Header = function ModalHeader({ children }: { children: React.ReactNode }) {
  return <div className="modal-header">{children}</div>;
};

Modal.Body = function ModalBody({ children }: { children: React.ReactNode }) {
  return <div className="modal-body">{children}</div>;
};

Modal.Footer = function ModalFooter({ children }: { children: React.ReactNode }) {
  return <div className="modal-footer">{children}</div>;
};

// 사용
<Modal>
  <Modal.Header>
    <h2>Title</h2>
    <CloseButton />
  </Modal.Header>
  <Modal.Body>
    <p>Content</p>
  </Modal.Body>
  <Modal.Footer>
    <Button>Cancel</Button>
    <Button>Save</Button>
  </Modal.Footer>
</Modal>
```

## Children을 활용한 조합

```tsx
function Card({ children }: { children: React.ReactNode }) {
  return <div className="card">{children}</div>;
}

// 사용자가 원하는 대로 조합
<Card>
  <img src="..." />
  <h3>Title</h3>
  <p>Description</p>
  <Button>Action</Button>
</Card>

// 또는 다른 조합
<Card>
  <h3>Title</h3>
  <p>Description</p>
</Card>
```

## Render Props와의 조합

```tsx
function DataTable({ data, renderRow }: { 
  data: any[]; 
  renderRow: (item: any) => React.ReactNode 
}) {
  return (
    <table>
      <tbody>
        {data.map((item, index) => (
          <tr key={index}>{renderRow(item)}</tr>
        ))}
      </tbody>
    </table>
  );
}

// 사용자가 row를 어떻게 렌더링할지 결정
<DataTable 
  data={users} 
  renderRow={(user) => (
    <>
      <td>{user.name}</td>
      <td>{user.email}</td>
      <td><Button>Edit</Button></td>
    </>
  )} 
/>
```

## 장점

- **확장성**: 새로운 기능 추가가 쉬움
- **유연성**: 사용자가 원하는 대로 조합 가능
- **Props 폭발 방지**: props 개수가 크게 줄어듦
- **재사용성**: 작은 컴포넌트를 조합하여 다양한 형태 생성

## 단점

- **초기 복잡도**: 처음에는 더 복잡해 보일 수 있음
- **학습 곡선**: 사용자가 조합 방법을 이해해야 함

## 원칙

1. **작은 컴포넌트를 조합**: 큰 컴포넌트보다 작은 컴포넌트를 조합
2. **Children 활용**: 가능한 한 children을 활용
3. **명확한 책임**: 각 컴포넌트는 하나의 명확한 책임
4. **확장 가능한 API**: 새로운 요구사항에 대응 가능한 구조

## 결론

Configuration보다 Composition을 선호하면 더 유연하고 확장 가능한 컴포넌트를 만들 수 있습니다. 이는 props 폭발을 방지하고, 사용자가 원하는 대로 컴포넌트를 조합할 수 있게 해줍니다.
