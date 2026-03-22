# 26. Context / Provider

컨텍스트 패턴 (prop drilling 해결)

## 동작 원리

- `createContext`로 Context를 생성하고, `Provider`로 값을 제공
- 하위 컴포넌트에서 `useContext`(또는 React 19의 `use`)로 값을 읽음
- **prop drilling 없이** 컴포넌트 트리 깊은 곳까지 데이터를 전달
- Context 값이 변경되면 해당 Context를 구독하는 **모든 소비자가 리렌더링**됨

## 장점

- 중간 컴포넌트를 거치지 않고 데이터를 깊이 전달 가능
- 테마, 인증, 로케일 등 전역 상태에 적합
- Provider 패턴으로 관심사 분리가 명확해짐
- React 내장 API이므로 추가 의존성 불필요

## 단점/주의사항

- Context 값 변경 시 **모든 소비자가 리렌더링** — 도메인별 Context 분리 필수
- 모놀리식 Context(`{ user, theme, locale, cart }`)는 성능 문제의 원인
- 자주 변경되는 값(마우스 좌표, 스크롤 위치 등)에는 부적합 — 외부 스토어 권장
- Context가 남용되면 컴포넌트 간 암묵적 의존이 생겨 테스트와 재사용이 어려워짐

## 적합한 사용 사례

- 테마(다크모드/라이트모드) 관리
- 인증 상태 (로그인 사용자 정보)
- 다국어(i18n) 로케일 설정
- Compound Component의 내부 상태 공유
- 변경 빈도가 낮은 전역 설정

## 구현 예제

### 도메인별 Context 분리

```tsx
// 도메인별 Context 분리
const AuthContext = createContext<AuthState>(null!)
const ThemeContext = createContext<ThemeState>(null!)

// BAD — 모놀리식 Context (모든 변경이 전체 리렌더링)
const AppContext = createContext({ user, theme, locale, cart })

// GOOD — 분리된 Provider
<AuthProvider>
  <ThemeProvider>
    <App />
  </ThemeProvider>
</AuthProvider>
```
