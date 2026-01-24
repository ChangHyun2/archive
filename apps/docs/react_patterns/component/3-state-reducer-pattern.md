# State Reducer Pattern

## 개요

State Reducer 패턴은 컴포넌트의 내부 상태 업데이트 로직을 외부에서 개입하거나 오버라이드할 수 있게 만들어 커스터마이징을 허용하는 패턴입니다.

## 특징

- **확장성**: 기본 동작을 유지하면서 특정 케이스만 커스터마이즈 가능
- **제어권**: 사용자가 상태 업데이트 로직을 완전히 제어 가능
- **유연성**: 다양한 엣지 케이스에 대응 가능

## 기본 구조

```tsx
function useToggle({ reducer = (state, action) => {
  switch (action.type) {
    case 'toggle':
      return { on: !state.on };
    case 'reset':
      return { on: false };
    default:
      return state;
  }
}}) {
  const [state, dispatch] = useReducer(reducer, { on: false });
  
  const toggle = () => dispatch({ type: 'toggle' });
  const reset = () => dispatch({ type: 'reset' });
  
  return { on: state.on, toggle, reset };
}
```

## 사용 예시

```tsx
// 기본 사용
const { on, toggle } = useToggle();

// 커스터마이즈된 reducer
function customReducer(state, action) {
  if (action.type === 'toggle' && state.on) {
    // 이미 켜져있으면 토글하지 않음
    return state;
  }
  // 기본 동작
  switch (action.type) {
    case 'toggle':
      return { on: !state.on };
    default:
      return state;
  }
}

const { on, toggle } = useToggle({ reducer: customReducer });
```

## 장점

- 기본 동작을 유지하면서 특정 케이스만 커스터마이즈 가능
- 테스트하기 쉬움 (reducer를 모킹 가능)
- 복잡한 상태 로직을 외부에서 제어 가능

## 단점

- 초기 학습 곡선이 있음
- reducer 함수 작성이 필요하여 사용이 복잡할 수 있음
- 내부 상태 구조 변경 시 breaking change 가능

## 활용 사례

- 토글 컴포넌트에서 특정 조건에서만 토글 허용
- 폼 컴포넌트에서 커스텀 유효성 검사
- 리스트 컴포넌트에서 커스텀 필터링/정렬 로직
