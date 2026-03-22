# 23. Portal / createPortal

DOM 트리 탈출 렌더링

## 동작 원리

- `createPortal`을 사용하여 컴포넌트를 **DOM 트리의 다른 위치에 렌더링** (부모 DOM 밖으로 탈출)
- 일반적으로 `document.body` 바로 아래나 별도의 root 요소(`#modal-root` 등)에 렌더링
- DOM 트리에서는 분리되지만, **React 이벤트 버블링은 React 컴포넌트 트리를 따름**
- 부모의 `overflow: hidden`, `z-index` 등 CSS 제약에서 벗어날 수 있음

## 장점

- `overflow: hidden`이나 `z-index` 스택 컨텍스트 문제를 근본적으로 해결
- React 이벤트 시스템은 React 트리를 따르므로, Portal 내부의 이벤트가 부모 컴포넌트에서 정상 감지됨
- 모달, 툴팁 등을 구현할 때 DOM 구조 걱정 없이 선언적으로 작성 가능

## 단점/주의사항

- 접근성(a11y) 관리에 주의 필요 (포커스 트래핑, aria 속성 등)
- SSR 환경에서 `document` 객체가 없으므로 서버에서는 Portal을 렌더링할 수 없음
- Portal 대상 DOM 노드가 존재하지 않으면 에러 발생
- CSS 상속은 DOM 트리를 따르므로, Portal 내부 요소는 부모 컴포넌트의 CSS를 상속받지 않음

## 적합한 사용 사례

- 모달 / 다이얼로그
- 툴팁 / 팝오버
- 토스트 알림
- 드롭다운 메뉴 (부모의 overflow에 잘리지 않도록)
- 전체 화면 오버레이

## 구현 예제

### 모달 컴포넌트

```tsx
function Modal({ children, isOpen }: { children: ReactNode; isOpen: boolean }) {
  if (!isOpen) return null

  return createPortal(
    <div className="modal-overlay">
      <div className="modal-content" role="dialog" aria-modal="true">
        {children}
      </div>
    </div>,
    document.getElementById('modal-root')!  // body 바로 아래에 렌더링
  )
}
```
