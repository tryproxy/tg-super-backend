# Prototype tech stack

Approved for the Prototype. Serverless operation and use of free tiers are priorities. Details and limits: [architecture.md](../architecture.md).

| Purpose | Choice | Prototype cost |
| --- | --- | --- |
| Language and local tooling | TypeScript, Node.js 22, pnpm 11 | Free |
| HTTP and admin API | Hono | Free, open source |
| Telegram Bot API | grammY | Free, open source |
| Hosting | Cloudflare Workers Free | Free within plan limits |
| Database | Cloudflare D1 Free | Free within plan limits |
| Input validation | Zod | Free, open source |
| Local development and deployment | Wrangler | Free tooling |
| Tests | Vitest | Free, open source |
