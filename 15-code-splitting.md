# 15. Code Splitting / Lazy Loading

코드 분할 — `React.lazy()`와 `<Suspense>`를 활용하여 컴포넌트를 필요한 시점에 동적으로 로드하는 패턴

## 동작 원리

`React.lazy()`는 동적 `import()` 문을 감싸서 컴포넌트를 lazy하게 로드한다. 해당 컴포넌트가 처음 렌더링될 때 번들러(Webpack, Vite 등)가 분리한 별도의 JavaScript 청크를 네트워크로 가져온다. 청크가 로드되는 동안 가장 가까운 `<Suspense>` 경계의 `fallback` UI가 표시된다.

번들러는 `import()` 문을 기준으로 코드를 자동으로 별도 청크로 분리하므로, 개발자는 어떤 컴포넌트를 분할할지만 결정하면 된다. 일반적으로 **라우트 단위**로 코드를 분할하는 것이 가장 효과적이다.

## 장점

- **초기 번들 크기 감소**: 사용자가 방문하지 않을 수 있는 페이지의 코드를 초기 번들에 포함하지 않음
- **초기 로딩 속도 개선**: 필요한 코드만 먼저 로드하여 Time to Interactive(TTI) 단축
- **온디맨드 로딩**: 특정 사용자 액션(탭 클릭, 모달 열기)에 의해 트리거되는 컴포넌트를 지연 로드
- **Suspense와 자연스러운 통합**: 로딩 상태를 선언적으로 관리 가능

## 단점/주의사항

- **로딩 지연**: 청크를 네트워크에서 가져오는 동안 사용자가 fallback UI를 봐야 함
- **네트워크 요청 증가**: 과도한 분할은 오히려 많은 HTTP 요청을 유발
- **청크 로드 실패**: 네트워크 문제 시 Error Boundary로 에러 처리 필요
- **SSR 제한**: `React.lazy()`는 기본적으로 클라이언트 사이드에서만 동작. SSR이 필요한 경우 프레임워크 자체 솔루션(Next.js `dynamic`) 활용
- **Named Export 미지원**: `React.lazy()`는 default export만 지원. Named export는 중간 모듈로 re-export 필요

## 적합한 사용 사례

- 라우트 기반 코드 분할 (가장 효과적)
- 무거운 라이브러리를 사용하는 컴포넌트 (차트, 에디터, 지도)
- 모달, 드로어 등 사용자 액션에 의해 표시되는 컴포넌트
- 관리자 전용 페이지 등 일부 사용자만 접근하는 기능

## 구현 예제

```tsx
const HeavyChart = lazy(() => import('./HeavyChart'))
const AdminPanel = lazy(() => import('./AdminPanel'))

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Routes>
        <Route path="/dashboard" element={<HeavyChart />} />
        <Route path="/admin" element={<AdminPanel />} />
      </Routes>
    </Suspense>
  )
}
```
