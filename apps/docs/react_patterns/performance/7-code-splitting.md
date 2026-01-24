# Code Splitting (Route/Feature 단위)

## 개요

애플리케이션의 코드를 여러 번들로 나누어 초기 로딩 시간을 줄이고, 필요한 코드만 동적으로 로드하는 패턴입니다. React.lazy와 Suspense를 사용하여 구현합니다.

## 핵심 원칙

- **초기 번들 최소화**: 처음 화면에 필요한 코드만 로드
- **지연 로딩**: 라우트나 기능 단위로 필요할 때 로드
- **사용자 경험 개선**: 초기 로딩 시간 단축으로 체감 성능 향상

## 사용 방법

### React.lazy와 Suspense

```tsx
import { lazy, Suspense } from 'react';

// ❌ 나쁜 예: 모든 컴포넌트를 한 번에 로드
import HomePage from './pages/HomePage';
import AboutPage from './pages/AboutPage';
import ContactPage from './pages/ContactPage';

function App() {
  return (
    <Router>
      <Route path="/" component={HomePage} />
      <Route path="/about" component={AboutPage} />
      <Route path="/contact" component={ContactPage} />
    </Router>
  );
}

// ✅ 좋은 예: 라우트별 코드 스플리팅
const HomePage = lazy(() => import('./pages/HomePage'));
const AboutPage = lazy(() => import('./pages/AboutPage'));
const ContactPage = lazy(() => import('./pages/ContactPage'));

function App() {
  return (
    <Router>
      <Suspense fallback={<Loading />}>
        <Route path="/" component={HomePage} />
        <Route path="/about" component={AboutPage} />
        <Route path="/contact" component={ContactPage} />
      </Suspense>
    </Router>
  );
}
```

### 기능 단위 코드 스플리팅

```tsx
// ✅ 모달, 차트 등 무거운 컴포넌트를 지연 로딩
const HeavyChart = lazy(() => import('./components/HeavyChart'));
const DataVisualization = lazy(() => import('./components/DataVisualization'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

### 동적 import와 조건부 로딩

```tsx
// ✅ 사용자 권한에 따른 조건부 로딩
function AdminPanel() {
  const [AdminComponent, setAdminComponent] = useState(null);
  
  useEffect(() => {
    if (user.isAdmin) {
      import('./AdminPanel').then(module => {
        setAdminComponent(() => module.default);
      });
    }
  }, [user.isAdmin]);
  
  if (!AdminComponent) return null;
  
  return <AdminComponent />;
}
```

### 라이브러리별 코드 스플리팅

```tsx
// ✅ React Router v6
import { lazy } from 'react';
import { Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Suspense>
  );
}

// ✅ Next.js (자동 코드 스플리팅)
// pages 디렉토리 구조가 자동으로 코드 스플리팅됨
```

## Webpack 설정 (필요한 경우)

```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
      },
    },
  },
};
```

## Best Practices

1. **라우트 단위 스플리팅**: 각 페이지를 별도 청크로 분리
2. **무거운 라이브러리 분리**: 차트, 에디터 등 큰 라이브러리는 별도 청크
3. **공통 코드 추출**: 여러 청크에서 사용하는 코드는 공통 청크로
4. **로딩 상태 제공**: Suspense fallback으로 사용자 경험 개선
5. **프리로딩 고려**: 다음에 사용할 가능성이 높은 코드는 미리 로드

## 주의사항

- 너무 많은 작은 청크는 오히려 성능 저하를 일으킬 수 있음
- 네트워크 요청이 많아질 수 있으므로 균형 필요
- SSR 환경에서는 추가 설정이 필요할 수 있음
