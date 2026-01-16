# מדריך מעשי ופרקטי - מערכת סוכנים ארגונית

## תוכן עניינים
1. [מבנה הפרויקט](#מבנה-הפרויקט)
2. [הסוכנים והחלוקה](#הסוכנים-והחלוקה)
3. [דרך 1: Docker Compose (Local/Dev)](#דרך-1-docker-compose)
4. [דרך 2: Kubernetes (Production)](#דרך-2-kubernetes)
5. [דרך 3: AWS Bedrock + Lambda + API Gateway](#דרך-3-aws-bedrock)
6. [השוואה והמלצות](#השוואה-והמלצות)
7. [צעדים ראשונים](#צעדים-ראשונים)

---

## מבנה הפרויקט

```
enterprise-agents/
├── README.md
├── docker-compose.yml              # Local development
├── docker-compose.prod.yml         # Production with monitoring
│
├── packages/                       # Monorepo structure
│   ├── shared/                     # Shared code between agents
│   │   ├── src/
│   │   │   ├── types/
│   │   │   ├── utils/
│   │   │   ├── mcp-client/
│   │   │   └── llm/
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── orchestrator/               # Main orchestrator service
│   │   ├── src/
│   │   │   ├── server.ts
│   │   │   ├── webhooks/
│   │   │   ├── coordinator.ts
│   │   │   └── routes/
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── discovery-agent/            # Repository discovery
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── tools/
│   │   │   └── prompts/
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   ├── planning-agent/             # Task planning
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── tools/
│   │   │   └── templates/
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   ├── execution-agent/            # Code implementation
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── tools/
│   │   │   └── code-gen/
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   ├── cicd-agent/                 # CI/CD monitoring & fixing
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── tools/
│   │   │   └── analyzers/
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   └── slack-agent/                # Slack notifications & commands
│       ├── src/
│       │   ├── app.ts
│       │   ├── commands/
│       │   ├── events/
│       │   └── templates/
│       ├── Dockerfile
│       └── package.json
│
├── infrastructure/
│   ├── kubernetes/                 # K8s manifests
│   │   ├── namespace.yaml
│   │   ├── configmap.yaml
│   │   ├── secrets.yaml
│   │   ├── orchestrator/
│   │   ├── discovery-agent/
│   │   ├── planning-agent/
│   │   ├── execution-agent/
│   │   ├── cicd-agent/
│   │   ├── slack-agent/
│   │   └── monitoring/
│   │
│   ├── terraform/                  # AWS infrastructure
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── bedrock.tf
│   │   ├── lambda.tf
│   │   ├── api-gateway.tf
│   │   └── outputs.tf
│   │
│   └── aws-cdk/                    # Alternative: AWS CDK
│       ├── bin/
│       ├── lib/
│       └── cdk.json
│
├── config/
│   ├── organization.yaml           # Org-specific config
│   ├── agents.yaml                 # Agent configurations
│   └── .env.example                # Environment variables template
│
└── scripts/
    ├── setup.sh                    # Initial setup
    ├── deploy-docker.sh            # Docker deployment
    ├── deploy-k8s.sh               # Kubernetes deployment
    └── deploy-aws.sh               # AWS deployment
```

---

## הסוכנים והחלוקה

### תרשים הסוכנים

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            AGENT ARCHITECTURE                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                         ORCHESTRATOR                                     │   │
│  │  Port: 3000 | Stateless | Horizontally Scalable                         │   │
│  │  • Webhook receiver (Jira, GitHub, Sentry)                              │   │
│  │  • Task queue management                                                 │   │
│  │  • Agent coordination                                                    │   │
│  │  • State management (Redis/PostgreSQL)                                  │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                              │                                                  │
│         ┌────────────────────┼────────────────────┐                            │
│         │                    │                    │                            │
│         ▼                    ▼                    ▼                            │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                      │
│  │ DISCOVERY   │     │ PLANNING    │     │ EXECUTION   │                      │
│  │ AGENT       │     │ AGENT       │     │ AGENT       │                      │
│  │ Port: 3001  │     │ Port: 3002  │     │ Port: 3003  │                      │
│  │             │     │             │     │             │                      │
│  │ • Find repos│     │ • Analyze   │     │ • Write code│                      │
│  │ • Search    │     │ • Architect │     │ • Tests     │                      │
│  │ • Map deps  │     │ • Decompose │     │ • Review    │                      │
│  └─────────────┘     └─────────────┘     └─────────────┘                      │
│                                                 │                              │
│                                                 ▼                              │
│                                          ┌─────────────┐                      │
│                                          │ CI/CD       │                      │
│                                          │ AGENT       │                      │
│                                          │ Port: 3004  │                      │
│                                          │             │                      │
│                                          │ • Monitor CI│                      │
│                                          │ • Analyze   │                      │
│                                          │ • Auto-fix  │                      │
│                                          └─────────────┘                      │
│                                                                                │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                         SLACK AGENT                                      │  │
│  │  Port: 3005 | Always Running | WebSocket to Slack                       │  │
│  │  • Notifications • Commands • Interactive buttons                        │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                      EXTERNAL MCP SERVERS                                │  │
│  │  GitHub MCP | Atlassian Rovo MCP | Sentry MCP (All Official)            │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### הגדרת כל סוכן

| סוכן | תפקיד | Deploy נפרד | Scaling |
|------|-------|-------------|---------|
| **Orchestrator** | ניתוב, תיאום, state | ✅ חובה | Horizontal |
| **Discovery Agent** | חיפוש repos וקוד | ✅ כן | By demand |
| **Planning Agent** | תכנון וארכיטקטורה | ✅ כן | By demand |
| **Execution Agent** | כתיבת קוד | ✅ כן | By demand |
| **CI/CD Agent** | מעקב CI ותיקונים | ✅ כן | By demand |
| **Slack Agent** | הודעות ופקודות | ✅ כן | Single (WebSocket) |

---

## דרך 1: Docker Compose

### למי מתאים?
- 🟢 פיתוח מקומי
- 🟢 צוותים קטנים (עד 10 מפתחים)
- 🟢 POC/MVP
- 🟡 Production קטן

### הקמה מלאה

#### 1. Clone וsetup

```bash
# Clone the repo
git clone https://github.com/your-org/enterprise-agents.git
cd enterprise-agents

# Copy environment file
cp config/.env.example .env

# Edit .env with your credentials
nano .env
```

#### 2. קובץ .env

```bash
# .env

# === EXTERNAL SERVICES ===
# GitHub
GITHUB_TOKEN=ghp_xxxxxxxxxxxx
GITHUB_ORG=your-org-name

# Atlassian (for OAuth, set up OAuth app first)
ATLASSIAN_OAUTH_CLIENT_ID=your-client-id
ATLASSIAN_OAUTH_CLIENT_SECRET=your-client-secret

# Sentry
SENTRY_AUTH_TOKEN=sntrys_xxxxxxxxxxxx
SENTRY_ORG=your-sentry-org

# Slack
SLACK_BOT_TOKEN=xoxb-xxxxxxxxxxxx
SLACK_SIGNING_SECRET=xxxxxxxxxxxx
SLACK_APP_TOKEN=xapp-xxxxxxxxxxxx

# === LLM ===
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxx
LLM_MODEL=claude-sonnet-4-20250514

# === INFRASTRUCTURE ===
DATABASE_URL=postgresql://postgres:postgres@postgres:5432/agents
REDIS_URL=redis://redis:6379

# === ORGANIZATION CONFIG ===
JIRA_BASE_URL=https://your-company.atlassian.net
JIRA_PROJECT_KEY=PROJ
JIRA_AI_LABEL=AI
GITHUB_BRANCH_CONVENTION=feature/{ticketId}-{slug}
```

#### 3. docker-compose.yml

```yaml
# docker-compose.yml

version: '3.8'

services:
  # ==================== ORCHESTRATOR ====================
  orchestrator:
    build:
      context: .
      dockerfile: packages/orchestrator/Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - LLM_MODEL=${LLM_MODEL}
      # Agent URLs
      - DISCOVERY_AGENT_URL=http://discovery-agent:3001
      - PLANNING_AGENT_URL=http://planning-agent:3002
      - EXECUTION_AGENT_URL=http://execution-agent:3003
      - CICD_AGENT_URL=http://cicd-agent:3004
      - SLACK_AGENT_URL=http://slack-agent:3005
    depends_on:
      - postgres
      - redis
      - discovery-agent
      - planning-agent
      - execution-agent
      - cicd-agent
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped
    networks:
      - agent-network

  # ==================== DISCOVERY AGENT ====================
  discovery-agent:
    build:
      context: .
      dockerfile: packages/discovery-agent/Dockerfile
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - PORT=3001
      - GITHUB_TOKEN=${GITHUB_TOKEN}
      - GITHUB_ORG=${GITHUB_ORG}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - LLM_MODEL=${LLM_MODEL}
      - REDIS_URL=${REDIS_URL}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped
    networks:
      - agent-network

  # ==================== PLANNING AGENT ====================
  planning-agent:
    build:
      context: .
      dockerfile: packages/planning-agent/Dockerfile
    ports:
      - "3002:3002"
    environment:
      - NODE_ENV=production
      - PORT=3002
      - GITHUB_TOKEN=${GITHUB_TOKEN}
      - GITHUB_ORG=${GITHUB_ORG}
      - JIRA_BASE_URL=${JIRA_BASE_URL}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - LLM_MODEL=${LLM_MODEL}
      - GITHUB_BRANCH_CONVENTION=${GITHUB_BRANCH_CONVENTION}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3002/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped
    networks:
      - agent-network

  # ==================== EXECUTION AGENT ====================
  execution-agent:
    build:
      context: .
      dockerfile: packages/execution-agent/Dockerfile
    ports:
      - "3003:3003"
    environment:
      - NODE_ENV=production
      - PORT=3003
      - GITHUB_TOKEN=${GITHUB_TOKEN}
      - GITHUB_ORG=${GITHUB_ORG}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - LLM_MODEL=${LLM_MODEL}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3003/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped
    networks:
      - agent-network

  # ==================== CI/CD AGENT ====================
  cicd-agent:
    build:
      context: .
      dockerfile: packages/cicd-agent/Dockerfile
    ports:
      - "3004:3004"
    environment:
      - NODE_ENV=production
      - PORT=3004
      - GITHUB_TOKEN=${GITHUB_TOKEN}
      - GITHUB_ORG=${GITHUB_ORG}
      - SENTRY_AUTH_TOKEN=${SENTRY_AUTH_TOKEN}
      - SENTRY_ORG=${SENTRY_ORG}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - LLM_MODEL=${LLM_MODEL}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3004/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped
    networks:
      - agent-network

  # ==================== SLACK AGENT ====================
  slack-agent:
    build:
      context: .
      dockerfile: packages/slack-agent/Dockerfile
    ports:
      - "3005:3005"
    environment:
      - NODE_ENV=production
      - PORT=3005
      - SLACK_BOT_TOKEN=${SLACK_BOT_TOKEN}
      - SLACK_SIGNING_SECRET=${SLACK_SIGNING_SECRET}
      - SLACK_APP_TOKEN=${SLACK_APP_TOKEN}
      - ORCHESTRATOR_URL=http://orchestrator:3000
      - DATABASE_URL=${DATABASE_URL}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3005/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped
    networks:
      - agent-network

  # ==================== DATABASES ====================
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=agents
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - agent-network

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - agent-network

  # ==================== MONITORING (Optional) ====================
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./infrastructure/monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    restart: unless-stopped
    networks:
      - agent-network

  grafana:
    image: grafana/grafana:latest
    volumes:
      - ./infrastructure/monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./infrastructure/monitoring/grafana/datasources:/etc/grafana/provisioning/datasources
      - grafana-data:/var/lib/grafana
    ports:
      - "3010:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    restart: unless-stopped
    networks:
      - agent-network

networks:
  agent-network:
    driver: bridge

volumes:
  postgres-data:
  redis-data:
  prometheus-data:
  grafana-data:
```

#### 4. Dockerfile לכל סוכן

```dockerfile
# packages/orchestrator/Dockerfile

FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY packages/shared/package.json ./packages/shared/
COPY packages/orchestrator/package.json ./packages/orchestrator/

# Install dependencies
RUN npm install -g pnpm && pnpm install --frozen-lockfile

# Copy source
COPY packages/shared ./packages/shared
COPY packages/orchestrator ./packages/orchestrator

# Build
RUN pnpm --filter @enterprise-agents/shared build
RUN pnpm --filter @enterprise-agents/orchestrator build

# Production image
FROM node:20-alpine AS runner

WORKDIR /app

# Copy built files
COPY --from=builder /app/packages/orchestrator/dist ./dist
COPY --from=builder /app/packages/orchestrator/package.json ./
COPY --from=builder /app/node_modules ./node_modules

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

```dockerfile
# packages/discovery-agent/Dockerfile

FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY packages/shared/package.json ./packages/shared/
COPY packages/discovery-agent/package.json ./packages/discovery-agent/

RUN npm install -g pnpm && pnpm install --frozen-lockfile

COPY packages/shared ./packages/shared
COPY packages/discovery-agent ./packages/discovery-agent

RUN pnpm --filter @enterprise-agents/shared build
RUN pnpm --filter @enterprise-agents/discovery-agent build

FROM node:20-alpine AS runner

WORKDIR /app

COPY --from=builder /app/packages/discovery-agent/dist ./dist
COPY --from=builder /app/packages/discovery-agent/package.json ./
COPY --from=builder /app/node_modules ./node_modules

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3001/health || exit 1

EXPOSE 3001

CMD ["node", "dist/index.js"]
```

#### 5. הפעלה

```bash
# Build all images
docker-compose build

# Start all services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f orchestrator

# Scale specific agent
docker-compose up -d --scale execution-agent=3
```

#### 6. Webhook Setup

```bash
# Expose locally with ngrok (for development)
ngrok http 3000

# Configure webhooks:
# Jira: https://your-ngrok-url.ngrok.io/webhooks/jira
# GitHub: https://your-ngrok-url.ngrok.io/webhooks/github
# Sentry: https://your-ngrok-url.ngrok.io/webhooks/sentry
```

---

## דרך 2: Kubernetes

### למי מתאים?
- 🟢 Production
- 🟢 צוותים גדולים
- 🟢 High availability נדרש
- 🟢 Auto-scaling נדרש

### הקמה מלאה

#### 1. Namespace ו-ConfigMap

```yaml
# infrastructure/kubernetes/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: enterprise-agents
  labels:
    app.kubernetes.io/name: enterprise-agents
```

```yaml
# infrastructure/kubernetes/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: agent-config
  namespace: enterprise-agents
data:
  NODE_ENV: "production"
  LLM_MODEL: "claude-sonnet-4-20250514"
  GITHUB_ORG: "your-org"
  JIRA_BASE_URL: "https://your-company.atlassian.net"
  JIRA_PROJECT_KEY: "PROJ"
  JIRA_AI_LABEL: "AI"
  GITHUB_BRANCH_CONVENTION: "feature/{ticketId}-{slug}"
```

```yaml
# infrastructure/kubernetes/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: agent-secrets
  namespace: enterprise-agents
type: Opaque
stringData:
  GITHUB_TOKEN: "ghp_xxxxxxxxxxxx"
  ANTHROPIC_API_KEY: "sk-ant-xxxxxxxxxxxx"
  SENTRY_AUTH_TOKEN: "sntrys_xxxxxxxxxxxx"
  SLACK_BOT_TOKEN: "xoxb-xxxxxxxxxxxx"
  SLACK_SIGNING_SECRET: "xxxxxxxxxxxx"
  SLACK_APP_TOKEN: "xapp-xxxxxxxxxxxx"
  DATABASE_URL: "postgresql://user:pass@postgres:5432/agents"
  REDIS_URL: "redis://redis:6379"
```

#### 2. Deployment לכל סוכן

```yaml
# infrastructure/kubernetes/orchestrator/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orchestrator
  namespace: enterprise-agents
  labels:
    app: orchestrator
spec:
  replicas: 2
  selector:
    matchLabels:
      app: orchestrator
  template:
    metadata:
      labels:
        app: orchestrator
    spec:
      containers:
        - name: orchestrator
          image: your-registry/orchestrator:latest
          ports:
            - containerPort: 3000
          envFrom:
            - configMapRef:
                name: agent-config
            - secretRef:
                name: agent-secrets
          env:
            - name: PORT
              value: "3000"
            - name: DISCOVERY_AGENT_URL
              value: "http://discovery-agent:3001"
            - name: PLANNING_AGENT_URL
              value: "http://planning-agent:3002"
            - name: EXECUTION_AGENT_URL
              value: "http://execution-agent:3003"
            - name: CICD_AGENT_URL
              value: "http://cicd-agent:3004"
            - name: SLACK_AGENT_URL
              value: "http://slack-agent:3005"
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: orchestrator
  namespace: enterprise-agents
spec:
  selector:
    app: orchestrator
  ports:
    - port: 3000
      targetPort: 3000
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: orchestrator-hpa
  namespace: enterprise-agents
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: orchestrator
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

```yaml
# infrastructure/kubernetes/discovery-agent/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: discovery-agent
  namespace: enterprise-agents
  labels:
    app: discovery-agent
spec:
  replicas: 1
  selector:
    matchLabels:
      app: discovery-agent
  template:
    metadata:
      labels:
        app: discovery-agent
    spec:
      containers:
        - name: discovery-agent
          image: your-registry/discovery-agent:latest
          ports:
            - containerPort: 3001
          envFrom:
            - configMapRef:
                name: agent-config
            - secretRef:
                name: agent-secrets
          env:
            - name: PORT
              value: "3001"
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "1000m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3001
            initialDelaySeconds: 30
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: discovery-agent
  namespace: enterprise-agents
spec:
  selector:
    app: discovery-agent
  ports:
    - port: 3001
      targetPort: 3001
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: discovery-agent-hpa
  namespace: enterprise-agents
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: discovery-agent
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

#### 3. Ingress

```yaml
# infrastructure/kubernetes/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: agent-ingress
  namespace: enterprise-agents
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
    - hosts:
        - agents.your-company.com
      secretName: agent-tls
  rules:
    - host: agents.your-company.com
      http:
        paths:
          - path: /webhooks
            pathType: Prefix
            backend:
              service:
                name: orchestrator
                port:
                  number: 3000
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: orchestrator
                port:
                  number: 3000
```

#### 4. Deploy Script

```bash
#!/bin/bash
# scripts/deploy-k8s.sh

set -e

NAMESPACE="enterprise-agents"
REGISTRY="your-registry.com"

echo "🔨 Building images..."
docker build -t $REGISTRY/orchestrator:latest -f packages/orchestrator/Dockerfile .
docker build -t $REGISTRY/discovery-agent:latest -f packages/discovery-agent/Dockerfile .
docker build -t $REGISTRY/planning-agent:latest -f packages/planning-agent/Dockerfile .
docker build -t $REGISTRY/execution-agent:latest -f packages/execution-agent/Dockerfile .
docker build -t $REGISTRY/cicd-agent:latest -f packages/cicd-agent/Dockerfile .
docker build -t $REGISTRY/slack-agent:latest -f packages/slack-agent/Dockerfile .

echo "📤 Pushing images..."
docker push $REGISTRY/orchestrator:latest
docker push $REGISTRY/discovery-agent:latest
docker push $REGISTRY/planning-agent:latest
docker push $REGISTRY/execution-agent:latest
docker push $REGISTRY/cicd-agent:latest
docker push $REGISTRY/slack-agent:latest

echo "🚀 Deploying to Kubernetes..."
kubectl apply -f infrastructure/kubernetes/namespace.yaml
kubectl apply -f infrastructure/kubernetes/configmap.yaml
kubectl apply -f infrastructure/kubernetes/secrets.yaml
kubectl apply -f infrastructure/kubernetes/orchestrator/
kubectl apply -f infrastructure/kubernetes/discovery-agent/
kubectl apply -f infrastructure/kubernetes/planning-agent/
kubectl apply -f infrastructure/kubernetes/execution-agent/
kubectl apply -f infrastructure/kubernetes/cicd-agent/
kubectl apply -f infrastructure/kubernetes/slack-agent/
kubectl apply -f infrastructure/kubernetes/ingress.yaml

echo "⏳ Waiting for rollout..."
kubectl rollout status deployment/orchestrator -n $NAMESPACE
kubectl rollout status deployment/discovery-agent -n $NAMESPACE
kubectl rollout status deployment/planning-agent -n $NAMESPACE
kubectl rollout status deployment/execution-agent -n $NAMESPACE
kubectl rollout status deployment/cicd-agent -n $NAMESPACE
kubectl rollout status deployment/slack-agent -n $NAMESPACE

echo "✅ Deployment complete!"
kubectl get pods -n $NAMESPACE
```

---

## דרך 3: AWS Bedrock

### למי מתאים?
- 🟢 כבר ב-AWS ecosystem
- 🟢 רוצים managed agents
- 🟢 Serverless מועדף
- 🟢 Auto-scaling מלא

### ארכיטקטורה AWS

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                              AWS BEDROCK ARCHITECTURE                                    │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                          │
│  ┌─────────────────┐                                                                    │
│  │   API Gateway   │◄──── Webhooks (Jira, GitHub, Sentry)                               │
│  │   (REST/HTTP)   │                                                                    │
│  └────────┬────────┘                                                                    │
│           │                                                                              │
│           ▼                                                                              │
│  ┌─────────────────┐     ┌─────────────────┐                                           │
│  │ Lambda:         │     │   EventBridge   │                                           │
│  │ Webhook Handler │────►│   (Event Bus)   │                                           │
│  └─────────────────┘     └────────┬────────┘                                           │
│                                   │                                                     │
│                                   ▼                                                     │
│                          ┌─────────────────┐                                           │
│                          │ Step Functions  │                                           │
│                          │ (Orchestrator)  │                                           │
│                          └────────┬────────┘                                           │
│                                   │                                                     │
│         ┌─────────────────────────┼─────────────────────────┐                          │
│         │                         │                         │                          │
│         ▼                         ▼                         ▼                          │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐                  │
│  │ Bedrock Agent:  │     │ Bedrock Agent:  │     │ Bedrock Agent:  │                  │
│  │ Discovery       │     │ Planning        │     │ Execution       │                  │
│  │                 │     │                 │     │                 │                  │
│  │ Action Groups:  │     │ Action Groups:  │     │ Action Groups:  │                  │
│  │ ┌─────────────┐ │     │ ┌─────────────┐ │     │ ┌─────────────┐ │                  │
│  │ │GitHub Lambda│ │     │ │ Jira Lambda │ │     │ │GitHub Lambda│ │                  │
│  │ └─────────────┘ │     │ └─────────────┘ │     │ └─────────────┘ │                  │
│  │ ┌─────────────┐ │     │ ┌─────────────┐ │     │ ┌─────────────┐ │                  │
│  │ │ LLM Analysis│ │     │ │GitHub Lambda│ │     │ │  LLM Coder  │ │                  │
│  │ └─────────────┘ │     │ └─────────────┘ │     │ └─────────────┘ │                  │
│  └─────────────────┘     └─────────────────┘     └─────────────────┘                  │
│                                                                                         │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐                  │
│  │ Bedrock Agent:  │     │ Lambda:         │     │   DynamoDB      │                  │
│  │ CI/CD           │     │ Slack Handler   │     │   (State)       │                  │
│  │                 │     │                 │     │                 │                  │
│  │ Action Groups:  │     │ • Notifications │     │ • Tasks         │                  │
│  │ ┌─────────────┐ │     │ • Commands      │     │ • Threads       │                  │
│  │ │GitHub Lambda│ │     │ • Buttons       │     │ • Costs         │                  │
│  │ └─────────────┘ │     │                 │     │                 │                  │
│  │ ┌─────────────┐ │     └─────────────────┘     └─────────────────┘                  │
│  │ │Sentry Lambda│ │                                                                   │
│  │ └─────────────┘ │                                                                   │
│  └─────────────────┘                                                                   │
│                                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐  │
│  │                          MONITORING                                              │  │
│  │  CloudWatch Logs │ CloudWatch Metrics │ X-Ray │ Cost Explorer                   │  │
│  └─────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                         │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

### Terraform Implementation

```hcl
# infrastructure/terraform/main.tf

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# Variables
variable "aws_region" {
  default = "us-east-1"
}

variable "project_name" {
  default = "enterprise-agents"
}

variable "github_token" {
  sensitive = true
}

variable "anthropic_api_key" {
  sensitive = true
}

variable "slack_bot_token" {
  sensitive = true
}

variable "sentry_auth_token" {
  sensitive = true
}
```

```hcl
# infrastructure/terraform/dynamodb.tf

resource "aws_dynamodb_table" "tasks" {
  name           = "${var.project_name}-tasks"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "pk"
  range_key      = "sk"

  attribute {
    name = "pk"
    type = "S"
  }

  attribute {
    name = "sk"
    type = "S"
  }

  attribute {
    name = "gsi1pk"
    type = "S"
  }

  attribute {
    name = "gsi1sk"
    type = "S"
  }

  global_secondary_index {
    name            = "gsi1"
    hash_key        = "gsi1pk"
    range_key       = "gsi1sk"
    projection_type = "ALL"
  }

  ttl {
    attribute_name = "ttl"
    enabled        = true
  }

  tags = {
    Project = var.project_name
  }
}
```

```hcl
# infrastructure/terraform/secrets.tf

resource "aws_secretsmanager_secret" "github_token" {
  name = "${var.project_name}/github-token"
}

resource "aws_secretsmanager_secret_version" "github_token" {
  secret_id     = aws_secretsmanager_secret.github_token.id
  secret_string = var.github_token
}

resource "aws_secretsmanager_secret" "anthropic_api_key" {
  name = "${var.project_name}/anthropic-api-key"
}

resource "aws_secretsmanager_secret_version" "anthropic_api_key" {
  secret_id     = aws_secretsmanager_secret.anthropic_api_key.id
  secret_string = var.anthropic_api_key
}

resource "aws_secretsmanager_secret" "slack_bot_token" {
  name = "${var.project_name}/slack-bot-token"
}

resource "aws_secretsmanager_secret_version" "slack_bot_token" {
  secret_id     = aws_secretsmanager_secret.slack_bot_token.id
  secret_string = var.slack_bot_token
}

resource "aws_secretsmanager_secret" "sentry_auth_token" {
  name = "${var.project_name}/sentry-auth-token"
}

resource "aws_secretsmanager_secret_version" "sentry_auth_token" {
  secret_id     = aws_secretsmanager_secret.sentry_auth_token.id
  secret_string = var.sentry_auth_token
}
```

```hcl
# infrastructure/terraform/lambda.tf

# IAM Role for Lambda
resource "aws_iam_role" "lambda_role" {
  name = "${var.project_name}-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "lambda.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "lambda_policy" {
  name = "${var.project_name}-lambda-policy"
  role = aws_iam_role.lambda_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "arn:aws:logs:*:*:*"
      },
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:GetSecretValue"
        ]
        Resource = "arn:aws:secretsmanager:${var.aws_region}:*:secret:${var.project_name}/*"
      },
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:UpdateItem",
          "dynamodb:DeleteItem",
          "dynamodb:Query",
          "dynamodb:Scan"
        ]
        Resource = [
          aws_dynamodb_table.tasks.arn,
          "${aws_dynamodb_table.tasks.arn}/index/*"
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "bedrock:InvokeAgent",
          "bedrock:InvokeModel"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "states:StartExecution"
        ]
        Resource = "*"
      }
    ]
  })
}

# Webhook Handler Lambda
resource "aws_lambda_function" "webhook_handler" {
  filename         = "${path.module}/lambda/webhook-handler.zip"
  function_name    = "${var.project_name}-webhook-handler"
  role             = aws_iam_role.lambda_role.arn
  handler          = "index.handler"
  runtime          = "nodejs20.x"
  timeout          = 30
  memory_size      = 256

  environment {
    variables = {
      STATE_MACHINE_ARN = aws_sfn_state_machine.orchestrator.arn
      TASKS_TABLE       = aws_dynamodb_table.tasks.name
    }
  }
}

# GitHub Actions Lambda
resource "aws_lambda_function" "github_actions" {
  filename         = "${path.module}/lambda/github-actions.zip"
  function_name    = "${var.project_name}-github-actions"
  role             = aws_iam_role.lambda_role.arn
  handler          = "index.handler"
  runtime          = "nodejs20.x"
  timeout          = 60
  memory_size      = 512

  environment {
    variables = {
      GITHUB_SECRET_ID = aws_secretsmanager_secret.github_token.id
      GITHUB_ORG       = var.github_org
    }
  }
}

# Jira Actions Lambda
resource "aws_lambda_function" "jira_actions" {
  filename         = "${path.module}/lambda/jira-actions.zip"
  function_name    = "${var.project_name}-jira-actions"
  role             = aws_iam_role.lambda_role.arn
  handler          = "index.handler"
  runtime          = "nodejs20.x"
  timeout          = 30
  memory_size      = 256

  environment {
    variables = {
      JIRA_BASE_URL = var.jira_base_url
    }
  }
}

# Sentry Actions Lambda
resource "aws_lambda_function" "sentry_actions" {
  filename         = "${path.module}/lambda/sentry-actions.zip"
  function_name    = "${var.project_name}-sentry-actions"
  role             = aws_iam_role.lambda_role.arn
  handler          = "index.handler"
  runtime          = "nodejs20.x"
  timeout          = 30
  memory_size      = 256

  environment {
    variables = {
      SENTRY_SECRET_ID = aws_secretsmanager_secret.sentry_auth_token.id
      SENTRY_ORG       = var.sentry_org
    }
  }
}

# Slack Handler Lambda
resource "aws_lambda_function" "slack_handler" {
  filename         = "${path.module}/lambda/slack-handler.zip"
  function_name    = "${var.project_name}-slack-handler"
  role             = aws_iam_role.lambda_role.arn
  handler          = "index.handler"
  runtime          = "nodejs20.x"
  timeout          = 30
  memory_size      = 256

  environment {
    variables = {
      SLACK_SECRET_ID = aws_secretsmanager_secret.slack_bot_token.id
      TASKS_TABLE     = aws_dynamodb_table.tasks.name
    }
  }
}
```

```hcl
# infrastructure/terraform/api-gateway.tf

resource "aws_apigatewayv2_api" "main" {
  name          = "${var.project_name}-api"
  protocol_type = "HTTP"

  cors_configuration {
    allow_origins = ["*"]
    allow_methods = ["POST", "GET", "OPTIONS"]
    allow_headers = ["*"]
  }
}

resource "aws_apigatewayv2_stage" "prod" {
  api_id      = aws_apigatewayv2_api.main.id
  name        = "prod"
  auto_deploy = true

  access_log_settings {
    destination_arn = aws_cloudwatch_log_group.api_logs.arn
    format = jsonencode({
      requestId      = "$context.requestId"
      ip             = "$context.identity.sourceIp"
      requestTime    = "$context.requestTime"
      httpMethod     = "$context.httpMethod"
      routeKey       = "$context.routeKey"
      status         = "$context.status"
      responseLength = "$context.responseLength"
    })
  }
}

resource "aws_cloudwatch_log_group" "api_logs" {
  name              = "/aws/apigateway/${var.project_name}"
  retention_in_days = 14
}

# Webhook routes
resource "aws_apigatewayv2_integration" "webhook" {
  api_id             = aws_apigatewayv2_api.main.id
  integration_type   = "AWS_PROXY"
  integration_uri    = aws_lambda_function.webhook_handler.invoke_arn
  integration_method = "POST"
}

resource "aws_apigatewayv2_route" "jira_webhook" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "POST /webhooks/jira"
  target    = "integrations/${aws_apigatewayv2_integration.webhook.id}"
}

resource "aws_apigatewayv2_route" "github_webhook" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "POST /webhooks/github"
  target    = "integrations/${aws_apigatewayv2_integration.webhook.id}"
}

resource "aws_apigatewayv2_route" "sentry_webhook" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "POST /webhooks/sentry"
  target    = "integrations/${aws_apigatewayv2_integration.webhook.id}"
}

# Slack routes
resource "aws_apigatewayv2_integration" "slack" {
  api_id             = aws_apigatewayv2_api.main.id
  integration_type   = "AWS_PROXY"
  integration_uri    = aws_lambda_function.slack_handler.invoke_arn
  integration_method = "POST"
}

resource "aws_apigatewayv2_route" "slack_commands" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "POST /slack/commands"
  target    = "integrations/${aws_apigatewayv2_integration.slack.id}"
}

resource "aws_apigatewayv2_route" "slack_interactions" {
  api_id    = aws_apigatewayv2_api.main.id
  route_key = "POST /slack/interactions"
  target    = "integrations/${aws_apigatewayv2_integration.slack.id}"
}

# Lambda permissions
resource "aws_lambda_permission" "webhook_api" {
  statement_id  = "AllowAPIGateway"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.webhook_handler.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_apigatewayv2_api.main.execution_arn}/*/*"
}

resource "aws_lambda_permission" "slack_api" {
  statement_id  = "AllowAPIGateway"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.slack_handler.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_apigatewayv2_api.main.execution_arn}/*/*"
}

output "api_endpoint" {
  value = aws_apigatewayv2_stage.prod.invoke_url
}
```

```hcl
# infrastructure/terraform/bedrock.tf

# IAM Role for Bedrock Agents
resource "aws_iam_role" "bedrock_agent_role" {
  name = "${var.project_name}-bedrock-agent-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "bedrock.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "bedrock_agent_policy" {
  name = "${var.project_name}-bedrock-agent-policy"
  role = aws_iam_role.bedrock_agent_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "bedrock:InvokeModel"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "lambda:InvokeFunction"
        ]
        Resource = [
          aws_lambda_function.github_actions.arn,
          aws_lambda_function.jira_actions.arn,
          aws_lambda_function.sentry_actions.arn
        ]
      }
    ]
  })
}

# Discovery Agent
resource "aws_bedrockagent_agent" "discovery" {
  agent_name              = "${var.project_name}-discovery-agent"
  agent_resource_role_arn = aws_iam_role.bedrock_agent_role.arn
  foundation_model        = "anthropic.claude-sonnet-4-20250514-v1:0"
  idle_session_ttl_in_seconds = 600

  instruction = <<-EOT
    You are a code discovery agent for an enterprise organization.
    Your responsibilities are:
    1. Search across organization repositories to find relevant code
    2. Analyze repository structures and conventions
    3. Identify files and components related to tasks
    4. Map cross-repository dependencies
    
    When given a task description:
    - Extract keywords and technologies mentioned
    - Search for matching repositories
    - Analyze file structures of relevant repos
    - Return a ranked list of relevant repositories with explanations
    
    Always provide clear reasoning for your recommendations.
  EOT
}

resource "aws_bedrockagent_agent_action_group" "discovery_github" {
  agent_id          = aws_bedrockagent_agent.discovery.id
  agent_version     = "DRAFT"
  action_group_name = "GitHubActions"
  
  action_group_executor {
    lambda = aws_lambda_function.github_actions.arn
  }

  api_schema {
    payload = jsonencode({
      openapi = "3.0.0"
      info = {
        title   = "GitHub Actions"
        version = "1.0.0"
      }
      paths = {
        "/search-code" = {
          post = {
            operationId = "searchCode"
            description = "Search for code across repositories"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      query = { type = "string", description = "Search query" }
                      org   = { type = "string", description = "GitHub organization" }
                    }
                    required = ["query"]
                  }
                }
              }
            }
          }
        }
        "/get-repo-tree" = {
          post = {
            operationId = "getRepoTree"
            description = "Get repository file structure"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      repo = { type = "string" }
                      ref  = { type = "string" }
                    }
                    required = ["repo"]
                  }
                }
              }
            }
          }
        }
        "/get-file" = {
          post = {
            operationId = "getFile"
            description = "Get file contents"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      repo = { type = "string" }
                      path = { type = "string" }
                      ref  = { type = "string" }
                    }
                    required = ["repo", "path"]
                  }
                }
              }
            }
          }
        }
      }
    })
  }
}

# Planning Agent
resource "aws_bedrockagent_agent" "planning" {
  agent_name              = "${var.project_name}-planning-agent"
  agent_resource_role_arn = aws_iam_role.bedrock_agent_role.arn
  foundation_model        = "anthropic.claude-sonnet-4-20250514-v1:0"
  idle_session_ttl_in_seconds = 900

  instruction = <<-EOT
    You are a software planning agent.
    Your responsibilities are:
    1. Analyze Jira tickets and requirements
    2. Design technical solutions based on codebase analysis
    3. Create detailed implementation plans with TDD approach
    4. Create branches and PRs with PLAN.md files
    
    Plans should include:
    - Clear scope definition (in scope / out of scope)
    - Architecture overview with affected components
    - Test cases with acceptance criteria
    - Implementation tasks ordered by dependency
    - Cross-repo dependencies if applicable
    
    Always follow organization conventions for branch naming.
  EOT
}

resource "aws_bedrockagent_agent_action_group" "planning_github" {
  agent_id          = aws_bedrockagent_agent.planning.id
  agent_version     = "DRAFT"
  action_group_name = "GitHubActions"
  
  action_group_executor {
    lambda = aws_lambda_function.github_actions.arn
  }

  api_schema {
    payload = jsonencode({
      openapi = "3.0.0"
      info = {
        title   = "GitHub Actions for Planning"
        version = "1.0.0"
      }
      paths = {
        "/create-branch" = {
          post = {
            operationId = "createBranch"
            description = "Create a new branch"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      repo        = { type = "string" }
                      branch      = { type = "string" }
                      from_branch = { type = "string" }
                    }
                    required = ["repo", "branch"]
                  }
                }
              }
            }
          }
        }
        "/create-file" = {
          post = {
            operationId = "createFile"
            description = "Create or update a file"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      repo    = { type = "string" }
                      path    = { type = "string" }
                      content = { type = "string" }
                      message = { type = "string" }
                      branch  = { type = "string" }
                    }
                    required = ["repo", "path", "content", "message", "branch"]
                  }
                }
              }
            }
          }
        }
        "/create-pr" = {
          post = {
            operationId = "createPR"
            description = "Create a pull request"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      repo  = { type = "string" }
                      title = { type = "string" }
                      head  = { type = "string" }
                      base  = { type = "string" }
                      body  = { type = "string" }
                      draft = { type = "boolean" }
                    }
                    required = ["repo", "title", "head", "base"]
                  }
                }
              }
            }
          }
        }
      }
    })
  }
}

resource "aws_bedrockagent_agent_action_group" "planning_jira" {
  agent_id          = aws_bedrockagent_agent.planning.id
  agent_version     = "DRAFT"
  action_group_name = "JiraActions"
  
  action_group_executor {
    lambda = aws_lambda_function.jira_actions.arn
  }

  api_schema {
    payload = jsonencode({
      openapi = "3.0.0"
      info = {
        title   = "Jira Actions"
        version = "1.0.0"
      }
      paths = {
        "/get-issue" = {
          post = {
            operationId = "getIssue"
            description = "Get Jira issue details"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      issue_key = { type = "string" }
                    }
                    required = ["issue_key"]
                  }
                }
              }
            }
          }
        }
        "/add-comment" = {
          post = {
            operationId = "addComment"
            description = "Add comment to Jira issue"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      issue_key = { type = "string" }
                      comment   = { type = "string" }
                    }
                    required = ["issue_key", "comment"]
                  }
                }
              }
            }
          }
        }
      }
    })
  }
}

# Execution Agent
resource "aws_bedrockagent_agent" "execution" {
  agent_name              = "${var.project_name}-execution-agent"
  agent_resource_role_arn = aws_iam_role.bedrock_agent_role.arn
  foundation_model        = "anthropic.claude-sonnet-4-20250514-v1:0"
  idle_session_ttl_in_seconds = 1800

  instruction = <<-EOT
    You are a code execution agent.
    Your responsibilities are:
    1. Implement code according to task specifications
    2. Write tests following TDD principles
    3. Self-review code before pushing
    4. Commit and push changes to PRs
    
    Guidelines:
    - Follow existing code patterns and conventions
    - Write clean, maintainable code
    - Handle edge cases and errors
    - Add appropriate comments
    - Ensure tests pass before committing
    
    Never compromise on code quality.
  EOT
}

# CI/CD Agent
resource "aws_bedrockagent_agent" "cicd" {
  agent_name              = "${var.project_name}-cicd-agent"
  agent_resource_role_arn = aws_iam_role.bedrock_agent_role.arn
  foundation_model        = "anthropic.claude-sonnet-4-20250514-v1:0"
  idle_session_ttl_in_seconds = 600

  instruction = <<-EOT
    You are a CI/CD monitoring agent.
    Your responsibilities are:
    1. Monitor GitHub Actions workflows
    2. Analyze CI failures and identify root causes
    3. Attempt automatic fixes for common issues:
       - Lint errors (auto-fixable)
       - Formatting issues
       - Simple type errors
    4. Report status to Slack
    
    For failures you cannot fix automatically:
    - Provide detailed analysis
    - Suggest potential fixes
    - Flag for human intervention
    
    Maximum 3 auto-fix attempts before escalating to human.
  EOT
}

resource "aws_bedrockagent_agent_action_group" "cicd_github" {
  agent_id          = aws_bedrockagent_agent.cicd.id
  agent_version     = "DRAFT"
  action_group_name = "GitHubCIActions"
  
  action_group_executor {
    lambda = aws_lambda_function.github_actions.arn
  }

  api_schema {
    payload = jsonencode({
      openapi = "3.0.0"
      info = {
        title   = "GitHub CI Actions"
        version = "1.0.0"
      }
      paths = {
        "/list-workflow-runs" = {
          post = {
            operationId = "listWorkflowRuns"
            description = "List workflow runs for a repository"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      repo        = { type = "string" }
                      workflow_id = { type = "string" }
                    }
                    required = ["repo"]
                  }
                }
              }
            }
          }
        }
        "/get-job-logs" = {
          post = {
            operationId = "getJobLogs"
            description = "Get logs for a workflow job"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      repo   = { type = "string" }
                      job_id = { type = "number" }
                    }
                    required = ["repo", "job_id"]
                  }
                }
              }
            }
          }
        }
      }
    })
  }
}

resource "aws_bedrockagent_agent_action_group" "cicd_sentry" {
  agent_id          = aws_bedrockagent_agent.cicd.id
  agent_version     = "DRAFT"
  action_group_name = "SentryActions"
  
  action_group_executor {
    lambda = aws_lambda_function.sentry_actions.arn
  }

  api_schema {
    payload = jsonencode({
      openapi = "3.0.0"
      info = {
        title   = "Sentry Actions"
        version = "1.0.0"
      }
      paths = {
        "/list-issues" = {
          post = {
            operationId = "listIssues"
            description = "List Sentry issues"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      project = { type = "string" }
                      query   = { type = "string" }
                    }
                    required = ["project"]
                  }
                }
              }
            }
          }
        }
        "/get-issue" = {
          post = {
            operationId = "getIssue"
            description = "Get Sentry issue details"
            requestBody = {
              content = {
                "application/json" = {
                  schema = {
                    type = "object"
                    properties = {
                      issue_id = { type = "string" }
                    }
                    required = ["issue_id"]
                  }
                }
              }
            }
          }
        }
      }
    })
  }
}

# Agent Aliases (for invoking)
resource "aws_bedrockagent_agent_alias" "discovery" {
  agent_id         = aws_bedrockagent_agent.discovery.id
  agent_alias_name = "prod"
}

resource "aws_bedrockagent_agent_alias" "planning" {
  agent_id         = aws_bedrockagent_agent.planning.id
  agent_alias_name = "prod"
}

resource "aws_bedrockagent_agent_alias" "execution" {
  agent_id         = aws_bedrockagent_agent.execution.id
  agent_alias_name = "prod"
}

resource "aws_bedrockagent_agent_alias" "cicd" {
  agent_id         = aws_bedrockagent_agent.cicd.id
  agent_alias_name = "prod"
}

# Outputs
output "discovery_agent_id" {
  value = aws_bedrockagent_agent.discovery.id
}

output "discovery_agent_alias_id" {
  value = aws_bedrockagent_agent_alias.discovery.agent_alias_id
}

output "planning_agent_id" {
  value = aws_bedrockagent_agent.planning.id
}

output "planning_agent_alias_id" {
  value = aws_bedrockagent_agent_alias.planning.agent_alias_id
}

output "execution_agent_id" {
  value = aws_bedrockagent_agent.execution.id
}

output "execution_agent_alias_id" {
  value = aws_bedrockagent_agent_alias.execution.agent_alias_id
}

output "cicd_agent_id" {
  value = aws_bedrockagent_agent.cicd.id
}

output "cicd_agent_alias_id" {
  value = aws_bedrockagent_agent_alias.cicd.agent_alias_id
}
```

```hcl
# infrastructure/terraform/step-functions.tf

resource "aws_sfn_state_machine" "orchestrator" {
  name     = "${var.project_name}-orchestrator"
  role_arn = aws_iam_role.step_functions_role.arn

  definition = jsonencode({
    Comment = "Agent Orchestrator - coordinates agent workflows"
    StartAt = "DetermineWorkflowType"
    
    States = {
      DetermineWorkflowType = {
        Type = "Choice"
        Choices = [
          {
            Variable      = "$.source"
            StringEquals  = "jira"
            Next          = "JiraWorkflow"
          },
          {
            Variable      = "$.source"
            StringEquals  = "sentry"
            Next          = "SentryWorkflow"
          },
          {
            Variable      = "$.source"
            StringEquals  = "github"
            Next          = "GitHubCommandWorkflow"
          }
        ]
        Default = "UnknownSource"
      }

      JiraWorkflow = {
        Type = "Task"
        Resource = "arn:aws:states:::bedrock:invokeAgent"
        Parameters = {
          "agentId"      = aws_bedrockagent_agent.discovery.id
          "agentAliasId" = aws_bedrockagent_agent_alias.discovery.agent_alias_id
          "sessionId.$"  = "$.sessionId"
          "inputText.$"  = "States.Format('Find relevant repositories for task: {} - {}', $.ticketId, $.summary)"
        }
        ResultPath = "$.discovery"
        Next = "CreatePlan"
      }

      CreatePlan = {
        Type = "Task"
        Resource = "arn:aws:states:::bedrock:invokeAgent"
        Parameters = {
          "agentId"      = aws_bedrockagent_agent.planning.id
          "agentAliasId" = aws_bedrockagent_agent_alias.planning.agent_alias_id
          "sessionId.$"  = "$.sessionId"
          "inputText.$"  = "States.Format('Create implementation plan for ticket {} with discovery: {}', $.ticketId, States.JsonToString($.discovery))"
        }
        ResultPath = "$.plan"
        Next = "NotifySlack"
      }

      NotifySlack = {
        Type = "Task"
        Resource = aws_lambda_function.slack_handler.arn
        Parameters = {
          "action"   = "notify_plan_ready"
          "taskId.$" = "$.taskId"
          "plan.$"   = "$.plan"
        }
        Next = "WaitForApproval"
      }

      WaitForApproval = {
        Type = "Task"
        Resource = "arn:aws:states:::dynamodb:getItem.waitForTaskToken"
        Parameters = {
          TableName = aws_dynamodb_table.tasks.name
          Key = {
            pk = { "S.$" = "States.Format('TASK#{}', $.taskId)" }
            sk = { S = "APPROVAL" }
          }
        }
        ResultPath = "$.approval"
        Next = "ExecutePlan"
      }

      ExecutePlan = {
        Type = "Task"
        Resource = "arn:aws:states:::bedrock:invokeAgent"
        Parameters = {
          "agentId"      = aws_bedrockagent_agent.execution.id
          "agentAliasId" = aws_bedrockagent_agent_alias.execution.agent_alias_id
          "sessionId.$"  = "$.sessionId"
          "inputText.$"  = "States.Format('Execute plan: {}', States.JsonToString($.plan))"
        }
        ResultPath = "$.execution"
        Next = "MonitorCI"
      }

      MonitorCI = {
        Type = "Task"
        Resource = "arn:aws:states:::bedrock:invokeAgent"
        Parameters = {
          "agentId"      = aws_bedrockagent_agent.cicd.id
          "agentAliasId" = aws_bedrockagent_agent_alias.cicd.agent_alias_id
          "sessionId.$"  = "$.sessionId"
          "inputText.$"  = "States.Format('Monitor CI for PR in repo {} number {}', $.repo, $.prNumber)"
        }
        ResultPath = "$.ciResult"
        Next = "CheckCIResult"
      }

      CheckCIResult = {
        Type = "Choice"
        Choices = [
          {
            Variable     = "$.ciResult.success"
            BooleanEquals = true
            Next         = "TaskComplete"
          }
        ]
        Default = "HandleCIFailure"
      }

      HandleCIFailure = {
        Type = "Task"
        Resource = "arn:aws:states:::bedrock:invokeAgent"
        Parameters = {
          "agentId"      = aws_bedrockagent_agent.cicd.id
          "agentAliasId" = aws_bedrockagent_agent_alias.cicd.agent_alias_id
          "sessionId.$"  = "$.sessionId"
          "inputText"    = "Analyze CI failure and attempt auto-fix"
        }
        ResultPath = "$.fixAttempt"
        Next = "CheckFixResult"
      }

      CheckFixResult = {
        Type = "Choice"
        Choices = [
          {
            Variable      = "$.fixAttempt.fixed"
            BooleanEquals = true
            Next          = "MonitorCI"
          }
        ]
        Default = "EscalateToHuman"
      }

      EscalateToHuman = {
        Type = "Task"
        Resource = aws_lambda_function.slack_handler.arn
        Parameters = {
          "action"   = "escalate"
          "taskId.$" = "$.taskId"
          "reason.$" = "$.fixAttempt.reason"
        }
        End = true
      }

      TaskComplete = {
        Type = "Task"
        Resource = aws_lambda_function.slack_handler.arn
        Parameters = {
          "action"   = "task_complete"
          "taskId.$" = "$.taskId"
        }
        End = true
      }

      SentryWorkflow = {
        Type = "Pass"
        End  = true
      }

      GitHubCommandWorkflow = {
        Type = "Pass"
        End  = true
      }

      UnknownSource = {
        Type  = "Fail"
        Error = "UnknownSource"
        Cause = "Event source not recognized"
      }
    }
  })
}

resource "aws_iam_role" "step_functions_role" {
  name = "${var.project_name}-step-functions-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "states.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "step_functions_policy" {
  name = "${var.project_name}-step-functions-policy"
  role = aws_iam_role.step_functions_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "bedrock:InvokeAgent"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "lambda:InvokeFunction"
        ]
        Resource = [
          aws_lambda_function.slack_handler.arn,
          aws_lambda_function.webhook_handler.arn
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:UpdateItem"
        ]
        Resource = aws_dynamodb_table.tasks.arn
      }
    ]
  })
}
```

#### Deploy Script

```bash
#!/bin/bash
# scripts/deploy-aws.sh

set -e

echo "🚀 Deploying AWS Bedrock Agent System"

# Check prerequisites
command -v terraform >/dev/null 2>&1 || { echo "❌ Terraform required"; exit 1; }
command -v aws >/dev/null 2>&1 || { echo "❌ AWS CLI required"; exit 1; }

# Navigate to terraform directory
cd infrastructure/terraform

# Initialize Terraform
echo "📦 Initializing Terraform..."
terraform init

# Plan
echo "📋 Planning deployment..."
terraform plan -out=tfplan

# Confirm
read -p "Deploy? (y/n) " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    exit 1
fi

# Apply
echo "🚀 Applying..."
terraform apply tfplan

# Get outputs
echo "✅ Deployment complete!"
echo ""
echo "API Endpoint: $(terraform output -raw api_endpoint)"
echo ""
echo "Configure webhooks:"
echo "  Jira:   $(terraform output -raw api_endpoint)/webhooks/jira"
echo "  GitHub: $(terraform output -raw api_endpoint)/webhooks/github"
echo "  Sentry: $(terraform output -raw api_endpoint)/webhooks/sentry"
echo "  Slack:  $(terraform output -raw api_endpoint)/slack/commands"
```

---

## השוואה והמלצות

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                            COMPARISON MATRIX                                            │
├────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                         │
│  Criteria              │ Docker Compose   │ Kubernetes       │ AWS Bedrock            │
│  ─────────────────────────────────────────────────────────────────────────────────────│
│  Setup Time            │ 🟢 1-2 hours     │ 🟡 1-2 days      │ 🟡 1 day               │
│  Complexity            │ 🟢 Low           │ 🔴 High          │ 🟡 Medium              │
│  Cost (small)          │ 🟢 $50-100/mo    │ 🟡 $200-400/mo   │ 🟢 $80-150/mo          │
│  Cost (large)          │ 🔴 $500+/mo      │ 🟢 $300-600/mo   │ 🟢 $200-400/mo         │
│  Scaling               │ 🟡 Manual        │ 🟢 Auto (HPA)    │ 🟢 Auto (Serverless)   │
│  High Availability     │ 🔴 Limited       │ 🟢 Built-in      │ 🟢 Built-in            │
│  Maintenance           │ 🟢 Low           │ 🔴 High          │ 🟢 Low (Managed)       │
│  Flexibility           │ 🟢 Full control  │ 🟢 Full control  │ 🟡 AWS ecosystem       │
│  Deploy Independence   │ 🟢 Per container │ 🟢 Per pod       │ 🟢 Per Lambda/Agent    │
│  Monitoring            │ 🟡 DIY           │ 🟢 Prometheus    │ 🟢 CloudWatch          │
│  ─────────────────────────────────────────────────────────────────────────────────────│
│                                                                                         │
│  RECOMMENDATIONS:                                                                       │
│  • Startup/POC         → Docker Compose                                                │
│  • Small team (≤10)    → Docker Compose or AWS Bedrock                                 │
│  • Medium team (10-50) → Kubernetes or AWS Bedrock                                     │
│  • Enterprise (50+)    → Kubernetes with full monitoring                               │
│  • AWS-native org      → AWS Bedrock                                                   │
│                                                                                         │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## צעדים ראשונים

### Quick Start - Docker Compose (5 דקות)

```bash
# 1. Clone
git clone https://github.com/your-org/enterprise-agents.git
cd enterprise-agents

# 2. Configure
cp config/.env.example .env
# Edit .env with your tokens

# 3. Run
docker-compose up -d

# 4. Verify
curl http://localhost:3000/health

# 5. Setup webhooks (use ngrok for local dev)
ngrok http 3000
```

### Quick Start - AWS Bedrock (30 דקות)

```bash
# 1. Clone
git clone https://github.com/your-org/enterprise-agents.git
cd enterprise-agents/infrastructure/terraform

# 2. Configure
cp terraform.tfvars.example terraform.tfvars
# Edit with your values

# 3. Deploy
terraform init
terraform apply

# 4. Get endpoint
terraform output api_endpoint
```

---

## סיכום

| דרך | זמן הקמה | עלות | מורכבות | המלצה |
|-----|---------|------|---------|-------|
| **Docker Compose** | שעה | $50-100 | נמוכה | 🟢 התחל כאן |
| **Kubernetes** | 1-2 ימים | $200-600 | גבוהה | Production גדול |
| **AWS Bedrock** | יום | $80-400 | בינונית | AWS shops |

### הצעד הבא שלך:
1. בחר דרך (Docker Compose מומלץ להתחלה)
2. Clone את ה-repo
3. הגדר את ה-.env
4. הרץ `docker-compose up -d`
5. בדוק עם `curl localhost:3000/health`
6. הגדר webhooks
7. נסה עם טיקט ראשון!
