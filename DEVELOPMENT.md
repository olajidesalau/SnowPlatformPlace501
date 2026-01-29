# Development Setup Guide

## Project Structure After Azure Restructuring

```
SnowWplace201/
├── .azure/                          # AZD environment state (generated)
├── .github/
│   ├── workflows/
│   │   ├── azure-deploy.yml        # CI/CD pipeline for Azure
│   │   └── docker-test.yml         # Docker build tests
│   └── copilot-instructions.md     # Original project guidelines
│
├── infra/                          # Infrastructure as Code (Bicep)
│   ├── main.bicep                  # Main infrastructure template
│   ├── abbreviations.json          # Azure resource naming
│   └── core/
│       ├── infrastructure.bicep    # ACR, Container App Env, Log Analytics
│       ├── database.bicep          # Azure SQL Database
│       ├── storage.bicep           # Azure Blob Storage
│       └── app.bicep               # Container App deployment
│
├── scripts/
│   └── setup-db.ps1                # Database initialization script
│
├── src/
│   ├── index.tsx                   # Original Cloudflare app
│   ├── azure-index.tsx             # New Azure SQL + Storage version
│   ├── renderer.tsx
│   ├── routes.ts
│   ├── components/
│   └── pages/
│
├── migrations/                      # SQL migration files
│   └── 0001_initial_schema.sql    # Schema migrations for Azure SQL
│
├── public/
│   └── static/                      # Static assets
│
├── azure.yaml                       # AZD configuration
├── docker-compose.yml               # Local dev environment
├── Dockerfile                       # Container image definition
├── package.json                     # Updated with Azure dependencies
├── tsconfig.json
├── vite.config.ts
├── wrangler.jsonc                   # Kept for reference (Cloudflare config)
│
├── QUICK_START_AZURE.md             # 5-step deployment guide
├── AZURE_DEPLOYMENT.md              # Comprehensive deployment guide
├── MIGRATION_GUIDE.md               # Cloudflare to Azure migration
├── DEVELOPMENT.md                   # This file
│
├── .env.local.example               # Local Docker environment
├── .env.dev.example                 # Development Azure environment
└── .env.prod.example                # Production Azure environment
```

## Development Workflows

### Option 1: Local Docker Development (Recommended for Testing)

**Best for:** Testing with real SQL Server and Storage, close to production

```bash
# Start local environment
docker-compose up -d

# Check services
docker-compose ps

# View logs
docker-compose logs -f

# Stop when done
docker-compose down
```

**Services started:**
- SQL Server on localhost:1433
- Azurite (Storage Emulator) on localhost:10000
- Application on localhost:3000

### Option 2: Azure Development (Recommended for Integration)

**Best for:** Testing against real Azure services, full CI/CD testing

```bash
# Create dev environment
azd env new dev

# Deploy to Azure
azd up

# Monitor
azd monitor

# Deploy code changes
azd deploy

# Cleanup when done
azd down
```

### Option 3: Cloudflare Development (Original, for reference)

**Best for:** Quick local testing, legacy support

```bash
# Run original Hono app
npm run dev

# Or with D1
npm run dev:d1
```

---

## Setup Instructions

### 1. Prerequisites

Install required tools:

```bash
# Node.js 20+
node --version

# npm
npm --version

# Docker & Docker Compose
docker --version
docker-compose --version

# Azure CLI
az --version

# Azure Developer CLI
azd version
```

### 2. Clone & Install

```bash
git clone <repo>
cd SnowWplace201
npm install
```

### 3. Local Development Setup

Copy environment template:

```bash
cp .env.local.example .env.local
```

Edit `.env.local` if needed (defaults work for Docker Compose):

```env
DATABASE_URL=Server=mssql,1433;Database=snowplatformdata;User Id=SA;Password=YourPassword123!;...
STORAGE_ACCOUNT_NAME=devstoreaccount1
JWT_SECRET=local-dev-secret
```

---

## Running the Application

### Using Docker Compose

```bash
# Start everything
docker-compose up -d

# View status
docker-compose ps

# View logs
docker-compose logs -f app

# Access app
curl http://localhost:3000/health

# Stop
docker-compose down
```

### Using Azure (Dev Environment)

```bash
# First time
azd init --template .
azd up

# Subsequent deploys
azd deploy

# Monitor
azd monitor

# View logs
az containerapp logs show --name snowplatform --resource-group <rg> --follow
```

### Using Original Cloudflare Setup (Reference)

```bash
npm run dev          # Development mode
npm run dev:d1       # With D1 database
npm run build        # Build for production
npm run deploy       # Deploy to Cloudflare
```

---

## Database Management

### Local SQL Server (Docker)

```bash
# Connect to local SQL
sqlcmd -S localhost -U SA -P 'YourPassword123!' -d snowplatformdata

# Create database
CREATE DATABASE snowplatformdata;

# Apply migrations
sqlcmd -S localhost -U SA -P 'YourPassword123!' -i migrations/0001_initial_schema.sql -d snowplatformdata
```

### Azure SQL Database

```bash
# Connect to Azure SQL
sqlcmd -S <server>.database.windows.net -U sqladmin -P '<password>' -d snowplatformdata

# Or use Azure CLI
az sql db show --resource-group <rg> --server <server> --name snowplatformdata

# Apply migrations via script
pwsh scripts/setup-db.ps1
```

### View/Edit Migrations

Migrations are SQL files in `migrations/` directory:

```bash
# Create new migration
cat > migrations/0002_add_bookings_table.sql << 'EOF'
CREATE TABLE bookings (
    id UNIQUEIDENTIFIER DEFAULT NEWID() PRIMARY KEY,
    user_id UNIQUEIDENTIFIER FOREIGN KEY REFERENCES users(id),
    provider_id UNIQUEIDENTIFIER FOREIGN KEY REFERENCES users(id),
    service_id NVARCHAR(255) NOT NULL,
    scheduled_date DATETIME2 NOT NULL,
    status NVARCHAR(50) DEFAULT 'pending',
    created_at DATETIME2 DEFAULT GETUTCDATE()
);
EOF

# Apply new migration
sqlcmd -S <server> -U <user> -P '<password>' -i migrations/0002_add_bookings_table.sql
```

---

## Code Changes & Deployment

### Making Code Changes

**For API/Routes:**
- Edit `src/azure-index.tsx` (new Azure version)
- Or `src/index.tsx` (original Cloudflare version)

**For Pages/UI:**
- Edit files in `src/pages/`
- Edit files in `src/components/`

**For Static Assets:**
- Add to `public/static/`

### Building

```bash
# Build for Azure
npm run build

# Build Docker image
npm run docker:build

# Build for Azure with Azure-specific config
npm run docker:build:azure
```

### Local Testing (Docker)

```bash
# Rebuild and restart
docker-compose down
npm run docker:build
docker-compose up -d

# View logs
docker-compose logs -f app
```

### Azure Testing

```bash
# Deploy code changes
azd deploy

# Or rebuild image and deploy
npm run docker:build:azure
docker tag snowplatform:azure <acr>.azurecr.io/snowplatform:latest
docker push <acr>.azurecr.io/snowplatform:latest
az containerapp update --name snowplatform --resource-group <rg> --image ...

# Monitor
azd monitor
```

---

## Application Architecture

### Database Layer

**Using Azure SQL:**
```typescript
import sql from 'mssql'

const pool = new sql.ConnectionPool(connectionString)
const result = await pool.request()
  .input('email', sql.VarChar, email)
  .query('SELECT * FROM users WHERE email = @email')
```

**vs. Original D1:**
```typescript
const result = await env.DB.prepare('SELECT * FROM users WHERE email = ?').bind(email).first()
```

### Storage Layer

**Using Azure Blob Storage:**
```typescript
import { BlobServiceClient } from '@azure/storage-blob'

const containerClient = blobServiceClient.getContainerClient('uploads')
await containerClient.uploadBlockBlob(name, data)
```

### API Endpoints

Both versions support same endpoints:
- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/users/:userId`
- `PUT /api/users/:userId`
- `POST /api/upload/profile-picture/:userId`
- `GET /api/providers/search`

---

## Debugging

### Enable Debug Logging

Set environment variable:
```bash
LOG_LEVEL=debug
NODE_ENV=development
```

### View Detailed Logs

Docker:
```bash
docker-compose logs -f app
```

Azure:
```bash
azd monitor
az containerapp logs show --name snowplatform --resource-group <rg> --follow
```

### Debug Node.js Application

```bash
# Start with debug flag
node --inspect-brk dist/index.js

# Connect VS Code debugger to localhost:9229
```

---

## Testing

### Unit Tests

```bash
# Add test files to src/tests/
# Update package.json with test script
npm test
```

### Integration Tests

```bash
# Test with local Docker
npm run docker:build
docker-compose up -d
npm run test:integration
```

### End-to-End Tests

```bash
# Deploy to Azure dev environment
azd up

# Run E2E tests against deployment
npm run test:e2e
```

---

## Troubleshooting

### Docker Issues

```bash
# Container won't start
docker logs snowplatform-app

# Permission denied
sudo chmod 666 /var/run/docker.sock

# Network issues
docker-compose down -v
docker-compose up -d
```

### Database Issues

```bash
# Can't connect to SQL
docker ps  # Verify mssql container running
docker logs snowplatform-mssql

# SQL query errors
sqlcmd -S localhost -U SA -P 'YourPassword123!' -Q "SELECT @@VERSION"
```

### Azure Deployment Issues

```bash
# Container won't start
az containerapp revision list --name snowplatform --resource-group <rg>
az containerapp logs show --name snowplatform --resource-group <rg> --follow

# Database connection
az sql server firewall-rule list --server <server> --resource-group <rg>

# Storage access
az storage account show-connection-string --name <name> --resource-group <rg>
```

---

## Environment Management

### Azure Environments

```bash
# List
azd env list

# Create new
azd env new staging

# Select
azd env select dev

# Remove
azd env remove staging
```

### Local Environments

Use `.env.local`, `.env.dev`, `.env.prod` files:

```bash
# Local with Docker
cp .env.local.example .env.local
docker-compose up

# Development Azure
cp .env.dev.example .env.dev
export $(cat .env.dev | xargs)
azd up

# Production Azure
cp .env.prod.example .env.prod
export $(cat .env.prod | xargs)
azd env select prod
azd up
```

---

## Performance Optimization

### Database

```sql
-- Index frequently searched columns
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_bookings_user_id ON bookings(user_id);
```

### Container

Update in `infra/core/app.bicep`:

```bicep
cpu: '0.5'        # CPU cores
memory: '1Gi'     # Memory allocation
minReplicas: 2    # Minimum instances
maxReplicas: 10   # Maximum instances
```

### Storage

Use Azure Storage lifecycle policies:

```bash
az storage account management-policy create \
  --account-name <storage> \
  --resource-group <rg> \
  --policy policy.json
```

---

## Security

### Local Development

```bash
# Change default passwords in docker-compose.yml
# Don't commit .env.local to git
echo ".env.local" >> .gitignore
echo ".env.dev" >> .gitignore
echo ".env.prod" >> .gitignore
```

### Azure Deployment

```bash
# Use Key Vault for secrets
az keyvault secret set --vault-name <kv> --name jwt-secret --value <secret>

# Don't hardcode credentials
# Use managed identities instead

# Enable firewall rules
az sql server firewall-rule create \
  --resource-group <rg> \
  --server <server> \
  --name AllowApp \
  --start-ip-address <ip> \
  --end-ip-address <ip>
```

---

## Next Steps

1. **Local Development**: Start with `docker-compose up`
2. **Deploy to Dev**: Use `azd env new dev && azd up`
3. **Integrate CI/CD**: See `.github/workflows/`
4. **Production**: Create prod environment and deploy
5. **Monitoring**: Set up alerts in Azure Portal

---

## Useful Links

- [QUICK_START_AZURE.md](./QUICK_START_AZURE.md) - 5-minute quick start
- [AZURE_DEPLOYMENT.md](./AZURE_DEPLOYMENT.md) - Comprehensive guide
- [MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md) - Cloudflare to Azure
- [Azure Docs](https://learn.microsoft.com/azure/)
- [Docker Docs](https://docs.docker.com/)
