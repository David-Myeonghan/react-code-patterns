# 28. External Store (Zustand / Jotai / Redux)

외부 스토어 패턴 (구독 기반 상태 관리)

## 동작 원리

- React 외부에 상태를 두고, **구독(subscribe) 기반**으로 상태 변경 시 관련 컴포넌트만 리렌더링
- React의 `useSyncExternalStore`를 내부적으로 활용하여 React와 동기화
- **Zustand**: Store 패턴 — `create`로 스토어 생성, selector로 필요한 슬라이스만 구독
- **Jotai**: Atomic 패턴 — `atom`으로 최소 단위 상태 정의, 파생 atom으로 계산값 생성
- **Redux Toolkit**: Flux 패턴 — slice 기반 reducer, 미들웨어 지원

## 장점

- 필요한 상태 조각만 구독하여 **선택적 리렌더링** — Context의 전체 리렌더링 문제 해결
- React 컴포넌트 트리 외부에서도 상태 접근 가능 (유틸 함수, 미들웨어 등)
- DevTools 지원으로 상태 변경 추적과 디버깅이 용이
- 서버 상태는 React Query/TanStack Query, 클라이언트 상태는 외부 스토어로 역할 분리

## 단점/주의사항

- 외부 의존성 추가 — 번들 크기 증가 (Zustand는 ~1KB로 경량)
- 단순한 앱에서는 `useState` + Context로 충분할 수 있음
- 여러 스토어 라이브러리를 혼용하면 데이터 흐름 추적이 어려워짐
- SSR 환경에서 스토어 초기화/hydration 처리에 주의 필요

## 적합한 사용 사례

- 여러 컴포넌트에서 공유하는 클라이언트 상태 (장바구니, UI 설정, 사용자 선호)
- 자주 변경되는 상태 (실시간 데이터, 알림 카운트)
- 복잡한 상태 로직이 필요한 대규모 애플리케이션
- 서버 상태와 클라이언트 상태를 명확히 분리하고 싶을 때

## 구현 예제

### Zustand — Store 패턴

```tsx
// Zustand — Store 패턴 (가장 인기)
import { create } from 'zustand'

const useCartStore = create<CartStore>((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) => set((state) => ({
    items: state.items.filter(i => i.id !== id)
  })),
  total: 0,
}))

// 컴포넌트 — 필요한 슬라이스만 구독 (선택적 리렌더링)
function CartCount() {
  const count = useCartStore(state => state.items.length)  // items.length만 구독
  return <span>{count}</span>
}
```

### Jotai — Atomic 패턴

```tsx
// Jotai — Atomic 패턴
import { atom, useAtom } from 'jotai'

const countAtom = atom(0)
const doubledAtom = atom((get) => get(countAtom) * 2)  // 파생 atom

function Counter() {
  const [count, setCount] = useAtom(countAtom)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```
