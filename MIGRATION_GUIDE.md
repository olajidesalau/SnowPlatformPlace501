# Azure Mobile App Migration Guide

## From Cloudflare to Azure

This guide details the migration path from Cloudflare Pages + D1 to Azure Container Apps + SQL Database + Storage Account.

## Key Changes

### 1. Runtime Environment

| Aspect | Cloudflare | Azure |
|--------|-----------|-------|
| **Compute** | Pages/Workers | Container Apps |
| **Database** | D1 (SQLite) | Azure SQL Database |
| **Storage** | R2 (S3-like) | Blob Storage |
| **Deployment** | `wrangler` | `azd` / Azure CLI |
| **Containerization** | Not required | Docker required |

### 2. Configuration Files

#### Removed (Cloudflare-specific)
- `wrangler.jsonc` - No longer needed (keeping for reference)
- `.wrangler/` - Local D1 state directory

#### Added (Azure-specific)
- `azure.yaml` - AZD configuration
- `Dockerfile` - Container image definition
- `infra/` - Bicep Infrastructure as Code
- `scripts/setup-db.ps1` - Database setup script
- `.azure/` - AZD environment state

### 3. Environment Variables

#### Cloudflare (D1 Binding)
```typescript
// Direct D1 binding
type Bindings = {
  DB: D1Database
}

const result = await env.DB.prepare('SELECT ...').all()
```

#### Azure (Connection String)
```typescript
// Connection string to SQL Server
const connectionString = process.env.DATABASE_URL
const pool = new sql.ConnectionPool(connectionString)
```

### 4. File Upload Changes

#### Cloudflare (R2)
```typescript
// Not implemented in current version
```

#### Azure (Blob Storage)
```typescript
// Using @azure/storage-blob
const blobServiceClient = BlobServiceClient.fromConnectionString(connectionString)
const containerClient = blobServiceClient.getContainerClient('uploads')
await containerClient.uploadBlockBlob(name, data)
```

## Migration Checklist

- [ ] Read this guide and AZURE_DEPLOYMENT.md
- [ ] Install Azure CLI and AZD
- [ ] Create Azure subscription
- [ ] Update dependencies in package.json
- [ ] Create Dockerfile
- [ ] Create Bicep infrastructure files
- [ ] Update application code to use SQL + Blob Storage
- [ ] Test locally with Docker and SQL Server (local)
- [ ] Initialize Azure environment (azd init)
- [ ] Provision Azure resources (azd infra create)
- [ ] Build and push Docker image
- [ ] Deploy to Azure Container Apps (azd deploy)
- [ ] Run database migrations
- [ ] Load seed data
- [ ] Test endpoints
- [ ] Configure monitoring and alerts
- [ ] Set up CI/CD pipeline
- [ ] Decommission Cloudflare deployment

## Code Migration Examples

### 1. Database Access

**Before (Cloudflare D1):**
```typescript
const result = await env.DB.prepare('SELECT * FROM users WHERE email = ?').bind(email).first()
```

**After (Azure SQL):**
```typescript
import sql from 'mssql'

const pool = new sql.ConnectionPool(connectionString)
const request = pool.request()
const result = await request
  .input('email', sql.VarChar, email)
  .query('SELECT * FROM users WHERE email = @email')

const user = result.recordset[0]
```

### 2. File Uploads

**Before (Not implemented):**
```typescript
// No file upload support
```

**After (Azure Blob Storage):**
```typescript
import { BlobServiceClient } from '@azure/storage-blob'

const blobServiceClient = BlobServiceClient.fromConnectionString(connectionString)
const containerClient = blobServiceClient.getContainerClient('uploads')
const blockBlobClient = containerClient.getBlockBlobClient(fileName)
await blockBlobClient.upload(data, data.length)
```

### 3. Configuration

**Before (wrangler.jsonc):**
```jsonc
{
  "name": "snowplatform",
  "main": "src/index.tsx",
  "d1_databases": [{
    "binding": "DB",
    "database_name": "snowplatformdata101",
    "database_id": "b424f388-e4d7-4fd0-a656-8a0fc230dcb5"
  }]
}
```

**After (azure.yaml):**
```yaml
name: snowplatform
services:
  web:
    project: .
    language: ts
    host: containerapp
    docker:
      path: ./Dockerfile
    env:
      - name: DATABASE_URL
        value: ${AZURE_SQL_CONNECTION_STRING}
      - name: STORAGE_ACCOUNT_NAME
        value: ${STORAGE_ACCOUNT_NAME}
```

## Infrastructure Comparison

### Cloudflare Architecture
```
Wrangler Pages Dev
    ↓
Vite Build (dist/)
    ↓
Wrangler Pages Deploy
    ↓
Cloudflare Pages + Workers
    ↓
D1 Database (SQLite)
```

### Azure Architecture
```
Docker Build
    ↓
Azure Container Registry (ACR)
    ↓
AZD Deploy
    ↓
Container Apps Environment
    ↓
Azure SQL Database + Blob Storage
```

## Performance Considerations

### Database

**D1 (SQLite)**
- Single-threaded
- File-based
- Limited to one connection at a time
- Good for: Prototypes, low-traffic apps

**Azure SQL**
- Multi-threaded
- Connection pooling
- ACID compliance
- Scaling: DTU-based (Standard S1-S12, Premium P1-P11)
- Good for: Production apps, high-traffic, complex queries

### Storage

**R2 (Cloudflare)**
- S3-compatible API
- Global distribution
- Low-cost

**Azure Blob Storage**
- Separate compute/storage billing
- Multiple access tiers (Hot/Cool/Archive)
- Better for mixed workloads
- Lifecycle management policies

## Cost Comparison

### Monthly Estimates (US East)

| Service | Cloudflare | Azure |
|---------|-----------|-------|
| Compute | $0 (Pages) | ~$30-60 (Container Apps) |
| Database | ~$0 (D1 included) | ~$20-40 (SQL S1) |
| Storage | ~$1-5 (R2) | ~$5-20 (Blob) |
| Bandwidth | Variable | Variable |
| **Total** | ~$1-5 | ~$55-120 |

*Actual costs vary based on usage. Use Azure pricing calculator for estimates.*

## Local Development Setup

### Azure Emulation

```bash
# Install Azure Storage Emulator
# OR use Docker
docker run -p 10000:10000 -p 10001:10001 -p 10002:10002 mcr.microsoft.com/azure-storage/azurite

# Install SQL Server Express locally
# Download from: https://www.microsoft.com/sql-server/sql-server-downloads

# Or use Docker
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourPassword123!" -p 1433:1433 mcr.microsoft.com/mssql/server:latest
```

### Local Testing

```bash
# Build local Docker image
npm run docker:build

# Run with local SQL and Storage
docker-compose up -d

# Test application
curl http://localhost:3000/health
```

## Deployment Timelines

### First-time Setup
- Prerequisites installation: 30 minutes
- Learning Bicep/AZD: 1-2 hours
- Infrastructure provisioning: 10-15 minutes
- Application deployment: 5-10 minutes
- Testing & verification: 15-20 minutes
- **Total: 2-4 hours**

### Subsequent Deployments
- Code changes: Varies
- Build Docker image: 3-5 minutes
- Push to ACR: 1-2 minutes
- Deploy to Container Apps: 1-2 minutes
- **Total: 5-10 minutes**

## Rollback Procedures

### Rollback Last Deployment

```bash
# List revisions
az containerapp revision list --name snowplatform --resource-group <rg-name>

# Activate previous revision
az containerapp revision activate \
  --resource-group <rg-name> \
  --name snowplatform \
  --revision <previous-revision-name>
```

### Database Rollback

```bash
# SQL Server has automated backups
# Restore database to point-in-time
az sql db restore \
  --resource-group <rg-name> \
  --server <server-name> \
  --name snowplatformdata \
  --time <timestamp> \
  --dest-name snowplatformdata-restored

# Verify and switch
```

## Next Steps

1. **Review Architecture**: Study the Bicep files in `infra/`
2. **Local Testing**: Run Docker locally with SQL Server
3. **Azure Setup**: Complete AZURE_DEPLOYMENT.md
4. **Production**: Set up monitoring and CI/CD
5. **Optimization**: Monitor costs and performance

## Support Resources

- Azure Migration Guide: https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/
- Bicep Tutorial: https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/
- Container Apps Guide: https://learn.microsoft.com/en-us/azure/container-apps/
- SQL Database Migration: https://learn.microsoft.com/en-us/azure/dms/

## Questions?

Refer to:
- [AZURE_DEPLOYMENT.md](./AZURE_DEPLOYMENT.md) - Detailed deployment guide
- [copilot-instructions.md](./.github/copilot-instructions.md) - Project guidelines
- Azure Documentation - Official Microsoft resources
