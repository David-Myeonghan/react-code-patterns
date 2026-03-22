# 24. Error Boundary

에러 바운더리 (렌더링 에러 복구)

## 동작 원리

- 자식 컴포넌트 트리에서 발생한 **렌더링 에러를 캐치**하여 앱 전체가 크래시되는 것을 방지
- 에러 발생 시 **폴백 UI**를 대신 표시
- 클래스 컴포넌트의 `getDerivedStateFromError`와 `componentDidCatch` 라이프사이클 메서드로 구현
- 함수형 컴포넌트로는 직접 구현 불가 — `react-error-boundary` 라이브러리 사용 권장
- Suspense와 함께 사용하면 로딩 + 에러 상태를 선언적으로 관리 가능

## 장점

- 하나의 컴포넌트 에러가 전체 앱을 다운시키지 않음
- 에러 복구 UI를 선언적으로 구성 가능
- 에러 발생 지점을 세분화하여 영향 범위를 최소화
- `resetErrorBoundary`로 에러 상태를 리셋하고 재시도 가능

## 단점/주의사항

- 클래스 컴포넌트로만 구현 가능 (함수형 컴포넌트 미지원)
- 이벤트 핸들러 내부의 에러는 캐치하지 못함 (try/catch 사용 필요)
- 비동기 코드(setTimeout, requestAnimationFrame 등)의 에러도 캐치 불가
- SSR 중 발생하는 에러는 Error Boundary로 처리할 수 없음

## 적합한 사용 사례

- 앱 전체를 감싸는 최상위 에러 처리
- 독립적인 위젯/섹션별 에러 격리
- Suspense와 조합한 데이터 로딩 + 에러 처리
- 서드파티 컴포넌트의 예기치 않은 에러 방어

## 구현 예제

### react-error-boundary 사용

```tsx
import { ErrorBoundary } from 'react-error-boundary'

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>문제가 발생했습니다: {error.message}</p>
      <button onClick={resetErrorBoundary}>다시 시도</button>
    </div>
  )
}

// Suspense + Error Boundary 조합
<ErrorBoundary FallbackComponent={ErrorFallback}>
  <Suspense fallback={<Spinner />}>
    <DataComponent />
  </Suspense>
</ErrorBoundary>
```
