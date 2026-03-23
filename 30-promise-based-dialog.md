# 30. Promise-based Dialog

명령형 Promise 다이얼로그 패턴

## 동작 원리

- `window.confirm()`처럼 다이얼로그를 **호출하고 결과를 `await`**하는 명령형(imperative) 패턴
- Promise의 `resolve`를 ref에 저장하고, 사용자가 버튼을 클릭하면 해당 resolve를 호출
- 상태 관리(isOpen, onConfirm, result)를 Promise 하나로 통합하여 코드 흐름이 직관적

```
기존 방식:
  useState(isOpen) → 열기 → 콜백 등록 → 닫기 → 콜백 실행
  (상태가 여러 곳에 분산)

Promise 방식:
  const result = await confirm('삭제?')
  if (result) { /* 삭제 */ }
  (한 줄에 모든 흐름이 표현)
```

## 장점

- **직관적인 코드 흐름**: `await`으로 동기 코드처럼 읽힘
- **상태 관리 단순화**: isOpen, onConfirm 등 별도 상태 불필요
- **재사용성**: 하나의 훅으로 어디서든 다이얼로그 호출 가능
- **타입 안전**: `Promise<boolean>` 또는 `Promise<T>`로 반환값 타입 지정

## 단점 / 주의사항

- 컴포넌트 트리에 Dialog를 렌더링할 위치 필요 (Portal 조합 권장)
- Promise가 resolve/reject 되지 않으면 메모리 누수 가능 → 언마운트 시 정리 필요
- 여러 다이얼로그를 동시에 열어야 하는 경우 큐 관리가 복잡해질 수 있음

## 적합한 사용 사례

- 삭제, 저장 등 **확인 다이얼로그** (`window.confirm` 대체)
- 결제, 권한 요청 등 **사용자 동의가 필요한 흐름**
- 폼 입력을 다이얼로그로 받아 결과를 반환하는 경우
- 순차적 다이얼로그 흐름 (1단계 확인 → 2단계 입력 → 3단계 완료)

## 구현 예제

### 기본: useConfirmDialog 훅

```tsx
import { useCallback, useRef, useState, ReactNode } from 'react'
import { createPortal } from 'react-dom'

type ConfirmOptions = {
  title: string
  message: string
  confirmText?: string
  cancelText?: string
}

export function useConfirmDialog() {
  const [isOpen, setIsOpen] = useState(false)
  const [options, setOptions] = useState<ConfirmOptions>({
    title: '',
    message: '',
  })
  const resolveRef = useRef<(value: boolean) => void>()

  const confirm = useCallback((opts: ConfirmOptions): Promise<boolean> => {
    setOptions(opts)
    setIsOpen(true)
    return new Promise<boolean>((resolve) => {
      resolveRef.current = resolve  // resolve를 ref에 저장
    })
  }, [])

  const handleConfirm = useCallback(() => {
    resolveRef.current?.(true)   // Promise를 true로 resolve
    setIsOpen(false)
  }, [])

  const handleCancel = useCallback(() => {
    resolveRef.current?.(false)  // Promise를 false로 resolve
    setIsOpen(false)
  }, [])

  const ConfirmDialog = useCallback(() => {
    if (!isOpen) return null

    return createPortal(
      <div className="fixed inset-0 bg-black/50 flex items-center justify-center">
        <div role="alertdialog" aria-modal="true" className="bg-white rounded-lg p-6">
          <h2 className="text-lg font-bold">{options.title}</h2>
          <p className="mt-2">{options.message}</p>
          <div className="mt-4 flex gap-2 justify-end">
            <button onClick={handleCancel}>
              {options.cancelText ?? '취소'}
            </button>
            <button onClick={handleConfirm}>
              {options.confirmText ?? '확인'}
            </button>
          </div>
        </div>
      </div>,
      document.body
    )
  }, [isOpen, options, handleConfirm, handleCancel])

  return { confirm, ConfirmDialog }
}
```

### 사용 예시

```tsx
function ItemList() {
  const { confirm, ConfirmDialog } = useConfirmDialog()

  const handleDelete = async (id: string) => {
    // window.confirm()처럼 한 줄로 확인
    const confirmed = await confirm({
      title: '항목 삭제',
      message: '정말 삭제하시겠습니까? 이 작업은 되돌릴 수 없습니다.',
      confirmText: '삭제',
      cancelText: '취소',
    })

    if (confirmed) {
      await deleteItem(id)
      toast.success('삭제되었습니다')
    }
  }

  return (
    <>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.name}
            <button onClick={() => handleDelete(item.id)}>삭제</button>
          </li>
        ))}
      </ul>
      <ConfirmDialog />
    </>
  )
}
```

### 고급: 값을 반환하는 Promise Dialog

```tsx
// 단순 boolean이 아닌, 사용자 입력값을 반환하는 패턴
function usePromptDialog() {
  const [isOpen, setIsOpen] = useState(false)
  const [label, setLabel] = useState('')
  const resolveRef = useRef<(value: string | null) => void>()

  const prompt = useCallback((promptLabel: string): Promise<string | null> => {
    setLabel(promptLabel)
    setIsOpen(true)
    return new Promise((resolve) => {
      resolveRef.current = resolve
    })
  }, [])

  const PromptDialog = () => {
    const [input, setInput] = useState('')
    if (!isOpen) return null

    return createPortal(
      <div className="modal-overlay">
        <div role="dialog" aria-modal="true">
          <label>{label}</label>
          <input value={input} onChange={e => setInput(e.target.value)} autoFocus />
          <button onClick={() => { resolveRef.current?.(null); setIsOpen(false) }}>
            취소
          </button>
          <button onClick={() => { resolveRef.current?.(input); setIsOpen(false) }}>
            확인
          </button>
        </div>
      </div>,
      document.body
    )
  }

  return { prompt, PromptDialog }
}

// 사용
const { prompt, PromptDialog } = usePromptDialog()
const name = await prompt('새 폴더 이름을 입력하세요')
if (name) {
  await createFolder(name)
}
```

### 고급: Context 기반 전역 다이얼로그

```tsx
// ConfirmProvider.tsx — 앱 전체에서 사용 가능한 전역 confirm
const ConfirmContext = createContext<{
  confirm: (opts: ConfirmOptions) => Promise<boolean>
}>(null!)

export function ConfirmProvider({ children }: { children: ReactNode }) {
  const { confirm, ConfirmDialog } = useConfirmDialog()

  return (
    <ConfirmContext.Provider value={{ confirm }}>
      {children}
      <ConfirmDialog />
    </ConfirmContext.Provider>
  )
}

export const useConfirm = () => useContext(ConfirmContext)

// 앱 루트에서 Provider 감싸기
<ConfirmProvider>
  <App />
</ConfirmProvider>

// 어디서든 사용
function AnyComponent() {
  const { confirm } = useConfirm()

  const handleAction = async () => {
    if (await confirm({ title: '확인', message: '계속하시겠습니까?' })) {
      // 진행
    }
  }
}
```

### 순차적 다이얼로그 체이닝

```tsx
async function handleCheckout() {
  // 1단계: 약관 동의
  const agreed = await confirm({
    title: '약관 동의',
    message: '이용약관에 동의하시겠습니까?'
  })
  if (!agreed) return

  // 2단계: 배송지 입력
  const address = await prompt('배송지를 입력하세요')
  if (!address) return

  // 3단계: 최종 확인
  const finalConfirm = await confirm({
    title: '결제 확인',
    message: `${address}로 배송됩니다. 결제하시겠습니까?`
  })
  if (!finalConfirm) return

  await processPayment()
}
```

## 관련 라이브러리

- [@prezly/react-promise-modal](https://github.com/prezly/react-promise-modal)
- [react-modal-promise](https://github.com/cudr/react-modal-promise)
- [use-async-modal](https://github.com/Harasz/use-async-modal)
