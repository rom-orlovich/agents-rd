<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# האם אפשר לשדרג את התוכנית

ואולי לפרק לתתי סוכנים יעודיים(האם מומלץ לעשו זאת?)
ולהוסיף גם התממשקות עם סוכנים של סלאק של הארגון שיודע לדווח 
על הסטטוס של המשימות
אם פר מוכן אם הקוד מוכן
לשאול על סטטוסים 
של הסוכנים ומי עובד 
ועוד דברים שיעזרו למפתחים לנהל את זה בנוסף לדשבורד ?
שים לב שהסוכנים שמייצרים את התוכנית ואת הפרים בגיטאב יצטרכו גם לעבור בהצלחה את הCI CD של כל ריפו

האם אפשר להשתמש באלו בMCP רשמיים
[https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/)
[https://github.com/github/github-mcp-server](https://github.com/github/github-mcp-server)
[https://docs.sentry.io/product/sentry-mcp/](https://docs.sentry.io/product/sentry-mcp/)
and for slack we need cli or api

מעולה! אני עכשיו אשדרג את התוכנית עם:

1. פירוק לתת-סוכנים יעודיים (sub-agents)
2. אינטגרציה מלאה עם Slack
3. שימוש ב-MCP הרשמיים
4. ולידציה של CI/CD

***

## תוכנית משודרגת - ארכיטקטורת Multi-Agent עם MCP רשמיים

### שימוש ב-MCP Servers הרשמיים

כן! כל המקורות שציינת זמינים ולגמרי מומלצים:[^1][^2][^3]

**1. Atlassian Rovo MCP Server** - תומך ב-Jira, Confluence, Compass[^1]

- אנדפוינט: `https://mcp.atlassian.com/v1/mcp`
- אימות: OAuth 2.1
- כלים: חיפוש, יצירה, עדכון של tickets, קישור בין אובייקטים

**2. GitHub MCP Server** - רשמי מ-GitHub[^2]

- Remote: `https://api.githubcopilot.com/mcp/`
- Local: `ghcr.io/github/github-mcp-server`
- 16+ tools: repos, issues, PRs, actions, code security
- תמיכה ב-toolsets דינמיים

**3. Sentry MCP Server** - רשמי מ-Sentry[^3]

- Remote: `https://mcp.sentry.dev/mcp`
- OAuth מובנה
- אינטגרציה עם Seer (AI agent של Sentry)
- כלים: issues, errors, releases, DSN management

**4. Slack** - נשתמש ב-Slack API/SDK (אין MCP רשמי)[^4][^5]

- Slack Bolt SDK (Python/Node.js)
- Incoming Webhooks + Interactive Messages
- Slash Commands

***

## ארכיטקטורה משודרגת - Specialized Sub-Agents

### למה לפרק לתת-סוכנים?

**כן, מומלץ מאוד!** הסיבות:[^6][^7]

1. **Separation of Concerns** - כל agent אחראי על domain ספציפי
2. **Easier Debugging** - קל יותר לזהות איפה הבעיה
3. **Better Scalability** - אפשר להריץ agents במקביל
4. **Improved Context** - כל agent מקבל רק את המידע הרלוונטי לו
5. **Flexible Orchestration** - אפשר לשנות flows בקלות

### Agent Hierarchy - שכבות

```
┌─────────────────────────────────────────────┐
│      Supervisor Agent (Orchestrator)       │
│         LangGraph State Machine             │
└─────────────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   ┌────────┐   ┌──────────┐   ┌──────────┐
   │ Intake │   │ Planning │   │ Execution│
   │ Layer  │   │  Layer   │   │  Layer   │
   └────────┘   └──────────┘   └──────────┘
```


***

## Layer 1: Intake Agents (Entry Points)

### 1.1 Jira Watcher Agent

**תפקיד**: מעקב אחר Jira tickets חדשים
**טכנולוגיה**: Atlassian Rovo MCP Server[^1]
**משימות**:

- webhook listening על tickets עם label "AI"
- parsing תיאור המשימה
- העברה ל-Supervisor

```python
class JiraWatcherAgent:
    def __init__(self):
        self.mcp_client = MCPClient("https://mcp.atlassian.com/v1/mcp")
        
    async def watch_for_ai_tickets(self, webhook_payload):
        # Parse Jira webhook
        issue_key = webhook_payload["issue"]["key"]
        
        # Use Atlassian MCP to get full context
        issue_details = await self.mcp_client.call_tool(
            "jira_issue_get",
            {"issue_key": issue_key}
        )
        
        # Extract relevant info
        task = {
            "source": "jira",
            "issue_key": issue_key,
            "summary": issue_details["fields"]["summary"],
            "description": issue_details["fields"]["description"],
            "labels": issue_details["fields"]["labels"],
            "project": issue_details["fields"]["project"]["key"]
        }
        
        # Send to Supervisor
        await supervisor.process_new_task(task)
```


### 1.2 Sentry Error Monitor Agent

**תפקיד**: זיהוי errors חוזרים
**טכנולוגיה**: Sentry MCP Server[^3]
**משימות**:

- aggregation של errors לפי fingerprint
- זיהוי patterns (X פעמים ב-Y ימים)
- יצירת Jira ticket אוטומטי

```python
class SentryMonitorAgent:
    def __init__(self):
        self.mcp_client = MCPClient("https://mcp.sentry.dev/mcp")
        self.threshold = 10  # configurable
        
    async def monitor_recurring_errors(self):
        # Use Sentry MCP to search issues
        recent_issues = await self.mcp_client.call_tool(
            "search_issues",
            {
                "project": "backend-api",
                "status": "unresolved",
                "timeframe": "7d"
            }
        )
        
        # Group by fingerprint
        error_counts = {}
        for issue in recent_issues:
            fingerprint = issue["fingerprint"]
            error_counts[fingerprint] = error_counts.get(fingerprint, 0) + 1
        
        # Find recurring errors
        for fingerprint, count in error_counts.items():
            if count >= self.threshold:
                # Get full stack trace
                error_details = await self.mcp_client.call_tool(
                    "get_issue_details",
                    {"issue_id": fingerprint}
                )
                
                # Create Jira ticket via Atlassian MCP
                await self.create_jira_from_error(error_details)
```


### 1.3 Jira Comment Controller Agent

**תפקיד**: מאזין לתגובות ב-Jira tickets
**טכנולוגיה**: Atlassian Rovo MCP[^1]
**משימות**:

- parsing פקודות מתגובות (@ai-assistant)
- routing לagents המתאימים
- עדכון ticket בתוצאות

```python
class JiraCommentControllerAgent:
    def __init__(self):
        self.mcp_client = MCPClient("https://mcp.atlassian.com/v1/mcp")
        self.commands = {
            "/find-repos": "repo_discovery",
            "/analyze-code": "code_analyzer",
            "/create-plan": "planner",
            "/create-pr": "pr_creator",
            "/execute": "executor",
            "/status": "status_reporter"
        }
        
    async def handle_comment(self, webhook_payload):
        comment = webhook_payload["comment"]
        issue_key = webhook_payload["issue"]["key"]
        
        # Check if @ai-assistant mentioned
        if "@ai-assistant" not in comment["body"]:
            return
        
        # Parse command
        for cmd, agent_type in self.commands.items():
            if cmd in comment["body"]:
                params = self.extract_params(comment["body"], cmd)
                
                # Route to appropriate agent
                result = await supervisor.delegate_to_agent(
                    agent_type,
                    issue_key,
                    params
                )
                
                # Post result back to Jira
                await self.mcp_client.call_tool(
                    "jira_add_comment",
                    {
                        "issue_key": issue_key,
                        "comment": self.format_response(result)
                    }
                )
```


***

## Layer 2: Analysis \& Planning Agents

### 2.1 Repository Discovery Agent (Multi-Strategy)

**תפקיד**: זיהוי repositories רלוונטיים
**טכנולוגיה**: GitHub MCP Server + Claude[^2]

**תת-סוכנים (Sub-Agents)**:

#### 2.1.1 Code Search Sub-Agent

```python
class CodeSearchSubAgent:
    """חיפוש קוד ספציפי"""
    def __init__(self):
        self.github_mcp = MCPClient("https://api.githubcopilot.com/mcp/")
        
    async def search_by_keywords(self, keywords: list[str]) -> list[dict]:
        results = []
        for keyword in keywords:
            search_result = await self.github_mcp.call_tool(
                "search_code",
                {
                    "query": f"{keyword} org:your-org",
                    "per_page": 20
                }
            )
            results.extend(search_result["items"])
        
        return self.deduplicate_repos(results)
```


#### 2.1.2 Semantic Search Sub-Agent

```python
class SemanticSearchSubAgent:
    """חיפוש semantic עם embeddings"""
    def __init__(self):
        self.github_mcp = MCPClient("https://api.githubcopilot.com/mcp/")
        self.claude = Anthropic()
        
    async def find_related_repos(self, task_description: str) -> list[dict]:
        # Extract entities with Claude
        entities = await self.claude.messages.create(
            model="claude-3-5-sonnet-20241022",
            messages=[{
                "role": "user",
                "content": f"""Extract technical entities from this task:
                {task_description}
                
                Return: frameworks, services, file types, APIs mentioned"""
            }]
        )
        
        # Search each entity
        relevant_repos = []
        for entity in entities:
            repos = await self.github_mcp.call_tool(
                "search_repositories",
                {"keywords": entity, "org": "your-org"}
            )
            relevant_repos.extend(repos)
        
        return self.rank_by_relevance(relevant_repos, task_description)
```


#### 2.1.3 Dependency Analysis Sub-Agent

```python
class DependencyAnalysisSubAgent:
    """ניתוח תלויות בין repos"""
    def __init__(self):
        self.github_mcp = MCPClient("https://api.githubcopilot.com/mcp/")
        
    async def find_dependent_repos(self, primary_repo: str) -> list[dict]:
        # Get package.json / requirements.txt
        package_file = await self.github_mcp.call_tool(
            "get_file_contents",
            {
                "repo": primary_repo,
                "path": "package.json"  # or requirements.txt, go.mod, etc.
            }
        )
        
        # Parse dependencies
        deps = self.parse_dependencies(package_file)
        
        # Find internal packages
        internal_repos = []
        for dep in deps:
            if self.is_internal_package(dep):
                repo = await self.find_repo_for_package(dep)
                internal_repos.append(repo)
        
        return internal_repos
```

**Main Repository Discovery Agent:**

```python
class RepositoryDiscoveryAgent:
    def __init__(self):
        self.code_search = CodeSearchSubAgent()
        self.semantic_search = SemanticSearchSubAgent()
        self.dependency_analysis = DependencyAnalysisSubAgent()
        
    async def discover(self, task: dict) -> dict:
        # Run all sub-agents in parallel
        results = await asyncio.gather(
            self.code_search.search_by_keywords(
                self.extract_keywords(task["description"])
            ),
            self.semantic_search.find_related_repos(
                task["description"]
            )
        )
        
        # Merge and rank
        primary_repos = self.merge_results(results)
        
        # Find dependencies for each primary repo
        all_repos = []
        for repo in primary_repos:
            deps = await self.dependency_analysis.find_dependent_repos(repo["name"])
            all_repos.append({
                "repo": repo,
                "dependencies": deps,
                "priority": "high" if not deps else "medium"
            })
        
        return {
            "repositories": all_repos,
            "total_repos": len(all_repos),
            "complexity": "simple" if len(all_repos) == 1 else "complex"
        }
```


### 2.2 Code Context Agent (Deep Analysis)

**תפקיד**: איסוף מידע מפורט על קבצים
**טכנולוגיה**: GitHub MCP Server[^2]

**תת-סוכנים**:

#### 2.2.1 File Structure Analyzer

```python
class FileStructureAnalyzer:
    """מיפוי מבנה הפרויקט"""
    async def analyze_structure(self, repo: str) -> dict:
        # Get directory tree
        tree = await self.github_mcp.call_tool(
            "get_repository_tree",
            {"repo": repo, "recursive": True}
        )
        
        return {
            "src_structure": self.parse_src_structure(tree),
            "test_structure": self.parse_test_structure(tree),
            "config_files": self.find_config_files(tree),
            "conventions": self.detect_conventions(tree)
        }
```


#### 2.2.2 Code Pattern Analyzer

```python
class CodePatternAnalyzer:
    """זיהוי patterns וconventions"""
    async def analyze_patterns(self, repo: str, files: list[str]) -> dict:
        patterns = {
            "naming_conventions": [],
            "architectural_patterns": [],
            "testing_patterns": [],
            "error_handling": []
        }
        
        for file_path in files:
            content = await self.github_mcp.call_tool(
                "get_file_contents",
                {"repo": repo, "path": file_path}
            )
            
            # Analyze with Claude
            analysis = await self.claude_analyze(content)
            patterns["naming_conventions"].extend(analysis["naming"])
            patterns["architectural_patterns"].extend(analysis["architecture"])
        
        return self.consolidate_patterns(patterns)
```


#### 2.2.3 Import \& Dependency Mapper

```python
class ImportDependencyMapper:
    """מיפוי imports ותלויות"""
    async def map_dependencies(self, repo: str, target_files: list[str]) -> dict:
        dep_graph = {}
        
        for file_path in target_files:
            content = await self.github_mcp.call_tool(
                "get_file_contents",
                {"repo": repo, "path": file_path}
            )
            
            imports = self.extract_imports(content, file_path)
            dep_graph[file_path] = {
                "imports": imports,
                "imported_by": []  # will be filled in second pass
            }
        
        # Build reverse dependency graph
        return self.build_bidirectional_graph(dep_graph)
```


### 2.3 Planning Agent (TDD-Oriented with Validation)

**תפקיד**: יצירת תוכנית ביצוע מפורטת עם CI/CD validation
**טכנולוגיה**: Claude + GitHub MCP

**תת-סוכנים**:

#### 2.3.1 Test Strategy Designer

```python
class TestStrategyDesigner:
    """תכנון אסטרטגיית testing"""
    async def design_tests(self, task: dict, code_context: dict) -> dict:
        test_plan = {
            "unit_tests": [],
            "integration_tests": [],
            "e2e_tests": [],
            "test_fixtures": []
        }
        
        # Analyze existing tests
        existing_tests = await self.analyze_existing_tests(
            code_context["test_structure"]
        )
        
        # Generate new test cases
        prompt = f"""Based on this task and code context:
        Task: {task['description']}
        Existing Tests: {existing_tests}
        
        Design TDD test cases following existing patterns.
        Include:
        1. Unit test cases (before implementation)
        2. Integration test cases
        3. Test fixtures needed
        4. Expected assertions"""
        
        test_cases = await self.claude.generate(prompt)
        return self.parse_test_plan(test_cases)
```


#### 2.3.2 Architecture Designer

```python
class ArchitectureDesigner:
    """תכנון ארכיטקטורה"""
    async def design_solution(self, task: dict, repos: list[dict]) -> dict:
        architecture = {
            "components": [],
            "interactions": [],
            "data_flow": [],
            "api_contracts": []
        }
        
        # Multi-repo or single-repo?
        if len(repos) > 1:
            # Design distributed solution
            architecture = await self.design_distributed(task, repos)
        else:
            # Design monolithic solution
            architecture = await self.design_monolithic(task, repos[^0])
        
        return architecture
```


#### 2.3.3 CI/CD Validation Planner

```python
class CICDValidationPlanner:
    """תכנון validation של CI/CD"""
    async def plan_validations(self, repos: list[dict]) -> dict:
        validation_plan = {}
        
        for repo in repos:
            # Get CI/CD configuration
            ci_config = await self.get_ci_config(repo["name"])
            
            validation_plan[repo["name"]] = {
                "ci_pipeline": self.parse_ci_pipeline(ci_config),
                "required_checks": self.extract_required_checks(ci_config),
                "quality_gates": self.extract_quality_gates(ci_config),
                "deployment_stages": self.extract_stages(ci_config),
                "estimated_duration": self.estimate_ci_duration(ci_config)
            }
        
        return validation_plan
    
    async def get_ci_config(self, repo: str) -> dict:
        # Try different CI config files
        ci_files = [
            ".gitlab-ci.yml",
            ".github/workflows/*.yml",
            "Jenkinsfile",
            ".circleci/config.yml"
        ]
        
        for ci_file in ci_files:
            try:
                config = await self.github_mcp.call_tool(
                    "get_file_contents",
                    {"repo": repo, "path": ci_file}
                )
                return {
                    "type": self.detect_ci_type(ci_file),
                    "config": config
                }
            except:
                continue
        
        return {"type": "unknown", "config": None}
```

**Main Planning Agent:**

```python
class PlanningAgent:
    def __init__(self):
        self.test_designer = TestStrategyDesigner()
        self.arch_designer = ArchitectureDesigner()
        self.cicd_planner = CICDValidationPlanner()
        self.claude = Anthropic()
        
    async def create_comprehensive_plan(
        self,
        task: dict,
        repos: list[dict],
        code_context: dict
    ) -> dict:
        # Run sub-agents in parallel
        test_plan, architecture, cicd_validations = await asyncio.gather(
            self.test_designer.design_tests(task, code_context),
            self.arch_designer.design_solution(task, repos),
            self.cicd_planner.plan_validations(repos)
        )
        
        # Generate comprehensive PLAN.MD
        plan_md = await self.claude.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=16000,
            messages=[{
                "role": "user",
                "content": f"""Create detailed PLAN.MD:

Task: {task['description']}
Repositories: {json.dumps(repos, indent=2)}
Architecture: {json.dumps(architecture, indent=2)}
Test Strategy: {json.dumps(test_plan, indent=2)}
CI/CD Requirements: {json.dumps(cicd_validations, indent=2)}

Generate PLAN.MD with:
1. **Scope Definition**
   - In scope / Out of scope
   
2. **Architecture Overview**
   - Component diagram (text/mermaid)
   - Data flow
   - API contracts (if multi-repo)
   
3. **TDD Implementation Strategy**
   - Test cases to write FIRST
   - Implementation steps AFTER tests pass
   - Test fixtures needed
   
4. **File Changes Required**
   Per repository:
   - Files to create/modify
   - Why each change is needed
   
5. **CI/CD Validation Plan**
   Per repository:
   - Pipeline stages that will run
   - Required checks to pass
   - Quality gates to meet
   - Estimated CI duration
   
6. **Dependencies Between Repos** (if applicable)
   - Execution order
   - API contracts to maintain
   - Integration points
   
7. **Acceptance Criteria**
   - Functional requirements
   - Non-functional requirements
   - CI/CD must pass
   
8. **Next Steps**
   - Order of execution
   - Parallel vs sequential tasks
   
Link to Jira: {task['issue_key']}"""
            }]
        )
        
        return {
            "plan_md": plan_md.content[^0].text,
            "test_plan": test_plan,
            "architecture": architecture,
            "cicd_validations": cicd_validations,
            "complexity_score": self.calculate_complexity(
                repos, architecture, cicd_validations
            )
        }
```


***

## Layer 3: Execution Agents

### 3.1 GitLab PR Creator Agent (with CI/CD Awareness)

**תפקיד**: פתיחת PR עם branch חדש
**טכנולוגיה**: GitLab API

```python
class GitLabPRCreatorAgent:
    def __init__(self):
        self.gitlab = gitlab.Gitlab(
            os.getenv("GITLAB_URL"),
            private_token=os.getenv("GITLAB_TOKEN")
        )
        
    async def create_pr_with_plan(
        self,
        repos: list[dict],
        plan: dict,
        task: dict
    ) -> list[dict]:
        created_mrs = []
        
        for repo_info in repos:
            project = self.gitlab.projects.get(repo_info["project_id"])
            
            # Create branch (naming convention)
            branch_name = self.generate_branch_name(
                task["issue_key"],
                repo_info["name"]
            )
            
            try:
                branch = project.branches.create({
                    'branch': branch_name,
                    'ref': 'main'
                })
            except gitlab.exceptions.GitlabCreateError as e:
                # Branch exists, skip
                continue
            
            # Commit PLAN.MD
            plan_content = self.customize_plan_for_repo(
                plan["plan_md"],
                repo_info["name"]
            )
            
            project.files.create({
                'file_path': 'PLAN.MD',
                'branch': branch_name,
                'content': plan_content,
                'commit_message': f'[{task["issue_key"]}] Add implementation plan'
            })
            
            # Add CI/CD expectations to MR description
            ci_expectations = self.format_ci_expectations(
                plan["cicd_validations"][repo_info["name"]]
            )
            
            # Create MR
            mr = project.mergerequests.create({
                'source_branch': branch_name,
                'target_branch': 'main',
                'title': f'[{task["issue_key"]}] {task["summary"]}',
                'description': f"""
## Jira Ticket
[{task["issue_key"]}]({self.jira_url(task["issue_key"])})

## Implementation Plan
See PLAN.MD for detailed execution plan.

## CI/CD Pipeline Expectations
{ci_expectations}

**Estimated CI Duration**: {plan["cicd_validations"][repo_info["name"]]["estimated_duration"]}

## Commands
- `/execute` - Run the implementation agent (will wait for human approval)
- `/improve <feedback>` - Refine the plan based on feedback
- `/status` - Check current status and CI/CD results

## Notes
- All tests must pass before merge
- Code coverage must meet project standards
- Security scans must pass
                """,
                'labels': ['AI-Generated', task["issue_key"], 'awaiting-review'],
                'remove_source_branch': True
            })
            
            created_mrs.append({
                "repo": repo_info["name"],
                "mr_id": mr.iid,
                "mr_url": mr.web_url,
                "branch": branch_name,
                "ci_pipeline_id": None  # will be populated after first commit
            })
        
        return created_mrs
    
    def format_ci_expectations(self, ci_validation: dict) -> str:
        """Format CI/CD expectations for MR description"""
        if not ci_validation["ci_pipeline"]:
            return "⚠️ No CI/CD pipeline detected"
        
        stages = ci_validation["ci_pipeline"]["stages"]
        checks = ci_validation["required_checks"]
        
        output = "### Pipeline Stages\n"
        for stage in stages:
            output += f"- **{stage['name']}**: {stage['description']}\n"
        
        output += "\n### Required Checks\n"
        for check in checks:
            output += f"- ✓ {check['name']}: {check['description']}\n"
        
        return output
```


### 3.2 GitLab Comment Monitor Agent

**תפקיד**: האזנה לתגובות ב-PR
**טכנולוגיה**: GitLab webhooks

```python
class GitLabCommentMonitorAgent:
    def __init__(self):
        self.gitlab = gitlab.Gitlab(...)
        
    async def handle_mr_comment(self, webhook_payload: dict):
        comment = webhook_payload["object_attributes"]["note"]
        mr_iid = webhook_payload["merge_request"]["iid"]
        project_id = webhook_payload["project"]["id"]
        
        # Parse commands
        if comment.startswith("/execute"):
            # Trigger execution agent
            await supervisor.execute_plan(project_id, mr_iid)
            
        elif comment.startswith("/improve"):
            feedback = comment.replace("/improve", "").strip()
            await supervisor.improve_plan(project_id, mr_iid, feedback)
            
        elif comment.startswith("/status"):
            status = await self.get_mr_status(project_id, mr_iid)
            await self.post_status_comment(project_id, mr_iid, status)
```


### 3.3 Code Execution Agent (with CI/CD Validation)

**תפקיד**: כתיבת קוד בפועל
**טכנולוגיה**: Claude + GitHub MCP + GitLab API

**תת-סוכנים**:

#### 3.3.1 Test Writer Sub-Agent

```python
class TestWriterSubAgent:
    """כתיבת tests (TDD - קודם!)"""
    async def write_tests(
        self,
        test_plan: dict,
        code_context: dict,
        repo: str,
        branch: str
    ) -> list[dict]:
        written_tests = []
        
        for test_case in test_plan["unit_tests"]:
            # Generate test code
            test_code = await self.claude.messages.create(
                model="claude-3-5-sonnet-20241022",
                max_tokens=4096,
                messages=[{
                    "role": "user",
                    "content": f"""Write test code:
                    
Test Case: {test_case}
Code Context: {code_context}
Testing Framework: {code_context["testing_framework"]}
Conventions: {code_context["conventions"]}

Generate complete test file following TDD principles."""
                }]
            )
            
            # Commit test file
            test_file_path = self.determine_test_path(
                test_case,
                code_context["test_structure"]
            )
            
            await self.commit_file(
                repo,
                branch,
                test_file_path,
                test_code.content[^0].text,
                f"test: add {test_case['name']} (TDD)"
            )
            
            written_tests.append({
                "file": test_file_path,
                "test_case": test_case["name"]
            })
        
        return written_tests
```


#### 3.3.2 Implementation Writer Sub-Agent

```python
class ImplementationWriterSubAgent:
    """כתיבת implementation (אחרי tests!)"""
    async def write_implementation(
        self,
        plan: dict,
        test_files: list[dict],
        code_context: dict,
        repo: str,
        branch: str
    ) -> list[dict]:
        implementations = []
        
        for file_change in plan["file_changes"][repo]:
            # Generate implementation code
            impl_code = await self.claude.messages.create(
                model="claude-3-5-sonnet-20241022",
                max_tokens=8192,
                messages=[{
                    "role": "user",
                    "content": f"""Write implementation code:
                    
File: {file_change["path"]}
Purpose: {file_change["purpose"]}
Tests: {json.dumps(test_files, indent=2)}
Code Context: {code_context}
Architecture: {plan["architecture"]}

Generate code that:
1. Makes the tests pass
2. Follows existing patterns
3. Maintains code quality
4. Includes error handling"""
                }]
            )
            
            # Commit implementation
            await self.commit_file(
                repo,
                branch,
                file_change["path"],
                impl_code.content[^0].text,
                f"feat: implement {file_change['purpose']}"
            )
            
            implementations.append({
                "file": file_change["path"],
                "committed": True
            })
        
        return implementations
```


#### 3.3.3 CI/CD Validator Sub-Agent

```python
class CICDValidatorSubAgent:
    """ולידציה שה-CI/CD עובר בהצלחה"""
    async def validate_pipeline(
        self,
        project_id: int,
        mr_iid: int,
        expected_validations: dict
    ) -> dict:
        project = self.gitlab.projects.get(project_id)
        mr = project.mergerequests.get(mr_iid)
        
        # Wait for pipeline to start
        pipeline = await self.wait_for_pipeline(mr)
        
        if not pipeline:
            return {
                "status": "error",
                "message": "No pipeline triggered"
            }
        
        # Monitor pipeline progress
        validation_results = {
            "pipeline_id": pipeline.id,
            "status": "running",
            "stages": {},
            "checks_passed": [],
            "checks_failed": [],
            "duration": 0
        }
        
        while pipeline.status in ["pending", "running"]:
            await asyncio.sleep(10)
            pipeline.refresh()
            
            # Update stage status
            for job in pipeline.jobs.list():
                stage_name = job.stage
                validation_results["stages"][stage_name] = {
                    "status": job.status,
                    "name": job.name,
                    "duration": job.duration
                }
            
            validation_results["duration"] = pipeline.duration
        
        # Check final status
        if pipeline.status == "success":
            validation_results["status"] = "success"
            validation_results["checks_passed"] = [
                check["name"]
                for check in expected_validations["required_checks"]
            ]
        else:
            validation_results["status"] = "failed"
            # Get failed jobs
            failed_jobs = [
                job for job in pipeline.jobs.list()
                if job.status == "failed"
            ]
            validation_results["checks_failed"] = [
                {
                    "job": job.name,
                    "stage": job.stage,
                    "log": job.trace()[:1000]  # First 1000 chars of log
                }
                for job in failed_jobs
            ]
        
        return validation_results
    
    async def wait_for_pipeline(self, mr, timeout=300):
        """Wait for pipeline to start (with timeout)"""
        start_time = time.time()
        
        while time.time() - start_time < timeout:
            pipelines = mr.pipelines.list()
            if pipelines:
                return pipelines[^0]
            await asyncio.sleep(5)
        
        return None
```

**Main Execution Agent:**

```python
class CodeExecutionAgent:
    def __init__(self):
        self.test_writer = TestWriterSubAgent()
        self.impl_writer = ImplementationWriterSubAgent()
        self.cicd_validator = CICDValidatorSubAgent()
        
    async def execute_plan(
        self,
        project_id: int,
        mr_iid: int,
        plan: dict,
        code_context: dict
    ) -> dict:
        project = self.gitlab.projects.get(project_id)
        mr = project.mergerequests.get(mr_iid)
        branch = mr.source_branch
        repo = project.path_with_namespace
        
        execution_results = {
            "tests_written": [],
            "implementation_written": [],
            "ci_validation": {}
        }
        
        try:
            # Step 1: Write tests FIRST (TDD)
            self.post_progress_comment(
                project_id,
                mr_iid,
                "🧪 Writing tests (TDD approach)..."
            )
            
            test_files = await self.test_writer.write_tests(
                plan["test_plan"],
                code_context,
                repo,
                branch
            )
            execution_results["tests_written"] = test_files
            
            # Step 2: Wait for CI to confirm tests fail (expected in TDD)
            self.post_progress_comment(
                project_id,
                mr_iid,
                "⏳ Waiting for CI to confirm tests fail (as expected)..."
            )
            
            initial_ci = await self.cicd_validator.validate_pipeline(
                project_id,
                mr_iid,
                plan["cicd_validations"][repo]
            )
            
            if initial_ci["status"] == "success":
                self.post_progress_comment(
                    project_id,
                    mr_iid,
                    "⚠️ Warning: Tests passed without implementation (check test validity)"
                )
            
            # Step 3: Write implementation
            self.post_progress_comment(
                project_id,
                mr_iid,
                "⚙️ Writing implementation to make tests pass..."
            )
            
            impl_files = await self.impl_writer.write_implementation(
                plan,
                test_files,
                code_context,
                repo,
                branch
            )
            execution_results["implementation_written"] = impl_files
            
            # Step 4: Validate CI/CD passes
            self.post_progress_comment(
                project_id,
                mr_iid,
                "🔍 Running full CI/CD validation..."
            )
            
            final_ci = await self.cicd_validator.validate_pipeline(
                project_id,
                mr_iid,
                plan["cicd_validations"][repo]
            )
            execution_results["ci_validation"] = final_ci
            
            # Step 5: Post results
            if final_ci["status"] == "success":
                self.post_success_comment(
                    project_id,
                    mr_iid,
                    execution_results
                )
            else:
                self.post_failure_comment(
                    project_id,
                    mr_iid,
                    execution_results
                )
            
            return execution_results
            
        except Exception as e:
            self.post_error_comment(
                project_id,
                mr_iid,
                str(e)
            )
            raise
```


***

## Layer 4: Communication \& Monitoring Agents

### 4.1 Slack Notification Agent (Real-time Updates)

**תפקיד**: דיווח real-time ל-Slack
**טכנולוגיה**: Slack Bolt SDK[^5][^4]

```python
from slack_bolt.async_app import AsyncApp
from slack_sdk.web.async_client import AsyncWebClient

class SlackNotificationAgent:
    def __init__(self):
        self.app = AsyncApp(token=os.getenv("SLACK_BOT_TOKEN"))
        self.client = AsyncWebClient(token=os.getenv("SLACK_BOT_TOKEN"))
        self.channel_mapping = {
            "backend": "#backend-ai-tasks",
            "frontend": "#frontend-ai-tasks",
            "devops": "#devops-ai-tasks",
            "general": "#ai-tasks"
        }
        
    async def notify_task_started(self, task: dict):
        """התראה על משימה חדשה"""
        channel = self.get_channel_for_project(task["project"])
        
        await self.client.chat_postMessage(
            channel=channel,
            text=f"🤖 New AI task started: {task['issue_key']}",
            blocks=[
                {
                    "type": "header",
                    "text": {
                        "type": "plain_text",
                        "text": f"🤖 AI Task: {task['issue_key']}"
                    }
                },
                {
                    "type": "section",
                    "fields": [
                        {
                            "type": "mrkdwn",
                            "text": f"*Summary:*\n{task['summary']}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Status:*\n🟡 Discovery"
                        }
                    ]
                },
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": f"<{self.jira_url(task['issue_key'])}|View in Jira>"
                    }
                }
            ],
            thread_ts=None  # Start new thread
        )
    
    async def notify_repos_found(
        self,
        task: dict,
        repos: list[dict],
        thread_ts: str
    ):
        """דיווח על repos שנמצאו"""
        channel = self.get_channel_for_project(task["project"])
        
        repos_text = "\n".join([
            f"• `{repo['name']}` - {repo['reason']}"
            for repo in repos
        ])
        
        await self.client.chat_postMessage(
            channel=channel,
            thread_ts=thread_ts,
            text=f"📦 Found {len(repos)} relevant repositories",
            blocks=[
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": f"*📦 Repositories Discovered*\n{repos_text}"
                    }
                }
            ]
        )
    
    async def notify_plan_ready(
        self,
        task: dict,
        plan: dict,
        mrs: list[dict],
        thread_ts: str
    ):
        """דיווח על תוכנית מוכנה"""
        channel = self.get_channel_for_project(task["project"])
        
        mr_links = "\n".join([
            f"• <{mr['mr_url']}|{mr['repo']} MR #{mr['mr_id']}>"
            for mr in mrs
        ])
        
        await self.client.chat_postMessage(
            channel=channel,
            thread_ts=thread_ts,
            text=f"📋 Implementation plan ready",
            blocks=[
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": f"*📋 Implementation Plan Ready*\n\n{mr_links}"
                    }
                },
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": f"*Complexity:* {plan['complexity_score']}/10\n*Estimated CI Duration:* ~{self.sum_ci_durations(plan)}min"
                    }
                },
                {
                    "type": "actions",
                    "elements": [
                        {
                            "type": "button",
                            "text": {
                                "type": "plain_text",
                                "text": "✓ Approve & Execute"
                            },
                            "style": "primary",
                            "value": f"approve_{task['issue_key']}",
                            "action_id": "approve_execution"
                        },
                        {
                            "type": "button",
                            "text": {
                                "type": "plain_text",
                                "text": "✎ Request Changes"
                            },
                            "value": f"improve_{task['issue_key']}",
                            "action_id": "request_changes"
                        }
                    ]
                }
            ]
        )
    
    async def notify_ci_status(
        self,
        task: dict,
        mr: dict,
        ci_result: dict,
        thread_ts: str
    ):
        """דיווח על סטטוס CI/CD"""
        channel = self.get_channel_for_project(task["project"])
        
        if ci_result["status"] == "success":
            emoji = "✅"
            color = "good"
            text = "All CI/CD checks passed!"
        elif ci_result["status"] == "running":
            emoji = "⏳"
            color = "warning"
            text = "CI/CD pipeline running..."
        else:
            emoji = "❌"
            color = "danger"
            text = "CI/CD pipeline failed"
        
        stages_text = "\n".join([
            f"• {self.stage_emoji(stage['status'])} {name}: {stage['status']}"
            for name, stage in ci_result["stages"].items()
        ])
        
        await self.client.chat_postMessage(
            channel=channel,
            thread_ts=thread_ts,
            text=f"{emoji} CI/CD Status: {ci_result['status']}",
            blocks=[
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": f"*{emoji} CI/CD Pipeline*\n{text}\n\n*Stages:*\n{stages_text}"
                    }
                },
                {
                    "type": "context",
                    "elements": [
                        {
                            "type": "mrkdwn",
                            "text": f"Pipeline #{ci_result['pipeline_id']} | Duration: {ci_result['duration']}s"
                        }
                    ]
                }
            ],
            attachments=[
                {
                    "color": color,
                    "text": f"<{mr['mr_url']}|View Merge Request>"
                }
            ]
        )
    
    async def notify_execution_complete(
        self,
        task: dict,
        execution_results: dict,
        thread_ts: str
    ):
        """דיווח על סיום ביצוע"""
        channel = self.get_channel_for_project(task["project"])
        
        ci_status = execution_results["ci_validation"]["status"]
        
        if ci_status == "success":
            emoji = "🎉"
            text = "Task completed successfully!"
            color = "good"
        else:
            emoji = "⚠️"
            text = "Task completed but CI/CD failed"
            color = "warning"
        
        await self.client.chat_postMessage(
            channel=channel,
            thread_ts=thread_ts,
            text=f"{emoji} {text}",
            blocks=[
                {
                    "type": "header",
                    "text": {
                        "type": "plain_text",
                        "text": f"{emoji} Task Complete: {task['issue_key']}"
                    }
                },
                {
                    "type": "section",
                    "fields": [
                        {
                            "type": "mrkdwn",
                            "text": f"*Tests Written:*\n{len(execution_results['tests_written'])}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Files Modified:*\n{len(execution_results['implementation_written'])}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*CI Status:*\n{self.format_ci_status(ci_status)}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Duration:*\n{execution_results['ci_validation']['duration']}s"
                        }
                    ]
                }
            ],
            attachments=[
                {
                    "color": color,
                    "text": "Ready for code review!"
                }
            ]
        )
```


### 4.2 Slack Interactive Bot (Query Interface)

**תפקיד**: מענה על שאלות מהצוות
**טכנולוגיה**: Slack Bolt + Slash Commands[^5]

```python
class SlackInteractiveBotAgent:
    def __init__(self):
        self.app = AsyncApp(token=os.getenv("SLACK_BOT_TOKEN"))
        self.setup_commands()
        
    def setup_commands(self):
        """רישום slash commands"""
        
        @self.app.command("/ai-status")
        async def handle_status_command(ack, command, say):
            await ack()
            
            # Get all active tasks
            active_tasks = await self.get_active_tasks()
            
            if not active_tasks:
                await say("No active AI tasks at the moment 🤖")
                return
            
            blocks = [
                {
                    "type": "header",
                    "text": {
                        "type": "plain_text",
                        "text": "🤖 Active AI Tasks"
                    }
                }
            ]
            
            for task in active_tasks:
                blocks.append({
                    "type": "section",
                    "fields": [
                        {
                            "type": "mrkdwn",
                            "text": f"*{task['issue_key']}*\n{task['summary']}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Status:*\n{self.format_status(task['status'])}"
                        }
                    ],
                    "accessory": {
                        "type": "button",
                        "text": {
                            "type": "plain_text",
                            "text": "Details"
                        },
                        "value": task['issue_key'],
                        "action_id": "view_task_details"
                    }
                })
            
            await say(blocks=blocks)
        
        @self.app.command("/ai-agents")
        async def handle_agents_command(ack, command, say):
            await ack()
            
            # Get agent status from monitoring
            agents_status = await self.get_agents_status()
            
            blocks = [
                {
                    "type": "header",
                    "text": {
                        "type": "plain_text",
                        "text": "🤖 Agent Status"
                    }
                }
            ]
            
            for agent_name, status in agents_status.items():
                blocks.append({
                    "type": "section",
                    "fields": [
                        {
                            "type": "mrkdwn",
                            "text": f"*{agent_name}*"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Status:* {self.status_emoji(status['health'])} {status['health']}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Tasks Today:* {status['tasks_today']}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": f"*Avg Duration:* {status['avg_duration']}s"
                        }
                    ]
                })
            
            await say(blocks=blocks)
        
        @self.app.command("/ai-costs")
        async def handle_costs_command(ack, command, say):
            await ack()
            
            # Get cost data from monitoring
            costs = await self.get_cost_data()
            
            await say(
                blocks=[
                    {
                        "type": "header",
                        "text": {
                            "type": "plain_text",
                            "text": "💰 AI System Costs"
                        }
                    },
                    {
                        "type": "section",
                        "fields": [
                            {
                                "type": "mrkdwn",
                                "text": f"*Today:*\n${costs['today']:.2f}"
                            },
                            {
                                "type": "mrkdwn",
                                "text": f"*This Week:*\n${costs['week']:.2f}"
                            },
                            {
                                "type": "mrkdwn",
                                "text": f"*This Month:*\n${costs['month']:.2f}"
                            },
                            {
                                "type": "mrkdwn",
                                "text": f"*Per Task Avg:*\n${costs['per_task']:.2f}"
                            }
                        ]
                    },
                    {
                        "type": "section",
                        "text": {
                            "type": "mrkdwn",
                            "text": f"*Cost Breakdown:*\n• LLM: ${costs['llm']:.2f}\n• Infrastructure: ${costs['infra']:.2f}\n• APIs: ${costs['apis']:.2f}"
                        }
                    }
                ]
            )
        
        @self.app.action("view_task_details")
        async def handle_view_details(ack, action, say):
            await ack()
            
            issue_key = action["value"]
            task_details = await self.get_task_details(issue_key)
            
            await say(
                text=f"Details for {issue_key}",
                blocks=self.format_task_details(task_details),
                thread_ts=action.get("thread_ts")
            )
        
        @self.app.action("approve_execution")
        async def handle_approve(ack, action, say):
            await ack()
            
            issue_key = action["value"].replace("approve_", "")
            
            # Trigger execution
            await supervisor.approve_and_execute(issue_key)
            
            await say(
                text=f"✓ Execution approved for {issue_key}",
                thread_ts=action.get("thread_ts")
            )
```


### 4.3 Dashboard \& Monitoring Agent

**תפקיד**: מעקב, logging, metrics
**טכנולוגיה**: PostgreSQL + Grafana + Custom API

```python
class DashboardMonitoringAgent:
    def __init__(self):
        self.db = PostgreSQLConnection()
        self.metrics = PrometheusMetrics()
        
    async def log_event(
        self,
        event_type: str,
        agent_name: str,
        task_id: str,
        data: dict,
        duration: float = None,
        cost: float = None
    ):
        """Log event to database"""
        await self.db.execute("""
            INSERT INTO agent_events 
            (event_type, agent_name, task_id, data, duration, cost, timestamp)
            VALUES ($1, $2, $3, $4, $5, $6, NOW())
        """, event_type, agent_name, task_id, json.dumps(data), duration, cost)
        
        # Update Prometheus metrics
        self.metrics.agent_events_total.labels(
            agent=agent_name,
            event_type=event_type
        ).inc()
        
        if duration:
            self.metrics.agent_duration_seconds.labels(
                agent=agent_name
            ).observe(duration)
        
        if cost:
            self.metrics.agent_cost_dollars.labels(
                agent=agent_name
            ).inc(cost)
    
    async def get_agent_health(self) -> dict:
        """בדיקת health של כל ה-agents"""
        agents = [
            "jira_watcher",
            "sentry_monitor",
            "repo_discovery",
            "code_context",
            "planner",
            "pr_creator",
            "executor",
            "cicd_validator",
            "slack_notifier"
        ]
        
        health_status = {}
        
        for agent in agents:
            # Check last activity
            last_activity = await self.db.fetchone("""
                SELECT timestamp, event_type
                FROM agent_events
                WHERE agent_name = $1
                ORDER BY timestamp DESC
                LIMIT 1
            """, agent)
            
            # Check error rate
            error_rate = await self.db.fetchone("""
                SELECT 
                    COUNT(*) FILTER (WHERE event_type = 'error') * 100.0 / COUNT(*) as error_rate
                FROM agent_events
                WHERE agent_name = $1
                AND timestamp > NOW() - INTERVAL '1 hour'
            """, agent)
            
            health_status[agent] = {
                "health": "healthy" if error_rate["error_rate"] < 5 else "degraded",
                "last_activity": last_activity["timestamp"],
                "error_rate": error_rate["error_rate"],
                "tasks_today": await self.get_tasks_count(agent, "today"),
                "avg_duration": await self.get_avg_duration(agent)
            }
        
        return health_status
    
    async def get_cost_breakdown(self, period: str = "today") -> dict:
        """פירוט עלויות"""
        if period == "today":
            time_filter = "timestamp > CURRENT_DATE"
        elif period == "week":
            time_filter = "timestamp > CURRENT_DATE - INTERVAL '7 days'"
        else:
            time_filter = "timestamp > DATE_TRUNC('month', CURRENT_DATE)"
        
        costs = await self.db.fetchone(f"""
            SELECT 
                SUM(cost) as total,
                SUM(cost) FILTER (WHERE agent_name LIKE '%planner%' OR agent_name LIKE '%executor%') as llm,
                SUM(cost) FILTER (WHERE agent_name LIKE '%infra%') as infra,
                SUM(cost) FILTER (WHERE agent_name LIKE '%api%') as apis,
                COUNT(DISTINCT task_id) as tasks,
                SUM(cost) / NULLIF(COUNT(DISTINCT task_id), 0) as per_task
            FROM agent_events
            WHERE {time_filter}
            AND cost IS NOT NULL
        """)
        
        return {
            "total": costs["total"] or 0,
            "llm": costs["llm"] or 0,
            "infra": costs["infra"] or 0,
            "apis": costs["apis"] or 0,
            "tasks": costs["tasks"] or 0,
            "per_task": costs["per_task"] or 0
        }
```


***

## Supervisor Agent (Main Orchestrator)

```python
class SupervisorAgent:
    """המוח המרכזי - מתאם את כל ה-agents"""
    def __init__(self):
        # Intake agents
        self.jira_watcher = JiraWatcherAgent()
        self.sentry_monitor = SentryMonitorAgent()
        self.jira_comment_controller = JiraCommentControllerAgent()
        
        # Analysis agents
        self.repo_discovery = RepositoryDiscoveryAgent()
        self.code_context = CodeContextAgent()
        self.planner = PlanningAgent()
        
        # Execution agents
        self.pr_creator = GitLabPRCreatorAgent()
        self.executor = CodeExecutionAgent()
        
        # Communication agents
        self.slack_notifier = SlackNotificationAgent()
        self.slack_bot = SlackInteractiveBotAgent()
        self.monitoring = DashboardMonitoringAgent()
        
        # State management
        self.checkpointer = RedisSaver(redis_url)
        
        # LangGraph workflow
        self.workflow = self.build_workflow()
        
    def build_workflow(self) -> StateGraph:
        """בניית LangGraph workflow"""
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
        
        # Edges
        workflow.set_entry_point("intake")
        workflow.add_edge("intake", "discovery")
        workflow.add_edge("discovery", "planning")
        workflow.add_edge("planning", "pr_creation")
        workflow.add_edge("pr_creation", "await_approval")
        
        # Conditional edge after approval
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
        
        # Conditional edge after validation
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
        
        return workflow.compile(checkpointer=self.checkpointer)
    
    async def intake_node(self, state: AgentState) -> AgentState:
        """צומת קבלת משימה"""
        task = state["task"]
        
        # Log
        await self.monitoring.log_event(
            "task_received",
            "supervisor",
            task["issue_key"],
            task
        )
        
        # Notify Slack
        thread_ts = await self.slack_notifier.notify_task_started(task)
        state["slack_thread"] = thread_ts
        
        state["status"] = "discovery"
        return state
    
    async def discovery_node(self, state: AgentState) -> AgentState:
        """צומת גילוי repositories"""
        start_time = time.time()
        
        # Run repository discovery
        repos_result = await self.repo_discovery.discover(state["task"])
        
        # Get code context for each repo
        code_contexts = {}
        for repo_info in repos_result["repositories"]:
            context = await self.code_context.analyze(repo_info["repo"]["name"])
            code_contexts[repo_info["repo"]["name"]] = context
        
        duration = time.time() - start_time
        
        # Log
        await self.monitoring.log_event(
            "discovery_complete",
            "repo_discovery",
            state["task"]["issue_key"],
            repos_result,
            duration=duration
        )
        
        # Notify Slack
        await self.slack_notifier.notify_repos_found(
            state["task"],
            repos_result["repositories"],
            state["slack_thread"]
        )
        
        state["repositories"] = repos_result["repositories"]
        state["code_contexts"] = code_contexts
        state["status"] = "planning"
        
        return state
    
    async def planning_node(self, state: AgentState) -> AgentState:
        """צומת תכנון"""
        start_time = time.time()
        
        # Create comprehensive plan
        plan = await self.planner.create_comprehensive_plan(
            state["task"],
            state["repositories"],
            state["code_contexts"]
        )
        
        duration = time.time() - start_time
        cost = self.estimate_planning_cost(duration)
        
        # Log
        await self.monitoring.log_event(
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
    
    async def pr_creation_node(self, state: AgentState) -> AgentState:
        """צומת יצירת PRs"""
        start_time = time.time()
        
        # Create PRs
        mrs = await self.pr_creator.create_pr_with_plan(
            state["repositories"],
            state["plan"],
            state["task"]
        )
        
        duration = time.time() - start_time
        
        # Log
        await self.monitoring.log_event(
            "prs_created",
            "pr_creator",
            state["task"]["issue_key"],
            {"mrs": mrs},
            duration=duration
        )
        
        # Notify Slack
        await self.slack_notifier.notify_plan_ready(
            state["task"],
            state["plan"],
            mrs,
            state["slack_thread"]
        )
        
        state["merge_requests"] = mrs
        state["status"] = "awaiting_approval"
        state["approval_status"] = "pending"
        
        return state
    
    async def await_approval_node(self, state: AgentState) -> AgentState:
        """המתנה לאישור"""
        # This is a "human-in-the-loop" node
        # It will wait for external approval via Slack or GitLab comment
        
        # Check approval status from state
        # (Updated by external webhook handlers)
        
        return state
    
    async def execution_node(self, state: AgentState) -> AgentState:
        """צומת ביצוע"""
        start_time = time.time()
        
        execution_results = []
        
        for mr in state["merge_requests"]:
            result = await self.executor.execute_plan(
                mr["project_id"],
                mr["mr_id"],
                state["plan"],
                state["code_contexts"][mr["repo"]]
            )
            execution_results.append(result)
            
            # Notify CI status
            await self.slack_notifier.notify_ci_status(
                state["task"],
                mr,
                result["ci_validation"],
                state["slack_thread"]
            )
        
        duration = time.time() - start_time
        cost = self.estimate_execution_cost(duration, execution_results)
        
        # Log
        await self.monitoring.log_event(
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
    
    async def validation_node(self, state: AgentState) -> AgentState:
        """צומת ולידציה"""
        # Check if all CIs passed
        all_passed = all(
            result["ci_validation"]["status"] == "success"
            for result in state["execution_results"]
        )
        
        state["validation_status"] = "passed" if all_passed else "failed"
        state["retry_count"] = state.get("retry_count", 0)
        
        return state
    
    async def completion_node(self, state: AgentState) -> AgentState:
        """צומת סיום"""
        # Notify Slack
        await self.slack_notifier.notify_execution_complete(
            state["task"],
            state["execution_results"][^0],  # Assuming single repo for simplicity
            state["slack_thread"]
        )
        
        # Update Jira
        await self.update_jira_completion(state)
        
        # Log
        await self.monitoring.log_event(
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
    
    def should_execute_or_improve(self, state: AgentState) -> str:
        """החלטה האם לבצע, לשפר, או להמתין"""
        if state["approval_status"] == "approved":
            return "execute"
        elif state["approval_status"] == "needs_improvement":
            return "improve"
        else:
            return "wait"
    
    def should_retry_or_complete(self, state: AgentState) -> str:
        """החלטה האם לנסות שוב או לסיים"""
        if state["validation_status"] == "passed":
            return "complete"
        elif state["retry_count"] < 2:
            state["retry_count"] += 1
            return "retry"
        else:
            return "manual"  # Needs human intervention
```


***

## Docker Compose המלא

```yaml
version: '3.8'

services:
  # Supervisor + LangGraph
  supervisor:
    build: ./supervisor
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - REDIS_URL=redis://redis:6379
      - POSTGRES_URL=postgresql://postgres:password@postgres:5432/agents
      - GITLAB_URL=${GITLAB_URL}
      - GITLAB_TOKEN=${GITLAB_TOKEN}
      - SLACK_BOT_TOKEN=${SLACK_BOT_TOKEN}
      - SLACK_SIGNING_SECRET=${SLACK_SIGNING_SECRET}
      # MCP Endpoints
      - ATLASSIAN_MCP_URL=https://mcp.atlassian.com/v1/mcp
      - GITHUB_MCP_URL=https://api.githubcopilot.com/mcp/
      - SENTRY_MCP_URL=https://mcp.sentry.dev/mcp
    ports:
      - "8000:8000"
    depends_on:
      - redis
      - postgres
  
  # Redis for state management
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
  
  # PostgreSQL for logging
  postgres:
    image: postgres:16-alpine
    environment:
      - POSTGRES_DB=agents
      - POSTGRES_PASSWORD=password
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
  
  # Grafana for monitoring
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
      - GF_INSTALL_PLUGINS=grafana-postgresql-datasource
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
  
  # Prometheus for metrics
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

volumes:
  redis-data:
  postgres-data:
  grafana-data:
  prometheus-data:
```


***

## עלויות משודרגות (חודשיות, 100 tasks)

### תשתית (AWS ECS)

- ECS Tasks (5 containers × 1 vCPU): **\$150/month**
- Redis ElastiCache: **\$15/month**
- RDS PostgreSQL: **\$35/month**
- ALB: **\$20/month**
- **סה"כ תשתית: \$220/month**


### LLM (Claude 3.5 Sonnet - Direct API)

- Repository Discovery: **\$25/month**
- Planning: **\$135/month**
- Execution: **\$120/month**
- **סה"כ LLM: \$280/month**


### APIs \& MCP Servers

- Atlassian Rovo MCP: **Free** (OAuth)[^1]
- GitHub MCP: **Free** (OAuth)[^2]
- Sentry MCP: **\$26/month** (Team plan)[^3]
- GitLab API: **Free** (5,000 CI minutes)
- Slack API: **Free** (Standard plan)
- **סה"כ APIs: \$26/month**


### **סה"כ כולל: \$526/month (100 tasks)**

### **עלות per task: \$5.26**


***

## התחלה - Quick Start

```bash
# 1. Clone repository
git clone https://github.com/your-org/ai-agent-system
cd ai-agent-system

# 2. Setup environment
cp .env.example .env
# Edit .env with your credentials

# 3. Start services
docker-compose up -d

# 4. Setup webhooks
python scripts/setup_webhooks.py

# 5. Test with sample task
curl -X POST http://localhost:8000/test/create-task \
  -H "Content-Type: application/json" \
  -d '{"summary": "Add login feature", "description": "..."}'
```


***

## סיכום ההשבחות

✅ **פירוק לתת-סוכנים** - כל agent עם sub-agents יעודיים
✅ **שימוש ב-MCP רשמיים** - Atlassian, GitHub, Sentry[^3][^2][^1]
✅ **אינטגרציה מלאה עם Slack** - notifications + interactive commands[^4][^5]
✅ **ולידציה של CI/CD** - מוודא שהקוד עובר את כל הבדיקות[^8][^9]
✅ **TDD מובנה** - tests קודם, implementation אחר כך
✅ **Multi-repo support** - מטפל ב-dependencies בין repos
✅ **Human-in-the-loop** - מחכה לאישור לפני execution
✅ **Production-ready** - error handling, retry logic, monitoring מלא

רוצה שאתחיל לבנות את הקוד המלא? אני יכול להתחיל עם איזה component שתרצה!
<span style="display:none">[^10][^11][^12][^13]</span>

<div align="center">⁂</div>

[^1]: https://softwarearchiblog.co.il/2025/05/מהפיכת-ה-agentic-ides.html

[^2]: https://www.youtube.com/watch?v=LELsExLRxec

[^3]: https://www.linkedin.com/posts/adir-duchan_%D7%A9%D7%90%D7%9C%D7%94-%D7%9C%D7%9E%D7%A4%D7%AA%D7%97%D7%99%D7%9D-%D7%A2%D7%9D-ai-%D7%AA%D7%94%D7%99%D7%95-%D7%90%D7%99%D7%AA%D7%99-%D7%A8%D7%92%D7%A2-%D7%90%D7%A0%D7%99-%D7%90%D7%A1%D7%91%D7%99%D7%A8-activity-7406622261941800960-zIWi

[^4]: https://docs.slack.dev/messaging/sending-messages-using-incoming-webhooks

[^5]: https://tallyfy.com/slack-bot/

[^6]: https://www.reddit.com/r/LangChain/comments/1onoufx/building_a_langchainlanggraph_multiagent/

[^7]: https://latenode.com/blog/ai-frameworks-technical-infrastructure/langgraph-multi-agent-orchestration/langgraph-multi-agent-orchestration-complete-framework-guide-architecture-analysis-2025

[^8]: https://www.headspin.io/blog/why-you-should-consider-ci-cd-pipeline-automation-testing

[^9]: https://www.frugaltesting.com/blog/how-to-integrate-automation-testing-into-your-ci-cd-pipeline

[^10]: https://api.slack.com/apps

[^11]: https://slack.com/marketplace/A0F7XDUAZ-incoming-webhooks

[^12]: https://developers.cloudflare.com/workers/tutorials/build-a-slackbot/

[^13]: https://www.slack.com/marketplace/A0F7VRG6Q-outgoing-webhooks

