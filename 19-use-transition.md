# 19. useTransition

useTransition / startTransition — 상태 업데이트를 낮은 우선순위로 표시하여 UI 응답성을 유지하는 패턴

## 동작 원리

`useTransition`은 상태 업데이트를 **긴급(urgent)** 업데이트와 **전환(transition)** 업데이트로 분리한다. `startTransition`으로 감싼 상태 업데이트는 낮은 우선순위로 처리되며, 이 업데이트가 진행되는 동안에도 사용자 입력(타이핑, 클릭 등) 같은 긴급 업데이트가 즉시 반영된다.

React의 Concurrent Rendering 엔진은 전환 업데이트를 렌더링하는 도중에 더 긴급한 업데이트가 들어오면, 진행 중인 전환 렌더링을 **중단(interrupt)** 하고 긴급 업데이트를 먼저 처리한다. `isPending` 플래그를 통해 전환이 진행 중인지 확인하여 로딩 인디케이터를 표시할 수 있다.

## 장점

- **UI 응답성 유지**: 무거운 상태 업데이트 중에도 사용자 입력이 즉시 반응
- **자동 중단**: 새로운 전환 업데이트가 들어오면 이전 전환 렌더링을 자동으로 중단
- **isPending 플래그**: 전환 진행 상태를 쉽게 추적하여 시각적 피드백 제공
- **Suspense와 연동**: 전환 중 Suspense fallback 대신 기존 UI를 유지할 수 있음

## 단점/주의사항

- **동기 작업만 해당**: `startTransition`은 setState 같은 동기 상태 업데이트에만 적용됨. 비동기 네트워크 요청 자체를 지연시키지는 않음
- **전환 우선순위 이해 필요**: 어떤 업데이트를 긴급으로, 어떤 것을 전환으로 분류할지 개발자가 판단해야 함
- **과도한 사용 주의**: 모든 상태 업데이트를 전환으로 감싸면 오히려 반응성이 떨어질 수 있음
- **useDeferredValue와의 차이**: `useTransition`은 상태 업데이트를 감싸고, `useDeferredValue`는 값 자체를 지연. 상태 업데이트에 접근 가능하면 `useTransition`, 외부에서 받은 값이면 `useDeferredValue` 사용

## 적합한 사용 사례

- 검색 필터링: 입력은 즉시, 결과 리스트 업데이트는 전환으로 처리
- 탭 전환: 탭 클릭은 즉시, 콘텐츠 렌더링은 전환으로 처리
- 대규모 리스트/차트 업데이트: 데이터 변경 시 무거운 리렌더링을 전환으로 분류
- 페이지 네비게이션: 라우트 전환 시 이전 페이지를 유지하면서 새 페이지 준비

## 구현 예제

```tsx
function FilterableList({ items }: { items: Item[] }) {
  const [query, setQuery] = useState('')
  const [filtered, setFiltered] = useState(items)
  const [isPending, startTransition] = useTransition()

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    setQuery(e.target.value)            // 즉시 반영 (높은 우선순위)
    startTransition(() => {
      setFiltered(items.filter(         // 나중에 반영 (낮은 우선순위)
        item => item.name.includes(e.target.value)
      ))
    })
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <List items={filtered} />
    </>
  )
}
```
