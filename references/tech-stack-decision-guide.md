# Tech Stack Decision Guide (2025-2026)

A practical reference for software architects choosing technologies. Organized by decision criteria — team size, scale, budget, and use case — rather than exhaustive feature lists.

---

## 1. Frontend Frameworks

### Component Frameworks (UI Libraries)

| Framework | Best For | Avoid When |
|-----------|----------|------------|
| **React** | Large teams, big hiring pool, complex SPAs, rich ecosystem needed | You need maximum performance on low-end devices; bundle size is critical |
| **Vue** | Small-to-mid teams wanting gentle learning curve, progressive adoption into existing apps | You need the deepest enterprise ecosystem or largest talent pool |
| **Svelte** | Performance-critical UIs, smaller teams, projects where bundle size matters | You need a massive ecosystem of third-party component libraries |
| **Angular** | Large enterprise teams wanting opinionated structure, TypeScript-first, complex forms-heavy apps | Small teams, rapid prototyping, projects where bundle size is critical |
| **Solid** | React-like DX with fine-grained reactivity and top-tier performance | You need a mature ecosystem; team is unfamiliar with signals model |

**Decision shortcut:**
- Default enterprise choice -> **React** (ecosystem, hiring)
- Startup / small team wanting speed -> **Vue** or **Svelte**
- Large enterprise with strict conventions needed -> **Angular**
- Performance is the top priority -> **Solid** or **Svelte**

### Meta-Frameworks (Full-Stack / SSR / SSG)

| Framework | Built On | Best For |
|-----------|----------|----------|
| **Next.js** | React | Full-stack React apps, hybrid SSR/SSG/ISR, API routes, large teams. The default choice for React projects that need server rendering. App Router (RSC) is the current direction. |
| **Nuxt** | Vue | Same role as Next.js but for Vue. Excellent DX, auto-imports, file-based routing. |
| **SvelteKit** | Svelte | Full-stack Svelte apps. Lean, fast, great DX. Adapter system for flexible deployment targets. |
| **Astro** | Any (React, Vue, Svelte, etc.) | Content-heavy sites, marketing pages, blogs, docs. Ships zero JS by default ("islands architecture"). Use when content outweighs interactivity. |
| **Remix** | React | Data-heavy apps, progressive enhancement, nested routing. Merged into React Router v7; strong forms/mutations model. |

**Decision shortcut:**
- Content/marketing site -> **Astro**
- React full-stack app -> **Next.js** (or Remix/React Router v7 for progressive enhancement focus)
- Vue full-stack app -> **Nuxt**
- Svelte full-stack app -> **SvelteKit**

---

## 2. Backend Frameworks

### Node.js

| Framework | Best For | Notes |
|-----------|----------|-------|
| **Express** | Legacy projects, maximum middleware ecosystem | Mature but aging. Express 5 is stable. Minimal by design. |
| **Fastify** | New Node APIs where performance matters | 2-3x faster than Express, schema-based validation, better plugin architecture. Prefer over Express for new projects. |
| **Hono** | Edge/serverless, multi-runtime (Bun, Deno, CF Workers, Node) | Ultralight, Web Standards API. Great for Cloudflare Workers and edge deployments. |
| **NestJS** | Enterprise Node.js, teams wanting Angular-like structure | Opinionated, DI-based, TypeScript-first. Sits atop Express or Fastify. |

### Python

| Framework | Best For | Notes |
|-----------|----------|-------|
| **Django** | Full-featured web apps, admin-heavy CRUD, content management | Batteries-included (ORM, admin, auth). Large ecosystem. Async support maturing. |
| **FastAPI** | APIs, microservices, ML model serving | Async, auto-generated OpenAPI docs, Pydantic validation. The default for new Python APIs. |
| **Flask** | Simple APIs, small projects, learning | Minimal, flexible. Consider FastAPI instead for new projects unless you need specific Flask extensions. |

### Java / JVM

| Framework | Best For | Notes |
|-----------|----------|-------|
| **Spring Boot** | Enterprise Java, large organizations with Java teams | Massive ecosystem, mature, well-understood. Spring Boot 3.x + GraalVM native image for faster startup. |
| **Quarkus** | Cloud-native Java, serverless, containers | Faster startup and lower memory than Spring Boot. Kubernetes-native. |
| **Micronaut** | Microservices, serverless | Compile-time DI, low memory footprint. |

### Go

| Framework | Best For | Notes |
|-----------|----------|-------|
| **stdlib `net/http`** | Simple services, teams that prefer minimal dependencies | Go 1.22+ enhanced routing makes stdlib viable for many use cases. |
| **Gin** | REST APIs needing more structure | Most popular Go web framework, good middleware ecosystem. |
| **Echo** | Similar to Gin with slightly different API preferences | Both Gin and Echo are solid; pick based on team preference. |

### Rust

| Framework | Best For | Notes |
|-----------|----------|-------|
| **Axum** | New Rust web services | Built on Tokio/Tower ecosystem, modular, type-safe extractors. The current community favorite. |
| **Actix Web** | High-throughput services | Extremely fast, mature, slightly more opinionated than Axum. |

### Other

| Framework | Best For | Notes |
|-----------|----------|-------|
| **Rails (Ruby)** | Rapid prototyping, startups, CRUD-heavy apps | Convention over configuration. Excellent for shipping fast with small teams. Rails 8 with Solid Queue/Cable/Cache reduces infrastructure deps. |
| **Phoenix (Elixir)** | Real-time apps (chat, IoT, collaboration), high concurrency | LiveView for server-rendered interactivity. BEAM VM provides fault tolerance and massive concurrency. |
| **ASP.NET Core (C#)** | Microsoft ecosystem shops, enterprise, high performance | Excellent performance, strong typing, good for teams already in the .NET ecosystem. |

### Backend Decision Matrix

| Criteria | Recommendation |
|----------|---------------|
| Fastest time-to-market, small team | Rails, Django, or Next.js API routes |
| High-performance API | Go, Rust (Axum), Fastify, ASP.NET Core |
| ML/AI model serving | FastAPI |
| Enterprise, large Java team | Spring Boot |
| Real-time / high concurrency | Phoenix (Elixir), Go |
| Serverless / edge | Hono, Fastify, Go |
| Microsoft shop | ASP.NET Core |

---

## 3. Databases

### Relational

| Database | Choose When | Avoid When |
|----------|-------------|------------|
| **PostgreSQL** | Default choice for relational data. Advanced features (JSONB, full-text search, extensions like pgvector, PostGIS). Handles complex queries well. | You need the absolute simplest embedded DB or ultra-low-resource environments. |
| **MySQL** | Existing MySQL expertise, WordPress/PHP ecosystem, read-heavy workloads. PlanetScale (Vitess) for managed MySQL at scale. | You need advanced SQL features, JSON querying, or full-text search quality that PostgreSQL handles better. |
| **SQLite** | Embedded/mobile apps, single-server apps, development/testing, edge (via Turso/LiteFS). Surprisingly capable for production single-node. | Multi-writer concurrency, distributed systems, or large teams needing concurrent schema migrations. |

**Default recommendation:** PostgreSQL. It handles 90% of use cases and its extension ecosystem (pgvector, PostGIS, TimescaleDB, pg_cron) means you can often avoid adding separate infrastructure.

### Document

| Database | Choose When | Avoid When |
|----------|-------------|------------|
| **MongoDB** | Flexible schema, rapid prototyping, document-oriented data, geospatial. Atlas for managed. | You need strong relational integrity, complex joins, or transactions across many collections. |
| **DynamoDB** | AWS-native, predictable latency at any scale, serverless architectures, simple access patterns. | Complex queries, ad-hoc analytics, you want to avoid vendor lock-in. You must design access patterns upfront. |
| **CouchDB** | Offline-first sync, multi-master replication. | General-purpose use; community has shrunk. Consider Firestore or MongoDB for most document needs. |

### Key-Value / Cache

| Database | Choose When | Avoid When |
|----------|-------------|------------|
| **Redis** | Caching, session storage, rate limiting, pub/sub, leaderboards, queues. The Swiss Army knife. Redis 8 (post-license revert to open source). | Your only need is simple caching and Memcached would suffice at lower memory cost. |
| **Valkey** | Same use cases as Redis, want a fully open-source fork (Linux Foundation backed). Drop-in Redis replacement. | You specifically need Redis Stack modules (RedisJSON, RediSearch) that Valkey hasn't replicated. |
| **Memcached** | Simple caching, multi-threaded performance on large cache pools. | You need data structures beyond key-value, persistence, or pub/sub. |

### Graph

| Database | Choose When | Avoid When |
|----------|-------------|------------|
| **Neo4j** | Relationship-heavy data (social networks, fraud detection, knowledge graphs, recommendations). Cypher query language. | Your data is primarily tabular; graph structure adds complexity without benefit. |
| **Amazon Neptune** | AWS-native graph needs, both property graph and RDF support. | You want to avoid vendor lock-in or need the strongest community/tooling (Neo4j wins here). |

**Note:** PostgreSQL with Apache AGE extension can handle moderate graph queries without a separate database.

### Time-Series

| Database | Choose When | Avoid When |
|----------|-------------|------------|
| **TimescaleDB** | Time-series data and you already use PostgreSQL. Runs as a PostgreSQL extension — one less database to operate. | You need a standalone, horizontally-scaled time-series cluster. |
| **InfluxDB** | Dedicated time-series workloads (IoT, metrics, observability). Purpose-built with its own query language (Flux/InfluxQL). | You want to avoid learning another query language or managing another database. |

**Lean recommendation:** If you already run PostgreSQL, start with TimescaleDB. Add InfluxDB only if you hit PostgreSQL's limits for time-series at scale.

### Vector (AI/Embedding Search)

| Database | Choose When | Avoid When |
|----------|-------------|------------|
| **pgvector** | You already use PostgreSQL, moderate vector search needs (<10M vectors), want to minimize infrastructure. | You need sub-millisecond latency on billions of vectors. |
| **Pinecone** | Managed service, serverless vector search, minimal ops. | Budget-sensitive, need to avoid vendor lock-in, want to self-host. |
| **Qdrant** | Self-hosted vector DB with strong filtering, Rust-based performance. | You want fully managed with zero ops (though Qdrant Cloud exists). |
| **Milvus** | Large-scale vector search (billions of vectors), complex hybrid queries. | Small-to-medium workloads where pgvector or Qdrant would be simpler. |

**Default recommendation:** Start with pgvector if you use PostgreSQL. Graduate to Qdrant or Pinecone when you outgrow it.

### Search

| Engine | Choose When | Avoid When |
|--------|-------------|------------|
| **Elasticsearch** | Large-scale search, log analytics (ELK stack), complex aggregations, geo-search. Mature and battle-tested. | Simple search needs where the operational overhead isn't justified. |
| **Meilisearch** | Typo-tolerant instant search, simple setup, developer-friendly. Great for product search, site search. | Complex aggregations, log analytics, or very large datasets (>10M docs). |
| **Typesense** | Similar to Meilisearch — fast, typo-tolerant, easy to operate. Slightly different tuning knobs. | Same caveats as Meilisearch. |

**Note:** PostgreSQL full-text search is often "good enough" for basic search and avoids adding another service.

---

## 4. Message Brokers & Event Streaming

| Broker | Choose When | Avoid When |
|--------|-------------|------------|
| **Apache Kafka** | Event streaming, event sourcing, high-throughput durable log, cross-service communication at scale. Replay capability is critical. | Simple task queues, small scale, team lacks Kafka operational expertise. Consider managed (Confluent, AWS MSK) to reduce ops burden. |
| **RabbitMQ** | Traditional message queuing, task distribution, routing flexibility (topic/fanout/header exchanges), moderate scale. | You need event replay/log semantics, or throughput >100K msg/s sustained. |
| **NATS** | Lightweight pub/sub, microservices communication, edge/IoT. JetStream adds persistence. Very low latency. | You need complex routing rules (RabbitMQ) or durable log replay (Kafka). |
| **Redis Streams** | You already run Redis and need simple streaming/queuing without adding infrastructure. Consumer groups for fan-out. | You need guaranteed delivery with sophisticated dead-letter handling or event replay at scale. |
| **AWS SQS/SNS** | AWS-native, fully managed, simple queue (SQS) or pub/sub (SNS). Zero ops. FIFO ordering available. | Multi-cloud, need replay, want to avoid vendor lock-in, need <100ms latency. |

### Decision Shortcut

| Need | Pick |
|------|------|
| Durable event log, replay, high throughput | **Kafka** |
| Task queue with routing | **RabbitMQ** |
| Lightweight pub/sub, low latency | **NATS** |
| Already using Redis, simple needs | **Redis Streams** |
| AWS-native, zero ops | **SQS/SNS** |

---

## 5. Infrastructure & Cloud

### Major Cloud Comparison (Key Services)

| Capability | AWS | GCP | Azure |
|------------|-----|-----|-------|
| Compute | EC2, ECS, EKS, Lambda | Compute Engine, GKE, Cloud Run, Cloud Functions | VMs, AKS, Container Apps, Azure Functions |
| Managed DB | RDS, Aurora, DynamoDB | Cloud SQL, AlloyDB, Firestore, Spanner | Azure SQL, Cosmos DB |
| Object Storage | S3 | Cloud Storage | Blob Storage |
| Serverless Containers | Fargate, App Runner | Cloud Run | Container Apps |
| AI/ML Platform | Bedrock, SageMaker | Vertex AI | Azure OpenAI, Azure ML |
| CDN | CloudFront | Cloud CDN | Azure CDN / Front Door |
| Message Queue | SQS, SNS, EventBridge | Pub/Sub, Cloud Tasks | Service Bus, Event Grid |

**Decision factors:**
- **AWS**: Largest market share, deepest service catalog, most third-party integrations. Default for most enterprises.
- **GCP**: Best Kubernetes experience (GKE), strongest data/analytics (BigQuery), best AI/ML platform. Good for data-heavy or AI-focused workloads.
- **Azure**: Natural fit for Microsoft / .NET shops, strong enterprise identity (Entra ID), Azure OpenAI for GPT model access.
- **Multi-cloud**: Generally avoid unless compliance requires it. The complexity cost is real. Pick one primary, use a second only for specific best-of-breed services.

### JAMstack / Frontend Hosting

| Platform | Best For | Notes |
|----------|----------|-------|
| **Vercel** | Next.js apps (they maintain it), serverless functions, preview deployments. Best DX for frontend teams. | Can get expensive at scale. Vendor lock-in with some Next.js features. |
| **Netlify** | Static sites, Astro, Hugo, JAMstack. Good forms/identity add-ons. | Slightly less Next.js-optimized than Vercel. |
| **Cloudflare Pages/Workers** | Edge-first apps, lowest latency globally, cost-effective at scale. Workers for edge compute. | Workers runtime has some Node.js API limitations. D1 (SQLite at edge) and R2 (S3-compatible storage) are compelling. |
| **AWS Amplify** | AWS-native frontend hosting, teams already deep in AWS. | Less polished DX than Vercel/Netlify. |

**Decision shortcut:**
- Next.js -> **Vercel** (or Cloudflare if cost-sensitive)
- Static/content site -> **Cloudflare Pages** or **Netlify**
- Edge-first with compute -> **Cloudflare Workers**
- AWS-native requirement -> **Amplify**

---

## 6. Authentication

| Solution | Best For | Trade-offs |
|----------|----------|------------|
| **Auth0 (Okta)** | Enterprise SSO, complex identity requirements, large organizations. Broadest protocol support. | Expensive at scale. Can be complex to configure. |
| **Clerk** | Developer-friendly auth for modern apps. Beautiful prebuilt UI components, React/Next.js focus. | Newer, smaller ecosystem. Pricing can climb with MAUs. |
| **Supabase Auth** | Already using Supabase. Row-level security integration. Good for PostgreSQL-centric stacks. | Tied to Supabase ecosystem. |
| **Firebase Auth** | Mobile apps, Google ecosystem, simple email/social auth. | Tied to Firebase/Google ecosystem. Limited customization. |
| **Keycloak** | Self-hosted enterprise identity, full OIDC/SAML, need to own your auth infrastructure. | Significant ops burden. Java-based, resource-heavy. |
| **Custom JWT** | You fully understand the security implications, have very specific requirements, or need to minimize external dependencies. | Almost always the wrong choice. Auth is a security-critical surface — bugs are costly. |

### Decision Shortcut

| Scenario | Pick |
|----------|------|
| Enterprise, SSO, compliance | **Auth0** or **Keycloak** (self-hosted) |
| Modern web app, great DX | **Clerk** |
| Using Supabase | **Supabase Auth** |
| Mobile-first, Google ecosystem | **Firebase Auth** |
| Must self-host, open source | **Keycloak** |
| Building it yourself | **Reconsider.** If you must, use a battle-tested library (e.g., `next-auth`/`Auth.js`, Passport.js, or Lucia). |

---

## 7. AI/ML Integration

### API vs Self-Hosted Models

| Approach | Choose When | Avoid When |
|----------|-------------|------------|
| **API (OpenAI, Anthropic, Google)** | Rapid development, don't want to manage GPU infra, need frontier model capabilities (GPT-4o, Claude, Gemini). | Data residency requirements prohibit external API calls, cost at high volume is prohibitive, you need full control over model behavior. |
| **Self-hosted open models (Llama, Mistral, Qwen)** | Data privacy/residency requirements, high-volume inference where API costs exceed infra costs, need fine-tuned models, offline/air-gapped environments. | Your team lacks ML ops expertise, you need frontier-level quality, or you're prototyping (start with APIs). |

**Cost crossover point:** Self-hosting typically becomes cost-effective at roughly >1M tokens/day sustained. Below that, API costs are usually cheaper than GPU rental.

**Serving infrastructure for self-hosted:** vLLM, TGI (Text Generation Inference by HuggingFace), Ollama (local development), or managed endpoints (AWS Bedrock, Azure AI, GCP Vertex).

### Choosing an LLM API Provider

| Provider | Strengths |
|----------|-----------|
| **Anthropic (Claude)** | Strongest for long-context, instruction following, coding, safety. Claude Opus for highest quality, Haiku for speed/cost. |
| **OpenAI (GPT)** | Broadest ecosystem, function calling, vision, largest community. GPT-4o for quality, GPT-4o-mini for cost. |
| **Google (Gemini)** | Best multimodal (native image/video/audio understanding), long context (1M+ tokens), competitive pricing. |
| **Open models (via API)** | Groq (fast inference), Together AI, Fireworks — host open models (Llama, Mistral) with API convenience. Good for cost-sensitive workloads where frontier quality isn't needed. |

### Vector Database Selection for AI (See Also: Section 3)

| Scale | Recommendation |
|-------|---------------|
| Prototype / <1M vectors | **pgvector** (if you use PostgreSQL) or **SQLite-vec** |
| Production / 1-100M vectors | **Qdrant** (self-hosted) or **Pinecone** (managed) |
| Large scale / >100M vectors | **Milvus** or **Pinecone** (managed) |

### RAG (Retrieval-Augmented Generation) Patterns

**Basic RAG pipeline:**
1. **Ingest:** Chunk documents -> generate embeddings (OpenAI `text-embedding-3-small`, Cohere `embed-v4`, or open-source `bge-m3`) -> store in vector DB
2. **Query:** Embed user query -> vector similarity search -> inject top-K results into LLM prompt as context
3. **Generate:** LLM produces grounded response

**When to use RAG:**
- You have domain-specific knowledge the LLM doesn't know
- Data changes frequently (vs. fine-tuning which is a point-in-time snapshot)
- You need source attribution / citations
- You want to reduce hallucination on factual queries

**RAG architecture tips:**
- **Chunking strategy matters more than vector DB choice.** Experiment with chunk size (512-1024 tokens typical), overlap, and semantic chunking.
- **Hybrid search** (combine vector similarity + keyword/BM25) consistently outperforms pure vector search. Qdrant, Elasticsearch, and pgvector + pg_trgm all support this.
- **Re-ranking** (Cohere Rerank, cross-encoder models) on top-K results before passing to LLM significantly improves relevance.
- **Don't over-engineer.** Start with a simple pipeline (LangChain/LlamaIndex or even raw API calls + pgvector). Add complexity only when retrieval quality is the bottleneck.

**Framework choices for RAG:**
| Framework | Use When |
|-----------|----------|
| **LangChain** | Need many integrations, complex chains, broad ecosystem. Can be over-abstracted for simple use cases. |
| **LlamaIndex** | Document-focused RAG, structured data querying. Better abstractions for indexing/retrieval. |
| **Raw API + vector DB** | Simple RAG, you want full control, don't want framework overhead. Often the right call for production. |
| **Vercel AI SDK** | Building AI-powered web apps with streaming, React/Next.js integration. |

---

## Quick Reference: Stack Templates

### Startup MVP (Small Team, Ship Fast)
- **Frontend:** Next.js or SvelteKit
- **Backend:** Next.js API routes, or FastAPI/Rails for separate API
- **Database:** PostgreSQL (via Supabase or Neon)
- **Auth:** Clerk or Supabase Auth
- **Hosting:** Vercel or Cloudflare
- **AI:** OpenAI/Anthropic API + pgvector

### Enterprise SaaS
- **Frontend:** React + Next.js (or Angular for large regulated teams)
- **Backend:** Spring Boot, ASP.NET Core, or NestJS
- **Database:** PostgreSQL (Aurora/AlloyDB), Redis for caching
- **Message Broker:** Kafka or RabbitMQ
- **Auth:** Auth0 or Keycloak
- **Cloud:** AWS or Azure
- **Search:** Elasticsearch

### Real-Time Application (Chat, Collaboration)
- **Frontend:** React/Next.js or SvelteKit
- **Backend:** Phoenix (Elixir) or Go + WebSockets
- **Database:** PostgreSQL + Redis
- **Message Broker:** NATS or Redis Pub/Sub
- **Auth:** Clerk or Auth0

### Content / Marketing Site
- **Framework:** Astro (with React/Svelte islands for interactivity)
- **CMS:** Sanity, Contentful, or Strapi
- **Hosting:** Cloudflare Pages or Netlify
- **Database:** Usually none (or SQLite for simple dynamic features)

### Data-Intensive / ML Platform
- **Backend:** FastAPI or Go
- **Database:** PostgreSQL + TimescaleDB, Redis for caching
- **Vector DB:** Qdrant or Milvus
- **Message Broker:** Kafka
- **Cloud:** GCP (BigQuery, Vertex AI) or AWS (SageMaker, Bedrock)
- **AI:** Mix of API (frontier models) + self-hosted (fine-tuned / high-volume)

---

## Guiding Principles

1. **Minimize moving parts.** Every additional database, broker, or service is operational burden. PostgreSQL alone covers relational, JSON documents, full-text search, vector search, time-series, and job queues (with SKIP LOCKED). Start there.

2. **Optimize for your team.** The best technology is the one your team can ship and maintain. A well-built Rails app will outperform a poorly operated microservices architecture every time.

3. **Defer decisions.** Don't pick Kafka on day one because you "might need it." Start with a database-backed queue and upgrade when you have evidence.

4. **Managed services over self-hosted** unless you have specific compliance, cost, or customization requirements. Your engineers should build product, not operate databases.

5. **Boring technology works.** PostgreSQL, Redis, and a mainstream backend framework solve most problems. Reach for specialized tools only when you have specialized needs.
