# 18. Suspense

서스펜스 — 비동기 작업(데이터 로딩, 코드 스플리팅) 중 폴백 UI를 선언적으로 표시하는 패턴

## 동작 원리

`<Suspense>`는 자식 컴포넌트 트리에서 아직 렌더링할 준비가 되지 않은 부분이 있을 때 **폴백(fallback) UI**를 대신 표시한다. 내부적으로 자식 컴포넌트가 Promise를 throw하면(suspend), 가장 가까운 Suspense 경계가 이를 감지하고 `fallback` prop에 지정된 UI를 렌더링한다. Promise가 resolve되면 원래 컴포넌트를 다시 렌더링한다.

Suspense 경계는 **중첩이 가능**하다. 이를 통해 페이지의 각 섹션마다 독립적인 로딩 상태를 관리할 수 있다. 상위 Suspense 경계는 하위 경계가 처리하지 못한 suspend를 잡아준다.

## 장점

- **선언적 로딩 상태**: `isLoading` 같은 상태를 수동으로 관리하지 않아도 됨
- **중첩 가능**: 페이지의 각 섹션마다 독립적인 로딩 경계를 설정 가능
- **코드 스플리팅과 통합**: `React.lazy()`와 함께 사용하여 지연 로딩 컴포넌트의 로딩 UI 표시
- **스트리밍 SSR 지원**: 서버에서 HTML을 점진적으로 스트리밍할 때 각 Suspense 경계가 독립적으로 해결
- **useTransition과 연동**: 전환 중 기존 UI를 유지하면서 새 콘텐츠가 준비되기를 기다릴 수 있음

## 단점/주의사항

- **Suspense 호환 데이터 소스 필요**: 일반적인 `useEffect` + `fetch`로는 Suspense가 동작하지 않음. React Query, Relay, Next.js 등 Suspense를 지원하는 라이브러리/프레임워크가 필요
- **폴백 UI 설계**: Skeleton, Spinner 등 적절한 폴백 UI를 각 경계마다 설계해야 함
- **레이아웃 시프트**: 폴백에서 실제 콘텐츠로 전환될 때 레이아웃이 이동할 수 있음 (Skeleton으로 완화)
- **워터폴 방지**: 중첩된 Suspense가 순차적 요청(waterfall)을 유발하지 않도록 병렬 데이터 페칭 전략 필요

## 적합한 사용 사례

- 코드 스플리팅 (`React.lazy()`)과 함께 지연 로드되는 컴포넌트의 로딩 UI
- 데이터 페칭 중 Skeleton/Spinner 표시 (React Query, Next.js 등)
- 페이지의 독립적인 섹션별 로딩 상태 관리
- 스트리밍 SSR에서 점진적 콘텐츠 렌더링

## 구현 예제

```tsx
<Suspense fallback={<PageSkeleton />}>
  <Header />
  <Suspense fallback={<ContentSkeleton />}>
    <MainContent />     {/* 데이터 로딩 중 ContentSkeleton 표시 */}
  </Suspense>
  <Suspense fallback={<SidebarSkeleton />}>
    <Sidebar />          {/* 독립적으로 로딩 상태 관리 */}
  </Suspense>
</Suspense>
```
