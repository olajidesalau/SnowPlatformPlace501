# Azure Mobile App Quick Start

Get SnowPlatform running on Azure in 30 minutes.

## Prerequisites

```bash
# Windows (using winget)
winget install Microsoft.AzureCLI
winget install Microsoft.Azd
winget install Docker.DockerDesktop
winget install OpenJS.NodeJS

# macOS
brew install azure-cli azure-developer-cli docker node

# Verify installations
az --version
azd version
docker --version
node --version
npm --version
```

## Quick Deploy (5 steps)

### 1. Clone & Setup (5 min)

```bash
cd SnowWplace201
npm install
```

### 2. Login to Azure (2 min)

```bash
az login
az account show
```

### 3. Initialize AZD (3 min)

```bash
azd init --template .
# Select location: eastus
# Set environment name: snowplatform-dev
```

### 4. Deploy Infrastructure & App (15 min)

```bash
# This does everything: creates resources, builds, deploys
azd up
```

### 5. Test Deployment (2 min)

```bash
# Get app URL
CONTAINER_APP=$(az containerapp show \
  --name snowplatform \
  --resource-group rg-snowplatform-dev \
  --query properties.configuration.ingress.fqdn -o tsv)

# Test endpoints
curl https://$CONTAINER_APP/health
curl https://$CONTAINER_APP/ready
```

**Done!** Your app is now running on Azure.

---

## Local Development (with Docker)

### Start Local Environment

```bash
# Copy environment file
cp .env.local.example .env.local

# Start services (SQL, Storage, App)
docker-compose up -d

# Wait for services to be healthy
docker-compose ps

# Test local app
curl http://localhost:3000/health
```

### View Logs

```bash
# App logs
docker-compose logs -f app

# Database logs
docker-compose logs -f mssql

# Storage logs
docker-compose logs -f azurite
```

### Stop Services

```bash
docker-compose down

# With data cleanup
docker-compose down -v
```

---

## Common Commands

### Azure Management

```bash
# View all environments
azd env list

# Switch environment
azd env select prod

# Monitor app
azd monitor

# Deploy after code changes
azd deploy

# Delete everything
azd down
```

### Docker

```bash
# Build image
npm run docker:build

# Run image
docker run -p 3000:3000 snowplatform:latest

# Build for Azure
npm run docker:build:azure
```

### Database

```bash
# Apply migrations
pwsh scripts/setup-db.ps1

# Connect to local SQL
sqlcmd -S localhost -U SA -P 'YourPassword123!' -d snowplatformdata

# Connect to Azure SQL
sqlcmd -S <server>.database.windows.net -U sqladmin -P '<password>' -d snowplatformdata
```

---

## Architecture

```
┌─────────────────────────┐
│   Azure Container App   │
│  (snowplatform:latest)  │
└────────┬────────────────┘
         │
    ┌────┴─────┬────────────┐
    │           │            │
┌───▼──┐  ┌────▼────┐  ┌───▼───┐
│ SQL  │  │ Storage │  │Insights│
│ DB   │  │ Blobs   │  │Logs    │
└──────┘  └─────────┘  └────────┘
```

---

## Configuration

### Environment Variables

| Variable | Dev | Prod |
|----------|-----|------|
| `DATABASE_URL` | Local SQL | Azure SQL |
| `STORAGE_ACCOUNT_NAME` | `devstoreaccount1` | Azure account |
| `JWT_SECRET` | Test key | Key Vault |
| `ENVIRONMENT` | `development` | `production` |

See `.env.*.example` files for all options.

---

## Monitoring

### View Logs

```bash
# Application logs
azd monitor

# Container logs
az containerapp logs show --name snowplatform --resource-group <rg> --follow

# Application Insights
az monitor app-insights show --name <name> --resource-group <rg>
```

### Set Alerts

```bash
# High CPU alert
az monitor metrics alert create \
  --name cpu-alert \
  --resource-group <rg> \
  --condition "avg cpu > 80"
```

---

## Troubleshooting

### App won't start

```bash
# Check container status
az containerapp revision list --name snowplatform --resource-group <rg>

# View detailed logs
az containerapp logs show --name snowplatform --resource-group <rg> --follow

# Restart container
az containerapp revision deactivate --name snowplatform --resource-group <rg> --revision <rev>
```

### Database connection error

```bash
# Check SQL firewall
az sql server firewall-rule list --server <server> --resource-group <rg>

# Test connection
sqlcmd -S <server>.database.windows.net -U sqladmin -P '<pwd>' -Q "SELECT 1"
```

### Storage issues

```bash
# List containers
az storage container list --account-name <storage-name>

# Check connection string
az storage account show-connection-string --name <storage-name> --resource-group <rg>
```

---

## Cost Estimation

**Monthly (US East):**
- Container Apps: $30-60
- SQL Database (S1): $20-40
- Storage: $5-20
- **Total: ~$55-120**

Use [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/) for accurate estimates.

---

## Next Steps

1. ✅ **Local Development**: Use `docker-compose up`
2. ✅ **Deploy to Dev**: `azd up` (Environment: dev)
3. ✅ **Production Deployment**: Create prod environment and deploy
4. ✅ **CI/CD Pipeline**: GitHub Actions (see `.github/workflows/`)
5. ✅ **Monitoring Setup**: Configure alerts in Azure Portal

---

## Learn More

- [Full Deployment Guide](./AZURE_DEPLOYMENT.md)
- [Migration from Cloudflare](./MIGRATION_GUIDE.md)
- [Azure Docs](https://learn.microsoft.com/azure/)
- [Container Apps](https://learn.microsoft.com/azure/container-apps/)
- [SQL Database](https://learn.microsoft.com/azure/azure-sql/)

---

## Support

- Issue: Check [troubleshooting section](#troubleshooting)
- Question: Review [AZURE_DEPLOYMENT.md](./AZURE_DEPLOYMENT.md)
- Documentation: See links above
