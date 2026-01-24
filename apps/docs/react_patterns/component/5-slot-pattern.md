# Slot Pattern (Named Slots / Children-as-Slots)

## 개요

Slot Pattern은 `header`, `footer`, `actions` 같은 명명된 영역에 children을 배치해 조립성을 높이는 패턴입니다.

## 특징

- **명확한 구조**: 각 영역의 역할이 명확하게 정의됨
- **조립성**: 필요한 영역만 선택적으로 구성 가능
- **유연성**: 다양한 레이아웃 조합 가능

## 기본 구현

```tsx
type CardProps = {
  header?: React.ReactNode;
  footer?: React.ReactNode;
  actions?: React.ReactNode;
  children: React.ReactNode;
};

function Card({ header, footer, actions, children }: CardProps) {
  return (
    <div className="card">
      {header && <div className="card-header">{header}</div>}
      <div className="card-body">{children}</div>}
      {actions && <div className="card-actions">{actions}</div>}
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}
```

## 사용 예시

```tsx
<Card
  header={<h2>Title</h2>}
  actions={
    <>
      <Button>Cancel</Button>
      <Button>Save</Button>
    </>
  }
  footer={<small>Last updated: today</small>}
>
  <p>Card content goes here</p>
</Card>
```

## Children-as-Slots 패턴

children을 객체로 받아서 명명된 슬롯으로 사용

```tsx
type CardSlots = {
  header?: React.ReactNode;
  body: React.ReactNode;
  footer?: React.ReactNode;
};

function Card({ children }: { children: CardSlots | React.ReactNode }) {
  if (typeof children === 'object' && 'body' in children) {
    const slots = children as CardSlots;
    return (
      <div className="card">
        {slots.header && <div className="card-header">{slots.header}</div>}
        <div className="card-body">{slots.body}</div>}
        {slots.footer && <div className="card-footer">{slots.footer}</div>}
      </div>
    );
  }
  
  return <div className="card">{children}</div>;
}
```

## Compound Components와의 조합

```tsx
function Card({ children }: { children: React.ReactNode }) {
  return <div className="card">{children}</div>;
}

Card.Header = function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>;
};

Card.Body = function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>;
};

Card.Footer = function CardFooter({ children }: { children: React.ReactNode }) {
  return <div className="card-footer">{children}</div>;
};

// 사용
<Card>
  <Card.Header>Title</Card.Header>
  <Card.Body>Content</Card.Body>
  <Card.Footer>Footer</Card.Footer>
</Card>
```

## 장점

- 레이아웃 구조가 명확하고 예측 가능
- 필요한 영역만 선택적으로 사용 가능
- 스타일링과 구조가 분리되어 유지보수 용이

## 단점

- props가 많아질 수 있음
- children-as-slots 패턴은 타입 정의가 복잡할 수 있음

## 활용 사례

- 카드 컴포넌트 (header, body, footer)
- 모달 컴포넌트 (header, content, footer, actions)
- 레이아웃 컴포넌트 (sidebar, main, aside)
