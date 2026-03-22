# 22. use() Hook

React 19의 새로운 데이터 읽기 API

## 동작 원리

- React 19에서 도입된 `use()` 훅으로 렌더링 중에 **Promise나 Context를 읽음**
- Promise를 전달하면 resolve될 때까지 컴포넌트가 suspend됨 (Suspense와 결합)
- Context를 전달하면 `useContext`와 동일하게 동작하지만, **조건문 안에서도 사용 가능**
- 기존 훅 규칙(최상위에서만 호출)에서 자유로운 유일한 훅

## 장점

- 조건부로 Context를 읽을 수 있어 불필요한 구독 방지
- Promise를 직접 읽어 데이터 페칭 코드가 간결해짐
- Suspense와 자연스럽게 통합되어 선언적 로딩 처리 가능
- 기존 `useContext` 대비 더 유연한 사용 패턴

## 단점/주의사항

- React 19 이상에서만 사용 가능
- Promise를 컴포넌트 내부에서 생성하면 매 렌더마다 새 Promise가 생성됨 — 반드시 외부에서 전달해야 함
- Suspense 경계가 없으면 Promise 사용 시 에러 발생
- 아직 생태계 전반의 채택이 진행 중

## 적합한 사용 사례

- 서버 컴포넌트에서 클라이언트 컴포넌트로 Promise를 전달하여 데이터 읽기
- 조건에 따라 다른 Context를 읽어야 하는 경우
- Suspense 기반 데이터 페칭 패턴
- 기존 `useContext` 호출을 조건부로 최적화

## 구현 예제

### Promise 읽기 — Suspense와 결합

```tsx
// Promise 읽기 — Suspense와 결합
function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise)  // Promise가 resolve될 때까지 suspend
  return <h1>{user.name}</h1>
}

// 조건부 Context 읽기
function Theme({ isSpecial }: { isSpecial: boolean }) {
  if (isSpecial) {
    const theme = use(ThemeContext)  // 조건문 안에서 OK (useContext는 불가)
    return <div style={{ color: theme.primary }}>Special</div>
  }
  return <div>Normal</div>
}
```
