# 2. List Rendering

리스트 렌더링 — 배열 데이터를 `.map()`으로 컴포넌트 리스트로 변환하는 패턴

---

## 동작 원리

JavaScript 배열의 `.map()` 메서드를 사용하여 데이터 배열을 JSX 요소 배열로 변환한다. React는 각 요소를 구분하기 위해 `key` prop을 요구하며, 이 key를 기반으로 재조정(reconciliation) 알고리즘이 어떤 요소가 추가/삭제/변경되었는지 판단한다.

**key의 역할**: React가 리스트 아이템을 식별하기 위한 고유값으로, 재조정 성능의 핵심이다. key가 안정적이고 고유해야 React가 DOM 업데이트를 최소화할 수 있다.

---

## 장점

- 데이터 기반의 선언적 UI 구성이 가능
- 데이터 변경 시 React가 자동으로 최소한의 DOM 업데이트를 수행
- 중첩 리스트, 조건부 렌더링 등 다른 패턴과 자연스럽게 결합

---

## 단점/주의사항

- **index를 key로 사용하면 안 되는 경우**: 리스트 항목의 순서가 변경되거나 삭제될 때, index 기반 key는 React가 잘못된 요소를 재사용하게 만들어 버그가 발생한다 (입력값이 다른 항목에 남아있는 등)
- 대규모 리스트(수천 개 이상)에서는 가상화(Virtualization) 패턴을 함께 적용해야 성능 문제를 방지할 수 있다
- 빈 리스트에 대한 처리(Empty State)를 반드시 고려해야 한다

---

## 적합한 사용 사례

- 게시글 목록, 댓글 목록, 상품 리스트 등 배열 데이터 표시
- 카테고리별 중첩 리스트
- 검색 결과 표시
- 테이블 행 렌더링

---

## 구현 예제

```tsx
// 기본 리스트 렌더링
{items.map(item => (
  <ListItem key={item.id} item={item} />
))}

// 중첩 리스트
{categories.map(cat => (
  <section key={cat.id}>
    <h2>{cat.name}</h2>
    {cat.items.map(item => (
      <Item key={item.id} item={item} />
    ))}
  </section>
))}

// 빈 리스트 처리
{items.length > 0 ? (
  items.map(item => <Item key={item.id} {...item} />)
) : (
  <EmptyState message="항목이 없습니다" />
)}
```
