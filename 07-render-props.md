# 7. Render Props

렌더 프롭스 — 함수를 prop으로 전달하여, 부모의 내부 상태/로직에 접근하면서 렌더링을 소비자가 결정하는 패턴

---

## 동작 원리

컴포넌트가 자신의 내부 상태나 로직을 함수 형태의 prop(주로 `render` 또는 `children`)을 통해 외부에 노출한다. 소비자는 이 함수에서 전달받은 값을 사용하여 원하는 UI를 렌더링한다.

이 패턴은 **로직의 재사용**과 **렌더링의 유연성**을 동시에 제공한다. 컴포넌트가 "무엇을 렌더링할지"를 소비자에게 위임하면서, "어떤 데이터를 제공할지"만 담당한다.

---

## 장점

- 로직(상태, 이벤트 처리)과 UI(렌더링)의 완전한 분리
- 같은 로직에 다양한 UI를 적용할 수 있는 높은 유연성
- HOC와 달리 컴포넌트 합성이 명시적이므로 디버깅이 용이

---

## 단점/주의사항

- **대부분 커스텀 훅으로 대체됨**: 현재는 렌더링 범위 제어가 필요한 특수한 경우에만 사용
- 중첩이 깊어지면 "콜백 지옥"과 유사한 가독성 문제 발생 (wrapper hell)
- 매 렌더링마다 새로운 함수가 생성되어 성능에 영향을 줄 수 있음
- 현재 권장도: ★☆☆ (레거시)

---

## 적합한 사용 사례

- 동일한 로직에 서로 다른 렌더링이 필요한 경우
- 라이브러리에서 렌더링 커스터마이징을 제공할 때
- 커스텀 훅으로 대체하기 어려운 렌더링 범위 제어가 필요한 경우

---

## 구현 예제

```tsx
// Render Props 패턴
function MouseTracker({ render }: { render: (pos: {x:number, y:number}) => ReactNode }) {
  const [pos, setPos] = useState({ x: 0, y: 0 })

  return (
    <div onMouseMove={e => setPos({ x: e.clientX, y: e.clientY })}>
      {render(pos)}
    </div>
  )
}

<MouseTracker render={({ x, y }) => (
  <p>마우스 위치: {x}, {y}</p>
)} />

// 동일한 로직을 커스텀 훅으로 (현대적 대안)
function useMousePosition() {
  const [pos, setPos] = useState({ x: 0, y: 0 })
  useEffect(() => {
    const handler = (e: MouseEvent) => setPos({ x: e.clientX, y: e.clientY })
    window.addEventListener('mousemove', handler)
    return () => window.removeEventListener('mousemove', handler)
  }, [])
  return pos
}
```
