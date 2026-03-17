---
created: 2026-03-17
category: Infrastructure
type: flashcard
difficulty: intermediate
frequency: high
status: new
next_review: 
tags: [git, merge, rebase, flashcard/infrastructure]
related: [[05_Infrastructure/01_Infrastructure_Networks_Docker#2-1-Merge-vs-Rebase]]
---

# git merge vs git rebase

## Вопрос
В чём разница между `git merge` и `git rebase`?
?
## Ответ

**git merge:**
- Создаёт новый коммит слияния
- Сохраняет исходную историю
- История нелинейная (с ветвлениями)

```
Before merge:
      A---B---C  (feature)
     /
D---E---F---G  (main)

After merge:
      A---B---C
     /         \
D---E---F---G---M  (main)
                ↑
            Merge commit
```

**git rebase:**
- Переписывает историю
- Переносит коммиты поверх целевой ветки
- История линейная

```
Before rebase:
      A---B---C  (feature)
     /
D---E---F---G  (main)

After rebase:
              A'--B'--C'  (feature)
             /
D---E---F---G  (main)
```

**Когда что использовать:**
- **merge** — публичные ветки, сохранение истории
- **rebase** — локальные ветки перед merge

**⚠️ Никогда не делайте rebase:**
- Если ветка уже запушена
- Если над веткой работают другие

---

#flashcard #git #merge #rebase #version-control
