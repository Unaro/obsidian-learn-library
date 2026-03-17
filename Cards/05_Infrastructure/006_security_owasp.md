---
created: 2026-03-17
category: Infrastructure
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review: 
tags: [security, xss, sqli, cors, flashcard/infrastructure]
related: [[05_Infrastructure/03_TypeScript_Security_Testing#2-Web-Безопасность-OWASP]]
---

# OWASP Top 10 уязвимости

## Вопрос
Какие основные уязвимости безопасности по OWASP? Как защититься?
?
## Ответ

**XSS (Cross-Site Scripting):**
- Внедрение вредоносного JS-кода
- **Защита:** экранирование, CSP, безопасный парсинг

**SQL Injection:**
- Внедрение вредоносного SQL-кода
- **Защита:** параметризованные запросы, ORM

**CORS (Cross-Origin Resource Sharing):**
- Браузер блокирует запросы к другим доменам
- **Защита:** заголовок `Access-Control-Allow-Origin`

**Примеры атак:**
```javascript
// XSS
<input value="<script>steal()</script>">

// SQL Injection
"SELECT * FROM users WHERE name = '" + userInput + "'"
// userInput = "admin' OR '1'='1"
```

**Защита:**
```python
# ✅ Параметризованный запрос
cursor.execute(
    "SELECT * FROM users WHERE name = %s",
    (user_input,)
)
```

**OWASP Top 10:**
1. Broken Access Control
2. Cryptographic Failures
3. Injection
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable Components
7. Authentication Failures
8. Software and Data Integrity
9. Security Logging
10. SSRF

---

#flashcard #security #owasp #xss #sqli #cors
