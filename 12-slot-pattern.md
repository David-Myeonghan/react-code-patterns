# 12. Slot Pattern (asChild)

슬롯 패턴 — 컴포넌트의 기본 DOM 요소를 소비자의 자식 요소로 완전히 교체하는 패턴

## 동작 원리

Slot Pattern은 `asChild` prop이 `true`일 때, 컴포넌트가 자체적으로 렌더링하는 기본 DOM 요소 대신 소비자가 전달한 자식 요소를 렌더링한다. 이때 컴포넌트의 props(스타일, 이벤트 핸들러, aria 속성 등)가 자식 요소에 **병합(merge)** 된다. 내부적으로 `React.cloneElement`를 사용하여 자식 요소에 추가 props를 주입하는 방식이 일반적이다.

Polymorphic(`as`) 패턴과의 차이점: `as`는 HTML 요소 타입만 변경하는 반면, `asChild`는 자식 요소를 완전히 대체하므로 React Router의 `<Link>`, Next.js의 `<Link>` 등 서드파티 컴포넌트로도 교체할 수 있다.

## 장점

- **완전한 요소 교체**: 서드파티 컴포넌트(React Router Link, Next.js Link 등)로도 교체 가능
- **Props 병합**: 컴포넌트의 스타일, 접근성 속성, 이벤트 핸들러가 자식 요소에 자동 병합
- **타입 안전성**: Polymorphic 패턴보다 TypeScript 타입 정의가 단순
- **유연한 합성**: 소비자가 렌더링되는 요소의 완전한 제어권을 가짐

## 단점/주의사항

- **단일 자식 제약**: `asChild` 사용 시 반드시 하나의 자식 요소만 전달해야 함
- **Props 충돌**: 컴포넌트 props와 자식 요소 props가 충돌할 때 병합 규칙이 복잡해질 수 있음
- **디버깅 어려움**: `cloneElement` 기반이라 props가 어디서 주입되었는지 추적하기 어려울 수 있음
- **학습 곡선**: 패턴 자체가 다소 암묵적이어서 처음 접하는 개발자에게 혼란을 줄 수 있음

## 적합한 사용 사례

- 컴포넌트 라이브러리 (Radix UI가 대표적 사례)
- 라우터 Link 컴포넌트와 디자인 시스템 Button의 결합
- 시맨틱 HTML 요소를 유지하면서 컴포넌트의 동작/스타일을 적용해야 할 때

## 구현 예제

```tsx
// Radix UI 스타일 asChild
<Button>기본 button 태그</Button>

<Button asChild>
  <a href="/about">a 태그로 렌더링</a>  {/* Button의 스타일+동작, a의 시맨틱 */}
</Button>

<Button asChild>
  <Link to="/home">React Router Link로 렌더링</Link>
</Button>
```
