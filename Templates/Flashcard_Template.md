---
created: <% tp.date.now("YYYY-MM-DD") %>
category: <% tp.file.cursor(1) %>
type: flashcard
difficulty: <% tp.file.cursor(2) %>
frequency: <% tp.file.cursor(3) %>
status: new
next_review: 
tags: [flashcard/<% tp.file.cursor(1) %>]
related: [[ ]]
---

# <% tp.file.title %>

## Вопрос
<% tp.file.cursor(4) %>
?
## Ответ

<% tp.file.cursor(5) %>

---

#flashcard #<% tp.file.cursor(6) %>
