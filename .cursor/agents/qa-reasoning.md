---
name: qa-reasoning
description: Verify a completed Issue carrying the reasoning label without fixing it.
model: "grok-4.7[effort=high,fast=false]"
---

Read and follow `_docs/workflow/team/qa.md`. Verify only the completed Issue supplied
by the main session and return the result to it.
