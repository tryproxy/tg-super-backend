# Open questions

Accepted Prototype assumptions are kept with their original questions below. MVP and later questions remain unresolved. Cross-task rules are in [decisions.md](decisions.md).

## Prototype — accepted assumptions

1. **Existing product bots:** must a provided bot be dedicated to this support service, or can it already have a webhook/backend that must keep working?

   (Принятое допущение для Prototype: бот каждого продукта выделен для этого сервиса; сохранять другой backend или webhook не требуется.)
2. **Media details:** what should the service report for forwarded messages, stickers, videos, and reply context that the Prototype does not support? Albums and edits are already outside Prototype.

   (Принятое допущение для Prototype: поддерживаемое содержимое пересланных сообщений передаётся как обычное сообщение, без метаданных пересылки и связи reply. Неподдерживаемый формат создаёт топик даже при первом обращении: менеджер видит причину отказа, отправитель получает уведомление.)
3. **Topic recovery:** what should happen if a customer topic is manually closed or deleted?

   (Принятое допущение для Prototype: закрытый топик открывается снова; взамен удалённого создаётся новый и обновляется связь с клиентом.)
4. **Operational history:** how long should processed update IDs, delivery state, and other customer-related metadata be retained?

   (Принятое допущение для Prototype: ID обработанных обновлений хранятся 7 дней, статусы доставки — 30 дней, связь клиента с топиком — до удаления интеграции.)
5. **Delivery guarantee:** what should happen if a Worker stops after sending to Telegram but before storing the delivery result?

   (Принятое допущение для Prototype: перед отправкой сохраняется статус попытки. Если результат после сбоя неизвестен, автоматического повтора нет: оператор видит этот статус и сверяет доставку вручную.)
6. **Environments:** which bot identities, groups, D1 databases, and webhook addresses will be used for local, staging, and production verification?

   (Принятое допущение для Prototype: для разработки и демонстрации используются отдельные боты, группы, вебхуки и D1. Локальная разработка использует тестовый набор; production для Prototype не подготавливается. Конкретные токены и ID задаются при развёртывании.)

## Before MVP

7. **Business chat scope:** does the connected bot handle all eligible personal chats or only chats explicitly selected in Telegram?
8. **Business permissions:** which rights are mandatory, and what should happen when Telegram no longer permits a reply through the connection?
9. **CSV contract:** which columns are required, and who distributes manager invite links?
10. **Shell ownership:** which Shell product/remote identifier is authoritative for this backend, and who can change an integration?

## After MVP

11. What exactly is supplied for MTProto provisioning: a fresh login, an authorized session, or another account export?
12. Who handles Telegram login codes and two-factor authentication, and where may encrypted user sessions live?
13. Are direct MTProto invitations worth implementing after link-based onboarding works?
