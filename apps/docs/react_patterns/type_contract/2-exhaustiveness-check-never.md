# Exhaustiveness Check (never)

## 개요

switch/조건 분기에서 모든 케이스를 처리했는지 컴파일 타임에 강제하는 패턴입니다. 불가능한 props 조합을 타입으로 차단합니다.

## 문제점

switch 문이나 조건 분기에서 모든 케이스를 처리하지 않으면 런타임 에러가 발생할 수 있습니다:

```typescript
// ❌ 나쁜 예
type Status = 'loading' | 'error' | 'success';

function handleStatus(status: Status) {
  switch (status) {
    case 'loading':
      return 'Loading...';
    case 'error':
      return 'Error occurred';
    // 'success' 케이스를 처리하지 않았지만 TypeScript가 경고하지 않음
  }
}
```

## 해결 방법

`never` 타입을 활용한 Exhaustiveness Check:

```typescript
// ✅ 좋은 예
type Status = 'loading' | 'error' | 'success';

function handleStatus(status: Status): string {
  switch (status) {
    case 'loading':
      return 'Loading...';
    case 'error':
      return 'Error occurred';
    case 'success':
      return 'Success!';
    default:
      // 모든 케이스를 처리했다면 이 분기는 never 타입이 됩니다
      const _exhaustive: never = status;
      throw new Error(`Unhandled status: ${_exhaustive}`);
  }
}

// Status에 새로운 케이스가 추가되면 컴파일 에러 발생
type Status = 'loading' | 'error' | 'success' | 'pending'; // 새 케이스 추가

// 이제 handleStatus 함수에서 컴파일 에러 발생
// 'pending' 케이스를 처리해야 함
```

## 불가능한 Props 조합 차단

XOR/Union 설계로 불가능한 props 조합을 타입으로 차단:

```typescript
// ✅ 좋은 예: href가 있으면 button props 일부 금지
type BaseButtonProps = {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary';
};

type LinkButtonProps = BaseButtonProps & {
  href: string;
  onClick?: never; // href가 있으면 onClick 금지
};

type ButtonProps = BaseButtonProps & {
  href?: never; // onClick이 있으면 href 금지
  onClick: () => void;
};

type ButtonComponentProps = LinkButtonProps | ButtonProps;

function Button(props: ButtonComponentProps) {
  if ('href' in props && props.href) {
    // href가 있으면 링크로 렌더링
    return <a href={props.href}>{props.children}</a>;
  }
  
  // onClick이 필수인 경우
  return <button onClick={props.onClick}>{props.children}</button>;
}

// 사용 예시
<Button href="/page">Link</Button> // ✅
<Button onClick={() => {}}>Button</Button> // ✅
<Button href="/page" onClick={() => {}}>Error</Button> // ❌ 컴파일 에러
```

## 더 복잡한 예시

```typescript
// 조건부 props 패턴
type ConditionalProps =
  | { variant: 'input'; value: string; onChange: (value: string) => void }
  | { variant: 'display'; value: string; onChange?: never }
  | { variant: 'readonly'; value: string; onChange?: never };

function Field(props: ConditionalProps) {
  switch (props.variant) {
    case 'input':
      return (
        <input
          value={props.value}
          onChange={(e) => props.onChange(e.target.value)}
        />
      );
    case 'display':
    case 'readonly':
      return <div>{props.value}</div>;
    default:
      const _exhaustive: never = props;
      throw new Error(`Unhandled variant: ${_exhaustive}`);
  }
}
```

## Helper 함수

```typescript
// Exhaustiveness Check를 위한 헬퍼 함수
function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`);
}

// 사용
function handleStatus(status: Status) {
  switch (status) {
    case 'loading':
      return 'Loading...';
    case 'error':
      return 'Error occurred';
    case 'success':
      return 'Success!';
    default:
      return assertNever(status); // 모든 케이스 처리 확인
  }
}
```

## 장점

1. **완전성 보장**: 모든 케이스를 처리하도록 강제
2. **리팩토링 안전성**: 새로운 케이스 추가 시 모든 사용처에서 에러 발생
3. **타입 안전성**: 불가능한 props 조합을 컴파일 타임에 차단
