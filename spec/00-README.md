# The FastAPI Path

Eight projects, ordered by difficulty, that take you from "I can return JSON" to "I can build a service that survives concurrent duplicate requests, multiple workers, and an unreliable upstream." Each one teaches something FastAPI-specific that a tutorial skips.

---

## Read This First

There is no to-do API here and no blog API. Those teach you nothing the documentation doesn't, and building them produces the specific false confidence of having finished something without having learned anything. Every project on this path has a part that breaks in a way you have to think about. That part is the project. The CRUD around it is scaffolding.

The ladder is real. Project 3 is where async stops being syntax you copy and becomes a design decision you get wrong. Project 6 is where request-response stops being safe to treat as stateless. If you skip rungs, you will hit a project whose hard part assumes a lesson you haven't learned yet, and you will conclude the project is badly designed when actually you arrived early.

**Who this is for.** Someone who can already write a route, a Pydantic model, and a database query, and wants to stop building toys. If you are still learning what a route *is*, do the official FastAPI tutorial first, then come back. If you already build production services in your sleep, projects 1 and 2 are beneath you; start at 3, and know that past project 8 the framework stops being the interesting variable and you should move to a distributed systems path instead.

**The rule that separates this from a tutorial:** each project has a verifier, a program that tries to prove your service is wrong. A service that has only been clicked through by hand is a service you are hoping works. On the later projects, hoping is exactly the failure the verifier exists to catch.

## The three deliverables per project

1. **A short DESIGN note before you code:** what the service does, what it promises the caller, and the one failure you are designing against. One page.
2. **The implementation.**
3. **NOTES.md after:** the answers to the questions at the end of each spec, with numbers where numbers exist. This is the document that proves you understood the project rather than completed it.

## The projects

| # | File | Project | Teaches | Rough effort |
|---|---|---|---|---|
| 1 | `01-shortener.md` | URL shortener with analytics | Status codes, background writes, the hot path | Weekend |
| 2 | `02-auth.md` | Auth service done properly | Dependency injection, tokens, the logout problem | 1 week |
| 3 | `03-uploads.md` | Upload and async processing | Async as a design concern, task queues | 1-2 weeks |
| 4 | `04-realtime.md` | WebSocket chat / notifications | Connection lifecycle, multi-process state | 1-2 weeks |
| 5 | `05-aggregator.md` | Resilient API aggregator | Concurrent outbound calls, caching, degradation | 1-2 weeks |
| 6 | `06-idempotent.md` | Idempotent payment-style API | Idempotency under concurrent duplicates | 2-3 weeks |
| 7 | `07-multitenant.md` | Multi-tenant API with quotas | Distributed state across workers, atomic counters | 2-3 weeks |
| 8 | `08-outbox.md` | Event-driven service with an outbox | The dual-write problem, at-least-once delivery | 3-4 weeks |

## How the ladder is built

1 and 2 are fundamentals. 3 is the pivot: async becomes a design concern, not syntax. 4 and 5 are real-time and resilience, and both deliver your first "my design assumed one process and production has many" moment. 6, 7, and 8 are where FastAPI stops being the thing you're learning and becomes the thing you build *with* while you learn distributed systems.

Each of the last three maps onto a harder project on the backend systems path, so you can graduate cleanly: project 6 (idempotency) leads into payment and durable-execution work, project 7 (quotas) into distributed rate limiting, project 8 (outbox) into change-data-capture pipelines.

**The path stops at 8 on purpose.** Past here, the interesting difficulty is entirely in the distributed protocol and the web framework is incidental. A "project 9" would be a saga coordinator, and at that point you should be on the systems path, not pretending Uvicorn is the point.
