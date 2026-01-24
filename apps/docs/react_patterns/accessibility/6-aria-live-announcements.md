# ARIA Live / Announcements

## 개요

ARIA Live / Announcements는 토스트, 폼 에러, 검색 결과 갱신 같은 "동적 변화"를 스크린 리더에 알리는 패턴입니다. 특히 폼 검증 실패 메시지에서 사용자 체감 차이가 큽니다. aria-live 속성을 사용하여 중요한 변경사항을 보조 기술에 실시간으로 전달합니다.

## 특징

- **실시간 알림**: 동적 콘텐츠 변경을 스크린 리더에 즉시 알림
- **우선순위 설정**: polite(비동기) 또는 assertive(즉시) 알림 레벨 선택
- **컨텍스트 제공**: 사용자가 현재 상황을 이해할 수 있도록 명확한 메시지
- **중복 방지**: 같은 메시지 반복 알림 방지

## 예시

```tsx
// ARIA Live Region 컴포넌트
export const LiveRegion = ({ 
  message, 
  priority = 'polite',
  id 
}: {
  message: string;
  priority?: 'polite' | 'assertive' | 'off';
  id?: string;
}) => {
  return (
    <div
      id={id}
      role="status"
      aria-live={priority}
      aria-atomic="true"
      className="sr-only"
    >
      {message}
    </div>
  );
};

// 폼 검증 에러 알림
export const FormWithValidation = () => {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  const [announcement, setAnnouncement] = useState('');

  const validateEmail = (value: string) => {
    if (!value) {
      setError('이메일을 입력해주세요.');
      setAnnouncement('이메일 입력 오류: 이메일을 입력해주세요.');
      return false;
    }
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
      setError('올바른 이메일 형식이 아닙니다.');
      setAnnouncement('이메일 형식 오류: 올바른 이메일 형식이 아닙니다.');
      return false;
    }
    setError('');
    setAnnouncement('');
    return true;
  };

  return (
    <form>
      <LiveRegion message={announcement} priority="assertive" />
      <div>
        <label htmlFor="email">이메일</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => {
            setEmail(e.target.value);
            if (error) {
              validateEmail(e.target.value);
            }
          }}
          onBlur={() => validateEmail(email)}
          aria-invalid={!!error}
          aria-describedby={error ? 'email-error' : undefined}
        />
        {error && (
          <div id="email-error" role="alert" aria-live="assertive">
            {error}
          </div>
        )}
      </div>
    </form>
  );
};

// 토스트 알림 시스템
export const ToastProvider = ({ children }) => {
  const [toasts, setToasts] = useState<Array<{ id: string; message: string }>>([]);

  const addToast = (message: string) => {
    const id = Math.random().toString(36);
    setToasts((prev) => [...prev, { id, message }]);
    setTimeout(() => {
      setToasts((prev) => prev.filter((t) => t.id !== id));
    }, 3000);
  };

  return (
    <ToastContext.Provider value={{ addToast }}>
      {children}
      <div
        role="region"
        aria-live="polite"
        aria-atomic="false"
        className="toast-container"
      >
        {toasts.map((toast) => (
          <div key={toast.id} role="status">
            {toast.message}
          </div>
        ))}
      </div>
    </ToastContext.Provider>
  );
};

// 검색 결과 갱신 알림
export const SearchResults = ({ query, results }) => {
  const [announcement, setAnnouncement] = useState('');

  useEffect(() => {
    if (results.length > 0) {
      setAnnouncement(`${results.length}개의 검색 결과를 찾았습니다.`);
    } else if (query) {
      setAnnouncement('검색 결과가 없습니다.');
    }
  }, [results, query]);

  return (
    <>
      <LiveRegion message={announcement} priority="polite" />
      <div role="region" aria-label="검색 결과">
        {results.map((result) => (
          <div key={result.id}>{result.title}</div>
        ))}
      </div>
    </>
  );
};

// 로딩 상태 알림
export const DataLoader = () => {
  const [isLoading, setIsLoading] = useState(false);
  const [data, setData] = useState(null);

  const loadData = async () => {
    setIsLoading(true);
    try {
      const result = await fetchData();
      setData(result);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <>
      <LiveRegion
        message={isLoading ? '데이터를 불러오는 중입니다.' : '데이터 로드가 완료되었습니다.'}
        priority="polite"
      />
      <button onClick={loadData}>Load Data</button>
      {isLoading && <div>Loading...</div>}
      {data && <div>{/* 데이터 표시 */}</div>}
    </>
  );
};
```

## 구현 방법

1. **aria-live 속성 설정**: polite(비동기) 또는 assertive(즉시) 선택
2. **role 속성 사용**: status(일반 알림) 또는 alert(중요 알림)
3. **aria-atomic 설정**: 전체 영역을 하나의 단위로 읽을지 결정
4. **메시지 명확성**: 사용자가 이해할 수 있는 명확한 메시지 작성
5. **중복 방지**: 같은 메시지 반복 알림 방지 로직 구현
6. **시각적으로 숨김**: 스크린 리더 전용 영역은 시각적으로 숨김 처리

## 장점

- 스크린 리더 사용자 경험 대폭 향상
- 동적 콘텐츠 변경을 실시간으로 인지 가능
- 폼 검증 등에서 즉각적인 피드백 제공
- 접근성 표준 준수

## 단점

- 과도한 알림은 사용자 경험 저하
- 메시지 작성 시 주의 필요
- 브라우저/스크린 리더별 동작 차이 가능
