# INEZ 2.0 - Implementation Quick Start Guide

## Overview
This guide provides the immediate first steps to begin implementing the INEZ 2.0 AI Collaboration System based on the PRD architecture.

---

## Phase 1 Sprint 1: Foundation (Week 1-2)

### Prerequisites Checklist
- [ ] Azure subscription with Owner/Contributor access
- [ ] Azure OpenAI Service access (requires application approval)
- [ ] Microsoft Teams admin access for bot registration
- [ ] GitHub/Azure DevOps repository setup
- [ ] Development team members added with appropriate permissions

---

## Day 1-2: Azure Environment Setup

### 1. Create Resource Groups
```bash
# Set variables
RESOURCE_GROUP="rg-inez-prod"
LOCATION="norwayeast"
ENV="production"

# Create resource group
az group create --name $RESOURCE_GROUP --location $LOCATION

# Tag for organization
az group update --name $RESOURCE_GROUP \
  --tags environment=$ENV project=inez2.0 owner=capgemini
```

### 2. Deploy Core Azure Services

#### Azure OpenAI
```bash
# Create Azure OpenAI resource
az cognitiveservices account create \
  --name inez-openai-prod \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --kind OpenAI \
  --sku S0 \
  --custom-domain inez-openai-prod

# Deploy models (adjust based on availability)
# GPT-4o for reasoning
az cognitiveservices account deployment create \
  --resource-group $RESOURCE_GROUP \
  --name inez-openai-prod \
  --deployment-name gpt-4o \
  --model-name gpt-4o \
  --model-version "2024-05-13" \
  --model-format OpenAI \
  --sku-capacity 10 \
  --sku-name "Standard"

# GPT-4o-mini for fast responses
az cognitiveservices account deployment create \
  --resource-group $RESOURCE_GROUP \
  --name inez-openai-prod \
  --deployment-name gpt-4o-mini \
  --model-name gpt-4o-mini \
  --model-version "2024-07-18" \
  --model-format OpenAI \
  --sku-capacity 10 \
  --sku-name "Standard"

# Whisper for transcription
az cognitiveservices account deployment create \
  --resource-group $RESOURCE_GROUP \
  --name inez-openai-prod \
  --deployment-name whisper \
  --model-name whisper \
  --model-version "001" \
  --model-format OpenAI \
  --sku-capacity 1 \
  --sku-name "Standard"

# TTS for voice output
az cognitiveservices account deployment create \
  --resource-group $RESOURCE_GROUP \
  --name inez-openai-prod \
  --deployment-name tts-1 \
  --model-name tts-1 \
  --model-version "001" \
  --model-format OpenAI \
  --sku-capacity 1 \
  --sku-name "Standard"

# text-embedding-3-large for RAG
az cognitiveservices account deployment create \
  --resource-group $RESOURCE_GROUP \
  --name inez-openai-prod \
  --deployment-name text-embedding-3-large \
  --model-name text-embedding-3-large \
  --model-version "1" \
  --model-format OpenAI \
  --sku-capacity 10 \
  --sku-name "Standard"
```

#### Azure AI Search (for RAG)
```bash
az search service create \
  --name inez-search-prod \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Basic \
  --partition-count 1 \
  --replica-count 1
```

#### Azure Cosmos DB (for state management)
```bash
az cosmosdb create \
  --name inez-cosmos-prod \
  --resource-group $RESOURCE_GROUP \
  --locations regionName=$LOCATION failoverPriority=0 \
  --capabilities EnableServerless \
  --default-consistency-level Session

# Create database and container
az cosmosdb sql database create \
  --account-name inez-cosmos-prod \
  --resource-group $RESOURCE_GROUP \
  --name inez-db

az cosmosdb sql container create \
  --account-name inez-cosmos-prod \
  --resource-group $RESOURCE_GROUP \
  --database-name inez-db \
  --name conversation-sessions \
  --partition-key-path "/session_id" \
  --throughput 400
```

#### Azure Storage
```bash
az storage account create \
  --name inezstorageprod \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2

# Create containers
az storage container create --name audio-recordings --account-name inezstorageprod
az storage container create --name knowledge-base --account-name inezstorageprod
az storage container create --name generated-reports --account-name inezstorageprod
```

#### Azure Key Vault
```bash
az keyvault create \
  --name inez-keyvault-prod \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --enable-rbac-authorization false

# Store secrets (retrieve from Azure Portal)
OPENAI_KEY=$(az cognitiveservices account keys list \
  --name inez-openai-prod \
  --resource-group $RESOURCE_GROUP \
  --query key1 -o tsv)

az keyvault secret set \
  --vault-name inez-keyvault-prod \
  --name "AzureOpenAI--ApiKey" \
  --value $OPENAI_KEY
```

---

## Day 3-4: Teams Bot Setup

### 1. Register Bot in Azure
```bash
# Create Azure Bot resource
az bot create \
  --resource-group $RESOURCE_GROUP \
  --name inez-bot-prod \
  --kind azurebot \
  --sku F0 \
  --location global
```

### 2. Configure Teams Channel
1. Go to Azure Portal → Bot Services → inez-bot-prod
2. Under "Channels", click "Microsoft Teams"
3. Enable "Calling" tab for voice support
4. Note the Bot ID and App Password for configuration

### 3. Create Teams App Manifest
Create `teams-manifest.json`:
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/teams/v1.16/MicrosoftTeams.schema.json",
  "manifestVersion": "1.16",
  "version": "1.0.0",
  "id": "{{BOT_APP_ID}}",
  "packageName": "com.capgemini.inez",
  "developer": {
    "name": "Capgemini",
    "websiteUrl": "https://www.capgemini.com",
    "privacyUrl": "https://www.capgemini.com/privacy",
    "termsOfUseUrl": "https://www.capgemini.com/terms"
  },
  "name": {
    "short": "INEZ AI Facilitator",
    "full": "INEZ 2.0 AI-Powered Workshop Facilitator"
  },
  "description": {
    "short": "AI assistant for collaborative workshops",
    "full": "INEZ is an AI-powered facilitator that helps guide workshops, provides insights based on Ingunn's theories, and generates comprehensive meeting documentation."
  },
  "icons": {
    "outline": "icon-outline.png",
    "color": "icon-color.png"
  },
  "accentColor": "#0078D4",
  "bots": [
    {
      "botId": "{{BOT_APP_ID}}",
      "scopes": ["team", "personal", "groupchat"],
      "supportsFiles": false,
      "isNotificationOnly": false,
      "supportsCalling": true,
      "supportsVideo": false
    }
  ],
  "permissions": [
    "identity",
    "messageTeamMembers"
  ],
  "validDomains": [
    "inez.azurewebsites.net"
  ]
}
```

---

## Day 5-7: Development Environment

### 1. Project Structure
```
inez2.0/
├── infrastructure/          # Terraform/ARM templates
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── orchestrator/            # Level 1: Conversation orchestrator
│   ├── src/
│   │   ├── main.py
│   │   ├── audio_processor.py
│   │   ├── transcription.py
│   │   ├── state_manager.py
│   │   └── agent_coordinator.py
│   ├── requirements.txt
│   └── Dockerfile
├── agents/                  # Level 2: MCP agents
│   ├── theory_expert/
│   │   ├── server.py
│   │   ├── rag_engine.py
│   │   └── mcp_config.json
│   ├── facilitator/
│   │   ├── server.py
│   │   ├── discussion_guide.py
│   │   └── mcp_config.json
│   └── summarization/
│       ├── server.py
│       ├── summary_generator.py
│       └── mcp_config.json
├── teams-integration/       # Teams bot interface
│   ├── src/
│   │   ├── bot.ts
│   │   ├── audioHandler.ts
│   │   └── teamsAdapter.ts
│   ├── package.json
│   └── Dockerfile
├── shared/                  # Shared utilities
│   ├── azure_clients.py
│   ├── mcp_protocol.py
│   └── config.py
└── knowledge_base/          # RAG content
    ├── ingestion/
    │   └── ingest_articles.py
    └── documents/
        └── ingunn_content/
```

### 2. Initialize Python Orchestrator
```bash
cd orchestrator

# Create virtual environment
python3.11 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
cat > requirements.txt <<EOF
fastapi==0.104.1
uvicorn[standard]==0.24.0
azure-cosmos==4.5.1
azure-storage-blob==12.19.0
azure-cognitiveservices-speech==1.34.0
openai==1.3.5
websockets==12.0
pydantic==2.5.0
python-dotenv==1.0.0
mcp==0.1.0  # Model Context Protocol SDK
EOF

pip install -r requirements.txt
```

### 3. Basic Orchestrator Implementation
Create `orchestrator/src/main.py`:
```python
from fastapi import FastAPI, WebSocket
from azure.identity import DefaultAzureCredential
from openai import AzureOpenAI
import os

app = FastAPI()

# Initialize Azure OpenAI client
credential = DefaultAzureCredential()
client = AzureOpenAI(
    api_key=os.getenv("AZURE_OPENAI_API_KEY"),
    api_version="2024-02-15-preview",
    azure_endpoint=os.getenv("AZURE_OPENAI_ENDPOINT")
)

@app.websocket("/ws/audio")
async def audio_stream(websocket: WebSocket):
    """Handle real-time audio streaming from Teams bot"""
    await websocket.accept()

    try:
        while True:
            # Receive audio chunk
            audio_data = await websocket.receive_bytes()

            # Transcribe using Whisper (streaming)
            # TODO: Implement streaming transcription

            # Process transcription
            # TODO: Implement conversation logic

            # Send response if needed
            # TODO: Implement response generation

    except Exception as e:
        print(f"Error: {e}")
    finally:
        await websocket.close()

@app.get("/health")
async def health_check():
    return {"status": "healthy", "service": "inez-orchestrator"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 4. Initialize Teams Bot (TypeScript)
```bash
cd teams-integration
npm init -y
npm install botbuilder botbuilder-core botframework-connector @microsoft/teams-js

# Create basic bot
cat > src/bot.ts <<EOF
import { ActivityHandler, TurnContext } from 'botbuilder';

export class InezBot extends ActivityHandler {
    constructor() {
        super();

        this.onMessage(async (context, next) => {
            const text = context.activity.text;
            console.log('Received message:', text);

            // Forward to orchestrator
            // TODO: Implement WebSocket connection to orchestrator

            await context.sendActivity('INEZ is listening...');
            await next();
        });

        this.onMembersAdded(async (context, next) => {
            const membersAdded = context.activity.membersAdded;
            for (let member of membersAdded) {
                if (member.id !== context.activity.recipient.id) {
                    await context.sendActivity('Hello! I am INEZ, your AI workshop facilitator.');
                }
            }
            await next();
        });
    }
}
EOF
```

---

## Day 8-10: RAG Knowledge Base Setup

### 1. Prepare Ingunn's Content
```bash
cd knowledge_base/documents/ingunn_content

# Organize documents
# - Article 1: [filename].pdf
# - Article 2: [filename].pdf
# - Book Chapter 1: [filename].pdf
# etc.
```

### 2. Create Ingestion Script
Create `knowledge_base/ingestion/ingest_articles.py`:
```python
import os
from pathlib import Path
from azure.search.documents import SearchClient
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import (
    SearchIndex,
    SimpleField,
    SearchableField,
    VectorSearch,
    VectorSearchProfile,
    HnswAlgorithmConfiguration
)
from azure.core.credentials import AzureKeyCredential
from openai import AzureOpenAI
import PyPDF2

# Azure AI Search setup
search_endpoint = os.getenv("AZURE_SEARCH_ENDPOINT")
search_key = os.getenv("AZURE_SEARCH_KEY")
index_name = "ingunn-knowledge-base"

# Azure OpenAI setup for embeddings
openai_client = AzureOpenAI(
    api_key=os.getenv("AZURE_OPENAI_API_KEY"),
    api_version="2024-02-01",
    azure_endpoint=os.getenv("AZURE_OPENAI_ENDPOINT")
)

def create_search_index():
    """Create vector search index"""
    index_client = SearchIndexClient(search_endpoint, AzureKeyCredential(search_key))

    fields = [
        SimpleField(name="id", type="Edm.String", key=True),
        SearchableField(name="content", type="Edm.String"),
        SearchableField(name="title", type="Edm.String"),
        SimpleField(name="source_file", type="Edm.String"),
        SimpleField(name="page_number", type="Edm.Int32"),
        SearchableField(name="content_vector", type="Collection(Edm.Single)",
                       vector_search_dimensions=3072, vector_search_profile_name="vector-profile")
    ]

    vector_search = VectorSearch(
        profiles=[VectorSearchProfile(name="vector-profile", algorithm_configuration_name="hnsw-config")],
        algorithms=[HnswAlgorithmConfiguration(name="hnsw-config")]
    )

    index = SearchIndex(name=index_name, fields=fields, vector_search=vector_search)
    index_client.create_or_update_index(index)
    print(f"Created index: {index_name}")

def extract_text_from_pdf(pdf_path):
    """Extract text from PDF file"""
    with open(pdf_path, 'rb') as file:
        reader = PyPDF2.PdfReader(file)
        text_by_page = []
        for page_num, page in enumerate(reader.pages):
            text = page.extract_text()
            text_by_page.append((page_num + 1, text))
        return text_by_page

def generate_embedding(text):
    """Generate embedding using Azure OpenAI"""
    response = openai_client.embeddings.create(
        model="text-embedding-3-large",
        input=text
    )
    return response.data[0].embedding

def ingest_documents(documents_dir):
    """Ingest all PDFs from directory"""
    search_client = SearchClient(search_endpoint, index_name, AzureKeyCredential(search_key))

    pdf_files = Path(documents_dir).glob("*.pdf")

    for pdf_path in pdf_files:
        print(f"Processing: {pdf_path.name}")
        pages = extract_text_from_pdf(pdf_path)

        for page_num, text in pages:
            if len(text.strip()) < 50:  # Skip nearly empty pages
                continue

            # Generate embedding
            embedding = generate_embedding(text)

            # Create document
            doc = {
                "id": f"{pdf_path.stem}_page_{page_num}",
                "content": text,
                "title": pdf_path.stem.replace("_", " ").title(),
                "source_file": pdf_path.name,
                "page_number": page_num,
                "content_vector": embedding
            }

            # Upload to search index
            search_client.upload_documents(documents=[doc])
            print(f"  Indexed page {page_num}")

if __name__ == "__main__":
    create_search_index()
    ingest_documents("../documents/ingunn_content")
    print("Ingestion complete!")
```

### 3. Run Ingestion
```bash
cd knowledge_base/ingestion

# Set environment variables
export AZURE_SEARCH_ENDPOINT="https://inez-search-prod.search.windows.net"
export AZURE_SEARCH_KEY="<your-key>"
export AZURE_OPENAI_ENDPOINT="https://inez-openai-prod.openai.azure.com/"
export AZURE_OPENAI_API_KEY="<your-key>"

# Install dependencies
pip install azure-search-documents azure-identity openai PyPDF2

# Run ingestion
python ingest_articles.py
```

---

## Week 2: First Integration Test

### Test Scenario: Simple Q&A
1. User asks (via Teams): "What does Ingunn say about organizational change?"
2. Orchestrator receives text
3. Theory Expert Agent searches RAG
4. GPT-4o generates response based on retrieved content
5. Response sent back to user

### Validation Checklist
- [ ] Teams bot receives messages
- [ ] Orchestrator processes requests
- [ ] RAG search returns relevant content
- [ ] GPT-4o generates coherent response
- [ ] Response delivered back to Teams
- [ ] End-to-end latency measured

---

## Configuration Management

### Environment Variables
Create `.env` file (DO NOT commit):
```bash
# Azure OpenAI
AZURE_OPENAI_ENDPOINT=https://inez-openai-prod.openai.azure.com/
AZURE_OPENAI_API_KEY=<from-keyvault>
AZURE_OPENAI_DEPLOYMENT_GPT4O=gpt-4o
AZURE_OPENAI_DEPLOYMENT_GPT4O_MINI=gpt-4o-mini
AZURE_OPENAI_DEPLOYMENT_WHISPER=whisper
AZURE_OPENAI_DEPLOYMENT_TTS=tts-1
AZURE_OPENAI_DEPLOYMENT_EMBEDDING=text-embedding-3-large

# Azure AI Search
AZURE_SEARCH_ENDPOINT=https://inez-search-prod.search.windows.net
AZURE_SEARCH_KEY=<from-portal>
AZURE_SEARCH_INDEX=ingunn-knowledge-base

# Azure Cosmos DB
AZURE_COSMOS_ENDPOINT=https://inez-cosmos-prod.documents.azure.com:443/
AZURE_COSMOS_KEY=<from-portal>
AZURE_COSMOS_DATABASE=inez-db
AZURE_COSMOS_CONTAINER=conversation-sessions

# Azure Storage
AZURE_STORAGE_ACCOUNT=inezstorageprod
AZURE_STORAGE_KEY=<from-portal>

# Teams Bot
TEAMS_BOT_ID=<from-bot-registration>
TEAMS_BOT_PASSWORD=<from-bot-registration>

# Application
ENVIRONMENT=production
LOG_LEVEL=INFO
```

---

## Next Steps After Week 2

1. **Voice Integration:** Implement real-time Whisper streaming
2. **MCP Agents:** Build out Theory Expert, Facilitator, and Summarization agents
3. **Advanced Orchestration:** Interruption detection logic
4. **Testing:** Conduct first live workshop test
5. **Iteration:** Refine based on user feedback

---

## Support & Resources

### Documentation
- Azure OpenAI: https://learn.microsoft.com/azure/ai-services/openai/
- MCP Protocol: https://modelcontextprotocol.io
- Teams Bot Framework: https://learn.microsoft.com/azure/bot-service/

### Team Contacts
- **Technical Lead:** [Name] - [email]
- **Azure Support:** [Support contact]
- **Product Owner (Ingunn):** [email]

### Monitoring & Debugging
- **Application Insights:** [Portal link]
- **Log Analytics:** [Portal link]
- **Cost Management:** [Portal link]

---

## Troubleshooting

### Azure OpenAI Access Denied
- Ensure Azure OpenAI Service is approved for your subscription
- Apply at: https://aka.ms/oai/access

### Teams Bot Not Responding
- Check bot endpoint configuration in Azure Portal
- Verify messaging endpoint is publicly accessible
- Check application logs for errors

### RAG Returns No Results
- Verify index exists: `az search index show --name ingunn-knowledge-base`
- Check if documents were ingested successfully
- Test embedding generation manually

### High Latency
- Check region configuration (should be Norway East)
- Monitor Azure OpenAI quota and throttling
- Consider using GPT-4o-mini for non-critical responses

---

**Document Version:** 1.0
**Last Updated:** 2025-11-12
