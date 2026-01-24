React 컴포넌트 설계 패턴 (Senior FE에서 많이 쓰는 것만)

1. Compound Components

여러 하위 컴포넌트를 조합해서 하나의 기능을 구성 (<Tabs.List/>, <Tabs.Panel/>)

2. Controlled / Uncontrolled + Control Props

외부에서 상태를 제어할 수도, 내부에서 자체 상태로 동작할 수도 있게 API 설계

3. State Reducer Pattern

내부 상태 업데이트 로직을 “외부에서 개입/오버라이드” 가능하게 만들어 커스터마이즈 허용

4. Polymorphic Component (as prop)

같은 컴포넌트를 button/a/div 등으로 렌더링 타입만 바꿔 재사용성↑

Slot Pattern (Named Slots / Children-as-Slots)

header, footer, actions 같은 명명된 영역에 children을 배치해 조립성↑

5. Render Props

컴포넌트가 “무엇을 렌더링할지”를 함수 children으로 받아 유연한 UI 구성

6. Headless Component Pattern

UI(스타일)는 없고 상태/행동/접근성 로직만 제공해서 디자인 시스템과 궁합 좋음

7. Container / Presentational (Smart/Dumb)

데이터/비즈니스 로직과 UI를 분리해서 재사용성과 테스트 용이성↑

(요즘은 “느슨하게 적용”하는 편이지만, 설계 원칙으로는 여전히 유효)

8. Composition over Configuration

옵션 props로 모든 걸 해결하려 하지 말고, 조합(Composition) 으로 확장 가능하게 설계

(props 폭발 방지용 시니어 필살기)

9. Component API Surface Minimization

public props를 최소화하고, 불필요한 구현 디테일 노출을 막는 설계

(캡슐화/변경 내성/사용 난이도 측면에서 핵심)
