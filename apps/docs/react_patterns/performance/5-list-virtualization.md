# List Virtualization

## 개요

긴 리스트를 렌더링할 때 화면에 보이는 항목만 렌더링하여 성능을 최적화하는 패턴입니다. react-window나 react-virtualized 같은 라이브러리를 사용하여 구현합니다.

## 핵심 원칙

- **보이는 것만 렌더**: 화면에 보이는 항목만 실제로 렌더링
- **스크롤 성능 개선**: 수천 개의 항목이 있어도 부드러운 스크롤
- **메모리 사용량 감소**: DOM 노드 수를 최소화

## 사용 방법

### react-window 사용

```tsx
import { FixedSizeList } from 'react-window';

// ❌ 나쁜 예: 모든 항목 렌더링
function LongList({ items }) {
  return (
    <div>
      {items.map(item => (
        <ListItem key={item.id} item={item} />
      ))}
    </div>
  );
}

// ✅ 좋은 예: 가상화 사용
function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      <ListItem item={items[index]} />
    </div>
  );

  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

### react-window 다양한 옵션

```tsx
import { VariableSizeList } from 'react-window';
import { FixedSizeGrid } from 'react-window';

// 가변 높이 리스트
function VariableHeightList({ items }) {
  const getItemSize = (index) => {
    return items[index].height || 50;
  };

  const Row = ({ index, style }) => (
    <div style={style}>
      <ListItem item={items[index]} />
    </div>
  );

  return (
    <VariableSizeList
      height={600}
      itemCount={items.length}
      itemSize={getItemSize}
      width="100%"
    >
      {Row}
    </VariableSizeList>
  );
}

// 그리드 레이아웃
function VirtualizedGrid({ items }) {
  const Cell = ({ columnIndex, rowIndex, style }) => (
    <div style={style}>
      {items[rowIndex][columnIndex]}
    </div>
  );

  return (
    <FixedSizeGrid
      columnCount={5}
      columnWidth={200}
      height={600}
      rowCount={items.length}
      rowHeight={100}
      width={1000}
    >
      {Cell}
    </FixedSizeGrid>
  );
}
```

## 언제 사용해야 할까?

- **100개 이상의 항목**을 렌더링할 때
- **스크롤 성능**이 저하될 때
- **메모리 사용량**이 문제가 될 때
- **동적 리스트**에서 항목이 자주 추가/제거될 때

## 주의사항

- 각 항목의 높이가 고정되어 있으면 `FixedSizeList` 사용
- 높이가 가변적이면 `VariableSizeList` 사용
- 스크롤 위치 복원이 필요하면 `scrollOffset` prop 사용
- 항목이 너무 적으면(50개 미만) 오버헤드가 더 클 수 있음
