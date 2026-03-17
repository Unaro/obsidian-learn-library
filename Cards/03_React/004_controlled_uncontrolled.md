---
created: 2026-03-17
category: React
type: flashcard
difficulty: beginner
frequency: medium
status: new
next_review:
tags:
  - react
  - controlled
  - uncontrolled
  - forms
  - "#flashcard/reactJS"
related:
  - - 03_React_Frontend/02_React_Internals_Optimization#3-Компоненты-высшего-порядка-HOC
---

# Controlled vs Uncontrolled компоненты

## Вопрос
В чём разница между Controlled и Uncontrolled компонентами в React?
?
## Ответ

**Controlled (Контролируемые):**
- Значение хранится в **state React**
- `value={state} onChange={setState}`
- React — единственный источник истины
- Легко валидировать, форматировать

```javascript
function ControlledForm() {
    const [value, setValue] = useState('');
    
    return (
        <input 
            value={value} 
            onChange={(e) => setValue(e.target.value)} 
        />
    );
}
```

**Uncontrolled (Неконтролируемые):**
- Значение хранится в **DOM**
- Доступ через `useRef`
- Похоже на обычный HTML
- Меньше кода, но сложнее управлять

```javascript
function UncontrolledForm() {
    const inputRef = useRef();
    
    const handleSubmit = () => {
        console.log(inputRef.current.value);
    };
    
    return <input ref={inputRef} />;
}
```

**Когда что использовать:**
- **Controlled** — валидация, форматирование, зависимые поля
- **Uncontrolled** — простые формы, файловые инпуты, интеграция с не-React кодом

---

#flashcard #react #controlled #uncontrolled #forms
