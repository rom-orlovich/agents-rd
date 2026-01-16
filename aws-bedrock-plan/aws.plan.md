
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
