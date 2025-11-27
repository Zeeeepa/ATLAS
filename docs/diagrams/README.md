# TandemAI Visual Documentation

This directory contains comprehensive visual diagrams documenting the ATLAS/TandemAI system architecture, APIs, and data flows.

## 📊 Available Diagrams

### 1. [Agent Architecture](./agent_architecture.md)
Complete multi-agent system design with:
- **Multi-Agent System Overview** - High-level agent hierarchy and relationships
- **Agent Hierarchy Detailed View** - Research workflow from query to response
- **State Persistence Architecture** - PostgreSQL checkpointing and thread management
- **Real-Time Communication Flow** - WebSocket event sequences
- **Evaluation Framework Architecture** - Judge agents and scoring system

**Key Diagrams**:
- 🤖 Multi-agent orchestration (Main Agent → Planning Agent → 5 Subagents)
- 💾 State persistence with PostgreSQL
- 📡 WebSocket real-time updates
- 🎯 Evaluation with 7 specialized judge agents

---

### 2. [API Endpoints](./api_endpoints.md)
Complete REST and WebSocket API reference with:
- **API Endpoint Overview** - All 18 REST + 2 WebSocket endpoints
- **Detailed API Specifications** - Request/response formats for every endpoint
- **Request/Response Flow Diagrams** - Sequence diagrams for key operations

**API Categories**:
- 🔍 **Research & Chat APIs** - `/api/chat`, `/api/research/plan/*`, `/api/conversation/*`
- 📁 **Workspace File APIs** - `/api/workspace/*` (tree, read, write, create)
- 💬 **Thread Management APIs** - `/api/threads/*` (create, list, update, delete)
- 🔐 **Auth & Approval APIs** - `/api/auth/token`, `/api/approval/decision`
- 🔌 **WebSocket Endpoints** - `/ws/plan`, `/ws/workspace/:path`
- ❤️ **Health & Monitoring** - `/health`

**Interactive Examples**: Request/response JSON with validation rules

---

### 3. [Data Flow](./data_flow.md)
End-to-end data movement through the system:
- **System-Wide Data Flow Overview** - Input → Processing → Storage → Output
- **Research Query Data Flow** - Complete query lifecycle (15+ steps)
- **State Persistence Data Flow** - Checkpointer serialization/deserialization
- **File System Data Flow** - Approval gates and file watching
- **WebSocket Event Data Flow** - Event broadcasting architecture
- **Evaluation Data Flow** - Test execution through judge evaluation
- **Message Flow Through Agent System** - Inter-agent communication
- **Plan Execution State Transitions** - State machine diagram

**Performance Metrics**: Latency targets, throughput, and data volumes

---

## 🎨 Diagram Technology

All diagrams use **Mermaid** syntax, which renders beautifully in:
- ✅ GitHub (native support)
- ✅ VS Code (with Mermaid extension)
- ✅ GitLab
- ✅ Notion
- ✅ Confluence
- ✅ Most modern markdown viewers

### Rendering Diagrams

#### In GitHub
Simply view the `.md` files - GitHub renders Mermaid automatically.

#### In VS Code
1. Install extension: `Markdown Preview Mermaid Support`
2. Open any diagram file
3. Press `Cmd/Ctrl + Shift + V` for preview

#### Online
Copy any diagram block and paste into [Mermaid Live Editor](https://mermaid.live)

---

## 📖 How to Read the Diagrams

### Color Coding

#### Agent Architecture
- 🔵 **Blue** - Main/Supervisor agents
- 🟢 **Green** - Planning agents
- 🔴 **Red** - Researcher agents
- 🟠 **Orange** - Data Scientist agents
- 🟣 **Purple** - Expert Analyst agents
- 🔵 **Light Blue** - Writer agents
- 🔴 **Dark Red** - Reviewer agents
- 🟡 **Yellow** - Judge agents

#### Data & Storage
- 🟦 **Dark Blue** - PostgreSQL databases
- ⬛ **Gray** - File systems
- 🟪 **Purple** - WebSocket connections

### Arrow Types
- **Solid arrows** (`→`) - Direct invocation, delegation, or data flow
- **Dashed arrows** (`-.->`) - State persistence or async operations
- **Bidirectional arrows** (`<-->`) - Two-way communication

### Node Shapes
- **Rectangles** - Services, agents, or processes
- **Rounded rectangles** - API endpoints or operations
- **Cylinders** - Databases or persistent storage
- **Diamonds** - Decision points or conditions
- **Circles** - Start/end states

---

## 🔍 Quick Navigation

### By Use Case

**I want to understand...**

1. **"How do agents work together?"**
   → [Agent Architecture - Multi-Agent System Overview](./agent_architecture.md#multi-agent-system-overview)

2. **"What APIs are available?"**
   → [API Endpoints - API Endpoint Overview](./api_endpoints.md#api-endpoint-overview)

3. **"How does data flow through the system?"**
   → [Data Flow - System-Wide Data Flow](./data_flow.md#system-wide-data-flow-overview)

4. **"How does real-time communication work?"**
   → [Agent Architecture - Real-Time Communication Flow](./agent_architecture.md#real-time-communication-flow)

5. **"How is state persisted?"**
   → [Agent Architecture - State Persistence](./agent_architecture.md#state-persistence-architecture)

6. **"How does the evaluation framework work?"**
   → [Agent Architecture - Evaluation Framework](./agent_architecture.md#evaluation-framework-architecture)

7. **"What's the message flow between agents?"**
   → [Data Flow - Message Flow](./data_flow.md#message-flow-through-agent-system)

8. **"How do approval gates work?"**
   → [Data Flow - File System Data Flow](./data_flow.md#file-system-data-flow)

---

## 🛠️ Extending the Diagrams

### Adding a New Diagram

1. Create new `.md` file in `docs/diagrams/`
2. Use Mermaid syntax wrapped in code blocks:
   ````markdown
   ```mermaid
   graph TB
       A[Start] --> B[Process]
       B --> C[End]
   ```
   ````
3. Add entry to this README
4. Test rendering in GitHub

### Diagram Best Practices

1. **Keep it focused** - One diagram, one concept
2. **Use color consistently** - Follow existing color scheme
3. **Add legends** - Explain colors and symbols
4. **Break down complexity** - Multiple simple diagrams > one complex diagram
5. **Include text descriptions** - Diagrams complement, not replace, documentation

---

## 📚 Related Documentation

- [CLAUDE.md](../../CLAUDE.md) - AI assistant guidance
- [README.md](../../README.md) - Project overview
- [docs/architecture/](../architecture/) - Detailed architecture docs
- [docs/setup/](../setup/) - Setup and installation guides

---

## 🔄 Maintenance

### Last Updated
November 27, 2025

### Update Frequency
Diagrams should be updated when:
- New agents/subagents are added
- API endpoints change
- Data flow patterns evolve
- Architecture is refactored

### Ownership
These diagrams are maintained by the TandemAI core team. For updates or corrections, please submit a PR or create an issue.

---

## 📊 Diagram Statistics

| Diagram Type | Count | Complexity |
|--------------|-------|------------|
| Architecture | 6 | Medium-High |
| API Maps | 1 | Medium |
| Sequence Diagrams | 4 | Medium |
| State Diagrams | 1 | Low |
| Flowcharts | 5 | High |
| **Total** | **17** | - |

**Lines of Mermaid Code**: ~800+  
**Estimated Render Time**: < 2 seconds per diagram

---

**Generated**: November 2025  
**Repository**: Zeeeepa/ATLAS (TandemAI)  
**License**: [Specify License]

