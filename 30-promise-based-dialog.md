# 30. Promise-based Dialog

`window.confirm()`처럼 다이얼로그를 호출하고 `await`으로 결과를 받는 명령형 패턴.

## 핵심

Promise의 `resolve`를 `useRef`에 저장 → 사용자가 버튼 클릭 시 resolve 호출 → `await`으로 결과 수신.
네이티브 `<dialog>`의 `showModal()`/`close()`를 사용하면 `isOpen` state 없이 ref만으로 동작.

```tsx
const confirmed = await confirm('정말 삭제하시겠습니까?')
if (confirmed) await deleteItem(id)
```

## 구현

```tsx
import { useCallback, useRef } from 'react'

export function useConfirmDialog() {
  const dialogRef = useRef<HTMLDialogElement>(null)
  const resolveRef = useRef<(value: boolean) => void>()
  const messageRef = useRef('')

  const confirm = useCallback((message: string): Promise<boolean> => {
    messageRef.current = message
    dialogRef.current?.showModal()
    return new Promise((resolve) => {
      resolveRef.current = resolve
    })
  }, [])

  const handleConfirm = useCallback(() => {
    resolveRef.current?.(true)
    dialogRef.current?.close()
  }, [])

  const handleCancel = useCallback(() => {
    resolveRef.current?.(false)
    dialogRef.current?.close()
  }, [])

  const Dialog = useCallback(() => (
    <dialog ref={dialogRef} onCancel={handleCancel}>
      <p>{messageRef.current}</p>
      <button onClick={handleCancel}>취소</button>
      <button onClick={handleConfirm}>확인</button>
    </dialog>
  ), [handleConfirm, handleCancel])

  return { confirm, Dialog }
}
```

## 사용

```tsx
function ItemList() {
  const { confirm, Dialog } = useConfirmDialog()

  const handleDelete = async (id: string) => {
    if (await confirm('정말 삭제하시겠습니까?')) {
      await deleteItem(id)
    }
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
