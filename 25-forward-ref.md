# 25. forwardRef / ref callback

Ref 전달 패턴

## 동작 원리

- 부모 컴포넌트가 자식 컴포넌트 내부의 **DOM 노드에 직접 접근**할 수 있도록 ref를 전달
- 기존에는 `forwardRef`로 감싸야만 함수형 컴포넌트에 ref를 전달할 수 있었음
- **React 19부터** `forwardRef` 없이 props에서 직접 `ref`를 받을 수 있음
- Ref callback을 사용하면 DOM 노드의 마운트/언마운트 시점을 감지 가능
- React 19에서 ref callback의 cleanup 함수 반환도 지원

## 장점

- 부모에서 자식 DOM 노드에 대한 포커스, 스크롤, 애니메이션 등을 직접 제어 가능
- 디자인 시스템의 재사용 가능한 입력 컴포넌트에 필수적
- React 19에서는 forwardRef 래퍼 없이 간결하게 작성 가능
- Ref callback으로 DOM 노드 측정(dimension), 외부 라이브러리 연동 등이 용이

## 단점/주의사항

- 자식 컴포넌트의 내부 구현이 부모에 노출되어 캡슐화가 약해질 수 있음
- `useImperativeHandle`로 노출 범위를 제한하는 것이 권장됨
- forwardRef 사용 시 TypeScript 제네릭 타이핑이 복잡해질 수 있음
- DOM 직접 조작은 React의 선언적 패러다임과 충돌할 수 있으므로 최소한으로 사용

## 적합한 사용 사례

- 커스텀 입력 컴포넌트에 포커스 전달
- 외부 라이브러리(D3, 지도 등)와 DOM 노드 연동
- 스크롤 위치 제어
- DOM 노드 크기 측정 (IntersectionObserver, ResizeObserver 등)

## 구현 예제

### React 19 — ref를 직접 props로 전달

```tsx
// React 19 — ref를 직접 props로 전달
function CustomInput({ ref, ...props }: { ref?: Ref<HTMLInputElement> }) {
  return <input ref={ref} {...props} className="custom-input" />
}

// 부모에서 사용
function Form() {
  const inputRef = useRef<HTMLInputElement>(null)
  return (
    <>
      <CustomInput ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>포커스</button>
    </>
  )
}

// Ref Callback — 마운트/언마운트 감지
<div ref={(node) => {
  if (node) console.log('마운트됨', node)
  return () => console.log('언마운트됨')  // React 19 cleanup
}} />
```
