# Azure Mobile App Deployment Guide

## Overview

This document provides a complete guide to deploying SnowPlatform to Azure using containerized deployment with Azure Container Apps, SQL Database, and Storage Account.

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Azure Resource Group                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────┐    ┌──────────────────────┐         │
│  │  Container Registry  │    │  Container App Env   │         │
│  │  (ACR)               │───▶│  - Log Analytics     │         │
│  │                      │    │  - App Insights      │         │
│  └──────────────────────┘    └──────────────────────┘         │
│           │                           │                        │
│           │                    ┌──────▼──────┐                │
│           │                    │ Container   │                │
│           │                    │ App         │                │
│           │                    │ (snowplat)  │                │
│           │                    └──────┬──────┘                │
│           │                           │                        │
│  ┌────────▼────────────┐    ┌─────────▼──────────┐           │
│  │ Storage Account     │    │ SQL Server         │           │
│  │ - Blob Containers   │    │ - Database         │           │
│  │ - File Shares       │    │ - Firewall Rules   │           │
│  └─────────────────────┘    └────────────────────┘           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Prerequisites

- Azure Subscription with active credits
- Azure CLI (`az`) installed and authenticated
- Azure Developer CLI (`azd`) installed
- Docker installed locally
- Node.js 20+ installed
- PowerShell 7+ (for scripts)

## Installation

### 1. Install Azure Tools

```bash
# Install Azure CLI
# Windows (using winget)
winget install Microsoft.AzureCLI

# macOS
brew install azure-cli

# Install Azure Developer CLI
winget install Microsoft.Azd

# Verify installations
az --version
azd version
```

### 2. Setup Local Development

```bash
# Install dependencies
npm install

# Install Azure SDK packages (already in package.json)
npm install @azure/storage-blob @azure/identity mssql

# Build Docker image
docker build -t snowplatform:latest .
```

## Deployment Steps

### Step 1: Initialize Azure Developer Environment

```bash
# Create new Azure environment
azd init --template .

# This will:
# - Create a new azure.yaml configuration
# - Initialize .azure directory with environment settings
# - Prompt for resource naming and location
```

### Step 2: Configure Environment Variables

Create `.env.local` file:

```env
# Azure Configuration
AZURE_ENV_NAME=snowplatform-dev
AZURE_LOCATION=eastus
AZURE_SUBSCRIPTION_ID=<your-subscription-id>

# Database
DATABASE_URL=Server=<sql-server>.database.windows.net;Database=snowplatformdata;User Id=sqladmin;Password=<password>;Encrypt=True;
SQL_USER=sqladmin
SQL_PASSWORD=<generated-password>

# Storage
STORAGE_ACCOUNT_NAME=stnowplatform<random>
STORAGE_ACCOUNT_KEY=<generated-key>

# Application
JWT_SECRET=<generate-random-secret>
NODE_ENV=production
ENVIRONMENT=production
```

### Step 3: Provision Azure Resources

```bash
# Preview what will be created
azd infra create --preview

# Create infrastructure (Bicep deployment)
azd infra create

# This will deploy:
# - Resource Group
# - Container Registry (ACR)
# - Container App Environment
# - Azure SQL Server & Database
# - Storage Account with containers
# - Log Analytics Workspace
# - Application Insights
```

### Step 4: Build and Push Docker Image

```bash
# Login to Azure Container Registry
az acr login --name <acr-name>

# Build image for Azure
npm run docker:build:azure

# Tag and push to ACR
docker tag snowplatform:azure <acr-name>.azurecr.io/snowplatform:latest
docker push <acr-name>.azurecr.io/snowplatform:latest

# Or use ACR build task
az acr build --registry <acr-name> --image snowplatform:latest .
```

### Step 5: Deploy Application

```bash
# Deploy to Container Apps
azd deploy

# This will:
# - Pull image from ACR
# - Deploy to Container App
# - Configure environment variables
# - Set up scaling policies
# - Configure health checks
```

### Step 6: Verify Deployment

```bash
# Check container app status
az containerapp show --name snowplatform --resource-group <rg-name>

# View application logs
azd monitor

# Test health endpoints
curl https://<container-app-url>/health
curl https://<container-app-url>/ready
```

## Environment-Specific Deployments

### Development Environment

```bash
# Create dev environment
azd env new dev
azd env select dev

# Deploy with development settings
azd up

# Environment variables for dev
AZURE_ENV_NAME=snowplatform-dev
ENVIRONMENT=development
```

### Production Environment

```bash
# Create prod environment
azd env new prod
azd env select prod

# Deploy with production settings
azd up

# Environment variables for prod
AZURE_ENV_NAME=snowplatform-prod
ENVIRONMENT=production
```

## Configuration Reference

### Container App Settings

Located in `infra/core/app.bicep`:

```bicep
CPU: 0.5 cores
Memory: 1 Gi
Min Replicas: 2
Max Replicas: 10
Auto-scaling: Based on HTTP requests
```

### SQL Database Configuration

- **Edition**: Standard
- **Size**: S1 (20 DTU)
- **Backup**: Automatic (35 days retention)
- **Encryption**: TLS 1.2 required

### Storage Account Configuration

- **Tier**: Standard
- **Replication**: LRS (Locally Redundant Storage)
- **Containers**:
  - `uploads` - Private uploads
  - `images` - Public profile images
  - `documents` - Private documents

## Database Setup

### Apply Migrations

```bash
# Create migrations directory if not exists
mkdir -p migrations

# Create migration file
cat > migrations/0001_initial_schema.sql << 'EOF'
CREATE TABLE users (
    id UNIQUEIDENTIFIER DEFAULT NEWID() PRIMARY KEY,
    email NVARCHAR(255) UNIQUE NOT NULL,
    password_hash NVARCHAR(MAX) NOT NULL,
    full_name NVARCHAR(255) NOT NULL,
    role NVARCHAR(50) NOT NULL DEFAULT 'customer',
    phone NVARCHAR(20),
    postcode NVARCHAR(10),
    profile_picture_url NVARCHAR(MAX),
    bio NVARCHAR(500),
    rating DECIMAL(3,2),
    created_at DATETIME2 DEFAULT GETUTCDATE()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
EOF

# Run migrations using PowerShell script
pwsh scripts/setup-db.ps1
```

### Seed Data

```bash
# Create seed file
cat > seed.sql << 'EOF'
-- Add initial data as needed
EOF

# Execute seed
sqlcmd -S <server>.database.windows.net -d snowplatformdata -U sqladmin -P '<password>' -i seed.sql
```

## Monitoring and Logs

### View Logs

```bash
# Stream application logs
azd monitor

# View specific container app logs
az containerapp logs show --name snowplatform --resource-group <rg-name> --follow

# View Application Insights
az monitor app-insights show --name <app-insights-name> --resource-group <rg-name>
```

### Set Alerts

```bash
# Create CPU alert
az monitor metrics alert create \
  --name cpu-alert \
  --resource-group <rg-name> \
  --scopes /subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.App/containerApps/snowplatform \
  --condition "avg cpu > 80" \
  --window-size 5m \
  --evaluation-frequency 1m

# Create HTTP 5xx alert
az monitor metrics alert create \
  --name http-errors-alert \
  --resource-group <rg-name> \
  --scopes /subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.App/containerApps/snowplatform \
  --condition "total http_errors_5xx > 10" \
  --window-size 5m \
  --evaluation-frequency 1m
```

## Scaling Configuration

### Manual Scaling

```bash
# Update revision with new replica count
az containerapp revision update \
  --name snowplatform \
  --resource-group <rg-name> \
  --scale-min 2 \
  --scale-max 10
```

### Auto-scaling Rules

Default configuration includes:
- **CPU-based**: Scale up when CPU > 70%
- **Memory-based**: Scale up when memory > 80%
- **Request-based**: Scale up when concurrent requests > 100

## Security Best Practices

### 1. Secret Management

Store secrets in Azure Key Vault:

```bash
# Create Key Vault
az keyvault create --name <kv-name> --resource-group <rg-name>

# Store secrets
az keyvault secret set --vault-name <kv-name> --name jwt-secret --value <secret>
az keyvault secret set --vault-name <kv-name> --name sql-password --value <password>

# Reference in Container App
# Update app.bicep to use Key Vault reference
```

### 2. Network Security

```bash
# Restrict SQL firewall to Azure services only
az sql server firewall-rule update \
  --resource-group <rg-name> \
  --server <sql-server-name> \
  --name AllowAzureServices

# Enable VNet integration
az containerapp update \
  --name snowplatform \
  --resource-group <rg-name> \
  --vnet-id <vnet-id>
```

### 3. Container Registry Access

```bash
# Use managed identity instead of admin credentials
az containerapp identity assign \
  --resource-group <rg-name> \
  --name snowplatform \
  --system-assigned

# Assign pull role on ACR
az role assignment create \
  --assignee <principal-id> \
  --role acrpull \
  --scope <acr-id>
```

## Cost Optimization

### 1. Right-size Resources

```bicep
# Reduce CPU/Memory for dev/test
CPU: 0.25 cores
Memory: 0.5 Gi
Min Replicas: 1
```

### 2. Use Spot Instances (if available)

```bicep
# Container Apps Spot pricing
# 40-50% cost reduction
```

### 3. Storage Optimization

```bash
# Use lifecycle management for old blobs
az storage account blob-service-properties update \
  --account-name <storage-name> \
  --enable-last-access-tracking true

# Archive old data
# Move blobs > 90 days to Archive tier
```

## Troubleshooting

### Container App won't start

```bash
# Check revision status
az containerapp revision list --name snowplatform --resource-group <rg-name>

# View detailed logs
az containerapp logs show --name snowplatform --resource-group <rg-name> --follow

# Check environment variables
az containerapp show --name snowplatform --resource-group <rg-name> --query properties.template
```

### Database connection issues

```bash
# Verify firewall rules
az sql server firewall-rule list --resource-group <rg-name> --server <sql-server>

# Test connection
sqlcmd -S <server>.database.windows.net -U sqladmin -P '<password>' -Q "SELECT 1"

# Check connection string format
# Server=tcp:<server>.database.windows.net,1433;Initial Catalog=<db>;User ID=sqladmin;Password=<pwd>;Encrypt=True;
```

### Storage access issues

```bash
# List storage accounts
az storage account list --resource-group <rg-name>

# Get connection string
az storage account show-connection-string --name <storage-name> --resource-group <rg-name>

# Test blob access
az storage container list --account-name <storage-name>
```

## Cleanup

### Remove All Resources

```bash
# Delete entire resource group (removes all resources)
az group delete --name <rg-name> --yes --no-wait

# Or use azd down
azd down --force
```

### Remove Specific Resources

```bash
# Delete container app
az containerapp delete --name snowplatform --resource-group <rg-name>

# Delete SQL database
az sql db delete --resource-group <rg-name> --server <sql-server> --name snowplatformdata

# Delete storage account
az storage account delete --name <storage-name> --resource-group <rg-name>
```

## CI/CD Integration

See [CICD_PIPELINE.md](./CICD_PIPELINE.md) for GitHub Actions workflow configuration for automated deployments.

## Additional Resources

- [Azure Container Apps Documentation](https://learn.microsoft.com/azure/container-apps/)
- [Azure SQL Database Documentation](https://learn.microsoft.com/azure/azure-sql/)
- [Azure Storage Documentation](https://learn.microsoft.com/azure/storage/)
- [Bicep Documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
- [Azure Developer CLI Documentation](https://learn.microsoft.com/azure/developer/azure-developer-cli/)

## Support

For issues or questions:
1. Check logs: `azd monitor`
2. Review Azure Portal
3. Check resource health
4. Consult Azure documentation
