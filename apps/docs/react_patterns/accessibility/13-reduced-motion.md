# Reduced Motion 지원

## 개요

Reduced Motion 지원은 prefers-reduced-motion 미디어 쿼리를 존중하여 애니메이션을 최소화하거나 대체하는 패턴입니다. 현업에서는 "배려"를 넘어 "품질 기준"에 가까운 필수 요구사항입니다. 일부 사용자는 애니메이션으로 인한 어지러움, 메스꺼움, 집중력 저하를 경험할 수 있으므로 이를 고려해야 합니다.

## 특징

- **미디어 쿼리 감지**: prefers-reduced-motion 미디어 쿼리로 사용자 선호도 확인
- **애니메이션 최소화**: 애니메이션을 완전히 제거하거나 최소화
- **대체 효과 제공**: 애니메이션 대신 페이드 인/아웃 등 부드러운 전환
- **접근성 향상**: 모든 사용자가 편안하게 사용 가능

## 예시

```tsx
// CSS로 Reduced Motion 처리
const styles = `
  @media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
  }
`;

// React에서 Reduced Motion 감지
export const useReducedMotion = () => {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(false);

  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    setPrefersReducedMotion(mediaQuery.matches);

    const handleChange = (e: MediaQueryListEvent) => {
      setPrefersReducedMotion(e.matches);
    };

    mediaQuery.addEventListener('change', handleChange);
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, []);

  return prefersReducedMotion;
};

// 애니메이션 컴포넌트 - Reduced Motion 지원
export const AnimatedCard = ({ children }) => {
  const prefersReducedMotion = useReducedMotion();

  return (
    <div
      className={prefersReducedMotion ? 'no-animation' : 'animate-in'}
      style={{
        transition: prefersReducedMotion ? 'none' : 'all 0.3s ease',
      }}
    >
      {children}
    </div>
  );
};

// 모달 컴포넌트 - Reduced Motion 지원
export const Modal = ({ isOpen, onClose, children }) => {
  const prefersReducedMotion = useReducedMotion();

  return (
    <AnimatePresence>
      {isOpen && (
        <motion.div
          initial={prefersReducedMotion ? { opacity: 1 } : { opacity: 0 }}
          animate={prefersReducedMotion ? { opacity: 1 } : { opacity: 1 }}
          exit={prefersReducedMotion ? { opacity: 1 } : { opacity: 0 }}
          transition={prefersReducedMotion ? { duration: 0 } : { duration: 0.2 }}
        >
          {children}
        </motion.div>
      )}
    </AnimatePresence>
  );
};

// Framer Motion과 함께 사용
import { motion, useReducedMotion } from 'framer-motion';

export const AnimatedButton = ({ children, onClick }) => {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.button
      onClick={onClick}
      whileHover={shouldReduceMotion ? {} : { scale: 1.05 }}
      whileTap={shouldReduceMotion ? {} : { scale: 0.95 }}
      transition={shouldReduceMotion ? { duration: 0 } : { duration: 0.2 }}
    >
      {children}
    </motion.button>
  );
};

// 페이지 전환 - Reduced Motion 지원
export const PageTransition = ({ children }) => {
  const prefersReducedMotion = useReducedMotion();

  return (
    <motion.div
      initial={prefersReducedMotion ? {} : { opacity: 0, y: 20 }}
      animate={prefersReducedMotion ? {} : { opacity: 1, y: 0 }}
      exit={prefersReducedMotion ? {} : { opacity: 0, y: -20 }}
      transition={prefersReducedMotion ? { duration: 0 } : { duration: 0.3 }}
    >
      {children}
    </motion.div>
  );
};

// 스크롤 애니메이션 - Reduced Motion 지원
export const ScrollReveal = ({ children }) => {
  const prefersReducedMotion = useReducedMotion();
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (prefersReducedMotion) return;

    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            entry.target.classList.add('revealed');
          }
        });
      },
      { threshold: 0.1 }
    );

    if (ref.current) {
      observer.observe(ref.current);
    }

    return () => observer.disconnect();
  }, [prefersReducedMotion]);

  return (
    <div
      ref={ref}
      className={prefersReducedMotion ? 'no-animation' : 'scroll-reveal'}
    >
      {children}
    </div>
  );
};

// CSS-in-JS 예시
import styled, { css } from 'styled-components';

const AnimatedBox = styled.div`
  transition: transform 0.3s ease;

  @media (prefers-reduced-motion: reduce) {
    transition: none;
  }

  &:hover {
    transform: translateY(-4px);

    @media (prefers-reduced-motion: reduce) {
      transform: none;
    }
  }
`;

// Tailwind CSS 예시
export const Component = () => {
  return (
    <div className="transition-all duration-300 motion-reduce:transition-none motion-reduce:transform-none hover:scale-105">
      Content
    </div>
  );
};
```

## 구현 방법

1. **미디어 쿼리 감지**: window.matchMedia로 prefers-reduced-motion 확인
2. **커스텀 훅 생성**: useReducedMotion 훅으로 전역적으로 사용
3. **조건부 애니메이션**: 애니메이션 적용 전 reduced motion 확인
4. **CSS 미디어 쿼리**: CSS에서 직접 prefers-reduced-motion 처리
5. **애니메이션 라이브러리 설정**: Framer Motion 등에서 reduced motion 옵션 활용
6. **대체 효과**: 애니메이션 대신 즉시 표시 또는 페이드 효과

## 장점

- 모든 사용자에게 편안한 경험 제공
- 접근성 표준 준수
- 사용자 선호도 존중
- 법적 요구사항 충족 (일부 지역)
- 사용자 만족도 향상

## 단점

- 모든 애니메이션에 대한 조건부 처리 필요
- 테스트 범위 확대
- 일부 시각적 효과 감소
