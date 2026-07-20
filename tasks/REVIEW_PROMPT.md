# Prompt for Codex review

Use this after each milestone implementation and before committing.

```text
Проведи security-oriented review всех незакоммиченных изменений.
Сначала прочитай AGENTS.md, docs/CODE_REVIEW.md, документы текущего
milestone и соответствующие ADR. На первом проходе не изменяй код.

Ищи прежде всего:

1. panic, unwrap, expect или indexing, достижимые через входные данные;
2. allocation до проверки длины, integer overflow и неограниченные буферы;
3. неоднозначное или non-canonical кодирование;
4. частичную доставку данных после ошибки аутентификации/декодирования;
5. replay, nonce reuse, неправильное разделение направлений или epoch;
6. overlap/duplicate ошибки и повторную доставку reassembled packet;
7. unbounded queue, deadlock, cancellation leak и зависание shutdown;
8. утечку ключей, токенов, заголовков или packet payload в логах;
9. смешивание protocol core с TUN, route, HTTP/3 или runtime;
10. изменение маршрутов без явного разрешения и ненадёжный rollback;
11. tunnel-specific поведение для unauthenticated decoy requests;
12. тесты, которые проходят без проверки заявленного свойства;
13. расхождение кода со SPEC, ADR и текущим task-файлом;
14. случайную реализацию следующего milestone.

Выдай findings по уровням Critical, High, Medium, Low. Для каждого
укажи файл, строки, сценарий отказа/атаки, почему существующий тест
не ловит проблему и минимальное исправление. Затем перечисли
проверенные области, в которых findings нет. Не преувеличивай и не
выдумывай проблемы без воспроизводимого объяснения.
```
