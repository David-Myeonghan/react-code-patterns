# 3. Controlled Components

제어 컴포넌트 — React state가 입력값의 단일 진실 공급원(single source of truth)이 되는 패턴

---

## 동작 원리

모든 입력 변경이 다음 흐름을 따른다:

1. 사용자가 입력값을 변경한다
2. `onChange` 핸들러가 호출되어 React state를 업데이트한다
3. state 변경으로 컴포넌트가 리렌더링된다
4. 리렌더링 시 `value` prop에 새로운 state가 반영된다

이 단방향 데이터 흐름 덕분에 React가 항상 입력값의 현재 상태를 알고 있으며, 중간에 값을 가공하거나 유효성을 검사할 수 있다.

---

## 장점

- **즉시 유효성 검사**: 매 입력마다 값을 검증하고 에러 메시지를 표시할 수 있음
- **입력 포맷팅**: 전화번호 자동 포맷팅, 대문자 변환 등 입력 중 값 변환이 가능
- **조건부 제출**: 모든 필드가 유효할 때만 제출 버튼 활성화
- **상태 추적**: 현재 폼의 정확한 상태를 언제든 알 수 있음

---

## 단점/주의사항

- 매 입력마다 리렌더링이 발생하므로, 대규모 폼에서 성능에 영향을 줄 수 있음
- 필드가 많은 폼에서는 boilerplate 코드가 증가함 — React Hook Form 같은 라이브러리로 해결 가능
- 단순히 제출 시점에만 값이 필요한 경우에는 과도한 제어일 수 있음 (비제어 컴포넌트가 더 적합)

---

## 적합한 사용 사례

- 실시간 유효성 검사가 필요한 폼 (회원가입, 결제 등)
- 입력값에 따라 다른 UI를 동적으로 표시하는 경우
- 여러 필드 간 상호 의존성이 있는 복잡한 폼
- 입력값의 포맷팅이나 마스킹이 필요한 경우

---

## 구현 예제

```tsx
function LoginForm() {
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={email}                          // React state가 값을 제어
        onChange={e => setEmail(e.target.value)} // 매 입력마다 state 업데이트
      />
      <input
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
      />
      <button disabled={!email || !password}>로그인</button>
    </form>
  )
}
```
