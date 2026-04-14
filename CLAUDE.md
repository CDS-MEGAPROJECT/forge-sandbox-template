# {{SANDBOX_NAME}} — CDS Forge Sandbox

## What This Is
This is a self-contained developer sandbox on the CDS Forge platform.
{{PROJECT_DESCRIPTION}}

## Your Resources
| Resource | Details |
|----------|---------|
| **Tier** | {{TIER}} |
| **CPU** | {{TIER_VCPU}} vCPU |
| **Memory** | {{TIER_MEMORY}} MB |
| **S3 Storage** | {{TIER_S3_LIMIT}} GB |
| **Database Storage** | {{TIER_DB_LIMIT}} GB |
| **Subdomain** | {{SANDBOX_NAME}}.dev.cds-megaproject.com |
| **Repository** | Private GitHub repo in CDS-MEGAPROJECT org |

## How to Connect to Your Database
Your sandbox has a dedicated PostgreSQL database. Credentials are stored in AWS Secrets Manager.

To connect from code, use the environment variables:
- `DB_HOST` — Aurora cluster endpoint
- `DB_PORT` — 5432
- `DB_NAME` — your sandbox database name
- `DB_USER` — your sandbox database user
- `DB_SECRET_ARN` — Secrets Manager ARN for the password

The `src/db/client.ts` file handles the connection automatically. To use it:
```typescript
import { pool } from './db/client';
const result = await pool.query('SELECT * FROM my_table');
```

If you need the full DATABASE_URL, fetch credentials from Secrets Manager:
```bash
aws secretsmanager get-secret-value --secret-id forge/sandbox/{{SANDBOX_NAME}}/db --query SecretString --output text
```

## How to Use S3 Storage
Your sandbox has an isolated S3 prefix:
- **Bucket**: Set in `S3_BUCKET` environment variable
- **Prefix**: `{{SANDBOX_NAME}}/`

All file operations are scoped to your prefix. Example:
```typescript
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
const s3 = new S3Client({});
await s3.send(new PutObjectCommand({
  Bucket: process.env.S3_BUCKET,
  Key: `${process.env.S3_PREFIX}myfile.csv`,
  Body: data,
}));
```

## Key Commands
- `npm run dev` — Start dev server with hot reload
- `npm run build` — Compile TypeScript
- `npm start` — Run production build
- `npm test` — Run Jest tests
- `npm run lint` — ESLint check
- `npm run migrate` — Run database migrations

## Project Structure
```
src/
  index.ts          — Express entrypoint with graceful shutdown
  routes/
    health.ts       — GET /api/health (ALB health check — do not remove)
    index.ts        — GET / main page
    example.ts      — GET /api/example sample DB query
  db/
    client.ts       — PostgreSQL pool with retry logic
    migrate.ts      — Migration runner
    migrations/     — SQL migration files (numbered, run in order)
  utils/
    logger.ts       — Structured JSON logger
tests/
  health.test.ts    — Health endpoint tests
```

## Important Rules
- **Do not remove or break `/api/health`** — the load balancer uses this to check if your app is running. If it fails, your sandbox will be marked unhealthy.
- **Do not change the listening port** — the app must listen on port 3000 (set via `PORT` env var).
- **AUTO_MIGRATE=true** — when this is set, the app runs database migrations on startup. Create new migrations in `src/db/migrations/` with numbered filenames (e.g., `002_create_users.sql`).
- **Commit and push to save** — your work is only permanently saved when pushed to GitHub.

## Coding Standards
- TypeScript strict mode — no `any` types
- ESLint + Prettier enforced
- Structured JSON logging (use `src/utils/logger.ts`)
- All database connections via the shared pool in `src/db/client.ts`

## Resource Limits
This sandbox runs on the **{{TIER}}** tier:
- CPU: {{TIER_VCPU}} vCPU
- Memory: {{TIER_MEMORY}} MB
- S3: {{TIER_S3_LIMIT}} GB
- Database: {{TIER_DB_LIMIT}} GB

The sandbox auto-hibernates after 2 hours of inactivity and shuts down nightly at 10 PM ET. It wakes automatically when accessed.

## What You Can Build
You have a full Express.js server with PostgreSQL. You can:
- Create API endpoints in `src/routes/`
- Create database tables via migrations in `src/db/migrations/`
- Serve web pages (HTML, CSS, JS) from your routes
- Upload and download files to/from S3
- Install npm packages (`npm install <package>`)
- Run background tasks and scheduled jobs

Your app is publicly accessible at `https://{{SANDBOX_NAME}}.dev.cds-megaproject.com` once deployed.
