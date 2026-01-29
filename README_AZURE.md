# SnowPlatform - Azure Mobile App Edition

A complete service marketplace platform restructured for Azure Cloud deployment with Container Apps, SQL Database, and Blob Storage.

## 🚀 Quick Start

Get running in 5 minutes:

```bash
npm install
azd init --template .
azd up
```

See [QUICK_START_AZURE.md](./QUICK_START_AZURE.md) for detailed steps.

## 📋 What's Changed

### From Cloudflare to Azure

| Component | Before | After |
|-----------|--------|-------|
| **Compute** | Cloudflare Pages/Workers | Azure Container Apps |
| **Database** | D1 (SQLite) | Azure SQL Database |
| **Storage** | R2 | Azure Blob Storage |
| **Deployment** | `wrangler deploy` | `azd up` / GitHub Actions |
| **Containerization** | Optional | Required (Docker) |

### New Infrastructure Files

```
infra/                          # Bicep Infrastructure as Code
├── main.bicep                 # Main orchestration
├── core/
│   ├── infrastructure.bicep   # ACR, App Environment, Monitoring
│   ├── database.bicep         # Azure SQL Server & Database
│   ├── storage.bicep          # Blob Storage with containers
│   └── app.bicep              # Container App deployment
```

### New Configuration Files

- **`azure.yaml`** - Azure Developer CLI configuration
- **`Dockerfile`** - Container image definition
- **`docker-compose.yml`** - Local development environment
- **`.github/workflows/`** - CI/CD pipelines for Azure

### Updated Application Code

- **`src/azure-index.tsx`** - New version using SQL Server + Blob Storage
- **`src/index.tsx`** - Original Cloudflare version (reference)

## 📚 Documentation

Start with one of these based on your needs:

| Guide | Purpose | Time |
|-------|---------|------|
| [QUICK_START_AZURE.md](./QUICK_START_AZURE.md) | Deploy in 5 minutes | 5 min |
| [AZURE_DEPLOYMENT.md](./AZURE_DEPLOYMENT.md) | Comprehensive deployment | 30 min |
| [MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md) | Cloudflare → Azure migration | 20 min |
| [DEVELOPMENT.md](./DEVELOPMENT.md) | Local development setup | 15 min |

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│        Azure Resource Group                     │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────┐          ┌─────────────────┐ │
│  │ Container    │          │ Container Apps  │ │
│  │ Registry     │─────────▶│ Environment     │ │
│  └──────────────┘          └────────┬────────┘ │
│                                     │          │
│                            ┌────────▼────────┐ │
│                            │ Application     │ │
│                            │ (Node.js)       │ │
│                            └────────┬────────┘ │
│                                     │          │
│  ┌──────────────┐    ┌──────────────▼────────┐ │
│  │ SQL Server   │    │ Storage Account       │ │
│  │ Database     │    │ - Blob Containers    │ │
│  │ - users      │    │ - File Shares        │ │
│  │ - bookings   │    │                      │ │
│  └──────────────┘    └──────────────────────┘ │
│                                                 │
│  ┌──────────────────────────────────────────┐ │
│  │ Monitoring & Logging                     │ │
│  │ - Application Insights                   │ │
│  │ - Log Analytics Workspace                │ │
│  └──────────────────────────────────────────┘ │
│                                                 │
└─────────────────────────────────────────────────┘
```

## 🛠️ Development

### Local Setup (with Docker)

```bash
docker-compose up -d
curl http://localhost:3000/health
```

Services:
- 🗄️ SQL Server on `localhost:1433`
- 💾 Azure Storage Emulator on `localhost:10000`
- 🌐 App on `localhost:3000`

See [DEVELOPMENT.md](./DEVELOPMENT.md) for details.

### Azure Deployment

```bash
azd env new dev
azd up
azd monitor
```

Environments:
- `dev` - Development
- `staging` - Staging
- `prod` - Production

## 📦 Project Structure

```
SnowWplace201/
├── .azure/                          # AZD environment state
├── .github/workflows/               # CI/CD pipelines
│   ├── azure-deploy.yml            # Azure deployment
│   └── docker-test.yml             # Docker testing
│
├── infra/                          # Infrastructure as Code
│   ├── main.bicep
│   ├── abbreviations.json
│   └── core/
│       ├── infrastructure.bicep
│       ├── database.bicep
│       ├── storage.bicep
│       └── app.bicep
│
├── src/
│   ├── index.tsx                   # Original app
│   ├── azure-index.tsx             # Azure version
│   ├── components/
│   ├── pages/
│   └── ...
│
├── migrations/                      # SQL migrations
├── scripts/                         # Setup scripts
├── public/                          # Static assets
│
├── azure.yaml                       # AZD config
├── Dockerfile                       # Container image
├── docker-compose.yml               # Local dev
├── package.json                     # Updated dependencies
│
├── QUICK_START_AZURE.md
├── AZURE_DEPLOYMENT.md
├── MIGRATION_GUIDE.md
├── DEVELOPMENT.md
└── README.md                        # This file
```

## 🔧 Technologies

### Backend
- **Hono** - Web framework
- **Node.js** - Runtime
- **TypeScript** - Language
- **mssql** - SQL Server driver
- **@azure/storage-blob** - Blob storage client

### Infrastructure
- **Bicep** - Infrastructure as Code
- **Docker** - Containerization
- **Azure Container Apps** - Serverless containers
- **Azure SQL Database** - Relational database
- **Azure Blob Storage** - Object storage
- **Azure Monitor** - Observability

### DevOps
- **Azure Developer CLI (azd)** - Development orchestration
- **GitHub Actions** - CI/CD pipelines
- **Docker Compose** - Local development

## 📋 Features

### User Management
- User registration and authentication
- Profile management with picture uploads
- Role-based access (customer, provider, admin)

### Provider Services
- Provider search and discovery
- Service offerings and availability
- Rating and reviews

### Bookings & Messaging
- Service booking system
- Real-time notifications
- Messaging between users

### Admin Dashboard
- User management
- Provider verification
- Booking management
- Platform analytics

## 🚀 Deployment

### First-time Deployment

1. **Setup** (5 min)
   ```bash
   npm install
   azd init --template .
   ```

2. **Deploy** (15 min)
   ```bash
   azd up
   ```

3. **Verify** (5 min)
   ```bash
   azd monitor
   ```

### Update Deployment

```bash
azd deploy    # Deploy code changes
azd up        # Full redeploy with infra
```

See [AZURE_DEPLOYMENT.md](./AZURE_DEPLOYMENT.md) for comprehensive guide.

## 🔐 Security

### Development
- SQLite D1 for local testing
- Basic JWT authentication
- CORS enabled for development

### Production
- Azure SQL with encryption
- Azure Key Vault for secrets
- TLS 1.2+ for all connections
- Firewall rules on SQL Server
- Managed identity for app authentication
- HTTPS enforcement

## 📊 Monitoring

### Built-in
- Application Insights integration
- Log Analytics Workspace
- Container App metrics
- Health check endpoints

### Commands
```bash
azd monitor                # View real-time logs
az containerapp logs show  # Container logs
az monitor app-insights    # Application Insights
```

## 💰 Cost Estimation

**Monthly (US East):**
- Container Apps: $30-60
- SQL Database (S1): $20-40
- Storage: $5-20
- **Total: ~$55-120**

[Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/)

## 🔄 CI/CD

### GitHub Actions Workflows

1. **Docker Test** - Build and test Docker image
2. **Azure Deploy** - Automated deployment to Azure

Trigger:
- Push to `main` → Deploy to production
- Push to `develop` → Deploy to staging
- Manual trigger → Deploy to selected environment

See `.github/workflows/` for details.

## 🤝 Migration from Cloudflare

Coming from Cloudflare Pages + D1?

See [MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md) for:
- Code migration examples
- Configuration changes
- Performance considerations
- Cost comparison
- Rollback procedures

## 🆘 Troubleshooting

### Common Issues

**Container won't start**
```bash
az containerapp logs show --name snowplatform --follow
```

**Database connection error**
```bash
sqlcmd -S <server>.database.windows.net -U sqladmin -P '<pwd>' -Q "SELECT 1"
```

**Storage access issue**
```bash
az storage container list --account-name <name>
```

See [DEVELOPMENT.md](./DEVELOPMENT.md#troubleshooting) for more.

## 📖 Learning Resources

- [Azure Container Apps](https://learn.microsoft.com/azure/container-apps/)
- [Azure SQL Database](https://learn.microsoft.com/azure/azure-sql/)
- [Bicep Tutorial](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
- [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/)
- [Docker Documentation](https://docs.docker.com/)

## 🤖 Commands Reference

### Development
```bash
npm install                    # Install dependencies
npm run build                  # Build application
npm run docker:build          # Build Docker image
docker-compose up             # Start local services
npm run dev                   # Run local dev server
```

### Azure
```bash
azd init --template .         # Initialize AZD
azd env new <name>            # Create environment
azd up                        # Deploy everything
azd deploy                    # Deploy code only
azd monitor                   # View logs
azd down                      # Delete resources
```

### Database
```bash
pwsh scripts/setup-db.ps1    # Setup database
sqlcmd -S ... -Q "SELECT 1"  # Test connection
```

## 📄 License

See LICENSE file for details.

## 👥 Contributing

1. Create feature branch
2. Make changes
3. Push and create pull request
4. CI/CD pipeline validates
5. Merge to deploy

## 📞 Support

- **Documentation**: See guides above
- **Issues**: Check troubleshooting section
- **Questions**: Review relevant guide
- **Azure Docs**: [learn.microsoft.com/azure/](https://learn.microsoft.com/azure/)

---

**Ready to deploy?** Start with [QUICK_START_AZURE.md](./QUICK_START_AZURE.md)
