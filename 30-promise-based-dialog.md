# 30. Promise-based Dialog

다이얼로그를 `await`으로 호출하고 결과를 받는 명령형 패턴.

## 동작 원리

- Promise의 `resolve`를 외부에 저장 → 사용자가 버튼 클릭 시 resolve 호출 → `await`으로 결과 수신
- 전통적 방식의 콜백 분산 문제를 선형적 async/await 흐름으로 해결

```tsx
// 전통적 방식 — 흐름이 3개 핸들러로 쪼개짐
const [isOpen, setIsOpen] = useState(false)
const handleDeleteClick = () => setIsOpen(true)
const handleConfirm = () => { setIsOpen(false); deleteItem(id) }
const handleCancel = () => setIsOpen(false)
```

```tsx
// Promise 방식 — 선형적 흐름
const handleDeleteClick = async () => {
  if (await confirmation.invoke()) {
    deleteItem(id)
  }
}
```

## 장점

- **직관적인 코드 흐름**: `await`으로 동기 코드처럼 읽힘
- **상태 관리 단순화**: isOpen, onConfirm 등 별도 상태 불필요
- **재사용성**: 하나의 훅으로 어디서든 다이얼로그 호출 가능
- **타입 안전**: `Promise<boolean>` 또는 `Promise<T>`로 반환값 타입 지정

## 단점 / 주의사항

- **메모리 누수**: Promise가 resolve되지 않으면 GC 안 됨 → 언마운트 시 정리 필수
- **stale closure**: `await` 이후 코드는 호출 시점의 closure를 캡처 → 최신 state/props가 아닐 수 있음
- **동시 호출**: 여러 다이얼로그를 동시에 열면 이전 Promise가 미해결 상태로 남을 수 있음

## 적합한 사용 사례

- 삭제, 저장 등 **확인 다이얼로그** (`window.confirm` 대체)
- 결제, 권한 요청 등 **사용자 동의가 필요한 흐름**
- 폼 입력을 다이얼로그로 받아 결과를 반환하는 경우
- 순차적 다이얼로그 흐름 (1단계 확인 → 2단계 입력 → 3단계 완료)

## 구현 방식 3가지

### 1. 별도 React root (안티패턴)

```tsx
// ❌ 별도 root에 모달을 마운트하는 방식
function confirm(message: string): Promise<boolean> {
  const container = document.createElement('div')
  const root = createRoot(container)
  document.body.appendChild(container)

  return new Promise((resolve) => {
    root.render(
      <Dialog
        message={message}
        onConfirm={() => { resolve(true); root.unmount(); container.remove() }}
        onCancel={() => { resolve(false); root.unmount(); container.remove() }}
      />
    )
  })
}
```

API는 깔끔하지만 **3가지 치명적 문제**:

| 문제 | 설명 |
|------|------|
| **Context 단절** | 메인 React 트리 밖이라 ThemeContext, i18n, 상태관리 등 모든 Context 접근 불가 |
| **반응성 없음** | 렌더링 후 부모 state/props가 변해도 모달에 반영 안 됨 (stale props) |
| **생명주기 단절** | 호출 컴포넌트가 언마운트돼도 모달이 남아있음. HMR도 깨짐 |

### 2. Hook이 element를 반환 (권장 — 컴포넌트별 모달)

`@prezly/react-promise-modal` v2가 1단계의 문제를 해결하기 위해 채택한 방식.
핵심: **모달 정의는 선언적**(React 트리 안), **호출은 명령형**(async/await).

```tsx
// ✅ 훅이 ReactElement를 반환 → JSX에 렌더링 → React 트리에 포함
const confirmation = usePromiseModal(({ show, onSubmit, onDismiss }) => (
  <ConfirmDialog
    open={show}
    onConfirm={() => onSubmit(true)}
    onCancel={onDismiss}
  />
))

async function handleDelete() {
  if (await confirmation.invoke()) {
    deleteItem(id)
  }
}

return (
  <>
    <button onClick={handleDelete}>삭제</button>
    {confirmation.modal}  {/* React 트리 안에 렌더링 */}
  </>
)
```

**왜 element 반환이 안티패턴이 아닌가:**
- 모달이 React 트리 안에 존재 → Context 접근, 반응성, 생명주기 모두 정상 동작
- 별도 root 방식의 3가지 문제를 모두 해결
- `@prezly/react-promise-modal`이 v1(별도 root)에서 v2(element 반환)로 전환한 이유가 정확히 이것

### 3. Context/Provider 방식 (권장 — 전역 confirm/alert)

모달을 Provider에서 한 번만 렌더링. 소비자는 `confirm()`만 호출.

```tsx
const ConfirmContext = createContext<(msg: string) => Promise<boolean>>(null!)

function ConfirmProvider({ children }: { children: ReactNode }) {
  const dialogRef = useRef<HTMLDialogElement>(null)
  const resolveRef = useRef<(v: boolean) => void>()
  const [message, setMessage] = useState('')

  const confirm = useCallback((msg: string): Promise<boolean> => {
    setMessage(msg)
    dialogRef.current?.showModal()
    return new Promise((resolve) => { resolveRef.current = resolve })
  }, [])

  // 언마운트 시 미해결 Promise 정리 → 메모리 누수 방지
  useEffect(() => {
    return () => { resolveRef.current?.(false) }
  }, [])

  return (
    <ConfirmContext.Provider value={confirm}>
      {children}
      <dialog
        ref={dialogRef}
        onCancel={() => { resolveRef.current?.(false); dialogRef.current?.close() }}
      >
        <p>{message}</p>
        <button onClick={() => { resolveRef.current?.(false); dialogRef.current?.close() }}>
          취소
        </button>
        <button onClick={() => { resolveRef.current?.(true); dialogRef.current?.close() }}>
          확인
        </button>
      </dialog>
    </ConfirmContext.Provider>
  )
}

const useConfirm = () => useContext(ConfirmContext)
```

```tsx
// 사용 — element 렌더링 불필요
function AnyComponent() {
  const confirm = useConfirm()

  const handleDelete = async (id: string) => {
    if (await confirm('정말 삭제하시겠습니까?')) {
      await deleteItem(id)
    }
  }
}
```

**주의:** `showModal` 호출 시점의 props 스냅샷이 고정되어 **stale props** 문제 발생 가능. 단순 confirm에는 문제없지만, 복잡한 폼 모달에서는 주의 필요.

## 방식 비교

| | 별도 root | Hook → element | Context/Provider |
|---|---|---|---|
| **Context 접근** | ❌ 단절 | ✅ 정상 | ✅ 정상 |
| **모달 UI 반응성** | ❌ 없음 | ✅ 부모와 동기화 | △ stale props 주의 |
| **생명주기** | ❌ 단절 | ✅ 부모와 연동 | ✅ Provider와 연동 |
| **API 편의성** | `await confirm()` | `await x.invoke()` + `{x.modal}` | `await confirm()` |
| **렌더링 위치** | 소비자 신경 안 씀 | 소비자가 `{x.modal}` 렌더링 | Provider가 렌더링 |
| **적합한 사용** | ❌ 사용 금지 | 컴포넌트별 모달 | 전역 confirm/alert |

> **참고:** "모달 UI 반응성"은 모달 컴포넌트가 부모 state 변경을 반영하는지를 의미. `await` 이후 코드의 stale closure 문제는 모든 방식에 공통.

## 핵심 구현 디테일

### Deferred 패턴 (Promise의 resolve를 외부화)

위 예제들에서 `resolveRef`로 수동 관리하는 대신, 클래스로 캡슐화하면 이중 resolve 방지와 상태 추적이 가능:

```tsx
class Deferred<T> {
  promise: Promise<T>
  resolve!: (value: T) => void
  reject!: (reason?: unknown) => void
  private settled = false

  constructor() {
    this.promise = new Promise((resolve, reject) => {
      this.resolve = (value) => {
        if (!this.settled) { this.settled = true; resolve(value) }
      }
      this.reject = (reason) => {
        if (!this.settled) { this.settled = true; reject(reason) }
      }
    })
  }

  get isSettled() { return this.settled }
}
```

`isSettled` 체크로 **이중 resolve 방지**. 언마운트 시 미해결 Promise를 `undefined`로 resolve하여 **메모리 누수 방지**.

### useLatest (stale closure 방지)

```tsx
function useLatest<T>(value: T) {
  const ref = useRef(value)
  ref.current = value  // 렌더 중 할당 — 실무에서 널리 사용되는 패턴
  return ref
}
```

비동기 콜백 안에서 항상 최신 props/state에 접근하기 위한 ref 패턴.
> strict concurrent mode에서는 `useInsertionEffect` 안에서 할당하는 것이 기술적으로 더 안전하지만, 실무에서는 렌더 중 할당이 일반적.

## 관련 라이브러리

| 라이브러리 | 주간 다운로드 | 방식 | 특징 |
|-----------|------------|------|------|
| [@ebay/nice-modal-react](https://github.com/eBay/nice-modal-react) | ~208K | Context + Provider | `NiceModal.show()`이 Promise 반환. `modal.resolve()`/`modal.reject()`로 해결. MUI/Ant Design 바인딩 내장 |
| [react-modal-promise](https://github.com/cudr/react-modal-promise) | ~51K | Container + `create()` | `<Container />` 한 번 마운트, `create(Component)`로 래핑 → 함수 호출로 모달 열기 |
| [react-confirm](https://github.com/haradakunihiko/react-confirm) | ~53K | `createConfirmation()` HOC | 별도 DOM 노드에 마운트 (v1 방식). `confirmable()` HOC + `createConfirmation()`. Context-aware 변형도 지원 |
| [@prezly/react-promise-modal](https://github.com/prezly/react-promise-modal) | ~2K | Hook → element | `usePromiseModal()` 훅. 트랜지션 지원 (`show` boolean으로 enter/exit 애니메이션) |

## Sources

- [Inventing React Promise Modals — The Ugly and The Nice](https://voskoboinyk.com/posts/2024-12-27-inventing-react-promise-modals-part-1)
- [Fixing React Promise Modals — The Nice and The Proper](https://voskoboinyk.com/posts/2024-12-31-fixing-react-promise-modals)
- [Promise based modals — Marius Bajorunas](https://www.bajorunas.tech/blog/promise-based-modals)
- [@prezly/react-promise-modal GitHub](https://github.com/prezly/react-promise-modal)
- [@ebay/nice-modal-react GitHub](https://github.com/eBay/nice-modal-react)
