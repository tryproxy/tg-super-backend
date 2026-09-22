# Спецификация продукта: Telegram support

Одной фразой: клиент пишет боту продукта, команда видит переписку в отдельном топике Telegram-супергруппы и отвечает из этого топика.

## Для кого и зачем

- Клиенту нужен привычный Telegram-чат поддержки конкретного продукта.
- Менеджеру нужна общая очередь обращений прямо в Telegram.
- Оператору интеграции нужно связать продукт, его бота и рабочую группу без изменения Shell.

Продукт здесь — отдельная единица поддержки. В будущем она может соответствовать remote в Runtime MF Shell.

## Роли и идентификаторы

| Роль | Значение |
| --- | --- |
| Продукт | Отдельное направление поддержки; позже может соответствовать remote в Shell. |
| Клиентский канал | Telegram-бот продукта в Prototype; Business-аккаунт добавляется в MVP. |
| Рабочая группа | Одна активная приватная супергруппа с темами на продукт. |
| Владелец группы | Пользовательский Telegram-аккаунт, управляющий группой; Prototype не входит в него через backend. |
| Служебный бот | Один бот платформы для всех рабочих групп и пересылки сообщений. |
| Менеджер | Участник рабочей группы, отвечающий клиенту. |
| Диалог | Пара клиентского чата и канала конкретного продукта. |
| Клиентский топик | Активный топик, связанный с диалогом; удалённый топик заменяется. |

Владелец группы, клиентский канал и менеджеры могут принадлежать разным аккаунтам.

## Как работает Prototype

Для каждого продукта оператор вручную создаёт или выбирает одну приватную супергруппу с включёнными темами. В группу добавляется общий служебный бот платформы с правами, необходимыми для чтения сообщений и управления темами. Менеджеры вступают в группу вручную. Клиенту предоставляется отдельный бот продукта.

Клиент пишет боту продукта. Первое обращение создаёт в группе продукта постоянный топик с именем клиента. Следующие сообщения того же клиента в тот же бот попадают в этот топик. Если тот же клиент пишет боту другого продукта, это отдельный топик в другой группе.

Менеджер отправляет сообщение в топике; клиент получает его от бота продукта. Обычное сообщение в клиентском топике считается ответом клиенту сразу после отправки в Telegram. Внутреннее обсуждение ведётся вне клиентского топика.

Prototype поддерживает текст, фотографии размером до 10 МБ, а также документы и голосовые сообщения размером до 20 МБ. Если сообщение нельзя передать, ошибка должна быть видна менеджеру в топике. Проверяемые правила Prototype приведены ниже в этом плане.

## Основные правила

- Один продукт имеет одну активную рабочую форум-группу.
- У продукта в Prototype один клиентский канал — его обычный Telegram-бот.
- Один общий служебный бот работает во всех рабочих группах.
- Один клиентский диалог имеет один постоянный топик.
- Ключ диалога включает продукт, клиентский канал и внешний Telegram chat ID.
- Входящие обновления Telegram могут повторяться; повтор не должен создавать новый топик или дублировать доставку.
- Владелец группы, клиентский бот и менеджеры — разные роли; совпадение аккаунтов не требуется.
- Доступ к настройке интеграции пока предоставляет защищённый admin API. У Shell интерфейса в Prototype нет.

## Проверяемые правила Prototype

Эти критерии служат источником для задач Prototype. Принятые общие решения остаются в [decisions.md](../decisions.md), а история ответов и нерешённые вопросы — в [open-questions.md](../open-questions.md).

### Настройка и доступ

- P-01: An operator can register more than one product through a protected admin API.
- P-02: For each product, the operator can register one dedicated product-owned bot token and one existing private forum supergroup. Prototype setup does not preserve another backend or webhook for that bot.
- P-03: Activation verifies that the customer bot is usable and the common service bot can read group messages and create forum topics. Invalid setup produces an actionable status.
- P-04: The same support group cannot be active for two products.
- P-05: Managers are added to the Telegram group manually. Their group membership, rather than an imported email address, establishes access in the Prototype.
- P-06: Secrets are not returned by read endpoints or written to logs.

### Сообщения клиента

- P-07: A private message to a configured product bot resolves to that product and its customer channel.
- P-08: The first message from a customer creates one topic in the product's forum group and persists the conversation-to-topic mapping, including when its format is unsupported.
- P-09: Later messages from the same customer to the same channel reuse the active mapped topic. A closed topic is reopened; a deleted topic is replaced and the mapping updated. Another product or channel gets a separate conversation. The topic title uses the customer's Telegram display name, or `Client <chat ID>` when no name is available. If the display name later changes, the topic title is updated without creating a new conversation.
- P-10: Text, photos up to 10 MB, and documents and voice messages up to 20 MB are relayed with their captions where applicable. Supported content from forwarded messages is treated as ordinary content; forwarding metadata and reply linkage are not preserved. Media is downloaded and uploaded because the product support bot and service bot cannot reuse each other's `file_id`.
- P-11: Unsupported or oversized content produces a topic notice explaining what was not relayed, including for a first message. A customer sender is notified through the product bot; a manager sender sees the notice in the topic. The system does not claim successful delivery.
- P-12: Repeated webhook updates are deduplicated within the 7-day processed-ID retention window and do not create additional active topics or duplicate messages. An outbound attempt with an unknown result is not resent automatically.

### Ответы менеджеров

- P-13: An ordinary message sent in a mapped customer topic by a group participant is treated as an immediate reply to the customer.
- P-14: The reply is delivered through the same product bot the customer contacted. The customer sees the bot identity.
- P-15: Messages from another group, an unmapped topic, or the service bot's own output are not sent to a customer.
- P-16: A confirmed delivery failure produces a clear notice in the same topic without creating a reply loop. If an interrupted attempt has an unknown result, the admin API and mapped topic show that the message may already have been delivered. The backend does not retry it automatically; a manager may explicitly send a new message as a separate attempt.
- P-17: Internal discussion takes place outside mapped customer topics.

### Операционные данные

- P-18: Persist an in-progress delivery attempt before calling Telegram. Expose attempts with unknown outcomes through the protected admin API and warn the manager in the mapped topic without changing the attempt to success or failure.
- P-19: Retain processed update IDs for 7 days and delivery statuses for 30 days. Retain conversation-to-topic mappings until the integration is deleted.

### Проверка завершения Prototype

- Configure two products and two groups manually.
- Send first and subsequent messages to both bots; verify correct topics, media transfer, and isolation.
- Send manager replies from the mapped topics; verify each customer receives the reply from the correct bot.
- Repeat inbound updates within the retention window and simulate confirmed failures and interrupted outbound attempts; verify deduplication, visible failure, and an operator-visible unknown state without automatic resend.
- Send an unsupported first message and verify that it creates a topic, explains the rejection there, and notifies the sender.
- Close and delete mapped topics; verify reopening and replacement with an updated mapping.
- Verify an invalid bot token or missing group permissions prevents activation.
- Verify the flow runs through a webhook-based Cloudflare Worker and D1 without a continuously running backend process. Use separate test and demo Telegram resources; local development uses test resources.

## Не входит в Prototype

- Telegram Business как клиентский канал;
- настройка через Runtime MF Shell;
- CSV-импорт менеджеров, автоматические приглашения и выдача ролей;
- автоматическое создание группы от имени пользовательского Telegram-аккаунта;
- отдельные тикеты для одного клиента, внутренние заметки внутри клиентского топика, альбомы и синхронизация правок сообщений.

## MVP

MVP добавляет Telegram Business как второй тип клиентского канала, настройку и статусы интеграции в Runtime MF Shell, а также CSV onboarding менеджеров с приглашениями. Продукт может иметь обычного бота и Business-канал одновременно; для каждого канала и клиента сохраняется отдельный диалог и топик. Business-аккаунты подключаются к общему боту-коннектору платформы. Общий принцип «клиентский канал → диалог → топик → ответ» сохраняется. Точные условия подключения Business и контракт CSV фиксируются до реализации соответствующих задач.

Отдельные требования MVP: одноразовая привязка Business-соединения к продукту с проверкой прав; подключение к другому Business-боту отклоняется. Ручные ответы самого Business-аккаунта отражаются в топике как уже отправленные. Shell использует backend API и после регистрации не получает постоянные Telegram-секреты. CSV сообщает об ошибочных строках; email не считается Telegram-идентификатором. Приглашения или заявки на вступление позволяют сопоставить присоединившегося менеджера с его Telegram user ID и выдать минимальные права.

## Release

Release включает автоматическое создание супергрупп через отдельно авторизованный MTProto-компонент и прямые приглашения менеджеров.

Прямые приглашения через MTProto возможны только там, где Telegram их допускает; ссылка-приглашение остаётся запасным способом.

## Принятые решения и открытые вопросы

Принятые общие правила записаны в [decisions.md](../decisions.md). Вопросы, по которым решения ещё нет, находятся в [open-questions.md](../open-questions.md).
