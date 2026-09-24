# Open questions

Принятые допущения и решения сохраняются рядом с исходными вопросами. Если ответ принят только частично, оставшаяся часть указана как открытая. Общие правила собраны в [decisions.md](decisions.md).

## Prototype — accepted assumptions

1. **Existing product bots:** must a provided bot be dedicated to this support service, or can it already have a webhook/backend that must keep working?
   > *(Принятое допущение для Prototype: бот каждого продукта выделен для этого сервиса; сохранять другой backend или webhook не требуется.)*

2. **Media details:** how should supported forwarded content and Telegram replies be handled, and what should happen with unsupported formats such as stickers and videos? Albums and edits are already outside Prototype.
   > *(Принятое допущение для Prototype: поддерживаемое содержимое пересланных сообщений передаётся как обычное сообщение без метаданных пересылки. Если клиент или менеджер нажал «Ответить» на конкретное сообщение, содержимое ответа передаётся как новое сообщение без связи с исходным. Сохранять эту связь в Prototype, MVP и Release не планируем. Неподдерживаемый формат или превышение лимита размера создаёт топик даже при первом обращении: менеджер видит причину отказа, отправитель получает уведомление.)*

3. **Topic recovery:** what should happen if a customer topic is manually closed or deleted?
   > *(Принятое допущение для Prototype: закрытый топик открывается снова; взамен удалённого создаётся новый и обновляется связь с клиентом.)*

4. **Operational history:** how long should processed update IDs, delivery state, and other customer-related metadata be retained?
   > *(Принятое допущение для Prototype: ID обработанных обновлений хранятся 7 дней, статусы доставки — 30 дней, связь клиента с топиком — до удаления интеграции.)*

5. **Delivery guarantee:** what should happen if a Worker stops after sending to Telegram but before storing the delivery result?
   > *(Принятое допущение для Prototype: перед отправкой сохраняется статус попытки. Если после сбоя неизвестно, было ли доставлено сообщение, в топике появляется предупреждение. Автоматического повтора нет. Менеджер сам решает, отправлять ли сообщение ещё раз, понимая, что клиент мог уже получить первый вариант.)*

6. **Environments:** which bot identities, groups, D1 databases, and webhook addresses will be used for local, staging, and production verification?
   > *(Принятое допущение для Prototype: для разработки и демонстрации используются отдельные боты, группы, вебхуки и D1. Локальная разработка использует тестовый набор; production для Prototype не подготавливается. Конкретные токены и ID задаются при развёртывании.)*

## MVP

7. **Business chat scope:** does the connected bot handle all eligible personal chats or only chats explicitly selected in Telegram?
   > *(Принятое решение для MVP: владелец выбирает в Telegram, какие переписки передать в поддержку. Только сообщения из этих переписок появляются в супергруппе. Telegram передаёт подключённому боту сообщения из разрешённых переписок; наш бэкенд обрабатывает их и не хранит отдельный список разрешённых чатов.)*

8. **Business permissions:** which rights are mandatory, and what should happen when Telegram no longer permits a reply through the connection?

9. **CSV contract and delivery:** which columns are required, and who distributes the prepared manager invite links?
   > *(Уже принято для MVP: после загрузки CSV система готовит отдельную ссылку-приглашение для каждой корректной строки и показывает ошибки в остальных строках. Открыто: состав колонок и способ передачи ссылок менеджерам.)*

10. **Shell ownership:** which Shell product/remote identifier is authoritative for this backend, and who can change an integration?

## Release

11. What exactly is supplied for MTProto provisioning: a fresh login, an authorized session, or another account export?
   > *(Принятое решение для Release: Shell показывает QR-код, который владелец подтверждает в уже авторизованном Telegram-приложении. Передавать оператору готовую сессию или экспорт аккаунта не требуется. Хранение полученной сессии и запасной способ входа остаются открытым вопросом 12.)*

12. Where may the authorized user session be stored, for how long, and what fallback or additional verification is needed when QR login cannot be used?

13. Are direct MTProto invitations worth implementing after link-based onboarding works?
   > *(Принятое решение для Release: нет. Менеджеры вступают по ссылкам-приглашениям с подтверждением владельца продукта, как определено для MVP. Прямые приглашения через MTProto не входят в запланированный Release.)*
