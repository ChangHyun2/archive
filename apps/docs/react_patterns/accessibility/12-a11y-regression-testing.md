# A11y Regression Testing

## 개요

A11y Regression Testing은 스토리북과 axe(또는 유사 도구)로 접근성 회귀 테스트를 수행하는 패턴입니다. 키보드 내비게이션과 포커스 동작을 스토리 단위로 검증하여 접근성 문제가 재발하지 않도록 보장합니다.

## 특징

- **자동화된 접근성 테스트**: axe-core로 접근성 위반 자동 감지
- **스토리 기반 테스트**: 각 컴포넌트의 다양한 상태를 스토리로 테스트
- **키보드 내비게이션 검증**: 실제 키보드 이벤트로 동작 확인
- **CI/CD 통합**: 빌드 파이프라인에 접근성 테스트 포함

## 예시

```tsx
// Storybook 설정 - a11y 애드온
// .storybook/main.js
export default {
  addons: [
    '@storybook/addon-a11y',
    '@storybook/addon-interactions',
  ],
};

// Button 스토리 - 접근성 테스트
import { expect, userEvent, within } from '@storybook/test';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

export default {
  title: 'Components/Button',
  component: Button,
  parameters: {
    a11y: {
      config: {
        rules: [
          {
            id: 'color-contrast',
            enabled: true,
          },
        ],
      },
    },
  },
};

export const Primary = {
  args: {
    children: 'Button',
    variant: 'primary',
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button');
    
    // 키보드 접근성 테스트
    await userEvent.tab();
    expect(button).toHaveFocus();
    
    await userEvent.keyboard('{Enter}');
    // Enter 키 동작 확인
    
    // axe 접근성 테스트
    const { violations } = await axe(canvasElement);
    expect(violations).toHaveLength(0);
  },
};

// Dialog 스토리 - 포커스 관리 테스트
export const DialogStory = {
  render: () => {
    const [isOpen, setIsOpen] = useState(false);
    return (
      <>
        <button onClick={() => setIsOpen(true)}>Open Dialog</button>
        <Dialog isOpen={isOpen} onClose={() => setIsOpen(false)}>
          <h2>Dialog Title</h2>
          <p>Dialog content</p>
        </Dialog>
      </>
    );
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    
    // 다이얼로그 열기
    const trigger = canvas.getByRole('button', { name: /open dialog/i });
    await userEvent.click(trigger);
    
    // 포커스가 다이얼로그로 이동했는지 확인
    const dialog = canvas.getByRole('dialog');
    expect(dialog).toBeInTheDocument();
    
    // 포커스 트랩 테스트
    const firstFocusable = within(dialog).getByRole('button');
    expect(firstFocusable).toHaveFocus();
    
    // Tab 키로 포커스 순환 확인
    await userEvent.tab();
    // 마지막 요소에서 첫 요소로 순환하는지 확인
    
    // ESC 키로 닫기
    await userEvent.keyboard('{Escape}');
    expect(dialog).not.toBeInTheDocument();
    
    // 포커스 복원 확인
    expect(trigger).toHaveFocus();
    
    // axe 테스트
    const { violations } = await axe(canvasElement);
    expect(violations).toHaveLength(0);
  },
};

// Form 스토리 - 폼 접근성 테스트
export const FormStory = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    
    // 라벨 연결 확인
    const emailInput = canvas.getByLabelText(/email/i);
    expect(emailInput).toHaveAttribute('aria-describedby');
    
    // 에러 메시지 접근성
    await userEvent.type(emailInput, 'invalid');
    await userEvent.tab();
    
    const errorMessage = canvas.getByRole('alert');
    expect(errorMessage).toBeInTheDocument();
    
    // ARIA Live Region 테스트
    // 스크린 리더가 에러 메시지를 읽는지 확인
    
    // axe 테스트
    const { violations } = await axe(canvasElement);
    expect(violations).toHaveLength(0);
  },
};

// Jest 테스트 - 접근성 회귀 테스트
import { render } from '@testing-library/react';
import { axe } from 'jest-axe';

describe('Button Accessibility', () => {
  it('should have no accessibility violations', async () => {
    const { container } = render(<Button>Click me</Button>);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
  
  it('should be keyboard accessible', () => {
    const { getByRole } = render(<Button>Click me</Button>);
    const button = getByRole('button');
    
    button.focus();
    expect(button).toHaveFocus();
    
    fireEvent.keyDown(button, { key: 'Enter' });
    // Enter 키 동작 확인
  });
});

// Playwright 테스트 - E2E 접근성 테스트
import { test, expect } from '@playwright/test';
import { injectAxe, checkA11y } from 'axe-playwright';

test('should have no accessibility violations', async ({ page }) => {
  await page.goto('/');
  await injectAxe(page);
  await checkA11y(page);
});

test('keyboard navigation works', async ({ page }) => {
  await page.goto('/');
  
  // Tab 키로 포커스 이동
  await page.keyboard.press('Tab');
  const focusedElement = await page.evaluate(() => document.activeElement?.tagName);
  expect(focusedElement).toBe('BUTTON');
  
  // Enter 키로 활성화
  await page.keyboard.press('Enter');
  // 동작 확인
});

// CI/CD 통합 예시
// .github/workflows/a11y-test.yml
name: Accessibility Tests

on: [push, pull_request]

jobs:
  a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm install
      - run: npm run test:a11y
      - run: npm run storybook:test
```

## 구현 방법

1. **axe-core 통합**: jest-axe 또는 @axe-core/react로 접근성 테스트 도구 통합
2. **스토리북 설정**: @storybook/addon-a11y 애드온 설치 및 설정
3. **스토리별 테스트**: 각 스토리에 play 함수로 키보드 내비게이션 테스트
4. **자동화된 검증**: CI/CD 파이프라인에 접근성 테스트 추가
5. **키보드 테스트**: userEvent로 실제 키보드 이벤트 시뮬레이션
6. **포커스 테스트**: 포커스 이동, 복원, 트랩 등 포커스 관련 동작 검증

## 장점

- 접근성 문제 조기 발견
- 회귀 방지
- 자동화로 인한 시간 절약
- 일관된 접근성 품질 유지
- CI/CD 통합으로 배포 전 검증

## 단점

- 초기 설정 및 학습 곡선
- 테스트 작성 시간 필요
- 일부 접근성 문제는 자동 감지 어려움
- false positive 가능성
