# Open Questions

Status: Draft. These are unresolved decisions, not implied requirements.

## Decision blockers for the MVP

1. **Customer-facing identity**
   Does the MVP require a regular Telegram bot, a Telegram Business account, or only one of them initially?

2. **Bot ownership**
   For a regular bot, must the bot belong to the product owner, may it be managed by the platform, or must both onboarding modes be supported? Must the product owner retain the bot after leaving the platform?

3. **Bot topology**
   Does every product require a distinct bot, or may several products share a platform bot and be selected through a deep link or prior product context?

4. **Support group topology**
   Does every product always have exactly one private forum supergroup? Can a group serve multiple products, or can a product have multiple groups?

5. **Group creation**
   Must the MVP create the supergroup automatically, or may the owner create/select it manually and register it in the Shell?

6. **Owner-account relationship**
   Must every product have a distinct Telegram owner account, or may one company-controlled account own several product groups? Is the owner account expected to be the same account customers contact?

7. **Manager message delivery**
   Does every ordinary manager message sent in a customer topic immediately go to the customer? If yes, where do managers place internal discussion? If no, what explicit send action is required?

8. **Manager permissions**
   Are managers normal group members with permission to write, or do any roles require administrator rights such as managing topics, inviting users, or deleting messages?

9. **CSV contract**
   Which columns are guaranteed: name, email, username, phone, Telegram user ID, requested role? Who sends personal invite links when email is present?

10. **Initial message scope**
    Is text-only sufficient for the first release, or must photos, files, voice messages, albums, replies, and edits work from day one?

## Product behavior

11. Is there one persistent topic per customer/product pair, or can the same customer open multiple tickets?
12. What closes a conversation, and what happens when either side sends a new message after closure?
13. What should the topic title contain when the customer has no username or changes their display name?
14. Should managers see the customer's Telegram username, numeric ID, language, product metadata, or prior conversations in a pinned topic card?
15. Should the customer see which manager answered, or should all replies remain anonymous behind the bot/business identity?
16. Do managers need assignment, ownership, escalation, SLA, tags, or status beyond Telegram's topic state?
17. What notification behavior is required beyond each manager's Telegram group notification settings?
18. What should happen when the customer blocks the bot or delivery fails?

## Telegram Business

19. If Business accounts are supported, will owners connect a shared platform connector bot or provide their own Business-enabled bot?
20. What should happen when the account already has another Business Bot connected?
21. Which Business Bot rights are mandatory, and which are optional?
22. How should the system handle a manager response when the connected bot is outside Telegram's eligible recent-incoming-message window?
23. Are the customer-facing Business account and the support-group owner account allowed or expected to differ?

## Provisioning and security

24. What is meant by an "account file": an authorized Telegram session, phone numbers, usernames, or another export format?
25. Who is responsible for Telegram login codes and two-factor authentication during automated provisioning?
26. Where may encrypted user sessions and bot tokens be stored, and who may rotate or revoke them?
27. Is direct MTProto invitation of managers a requirement, or is a personal invite link acceptable?
28. What audit history and retention policy apply to customer messages, manager replies, delivery failures, and account actions?

## Runtime and deployment

29. Is Cloudflare the required deployment platform or the current preference?
30. Is D1 sufficient as the source of truth, or must this service share PostgreSQL data with another system?
31. What delivery guarantees are required before adding a queue and dead-letter handling?
32. What expected message volume and burst concurrency should guide the need for per-conversation serialization?
33. Which environments are required, and how are Telegram webhooks isolated between local, staging, and production bots?
