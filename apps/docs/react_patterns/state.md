상태관리 / 데이터 흐름 패턴 (Senior FE)

1. Server State vs Client State 분리

서버에서 온 데이터(캐시/동기화/재시도)는 React Query/SWR 같은 걸로

2. UI/로컬 상태(토글, 필터, 모달 등)는 useState/Zustand/Redux 등으로 분리

Single Source of Truth

같은 의미의 상태를 여러 곳에 중복 저장하지 않기

(폼 값 + derived 값 + URL 파라미터가 서로 불일치하는 사고 방지)

3. Derived State 최소화

“계산으로 얻을 수 있는 값”은 state로 들고 있지 말고 selector / useMemo로 파생

(동기화 버그의 80%가 여기서 나옴…)

4. Normalize State (정규화)

리스트/엔티티는 { byId, allIds } 형태로 관리해서 업데이트/참조를 안정적으로

(특히 댓글/상품/유저처럼 관계가 얽히면 필수)

5. Selector Pattern

store에서 필요한 조각만 읽는 선택자(selector)를 두고, 컴포넌트는 selector만 구독

리렌더 최소화 + 결합도 감소

6. State Colocation

상태는 “쓰는 곳”에 최대한 가깝게 둔다 (전역부터 만들지 말기)

공유가 필요해지는 순간에만 끌어올리기(lift up) or store로 승격

7. Lifting State Up

두 컴포넌트가 같은 상태를 공유해야 하면 가장 가까운 공통 부모로 끌어올림

(전역 상태로 점프하기 전에 먼저 고려)

8. Command/Intent 기반 업데이트

setState(value) 남발 대신 addItem/removeItem/applyFilter처럼 “의도”가 드러나는 API로 업데이트

(규칙/검증/로깅/테스트가 쉬워짐)

9. Optimistic Update + Rollback

UI를 먼저 바꾸고 서버 실패 시 되돌리기(롤백)

(서버 상태 라이브러리의 mutation 패턴과 찰떡)

10. Stale-While-Revalidate (SWR)

캐시된 데이터를 즉시 보여주고, 뒤에서 재검증/갱신

로딩 UX 개선 + 네트워크 비용/지연을 제품 관점에서 최적화

11. Event-driven State (제한적 Pub/Sub)

토스트, 전역 모달, 트래킹 같은 “횡단 관심사”에만 이벤트로 연결

핵심 도메인 상태는 이벤트 지옥 안 만들게 조심

12. URL State (Router State)

검색/필터/페이지네이션 같은 건 URL에 싣는 패턴

공유/북마크/뒤로가기/새로고침에 강해짐
