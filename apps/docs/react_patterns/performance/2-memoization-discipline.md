# Memoization Discipline (React.memo, useMemo, useCallback)

## 개요

React의 메모이제이션 API(React.memo, useMemo, useCallback)를 선택적이고 신중하게 사용하는 패턴입니다. 모든 곳에 적용하는 것이 아니라, 실제로 성능 이점이 있는 경우에만 사용합니다.

## 핵심 원칙

- **비용이 큰 렌더/계산에만 적용**: 메모이제이션 자체도 비용이 있으므로 신중하게 사용
- **참조 안정성이 의미 있는 경우에만**: 자식 컴포넌트에 props로 함수를 전달할 때 등
- **선택적 적용**: 모든 곳에 적용하지 말고 필요한 곳만

## 사용 시나리오

### React.memo

- 자주 리렌더링되지만 props가 자주 변하지 않는 컴포넌트
- 비용이 큰 렌더링을 하는 컴포넌트

### useMemo

- 비용이 큰 계산 결과를 캐싱
- 참조 안정성이 필요한 객체/배열 생성

### useCallback

- 자식 컴포넌트에 props로 전달하는 함수
- 의존성 배열이 있는 함수를 안정적으로 유지

## 주의사항

- 메모이제이션 자체도 비용이 있음
- 과도한 사용은 오히려 성능 저하를 일으킬 수 있음
- 의존성 배열을 정확히 관리해야 함

## 예시

```tsx
// ❌ 나쁜 예: 모든 곳에 memo 적용
const SimpleComponent = React.memo(({ text }) => {
  return <div>{text}</div>; // 단순한 렌더링에 불필요한 memo
});

// ✅ 좋은 예: 선택적 memo 적용
// 비용이 큰 계산
const ExpensiveComponent = ({ data }) => {
  const processedData = useMemo(() => {
    return data.map(item => expensiveTransformation(item));
  }, [data]);

  return <ComplexVisualization data={processedData} />;
};

// 자식에 함수 전달
const Parent = ({ items }) => {
  const handleClick = useCallback((id) => {
    // 핸들러 로직
  }, []);

  return (
    <div>
      {items.map(item => (
        <Child key={item.id} onClick={handleClick} />
      ))}
    </div>
  );
};

// 참조 안정성이 필요한 경우
const Component = ({ userId }) => {
  const config = useMemo(() => ({
    userId,
    options: { ... }
  }), [userId]);

  return <Child config={config} />;
};
```
