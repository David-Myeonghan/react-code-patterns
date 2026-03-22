# 4. Uncontrolled Components

비제어 컴포넌트 — DOM 자체가 값을 보유하고, React는 필요할 때만 `ref`로 읽는 패턴

---

## 동작 원리

제어 컴포넌트와 달리, 입력값을 React state로 관리하지 않는다. 대신 DOM이 자체적으로 입력값을 보유하며, React는 `useRef`를 통해 필요한 시점(주로 폼 제출 시)에만 DOM에서 값을 읽어온다.

`defaultValue` prop으로 초기값을 설정할 수 있으며, 이후 입력값 변경은 React의 관여 없이 브라우저 네이티브 동작으로 처리된다.

---

## 장점

- **리렌더링 최소화**: 매 입력마다 state 업데이트가 없으므로 리렌더링이 발생하지 않음
- **단순한 코드**: state와 onChange 핸들러 없이 간결하게 작성 가능
- **서드파티 라이브러리 연동 용이**: DOM을 직접 조작하는 라이브러리와 자연스럽게 통합

---

## 단점/주의사항

- 실시간 유효성 검사, 입력 포맷팅 등 **입력 중** 값을 제어하는 것이 어려움
- 현재 폼 상태를 React에서 즉시 알 수 없으므로, 조건부 렌더링이 불편함
- 복잡한 폼에서는 여러 ref를 관리해야 하므로 코드가 복잡해질 수 있음

---

## 적합한 사용 사례

- **파일 업로드**: `<input type="file" />`은 읽기 전용이므로 반드시 비제어로 사용
- 제출 시점에만 값이 필요한 단순한 폼
- 서드파티 DOM 조작 라이브러리와의 연동
- 성능이 중요한 대량의 입력 필드

---

## 구현 예제

```tsx
function SimpleForm() {
  const inputRef = useRef<HTMLInputElement>(null)

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault()
    console.log(inputRef.current?.value)  // 제출 시점에만 DOM에서 값 읽기
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} defaultValue="기본값" />
      <button type="submit">제출</button>
    </form>
  )
}
```
