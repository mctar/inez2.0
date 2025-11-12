# Product Requirements Document: AI-Powered Collaborative Workshop System (INEZ 2.0)

**Version:** 1.0
**Date:** 2025-11-12
**Author:** Architecture Team
**Stakeholders:** Ingunn (Product Owner), Capgemini, Equinor, Forsvaret (Norwegian Defense)

---

## Executive Summary

INEZ 2.0 is an AI-powered collaborative workshop facilitation system that enables natural, real-time conversations between multiple human participants and an intelligent AI system. The system acts as both an active participant and a facilitator, grounded in Ingunn's theoretical frameworks, capable of guiding discussions, providing insights, and generating comprehensive meeting documentation.

**Key Innovation:** Multi-level agent architecture where an orchestrator manages conversation flow while specialized agents provide domain expertise, summarization, and facilitation—all in real-time without disrupting natural human dialogue.

---

## 1. Strategic Context

### 1.1 Target Users
- **Primary:** Workshop facilitators and participants in enterprise settings (Equinor, Forsvaret)
- **Secondary:** AI researchers, organizational development consultants
- **Tertiary:** Capgemini clients seeking AI-augmented collaboration tools

### 1.2 Business Objectives
1. Create a POC that demonstrates enterprise-ready AI collaboration
2. Build a scalable foundation for deployment in highly regulated environments
3. Showcase Capgemini's AI innovation capabilities
4. Establish platform for testing and refining Ingunn's theories in practice

### 1.3 Success Metrics
- **Technical:** <500ms response latency, 95%+ transcription accuracy
- **User Experience:** 4+ rating on naturalness of conversation, <2 inappropriate interruptions per session
- **Enterprise Adoption:** Security audit approval from Equinor and Forsvaret
- **Business Value:** 50%+ reduction in post-workshop documentation time

---

## 2. Functional Requirements

### 2.1 Core Capabilities

#### FR-1: Multi-Party Voice Communication
- **FR-1.1:** Support 2-10 simultaneous human participants
- **FR-1.2:** Real-time speech-to-text transcription in Norwegian and English (expandable to other languages)
- **FR-1.3:** Text-to-speech output with natural prosody and appropriate pacing
- **FR-1.4:** Speaker identification and diarization (who said what)
- **FR-1.5:** Audio quality management with noise cancellation

#### FR-2: Intelligent Conversation Management
- **FR-2.1:** Continuous listening without interrupting active human dialogue
- **FR-2.2:** Detection of direct questions/requests to AI system
- **FR-2.3:** Identification of conversation stalling or need for intervention
- **FR-2.4:** Context-aware response generation (10-50 words default, expandable on request)
- **FR-2.5:** Multi-turn dialogue tracking with conversation state management

#### FR-3: Knowledge Grounding (RAG)
- **FR-3.1:** Integration of Ingunn's articles, book chapters, and theoretical frameworks
- **FR-3.2:** Context-aware retrieval based on conversation topics
- **FR-3.3:** Citation and reference management for AI responses
- **FR-3.4:** Version control for knowledge base updates
- **FR-3.5:** Configurable knowledge sources per workshop type

#### FR-4: Workshop Facilitation
- **FR-4.1:** Structured discussion guidance based on selected frameworks
- **FR-4.2:** Time management and agenda tracking
- **FR-4.3:** Proactive suggestions for next topics or activities
- **FR-4.4:** Synthesis of multiple viewpoints during discussion
- **FR-4.5:** Question generation to deepen dialogue

#### FR-5: Documentation & Summarization
- **FR-5.1:** Real-time note-taking during conversation
- **FR-5.2:** Post-session summary generation with key themes
- **FR-5.3:** Action item extraction with owner assignments
- **FR-5.4:** Decision log with rationales
- **FR-5.5:** Exportable reports in multiple formats (PDF, Word, Markdown)

### 2.2 User Experience Requirements

#### UX-1: Natural Conversation Flow
- Responses must feel conversational, not robotic
- AI should avoid dominating the conversation
- Support for casual interjections ("um," "uh," backtracking)
- Graceful handling of crosstalk and overlapping speech

#### UX-2: Microsoft Teams Integration (POC)
- Deploy as a Teams bot for Phase 1
- Leverage Teams' native audio handling and noise cancellation
- Integrate with Teams meeting infrastructure
- Use Teams permissions and authentication

#### UX-3: Response Time Management
- Real-time transcription display (<1s latency)
- Voice responses begin within 500ms of request
- Visual indicators for "AI is thinking" vs "AI is listening"
- Buffered audio for smooth playback without stuttering

---

## 3. Technical Architecture

### 3.1 Architecture Decision: Hybrid Custom + Azure

**Recommendation:** Build custom agent orchestration using MCP (Model Context Protocol) architecture on Azure infrastructure.

**Rationale:**
- ✅ **Enterprise Compliance:** Azure provides required certifications for Equinor and Forsvaret (ISO 27001, SOC 2, Norwegian data residency)
- ✅ **Flexibility:** Custom MCP architecture enables sophisticated multi-level agent coordination
- ✅ **Modern Standards:** MCP is emerging as industry standard for agent communication
- ✅ **Scalability:** Can scale from POC to production without architectural rework
- ✅ **Cost Efficiency:** Pay only for services used, avoid platform lock-in overhead
- ❌ **NOT MS Power Apps:** Insufficient flexibility for real-time multi-agent orchestration

### 3.2 System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Microsoft Teams Client                       │
│                  (User Interface & Audio I/O)                    │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│                    LEVEL 1: Orchestrator Layer                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         Conversation Orchestrator (MCP Server)           │  │
│  │  - Audio Stream Processing                               │  │
│  │  - Speaker Diarization                                    │  │
│  │  - Interruption Detection Logic                          │  │
│  │  - Context & State Management                            │  │
│  │  - Agent Coordination                                     │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│                   LEVEL 2: Specialized Agents                    │
│                        (MCP Servers)                             │
│                                                                   │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────────┐  │
│  │ Theory Expert  │  │  Facilitator   │  │  Summarization   │  │
│  │     Agent      │  │     Agent      │  │      Agent       │  │
│  │                │  │                │  │                  │  │
│  │ - RAG Access   │  │ - Discussion   │  │ - Note Taking    │  │
│  │ - Citations    │  │   Guidance     │  │ - Action Items   │  │
│  │ - Concept      │  │ - Time Mgmt    │  │ - Report Gen     │  │
│  │   Explanation  │  │ - Process      │  │                  │  │
│  └────────────────┘  └────────────────┘  └──────────────────┘  │
│                                                                   │
│  ┌────────────────┐  ┌────────────────┐                         │
│  │   Language     │  │    Analytics   │                         │
│  │     Agent      │  │      Agent     │                         │
│  │                │  │                │                         │
│  │ - Translation  │  │ - Sentiment    │                         │
│  │ - Multi-lang   │  │ - Engagement   │                         │
│  │   Detection    │  │   Metrics      │                         │
│  └────────────────┘  └────────────────┘                         │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Foundation Services Layer                     │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐    │
│  │ Azure OpenAI │  │   Azure AI   │  │   Azure Cosmos DB  │    │
│  │              │  │    Search    │  │                    │    │
│  │ - GPT-4o     │  │              │  │ - Session State    │    │
│  │ - Whisper    │  │ - Vector DB  │  │ - Conversation     │    │
│  │ - TTS        │  │ - RAG Index  │  │   History          │    │
│  └──────────────┘  └──────────────┘  └────────────────────┘    │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐    │
│  │  Azure Bot   │  │  Azure Event │  │   Azure Storage    │    │
│  │   Service    │  │     Hub      │  │                    │    │
│  │              │  │              │  │ - Audio Archives   │    │
│  │ - Teams      │  │ - Event      │  │ - Documents        │    │
│  │   Integration│  │   Streaming  │  │ - Knowledge Base   │    │
│  └──────────────┘  └──────────────┘  └────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### 3.3 Technology Stack

#### Core Infrastructure
- **Cloud Platform:** Azure (Norway East region for data residency)
- **Container Orchestration:** Azure Kubernetes Service (AKS)
- **API Gateway:** Azure API Management
- **Identity:** Azure Active Directory (Microsoft Entra ID)

#### AI/ML Services
- **LLM:** Azure OpenAI Service (GPT-4o for reasoning, GPT-4o-mini for quick responses)
- **Speech-to-Text:** Azure OpenAI Whisper (real-time streaming mode)
- **Text-to-Speech:** Azure OpenAI TTS (optimized for conversational voice)
- **Vector Database:** Azure AI Search with semantic ranking
- **Embedding Model:** text-embedding-3-large

#### Agent Framework
- **Protocol:** MCP (Model Context Protocol) for inter-agent communication
- **Orchestration:** Custom Python orchestrator with asyncio for real-time coordination
- **Agent Runtime:** FastAPI servers for each MCP agent
- **Message Bus:** Azure Event Hub for agent-to-agent messaging

#### Data Layer
- **State Management:** Azure Cosmos DB (session state, conversation context)
- **Document Storage:** Azure Blob Storage (audio recordings, generated reports)
- **Cache:** Azure Redis Cache (for low-latency agent responses)
- **Knowledge Base:** Azure AI Search (vector + keyword hybrid search)

#### Frontend/Integration
- **Teams Integration:** Azure Bot Framework + Teams Toolkit
- **Real-time Communication:** WebRTC for audio streaming
- **Admin Dashboard:** React + TypeScript (for configuration and monitoring)

#### Development & Operations
- **Languages:** Python 3.11+ (agents), TypeScript (Teams integration)
- **IaC:** Terraform for infrastructure provisioning
- **CI/CD:** Azure DevOps Pipelines
- **Monitoring:** Azure Application Insights, Log Analytics
- **Security:** Azure Key Vault, Managed Identities

### 3.4 Data Flow: Real-Time Conversation

```
1. Teams Meeting → Audio Stream → Azure Bot Service
                                         ↓
2. Bot Service → WebSocket → Orchestrator (streaming transcription)
                                         ↓
3. Orchestrator → Speaker Diarization + Context Tracking
                                         ↓
4. Parallel Processing:
   a) Theory Expert Agent: Check for relevant concepts in RAG
   b) Facilitator Agent: Monitor for stalls or need for guidance
   c) Summarization Agent: Update running notes
                                         ↓
5. Interruption Decision Logic:
   - Direct question detected? → Respond immediately
   - Conversation stalled? → Gentle intervention
   - Otherwise → Continue listening
                                         ↓
6. If Response Needed:
   a) Generate response (GPT-4o-mini for speed)
   b) Synthesize to speech (TTS)
   c) Stream audio back to Teams
                                         ↓
7. Continuous Loop: Return to step 1
```

### 3.5 Key Architectural Patterns

#### Pattern 1: Event-Driven Agent Communication
- Agents communicate via event bus (Azure Event Hub)
- Loose coupling enables independent scaling and deployment
- Events include: `transcript_available`, `question_detected`, `response_needed`, `summary_requested`

#### Pattern 2: Streaming First
- All audio/transcription uses streaming (not batch)
- Incremental processing enables sub-second latencies
- WebSocket connections maintain real-time bidirectional communication

#### Pattern 3: Hierarchical Context
- Orchestrator maintains global conversation context
- Each agent maintains specialized context (e.g., Theory Expert has RAG context)
- Context pruning to manage token limits (sliding window + summarization)

#### Pattern 4: Graceful Degradation
- If specialized agent fails, orchestrator provides basic response
- Queue backup for offline processing if real-time fails
- Fallback to text-only mode if audio fails

---

## 4. Security & Compliance

### 4.1 Enterprise Requirements

#### Data Residency
- All data stored in Azure Norway East region
- No data transfer outside Norwegian borders without explicit consent
- Compliance with Norwegian Data Protection Act (Personopplysningsloven)

#### Authentication & Authorization
- Azure AD integration (SSO for Equinor/Forsvaret)
- Role-based access control (RBAC):
  - **Participant:** Join sessions, view own transcripts
  - **Facilitator:** Create sessions, configure AI behavior, access all transcripts
  - **Admin:** System configuration, knowledge base management, analytics

#### Data Encryption
- TLS 1.3 for all data in transit
- AES-256 encryption for data at rest
- Azure Key Vault for credential management
- Encrypted audio streams (SRTP)

#### Audit & Monitoring
- Comprehensive logging of all AI decisions
- Audit trail for data access and modifications
- Compliance reporting dashboard
- Automatic PII detection and redaction options

### 4.2 Privacy Considerations
- Explicit opt-in for audio recording
- Automatic audio deletion after configurable period (default 90 days)
- Ability to exclude sensitive discussions from transcription
- Participant consent tracking

---

## 5. Phased Implementation Roadmap

### Phase 1: MVP/POC (8-10 weeks)

**Goal:** Demonstrate core concept with single-facilitator workshops

**Deliverables:**
1. **Week 1-2: Infrastructure & Foundation**
   - Azure environment setup (resource groups, networking, security)
   - Teams Bot registration and basic integration
   - Audio pipeline: Teams → Transcription → Storage
   - Basic orchestrator with simple state management

2. **Week 3-4: Single-Agent RAG System**
   - Ingest Ingunn's content into Azure AI Search
   - Implement Theory Expert Agent (MCP server)
   - Basic Q&A: user asks question → RAG retrieval → GPT-4 response
   - Text-based interaction first (no voice yet)

3. **Week 5-6: Voice Integration**
   - Real-time Whisper transcription integration
   - TTS integration for AI responses
   - Basic interruption detection (keyword-based: "Inez," "AI," etc.)
   - Short response optimization (10-50 word responses)

4. **Week 7-8: Summarization & Documentation**
   - Implement Summarization Agent
   - Post-session report generation (summary, key points, action items)
   - Export to PDF and Word formats
   - Basic facilitator dashboard for configuration

5. **Week 9-10: Testing & Refinement**
   - Internal testing with 3-5 workshops
   - Latency optimization (<500ms target)
   - UX refinement based on feedback
   - Security review and hardening

**Success Criteria:**
- ✓ Successful 1-hour workshop with 3-5 participants
- ✓ AI responds appropriately when asked direct questions
- ✓ Generated summary captures 80%+ of key discussion points
- ✓ Average response latency <1 second
- ✓ Positive qualitative feedback on naturalness

---

### Phase 2: Multi-Party & Advanced Orchestration (6-8 weeks)

**Goal:** Enable natural group conversations with intelligent facilitation

**Deliverables:**
1. **Week 11-12: Speaker Diarization**
   - Multi-speaker identification and labeling
   - Track who said what in transcripts
   - Context awareness of conversation flow between participants

2. **Week 13-14: Advanced Interruption Logic**
   - ML-based conversation stall detection
   - Sentiment analysis for identifying confusion or conflict
   - Proactive facilitation suggestions
   - Configurable interruption thresholds

3. **Week 15-16: Facilitator Agent**
   - Workshop structure guidance (agenda tracking)
   - Time management alerts
   - Topic transition suggestions
   - Group dynamic monitoring

4. **Week 17-18: Enhanced RAG & Multi-Agent Coordination**
   - Multi-agent reasoning (Theory + Facilitator collaborate)
   - Cross-referencing multiple sources in responses
   - Confidence scoring for AI suggestions
   - Expandable knowledge base (add documents on-the-fly)

**Success Criteria:**
- ✓ 5+ person workshops with natural conversation flow
- ✓ AI interrupts appropriately <2 times per hour when not directly asked
- ✓ AI proactively suggests next topics when conversation stalls
- ✓ Facilitator satisfaction rating 4+/5

---

### Phase 3: Enterprise Deployment (8-10 weeks)

**Goal:** Production-ready system for Equinor and Forsvaret

**Deliverables:**
1. **Week 19-20: Security Audit & Compliance**
   - Third-party security audit
   - Penetration testing
   - Compliance documentation (ISO 27001, SOC 2)
   - Data residency verification

2. **Week 21-22: Enterprise Features**
   - Multi-tenant architecture (separate environments per organization)
   - Custom knowledge bases per client
   - Advanced RBAC with organizational hierarchies
   - Integration with enterprise SSO systems

3. **Week 23-24: Analytics & Insights**
   - Workshop effectiveness metrics
   - AI performance dashboard
   - Conversation analytics (participation balance, topic coverage)
   - Continuous improvement recommendations

4. **Week 25-26: Scalability & Performance**
   - Load testing (10+ concurrent workshops)
   - Auto-scaling configuration
   - Geographic distribution (if needed)
   - DR/backup procedures

5. **Week 27-28: Pilot Programs**
   - Pilot with Equinor team (3-5 workshops)
   - Pilot with Forsvaret team (3-5 workshops)
   - Feedback collection and iteration
   - Training and documentation

**Success Criteria:**
- ✓ Security audit approval
- ✓ Support for 10+ concurrent workshops
- ✓ 99.5% uptime SLA
- ✓ Successful pilots with both partners
- ✓ Production deployment commitment from at least one partner

---

## 6. Non-Functional Requirements

### Performance
- **Latency:** <500ms from user question to AI voice response start
- **Transcription Accuracy:** >95% word error rate (WER)
- **Concurrency:** Support 10+ simultaneous workshops (Phase 3)
- **Scalability:** Linear scaling to 100+ workshops via AKS

### Reliability
- **Uptime:** 99.5% availability during business hours (Phase 3)
- **Failover:** <30 second recovery from Azure region failure
- **Data Durability:** 99.999999999% (11 nines) via Azure Storage

### Usability
- **Setup Time:** <5 minutes for facilitator to start a session
- **Learning Curve:** <30 minutes training for new facilitators
- **Accessibility:** WCAG 2.1 AA compliance for admin dashboard

### Maintainability
- **Deployment:** Zero-downtime updates via blue-green deployment
- **Monitoring:** Real-time alerting with <5 minute detection
- **Documentation:** Comprehensive API docs, runbooks, architecture diagrams

---

## 7. Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Real-time latency exceeds 500ms | High | Medium | Use GPT-4o-mini for quick responses; implement response caching; optimize network topology |
| Inappropriate AI interruptions frustrate users | High | High | Extensive testing with adjustable sensitivity; manual override button; learn from feedback |
| RAG retrieves irrelevant content | Medium | Medium | Hybrid search (vector + keyword); semantic reranking; relevance threshold tuning |
| Security audit failure blocks enterprise deployment | High | Low | Early security review; follow Azure best practices; hire compliance consultant |
| Teams API limitations constrain features | Medium | Medium | Maintain abstraction layer for multi-platform support; engage Microsoft support early |
| Cost overruns from Azure OpenAI usage | Medium | Medium | Implement token budgets; use cheaper models for simple tasks; monitor and alert |
| Multi-speaker diarization accuracy insufficient | Medium | High | Test multiple diarization services; fallback to manual labeling; set expectations |

---

## 8. Cost Estimation (Monthly, Phase 1 POC)

### Azure Services
- **Azure OpenAI:** ~$500 (assuming 10 workshops/month, 1hr each, GPT-4o + Whisper)
- **Azure AI Search:** ~$100 (Basic tier for RAG)
- **Azure Cosmos DB:** ~$50 (low traffic serverless)
- **Azure Bot Service:** ~$30 (standard tier)
- **Azure Storage:** ~$20 (audio + documents)
- **Azure Kubernetes Service:** ~$200 (small cluster, 2-3 nodes)
- **Monitoring & Misc:** ~$100

**Total Phase 1:** ~$1,000/month

### Phase 3 Production (estimated)
- **Scale:** ~$5,000-$10,000/month for 100+ workshops/month
- **Cost Optimization:** Reserved instances, commitment tiers, model caching

---

## 9. Success Metrics & KPIs

### Technical KPIs
- **Response Latency (p95):** <500ms
- **Transcription Accuracy:** >95% WER
- **System Uptime:** >99.5%
- **API Success Rate:** >99%

### User Experience KPIs
- **Facilitator Satisfaction:** >4/5 rating
- **Participant Satisfaction:** >3.5/5 rating
- **Naturalness Rating:** >4/5 ("felt like talking to a human")
- **Interruption Appropriateness:** <2 inappropriate interruptions per session

### Business KPIs
- **Documentation Time Saved:** >50% reduction vs manual note-taking
- **Workshop Effectiveness:** Participant feedback improves vs non-AI workshops
- **Adoption Rate:** >70% of pilot users continue after trial
- **Enterprise Deals:** 2+ production contracts within 6 months of Phase 3

---

## 10. Appendix

### A. Alternative Architectures Considered

#### Option 1: MS Power Apps + Power Virtual Agents
**Rejected because:**
- Insufficient real-time audio processing capabilities
- Limited control over multi-agent orchestration
- Cannot implement sophisticated interruption logic
- Higher long-term costs at scale

#### Option 2: Full Custom Stack (Non-Azure)
**Rejected because:**
- Longer time to compliance certification for Equinor/Forsvaret
- Higher operational overhead (manage own infrastructure)
- Less seamless Teams integration
- Higher risk for POC timeline

#### Option 3: Anthropic Claude + AWS
**Considered but deprioritized:**
- Excellent model quality, but less mature enterprise contracts in Norway
- Would require AWS compliance work (vs existing Azure relationships)
- Anthropic's real-time audio capabilities still emerging
- Could revisit for Phase 3 as multi-LLM strategy

### B. MCP Server Specifications

Each MCP server exposes:
- **Tools:** Functions the agent can perform (e.g., `search_knowledge_base`, `generate_summary`)
- **Resources:** Data the agent can access (e.g., conversation context, knowledge base)
- **Prompts:** Reusable prompt templates for consistency

Example: Theory Expert Agent MCP Interface
```json
{
  "tools": [
    {
      "name": "search_theory",
      "description": "Search Ingunn's theories for relevant concepts",
      "parameters": {
        "query": "string",
        "max_results": "integer"
      }
    },
    {
      "name": "explain_concept",
      "description": "Explain a concept from Ingunn's work",
      "parameters": {
        "concept_name": "string",
        "depth": "enum[brief, detailed]"
      }
    }
  ],
  "resources": [
    {
      "uri": "theory://articles/*",
      "name": "Ingunn's Published Articles"
    },
    {
      "uri": "theory://books/*",
      "name": "Book Chapters"
    }
  ]
}
```

### C. Conversation State Schema

```json
{
  "session_id": "uuid",
  "workshop_type": "string",
  "participants": [
    {
      "id": "uuid",
      "name": "string",
      "role": "facilitator|participant",
      "speaking_time": "integer (seconds)"
    }
  ],
  "conversation": [
    {
      "timestamp": "ISO8601",
      "speaker_id": "uuid",
      "text": "string",
      "audio_url": "string",
      "sentiment": "positive|neutral|negative",
      "topics": ["array of strings"]
    }
  ],
  "ai_context": {
    "active_topics": ["array"],
    "referenced_theories": ["array"],
    "pending_questions": ["array"],
    "conversation_phase": "opening|exploration|deepening|closing"
  },
  "summary": {
    "key_themes": ["array"],
    "decisions": ["array"],
    "action_items": [
      {
        "description": "string",
        "owner": "string",
        "due_date": "ISO8601"
      }
    ]
  }
}
```

### D. Glossary

- **RAG (Retrieval-Augmented Generation):** Technique for grounding LLM responses in specific documents
- **MCP (Model Context Protocol):** Open standard for connecting AI models to external tools and data
- **Speaker Diarization:** Process of identifying who spoke when in audio
- **WER (Word Error Rate):** Standard metric for transcription accuracy (lower is better)
- **TTS (Text-to-Speech):** AI-generated voice output
- **Orchestrator:** Central coordinator that manages multiple AI agents
- **Agent:** Autonomous AI component with specialized capabilities

---

## Document Control

**Version History:**
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-12 | Architecture Team | Initial PRD |

**Approvals:**
- [ ] Product Owner (Ingunn)
- [ ] Technical Lead
- [ ] Capgemini Stakeholder
- [ ] Security/Compliance Representative

**Next Steps:**
1. Review and approval by stakeholders
2. Refinement based on feedback
3. Kick-off meeting with development team
4. Infrastructure provisioning (Week 1)
