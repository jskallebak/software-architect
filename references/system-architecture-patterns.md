# System-Level Software Architecture Patterns

A reference guide for selecting architecture patterns when designing systems. Each pattern includes practical guidance on when to use it, when to avoid it, and the key tradeoffs involved.

---

## 1. Monolithic Patterns

### Modular Monolith

**Description:** A single deployable unit with strictly enforced internal module boundaries, where each module encapsulates a specific business domain with explicit interfaces between modules.

**When to use:**
- Greenfield projects where domain boundaries are not yet well understood (start here, extract services later)
- Small-to-medium teams (2-20 developers) that want clean architecture without distributed systems overhead
- Applications where deployment simplicity is valued over independent scalability
- When the team lacks operational maturity for microservices (no container orchestration, service mesh, etc.)
- When strong data consistency across modules is required

**When NOT to use:**
- Teams exceeding 50+ engineers where deployment coordination becomes a bottleneck
- When individual modules have drastically different scaling requirements (e.g., one module needs 100x the compute)
- When you need polyglot persistence or mixed technology stacks per module
- When independent deployment cadences per team are a hard requirement

**Key tradeoffs:**
- (+) Single deployment artifact, simple ops, easy local development, strong consistency via shared database
- (+) Refactoring across module boundaries is straightforward (same process, same language)
- (-) Entire application must be deployed together; one module's bug can block all releases
- (-) Scaling is all-or-nothing; cannot scale a hot module independently
- (-) Risk of module boundary erosion over time without disciplined enforcement (linting, architecture tests)

---

### Layered Architecture (N-Tier)

**Description:** Application organized into horizontal layers (presentation, business logic, data access), where each layer only depends on the layer directly below it.

**When to use:**
- Traditional enterprise CRUD applications with well-understood domain logic
- Teams new to software architecture who need a simple, well-documented pattern
- Applications with low update frequency where agility is less important
- Migrating legacy applications to cloud with minimal structural changes
- Internal business tools, admin panels, and reporting applications

**When NOT to use:**
- Applications with complex cross-cutting concerns that don't fit neatly into layers
- High-performance systems where the overhead of passing through layers adds unacceptable latency
- Applications requiring frequent, rapid iteration (layer coupling slows change)
- Systems with rich domain logic (prefer domain-driven design patterns instead)

**Key tradeoffs:**
- (+) Simple mental model, easy to understand and onboard new developers
- (+) Separation of concerns is clear; layers can be tested independently
- (+) Well-supported by most frameworks (Spring, Django, ASP.NET, Rails)
- (-) Changes often cascade across layers (adding a field touches every layer)
- (-) Tendency toward "anemic domain models" where business logic leaks into service/controller layers
- (-) Horizontal slicing makes it difficult to extract vertical features later

---

## 2. Distributed Patterns

### Microservices

**Description:** Application decomposed into small, independently deployable services, each owning its own data and implementing a single bounded context, communicating over network APIs.

**When to use:**
- Large organizations (50+ engineers) with multiple autonomous teams
- Complex domains where different parts evolve at different rates
- When independent deployment and scaling of components is a hard requirement
- When polyglot technology choices are needed (Python for ML, Go for performance, Node for I/O)
- Systems requiring fault isolation (one service failure must not cascade to the entire system)

**When NOT to use:**
- Small teams (under 10 engineers) where the operational overhead exceeds the organizational benefit
- Early-stage startups where domain boundaries are still being discovered
- Simple CRUD applications without complex domain logic
- When the team lacks DevOps maturity (CI/CD, monitoring, distributed tracing, container orchestration)
- When strong cross-service transactional consistency is needed everywhere

**Key tradeoffs:**
- (+) Independent deployment, scaling, and technology choice per service
- (+) Fault isolation; clear team ownership boundaries
- (+) Enables parallel development across teams
- (-) Significant operational complexity: service discovery, distributed tracing, config management
- (-) Data consistency is hard (eventual consistency, sagas, no cross-service transactions)
- (-) Network latency and partial failure handling add complexity to every call
- (-) Testing end-to-end flows across services is substantially harder

---

### Service-Oriented Architecture (SOA)

**Description:** Application organized as a collection of coarse-grained services that communicate through an enterprise service bus (ESB) or middleware, typically sharing data schemas and using standardized protocols.

**When to use:**
- Enterprise integration scenarios connecting multiple large legacy systems
- Organizations with existing investment in ESB infrastructure (MuleSoft, IBM Integration Bus)
- When centralized governance over service contracts and message formats is required
- Systems that need protocol mediation between diverse technologies (SOAP, REST, MQ, files)

**When NOT to use:**
- Greenfield projects (prefer microservices or modular monolith instead)
- When you want lightweight, team-autonomous services (SOA's centralized ESB is a bottleneck)
- Small-to-medium applications that don't need enterprise integration middleware
- When vendor lock-in to ESB products is unacceptable

**Key tradeoffs:**
- (+) Powerful integration capabilities across heterogeneous enterprise systems
- (+) Centralized service governance, message transformation, and routing
- (+) Well-suited for orchestrating complex multi-system business processes
- (-) ESB becomes a single point of failure and performance bottleneck
- (-) Heavy, often XML-based protocols add overhead
- (-) Shared data schemas create coupling between services
- (-) Slower to evolve compared to microservices; changes require ESB configuration updates

---

### Service Mesh

**Description:** A dedicated infrastructure layer (sidecar proxies alongside each service) that handles service-to-service communication concerns: load balancing, encryption, observability, retries, and traffic management.

**When to use:**
- Microservices architectures at scale (dozens to hundreds of services)
- When consistent mutual TLS (mTLS), retries, and circuit breaking across all services is required
- When you need fine-grained traffic control (canary deployments, A/B routing, traffic mirroring)
- When you want to decouple cross-cutting communication concerns from application code
- Kubernetes-native environments (Istio, Linkerd, Consul Connect are built for K8s)

**When NOT to use:**
- Fewer than 5-10 services (the complexity is not justified)
- Environments without container orchestration (service meshes assume Kubernetes or similar)
- When the team cannot absorb the operational overhead of managing the mesh control plane
- Latency-critical paths where sidecar proxy hop adds unacceptable overhead (typically 1-3ms)

**Key tradeoffs:**
- (+) Uniform observability, security, and traffic management without changing application code
- (+) Language-agnostic; works regardless of what each service is written in
- (+) Enables sophisticated deployment strategies (canary, blue-green) at the network level
- (-) Adds latency per hop (sidecar proxy interception)
- (-) Significant operational complexity in managing the mesh itself (control plane upgrades, config)
- (-) Debugging becomes harder when issues are in the mesh layer vs. application layer
- (-) Resource overhead: each sidecar consumes CPU and memory

---

## 3. Event-Driven Patterns

### Event-Driven Architecture (EDA)

**Description:** System design where components communicate by producing and consuming events through a broker, with producers unaware of consumers and processing happening asynchronously.

**When to use:**
- Systems requiring real-time reactions to state changes (IoT, monitoring, alerts)
- High-throughput scenarios where decoupling producers from consumers enables independent scaling
- When multiple subsystems need to react to the same event in different ways (fan-out)
- Systems that need high availability and fault tolerance (events buffer during consumer downtime)
- Cross-system integration where loose coupling is essential

**When NOT to use:**
- Simple request-response CRUD applications where synchronous calls suffice
- When strict ordering and exactly-once processing guarantees are hard requirements
- When the team is unfamiliar with eventual consistency and asynchronous debugging
- Systems where every action requires an immediate, synchronous response to the caller

**Key tradeoffs:**
- (+) Loose coupling; producers and consumers evolve independently
- (+) Excellent scalability (add consumers without modifying producers)
- (+) Natural fit for audit trails and temporal queries (events are facts that happened)
- (-) Eventual consistency makes reasoning about system state harder
- (-) Debugging asynchronous flows requires distributed tracing and correlation IDs
- (-) Message ordering, deduplication, and exactly-once delivery are hard problems
- (-) Error handling is more complex (dead letter queues, compensating transactions)

---

### Event Sourcing

**Description:** Instead of storing current state, store an append-only log of all state-changing events. Current state is derived by replaying events from the log.

**When to use:**
- Domains requiring complete audit trails (finance, healthcare, legal, compliance)
- When the ability to reconstruct past states or answer "what was the state at time T?" is needed
- Systems where understanding *why* state changed (not just *what* it is) has business value
- When combined with CQRS for high-performance read models
- Domains with complex state transitions (order workflows, booking systems)

**When NOT to use:**
- Simple CRUD applications where current state is all that matters
- When the team has no experience with event-driven patterns (steep learning curve)
- Systems where data deletion is a regulatory requirement (events are immutable; GDPR right-to-erasure requires crypto-shredding or similar workarounds)
- When event schema evolution is expected to be frequent and difficult to manage

**Key tradeoffs:**
- (+) Complete audit trail; every change is recorded as a first-class fact
- (+) Temporal queries: reconstruct state at any point in time
- (+) Natural fit for event-driven architectures and CQRS
- (+) Enables powerful debugging ("replay events to reproduce the bug")
- (-) Increased storage requirements (event log grows forever)
- (-) Read queries require projecting/replaying events (need separate read models)
- (-) Event schema versioning is complex and must be planned from day one
- (-) Mental model shift: developers must think in events, not state mutations

---

### CQRS (Command Query Responsibility Segregation)

**Description:** Separate the write model (commands that change state) from the read model (queries that return data), allowing each to be independently optimized, scaled, and evolved.

**When to use:**
- Systems with highly asymmetric read/write patterns (many reads, few writes, or vice versa)
- When read and write models have fundamentally different shapes (e.g., normalized writes, denormalized read views)
- Combined with event sourcing to build optimized read projections
- When different scaling strategies are needed for reads vs. writes
- Collaborative domains where multiple users modify the same data and conflict resolution is needed

**When NOT to use:**
- Simple domains where read and write models are nearly identical
- CRUD-heavy applications without complex query requirements
- When eventual consistency between write and read models is unacceptable to the business
- Small applications where the additional complexity is not justified

**Key tradeoffs:**
- (+) Read and write sides can be independently optimized and scaled
- (+) Read models can be purpose-built for specific query patterns (denormalized, pre-computed)
- (+) Enables polyglot persistence (SQL for writes, Elasticsearch for search, Redis for cache)
- (-) Eventual consistency between command and query sides (read-after-write may see stale data)
- (-) More code to maintain: separate models, synchronization logic, event handlers
- (-) Increased infrastructure complexity (multiple databases, event buses)

---

### Publish/Subscribe (Pub/Sub)

**Description:** Messaging pattern where publishers send messages to topics without knowledge of subscribers; subscribers express interest in topics and receive relevant messages asynchronously.

**When to use:**
- When multiple consumers need to react to the same event independently
- Decoupling components that should not have direct dependencies
- Broadcasting notifications, updates, or state changes across a system
- When new consumers may be added in the future without modifying the publisher
- Cross-service communication in microservices architectures

**When NOT to use:**
- When point-to-point communication with guaranteed delivery to one consumer is needed (use a queue)
- When strict message ordering across all subscribers is required
- When the system requires synchronous request-response semantics
- When message volume is low and direct API calls are simpler

**Key tradeoffs:**
- (+) Extreme decoupling; publishers and subscribers are completely independent
- (+) Easy to add new subscribers without changing existing code
- (+) Scales well with many consumers
- (-) No built-in response mechanism (fire-and-forget by default)
- (-) Debugging which subscriber handled (or failed to handle) a message can be difficult
- (-) Topic proliferation can become unmanageable without governance
- (-) At-least-once delivery means subscribers must be idempotent

---

## 4. Data Patterns

### Database per Service

**Description:** Each microservice owns its private database that no other service can access directly; data sharing happens only through the service's public API.

**When to use:**
- Microservices architectures where service independence is a core requirement
- When different services have different data storage needs (polyglot persistence)
- When services must be independently deployable without database schema coordination
- When you need to prevent tight coupling between services via shared tables

**When NOT to use:**
- When cross-service queries or joins are frequently needed
- Small applications where a shared database is simpler and sufficient
- When the organization cannot manage multiple database instances operationally
- When strong cross-service transactional consistency is needed (ACID across services)

**Key tradeoffs:**
- (+) True service autonomy; schema changes are local to the service
- (+) Each service can choose the optimal database technology
- (+) Fault isolation: one database going down does not take down all services
- (-) Cross-service queries require API composition or data duplication
- (-) Maintaining data consistency across services requires sagas or eventual consistency
- (-) Operational overhead of managing many database instances
- (-) Reporting/analytics across services requires a separate data warehouse or data lake

---

### Shared Database

**Description:** Multiple services or modules access the same database instance and may read/write to shared tables.

**When to use:**
- Modular monoliths where modules need transactional consistency
- Early stages of a project before service boundaries are well-defined
- Applications with heavy cross-module reporting requirements
- Small teams that cannot justify managing multiple databases
- When refactoring from monolith and service extraction is gradual

**When NOT to use:**
- Microservices architectures where service independence is a goal
- When different services need different database technologies
- When schema changes in one service frequently break other services
- When services need independent scaling of their data layers

**Key tradeoffs:**
- (+) Simple cross-module queries and joins
- (+) ACID transactions across module boundaries
- (+) Less operational overhead (one database to manage, backup, monitor)
- (-) Schema coupling: changes to shared tables can break multiple services
- (-) Scaling bottleneck: all services compete for the same database resources
- (-) No polyglot persistence; all modules use the same database technology
- (-) Deployment coupling: schema migrations affect all dependent services simultaneously

---

### Saga Pattern

**Description:** A sequence of local transactions across multiple services, where each step publishes an event or message that triggers the next step, and compensating transactions handle rollback on failure.

**When to use:**
- Distributed transactions spanning multiple microservices (e.g., order placement: reserve inventory, charge payment, schedule shipping)
- When two-phase commit (2PC) is not viable due to performance or availability requirements
- Long-running business processes that span minutes to days
- When eventual consistency is acceptable to the business

**When NOT to use:**
- When ACID transactions within a single database can solve the problem
- When the number of steps is large (>5-7) and compensating transactions become unmanageable
- When strict isolation between concurrent saga instances is required
- When the team has no experience with compensating transaction logic

**Key tradeoffs:**
- (+) Enables data consistency across services without distributed transactions
- (+) Each service maintains its own ACID guarantees locally
- (+) Two styles: choreography (event-driven, decentralized) or orchestration (centralized coordinator)
- (-) Compensating transactions must be carefully designed and tested for every failure scenario
- (-) No built-in isolation: concurrent sagas can interfere with each other (requires semantic locks)
- (-) Debugging distributed sagas across services is significantly harder than debugging local transactions
- (-) Choreography sagas can become hard to understand; orchestration sagas create a central coordinator dependency

---

### Data Mesh

**Description:** A decentralized data architecture where domain teams own and serve their analytical data as products, with federated governance and self-serve data infrastructure.

**When to use:**
- Large organizations (100+ engineers) with multiple domain teams producing data
- When centralized data teams are a bottleneck for analytics and reporting
- When domain expertise is required to ensure data quality (domain teams understand their data best)
- Organizations investing in data-as-a-product thinking
- When different domains have fundamentally different data characteristics and processing needs

**When NOT to use:**
- Small organizations where a centralized data team is efficient
- When domain teams lack the skills or willingness to own their data products
- When uniform data governance and a single source of truth are paramount
- Early-stage companies with limited data infrastructure
- When the overhead of self-serve data platforms exceeds the organizational benefit

**Key tradeoffs:**
- (+) Scales data architecture with organizational growth (avoids central bottleneck)
- (+) Domain teams have ownership and accountability for data quality
- (+) Enables domain-specific data modeling and technology choices
- (-) Requires significant investment in self-serve data platform infrastructure
- (-) Federated governance is harder to enforce than centralized governance
- (-) Data discovery across domains requires dedicated tooling (data catalogs)
- (-) Risk of inconsistent data definitions and quality standards across domains

---

## 5. API Patterns

### API Gateway

**Description:** A single entry point for all client requests that routes to appropriate backend services, handling cross-cutting concerns like authentication, rate limiting, request transformation, and SSL termination.

**When to use:**
- Microservices architectures where clients should not call backend services directly
- When cross-cutting concerns (auth, rate limiting, logging) need to be centralized
- When you need to aggregate responses from multiple backend services into one client call
- When backend service topology should be hidden from external clients
- API versioning and protocol translation needs

**When NOT to use:**
- Simple applications with a single backend service
- When the gateway would become a development bottleneck (all teams depend on gateway changes)
- When sub-millisecond latency is critical (gateway adds a hop)
- When teams need fully independent deployment without coordinating gateway routes

**Key tradeoffs:**
- (+) Single entry point simplifies client integration and provides consistent security enforcement
- (+) Cross-cutting concerns implemented once, not in every service
- (+) Enables backend service refactoring without changing client contracts
- (-) Single point of failure if not properly replicated and load balanced
- (-) Can become a bottleneck if overloaded with business logic (keep it thin)
- (-) Adds latency (extra network hop)
- (-) Risk of "smart pipe" anti-pattern: gateway accumulates too much logic over time

---

### BFF (Backend for Frontend)

**Description:** A dedicated backend service for each frontend type (web, mobile, IoT) that tailors API responses, aggregation, and protocols to the specific needs of that client.

**When to use:**
- When different clients need significantly different data shapes or protocols (mobile needs minimal payload, web needs rich data)
- When a single API gateway is becoming bloated with client-specific logic
- When different frontend teams want autonomy over their backend contracts
- When mobile apps need offline-first optimizations that web does not
- When performance optimization per client type is important (e.g., mobile gets compressed, pre-aggregated data)

**When NOT to use:**
- When all clients consume the same API shape (one API is sufficient)
- Small teams where maintaining multiple BFF services is unaffordable
- When the BFF simply proxies requests without adding value (unnecessary layer)
- When GraphQL or similar can provide client-specific queries from a single endpoint

**Key tradeoffs:**
- (+) Each frontend gets an API optimized for its specific needs
- (+) Frontend teams can iterate on their BFF independently
- (+) Reduces over-fetching and under-fetching per client type
- (-) Code duplication across BFFs (common logic may need to be extracted into shared libraries)
- (-) More services to maintain, deploy, and monitor
- (-) Risk of BFFs accumulating business logic that belongs in domain services

---

### GraphQL Federation

**Description:** A distributed GraphQL architecture where each service defines and owns a portion of the overall graph schema, and a gateway composes them into a single unified API for clients.

**When to use:**
- Organizations with multiple teams contributing to a single, unified API graph
- When clients need flexible, client-driven queries across multiple backend domains
- When reducing over-fetching and under-fetching is important for performance
- When the schema is large and no single team should own the entire API surface
- Mobile applications that benefit from requesting exactly the data they need

**When NOT to use:**
- Simple APIs with few entities where REST is sufficient
- When caching at the HTTP level (CDN, reverse proxy) is critical (GraphQL POST requests are harder to cache)
- When the team lacks GraphQL expertise (non-trivial learning curve)
- When APIs are primarily machine-to-machine (REST/gRPC is simpler and more performant)
- When fine-grained authorization per field is needed (complex with federated schemas)

**Key tradeoffs:**
- (+) Single graph across the organization; clients get exactly the data they need
- (+) Each team owns its subgraph independently
- (+) Reduces the need for BFF pattern (clients compose queries themselves)
- (-) Federation gateway is a critical piece of infrastructure to operate
- (-) Query complexity can lead to performance issues (N+1 queries, deep nesting)
- (-) HTTP-level caching is harder; requires application-level caching strategies
- (-) Schema evolution across federated services requires careful coordination

---

## 6. Deployment Patterns

### Serverless (Functions as a Service)

**Description:** Application logic runs in stateless, event-triggered compute containers that are fully managed by the cloud provider, scaling automatically from zero to peak demand.

**When to use:**
- Event-driven workloads with unpredictable or bursty traffic patterns
- Simple, stateless functions (webhooks, data transformations, cron jobs, API endpoints)
- Startups and prototypes that want zero infrastructure management
- Workloads with long idle periods (scale to zero saves cost)
- Glue code between cloud services (file upload triggers, queue processors)

**When NOT to use:**
- Latency-sensitive applications where cold starts (100ms-5s) are unacceptable
- Long-running processes exceeding execution time limits (typically 5-15 minutes)
- Applications requiring persistent connections (WebSockets, long polling)
- When vendor lock-in is unacceptable (serverless functions are highly provider-specific)
- High and constant throughput workloads (always-on containers are cheaper)
- Stateful applications that need in-memory state across requests

**Key tradeoffs:**
- (+) Zero infrastructure management; automatic scaling including to zero
- (+) Pay-per-execution pricing can be dramatically cheaper for bursty/low-traffic workloads
- (+) Rapid development for simple, event-driven functions
- (-) Cold start latency can impact user experience
- (-) Execution time limits constrain workload types
- (-) Debugging and local development are harder (cloud-specific runtime environment)
- (-) Vendor lock-in: difficult to migrate between cloud providers
- (-) Observability and distributed tracing require extra instrumentation

---

### Containers / Kubernetes

**Description:** Applications packaged as container images and orchestrated by Kubernetes (or similar platforms) for automated deployment, scaling, networking, and lifecycle management.

**When to use:**
- Microservices architectures requiring automated orchestration of many services
- When consistent environments across development, staging, and production are needed
- Applications requiring fine-grained resource allocation and auto-scaling
- Multi-cloud or hybrid cloud strategies where portability matters
- When the team needs sophisticated deployment strategies (rolling, canary, blue-green)

**When NOT to use:**
- Simple applications that can run on a single server or PaaS
- Small teams without dedicated DevOps/platform engineering capabilities
- When managed PaaS offerings (Heroku, App Engine, App Service) meet all requirements more simply
- Short-lived prototypes where infrastructure setup time exceeds development time

**Key tradeoffs:**
- (+) Portable across cloud providers and on-premises environments
- (+) Excellent resource utilization through bin-packing and auto-scaling
- (+) Rich ecosystem for service mesh, monitoring, secrets management, CI/CD
- (+) Declarative infrastructure that can be version-controlled (GitOps)
- (-) Steep learning curve for Kubernetes (networking, RBAC, storage, CRDs)
- (-) Operational overhead: cluster upgrades, node management, security patching
- (-) Overhead of running the control plane (or cost of managed Kubernetes)
- (-) Over-engineering risk: Kubernetes for a simple app is a poor investment

---

### Edge Computing

**Description:** Running compute and data processing at locations geographically close to end users or data sources, rather than in centralized cloud data centers.

**When to use:**
- Ultra-low latency requirements (gaming, AR/VR, real-time video processing)
- IoT scenarios where data must be processed near the source to reduce bandwidth
- Applications subject to data sovereignty regulations (data must stay in specific regions)
- Content-heavy applications that benefit from compute at CDN edge nodes (edge SSR, personalization)
- Offline-capable or intermittent connectivity scenarios

**When NOT to use:**
- Applications where centralized processing is sufficient and latency is not critical
- Heavy compute workloads that require GPU clusters or large databases
- When edge deployment complexity outweighs latency benefits
- Applications requiring real-time access to a large central database

**Key tradeoffs:**
- (+) Dramatically reduced latency for end users (single-digit milliseconds)
- (+) Reduced bandwidth costs by processing data near the source
- (+) Enables compliance with data locality regulations
- (-) Limited compute and storage capacity at edge locations
- (-) Managing deployments across hundreds or thousands of edge locations is complex
- (-) Data synchronization between edge and central systems is challenging
- (-) Debugging and monitoring distributed edge deployments is harder than centralized systems

---

## 7. Resilience Patterns

### Circuit Breaker

**Description:** A proxy that monitors failures in calls to a downstream service and "opens the circuit" (fast-fails) when failures exceed a threshold, preventing cascading failures and allowing the downstream service time to recover.

**When to use:**
- Any synchronous call to a remote service or database that could fail or become slow
- When a downstream dependency being slow is worse than it being down (prevents thread/connection pool exhaustion)
- Protecting against cascading failures in microservices architectures
- When graceful degradation (returning cached data, default values, or partial results) is possible

**When NOT to use:**
- Local in-process operations that cannot fail due to network issues
- When the caller cannot function at all without the downstream service (no graceful degradation possible)
- For transient failures that are better handled by simple retries

**Key tradeoffs:**
- (+) Prevents cascading failures and resource exhaustion
- (+) Gives failing services time to recover (back-off effect)
- (+) Enables graceful degradation with fallback responses
- (-) Requires tuning thresholds (failure count, timeout, recovery probe interval)
- (-) Half-open state logic adds complexity
- (-) Can mask underlying problems if fallback is too seamless (teams ignore the root cause)

---

### Bulkhead

**Description:** Isolates components into pools (thread pools, connection pools, process pools) so that a failure or resource exhaustion in one pool does not affect others.

**When to use:**
- When a single slow or failing downstream service could exhaust shared resources (connections, threads)
- Multi-tenant systems where one tenant's workload should not degrade service for others
- When different operations have different criticality levels and should not compete for resources
- Combined with circuit breakers for defense-in-depth resilience

**When NOT to use:**
- Simple applications with only one downstream dependency
- When resource overhead of maintaining multiple isolated pools is unacceptable
- When workloads are uniform and isolation provides no benefit

**Key tradeoffs:**
- (+) Prevents one failing component from consuming all system resources
- (+) Enables prioritization (critical paths get dedicated resources)
- (+) Fault isolation between different types of operations
- (-) Less efficient resource utilization (dedicated pools may have idle capacity)
- (-) More complex configuration and tuning (pool sizes, queue depths)
- (-) Adds overhead in thread/connection pool management

---

### Retry with Exponential Backoff

**Description:** Automatically retry failed operations with increasing delays between attempts (exponential backoff), typically with jitter (randomness) to prevent thundering herd.

**When to use:**
- Transient failures: network blips, temporary service unavailability, rate limiting (HTTP 429, 503)
- Cloud service calls where brief outages are common and expected
- Any idempotent operation where retrying is safe
- Combined with circuit breakers (retry for transient failures; circuit break for persistent ones)

**When NOT to use:**
- Non-idempotent operations where retrying could cause duplicate side effects (e.g., charging a credit card)
- When the failure is clearly permanent (HTTP 400, 404, authentication errors)
- Real-time operations where the retry delay exceeds the acceptable response time
- When retrying could worsen the problem (overloading an already struggling service)

**Key tradeoffs:**
- (+) Transparently handles transient failures without user impact
- (+) Exponential backoff with jitter prevents thundering herd on recovery
- (+) Simple to implement with most HTTP client libraries
- (-) Adds latency during failure scenarios (retries take time)
- (-) Without jitter, synchronized retries from many clients can overload a recovering service
- (-) Must be combined with a maximum retry count and circuit breaker to avoid infinite loops
- (-) Non-idempotent operations require idempotency keys or deduplication

---

### Sidecar

**Description:** Deploy a helper process alongside the main application container that provides supporting functionality (logging, monitoring, networking, configuration) without modifying the application itself.

**When to use:**
- Adding cross-cutting concerns (logging, metrics, tracing, TLS) to services in any language
- Service mesh implementations (Envoy, Linkerd proxy run as sidecars)
- When you need consistent infrastructure capabilities across heterogeneous services
- Legacy applications that cannot be modified but need modern observability/security

**When NOT to use:**
- Small applications where the overhead of an additional process is not justified
- When the cross-cutting concern can be handled by a simple library within the application
- Latency-critical paths where inter-process communication overhead matters
- Resource-constrained environments where the sidecar's CPU/memory overhead is prohibitive

**Key tradeoffs:**
- (+) Language-agnostic: works with any application regardless of technology stack
- (+) Independent lifecycle: sidecar can be updated without redeploying the application
- (+) Clean separation of concerns between business logic and infrastructure
- (-) Additional resource consumption (each sidecar uses CPU, memory, and network)
- (-) Adds inter-process communication latency
- (-) Increases deployment complexity and container count
- (-) Debugging issues can be harder when behavior is split across processes

---

## 8. Scaling Patterns

### Horizontal Scaling

**Description:** Adding more instances of a service behind a load balancer to handle increased load, rather than increasing the resources of a single instance (vertical scaling).

**When to use:**
- Stateless services that can handle any request without relying on local state
- When demand fluctuates and auto-scaling based on metrics (CPU, request count, queue depth) is needed
- When vertical scaling has hit hardware limits or becomes cost-prohibitive
- Cloud-native applications designed for elasticity

**When NOT to use:**
- Stateful applications that store session data locally (unless using sticky sessions or external state)
- When the bottleneck is a single database that cannot be horizontally scaled
- Applications with strong leader-election requirements where only one instance should be active
- When licensing costs scale per instance

**Key tradeoffs:**
- (+) Near-linear scaling for stateless workloads
- (+) Built-in redundancy (multiple instances provide fault tolerance)
- (+) Cloud auto-scaling enables cost optimization (scale down during low traffic)
- (-) Requires stateless application design (externalize session state, caches, etc.)
- (-) Load balancer becomes a critical component
- (-) Database often becomes the bottleneck (scaling compute without scaling data is incomplete)
- (-) More instances means more complex deployment, monitoring, and log aggregation

---

### Database Sharding

**Description:** Partitioning data horizontally across multiple database instances based on a shard key, where each shard holds a subset of the total data.

**When to use:**
- When a single database instance cannot handle the data volume or query throughput
- Multi-tenant applications where tenants can be naturally isolated by shard key (tenant ID)
- When data has a natural partition key with relatively even distribution (user ID, region, date)
- When geographic data locality is needed (shard by region)

**When NOT to use:**
- When vertical scaling or read replicas can solve the performance problem more simply
- When queries frequently need data from multiple shards (cross-shard joins)
- When the data does not have a natural, evenly-distributed partition key
- When the dataset fits comfortably on a single instance

**Key tradeoffs:**
- (+) Near-linear horizontal scalability of both storage and throughput
- (+) Each shard is smaller, improving query performance within a shard
- (+) Natural fit for multi-tenant isolation
- (-) Cross-shard queries are expensive or impossible
- (-) Shard key selection is critical and hard to change later
- (-) Uneven data distribution (hot shards) requires re-balancing
- (-) Operational complexity: managing many database instances, coordinating schema changes
- (-) Application logic must be shard-aware (routing queries to the correct shard)

---

### Read Replicas

**Description:** Maintaining one or more read-only copies of a database that receive replicated data from the primary, distributing read queries across replicas to reduce load on the primary.

**When to use:**
- Read-heavy workloads (>80% reads) where the primary database is the bottleneck
- When analytics/reporting queries should not impact transactional performance
- Geographic distribution of reads (replicas in different regions for lower latency)
- As a simpler alternative to sharding for read scaling

**When NOT to use:**
- Write-heavy workloads (replicas do not help with write throughput)
- When reads require perfectly up-to-date data (replication lag causes stale reads)
- When the additional cost of replica instances is not justified by the read volume

**Key tradeoffs:**
- (+) Simple way to scale read throughput without changing application architecture significantly
- (+) Replicas provide read availability even if primary fails (with promotion)
- (+) Geographic distribution reduces read latency for distant users
- (-) Replication lag means replicas may serve stale data (eventually consistent)
- (-) Does not help with write scaling
- (-) Additional cost for replica instances and storage
- (-) Application must be aware of read/write splitting (send writes to primary, reads to replicas)

---

### CDN (Content Delivery Network)

**Description:** A globally distributed network of edge servers that caches and serves static content (and sometimes dynamic content) close to end users.

**When to use:**
- Serving static assets (images, CSS, JS, videos, fonts) to a geographically distributed user base
- Reducing latency for content-heavy applications (media streaming, e-commerce product images)
- Offloading traffic from origin servers (reducing bandwidth costs and origin load)
- DDoS protection (CDNs absorb volumetric attacks at the edge)
- When time-to-first-byte (TTFB) is a key performance metric

**When NOT to use:**
- Purely internal applications with users close to the origin server
- Highly dynamic, personalized content that cannot be cached
- When content freshness is critical and cache invalidation complexity is unacceptable
- Small-scale applications where CDN costs exceed savings

**Key tradeoffs:**
- (+) Dramatically reduced latency for static content globally
- (+) Offloads origin server traffic, reducing infrastructure costs at scale
- (+) Built-in DDoS mitigation and edge security features
- (-) Cache invalidation complexity ("the two hard things in computer science")
- (-) CDN costs can be significant for high-bandwidth applications (egress fees)
- (-) Debugging cached vs. fresh content issues requires careful cache header management
- (-) Dynamic or personalized content requires edge compute or bypass strategies

---

### Caching Layers

**Description:** Storing frequently accessed data in fast, in-memory stores (Redis, Memcached) or application-level caches to reduce database load and improve response times.

**When to use:**
- Data that is read frequently but changes infrequently (reference data, user profiles, product catalogs)
- Expensive computations or queries whose results can be reused
- Session storage for horizontally scaled applications
- Rate limiting and counting (leveraging atomic in-memory operations)
- As a buffer between application and database to handle traffic spikes

**When NOT to use:**
- Highly volatile data that changes with every request (cache hit rate would be too low)
- When data consistency is critical and even brief staleness is unacceptable
- When the working set is too large to fit in memory cost-effectively
- Write-heavy workloads where cache invalidation overhead exceeds the read benefit

**Key tradeoffs:**
- (+) Orders-of-magnitude latency improvement for cached data (sub-millisecond vs. milliseconds)
- (+) Dramatically reduces database load for read-heavy workloads
- (+) Enables applications to handle traffic spikes without scaling the database
- (-) Cache invalidation is hard; stale data is a constant risk
- (-) Additional infrastructure to manage (Redis/Memcached clusters, high availability)
- (-) Cache stampede risk: when cache expires, many simultaneous requests hit the database
- (-) Memory costs for large cache working sets

---

## 9. Communication Patterns

### Synchronous: REST

**Description:** Request-response communication over HTTP using resource-oriented URLs, standard HTTP methods (GET, POST, PUT, DELETE), and typically JSON payloads.

**When to use:**
- Public-facing APIs that need to be easily consumable by diverse clients
- CRUD-oriented services where resource semantics map naturally
- When broad tooling support, developer familiarity, and ecosystem maturity matter
- When HTTP caching (ETags, Cache-Control) is valuable for performance
- Browser-to-server communication

**When NOT to use:**
- High-performance internal service-to-service communication where payload efficiency matters (consider gRPC)
- Real-time bidirectional communication (consider WebSockets)
- When strict API contracts with code generation are needed (consider gRPC or OpenAPI with codegen)
- Streaming data transfers (REST is request-response, not streaming)

**Key tradeoffs:**
- (+) Universal tooling support, easy to test with curl/Postman, human-readable JSON
- (+) HTTP caching at every layer (CDN, reverse proxy, browser)
- (+) Stateless by design, works well with horizontal scaling
- (-) Over-fetching and under-fetching (fixed response shapes unless using sparse fieldsets)
- (-) JSON serialization/deserialization overhead compared to binary protocols
- (-) No built-in contract enforcement (OpenAPI helps but is optional)
- (-) Chatty for complex operations requiring multiple round trips

---

### Synchronous: gRPC

**Description:** High-performance RPC framework using Protocol Buffers for binary serialization, HTTP/2 for multiplexed transport, and strongly-typed service definitions with code generation.

**When to use:**
- Internal service-to-service communication in microservices where performance matters
- When strongly-typed contracts with automatic code generation are desired
- Streaming use cases (server streaming, client streaming, bidirectional streaming)
- Polyglot environments where services in different languages need shared contracts
- Low-latency, high-throughput communication paths

**When NOT to use:**
- Browser-to-server communication (limited browser gRPC support without gRPC-Web proxy)
- Public APIs where ease of consumption and discoverability matter
- When human-readable payloads are needed for debugging
- Simple CRUD APIs where REST's tooling ecosystem is more valuable

**Key tradeoffs:**
- (+) Significantly faster than REST/JSON (binary serialization, HTTP/2 multiplexing)
- (+) Strongly-typed contracts generate client/server code, catching breaking changes at compile time
- (+) Native streaming support (server, client, and bidirectional)
- (-) Not human-readable (binary Protocol Buffers require tools to inspect)
- (-) Limited browser support without a proxy (gRPC-Web)
- (-) Steeper learning curve than REST
- (-) Load balancing is more complex (HTTP/2 long-lived connections can cause uneven distribution)

---

### Asynchronous: Message Queues

**Description:** Point-to-point messaging where producers send messages to a queue and a single consumer processes each message, with the queue providing durability, buffering, and delivery guarantees.

**When to use:**
- Work distribution across multiple worker instances (task queues, job processing)
- Decoupling producers from consumers to handle different processing speeds
- Guaranteeing that each message is processed exactly once (or at least once) by a single consumer
- Background job processing (email sending, image processing, report generation)
- Load leveling during traffic spikes (queue absorbs burst, workers process at steady rate)

**When NOT to use:**
- When multiple consumers need the same message (use pub/sub or topics instead)
- When real-time, low-latency communication is needed
- When message ordering within the queue is not important and simple fire-and-forget suffices
- Simple request-response interactions

**Key tradeoffs:**
- (+) Reliable delivery with persistence and acknowledgment
- (+) Load leveling: absorb spikes and process at a sustainable rate
- (+) Producers and consumers can scale independently
- (-) Adds latency (asynchronous processing means the result is not immediate)
- (-) Requires handling of poison messages (messages that repeatedly fail processing)
- (-) Queue management overhead (monitoring depth, scaling consumers, dead letter queues)
- (-) Debugging message flow is harder than tracing synchronous calls

---

### Asynchronous: Event Streaming

**Description:** Durable, ordered log of events (e.g., Apache Kafka, Amazon Kinesis, Azure Event Hubs) where producers append events and multiple consumer groups read independently at their own pace, with events retained for a configurable period.

**When to use:**
- High-throughput event processing (100K+ events/second)
- When multiple consumer groups need to independently process the same stream of events
- Event sourcing implementations needing a durable, ordered event log
- Real-time analytics and data pipelines
- Replay capability: new consumers can process historical events from the beginning

**When NOT to use:**
- Low-volume messaging where a simple queue is sufficient
- When messages need complex routing logic (use a message broker with routing capabilities)
- When exactly-once semantics are a hard requirement without idempotent consumers
- Small systems where Kafka/Kinesis operational overhead is not justified

**Key tradeoffs:**
- (+) Extremely high throughput and horizontal scalability
- (+) Events are durable and replayable (consumers can rewind and reprocess)
- (+) Multiple independent consumer groups from the same stream
- (+) Ordering guarantees within a partition
- (-) Ordering is only guaranteed within a partition, not globally
- (-) Operational complexity of running and tuning a streaming platform (especially Kafka)
- (-) Consumer offset management and rebalancing add complexity
- (-) Storage costs for retaining large event volumes over long periods

---

## 10. Real-Time Patterns

### WebSockets

**Description:** Full-duplex, persistent TCP connection between client and server enabling bidirectional real-time communication over a single long-lived connection.

**When to use:**
- Real-time interactive applications (chat, multiplayer games, collaborative editing)
- Live data feeds requiring server-to-client push (trading dashboards, sports scores)
- Applications requiring low-latency bidirectional communication
- When both client and server need to send messages independently at any time

**When NOT to use:**
- Unidirectional server-to-client updates (SSE is simpler)
- Request-response interactions (REST/gRPC is simpler and better supported)
- When clients are behind restrictive proxies or firewalls that block WebSocket upgrades
- When connection scalability is a concern and the server cannot handle many persistent connections

**Key tradeoffs:**
- (+) True bidirectional real-time communication with minimal overhead per message
- (+) Lower latency than HTTP polling (no connection setup per message)
- (+) Widely supported in browsers and client libraries
- (-) Persistent connections consume server resources (memory, file descriptors per connection)
- (-) Load balancing is harder (sticky sessions or connection-aware routing needed)
- (-) No built-in reconnection, message delivery guarantees, or multiplexing (must implement)
- (-) Not cacheable by CDNs or HTTP proxies

---

### Server-Sent Events (SSE)

**Description:** Unidirectional server-to-client streaming over HTTP where the server pushes events to the client over a long-lived HTTP connection, with built-in reconnection and event ID tracking.

**When to use:**
- Server-to-client push notifications (live updates, news feeds, progress indicators)
- When only the server needs to push data (client sends data via regular HTTP requests)
- As a simpler alternative to WebSockets for unidirectional streaming
- When HTTP/2 is available (multiplexed SSE connections are efficient)
- LLM streaming responses and real-time log tailing

**When NOT to use:**
- When bidirectional communication is needed (use WebSockets)
- When binary data streaming is required (SSE is text-based)
- When very high message frequency would create excessive HTTP overhead
- When you need to stream to non-browser clients that don't support SSE natively

**Key tradeoffs:**
- (+) Simple: works over standard HTTP, no protocol upgrade needed
- (+) Built-in reconnection with last-event-ID for resumption
- (+) Works through HTTP proxies, firewalls, and CDNs more reliably than WebSockets
- (+) Native browser support via EventSource API
- (-) Unidirectional only (server to client)
- (-) Text-based only (no binary data without encoding)
- (-) Limited to ~6 concurrent connections per domain in HTTP/1.1 (resolved with HTTP/2)
- (-) Less efficient than WebSockets for high-frequency bidirectional messaging

---

### CRDTs (Conflict-Free Replicated Data Types)

**Description:** Data structures that can be replicated across multiple nodes, updated independently and concurrently without coordination, and merged automatically without conflicts, guaranteeing eventual consistency.

**When to use:**
- Collaborative real-time editing (Google Docs-style concurrent editing)
- Offline-first applications that sync when connectivity is restored
- Distributed systems where network partitions are common and availability is paramount
- Peer-to-peer architectures without a central authority
- When conflict resolution must be automatic and deterministic without user intervention

**When NOT to use:**
- When a central server can serialize all writes (simpler consistency model)
- When the data structures needed are too complex for existing CRDT types
- When storage overhead from CRDT metadata is unacceptable
- When operation-level intent matters more than state convergence (e.g., "set to exactly 5" vs. increment counters)

**Key tradeoffs:**
- (+) Guaranteed eventual consistency without coordination (no consensus protocol needed)
- (+) Enables true offline-first and peer-to-peer architectures
- (+) No merge conflicts by construction
- (-) Limited set of data types (counters, sets, registers, sequences; complex structures are hard)
- (-) Metadata overhead can be significant (vector clocks, tombstones for deletions)
- (-) "Eventual consistency" may produce surprising merged states that are technically correct but not user-intended
- (-) Garbage collection of metadata (tombstone removal) is complex in open networks

---

### Operational Transform (OT)

**Description:** An algorithm for concurrent editing where operations are transformed against each other to maintain consistency, typically requiring a central server to determine the canonical operation order.

**When to use:**
- Real-time collaborative text editing (the algorithm behind Google Docs)
- When a central server is available to order operations
- Rich text editing where character-by-character operations are well-understood
- When fine-grained operation semantics matter (insert, delete, retain with attributes)

**When NOT to use:**
- Decentralized or peer-to-peer systems without a central server (CRDTs are better suited)
- Data types beyond text/lists where transform functions are hard to define correctly
- When the implementation complexity of transform functions is unacceptable
- When offline-first capability with long disconnection periods is needed

**Key tradeoffs:**
- (+) Well-proven for collaborative text editing (decades of production use in Google Docs)
- (+) Fine-grained operational semantics enable rich undo/redo
- (+) Lower metadata overhead than CRDTs for text editing use cases
- (-) Requires a central server to serialize operations (not truly decentralized)
- (-) Transform function correctness is notoriously hard to prove for complex operations
- (-) Does not work well offline (needs server round-trips to transform operations)
- (-) Scaling to many concurrent editors can strain the central server

---

## Quick Selection Guide

| If you need... | Start with... | Consider upgrading to... |
|---|---|---|
| Simple web app, small team | Modular monolith | Microservices when team/complexity grows |
| Multiple autonomous teams | Microservices | Service mesh at 20+ services |
| Audit trail / temporal queries | Event sourcing + CQRS | Add streaming platform for scale |
| Cross-service data consistency | Saga pattern | Orchestration saga for complex flows |
| Multiple client types | BFF per client | GraphQL federation for flexibility |
| Real-time server push | SSE | WebSockets for bidirectional |
| Collaborative editing | OT (with central server) | CRDTs for offline/P2P |
| Bursty, unpredictable traffic | Serverless | Containers for steady-state workloads |
| Read-heavy scaling | Read replicas + caching | Sharding when write scaling needed |
| Fault tolerance in distributed systems | Circuit breaker + retry | Add bulkhead + sidecar for defense-in-depth |
