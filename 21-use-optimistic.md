# 21. useOptimistic

낙관적 업데이트 (React 19)

## 동작 원리

- 서버 응답을 기다리지 않고 **즉시 UI를 업데이트**하여 사용자에게 빠른 피드백 제공
- React 19에서 도입된 `useOptimistic` 훅을 사용
- 실제 서버 요청이 진행되는 동안 낙관적(optimistic) 상태를 표시
- 서버 요청이 실패하면 자동으로 원래 상태로 롤백
- 서버 응답이 성공하면 실제 데이터로 교체

## 장점

- 사용자 체감 속도가 크게 향상 (네트워크 지연이 느껴지지 않음)
- 서버 응답 전에도 UI가 즉각 반응하여 앱이 빠르게 느껴짐
- 실패 시 자동 롤백으로 데이터 일관성 보장
- form action과 자연스럽게 결합 가능

## 단점/주의사항

- React 19 이상에서만 사용 가능
- 낙관적 상태와 실제 상태 사이의 불일치를 고려해야 함
- 실패 시 사용자에게 적절한 피드백(에러 메시지 등)을 별도로 제공해야 함
- 복잡한 데이터 관계(연쇄 업데이트)에서는 롤백 로직이 까다로울 수 있음

## 적합한 사용 사례

- 좋아요/북마크 토글 (즉시 반영, 실패 시 롤백)
- Todo 리스트 항목 추가/삭제
- 댓글 작성 (작성 즉시 표시)
- 장바구니 아이템 추가
- SNS 피드에서의 인터랙션 (리트윗, 팔로우 등)

## 구현 예제

### 기본 사용법 — Todo 리스트

```tsx
function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: Todo) => [...state, { ...newTodo, pending: true }]
  )

  async function addTodo(formData: FormData) {
    const title = formData.get('title') as string
    addOptimisticTodo({ id: crypto.randomUUID(), title, pending: true })
    await saveTodoToServer(title)  // 실패 시 자동으로 원래 상태로 롤백
  }

  return (
    <form action={addTodo}>
      <input name="title" />
      <button type="submit">추가</button>
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id} style={{ opacity: todo.pending ? 0.5 : 1 }}>
            {todo.title}
          </li>
        ))}
      </ul>
    </form>
  )
}
```
