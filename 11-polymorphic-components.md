# 11. Polymorphic Components

다형성 컴포넌트 — `as` prop으로 렌더링되는 HTML 요소를 동적으로 변경하는 패턴

## 동작 원리

Polymorphic Component는 `as` prop을 통해 컴포넌트가 실제로 렌더링하는 HTML 요소(또는 다른 React 컴포넌트)를 소비자가 결정할 수 있게 한다. 내부적으로 `as` prop에 전달된 값을 컴포넌트 변수에 할당하고, JSX에서 해당 변수를 동적 태그로 사용한다. TypeScript의 `ElementType`과 `ComponentPropsWithoutRef` 제네릭을 활용하면 전달된 요소 타입에 맞는 prop 자동완성과 타입 체크를 제공할 수 있다.

## 장점

- **유연한 시맨틱 HTML**: 동일한 디자인 시스템 컴포넌트를 `<span>`, `<h1>`, `<a>`, `<label>` 등 다양한 HTML 요소로 렌더링 가능
- **코드 재사용**: 스타일과 로직을 하나의 컴포넌트에 유지하면서 시맨틱을 소비자가 결정
- **타입 안전성**: TypeScript 제네릭을 활용하면 `as="a"`일 때 `href` prop이 자동으로 사용 가능해짐
- **디자인 시스템에 적합**: Mantine, Chakra UI 등 주요 디자인 시스템에서 채택한 검증된 패턴

## 단점/주의사항

- **TypeScript 복잡도**: 제네릭 타입 정의가 복잡하고, 특히 `as`와 나머지 props의 교차 타입을 정확히 정의하기 어려움
- **ref 전달 문제**: `as`로 전달된 컴포넌트에 ref를 올바르게 전달하려면 추가 타입 작업 필요
- **Props 충돌**: 커스텀 props와 네이티브 HTML attributes가 이름이 겹칠 수 있음 (e.g., `size`, `color`)
- **Slot Pattern(asChild)과 비교**: Radix UI 등은 `as` 대신 `asChild` 패턴을 선호하는 추세 — `as`는 요소 타입만 변경하지만 `asChild`는 자식 요소를 완전히 대체

## 적합한 사용 사례

- 디자인 시스템의 기본 컴포넌트 (`Text`, `Box`, `Button`, `Heading`)
- 시맨틱 HTML이 중요한 접근성 중심 컴포넌트
- 동일한 스타일을 여러 HTML 요소에 적용해야 하는 경우

## 구현 예제

```tsx
type PolymorphicProps<E extends ElementType> = {
  as?: E
  children: ReactNode
} & ComponentPropsWithoutRef<E>

function Text<E extends ElementType = 'span'>({
  as, children, ...props
}: PolymorphicProps<E>) {
  const Component = as || 'span'
  return <Component {...props}>{children}</Component>
}

// 사용 — 같은 컴포넌트, 다른 HTML 요소
<Text>기본 span</Text>
<Text as="h1" className="title">제목</Text>
<Text as="a" href="/about">링크</Text>
<Text as="label" htmlFor="name">라벨</Text>
```
