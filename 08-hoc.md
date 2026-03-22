# 8. Higher-Order Components (HOC)

고차 컴포넌트 — 컴포넌트를 입력받아 기능이 추가된 새 컴포넌트를 반환하는 함수 패턴

---

## 동작 원리

HOC는 컴포넌트를 인자로 받아, 추가 기능(인증 체크, 로깅, 데이터 주입 등)을 래핑한 새로운 컴포넌트를 반환하는 **함수**이다. 원본 컴포넌트를 수정하지 않고 기능을 확장하는 데코레이터 패턴과 유사하다.

```
InputComponent → withSomething(InputComponent) → EnhancedComponent
```

원본 컴포넌트의 props는 그대로 전달하면서, HOC가 추가 로직(조건부 렌더링, 데이터 주입 등)을 처리한다.

---

## 장점

- 횡단 관심사(cross-cutting concerns)를 한 곳에서 관리
- 원본 컴포넌트를 수정하지 않고 기능 확장 (개방-폐쇄 원칙)
- 여러 HOC를 합성하여 기능을 누적할 수 있음

---

## 단점/주의사항

- **대부분 커스텀 훅으로 대체됨**: 현재는 특수한 래핑 패턴에서만 사용
- "래퍼 지옥(wrapper hell)": 여러 HOC를 중첩하면 컴포넌트 트리가 깊어지고 디버깅이 어려움
- Props 충돌: HOC가 주입하는 props와 원본 props의 이름이 겹칠 수 있음
- 정적 메서드가 자동으로 복사되지 않음
- Ref 전달에 추가 처리가 필요함
- 현재 권장도: ★☆☆ (레거시)

---

## 적합한 사용 사례

- **인증 체크**: 로그인되지 않은 사용자를 리디렉션
- **로깅/분석**: 컴포넌트 마운트/언마운트 추적
- **테마/스타일 주입**: 컴포넌트에 스타일 props 자동 주입
- 레거시 코드베이스에서 기존 HOC 유지보수

---

## 구현 예제

```tsx
// HOC — 인증 체크
function withAuth<P extends object>(WrappedComponent: ComponentType<P>) {
  return function AuthenticatedComponent(props: P) {
    const { user, isLoading } = useAuth()

    if (isLoading) return <Spinner />
    if (!user) return <Navigate to="/login" />

    return <WrappedComponent {...props} />
  }
}

const ProtectedDashboard = withAuth(Dashboard)

// 현대적 대안 — 커스텀 훅
function useRequireAuth() {
  const { user, isLoading } = useAuth()
  useEffect(() => {
    if (!isLoading && !user) navigate('/login')
  }, [user, isLoading])
  return { user, isLoading }
}
```
