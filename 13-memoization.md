# 13. Memoization

메모이제이션 — React.memo, useMemo, useCallback을 활용한 렌더링 최적화 패턴

## 동작 원리

React에서 메모이제이션은 세 가지 주요 API를 통해 이루어진다:

- **React.memo**: Higher-Order Component로, 컴포넌트를 래핑하여 이전 props와 현재 props를 **얕은 비교(shallow comparison)** 한다. props가 변경되지 않았으면 리렌더링을 건너뛰고 이전 렌더링 결과를 재사용한다.
- **useMemo**: 의존성 배열의 값이 변경될 때만 계산 함수를 재실행하고, 그렇지 않으면 캐시된 결과를 반환한다. 비용이 큰 계산(필터링, 정렬, 변환)에 적합하다.
- **useCallback**: 함수 자체를 메모이제이션한다. 의존성 배열이 변경되지 않으면 동일한 함수 참조를 유지하여, `React.memo`로 래핑된 자식 컴포넌트에 전달할 때 불필요한 리렌더링을 방지한다.

## 장점

- **불필요한 리렌더링 방지**: 비싼 컴포넌트가 props 변경 없이 부모 리렌더링에 의해 재렌더링되는 것을 방지
- **비싼 계산 캐싱**: 매 렌더마다 대규모 데이터를 필터링/정렬하는 것을 방지
- **참조 안정성**: `useCallback`으로 함수 참조를 안정화하여 `React.memo` 등과 조합 가능
- **세밀한 제어**: 정확히 어떤 값/함수/컴포넌트를 메모이제이션할지 개발자가 결정

## 단점/주의사항

- **과도한 사용 주의**: 메모이제이션 자체에도 비용(메모리, 비교 연산)이 있음. 실제 성능 병목이 확인된 후 적용해야 함
- **의존성 배열 관리**: 잘못된 의존성 배열은 stale data 버그의 원인
- **얕은 비교 한계**: `React.memo`는 기본적으로 shallow comparison만 수행 — 객체/배열이 매번 새로 생성되면 효과 없음
- **React Compiler 등장**: React Compiler(2025)가 자동 메모이제이션을 제공하므로, 수동 메모이제이션의 필요성이 점차 감소 중

## 적합한 사용 사례

- 대규모 리스트를 렌더링하는 컴포넌트 (`React.memo`)
- 비용이 큰 계산 (필터링, 정렬, 집계)을 매 렌더마다 반복하는 경우 (`useMemo`)
- `React.memo`로 최적화된 자식 컴포넌트에 함수를 전달하는 경우 (`useCallback`)
- 프로파일러로 확인된 실제 성능 병목 지점

## 구현 예제

```tsx
// React.memo — props 변경 없으면 리렌더링 스킵
const ExpensiveList = memo(function ExpensiveList({ items }: { items: Item[] }) {
  return items.map(item => <ListItem key={item.id} item={item} />)
})

// useMemo — 비싼 계산 결과 캐시
const filteredItems = useMemo(
  () => items.filter(item => item.name.includes(query)),
  [items, query]  // items나 query가 바뀔 때만 재계산
)

// useCallback — 함수 참조 안정화
const handleClick = useCallback((id: string) => {
  setSelected(id)
}, [])  // 빈 배열 → 함수가 절대 재생성되지 않음

<MemoizedChild onClick={handleClick} />  // handleClick 참조 안정 → 리렌더링 방지
```
