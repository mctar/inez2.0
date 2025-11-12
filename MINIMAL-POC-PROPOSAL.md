# Minimal PoC Proposal: INEZ 2.0 Text-First Workshop Assistant

**Version:** 1.0
**Date:** 2025-11-12
**Scope:** 2-3 weeks
**Goal:** Validate core concept before investing in full Phase 1

---

## Executive Summary

This proposal strips the full INEZ 2.0 vision down to its absolute essence: **a text-based AI assistant grounded in Ingunn's theories that can participate in workshop discussions**. By deferring voice, Teams integration, and multi-agent complexity, we can validate the core value proposition in 2-3 weeks instead of 8-10 weeks.

**Key Decision:** Build text-first, add voice later (if PoC succeeds).

---

## What We're Building

### User Experience

1. **Facilitator** opens web app, creates a new workshop session
2. **Participants** (3-5 people) join via shared link
3. **Everyone types** messages in a shared chat (like Slack/Discord)
4. **Users tag AI** when they want input: `@Inez what does your research say about team dynamics?`
5. **AI responds** with answer + citations from Ingunn's work
6. **At end**, facilitator clicks "End Session" → AI generates summary document

### Example Interaction

```
[Facilitator joins: "Sarah"]
Sarah: Welcome everyone! Today we're discussing how to improve collaboration in our team.

[Participant joins: "Marcus"]
Marcus: I think our main issue is lack of trust between departments.

[Participant joins: "Emma"]
Emma: Agree, but also we don't have clear communication channels.

Sarah: @Inez what does research say about building trust in teams?

Inez: Based on organizational research, trust in teams develops through three key mechanisms:

1. **Consistency in actions** - When team members reliably follow through on commitments [1]
2. **Psychological safety** - Creating environments where people feel safe to take risks [2]
3. **Shared experiences** - Building common ground through collaborative work [1]

In your context, you might start by establishing small, achievable commitments that
demonstrate reliability across departments.

[1] "Building Collaborative Organizations" (Chapter 3, p. 47)
[2] "Trust and Innovation in Teams" (Article, 2023)

Marcus: That's helpful. The "small commitments" idea resonates.

[... conversation continues ...]

Sarah: @Inez can you suggest what we should discuss next?

Inez: Based on the conversation, you've identified trust as a core issue.
I'd suggest exploring:

1. What specific "small commitments" could your departments make to each other?
2. How might you measure whether trust is improving over time?
3. Who would be accountable for fostering these cross-department relationships?

Would any of these directions be useful?

Emma: Let's tackle #1 - specific commitments.

[... conversation continues ...]

Sarah: [Clicks "End Session"]

Inez: [Generates summary document with themes, decisions, action items]
```

---

## What We're NOT Building (Yet)

| Feature | Status | Rationale |
|---------|--------|-----------|
| Voice/Audio | **Phase 2** | Adds transcription, TTS, interruption detection complexity |
| Teams Integration | **Phase 2** | Adds Bot Framework, enterprise auth overhead |
| Multi-Agent Architecture | **Phase 2** | Can achieve core value with single GPT-4o instance |
| Speaker Diarization | **Phase 2** | Users manually select their name (dropdown) |
| Real-time Streaming | **Phase 2** | Simple request/response is fine for text |
| Proactive Interruptions | **Phase 2** | AI only speaks when @mentioned |
| Enterprise Security | **Phase 3** | Basic username/password, defer Azure AD integration |
| Advanced Analytics | **Phase 3** | Focus on core workshop experience first |

---

## Technical Architecture (Minimal)

### System Diagram

```
┌─────────────────────────────────────────┐
│         Web Browser (React/Streamlit)    │
│  - Chat interface                        │
│  - Session management                    │
│  - Export controls                       │
└──────────────────┬──────────────────────┘
                   │ HTTPS
                   ↓
┌─────────────────────────────────────────┐
│        FastAPI Backend (Python)          │
│  - Session state management              │
│  - Message routing                       │
│  - RAG orchestration                     │
│  - Summary generation                    │
└──────────────────┬──────────────────────┘
                   │
      ┌────────────┼────────────┐
      │            │            │
      ↓            ↓            ↓
┌──────────┐ ┌──────────┐ ┌──────────┐
│ SQLite   │ │ Azure    │ │ Azure AI │
│          │ │ OpenAI   │ │ Search   │
│ - Sessions│ │          │ │          │
│ - Messages│ │ - GPT-4o │ │ - Vector │
│ - Users  │ │ - Embed  │ │   Store  │
└──────────┘ └──────────┘ └──────────┘
```

### Tech Stack

**Frontend:**
- **Option A (Fast):** Streamlit (pure Python, can build in hours)
- **Option B (Better UX):** React + TypeScript (if web dev available)

**Backend:**
- **Framework:** FastAPI (Python 3.11+)
- **LLM:** Azure OpenAI GPT-4o
- **Embeddings:** text-embedding-3-large
- **Vector DB:** Azure AI Search (Basic tier, $100/month) or FAISS (in-memory, free)
- **Database:** SQLite (file-based, upgrade to Cosmos later)
- **Document Processing:** LangChain or LlamaIndex for RAG pipeline

**Infrastructure:**
- **Deployment:** Azure App Service (single instance, ~$50/month)
- **Storage:** Local filesystem (upgrade to Blob Storage later)
- **Monitoring:** Basic logging to console (Application Insights later)

**Total Monthly Cost:** ~$100-150 (vs $1,000 for full Phase 1)

### Data Models

**Session**
```python
{
  "session_id": "uuid",
  "title": "Team Collaboration Workshop",
  "created_at": "2025-11-12T10:00:00Z",
  "ended_at": null,
  "facilitator": "Sarah"
}
```

**Message**
```python
{
  "message_id": "uuid",
  "session_id": "uuid",
  "timestamp": "2025-11-12T10:15:23Z",
  "speaker": "Marcus",  # or "Inez" for AI
  "text": "I think our main issue is lack of trust...",
  "citations": []  # populated for AI messages
}
```

**Knowledge Source**
```python
{
  "doc_id": "uuid",
  "title": "Building Collaborative Organizations",
  "type": "book_chapter",
  "content": "...",
  "embedding": [0.123, 0.456, ...]  # vector
}
```

---

## Implementation Plan (2-3 Weeks)

### Week 1: RAG Foundation

**Day 1-2: Setup & Infrastructure**
- [ ] Create Azure resource group
- [ ] Provision Azure OpenAI resource
- [ ] Set up Azure AI Search (or local FAISS)
- [ ] Create Python project structure
- [ ] Set up FastAPI with basic endpoints

**Day 3-4: Document Ingestion**
- [ ] Collect 3-5 key documents from Ingunn (PDFs, articles)
- [ ] Build ingestion pipeline: PDF → chunks → embeddings → vector store
- [ ] Test retrieval: query → relevant chunks
- [ ] Implement citation formatting

**Day 5: RAG Chat (Single User)**
- [ ] Build `/ask` endpoint: question → RAG → GPT-4o → response
- [ ] Implement simple prompt template
- [ ] Test with sample questions about theories
- [ ] Basic Streamlit UI (single text box + response)

**Deliverable:** Demo video of asking Inez questions about theories

---

### Week 2: Multi-User Sessions

**Day 6-7: Session Management**
- [ ] SQLite schema for sessions, messages, users
- [ ] `/create-session` endpoint
- [ ] `/join-session/:id` endpoint
- [ ] `/send-message` endpoint (stores message, broadcasts to session)
- [ ] Basic authentication (username only, no password yet)

**Day 8-9: Multi-User Chat UI**
- [ ] Chat interface with message history
- [ ] User selector dropdown ("Posting as: [Name]")
- [ ] Real-time updates (polling every 2s, or WebSocket if time permits)
- [ ] @Inez mention detection

**Day 10: Context-Aware AI Responses**
- [ ] Modify RAG to include recent conversation history
- [ ] Prompt engineering: "You are facilitating a workshop. Conversation so far: ..."
- [ ] Test with multi-turn conversation

**Deliverable:** 3 people can have a conversation with AI via web interface

---

### Week 3: Summarization & Polish

**Day 11-12: Summarization**
- [ ] `/end-session/:id` endpoint
- [ ] Summary generation prompt (extract themes, decisions, action items)
- [ ] Export to Markdown
- [ ] Export to PDF (using markdown-pdf library)

**Day 13-14: Testing & Refinement**
- [ ] Internal test with real workshop scenario (30 minutes)
- [ ] Collect feedback on AI response quality
- [ ] Tune prompts and retrieval parameters
- [ ] Fix bugs and UX issues

**Day 15: Demo Preparation**
- [ ] Create demo script
- [ ] Prepare sample workshop scenario
- [ ] Record demo video
- [ ] Document setup instructions

**Deliverable:** Working system + demo for stakeholders

---

## Sample Code Snippets

### RAG Endpoint (FastAPI)

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from openai import AzureOpenAI
from azure.search.documents import SearchClient

app = FastAPI()
openai_client = AzureOpenAI(...)
search_client = SearchClient(...)

class Question(BaseModel):
    text: str
    session_id: str

@app.post("/ask")
async def ask_inez(question: Question):
    # 1. Retrieve relevant documents
    results = search_client.search(
        search_text=question.text,
        top=3,
        select=["title", "content", "page"]
    )

    context_chunks = [
        f"[{r['title']}, p.{r['page']}]: {r['content']}"
        for r in results
    ]

    # 2. Build prompt with context
    prompt = f"""You are Inez, an AI assistant grounded in organizational research.

Context from research:
{chr(10).join(context_chunks)}

Conversation history:
{get_recent_messages(question.session_id)}

User question: {question.text}

Provide a helpful, concise response (2-4 sentences) with citations."""

    # 3. Generate response
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.7,
        max_tokens=200
    )

    return {
        "response": response.choices[0].message.content,
        "citations": [r['title'] for r in results]
    }
```

### Streamlit UI (Option A)

```python
import streamlit as st
import requests

st.title("INEZ Workshop Assistant")

session_id = st.text_input("Session ID", "demo-session")
user_name = st.selectbox("Your name", ["Sarah", "Marcus", "Emma"])

# Display conversation
messages = requests.get(f"http://localhost:8000/messages/{session_id}").json()
for msg in messages:
    st.chat_message(msg['speaker']).write(msg['text'])

# Input
user_input = st.chat_input("Type your message...")
if user_input:
    if "@Inez" in user_input:
        # AI response
        response = requests.post("http://localhost:8000/ask", json={
            "text": user_input.replace("@Inez", ""),
            "session_id": session_id
        }).json()
        st.chat_message("Inez").write(response['response'])
    else:
        # Regular message
        requests.post("http://localhost:8000/send-message", json={
            "session_id": session_id,
            "speaker": user_name,
            "text": user_input
        })
    st.rerun()
```

---

## Success Criteria

### Technical Validation
- [ ] AI responds to questions in <3 seconds
- [ ] Retrieval finds relevant content 80%+ of the time
- [ ] Citations are accurate and properly formatted
- [ ] System handles 5 concurrent users without crashing

### User Validation
- [ ] Stakeholders complete 30-minute workshop using system
- [ ] Feedback: "AI responses were relevant and helpful" (3+/5)
- [ ] Feedback: "Would use this in real workshop" (yes/no → 60%+ yes)
- [ ] Generated summary captures main discussion points (70%+ accuracy)

### Business Validation
- [ ] Decision to proceed with Phase 1 (voice + Teams)
- [ ] OR: Pivot based on learnings (different use case, different approach)
- [ ] OR: Pause project (concept doesn't resonate with users)

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| **Retrieval quality poor** | Start with high-quality, well-structured documents; tune chunk size; use hybrid search |
| **AI responses too long** | Strict prompt engineering (max tokens, explicit length instructions); use GPT-4o-mini for speed |
| **Not enough documents** | Focus on 3-5 key pieces initially; quality > quantity for PoC |
| **UI too basic** | Streamlit is fine for PoC; can rebuild in React later if validated |
| **Sessions don't persist** | SQLite is sufficient; can migrate to Cosmos DB if needed |

---

## Decision Points

After PoC completion, decide:

### Option 1: Full Speed Ahead
✅ PoC validated → Proceed with Phase 1 (add voice, Teams, multi-agent)
**Investment:** 8-10 weeks, ~$50K development cost

### Option 2: Iterate
⚠️ Concept promising but needs refinement → Build PoC 2.0 with specific improvements
**Investment:** 2-3 more weeks, <$10K

### Option 3: Pivot
⚠️ Different use case shows more promise → Reframe INEZ for different context
**Investment:** 1-2 weeks planning, then new PoC

### Option 4: Pause
❌ Core value not validated → Hold project until new insights emerge
**Investment:** $0, learnings documented

---

## Next Steps

1. **Stakeholder Review:** Share this proposal with Ingunn, Capgemini, Equinor, Forsvaret
2. **Document Collection:** Gather 3-5 key documents from Ingunn's work
3. **Team Formation:** Assign 1-2 developers (full-time for 3 weeks)
4. **Azure Setup:** Provision OpenAI and AI Search resources
5. **Kickoff Meeting:** Week of [TBD]

---

## Appendix: Comparison to Full PRD

| Capability | Full PRD (Phase 1) | Minimal PoC | Time Savings |
|------------|-------------------|-------------|--------------|
| Voice I/O | ✅ Weeks 5-6 | ❌ Deferred | -2 weeks |
| Teams Integration | ✅ Weeks 1-2 | ❌ Deferred | -2 weeks |
| Multi-Agent | ✅ Weeks 3-4 | ❌ Single Agent | -2 weeks |
| RAG + Chat | ✅ Weeks 3-4 | ✅ Week 1-2 | Same |
| Summarization | ✅ Weeks 7-8 | ✅ Week 3 | -4 weeks |
| Testing | ✅ Weeks 9-10 | ✅ Week 3 | -6 weeks |
| **TOTAL** | **8-10 weeks** | **2-3 weeks** | **-6 weeks** |

**Why This Works:**
- Text is 10x simpler than voice (no transcription, TTS, interruption logic)
- Web app is 5x simpler than Teams integration (no Bot Framework)
- Single agent is 3x simpler than orchestration (no MCP, no coordination)
- Still proves core value: "Can AI grounded in theories facilitate workshops?"

**Upgrade Path:**
If PoC succeeds, all deferred features can be added incrementally without rework.

---

**Ready to start? Let's build this in 3 weeks and validate the vision before investing in the full system.**
