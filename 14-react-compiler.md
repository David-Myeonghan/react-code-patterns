# 14. React Compiler

React Compiler — 수동 메모이제이션을 자동화하는 빌드 타임 최적화 도구

## 동작 원리

React Compiler(2025년 1.0 출시)는 빌드 시점에 React 컴포넌트 코드를 분석하여 `useMemo`, `useCallback`, `React.memo`에 해당하는 최적화를 **자동으로 삽입**한다. 컴파일러는 컴포넌트 내부의 값과 함수가 어떤 의존성에 기반하는지 정적 분석을 통해 파악하고, 의존성이 변경되지 않았을 때 이전 결과를 재사용하도록 코드를 변환한다.

개발자는 메모이제이션을 신경 쓰지 않고 자연스러운 JavaScript로 코드를 작성하면 되고, 컴파일러가 최적의 메모이제이션 전략을 자동 적용한다.

## 장점

- **개발자 경험 향상**: `useMemo`, `useCallback`, `React.memo`를 수동으로 작성할 필요 없음
- **실수 방지**: 의존성 배열 누락, 과도한 메모이제이션, 불필요한 메모이제이션 등의 실수를 컴파일러가 방지
- **코드 가독성 개선**: 메모이제이션 boilerplate 없이 의도가 명확한 코드 작성 가능
- **일관된 최적화**: 팀 전체에서 일관된 수준의 성능 최적화를 자동으로 달성

## 단점/주의사항

- **코드 규칙 준수 필요**: React의 Rules of React(순수 렌더링, 훅 규칙 등)를 엄격히 따라야 컴파일러가 올바르게 최적화 가능
- **디버깅 복잡성**: 컴파일된 코드가 원본과 다르기 때문에 디버깅 시 소스맵 필요
- **서드파티 호환성**: 모든 서드파티 라이브러리가 컴파일러와 완벽히 호환되는 것은 아님
- **여전히 수동이 필요한 경우**: 서드파티 API의 참조 동등성 의존, 의도적 캐싱 전략 등에서는 수동 메모이제이션이 여전히 필요

## 적합한 사용 사례

- React 19 이상을 사용하는 새 프로젝트
- 기존 프로젝트에서 수동 메모이제이션 boilerplate를 제거하고 싶을 때
- 팀 전체의 성능 최적화 수준을 일관되게 맞추고 싶을 때

## 구현 예제

```tsx
// React Compiler 이전 — 수동 메모이제이션 필요
const filtered = useMemo(() => items.filter(predicate), [items, predicate])
const handler = useCallback(() => doSomething(id), [id])

// React Compiler 이후 — 그냥 작성하면 컴파일러가 최적화
const filtered = items.filter(predicate)  // 컴파일러가 자동 메모이제이션
const handler = () => doSomething(id)     // 컴파일러가 자동 안정화
```
