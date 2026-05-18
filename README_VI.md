# Regulatory Compliance Intelligence Agent

> Hệ thống chuyển đổi văn bản quy định từ tài liệu pháp lý tĩnh thành lớp trí tuệ tuân thủ có thể hành động được: nghĩa vụ, rủi ro, quy trình bị ảnh hưởng, đơn vị phụ trách, thời hạn và kế hoạch xử lý có dẫn chứng.

---

## 1. Product Vision

Thay vì chỉ xây một chatbot hỏi đáp quy định, hệ thống sẽ:

```text
Regulatory documents
→ Legal clause parsing
→ Obligation extraction
→ Risk classification
→ Business impact mapping
→ Evidence-backed action plan
→ Human review
```

Mục tiêu cuối cùng là giúp tổ chức hiểu nhanh:

```text
Quy định mới yêu cầu gì?
Ảnh hưởng đến quy trình nào?
Rủi ro tuân thủ ở mức nào?
Bộ phận nào cần xử lý?
Cần hành động gì trước deadline?
Bằng chứng nằm ở điều/khoản nào?
```

---

## 2. Hackathon Architecture

### 2.1 Scope

Trong phạm vi hackathon, mục tiêu là một **end-to-end functional MVP** chứng minh được workflow cốt lõi:

```text
Upload regulation PDF
→ Parse legal clauses (Vietnamese legal structure)
→ Extract obligations
→ Classify risks
→ Map to business impact
→ Generate action plan
→ Show evidence for human review
```

### 2.2 Architecture Diagram

```text
┌─────────────────────────────────────────────┐
│              Regulatory Document            │
│              PDF / DOCX Upload              │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│           Document Ingestion Layer           │
│ - Upload file                                │
│ - Extract text                               │
│ - Extract basic metadata                     │
│ - Store raw document                         │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│         Vietnamese Legal Document Parser     │
│ - Split by Phần/Chương/Mục/Điều/Khoản/Điểm │
│ - Preserve legal hierarchy                   │
│ - Store structured clauses                   │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│            Retrieval & Evidence Layer        │
│ - Clause-level search                        │
│ - Vector / keyword retrieval                 │
│ - Evidence citation                          │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│              LLM Agent Workflow              │
│                                             │
│  1. Obligation Extraction Agent              │
│  2. Risk Classification Agent                │
│  3. Business Impact Agent                    │
│  4. Action Recommendation Agent              │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│           AI Output Validation               │
│ - Structured JSON schema validation          │
│ - Evidence required for every obligation     │
│ - Confidence score                           │
│ - Human review gate                          │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│        Compliance Intelligence Store         │
│ - Documents, Clauses, Obligations            │
│ - Risks, Business impacts                    │
│ - Recommended actions                        │
│ - Evidence references                        │
└──────────────────────┬──────────────────────┘
                       ↓
┌─────────────────────────────────────────────┐
│         Review, Report & Graph UI            │
│ - Document dashboard                         │
│ - Obligation explorer                        │
│ - Impact report                              │
│ - Compliance graph visualization             │
│ - Evidence viewer                            │
│ - Approve / reject / edit                    │
└─────────────────────────────────────────────┘
```

---

### 2.3 Core Workflow

#### Step 1 — Document Upload

```json
{
  "document_id": "doc_001",
  "title": "Thông tư về xác minh khách hàng",
  "document_type": "circular",
  "domain": "banking",
  "source": "manual_upload",
  "status": "processing"
}
```

#### Step 2 — Vietnamese Legal Clause Parsing

Parser hỗ trợ cấu trúc văn bản pháp luật Việt Nam:

```text
Phần → Chương → Mục → Điều → Khoản → Điểm
```

Pattern:

```text
Điều 12. ...
1. ...
a) ...
b) ...
```

Output:

```json
{
  "clause_id": "clause_001",
  "document_id": "doc_001",
  "article": "Điều 12",
  "clause": "Khoản 2",
  "text": "Tổ chức tín dụng phải xác minh danh tính khách hàng trước khi mở tài khoản.",
  "parent_section": "Chương III - Nhận biết khách hàng"
}
```

Điểm quan trọng: mọi output sau này đều phải trace ngược lại được **Điều/Khoản/Điểm** gốc, không chỉ "chunk 14".

#### Step 3 — Obligation Extraction Agent

```json
{
  "obligation_id": "obl_001",
  "obligation": "Xác minh danh tính khách hàng trước khi mở tài khoản.",
  "obligation_type": "KYC",
  "mandatory_level": "must",
  "applicable_entity": "tổ chức tín dụng",
  "deadline": "trước khi mở tài khoản",
  "evidence": {
    "clause_id": "clause_001",
    "source_text": "Tổ chức tín dụng phải xác minh danh tính khách hàng trước khi mở tài khoản."
  }
}
```

#### Step 4 — Risk Classification Agent

```text
Low      → clarification, no operational change
Medium   → policy or process update needed
High     → workflow/system change needed
Critical → legal exposure, penalty, urgent deadline
```

```json
{
  "obligation_id": "obl_001",
  "risk_level": "high",
  "risk_reason": "Ảnh hưởng đến quy trình mở tài khoản, yêu cầu xác minh KYC trước khi tạo account.",
  "risk_dimensions": ["regulatory_compliance", "customer_onboarding", "operational_process"]
}
```

#### Step 5 — Business Impact Agent

Business process inventory (pre-built cho hackathon, configurable cho production):

```json
[
  {
    "process": "Customer Onboarding",
    "departments": ["Compliance", "Product", "Engineering"],
    "systems": ["Mobile App", "Core Banking", "KYC Service"]
  }
]
```

Output:

```json
{
  "obligation_id": "obl_001",
  "affected_processes": [
    {
      "process": "Customer Onboarding",
      "impact": "Phải chặn tạo tài khoản cho đến khi xác minh danh tính hoàn tất.",
      "affected_departments": ["Compliance", "Product", "Engineering"],
      "affected_systems": ["Mobile App", "Core Banking", "KYC Service"]
    }
  ]
}
```

#### Step 6 — Action Recommendation Agent

```json
{
  "obligation_id": "obl_001",
  "recommended_actions": [
    {
      "action": "Cập nhật workflow onboarding để yêu cầu KYC trước khi tạo tài khoản.",
      "owner": "Product + Engineering",
      "priority": "P0",
      "deadline": "Trước ngày hiệu lực quy định",
      "evidence_clause_id": "clause_001"
    },
    {
      "action": "Sửa đổi chính sách KYC nội bộ.",
      "owner": "Compliance",
      "priority": "P1",
      "deadline": "Trong 2 tuần",
      "evidence_clause_id": "clause_001"
    }
  ]
}
```

---

### 2.4 Change Detection (Must-have for demo impact)

Lightweight clause-level diff + LLM summarization:

```text
Upload old version + new version
→ Parse both into clauses
→ Match similar clauses
→ Detect added / removed / modified clauses
→ LLM summarize regulatory impact
```

Output:

```json
{
  "change_type": "modified",
  "old_requirement": "Xác minh danh tính đối với khách hàng rủi ro cao.",
  "new_requirement": "Xác minh danh tính đối với tất cả khách hàng trước khi mở tài khoản.",
  "impact_level": "high",
  "reason": "Phạm vi yêu cầu mở rộng từ khách hàng rủi ro cao sang tất cả khách hàng."
}
```

> Change Detection is feasible for hackathon if implemented as a lightweight clause-level diff plus LLM summarization. A production-grade semantic change detection engine should remain part of the post-hackathon roadmap.

---

### 2.5 Compliance Graph Visualization

Graph tối thiểu render bằng React Flow từ PostgreSQL data:

```text
Regulation
→ Clause
→ Obligation
→ Risk Level
→ Business Process
→ Department
→ Recommended Action
```

Không cần graph database. Generate nodes/edges từ relational data rồi render client-side.

---

### 2.6 AI Output Validation

Hackathon scope:

```text
- Structured JSON schema validation (Pydantic)
- Evidence citation required for every obligation
- Confidence score per extraction
- Human review status (pending / approved / rejected)
```

Production scope (post-hackathon):

```text
- Evaluation dataset + gold-set comparison
- Regression tests for prompt changes
- Prompt/model versioning
- Guardrails
- Model fallback
```

---

### 2.7 Hackathon Tech Stack

```text
Frontend:
- React + Vite (hoặc Next.js)
- TailwindCSS
- React Flow (graph visualization)

Backend:
- FastAPI
- Pydantic (structured output validation)
- SQLAlchemy + Alembic
- PostgreSQL + pgvector

AI Layer:
- OpenAI / Anthropic API
- Structured output (JSON mode)
- Prompt templates
- Evidence-aware extraction

Search:
- PostgreSQL full-text search
- pgvector semantic search

Async:
- FastAPI BackgroundTasks

Storage:
- Local file storage
```

### 2.8 Hackathon Features

**Must-have:**

```text
✅ Document upload + text extraction
✅ Vietnamese legal clause parsing (Điều/Khoản/Điểm)
✅ Obligation extraction with evidence
✅ Risk classification
✅ Business impact mapping
✅ Action recommendation
✅ Evidence citation
✅ AI output validation (schema + evidence + confidence)
✅ Impact report UI
✅ Human review status
```

**Must-have for demo impact:**

```text
✅ Change detection (clause-level diff + LLM summary)
✅ Compliance graph visualization (React Flow)
```

**Out of scope:**

```text
❌ OCR for scanned documents
❌ Auto-crawling law portals
❌ Full authentication / RBAC
❌ Kubernetes / cloud deployment
❌ Enterprise audit and governance
```

---

## 3. Production Architecture

Sau hackathon, hệ thống mở rộng thành **production-grade regulatory compliance platform**.

### 3.1 Production Architecture Diagram

```text
┌────────────────────────────────────────────────────────────┐
│                    Regulatory Sources                      │
│  Law portals | APIs | PDFs | Emails | Internal Policies     │
└───────────────────────────┬────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│              Regulatory Ingestion Platform                  │
│ - Manual upload + scheduled crawler                         │
│ - OCR for scanned documents                                 │
│ - Document validation + malware checks                      │
└───────────────────────────┬────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│              Document Processing Pipeline                   │
│ - Layout-aware parsing                                      │
│ - Vietnamese legal hierarchy parsing                        │
│ - Metadata extraction + language detection                  │
│ - Version hashing + change detection                        │
└───────────────────────────┬────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│              Retrieval & Knowledge Layer                    │
│ - Elasticsearch/OpenSearch BM25                             │
│ - Vector search + hybrid retrieval                          │
│ - Reranking + metadata filtering                            │
│ - Clause-level citation                                     │
└───────────────────────────┬────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│              AI Orchestration Layer                         │
│                                                            │
│  1. Obligation Extraction Agent                             │
│  2. Applicability Agent                                     │
│  3. Change Detection Agent                                  │
│  4. Risk Classification Agent                               │
│  5. Business Impact Agent                                   │
│  6. Control Mapping Agent                                   │
│  7. Action Recommendation Agent                             │
│  8. Report Generation Agent                                 │
│                                                            │
│ - Prompt versioning + structured output validation          │
│ - Guardrails + evaluation pipeline + model fallback         │
└───────────────────────────┬────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│              Compliance Knowledge Graph                     │
│                                                            │
│ Regulation → Clause → Obligation → Control                  │
│            → Risk → Process → System → Owner → Action        │
└───────────────────────────┬────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│              Workflow & Governance Layer                    │
│ - Maker-checker approval + RBAC                             │
│ - Immutable audit trail                                     │
│ - SLA tracking + regulatory calendar                        │
│ - Task assignment + exception management                    │
└───────────────────────────┬────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│              Enterprise Integrations                        │
│ - Jira / ServiceNow / Slack / Teams                         │
│ - GRC systems / BI dashboards                               │
└────────────────────────────────────────────────────────────┘
```

### 3.2 Production Tech Stack

```text
Frontend:    Next.js + TailwindCSS + React Flow + TanStack Query
Backend:     FastAPI + SQLAlchemy + Celery + Redis
Database:    PostgreSQL + pgvector → Elasticsearch/OpenSearch
Storage:     S3 / GCS for raw documents
AI:          LLM gateway + prompt registry + evaluation pipeline
Security:    OAuth/SSO + RBAC + encryption at rest + audit log
Deployment:  Docker → Cloud Run / ECS / Kubernetes
Observability: OpenTelemetry + Prometheus + Grafana + Sentry
```

### 3.3 Business Process Inventory

- **Hackathon**: Pre-built inventory cho demo (banking domain).
- **Production**: Organizations configure their own process inventory, departments, systems, controls, and owners.

---

## 4. Hackathon vs Production

| Area | Hackathon MVP | Production |
|------|---------------|------------|
| Data source | Manual upload | Upload + crawler + APIs |
| Document type | Digital PDF/DOCX | PDF/DOCX/HTML/scanned/email |
| Parsing | Vietnamese legal hierarchy | Layout-aware + OCR + validation |
| Search | pgvector + full-text | Elasticsearch + vector + reranker |
| AI workflow | 4 core agents | 8 agents + validation + fallback + eval |
| Validation | Schema + evidence + human review | + evaluation dataset + regression tests |
| Storage | PostgreSQL | PostgreSQL + object storage + optional graph DB |
| Graph | React Flow from relational data | Full compliance knowledge graph |
| Review | Basic approve/reject | Maker-checker, SLA, exception management |
| Process inventory | Pre-built | Organization-configurable |
| Security | Minimal | SSO, RBAC, encryption, governance |
| Deployment | Docker Compose | Cloud-native |

---

## 5. Expansion Roadmap

### Phase 1 — Hackathon MVP

```text
✅ Vietnamese legal clause parsing
✅ Obligation extraction with evidence
✅ Risk classification
✅ Business impact mapping
✅ Action recommendation
✅ Change detection (lightweight)
✅ Graph visualization
✅ Human review UI
```

### Phase 2 — Post-hackathon Hardening

```text
Document versioning
Better retrieval (hybrid search)
Prompt versioning + evaluation dataset
Basic audit log
Export report
Background job processing
```

### Phase 3 — Early Production

```text
Authentication + RBAC
Elasticsearch/OpenSearch
Queue-based processing (Celery)
Human review workflow
Task assignment + notifications
Monitoring + error handling
Confidence scoring
```

### Phase 4 — Enterprise

```text
Regulatory source connectors + scheduled monitoring
OCR + multi-document reasoning
GRC integration + Jira/ServiceNow
Maker-checker approval + immutable audit logs
SLA tracking + regulatory calendar
Control mapping + advanced analytics
```

---

## 6. Repo Structure

```text
regulatory-compliance-agent/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── agents/
│   │   │   ├── obligation_extractor.py
│   │   │   ├── change_detector.py
│   │   │   ├── risk_classifier.py
│   │   │   ├── impact_analyzer.py
│   │   │   └── action_recommender.py
│   │   ├── parsers/
│   │   │   └── vietnamese_legal_parser.py
│   │   ├── retrieval/
│   │   ├── schemas/
│   │   ├── services/
│   │   ├── models/
│   │   └── main.py
│   ├── alembic/
│   └── pyproject.toml
├── frontend/
├── infra/
│   └── docker-compose.yml
├── data/
│   ├── sample_regulations/
│   └── business_process_inventory.json
├── docs/
│   └── architecture.md
├── LICENSE
└── README.md
```

---

## 7. Positioning

**Hackathon:**

> We focus on an end-to-end compliance intelligence MVP: users upload a regulatory document, the system parses Vietnamese legal clauses, extracts obligations, assesses risks, maps business impact, and generates evidence-backed action plans for human review.

**Production vision:**

> This architecture extends into a production-grade regulatory compliance platform with automated source monitoring, document versioning, semantic change detection, hybrid retrieval, compliance knowledge graph, human approval workflow, audit trail, and enterprise integrations.

**One-line:**

> The hackathon version proves the intelligence workflow; the production version turns it into a governed, auditable, and enterprise-ready compliance operating system.
