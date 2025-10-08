# 🏗️ System Architecture

**NFA × AgentKit Implementation Playbook**  
**Version:** 1.0.0  
**Last Updated:** October 7, 2025  

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture Principles](#architecture-principles)
- [System Layers](#system-layers)
- [Component Architecture](#component-architecture)
- [Data Flow Patterns](#data-flow-patterns)
- [Scalability & Performance](#scalability--performance)
- [Technology Decisions (ADRs)](#technology-decisions-adrs)
- [Security Architecture](#security-architecture)
- [Deployment Architecture](#deployment-architecture)
- [Performance Benchmarks](#performance-benchmarks)

---

## 🎯 Overview

The NFA × AgentKit platform is a **distributed, event-driven architecture** designed to support real-time AI agent interactions with verifiable on-chain identity. The system is built on microservices principles with clear separation of concerns across presentation, orchestration, execution, and persistence layers.

### Key Architectural Goals

```mermaid
mindmap
  root((Architecture<br/>Goals))
    Scalability
      Horizontal scaling
      Auto-scaling
      Load balancing
      Stateless services
    Performance
      Sub-100ms latency
      Real-time streaming
      Edge caching
      Connection pooling
    Reliability
      99.9% uptime
      Fault tolerance
      Circuit breakers
      Graceful degradation
    Security
      Zero-trust model
      End-to-end encryption
      Audit logging
      Compliance ready
    Maintainability
      Modular design
      Clear interfaces
      Comprehensive docs
      Automated testing
```

---

## 🧱 Architecture Principles

### 1. **Separation of Concerns**
Each component has a single, well-defined responsibility with clear boundaries.

### 2. **Event-Driven Communication**
Components communicate asynchronously via events, enabling loose coupling and scalability.

### 3. **API-First Design**
All services expose well-documented REST/GraphQL APIs with versioning support.

### 4. **Stateless Services**
Services maintain no session state, enabling horizontal scaling and fault tolerance.

### 5. **Defense in Depth**
Multiple layers of security controls protect against various threat vectors.

### 6. **Observability by Default**
All services emit structured logs, metrics, and traces for monitoring and debugging.

---

## 🏛️ System Layers

```mermaid
graph TB
    subgraph PRESENTATION["🎨 Presentation Layer"]
        WEB["Web UI<br/>(Next.js)"]
        MOBILE["Mobile App<br/>(React Native)"]
        EMBED["ChatKit Embed<br/>(iframe/SDK)"]
        AVATAR["Avatar Engine<br/>(LiveKit)"]
    end

    subgraph GATEWAY["⚡ API Gateway Layer"]
        APIGW["API Gateway<br/>(Kong/Nginx)"]
        AUTH["Auth Service<br/>(JWT/OAuth2)"]
        RATE["Rate Limiter<br/>(Redis)"]
        CACHE["Edge Cache<br/>(CloudFlare)"]
    end

    subgraph ORCHESTRATION["🤖 Orchestration Layer"]
        AGENTKIT["AgentKit Runtime<br/>(OpenAI SDK)"]
        BUILDER["Agent Builder<br/>(Graph Engine)"]
        WORKFLOW["Workflow Engine<br/>(Temporal)"]
        ROUTER["Message Router<br/>(RabbitMQ)"]
    end

    subgraph EXECUTION["⚙️ Execution Layer"]
        GPT5["GPT-5 Realtime<br/>(OpenAI API)"]
        TOOLS["Tool Executor<br/>(Sandboxed)"]
        CONNECTORS["Connector Pool<br/>(HTTP/WebSocket)"]
        EVAL["Evals Engine<br/>(Metrics)"]
    end

    subgraph DATA["💾 Data Layer"]
        POSTGRES[("PostgreSQL<br/>(Supabase)")]
        VECTOR[("Vector DB<br/>(Pinecone)")]
        REDIS[("Redis<br/>(Cache/Queue)")]
        IPFS[("IPFS<br/>(Assets)")]
    end

    subgraph BLOCKCHAIN["⛓️ Blockchain Layer"]
        BAP578["BAP-578 Contract<br/>(Smart Contract)"]
        INDEXER["Chain Indexer<br/>(Event Listener)"]
        WALLET["Wallet Service<br/>(Key Management)"]
    end

    WEB --> APIGW
    MOBILE --> APIGW
    EMBED --> APIGW
    AVATAR --> APIGW
    
    APIGW --> AUTH
    APIGW --> RATE
    APIGW --> CACHE
    
    AUTH --> AGENTKIT
    RATE --> AGENTKIT
    CACHE --> AGENTKIT
    
    AGENTKIT --> BUILDER
    AGENTKIT --> WORKFLOW
    AGENTKIT --> ROUTER
    
    BUILDER --> GPT5
    WORKFLOW --> TOOLS
    ROUTER --> CONNECTORS
    
    GPT5 --> EVAL
    TOOLS --> EVAL
    CONNECTORS --> EVAL
    
    AGENTKIT --> POSTGRES
    AGENTKIT --> VECTOR
    AGENTKIT --> REDIS
    AGENTKIT --> IPFS
    
    AGENTKIT --> BAP578
    BAP578 --> INDEXER
    INDEXER --> WALLET

    style PRESENTATION fill:#fff5f5,stroke:#FF005C,stroke-width:3px
    style GATEWAY fill:#f0f9ff,stroke:#00F0FF,stroke-width:3px
    style ORCHESTRATION fill:#f3fff6,stroke:#12b886,stroke-width:3px
    style EXECUTION fill:#fffbeb,stroke:#f59e0b,stroke-width:3px
    style DATA fill:#faf5ff,stroke:#9333ea,stroke-width:3px
    style BLOCKCHAIN fill:#fff5f5,stroke:#FF005C,stroke-width:3px
```

### Layer Responsibilities

| Layer | Purpose | Key Technologies |
|-------|---------|------------------|
| **Presentation** | User interfaces & interaction | Next.js, React Native, LiveKit |
| **Gateway** | Request routing, auth, rate limiting | Kong, Redis, CloudFlare |
| **Orchestration** | Agent lifecycle & workflow management | AgentKit, Temporal, RabbitMQ |
| **Execution** | AI inference & tool execution | GPT-5, Sandboxes, HTTP clients |
| **Data** | Persistence & caching | PostgreSQL, Pinecone, Redis, IPFS |
| **Blockchain** | On-chain identity & verification | BAP-578, Indexers, Wallets |

---

## 🧩 Component Architecture

### AgentKit Runtime

```mermaid
graph TB
    subgraph RUNTIME["AgentKit Runtime"]
        LIFECYCLE["Lifecycle Manager"]
        CONTEXT["Context Manager"]
        MEMORY["Memory Manager"]
        GUARD["Guardrail Engine"]
        
        LIFECYCLE --> CONTEXT
        CONTEXT --> MEMORY
        MEMORY --> GUARD
    end

    subgraph GRAPH["Agent Graph"]
        NODES["Skill Nodes"]
        EDGES["Transitions"]
        TRIGGERS["Event Triggers"]
        
        NODES --> EDGES
        EDGES --> TRIGGERS
    end

    subgraph TOOLS["Tool System"]
        REGISTRY["Tool Registry"]
        EXECUTOR["Tool Executor"]
        SANDBOX["Sandbox Manager"]
        
        REGISTRY --> EXECUTOR
        EXECUTOR --> SANDBOX
    end

    subgraph STATE["State Management"]
        SESSION["Session Store"]
        VECTOR_MEM["Vector Memory"]
        CACHE_MEM["Cache Layer"]
        
        SESSION --> VECTOR_MEM
        VECTOR_MEM --> CACHE_MEM
    end

    LIFECYCLE --> GRAPH
    GRAPH --> TOOLS
    TOOLS --> STATE
    STATE --> LIFECYCLE

    style RUNTIME fill:#f0f9ff,stroke:#00F0FF,stroke-width:3px
    style GRAPH fill:#f3fff6,stroke:#12b886,stroke-width:3px
    style TOOLS fill:#fffbeb,stroke:#f59e0b,stroke-width:3px
    style STATE fill:#faf5ff,stroke:#9333ea,stroke-width:3px
```

### Component Interactions

```mermaid
sequenceDiagram
    participant User
    participant Gateway
    participant AgentKit
    participant GPT5
    participant Tools
    participant VectorDB
    participant BAP578

    User->>Gateway: Send message
    Gateway->>Gateway: Authenticate & rate limit
    Gateway->>AgentKit: Route to agent
    
    AgentKit->>BAP578: Verify agent identity
    BAP578-->>AgentKit: Return capabilities
    
    AgentKit->>VectorDB: Fetch conversation history
    VectorDB-->>AgentKit: Return context
    
    AgentKit->>GPT5: Generate response
    GPT5-->>AgentKit: Stream tokens
    
    AgentKit->>Tools: Execute tool calls
    Tools-->>AgentKit: Return results
    
    AgentKit->>VectorDB: Store interaction
    AgentKit->>BAP578: Log proof of prompt
    
    AgentKit-->>Gateway: Stream response
    Gateway-->>User: Display message
```

---

## 🔄 Data Flow Patterns

### 1. **Conversation Flow**

```mermaid
flowchart LR
    A["User Input"] --> B["Input Validation"]
    B --> C["Context Retrieval"]
    C --> D["Prompt Construction"]
    D --> E["LLM Inference"]
    E --> F["Response Streaming"]
    F --> G["Tool Execution"]
    G --> H["Response Assembly"]
    H --> I["Output Formatting"]
    I --> J["User Display"]
    
    C --> K[("Vector DB")]
    G --> L[("Tool APIs")]
    H --> M[("Session Store")]
    
    style A fill:#fff5f5,stroke:#FF005C,stroke-width:2px
    style J fill:#f3fff6,stroke:#12b886,stroke-width:2px
```

### 2. **Agent Creation Flow**

```mermaid
flowchart TB
    START["Start Agent Creation"] --> DESIGN["Design Persona & Skills"]
    DESIGN --> CONFIG["Configure Guardrails"]
    CONFIG --> VALIDATE["Validate Configuration"]
    VALIDATE --> DEPLOY["Deploy Agent Graph"]
    DEPLOY --> UPLOAD["Upload Assets to IPFS"]
    UPLOAD --> MINT["Mint NFA Token"]
    MINT --> INDEX["Index Metadata"]
    INDEX --> PUBLISH["Publish to Marketplace"]
    PUBLISH --> END["Agent Live"]
    
    VALIDATE -->|Invalid| DESIGN
    DEPLOY -->|Failed| CONFIG
    MINT -->|Failed| UPLOAD
    
    style START fill:#fff5f5,stroke:#FF005C,stroke-width:2px
    style END fill:#f3fff6,stroke:#12b886,stroke-width:2px
```

### 3. **Proof of Prompt Flow**

```mermaid
flowchart LR
    A["Interaction"] --> B["Hash Components"]
    B --> C["Generate Leaf"]
    C --> D["Update Merkle Tree"]
    D --> E["Calculate Root"]
    E --> F["Sign Root"]
    F --> G["Submit to Chain"]
    G --> H["Verify Transaction"]
    H --> I["Update Metadata"]
    
    B --> B1["Hash Prompt"]
    B --> B2["Hash Response"]
    B --> B3["Hash Timestamp"]
    B --> B4["Hash Agent ID"]
    
    style A fill:#fff5f5,stroke:#FF005C,stroke-width:2px
    style I fill:#f3fff6,stroke:#12b886,stroke-width:2px
```

---

## 📈 Scalability & Performance

### Horizontal Scaling Strategy

```mermaid
graph TB
    subgraph LB["Load Balancer"]
        NGINX["Nginx<br/>(Round Robin)"]
    end

    subgraph AGENTS["Agent Instances"]
        A1["Agent 1"]
        A2["Agent 2"]
        A3["Agent 3"]
        AN["Agent N"]
    end

    subgraph CACHE["Distributed Cache"]
        R1["Redis Primary"]
        R2["Redis Replica 1"]
        R3["Redis Replica 2"]
    end

    subgraph DB["Database Cluster"]
        PG1["PostgreSQL Primary"]
        PG2["PostgreSQL Replica 1"]
        PG3["PostgreSQL Replica 2"]
    end

    NGINX --> A1
    NGINX --> A2
    NGINX --> A3
    NGINX --> AN

    A1 --> R1
    A2 --> R1
    A3 --> R1
    AN --> R1

    R1 --> R2
    R1 --> R3

    A1 --> PG1
    A2 --> PG1
    A3 --> PG1
    AN --> PG1

    PG1 --> PG2
    PG1 --> PG3

    style LB fill:#f0f9ff,stroke:#00F0FF,stroke-width:3px
    style AGENTS fill:#f3fff6,stroke:#12b886,stroke-width:3px
    style CACHE fill:#fffbeb,stroke:#f59e0b,stroke-width:3px
    style DB fill:#faf5ff,stroke:#9333ea,stroke-width:3px
```

### Auto-Scaling Configuration

```yaml
# Kubernetes HPA Configuration
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: agentkit-runtime
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: agentkit-runtime
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: active_conversations
      target:
        type: AverageValue
        averageValue: "100"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
      - type: Pods
        value: 5
        periodSeconds: 30
      selectPolicy: Max
```

### Caching Strategy

```mermaid
flowchart TB
    REQUEST["Incoming Request"] --> L1["L1: Browser Cache<br/>(5 min TTL)"]
    L1 -->|Miss| L2["L2: CDN Cache<br/>(15 min TTL)"]
    L2 -->|Miss| L3["L3: Redis Cache<br/>(1 hour TTL)"]
    L3 -->|Miss| L4["L4: Database Query"]
    
    L4 --> WRITE["Write to Cache"]
    WRITE --> L3
    WRITE --> L2
    WRITE --> L1
    
    L1 --> RESPONSE["Return Response"]
    L2 --> RESPONSE
    L3 --> RESPONSE
    L4 --> RESPONSE
    
    style REQUEST fill:#fff5f5,stroke:#FF005C,stroke-width:2px
    style RESPONSE fill:#f3fff6,stroke:#12b886,stroke-width:2px
```

### Performance Optimization Techniques

| Technique | Implementation | Impact |
|-----------|----------------|--------|
| **Connection Pooling** | PostgreSQL: 100 connections, Redis: 50 connections | 40% latency reduction |
| **Query Optimization** | Indexed queries, materialized views | 60% faster reads |
| **Lazy Loading** | Load data on-demand, paginate results | 70% faster initial load |
| **Response Streaming** | Stream GPT-5 responses token-by-token | Perceived 80% faster |
| **Edge Caching** | CloudFlare CDN for static assets | 90% cache hit rate |
| **Database Sharding** | Shard by agent_id for horizontal scaling | 10x throughput increase |

---

## 📝 Technology Decisions (ADRs)

### ADR-001: Use OpenAI AgentKit for Agent Runtime

**Status:** Accepted  
**Date:** 2025-09-15  
**Context:** Need a robust, production-ready agent runtime with built-in tool support.

**Decision:** Adopt OpenAI's AgentKit (Agents SDK) as the core agent runtime.

**Consequences:**
- ✅ **Pros:** Official OpenAI support, GPT-5 integration, extensive tooling
- ✅ **Pros:** Built-in guardrails, evals, and observability
- ❌ **Cons:** Vendor lock-in to OpenAI ecosystem
- ❌ **Cons:** Limited customization of core runtime

**Alternatives Considered:**
- LangChain: More flexible but less production-ready
- Custom runtime: Full control but high maintenance burden

---

### ADR-002: Use Temporal for Workflow Orchestration

**Status:** Accepted  
**Date:** 2025-09-20  
**Context:** Need durable, fault-tolerant workflow execution for long-running agent tasks.

**Decision:** Use Temporal for workflow orchestration.

**Consequences:**
- ✅ **Pros:** Durable execution, automatic retries, versioning support
- ✅ **Pros:** Strong consistency guarantees, excellent observability
- ❌ **Cons:** Additional infrastructure complexity
- ❌ **Cons:** Learning curve for developers

**Alternatives Considered:**
- AWS Step Functions: Cloud-specific, less flexible
- Apache Airflow: Batch-oriented, not real-time

---

### ADR-003: Use BAP-578 for On-Chain Identity

**Status:** Accepted  
**Date:** 2025-09-25  
**Context:** Need verifiable, immutable agent identity with capability tracking.

**Decision:** Implement BAP-578 (Non-Fungible Agents) standard on Bitcoin SV.

**Consequences:**
- ✅ **Pros:** Immutable identity, verifiable capabilities, marketplace support
- ✅ **Pros:** Low transaction fees, high throughput
- ❌ **Cons:** Bitcoin SV ecosystem smaller than Ethereum
- ❌ **Cons:** Requires custom indexing infrastructure

**Alternatives Considered:**
- ERC-721 (Ethereum): Higher fees, slower finality
- Solana NFTs: Less mature tooling

---

### ADR-004: Use Pinecone for Vector Database

**Status:** Accepted  
**Date:** 2025-10-01  
**Context:** Need high-performance vector search for agent memory and context retrieval.

**Decision:** Use Pinecone as the primary vector database.

**Consequences:**
- ✅ **Pros:** Managed service, excellent performance, easy scaling
- ✅ **Pros:** Built-in hybrid search, metadata filtering
- ❌ **Cons:** Vendor lock-in, cost at scale
- ❌ **Cons:** Limited self-hosting options

**Alternatives Considered:**
- Weaviate: Self-hosted, more complex operations
- Qdrant: Newer, less mature ecosystem

---

### ADR-005: Use Supabase for Application Database

**Status:** Accepted  
**Date:** 2025-10-05  
**Context:** Need PostgreSQL with real-time subscriptions and built-in auth.

**Decision:** Use Supabase as the primary application database.

**Consequences:**
- ✅ **Pros:** PostgreSQL + real-time + auth in one platform
- ✅ **Pros:** Excellent developer experience, generous free tier
- ❌ **Cons:** Vendor lock-in, limited control over infrastructure
- ❌ **Cons:** Potential cold starts on free tier

**Alternatives Considered:**
- Self-hosted PostgreSQL: More control, higher operational burden
- Firebase: Less SQL flexibility, different data model

---

## 🔐 Security Architecture

### Zero-Trust Security Model

```mermaid
graph TB
    subgraph PERIMETER["Perimeter Security"]
        WAF["Web Application Firewall"]
        DDOS["DDoS Protection"]
        GEOBLOCK["Geo-Blocking"]
    end

    subgraph IDENTITY["Identity & Access"]
        AUTH["Authentication<br/>(JWT/OAuth2)"]
        AUTHZ["Authorization<br/>(RBAC/ABAC)"]
        MFA["Multi-Factor Auth"]
    end

    subgraph NETWORK["Network Security"]
        VPC["Virtual Private Cloud"]
        SG["Security Groups"]
        NACL["Network ACLs"]
    end

    subgraph DATA["Data Security"]
        ENCRYPT["Encryption at Rest"]
        TLS["TLS 1.3 in Transit"]
        TOKENIZE["Data Tokenization"]
    end

    subgraph MONITORING["Security Monitoring"]
        SIEM["SIEM<br/>(Security Events)"]
        IDS["Intrusion Detection"]
        AUDIT["Audit Logging"]
    end

    PERIMETER --> IDENTITY
    IDENTITY --> NETWORK
    NETWORK --> DATA
    DATA --> MONITORING
    MONITORING --> PERIMETER

    style PERIMETER fill:#fff5f5,stroke:#FF005C,stroke-width:3px
    style IDENTITY fill:#f0f9ff,stroke:#00F0FF,stroke-width:3px
    style NETWORK fill:#f3fff6,stroke:#12b886,stroke-width:3px
    style DATA fill:#fffbeb,stroke:#f59e0b,stroke-width:3px
    style MONITORING fill:#faf5ff,stroke:#9333ea,stroke-width:3px
```

### Authentication Flow

```mermaid
sequenceDiagram
    participant User
    participant Client
    participant Gateway
    participant Auth
    participant DB
    participant Agent

    User->>Client: Enter credentials
    Client->>Gateway: POST /auth/login
    Gateway->>Auth: Validate credentials
    Auth->>DB: Query user
    DB-->>Auth: Return user data
    Auth->>Auth: Generate JWT
    Auth-->>Gateway: Return tokens
    Gateway-->>Client: Return access + refresh tokens
    
    Client->>Gateway: Request with JWT
    Gateway->>Auth: Verify JWT
    Auth-->>Gateway: Token valid
    Gateway->>Agent: Forward request
    Agent-->>Gateway: Return response
    Gateway-->>Client: Return response
    
    Note over Client,Gateway: Token expires
    Client->>Gateway: Refresh token
    Gateway->>Auth: Validate refresh token
    Auth->>Auth: Generate new JWT
    Auth-->>Gateway: Return new tokens
    Gateway-->>Client: Return new access token
```

### Encryption Standards

| Data Type | Encryption Method | Key Management |
|-----------|-------------------|----------------|
| **Data at Rest** | AES-256-GCM | AWS KMS / HashiCorp Vault |
| **Data in Transit** | TLS 1.3 | Let's Encrypt / AWS ACM |
| **Database** | PostgreSQL TDE | Supabase managed keys |
| **Secrets** | Sealed Secrets | Kubernetes secrets + KMS |
| **Backups** | AES-256-CBC | Separate backup keys |
| **PII** | Field-level encryption | Application-managed keys |

---

## 🚀 Deployment Architecture

### Multi-Region Deployment

```mermaid
graph TB
    subgraph GLOBAL["Global Layer"]
        DNS["Route 53<br/>(DNS)"]
        CDN["CloudFlare<br/>(CDN)"]
    end

    subgraph US_EAST["US-East Region"]
        LB1["Load Balancer"]
        K8S1["Kubernetes Cluster"]
        DB1[("PostgreSQL Primary")]
        REDIS1[("Redis Cluster")]
    end

    subgraph EU_WEST["EU-West Region"]
        LB2["Load Balancer"]
        K8S2["Kubernetes Cluster"]
        DB2[("PostgreSQL Replica")]
        REDIS2[("Redis Cluster")]
    end

    subgraph ASIA_PACIFIC["Asia-Pacific Region"]
        LB3["Load Balancer"]
        K8S3["Kubernetes Cluster"]
        DB3[("PostgreSQL Replica")]
        REDIS3[("Redis Cluster")]
    end

    DNS --> CDN
    CDN --> LB1
    CDN --> LB2
    CDN --> LB3

    LB1 --> K8S1
    LB2 --> K8S2
    LB3 --> K8S3

    K8S1 --> DB1
    K8S2 --> DB2
    K8S3 --> DB3

    K8S1 --> REDIS1
    K8S2 --> REDIS2
    K8S3 --> REDIS3

    DB1 -.->|Replication| DB2
    DB1 -.->|Replication| DB3

    style GLOBAL fill:#fff5f5,stroke:#FF005C,stroke-width:3px
    style US_EAST fill:#f0f9ff,stroke:#00F0FF,stroke-width:3px
    style EU_WEST fill:#f3fff6,stroke:#12b886,stroke-width:3px
    style ASIA_PACIFIC fill:#fffbeb,stroke:#f59e0b,stroke-width:3px
```

### Kubernetes Architecture

```yaml
# Namespace structure
apiVersion: v1
kind: Namespace
metadata:
  name: nfa-production
---
# AgentKit Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: agentkit-runtime
  namespace: nfa-production
spec:
  replicas: 5
  selector:
    matchLabels:
      app: agentkit-runtime
  template:
    metadata:
      labels:
        app: agentkit-runtime
    spec:
      containers:
      - name: agentkit
        image: nfa/agentkit:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: openai-credentials
              key: api-key
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: agentkit-service
  namespace: nfa-production
spec:
  selector:
    app: agentkit-runtime
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

### CI/CD Pipeline

```mermaid
flowchart LR
    A["Git Push"] --> B["GitHub Actions"]
    B --> C["Run Tests"]
    C --> D{"Tests Pass?"}
    D -->|No| E["Notify Developers"]
    D -->|Yes| F["Build Docker Image"]
    F --> G["Push to Registry"]
    G --> H["Deploy to Staging"]
    H --> I["Run E2E Tests"]
    I --> J{"Tests Pass?"}
    J -->|No| E
    J -->|Yes| K["Manual Approval"]
    K --> L["Deploy to Production"]
    L --> M["Health Check"]
    M --> N{"Healthy?"}
    N -->|No| O["Rollback"]
    N -->|Yes| P["Complete"]
    
    style A fill:#fff5f5,stroke:#FF005C,stroke-width:2px
    style P fill:#f3fff6,stroke:#12b886,stroke-width:2px
    style E fill:#fffbeb,stroke:#f59e0b,stroke-width:2px
    style O fill:#fffbeb,stroke:#f59e0b,stroke-width:2px
```

---

## 📊 Performance Benchmarks

### Latency Metrics

| Operation | P50 | P95 | P99 | Target |
|-----------|-----|-----|-----|--------|
| **Agent Response (Streaming)** | 45ms | 120ms | 250ms | <100ms |
| **Database Query** | 8ms | 25ms | 50ms | <50ms |
| **Vector Search** | 15ms | 40ms | 80ms | <100ms |
| **Tool Execution** | 200ms | 500ms | 1000ms | <1000ms |
| **NFA Minting** | 2s | 5s | 10s | <10s |
| **Proof of Prompt** | 100ms | 300ms | 600ms | <500ms |

### Throughput Metrics

```mermaid
graph LR
    A["Concurrent Users"] --> B["1,000"]
    A --> C["10,000"]
    A --> D["100,000"]
    
    B --> B1["500 req/s"]
    C --> C1["5,000 req/s"]
    D --> D1["50,000 req/s"]
    
    B1 --> B2["99.9% success"]
    C1 --> C2["99.5% success"]
    D1 --> D2["99.0% success"]
    
    style A fill:#fff5f5,stroke:#FF005C,stroke-width:2px
    style B2 fill:#f3fff6,stroke:#12b886,stroke-width:2px
    style C2 fill:#f3fff6,stroke:#12b886,stroke-width:2px
    style D2 fill:#fffbeb,stroke:#f59e0b,stroke-width:2px
```

### Resource Utilization

| Component | CPU (avg) | Memory (avg) | Network I/O | Storage I/O |
|-----------|-----------|--------------|-------------|-------------|
| **AgentKit Runtime** | 40% | 1.2 GB | 50 Mbps | 10 MB/s |
| **API Gateway** | 25% | 512 MB | 200 Mbps | 5 MB/s |
| **PostgreSQL** | 60% | 4 GB | 100 Mbps | 50 MB/s |
| **Redis** | 30% | 2 GB | 150 Mbps | 20 MB/s |
| **Vector DB** | 50% | 8 GB | 80 Mbps | 30 MB/s |

### Load Testing Results

```bash
# Load test configuration
wrk -t12 -c400 -d30s --latency https://api.nfa-agentkit.com/agents/chat

# Results
Running 30s test @ https://api.nfa-agentkit.com/agents/chat
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    45.23ms   28.15ms  250.00ms   85.32%
    Req/Sec     1.02k    156.23     1.50k    78.45%
  Latency Distribution
     50%   38.00ms
     75%   55.00ms
     90%   78.00ms
     99%  180.00ms
  366,234 requests in 30.00s, 1.23GB read
Requests/sec:  12,207.80
Transfer/sec:     42.05MB
```

---

## 🔍 Observability

### Monitoring Stack

```mermaid
graph TB
    subgraph COLLECTION["Data Collection"]
        LOGS["Logs<br/>(Fluentd)"]
        METRICS["Metrics<br/>(Prometheus)"]
        TRACES["Traces<br/>(Jaeger)"]
    end

    subgraph STORAGE["Storage"]
        LOKI["Loki<br/>(Log Storage)"]
        PROM["Prometheus<br/>(Metrics Storage)"]
        TEMPO["Tempo<br/>(Trace Storage)"]
    end

    subgraph VISUALIZATION["Visualization"]
        GRAFANA["Grafana<br/>(Dashboards)"]
        ALERTS["AlertManager<br/>(Alerts)"]
    end

    LOGS --> LOKI
    METRICS --> PROM
    TRACES --> TEMPO

    LOKI --> GRAFANA
    PROM --> GRAFANA
    TEMPO --> GRAFANA

    PROM --> ALERTS
    ALERTS --> SLACK["Slack"]
    ALERTS --> PAGER["PagerDuty"]

    style COLLECTION fill:#f0f9ff,stroke:#00F0FF,stroke-width:3px
    style STORAGE fill:#f3fff6,stroke:#12b886,stroke-width:3px
    style VISUALIZATION fill:#fffbeb,stroke:#f59e0b,stroke-width:3px
```

### Key Dashboards

1. **System Health Dashboard**
   - Service uptime
   - Error rates
   - Latency percentiles
   - Resource utilization

2. **Agent Performance Dashboard**
   - Active agents
   - Conversation metrics
   - Tool usage statistics
   - Response quality scores

3. **Business Metrics Dashboard**
   - New agent creations
   - Marketplace transactions
   - User engagement
   - Revenue metrics

---

## 📚 Additional Resources

- [API Documentation](./API.md)
- [Security Guidelines](./SECURITY.md)
- [Deployment Guide](./DEPLOYMENT.md)
- [Contributing Guide](./CONTRIBUTING.md)

---

## 📄 Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2025-10-07 | Architecture Team | Initial architecture documentation |

