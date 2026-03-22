# 9. Custom Hooks

커스텀 훅 — `use`로 시작하는 함수로 상태 로직을 추출하고 재사용하는 패턴

---

## 동작 원리

커스텀 훅은 React의 내장 훅(`useState`, `useEffect`, `useRef` 등)을 조합하여 재사용 가능한 상태 로직을 캡슐화한 함수이다. `use`로 시작하는 네이밍 컨벤션을 따르며, 호출하는 컴포넌트마다 독립된 상태를 가진다.

HOC와 Render Props가 해결하던 로직 재사용 문제를 더 간결하고 직관적으로 해결하여, 현재 React에서 **가장 주류인 패턴**(90%+ 사용)이다.

---

## 장점

- **간결함**: HOC나 Render Props 대비 코드량이 적고 가독성이 높음
- **합성 용이**: 여러 훅을 하나의 커스텀 훅에서 조합하여 복잡한 로직 구성 가능
- **독립된 상태**: 같은 훅을 여러 컴포넌트에서 사용해도 각각 독립된 상태를 유지
- **테스트 용이**: 훅을 독립적으로 테스트할 수 있음 (`@testing-library/react-hooks`)
- **래퍼 없음**: 컴포넌트 트리에 불필요한 래퍼를 추가하지 않음

---

## 단점/주의사항

- **훅 규칙(Rules of Hooks)**: 최상위에서만 호출해야 하며, 조건문/반복문/중첩 함수 안에서 호출 불가
- 컴포넌트가 아닌 일반 함수에서는 사용할 수 없음 (훅 또는 컴포넌트 내부에서만)
- 과도한 추상화는 오히려 코드를 이해하기 어렵게 만들 수 있음 — 2~3줄짜리 로직은 훅으로 추출할 필요 없음
- 현재 권장도: ★★★ (주류)

---

## 적합한 사용 사례

- **데이터 페칭**: API 호출, 로딩/에러 상태 관리
- **폼 관리**: 입력값, 유효성 검사, 제출 로직
- **브라우저 API**: localStorage, 미디어 쿼리, 인터섹션 옵저버
- **이벤트 핸들링**: 마우스 위치, 키보드 단축키, 스크롤 위치
- **타이머/인터벌**: 디바운스, 쓰로틀, 카운트다운
- **인증 상태**: 로그인 여부, 권한 확인

---

## 구현 예제

```tsx
// 데이터 페칭 훅
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null)
  const [error, setError] = useState<Error | null>(null)
  const [isLoading, setIsLoading] = useState(true)

  useEffect(() => {
    const controller = new AbortController()
    fetch(url, { signal: controller.signal })
      .then(r => r.json())
      .then(setData)
      .catch(setError)
      .finally(() => setIsLoading(false))
    return () => controller.abort()
  }, [url])

  return { data, error, isLoading }
}

// 사용
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading } = useFetch<User>(`/api/users/${userId}`)
  if (isLoading) return <Spinner />
  return <h1>{user.name}</h1>
}
```
