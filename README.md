# React Code Patterns

React 앱 코드 안에서 컴포넌트가 렌더링을 제어하는 **29가지 패턴**을 정리한 종합 가이드입니다.
각 패턴별 동작 원리, 구현 코드 예제, 언제 사용하고 언제 피해야 하는지를 다룹니다.

> 서버 렌더링 전략(CSR/SSR/SSG/ISR/RSC/PPR 등)은 [react-rendering-patterns](https://github.com/David-Myeonghan/react-rendering-patterns)을 참고하세요.

## Patterns

### A. 기본 렌더링

| # | 패턴 | 핵심 | 권장도 | 문서 |
|---|------|------|--------|------|
| 1 | **Conditional Rendering** | 조건에 따라 다른 UI 렌더링 | ★★★ 필수 | [상세](01-conditional-rendering.md) |
| 2 | **List Rendering** | 배열을 `.map()`으로 컴포넌트 리스트 변환 | ★★★ 필수 | [상세](02-list-rendering.md) |
| 3 | **Controlled Components** | React state가 입력값의 단일 진실 공급원 | ★★★ 필수 | [상세](03-controlled-components.md) |
| 4 | **Uncontrolled Components** | DOM이 값을 보유, ref로 필요시 읽기 | ★★☆ 상황적 | [상세](04-uncontrolled-components.md) |

### B. 컴포넌트 합성

| # | 패턴 | 핵심 | 권장도 | 문서 |
|---|------|------|--------|------|
| 5 | **Children / Composition** | children prop으로 임의 JSX 삽입 | ★★★ 필수 | [상세](05-children-composition.md) |
| 6 | **Compound Components** | 부모-자식이 암묵적 상태 공유 | ★★★ 권장 | [상세](06-compound-components.md) |
| 7 | **Render Props** | 함수 prop으로 렌더링 위임 | ★☆☆ 레거시 | [상세](07-render-props.md) |
| 8 | **HOC** | 컴포넌트를 감싸서 기능 추가 | ★☆☆ 레거시 | [상세](08-hoc.md) |
| 9 | **Custom Hooks** | use 함수로 상태 로직 추출/재사용 | ★★★ 주류 | [상세](09-custom-hooks.md) |
| 10 | **Headless Components** | 로직만 제공, UI는 소비자 결정 | ★★★ 권장 | [상세](10-headless-components.md) |
| 11 | **Polymorphic Components** | `as` prop으로 HTML 요소 동적 변경 | ★★☆ 상황적 | [상세](11-polymorphic-components.md) |
| 12 | **Slot Pattern (asChild)** | 기본 DOM을 자식 요소로 교체 | ★★☆ 상황적 | [상세](12-slot-pattern.md) |

### C. 성능 최적화

| # | 패턴 | 핵심 | 권장도 | 문서 |
|---|------|------|--------|------|
| 13 | **Memoization** | React.memo / useMemo / useCallback | ★★☆ 감소 중 | [상세](13-memoization.md) |
| 14 | **React Compiler** | 자동 메모이제이션 (2025 출시) | ★★★ 신규 | [상세](14-react-compiler.md) |
| 15 | **Code Splitting** | React.lazy + Suspense 지연 로딩 | ★★★ 필수 | [상세](15-code-splitting.md) |
| 16 | **Virtualization** | 뷰포트 내 아이템만 DOM 렌더링 | ★★★ 필수(대량) | [상세](16-virtualization.md) |
| 17 | **State Colocation** | 상태를 사용처 가까이 배치 | ★★★ 필수 | [상세](17-state-colocation.md) |

### D. 동시성 렌더링

| # | 패턴 | 핵심 | 권장도 | 문서 |
|---|------|------|--------|------|
| 18 | **Suspense** | 비동기 작업 중 폴백 UI 선언적 표시 | ★★★ 필수 | [상세](18-suspense.md) |
| 19 | **useTransition** | 상태 업데이트를 낮은 우선순위로 | ★★★ 권장 | [상세](19-use-transition.md) |
| 20 | **useDeferredValue** | 값 업데이트를 지연하여 우선순위 낮춤 | ★★☆ 상황적 | [상세](20-use-deferred-value.md) |
| 21 | **useOptimistic** | 서버 응답 전 즉시 UI 업데이트 | ★★★ 권장 | [상세](21-use-optimistic.md) |
| 22 | **use() Hook** | 렌더링 중 Promise/Context 읽기 | ★★☆ 신규 | [상세](22-use-hook.md) |

### E. DOM / 구조 패턴

| # | 패턴 | 핵심 | 권장도 | 문서 |
|---|------|------|--------|------|
| 23 | **Portal** | DOM 트리의 다른 위치에 렌더링 | ★★★ 필수 | [상세](23-portal.md) |
| 24 | **Error Boundary** | 렌더링 에러 캐치 + 폴백 UI | ★★★ 필수 | [상세](24-error-boundary.md) |
| 25 | **forwardRef** | 부모가 자식 DOM 노드에 접근 | ★★★ 필수 | [상세](25-forward-ref.md) |

### F. 상태 관리 렌더링

| # | 패턴 | 핵심 | 권장도 | 문서 |
|---|------|------|--------|------|
| 26 | **Context / Provider** | prop drilling 없이 데이터 전파 | ★★★ 필수 | [상세](26-context-provider.md) |
| 27 | **Reducer Pattern** | action + reducer로 상태 중앙화 | ★★☆ 상황적 | [상세](27-reducer-pattern.md) |
| 28 | **External Store** | Zustand/Jotai/Redux 외부 스토어 | ★★★ 권장 | [상세](28-external-store.md) |
| 29 | **State Machine** | 명시적 상태/전환 정의 (XState) | ★★☆ 상황적 | [상세](29-state-machine.md) |

### G. 명령형 패턴

| # | 패턴 | 핵심 | 권장도 | 문서 |
|---|------|------|--------|------|
| 30 | **Promise-based Dialog** | await으로 다이얼로그 결과를 기다리는 명령형 패턴 | ★★★ 권장 | [상세](30-promise-based-dialog.md) |

## Sources

- [React Design Patterns: Complete Guide (2026)](https://www.turbodocx.com/blog/react-design-patterns)
- [React Stack Patterns (patterns.dev)](https://www.patterns.dev/react/react-2026/)
- [Render Props Pattern](https://www.patterns.dev/react/render-props-pattern/)
- [Compound Pattern](https://www.patterns.dev/react/compound-pattern/)
- [Headless Component Pattern - Martin Fowler](https://martinfowler.com/articles/headless-component.html)
- [React Compiler Introduction](https://react.dev/learn/react-compiler/introduction)
- [React v19 Official Blog](https://react.dev/blog/2024/12/05/react-19)
- [useOptimistic - React Docs](https://react.dev/reference/react/useOptimistic)
- [useTransition - React Docs](https://react.dev/reference/react/useTransition)
- [createPortal - React Docs](https://react.dev/reference/react-dom/createPortal)
- [TanStack Virtual](https://tanstack.com/virtual/latest)
- [Slot Pattern (asChild)](https://dev.to/pandresdev/aschild-understanding-the-slot-pattern-in-react-ifo)
- [Polymorphic Components - Mantine](https://mantine.dev/guides/polymorphic/)
- [State Management in 2026](https://dev.to/jsgurujobs/state-management-in-2026-zustand-vs-jotai-vs-redux-toolkit-vs-signals-2gge)
- [Concurrent Rendering & Suspense](https://medium.com/@rupalsinghal/from-freeze-to-flow-how-react-in-2025-uses-concurrent-rendering-and-suspense-to-change-everything-40d202f732e2)
