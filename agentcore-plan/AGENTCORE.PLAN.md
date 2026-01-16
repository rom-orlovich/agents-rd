# AWS Bedrock AgentCore - Complete Implementation Plan

## Executive Summary

Amazon Bedrock AgentCore is a comprehensive agentic platform for building, deploying, and operating AI agents at scale with zero infrastructure management. This document provides a complete implementation guide with production-ready code examples.

## What is AgentCore?

AgentCore is a modular, framework-agnostic platform that provides:

| Service | Description | Key Features |
|---------|-------------|--------------|
| **Runtime** | Serverless agent deployment | Session isolation, 8-hour workloads, code/container deploy |
| **Memory** | Persistent context management | Short-term events, long-term memory, retrieval |
| **Gateway** | Tool/API integration | OpenAPI→MCP, Lambda→MCP, semantic search |
| **Identity** | Agent authentication | OAuth, OIDC, credential delegation |
| **Policy** | Access control | Cedar policies, natural language rules |
| **Code Interpreter** | Secure code execution | Python, JS, multi-language sandbox |
| **Browser** | Web automation | Headless browser, CAPTCHA reduction |
| **Observability** | Monitoring | CloudWatch, OpenTelemetry, tracing |
| **Evaluations** | Quality assurance | Live scoring, custom evaluators |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                          AWS BEDROCK AGENTCORE ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                          │
│  ┌─────────────────────────────────────────────────────────────────────────────────────┐│
│  │                              AGENT FRAMEWORKS (Your Choice)                         ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          ││
│  │  │  LangGraph  │    │   CrewAI    │    │   Strands   │    │   Custom    │          ││
│  │  │    Agent    │    │    Agent    │    │    Agent    │    │    Agent    │          ││
│  │  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘    └──────┬──────┘          ││
│  └─────────┼──────────────────┼──────────────────┼──────────────────┼──────────────────┘│
│            │                  │                  │                  │                   │
│            ▼                  ▼                  ▼                  ▼                   │
│  ┌─────────────────────────────────────────────────────────────────────────────────────┐│
│  │                              AGENTCORE RUNTIME                                      ││
│  │  • Serverless execution (pay per second)                                            ││
│  │  • Session isolation                                                                 ││
│  │  • 8-hour async workloads                                                           ││
│  │  • Auto-scaling                                                                      ││
│  └─────────────────────────────────────────────────────────────────────────────────────┘│
│                                         │                                               │
│     ┌───────────────┬───────────────────┼───────────────────┬───────────────┐          │
│     │               │                   │                   │               │          │
│     ▼               ▼                   ▼                   ▼               ▼          │
│  ┌──────────┐  ┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐     │
│  │ GATEWAY  │  │ MEMORY   │      │ IDENTITY │      │  CODE    │      │ BROWSER  │     │
│  │          │  │          │      │          │      │INTERPRETER│      │          │     │
│  │ •OpenAPI │  │ •Short   │      │ •OAuth   │      │ •Python  │      │ •Headless│     │
│  │ •Lambda  │  │  Term    │      │ •OIDC    │      │ •JS/TS   │      │ •CAPTCHA │     │
│  │ •MCP     │  │ •Long    │      │ •IAM     │      │ •Sandbox │      │ •Scaling │     │
│  │ •Search  │  │  Term    │      │ •Delegat │      │          │      │          │     │
│  └──────────┘  └──────────┘      └──────────┘      └──────────┘      └──────────┘     │
│                                         │                                               │
│                                         ▼                                               │
│  ┌─────────────────────────────────────────────────────────────────────────────────────┐│
│  │                           OBSERVABILITY & EVALUATIONS                               ││
│  │  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐                 ││
│  │  │  CloudWatch     │    │  OpenTelemetry  │    │   Evaluators    │                 ││
│  │  │  • Metrics      │    │  • Traces       │    │  • Quality      │                 ││
│  │  │  • Logs         │    │  • Spans        │    │  • Correctness  │                 ││
│  │  │  • Dashboards   │    │  • Context      │    │  • Safety       │                 ││
│  │  └─────────────────┘    └─────────────────┘    └─────────────────┘                 ││
│  └─────────────────────────────────────────────────────────────────────────────────────┘│
│                                                                                          │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
enterprise-agentcore/
├── pyproject.toml                    # UV/Poetry configuration
├── .python-version                   # Python 3.12+
├── src/
│   ├── __init__.py
│   ├── config/
│   │   ├── __init__.py
│   │   └── settings.py               # Environment configuration
│   │
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── base.py                   # Base agent interface
│   │   ├── discovery_agent.py        # Repository discovery
│   │   ├── planning_agent.py         # Task planning
│   │   ├── execution_agent.py        # Code execution
│   │   └── cicd_agent.py            # CI/CD monitoring
│   │
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── github_tools.py           # GitHub API integration
│   │   ├── jira_tools.py             # Jira API integration
│   │   ├── slack_tools.py            # Slack integration
│   │   └── sentry_tools.py           # Sentry integration
│   │
│   ├── services/
│   │   ├── __init__.py
│   │   ├── agentcore_runtime.py      # Runtime management
│   │   ├── agentcore_memory.py       # Memory service
│   │   ├── agentcore_gateway.py      # Gateway management
│   │   ├── agentcore_identity.py     # Identity/auth
│   │   ├── code_interpreter.py       # Code execution
│   │   └── browser_service.py        # Browser automation
│   │
│   ├── workflows/
│   │   ├── __init__.py
│   │   ├── jira_workflow.py          # Jira ticket workflow
│   │   ├── sentry_workflow.py        # Error handling workflow
│   │   └── github_workflow.py        # PR/issue workflow
│   │
│   └── api/
│       ├── __init__.py
│       ├── app.py                    # FastAPI application
│       └── routes/
│           ├── webhooks.py           # Webhook handlers
│           └── commands.py           # Slack commands
│
├── infrastructure/
│   └── terraform/
│       ├── main.tf
│       ├── agentcore.tf              # AgentCore resources
│       ├── gateway.tf                # API Gateway
│       ├── iam.tf                    # IAM roles
│       └── outputs.tf
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
└── scripts/
    ├── deploy.sh
    ├── setup-local.sh
    └── test.sh
```

---

## Phase 1: Core Configuration

### pyproject.toml

```toml
[project]
name = "enterprise-agentcore"
version = "1.0.0"
description = "Enterprise AI Agent System using AWS Bedrock AgentCore"
requires-python = ">=3.12"
dependencies = [
    "boto3>=1.35.0",
    "bedrock-agentcore>=0.2.0",
    "langgraph>=0.4.0",
    "langchain-aws>=0.3.0",
    "langchain-core>=0.3.0",
    "fastapi>=0.115.0",
    "uvicorn>=0.34.0",
    "pydantic>=2.10.0",
    "pydantic-settings>=2.6.0",
    "httpx>=0.28.0",
    "structlog>=24.4.0",
    "opentelemetry-api>=1.29.0",
    "opentelemetry-sdk>=1.29.0",
    "opentelemetry-exporter-otlp>=1.29.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.24.0",
    "pytest-cov>=6.0.0",
    "ruff>=0.8.0",
    "mypy>=1.13.0",
    "moto>=5.0.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "W", "F", "I", "N", "UP", "B", "A", "C4", "PIE", "T20", "RET", "SIM", "TCH"]

[tool.mypy]
python_version = "3.12"
strict = true

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
```

### src/config/settings.py

```python
from pydantic_settings import BaseSettings
from functools import lru_cache
from typing import Literal

class Settings(BaseSettings):
    environment: Literal["local", "dev", "staging", "prod"] = "local"
    
    aws_region: str = "us-east-1"
    aws_access_key_id: str | None = None
    aws_secret_access_key: str | None = None
    
    agentcore_runtime_arn: str = ""
    agentcore_memory_arn: str = ""
    agentcore_gateway_arn: str = ""
    
    bedrock_model_id: str = "anthropic.claude-sonnet-4-20250514-v1:0"
    
    github_token: str = ""
    github_org: str = ""
    
    jira_base_url: str = ""
    jira_email: str = ""
    jira_api_token: str = ""
    
    slack_bot_token: str = ""
    slack_signing_secret: str = ""
    
    sentry_auth_token: str = ""
    sentry_org: str = ""
    
    log_level: str = "INFO"
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

@lru_cache
def get_settings() -> Settings:
    return Settings()
```

---

## Phase 2: AgentCore Services Implementation

### src/services/agentcore_runtime.py

```python
import boto3
from typing import Any, AsyncIterator
from dataclasses import dataclass
from bedrock_agentcore.runtime import AgentGatewayClient
from bedrock_agentcore.runtime.models import InvokeAgentRequest
import structlog

from src.config.settings import get_settings

logger = structlog.get_logger()

@dataclass
class AgentRuntimeConfig:
    agent_runtime_id: str
    session_timeout: int = 3600
    max_turns: int = 20
    trace_enabled: bool = True

class AgentCoreRuntimeService:
    def __init__(self):
        self.settings = get_settings()
        self.boto_client = boto3.client(
            "bedrock-agentcore-runtime",
            region_name=self.settings.aws_region
        )
        self.agentcore_client = AgentGatewayClient()
    
    async def create_runtime(
        self,
        name: str,
        agent_code_path: str,
        memory_size_mb: int = 2048,
        timeout_seconds: int = 900
    ) -> dict[str, Any]:
        response = self.boto_client.create_agent_runtime(
            agentRuntimeName=name,
            roleArn=self.settings.agentcore_runtime_arn,
            runtimeConfiguration={
                "memory": memory_size_mb,
                "timeout": timeout_seconds,
                "vcpu": 1.0
            },
            agentSource={
                "sourceType": "CODE",
                "codeConfiguration": {
                    "sourceDirectory": agent_code_path,
                    "handler": "main.handler"
                }
            }
        )
        logger.info("created_agent_runtime", runtime_id=response["agentRuntimeId"])
        return response
    
    async def invoke_agent(
        self,
        runtime_endpoint: str,
        session_id: str,
        input_text: str,
        context: dict[str, Any] | None = None
    ) -> AsyncIterator[str]:
        request = InvokeAgentRequest(
            session_id=session_id,
            input_text=input_text,
            session_attributes=context or {}
        )
        
        async for event in self.agentcore_client.invoke_agent_streaming(
            endpoint=runtime_endpoint,
            request=request
        ):
            if hasattr(event, "chunk"):
                yield event.chunk.bytes.decode("utf-8")
            elif hasattr(event, "trace"):
                logger.debug("agent_trace", trace=event.trace)
    
    async def invoke_agent_sync(
        self,
        runtime_endpoint: str,
        session_id: str,
        input_text: str,
        context: dict[str, Any] | None = None
    ) -> str:
        chunks = []
        async for chunk in self.invoke_agent(
            runtime_endpoint, session_id, input_text, context
        ):
            chunks.append(chunk)
        return "".join(chunks)
    
    def get_runtime_status(self, runtime_id: str) -> dict[str, Any]:
        return self.boto_client.get_agent_runtime(agentRuntimeId=runtime_id)
    
    def list_runtimes(self) -> list[dict[str, Any]]:
        paginator = self.boto_client.get_paginator("list_agent_runtimes")
        runtimes = []
        for page in paginator.paginate():
            runtimes.extend(page.get("agentRuntimes", []))
        return runtimes
    
    def delete_runtime(self, runtime_id: str) -> None:
        self.boto_client.delete_agent_runtime(agentRuntimeId=runtime_id)
        logger.info("deleted_agent_runtime", runtime_id=runtime_id)
```

### src/services/agentcore_memory.py

```python
import boto3
from typing import Any
from dataclasses import dataclass, field
from datetime import datetime
import uuid
import structlog

from src.config.settings import get_settings

logger = structlog.get_logger()

@dataclass
class MemoryEvent:
    event_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    event_type: str = "conversation"
    content: str = ""
    role: str = "user"
    timestamp: datetime = field(default_factory=datetime.utcnow)
    metadata: dict[str, Any] = field(default_factory=dict)

@dataclass
class Memory:
    memory_id: str
    content: str
    memory_type: str
    created_at: datetime
    relevance_score: float = 0.0
    metadata: dict[str, Any] = field(default_factory=dict)

class AgentCoreMemoryService:
    def __init__(self):
        self.settings = get_settings()
        self.client = boto3.client(
            "bedrock-agentcore-memory",
            region_name=self.settings.aws_region
        )
    
    async def create_memory_store(self, name: str, config: dict[str, Any] | None = None) -> dict:
        config = config or {
            "shortTermConfig": {
                "maxEvents": 100,
                "ttlSeconds": 3600
            },
            "longTermConfig": {
                "embeddingModelId": "amazon.titan-embed-text-v2:0",
                "vectorDimensions": 1024
            }
        }
        
        response = self.client.create_memory(
            memoryName=name,
            memoryConfiguration=config
        )
        logger.info("created_memory_store", memory_id=response["memoryId"])
        return response
    
    async def add_short_term_event(
        self,
        memory_id: str,
        session_id: str,
        event: MemoryEvent
    ) -> dict:
        response = self.client.create_memory_record(
            memoryId=memory_id,
            sessionId=session_id,
            content={
                "type": "SHORT_TERM",
                "event": {
                    "eventId": event.event_id,
                    "eventType": event.event_type,
                    "content": event.content,
                    "role": event.role,
                    "timestamp": event.timestamp.isoformat(),
                    "metadata": event.metadata
                }
            }
        )
        return response
    
    async def add_conversation_turn(
        self,
        memory_id: str,
        session_id: str,
        user_message: str,
        assistant_response: str
    ) -> None:
        user_event = MemoryEvent(
            content=user_message,
            role="user",
            event_type="message"
        )
        await self.add_short_term_event(memory_id, session_id, user_event)
        
        assistant_event = MemoryEvent(
            content=assistant_response,
            role="assistant",
            event_type="message"
        )
        await self.add_short_term_event(memory_id, session_id, assistant_event)
    
    async def store_long_term_memory(
        self,
        memory_id: str,
        content: str,
        memory_type: str = "knowledge",
        metadata: dict[str, Any] | None = None
    ) -> dict:
        response = self.client.create_memory_record(
            memoryId=memory_id,
            content={
                "type": "LONG_TERM",
                "memory": {
                    "content": content,
                    "memoryType": memory_type,
                    "metadata": metadata or {}
                }
            }
        )
        logger.info("stored_long_term_memory", memory_id=memory_id)
        return response
    
    async def retrieve_memories(
        self,
        memory_id: str,
        query: str,
        limit: int = 10,
        threshold: float = 0.7
    ) -> list[Memory]:
        response = self.client.retrieve_memory_records(
            memoryId=memory_id,
            query=query,
            maxResults=limit,
            minRelevanceScore=threshold
        )
        
        memories = []
        for record in response.get("memoryRecords", []):
            memories.append(Memory(
                memory_id=record["recordId"],
                content=record["content"],
                memory_type=record.get("memoryType", "unknown"),
                created_at=datetime.fromisoformat(record["createdAt"]),
                relevance_score=record.get("relevanceScore", 0.0),
                metadata=record.get("metadata", {})
            ))
        return memories
    
    async def get_session_history(
        self,
        memory_id: str,
        session_id: str,
        limit: int = 50
    ) -> list[MemoryEvent]:
        response = self.client.get_session_memory(
            memoryId=memory_id,
            sessionId=session_id,
            maxEvents=limit
        )
        
        events = []
        for event_data in response.get("events", []):
            events.append(MemoryEvent(
                event_id=event_data["eventId"],
                event_type=event_data["eventType"],
                content=event_data["content"],
                role=event_data["role"],
                timestamp=datetime.fromisoformat(event_data["timestamp"]),
                metadata=event_data.get("metadata", {})
            ))
        return events
    
    async def clear_session(self, memory_id: str, session_id: str) -> None:
        self.client.delete_session_memory(
            memoryId=memory_id,
            sessionId=session_id
        )
        logger.info("cleared_session_memory", session_id=session_id)
```

### src/services/agentcore_gateway.py

```python
import boto3
from typing import Any
from dataclasses import dataclass
import json
import structlog

from src.config.settings import get_settings

logger = structlog.get_logger()

@dataclass
class ToolDefinition:
    name: str
    description: str
    input_schema: dict[str, Any]
    source_type: str
    source_config: dict[str, Any]

@dataclass
class MCPTool:
    name: str
    description: str
    parameters: dict[str, Any]

class AgentCoreGatewayService:
    def __init__(self):
        self.settings = get_settings()
        self.client = boto3.client(
            "bedrock-agentcore-gateway",
            region_name=self.settings.aws_region
        )
    
    async def create_gateway(
        self,
        name: str,
        description: str = ""
    ) -> dict[str, Any]:
        response = self.client.create_gateway(
            gatewayName=name,
            description=description
        )
        logger.info("created_gateway", gateway_id=response["gatewayId"])
        return response
    
    async def register_openapi_tool(
        self,
        gateway_id: str,
        tool_name: str,
        openapi_spec: dict[str, Any],
        base_url: str,
        auth_config: dict[str, Any] | None = None
    ) -> dict[str, Any]:
        tool_config = {
            "toolName": tool_name,
            "sourceType": "OPENAPI",
            "openApiConfiguration": {
                "specification": json.dumps(openapi_spec),
                "baseUrl": base_url
            }
        }
        
        if auth_config:
            tool_config["authentication"] = auth_config
        
        response = self.client.create_gateway_tool(
            gatewayId=gateway_id,
            **tool_config
        )
        logger.info("registered_openapi_tool", tool_name=tool_name)
        return response
    
    async def register_lambda_tool(
        self,
        gateway_id: str,
        tool_name: str,
        lambda_arn: str,
        description: str,
        input_schema: dict[str, Any]
    ) -> dict[str, Any]:
        response = self.client.create_gateway_tool(
            gatewayId=gateway_id,
            toolName=tool_name,
            description=description,
            sourceType="LAMBDA",
            lambdaConfiguration={
                "lambdaArn": lambda_arn
            },
            inputSchema=input_schema
        )
        logger.info("registered_lambda_tool", tool_name=tool_name, lambda_arn=lambda_arn)
        return response
    
    async def connect_mcp_server(
        self,
        gateway_id: str,
        mcp_server_url: str,
        server_name: str
    ) -> dict[str, Any]:
        response = self.client.create_mcp_connection(
            gatewayId=gateway_id,
            mcpServerUrl=mcp_server_url,
            connectionName=server_name
        )
        logger.info("connected_mcp_server", server_name=server_name)
        return response
    
    async def list_available_tools(
        self,
        gateway_id: str,
        query: str | None = None
    ) -> list[MCPTool]:
        params = {"gatewayId": gateway_id}
        if query:
            params["searchQuery"] = query
        
        response = self.client.list_gateway_tools(**params)
        
        tools = []
        for tool_data in response.get("tools", []):
            tools.append(MCPTool(
                name=tool_data["toolName"],
                description=tool_data.get("description", ""),
                parameters=tool_data.get("inputSchema", {})
            ))
        return tools
    
    async def invoke_tool(
        self,
        gateway_id: str,
        tool_name: str,
        parameters: dict[str, Any]
    ) -> dict[str, Any]:
        response = self.client.invoke_gateway_tool(
            gatewayId=gateway_id,
            toolName=tool_name,
            inputParameters=parameters
        )
        return response.get("output", {})
    
    async def enable_semantic_search(
        self,
        gateway_id: str,
        embedding_model_id: str = "amazon.titan-embed-text-v2:0"
    ) -> None:
        self.client.update_gateway(
            gatewayId=gateway_id,
            semanticSearchConfiguration={
                "enabled": True,
                "embeddingModelId": embedding_model_id
            }
        )
        logger.info("enabled_semantic_search", gateway_id=gateway_id)
    
    async def search_tools(
        self,
        gateway_id: str,
        query: str,
        max_results: int = 5
    ) -> list[MCPTool]:
        response = self.client.search_gateway_tools(
            gatewayId=gateway_id,
            query=query,
            maxResults=max_results
        )
        
        tools = []
        for result in response.get("results", []):
            tools.append(MCPTool(
                name=result["toolName"],
                description=result.get("description", ""),
                parameters=result.get("inputSchema", {})
            ))
        return tools
```

### src/services/code_interpreter.py

```python
import boto3
from typing import Any
from dataclasses import dataclass, field
from enum import Enum
import base64
import structlog

from src.config.settings import get_settings

logger = structlog.get_logger()

class CodeLanguage(Enum):
    PYTHON = "python"
    JAVASCRIPT = "javascript"
    TYPESCRIPT = "typescript"
    BASH = "bash"

@dataclass
class CodeExecutionResult:
    output: str
    error: str | None = None
    exit_code: int = 0
    execution_time_ms: float = 0.0
    files: list[dict[str, Any]] = field(default_factory=list)
    visualizations: list[str] = field(default_factory=list)

class CodeInterpreterService:
    def __init__(self):
        self.settings = get_settings()
        self.client = boto3.client(
            "bedrock-agentcore-codeinterpreter",
            region_name=self.settings.aws_region
        )
        self._active_sessions: dict[str, str] = {}
    
    async def create_session(
        self,
        session_id: str,
        language: CodeLanguage = CodeLanguage.PYTHON,
        timeout_seconds: int = 300
    ) -> dict[str, Any]:
        response = self.client.create_code_interpreter_session(
            sessionId=session_id,
            configuration={
                "language": language.value,
                "timeoutSeconds": timeout_seconds,
                "resourceLimits": {
                    "maxMemoryMB": 512,
                    "maxCpuSeconds": 60
                }
            }
        )
        self._active_sessions[session_id] = response["interpreterSessionId"]
        logger.info("created_code_interpreter_session", session_id=session_id)
        return response
    
    async def execute_code(
        self,
        session_id: str,
        code: str,
        language: CodeLanguage | None = None
    ) -> CodeExecutionResult:
        interpreter_session_id = self._active_sessions.get(session_id)
        if not interpreter_session_id:
            await self.create_session(session_id, language or CodeLanguage.PYTHON)
            interpreter_session_id = self._active_sessions[session_id]
        
        response = self.client.invoke_code_interpreter(
            interpreterSessionId=interpreter_session_id,
            code=code
        )
        
        visualizations = []
        for viz in response.get("visualizations", []):
            if viz.get("type") == "image":
                visualizations.append(base64.b64decode(viz["data"]).decode("utf-8"))
        
        return CodeExecutionResult(
            output=response.get("output", ""),
            error=response.get("error"),
            exit_code=response.get("exitCode", 0),
            execution_time_ms=response.get("executionTimeMs", 0),
            files=response.get("files", []),
            visualizations=visualizations
        )
    
    async def upload_file(
        self,
        session_id: str,
        file_name: str,
        content: bytes
    ) -> dict[str, Any]:
        interpreter_session_id = self._active_sessions.get(session_id)
        if not interpreter_session_id:
            raise ValueError(f"No active session found for {session_id}")
        
        response = self.client.upload_file_to_session(
            interpreterSessionId=interpreter_session_id,
            fileName=file_name,
            fileContent=base64.b64encode(content).decode("utf-8")
        )
        logger.info("uploaded_file", session_id=session_id, file_name=file_name)
        return response
    
    async def download_file(
        self,
        session_id: str,
        file_path: str
    ) -> bytes:
        interpreter_session_id = self._active_sessions.get(session_id)
        if not interpreter_session_id:
            raise ValueError(f"No active session found for {session_id}")
        
        response = self.client.download_file_from_session(
            interpreterSessionId=interpreter_session_id,
            filePath=file_path
        )
        return base64.b64decode(response["fileContent"])
    
    async def install_package(
        self,
        session_id: str,
        package_name: str,
        version: str | None = None
    ) -> CodeExecutionResult:
        package_spec = f"{package_name}=={version}" if version else package_name
        code = f"!pip install {package_spec}"
        return await self.execute_code(session_id, code)
    
    async def close_session(self, session_id: str) -> None:
        interpreter_session_id = self._active_sessions.pop(session_id, None)
        if interpreter_session_id:
            self.client.delete_code_interpreter_session(
                interpreterSessionId=interpreter_session_id
            )
            logger.info("closed_code_interpreter_session", session_id=session_id)
```

### src/services/browser_service.py

```python
import boto3
from typing import Any
from dataclasses import dataclass
from enum import Enum
import base64
import structlog

from src.config.settings import get_settings

logger = structlog.get_logger()

class BrowserAction(Enum):
    NAVIGATE = "navigate"
    CLICK = "click"
    TYPE = "type"
    SCROLL = "scroll"
    SCREENSHOT = "screenshot"
    EXTRACT = "extract"
    WAIT = "wait"

@dataclass
class BrowserActionResult:
    success: bool
    data: dict[str, Any] | None = None
    screenshot: bytes | None = None
    error: str | None = None
    page_url: str = ""
    page_title: str = ""

class BrowserService:
    def __init__(self):
        self.settings = get_settings()
        self.client = boto3.client(
            "bedrock-agentcore-browser",
            region_name=self.settings.aws_region
        )
        self._active_sessions: dict[str, str] = {}
    
    async def create_session(
        self,
        session_id: str,
        viewport_width: int = 1920,
        viewport_height: int = 1080,
        timeout_seconds: int = 600
    ) -> dict[str, Any]:
        response = self.client.create_browser_session(
            sessionId=session_id,
            configuration={
                "viewport": {
                    "width": viewport_width,
                    "height": viewport_height
                },
                "timeoutSeconds": timeout_seconds,
                "javascript": True,
                "cookies": True
            }
        )
        self._active_sessions[session_id] = response["browserSessionId"]
        logger.info("created_browser_session", session_id=session_id)
        return response
    
    async def navigate(
        self,
        session_id: str,
        url: str,
        wait_until: str = "networkidle"
    ) -> BrowserActionResult:
        return await self._execute_action(
            session_id,
            BrowserAction.NAVIGATE,
            {"url": url, "waitUntil": wait_until}
        )
    
    async def click(
        self,
        session_id: str,
        selector: str,
        wait_for_navigation: bool = False
    ) -> BrowserActionResult:
        return await self._execute_action(
            session_id,
            BrowserAction.CLICK,
            {"selector": selector, "waitForNavigation": wait_for_navigation}
        )
    
    async def type_text(
        self,
        session_id: str,
        selector: str,
        text: str,
        delay_ms: int = 50
    ) -> BrowserActionResult:
        return await self._execute_action(
            session_id,
            BrowserAction.TYPE,
            {"selector": selector, "text": text, "delayMs": delay_ms}
        )
    
    async def screenshot(
        self,
        session_id: str,
        full_page: bool = False
    ) -> BrowserActionResult:
        return await self._execute_action(
            session_id,
            BrowserAction.SCREENSHOT,
            {"fullPage": full_page}
        )
    
    async def extract_content(
        self,
        session_id: str,
        selector: str | None = None,
        extract_type: str = "text"
    ) -> BrowserActionResult:
        return await self._execute_action(
            session_id,
            BrowserAction.EXTRACT,
            {"selector": selector, "extractType": extract_type}
        )
    
    async def wait_for_selector(
        self,
        session_id: str,
        selector: str,
        timeout_ms: int = 30000
    ) -> BrowserActionResult:
        return await self._execute_action(
            session_id,
            BrowserAction.WAIT,
            {"selector": selector, "timeoutMs": timeout_ms}
        )
    
    async def _execute_action(
        self,
        session_id: str,
        action: BrowserAction,
        parameters: dict[str, Any]
    ) -> BrowserActionResult:
        browser_session_id = self._active_sessions.get(session_id)
        if not browser_session_id:
            await self.create_session(session_id)
            browser_session_id = self._active_sessions[session_id]
        
        response = self.client.execute_browser_action(
            browserSessionId=browser_session_id,
            action=action.value,
            parameters=parameters
        )
        
        screenshot = None
        if response.get("screenshot"):
            screenshot = base64.b64decode(response["screenshot"])
        
        return BrowserActionResult(
            success=response.get("success", True),
            data=response.get("data"),
            screenshot=screenshot,
            error=response.get("error"),
            page_url=response.get("pageUrl", ""),
            page_title=response.get("pageTitle", "")
        )
    
    async def close_session(self, session_id: str) -> None:
        browser_session_id = self._active_sessions.pop(session_id, None)
        if browser_session_id:
            self.client.delete_browser_session(browserSessionId=browser_session_id)
            logger.info("closed_browser_session", session_id=session_id)

---

## Phase 3: Agent Implementations

### src/agents/base.py

```python
from abc import ABC, abstractmethod
from typing import Any, AsyncIterator
from dataclasses import dataclass, field
from enum import Enum
import uuid
from datetime import datetime
import structlog

from src.services.agentcore_runtime import AgentCoreRuntimeService
from src.services.agentcore_memory import AgentCoreMemoryService
from src.services.agentcore_gateway import AgentCoreGatewayService

logger = structlog.get_logger()

class AgentStatus(Enum):
    IDLE = "idle"
    THINKING = "thinking"
    EXECUTING = "executing"
    WAITING = "waiting"
    ERROR = "error"
    COMPLETE = "complete"

@dataclass
class AgentContext:
    session_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    task_id: str | None = None
    parent_agent_id: str | None = None
    metadata: dict[str, Any] = field(default_factory=dict)
    created_at: datetime = field(default_factory=datetime.utcnow)

@dataclass
class AgentResponse:
    content: str
    status: AgentStatus
    tool_calls: list[dict[str, Any]] = field(default_factory=list)
    artifacts: list[dict[str, Any]] = field(default_factory=list)
    next_steps: list[str] = field(default_factory=list)
    metadata: dict[str, Any] = field(default_factory=dict)

class BaseAgent(ABC):
    def __init__(
        self,
        agent_id: str,
        name: str,
        description: str
    ):
        self.agent_id = agent_id
        self.name = name
        self.description = description
        self.status = AgentStatus.IDLE
        
        self.runtime_service = AgentCoreRuntimeService()
        self.memory_service = AgentCoreMemoryService()
        self.gateway_service = AgentCoreGatewayService()
    
    @property
    @abstractmethod
    def system_prompt(self) -> str:
        pass
    
    @property
    @abstractmethod
    def available_tools(self) -> list[dict[str, Any]]:
        pass
    
    @abstractmethod
    async def process(
        self,
        input_text: str,
        context: AgentContext
    ) -> AgentResponse:
        pass
    
    async def stream_response(
        self,
        input_text: str,
        context: AgentContext
    ) -> AsyncIterator[str]:
        self.status = AgentStatus.THINKING
        try:
            async for chunk in self.runtime_service.invoke_agent(
                runtime_endpoint=self._get_runtime_endpoint(),
                session_id=context.session_id,
                input_text=input_text,
                context=context.metadata
            ):
                yield chunk
            self.status = AgentStatus.COMPLETE
        except Exception as e:
            self.status = AgentStatus.ERROR
            logger.error("agent_stream_error", error=str(e), agent_id=self.agent_id)
            raise
    
    async def remember(
        self,
        context: AgentContext,
        content: str,
        memory_type: str = "knowledge"
    ) -> None:
        await self.memory_service.store_long_term_memory(
            memory_id=self._get_memory_id(),
            content=content,
            memory_type=memory_type,
            metadata={"agent_id": self.agent_id, "task_id": context.task_id}
        )
    
    async def recall(
        self,
        query: str,
        limit: int = 5
    ) -> list[str]:
        memories = await self.memory_service.retrieve_memories(
            memory_id=self._get_memory_id(),
            query=query,
            limit=limit
        )
        return [m.content for m in memories]
    
    def _get_runtime_endpoint(self) -> str:
        from src.config.settings import get_settings
        settings = get_settings()
        return f"{settings.agentcore_runtime_arn}/agents/{self.agent_id}"
    
    def _get_memory_id(self) -> str:
        from src.config.settings import get_settings
        settings = get_settings()
        return f"{settings.agentcore_memory_arn}/{self.agent_id}"
```

### src/agents/discovery_agent.py

```python
from typing import Any
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolNode
from langchain_aws import ChatBedrock
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage
from pydantic import BaseModel, Field
import structlog

from src.agents.base import BaseAgent, AgentContext, AgentResponse, AgentStatus
from src.tools.github_tools import GitHubTools
from src.config.settings import get_settings

logger = structlog.get_logger()

class DiscoveryState(BaseModel):
    messages: list[Any] = Field(default_factory=list)
    task_description: str = ""
    discovered_repos: list[dict[str, Any]] = Field(default_factory=list)
    relevant_files: list[dict[str, Any]] = Field(default_factory=list)
    analysis_complete: bool = False

class DiscoveryAgent(BaseAgent):
    def __init__(self):
        super().__init__(
            agent_id="discovery-agent",
            name="Discovery Agent",
            description="Searches and analyzes repositories to find relevant code"
        )
        self.settings = get_settings()
        self.github_tools = GitHubTools()
        self._graph = self._build_graph()
    
    @property
    def system_prompt(self) -> str:
        return """You are a code discovery agent for an enterprise organization.
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

Always provide clear reasoning for your recommendations."""

    @property
    def available_tools(self) -> list[dict[str, Any]]:
        return [
            {
                "name": "search_code",
                "description": "Search for code across repositories",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string", "description": "Search query"},
                        "org": {"type": "string", "description": "GitHub organization"}
                    },
                    "required": ["query"]
                }
            },
            {
                "name": "get_repo_tree",
                "description": "Get repository file structure",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "repo": {"type": "string"},
                        "ref": {"type": "string", "default": "main"}
                    },
                    "required": ["repo"]
                }
            },
            {
                "name": "get_file_content",
                "description": "Get file contents from a repository",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "repo": {"type": "string"},
                        "path": {"type": "string"},
                        "ref": {"type": "string", "default": "main"}
                    },
                    "required": ["repo", "path"]
                }
            }
        ]
    
    def _build_graph(self) -> StateGraph:
        llm = ChatBedrock(
            model_id=self.settings.bedrock_model_id,
            region_name=self.settings.aws_region
        )
        llm_with_tools = llm.bind_tools(self._get_langchain_tools())
        
        def should_continue(state: DiscoveryState) -> str:
            last_message = state.messages[-1] if state.messages else None
            if last_message and hasattr(last_message, "tool_calls") and last_message.tool_calls:
                return "tools"
            return "analyze"
        
        async def search_node(state: DiscoveryState) -> dict:
            messages = [SystemMessage(content=self.system_prompt)] + state.messages
            response = await llm_with_tools.ainvoke(messages)
            return {"messages": state.messages + [response]}
        
        async def analyze_node(state: DiscoveryState) -> dict:
            analysis_prompt = f"""Based on the search results, provide a summary of:
1. Most relevant repositories found
2. Key files to examine
3. Dependencies and relationships between components

Task: {state.task_description}
Results: {state.discovered_repos}"""
            
            messages = [
                SystemMessage(content=self.system_prompt),
                HumanMessage(content=analysis_prompt)
            ]
            response = await llm_with_tools.ainvoke(messages)
            return {
                "messages": state.messages + [response],
                "analysis_complete": True
            }
        
        tool_node = ToolNode(self._get_langchain_tools())
        
        graph = StateGraph(DiscoveryState)
        graph.add_node("search", search_node)
        graph.add_node("tools", tool_node)
        graph.add_node("analyze", analyze_node)
        
        graph.set_entry_point("search")
        graph.add_conditional_edges("search", should_continue, {
            "tools": "tools",
            "analyze": "analyze"
        })
        graph.add_edge("tools", "search")
        graph.add_edge("analyze", END)
        
        return graph.compile()
    
    def _get_langchain_tools(self) -> list:
        from langchain_core.tools import StructuredTool
        
        return [
            StructuredTool.from_function(
                func=self.github_tools.search_code_sync,
                name="search_code",
                description="Search for code across repositories"
            ),
            StructuredTool.from_function(
                func=self.github_tools.get_repo_tree_sync,
                name="get_repo_tree",
                description="Get repository file structure"
            ),
            StructuredTool.from_function(
                func=self.github_tools.get_file_content_sync,
                name="get_file_content",
                description="Get file contents"
            )
        ]
    
    async def process(
        self,
        input_text: str,
        context: AgentContext
    ) -> AgentResponse:
        self.status = AgentStatus.THINKING
        
        initial_state = DiscoveryState(
            messages=[HumanMessage(content=input_text)],
            task_description=input_text
        )
        
        try:
            final_state = await self._graph.ainvoke(initial_state)
            
            last_message = final_state["messages"][-1]
            content = last_message.content if hasattr(last_message, "content") else str(last_message)
            
            self.status = AgentStatus.COMPLETE
            return AgentResponse(
                content=content,
                status=self.status,
                artifacts=[{
                    "type": "discovery_results",
                    "repos": final_state.get("discovered_repos", []),
                    "files": final_state.get("relevant_files", [])
                }]
            )
        except Exception as e:
            self.status = AgentStatus.ERROR
            logger.error("discovery_agent_error", error=str(e))
            return AgentResponse(
                content=f"Discovery failed: {str(e)}",
                status=self.status
            )
```

### src/agents/planning_agent.py

```python
from typing import Any
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolNode
from langchain_aws import ChatBedrock
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage
from pydantic import BaseModel, Field
import structlog

from src.agents.base import BaseAgent, AgentContext, AgentResponse, AgentStatus
from src.tools.github_tools import GitHubTools
from src.tools.jira_tools import JiraTools
from src.config.settings import get_settings

logger = structlog.get_logger()

class PlanningState(BaseModel):
    messages: list[Any] = Field(default_factory=list)
    ticket_info: dict[str, Any] = Field(default_factory=dict)
    discovery_results: dict[str, Any] = Field(default_factory=dict)
    implementation_plan: str = ""
    branch_created: bool = False
    pr_created: bool = False
    plan_ready: bool = False

class PlanningAgent(BaseAgent):
    def __init__(self):
        super().__init__(
            agent_id="planning-agent",
            name="Planning Agent",
            description="Creates detailed implementation plans with TDD approach"
        )
        self.settings = get_settings()
        self.github_tools = GitHubTools()
        self.jira_tools = JiraTools()
        self._graph = self._build_graph()
    
    @property
    def system_prompt(self) -> str:
        return """You are a software planning agent.
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

Always follow organization conventions for branch naming."""

    @property
    def available_tools(self) -> list[dict[str, Any]]:
        return [
            {
                "name": "get_jira_issue",
                "description": "Get Jira issue details",
                "parameters": {
                    "type": "object",
                    "properties": {"issue_key": {"type": "string"}},
                    "required": ["issue_key"]
                }
            },
            {
                "name": "create_branch",
                "description": "Create a new git branch",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "repo": {"type": "string"},
                        "branch": {"type": "string"},
                        "from_branch": {"type": "string", "default": "main"}
                    },
                    "required": ["repo", "branch"]
                }
            },
            {
                "name": "create_file",
                "description": "Create or update a file in a repository",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "repo": {"type": "string"},
                        "path": {"type": "string"},
                        "content": {"type": "string"},
                        "message": {"type": "string"},
                        "branch": {"type": "string"}
                    },
                    "required": ["repo", "path", "content", "message", "branch"]
                }
            },
            {
                "name": "create_pull_request",
                "description": "Create a pull request",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "repo": {"type": "string"},
                        "title": {"type": "string"},
                        "head": {"type": "string"},
                        "base": {"type": "string", "default": "main"},
                        "body": {"type": "string"},
                        "draft": {"type": "boolean", "default": True}
                    },
                    "required": ["repo", "title", "head"]
                }
            }
        ]
    
    def _build_graph(self) -> StateGraph:
        llm = ChatBedrock(
            model_id=self.settings.bedrock_model_id,
            region_name=self.settings.aws_region
        )
        llm_with_tools = llm.bind_tools(self._get_langchain_tools())
        
        def router(state: PlanningState) -> str:
            if not state.ticket_info:
                return "fetch_ticket"
            if not state.implementation_plan:
                return "create_plan"
            if not state.branch_created:
                return "create_branch"
            if not state.pr_created:
                return "create_pr"
            return "complete"
        
        async def fetch_ticket_node(state: PlanningState) -> dict:
            messages = [
                SystemMessage(content=self.system_prompt),
                HumanMessage(content=f"Fetch details for the Jira ticket and analyze requirements. Discovery results: {state.discovery_results}")
            ] + state.messages
            
            response = await llm_with_tools.ainvoke(messages)
            return {"messages": state.messages + [response]}
        
        async def create_plan_node(state: PlanningState) -> dict:
            plan_prompt = f"""Create a detailed implementation plan for:
Ticket: {state.ticket_info}
Discovery: {state.discovery_results}

Include:
1. Scope (in/out)
2. Architecture changes
3. Test cases (TDD)
4. Implementation steps
5. Rollback plan"""
            
            messages = [
                SystemMessage(content=self.system_prompt),
                HumanMessage(content=plan_prompt)
            ]
            response = await llm_with_tools.ainvoke(messages)
            
            plan_content = response.content if hasattr(response, "content") else str(response)
            return {
                "messages": state.messages + [response],
                "implementation_plan": plan_content
            }
        
        async def create_branch_node(state: PlanningState) -> dict:
            ticket_id = state.ticket_info.get("key", "unknown")
            branch_name = f"feature/{ticket_id.lower()}"
            
            messages = [
                SystemMessage(content=self.system_prompt),
                HumanMessage(content=f"Create branch '{branch_name}' and commit the PLAN.md file with the implementation plan.")
            ]
            response = await llm_with_tools.ainvoke(messages)
            return {
                "messages": state.messages + [response],
                "branch_created": True
            }
        
        async def create_pr_node(state: PlanningState) -> dict:
            messages = [
                SystemMessage(content=self.system_prompt),
                HumanMessage(content="Create a draft PR for the implementation plan.")
            ]
            response = await llm_with_tools.ainvoke(messages)
            return {
                "messages": state.messages + [response],
                "pr_created": True,
                "plan_ready": True
            }
        
        tool_node = ToolNode(self._get_langchain_tools())
        
        graph = StateGraph(PlanningState)
        graph.add_node("fetch_ticket", fetch_ticket_node)
        graph.add_node("create_plan", create_plan_node)
        graph.add_node("create_branch", create_branch_node)
        graph.add_node("create_pr", create_pr_node)
        graph.add_node("tools", tool_node)
        
        graph.set_entry_point("fetch_ticket")
        
        for node in ["fetch_ticket", "create_plan", "create_branch", "create_pr"]:
            graph.add_conditional_edges(
                node,
                lambda s: "tools" if s.messages and hasattr(s.messages[-1], "tool_calls") and s.messages[-1].tool_calls else router(s),
                {"tools": "tools", "fetch_ticket": "fetch_ticket", "create_plan": "create_plan", 
                 "create_branch": "create_branch", "create_pr": "create_pr", "complete": END}
            )
        
        graph.add_edge("tools", "fetch_ticket")
        
        return graph.compile()
    
    def _get_langchain_tools(self) -> list:
        from langchain_core.tools import StructuredTool
        
        return [
            StructuredTool.from_function(
                func=self.jira_tools.get_issue_sync,
                name="get_jira_issue",
                description="Get Jira issue details"
            ),
            StructuredTool.from_function(
                func=self.github_tools.create_branch_sync,
                name="create_branch",
                description="Create a new branch"
            ),
            StructuredTool.from_function(
                func=self.github_tools.create_file_sync,
                name="create_file",
                description="Create or update a file"
            ),
            StructuredTool.from_function(
                func=self.github_tools.create_pr_sync,
                name="create_pull_request",
                description="Create a pull request"
            )
        ]
    
    async def process(
        self,
        input_text: str,
        context: AgentContext
    ) -> AgentResponse:
        self.status = AgentStatus.THINKING
        
        discovery_results = context.metadata.get("discovery_results", {})
        
        initial_state = PlanningState(
            messages=[HumanMessage(content=input_text)],
            discovery_results=discovery_results
        )
        
        try:
            final_state = await self._graph.ainvoke(initial_state)
            
            self.status = AgentStatus.COMPLETE
            return AgentResponse(
                content=final_state.get("implementation_plan", ""),
                status=self.status,
                artifacts=[{
                    "type": "implementation_plan",
                    "content": final_state.get("implementation_plan", ""),
                    "branch_created": final_state.get("branch_created", False),
                    "pr_created": final_state.get("pr_created", False)
                }],
                next_steps=["Await human approval", "Begin execution phase"]
            )
        except Exception as e:
            self.status = AgentStatus.ERROR
            logger.error("planning_agent_error", error=str(e))
            return AgentResponse(
                content=f"Planning failed: {str(e)}",
                status=self.status
            )
```

---

## Phase 4: Tools Implementation

### src/tools/github_tools.py

```python
import httpx
from typing import Any
import base64
import structlog

from src.config.settings import get_settings

logger = structlog.get_logger()

class GitHubTools:
    def __init__(self):
        self.settings = get_settings()
        self.base_url = "https://api.github.com"
        self.headers = {
            "Authorization": f"Bearer {self.settings.github_token}",
            "Accept": "application/vnd.github+json",
            "X-GitHub-Api-Version": "2022-11-28"
        }
    
    async def search_code(
        self,
        query: str,
        org: str | None = None
    ) -> list[dict[str, Any]]:
        org = org or self.settings.github_org
        search_query = f"{query} org:{org}"
        
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{self.base_url}/search/code",
                headers=self.headers,
                params={"q": search_query, "per_page": 30}
            )
            response.raise_for_status()
            data = response.json()
        
        results = []
        for item in data.get("items", []):
            results.append({
                "repo": item["repository"]["full_name"],
                "path": item["path"],
                "url": item["html_url"],
                "score": item.get("score", 0)
            })
        return results
    
    def search_code_sync(self, query: str, org: str | None = None) -> list[dict[str, Any]]:
        import asyncio
        return asyncio.get_event_loop().run_until_complete(self.search_code(query, org))
    
    async def get_repo_tree(
        self,
        repo: str,
        ref: str = "main"
    ) -> dict[str, Any]:
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{self.base_url}/repos/{repo}/git/trees/{ref}",
                headers=self.headers,
                params={"recursive": "1"}
            )
            response.raise_for_status()
            return response.json()
    
    def get_repo_tree_sync(self, repo: str, ref: str = "main") -> dict[str, Any]:
        import asyncio
        return asyncio.get_event_loop().run_until_complete(self.get_repo_tree(repo, ref))
    
    async def get_file_content(
        self,
        repo: str,
        path: str,
        ref: str = "main"
    ) -> str:
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{self.base_url}/repos/{repo}/contents/{path}",
                headers=self.headers,
                params={"ref": ref}
            )
            response.raise_for_status()
            data = response.json()
        
        if data.get("encoding") == "base64":
            return base64.b64decode(data["content"]).decode("utf-8")
        return data.get("content", "")
    
    def get_file_content_sync(self, repo: str, path: str, ref: str = "main") -> str:
        import asyncio
        return asyncio.get_event_loop().run_until_complete(self.get_file_content(repo, path, ref))
    
    async def create_branch(
        self,
        repo: str,
        branch: str,
        from_branch: str = "main"
    ) -> dict[str, Any]:
        async with httpx.AsyncClient() as client:
            ref_response = await client.get(
                f"{self.base_url}/repos/{repo}/git/refs/heads/{from_branch}",
                headers=self.headers
            )
            ref_response.raise_for_status()
            sha = ref_response.json()["object"]["sha"]
            
            create_response = await client.post(
                f"{self.base_url}/repos/{repo}/git/refs",
                headers=self.headers,
                json={"ref": f"refs/heads/{branch}", "sha": sha}
            )
            create_response.raise_for_status()
            return create_response.json()
    
    def create_branch_sync(self, repo: str, branch: str, from_branch: str = "main") -> dict[str, Any]:
        import asyncio
        return asyncio.get_event_loop().run_until_complete(self.create_branch(repo, branch, from_branch))
    
    async def create_file(
        self,
        repo: str,
        path: str,
        content: str,
        message: str,
        branch: str
    ) -> dict[str, Any]:
        async with httpx.AsyncClient() as client:
            try:
                existing = await client.get(
                    f"{self.base_url}/repos/{repo}/contents/{path}",
                    headers=self.headers,
                    params={"ref": branch}
                )
                sha = existing.json().get("sha") if existing.status_code == 200 else None
            except Exception:
                sha = None
            
            body = {
                "message": message,
                "content": base64.b64encode(content.encode()).decode(),
                "branch": branch
            }
            if sha:
                body["sha"] = sha
            
            response = await client.put(
                f"{self.base_url}/repos/{repo}/contents/{path}",
                headers=self.headers,
                json=body
            )
            response.raise_for_status()
            return response.json()
    
    def create_file_sync(self, repo: str, path: str, content: str, message: str, branch: str) -> dict[str, Any]:
        import asyncio
        return asyncio.get_event_loop().run_until_complete(self.create_file(repo, path, content, message, branch))
    
    async def create_pr(
        self,
        repo: str,
        title: str,
        head: str,
        base: str = "main",
        body: str = "",
        draft: bool = True
    ) -> dict[str, Any]:
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{self.base_url}/repos/{repo}/pulls",
                headers=self.headers,
                json={
                    "title": title,
                    "head": head,
                    "base": base,
                    "body": body,
                    "draft": draft
                }
            )
            response.raise_for_status()
            return response.json()
    
    def create_pr_sync(self, repo: str, title: str, head: str, base: str = "main", body: str = "", draft: bool = True) -> dict[str, Any]:
        import asyncio
        return asyncio.get_event_loop().run_until_complete(self.create_pr(repo, title, head, base, body, draft))
    
    async def get_workflow_runs(
        self,
        repo: str,
        workflow_id: str | None = None,
        status: str | None = None
    ) -> list[dict[str, Any]]:
        async with httpx.AsyncClient() as client:
            params = {"per_page": 10}
            if status:
                params["status"] = status
            
            if workflow_id:
                url = f"{self.base_url}/repos/{repo}/actions/workflows/{workflow_id}/runs"
            else:
                url = f"{self.base_url}/repos/{repo}/actions/runs"
            
            response = await client.get(url, headers=self.headers, params=params)
            response.raise_for_status()
            return response.json().get("workflow_runs", [])
    
    async def get_job_logs(
        self,
        repo: str,
        job_id: int
    ) -> str:
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{self.base_url}/repos/{repo}/actions/jobs/{job_id}/logs",
                headers=self.headers,
                follow_redirects=True
            )
            response.raise_for_status()
            return response.text
```

### src/tools/jira_tools.py

```python
import httpx
from typing import Any
import structlog

from src.config.settings import get_settings

logger = structlog.get_logger()

class JiraTools:
    def __init__(self):
        self.settings = get_settings()
        self.base_url = self.settings.jira_base_url
        self.auth = (self.settings.jira_email, self.settings.jira_api_token)
    
    async def get_issue(self, issue_key: str) -> dict[str, Any]:
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{self.base_url}/rest/api/3/issue/{issue_key}",
                auth=self.auth,
                headers={"Accept": "application/json"}
            )
            response.raise_for_status()
            data = response.json()
        
        return {
            "key": data["key"],
            "summary": data["fields"]["summary"],
            "description": data["fields"].get("description", {}).get("content", []),
            "status": data["fields"]["status"]["name"],
            "priority": data["fields"]["priority"]["name"] if data["fields"].get("priority") else "Medium",
            "assignee": data["fields"].get("assignee", {}).get("displayName"),
            "labels": data["fields"].get("labels", []),
            "components": [c["name"] for c in data["fields"].get("components", [])],
            "story_points": data["fields"].get("customfield_10016")
        }
    
    def get_issue_sync(self, issue_key: str) -> dict[str, Any]:
        import asyncio
        return asyncio.get_event_loop().run_until_complete(self.get_issue(issue_key))
    
    async def add_comment(
        self,
        issue_key: str,
        comment: str
    ) -> dict[str, Any]:
        body = {
            "body": {
                "type": "doc",
                "version": 1,
                "content": [{
                    "type": "paragraph",
                    "content": [{"type": "text", "text": comment}]
                }]
            }
        }
        
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{self.base_url}/rest/api/3/issue/{issue_key}/comment",
                auth=self.auth,
                headers={"Content-Type": "application/json"},
                json=body
            )
            response.raise_for_status()
            return response.json()
    
    async def transition_issue(
        self,
        issue_key: str,
        transition_name: str
    ) -> None:
        async with httpx.AsyncClient() as client:
            transitions_response = await client.get(
                f"{self.base_url}/rest/api/3/issue/{issue_key}/transitions",
                auth=self.auth
            )
            transitions_response.raise_for_status()
            transitions = transitions_response.json()["transitions"]
            
            transition_id = None
            for t in transitions:
                if t["name"].lower() == transition_name.lower():
                    transition_id = t["id"]
                    break
            
            if not transition_id:
                raise ValueError(f"Transition '{transition_name}' not found")
            
            await client.post(
                f"{self.base_url}/rest/api/3/issue/{issue_key}/transitions",
                auth=self.auth,
                json={"transition": {"id": transition_id}}
            )
    
    async def update_issue(
        self,
        issue_key: str,
        updates: dict[str, Any]
    ) -> None:
        async with httpx.AsyncClient() as client:
            response = await client.put(
                f"{self.base_url}/rest/api/3/issue/{issue_key}",
                auth=self.auth,
                headers={"Content-Type": "application/json"},
                json={"fields": updates}
            )
            response.raise_for_status()
```

### src/tools/slack_tools.py

```python
import httpx
from typing import Any
import structlog

from src.config.settings import get_settings

logger = structlog.get_logger()

class SlackTools:
    def __init__(self):
        self.settings = get_settings()
        self.base_url = "https://slack.com/api"
        self.headers = {
            "Authorization": f"Bearer {self.settings.slack_bot_token}",
            "Content-Type": "application/json"
        }
    
    async def send_message(
        self,
        channel: str,
        text: str,
        blocks: list[dict[str, Any]] | None = None,
        thread_ts: str | None = None
    ) -> dict[str, Any]:
        body = {"channel": channel, "text": text}
        if blocks:
            body["blocks"] = blocks
        if thread_ts:
            body["thread_ts"] = thread_ts
        
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{self.base_url}/chat.postMessage",
                headers=self.headers,
                json=body
            )
            response.raise_for_status()
            return response.json()
    
    async def send_plan_notification(
        self,
        channel: str,
        task_id: str,
        ticket_id: str,
        summary: str,
        pr_url: str
    ) -> dict[str, Any]:
        blocks = [
            {
                "type": "header",
                "text": {"type": "plain_text", "text": "📋 Implementation Plan Ready"}
            },
            {
                "type": "section",
                "fields": [
                    {"type": "mrkdwn", "text": f"*Ticket:* {ticket_id}"},
                    {"type": "mrkdwn", "text": f"*Task ID:* {task_id}"}
                ]
            },
            {
                "type": "section",
                "text": {"type": "mrkdwn", "text": f"*Summary:* {summary}"}
            },
            {
                "type": "section",
                "text": {"type": "mrkdwn", "text": f"*PR:* <{pr_url}|View Pull Request>"}
            },
            {
                "type": "actions",
                "elements": [
                    {
                        "type": "button",
                        "text": {"type": "plain_text", "text": "✅ Approve"},
                        "style": "primary",
                        "action_id": f"approve_plan_{task_id}"
                    },
                    {
                        "type": "button",
                        "text": {"type": "plain_text", "text": "❌ Reject"},
                        "style": "danger",
                        "action_id": f"reject_plan_{task_id}"
                    },
                    {
                        "type": "button",
                        "text": {"type": "plain_text", "text": "💬 Request Changes"},
                        "action_id": f"request_changes_{task_id}"
                    }
                ]
            }
        ]
        
        return await self.send_message(channel, f"Plan ready for {ticket_id}", blocks)
    
    async def send_completion_notification(
        self,
        channel: str,
        task_id: str,
        ticket_id: str,
        pr_url: str,
        ci_status: str
    ) -> dict[str, Any]:
        status_emoji = "✅" if ci_status == "success" else "❌"
        
        blocks = [
            {
                "type": "header",
                "text": {"type": "plain_text", "text": f"{status_emoji} Task Complete"}
            },
            {
                "type": "section",
                "fields": [
                    {"type": "mrkdwn", "text": f"*Ticket:* {ticket_id}"},
                    {"type": "mrkdwn", "text": f"*CI Status:* {ci_status}"}
                ]
            },
            {
                "type": "section",
                "text": {"type": "mrkdwn", "text": f"*PR:* <{pr_url}|View Pull Request>"}
            }
        ]
        
        return await self.send_message(channel, f"Task {task_id} complete", blocks)
    
    async def send_escalation(
        self,
        channel: str,
        task_id: str,
        reason: str,
        details: str
    ) -> dict[str, Any]:
        blocks = [
            {
                "type": "header",
                "text": {"type": "plain_text", "text": "🚨 Human Intervention Required"}
            },
            {
                "type": "section",
                "text": {"type": "mrkdwn", "text": f"*Task ID:* {task_id}"}
            },
            {
                "type": "section",
                "text": {"type": "mrkdwn", "text": f"*Reason:* {reason}"}
            },
            {
                "type": "section",
                "text": {"type": "mrkdwn", "text": f"*Details:*\n```{details}```"}
            }
        ]
        
        return await self.send_message(channel, f"Escalation: {task_id}", blocks)
```

---

## Phase 5: FastAPI Application

### src/api/app.py

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI, Request, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import structlog

from src.api.routes import webhooks, commands
from src.config.settings import get_settings

logger = structlog.get_logger()

@asynccontextmanager
async def lifespan(app: FastAPI):
    logger.info("starting_application")
    yield
    logger.info("shutting_down_application")

app = FastAPI(
    title="Enterprise AgentCore API",
    version="1.0.0",
    lifespan=lifespan
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)

app.include_router(webhooks.router, prefix="/webhooks", tags=["webhooks"])
app.include_router(commands.router, prefix="/slack", tags=["slack"])

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

### src/api/routes/webhooks.py

```python
from fastapi import APIRouter, Request, HTTPException, BackgroundTasks
from pydantic import BaseModel
import hashlib
import hmac
import structlog

from src.workflows.jira_workflow import JiraWorkflow
from src.workflows.github_workflow import GitHubWorkflow
from src.workflows.sentry_workflow import SentryWorkflow
from src.config.settings import get_settings

logger = structlog.get_logger()
router = APIRouter()

class JiraWebhookPayload(BaseModel):
    webhookEvent: str
    issue: dict
    user: dict | None = None

class GitHubWebhookPayload(BaseModel):
    action: str
    repository: dict
    issue: dict | None = None
    pull_request: dict | None = None
    comment: dict | None = None

@router.post("/jira")
async def jira_webhook(
    request: Request,
    payload: JiraWebhookPayload,
    background_tasks: BackgroundTasks
):
    logger.info("jira_webhook_received", event=payload.webhookEvent)
    
    if payload.webhookEvent == "jira:issue_created":
        workflow = JiraWorkflow()
        background_tasks.add_task(
            workflow.handle_new_ticket,
            payload.issue["key"]
        )
    
    return {"status": "accepted"}

@router.post("/github")
async def github_webhook(
    request: Request,
    background_tasks: BackgroundTasks
):
    settings = get_settings()
    body = await request.body()
    
    signature = request.headers.get("X-Hub-Signature-256", "")
    expected = "sha256=" + hmac.new(
        settings.github_webhook_secret.encode(),
        body,
        hashlib.sha256
    ).hexdigest()
    
    if not hmac.compare_digest(signature, expected):
        raise HTTPException(status_code=401, detail="Invalid signature")
    
    payload = await request.json()
    event_type = request.headers.get("X-GitHub-Event")
    
    logger.info("github_webhook_received", event=event_type, action=payload.get("action"))
    
    if event_type == "issue_comment" and payload.get("action") == "created":
        comment_body = payload.get("comment", {}).get("body", "")
        if comment_body.startswith("@agent"):
            workflow = GitHubWorkflow()
            background_tasks.add_task(
                workflow.handle_mention,
                payload
            )
    
    return {"status": "accepted"}

@router.post("/sentry")
async def sentry_webhook(
    request: Request,
    background_tasks: BackgroundTasks
):
    payload = await request.json()
    logger.info("sentry_webhook_received", action=payload.get("action"))
    
    if payload.get("action") == "triggered":
        workflow = SentryWorkflow()
        background_tasks.add_task(
            workflow.handle_error,
            payload.get("data", {}).get("issue", {})
        )
    
    return {"status": "accepted"}
```

### src/api/routes/commands.py

```python
from fastapi import APIRouter, Request, Form
from fastapi.responses import JSONResponse
import structlog

from src.tools.slack_tools import SlackTools
from src.workflows.jira_workflow import JiraWorkflow
from src.config.settings import get_settings

logger = structlog.get_logger()
router = APIRouter()

@router.post("/commands")
async def slack_command(
    request: Request,
    command: str = Form(...),
    text: str = Form(""),
    user_id: str = Form(...),
    channel_id: str = Form(...),
    response_url: str = Form(...)
):
    logger.info("slack_command_received", command=command, text=text)
    
    if command == "/agent":
        parts = text.split(maxsplit=1)
        subcommand = parts[0] if parts else "help"
        args = parts[1] if len(parts) > 1 else ""
        
        if subcommand == "start":
            workflow = JiraWorkflow()
            await workflow.start_from_slack(args, channel_id, user_id)
            return JSONResponse({
                "response_type": "in_channel",
                "text": f"🚀 Starting task for: {args}"
            })
        
        elif subcommand == "status":
            return JSONResponse({
                "response_type": "ephemeral",
                "text": "📊 Status check coming soon"
            })
        
        else:
            return JSONResponse({
                "response_type": "ephemeral",
                "text": """Available commands:
• `/agent start <TICKET-ID>` - Start working on a ticket
• `/agent status` - Check current task status
• `/agent help` - Show this help message"""
            })
    
    return JSONResponse({"text": "Unknown command"})

@router.post("/interactions")
async def slack_interaction(request: Request):
    payload = (await request.form()).get("payload")
    import json
    data = json.loads(payload)
    
    action_id = data.get("actions", [{}])[0].get("action_id", "")
    
    if action_id.startswith("approve_plan_"):
        task_id = action_id.replace("approve_plan_", "")
        workflow = JiraWorkflow()
        await workflow.approve_plan(task_id)
        
        return JSONResponse({
            "response_type": "in_channel",
            "text": f"✅ Plan approved for task {task_id}"
        })
    
    elif action_id.startswith("reject_plan_"):
        task_id = action_id.replace("reject_plan_", "")
        return JSONResponse({
            "response_type": "in_channel",
            "text": f"❌ Plan rejected for task {task_id}"
        })
    
    return JSONResponse({"text": "Action processed"})
```

---

## Phase 6: Workflow Orchestration

### src/workflows/jira_workflow.py

```python
from typing import Any
from dataclasses import dataclass, field
from enum import Enum
import uuid
import structlog

from src.agents.discovery_agent import DiscoveryAgent
from src.agents.planning_agent import PlanningAgent
from src.agents.base import AgentContext
from src.tools.jira_tools import JiraTools
from src.tools.slack_tools import SlackTools
from src.services.agentcore_memory import AgentCoreMemoryService

logger = structlog.get_logger()

class WorkflowState(Enum):
    PENDING = "pending"
    DISCOVERING = "discovering"
    PLANNING = "planning"
    AWAITING_APPROVAL = "awaiting_approval"
    EXECUTING = "executing"
    MONITORING_CI = "monitoring_ci"
    COMPLETE = "complete"
    FAILED = "failed"

@dataclass
class TaskContext:
    task_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    ticket_id: str = ""
    session_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    state: WorkflowState = WorkflowState.PENDING
    channel_id: str = ""
    pr_url: str = ""
    metadata: dict[str, Any] = field(default_factory=dict)

class JiraWorkflow:
    def __init__(self):
        self.discovery_agent = DiscoveryAgent()
        self.planning_agent = PlanningAgent()
        self.jira_tools = JiraTools()
        self.slack_tools = SlackTools()
        self.memory_service = AgentCoreMemoryService()
    
    async def handle_new_ticket(self, ticket_id: str) -> None:
        logger.info("handling_new_ticket", ticket_id=ticket_id)
        
        ticket_info = await self.jira_tools.get_issue(ticket_id)
        
        if "agent-enabled" not in ticket_info.get("labels", []):
            logger.info("ticket_not_agent_enabled", ticket_id=ticket_id)
            return
        
        task_context = TaskContext(ticket_id=ticket_id)
        
        await self._run_workflow(task_context, ticket_info)
    
    async def start_from_slack(
        self,
        ticket_id: str,
        channel_id: str,
        user_id: str
    ) -> None:
        logger.info("starting_from_slack", ticket_id=ticket_id, user_id=user_id)
        
        ticket_info = await self.jira_tools.get_issue(ticket_id)
        
        task_context = TaskContext(
            ticket_id=ticket_id,
            channel_id=channel_id
        )
        
        await self._run_workflow(task_context, ticket_info)
    
    async def _run_workflow(
        self,
        task_context: TaskContext,
        ticket_info: dict[str, Any]
    ) -> None:
        try:
            task_context.state = WorkflowState.DISCOVERING
            
            agent_context = AgentContext(
                session_id=task_context.session_id,
                task_id=task_context.task_id,
                metadata={"ticket_info": ticket_info}
            )
            
            discovery_result = await self.discovery_agent.process(
                f"Find relevant repositories for: {ticket_info['summary']}\n\n{ticket_info.get('description', '')}",
                agent_context
            )
            
            logger.info("discovery_complete", task_id=task_context.task_id)
            
            task_context.state = WorkflowState.PLANNING
            
            agent_context.metadata["discovery_results"] = discovery_result.artifacts
            
            planning_result = await self.planning_agent.process(
                f"Create implementation plan for ticket {task_context.ticket_id}",
                agent_context
            )
            
            logger.info("planning_complete", task_id=task_context.task_id)
            
            task_context.state = WorkflowState.AWAITING_APPROVAL
            
            if task_context.channel_id:
                pr_url = planning_result.artifacts[0].get("pr_url", "") if planning_result.artifacts else ""
                task_context.pr_url = pr_url
                
                await self.slack_tools.send_plan_notification(
                    channel=task_context.channel_id,
                    task_id=task_context.task_id,
                    ticket_id=task_context.ticket_id,
                    summary=ticket_info["summary"],
                    pr_url=pr_url
                )
            
            await self.jira_tools.add_comment(
                task_context.ticket_id,
                f"🤖 Implementation plan created by AI Agent.\n\nTask ID: {task_context.task_id}\n\nAwaiting human approval."
            )
            
        except Exception as e:
            task_context.state = WorkflowState.FAILED
            logger.error("workflow_failed", task_id=task_context.task_id, error=str(e))
            
            if task_context.channel_id:
                await self.slack_tools.send_escalation(
                    channel=task_context.channel_id,
                    task_id=task_context.task_id,
                    reason="Workflow failed",
                    details=str(e)
                )
    
    async def approve_plan(self, task_id: str) -> None:
        logger.info("plan_approved", task_id=task_id)
```

---

## Phase 7: Terraform Infrastructure

### infrastructure/terraform/main.tf

```hcl
terraform {
  required_version = ">= 1.6.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.80"
    }
  }
  
  backend "s3" {
    bucket         = "enterprise-agents-tfstate"
    key            = "agentcore/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

variable "aws_region" {
  default = "us-east-1"
}

variable "project_name" {
  default = "enterprise-agentcore"
}

variable "environment" {
  default = "prod"
}

variable "github_token" {
  sensitive = true
}

variable "jira_api_token" {
  sensitive = true
}

variable "slack_bot_token" {
  sensitive = true
}
```

### infrastructure/terraform/agentcore.tf

```hcl
resource "aws_bedrockagentcore_runtime" "main" {
  runtime_name = "${var.project_name}-runtime"
  
  runtime_configuration {
    vcpu               = 1.0
    memory_mb          = 2048
    timeout_seconds    = 900
    session_ttl_seconds = 3600
  }
  
  agent_source {
    source_type = "ECR"
    ecr_configuration {
      repository_uri = aws_ecr_repository.agent.repository_url
      image_tag      = "latest"
    }
  }
  
  execution_role_arn = aws_iam_role.agentcore_execution.arn
  
  observability_configuration {
    cloudwatch_logs_enabled = true
    log_group_arn          = aws_cloudwatch_log_group.agentcore.arn
    opentelemetry_enabled  = true
  }
}

resource "aws_bedrockagentcore_memory" "main" {
  memory_name = "${var.project_name}-memory"
  
  short_term_configuration {
    max_events   = 100
    ttl_seconds  = 3600
  }
  
  long_term_configuration {
    embedding_model_id  = "amazon.titan-embed-text-v2:0"
    vector_dimensions   = 1024
    storage_type        = "OPENSEARCH_SERVERLESS"
  }
}

resource "aws_bedrockagentcore_gateway" "main" {
  gateway_name = "${var.project_name}-gateway"
  description  = "Tool gateway for enterprise agents"
  
  semantic_search_configuration {
    enabled            = true
    embedding_model_id = "amazon.titan-embed-text-v2:0"
  }
}

resource "aws_bedrockagentcore_gateway_tool" "github" {
  gateway_id = aws_bedrockagentcore_gateway.main.id
  tool_name  = "github"
  
  source_type = "LAMBDA"
  lambda_configuration {
    lambda_arn = aws_lambda_function.github_tools.arn
  }
  
  description = "GitHub API operations"
  
  input_schema = jsonencode({
    type = "object"
    properties = {
      operation = {
        type = "string"
        enum = ["search_code", "get_file", "create_branch", "create_pr"]
      }
      parameters = {
        type = "object"
      }
    }
  })
}

resource "aws_bedrockagentcore_gateway_tool" "jira" {
  gateway_id = aws_bedrockagentcore_gateway.main.id
  tool_name  = "jira"
  
  source_type = "OPENAPI"
  openapi_configuration {
    specification_s3_uri = "s3://${aws_s3_bucket.config.bucket}/openapi/jira.yaml"
    base_url            = var.jira_base_url
  }
  
  authentication {
    type = "BASIC"
    credentials_secret_arn = aws_secretsmanager_secret.jira_credentials.arn
  }
}

resource "aws_iam_role" "agentcore_execution" {
  name = "${var.project_name}-agentcore-execution"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "bedrock-agentcore.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy" "agentcore_permissions" {
  name = "${var.project_name}-agentcore-permissions"
  role = aws_iam_role.agentcore_execution.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "bedrock:InvokeModel",
          "bedrock:InvokeModelWithResponseStream"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:GetSecretValue"
        ]
        Resource = [
          aws_secretsmanager_secret.github_token.arn,
          aws_secretsmanager_secret.jira_credentials.arn,
          aws_secretsmanager_secret.slack_token.arn
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "lambda:InvokeFunction"
        ]
        Resource = [
          aws_lambda_function.github_tools.arn,
          aws_lambda_function.slack_handler.arn
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "${aws_cloudwatch_log_group.agentcore.arn}:*"
      }
    ]
  })
}

resource "aws_cloudwatch_log_group" "agentcore" {
  name              = "/aws/bedrock-agentcore/${var.project_name}"
  retention_in_days = 30
}

resource "aws_ecr_repository" "agent" {
  name                 = "${var.project_name}-agent"
  image_tag_mutability = "MUTABLE"
  
  image_scanning_configuration {
    scan_on_push = true
  }
}

resource "aws_secretsmanager_secret" "github_token" {
  name = "${var.project_name}/github-token"
}

resource "aws_secretsmanager_secret_version" "github_token" {
  secret_id     = aws_secretsmanager_secret.github_token.id
  secret_string = var.github_token
}

resource "aws_secretsmanager_secret" "jira_credentials" {
  name = "${var.project_name}/jira-credentials"
}

resource "aws_secretsmanager_secret" "slack_token" {
  name = "${var.project_name}/slack-token"
}

resource "aws_secretsmanager_secret_version" "slack_token" {
  secret_id     = aws_secretsmanager_secret.slack_token.id
  secret_string = var.slack_bot_token
}

output "runtime_endpoint" {
  value = aws_bedrockagentcore_runtime.main.runtime_endpoint
}

output "memory_id" {
  value = aws_bedrockagentcore_memory.main.id
}

output "gateway_id" {
  value = aws_bedrockagentcore_gateway.main.id
}
```

---

## Pricing Breakdown

| Service | Unit | Price | Monthly Est (50K sessions) |
|---------|------|-------|---------------------------|
| **Runtime** | vCPU-second | $0.0000125 | ~$11.25 |
| **Runtime** | GB-second | $0.00000125 | ~$7.50 |
| **Memory (Short-term)** | 1K events | $0.25 | ~$12.50 |
| **Memory (Long-term)** | 1K stored | $0.75 | ~$7.50 |
| **Memory (Retrieval)** | 1K reads | $0.50 | ~$5.00 |
| **Gateway** | 1K API calls | $0.005 | ~$2.50 |
| **Gateway** | 1K searches | $0.025 | ~$1.25 |
| **Code Interpreter** | Session | $0.00003/sec | ~$15.00 |
| **Browser** | Session | $0.0001/sec | ~$25.00 |
| **Bedrock Claude Sonnet** | 1M tokens in | $3.00 | ~$150.00 |
| **Bedrock Claude Sonnet** | 1M tokens out | $15.00 | ~$300.00 |
| | | **Total** | **~$537.50/month** |

> Note: Estimates based on 50K sessions/month with average 20 turns per session

---

## Deployment

### scripts/deploy.sh

```bash
#!/bin/bash
set -euo pipefail

PROJECT_NAME="enterprise-agentcore"
AWS_REGION="${AWS_REGION:-us-east-1}"
ENVIRONMENT="${ENVIRONMENT:-prod}"

echo "🚀 Deploying $PROJECT_NAME to $ENVIRONMENT"

cd "$(dirname "$0")/.."

echo "📦 Building container image..."
docker build -t $PROJECT_NAME:latest .

echo "🔑 Logging into ECR..."
aws ecr get-login-password --region $AWS_REGION | \
    docker login --username AWS --password-stdin \
    $(aws sts get-caller-identity --query Account --output text).dkr.ecr.$AWS_REGION.amazonaws.com

ECR_URI=$(terraform -chdir=infrastructure/terraform output -raw ecr_repository_url 2>/dev/null || echo "")
if [ -z "$ECR_URI" ]; then
    echo "⚠️  ECR repository not found, running terraform first..."
    cd infrastructure/terraform
    terraform init
    terraform apply -target=aws_ecr_repository.agent -auto-approve
    ECR_URI=$(terraform output -raw ecr_repository_url)
    cd ../..
fi

echo "📤 Pushing image to ECR..."
docker tag $PROJECT_NAME:latest $ECR_URI:latest
docker push $ECR_URI:latest

echo "🏗️  Applying Terraform..."
cd infrastructure/terraform
terraform init
terraform plan -out=tfplan
terraform apply tfplan

RUNTIME_ENDPOINT=$(terraform output -raw runtime_endpoint)
GATEWAY_ID=$(terraform output -raw gateway_id)
MEMORY_ID=$(terraform output -raw memory_id)

cd ../..
echo ""
echo "✅ Deployment complete!"
echo ""
echo "Runtime Endpoint: $RUNTIME_ENDPOINT"
echo "Gateway ID: $GATEWAY_ID"
echo "Memory ID: $MEMORY_ID"
echo ""
echo "🔧 Configure webhooks:"
echo "  Jira:   https://your-api-gateway.execute-api.$AWS_REGION.amazonaws.com/prod/webhooks/jira"
echo "  GitHub: https://your-api-gateway.execute-api.$AWS_REGION.amazonaws.com/prod/webhooks/github"
echo "  Slack:  https://your-api-gateway.execute-api.$AWS_REGION.amazonaws.com/prod/slack/commands"
```

### Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN pip install uv

COPY pyproject.toml .
RUN uv pip install --system -e .

COPY src/ src/

ENV PYTHONPATH=/app
ENV PORT=8080

EXPOSE 8080

CMD ["uvicorn", "src.api.app:app", "--host", "0.0.0.0", "--port", "8080"]
```

---

## Quick Start

```bash
git clone https://github.com/your-org/enterprise-agentcore.git
cd enterprise-agentcore

uv venv
source .venv/bin/activate

uv pip install -e ".[dev]"

cp .env.example .env

uvicorn src.api.app:app --reload

./scripts/deploy.sh
```

---

## Summary

This implementation provides:

1. **Complete AgentCore Integration** - Runtime, Memory, Gateway, Code Interpreter, Browser
2. **Multi-Agent System** - Discovery, Planning, Execution, CI/CD agents
3. **LangGraph Workflows** - Stateful, graph-based agent orchestration
4. **Enterprise Integrations** - GitHub, Jira, Slack, Sentry
5. **Production Infrastructure** - Terraform, Docker, CI/CD ready
6. **Pay-per-use Pricing** - Only pay for active resources

AgentCore provides the critical infrastructure layer that makes production AI agents possible at enterprise scale.

---

## Official MCP Servers Integration

### Official MCP Servers Available

| Server | Repository | Description |
|--------|------------|-------------|
| **GitHub** | `github/github-mcp-server` | Official GitHub MCP - repos, issues, PRs |
| **Atlassian Rovo** | `atlassian/atlassian-mcp-server` | Official Jira + Confluence MCP |

> **Note**: Sentry does NOT have an official MCP server. Use the **Sentry Python SDK** instead.

### GitHub Official MCP Server

**Repository**: https://github.com/github/github-mcp-server

```bash
docker run -i --rm \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=<your-token> \
  ghcr.io/github/github-mcp-server
```

| Tool | Description |
|------|-------------|
| `search_repositories` | Search GitHub repositories |
| `search_code` | Search code across repos |
| `get_file_contents` | Get file contents |
| `create_issue` | Create issue |
| `create_pull_request` | Create PR |
| `create_branch` | Create branch |
| `list_commits` | List commits |
| `fork_repository` | Fork repo |

### Atlassian Rovo MCP Server

**Repository**: https://github.com/atlassian/atlassian-mcp-server

| Tool | Description |
|------|-------------|
| `jira_search` | Search issues using JQL |
| `jira_get_issue` | Get issue details |
| `jira_create_issue` | Create new issue |
| `jira_update_issue` | Update issue fields |
| `jira_transition_issue` | Change issue status |
| `confluence_search` | Search pages (CQL) |
| `confluence_get_page` | Get page content |

### src/services/official_mcp_manager.py

```python
import boto3
from typing import Any
import structlog

from src.config.settings import get_settings

logger = structlog.get_logger()

class OfficialMCPManager:
    def __init__(self):
        self.settings = get_settings()
        self.client = boto3.client(
            "bedrock-agentcore-gateway",
            region_name=self.settings.aws_region
        )
    
    async def register_github_mcp(self, gateway_id: str) -> dict[str, Any]:
        response = self.client.create_mcp_connection(
            gatewayId=gateway_id,
            connectionName="github-official",
            mcpServerConfiguration={
                "serverType": "REMOTE",
                "remoteConfiguration": {
                    "serverUrl": "https://api.mcp.github.com",
                    "transportProtocol": "SSE"
                }
            },
            authentication={
                "type": "TOKEN",
                "credentialsSecretArn": f"arn:aws:secretsmanager:{self.settings.aws_region}:*:secret:github-token"
            }
        )
        logger.info("registered_github_official_mcp", connection_id=response["connectionId"])
        return response
    
    async def register_atlassian_mcp(self, gateway_id: str) -> dict[str, Any]:
        response = self.client.create_mcp_connection(
            gatewayId=gateway_id,
            connectionName="atlassian-rovo",
            mcpServerConfiguration={
                "serverType": "REMOTE",
                "remoteConfiguration": {
                    "serverUrl": "https://mcp.atlassian.com",
                    "transportProtocol": "SSE"
                }
            },
            authentication={
                "type": "OAUTH",
                "oauthConfiguration": {
                    "provider": "ATLASSIAN",
                    "scopes": ["read:jira-work", "write:jira-work", "read:confluence-content.all"]
                }
            }
        )
        logger.info("registered_atlassian_official_mcp", connection_id=response["connectionId"])
        return response
    
    async def register_all(self, gateway_id: str) -> dict[str, Any]:
        results = {}
        results["github"] = await self.register_github_mcp(gateway_id)
        results["atlassian"] = await self.register_atlassian_mcp(gateway_id)
        return results
```

---

## Sentry Integration (SDK - NOT MCP)

> ⚠️ **Sentry does NOT have an official MCP server.**
> Use the **Sentry Python SDK** for AI agent monitoring.

```bash
pip install sentry-sdk[openai,langchain]
```

### src/services/sentry_service.py

```python
import sentry_sdk
from sentry_sdk.integrations.langchain import LangchainIntegration
import structlog

from src.config.settings import get_settings

logger = structlog.get_logger()

class SentryService:
    def __init__(self):
        self.settings = get_settings()
        self._initialized = False
    
    def initialize(self) -> None:
        sentry_sdk.init(
            dsn=self.settings.sentry_dsn,
            traces_sample_rate=1.0,
            profiles_sample_rate=1.0,
            send_default_pii=True,
            integrations=[LangchainIntegration()],
            environment=self.settings.environment,
        )
        self._initialized = True
        logger.info("initialized_sentry_sdk")
    
    def capture_exception(self, error: Exception) -> str:
        return sentry_sdk.capture_exception(error)
    
    def start_transaction(self, name: str, op: str = "agent.workflow"):
        return sentry_sdk.start_transaction(name=name, op=op)
```

### AI Agent Monitoring Features
| Feature | Description |
|---------|-------------|
| **Token Usage** | Track input/output tokens per LLM call |
| **Latency** | Measure response times |
| **Tool Traces** | See tool calls as spans |
| **Error Tracking** | Capture agent failures |

---

## Terraform for Official MCP

### infrastructure/terraform/official_mcp.tf

```hcl
resource "aws_bedrockagentcore_gateway_mcp_connection" "github" {
  gateway_id      = aws_bedrockagentcore_gateway.main.id
  connection_name = "github-official"
  
  mcp_server_configuration {
    server_type = "REMOTE"
    remote_configuration {
      server_url         = "https://api.mcp.github.com"
      transport_protocol = "SSE"
    }
  }
  
  authentication {
    type                   = "TOKEN"
    credentials_secret_arn = aws_secretsmanager_secret.github_token.arn
  }
}

resource "aws_bedrockagentcore_gateway_mcp_connection" "atlassian" {
  gateway_id      = aws_bedrockagentcore_gateway.main.id
  connection_name = "atlassian-rovo"
  
  mcp_server_configuration {
    server_type = "REMOTE"
    remote_configuration {
      server_url         = "https://mcp.atlassian.com"
      transport_protocol = "SSE"
    }
  }
  
  authentication {
    type = "OAUTH"
    oauth_configuration {
      provider = "ATLASSIAN"
      scopes   = ["read:jira-work", "write:jira-work", "read:confluence-content.all"]
    }
  }
}
```

```
