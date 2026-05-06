# CI/CD Pipeline Template: GitHub Actions + Docker

## Production-Ready Workflow for Building, Testing, and Deploying

---

## Pipeline Overview

```yaml
Code Push to Main/PR
    ↓
Lint & Format Check
    ↓
Build & Unit Tests
    ↓
Integration Tests
    ↓
Build Docker Image
    ↓
Push to Registry
    ↓
Deploy to Staging
    ↓
Smoke Tests
    ↓
Manual Approval (for production)
    ↓
Deploy to Production
    ↓
Health Checks
```

---

## GitHub Actions Workflow Template

```yaml
# .github/workflows/ci-cd.yml

name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  lint:
    runs-on: ubuntu-latest
    name: Lint & Format
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Check formatting
        run: npm run format:check

  test:
    runs-on: ubuntu-latest
    name: Unit Tests
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  build:
    runs-on: ubuntu-latest
    name: Build Docker Image
    needs: [ lint, test ]
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=semver,pattern={{version}}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:buildcache
          cache-to: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:buildcache,mode=max

  integration-tests:
    runs-on: ubuntu-latest
    name: Integration Tests
    needs: build
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_DB: test_db
          POSTGRES_PASSWORD: test_password
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run integration tests
        env:
          DATABASE_URL: postgresql://postgres:test_password@localhost:5432/test_db
        run: npm run test:integration

  deploy-staging:
    runs-on: ubuntu-latest
    name: Deploy to Staging
    needs: [ build, integration-tests ]
    if: github.event_name == 'push' && github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to staging
        env:
          DEPLOYMENT_KEY: ${{ secrets.STAGING_DEPLOYMENT_KEY }}
          STAGING_SERVER: ${{ secrets.STAGING_SERVER }}
        run: |
          mkdir -p ~/.ssh
          echo "$DEPLOYMENT_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H $STAGING_SERVER >> ~/.ssh/known_hosts
          ssh -i ~/.ssh/deploy_key deploy@$STAGING_SERVER 'cd /app && docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:develop && docker-compose up -d'
      
      - name: Health check
        run: |
          for i in {1..30}; do
            if curl -f https://staging.example.com/health; then
              echo "✓ Staging is healthy"
              exit 0
            fi
            echo "Attempt $i/30 - waiting for deployment..."
            sleep 10
          done
          echo "✗ Staging health check failed"
          exit 1
      
      - name: Smoke tests
        run: |
          npm install
          npm run test:smoke -- --baseUrl https://staging.example.com

  deploy-production:
    runs-on: ubuntu-latest
    name: Deploy to Production
    needs: [ build, integration-tests ]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Wait for approval
        run: |
          echo "⏳ Waiting for manual approval before production deployment..."
          # This is handled by GitHub's environment protection rules
      
      - name: Deploy to production
        env:
          DEPLOYMENT_KEY: ${{ secrets.PROD_DEPLOYMENT_KEY }}
          PROD_SERVER: ${{ secrets.PROD_SERVER }}
        run: |
          mkdir -p ~/.ssh
          echo "$DEPLOYMENT_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H $PROD_SERVER >> ~/.ssh/known_hosts
          ssh -i ~/.ssh/deploy_key deploy@$PROD_SERVER 'cd /app && docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main && docker-compose up -d'
      
      - name: Health check
        run: |
          for i in {1..30}; do
            if curl -f https://example.com/health; then
              echo "✓ Production is healthy"
              exit 0
            fi
            echo "Attempt $i/30 - waiting for deployment..."
            sleep 10
          done
          echo "✗ Production health check failed"
          exit 1
      
      - name: Notify deployment
        if: success()
        uses: 8398a7/action-slack@v3
        with:
          status: custom
          custom_payload: |
            {
              text: `✓ Production deployment successful`,
              attachments: [{
                color: 'good',
                text: `Version: ${{ github.sha }}\nCommit: ${{ github.event.head_commit.message }}`
              }]
            }
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
      
      - name: Notify deployment failure
        if: failure()
        uses: 8398a7/action-slack@v3
        with:
          status: custom
          custom_payload: |
            {
              text: `✗ Production deployment failed`,
              attachments: [{
                color: 'danger',
                text: `Version: ${{ github.sha }}\n⚠️ Please investigate immediately`
              }]
            }
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Dockerfile Template

```dockerfile
# Multi-stage build for smaller image size

FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM node:18-alpine

WORKDIR /app

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Copy from builder
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"

EXPOSE 3000

# Use dumb-init to handle signals properly
ENTRYPOINT ["dumb-init", "--"]

CMD ["node", "dist/server.js"]
```

---

## Docker Compose Template

```yaml
# docker-compose.yml for local development and deployments

version: '3.9'

services:
  app:
    image: ghcr.io/myrepo/myapp:main
    container_name: myapp
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://postgres:password@postgres:5432/myapp
      REDIS_URL: redis://redis:6379/0
      LOG_LEVEL: info
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    volumes:
      - ./logs:/app/logs
    networks:
      - app-network
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://localhost:3000/health" ]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s

  postgres:
    image: postgres:14-alpine
    container_name: myapp-postgres
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: password
      POSTGRES_INITDB_ARGS: "--encoding=UTF8"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    restart: unless-stopped
    networks:
      - app-network
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -U postgres" ]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    volumes:
      - redis_data:/data
    restart: unless-stopped
    networks:
      - app-network
    command: redis-server --appendonly yes

  nginx:
    image: nginx:alpine
    container_name: myapp-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    restart: unless-stopped
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

---

## Best Practices

### 1. Security
- ✅ Use secrets for sensitive data (API keys, passwords)
- ✅ Scan dependencies for vulnerabilities
- ✅ Use minimal base images (alpine)
- ✅ Run containers as non-root user
- ✅ Sign commits and tags

### 2. Performance
- ✅ Use Docker layer caching
- ✅ Parallel test execution
- ✅ Cache dependencies (npm, pip)
- ✅ Optimize Docker image size
- ✅ Use fast CI runners

### 3. Reliability
- ✅ Run tests before building
- ✅ Health checks after deployment
- ✅ Automated rollback on failure
- ✅ Monitor deployments
- ✅ Keep logs for debugging

### 4. Deployment
- ✅ Tag Docker images with version
- ✅ Manual approval for production
- ✅ Blue-green deployment for zero downtime
- ✅ Database migrations before code
- ✅ Smoke tests after deployment

---

## Troubleshooting

**Docker build fails:**
- Check Dockerfile syntax
- Ensure all dependencies listed
- Clean Docker cache: `docker system prune`

**Tests fail in CI but pass locally:**
- Different Node version
- Environment variables missing
- File permissions on Linux

**Deployment fails:**
- Check secrets are set correctly
- SSH key permissions (600)
- Server has enough disk space

---

## About Rework Digital

This template was created by **Rework Digital** - Resources Department for automation professionals.

**Resources Department Contact:** resource@reworkdigital.io  
**Follow us on GitHub:** https://github.com/Reworkdigital-io

---

*Last Updated: 2026-04-10*  
*Version: 1.0*
