# Application configuration

`apps/api` uses ASP.NET Core's built-in configuration layering — each source overrides the one before it:

1. `appsettings.json` — base, checked in, no secrets. Shared by every environment.
2. `appsettings.{ASPNETCORE_ENVIRONMENT}.json` — checked in, environment-specific but still no secrets (except `Development`, see below).
3. **Azure Key Vault** — real secrets, injected at startup, never checked in.
4. Environment variables — last resort, mainly container-runtime overrides Container Apps sets directly (e.g., `ASPNETCORE_ENVIRONMENT` itself).

`ASPNETCORE_ENVIRONMENT` is set once per Container Apps environment (`Development`, `Test`, `Staging`, `Production` — see `apps/api/appsettings.*.json`) and picks which overlay file loads. `Test` here is the same tier `release-engineering.html` calls "QA / Test" — one name, to match how this repo's environments are actually provisioned in `trakolo-infra/environments/`.

## Why `Development.json` has real-looking values and the others don't

`appsettings.Development.json` commits a real local connection string (`Host=localhost;...;Password=changeme`) because it only ever points at a throwaway local Postgres/Redis a developer runs themselves — there's nothing to protect. Every other environment's overlay contains **only** a Key Vault URI, never a connection string or credential. The actual secret values are pulled at startup:

```csharp
// Program.cs
if (builder.Configuration.GetValue<bool>("KeyVault:Enabled"))
{
    var vaultUri = new Uri(builder.Configuration["KeyVault:VaultUri"]!);
    builder.Configuration.AddAzureKeyVault(vaultUri, new DefaultAzureCredential());
}
```

Key Vault secret names map to configuration keys by replacing `:` with `--`, so a secret named `ConnectionStrings--Postgres` becomes available as `Configuration["ConnectionStrings:Postgres"]` exactly as if it had been in `appsettings.json` — the application code never checks which environment it's in to decide how to read a connection string.

## Feature flags

Read from `core.feature_flags` (see `db/schema.sql`), not from `appsettings.json` — flags need to change per-tenant at runtime without a redeploy. The `FeatureFlags` section in the base `appsettings.json` only configures the *provider* (database) and cache refresh interval, matching the feature-flag-driven release pattern in `CONTRIBUTING.md` and `release-engineering.html`.

## Matching infra environment to app environment

| `ASPNETCORE_ENVIRONMENT` | `trakolo-infra` environment folder | Key Vault |
|---|---|---|
| `Development` | *(none — runs locally, no cloud resources)* | disabled |
| `Test` | `environments/test/` | `trakolo-test-kv` |
| `Staging` | `environments/staging/` | `trakolo-staging-kv` |
| `Production` | `environments/production/` | `trakolo-prod-kv` |

The Key Vault name in each `appsettings.{Env}.json` must match the vault Terraform actually creates for that environment — see `trakolo-infra/environments/<env>/*.tfvars`.
