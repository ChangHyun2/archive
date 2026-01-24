# Heavy Work Offloading

## 개요

무거운 계산, 파싱, 정렬 등의 작업을 메인 스레드에서 분리하여 Web Worker로 처리하거나, 서버로 보내거나, 캐시하여 메인 스레드의 프리즈를 방지하는 패턴입니다.

## 핵심 원칙

- **메인 스레드 보호**: UI 스레드를 블로킹하지 않기
- **비동기 처리**: 무거운 작업은 별도 스레드나 서버에서 처리
- **캐싱 활용**: 반복되는 계산은 캐시하여 재사용

## 해결 방법

### 1. Web Worker 사용

```tsx
// worker.js
self.onmessage = function(e) {
  const { data } = e.data;
  
  // 무거운 계산
  const result = heavyComputation(data);
  
  self.postMessage({ result });
};

// Component
function HeavyComputationComponent({ inputData }) {
  const [result, setResult] = useState(null);
  const [isProcessing, setIsProcessing] = useState(false);
  
  useEffect(() => {
    if (!inputData) return;
    
    setIsProcessing(true);
    const worker = new Worker('/worker.js');
    
    worker.postMessage({ data: inputData });
    
    worker.onmessage = (e) => {
      setResult(e.data.result);
      setIsProcessing(false);
      worker.terminate();
    };
    
    worker.onerror = (error) => {
      console.error('Worker error:', error);
      setIsProcessing(false);
      worker.terminate();
    };
    
    return () => {
      worker.terminate();
    };
  }, [inputData]);
  
  if (isProcessing) {
    return <div>Processing...</div>;
  }
  
  return <div>{result}</div>;
}
```

### 2. 서버로 오프로딩

```tsx
// ❌ 나쁜 예: 클라이언트에서 무거운 작업
function DataProcessor({ rawData }) {
  const [processed, setProcessed] = useState(null);
  
  useEffect(() => {
    // 메인 스레드 블로킹
    const result = heavyDataProcessing(rawData);
    setProcessed(result);
  }, [rawData]);
  
  return <Results data={processed} />;
}

// ✅ 좋은 예: 서버로 오프로딩
function DataProcessor({ rawData }) {
  const [processed, setProcessed] = useState(null);
  const [isProcessing, setIsProcessing] = useState(false);
  
  useEffect(() => {
    setIsProcessing(true);
    
    fetch('/api/process-data', {
      method: 'POST',
      body: JSON.stringify({ data: rawData })
    })
      .then(res => res.json())
      .then(result => {
        setProcessed(result);
        setIsProcessing(false);
      })
      .catch(error => {
        console.error(error);
        setIsProcessing(false);
      });
  }, [rawData]);
  
  if (isProcessing) {
    return <ProcessingIndicator />;
  }
  
  return <Results data={processed} />;
}
```

### 3. 캐싱 활용

```tsx
import { useMemo } from 'react';

// ✅ 계산 결과 캐싱
function ExpensiveCalculation({ input }) {
  const result = useMemo(() => {
    return heavyCalculation(input);
  }, [input]);
  
  return <div>{result}</div>;
}

// ✅ 서버 응답 캐싱 (React Query 예시)
import { useQuery } from '@tanstack/react-query';

function DataVisualization({ filters }) {
  const { data, isLoading } = useQuery({
    queryKey: ['processed-data', filters],
    queryFn: () => fetchProcessedData(filters),
    staleTime: 5 * 60 * 1000, // 5분간 캐시
  });
  
  if (isLoading) return <Loading />;
  
  return <Chart data={data} />;
}
```

### 4. 청크 단위 처리

```tsx
// ✅ 큰 데이터를 작은 청크로 나누어 처리
function processLargeDataset(data, onProgress) {
  const chunkSize = 1000;
  const chunks = [];
  
  for (let i = 0; i < data.length; i += chunkSize) {
    chunks.push(data.slice(i, i + chunkSize));
  }
  
  return new Promise((resolve) => {
    const results = [];
    let processed = 0;
    
    const processChunk = (index) => {
      if (index >= chunks.length) {
        resolve(results.flat());
        return;
      }
      
      // 각 청크를 비동기로 처리
      setTimeout(() => {
        const chunkResult = processChunkData(chunks[index]);
        results.push(chunkResult);
        processed++;
        
        onProgress?.(processed / chunks.length);
        
        processChunk(index + 1);
      }, 0);
    };
    
    processChunk(0);
  });
}

function LargeDataProcessor({ data }) {
  const [result, setResult] = useState(null);
  const [progress, setProgress] = useState(0);
  
  useEffect(() => {
    processLargeDataset(data, (progress) => {
      setProgress(progress);
    }).then(setResult);
  }, [data]);
  
  return (
    <div>
      <ProgressBar value={progress} />
      {result && <Results data={result} />}
    </div>
  );
}
```

### 5. requestIdleCallback 활용

```tsx
// ✅ 브라우저 유휴 시간에 처리
function useIdleWork(callback, deps) {
  useEffect(() => {
    if (!window.requestIdleCallback) {
      // 폴백: setTimeout
      const timeoutId = setTimeout(callback, 0);
      return () => clearTimeout(timeoutId);
    }
    
    const idleCallbackId = requestIdleCallback(callback, {
      timeout: 2000 // 최대 2초 대기
    });
    
    return () => cancelIdleCallback(idleCallbackId);
  }, deps);
}

function Component() {
  useIdleWork(() => {
    // 유휴 시간에 실행할 작업
    performNonCriticalWork();
  }, []);
  
  return <div>...</div>;
}
```

## 언제 어떤 방법을 사용할까?

- **Web Worker**: 순수 계산 작업, 이미지 처리, 데이터 변환
- **서버 오프로딩**: 데이터베이스 쿼리, 복잡한 비즈니스 로직
- **캐싱**: 반복되는 계산, API 응답
- **청크 처리**: 큰 데이터셋 처리
- **requestIdleCallback**: 중요하지 않은 백그라운드 작업

## Best Practices

1. **작업 분류**: 작업의 특성에 맞는 방법 선택
2. **진행 상황 표시**: 오래 걸리는 작업은 진행 상황 표시
3. **에러 처리**: 모든 비동기 작업에 에러 처리
4. **리소스 정리**: Worker, 타이머 등 리소스 정리
5. **폴백 제공**: Web Worker 미지원 환경 대비

## 주의사항

- Web Worker는 DOM 접근 불가
- 서버 오프로딩은 네트워크 비용 고려
- 캐싱은 메모리 사용량 고려
- requestIdleCallback은 모든 브라우저에서 지원되지 않음
