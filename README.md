# Nesta Foods

Nesta Foods is a multi-tenant restaurant management platform organized as five
independently deployable GitHub repositories: three Hono/TypeScript
microservices and two Next.js applications. Auth runs on Bun; Catalog and
Order use the Node.js runtime in their production containers.

The customer storefront is live at
[nesta-foods.vercel.app](https://nesta-foods.vercel.app/). The admin dashboard
is configured as `https://nesta-admin.vercel.app` in the backend CORS
configuration.

## What it provides

- Customer registration, sign-in, session refresh, profile and address
  management.
- Tenant-aware menu browsing with categories, products, variants and add-ons.
- Customer cart and checkout flows with order history and cancellation.
- Admin and manager workflows for users, tenants, catalog availability and
  order management.
- Role-based access control for `admin`, `manager`, `staff` and `customer`
  roles.
- Product image upload support through Cloudinary.
- Health endpoints and OpenAPI/Swagger documentation in the catalog and order
  services.

## Technology stack

| Area | Technologies |
| --- | --- |
| Backend | Bun, TypeScript, Hono, Zod, Drizzle ORM |
| Frontend | Next.js 16, React 19, TypeScript, Axios, TanStack Query, Zustand |
| Data | PostgreSQL-compatible databases; Neon serverless PostgreSQL is used by catalog and order |
| Delivery | Docker, GitHub Actions, GitHub Container Registry, Azure App Service |
| Edge and hosting | Azure API Management (gateway); Vercel-hosted Next.js applications |
| Media | Cloudinary for catalog product images |

## Repositories

| Component | Repository |
| --- | --- |
| Customer Web App | [microservice-client-fe](https://github.com/PunitMudgal/microservice-client-fe) |
| Admin Dashboard | [microservice-admin-fe](https://github.com/PunitMudgal/microservice-admin-fe) |
| Auth Service | [auth-service](https://github.com/PunitMudgal/auth-service) |
| Catalog Service | [catalog-service](https://github.com/PunitMudgal/catalog-service) |
| Order Service | [order-service](https://github.com/PunitMudgal/order-service) |

## How the repositories work together

Both frontends send credentialed HTTP requests to the configured gateway URL.
They select a backend by path:

- `/auth/api/v1` → Auth Service
- `/catalog/api/v1` → Catalog Service
- `/order/api/v1` → Order Service

The Auth Service issues and refreshes the access/refresh-token session and
provides the JWT claims used by the other services. The Catalog Service owns
tenant menus and product data. When an order is created, the Order Service
calls the Catalog Service directly to validate the selected products, variants
and add-ons and snapshots their names and prices into the order. The
frontends then use the order APIs for customer history or dashboard
operations.

## Local development

Each repository is developed and deployed independently. Install the package
manager expected by that repository, copy its environment variables into a
local environment file, and follow its README:

```sh
# Auth Service
bun install && bun run dev

# Catalog and Order Services
pnpm install && pnpm run dev

# Either Next.js application
bun install && bun run dev
```

The services require database and authentication configuration before they can
serve real requests. The frontend applications require a gateway URL and,
for the customer app, a tenant ID. See
[Deployment and configuration](docs/deployment.md) for the verified variable
names and deployment flow.

## Documentation

- [Architecture](docs/architecture.md)
- [Deployment and configuration](docs/deployment.md)

## Verification notes

The application repositories contain the service implementations and CI/CD
workflows, but no Azure API Management policy or infrastructure-as-code
definition. The gateway route mapping documented here is therefore based on
the paths consumed by both frontends; APIM resource names, policies and
backends must be maintained in the Azure environment.