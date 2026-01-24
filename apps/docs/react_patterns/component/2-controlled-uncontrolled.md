# Controlled / Uncontrolled + Control Props Pattern

## 개요

Controlled / Uncontrolled 패턴은 컴포넌트가 외부에서 상태를 제어할 수도 있고, 내부에서 자체 상태로 동작할 수도 있게 API를 설계하는 패턴입니다.

## 특징

- **Dual Mode**: Controlled와 Uncontrolled 모드를 모두 지원
- **유연성**: 사용 사례에 따라 적절한 모드 선택 가능
- **하위 호환성**: 기존 Uncontrolled 컴포넌트를 Controlled로 전환 가능

## Controlled Component

외부에서 `value`와 `onChange`를 통해 상태를 완전히 제어하는 컴포넌트

```tsx
const [value, setValue] = useState('');

<Input 
  value={value} 
  onChange={(e) => setValue(e.target.value)} 
/>
```

## Uncontrolled Component

컴포넌트 내부에서 자체적으로 상태를 관리하는 컴포넌트

```tsx
<Input defaultValue="initial" />
```

## Control Props Pattern

`value` prop의 존재 여부로 Controlled/Uncontrolled 모드를 자동 판단

```tsx
function Input({ value, onChange, defaultValue, ...props }) {
  const isControlled = value !== undefined;
  const [internalValue, setInternalValue] = useState(defaultValue || '');
  
  const currentValue = isControlled ? value : internalValue;
  const handleChange = (e) => {
    if (isControlled) {
      onChange?.(e);
    } else {
      setInternalValue(e.target.value);
    }
  };
  
  return <input value={currentValue} onChange={handleChange} {...props} />;
}
```

## 장점

- 사용자가 필요에 따라 제어 수준 선택 가능
- 점진적 마이그레이션 가능 (Uncontrolled → Controlled)
- 다양한 사용 사례에 대응 가능

## 주의사항

- `value`와 `defaultValue`를 동시에 사용하면 안 됨
- Controlled 모드에서는 `onChange` 필수
- 상태 동기화 문제 주의
