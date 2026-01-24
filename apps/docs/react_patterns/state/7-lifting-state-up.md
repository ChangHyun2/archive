# Lifting State Up

## 개요

두 컴포넌트가 같은 상태를 공유해야 하면 가장 가까운 공통 부모로 끌어올리는 패턴입니다. 전역 상태로 점프하기 전에 먼저 고려해야 합니다.

## 특징

- **공통 부모 활용**: 상태를 공유하는 컴포넌트들의 가장 가까운 공통 부모에 상태 배치
- **Props로 전달**: 상태와 setter를 props로 하위 컴포넌트에 전달
- **전역 상태 대안**: 전역 상태 관리 전에 먼저 시도해볼 패턴

## 예시

```tsx
// ❌ 나쁜 예: 각각 독립적인 상태
function TemperatureInput({ scale }) {
  const [temperature, setTemperature] = useState('');
  
  return (
    <input
      value={temperature}
      onChange={(e) => setTemperature(e.target.value)}
      placeholder={`Enter ${scale} temperature`}
    />
  );
}

function App() {
  return (
    <div>
      <TemperatureInput scale="Celsius" />
      <TemperatureInput scale="Fahrenheit" />
      {/* 두 입력값이 동기화되지 않음 */}
    </div>
  );
}

// ✅ 좋은 예: 상태를 공통 부모로 끌어올림
function TemperatureInput({ temperature, onTemperatureChange, scale }) {
  return (
    <input
      value={temperature}
      onChange={(e) => onTemperatureChange(e.target.value)}
      placeholder={`Enter ${scale} temperature`}
    />
  );
}

function App() {
  const [temperature, setTemperature] = useState('');
  const [scale, setScale] = useState('c');
  
  const celsius = scale === 'c' ? temperature : convertToCelsius(temperature);
  const fahrenheit = scale === 'f' ? temperature : convertToFahrenheit(temperature);
  
  return (
    <div>
      <TemperatureInput
        scale="celsius"
        temperature={celsius}
        onTemperatureChange={(value) => {
          setTemperature(value);
          setScale('c');
        }}
      />
      <TemperatureInput
        scale="fahrenheit"
        temperature={fahrenheit}
        onTemperatureChange={(value) => {
          setTemperature(value);
          setScale('f');
        }}
      />
    </div>
  );
}
```

## 구현 방법

1. **공통 부모 찾기**: 상태를 공유하는 컴포넌트들의 가장 가까운 공통 부모 식별
2. **상태 이동**: 상태를 공통 부모로 이동
3. **Props 전달**: 상태와 setter를 props로 하위 컴포넌트에 전달
4. **단방향 데이터 흐름**: 데이터는 위에서 아래로, 이벤트는 아래에서 위로

## 장점

- 전역 상태 없이도 상태 공유 가능
- 데이터 흐름이 명확하고 예측 가능
- 컴포넌트 재사용성 유지
- 디버깅 용이

## 단점

- Props drilling 문제 발생 가능
- 깊은 트리 구조에서는 비효율적
- 공통 부모가 멀리 있으면 복잡해질 수 있음
