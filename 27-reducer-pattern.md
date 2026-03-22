# 27. Reducer Pattern / useReducer

리듀서 패턴 (복잡한 상태 관리)

## 동작 원리

- `useReducer` 훅으로 상태 업데이트 로직을 **reducer 함수에 중앙화**
- 컴포넌트는 `dispatch(action)`으로 상태 변경을 요청하고, reducer가 새 상태를 계산
- Redux의 action/reducer 패턴과 동일한 개념이지만, React 내장 훅으로 로컬 사용
- 상태 전환이 예측 가능하고, 각 action이 명시적으로 정의됨

## 장점

- 복잡한 상태 로직이 하나의 reducer 함수에 모여 관리와 테스트가 용이
- 여러 연관된 상태 값을 하나의 state 객체로 통합 관리
- dispatch는 참조가 안정적이므로 자식 컴포넌트에 전달 시 불필요한 리렌더링 방지
- 상태 전환 로직을 컴포넌트 외부에 분리할 수 있어 테스트하기 쉬움

## 단점/주의사항

- 단순한 상태(boolean 토글, 단일 값)에는 `useState`가 더 적합 — 오버엔지니어링 주의
- boilerplate가 `useState` 대비 많음 (action type 정의, switch 문 등)
- 비동기 로직을 직접 처리하지 못함 — 미들웨어 없이는 side effect 처리가 별도 필요
- 상태가 너무 커지면 외부 스토어(Zustand, Redux Toolkit) 전환을 고려해야 함

## 적합한 사용 사례

- 여러 필드가 연관된 복잡한 폼 상태
- 상태 전환이 명시적이어야 하는 UI (토글, 스텝 진행, 모달 상태 등)
- 이전 상태에 의존하는 업데이트가 많은 경우
- Context + useReducer 조합으로 간단한 전역 상태 관리

## 구현 예제

### 카운터 — action 기반 상태 관리

```tsx
type State = { count: number; step: number }
type Action = { type: 'increment' } | { type: 'decrement' } | { type: 'setStep'; step: number }

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment': return { ...state, count: state.count + state.step }
    case 'decrement': return { ...state, count: state.count - state.step }
    case 'setStep':   return { ...state, step: action.step }
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 })
  return (
    <>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+{state.step}</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-{state.step}</button>
    </>
  )
}
```
