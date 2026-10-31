# Deployment and configuration

## Verified deployment flow

The three backend repositories contain GitHub Actions workflows that run on
pull requests and pushes to `master`. The CI job installs dependencies,
builds/tests or type-checks the service, and a successful push to `master`
then:

1. Builds the production Docker image.
2. Publishes a commit-tagged image and `latest` to GitHub Container Registry
   (`ghcr.io/punitmudgal/<service>`).
3. Deploys that image to an Azure App Service using
   `azure/webapps-deploy@v3`.

The workflows use an Azure App Service name and publish profile supplied as
GitHub Actions secrets. Secret values are intentionally not documented here.

The Next.js repositories are standard Next.js applications with Vercel
deployment instructions. The customer deployment is verified at
https://nesta-foods.vercel.app/. The admin URL used by backend CORS examples
is https://nesta-admin.vercel.app/.

## Azure and supporting services

| Service | Verified responsibility |
| --- | --- |
| Azure App Service | Runs the containerized Auth, Catalog and Order APIs |
| Azure API Management | Gateway entry point used by both frontends; route policies are not stored in these repositories |
| GitHub Container Registry | Stores CI-built backend images |
| Vercel | Hosts the Next.js frontend deployments; customer URL is live |

The source repositories also configure these data/media dependencies:

- PostgreSQL for Auth Service.
- Neon serverless PostgreSQL for Catalog Service and Order Service.
- Cloudinary for catalog product images.

## Backend environment variables

Set these in the relevant App Service configuration or local environment. Do
not commit values.

### Auth Service

| Variable | Purpose |
| --- | --- |
| `PORT` | HTTP port |
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_ACCESS_SECRET` | Access-token signing secret |
| `JWT_REFRESH_SECRET` | Refresh-token signing secret |
| `JWT_ACCESS_EXPIRES_IN` | Access-token lifetime |
| `JWT_REFRESH_EXPIRES_IN` | Refresh-token lifetime |
| `JWT_BCRYPT_COST` | Password hashing cost |
| `RESEND_API_KEY` | Password-reset/welcome email provider credential |
| `RESEND_FROM_EMAIL` | Sender address |
| `CLIENT_FRONTEND_URL` | Customer origin allowed by CORS |
| `ADMIN_FRONTEND_URL` | Admin origin allowed by CORS |

### Catalog Service

| Variable | Purpose |
| --- | --- |
| `PORT` | HTTP port |
| `DATABASE_URL` | Neon/PostgreSQL connection string |
| `JWT_SECRET` | Shared HS256 access-token verification secret |
| `CLIENT_FRONTEND_URL` | Customer origin allowed by CORS |
| `ADMIN_FRONTEND_URL` | Admin origin allowed by CORS |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary account identifier |
| `CLOUDINARY_API_KEY` | Cloudinary API credential |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret |

### Order Service

| Variable | Purpose |
| --- | --- |
| `NODE_ENV` | Runtime environment |
| `PORT` | HTTP port; production Docker image defaults to `9002` |
| `DATABASE_URL` | Pooled Neon/PostgreSQL connection string |
| `DATABASE_URL_UNPOOLED` | Optional direct connection for migrations |
| `JWT_SECRET` | Same access-token secret used by Auth/Catalog |
| `CATALOG_SERVICE_URL` | Internal Catalog Service base URL |
| `CATALOG_SERVICE_TIMEOUT_MS` | Catalog request timeout |
| `CLIENT_FRONTEND_URL` | Customer origin allowed by CORS |
| `ADMIN_FRONTEND_URL` | Admin origin allowed by CORS |

## Frontend environment variables

| Application | Variable | Purpose |
| --- | --- | --- |
| Customer Web App | `NEXT_PUBLIC_BACKEND_URL` | API gateway base URL |
| Customer Web App | `NEXT_PUBLIC_TENANT_ID` | Tenant whose menu is displayed |
| Admin Dashboard | `NEXT_PUBLIC_API_GATEWAY_URL` | API gateway base URL |

Only public gateway URLs and tenant identifiers belong in `NEXT_PUBLIC_*`
variables. Secrets must remain in backend/App Service or provider-managed
secret configuration.

## Operational notes

All backend services expose `/api/v1/health` routes; Catalog and Order also
ship live/readiness checks and OpenAPI/Swagger endpoints. Container
healthchecks call the live endpoint. Database migrations are managed with
Drizzle commands from each backend repository and should be run with the
appropriate database connection before or during a controlled deployment.
