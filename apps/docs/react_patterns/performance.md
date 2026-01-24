성능(리렌더/렌더링) 패턴 (Senior FE)

1. 측정 기반 최적화 (Measure → Optimize)

React DevTools Profiler / Performance 탭으로 “병목” 먼저 찍고, 거기만 손대기

(감으로 memo 남발하면 오히려 느려지는 경우 많음)

2. Memoization Discipline (React.memo, useMemo, useCallback)

비용 큰 렌더/계산, 자식에 props로 함수 내려줄 때만 선택적으로

“참조 안정성”이 실제로 의미 있는 구간에만 적용

3. Context 리렌더 폭발 방지

Context 쪼개기: 한 Provider에 모든 상태 넣지 않기

Selector/Subscription 기반 Context: 필요한 값만 구독(가능하면)

4. State Colocation + Render Boundary

자주 바뀌는 state는 가장 아래(가까운 곳)에 두고, 위쪽 트리는 고정시키기

“자주 바뀌는 영역”과 “안 바뀌는 영역”을 컴포넌트 경계로 분리

5. List Virtualization

긴 리스트는 windowing(react-window류)로 “보이는 것만 렌더”

스크롤 성능/메모리 사용량에 직격

6. Stable Keys 전략

리스트 key는 index 지양, 데이터의 stable id 사용

key가 흔들리면 불필요한 마운트/언마운트 + 상태 꼬임 + 성능 저하

7. Code Splitting (Route/Feature 단위)

초기 번들 줄이기: 라우트/기능 단위로 lazy load

“처음 화면”은 가볍게, 나머지는 필요할 때

8. Progressive Rendering (Skeleton/Placeholder)

모든 데이터 기다렸다가 한 번에 렌더하지 말고, 단계적으로 보여주기

체감 성능(UX)을 확 끌어올림

9. Suspense 기반 로딩 경계

로딩을 컴포넌트 트리의 “경계”로 다뤄서 fallback을 체계적으로

스트리밍/중첩 로딩 같은 고급 UX에 유리

10. startTransition / useDeferredValue로 입력 UX 개선

검색/필터처럼 연산이 큰 UI에서 “입력 반응”을 우선 처리하고 렌더는 뒤로

타이핑 끊김 방지

11. Heavy Work Offloading

큰 계산/파싱/정렬은 Web Worker로 빼거나, 서버로 보내거나, 캐시하기

메인 스레드 프리즈를 원천 차단

12. Throttle/Debounce + Idle Work

스크롤/리사이즈/입력 이벤트는 throttle/debounce

가능하면 idle 시간에 처리(requestIdleCallback 등)
