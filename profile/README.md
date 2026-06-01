<div align="center">

<img src="https://raw.githubusercontent.com/vaultaris/.github/main/assets/logo.svg" alt="Vaultaris" width="120" />

# Vaultaris

**Open-source Identity & Access Management — built for the modern stack.**

[![License](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue?style=flat-square)](LICENSE)
[![Rust](https://img.shields.io/badge/rust-2024%20edition-orange?style=flat-square&logo=rust)](https://www.rust-lang.org)
[![PostgreSQL](https://img.shields.io/badge/postgres-14+-336791?style=flat-square&logo=postgresql&logoColor=white)](https://www.postgresql.org)
[![Docker](https://img.shields.io/badge/docker-ready-2496ED?style=flat-square&logo=docker&logoColor=white)](https://hub.docker.com)
[![Kubernetes](https://img.shields.io/badge/kubernetes-native-326CE5?style=flat-square&logo=kubernetes&logoColor=white)](https://kubernetes.io)
[![OpenID Connect](https://img.shields.io/badge/OpenID-Connect-F78C40?style=flat-square)](https://openid.net/connect/)

*Self-hosted · Multi-tenant · ~50 MB RAM · 150+ endpoints*

[Documentation](#) · [Quickstart](#getting-started) · [Roadmap](ROADMAP.md) · [API Docs](#)

</div>

---

## What is Vaultaris?

Vaultaris is a **production-grade, self-hostable IAM platform** written entirely in Rust. It handles authentication, authorization, and identity for your applications — without the cost of a SaaS provider or the weight of a JVM-based solution.

Think of it as the IAM layer your stack deserves: fast, modern, and fully under your control.

```
Your Applications → Vaultaris → Verified Identities
```

It speaks **OAuth 2.0 / OpenID Connect** natively, supports **WebAuthn / Passkeys**, implements **RBAC + ABAC**, and was built **multi-tenant from day one**.

---

## Why Vaultaris?

### The problem with existing options

| | Auth0 / Okta | Keycloak | Vaultaris |
|---|---|---|---|
| **Cost** | $$$$ per user/month | Free (but ops-heavy) | Free, open-source |
| **Self-hosted** | No | Yes | Yes |
| **RAM usage** | — | ~500 MB+ (JVM) | ~50 MB |
| **Startup time** | — | 30-60 seconds | < 1 second |
| **Multi-tenant** | Paid add-on | Complex setup | Built-in |
| **WebAuthn/Passkeys** | Paid tier | Limited | Full FIDO2 |
| **Plugin system** | No | Limited | 12-category SDK |
| **Kubernetes-native** | No | Adapter needed | Native webhooks |
| **License** | Proprietary | Apache 2.0 (BSL parts) | MIT OR Apache-2.0 |

### The Vaultaris advantage

**Lightweight by design.** Rust has no garbage collector, no JVM warmup, no runtime overhead. Vaultaris starts in milliseconds and stays under 50 MB of RAM in production — a 10x improvement over Keycloak on the same workload.

**Multi-tenancy is not an afterthought.** Every resource — users, roles, clients, identity providers, policies — is scoped to a tenant. Run a single instance to serve dozens of independent organizations with full isolation.

**Modern security primitives out of the box.** WebAuthn/FIDO2/Passkeys, TOTP with backup codes, ABAC policies with 20+ operators, device trust registry, and immutable audit logs. No paid tier required.

**Extensible without forking.** A 12-category plugin system with ABI stability (via `stabby`) lets you add custom credential checkers, storage backends, email providers, rate limiters, audit sinks, and more — without touching core code.

**Kubernetes-native.** Native `TokenReview` and `SubjectAccessReview` webhook handlers. Drop Vaultaris in front of your cluster and use it as the auth layer without operators or sidecars.

---

## Features

### Authentication

- **OAuth 2.0 / OpenID Connect** — Authorization Code (+ PKCE), Client Credentials, Password, Refresh Token, Implicit flows
- **JWT management** — HS256 / RS256 / ES256, configurable lifetimes per tenant, key rotation
- **WebAuthn / FIDO2 / Passkeys** — hardware keys, Touch ID, Face ID, Windows Hello, clone detection via sign counter
- **TOTP MFA** — RFC 6238, QR code generation, backup codes
- **Password security** — Argon2id hashing, policy enforcement (history, rotation, lockout, complexity), reset flows

### Authorization

- **RBAC** — hierarchical roles with inheritance, composite roles, group-user assignment
- **ABAC** — Rego-like policy engine with 20+ operators, conditions on any resource attribute
- **Fine-grained permissions** — `resource:action` model (e.g. `orders:create`) with version tracking
- **API key management** — keys with scoped RBAC/ABAC, IP restrictions, time-based conditions

### Multi-tenancy

- Complete isolation per organization — branding, security policies, token lifetimes, all configurable
- Hosted tenants — tenant-on-tenant for SaaS platforms
- Billing shadow mode for chargeback setups

### Federation & SSO

- **Identity provider federation** — Google, GitHub, Azure, any generic OAuth2/OIDC provider
- Attribute mapping, auto-user creation, group sync, login restrictions
- **Global Sessions / Cross-domain SSO** — domain transfer tokens, wildcard domain matching
- **LDAP/AD** — entity model ready, sync engine in progress
- **SAML 2.0** — on the GA roadmap

### Developer Experience

- 150+ RESTful endpoints with auto-generated OpenAPI / Scalar documentation
- **SDKs**: Rust (native), Python (PyO3 bindings), Node.js (napi-rs in progress)
- Health endpoints: `/health`, `/ready`, `/live`
- Prometheus metrics at `/metrics`
- Comprehensive integration examples

### Observability & Compliance

- **Immutable audit logs** with entity versioning, before/after diffs
- **GeoIP tracking** on login events (local MaxMind database)
- Structured JSON logging via `tracing`
- SOC 2 / GDPR compliance tooling on the roadmap

### Plugin System

Extend Vaultaris without forking:

| Category | Examples |
|---|---|
| `AuthProvider` | Custom credential validators |
| `IdentityProvider` | LDAP, SAML, custom federation |
| `StorageBackend` | HashiCorp Vault, AWS KMS |
| `NotificationChannel` | Email, SMS, push, Slack |
| `EncryptionProvider` | HSM / KMS integration |
| `PolicyEngine` | Custom ABAC rules |
| `AuditSink` | SIEM, S3, syslog |
| `JwtCustomizer` | Add / transform JWT claims |
| `RateLimiter` | Token bucket, vendor APIs |
| `MetricsCollector` | Datadog, OpenTelemetry |
| `InputValidator` | Custom validation logic |
| `LifecycleHook` | Cross-cutting observers |

Plugins ship as native shared libraries (`.so` / `.dylib`) with TOML manifests and ABI stability guaranteed via the `stabby` crate. Per-tenant slot isolation. WASM and scripting language wrappers are in progress.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│┌───────────────────────────────────────────────────────────────┐│
││                         Vaultaris Stack                       ││
│├─────────────┬──────────────────────┬──────────────────────────┤│
││  Dashboard  │   Control Plane UI   │   Marketing / Docs Site  ││
││  (Nuxt 3)   │      (Nuxt 3)        │        (Astro)           ││
│├─────────────┴──────────────────────┴──────────────────────────┤│
││                      Traefik (TLS termination)                ││
│├────────────────────────────┬──────────────────────────────────┤│
││     Vaultaris API          │    Vaultaris Control Plane       ││
││  (Rust · Axum · Tokio)     │     (Rust · Axum · Tokio)        ││
│├────────────────────────────┴──────────────────────────────────┤│
││                 PostgreSQL 17          Redis (optional)       ││
│└───────────────────────────────────────────────────────────────┘│
│                      Plugin Marketplace (coming)                │
└─────────────────────────────────────────────────────────────────┘
```
<!--
### Backend structure

```
src/
├── api/
│   ├── handlers/       # OAuth, users, roles, apps, sessions …
│   ├── extractors/     # Auth extraction (tenant, token, session)
│   ├── middleware/     # CORS, rate limiting, request ID, timeouts
│   └── routes/         # Route registration
├── domain/
│   ├── entities/       # User, Tenant, Role, Permission, Policy …
│   ├── services/       # Business logic (JWT, OAuth, MFA, freeze …)
│   └── repositories/   # Data access abstractions
└── infrastructure/
    ├── database/       # SQLx compile-time queries
    ├── security/       # Argon2id, WebAuthn, JWT signing
    ├── plugins/        # Plugin loader & dispatcher
    └── background/     # Cleanup, webhook delivery, heartbeat
```

### Workspace crates

| Crate | Purpose |
|---|---|
| `vaultaris-types` | Shared types (TokenValidation, UserProfile …) |
| `vaultaris-plugin-sdk` | Plugin development kit (12 trait categories) |
| `vaultaris-plugin-macros` | Proc-macros (`#[plugin_trait]`, `#[plugin_export]`) |
| `vaultaris-plugin-host` | Plugin runtime (loader, dispatcher, registry) |
| `vaultaris-control-plane` | Separate service for instance / license management |
| `vaultaris-crypto` | `EncryptedField<T>`, `KeyManager` |
| `vaultaris-email` | SMTP · SendGrid · Mailgun · AWS SES · Brevo |
| `vaultaris-tenant-export` | Tenant export / import framework |
| `vaultaris-polar` | Polar.sh billing integration |

-->
---

## Technology Choices

Every technology in Vaultaris was chosen deliberately.

### Rust — because correctness and performance are not optional in auth

Auth is the most security-critical layer of any application. Memory safety bugs — buffer overflows, use-after-free, data races — are the root cause of the majority of CVEs. Rust eliminates this entire class of vulnerabilities **at compile time**, with zero runtime overhead.

Beyond safety: Rust's async ecosystem (`tokio` + `axum`) achieves throughput comparable to Go and C++ while being easier to reason about under concurrency. The result is an IAM server that handles thousands of concurrent OAuth flows on commodity hardware without tuning.

**Why not Go?** Go has a GC (adds latency jitter under load) and a weaker type system — easier to introduce logic errors silently. For a security-critical service, we preferred the Rust trade-off.

**Why not Java/Keycloak?** JVM startup cost, ~500 MB baseline RAM, and GC pauses are disqualifying for cloud-native workloads where you pay per resource and scale to zero.

### Axum — ergonomic, composable, zero-cost abstractions

Axum is the de-facto standard async web framework in Rust. It composes Tower middleware natively, has first-class extractor support (exactly what you want for `TenantId`, `AuthToken`, `SessionContext`), and generates zero overhead — handlers compile down to plain function calls.

### PostgreSQL — the only relational database worth trusting for IAM data

IAM data is relational: users belong to tenants, roles belong to users, permissions belong to roles. Joins are not optional. PostgreSQL's ACID guarantees, mature replication, fine-grained row-level security, and JSONB support for flexible metadata make it the only reasonable choice. SQLite would crumble under concurrent write workloads; MySQL lacks the type richness and ecosystem.

**sqlx** — compile-time query checking. Every SQL query in Vaultaris is verified against the live schema at compile time. Schema drift that would cause a runtime panic in other ORMs becomes a **compile error** here.

### Nuxt 3 (Vue 3 + TypeScript) — for the dashboard

The admin dashboard is a complex SPA: data tables, policy editors, session management, multi-step wizards. Nuxt 3's composables, SSR capabilities, and TypeScript-first approach give a clear separation of concerns and fast iteration. Tailwind CSS and CodeMirror 6 (for JSON / CSS editors) round out the component stack.

**Why not React?** Vue's reactivity model is a better fit for form-heavy CRUD interfaces, and Nuxt's conventions reduce boilerplate. This was a deliberate trade-off — React is not wrong, just not the right fit here.

### Astro — for the marketing site

Marketing pages are read-mostly and SEO-critical. Astro's Islands Architecture ships zero JavaScript by default, scores perfect Lighthouse scores, and lets you embed Vue components only where interactivity is needed. The result is a fast, indexable site with a single build tool.

### Docker + Traefik + Kubernetes

**Docker / Compose** for local development — one command spins up the full stack including two PostgreSQL databases, Redis, all three frontends, and the API server.

**Traefik** for reverse proxy — automatic TLS via Let's Encrypt, zero-config routing with labels, and native Docker / Kubernetes provider support.

**Kubernetes** via Helm chart + native webhooks. No sidecar required. Vaultaris responds to `TokenReview` and `SubjectAccessReview` directly, making it a drop-in auth layer for any Kubernetes cluster.

---

## Getting Started

### Docker Compose (recommended)

```bash
git clone https://github.com/vaultaris/vaultaris.git
cd vaultaris
docker compose up -d
```

Dashboard → `http://localhost:3000`
API → `http://localhost:8080`

### Pre-built binary

```bash
curl -L https://github.com/vaultaris/vaultaris/releases/latest/download/vaultaris-linux-x86_64.tar.gz | tar xz
DATABASE_URL="postgres://user:pass@localhost/vaultaris" \
JWT_SECRET="your-secret-here-min-32-chars" \
./vaultaris
```

### Kubernetes (Helm)

```bash
helm install vaultaris oci://ghcr.io/vaultaris/helm/vaultaris \
  --set config.database.url="postgresql://user:pass@postgres/vaultaris" \
  --set config.jwt.secret="your-secret"
```

### From source

```bash
cargo build --release
DATABASE_URL="..." JWT_SECRET="..." ./target/release/vaultaris
```

---

## Roadmap

| Version | Status | Highlights |
|---|---|---|
| v0.1.0 | Done | OAuth2/OIDC, RBAC, multi-tenancy, audit logging |
| v0.2.0 | Done | WebAuthn, TOTP MFA, ABAC, IdP federation, plugin system, SDKs |
| v1.0.0 GA | In progress | LDAP sync, SAML 2.0, SCIM, full UI coverage, HA docs |
| Post-GA | Planned | Magic links, risk-based auth, Kubernetes Operator, Terraform provider, GraphQL, gRPC |

Full details in [ROADMAP.md](ROADMAP.md).

---

## License

Vaultaris is dual-licensed under **MIT** and **Apache 2.0** — your choice. See [LICENSE-MIT](LICENSE-MIT) and [LICENSE-APACHE](LICENSE-APACHE).

---

<div align="center">

Built with Rust. Designed for production. Free forever.

[GitHub](https://github.com/vaultaris) · [Documentation](#) · [Issues](https://github.com/vaultaris/vaultaris/issues) · [Discussions](https://github.com/vaultaris/vaultaris/discussions)

</div>
