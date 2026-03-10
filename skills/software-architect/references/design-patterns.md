# Software Design Patterns Reference

A practical reference for code-level and module-level design patterns. For each pattern: what it is, when to reach for it, and what goes wrong when misapplied.

---

## 1. Creational Patterns

Patterns that abstract object instantiation, decoupling a system from how its objects are created.

### Factory Method

- **Description**: Defines an interface for creating an object but lets subclasses or configuration decide which concrete class to instantiate.
- **When to use**: You have a family of related classes and the caller shouldn't know which concrete class it gets — e.g., a `NotificationFactory` that returns `EmailNotification` or `SMSNotification` based on user preferences.
- **Pitfalls**: Over-engineering simple construction. If you only ever have one implementation, a factory adds indirection for no benefit. Also avoid factories that accumulate dozens of unrelated creation methods — that signals a missing abstraction.

### Abstract Factory

- **Description**: Provides an interface for creating families of related objects without specifying their concrete classes.
- **When to use**: You need to create coordinated sets of objects — e.g., a UI toolkit factory that produces matching buttons, scrollbars, and text fields for a given platform (Windows, macOS). The key distinction from Factory Method: it creates *multiple* related products that must be compatible with each other.
- **Pitfalls**: Extremely heavy. Adding a new product to the family requires changing every factory implementation. Only justified when you genuinely have cross-cutting product families. Most applications never need this.

### Builder

- **Description**: Separates the construction of a complex object from its representation, allowing the same construction process to create different representations.
- **When to use**: An object has many optional parameters or configuration steps — e.g., building an HTTP request, constructing a query, assembling a complex domain object. Especially valuable when construction has validation rules (certain combinations of fields are invalid).
- **Pitfalls**: Using Builder for simple objects with 2-3 fields where a constructor or static factory method suffices. Builder adds API surface that must be maintained. Also: mutable builders that can be reused accidentally, producing shared-state bugs.

### Singleton

- **Description**: Ensures a class has only one instance and provides a global access point to it.
- **When to use**: Genuinely scarce resources — a database connection pool, a configuration registry, a logging subsystem. The resource must be expensive to create AND there must be a real reason to prevent multiple instances.
- **Pitfalls**: Singleton is the most abused pattern. Problems: hidden dependencies (anything can reach the global), untestable code (can't substitute a mock easily), concurrency hazards (lazy initialization races). Prefer dependency injection in almost all cases. If you need "one instance," create one instance and pass it around — don't enforce it at the class level.

### Prototype

- **Description**: Creates new objects by cloning an existing instance rather than calling a constructor.
- **When to use**: Object creation is expensive (e.g., involves database calls or complex computation to set up initial state) and you need many similar objects. Also useful when the concrete type is unknown at compile time — you have a reference instance and just clone it.
- **Pitfalls**: Deep vs. shallow copy bugs are the primary hazard. Cloned objects may share mutable references with the original. In languages with garbage collection, prototype rarely offers performance benefits — profile before adopting.

---

## 2. Structural Patterns

Patterns for composing classes and objects into larger structures while keeping those structures flexible and efficient.

### Adapter

- **Description**: Converts the interface of a class into another interface that clients expect, enabling classes with incompatible interfaces to work together.
- **When to use**: Integrating third-party libraries, legacy systems, or external APIs whose interface doesn't match your domain model. Classic example: wrapping a vendor's payment SDK behind your own `PaymentGateway` interface so you can swap providers later.
- **Pitfalls**: Adapter layers that do too much — if you're adding business logic in the adapter, it's no longer an adapter. Also: creating adapters preemptively for every dependency ("what if we switch?") when you have no realistic plan to switch.

### Bridge

- **Description**: Decouples an abstraction from its implementation so the two can vary independently.
- **When to use**: You have two orthogonal dimensions of variation. Classic example: shapes (circle, square) and rendering APIs (OpenGL, DirectX). Without Bridge, you'd need `CircleOpenGL`, `CircleDirectX`, `SquareOpenGL`, etc. — a combinatorial explosion.
- **Pitfalls**: Difficult to identify when you genuinely have two independent dimensions vs. one dimension that just needs a strategy. Over-applied, Bridge creates unnecessary indirection. Apply it when you see the combinatorial explosion actually forming, not preemptively.

### Composite

- **Description**: Composes objects into tree structures, letting clients treat individual objects and compositions uniformly.
- **When to use**: Naturally hierarchical structures — file systems, UI component trees, organizational charts, permission trees, menu structures. The test: "do I need to treat a group of things the same as a single thing?"
- **Pitfalls**: Operations that don't make sense on both leaves and composites (e.g., `addChild` on a leaf). Leaking the tree structure into business logic that shouldn't care about hierarchy depth. Performance issues with deep or wide trees without lazy loading.

### Decorator

- **Description**: Attaches additional responsibilities to an object dynamically by wrapping it in a decorator object with the same interface.
- **When to use**: You need to add behavior (logging, caching, compression, authorization) to an object without modifying its class. Especially powerful when behaviors are combinable — e.g., a `LoggingStream(CompressingStream(EncryptingStream(fileStream)))` pipeline.
- **Pitfalls**: Deep decorator chains become hard to debug — you lose track of which wrapper is doing what. Type identity breaks (`decorator instanceof ConcreteClass` fails). Order-dependent behavior in the chain can create subtle bugs. Consider middleware pipelines as an alternative when there are many cross-cutting concerns.

### Facade

- **Description**: Provides a simplified interface to a complex subsystem.
- **When to use**: A subsystem has grown complex and most callers only need a subset of its functionality. Common examples: a `VideoConverter.convert(file, format)` method hiding codec selection, stream handling, and encoding details. Also useful at module boundaries in a monolith.
- **Pitfalls**: Facades that become "god objects" by accumulating every possible operation. The subsystem should still be accessible for advanced use cases — a facade that completely hides the subsystem creates a ceiling. Also: facades that leak subsystem types in their API, defeating the simplification.

### Proxy

- **Description**: Provides a surrogate or placeholder for another object to control access to it.
- **When to use**: Lazy loading (virtual proxy), access control (protection proxy), remote calls (remote proxy), or caching (smart proxy). Examples: lazy-loading expensive database entities, adding auth checks before method calls, caching RPC results.
- **Pitfalls**: Performance overhead if the proxy is on a hot path and the proxied operation is cheap. Confusion when the proxy's behavior diverges from the real object (the proxy should be a transparent stand-in, not a place for business logic).

### Repository

- **Description**: Mediates between the domain and data-mapping layers, providing a collection-like interface for accessing domain objects.
- **When to use**: You want to decouple domain logic from data access. The domain layer works with a `UserRepository.findByEmail(email)` instead of writing SQL directly. Essential in Clean Architecture and DDD.
- **Pitfalls**: Repositories that expose query-builder or ORM-specific return types (leaking the abstraction). Repositories that grow to 50+ methods — this usually means you need a query object or specification pattern. Generic `Repository<T>` base classes that force every entity into the same interface regardless of its actual access patterns.

---

## 3. Behavioral Patterns

Patterns concerned with algorithms, assignment of responsibilities, and communication between objects.

### Observer

- **Description**: Defines a one-to-many dependency so that when one object changes state, all dependents are notified automatically.
- **When to use**: Decoupling event producers from consumers. UI frameworks (React's state updates), event buses, domain events (OrderPlaced triggers inventory update, email notification, analytics). Use when the source shouldn't know or care who's listening.
- **Pitfalls**: Memory leaks from forgotten subscriptions ("lapsed listener" problem). Cascade storms where one event triggers another triggers another. Debugging is hard because control flow is implicit. Performance issues when observer count grows large. In modern code, prefer explicit event types and structured event buses over raw observer interfaces.

### Strategy

- **Description**: Defines a family of algorithms, encapsulates each one, and makes them interchangeable at runtime.
- **When to use**: You have multiple ways to do something and need to switch between them — sorting algorithms, pricing calculations, authentication methods, validation rules, serialization formats. The test: "do I have an if/else or switch that selects an algorithm?"
- **Pitfalls**: Over-applying it to choices that will never change (YAGNI). Strategies that need different inputs — if each strategy requires different constructor parameters, the abstraction may be leaking. In functional languages, strategy is often just "pass a function."

### Command

- **Description**: Encapsulates a request as an object, allowing parameterization, queuing, logging, and undo of operations.
- **When to use**: Undo/redo functionality, operation queuing, macro recording, transactional behavior, audit logging. The pattern shines when you need to treat operations as first-class data — serialize them, replay them, batch them.
- **Pitfalls**: Every action becomes a class, creating a proliferation of small command objects. State management for undo can be complex (how do you undo a "send email"?). Consider event sourcing as an alternative if you need a complete operation history.

### Chain of Responsibility

- **Description**: Passes a request along a chain of handlers, where each handler decides whether to process it or pass it along.
- **When to use**: Middleware pipelines (HTTP request processing), logging level filtering, approval workflows, input validation chains. Key insight: the sender doesn't know which handler will process the request.
- **Pitfalls**: Requests that fall off the end of the chain unhandled (always have a default/fallback). Chains that are hard to reason about because processing order matters but isn't obvious. Performance issues with long chains when most requests go to the end.

### Mediator

- **Description**: Defines an object that encapsulates how a set of objects interact, promoting loose coupling by preventing objects from referring to each other directly.
- **When to use**: Complex interactions between multiple objects — chat rooms, air traffic control, form field cross-validation, GUI component coordination. Use when N objects would otherwise need N*(N-1)/2 direct connections.
- **Pitfalls**: The mediator becomes a god object that contains all the interaction logic. This trades distributed complexity for centralized complexity — make sure that's a better tradeoff. In modern architectures, message buses and event-driven patterns often serve this role better.

### State

- **Description**: Allows an object to alter its behavior when its internal state changes, appearing to change its class.
- **When to use**: Objects with complex state-dependent behavior — order processing (draft -> submitted -> approved -> shipped), connection handling (connecting -> connected -> disconnecting), document workflows. The test: "do I have many methods with switch statements on the same state variable?"
- **Pitfalls**: State explosion — if you have many states with many transitions, the number of state classes grows rapidly. Consider state machines (explicit transition tables) for complex cases. State objects that accumulate context they shouldn't know about.

---

## 4. Concurrency Patterns

Patterns for managing parallel execution, synchronization, and asynchronous operations.

### Actor Model

- **Description**: Concurrent computation model where "actors" are the universal primitive — each actor has private state, communicates exclusively via asynchronous messages, and processes one message at a time.
- **When to use**: Systems with many independent concurrent entities that need their own state — IoT device management, chat systems, game entities, real-time trading systems. Frameworks: Akka (JVM), Erlang/OTP, Microsoft Orleans, Proto.Actor.
- **Pitfalls**: Debugging is hard — message traces replace stack traces. Mailbox overflow when producers outpace consumers (apply backpressure). Deadlocks from circular message dependencies. Not suitable for tight request-response patterns where you need synchronous results — the overhead of message passing outweighs benefits.

### Producer-Consumer

- **Description**: Decouples work generation from work processing using a shared buffer or queue.
- **When to use**: Background job processing, log aggregation, ETL pipelines, any scenario where production rate differs from consumption rate. Implementations: in-memory queues, message brokers (RabbitMQ, Kafka, SQS).
- **Pitfalls**: Unbounded queues lead to memory exhaustion — always set capacity limits and define backpressure strategy. Poison messages that crash consumers repeatedly (use dead-letter queues). Message ordering guarantees vary by implementation — verify your broker's guarantees match your requirements. At-least-once vs. exactly-once semantics matter.

### Thread Pool

- **Description**: Maintains a pool of reusable threads to execute tasks, avoiding the overhead of thread creation/destruction per task.
- **When to use**: Server request handling, parallel computation, I/O-bound task batching. Most languages provide built-in pools (Java's `ExecutorService`, .NET's `ThreadPool`, Go's goroutine scheduler).
- **Pitfalls**: Pool sizing — too small starves throughput, too large causes context-switching overhead and memory waste. Rule of thumb: CPU-bound tasks = number of cores; I/O-bound tasks = cores * (1 + wait/compute ratio). Thread pool starvation from blocking calls (don't block a pool thread on I/O; use async I/O instead). Task interdependencies within the pool can deadlock.

### Async/Await Patterns

- **Description**: Language-level support for writing asynchronous code that reads like synchronous code, using cooperative multitasking (coroutines/futures/promises).
- **When to use**: I/O-bound operations — HTTP calls, database queries, file operations — where you want concurrency without threads. Standard in JavaScript/TypeScript, Python, C#, Rust, Kotlin, Swift.
- **Pitfalls**: **Colored functions** — async infects the call chain (a sync function can't easily call an async one). **Structured concurrency** — fire-and-forget tasks that crash silently; prefer structured approaches (Python's `TaskGroup`, Kotlin's `coroutineScope`). **Blocking the event loop** — accidentally running CPU-heavy work on the async executor starves other tasks. **Error handling** — unobserved exceptions in detached tasks. **Backpressure** — creating thousands of concurrent tasks without limiting parallelism (use semaphores or concurrency-limited combinators like `asyncio.Semaphore`).

---

## 5. Modern Application Architecture Patterns

### Clean Architecture / Hexagonal Architecture / Ports & Adapters

These are variations on the same core idea: **dependencies point inward**, business logic sits at the center, and infrastructure is on the outside.

- **Description**: Organize code in concentric layers — entities and use cases at the core, with interface adapters and frameworks/drivers on the outer rings. The domain never depends on infrastructure; infrastructure depends on domain interfaces (ports) via concrete implementations (adapters).
- **When to use**: Applications with significant business logic that must survive technology changes. Backend services, enterprise applications, systems where testability matters. Especially valuable when you have multiple delivery mechanisms (REST API + CLI + message consumer all calling the same use cases).
- **Pitfalls**: Massive over-engineering for simple CRUD apps — if your "business logic" is just "save to database and return," three layers of indirection add cost for no benefit. Too many mapping layers (DTO -> domain model -> persistence model) when the models are identical. Dogmatic application — adapt the layer count to your actual complexity. Start with two layers (domain + infrastructure) and add more only when pain emerges.

**Layer guidance**:
| Layer | Contains | Depends On |
|-------|----------|------------|
| Entities / Domain | Business rules, domain objects | Nothing external |
| Use Cases / Application | Application-specific logic, orchestration | Domain layer |
| Interface Adapters | Controllers, presenters, gateways | Use Cases |
| Frameworks & Drivers | Web framework, DB, external APIs | Interface Adapters |

### Domain-Driven Design (DDD)

A methodology for tackling complex domains by centering the design on the domain model.

#### Bounded Contexts

- **Description**: A explicit boundary within which a domain model is defined and applicable. Different bounded contexts can have different models for the same real-world concept (e.g., "Customer" means different things in Billing vs. Shipping).
- **When to use**: Systems large enough that a single unified model becomes contradictory or unwieldy. Each microservice or major module typically maps to a bounded context.
- **Pitfalls**: Drawing context boundaries too small (creating a distributed monolith) or too large (failing to capture semantic differences). Ignoring context mapping — you need explicit strategies for how contexts communicate (anticorruption layer, shared kernel, published language).

#### Aggregates

- **Description**: A cluster of domain objects treated as a single unit for data changes, with one entity designated as the aggregate root. All external access goes through the root.
- **When to use**: Enforcing business invariants that span multiple entities — e.g., an `Order` aggregate containing `OrderLines` ensures the total never exceeds a credit limit.
- **Pitfalls**: Aggregates that are too large (loading the entire object graph for every operation). Rule of thumb: design aggregates for consistency boundaries, not for convenient data access. Reference other aggregates by ID, not by direct object reference. Eventual consistency between aggregates is often acceptable and enables better performance.

#### Entities

- **Description**: Objects defined by their identity (an ID) rather than their attributes. Two entities with the same attributes but different IDs are different.
- **When to use**: Domain objects with a lifecycle — users, orders, accounts, devices. The key question: "does it matter *which* one this is, or just *what* it is?"
- **Pitfalls**: Making everything an entity. If two objects with identical attributes are interchangeable, it's a value object, not an entity.

#### Value Objects

- **Description**: Objects defined by their attributes, with no identity. Two value objects with the same attributes are equal. Immutable.
- **When to use**: Money (amount + currency), addresses, date ranges, coordinates, email addresses — any concept where equality is determined by value, not identity.
- **Pitfalls**: Modeling value objects as primitives ("primitive obsession") — using `string` for email instead of an `EmailAddress` value object that validates format. Forgetting immutability — value objects must be immutable to be safely shared.

### SOLID Principles at Architecture Level

SOLID principles, originally for classes, apply equally at the module and service level.

| Principle | Class Level | Architecture Level |
|-----------|------------|-------------------|
| **Single Responsibility** | A class has one reason to change | A service/module owns one business capability |
| **Open-Closed** | Extend via inheritance/composition, don't modify | Extend via new services/plugins, don't modify existing ones |
| **Liskov Substitution** | Subtypes must be substitutable | Service replacements must honor the same contracts |
| **Interface Segregation** | Don't force clients to depend on methods they don't use | APIs should be focused; split fat APIs into specific ones |
| **Dependency Inversion** | Depend on abstractions, not concretions | High-level modules define interfaces; low-level modules implement them |

**Practical application**:
- **SRP**: If a service handles user auth AND sends marketing emails, split it. Signs of violation: a service that needs to be redeployed for unrelated reasons.
- **OCP**: Design extension points (plugins, webhooks, event handlers) so new features don't require changing existing code.
- **LSP**: If you replace PostgreSQL with MySQL behind a repository interface, everything must still work. If it doesn't, your abstraction is leaking.
- **ISP**: Don't force a mobile client to download a 200-field response when it needs 5 fields. Provide tailored APIs (or use GraphQL).
- **DIP**: Your order processing logic depends on a `PaymentGateway` interface, not on `StripeSDK` directly. The composition root wires the concrete implementation.

### Dependency Injection

- **Description**: A technique where an object receives its dependencies from the outside rather than creating them internally. Dependencies are "injected" via constructor parameters, method parameters, or property setters.
- **When to use**: Always, for any dependency that is external, configurable, or needs to be mocked in tests. This is less a "pattern to reach for" and more a default coding practice.
- **Pitfalls**: DI containers (Spring, .NET DI, Dagger) that become magic — registration happens far from usage, making it hard to trace what gets injected. Service locator anti-pattern (pulling dependencies from a container instead of declaring them in the constructor). Over-injection — if a constructor takes 10 dependencies, the class is doing too much.

**Implementation guidance**:
- Prefer constructor injection (makes dependencies explicit, enables immutability)
- Use interfaces for dependencies that have multiple implementations or need mocking
- Keep the composition root (where everything is wired together) at the application entry point
- Avoid injecting the DI container itself — that's the Service Locator anti-pattern

---

## 6. Frontend Patterns

### Component Architecture

- **Description**: UI is built from self-contained, reusable components that own their own rendering logic, styles, and (optionally) state. Each component has a defined interface (props/inputs) and manages a piece of the UI tree.
- **When to use**: Any modern frontend application. This is the foundational pattern in React, Vue, Angular, Svelte, and web components.
- **Pitfalls**: Prop drilling — passing data through many layers of components that don't use it (solve with context/providers or state management). Components that are too granular (a `<Bold>` wrapper) or too monolithic (a 500-line component). Tight coupling between components that should be independent. Not defining clear component API contracts.

**Component design guidelines**:
- **Smart vs. Presentational**: Smart components manage state and logic; presentational components only render based on props. Keeps most components pure and testable.
- **Composition over configuration**: Prefer `<Card><CardHeader/><CardBody/></Card>` over `<Card header="..." body="..." />`. Composition is more flexible and avoids prop explosion.
- **Co-location**: Keep a component's styles, tests, and types next to the component file.

### State Management (Redux/Flux)

- **Description**: Centralized, predictable state management using unidirectional data flow: actions describe events, reducers produce new state, the store holds the single source of truth, and the view re-renders from state.
- **When to use**: Applications with complex shared state — multi-step forms, real-time collaboration, offline sync, undo/redo, state that many components need. Key signal: you're passing the same state through 4+ levels of components.
- **Pitfalls**: Using global state management for local UI state (a modal's open/closed state doesn't belong in Redux). Boilerplate fatigue — modern solutions (Redux Toolkit, Zustand, Jotai, Pinia) reduce this significantly. Normalizing everything when simple nested state would suffice. Not using selectors/memoization, causing unnecessary re-renders.

**Modern state management spectrum** (from simple to complex):
| Approach | When to Use |
|----------|------------|
| Component local state | State used by one component |
| Context / Providers | State shared by a subtree (theme, auth, locale) |
| Lightweight stores (Zustand, Jotai, Signals) | Shared state with simple read/write patterns |
| Redux / full Flux | Complex state with middleware, time-travel debugging, devtools needs |
| Server state libraries (TanStack Query, SWR) | Remote data caching, sync, and invalidation |

### Micro-Frontends

- **Description**: Architectural style where a frontend application is decomposed into smaller, independently deliverable frontend applications, each owned by a different team.
- **When to use**: Large organizations with multiple teams that need to ship independently. When different parts of a frontend have different tech stack requirements or release cadences. When you can't coordinate deployments across teams.
- **Pitfalls**: Significant overhead for small teams — you need shared design systems, routing coordination, and a shell/container app. Bundle size bloat from duplicated dependencies. Inconsistent UX across micro-frontends. Shared state between micro-frontends is hard — prefer backend-for-frontend (BFF) patterns. Integration approaches (iframes, web components, module federation) each have tradeoffs.

### Islands Architecture

- **Description**: A server-rendered page where interactive components ("islands") are hydrated independently, while the rest of the page remains static HTML. Pioneered by Astro; also used by Fresh (Deno) and similar frameworks.
- **When to use**: Content-heavy sites with isolated interactive widgets — blogs, documentation sites, marketing pages, e-commerce product pages. When you want fast initial load and good SEO with selective interactivity.
- **Pitfalls**: Not suited for highly interactive SPAs (dashboards, collaborative editors). Communication between islands is complex — they're intentionally isolated. Framework lock-in (each framework has its own island implementation). Can lead to jarring UX if islands load at different times without skeleton/loading states.

---

## 7. Data Access Patterns

### Repository Pattern

- **Description**: Provides a collection-like interface for accessing domain entities, encapsulating query logic behind domain-oriented methods.
- **When to use**: When you want to decouple domain logic from persistence technology. Standard in DDD and Clean Architecture.
- **Pitfalls**: See the structural patterns section above. Key additional note: repositories should return domain objects, not ORM entities or database rows.

### Unit of Work

- **Description**: Maintains a list of objects affected by a business transaction and coordinates writing out changes as a single atomic operation.
- **When to use**: When multiple repository operations must succeed or fail together. When you want to batch database writes for efficiency. ORMs like Entity Framework and Hibernate implement this internally.
- **Pitfalls**: Long-lived units of work that hold database connections or locks. Conflating the unit of work with the HTTP request lifecycle in web apps (fine for simple cases, problematic for background jobs or multi-step operations). Over-abstraction when the ORM already handles this.

### Active Record

- **Description**: An object wraps a database row, adds domain logic, and handles its own persistence (`user.save()`, `User.find(id)`).
- **When to use**: Simple CRUD applications with straightforward data models. Frameworks: Rails ActiveRecord, Django ORM, Laravel Eloquent. Works well when business logic is thin and closely mirrors the database schema.
- **Pitfalls**: Domain logic and persistence logic are intertwined — makes testing hard (need a database for unit tests), makes changing the database schema painful. Doesn't scale well with complex business rules. Fat models that accumulate hundreds of methods. Encourages anemic domain models — behavior ends up in service classes anyway.

### Data Mapper

- **Description**: A layer of mappers that moves data between domain objects and the database while keeping them independent. Domain objects have no knowledge of the database; the mapper handles translation.
- **When to use**: Complex domains where business logic should be fully independent of persistence. Systems where the domain model differs significantly from the database schema. Pair with Repository for a clean data access layer.
- **Pitfalls**: Significant boilerplate mapping code (mitigated by ORM features and code generation). Performance pitfalls from N+1 queries if the mapper loads associations lazily. The mapping layer must be kept in sync when either the domain model or schema changes.

### CQRS Read Models

- **Description**: Command Query Responsibility Segregation — use separate models for reading and writing data. The write model enforces business rules (normalized, consistent). The read model is optimized for queries (denormalized, fast).
- **When to use**: Systems with asymmetric read/write loads (reads vastly exceed writes). When read and write models naturally differ (e.g., writes update an `Order` aggregate, reads need a flattened `OrderSummary` view joining multiple entities). When you need different scaling strategies for reads vs. writes.
- **Pitfalls**: Eventual consistency between read and write models — users may not see their own writes immediately (solve with read-your-writes consistency). Operational complexity of keeping read models synchronized. CQRS adds significant complexity; don't use it for simple CRUD. If combined with event sourcing, complexity increases further — only justified for systems that genuinely need an audit trail and temporal queries.

---

## 8. Testing Patterns

### Test Pyramid

- **Description**: A testing strategy shaped like a pyramid — many fast unit tests at the base, fewer integration tests in the middle, and a handful of end-to-end tests at the top.

**Layer breakdown**:

| Layer | Quantity | Speed | What It Tests |
|-------|----------|-------|---------------|
| Unit | Many (70-80%) | Milliseconds | Individual functions, classes, business rules in isolation |
| Integration | Some (15-20%) | Seconds | Component interactions, database queries, API contracts |
| E2E / UI | Few (5-10%) | Minutes | Critical user journeys through the real system |

- **When to use**: Always — but adapt the shape. Some architectures favor a "testing trophy" (more integration tests than unit tests) when the units are thin and the value is in the integrations (e.g., a CRUD app with little business logic).
- **Pitfalls**: Inverting the pyramid (many slow E2E tests, few unit tests) — leads to slow CI, flaky tests, and difficult debugging. Unit tests that test implementation details rather than behavior (break on every refactor). Integration tests that are actually unit tests with extra steps (mocking everything except the class under test).

### Ports & Adapters for Testability

- **Description**: Design components with explicit interfaces (ports) for their dependencies, allowing test doubles (adapters) to be substituted during testing.
- **When to use**: Any code that interacts with external systems — databases, APIs, file systems, clocks, random number generators. This is the testing payoff of hexagonal architecture.
- **Pitfalls**: Mocks that don't match real behavior — your test passes but production breaks. Mitigate with contract tests (see below). Over-mocking — if every test mocks 10 dependencies, the tests verify wiring, not behavior. Consider using fakes (in-memory implementations) over mocks for complex dependencies.

**Testing double hierarchy** (use the simplest one that works):
| Double | Description | Use When |
|--------|------------|----------|
| Dummy | Passed around but never used | Satisfying a parameter requirement |
| Stub | Returns canned answers | You need a dependency to return specific data |
| Fake | Working implementation (simplified) | You need realistic behavior (in-memory DB, fake SMTP) |
| Mock | Records calls for verification | You need to verify interactions (was `sendEmail` called?) |

### Contract Testing

- **Description**: Tests that verify the contract between a service consumer and a service provider — the consumer defines expectations, the provider verifies it meets them. Tools: Pact, Spring Cloud Contract, Specmatic.
- **When to use**: Microservices or any system with API boundaries between independently deployed components. When you need confidence that services can communicate correctly without running full E2E tests. Especially valuable in polyglot environments or across team boundaries.
- **Pitfalls**: Contract tests only verify the interface shape, not business logic correctness. They require buy-in from both consumer and provider teams. Stale contracts that aren't updated when APIs evolve. Not a replacement for integration testing — it complements it by catching compatibility issues earlier.

**Contract testing workflow**:
1. Consumer writes a contract: "I expect `GET /users/1` to return `{id, name, email}`"
2. Contract is shared with the provider (via a broker or artifact)
3. Provider runs the contract against its actual implementation
4. If the contract breaks, the provider knows before deploying

---

## Quick Reference: Pattern Selection Guide

| Situation | Reach For |
|-----------|-----------|
| Need to swap implementations at runtime | Strategy |
| Need to add behavior without modifying a class | Decorator |
| Need undo/redo or operation queuing | Command |
| Complex object construction with many options | Builder |
| Tree structures with uniform treatment | Composite |
| Decoupling event source from handlers | Observer / Event Bus |
| Complex state transitions | State Machine / State Pattern |
| Multiple independent teams shipping frontend | Micro-frontends |
| Content site with isolated interactivity | Islands Architecture |
| Complex shared UI state | Redux / centralized store |
| Read-heavy system with different read/write shapes | CQRS |
| Business logic must survive tech stack changes | Clean Architecture / Hexagonal |
| Complex domain with multiple teams | DDD with Bounded Contexts |
| Coordinating multi-entity business rules | Aggregates |
| High-concurrency independent entities | Actor Model |
| Rate-decoupled processing | Producer-Consumer |
| Service API compatibility assurance | Contract Testing |
| I/O-bound concurrent operations | Async/Await + structured concurrency |
| Simple CRUD with thin logic | Active Record (keep it simple) |
| Rich domain model decoupled from DB | Data Mapper + Repository |

---

## Anti-Pattern Warnings

Patterns become anti-patterns when misapplied. Watch for these signals:

1. **Premature abstraction**: Adding patterns "just in case" before the complexity justifies them. Wait until you feel the pain, then apply the pattern.
2. **Pattern stacking**: Using Builder + Factory + Abstract Factory for object creation that a constructor could handle. Each layer must earn its place.
3. **Resume-driven architecture**: Choosing patterns or architectures to learn new technology rather than to solve the actual problem.
4. **Distributed monolith**: Microservices without actual independence — coupled deployments, shared databases, synchronous chains. Worse than a monolith.
5. **Abstraction addiction**: Interfaces for everything, even things with one implementation and no testing need. An interface should exist because something needs to vary, not because "best practices say so."

**The golden rule**: Choose the simplest pattern that solves your actual problem. The best architecture is the one your team can understand, maintain, and evolve.
