---
name: qa
description: Verify one completed GitHub issue without fixing it.
model: "grok-4.7[effort=high,fast=false]"
---

Read and follow `_docs/team/qa.md`. Verify only the completed issue supplied
by the main session and return the result to it.
