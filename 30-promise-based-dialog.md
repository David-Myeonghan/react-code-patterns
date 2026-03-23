# 30. Promise-based Dialog

`window.confirm()`처럼 다이얼로그를 호출하고 `await`으로 결과를 받는 명령형 패턴.

## 핵심

Promise의 `resolve`를 `useRef`에 저장 → 사용자가 버튼 클릭 시 resolve 호출 → `await`으로 결과 수신

```tsx
// 이 패턴의 본질
const confirmed = await confirm({ title: '삭제', message: '정말요?' })
if (confirmed) await deleteItem(id)
```

## 구현

```tsx
import { useCallback, useRef, useState } from 'react'

type ConfirmOptions = { title: string; message: string }

export function useConfirmDialog() {
  const [isOpen, setIsOpen] = useState(false)
  const [options, setOptions] = useState<ConfirmOptions>({ title: '', message: '' })
  const resolveRef = useRef<(value: boolean) => void>()

  const confirm = useCallback((opts: ConfirmOptions): Promise<boolean> => {
    setOptions(opts)
    setIsOpen(true)
    return new Promise((resolve) => {
      resolveRef.current = resolve
    })
  }, [])

  const handleConfirm = useCallback(() => {
    resolveRef.current?.(true)
    setIsOpen(false)
  }, [])

  const handleCancel = useCallback(() => {
    resolveRef.current?.(false)
    setIsOpen(false)
  }, [])

  const Dialog = useCallback(() => {
    if (!isOpen) return null
    return (
      <div className="overlay">
        <div role="alertdialog">
          <h2>{options.title}</h2>
          <p>{options.message}</p>
          <button onClick={handleCancel}>취소</button>
          <button onClick={handleConfirm}>확인</button>
        </div>
      </div>
    )
  }, [isOpen, options, handleConfirm, handleCancel])

  return { confirm, Dialog }
}
```

## 사용

```tsx
function ItemList() {
  const { confirm, Dialog } = useConfirmDialog()

  const handleDelete = async (id: string) => {
    const confirmed = await confirm({
      title: '항목 삭제',
      message: '정말 삭제하시겠습니까?',
    })
    if (confirmed) await deleteItem(id)
  }

  return (
    <>
      {items.map(item => (
        <button key={item.id} onClick={() => handleDelete(item.id)}>삭제</button>
      ))}
      <Dialog />
    </>
  )
}
```
