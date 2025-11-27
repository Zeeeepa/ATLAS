# Agent Architecture Diagram

## Multi-Agent System Overview

```mermaid
graph TB
    subgraph "Client Layer"
        UI[Next.js Frontend]
        CLI[CLI/Test Scripts]
    end
    
    subgraph "API Layer"
        FastAPI[FastAPI Backend<br/>backend_main.py]
        WS[WebSocket Manager]
    end
    
    subgraph "Agent Orchestration"
        MainAgent[Main Research Agent<br/>module_2_2_simple.py]
        PlanningAgent[Planning Agent<br/>planning_agent.py]
        
        MainAgent -->|creates plan| PlanningAgent
        PlanningAgent -->|returns plan| MainAgent
    end
    
    subgraph "Delegation Layer"
        DelegationTools[Delegation Tools<br/>delegation_tools.py]
        
        MainAgent -->|delegates via| DelegationTools
    end
    
    subgraph "Specialized Subagents"
        Researcher[Researcher Agent<br/>Web search & synthesis]
        DataScientist[Data Scientist Agent<br/>Analysis & processing]
        ExpertAnalyst[Expert Analyst Agent<br/>Domain expertise]
        Writer[Writer Agent<br/>Content generation]
        Reviewer[Reviewer Agent<br/>Quality control]
        
        DelegationTools -->|"delegate_to_researcher()"| Researcher
        DelegationTools -->|"delegate_to_data_scientist()"| DataScientist
        DelegationTools -->|"delegate_to_expert_analyst()"| ExpertAnalyst
        DelegationTools -->|"delegate_to_writer()"| Writer
        DelegationTools -->|"delegate_to_reviewer()"| Reviewer
    end
    
    subgraph "Tool Layer"
        TavilySearch[Tavily Search<br/>Web research]
        FileSystem[File System Tools<br/>read/write/edit]
        ApprovalGate[Human Approval<br/>approval_tools.py]
        
        Researcher -->|uses| TavilySearch
        Researcher -->|writes to| FileSystem
        DataScientist -->|uses| FileSystem
        Writer -->|writes to| FileSystem
        FileSystem -->|requires| ApprovalGate
    end
    
    subgraph "Persistence Layer"
        PostgreSQL[(PostgreSQL<br/>Checkpoints)]
        Workspace[Workspace FS<br/>backend/workspace/]
        
        MainAgent -.->|saves state| PostgreSQL
        PlanningAgent -.->|saves plan| PostgreSQL
        Researcher -.->|saves state| PostgreSQL
        DataScientist -.->|saves state| PostgreSQL
        ExpertAnalyst -.->|saves state| PostgreSQL
        Writer -.->|saves state| PostgreSQL
        Reviewer -.->|saves state| PostgreSQL
        
        FileSystem -->|reads/writes| Workspace
    end
    
    subgraph "Evaluation Framework"
        JudgeAgents[7 Judge Agents<br/>evaluation/judge_agents.py]
        TestRunner[Test Runner<br/>evaluation/test_runner.py]
        Rubrics[Rubrics<br/>evaluation/rubrics.py]
        
        TestRunner -->|invokes| MainAgent
        TestRunner -->|evaluates with| JudgeAgents
        JudgeAgents -->|uses| Rubrics
    end
    
    UI -->|HTTP/WebSocket| FastAPI
    CLI -->|invokes| TestRunner
    FastAPI -->|invokes| MainAgent
    FastAPI -->|broadcasts events| WS
    MainAgent -->|emits events| WS
    PlanningAgent -->|emits events| WS
    Researcher -->|emits events| WS
    DataScientist -->|emits events| WS
    ExpertAnalyst -->|emits events| WS
    Writer -->|emits events| WS
    Reviewer -->|emits events| WS
    
    style MainAgent fill:#4A90E2,stroke:#2E5C8A,color:#fff
    style PlanningAgent fill:#50C878,stroke:#2E7D4E,color:#fff
    style Researcher fill:#FF6B6B,stroke:#CC5555,color:#fff
    style DataScientist fill:#FFA07A,stroke:#CC7D5F,color:#fff
    style ExpertAnalyst fill:#9B59B6,stroke:#7A3D91,color:#fff
    style Writer fill:#3498DB,stroke:#2471A3,color:#fff
    style Reviewer fill:#E74C3C,stroke:#B93C30,color:#fff
    style PostgreSQL fill:#336791,stroke:#234A66,color:#fff
    style JudgeAgents fill:#F39C12,stroke:#C27C0E,color:#fff
```

## Agent Hierarchy Detailed View

```mermaid
graph LR
    subgraph "Research Flow"
        A[User Query] --> B{Planning Agent}
        B -->|generates plan| C[Main Agent]
        C -->|Step 1: Research| D[Researcher Agent]
        D -->|gathers data| E[Workspace Files]
        C -->|Step 2: Analyze| F[Data Scientist Agent]
        F -->|analyzes data| E
        C -->|Step 3: Expert Review| G[Expert Analyst Agent]
        G -->|provides insights| E
        C -->|Step 4: Write| H[Writer Agent]
        H -->|creates report| E
        C -->|Step 5: Review| I[Reviewer Agent]
        I -->|validates quality| E
        I -->|feedback| C
        C -->|final output| J[User Response]
    end
    
    style B fill:#50C878,stroke:#2E7D4E,color:#fff
    style C fill:#4A90E2,stroke:#2E5C8A,color:#fff
    style D fill:#FF6B6B,stroke:#CC5555,color:#fff
    style F fill:#FFA07A,stroke:#CC7D5F,color:#fff
    style G fill:#9B59B6,stroke:#7A3D91,color:#fff
    style H fill:#3498DB,stroke:#2471A3,color:#fff
    style I fill:#E74C3C,stroke:#B93C30,color:#fff
```

## State Persistence Architecture

```mermaid
graph TB
    subgraph "Agent State Management"
        Agent[Agent Instance]
        State[Agent State<br/>messages, logs, metadata]
        ThreadID[Thread ID<br/>Unique per conversation]
        
        Agent -->|maintains| State
        State -->|identified by| ThreadID
    end
    
    subgraph "Checkpointer Layer"
        Checkpointer[AsyncPostgresSaver<br/>LangGraph Checkpointer]
        
        Agent -->|saves to| Checkpointer
        Agent -->|loads from| Checkpointer
    end
    
    subgraph "Database Layer"
        CheckpointsTable[(checkpoints table)]
        WritesTable[(writes table)]
        UserThreadsTable[(user_threads table)]
        
        Checkpointer -->|writes| CheckpointsTable
        Checkpointer -->|writes| WritesTable
        Checkpointer -->|writes| UserThreadsTable
    end
    
    subgraph "Thread Management"
        CreateThread[Create Thread API]
        ListThreads[List Threads API]
        UpdateThread[Update Thread API]
        DeleteThread[Delete Thread API]
        
        CreateThread -->|inserts| UserThreadsTable
        ListThreads -->|queries| UserThreadsTable
        UpdateThread -->|updates| UserThreadsTable
        DeleteThread -->|soft delete| UserThreadsTable
    end
    
    style Checkpointer fill:#336791,stroke:#234A66,color:#fff
    style CheckpointsTable fill:#4A5568,stroke:#2D3748,color:#fff
    style WritesTable fill:#4A5568,stroke:#2D3748,color:#fff
    style UserThreadsTable fill:#4A5568,stroke:#2D3748,color:#fff
```

## Real-Time Communication Flow

```mermaid
sequenceDiagram
    participant Client as Frontend Client
    participant WS as WebSocket Manager
    participant Agent as Main Agent
    participant Subagent as Researcher Agent
    participant Tool as Tavily Search
    
    Client->>WS: Connect to /ws/plan
    WS-->>Client: Connection established
    
    Client->>Agent: POST /api/chat (query)
    
    Agent->>WS: Emit "agent_started" event
    WS-->>Client: Broadcast event
    
    Agent->>Agent: Create plan
    Agent->>WS: Emit "plan_created" event
    WS-->>Client: Display plan in UI
    
    Agent->>Subagent: delegate_to_researcher(task)
    Subagent->>WS: Emit "subagent_started" event
    WS-->>Client: Show subagent progress
    
    Subagent->>Tool: tavily_search(query)
    Tool-->>Subagent: Search results
    
    Subagent->>Subagent: Process results
    Subagent->>WS: Emit "subagent_progress" event
    WS-->>Client: Update progress bar
    
    Subagent->>Agent: Return results (ToolMessage)
    Agent->>WS: Emit "subagent_completed" event
    WS-->>Client: Mark step complete
    
    Agent->>Agent: Continue with next step
    Agent->>WS: Emit "plan_progress_update" event
    WS-->>Client: Update overall progress
    
    Agent->>Client: Stream final response
    Agent->>WS: Emit "agent_completed" event
    WS-->>Client: Show completion
```

## Evaluation Framework Architecture

```mermaid
graph TB
    subgraph "Test Configuration"
        TestConfig[Test Config<br/>e.g., test_config_1]
        TestQuery[Test Query]
        AgentWrapper[Agent Wrapper<br/>baseline/challenger]
        
        TestConfig -->|defines| TestQuery
        TestConfig -->|wraps| AgentWrapper
    end
    
    subgraph "Test Execution"
        TestRunner[Test Runner<br/>test_runner.py]
        Agent[Agent Under Test]
        
        TestRunner -->|invokes| AgentWrapper
        AgentWrapper -->|executes| Agent
        Agent -->|generates| Response[Agent Response]
    end
    
    subgraph "Judge Evaluation"
        JudgeRegistry[Judge Registry<br/>7 Specialized Judges]
        
        AccuracyJudge[Accuracy Judge]
        CompletenessJudge[Completeness Judge]
        RelevanceJudge[Relevance Judge]
        ClarityJudge[Clarity Judge]
        DepthJudge[Depth Judge]
        CoherenceJudge[Coherence Judge]
        ActionabilityJudge[Actionability Judge]
        
        JudgeRegistry -->|includes| AccuracyJudge
        JudgeRegistry -->|includes| CompletenessJudge
        JudgeRegistry -->|includes| RelevanceJudge
        JudgeRegistry -->|includes| ClarityJudge
        JudgeRegistry -->|includes| DepthJudge
        JudgeRegistry -->|includes| CoherenceJudge
        JudgeRegistry -->|includes| ActionabilityJudge
    end
    
    subgraph "Scoring & Analysis"
        Rubrics[Rubrics<br/>rubrics.py]
        Scores[Judge Scores]
        Aggregation[Score Aggregation]
        Report[Evaluation Report]
        
        Rubrics -->|defines criteria| JudgeRegistry
        JudgeRegistry -->|produces| Scores
        Scores -->|aggregated by| Aggregation
        Aggregation -->|generates| Report
    end
    
    Response -->|evaluated by| JudgeRegistry
    TestRunner -->|orchestrates| JudgeRegistry
    TestRunner -->|generates| Report
    
    style TestRunner fill:#F39C12,stroke:#C27C0E,color:#fff
    style JudgeRegistry fill:#E67E22,stroke:#BA6616,color:#fff
    style AccuracyJudge fill:#3498DB,stroke:#2471A3,color:#fff
    style CompletenessJudge fill:#3498DB,stroke:#2471A3,color:#fff
    style RelevanceJudge fill:#3498DB,stroke:#2471A3,color:#fff
    style ClarityJudge fill:#3498DB,stroke:#2471A3,color:#fff
    style DepthJudge fill:#3498DB,stroke:#2471A3,color:#fff
    style CoherenceJudge fill:#3498DB,stroke:#2471A3,color:#fff
    style ActionabilityJudge fill:#3498DB,stroke:#2471A3,color:#fff
```

---

## Legend

### Agent Types
- 🔵 **Main Agent** - Orchestrates the entire workflow
- 🟢 **Planning Agent** - Creates and manages execution plans
- 🔴 **Researcher Agent** - Web search and information gathering
- 🟠 **Data Scientist Agent** - Data analysis and processing
- 🟣 **Expert Analyst Agent** - Domain-specific expertise
- 🔵 **Writer Agent** - Content creation and synthesis
- 🔴 **Reviewer Agent** - Quality assurance and validation
- 🟡 **Judge Agents** - Evaluation and scoring

### Communication Patterns
- **Solid arrows** - Direct invocation/delegation
- **Dashed arrows** - State persistence
- **WebSocket events** - Real-time updates to frontend

### Persistence
- **PostgreSQL** - Conversation state and checkpoints
- **Workspace FS** - File-based artifacts (reports, data)

---

**Last Updated**: November 2025  
**Generated from**: ATLAS/TandemAI codebase analysis

