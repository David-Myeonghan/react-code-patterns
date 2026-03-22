# 1. Conditional Rendering

조건부 렌더링 — JSX 안에서 조건에 따라 다른 UI를 렌더링하는 패턴

---

## 동작 원리

JSX 내부에서 JavaScript의 조건 표현식을 활용하여, 특정 조건에 따라 서로 다른 컴포넌트나 요소를 렌더링한다. React는 매 렌더링마다 JSX를 평가하므로, 조건이 변경되면 자동으로 다른 UI가 표시된다.

주요 방법:
- **삼항 연산자** (`condition ? A : B`): 두 가지 분기를 명확히 표현
- **`&&` 단축평가** (`condition && <Component />`): 조건이 true일 때만 렌더링
- **객체 매핑**: switch/case를 대체하여 상태별 컴포넌트를 매핑
- **즉시실행함수(IIFE)**: 복잡한 분기 로직을 JSX 내부에서 처리

---

## 장점

- React의 가장 기본적인 패턴으로 학습 비용이 낮음
- 별도의 라이브러리나 추상화 없이 순수 JavaScript로 구현
- 객체 매핑 방식은 새로운 상태 추가 시 확장이 용이함
- 컴포넌트 단위로 조건을 분리하면 가독성이 높아짐

---

## 단점/주의사항

- **`&&` 단축평가의 falsy 함정**: `0 && <Component />`는 `0`을 화면에 렌더링한다. `number` 타입을 `&&` 앞에 사용할 때는 반드시 `Boolean(count)` 또는 `count > 0`으로 변환해야 한다
- 분기가 많아지면 JSX가 복잡해져 가독성이 떨어짐 — 이 경우 별도 컴포넌트로 분리하거나 객체 매핑 사용
- 중첩된 삼항 연산자는 코드 리뷰 시 이해하기 어려움

---

## 적합한 사용 사례

- 로그인/비로그인 상태에 따른 UI 전환
- 로딩/에러/성공 상태별 표시
- 권한(role)에 따른 패널 분기
- 알림 배지 등 조건부 표시 요소

---

## 구현 예제

```tsx
// 삼항 연산자
{isLoggedIn ? <Dashboard /> : <Login />}

// && 단축평가
{hasNotifications && <NotificationBadge count={count} />}

// 객체 매핑 (switch 대체)
const statusMap = {
  loading: <Spinner />,
  error: <ErrorMessage />,
  success: <DataView data={data} />,
}
return statusMap[status] ?? <Fallback />

// 즉시실행함수
{(() => {
  if (role === 'admin') return <AdminPanel />
  if (role === 'editor') return <EditorPanel />
  return <ViewerPanel />
})()}
```
