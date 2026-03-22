# 6. Compound Components

복합 컴포넌트 — 부모-자식 컴포넌트가 암묵적 상태를 공유하여 하나의 단위로 동작하는 패턴

---

## 동작 원리

부모 컴포넌트가 Context를 통해 내부 상태를 제공하고, 자식 컴포넌트들이 이 Context를 구독하여 부모의 상태에 접근한다. 사용자는 자식 컴포넌트의 순서와 조합을 자유롭게 변경할 수 있으며, 상태 관리는 부모가 암묵적으로 담당한다.

HTML의 `<select>` + `<option>` 관계와 유사하다. `<select>`가 상태를 관리하고, `<option>`은 그 상태에 따라 선택 여부가 결정된다.

---

## 장점

- **선언적 API**: 사용자가 구조를 보면 동작을 즉시 이해할 수 있음
- **유연한 조합**: 자식 컴포넌트의 순서, 포함 여부를 자유롭게 결정
- **캡슐화**: 내부 상태 관리가 숨겨져 있어 사용이 단순함
- **관심사 분리**: 각 자식 컴포넌트가 자신의 역할에만 집중

---

## 단점/주의사항

- Context 기반이므로 구현 복잡도가 높음
- 자식 컴포넌트가 부모 바깥에서 사용되면 Context 에러가 발생할 수 있음 — 방어적 코드 필요
- TypeScript 타입 정의가 다소 복잡해질 수 있음
- 너무 깊은 중첩은 오히려 가독성을 해침

---

## 적합한 사용 사례

- **탭(Tabs)**: Tabs.List, Tabs.Trigger, Tabs.Content
- **아코디언(Accordion)**: 열림/닫힘 상태 공유
- **드롭다운(Dropdown)**: 메뉴 열림/선택 상태 관리
- **모달(Modal)**: 트리거, 오버레이, 콘텐츠 분리
- Radix UI, Headless UI 등 UI 라이브러리의 핵심 패턴

---

## 구현 예제

```tsx
// 사용 예시 — 선언적이고 유연한 API
<Tabs defaultValue="tab1">
  <Tabs.List>
    <Tabs.Trigger value="tab1">탭 1</Tabs.Trigger>
    <Tabs.Trigger value="tab2">탭 2</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Content value="tab1">첫 번째 탭 내용</Tabs.Content>
  <Tabs.Content value="tab2">두 번째 탭 내용</Tabs.Content>
</Tabs>

// 구현 (Context 기반)
const TabsContext = createContext<TabsContextValue>(null!)

function Tabs({ defaultValue, children }) {
  const [active, setActive] = useState(defaultValue)
  return (
    <TabsContext.Provider value={{ active, setActive }}>
      {children}
    </TabsContext.Provider>
  )
}

function Trigger({ value, children }) {
  const { active, setActive } = useContext(TabsContext)
  return (
    <button
      role="tab"
      aria-selected={active === value}
      onClick={() => setActive(value)}
    >
      {children}
    </button>
  )
}

Tabs.List = ({ children }) => <div role="tablist">{children}</div>
Tabs.Trigger = Trigger
Tabs.Content = ({ value, children }) => {
  const { active } = useContext(TabsContext)
  return active === value ? <div role="tabpanel">{children}</div> : null
}
```
