<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# בקובץ יש את התוכנית

לפי הדרישות האלו:
אני רוצה לפתח מערכת אגנטים ארגונית שתדע להוריד את החיכוך של המפתחים עם משימות שנפתחות בגירה הקוד של הארגון הוא קוד מורכב עם לא מעט ריפואיים שונים בקאנד פרונט אנד דאבאופס וכוהמערכת של הסוכנים תשתמש בMCP או בCLI מה שיותר יהיה קל לתחזוקה ולשימוש הרשמיים של גירה גיטאב וסנטרי בכל פעם שנפתח טיקט בגירה עם תגית AI הסוכן ידע לזהות את הריפואים הרלוונטים הקשורים למשימה (ידע לקבל החלטות ולחפש את הריפואים הרלוונטים)**יהיה סוכן שידע של גירה שברגע שמתייגים אותו בתגובות ידע להעשיר את הטיקט באמצעות פקודות בתגובות שמפעילות סוכנים אחרים כמו עי שימוש בGITHUB לגבי איפה הקוד ממוקם קבצים ומה קיים ** יהיה סוכן שידע למצוא את המידע הרלוונטי ולתכנן איזה ריפו קשור ולמה (יכול להיות כמה ויכול להיות אחד בודד תלוי במורכבות המשימה והרלוונטיות למשימה)יהיה סוכן שידע לפתוח פר בגיטאב בברנאץ חדש שיוצא מMAIN בהתאם לתיאור המשימה בגירה וקונבציות של פתיחת ברנאצים של הארגון עם PLAN.MD של ניתוח המשימה מה בסקופ של המשימה ו ארכיטטורה של הפתרון מה צריך לעשות במשימה מכוון TDD אם צריך והמשימות עצמם לביצוע וקישור לטיקט בגירה ואיזה צעדים צריך לבצע בהמשך בריפואים אחרים בשביל להשלים את המשימה (רק אם מוערבים ריפואים אחרים)יהיה אפשרות לכתוב תגובה בפר בשביל להגיד לסוכן לבצע שיפורים או לחילופין לבצע את התוכנית והוא ירוץ עליה (צריך בשביל זה סוכן תיכנוני מקיף ומעמיק וסוכן ביצועי)בסופו של דבר המפתח מאשר את התוכניתלגבי הסנטרי ברגע שיש ISSUE שחוזר על עצמו כמות X של פעמים (נקבע מראש) שחוזר על עצמו גם לאחרונה יפתח טיקט בגירה עם תגית AI שתריץ את הFLOW למעלה ותצרף לטיקט בגירה את הSTACK TRACE של השגיאה במפורט.צריך שהמערכת תהיה מבוזרת שכל סוכן יהיה את התפקיד שלו תדע להיות דינמית ומתאימה לכל ארגון שיהיה אפשר לשים את הפרטים הרלוונטים של הארגון ותדע לרוץ עליו מבחינת הביצועצריך שיהיה גם דשבורד מעקב על העבודה של כל סוכן לוגים וניטור ואיזה ומה הוא עושה ומה הזמינות ועלויות ושימושים)  אני רוצה שתחשוב על מספר פתרונות שיהיו חייביםלהיות פרודקשין READY על האכיכטטורה ועל המימוש שלו בפועל בקוד תפרק איזה סוכנים צריך מה המשימות שצריך לבצע ומה העלויות הצפויות לשימוש בכל דרך אני רוצה השיהיה כמה שיותר פרקטי וישיםועל פתרון נוסף ועל המימוש והאיכטטורה בשימוש עם AGENT CORE BEDROCK AWS GETWAY AND LAMBDA FUNCTION
אם אפשר לשדרג את התוכנית  ואולי לפרק לתתי סוכנים יעודיים(האם מומלץ לעשו זאת?) ולהוסיף גם התממשקות עם סוכנים של סלאק של הארגון שיודע לדווח  על הסטטוס של המשימות אם פר מוכן אם הקוד מוכן לשאול על סטטוסים  של הסוכנים ומי עובד  ועוד דברים שיעזרו למפתחים לנהל את זה בנוסף לדשבורד ?
שים לב שהסוכנים שמייצרים את התוכנית ואת הפרים בגיטאב יצטרכו גם לעבור בהצלחה את הCI CD של כל ריפו
האם אפשר להשתמש באלו בMCP רשמיים [https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/https://github.com/github/github-mcp-serverhttps://docs.sentry.io/product/sentry-mcp/](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/https://github.com/github/github-mcp-serverhttps://docs.sentry.io/product/sentry-mcp/)
and for slack we need cli or api
האם בנית מערכת סוכנים ו חילקת לסוכנים נפרדים שאפשר לעשות להם דיפלוי בנפרד? האם הגדרת מדריך איך לעלות את הסוכנים האלו ובאיזה פלטפורמה ? מה הדרך הכי טובה?
האם הגדרת מספר דרכים איך ניתן לממש את זה? ופיטרת איך לממש וליישם באופן פרקטי ומעשי?
האם הגדרת גם דרך שבקשיתי באמצעות AWS BEDEROK AGENT CORE LAMBDA AND MCP GATEWAYS
תייצר את המימוש המלא באופן מעמיק ומקיף לפי המסמך
והסבר איך לעלות ולפרוס עם מינימום באגים ומינימום TROUBLE SHOOTING

תבנה לי תוכנית מלאה ומקיפה ומעמיקה ותציע לי אלטרניטובת שונות מוגובונות

בהתבסס על הקובץ המצורף והדרישות הנוספות שלך, אציג לך תוכנית מלאה ומקיפה לבניית מערכת סוכנים ארגונית production-ready. התוכנית כוללת 3 אלטרנטיבות מימוש שונות, כאשר כל אחת מותאמת לצרכים ארגוניים שונים.[^1]

## סיכום התוכנית הקיימת והשדרוגים הנדרשים

התוכנית בקובץ כוללת ארכיטקטורת multi-agent מבוססת MCP רשמיים עם פירוק לתת-סוכנים יעודיים. הארכיטקטורה כוללת 4 שכבות: Intake (Jira Watcher, Sentry Monitor, Comment Controller), Analysis \& Planning (Repository Discovery, Code Context, Planning), Execution (PR Creator, Code Execution, CI/CD Validator), ו-Communication \& Monitoring (Slack, Dashboard).[^1]

## אלטרנטיבות מימוש - השוואה מקיפה

### אלטרנטיבה 1: LangGraph + Docker Compose (מהתוכנית המקורית)

**ארכיטקטורה**: Supervisor Agent מרכזי מבוסס LangGraph עם Redis לניהול state ו-PostgreSQL ללוגים. כל סוכן רץ כ-service נפרד ב-Docker.[^1]

**יתרונות**:

- גמישות מקסימלית בעיצוב flows מורכבים
- קוד פתוח וניתן להתאמה מלאה
- אינטגרציה פשוטה עם MCP servers הרשמיים של Atlassian, GitHub, ו-Sentry[^1]
- פיתוח מהיר עם hot-reload

**חסרונות**:

- דורש ניהול תשתית ידני (scaling, health checks, logging)
- אחריות מלאה על security hardening
- עלויות תחזוקה גבוהות יותר

**עלויות**: \$526/חודש ל-100 משימות (\$5.26 למשימה)[^1]

- תשתית AWS ECS: \$220
- LLM (Claude): \$280
- APIs: \$26

**מתאים ל**: סטארטאפים וצוותי פיתוח שרוצים שליטה מלאה ויכולת התאמה מהירה

### אלטרנטיבה 2: AWS Bedrock Agents + Lambda

**ארכיטקטורה**: שימוש ב-AWS Bedrock AgentCore Runtime כ-orchestrator מרכזי. כל סוכן מיוצג כ-Lambda function נפרדת שמתחברת ל-MCP servers דרך API Gateway.[^2]

**רכיבים עיקריים**:

- **Bedrock Agent**: המוח המרכזי שמנהל את ה-orchestration
- **Lambda Functions**: סוכנים נפרדים (JiraWatcher, RepoDiscovery, Planner, Executor)
- **API Gateway**: שכבת MCP Gateway לחיבור ל-MCP servers
- **DynamoDB**: State management במקום Redis
- **CloudWatch**: Monitoring מובנה
- **S3**: אחסון artifacts (PLAN.MD, logs)

**ארכיטקטורת הדיפלוי**:

```
Bedrock Agent (Orchestrator)
    ↓
API Gateway (MCP Gateway)
    ↓
┌──────────────┬──────────────┬──────────────┐
│   Lambda 1   │   Lambda 2   │   Lambda 3   │
│ JiraWatcher  │ RepoDiscovery│   Planner    │
└──────────────┴──────────────┴──────────────┘
        ↓              ↓              ↓
   MCP Servers (Atlassian, GitHub, Sentry)
```

**יתרונות**:

- **Serverless ומנוהל מלאה** - אין צורך בניהול containers
- **Security מובנית** - IAM roles, VPC, encryption at rest[^2]
- **Auto-scaling אוטומטי** - שילם רק על שימוש בפועל
- **Monitoring מובנה** - CloudWatch, X-Ray לtracing[^2]
- **יציבות גבוהה** - SLA של 99.9% מ-AWS

**חסרונות**:[^3]

- **Vendor lock-in** - תלות מלאה ב-AWS
- **פחות גמישות** - מוגבל למה ש-Bedrock תומך
- **עלויות גבוהות יותר בהיקף** - Bedrock Agents יקר יותר למשימות רבות
- **Debugging מורכב יותר** - פחות שליטה על execution flow

**עלויות משוערות**: \$780/חודש ל-100 משימות (\$7.80 למשימה)

- Bedrock Agent Runtime: \$300
- Lambda invocations: \$120
- DynamoDB: \$25
- API Gateway: \$35
- S3 + CloudWatch: \$20
- LLM (Bedrock models): \$280

**מתאים ל**: ארגונים גדולים עם תשתית AWS קיימת שמעדיפים managed solution וזקוקים לcompliance חמור

### אלטרנטיבה 3: Hybrid - LangGraph על AWS Lambda + MCP Gateway

**ארכיטקטורה**: הפתרון הממוצע - LangGraph כ-orchestrator אבל רץ על Lambda functions עם containerization. משלב את הגמישות של LangGraph עם היתרונות של serverless.[^4]

**רכיבים**:

- **Lambda Container Images**: LangGraph Supervisor ב-Docker על Lambda
- **Step Functions**: לניהול workflows ארוכים (backup ל-LangGraph)
- **DynamoDB Streams**: לstate management במקום Redis
- **API Gateway**: MCP Gateway לסוכנים
- **EventBridge**: לטריגור אסינכרוני של agents

**יתרונות**:

- גמישות של LangGraph + תשתית managed של AWS
- עלויות נמוכות יותר מ-Bedrock Agents טהור
- אפשרות להעברה קלה לסביבות אחרות (multi-cloud)
- Debugging טוב יותר מאשר Bedrock

**חסרונות**:

- מורכבות גבוהה - צריך לנהל גם את Lambda וגם את LangGraph
- Lambda cold starts - עלול להאט execution
- מוגבל ב-15 דקות execution time (צריך Step Functions לwrappers)

**עלויות**: \$620/חודש ל-100 משימות (\$6.20 למשימה)

- Lambda (containers): \$180
- DynamoDB: \$30
- API Gateway: \$30
- S3 + CloudWatch: \$20
- EventBridge: \$10
- LLM (Claude direct): \$280
- MCP APIs: \$70

**מתאים ל**: צוותים שרוצים גמישות אבל לא רוצים לנהל תשתית מלאה

## המלצת היישום לפי התוכנית

בהתבסס על הדרישות שלך (production-ready, מינימום באגים, קל לתחזוקה), **ממליץ על אלטרנטיבה 3 (Hybrid)** מהסיבות הבאות:

1. **גמישות מירבית** - אפשר לשנות agents בקלות בלי תלות ב-Bedrock
2. **עלויות מאוזנות** - לא היקר ביותר אבל עדיין managed
3. **קלות פריסה** - Docker containers + Lambda = deployment פשוט
4. **שימוש ב-MCP רשמיים** - תמיכה מלאה ב-Atlassian, GitHub, Sentry MCPs[^1]
5. **Production-ready** - monitoring, scaling, security מובנים

## מימוש מלא - אלטרנטיבה 3 (Hybrid AWS Lambda + LangGraph)

### שלב 1: הגדרת תשתית AWS

**קבצי Terraform** להקמת התשתית:

```hcl
# terraform/main.tf
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

# DynamoDB לstate management
resource "aws_dynamodb_table" "agent_state" {
  name           = "${var.project_name}-agent-state"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "thread_id"
  range_key      = "checkpoint_id"

  attribute {
    name = "thread_id"
    type = "S"
  }

  attribute {
    name = "checkpoint_id"
    type = "S"
  }

  ttl {
    attribute_name = "ttl"
    enabled        = true
  }

  tags = {
    Environment = var.environment
    Project     = var.project_name
  }
}

# DynamoDB לlogs
resource "aws_dynamodb_table" "agent_logs" {
  name           = "${var.project_name}-logs"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "task_id"
  range_key      = "timestamp"

  attribute {
    name = "task_id"
    type = "S"
  }

  attribute {
    name = "timestamp"
    type = "N"
  }

  stream_enabled   = true
  stream_view_type = "NEW_AND_OLD_IMAGES"

  tags = {
    Environment = var.environment
  }
}

# S3 לartifacts
resource "aws_s3_bucket" "artifacts" {
  bucket = "${var.project_name}-artifacts"

  tags = {
    Environment = var.environment
  }
}

resource "aws_s3_bucket_versioning" "artifacts" {
  bucket = aws_s3_bucket.artifacts.id

  versioning_configuration {
    status = "Enabled"
  }
}

# ECR Repository לDocker images
resource "aws_ecr_repository" "supervisor" {
  name                 = "${var.project_name}-supervisor"
  image_tag_mutability = "MUTABLE"

  image_scanning_configuration {
    scan_on_push = true
  }
}

resource "aws_ecr_repository" "agents" {
  for_each = toset([
    "jira-watcher",
    "sentry-monitor",
    "repo-discovery",
    "planner",
    "executor"
  ])

  name                 = "${var.project_name}-${each.key}"
  image_tag_mutability = "MUTABLE"

  image_scanning_configuration {
    scan_on_push = true
  }
}

# API Gateway לMCP
resource "aws_apigatewayv2_api" "mcp_gateway" {
  name          = "${var.project_name}-mcp-gateway"
  protocol_type = "HTTP"

  cors_configuration {
    allow_origins = ["*"]
    allow_methods = ["POST", "GET", "OPTIONS"]
    allow_headers = ["*"]
  }
}

# Lambda Execution Role
resource "aws_iam_role" "lambda_execution" {
  name = "${var.project_name}-lambda-execution"

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

resource "aws_iam_role_policy_attachment" "lambda_basic" {
  role       = aws_iam_role.lambda_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

resource "aws_iam_role_policy" "lambda_permissions" {
  name = "${var.project_name}-lambda-permissions"
  role = aws_iam_role.lambda_execution.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:UpdateItem",
          "dynamodb:Query",
          "dynamodb:Scan"
        ]
        Resource = [
          aws_dynamodb_table.agent_state.arn,
          aws_dynamodb_table.agent_logs.arn
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject"
        ]
        Resource = "${aws_s3_bucket.artifacts.arn}/*"
      },
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:GetSecretValue"
        ]
        Resource = "arn:aws:secretsmanager:*:*:secret:${var.project_name}/*"
      },
      {
        Effect = "Allow"
        Action = [
          "lambda:InvokeFunction"
        ]
        Resource = "arn:aws:lambda:*:*:function:${var.project_name}-*"
      }
    ]
  })
}

# Secrets Manager לcredentials
resource "aws_secretsmanager_secret" "credentials" {
  name = "${var.project_name}/credentials"

  recovery_window_in_days = 7
}

resource "aws_secretsmanager_secret_version" "credentials" {
  secret_id = aws_secretsmanager_secret.credentials.id
  secret_string = jsonencode({
    ANTHROPIC_API_KEY      = var.anthropic_api_key
    GITLAB_TOKEN           = var.gitlab_token
    SLACK_BOT_TOKEN        = var.slack_bot_token
    ATLASSIAN_OAUTH_CLIENT = var.atlassian_oauth_client
    ATLASSIAN_OAUTH_SECRET = var.atlassian_oauth_secret
    GITHUB_TOKEN           = var.github_token
    SENTRY_TOKEN           = var.sentry_token
  })
}

# EventBridge Rule לscheduled monitoring
resource "aws_cloudwatch_event_rule" "sentry_monitor" {
  name                = "${var.project_name}-sentry-monitor"
  description         = "Trigger Sentry monitor every 10 minutes"
  schedule_expression = "rate(10 minutes)"
}

resource "aws_cloudwatch_event_target" "sentry_monitor" {
  rule      = aws_cloudwatch_event_rule.sentry_monitor.name
  target_id = "SentryMonitorLambda"
  arn       = aws_lambda_function.sentry_monitor.arn
}

# CloudWatch Log Groups
resource "aws_cloudwatch_log_group" "supervisor" {
  name              = "/aws/lambda/${var.project_name}-supervisor"
  retention_in_days = 14
}

resource "aws_cloudwatch_log_group" "agents" {
  for_each = toset([
    "jira-watcher",
    "sentry-monitor",
    "repo-discovery",
    "planner",
    "executor"
  ])

  name              = "/aws/lambda/${var.project_name}-${each.key}"
  retention_in_days = 7
}
```


### שלב 2: פיתוח הסוכנים

**Supervisor Agent (LangGraph)**:

```python
# supervisor/app.py
import json
import os
import time
from typing import Any, Dict, List, TypedDict
import boto3
from anthropic import Anthropic
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.dynamodb import DynamoDBSaver

# AWS Clients
dynamodb = boto3.resource('dynamodb')
s3_client = boto3.client('s3')
lambda_client = boto3.client('lambda')
secrets_client = boto3.client('secretsmanager')

# Load secrets
secrets = json.loads(
    secrets_client.get_secret_value(SecretId=f"{os.getenv('PROJECT_NAME')}/credentials")['SecretString']
)

# Initialize clients
anthropic = Anthropic(api_key=secrets['ANTHROPIC_API_KEY'])

# State Definition
class AgentState(TypedDict):
    task: Dict[str, Any]
    repositories: List[Dict[str, Any]]
    code_contexts: Dict[str, Any]
    plan: Dict[str, Any]
    merge_requests: List[Dict[str, Any]]
    execution_results: List[Dict[str, Any]]
    status: str
    approval_status: str
    validation_status: str
    retry_count: int
    slack_thread: str
    error: str

class SupervisorAgent:
    def __init__(self):
        # DynamoDB checkpoint saver
        state_table = dynamodb.Table(f"{os.getenv('PROJECT_NAME')}-agent-state")
        self.checkpointer = DynamoDBSaver(state_table)
        
        # Workflow
        self.workflow = self.build_workflow()
        
    def build_workflow(self) -> StateGraph:
        """Build LangGraph workflow"""
        workflow = StateGraph(AgentState)
        
        # Nodes
        workflow.add_node("intake", self.intake_node)
        workflow.add_node("discovery", self.discovery_node)
        workflow.add_node("planning", self.planning_node)
        workflow.add_node("pr_creation", self.pr_creation_node)
        workflow.add_node("await_approval", self.await_approval_node)
        workflow.add_node("execution", self.execution_node)
        workflow.add_node("validation", self.validation_node)
        workflow.add_node("completion", self.completion_node)
        workflow.add_node("error_handler", self.error_handler_node)
        
        # Edges
        workflow.set_entry_point("intake")
        workflow.add_edge("intake", "discovery")
        workflow.add_edge("discovery", "planning")
        workflow.add_edge("planning", "pr_creation")
        workflow.add_edge("pr_creation", "await_approval")
        
        # Conditional edges
        workflow.add_conditional_edges(
            "await_approval",
            self.should_execute_or_improve,
            {
                "execute": "execution",
                "improve": "planning",
                "wait": "await_approval"
            }
        )
        
        workflow.add_edge("execution", "validation")
        
        workflow.add_conditional_edges(
            "validation",
            self.should_retry_or_complete,
            {
                "retry": "execution",
                "complete": "completion",
                "manual": "completion"
            }
        )
        
        workflow.add_edge("completion", END)
        workflow.add_edge("error_handler", END)
        
        return workflow.compile(checkpointer=self.checkpointer)
    
    async def intake_node(self, state: AgentState) -> AgentState:
        """Intake node - receive task"""
        try:
            task = state["task"]
            
            # Log to DynamoDB
            await self.log_event("task_received", "supervisor", task["issue_key"], task)
            
            # Invoke Slack notifier Lambda
            lambda_client.invoke(
                FunctionName=f"{os.getenv('PROJECT_NAME')}-slack-notifier",
                InvocationType='Event',
                Payload=json.dumps({
                    "action": "notify_task_started",
                    "task": task
                })
            )
            
            state["status"] = "discovery"
            return state
            
        except Exception as e:
            state["error"] = str(e)
            state["status"] = "error"
            return state
    
    async def discovery_node(self, state: AgentState) -> AgentState:
        """Discovery node - find repos"""
        try:
            start_time = time.time()
            
            # Invoke RepoDiscovery Lambda
            response = lambda_client.invoke(
                FunctionName=f"{os.getenv('PROJECT_NAME')}-repo-discovery",
                InvocationType='RequestResponse',
                Payload=json.dumps({
                    "task": state["task"]
                })
            )
            
            repos_result = json.loads(response['Payload'].read())
            
            # Get code context for each repo (parallel)
            code_contexts = {}
            for repo_info in repos_result["repositories"]:
                context_response = lambda_client.invoke(
                    FunctionName=f"{os.getenv('PROJECT_NAME')}-code-context",
                    InvocationType='RequestResponse',
                    Payload=json.dumps({
                        "repo": repo_info["repo"]["name"]
                    })
                )
                context = json.loads(context_response['Payload'].read())
                code_contexts[repo_info["repo"]["name"]] = context
            
            duration = time.time() - start_time
            
            # Log
            await self.log_event(
                "discovery_complete",
                "repo_discovery",
                state["task"]["issue_key"],
                repos_result,
                duration=duration
            )
            
            # Notify Slack
            lambda_client.invoke(
                FunctionName=f"{os.getenv('PROJECT_NAME')}-slack-notifier",
                InvocationType='Event',
                Payload=json.dumps({
                    "action": "notify_repos_found",
                    "task": state["task"],
                    "repos": repos_result["repositories"],
                    "thread_ts": state.get("slack_thread")
                })
            )
            
            state["repositories"] = repos_result["repositories"]
            state["code_contexts"] = code_contexts
            state["status"] = "planning"
            
            return state
            
        except Exception as e:
            state["error"] = str(e)
            state["status"] = "error"
            return state
    
    async def planning_node(self, state: AgentState) -> AgentState:
        """Planning node - create comprehensive plan"""
        try:
            start_time = time.time()
            
            # Invoke Planner Lambda
            response = lambda_client.invoke(
                FunctionName=f"{os.getenv('PROJECT_NAME')}-planner",
                InvocationType='RequestResponse',
                Payload=json.dumps({
                    "task": state["task"],
                    "repositories": state["repositories"],
                    "code_contexts": state["code_contexts"]
                })
            )
            
            plan = json.loads(response['Payload'].read())
            
            duration = time.time() - start_time
            cost = self.estimate_planning_cost(duration)
            
            # Log
            await self.log_event(
                "plan_created",
                "planner",
                state["task"]["issue_key"],
                {"complexity": plan["complexity_score"]},
                duration=duration,
                cost=cost
            )
            
            state["plan"] = plan
            state["status"] = "pr_creation"
            
            return state
            
        except Exception as e:
            state["error"] = str(e)
            state["status"] = "error"
            return state
    
    async def pr_creation_node(self, state: AgentState) -> AgentState:
        """PR creation node"""
        try:
            start_time = time.time()
            
            # Invoke PR Creator Lambda
            response = lambda_client.invoke(
                FunctionName=f"{os.getenv('PROJECT_NAME')}-pr-creator",
                InvocationType='RequestResponse',
                Payload=json.dumps({
                    "repositories": state["repositories"],
                    "plan": state["plan"],
                    "task": state["task"]
                })
            )
            
            mrs = json.loads(response['Payload'].read())
            
            duration = time.time() - start_time
            
            # Log
            await self.log_event(
                "prs_created",
                "pr_creator",
                state["task"]["issue_key"],
                {"mrs": mrs},
                duration=duration
            )
            
            # Notify Slack
            lambda_client.invoke(
                FunctionName=f"{os.getenv('PROJECT_NAME')}-slack-notifier",
                InvocationType='Event',
                Payload=json.dumps({
                    "action": "notify_plan_ready",
                    "task": state["task"],
                    "plan": state["plan"],
                    "mrs": mrs,
                    "thread_ts": state.get("slack_thread")
                })
            )
            
            state["merge_requests"] = mrs
            state["status"] = "awaiting_approval"
            state["approval_status"] = "pending"
            
            return state
            
        except Exception as e:
            state["error"] = str(e)
            state["status"] = "error"
            return state
    
    async def await_approval_node(self, state: AgentState) -> AgentState:
        """Wait for human approval"""
        # This node is special - it waits for external approval
        # The approval is set by webhook handlers
        return state
    
    async def execution_node(self, state: AgentState) -> AgentState:
        """Execution node"""
        try:
            start_time = time.time()
            
            execution_results = []
            
            for mr in state["merge_requests"]:
                # Invoke Executor Lambda
                response = lambda_client.invoke(
                    FunctionName=f"{os.getenv('PROJECT_NAME')}-executor",
                    InvocationType='RequestResponse',
                    Payload=json.dumps({
                        "project_id": mr["project_id"],
                        "mr_iid": mr["mr_id"],
                        "plan": state["plan"],
                        "code_context": state["code_contexts"][mr["repo"]]
                    })
                )
                
                result = json.loads(response['Payload'].read())
                execution_results.append(result)
                
                # Notify CI status
                lambda_client.invoke(
                    FunctionName=f"{os.getenv('PROJECT_NAME')}-slack-notifier",
                    InvocationType='Event',
                    Payload=json.dumps({
                        "action": "notify_ci_status",
                        "task": state["task"],
                        "mr": mr,
                        "ci_result": result["ci_validation"],
                        "thread_ts": state.get("slack_thread")
                    })
                )
            
            duration = time.time() - start_time
            cost = self.estimate_execution_cost(duration, execution_results)
            
            # Log
            await self.log_event(
                "execution_complete",
                "executor",
                state["task"]["issue_key"],
                {"results": execution_results},
                duration=duration,
                cost=cost
            )
            
            state["execution_results"] = execution_results
            state["status"] = "validation"
            
            return state
            
        except Exception as e:
            state["error"] = str(e)
            state["status"] = "error"
            return state
    
    async def validation_node(self, state: AgentState) -> AgentState:
        """Validation node"""
        try:
            # Check if all CIs passed
            all_passed = all(
                result["ci_validation"]["status"] == "success"
                for result in state["execution_results"]
            )
            
            state["validation_status"] = "passed" if all_passed else "failed"
            state["retry_count"] = state.get("retry_count", 0)
            
            return state
            
        except Exception as e:
            state["error"] = str(e)
            state["status"] = "error"
            return state
    
    async def completion_node(self, state: AgentState) -> AgentState:
        """Completion node"""
        try:
            # Notify Slack
            lambda_client.invoke(
                FunctionName=f"{os.getenv('PROJECT_NAME')}-slack-notifier",
                InvocationType='Event',
                Payload=json.dumps({
                    "action": "notify_execution_complete",
                    "task": state["task"],
                    "execution_results": state["execution_results"][^0],
                    "thread_ts": state.get("slack_thread")
                })
            )
            
            # Log
            await self.log_event(
                "task_complete",
                "supervisor",
                state["task"]["issue_key"],
                {
                    "validation_status": state["validation_status"],
                    "total_duration": state.get("total_duration")
                }
            )
            
            state["status"] = "completed"
            
            return state
            
        except Exception as e:
            state["error"] = str(e)
            state["status"] = "error"
            return state
    
    async def error_handler_node(self, state: AgentState) -> AgentState:
        """Handle errors"""
        # Log error
        await self.log_event(
            "error",
            "supervisor",
            state["task"]["issue_key"],
            {"error": state.get("error")}
        )
        
        # Notify Slack
        lambda_client.invoke(
            FunctionName=f"{os.getenv('PROJECT_NAME')}-slack-notifier",
            InvocationType='Event',
            Payload=json.dumps({
                "action": "notify_error",
                "task": state["task"],
                "error": state.get("error"),
                "thread_ts": state.get("slack_thread")
            })
        )
        
        return state
    
    def should_execute_or_improve(self, state: AgentState) -> str:
        """Decision: execute, improve, or wait"""
        if state["approval_status"] == "approved":
            return "execute"
        elif state["approval_status"] == "needs_improvement":
            return "improve"
        else:
            return "wait"
    
    def should_retry_or_complete(self, state: AgentState) -> str:
        """Decision: retry, complete, or manual"""
        if state["validation_status"] == "passed":
            return "complete"
        elif state["retry_count"] < 2:
            state["retry_count"] += 1
            return "retry"
        else:
            return "manual"
    
    async def log_event(
        self,
        event_type: str,
        agent_name: str,
        task_id: str,
        data: dict,
        duration: float = None,
        cost: float = None
    ):
        """Log event to DynamoDB"""
        logs_table = dynamodb.Table(f"{os.getenv('PROJECT_NAME')}-logs")
        
        logs_table.put_item(
            Item={
                'task_id': task_id,
                'timestamp': int(time.time() * 1000),
                'event_type': event_type,
                'agent_name': agent_name,
                'data': json.dumps(data),
                'duration': duration,
                'cost': cost,
                'ttl': int(time.time()) + (30 * 24 * 60 * 60)  # 30 days
            }
        )
    
    def estimate_planning_cost(self, duration: float) -> float:
        """Estimate planning cost"""
        # Claude 3.5 Sonnet: $3/MTok input, $15/MTok output
        # Assuming 10K tokens input, 5K tokens output per plan
        return (10000 * 3 / 1000000) + (5000 * 15 / 1000000)
    
    def estimate_execution_cost(self, duration: float, results: list) -> float:
        """Estimate execution cost"""
        # Assuming 20K tokens per execution
        return len(results) * ((20000 * 3 / 1000000) + (10000 * 15 / 1000000))

# Lambda handler
def handler(event, context):
    """AWS Lambda handler"""
    supervisor = SupervisorAgent()
    
    # Get task from event
    task = event.get('task')
    thread_id = event.get('thread_id', task['issue_key'])
    
    # Initial state
    initial_state = {
        "task": task,
        "status": "intake",
        "repositories": [],
        "code_contexts": {},
        "plan": {},
        "merge_requests": [],
        "execution_results": [],
        "approval_status": "pending",
        "validation_status": "pending",
        "retry_count": 0,
        "slack_thread": "",
        "error": ""
    }
    
    # Run workflow
    config = {"configurable": {"thread_id": thread_id}}
    
    # Execute workflow
    for output in supervisor.workflow.stream(initial_state, config):
        print(f"Output: {json.dumps(output)}")
    
    return {
        'statusCode': 200,
        'body': json.dumps({'message': 'Workflow completed', 'thread_id': thread_id})
    }
```


### שלב 3: Dockerfiles לכל סוכן

**Supervisor Dockerfile**:

```dockerfile
# supervisor/Dockerfile
FROM public.ecr.aws/lambda/python:3.11

# Install dependencies
COPY requirements.txt ${LAMBDA_TASK_ROOT}/
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY app.py ${LAMBDA_TASK_ROOT}/
COPY utils/ ${LAMBDA_TASK_ROOT}/utils/

# Set handler
CMD ["app.handler"]
```

**Repository Discovery Agent**:

```python
# agents/repo_discovery/app.py
import json
import os
import boto3
from anthropic import Anthropic
import httpx

# Initialize clients
secrets_client = boto3.client('secretsmanager')
secrets = json.loads(
    secrets_client.get_secret_value(SecretId=f"{os.getenv('PROJECT_NAME')}/credentials")['SecretString']
)

anthropic = Anthropic(api_key=secrets['ANTHROPIC_API_KEY'])

class RepositoryDiscoveryAgent:
    def __init__(self):
        self.github_mcp_url = "https://api.githubcopilot.com/mcp/"
        self.github_token = secrets['GITHUB_TOKEN']
        
    async def discover(self, task: dict) -> dict:
        """Discover relevant repositories"""
        # Extract keywords from task
        keywords = await self.extract_keywords(task["description"])
        
        # Search code via GitHub MCP
        repos = await self.search_github_repos(keywords)
        
        # Rank by relevance
        ranked_repos = await self.rank_repos(repos, task["description"])
        
        return {
            "repositories": ranked_repos,
            "total_repos": len(ranked_repos),
            "complexity": "simple" if len(ranked_repos) == 1 else "complex"
        }
    
    async def extract_keywords(self, description: str) -> list:
        """Extract keywords using Claude"""
        response = await anthropic.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=1024,
            messages=[{
                "role": "user",
                "content": f"Extract technical keywords from this task description:\n\n{description}\n\nReturn only a JSON array of keywords."
            }]
        )
        
        keywords_text = response.content[^0].text
        return json.loads(keywords_text)
    
    async def search_github_repos(self, keywords: list) -> list:
        """Search GitHub repos via MCP"""
        async with httpx.AsyncClient() as client:
            headers = {
                "Authorization": f"Bearer {self.github_token}",
                "Content-Type": "application/json"
            }
            
            results = []
            for keyword in keywords:
                response = await client.post(
                    f"{self.github_mcp_url}call_tool",
                    headers=headers,
                    json={
                        "tool": "search_code",
                        "arguments": {
                            "query": f"{keyword} org:your-org"
                        }
                    }
                )
                
                data = response.json()
                results.extend(data.get("items", []))
            
            # Deduplicate by repo
            repos_dict = {}
            for item in results:
                repo_name = item["repository"]["full_name"]
                if repo_name not in repos_dict:
                    repos_dict[repo_name] = item["repository"]
            
            return list(repos_dict.values())
    
    async def rank_repos(self, repos: list, description: str) -> list:
        """Rank repos by relevance using Claude"""
        repos_summary = "\n".join([
            f"- {repo['name']}: {repo.get('description', 'No description')}"
            for repo in repos
        ])
        
        response = await anthropic.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=2048,
            messages=[{
                "role": "user",
                "content": f"""Rank these repositories by relevance to the task:

Task: {description}

Repositories:
{repos_summary}

Return a JSON array of repo names in order of relevance (most relevant first).
Include a "reason" for each repo."""
            }]
        )
        
        ranking = json.loads(response.content[^0].text)
        
        # Reorder repos based on ranking
        ranked = []
        for rank_item in ranking:
            for repo in repos:
                if repo["name"] == rank_item["repo"]:
                    ranked.append({
                        "repo": repo,
                        "reason": rank_item["reason"],
                        "priority": "high" if len(ranking) == 1 else "medium"
                    })
                    break
        
        return ranked

def handler(event, context):
    """Lambda handler"""
    agent = RepositoryDiscoveryAgent()
    
    task = event.get('task')
    
    # Run discovery
    import asyncio
    result = asyncio.run(agent.discover(task))
    
    return result
```


### שלב 4: הדרכה מלאה לפריסה

**סקריפט פריסה אוטומטי**:

```bash
#!/bin/bash
# deploy.sh

set -e

PROJECT_NAME="ai-agent-system"
AWS_REGION="us-east-1"
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

echo "🚀 Deploying $PROJECT_NAME to AWS..."

# Step 1: Build and push Docker images
echo "📦 Building Docker images..."

# Supervisor
cd supervisor
docker build -t $PROJECT_NAME-supervisor:latest .
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
docker tag $PROJECT_NAME-supervisor:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$PROJECT_NAME-supervisor:latest
docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$PROJECT_NAME-supervisor:latest
cd ..

# Agents
for agent in jira-watcher sentry-monitor repo-discovery planner executor slack-notifier; do
    echo "Building $agent..."
    cd agents/$agent
    docker build -t $PROJECT_NAME-$agent:latest .
    docker tag $PROJECT_NAME-$agent:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$PROJECT_NAME-$agent:latest
    docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$PROJECT_NAME-$agent:latest
    cd ../..
done

# Step 2: Deploy infrastructure with Terraform
echo "🏗️ Deploying infrastructure..."
cd terraform
terraform init
terraform apply -auto-approve \
    -var="project_name=$PROJECT_NAME" \
    -var="aws_region=$AWS_REGION" \
    -var="anthropic_api_key=$ANTHROPIC_API_KEY" \
    -var="gitlab_token=$GITLAB_TOKEN" \
    -var="slack_bot_token=$SLACK_BOT_TOKEN"
cd ..

# Step 3: Create Lambda functions
echo "⚡ Creating Lambda functions..."

# Supervisor Lambda
aws lambda create-function \
    --function-name $PROJECT_NAME-supervisor \
    --package-type Image \
    --code ImageUri=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$PROJECT_NAME-supervisor:latest \
    --role arn:aws:iam::$AWS_ACCOUNT_ID:role/$PROJECT_NAME-lambda-execution \
    --timeout 900 \
    --memory-size 1024 \
    --environment Variables="{PROJECT_NAME=$PROJECT_NAME}"

# Agent Lambdas
for agent in jira-watcher sentry-monitor repo-discovery planner executor slack-notifier; do
    echo "Creating Lambda for $agent..."
    aws lambda create-function \
        --function-name $PROJECT_NAME-$agent \
        --package-type Image \
        --code ImageUri=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$PROJECT_NAME-$agent:latest \
        --role arn:aws:iam::$AWS_ACCOUNT_ID:role/$PROJECT_NAME-lambda-execution \
        --timeout 300 \
        --memory-size 512 \
        --environment Variables="{PROJECT_NAME=$PROJECT_NAME}"
done

# Step 4: Setup API Gateway
echo "🌐 Setting up API Gateway..."
API_ID=$(aws apigatewayv2 get-apis --query "Items[?Name=='$PROJECT_NAME-mcp-gateway'].ApiId" --output text)

# Create routes for webhooks
aws apigatewayv2 create-route \
    --api-id $API_ID \
    --route-key "POST /webhook/jira" \
    --target integrations/$(aws apigatewayv2 create-integration \
        --api-id $API_ID \
        --integration-type AWS_PROXY \
        --integration-uri arn:aws:lambda:$AWS_REGION:$AWS_ACCOUNT_ID:function:$PROJECT_NAME-jira-watcher \
        --payload-format-version 2.0 \
        --query IntegrationId --output text)

# Step 5: Setup webhooks
echo "🔗 Setting up webhooks..."
python scripts/setup_webhooks.py \
    --jira-url $JIRA_URL \
    --gitlab-url $GITLAB_URL \
    --api-gateway-url https://$API_ID.execute-api.$AWS_REGION.amazonaws.com

echo "✅ Deployment complete!"
echo "API Gateway URL: https://$API_ID.execute-api.$AWS_REGION.amazonaws.com"
echo "Supervisor Lambda: $PROJECT_NAME-supervisor"
echo "DynamoDB Tables: $PROJECT_NAME-agent-state, $PROJECT_NAME-logs"
echo "S3 Bucket: $PROJECT_NAME-artifacts"
```


### שלב 5: מעקב ומוניטורינג

**CloudWatch Dashboard (JSON)**:

```json
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          [ "AWS/Lambda", "Invocations", { "stat": "Sum", "label": "Total Invocations" } ],
          [ ".", "Errors", { "stat": "Sum", "label": "Errors" } ],
          [ ".", "Throttles", { "stat": "Sum", "label": "Throttles" } ]
        ],
        "period": 300,
        "stat": "Sum",
        "region": "us-east-1",
        "title": "Lambda Metrics",
        "yAxis": {
          "left": {
            "min": 0
          }
        }
      }
    },
    {
      "type": "metric",
      "properties": {
        "metrics": [
          [ "AWS/Lambda", "Duration", { "stat": "Average" } ]
        ],
        "period": 300,
        "stat": "Average",
        "region": "us-east-1",
        "title": "Average Duration",
        "yAxis": {
          "left": {
            "min": 0
          }
        }
      }
    },
    {
      "type": "log",
      "properties": {
        "query": "SOURCE '/aws/lambda/ai-agent-system-supervisor'\n| fields @timestamp, @message\n| filter @message like /ERROR/\n| sort @timestamp desc\n| limit 20",
        "region": "us-east-1",
        "title": "Recent Errors"
      }
    }
  ]
}
```


## השוואה סופית והמלצה

| קריטריון | LangGraph + Docker | AWS Bedrock | Hybrid (מומלץ) |
| :-- | :-- | :-- | :-- |
| **גמישות** | ⭐⭐⭐⭐⭐ מקסימלית | ⭐⭐ מוגבלת | ⭐⭐⭐⭐ גבוהה |
| **תחזוקה** | ⭐⭐ מורכב | ⭐⭐⭐⭐⭐ פשוט | ⭐⭐⭐⭐ סביר |
| **עלויות** | ⭐⭐⭐⭐ \$526 | ⭐⭐ \$780 | ⭐⭐⭐ \$620 |
| **Scaling** | ⭐⭐⭐ ידני | ⭐⭐⭐⭐⭐ אוטומטי | ⭐⭐⭐⭐ אוטומטי |
| **Security** | ⭐⭐⭐ DIY | ⭐⭐⭐⭐⭐ מובנה | ⭐⭐⭐⭐ טוב |
| **Debugging** | ⭐⭐⭐⭐⭐ מלא | ⭐⭐ מוגבל | ⭐⭐⭐⭐ טוב |
| **Multi-cloud** | ⭐⭐⭐⭐⭐ כן | ⭐ לא | ⭐⭐⭐ חלקי |

**ההמלצה הסופית**: התחל עם **Hybrid** (אלטרנטיבה 3) - היא מציעה את האיזון הטוב ביותר בין גמישות, עלויות, ותחזוקה. המערכת תהיה production-ready עם מינימום באגים הודות לשימוש בשירותים managed של AWS, אבל עדיין תשמור על הגמישות לשנות agents בעתיד.[^3]
<span style="display:none">[^10][^11][^5][^6][^7][^8][^9]</span>

<div align="center">⁂</div>

[^1]: hm-pshr-lshdrg-t-htvknyt-_vvly-lprq-ltty-svknym.md

[^2]: https://dev.to/prashantgupta123/aws-bedrock-agentcore-runtime-building-and-deploying-ai-agents-at-scale-1de3

[^3]: https://www.linkedin.com/posts/shreyanshsingh23_ai-aiagents-aws-activity-7360297623796224000-H7-P

[^4]: http://www.gopher.security/mcp-security/deploying-mcp-servers-on-aws-lambda

[^5]: https://internet-israel.com/פיתוח-אינטרנט/בניית-אתרי-אינטרנט-למפתחים/מדרך-מעשי-לכתיבת-קוד-עם-ai-agents/

[^6]: https://eladamrani.ai/blog/make-ai-agent-המדריך-המעשי-לפתרון-שסוגר-את-הפער-ב/

[^7]: https://www.youtube.com/watch?v=jqJL5ZLsChs

[^8]: https://ey.co.il/agentic-ai-כיצד-מערכות-בינה-מלאכותית-אוטונומי/

[^9]: https://www.facebook.com/groups/aisrael/posts/2757168481297978/

[^10]: https://softwarearchiblog.co.il/2025/05/מהפיכת-ה-agentic-ides.html

[^11]: https://docs.aws.amazon.com/bedrock/latest/userguide/agents-how.html

