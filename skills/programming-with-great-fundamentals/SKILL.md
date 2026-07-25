---
name: programming-with-great-fundamentals
description: Checklist of XP, System Design, SOLID, Hexagonal Architecture, DDD, DS&A, and Clean Code principles to apply when designing, writing, or reviewing non-trivial code. Use when starting a new feature/service, doing a significant refactor, making an architectural decision, or when the user explicitly asks to "program with great fundamentals" / review against these principles.
---

Reference checklist for engineering a change the right way, not just a working way. Walk through the relevant sections below **before** writing code (design), and re-check them **after** (review). Skip sections that plainly don't apply (e.g. DS&A for a config change) — don't force a checklist onto trivial work.

## 1. XP (Extreme Programming)

### Pair programming → AI pair programming
- Treat this conversation as pairing, not dictation: narrate the design before coding, surface trade-offs, flag risky choices, and let the user redirect before you commit to an approach. Consistent with this user's global instruction to "explain before you code."
- Don't silently pick between two reasonable designs — say what you're choosing and why in one line.

### TDD → AI-assisted development
- Prefer writing (or asking for) the test/spec before the implementation when adding new behavior, especially for business logic and bug fixes (a regression test that reproduces the bug first).
- Tests should drive the interface design — if a function is hard to test, that's a design smell, not a testing inconvenience.
- AI-assisted TDD means the test net is now cheap for every team, not just teams with strong test discipline — don't skip tests because "this is simple," the cost of writing them is low.

### CI/CD — deterministic pipeline rules
- Anything that can be checked by a machine (lint, type-check, tests, format) should be — don't rely on manual review to catch what a pipeline rule catches deterministically.
- New code should pass the project's existing lint/type/test commands before being considered done; run them, don't assume.
- Small, deterministic gates > large manual checklists.

### Refactoring: continuous improvement
- Leave code slightly better than you found it *within the scope of the task* — don't do drive-by rewrites of unrelated code.
- Refactor in a separate step/commit from behavior changes when the change is non-trivial, so the diff is reviewable.
- Never refactor without tests (existing or newly added) covering the behavior being preserved.

### KISS + YAGNI: simple, evolutionary design
- Solve the problem in front of you, not the one you imagine might show up later. No speculative abstractions, no config flags for hypothetical futures.
- Three similar lines beats a premature abstraction. Extract only once a real third use appears.
- Prefer the boring, obvious solution over the clever one unless the clever one is materially better.

### Small, frequent releases
- Prefer small, reviewable, independently-shippable increments over one large batched change — smaller diffs mean smaller blast radius and faster rollback if something's wrong.
- If a task naturally splits into independent pieces, say so and propose landing them separately rather than one big PR.

## 2. System Design

### CAP theorem
- When a design involves distributed/replicated state, name the CAP trade-off explicitly: what does this choose under partition — consistency or availability — and is that the right choice for this data (e.g. financial/PHI data usually wants C, a dashboard usually tolerates AP).

### Scalability & resiliency
- Identify the hot path and whether it needs caching, indexing, or async processing before it becomes a bottleneck — but don't add these preemptively without evidence (YAGNI still applies).
- No synchronous blocking calls inside event handlers; no manual retry loops without exponential backoff — these are hard blockers per this user's standing instructions.
- No shared mutable state across adapters/services; no two services sharing a database.
- Design for graceful degradation and idempotency where failures are expected (network calls, external APIs, queues).

### Security
- Auth: is access checked at the right boundary (not just at the UI)? Least privilege by default.
- Encryption: sensitive data encrypted at rest and in transit — non-negotiable for anything HIPAA-adjacent per this user's global instructions.
- Access rules: per-tenant/per-partner isolation must be enforced at the data layer, not just application logic, when handling multi-tenant or regulated data.
- User-based access control: enforce it as authorization, not just authentication — verify the caller both *is who they say* and *is allowed to touch this specific resource*, on every resource-scoped operation (read, write, delete), not just at login/route entry.
  - **Role-based (user vs. admin):** admin-only actions must be checked server-side by role/permission, never inferred from a hidden UI element or trusted from a client-supplied flag.
  - **Ownership (user vs. user):** a resource ID in a request is not authorization — always scope the query/check to the authenticated caller (e.g. `WHERE owner_id = current_user.id`, not `WHERE id = :resourceId` alone). This is what prevents IDOR (Insecure Direct Object Reference) — one user fetching/editing another user's data by guessing or changing an ID.
  - Default-deny: if a resource's owner/permissions can't be resolved, deny access rather than falling through to allow.
- If any of encryption at rest, immutable audit trails, per-partner access isolation, or deletability can't be met by a proposed data design — stop and ask before proceeding (hard blocker).

## 3. SOLID
- **S**ingle Responsibility — a module/class/function should have one reason to change. If describing what it does needs "and," it's doing too much.
- **O**pen/Closed — prefer extending behavior (new implementation, new case) over modifying stable, already-tested code paths.
- **L**iskov Substitution — a subtype/implementation must be usable anywhere its interface is expected, without surprising the caller.
- **I**nterface Segregation — don't force a consumer to depend on methods it doesn't use; split fat interfaces along real usage boundaries.
- **D**ependency Inversion — high-level logic depends on abstractions (ports/interfaces), not concrete low-level implementations (DBs, HTTP clients, SDKs).

## 4. Hexagonal architecture (ports & adapters) — inspired
- Keep business/domain logic free of framework, transport, and persistence details. The domain shouldn't import an HTTP client or an ORM model directly.
- Define ports (interfaces) for what the domain needs (a repository, a notifier, a clock) and let adapters implement them — this is what makes the domain testable without a real DB/API.
- Don't force full hexagonal ceremony (ports/adapters/DI wiring) onto a small script or a genuinely simple CRUD endpoint — apply this where domain logic is non-trivial or the boundary is likely to change (e.g. swapping providers), not everywhere by default.

## 5. Domain-Driven Design (DDD)
- **Ubiquitous language** — code should use the same vocabulary the domain experts/user use for the business concept; if the user calls it a "route," don't name it `TripPlan` in code.
- **Bounded contexts** — the same term can mean different things in different parts of the system (e.g. "Order" in Billing vs. Fulfillment); don't force one shared model across contexts where the concepts have actually diverged. This is what defines the seams for services/modules — and what a hexagonal port typically wraps.
- **Entities vs. value objects** — model things with identity/lifecycle (a `Customer`, an `Order`) as entities; model things defined purely by their attributes (money, an address, a date range) as immutable value objects. Don't give a value object an identity it doesn't need.
- **Aggregates** — group entities/value objects that must change together under one aggregate root, and enforce invariants at that root; don't let external code reach in and mutate a child directly, bypassing the invariant.
- **Domain logic lives in the domain** — business rules belong on domain objects/domain services, not scattered across controllers, DB triggers, or UI code ("anemic domain model" is a smell, not a default).
- Apply this where the domain has real, non-trivial business rules and vocabulary worth protecting — skip the ceremony (aggregates, bounded-context mapping) for simple CRUD or infrastructure-only work.

## 6. Data structures & algorithms
- Pick the data structure that matches the access pattern (lookup vs. ordered iteration vs. membership test) rather than defaulting to whatever's already in scope.
- Call out algorithmic complexity when it matters — an O(n²) loop over a bounded, small n is fine; the same shape over an unbounded/user-controlled collection is a latent bug.
- Don't over-engineer: a `Map`/`dict` and a loop is usually enough. Reach for a specialized structure only when the access pattern actually demands it.

## 7. Clean code
- Names say what something is/does; comments (sparingly) say *why*, only when the why isn't obvious from the code (a workaround, a non-obvious constraint, a subtle invariant).
- Functions do one thing at one level of abstraction; avoid deep nesting — prefer early returns/guard clauses.
- No dead code, no commented-out code, no half-finished implementations or fallback paths for scenarios that can't happen.
- Consistent with the project's existing conventions over introducing a new style, even a "better" one, mid-codebase.

---

## How to use this in a review

When asked to review or self-check a change against these principles, go section by section and report only what's actually relevant to the diff — don't pad the review with sections that don't apply. For each real finding: name the principle violated, the concrete risk (not just "this violates SRP" but what breaks because of it), and a fix.
