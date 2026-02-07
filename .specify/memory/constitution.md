<!-- Sync Impact Report:
Version change: 1.0.0 → 2.0.0
Modified principles:
  - Spec-First Development: Enhanced with Agentic Dev Stack workflow requirement
  - Security-by-Design: Expanded with Better Auth integration and stateless auth patterns
  - API-First Architecture: Updated to include MCP tools and OpenAI Agents SDK integration
  - Separation of Concerns: Enhanced with stateless design requirements
Added sections:
  - Agentic Development Workflow (replaces manual Deterministic Reproducibility)
  - Stateless Architecture (new principle)
  - MCP Tool Integration (new principle)
  - AI Agent Integration (new principle)
  - Natural Language Interface (new principle)
Removed sections:
  - Generic "Deterministic Reproducibility" (replaced with specific Phase 3 requirements)
Templates requiring updates:
  ✅ .specify/templates/plan-template.md - Constitution Check section aligned
  ✅ .specify/templates/spec-template.md - Success criteria aligned with stateless patterns
  ✅ .specify/templates/tasks-template.md - Task categorization includes agent/MCP phases
Follow-up TODOs: None
-->

# Hackathon II Phase 3: Todo AI Chatbot Constitution

## Project Identity

**Project Name**: Todo AI Chatbot (Phase 3)
**Version**: 2.0.0
**Ratified**: 2026-02-07
**Last Amended**: 2026-02-07
**Scope**: AI-powered conversational todo management system with natural language interface

## Core Principles

### 1. Spec-First Development

**Rule**: No implementation before specification. All features must originate from a written spec before implementation begins.

**Requirements**:
- Follow the Agentic Dev Stack workflow: Spec → Plan → Tasks → Implement
- All system behaviors must be explicitly defined with no assumptions
- Zero manual coding allowed - all code generated via Claude Code only
- Every feature must have a corresponding spec document in `specs/[###-feature-name]/`
- All business logic must be documented before implementation

**Rationale**: Prevents scope creep, ensures alignment with requirements, creates documentation trail for hackathon evaluation, and demonstrates process transparency.

---

### 2. Agentic Development Workflow

**Rule**: All development must strictly follow the four-phase Agentic Dev Stack workflow with no exceptions.

**Requirements**:
- **Phase 1: Specification** - Use `/sp.specify` to generate `spec.md`
- **Phase 2: Planning** - Use `/sp.plan` to generate `plan.md` and research artifacts
- **Phase 3: Task Breakdown** - Use `/sp.tasks` to generate `tasks.md` with dependency ordering
- **Phase 4: Implementation** - Use `/sp.implement` to execute tasks via Claude Code
- No manual coding at any stage
- All prompts and iterations must be recorded in Prompt History Records (PHRs)
- Constitutional compliance check required before moving between phases

**Rationale**: Ensures reproducibility, demonstrates systematic approach for hackathon judges, maintains quality through iterative review, and eliminates ad-hoc development patterns.

---

### 3. Stateless Architecture

**Rule**: All server components (FastAPI, MCP Server, OpenAI Agent) MUST be stateless. No in-memory state allowed.

**Requirements**:
- **No in-memory storage**: No global variables, no session caches, no singleton state
- **Database persistence**: All state (tasks, conversations, messages, tool calls) stored in PostgreSQL
- **Fresh instantiation**: OpenAI Agent must be instantiated fresh on each request
- **History retrieval**: Full conversation history fetched from database on every chat request
- **Stateless MCP tools**: Each MCP tool call operates independently with no shared state
- **Session management**: Use Better Auth for authentication tokens, not server-side sessions

**Rationale**: Enables horizontal scaling, simplifies deployment, prevents race conditions, aligns with serverless/cloud-native patterns, and ensures reliability in multi-user scenarios.

---

### 4. MCP Tool Integration

**Rule**: All AI agent capabilities must be exposed through MCP (Model Context Protocol) tools using the Official MCP Python SDK.

**Requirements**:
- **Required MCP Tools**:
  - `create_task(user_id, title, description?, due_date?, priority?)` - Add new task
  - `list_tasks(user_id, status?, limit?, offset?)` - Retrieve tasks with filters
  - `update_task(user_id, task_id, title?, description?, status?, due_date?, priority?)` - Modify task
  - `delete_task(user_id, task_id)` - Remove task
  - `complete_task(user_id, task_id)` - Shortcut for marking complete
- **Tool Implementation**: Use Official MCP Python SDK server patterns
- **Database Access**: All tools must use SQLModel for database operations
- **Stateless Design**: Each tool call is independent with no shared context
- **Error Handling**: Return structured error objects for all failure cases
- **Parameter Validation**: Strict validation of all inputs before database operations

**Rationale**: Standardizes AI-to-system interface, ensures tools are reusable across agents, provides clear contract boundaries, and enables tool testing independent of agent logic.

---

### 5. AI Agent Integration

**Rule**: Natural language processing must be handled exclusively through OpenAI Agents SDK with MCP tool orchestration.

**Requirements**:
- **Agent Framework**: OpenAI Agents SDK only (no custom LLM integrations)
- **Tool Orchestration**: Agent must call MCP tools via MCP client
- **Multi-step Reasoning**: Agent may chain tool calls (e.g., `list_tasks` before `delete_task` for name resolution)
- **Stateless Execution**: Fresh agent instance on every chat request
- **System Prompt**: Define agent behavior, available tools, and response patterns in system prompt
- **Tool Call Logging**: All tool calls and responses must be persisted to database
- **Error Recovery**: Agent must gracefully handle tool errors and provide helpful user feedback

**Rationale**: Leverages proven agent framework, separates AI logic from business logic, enables natural language understanding without custom NLP, and maintains flexibility to swap agents.

---

### 6. Natural Language Interface

**Rule**: Users interact with the system exclusively through natural language commands processed by the AI agent.

**Requirements**:
- **Intent Detection**: Agent detects user intent from natural language (e.g., "add buy milk" → `create_task`)
- **Entity Extraction**: Agent extracts task details from user input (title, priority, due date)
- **Ambiguity Resolution**: Agent asks clarifying questions when input is unclear
- **Confirmation Messages**: All actions confirmed in friendly natural language
  - Example: "✓ Task added successfully: Buy groceries"
  - Example: "Marked task #3 'Call mom' as complete!"
- **Error Communication**: Errors explained in plain language with suggested fixes
  - Example: "I couldn't find task #5. Try typing 'show my tasks' to see your list."
- **Conversation Context**: Agent maintains conversation flow across multiple messages

**Rationale**: Provides intuitive user experience, reduces learning curve, demonstrates AI capabilities for hackathon evaluation, and aligns with modern conversational UI patterns.

---

### 7. Security-by-Design

**Rule**: Authentication and authorization enforced at every boundary with zero-trust principles.

**Requirements**:
- **Authentication Framework**: Better Auth only (JWT-based sessions)
- **Token Validation**: Every API request (except `/auth/login`, `/auth/signup`) requires valid JWT
- **JWT Verification**:
  - Signature verified on backend
  - Expiration checked
  - User ID extracted from token payload
- **Authorization**: User ID from JWT must match URL path `user_id` parameter
- **Data Isolation**: Users can only access their own tasks and conversations
- **Cross-User Access Prevention**: 403 Forbidden for attempts to access other users' data
- **Error Security**: Never reveal existence of other users' resources (404, not 403, for inaccessible items)
- **No Hardcoded Secrets**: All secrets via environment variables
- **Database Security**: Use parameterized queries (SQLModel ORM) to prevent SQL injection

**Rationale**: Protects user data, prevents unauthorized access, meets security standards for production deployment, and demonstrates security-first approach for hackathon evaluation.

---

### 8. API-First Architecture

**Rule**: All client-server communication happens through well-defined RESTful API endpoints with proper HTTP semantics.

**Requirements**:
- **Framework**: FastAPI only
- **Endpoint Structure**: RESTful routes with resource-based naming
  - `POST /api/{user_id}/tasks` - Create task
  - `GET /api/{user_id}/tasks` - List tasks
  - `PUT /api/{user_id}/tasks/{task_id}` - Update task
  - `DELETE /api/{user_id}/tasks/{task_id}` - Delete task
  - `POST /api/{user_id}/chat` - Chat with AI agent
- **HTTP Status Codes**:
  - 200 OK - Successful read
  - 201 Created - Resource created
  - 204 No Content - Successful delete
  - 400 Bad Request - Validation error
  - 401 Unauthorized - Missing/invalid JWT
  - 403 Forbidden - Valid JWT but insufficient permissions
  - 404 Not Found - Resource doesn't exist or user can't access
  - 500 Internal Server Error - Unexpected server error
- **Request/Response Format**: JSON only
- **Validation**: Pydantic v2+ models for all request/response schemas
- **Error Structure**: Consistent error response format with error code and message
- **Documentation**: Auto-generated OpenAPI/Swagger docs

**Rationale**: Ensures frontend-backend contract clarity, enables independent development/testing, provides standard interface for tool integration, and simplifies debugging.

---

### 9. Separation of Concerns

**Rule**: Clear boundaries between frontend, backend, authentication, database, MCP tools, and AI agent.

**Requirements**:
- **Frontend (OpenAI ChatKit)**:
  - User interface only
  - Makes API calls to backend
  - Never directly accesses database
  - Passes JWT token with every request
- **Backend (FastAPI)**:
  - API endpoints and business logic
  - Authentication validation
  - Database operations via SQLModel
  - Agent orchestration
  - Stateless (no session storage)
- **MCP Server**:
  - Tool implementations only
  - Database CRUD via SQLModel
  - No business logic (delegated to agent)
  - Stateless tool execution
- **AI Agent (OpenAI Agents SDK)**:
  - Natural language understanding
  - Intent detection and entity extraction
  - Tool orchestration (calls MCP tools)
  - Response generation
  - Fresh instantiation per request
- **Database (Neon PostgreSQL)**:
  - Persistent storage only
  - Accessed only via SQLModel ORM
  - Schema managed via migrations

**Rationale**: Prevents tight coupling, enables independent testing, allows component replacement, simplifies debugging, and demonstrates architectural maturity for hackathon evaluation.

---

### 10. Traceability and Process Transparency

**Rule**: Every feature, decision, and implementation step must be documented and traceable from requirement to code.

**Requirements**:
- **Prompt History Records (PHRs)**: Every user input and Claude response recorded
- **Specification Documents**: Every feature has `spec.md` with requirements and acceptance criteria
- **Plan Documents**: Every feature has `plan.md` with technical approach and design decisions
- **Task Documents**: Every feature has `tasks.md` with ordered, testable implementation tasks
- **Architecture Decision Records (ADRs)**: Significant decisions documented with context, options, and rationale
- **Commit Messages**: Descriptive messages linking commits to tasks/specs
- **README Documentation**: Complete setup instructions, architecture diagrams, and API documentation

**Rationale**: Critical for hackathon evaluation (judges assess process), enables reproducibility, facilitates onboarding, and creates audit trail for troubleshooting.

---

## Technology Stack Constraints

### Mandatory Stack (Non-Negotiable)

**Backend**:
- Language: Python 3.11+
- Framework: FastAPI only
- ORM: SQLModel only
- Database: Neon Serverless PostgreSQL only

**Frontend**:
- Framework: OpenAI ChatKit (React-based) only
- UI Library: OpenAI ChatKit components only

**AI Integration**:
- Agent Framework: OpenAI Agents SDK only
- Tool Protocol: Official MCP Python SDK only
- LLM Provider: OpenAI API (via Agents SDK)

**Authentication**:
- Framework: Better Auth only
- Mechanism: JWT tokens (no server-side sessions)

**Development Tools**:
- Code Generation: Claude Code only (no manual coding)
- Workflow: Spec-Kit Plus commands (`/sp.specify`, `/sp.plan`, `/sp.tasks`, `/sp.implement`)

### Prohibited Technologies

- ❌ Manual code editing (must use Claude Code)
- ❌ Alternative ORMs (SQLAlchemy, Django ORM, etc.)
- ❌ Alternative databases (MongoDB, MySQL, SQLite, etc.)
- ❌ Alternative agent frameworks (LangChain, AutoGPT, custom agents)
- ❌ Alternative MCP implementations (custom protocols)
- ❌ Server-side session storage (Redis, Memcached, in-memory sessions)
- ❌ Monolithic architecture (all components must be separable)
- ❌ Hardcoded credentials or secrets

**Rationale**: Stack constraints ensure consistency, simplify integration, meet hackathon requirements, and prevent scope creep from technology exploration.

---

## Development Workflow

### Phase 1: Specification

**Entry Condition**: User describes feature in natural language

**Process**:
1. Run `/sp.specify <feature description>`
2. Claude generates `specs/[###-feature-name]/spec.md`
3. Specification includes:
   - User scenarios with acceptance criteria
   - Functional requirements
   - Success criteria (measurable, technology-agnostic)
   - Edge cases
4. Review and clarify with `/sp.clarify` if needed
5. Commit spec to repository

**Exit Condition**: Complete spec with no unresolved `[NEEDS CLARIFICATION]` markers

**Constitution Check**: Spec must not include implementation details (no tech stack mentions)

---

### Phase 2: Planning

**Entry Condition**: Approved specification exists

**Process**:
1. Run `/sp.plan`
2. Claude generates:
   - `specs/[###-feature-name]/plan.md` - Technical approach
   - `specs/[###-feature-name]/research.md` - Technology research
   - `specs/[###-feature-name]/data-model.md` - Database entities
   - `specs/[###-feature-name]/contracts/` - API endpoint contracts
3. Plan includes:
   - Technology choices (must match constitution stack)
   - Architecture decisions
   - Database schema
   - API endpoint definitions
   - MCP tool definitions (if applicable)
   - Agent behavior design (if applicable)
4. Review for constitutional compliance
5. Suggest ADRs for significant decisions

**Exit Condition**: Complete plan with all design artifacts, constitutional compliance verified

**Constitution Check**:
- Stack matches mandatory technologies
- Stateless design confirmed
- Security patterns included
- No prohibited technologies used

---

### Phase 3: Task Breakdown

**Entry Condition**: Approved plan exists

**Process**:
1. Run `/sp.tasks`
2. Claude generates `specs/[###-feature-name]/tasks.md`
3. Tasks include:
   - Sequential task IDs (T001, T002, ...)
   - Parallel markers `[P]` for independent tasks
   - User story markers `[US1]`, `[US2]` for traceability
   - Exact file paths for each task
   - Dependency ordering
4. Task organization:
   - Phase 1: Setup (project structure)
   - Phase 2: Foundational (blocking prerequisites)
   - Phase 3+: User stories (independent, testable increments)
   - Final Phase: Polish (cross-cutting concerns)
5. Each task must be testable and atomic

**Exit Condition**: Complete task list with dependencies resolved, ready for implementation

**Constitution Check**:
- All tasks reference real file paths
- Dependencies prevent conflicts
- Each user story independently implementable

---

### Phase 4: Implementation

**Entry Condition**: Approved task list exists

**Process**:
1. Run `/sp.implement`
2. Claude executes tasks in order:
   - Reads task description
   - Generates code using Claude Code tools
   - Tests code (if tests exist)
   - Commits with descriptive message
   - Moves to next task
3. For each task:
   - Read relevant specs/plans
   - Generate code following constitutional principles
   - Validate against acceptance criteria
   - Handle errors and iterate if needed
4. Create PHR for implementation session
5. Suggest `/sp.git.commit_pr` when complete

**Exit Condition**: All tasks completed, tests passing (if applicable), feature ready for review

**Constitution Check**:
- No manual code edits
- All generated code follows stack constraints
- Security patterns implemented
- Stateless design maintained

---

### Phase 5: Review and Documentation

**Entry Condition**: Implementation complete

**Process**:
1. Run `/sp.analyze` to validate cross-artifact consistency
2. Update README with new feature documentation
3. Create ADRs for significant decisions (if not already done)
4. Generate pull request with `/sp.git.commit_pr`
5. Link PR to spec, tasks, and PHRs

**Exit Condition**: Feature documented, PR created, ready for merge

---

## Stateless Design Patterns

### Chat Endpoint Flow (Critical Pattern)

**Endpoint**: `POST /api/{user_id}/chat`

**Stateless Implementation**:
1. **Validate Auth**: Extract `user_id` from JWT, verify matches URL parameter
2. **Fetch Conversation**: Load or create conversation from database
3. **Retrieve History**: Query all messages for conversation from database
4. **Save User Message**: Insert user's message into database
5. **Instantiate Agent**: Create fresh OpenAI Agent with MCP tools (no reuse)
6. **Execute Agent**: Run agent with full history + new message
7. **Save Agent Response**: Insert agent's message into database
8. **Persist Tool Calls**: Log all MCP tool calls and results to database
9. **Return Response**: Send agent response + tool calls to client
10. **Cleanup**: Agent instance discarded after request

**Anti-Patterns (Prohibited)**:
- ❌ Caching agent instance between requests
- ❌ Storing conversation history in memory
- ❌ Using global variables for state
- ❌ Relying on previous request context

---

### MCP Tool Call Pattern

**Stateless Tool Implementation**:
1. **Receive Parameters**: Get `user_id` and task parameters from agent
2. **Open Database Session**: Create fresh SQLModel session
3. **Execute Operation**: Perform CRUD operation (create, read, update, delete)
4. **Commit Transaction**: Save changes to database
5. **Return Result**: Send structured response to agent
6. **Close Session**: Clean up database connection

**Anti-Patterns (Prohibited)**:
- ❌ Caching database sessions
- ❌ Storing query results in global variables
- ❌ Sharing state between tool calls
- ❌ Maintaining open connections between requests

---

## Error Handling Standards

### User-Facing Errors (Agent Responses)

**Pattern**: Friendly, actionable natural language

**Examples**:
- ✅ "I couldn't find task #5. Try typing 'show my tasks' to see your current list."
- ✅ "I need more details. What would you like to name this task?"
- ✅ "Task 'Buy groceries' was successfully marked as complete!"
- ❌ "Task not found" (too terse)
- ❌ "Error: TASK_NOT_FOUND_404" (too technical)

---

### API Errors (HTTP Responses)

**Pattern**: Structured JSON with error code and message

**400 Bad Request**:
```json
{
  "error": "validation_error",
  "message": "Title is required and must be 1-200 characters",
  "details": {"field": "title", "constraint": "min_length"}
}
```

**401 Unauthorized**:
```json
{
  "error": "unauthorized",
  "message": "Missing or invalid JWT token"
}
```

**403 Forbidden**:
```json
{
  "error": "forbidden",
  "message": "You don't have permission to access this resource"
}
```

**404 Not Found**:
```json
{
  "error": "not_found",
  "message": "Task with id 123 not found"
}
```

**500 Internal Server Error**:
```json
{
  "error": "internal_error",
  "message": "An unexpected error occurred. Please try again."
}
```

---

### MCP Tool Errors

**Pattern**: Structured dict return values

**Examples**:
```python
# Success
{"success": True, "task_id": 123, "title": "Buy groceries"}

# Not Found
{"success": False, "error": "not_found", "message": "Task ID 123 does not exist"}

# Validation Error
{"success": False, "error": "validation_error", "message": "Title cannot be empty"}

# Database Error
{"success": False, "error": "internal_error", "message": "Could not complete operation"}
```

---

## Validation and Compliance

### Pre-Implementation Checklist

Before starting any implementation:
- [ ] Specification exists in `specs/[###-feature-name]/spec.md`
- [ ] Plan exists in `specs/[###-feature-name]/plan.md`
- [ ] Tasks exist in `specs/[###-feature-name]/tasks.md`
- [ ] All `[NEEDS CLARIFICATION]` markers resolved
- [ ] Technology stack matches constitution constraints
- [ ] Stateless design confirmed in plan
- [ ] Security patterns documented
- [ ] No prohibited technologies listed

### Post-Implementation Checklist

After completing implementation:
- [ ] All tasks in `tasks.md` marked complete
- [ ] PHR created for implementation session
- [ ] README updated with new feature documentation
- [ ] API endpoints follow RESTful conventions
- [ ] HTTP status codes used correctly
- [ ] Pydantic v2 models for all schemas
- [ ] SQLModel used for all database operations
- [ ] JWT validation on all authenticated endpoints
- [ ] User data isolation enforced
- [ ] Stateless design maintained (no global state)
- [ ] Agent instantiated fresh on each request
- [ ] Conversation history fetched from database
- [ ] All tool calls logged to database
- [ ] Error messages are user-friendly
- [ ] Code generated via Claude Code only (no manual edits)

### ADR Decision Criteria

Create an ADR when decision meets ALL three criteria:
1. **Impact**: Long-term consequences (framework choice, data model, API design, security pattern)
2. **Alternatives**: Multiple viable options considered
3. **Scope**: Cross-cutting, influences system design

**Examples requiring ADRs**:
- ✅ Choice of MCP SDK over custom protocol
- ✅ Stateless vs stateful architecture decision
- ✅ JWT-based auth vs session-based auth
- ✅ Agent instantiation strategy (fresh vs pooled)
- ❌ Variable naming convention (too local)
- ❌ Log level choice (reversible, low impact)

---

## Governance

### Constitutional Authority

This constitution governs all development for Hackathon II Phase 3: Todo AI Chatbot. All implementation must comply with these principles. Any deviation requires explicit constitutional amendment.

### Amendment Process

1. Identify constitutional conflict or missing principle
2. Propose amendment with rationale
3. Run `/sp.constitution` with amendment description
4. Review sync impact report
5. Update dependent templates if needed
6. Commit with version bump and changelog

**Version Bump Rules**:
- **MAJOR** (X.0.0): Backward-incompatible changes, principle removal/redefinition
- **MINOR** (x.Y.0): New principle added, materially expanded guidance
- **PATCH** (x.y.Z): Clarifications, wording fixes, non-semantic refinements

### Compliance Review

**Before Each Phase Transition**:
- Verify all constitutional requirements met
- Check technology stack constraints
- Validate stateless design patterns
- Confirm security measures implemented
- Review traceability artifacts (specs, plans, tasks, PHRs)

**Enforcement**:
- Code reviews must verify constitutional compliance
- Pull requests must link to specs and tasks
- Automated checks for prohibited patterns (where possible)
- Manual review for architecture alignment

### Hackathon Evaluation Alignment

This constitution is designed to maximize hackathon scores:

**Process Transparency** (Critical):
- All PHRs demonstrate iterative development
- Specs, plans, tasks show systematic approach
- ADRs document architectural decisions
- README shows complete setup and usage

**Technical Quality**:
- Stateless architecture demonstrates scalability
- Security-first approach shows production-readiness
- MCP integration shows modern tooling
- AI agent integration demonstrates innovation

**Reproducibility**:
- Agentic workflow ensures repeatability
- No manual coding eliminates human variance
- Comprehensive documentation enables recreation

---

**Version**: 2.0.0
**Ratified**: 2026-02-07
**Last Amended**: 2026-02-07
**Next Review**: Before Phase 4 (if project continues beyond Phase 3)
