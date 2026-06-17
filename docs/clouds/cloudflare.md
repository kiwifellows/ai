# Cloudflare Skills Guide

This file contains the essential Cloudflare setup items for AI tools that scaffold apps. The goal is to ensure a working Cloudflare Workers project with dev/prod environments, R2 storage, and required configuration.

## Required files

- `wrangler.toml` - primary Cloudflare Workers configuration file.
- `package.json` - scripts and dependencies.
- `src/index.js` or `src/index.ts` - worker entrypoint.
- Optional: `tsconfig.json`, `postcss.config.js`, or build config if using TypeScript or bundlers.

## Wrangler configuration

The `wrangler.toml` file must include:

- `name` - unique worker name.
- `main` - build entrypoint or module path.
- `compatibility_date` - required Cloudflare Workers compatibility date.
- `account_id` - Cloudflare account id.
- `workers_dev` - true for development or false if deploying to routes.
- `type` - `javascript`, `webpack`, or `javascript` with module syntax.
- `route` and `zone_id` for production deployment.
- `env` sections for `dev` and `production`.
- bindings for R2, KV, secrets, and other resources.

### Example `wrangler.toml`

```toml
name = "my-cloudflare-app"
main = "./dist/index.js"
compatibility_date = "2025-01-01"
account_id = "{{CF_ACCOUNT_ID}}"
workers_dev = true

[build]
command = "npm run build"

[env.dev]
workers_dev = true
zone_id = "{{CF_ZONE_ID_DEV}}"
route = ""

[env.production]
workers_dev = false
zone_id = "{{CF_ZONE_ID_PROD}}"
route = "example.com/*"

[[r2_buckets]]
binding = "MY_R2_BUCKET"
bucket_name = "my-app-r2-bucket"

[[kv_namespaces]]
binding = "MY_KV_NAMESPACE"
id = "{{CF_KV_NAMESPACE_ID}}"

[vars]
APP_ENV = "production"

[env.dev.vars]
APP_ENV = "development"
```

## Dev and production environments

- `env.dev` should be configured for local and preview deployment.
- `env.production` should include production `route`, `zone_id`, and production secrets.
- Keep environment-specific values out of source control when possible.
- Use `wrangler secret put` for secrets and `wrangler kv:namespace create` or R2 bucket creation commands in CI.

## R2 configuration

- Create an R2 bucket in Cloudflare dashboard or via CLI.
- Bind the bucket in `wrangler.toml` using `[[r2_buckets]]`.
- Use a consistent binding name such as `MY_R2_BUCKET`.
- Add environment-specific or shared bucket names as needed.

### Example R2 binding

```toml
[[r2_buckets]]
binding = "MY_R2_BUCKET"
bucket_name = "my-app-r2-bucket"
```

## Workers settings

- Set `workers_dev` appropriately for local development.
- For production, use `workers_dev = false` and configure a `route` and `zone_id`.
- Include `compatibility_date` and any feature flags required by the code.
- Add any durable object or service bindings under `[[durable_objects.bindings]]` or `[[services]]` if used.

## Secrets and environment variables

- Add Cloudflare secrets with `wrangler secret put NAME`.
- Add non-secret config values under `[vars]` and `[env.<name>.vars]`.
- Common values:
  - `CF_ACCOUNT_ID`
  - `CF_ZONE_ID_DEV`
  - `CF_ZONE_ID_PROD`
  - `CF_KV_NAMESPACE_ID`
  - `MY_R2_BUCKET`
  - `APP_ENV`

## Checklist for scaffold validation

- `wrangler.toml` exists.
- `wrangler.toml` contains `name`, `account_id`, and `compatibility_date`.
- `wrangler.toml` includes `env.dev` and `env.production` sections.
- `workers_dev` is configured for development and production separately.
- R2 bucket binding is present.
- Worker entrypoint file path is configured.
- Production route and zone id are set.
- Build command is present when using a bundler.
- Secrets are documented and configured for CI/deployment.

## Notes

- Use explicit environment names to avoid mixing dev and prod settings.
- Document any additional bindings, services, or storage resources required by the app.
- Keep the scaffold reproducible with clear defaults and example values.
