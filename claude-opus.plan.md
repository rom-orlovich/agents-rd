# מערכת אגנטים ארגונית V3 - שימוש ב-MCP Servers רשמיים

## תוכן עניינים
1. [סקירת ה-MCP Servers הרשמיים](#סקירת-ה-mcp-servers-הרשמיים)
2. [ארכיטקטורה משודרגת](#ארכיטקטורה-משודרגת)
3. [קונפיגורציה](#קונפיגורציה)
4. [Slack Integration](#slack-integration)
5. [מימוש מלא](#מימוש-מלא)
6. [יתרונות וחסרונות](#יתרונות-וחסרונות)
7. [עלויות](#עלויות)

---

## סקירת ה-MCP Servers הרשמיים

### ✅ GitHub MCP Server (Official)
**מקור**: https://github.com/github/github-mcp-server

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         GITHUB OFFICIAL MCP SERVER                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  🔗 Remote Server: https://api.githubcopilot.com/mcp/                           │
│  🐳 Docker: ghcr.io/github/github-mcp-server                                    │
│                                                                                  │
│  📦 TOOLSETS AVAILABLE:                                                         │
│  ├── repos      - Repository management, files, branches                        │
│  ├── issues     - Issue CRUD, comments, labels, sub-issues                      │
│  ├── pull_requests - PRs, reviews, merges                                       │
│  ├── actions    - Workflow runs, CI/CD monitoring, artifacts                    │
│  ├── code_security - Code scanning alerts                                       │
│  ├── discussions - GitHub Discussions                                           │
│  ├── users      - User search and info                                          │
│  ├── projects   - GitHub Projects (boards)                                      │
│  ├── notifications - Notification management                                     │
│  └── git        - Low-level git operations (tree, commits)                      │
│                                                                                  │
│  🔐 Auth: OAuth 2.0 or Personal Access Token (PAT)                              │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Tools מרכזיים לפרויקט שלנו:**
- `create_branch` - יצירת branch
- `create_or_update_file` - יצירת/עדכון קבצים
- `create_pull_request` - פתיחת PR
- `search_code` - חיפוש קוד
- `get_file_contents` - קריאת קבצים
- `list_workflow_runs` - מעקב CI
- `get_job_logs` - לוגים של CI

---

### ✅ Atlassian Rovo MCP Server (Official - Jira & Confluence)
**מקור**: https://support.atlassian.com/atlassian-rovo-mcp-server/

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      ATLASSIAN ROVO MCP SERVER                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  🔗 Remote Server: https://mcp.atlassian.com/v1/mcp                             │
│                                                                                  │
│  📦 CAPABILITIES:                                                               │
│  ├── Jira Workflows:                                                            │
│  │   ├── Search issues ("Find all open bugs in Project Alpha")                  │
│  │   ├── Create/Update issues                                                   │
│  │   ├── Bulk create issues from notes                                          │
│  │   └── Link issues to Confluence pages                                        │
│  │                                                                              │
│  ├── Confluence Workflows:                                                      │
│  │   ├── Summarize pages                                                        │
│  │   ├── Create/Update pages                                                    │
│  │   └── Navigate spaces                                                        │
│  │                                                                              │
│  └── Compass Workflows:                                                         │
│      ├── Create service components                                              │
│      └── Query dependencies                                                     │
│                                                                                  │
│  🔐 Auth: OAuth 2.1 (Browser-based)                                             │
│                                                                                  │
│  ⚠️ Important: Respects user permissions in Atlassian Cloud                     │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

### ✅ Sentry MCP Server (Official)
**מקור**: https://docs.sentry.io/product/sentry-mcp/

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         SENTRY MCP SERVER                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  🔗 Remote Server: https://mcp.sentry.dev/mcp                                   │
│  💻 Local: npx @sentry/mcp-server@latest --access-token=TOKEN                   │
│                                                                                  │
│  📦 16+ TOOLS AVAILABLE:                                                        │
│  ├── Core Tools:                                                                │
│  │   ├── Organizations - List and query                                         │
│  │   ├── Projects - Find, list, create                                          │
│  │   ├── Teams - Manage and query                                               │
│  │   ├── Issues - Access details, search, analyze                               │
│  │   └── DSNs - List and create                                                 │
│  │                                                                              │
│  ├── Analysis Tools:                                                            │
│  │   ├── Error Searching - Find errors in specific files                        │
│  │   ├── Issue Analysis - Detailed investigation with context                   │
│  │   └── Seer Integration - AI agent for root cause & auto-fix                  │
│  │                                                                              │
│  └── Advanced:                                                                  │
│      ├── Release Management                                                     │
│      └── Performance Monitoring                                                 │
│                                                                                  │
│  🔐 Auth: OAuth (recommended) or User Auth Token                                │
│                                                                                  │
│  🤖 Seer Integration: Invoke AI for automated issue fixes!                      │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

### ❌ Slack - אין MCP רשמי (נכון לינואר 2026)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         SLACK INTEGRATION OPTIONS                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ⚠️ Slack does NOT have an official MCP Server yet                              │
│                                                                                  │
│  📦 OPTIONS:                                                                    │
│                                                                                  │
│  Option 1: Slack Bolt SDK (Recommended) ✅                                      │
│  ├── Full-featured Node.js/Python SDK                                           │
│  ├── Socket Mode for real-time events                                           │
│  ├── Slash commands, Interactive components                                     │
│  ├── Event subscriptions                                                        │
│  └── npm: @slack/bolt                                                           │
│                                                                                  │
│  Option 2: Slack Web API                                                        │
│  ├── REST API for all Slack operations                                          │
│  ├── chat.postMessage, conversations.*, etc.                                    │
│  └── npm: @slack/web-api                                                        │
│                                                                                  │
│  Option 3: Community MCP Server                                                 │
│  ├── https://github.com/modelcontextprotocol/servers                            │
│  └── Third-party, less stable                                                   │
│                                                                                  │
│  🎯 RECOMMENDATION: Use Slack Bolt SDK as dedicated Slack Agent                 │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## ארכיטקטורה משודרגת

### תרשים עם MCPs רשמיים

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                    TRIGGER SOURCES                                       │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│  Jira Webhook      │  GitHub Webhook    │  Sentry Webhook    │  Slack Event             │
│  (ticket+AI label) │  (PR comment)      │  (recurring issue) │  (/agent command)        │
└──────────┬─────────┴──────────┬─────────┴──────────┬─────────┴──────────┬───────────────┘
           │                    │                    │                    │
           ▼                    ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                              ORCHESTRATOR SERVICE                                        │
│  ┌─────────────────────────────────────────────────────────────────────────────────┐   │
│  │                           MCP CLIENT MANAGER                                     │   │
│  │  Manages connections to all official MCP servers                                 │   │
│  └─────────────────────────────────────────────────────────────────────────────────┘   │
│           │                    │                    │                    │              │
│           ▼                    ▼                    ▼                    ▼              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐   │
│  │ ATLASSIAN ROVO  │  │ GITHUB MCP      │  │ SENTRY MCP      │  │ SLACK BOLT      │   │
│  │ MCP SERVER      │  │ SERVER          │  │ SERVER          │  │ (Custom Agent)  │   │
│  │ (Official)      │  │ (Official)      │  │ (Official)      │  │                 │   │
│  │                 │  │                 │  │                 │  │                 │   │
│  │ mcp.atlassian   │  │ api.github      │  │ mcp.sentry.dev  │  │ Slack Web API   │   │
│  │ .com/v1/mcp     │  │ copilot.com/mcp │  │ /mcp            │  │                 │   │
│  │                 │  │                 │  │                 │  │                 │   │
│  │ • Jira issues   │  │ • Repos         │  │ • Issues        │  │ • Messages      │   │
│  │ • Create/Update │  │ • PRs           │  │ • Stack traces  │  │ • Commands      │   │
│  │ • Search        │  │ • CI/Actions    │  │ • Seer AI       │  │ • Threads       │   │
│  │ • Comments      │  │ • Files         │  │ • Projects      │  │ • Buttons       │   │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  └─────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────────────┘
           │                    │                    │                    │
           ▼                    ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                              AGENT LOGIC LAYER                                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐   │
│  │ DISCOVERY       │  │ PLANNING        │  │ EXECUTION       │  │ CI/CD           │   │
│  │ AGENT           │  │ AGENT           │  │ AGENT           │  │ AGENT           │   │
│  │                 │  │                 │  │                 │  │                 │   │
│  │ Uses:           │  │ Uses:           │  │ Uses:           │  │ Uses:           │   │
│  │ • GitHub search │  │ • Jira API      │  │ • GitHub files  │  │ • GitHub Actions│   │
│  │ • GitHub tree   │  │ • GitHub PR     │  │ • GitHub PR     │  │ • Sentry Seer   │   │
│  │ • LLM analysis  │  │ • LLM planning  │  │ • LLM coding    │  │ • Auto-fix      │   │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  └─────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## קונפיגורציה

### MCP Client Configuration

```typescript
// config/mcp-servers.ts

export const mcpServersConfig = {
  // Atlassian Rovo MCP (Jira & Confluence)
  atlassian: {
    // Option 1: Remote (Recommended - OAuth)
    remote: {
      url: 'https://mcp.atlassian.com/v1/mcp',
      transport: 'http',
      auth: 'oauth2.1', // Browser-based OAuth
    },
    // Option 2: Local with mcp-remote proxy
    local: {
      command: 'npx',
      args: ['-y', 'mcp-remote@latest', 'https://mcp.atlassian.com/v1/mcp'],
    },
  },

  // GitHub MCP Server
  github: {
    // Option 1: Remote (OAuth via GitHub Copilot)
    remote: {
      url: 'https://api.githubcopilot.com/mcp/',
      transport: 'http',
      auth: 'oauth', // Or PAT via header
    },
    // Option 2: Docker (PAT auth)
    docker: {
      command: 'docker',
      args: [
        'run', '-i', '--rm',
        '-e', 'GITHUB_PERSONAL_ACCESS_TOKEN',
        '-e', 'GITHUB_TOOLSETS=repos,issues,pull_requests,actions,code_security',
        'ghcr.io/github/github-mcp-server'
      ],
      env: {
        GITHUB_PERSONAL_ACCESS_TOKEN: process.env.GITHUB_TOKEN!,
      },
    },
    // Option 3: Local binary
    local: {
      command: './bin/github-mcp-server',
      args: ['stdio', '--toolsets', 'repos,issues,pull_requests,actions'],
      env: {
        GITHUB_PERSONAL_ACCESS_TOKEN: process.env.GITHUB_TOKEN!,
      },
    },
  },

  // Sentry MCP Server
  sentry: {
    // Option 1: Remote (OAuth)
    remote: {
      url: 'https://mcp.sentry.dev/mcp',
      transport: 'http',
      auth: 'oauth',
    },
    // Option 2: Local STDIO
    local: {
      command: 'npx',
      args: ['@sentry/mcp-server@latest'],
      env: {
        SENTRY_ACCESS_TOKEN: process.env.SENTRY_AUTH_TOKEN!,
        SENTRY_HOST: process.env.SENTRY_HOST || 'sentry.io',
      },
    },
  },
};
```

### Docker Compose Configuration

```yaml
# docker-compose.yml

version: '3.8'

services:
  orchestrator:
    build: ./packages/orchestrator
    ports:
      - "3000:3000"
    environment:
      # GitHub MCP
      - GITHUB_PERSONAL_ACCESS_TOKEN=${GITHUB_TOKEN}
      - GITHUB_ORG=${GITHUB_ORG}
      - GITHUB_TOOLSETS=repos,issues,pull_requests,actions,code_security
      
      # Sentry MCP (if using local mode)
      - SENTRY_ACCESS_TOKEN=${SENTRY_AUTH_TOKEN}
      - SENTRY_HOST=${SENTRY_HOST:-sentry.io}
      
      # Slack (Web API)
      - SLACK_BOT_TOKEN=${SLACK_BOT_TOKEN}
      - SLACK_SIGNING_SECRET=${SLACK_SIGNING_SECRET}
      - SLACK_APP_TOKEN=${SLACK_APP_TOKEN}
      
      # Atlassian (OAuth - handled via browser)
      - ATLASSIAN_OAUTH_CLIENT_ID=${ATLASSIAN_OAUTH_CLIENT_ID}
      - ATLASSIAN_OAUTH_CLIENT_SECRET=${ATLASSIAN_OAUTH_CLIENT_SECRET}
      
      # LLM
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      
      # Database
      - DATABASE_URL=postgresql://postgres:postgres@postgres:5432/agents
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
      - github-mcp
      - sentry-mcp
    volumes:
      - ./config:/app/config:ro

  # GitHub MCP Server (Docker mode)
  github-mcp:
    image: ghcr.io/github/github-mcp-server:latest
    environment:
      - GITHUB_PERSONAL_ACCESS_TOKEN=${GITHUB_TOKEN}
      - GITHUB_TOOLSETS=repos,issues,pull_requests,actions,code_security,git
    stdin_open: true
    tty: true

  # Sentry MCP Server (Local STDIO mode)
  sentry-mcp:
    build:
      context: .
      dockerfile: Dockerfile.sentry-mcp
    environment:
      - SENTRY_ACCESS_TOKEN=${SENTRY_AUTH_TOKEN}
    stdin_open: true
    tty: true

  # Slack Agent (Custom - using Bolt SDK)
  slack-agent:
    build: ./packages/slack-agent
    environment:
      - SLACK_BOT_TOKEN=${SLACK_BOT_TOKEN}
      - SLACK_SIGNING_SECRET=${SLACK_SIGNING_SECRET}
      - SLACK_APP_TOKEN=${SLACK_APP_TOKEN}
      - ORCHESTRATOR_URL=http://orchestrator:3000
    ports:
      - "3001:3001"

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=agents
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:
```

---

## Slack Integration

### Slack Agent with Bolt SDK

```typescript
// packages/slack-agent/src/app.ts

import { App, LogLevel } from '@slack/bolt';
import { WebClient } from '@slack/web-api';

const app = new App({
  token: process.env.SLACK_BOT_TOKEN,
  signingSecret: process.env.SLACK_SIGNING_SECRET,
  socketMode: true,
  appToken: process.env.SLACK_APP_TOKEN,
  logLevel: LogLevel.INFO,
});

// Store task threads
const taskThreads = new Map<string, { channel: string; ts: string }>();

// ==================== COMMANDS ====================

app.command('/agent', async ({ command, ack, respond, client }) => {
  await ack();
  
  const [subCommand, ...args] = command.text.trim().split(/\s+/);
  
  switch (subCommand?.toLowerCase()) {
    case 'status':
      await handleStatusCommand(args, command.channel_id, respond);
      break;
    case 'assign':
      await handleAssignCommand(args, command.user_id, command.channel_id, client);
      break;
    case 'pause':
      await handlePauseCommand(args[0], command.user_id, respond);
      break;
    case 'resume':
      await handleResumeCommand(args[0], command.user_id, respond);
      break;
    case 'cancel':
      await handleCancelCommand(args[0], command.user_id, respond);
      break;
    case 'costs':
      await handleCostsCommand(args[0] || '7d', command.channel_id, respond);
      break;
    case 'health':
      await handleHealthCommand(respond);
      break;
    default:
      await respond(getHelpMessage());
  }
});

// ==================== NOTIFICATIONS API ====================

export class SlackNotifier {
  private client: WebClient;
  private defaultChannel: string;
  
  constructor() {
    this.client = new WebClient(process.env.SLACK_BOT_TOKEN);
    this.defaultChannel = process.env.SLACK_DEFAULT_CHANNEL || '#dev-agents';
  }

  async notifyTaskStarted(task: TaskInfo): Promise<string> {
    const result = await this.client.chat.postMessage({
      channel: this.defaultChannel,
      blocks: [
        {
          type: 'header',
          text: { type: 'plain_text', text: '🚀 Agent Started Working', emoji: true },
        },
        {
          type: 'section',
          fields: [
            { type: 'mrkdwn', text: `*Ticket:*\n<${task.jiraUrl}|${task.ticketId}>` },
            { type: 'mrkdwn', text: `*Title:*\n${task.title}` },
          ],
        },
        {
          type: 'context',
          elements: [
            { type: 'mrkdwn', text: `Task ID: \`${task.taskId}\` • Updates in thread` },
          ],
        },
      ],
    });
    
    // Store thread for future updates
    if (result.ts && result.channel) {
      taskThreads.set(task.taskId, { channel: result.channel, ts: result.ts });
    }
    
    return result.ts!;
  }

  async notifyPlanReady(task: TaskInfo, plan: PlanSummary): Promise<void> {
    const thread = taskThreads.get(task.taskId);
    
    await this.client.chat.postMessage({
      channel: thread?.channel || this.defaultChannel,
      thread_ts: thread?.ts,
      blocks: [
        {
          type: 'header',
          text: { type: 'plain_text', text: '📋 Implementation Plan Ready', emoji: true },
        },
        {
          type: 'section',
          fields: [
            { type: 'mrkdwn', text: `*Repos:*\n${plan.repos.join(', ')}` },
            { type: 'mrkdwn', text: `*Tasks:*\n${plan.taskCount} tasks` },
            { type: 'mrkdwn', text: `*Est. Time:*\n${plan.estimatedHours}h` },
            { type: 'mrkdwn', text: `*PR:*\n<${plan.prUrl}|#${plan.prNumber}>` },
          ],
        },
        {
          type: 'actions',
          elements: [
            {
              type: 'button',
              text: { type: 'plain_text', text: '👀 Review Plan', emoji: true },
              url: plan.prUrl,
            },
            {
              type: 'button',
              text: { type: 'plain_text', text: '✅ Approve', emoji: true },
              action_id: `approve_${task.taskId}`,
              style: 'primary',
            },
            {
              type: 'button',
              text: { type: 'plain_text', text: '✏️ Request Changes', emoji: true },
              action_id: `modify_${task.taskId}`,
            },
          ],
        },
      ],
    });
  }

  async notifyCIStatus(task: TaskInfo, ci: CIStatus): Promise<void> {
    const thread = taskThreads.get(task.taskId);
    const emoji = ci.success ? '✅' : '❌';
    const status = ci.success ? 'CI Passed' : 'CI Failed';
    
    const blocks: any[] = [
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `${emoji} *${status}* for <${ci.prUrl}|PR #${ci.prNumber}>`,
        },
      },
    ];
    
    if (!ci.success && ci.failedChecks) {
      blocks.push({
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `*Failed Checks:*\n${ci.failedChecks.map(c => `• ${c.name}: ${c.reason}`).join('\n')}`,
        },
      });
      
      if (ci.autoFixAttempted) {
        blocks.push({
          type: 'context',
          elements: [
            { type: 'mrkdwn', text: '🔧 _Auto-fix attempted..._' },
          ],
        });
      }
    }
    
    await this.client.chat.postMessage({
      channel: thread?.channel || this.defaultChannel,
      thread_ts: thread?.ts,
      blocks,
    });
  }

  async notifyTaskCompleted(task: TaskInfo): Promise<void> {
    const thread = taskThreads.get(task.taskId);
    
    await this.client.chat.postMessage({
      channel: thread?.channel || this.defaultChannel,
      thread_ts: thread?.ts,
      blocks: [
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: `🎉 *Task Completed!*\n<${task.jiraUrl}|${task.ticketId}> is ready for review.`,
          },
        },
      ],
    });
    
    // Clean up
    taskThreads.delete(task.taskId);
  }

  async postThreadUpdate(taskId: string, message: string, emoji?: string): Promise<void> {
    const thread = taskThreads.get(taskId);
    if (!thread) return;
    
    await this.client.chat.postMessage({
      channel: thread.channel,
      thread_ts: thread.ts,
      text: emoji ? `${emoji} ${message}` : message,
    });
  }

  async postDailySummary(summary: DailySummary): Promise<void> {
    await this.client.chat.postMessage({
      channel: this.defaultChannel,
      blocks: [
        {
          type: 'header',
          text: { type: 'plain_text', text: '📊 Daily Agent Summary', emoji: true },
        },
        {
          type: 'section',
          fields: [
            { type: 'mrkdwn', text: `*Tasks Started:*\n${summary.tasksStarted}` },
            { type: 'mrkdwn', text: `*Completed:*\n${summary.tasksCompleted}` },
            { type: 'mrkdwn', text: `*PRs Created:*\n${summary.prsCreated}` },
            { type: 'mrkdwn', text: `*PRs Merged:*\n${summary.prsMerged}` },
            { type: 'mrkdwn', text: `*CI Success:*\n${summary.ciSuccessRate}%` },
            { type: 'mrkdwn', text: `*Total Cost:*\n$${summary.totalCost.toFixed(2)}` },
          ],
        },
      ],
    });
  }
}

// ==================== BUTTON HANDLERS ====================

app.action(/^approve_/, async ({ action, ack, body, client }) => {
  await ack();
  
  const taskId = (action as any).action_id.replace('approve_', '');
  
  // Call orchestrator
  await fetch(`${process.env.ORCHESTRATOR_URL}/tasks/${taskId}/approve`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ userId: body.user.id }),
  });
  
  // Update message
  await client.chat.update({
    channel: body.channel!.id,
    ts: (body as any).message.ts,
    text: `✅ Plan approved by <@${body.user.id}>`,
    blocks: [
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `✅ *Plan approved* by <@${body.user.id}>\nExecution starting...`,
        },
      },
    ],
  });
});

app.action(/^modify_/, async ({ action, ack, body, client }) => {
  await ack();
  
  const taskId = (action as any).action_id.replace('modify_', '');
  
  await client.views.open({
    trigger_id: (body as any).trigger_id,
    view: {
      type: 'modal',
      callback_id: `feedback_${taskId}`,
      title: { type: 'plain_text', text: 'Request Changes' },
      submit: { type: 'plain_text', text: 'Submit' },
      blocks: [
        {
          type: 'input',
          block_id: 'feedback_block',
          element: {
            type: 'plain_text_input',
            action_id: 'feedback_input',
            multiline: true,
            placeholder: { type: 'plain_text', text: 'Describe the changes...' },
          },
          label: { type: 'plain_text', text: 'Feedback' },
        },
      ],
    },
  });
});

app.view(/^feedback_/, async ({ ack, body, view }) => {
  await ack();
  
  const taskId = view.callback_id.replace('feedback_', '');
  const feedback = view.state.values.feedback_block.feedback_input.value;
  
  await fetch(`${process.env.ORCHESTRATOR_URL}/tasks/${taskId}/modify`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ userId: body.user.id, feedback }),
  });
});

// ==================== HELPER FUNCTIONS ====================

function getHelpMessage() {
  return {
    blocks: [
      {
        type: 'header',
        text: { type: 'plain_text', text: '🤖 Agent Commands', emoji: true },
      },
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `\`/agent status\` - All active tasks
\`/agent status [task-id]\` - Specific task
\`/agent assign PROJ-123\` - Assign agent to ticket
\`/agent pause [task-id]\` - Pause task
\`/agent resume [task-id]\` - Resume task
\`/agent cancel [task-id]\` - Cancel task
\`/agent costs [7d|30d]\` - Cost report
\`/agent health\` - System health`,
        },
      },
    ],
  };
}

// Start app
(async () => {
  await app.start(process.env.SLACK_PORT || 3001);
  console.log('⚡️ Slack Agent is running!');
})();

// Types
interface TaskInfo {
  taskId: string;
  ticketId: string;
  title: string;
  jiraUrl: string;
}

interface PlanSummary {
  repos: string[];
  taskCount: number;
  estimatedHours: number;
  prUrl: string;
  prNumber: number;
}

interface CIStatus {
  prNumber: number;
  prUrl: string;
  success: boolean;
  failedChecks?: { name: string; reason: string }[];
  autoFixAttempted?: boolean;
}

interface DailySummary {
  tasksStarted: number;
  tasksCompleted: number;
  prsCreated: number;
  prsMerged: number;
  ciSuccessRate: number;
  totalCost: number;
}
```

---

## מימוש מלא - Orchestrator עם MCPs רשמיים

```typescript
// packages/orchestrator/src/mcp-manager.ts

import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';
import { StreamableHTTPClientTransport } from '@modelcontextprotocol/sdk/client/streamableHttp.js';
import { spawn } from 'child_process';
import { mcpServersConfig } from '../config/mcp-servers.js';

export class MCPManager {
  private clients: Map<string, Client> = new Map();
  
  async initialize(): Promise<void> {
    // Initialize GitHub MCP (Docker)
    await this.connectGitHub();
    
    // Initialize Sentry MCP (Remote or Local)
    await this.connectSentry();
    
    // Initialize Atlassian MCP (Remote)
    await this.connectAtlassian();
    
    console.log('✅ All MCP connections established');
  }

  private async connectGitHub(): Promise<void> {
    const client = new Client({ name: 'orchestrator-github', version: '1.0.0' }, {});
    
    // Using Docker
    const config = mcpServersConfig.github.docker;
    const transport = new StdioClientTransport({
      command: config.command,
      args: config.args,
      env: { ...process.env, ...config.env },
    });
    
    await client.connect(transport);
    this.clients.set('github', client);
    
    // List available tools
    const tools = await client.listTools();
    console.log(`📦 GitHub MCP: ${tools.tools.length} tools available`);
  }

  private async connectSentry(): Promise<void> {
    const client = new Client({ name: 'orchestrator-sentry', version: '1.0.0' }, {});
    
    // Option 1: Remote (OAuth)
    if (process.env.SENTRY_USE_REMOTE === 'true') {
      const transport = new StreamableHTTPClientTransport({
        url: new URL(mcpServersConfig.sentry.remote.url),
      });
      await client.connect(transport);
    } else {
      // Option 2: Local STDIO
      const config = mcpServersConfig.sentry.local;
      const transport = new StdioClientTransport({
        command: config.command,
        args: config.args,
        env: { ...process.env, ...config.env },
      });
      await client.connect(transport);
    }
    
    this.clients.set('sentry', client);
    const tools = await client.listTools();
    console.log(`📦 Sentry MCP: ${tools.tools.length} tools available`);
  }

  private async connectAtlassian(): Promise<void> {
    const client = new Client({ name: 'orchestrator-atlassian', version: '1.0.0' }, {});
    
    // Remote server with OAuth (requires browser auth flow)
    // For server-side, use mcp-remote proxy
    const transport = new StdioClientTransport({
      command: 'npx',
      args: ['-y', 'mcp-remote@latest', mcpServersConfig.atlassian.remote.url],
    });
    
    await client.connect(transport);
    this.clients.set('atlassian', client);
    
    const tools = await client.listTools();
    console.log(`📦 Atlassian MCP: ${tools.tools.length} tools available`);
  }

  // ==================== GITHUB OPERATIONS ====================
  
  async github_searchCode(query: string, org?: string): Promise<any> {
    const client = this.clients.get('github')!;
    const fullQuery = org ? `${query} org:${org}` : query;
    
    return client.callTool({
      name: 'search_code',
      arguments: { query: fullQuery, perPage: 30 },
    });
  }

  async github_getFileContents(owner: string, repo: string, path: string, ref?: string): Promise<any> {
    const client = this.clients.get('github')!;
    return client.callTool({
      name: 'get_file_contents',
      arguments: { owner, repo, path, ref },
    });
  }

  async github_createBranch(owner: string, repo: string, branch: string, fromBranch?: string): Promise<any> {
    const client = this.clients.get('github')!;
    return client.callTool({
      name: 'create_branch',
      arguments: { owner, repo, branch, from_branch: fromBranch },
    });
  }

  async github_createOrUpdateFile(
    owner: string, 
    repo: string, 
    path: string, 
    content: string, 
    message: string,
    branch: string,
    sha?: string
  ): Promise<any> {
    const client = this.clients.get('github')!;
    return client.callTool({
      name: 'create_or_update_file',
      arguments: { owner, repo, path, content, message, branch, sha },
    });
  }

  async github_createPullRequest(
    owner: string,
    repo: string,
    title: string,
    head: string,
    base: string,
    body?: string,
    draft?: boolean
  ): Promise<any> {
    const client = this.clients.get('github')!;
    return client.callTool({
      name: 'create_pull_request',
      arguments: { owner, repo, title, head, base, body, draft },
    });
  }

  async github_listWorkflowRuns(owner: string, repo: string, workflowId: string): Promise<any> {
    const client = this.clients.get('github')!;
    return client.callTool({
      name: 'list_workflow_runs',
      arguments: { owner, repo, workflow_id: workflowId },
    });
  }

  async github_getJobLogs(owner: string, repo: string, jobId: number): Promise<any> {
    const client = this.clients.get('github')!;
    return client.callTool({
      name: 'get_job_logs',
      arguments: { owner, repo, job_id: jobId, return_content: true },
    });
  }

  async github_getRepositoryTree(owner: string, repo: string, ref?: string): Promise<any> {
    const client = this.clients.get('github')!;
    return client.callTool({
      name: 'get_repository_tree',
      arguments: { owner, repo, tree_sha: ref, recursive: true },
    });
  }

  // ==================== ATLASSIAN/JIRA OPERATIONS ====================
  
  async jira_searchIssues(query: string): Promise<any> {
    const client = this.clients.get('atlassian')!;
    // Atlassian MCP uses natural language queries
    return client.callTool({
      name: 'search_issues',
      arguments: { query },
    });
  }

  async jira_getIssue(issueKey: string): Promise<any> {
    const client = this.clients.get('atlassian')!;
    return client.callTool({
      name: 'get_issue',
      arguments: { issue_key: issueKey },
    });
  }

  async jira_createIssue(
    projectKey: string,
    issueType: string,
    summary: string,
    description?: string
  ): Promise<any> {
    const client = this.clients.get('atlassian')!;
    return client.callTool({
      name: 'create_issue',
      arguments: { project_key: projectKey, issue_type: issueType, summary, description },
    });
  }

  async jira_addComment(issueKey: string, comment: string): Promise<any> {
    const client = this.clients.get('atlassian')!;
    return client.callTool({
      name: 'add_comment',
      arguments: { issue_key: issueKey, comment },
    });
  }

  async jira_updateIssue(issueKey: string, fields: Record<string, any>): Promise<any> {
    const client = this.clients.get('atlassian')!;
    return client.callTool({
      name: 'update_issue',
      arguments: { issue_key: issueKey, ...fields },
    });
  }

  // ==================== SENTRY OPERATIONS ====================
  
  async sentry_listIssues(projectSlug: string, query?: string): Promise<any> {
    const client = this.clients.get('sentry')!;
    return client.callTool({
      name: 'list_issues',
      arguments: { project_slug: projectSlug, query },
    });
  }

  async sentry_getIssue(issueId: string): Promise<any> {
    const client = this.clients.get('sentry')!;
    return client.callTool({
      name: 'get_issue',
      arguments: { issue_id: issueId },
    });
  }

  async sentry_searchErrors(projectSlug: string, filename: string): Promise<any> {
    const client = this.clients.get('sentry')!;
    return client.callTool({
      name: 'search_errors',
      arguments: { project_slug: projectSlug, filename },
    });
  }

  async sentry_invokeSeer(issueId: string): Promise<any> {
    const client = this.clients.get('sentry')!;
    // Seer is Sentry's AI for root cause analysis and auto-fix
    return client.callTool({
      name: 'invoke_seer',
      arguments: { issue_id: issueId },
    });
  }

  async sentry_getSeerStatus(issueId: string): Promise<any> {
    const client = this.clients.get('sentry')!;
    return client.callTool({
      name: 'get_seer_status',
      arguments: { issue_id: issueId },
    });
  }

  // ==================== UTILITY ====================
  
  getClient(name: 'github' | 'atlassian' | 'sentry'): Client {
    const client = this.clients.get(name);
    if (!client) throw new Error(`MCP client '${name}' not initialized`);
    return client;
  }

  async disconnect(): Promise<void> {
    for (const [name, client] of this.clients) {
      await client.close();
      console.log(`Disconnected from ${name} MCP`);
    }
  }
}
```

### Updated Orchestrator with MCPs

```typescript
// packages/orchestrator/src/coordinator.ts

import { MCPManager } from './mcp-manager.js';
import { SlackNotifier } from '../slack-agent/app.js';
import { Anthropic } from '@anthropic-ai/sdk';
import { config } from './config.js';

export class AgentCoordinator {
  private mcp: MCPManager;
  private slack: SlackNotifier;
  private anthropic: Anthropic;

  constructor() {
    this.mcp = new MCPManager();
    this.slack = new SlackNotifier();
    this.anthropic = new Anthropic({ apiKey: config.anthropic.apiKey });
  }

  async initialize(): Promise<void> {
    await this.mcp.initialize();
  }

  async processJiraTicket(ticketId: string): Promise<string> {
    const taskId = this.generateTaskId();
    
    try {
      // 1. Get ticket from Jira via Atlassian MCP
      const ticketResult = await this.mcp.jira_getIssue(ticketId);
      const ticket = JSON.parse(ticketResult.content[0].text);
      
      // Notify Slack
      await this.slack.notifyTaskStarted({
        taskId,
        ticketId,
        title: ticket.summary,
        jiraUrl: `${config.jira.baseUrl}/browse/${ticketId}`,
      });
      
      // 2. Discovery - Find relevant repos via GitHub MCP
      await this.slack.postThreadUpdate(taskId, 'Searching for relevant repositories...', '🔍');
      
      const searchQuery = this.extractKeywords(ticket.summary, ticket.description);
      const searchResult = await this.mcp.github_searchCode(searchQuery, config.github.org);
      const codeMatches = JSON.parse(searchResult.content[0].text);
      
      // Analyze with LLM to determine relevant repos
      const discovery = await this.analyzeRelevantRepos(ticket, codeMatches);
      
      // 3. Planning - Create plan with LLM
      await this.slack.postThreadUpdate(taskId, 'Creating implementation plan...', '📋');
      
      const plan = await this.createPlan(ticket, discovery);
      
      // 4. Create branch and PR via GitHub MCP
      const primaryRepo = discovery.repos[0];
      const branchName = `feature/${ticketId.toLowerCase()}-${this.slugify(ticket.summary)}`;
      
      await this.mcp.github_createBranch(
        config.github.org,
        primaryRepo.repo,
        branchName
      );
      
      // Create PLAN.md
      const planContent = this.generatePlanMarkdown(plan, ticketId);
      await this.mcp.github_createOrUpdateFile(
        config.github.org,
        primaryRepo.repo,
        'PLAN.md',
        planContent,
        `Add implementation plan for ${ticketId}`,
        branchName
      );
      
      // Create PR
      const prResult = await this.mcp.github_createPullRequest(
        config.github.org,
        primaryRepo.repo,
        `[${ticketId}] ${ticket.summary}`,
        branchName,
        'main',
        this.generatePRDescription(plan, ticketId),
        true // draft
      );
      const pr = JSON.parse(prResult.content[0].text);
      
      // 5. Update Jira with PR link via Atlassian MCP
      await this.mcp.jira_addComment(ticketId, 
        `🤖 AI Agent created implementation plan.\n\n` +
        `**PR**: ${pr.html_url}\n\n` +
        `Please review and comment \`/approve\` to proceed.`
      );
      
      // Notify Slack with plan ready
      await this.slack.notifyPlanReady(
        { taskId, ticketId, title: ticket.summary, jiraUrl: `${config.jira.baseUrl}/browse/${ticketId}` },
        {
          repos: discovery.repos.map(r => r.repo),
          taskCount: plan.tasks.length,
          estimatedHours: plan.estimatedHours,
          prUrl: pr.html_url,
          prNumber: pr.number,
        }
      );
      
      return taskId;
      
    } catch (error) {
      await this.slack.postThreadUpdate(taskId, `Failed: ${error.message}`, '❌');
      throw error;
    }
  }

  async processSentryAlert(issueId: string, threshold: number): Promise<void> {
    // 1. Get issue details from Sentry MCP
    const issueResult = await this.mcp.sentry_getIssue(issueId);
    const issue = JSON.parse(issueResult.content[0].text);
    
    if (issue.count < threshold) return;
    
    // 2. Check if Jira ticket already exists
    const existingResult = await this.mcp.jira_searchIssues(`sentry-${issueId}`);
    const existing = JSON.parse(existingResult.content[0].text);
    
    if (existing.issues?.length > 0) return;
    
    // 3. Use Sentry's Seer for analysis (if available)
    let analysis = '';
    try {
      await this.mcp.sentry_invokeSeer(issueId);
      const seerResult = await this.mcp.sentry_getSeerStatus(issueId);
      analysis = JSON.parse(seerResult.content[0].text).analysis || '';
    } catch {
      // Seer not available or failed
    }
    
    // 4. Create Jira ticket via Atlassian MCP
    const ticketResult = await this.mcp.jira_createIssue(
      config.jira.projectKey,
      'Bug',
      `[Sentry] ${issue.title}`,
      this.formatSentryDescription(issue, analysis)
    );
    const ticket = JSON.parse(ticketResult.content[0].text);
    
    // 5. Add AI label to trigger the main flow
    await this.mcp.jira_updateIssue(ticket.key, {
      labels: [config.jira.aiLabel, 'bug', `sentry-${issueId}`],
    });
    
    // This will trigger the Jira webhook → processJiraTicket
  }

  async monitorCI(taskId: string, repo: string, prNumber: number): Promise<void> {
    await this.slack.postThreadUpdate(taskId, 'Monitoring CI pipeline...', '🔨');
    
    // Poll CI status via GitHub MCP
    const maxAttempts = 60;
    const pollInterval = 30000;
    let attempts = 0;
    
    while (attempts < maxAttempts) {
      const runsResult = await this.mcp.github_listWorkflowRuns(
        config.github.org,
        repo,
        'ci.yml'
      );
      const runs = JSON.parse(runsResult.content[0].text);
      const latestRun = runs.workflow_runs?.[0];
      
      if (!latestRun) {
        await new Promise(r => setTimeout(r, pollInterval));
        attempts++;
        continue;
      }
      
      if (latestRun.status === 'completed') {
        if (latestRun.conclusion === 'success') {
          await this.slack.notifyCIStatus(
            { taskId, ticketId: '', title: '', jiraUrl: '' },
            {
              prNumber,
              prUrl: `https://github.com/${config.github.org}/${repo}/pull/${prNumber}`,
              success: true,
            }
          );
          return;
        } else {
          // CI failed - analyze and attempt fix
          await this.handleCIFailure(taskId, repo, prNumber, latestRun.id);
          return;
        }
      }
      
      await new Promise(r => setTimeout(r, pollInterval));
      attempts++;
    }
  }

  private async handleCIFailure(
    taskId: string, 
    repo: string, 
    prNumber: number, 
    runId: number
  ): Promise<void> {
    // Get logs via GitHub MCP
    const logsResult = await this.mcp.github_getJobLogs(
      config.github.org,
      repo,
      runId
    );
    const logs = JSON.parse(logsResult.content[0].text);
    
    // Analyze with LLM
    const analysis = await this.analyzeCIFailure(logs);
    
    await this.slack.notifyCIStatus(
      { taskId, ticketId: '', title: '', jiraUrl: '' },
      {
        prNumber,
        prUrl: `https://github.com/${config.github.org}/${repo}/pull/${prNumber}`,
        success: false,
        failedChecks: analysis.failedChecks,
        autoFixAttempted: analysis.autoFixable,
      }
    );
    
    if (analysis.autoFixable) {
      await this.slack.postThreadUpdate(taskId, 'Attempting auto-fix...', '🔧');
      // Implement auto-fix logic
    }
  }

  // Helper methods...
  private generateTaskId(): string {
    return `task_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  private extractKeywords(summary: string, description: string): string {
    // Extract meaningful keywords for code search
    return `${summary} ${description || ''}`.slice(0, 200);
  }

  private slugify(text: string): string {
    return text.toLowerCase().replace(/[^a-z0-9]+/g, '-').slice(0, 50);
  }

  private async analyzeRelevantRepos(ticket: any, codeMatches: any): Promise<any> {
    // Use LLM to analyze and rank repos
    const response = await this.anthropic.messages.create({
      model: config.anthropic.model,
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: `Analyze which repositories are relevant for this task:

Task: ${ticket.summary}
Description: ${ticket.description}

Code search results:
${JSON.stringify(codeMatches, null, 2)}

Return JSON: { "repos": [{ "repo": "name", "relevance": 0.95, "reason": "..." }] }`
      }],
    });
    
    return JSON.parse(response.content[0].text);
  }

  private async createPlan(ticket: any, discovery: any): Promise<any> {
    // Use LLM to create implementation plan
    const response = await this.anthropic.messages.create({
      model: config.anthropic.model,
      max_tokens: 8192,
      messages: [{
        role: 'user',
        content: `Create implementation plan for:

Task: ${ticket.summary}
Description: ${ticket.description}

Relevant repos: ${JSON.stringify(discovery.repos)}

Return JSON with: title, scope, architecture, tasks, estimatedHours`
      }],
    });
    
    return JSON.parse(response.content[0].text);
  }

  private generatePlanMarkdown(plan: any, ticketId: string): string {
    return `# Implementation Plan: ${plan.title}

## Jira Ticket
[${ticketId}](${config.jira.baseUrl}/browse/${ticketId})

## Scope
${plan.scope?.inScope?.map(s => `- [ ] ${s}`).join('\n') || 'TBD'}

## Architecture
${plan.architecture || 'TBD'}

## Tasks
${plan.tasks?.map((t, i) => `${i + 1}. ${t.description}`).join('\n') || 'TBD'}

## Next Steps
- Review this plan
- Comment \`/approve\` to proceed
- Comment \`/modify [feedback]\` for changes
`;
  }

  private generatePRDescription(plan: any, ticketId: string): string {
    return `## ${plan.title}

**Jira**: [${ticketId}](${config.jira.baseUrl}/browse/${ticketId})

### Commands
- \`/approve\` - Approve and start implementation
- \`/modify [feedback]\` - Request changes

---
*🤖 Generated by AI Agent*`;
  }

  private formatSentryDescription(issue: any, analysis: string): string {
    return `## Sentry Issue

**Link**: ${issue.permalink}
**Count**: ${issue.count}
**First Seen**: ${issue.firstSeen}

## Stack Trace
\`\`\`
${issue.culprit || 'N/A'}
\`\`\`

${analysis ? `## AI Analysis\n${analysis}` : ''}
`;
  }

  private async analyzeCIFailure(logs: any): Promise<any> {
    // Analyze CI logs with LLM
    return {
      failedChecks: [],
      autoFixable: false,
    };
  }
}
```

---

## יתרונות וחסרונות

### יתרונות השימוש ב-MCPs רשמיים

| יתרון | הסבר |
|-------|------|
| **תחזוקה מינימלית** | Atlassian, GitHub, Sentry מתחזקים את ה-servers |
| **עדכונים אוטומטיים** | פיצ'רים חדשים מתווספים אוטומטית |
| **OAuth מובנה** | אין צורך לנהל tokens בעצמך |
| **אבטחה** | עומדים בסטנדרטים של החברות |
| **תמיכה רשמית** | יש לאן לפנות בבעיות |
| **תיעוד מלא** | כל ה-tools מתועדים |
| **Compliance** | עומדים בדרישות אבטחה ארגוניות |

### חסרונות

| חסרון | פתרון |
|-------|-------|
| **תלות בשרת חיצוני** | אפשר להריץ local (GitHub Docker, Sentry local) |
| **Rate limits** | עדיין כפופים ל-rate limits של ה-APIs |
| **OAuth flow** | דורש browser לאימות ראשוני |
| **פחות שליטה** | לא יכול להוסיף tools מותאמים אישית |
| **Slack חסר** | צריך לבנות Slack Agent בנפרד |

---

## עלויות

### עלויות עם MCPs רשמיים

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           COST BREAKDOWN V3                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  MCP SERVERS (Free!)                                                            │
│  ├── Atlassian Rovo MCP: $0 (included with Atlassian Cloud)                    │
│  ├── GitHub MCP: $0 (included with GitHub)                                      │
│  └── Sentry MCP: $0 (included with Sentry plan)                                │
│                                                                                  │
│  INFRASTRUCTURE                                                                  │
│  ├── Single server (t3.medium): $30-50/month                                   │
│  ├── Redis (cache): $15/month                                                   │
│  ├── PostgreSQL: $25/month                                                      │
│  └── Slack App: $0 (free tier)                                                  │
│                                                                                  │
│  LLM                                                                            │
│  └── Claude API (Sonnet): $15-30/month (depends on usage)                      │
│                                                                                  │
│  TOTAL: $85-120/month (vs $275-420 in V1)                                      │
│                                                                                  │
│  💰 SAVINGS: ~$200/month by using official MCPs!                               │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## סיכום

### מה השתנה מ-V1/V2

| רכיב | V1 (Custom) | V3 (Official MCPs) |
|------|-------------|-------------------|
| **Jira** | Custom MCP Server | Atlassian Rovo MCP ✅ |
| **GitHub** | Custom MCP Server | GitHub Official MCP ✅ |
| **Sentry** | Custom MCP Server | Sentry Official MCP ✅ |
| **Slack** | Custom MCP Server | Slack Bolt SDK (Web API) |
| **תחזוקה** | גבוהה | נמוכה |
| **עלות** | $275-420 | $85-120 |

### קונפיגורציה מינימלית

```json
{
  "mcpServers": {
    "atlassian": {
      "url": "https://mcp.atlassian.com/v1/mcp"
    },
    "github": {
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "sentry": {
      "url": "https://mcp.sentry.dev/mcp"
    }
  }
}
```

### המלצה סופית

**השתמש ב-MCPs הרשמיים!** 

זה:
- 🔒 יותר מאובטח
- 💰 יותר זול
- 🛠️ פחות תחזוקה
- 📚 מתועד טוב יותר
- ⚡ מהיר יותר להקמה

Slack הוא היוצא מן הכלל - צריך לבנות agent מותאם עם Bolt SDK, אבל זה פשוט וגמיש.
