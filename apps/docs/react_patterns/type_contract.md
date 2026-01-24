타입 / 계약(Types & API) 패턴 (TS 기준)

1. Discriminated Unions로 UI 상태 모델링

loading | error | success 같은 상태를 타입으로 강제해서 누락/분기 버그 방지

2. Exhaustiveness Check (never)

switch/조건 분기에서 모든 케이스를 처리했는지 컴파일 타임에 강제

불가능한 props 조합을 타입으로 차단

예: href가 있으면 button props 일부 금지, onClick 필수/금지 같은 “XOR/Union” 설계

3. Polymorphic 타입(as prop) 안전하게 만들기

as="a"면 anchor props, as="button"이면 button props가 자동 추론되게

“좁은” Public Props + 내부 타입 분리

외부에 노출되는 Props는 최소화하고, 내부 구현 타입은 분리해 변경 내성↑

4. API Response → UI Model 변환 계층(Mapper) + 타입 고정

서버 DTO 타입을 그대로 UI에서 쓰지 말고, 변환된 “앱 도메인 타입”을 기준으로 UI 개발

5. Schema-first Validation (zod/yup) + 타입 추론

런타임 검증 스키마를 단일 진실로 두고 z.infer<>로 타입 자동 생성

(서버/클라 계약 불일치 방지에 진짜 강함)

6. Branded Types / Nominal Typing 흉내

UserId와 PostId 같은 “같은 string”을 섞어 쓰는 실수 방지

7. Typed Error Contracts

에러를 ErrorCode 유니온으로 표준화해서 UI가 에러 처리 UX를 안정적으로 구현

8. Generic API 클라이언트(Endpoint 타입 안전)

client.get<Resp>('/users') 수준이 아니라, endpoint별 요청/응답/에러를 타입으로 고정

9. Form 계약 타입화

폼 값 타입, 서버 요청 타입, 검증 스키마를 한 흐름으로 묶어 “한 군데만 바꾸면” 전체가 따라오게

10. Public API에 any/unknown 방치 금지

외부 입력(JSON, querystring, localStorage)은 unknown으로 받고 스키마로 좁혀서 안전하게 사용
