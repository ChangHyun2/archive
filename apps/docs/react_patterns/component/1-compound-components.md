# Compound Components Pattern

## 개요

Compound Components 패턴은 여러 하위 컴포넌트를 조합해서 하나의 기능을 구성하는 패턴입니다. 대표적인 예시로 `<Tabs.List/>`, `<Tabs.Panel/>` 같은 구조가 있습니다.

## 특징

- **조합 가능성**: 여러 하위 컴포넌트를 자유롭게 조합하여 사용 가능
- **명확한 API**: 각 하위 컴포넌트의 역할이 명확하게 분리됨
- **유연성**: 필요한 컴포넌트만 선택적으로 사용 가능

## 예시

```tsx
// 사용 예시
<Tabs>
  <Tabs.List>
    <Tabs.Tab>Tab 1</Tabs.Tab>
    <Tabs.Tab>Tab 2</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel>Content 1</Tabs.Panel>
  <Tabs.Panel>Content 2</Tabs.Panel>
</Tabs>
```

## 구현 방법

1. Context API를 사용하여 하위 컴포넌트 간 상태 공유
2. 각 하위 컴포넌트를 메인 컴포넌트의 속성으로 노출
3. 내부적으로 상태를 관리하면서 외부에는 명확한 API 제공

## 장점

- 사용자가 필요한 부분만 선택적으로 사용 가능
- 컴포넌트 구조가 직관적이고 이해하기 쉬움
- 재사용성이 높음

## 단점

- 초기 구현이 복잡할 수 있음
- Context API 사용으로 인한 성능 고려 필요
