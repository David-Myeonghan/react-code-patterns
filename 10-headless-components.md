# 10. Headless Components

헤드리스 컴포넌트 — 로직과 상태만 제공하고, UI는 소비자가 완전히 결정하는 패턴

---

## 동작 원리

헤드리스 컴포넌트는 UI(스타일, 마크업)를 전혀 포함하지 않고, 오직 상태 관리와 동작 로직, 접근성(a11y) 속성만 제공한다. 주로 커스텀 훅 형태로 구현되며, 훅이 반환하는 props 객체를 소비자가 원하는 요소에 스프레드(`...`)하여 사용한다.

이 패턴은 "어떻게 동작하는가"와 "어떻게 보이는가"를 완전히 분리한다. Radix UI, Headless UI, Ark UI, Downshift 등의 라이브러리가 대표적이다.

---

## 장점

- **스타일링 완전 자유**: Tailwind, CSS Modules, styled-components 등 어떤 스타일링 방식이든 적용 가능
- **접근성(a11y) 내장**: `aria-pressed`, `role`, 키보드 이벤트 등 접근성 로직이 훅에 포함
- **테스트 용이**: 로직과 UI가 분리되어 있으므로 각각 독립적으로 테스트 가능
- **디자인 시스템 독립**: 특정 디자인 시스템에 종속되지 않아 다양한 프로젝트에서 재사용 가능
- 현재 권장도: ★★★ (권장)

---

## 단점/주의사항

- 사용자가 UI를 직접 구현해야 하므로 초기 설정 비용이 있음
- 접근성 속성을 올바르게 스프레드하지 않으면 a11y가 깨질 수 있음
- 간단한 UI에는 과도한 추상화일 수 있음 — 이미 스타일링된 컴포넌트 라이브러리가 더 빠를 수 있음
- 문서화가 잘 되어 있지 않으면 반환값의 사용법을 파악하기 어려움

---

## 적합한 사용 사례

- **UI 라이브러리 개발**: 스타일에 무관한 범용 컴포넌트 제공
- **디자인 시스템**: 여러 팀/프로젝트에서 서로 다른 스타일로 동일한 동작 사용
- **복잡한 인터랙션**: 드롭다운, 콤보박스, 날짜 선택기 등 접근성이 중요한 위젯
- **토글, 탭, 아코디언**: 동작 로직을 재사용하면서 다양한 비주얼 적용

---

## 구현 예제

```tsx
// 헤드리스 토글 훅
function useToggle(initial = false) {
  const [on, setOn] = useState(initial)
  const toggle = useCallback(() => setOn(v => !v), [])
  const buttonProps = {
    'aria-pressed': on,
    onClick: toggle,
  }
  return { on, toggle, buttonProps }
}

// 소비자가 UI를 자유롭게 결정
function FancyToggle() {
  const { on, buttonProps } = useToggle()
  return (
    <button {...buttonProps} className={on ? 'bg-green' : 'bg-gray'}>
      {on ? 'ON' : 'OFF'}
    </button>
  )
}
```
