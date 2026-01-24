Hooks & 로직 재사용 패턴 (Senior FE)

1. Custom Hook 추상화 (useX)

컴포넌트에서 비즈니스/데이터 로직을 빼서 useSomething()으로 캡슐화

UI는 “값/핸들러 사용”만 하게 만들기

2. Hook Layering (계층화)

저수준 → 고수준으로 쌓기

예: useFetch() → useUserQuery() → useCurrentUser()

(의존성과 책임이 명확해지고 재사용이 쉬워짐)

3. Reusable Hook + “옵션/어댑터”로 확장

Hook이 모든 케이스를 다 알게 하지 말고, options(callback, mapper 등)로 변형 가능하게

예: useAsync(fn, { onSuccess, onError, mapResult })

4. Headless Logic Hook

UI 없이 로직만 제공하는 Hook (useTabs, useCombobox, useDisclosure)

디자인 시스템/다양한 UI에 재사용 가능

5. Domain Hook (도메인 전용 Hook)

기술적 Hook 말고 “업무 개념”으로 Hook을 만든다

예: useCheckout(), useCart(), usePermission()

(시니어들이 코드 읽히게 만드는 핵심 습관)

Ref as Instance (인스턴스 보관)

렌더와 무관하게 유지해야 하는 인스턴스는 useRef에 저장

예: AbortController, timer id, WebSocket, 외부 라이브러리 instance

6. Latest Value Pattern (stale closure 방지)

이벤트 핸들러/비동기 콜백에서 최신 값을 쓰기 위해 useRef로 최신 값을 유지

(재사용 Hook 만들 때 특히 자주 필요)

7. Abortable Async Pattern

비동기 작업을 취소 가능하게 만들어 레이스 컨디션/언마운트 후 setState 방지

fetch는 AbortController, 다른 async는 cancel flag 등

Effect는 “연결/해제(동기화)”로 제한

Hook 내부에서 useEffect는 구독/리스너/네트워크 연결 같은 동기화에만 사용

핵심 로직은 명령 함수(핸들러)로 분리해서 예측 가능하게

8. Stable Public API Pattern (Hook 반환값 설계)

Hook이 반환하는 형태를 일관되게: { data, status, error, actions }

호출하는 쪽에서 사용법이 통일되어 팀 생산성↑

9. Inversion of Control (콜백 주입)

Hook이 모든 결정을 하지 않고, 중요한 지점은 외부 콜백으로 위임

예: useForm({ validate, onSubmit }), useAsync(fn, { onRetry })

10. Testable Hook Design

순수 로직(파서/계산/상태전이)은 Hook 밖의 pure function으로 분리

Hook은 “React glue”만 담당 → 테스트/리팩터링 난이도↓
