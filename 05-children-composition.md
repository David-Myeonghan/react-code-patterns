# 5. Children / Composition

자식 합성 — `children` prop으로 컴포넌트 내부에 임의의 JSX를 삽입하는 패턴

---

## 동작 원리

React에서 컴포넌트의 여는 태그와 닫는 태그 사이에 작성한 내용은 자동으로 `children` prop으로 전달된다. 컴포넌트는 이 `children`을 내부 JSX의 원하는 위치에 렌더링하여, 외부에서 내부 콘텐츠를 주입할 수 있다.

**Named Slots**: `children` 외에 여러 prop(`header`, `sidebar` 등)으로 다중 삽입 지점을 제공하면, Vue의 slot과 유사한 패턴을 구현할 수 있다.

---

## 장점

- **유연한 구성**: 부모가 자식의 내부 콘텐츠를 자유롭게 결정
- **재사용성**: 레이아웃, 카드, 모달 등 구조적 컴포넌트를 범용으로 활용
- **prop drilling 회피**: 중간 컴포넌트를 거치지 않고 직접 자식을 전달할 수 있음
- **관심사 분리**: 컨테이너(구조)와 콘텐츠(내용)를 명확히 분리

---

## 단점/주의사항

- `children`의 타입이 `ReactNode`로 매우 넓어, 예상치 못한 콘텐츠가 전달될 수 있음
- Named Slots가 많아지면 prop이 복잡해질 수 있음 — Compound Components 패턴이 대안
- 자식 컴포넌트에 부모의 상태를 전달하려면 `React.cloneElement` 또는 Render Props가 필요할 수 있음

---

## 적합한 사용 사례

- 카드, 패널, 모달 등 **레이아웃 래퍼** 컴포넌트
- 페이지 레이아웃 (header, sidebar, main 영역)
- 스타일링 컨테이너
- 에러 바운더리, 인증 래퍼 등 기능적 래퍼

---

## 구현 예제

```tsx
// 기본 children
function Card({ children }: { children: ReactNode }) {
  return <div className="card">{children}</div>
}

// Named Slots (다중 삽입 지점)
function Layout({ header, sidebar, children }: {
  header: ReactNode
  sidebar: ReactNode
  children: ReactNode
}) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
    </div>
  )
}

<Layout
  header={<Nav />}
  sidebar={<MenuList />}
>
  <PageContent />
</Layout>
```
