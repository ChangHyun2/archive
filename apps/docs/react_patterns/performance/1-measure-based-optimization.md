# 측정 기반 최적화 (Measure → Optimize)

## 개요

성능 최적화는 감이 아닌 데이터 기반으로 접근해야 합니다. React DevTools의 Profiler나 Performance 탭을 사용하여 실제 병목 지점을 먼저 측정하고, 그 부분만 집중적으로 최적화하는 패턴입니다.

## 핵심 원칙

- **측정 우선**: 최적화 전에 반드시 성능을 측정하고 병목 지점을 파악
- **선택적 최적화**: 실제로 문제가 되는 부분만 최적화
- **과도한 최적화 방지**: 감으로 memo를 남발하면 오히려 성능이 저하될 수 있음

## 사용 방법

### React DevTools Profiler 사용

1. React DevTools의 Profiler 탭 열기
2. "Record" 버튼 클릭
3. 성능을 측정할 액션 수행
4. "Stop" 버튼 클릭
5. 렌더링 시간이 긴 컴포넌트 확인

### Chrome Performance 탭 사용

1. Chrome DevTools의 Performance 탭 열기
2. "Record" 버튼 클릭
3. 액션 수행
4. "Stop" 버튼 클릭
5. Flame Chart에서 병목 지점 확인

## 주의사항

- 감으로 최적화하지 말 것
- 모든 컴포넌트에 memo를 적용하지 말 것
- 실제 병목 지점만 최적화할 것
- 최적화 전후 성능을 비교 측정할 것

## 예시

```tsx
// ❌ 나쁜 예: 감으로 최적화
const MyComponent = React.memo(({ data }) => {
  // 모든 컴포넌트에 memo 적용
  return <div>{data}</div>;
});

// ✅ 좋은 예: 측정 후 최적화
// 1. Profiler로 측정
// 2. 실제로 렌더링이 자주 발생하는 컴포넌트 확인
// 3. 그 컴포넌트만 선택적으로 최적화
const HeavyComponent = React.memo(({ expensiveData }) => {
  // 실제로 비용이 큰 렌더링만 memo 적용
  return <ComplexVisualization data={expensiveData} />;
});
```
