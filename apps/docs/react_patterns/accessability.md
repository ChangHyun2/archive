접근성 / 디자인 시스템 패턴 (Senior FE)

1. A11y-first Primitives (접근성 기본 프리미티브)

Button, Input, Dialog, Popover 같은 기본 컴포넌트를 처음부터 a11y 규격으로 만든 뒤 전사 재사용

(즉흥적으로 화면마다 aria 붙이는 방식은 결국 망함…)

2. WAI-ARIA Pattern 준수

Dialog, Menu, Combobox, Tabs 등은 “정석 role/aria 상태”를 따라 구현

올바른 role + aria-expanded/selected/controls 등 상태 연동이 핵심

3. Keyboard-first Interaction

마우스 없이 Tab/Shift+Tab/Enter/Esc/Arrow로 전 기능 사용 가능하게

특히 Menu, Select, Autocomplete에서 필수

4. Roving Tabindex

리스트/탭/메뉴 같은 “여러 아이템 중 하나만 포커스”되는 UI에서

Tab은 컨테이너 진입/이탈만, 내부 이동은 Arrow로 처리

5. Focus Management (포커스 제어)

모달 열림 → 포커스 이동, 닫힘 → 원래 트리거로 복원

Focus Trap(모달 내부 포커스 순환) + 스크롤 잠금까지 세트로

6. ARIA Live / Announcements

토스트/폼 에러/검색 결과 갱신 같은 “동적 변화”를 스크린리더에 알리기

(특히 폼 검증 실패 메시지에서 체감 차이 큼)

7. Semantic-first Markup

div 지옥 대신 button/label/fieldset/nav/main 등 시맨틱 태그 우선

접근성 + 테스트 안정성 + 유지보수성까지 같이 좋아짐

8. Headless + Styled 분리 (Design System 핵심)

접근성/상호작용 로직(Headless)과 스타일(UI)을 분리

디자인 토큰/테마 바꿔도 로직은 그대로 재사용 가능

9. Design Tokens (색/간격/타이포의 단일 진실)

색/폰트/spacing/radius/shadow를 토큰으로 관리하고 컴포넌트는 토큰만 소비

브랜드 스킨/다크모드/멀티 테마에 강함

10. Theme Provider + Multi-brand Theming

Theme를 Context로 주입하고, 제품/브랜드별 테마를 쉽게 교체 가능하게

“한 번 만든 컴포넌트로 여러 제품”이 목표일 때 필수

11. Variants 기반 API (일관된 스타일 확장)

size, variant, tone 같은 제한된 변형만 허용

자유로운 className 덕지덕지는 디자인 시스템 붕괴의 지름길

12. A11y Regression Testing

스토리북 + axe(또는 유사 도구)로 접근성 회귀 테스트

키보드 내비게이션/포커스 동작을 스토리 단위로 검증

13. Reduced Motion 지원

prefers-reduced-motion 존중: 애니메이션 최소화/대체

현업에서는 “배려”를 넘어 “품질 기준”에 가까움

14. Color Contrast & Non-color Cues

대비(contrast) 기준 지키기 + 색만으로 상태 전달하지 않기(아이콘/텍스트/패턴 추가)