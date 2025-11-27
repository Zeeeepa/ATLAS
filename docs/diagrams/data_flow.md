# Data Flow Diagrams

## System-Wide Data Flow Overview

```mermaid
graph TB
    subgraph "Input Layer"
        UserQuery[User Query]
        APIRequest[API Request]
    end
    
    subgraph "Processing Layer"
        FastAPI[FastAPI Gateway]
        ThreadMgmt[Thread Manager]
        Agent[Main Agent]
        Planning[Planning Agent]
        Subagents[Subagent Pool]
    end
    
    subgraph "Data Sources"
        Tavily[Tavily Search API]
        Firecrawl[Firecrawl API]
        GitHub[GitHub API]
    end
    
    subgraph "Storage Layer"
        Postgres[(PostgreSQL<br/>Checkpoints & Threads)]
        Workspace[(Workspace FS<br/>Reports & Artifacts)]
        Plans[(Plans Directory<br/>.plans/*.json)]
    end
    
    subgraph "Output Layer"
        StreamingResp[SSE Stream]
        WebSocketEvents[WebSocket Events]
        FileArtifacts[File Artifacts]
    end
    
    UserQuery --> APIRequest
    APIRequest --> FastAPI
    
    FastAPI --> ThreadMgmt
    ThreadMgmt <--> Postgres
    
    FastAPI --> Agent
    Agent --> Planning
    Planning --> Subagents
    
    Subagents --> Tavily
    Subagents --> Firecrawl
    Subagents --> GitHub
    
    Agent <--> Postgres
    Planning <--> Postgres
    Subagents <--> Postgres
    
    Agent --> Plans
    Agent --> Workspace
    Subagents --> Workspace
    
    Agent --> StreamingResp
    Agent --> WebSocketEvents
    Workspace --> FileArtifacts
    
    StreamingResp --> UserQuery
    WebSocketEvents --> UserQuery
    FileArtifacts --> UserQuery
    
    style FastAPI fill:#4A90E2,stroke:#2E5C8A,color:#fff
    style Agent fill:#50C878,stroke:#2E7D4E,color:#fff
    style Postgres fill:#336791,stroke:#234A66,color:#fff
    style Workspace fill:#95A5A6,stroke:#7F8C8D,color:#fff
```

## Research Query Data Flow

```mermaid
flowchart TB
    Start([User submits query]) --> ParseQuery[Parse & validate query]
    ParseQuery --> CheckThread{Thread exists?}
    
    CheckThread -->|No| CreateThread[Create new thread in DB]
    CheckThread -->|Yes| LoadThread[Load thread from DB]
    
    CreateThread --> InitAgent[Initialize agent with state]
    LoadThread --> InitAgent
    
    InitAgent --> CreatePlan[Generate research plan]
    CreatePlan --> SavePlan[Save plan to .plans/]
    SavePlan --> BroadcastPlan[Broadcast 'plan_created' event]
    
    BroadcastPlan --> ExecuteLoop{More steps?}
    
    ExecuteLoop -->|Yes| DetermineAgent{Which agent needed?}
    
    DetermineAgent -->|Research| Researcher[Delegate to Researcher]
    DetermineAgent -->|Analysis| DataScientist[Delegate to Data Scientist]
    DetermineAgent -->|Writing| Writer[Delegate to Writer]
    DetermineAgent -->|Review| Reviewer[Delegate to Reviewer]
    
    Researcher --> SearchWeb[Tavily web search]
    SearchWeb --> ProcessResults[Process search results]
    ProcessResults --> WriteFile[Write to workspace]
    
    DataScientist --> ReadData[Read workspace files]
    ReadData --> AnalyzeData[Analyze data]
    AnalyzeData --> WriteAnalysis[Write analysis]
    
    Writer --> ReadSources[Read research files]
    ReadSources --> GenerateContent[Generate content]
    GenerateContent --> WriteDraft[Write draft]
    
    Reviewer --> ReadDraft[Read draft]
    ReadDraft --> EvaluateQuality[Evaluate quality]
    EvaluateQuality --> ProvideFeedback[Provide feedback]
    
    WriteFile --> SaveState[Save agent state to DB]
    WriteAnalysis --> SaveState
    WriteDraft --> SaveState
    ProvideFeedback --> SaveState
    
    SaveState --> UpdateProgress[Update plan progress]
    UpdateProgress --> BroadcastProgress[Broadcast progress event]
    BroadcastProgress --> ExecuteLoop
    
    ExecuteLoop -->|No| CompileResults[Compile final results]
    CompileResults --> StreamResponse[Stream response to client]
    StreamResponse --> SaveFinalState[Save final state]
    SaveFinalState --> BroadcastComplete[Broadcast 'completed' event]
    BroadcastComplete --> End([Response delivered])
    
    style CreatePlan fill:#50C878,stroke:#2E7D4E,color:#fff
    style Researcher fill:#FF6B6B,stroke:#CC5555,color:#fff
    style DataScientist fill:#FFA07A,stroke:#CC7D5F,color:#fff
    style Writer fill:#3498DB,stroke:#2471A3,color:#fff
    style Reviewer fill:#E74C3C,stroke:#B93C30,color:#fff
    style SaveState fill:#336791,stroke:#234A66,color:#fff
```

## State Persistence Data Flow

```mermaid
flowchart LR
    subgraph "Agent Runtime"
        AgentState[Agent State<br/>messages, logs, metadata]
        StateChange[State Change Event]
    end
    
    subgraph "Checkpointer"
        Serializer[State Serializer]
        Checkpointer[AsyncPostgresSaver]
    end
    
    subgraph "Database"
        CheckpointsTable[(checkpoints)]
        WritesTable[(writes)]
        ThreadsTable[(user_threads)]
    end
    
    subgraph "Recovery"
        Deserializer[State Deserializer]
        LoadState[Load State]
    end
    
    AgentState -->|onChange| StateChange
    StateChange -->|serialize| Serializer
    Serializer -->|write| Checkpointer
    
    Checkpointer -->|INSERT| CheckpointsTable
    Checkpointer -->|INSERT| WritesTable
    Checkpointer -->|UPDATE| ThreadsTable
    
    CheckpointsTable -->|SELECT| Checkpointer
    Checkpointer -->|deserialize| Deserializer
    Deserializer -->|restore| LoadState
    LoadState -->|resume| AgentState
    
    style Checkpointer fill:#336791,stroke:#234A66,color:#fff
    style CheckpointsTable fill:#4A5568,stroke:#2D3748,color:#fff
```

## File System Data Flow

```mermaid
flowchart TB
    subgraph "Agent Layer"
        AgentWrite[Agent writes file]
        ApprovalCheck{Approval required?}
    end
    
    subgraph "Approval System"
        RequestApproval[Request human approval]
        UserDecision{User decision}
    end
    
    subgraph "File Operations"
        ValidatePath[Validate path<br/>no traversal]
        ResolveAbs[Resolve absolute path]
        FileWrite[Write to filesystem]
    end
    
    subgraph "Monitoring"
        FileWatcher[File Watcher<br/>inotify/FSEvents]
        DetectChange[Detect file change]
        BroadcastEvent[Broadcast WebSocket event]
    end
    
    subgraph "Storage"
        WorkspaceFS[(Workspace FS<br/>backend/workspace/)]
    end
    
    AgentWrite --> ApprovalCheck
    
    ApprovalCheck -->|Yes| RequestApproval
    RequestApproval --> UserDecision
    UserDecision -->|Approved| ValidatePath
    UserDecision -->|Rejected| Abort([Abort operation])
    
    ApprovalCheck -->|No| ValidatePath
    
    ValidatePath -->|Valid| ResolveAbs
    ValidatePath -->|Invalid| Error([403 Forbidden])
    
    ResolveAbs --> FileWrite
    FileWrite --> WorkspaceFS
    
    WorkspaceFS --> FileWatcher
    FileWatcher --> DetectChange
    DetectChange --> BroadcastEvent
    BroadcastEvent --> UIUpdate([UI updates in real-time])
    
    style ApprovalCheck fill:#F39C12,stroke:#C27C0E,color:#fff
    style FileWatcher fill:#9B59B6,stroke:#7A3D91,color:#fff
    style WorkspaceFS fill:#95A5A6,stroke:#7F8C8D,color:#fff
```

## WebSocket Event Data Flow

```mermaid
sequenceDiagram
    participant Agent as Agent/Subagent
    participant Manager as WebSocket Manager
    participant Redis as Redis Pub/Sub<br/>(optional)
    participant Conn1 as Connection 1
    participant Conn2 as Connection 2
    participant Conn3 as Connection 3
    
    Note over Agent,Manager: Event Generation
    Agent->>Manager: emit(event, thread_id)
    Manager->>Manager: Filter by thread_id
    
    Note over Manager,Redis: Optional Distributed Mode
    alt Distributed (multiple servers)
        Manager->>Redis: PUBLISH channel, event
        Redis-->>Manager: Broadcast to subscribers
    end
    
    Note over Manager,Conn3: Fan-out to Clients
    par Parallel Broadcast
        Manager->>Conn1: send(event)
        Manager->>Conn2: send(event)
        Manager->>Conn3: send(event)
    end
    
    Note over Conn1,Conn3: Client Processing
    Conn1->>Conn1: Update UI
    Conn2->>Conn2: Update UI
    Conn3->>Conn3: Update UI
```

## Evaluation Data Flow

```mermaid
flowchart TB
    subgraph "Test Setup"
        TestQuery[Test Query]
        TestConfig[Test Configuration]
    end
    
    subgraph "Agent Execution"
        AgentWrapper[Agent Wrapper]
        Agent[Agent Under Test]
        AgentResponse[Agent Response]
    end
    
    subgraph "Judge Evaluation"
        JudgePool[Judge Pool<br/>7 Judges]
        
        J1[Accuracy Judge]
        J2[Completeness Judge]
        J3[Relevance Judge]
        J4[Clarity Judge]
        J5[Depth Judge]
        J6[Coherence Judge]
        J7[Actionability Judge]
    end
    
    subgraph "Scoring"
        Rubrics[Rubrics<br/>Scoring Criteria]
        ScoreAgg[Score Aggregator]
        FinalScore[Final Score]
    end
    
    subgraph "Reporting"
        ExperimentLog[Experiment Log]
        ResultsFile[Results JSON]
        MLflowLog[MLflow Tracking]
    end
    
    TestQuery --> TestConfig
    TestConfig --> AgentWrapper
    AgentWrapper --> Agent
    Agent --> AgentResponse
    
    AgentResponse --> JudgePool
    
    JudgePool --> J1
    JudgePool --> J2
    JudgePool --> J3
    JudgePool --> J4
    JudgePool --> J5
    JudgePool --> J6
    JudgePool --> J7
    
    Rubrics --> J1
    Rubrics --> J2
    Rubrics --> J3
    Rubrics --> J4
    Rubrics --> J5
    Rubrics --> J6
    Rubrics --> J7
    
    J1 --> ScoreAgg
    J2 --> ScoreAgg
    J3 --> ScoreAgg
    J4 --> ScoreAgg
    J5 --> ScoreAgg
    J6 --> ScoreAgg
    J7 --> ScoreAgg
    
    ScoreAgg --> FinalScore
    
    FinalScore --> ExperimentLog
    FinalScore --> ResultsFile
    FinalScore --> MLflowLog
    
    style JudgePool fill:#F39C12,stroke:#C27C0E,color:#fff
    style J1 fill:#3498DB,stroke:#2471A3,color:#fff
    style J2 fill:#3498DB,stroke:#2471A3,color:#fff
    style J3 fill:#3498DB,stroke:#2471A3,color:#fff
    style J4 fill:#3498DB,stroke:#2471A3,color:#fff
    style J5 fill:#3498DB,stroke:#2471A3,color:#fff
    style J6 fill:#3498DB,stroke:#2471A3,color:#fff
    style J7 fill:#3498DB,stroke:#2471A3,color:#fff
```

## Message Flow Through Agent System

```mermaid
flowchart TB
    UserMsg[User Message] --> MainAgent[Main Agent]
    
    MainAgent --> ParseIntent{Parse Intent}
    
    ParseIntent -->|Simple query| DirectResponse[Generate direct response]
    ParseIntent -->|Complex task| CreatePlan[Create execution plan]
    
    CreatePlan --> Step1{Step 1:<br/>Research}
    
    Step1 --> SubReq1[Create Researcher Request]
    SubReq1 --> ResearcherAgent[Researcher Agent]
    ResearcherAgent --> TavilyAPI[Tavily API]
    TavilyAPI --> ResearcherAgent
    ResearcherAgent --> FileOut1[Write /workspace/research.md]
    FileOut1 --> ToolMsg1[ToolMessage: Success]
    ToolMsg1 --> MainAgent
    
    MainAgent --> Step2{Step 2:<br/>Analyze}
    
    Step2 --> SubReq2[Create Data Scientist Request]
    SubReq2 --> DataScientistAgent[Data Scientist Agent]
    DataScientistAgent --> FileIn[Read /workspace/research.md]
    FileIn --> DataScientistAgent
    DataScientistAgent --> Analyze[Analyze data]
    Analyze --> FileOut2[Write /workspace/analysis.md]
    FileOut2 --> ToolMsg2[ToolMessage: Success]
    ToolMsg2 --> MainAgent
    
    MainAgent --> Step3{Step 3:<br/>Write}
    
    Step3 --> SubReq3[Create Writer Request]
    SubReq3 --> WriterAgent[Writer Agent]
    WriterAgent --> FileIn2[Read workspace files]
    FileIn2 --> WriterAgent
    WriterAgent --> Generate[Generate report]
    Generate --> FileOut3[Write /workspace/report.md]
    FileOut3 --> ToolMsg3[ToolMessage: Success]
    ToolMsg3 --> MainAgent
    
    MainAgent --> Step4{Step 4:<br/>Review}
    
    Step4 --> SubReq4[Create Reviewer Request]
    SubReq4 --> ReviewerAgent[Reviewer Agent]
    ReviewerAgent --> FileIn3[Read /workspace/report.md]
    FileIn3 --> ReviewerAgent
    ReviewerAgent --> Evaluate[Evaluate quality]
    Evaluate --> Feedback[Provide feedback]
    Feedback --> ToolMsg4[ToolMessage: Feedback]
    ToolMsg4 --> MainAgent
    
    MainAgent --> FinalResp{Quality OK?}
    
    FinalResp -->|Yes| CompletedMsg[Return final response]
    FinalResp -->|No| Revise[Request revisions]
    Revise --> Step3
    
    DirectResponse --> CompletedMsg
    CompletedMsg --> UserResp[Response to user]
    
    style MainAgent fill:#4A90E2,stroke:#2E5C8A,color:#fff
    style ResearcherAgent fill:#FF6B6B,stroke:#CC5555,color:#fff
    style DataScientistAgent fill:#FFA07A,stroke:#CC7D5F,color:#fff
    style WriterAgent fill:#3498DB,stroke:#2471A3,color:#fff
    style ReviewerAgent fill:#E74C3C,stroke:#B93C30,color:#fff
```

## Plan Execution State Transitions

```mermaid
stateDiagram-v2
    [*] --> PlanCreated: User submits query
    
    PlanCreated --> ExecutingStep: Start first step
    ExecutingStep --> SubagentStarted: Delegate to subagent
    SubagentStarted --> SubagentRunning: Subagent processing
    SubagentRunning --> SubagentCompleted: Task complete
    SubagentCompleted --> StepCompleted: Update progress
    
    StepCompleted --> ExecutingStep: More steps
    StepCompleted --> PlanCompleted: All steps done
    
    ExecutingStep --> PlanError: Error occurred
    SubagentRunning --> PlanError: Subagent failed
    
    PlanError --> Replan: Replanning needed
    Replan --> ExecutingStep: Resume execution
    
    PlanCompleted --> [*]
    
    note right of PlanCreated
        plan_id generated
        steps defined
        progress = 0.0
    end note
    
    note right of SubagentStarted
        Create thread_id
        Initialize subagent
        Broadcast event
    end note
    
    note right of StepCompleted
        Update progress
        Save state to DB
        Broadcast progress
    end note
```

## Data Types & Schemas

### Core Message Schema
```python
class Message:
    role: Literal["user", "assistant", "system"]
    content: str | List[ContentBlock]
    timestamp: datetime
    metadata: Dict[str, Any]
```

### Agent State Schema
```python
class AgentState(TypedDict):
    messages: List[BaseMessage]
    logs: List[LogEntry]
    plan: Optional[Plan]
    current_step: int
    progress: float
    thread_id: str
    user_id: str
```

### Plan Schema
```python
class Plan(BaseModel):
    plan_id: str
    query: str
    steps: List[str]  # 1-7 steps
    current_step: int
    progress: float  # 0.0 to 1.0
    past_steps: List[Tuple[str, str]]  # (step, result)
    status: Literal["active", "completed", "error"]
```

### File Metadata Schema
```python
class FileMetadata(BaseModel):
    size: int
    lastModified: float  # Unix timestamp
    extension: Optional[str]
```

### WebSocket Event Schema
```python
class WebSocketEvent(BaseModel):
    type: str  # event type
    thread_id: str
    timestamp: str  # ISO 8601
    data: Dict[str, Any]  # event-specific payload
```

---

## Data Flow Performance Characteristics

### Latency Targets
- **API Response Time**: < 200ms (excluding agent processing)
- **WebSocket Event Delivery**: < 50ms
- **Database State Save**: < 100ms
- **File Write Operations**: < 50ms
- **Agent Delegation**: < 500ms (includes state setup)

### Throughput
- **Concurrent Requests**: 100+ (FastAPI async)
- **WebSocket Connections**: 1000+ per instance
- **Database Writes/sec**: 50+ (PostgreSQL)
- **File Operations/sec**: 100+ (SSD dependent)

### Data Volumes
- **Average Message Size**: 1-5 KB
- **State Checkpoint Size**: 10-50 KB
- **Plan File Size**: 2-10 KB
- **Workspace File Size**: 1-1000 KB (reports/artifacts)
- **Database Growth**: ~100 MB per 1000 conversations

---

**Last Updated**: November 2025  
**Generated from**: ATLAS/TandemAI codebase analysis

