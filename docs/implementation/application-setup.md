# Application Setup Guide - Email Platform

## Prerequisites

### System Requirements
- **Node.js**: 18.17.0 or later
- **npm**: 9.0.0 or later
- **Docker**: 24.0.0 or later (for local development)
- **Git**: 2.40.0 or later

### AWS Account Setup
- AWS Account with administrative privileges
- AWS CLI installed and configured
- Terraform CLI installed (version 1.6.0+)
- Domain registered and managed in Route53

### Development Tools
- **IDE**: VS Code or similar with TypeScript support
- **Database Tool**: SQL Server Management Studio or Azure Data Studio
- **API Testing**: Postman or similar REST client

## Project Structure

```
email-platform/
├── .github/                    # GitHub Actions workflows
│   └── workflows/
├── apps/
│   ├── web/                    # NextJS application
│   │   ├── src/
│   │   │   ├── app/           # App Router pages
│   │   │   ├── components/    # React components
│   │   │   ├── lib/           # Utility functions
│   │   │   └── types/         # TypeScript definitions
│   │   ├── public/            # Static assets
│   │   ├── package.json
│   │   └── next.config.js
│   └── email-engine/          # Email processing service
│       ├── src/
│       │   ├── handlers/      # Email processors
│       │   ├── services/      # Business logic
│       │   ├── utils/         # Utilities
│       │   └── types/         # TypeScript definitions
│       ├── Dockerfile
│       └── package.json
├── packages/
│   ├── database/              # Database schemas and migrations
│   ├── shared/                # Shared utilities and types
│   └── ui/                    # Shared UI components
├── infrastructure/            # Terraform configurations
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   └── modules/
├── docs/                      # Documentation
├── scripts/                   # Build and deployment scripts
├── package.json              # Root package.json
├── turbo.json               # Turborepo configuration
└── docker-compose.yml      # Local development environment
```

## Local Development Setup

### 1. Clone Repository
```bash
git clone https://github.com/your-org/email-platform.git
cd email-platform
```

### 2. Install Dependencies
```bash
# Install all dependencies using npm workspaces
npm install

# Or using yarn workspaces
yarn install
```

### 3. Environment Configuration

#### Copy Environment Templates
```bash
# Web application
cp apps/web/.env.example apps/web/.env.local

# Email engine
cp apps/email-engine/.env.example apps/email-engine/.env.local
```

#### Configure Environment Variables

**Web Application (.env.local)**
```env
# Database
DATABASE_URL="mssql://username:password@localhost:1433/EmailPlatform"

# AWS Configuration
AWS_REGION="us-east-1"
AWS_ACCESS_KEY_ID="your-access-key"
AWS_SECRET_ACCESS_KEY="your-secret-key"

# Cognito
NEXT_PUBLIC_COGNITO_USER_POOL_ID="us-east-1_xxxxxxxxx"
NEXT_PUBLIC_COGNITO_CLIENT_ID="xxxxxxxxxxxxxxxxxxxxxxxxxx"
COGNITO_CLIENT_SECRET="your-client-secret"

# SES Configuration
SES_CONFIGURATION_SET="email-platform-config-set"
SES_REGION="us-east-1"

# Redis Cache
REDIS_URL="redis://localhost:6379"

# NextJS
NEXTAUTH_SECRET="your-secret-key"
NEXTAUTH_URL="http://localhost:3000"

# API Keys
API_ENCRYPTION_KEY="your-32-character-encryption-key"

# Monitoring
NEXT_PUBLIC_SENTRY_DSN="your-sentry-dsn"
```

**Email Engine (.env.local)**
```env
# Database
DATABASE_URL="mssql://username:password@localhost:1433/EmailPlatform"

# AWS SES
AWS_REGION="us-east-1"
AWS_ACCESS_KEY_ID="your-access-key" 
AWS_SECRET_ACCESS_KEY="your-secret-key"
SES_CONFIGURATION_SET="email-platform-config-set"

# SQS Queues
EMAIL_QUEUE_URL="https://sqs.us-east-1.amazonaws.com/account/email-queue"
DLQ_URL="https://sqs.us-east-1.amazonaws.com/account/email-dlq"

# Redis Cache
REDIS_URL="redis://localhost:6379"

# Template Storage
S3_TEMPLATE_BUCKET="email-platform-templates"

# Monitoring
SENTRY_DSN="your-sentry-dsn"
```

### 4. Start Local Services

#### Using Docker Compose (Recommended)
```bash
# Start local SQL Server and Redis
docker-compose up -d db redis

# Wait for services to be ready
npm run wait-for-services
```

#### Manual Setup
```bash
# Start SQL Server (using Docker)
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourStrong@Passw0rd" \
  -p 1433:1433 --name sqlserver \
  -d mcr.microsoft.com/mssql/server:2022-latest

# Start Redis
docker run -d -p 6379:6379 --name redis redis:7-alpine
```

### 5. Database Setup

#### Run Migrations
```bash
# Navigate to database package
cd packages/database

# Run database migrations
npm run migrate:up

# Seed initial data
npm run seed:dev
```

#### Manual Database Setup (Alternative)
```sql
-- Connect to SQL Server and create database
CREATE DATABASE EmailPlatform;

-- Run the schema creation scripts
-- (Execute the SQL scripts from packages/database/migrations/)
```

### 6. Start Development Servers

#### Using Turborepo (Recommended)
```bash
# Start all applications in development mode
npm run dev

# Or start specific applications
npm run dev --filter=web
npm run dev --filter=email-engine
```

#### Manual Start
```bash
# Terminal 1 - Web Application
cd apps/web
npm run dev

# Terminal 2 - Email Engine
cd apps/email-engine
npm run dev
```

### 7. Verify Setup

#### Check Web Application
```bash
# Open browser to http://localhost:3000
# You should see the login page
curl http://localhost:3000/api/health
```

#### Check Email Engine
```bash
# Health check endpoint
curl http://localhost:3001/health

# Check queue processing
curl http://localhost:3001/api/queue/status
```

## AWS Infrastructure Setup

### 1. Configure AWS Credentials
```bash
aws configure
# Enter your AWS Access Key ID, Secret, Region, and output format
```

### 2. Deploy Infrastructure
```bash
cd infrastructure/environments/dev

# Initialize Terraform
terraform init

# Plan the deployment
terraform plan

# Apply the infrastructure
terraform apply
```

### 3. Configure Domain and DNS
```bash
# If using Route53 for DNS management
aws route53 create-hosted-zone --name yourdomain.com --caller-reference $(date +%s)

# Note the name servers and update your domain registrar
aws route53 get-hosted-zone --id /hostedzone/ZXXXXXXXXXXXXX
```

### 4. Set up SES Domain Verification
```bash
# Verify your sending domain
aws ses verify-domain-identity --domain yourdomain.com

# Get verification records
aws ses get-identity-verification-attributes --identities yourdomain.com
```

## Application Configuration

### 1. Tenant Configuration

#### Create Initial Tenant
```typescript
// Using the admin API or database directly
const tenant = {
  name: "Development Tenant",
  subdomain: "dev",
  settings: {
    defaultFromEmail: "noreply@yourdomain.com",
    defaultFromName: "Your Company",
    retentionPolicyDays: 30
  }
};
```

#### Configure Email Types
```typescript
const emailConfigurations = [
  {
    emailType: "welcome",
    enabled: true,
    additionalRecipients: [],
    ccRecipients: [],
    bccRecipients: ["admin@yourdomain.com"]
  },
  {
    emailType: "order_confirmation", 
    enabled: true,
    additionalRecipients: ["orders@yourdomain.com"],
    ccRecipients: [],
    bccRecipients: []
  }
];
```

### 2. User Setup

#### Create Admin User
```bash
# Using AWS CLI to create Cognito user
aws cognito-idp admin-create-user \
  --user-pool-id us-east-1_xxxxxxxxx \
  --username admin@yourdomain.com \
  --user-attributes Name=email,Value=admin@yourdomain.com \
  --temporary-password TempPassword123! \
  --message-action SUPPRESS
```

### 3. Template Setup

#### Import Sample Templates
```bash
# Run the template import script
npm run import-templates

# Or manually import through the web interface
# Navigate to /templates and upload XSLT files
```

## Development Workflow

### 1. Code Organization

#### Component Structure
```typescript
// apps/web/src/components/emails/EmailList.tsx
interface EmailListProps {
  tenantId: string;
  filters?: EmailFilters;
}

export function EmailList({ tenantId, filters }: EmailListProps) {
  // Component implementation
}
```

#### API Route Structure
```typescript
// apps/web/src/app/api/v1/emails/route.ts
import { NextRequest } from 'next/server';
import { validateTenant } from '@/lib/auth';

export async function POST(request: NextRequest) {
  const tenantId = await validateTenant(request);
  // API implementation
}
```

### 2. Database Operations

#### Using Prisma Client
```typescript
// packages/database/src/client.ts
import { PrismaClient } from '@prisma/client';

export const db = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL
    }
  }
});
```

#### Repository Pattern
```typescript
// apps/web/src/lib/repositories/EmailRepository.ts
export class EmailRepository {
  async sendEmail(tenantId: string, emailData: SendEmailRequest) {
    return await db.emailLogs.create({
      data: {
        tenantId,
        ...emailData
      }
    });
  }
}
```

### 3. Testing Setup

#### Unit Tests
```bash
# Run all tests
npm run test

# Run tests for specific package
npm run test --filter=web

# Run tests in watch mode
npm run test:watch
```

#### Integration Tests
```bash
# Start test environment
npm run test:integration:setup

# Run integration tests
npm run test:integration

# Cleanup test environment
npm run test:integration:cleanup
```

#### E2E Tests
```bash
# Install Playwright
npx playwright install

# Run E2E tests
npm run test:e2e

# Run tests in headed mode
npm run test:e2e:headed
```

## Production Deployment

### 1. Build Applications
```bash
# Build all applications
npm run build

# Build specific applications
npm run build --filter=web
npm run build --filter=email-engine
```

### 2. Docker Images
```bash
# Build Docker images
docker build -t email-platform-web -f apps/web/Dockerfile .
docker build -t email-platform-engine -f apps/email-engine/Dockerfile .

# Tag for ECR
docker tag email-platform-web:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/email-platform-web:latest

# Push to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/email-platform-web:latest
```

### 3. Deploy to ECS
```bash
# Update ECS service
aws ecs update-service \
  --cluster email-platform-cluster \
  --service email-platform-web \
  --force-new-deployment
```

## Monitoring and Logging

### 1. Application Monitoring
```typescript
// Configure Sentry for error tracking
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
});
```

### 2. CloudWatch Logs
```bash
# View application logs
aws logs describe-log-groups --log-group-name-prefix "/ecs/email-platform"

# Tail logs in real-time
aws logs tail /ecs/email-platform-web --follow
```

### 3. Performance Monitoring
```typescript
// Custom metrics for email processing
import { CloudWatch } from 'aws-sdk';

const cloudwatch = new CloudWatch();

cloudwatch.putMetricData({
  Namespace: 'EmailPlatform',
  MetricData: [{
    MetricName: 'EmailsSent',
    Value: 1,
    Unit: 'Count',
    Dimensions: [{
      Name: 'TenantId',
      Value: tenantId
    }]
  }]
}).promise();
```

## Troubleshooting

### Common Issues

#### Database Connection Issues
```bash
# Test database connectivity
npm run db:test-connection

# Check connection string format
# mssql://username:password@host:port/database
```

#### SES Configuration Issues
```bash
# Verify SES identity
aws ses get-identity-verification-attributes --identities yourdomain.com

# Check sending quota
aws ses get-send-quota

# Review bounce/complaint rates
aws ses get-reputation --identity yourdomain.com
```

#### Template Rendering Issues
```bash
# Test template compilation
npm run test-template --template-id=uuid --data='{"key":"value"}'

# Debug XSLT processing
# Check logs in CloudWatch or local console
```

### Debug Tools

#### Database Queries
```sql
-- Check email status
SELECT * FROM EmailLogs WHERE tenant_id = 'tenant-uuid' ORDER BY sent_at DESC;

-- Check event tracking
SELECT el.recipient_email, ee.event_type, ee.timestamp 
FROM EmailLogs el 
JOIN EmailEvents ee ON el.log_id = ee.log_id 
WHERE el.tenant_id = 'tenant-uuid';
```

#### API Testing
```bash
# Test email sending API
curl -X POST http://localhost:3000/api/v1/emails/send \
  -H "Content-Type: application/json" \
  -H "X-Tenant-ID: tenant-uuid" \
  -H "X-API-Key: your-api-key" \
  -d '{"type":"welcome","recipient":{"email":"test@example.com"},"data":{}}'
```

This setup guide provides a comprehensive foundation for developing and deploying the Email Platform. Follow each section carefully and refer to the troubleshooting section if you encounter issues.