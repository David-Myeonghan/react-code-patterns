# 20. useDeferredValue

useDeferredValue — 값의 업데이트를 지연시켜 렌더링 우선순위를 낮추는 패턴

## 동작 원리

`useDeferredValue`는 전달받은 값의 **지연된 버전**을 반환한다. 긴급 업데이트(사용자 입력 등)가 발생하면 React는 먼저 원래 값으로 긴급 렌더링을 수행하고, 그 후 백그라운드에서 지연된 값으로 **낮은 우선순위 리렌더링**을 진행한다.

`useTransition`과의 핵심 차이: `useTransition`은 상태 업데이트를 `startTransition`으로 감싸서 우선순위를 낮추지만, `useDeferredValue`는 **이미 존재하는 값**의 반영을 지연시킨다. 즉, 상태 업데이트 코드에 접근할 수 없는 경우(props로 받은 값, 외부 라이브러리가 관리하는 값)에 특히 유용하다.

원래 값과 지연된 값을 비교하여 `query !== deferredQuery` 같은 조건으로 "아직 반영되지 않은 상태(stale)"를 감지하고, 시각적 피드백(opacity 변경 등)을 제공할 수 있다.

## 장점

- **외부 값 지연 가능**: props로 받은 값처럼 상태 업데이트에 직접 접근할 수 없는 값도 지연 가능
- **간단한 API**: 값을 감싸기만 하면 되므로 `useTransition`보다 사용이 간단
- **stale 상태 감지**: 원래 값과 지연된 값을 비교하여 로딩/stale 상태 시각적 표현 가능
- **자동 중단**: 새 값이 들어오면 이전 지연 렌더링을 자동으로 중단

## 단점/주의사항

- **시각적 지연**: 사용자가 최신 값이 아닌 이전 값을 잠시 보게 됨 — 적절한 시각적 피드백(opacity 등) 필요
- **항상 한 박자 늦음**: 지연된 값은 항상 원래 값보다 뒤처짐
- **과도한 사용 주의**: 지연이 필요 없는 값에 사용하면 불필요한 이중 렌더링 발생
- **useTransition과 선택 기준**: 상태 업데이트에 접근 가능하면 `useTransition`이 더 적합

## 적합한 사용 사례

- 검색 결과 표시: 입력 값을 즉시 반영하되, 결과 리스트 렌더링은 지연
- 부모로부터 props로 받은 값에 기반한 무거운 렌더링
- 외부 라이브러리가 관리하는 값의 지연 반영
- 필터/정렬 기준 변경 시 대규모 리스트 업데이트 지연

## 구현 예제

```tsx
function SearchResults({ query }: { query: string }) {
  const deferredQuery = useDeferredValue(query)  // 입력보다 지연
  const isStale = query !== deferredQuery         // 아직 반영 안 됨

  return (
    <div style={{ opacity: isStale ? 0.5 : 1 }}>
      <HeavySearchResults query={deferredQuery} />
    </div>
  )
}
```
