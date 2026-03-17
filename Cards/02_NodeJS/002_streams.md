---
created: 2026-03-17
category: Node.js
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review:
tags:
  - nodejs
  - streams
  - backpressure
  - "#flashcard/nodeJS"
related:
  - - 02_JavaScript_NodeJS/03_NodeJS_Streams_Network#1-2-Типы-потоков
---

# Что такое Streams в Node.js?

## Вопрос
Что такое Streams в Node.js? Какие типы потоков существуют?
?
## Ответ

**Streams** — механизм обработки данных по частям (чанками), не загружая весь объём в память.

**Зачем нужны:**
- Чтение файлов > 1.4 ГБ (лимит V8)
- Обработка больших объёмов данных
- Экономия памяти (~64 KB вместо ГБ)

**Типы потоков:**

1. **Readable** — источник данных:
   - `fs.createReadStream()`
   - Входящий HTTP-запрос

2. **Writable** — приёмник данных:
   - `fs.createWriteStream()`
   - HTTP-ответ

3. **Duplex** — чтение и запись:
   - TCP-сокеты

4. **Transform** — модификация данных:
   - `zlib.createGzip()` (сжатие)
   - Парсинг CSV на лету

**Backpressure:**
- Механизм регулировки скорости
- Быстрый источник приостанавливается, пока медленный приёмник не освободится

**Pipe:**
```javascript
readStream.pipe(gzipStream).pipe(writeStream);
```

---

#flashcard #nodejs #streams #backpressure
