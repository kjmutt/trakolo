# Trakolo

Modular-monolith ITSM/SAM/Dev/Docs platform — React SPA + ASP.NET Core 8 API, PostgreSQL, background workers split by job type.

This is the **application repo**. It is deliberately one of only two repos in the Trakolo stack:

| Repo | Contains |
|---|---|
| `trakolo` (this repo) | `apps/web`, `apps/api` (itsm/sam/dev/docs modules), `apps/workers` (5 independently-deployed background workers), `db/` schema + migrations, `install/` (on-premise Docker Compose + Ansible), tests |
| `trakolo-infra` | Terraform/Bicep for every environment, monitoring-as-code, environment configuration |

Config and monitoring are *not* separate repos — they live in `trakolo-infra` as declarative files reviewed and deployed the same way the rest of infrastructure is. On-premise install artifacts live *here*, not in infra, because they version with the app release they support, not with Trakolo's own SaaS cloud environments.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the branching strategy and PR requirements.

## Why a modular monolith, not microservices

117 real foreign keys across the schema — including tickets and backlog items that reference each other directly — mean the modules are transactionally coupled. Splitting them into services would trade guaranteed referential integrity for distributed transactions across a genuinely tight data model. See the architecture documentation (in the companion mockup/design repo) for the full reasoning, the API Gateway layer, the hybrid multi-tenant data model, and the deployment options comparison (SaaS / Enterprise / on-premise / the Kubernetes alternative held in reserve for extreme scale).
