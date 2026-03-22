# 29. State Machine / XState

상태 머신 패턴 (명시적 상태 전환)

## 동작 원리

- 가능한 **상태(state)**와 **전환(transition)**을 명시적으로 정의
- 현재 상태에서 특정 이벤트가 발생하면 어떤 상태로 전환되는지 미리 선언
- **불가능한 상태 조합**을 원천 차단 (예: "로딩 중이면서 에러"인 상태가 존재할 수 없음)
- XState가 대표적인 라이브러리 — `createMachine`으로 머신 정의, `useMachine`으로 React에 연결
- 시각적 도구(XState Visualizer)로 상태 다이어그램을 확인할 수 있음

## 장점

- 모든 가능한 상태와 전환이 코드에 명시되어 예측 가능하고 디버깅이 쉬움
- 불가능한 상태 조합을 타입 수준에서 방지
- 복잡한 비동기 흐름(결제, 멀티스텝 폼 등)을 시각적으로 설계 가능
- 상태 다이어그램이 곧 문서 — 비개발자와의 커뮤니케이션에도 유용

## 단점/주의사항

- 학습 곡선이 가파름 — 상태 머신/상태도 개념에 대한 이해 필요
- 단순한 UI 상태에 사용하면 오버엔지니어링 (boolean 토글에 XState는 과함)
- XState 라이브러리 번들 크기가 상대적으로 큼
- 팀 전체가 상태 머신 패러다임에 익숙해야 유지보수가 원활함

## 적합한 사용 사례

- 결제 플로우 (idle → processing → success/failure → retry)
- 멀티스텝 폼/위자드 (step1 → step2 → ... → complete)
- 인증 흐름 (loggedOut → loggingIn → loggedIn → loggingOut)
- 미디어 플레이어 (stopped → playing → paused → stopped)
- 복잡한 모달/드로어 상태 관리

## 구현 예제

### XState 기본 사용법

```tsx
import { useMachine } from '@xstate/react'
import { createMachine } from 'xstate'

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: { on: { TOGGLE: 'active' } },
    active: { on: { TOGGLE: 'inactive' } },
  },
})

function Toggle() {
  const [state, send] = useMachine(toggleMachine)
  return (
    <button onClick={() => send({ type: 'TOGGLE' })}>
      {state.matches('active') ? 'ON' : 'OFF'}
    </button>
  )
}
```
