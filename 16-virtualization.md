# 16. Virtualization

가상화 (윈도잉) — 수천~수만 개 아이템 중 뷰포트에 보이는 것만 DOM에 렌더링하는 패턴

## 동작 원리

가상화는 전체 리스트의 아이템을 모두 DOM에 렌더링하지 않고, **현재 스크롤 위치에서 보이는 영역(viewport)에 해당하는 아이템만** 실제 DOM 노드로 렌더링한다. 스크롤 이벤트를 감지하여 보이는 범위가 변경될 때마다 렌더링되는 아이템을 동적으로 교체한다.

내부적으로 전체 리스트 높이에 해당하는 빈 컨테이너를 만들고, 각 아이템은 `position: absolute`와 `top` 값을 사용하여 정확한 위치에 배치된다. 이를 통해 스크롤바는 전체 리스트가 존재하는 것처럼 동작하지만, 실제 DOM 노드는 화면에 보이는 개수 + 약간의 오버스캔(overscan) 만큼만 존재한다.

## 장점

- **60FPS 스크롤**: 수만 개 아이템에서도 부드러운 스크롤 성능 유지
- **메모리 사용 대폭 감소**: 실제 DOM 노드 수가 화면에 보이는 만큼으로 제한
- **초기 렌더링 속도**: 전체 리스트를 렌더링하지 않으므로 초기 마운트가 빠름
- **대규모 데이터 처리**: 수십만 행의 테이블도 문제없이 표시 가능

## 단점/주의사항

- **구현 복잡도**: 가변 높이 아이템, 동적 콘텐츠, 양방향 스크롤 등 처리가 복잡
- **접근성(a11y)**: 스크린 리더가 가상화된 콘텐츠를 올바르게 읽지 못할 수 있음
- **브라우저 검색(Ctrl+F)**: DOM에 없는 아이템은 브라우저 내장 검색에 잡히지 않음
- **스크롤 점프**: 가변 높이 아이템에서 빠른 스크롤 시 점프 현상이 발생할 수 있음
- **소규모 리스트에 불필요**: 수백 개 이하의 아이템에는 오버엔지니어링

## 적합한 사용 사례

- 수천 개 이상의 아이템을 표시하는 리스트/테이블
- 무한 스크롤 피드 (소셜 미디어, 채팅)
- 대규모 데이터 그리드/스프레드시트
- 로그 뷰어, 코드 에디터 등 대량 텍스트 표시

## 구현 예제

```tsx
import { useVirtualizer } from '@tanstack/react-virtual'

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null)

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,  // 각 아이템 예상 높이
  })

  return (
    <div ref={parentRef} style={{ height: 400, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: virtualRow.start,
              height: virtualRow.size,
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  )
}
```

### 주요 라이브러리

| 라이브러리 | 특징 |
|-----------|------|
| **TanStack Virtual** | 경량, 헤드리스, 현재 권장 |
| **react-window** | 안정적, react-virtualized의 경량 후속작 |
| **react-virtuoso** | 가변 높이 자동 측정, 그룹핑 지원 |
