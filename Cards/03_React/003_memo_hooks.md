---
created: 2026-03-17
category: React
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review:
tags:
  - react
  - memo
  - useMemo
  - useCallback
  - "#flashcard/reactJS"
related:
  - - 03_React_Frontend/02_React_Internals_Optimization#5-2-React-memo-для-оптимизации-рендеров
---

# useMemo vs useCallback

## Вопрос
В чём разница между `useMemo` и `useCallback` в React?
?
## Ответ

**`useMemo` — кэширует **значение**:**
```javascript
const expensiveValue = useMemo(() => {
    return data.reduce((sum, item) => sum + item.value, 0);
}, [data]);

// Пересчитывается только при изменении data
```

**`useCallback` — кэширует **функцию**:**
```javascript
const handleClick = useCallback(() => {
    console.log('Clicked');
}, []);

// Возвращает ту же функцию при ререндерах
```

**Когда что использовать:**
- `useMemo` — дорогие вычисления значений
- `useCallback` — передача функций в дочерние компоненты (с React.memo)

**Пример вместе:**
```javascript
function TodoList({ todos, onAddTodo }) {
    const filteredTodos = useMemo(() => 
        todos.filter(todo => todo.completed), 
        [todos]
    );
    
    const handleAdd = useCallback((text) => 
        onAddTodo(text), 
        [onAddTodo]
    );
    
    return (
        <TodoForm onAdd={handleAdd} />
        <TodoItems items={filteredTodos} />
    );
}
```

**Важно:** Не использовать без необходимости — добавляет overhead!

---

#flashcard #react #memo #usememo #usecallback
