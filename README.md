# NFA × AgentKit Implementation Playbook
**Date:** 2025-10-07 · **Stack:** GPT‑5 Realtime • AgentKit (Agents SDK) • ChatKit • BAP‑578 (NFA) • Supabase/VectorDB • IPFS

This playbook gives engineers everything needed to build **Non‑Fungible Agents (NFAs)** that run on **OpenAI’s AgentKit / Agents SDK**, present as **visual avatars**, execute tasks, and anchor identity + provenance on‑chain via **BAP‑578**, with **Proof of Prompt (PoP)** as the verifiability primitive.

---

## 0) Quick Links (assets in this repo)
- Architecture PNG (4K): [`diagrams/NFA_AgentKit_Architecture.png`]
- Workflows PNG (4K): [`diagrams/NFA_AgentKit_Workflows.png`]
- Connector Distribution PNG (4K): [`diagrams/NFA_AgentKit_Connector.png`]
- Templates:
  - [`templates/companion.json`]
  - [`templates/travel.json`]
  - [`templates/research.json`]
- Metadata Spec: [`metadata/spec_proof_of_prompt.json`]

> Paste the Mermaid blocks below into Notion/GitHub.

---

## 1) Full‑Stack Architecture (Mermaid)
flowchart LR
  subgraph UI[Client Layer (ChatKit + Avatar)]
    CK[ChatKit Embed (web/mobile)]
    AV[Visual Avatar (LiveKit/Hedra)\nVoice (ElevenLabs)]
    STUDIO[Drag-&-Drop Studio (Agent Builder UI)]
    CK --- AV
  end

  subgraph EDGE[Edge & Realtime]
    RTC[WebRTC Gateway]
    APIGW[API Gateway\nAuth • Rate-limit • Routing]
    RTC --> APIGW
  end

  subgraph AGENTS[Agent Orchestration]
    AK[OpenAI AgentKit\n(Agents SDK / Responses)]
    FLOWS[Agent Builder Graphs\n(nodes • skills • guardrails)]
    EVAL[Evals / Tracing / Telemetry]
    TEMP[Durable Orchestration\nTemporal / Workers]
    AK --- FLOWS
    FLOWS --- EVAL
    FLOWS --- TEMP
  end

  subgraph TOOLS[Tools & Connectors]
    REG[Connector Registry\n(GDrive • GitHub • Web • Supabase • Zendesk • Telegram ...)]
    FX[Function Tooling\nHTTP APIs • Webhooks • Code sandboxes]
    REG --- FX
  end

  subgraph STATE[State & Memory]
    VDB[Vector DB\n(memories • embeddings)]
    OBJ[IPFS/S3\n(assets • voices • avatars)]
    SQL[(Postgres/Supabase)\n(agent configs • sessions)]
    VDB --- SQL
    OBJ --- SQL
  end

  subgraph CHAIN[On-Chain Identity (BAP-578)]
    BAP[BAP-578 / BEP-007\n(NFA token)]
    META[On-chain Metadata\n(personaHash • capabilityGraph • versionHistory)]
    POP[Proof of Prompt\n(Merkle roots of prompt/response traces)]
    BAP --- META
    META --- POP
  end

  subgraph MKT[Marketplace & Commerce]
    NFA[NFA.xyz Marketplace\n(list • preview • buy/sell)]
    PAY[Payments & Royalties\n(split to creators/skill authors)]
    NFA --- PAY
  end

  CK --> RTC
  AV --> RTC
  STUDIO --> APIGW

  APIGW --> AK
  AK --> REG
  AK --> FX
  AK --> VDB
  AK --> SQL
  AK --> OBJ
  AK --> EVAL
  TEMP --> FX

  AK --> META
  NFA --> CK
  NFA --> BAP
  PAY --> BAP
  META --> NFA

---

## 2) Top 6 Workflows (with on‑chain writes)
flowchart TB
  subgraph W1[1) Persona Training → Minting]
    W1a[Conversation & Correction]
    W1b[Build Capability Graph]
    W1c[Snapshot Persona/Memory]
    W1d[[On-chain: personaHash • capabilityGraph • proofOfPromptRoot • version]]
    W1a --> W1b --> W1c --> W1d
  end

  subgraph W2[2) Skill Upgrade / Add-On Marketplace]
    W2a[Browse/Import Skill Module]
    W2b[Guardrail Sandbox + Trace Eval]
    W2c[Activate Skill]
    W2d[[On-chain: capabilityGraph+ • proofOfPromptRoot(update) • royaltyEvent]]
    W2a --> W2b --> W2c --> W2d
  end

  subgraph W3[3) Delegated Task Execution]
    W3a[Plan ➜ Tool Calls ➜ Checkpoints]
    W3b[Deliverable / Output]
    W3c[[On-chain: proofOfPromptRoot(session)]]
    W3a --> W3b --> W3c
  end

  subgraph W4[4) Avatar-Driven Experience]
    W4a[Realtime Session (voice/video)]
    W4b[Interrupt/Barge-in & Multimodal Tools]
    W4c[[On-chain: avatarConfigHash • voiceIdHash • proofOfPromptRoot(session)]]
    W4a --> W4b --> W4c
  end

  subgraph W5[5) Multi-Agent Collaboration]
    W5a[Decompose Task]
    W5b[Handoffs Between Specialists]
    W5c[Aggregate & Review]
    W5d[[On-chain: bundle.links • proofOfPromptRoot(merged)]]
    W5a --> W5b --> W5c --> W5d
  end

  subgraph W6[6) Evaluation / Audit / Versioning]
    W6a[Run Eval Suites + Scoring]
    W6b[Threshold Gate ➜ Rollback/Version Bump]
    W6c[[On-chain: evalRootHash • versionHistory[]]]
    W6a --> W6b --> W6c
  end

---

## 3) NFA as Connector (distribution via Agents SDK)
flowchart LR
  APP[OpenAI Agent / ChatGPT App] --> SDK[Agents SDK\n(Agent • Tool • Session)]
  SDK --> CONN[NFA Connector (MCP server)\ninvoke | getMemory | useSkill]
  CONN --> RUNTIME[NFA Runtime\nGPT-5 Realtime • Skills • Guardrails]
  RUNTIME --> STORE[(Supabase/Vector/IPFS)]
  RUNTIME --> CHAIN[[BAP-578\nmetadata + proofOfPromptRoot + royalties]]

  subgraph Registry[Connector Registry / App Listing]
    LIST[NFA Connector Manifest\n(name • version • schema • auth • royalties)]
  end
  LIST --- CONN

  SDK -->|discovery| Registry

---

## 4) API Checklists (per-lane, copy‑paste)

**All lanes:** persist **Proof of Prompt (PoP)** → per step hash → Merkle root → write `proofOfPromptRoot` to BAP‑578 metadata.

### 4.1 Persona Training → Minting
- **Inputs:** `user_id`, `persona_template_id`, `system_prompt`, `memory_delta[]`, `skill_ids[]`, `eval_suite_id`
- **Nodes/Tools:** `ConversationLoop`, `CorrectionApply`, `SkillAttach`, `EvalRun`; tools: `memory.write`, `skills.attach`, `evals.run`
- **On‑chain:** `personaHash`, `capabilityGraph`, `proofOfPromptRoot`, `version`

### 4.2 Skill Upgrade / Add‑On Marketplace
- **Inputs:** `agent_id`, `skill_module_id@version`, `license_ref`, `sandbox_flag`
- **Nodes/Tools:** `SkillImport`, `GuardrailSandbox`, `TraceEval`, `Activate`; tools: `skills.install`, `guardrails.validate`
- **On‑chain:** `capabilityGraph+`, updated `proofOfPromptRoot`, `royaltyEvent`

### 4.3 Delegated Task Execution
- **Inputs:** `agent_id`, `task_type`, `payload`, `schedule?`
- **Nodes/Tools:** `Plan`, `ToolInvoke`, `Checkpoint`, `Deliver`; connectors: `web.search`, `github.commit`, `telegram.post`, etc.
- **On‑chain:** session `proofOfPromptRoot` (prompt/response + tool‑IO digests)

### 4.4 Avatar‑Driven Interactive Experience
- **Inputs:** `agent_id`, `session_token`, `rtc_offer`, `voice_id`, `avatar_cfg_id`
- **Nodes/Tools:** `RealtimeSession`, `BargeIn/Repair`, `Vision/ASR/TTS`
- **On‑chain:** `avatarConfigHash`, `voiceIdHash`, session `proofOfPromptRoot`

### 4.5 Multi‑Agent Collaboration (Team Agent)
- **Inputs:** `team_id`, `agent_ids[]`, `handoff_policy`, `task_spec`
- **Nodes/Tools:** `Decompose`, `Handoff`, `Aggregate`, `Review`
- **On‑chain:** `bundle.links`, merged `proofOfPromptRoot`

### 4.6 Evaluation / Audit / Versioning
- **Inputs:** `agent_id`, `eval_suite_id`, `schedule_cron`, `rollback_policy`
- **Nodes/Tools:** `EvalRun`, `Score`, `ThresholdGate`, `Rollback/Bump`
- **On‑chain:** `evalRootHash`, `versionHistory[]`

---

## 5) Starter AgentKit Templates (easy wins)

### 5.1 Companion Agent (realtime + memory)
{
  "name": "Companion",
  "model": "gpt-5-realtime-preview",
  "system": "Supportive, concise, safe. Remember key facts judiciously.",
  "nodes": [
    { "id":"realtime","type":"RealtimeSession" },
    { "id":"guard","type":"Guardrail","profile":"default_companion" },
    { "id":"memWrite","type":"Tool","ref":"memory.write" },
    { "id":"pop","type":"Hook","action":"persist_proof_of_prompt" }
  ],
  "edges": [
    ["realtime","guard"], ["guard","memWrite"], ["memWrite","pop"]
  ],
  "tools": {
    "memory.write": { "endpoint": "https://api.chatandbuild.com/memory/write" }
  }
}

### 5.2 Travel Booking Agent (search → plan → confirm)
{
  "name": "TravelPlanner",
  "model": "gpt-5",
  "system": "Plan options; always ask user to CONFIRM before booking; sandbox=true by default.",
  "nodes": [
    { "id":"gather","type":"Prompt","prompt":"Collect origin, destination, dates, budget, prefs." },
    { "id":"search","type":"Tool","ref":"travel.search" },
    { "id":"plan","type":"Reason" },
    { "id":"confirm","type":"Guardrail","policy":"require_token_CONFIRM" },
    { "id":"pay","type":"Tool","ref":"payments.charge","params":{"sandbox":true} },
    { "id":"pop","type":"Hook","action":"persist_proof_of_prompt" }
  ],
  "edges": [
    ["gather","search"], ["search","plan"], ["plan","confirm"], ["confirm","pay"], ["pay","pop"]
  ],
  "tools": {
    "travel.search": { "endpoint":"https://api.chatandbuild.com/travel/search" },
    "payments.charge": { "endpoint":"https://api.chatandbuild.com/payments/charge" }
  }
}

### 5.3 Research Agent (sources → cluster → summary)
{
  "name": "Researcher",
  "model": "gpt-5",
  "system": "Gather diverse sources, de-duplicate, provide balanced summary with citations.",
  "nodes": [
    { "id":"expand","type":"Prompt","prompt":"Derive targeted queries and synonyms." },
    { "id":"search","type":"Tool","ref":"web.search" },
    { "id":"fetch","type":"Tool","ref":"web.fetch" },
    { "id":"cluster","type":"EmbeddingCluster" },
    { "id":"write","type":"Reason","style":"markdown_with_citations" },
    { "id":"export","type":"Tool","ref":"export.markdown" },
    { "id":"pop","type":"Hook","action":"persist_proof_of_prompt" }
  ],
  "edges": [
    ["expand","search"], ["search","fetch"], ["fetch","cluster"], ["cluster","write"], ["write","export"], ["export","pop"]
  ],
  "tools": {
    "web.search": { "endpoint":"https://api.chatandbuild.com/search" },
    "web.fetch": { "endpoint":"https://api.chatandbuild.com/fetch" },
    "export.markdown": { "endpoint":"https://api.chatandbuild.com/export/md" }
  }
}

> **Tip:** convert these JSONs into AgentKit’s native graph import format or recreate them in the visual Builder.

---

## 6) Proof of Prompt (PoP) – Mini Spec

- **Scope:** per session, per critical step; include `prompt`, `response_summary`, `tool_io_digest` (hash of tool inputs/outputs, not raw data).
- **Storage:** raw traces off‑chain (IPFS or encrypted S3). On‑chain stores **Merkle root only**.
- **BAP‑578 metadata keys:**
  - `proofOfPromptRoot`
  - `proofOfPromptUri` (optional pointer to trace bundle)
  - `personaHash`, `capabilityGraph`, `version`, `evalRootHash?`

**Off‑chain record example**
{
  "agentId": "NFA-888",
  "sessionId": "sess_7f1",
  "steps": [
    {
      "ts": "2025-10-07T17:05:11Z",
      "prompt": "Plan trip SFO→FCO, 4 days, budget $2k.",
      "response_summary": "Three itineraries; Option A under $1.9k.",
      "tool_io_digest": "0x9b1c...cafe"
    }
  ],
  "merkleRoot": "0xabc123...ff9",
  "hashAlgo": "sha256"
}

---

## 7) Definition of Done (per sample)

- **Companion:** realtime chat OK; memory writes verified; session PoP stored; avatar latency < 400ms p50.
- **Travel:** search → plan → confirm (sandbox charge); PoP root + itinerary export.
- **Research:** ≥ 8 sources → clustered → summary with citations; PoP + MD/PDF export.

---

## 8) Security, Compliance, and Ops

- **Auth:** wallet signature + connector tokens; per‑tool ACLs.
- **Guardrails:** sanitize inputs; confirmation tokens for sensitive ops (payments/DMs).
- **Durability:** Temporal workers for retries/idempotency.
- **Observability:** AgentKit traces + Grafana dashboards; error budgets by lane.
- **Economics:** `RoyaltyPaid` events; usage metering; creator splits for skills/modules.

---

## 9) Roadmap (suggested)
- **P1:** NFA Connector (MCP) prototype + Companion template live
- **P2:** Travel + Research templates; on‑chain PoP writes
- **P3:** Connector Registry submission + ChatGPT app listing
- **P4:** Skill Marketplace (royalties) + Eval leaderboards
- **P5:** Federation across partner ecosystems

---

### License / Notes
Internal use. Replace endpoints with your infra. Align with OpenAI Agents SDK updates as they ship.
