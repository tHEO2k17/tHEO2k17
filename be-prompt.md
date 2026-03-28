You are refactoring the AfriLoop backend to match the exact architectural pattern used in Tekora.

This is not a generic NestJS cleanup.
This is not a light modularization pass.
This is a full architectural alignment to the Tekora backend pattern.

Your job is to refactor the existing AfriLoop backend codebase so it follows the same core structure, layering rules, CQRS discipline, modular monolith strategy, documentation style, and engineering conventions as Tekora.

You must preserve working business behaviour unless a change is required to align architecture, fix broken boundaries, remove duplication, or eliminate incorrect layering.

======================================================================
PRIMARY GOAL
======================================================================

Refactor AfriLoop backend to follow Tekora’s architectural pattern exactly in spirit and structure:

- modular monolith first
- separate API app and worker app
- bounded context modules
- strict internal layering:
  - domain
  - application
  - infrastructure
  - presentation
- CQRS with CommandBus, QueryBus, EventBus
- domain entities owning invariants
- repository interfaces in domain
- Prisma implementations only in infrastructure
- thin controllers
- external integrations hidden behind interfaces
- background jobs handled by BullMQ in worker
- realtime handled as transport concern, not domain concern
- shared kernel kept intentionally small
- DRY, KISS, SOLID and clean architecture throughout

Do not invent a different pattern.
Do not flatten the architecture.
Do not keep legacy shortcuts that violate the target architecture.

======================================================================
TEKORA PATTERN TO MATCH
======================================================================

The target architecture must align with these principles:

1. Modular monolith first
- One deployable API plus a separate worker process for async jobs
- Modules isolated enough for future extraction into microservices
- Distribution tax must be deferred until necessary

2. Bounded contexts
- Each major domain area is a bounded context / NestJS module
- Each module owns its own language, entities, repositories, commands, queries, events, and transport surface

3. Internal module structure
Every bounded context must follow:
- domain/
- application/
- infrastructure/
- presentation/
- <module>.module.ts

4. Dependency rule
- presentation -> application -> domain
- infrastructure -> domain
- domain must not import from application, infrastructure, or presentation

5. CQRS discipline
- controllers only dispatch commands and queries
- no controller business logic
- commands mutate state
- queries read state
- events publish side effects
- command handlers orchestrate, not domain-bloat or infra-bloat

6. Domain purity
- no Prisma types in domain
- no DTOs in domain
- no HTTP logic in domain
- no framework leakage in domain
- domain entities own invariants and state transitions

7. Infrastructure isolation
- Prisma repositories live in infrastructure only
- external services and adapters live in infrastructure only
- mapping happens at the infrastructure boundary

8. Shared kernel rules
- only truly cross-cutting primitives go into shared kernel
- no dumping context-specific logic into shared
- avoid bloated common folders

9. Documentation discipline
- architecture docs must describe the real implemented system
- docs must be updated after the refactor, not left stale

======================================================================
AFRILOOP DOMAIN TO MODEL
======================================================================

AfriLoop is an African social lifestyle commerce platform with these core business areas:

- Identity and Access
- User Profile
- Messaging
- Translation
- Social Feed
- Social Graph
- Marketplace / Commerce
- Events
- Discovery
- Notification
- Moderation
- Media
- Administration
- Realtime transport
- Background jobs

Use bounded contexts similar to Tekora, but adapted to AfriLoop’s domain.

Recommended AfriLoop bounded contexts:
- identity
- profiles
- messaging
- translation
- social-feed
- social-graph
- commerce
- events
- discovery
- notifications
- moderation
- media
- administration
- realtime

You may adjust names slightly only if the result is clearer and still matches the Tekora pattern.

======================================================================
TARGET REPOSITORY SHAPE
======================================================================

Refactor the repository toward a Tekora-style monorepo layout.

Target structure:

afriloop-backend/
├── apps/
│   ├── api/
│   │   ├── src/
│   │   │   ├── main.ts
│   │   │   ├── app.module.ts
│   │   │   └── health/
│   │   ├── test/
│   │   └── Dockerfile
│   └── worker/
│       ├── src/
│       │   ├── main.ts
│       │   └── worker.module.ts
│       └── Dockerfile
├── modules/
│   ├── identity/
│   ├── profiles/
│   ├── messaging/
│   ├── translation/
│   ├── social-feed/
│   ├── social-graph/
│   ├── commerce/
│   ├── events/
│   ├── discovery/
│   ├── notifications/
│   ├── moderation/
│   ├── media/
│   ├── administration/
│   └── realtime/
├── packages/
│   ├── shared-kernel/
│   ├── common/
│   └── config/
├── infrastructure/
│   ├── database/
│   │   ├── prisma/
│   │   │   ├── schema.prisma
│   │   │   ├── migrations/
│   │   │   └── seed.ts
│   │   └── prisma.module.ts
│   ├── cache/
│   ├── storage/
│   └── queue/
├── docs/
├── docker/
├── .env.example
├── package.json
└── tsconfig files

Within every module use:
- domain/
- application/
- infrastructure/
- presentation/
- <module>.module.ts

======================================================================
MODULE INTERNAL RULES
======================================================================

For every bounded context, enforce these rules:

DOMAIN
- aggregates, entities, value objects, domain events, repository interfaces, domain services if truly needed, domain exceptions
- plain TypeScript classes
- no Prisma imports
- no NestJS decorators unless absolutely necessary
- domain methods enforce invariants

APPLICATION
- commands and handlers
- queries and handlers
- application DTOs
- mappers for app response models when appropriate
- policies and orchestration
- event handlers for side effects inside application boundary
- no Prisma implementation details

INFRASTRUCTURE
- Prisma repositories
- persistence mappers
- external provider adapters
- queue producers / processors
- storage adapters
- translation provider adapters
- notification providers
- encryption services
- websocket infra helpers if needed

PRESENTATION
- HTTP controllers
- WebSocket gateways
- transport DTOs
- presenters
- guards and transport-specific concerns
- no business logic

======================================================================
NON-NEGOTIABLE CODE RULES
======================================================================

Refactor the codebase to enforce all of the following:

- no business logic in controllers
- no service classes acting as god services
- no direct Prisma usage from controllers or handlers unless it is inside an infrastructure repository or explicit read service designed for query side
- no Prisma types crossing into domain
- no DTOs mixed with entities
- no anemic domain model where entities are just data bags
- no circular dependencies between modules
- no duplicated command/query boilerplate where a clean shared abstraction can remove repetition
- no duplicated mapper logic
- no duplicated event handler boilerplate
- no repeated unknown enum casts unless isolated in one boundary mapper
- no dumping everything into shared or common
- no direct module-to-module concrete coupling when a command, query, repository interface, or event should be used

Keep the design DRY, KISS and SOLID.
Prefer clarity over over-abstraction.
Avoid premature generic frameworks inside the codebase.

======================================================================
CQRS RULES
======================================================================

Implement CQRS the Tekora way:

- controllers dispatch commands and queries only
- commands represent business mutations
- queries represent reads
- command handlers load aggregates, validate preconditions, invoke domain methods, persist, then publish events
- queries return read models / response DTOs
- events are published after persistence
- event handlers must be idempotent

Expected handler pattern:

1. load aggregate(s)
2. validate preconditions
3. invoke domain method
4. persist through repository interface
5. publish event

======================================================================
AFRILOOP-SPECIFIC BOUNDED CONTEXT RESPONSIBILITIES
======================================================================

Refactor AfriLoop around these responsibilities:

IDENTITY
- registration
- login
- email/password auth
- phone/OTP auth if already present or planned
- JWT access/refresh tokens
- refresh token rotation
- logout/logout all
- password reset
- user account status
- platform roles

PROFILES
- username
- display name
- bio
- avatar
- language preference
- country
- traveler/local flag
- creator/business flag

MESSAGING
- 1:1 conversations
- group conversations
- messages
- attachments
- read receipts
- typing indicators transport integration
- message lifecycle

TRANSLATION
- translation provider abstraction
- original and translated message content strategy
- translation metadata
- async translation jobs if needed

SOCIAL-FEED
- short video/image/text posts if relevant
- captions
- hashtags
- saves
- feed reads

SOCIAL-GRAPH
- likes
- comments
- replies
- follows
- blocks
- reports on social content where it fits better than moderation

COMMERCE
- listings
- categories
- seller linkage
- listing media
- listing status
- negotiation entry via messaging

EVENTS
- event creation
- event categories
- bookings/reservations
- attendee tracking
- saved events

DISCOVERY
- trending content
- nearby places
- featured events
- featured creators
- location-aware queries

NOTIFICATIONS
- in-app notifications
- unread counts
- preferences
- push-ready abstractions

MODERATION
- reports
- review queue
- takedown actions
- sanctions

MEDIA
- upload metadata
- MinIO/S3 integration
- media file lifecycle

ADMINISTRATION
- admin oversight endpoints
- moderation oversight
- content and user management dashboard reads

REALTIME
- websocket session and room transport only
- no business persistence
- transport concern only

======================================================================
READ / WRITE SEPARATION
======================================================================

Where reads are wide and aggregated, keep them query-side.
Do not force all reads through aggregate reconstitution.

For example:
- feed
- discovery
- admin dashboards
- notifications list
- marketplace listing views
- event discovery views

These may use read repositories / query services in the application + infrastructure boundary, while preserving domain purity on the write side.

======================================================================
AUTH / SECURITY RULES
======================================================================

Use the Tekora-style security posture:

- access token short-lived
- refresh token long-lived
- refresh token rotation
- refresh token hash stored, not raw token if architecture already supports it
- token reuse detection where possible
- JWT guard
- RBAC guards
- CurrentUser decorator
- public decorator where needed
- rate limiting on login/register/refresh/reset endpoints
- bcrypt or argon2
- no raw password logging
- secure secret handling
- config validation at startup

======================================================================
REALTIME / JOBS RULES
======================================================================

Realtime:
- Socket.io transport
- Redis adapter for horizontal scaling
- gateway in presentation
- broadcast service in application or transport adapter as appropriate
- no persistent business state in realtime module

Background jobs:
- BullMQ
- worker app separate from API app
- jobs for translation, notifications, media processing, and other slow operations
- idempotency guards
- job status exposed from DB-backed state, not only queue internals

======================================================================
OBSERVABILITY / OPERATIONS RULES
======================================================================

Refactor toward:
- structured logging
- correlation IDs
- health endpoints
- global exception filter
- validation pipe
- Swagger
- config validation
- graceful shutdown
- Docker local stack
- Bull Board dev only
- readiness for later OpenTelemetry integration if not already present

======================================================================
DOCUMENTATION UPDATE SCOPE
======================================================================

After the code refactor, update or create docs under docs/ so AfriLoop’s documentation reflects the new Tekora-aligned architecture.

Create or update these docs in AfriLoop, adapted from Tekora but rewritten for AfriLoop’s domain:

1. 01-architecture-overview.md
2. 02-tech-stack.md
3. 03-folder-structure.md
4. 04-bounded-contexts.md
5. 05-data-model.md
6. 06-api-design.md
7. 07-cqrs-design.md
8. 08-translation-pipeline.md
9. 09-realtime.md
10. 10-auth-security-rbac.md
11. 11-background-jobs.md
12. 12-observability.md
13. 13-testing-strategy.md
14. 14-configuration.md
15. 15-docker-deployment.md
16. 16-code-conventions.md
17. 17-starter-code.md
18. 18-migration-path.md
19. 19-operator-runbooks.md

Important:
- the docs must describe AfriLoop, not Tekora
- the docs must match the final implementation, not aspirational fluff
- keep the same seriousness, clarity, and operational usefulness as the Tekora docs
- where AfriLoop differs from Tekora, explain the difference explicitly
- do not copy irrelevant AI orchestration or repository bootstrap concepts from Tekora unless AfriLoop genuinely has an equivalent
- instead of Tekora’s AI decomposition pipeline, document AfriLoop’s message translation pipeline and any content/media async flows

======================================================================
DATABASE / PRISMA RULES
======================================================================

Refactor Prisma to support the new bounded contexts cleanly.

- keep Prisma schema in infrastructure/database/prisma/
- use explicit mappers from Prisma models to domain entities
- use Prisma only in infrastructure
- use indexes and relations intentionally
- maintain clear ownership of tables by bounded context
- do not let Prisma become the domain model

If the existing schema is messy:
- normalize names where needed
- keep migrations safe
- document the migration path
- do not break data silently

======================================================================
DELIVERY PROCESS
======================================================================

Execute the work in this order:

PHASE 1 — ANALYZE
- inspect current AfriLoop backend structure
- inspect current modules, services, controllers, schemas, queues, websocket code, and docs
- identify architecture violations relative to Tekora pattern
- identify duplicated logic
- identify mixed-layer files
- identify god services and direct Prisma leakage
- identify incorrect module boundaries

PHASE 2 — PLAN
- produce a concrete refactor plan
- map current code into target bounded contexts
- show file moves, renames, splits, and deletions
- list risky areas and mitigation approach

PHASE 3 — REFACTOR
- move files into target structure
- split mixed-responsibility files
- introduce repository interfaces in domain
- move Prisma repos to infrastructure
- move business rules into domain entities/value objects
- introduce commands, queries, handlers, events
- thin down controllers
- isolate adapters
- add worker separation where needed
- update module wiring
- remove dead code
- preserve behaviour

PHASE 4 — DOCUMENT
- update the architecture docs listed above
- ensure docs reflect final state
- add migration notes where legacy code was changed significantly

PHASE 5 — VERIFY
- ensure imports resolve
- ensure modules compile
- ensure no circular dependencies
- ensure lint/build/test commands are updated
- ensure Docker and env docs are aligned

======================================================================
OUTPUT EXPECTATIONS
======================================================================

As you work:

- first show the architecture diff between current AfriLoop and target Tekora-aligned structure
- then perform the refactor incrementally
- explain major architectural decisions briefly
- keep code changes reviewable and grouped by concern
- when updating docs, keep them precise and implementation-grounded

======================================================================
STRICT GUARDRAILS
======================================================================

Do not:
- keep legacy service-oriented structure if it violates the target architecture
- create fake DDD by renaming folders only
- leave business logic in controllers
- leave Prisma in handlers when it belongs in repositories
- create bloated base services
- overabstract with generic repository frameworks
- dump module-specific concerns into shared
- produce docs that no longer match code

Do:
- make the architecture genuinely match Tekora’s pattern
- keep the codebase practical
- keep file boundaries crisp
- make the docs trustworthy
- preserve DRY, KISS, and SOLID throughout
