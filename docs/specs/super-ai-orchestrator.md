# مواصفات تقنية كاملة لمشروع «الذكاء الصناعي الخارق» (Super AI Orchestrator)

> **الغرض**: ملف واحد شامل تُمرّره إلى Codex ليولّد الكود والبنية الأولية (MVP) مع قابلية التوسع لاحقاً.

---

## 1) نظرة عامة

* **الفكرة**: نظام «عقل مركزي» ينسّق بين عدة وكلاء (Agents) ونماذج ذكاء صناعي ووحدات أدوات (Tools) لتنفيذ مهام متعددة الوسائط (نص/صوت/صورة/فيديو)، مع ذاكرة طويلة المدى وسجل قرار واضح.

* **نطاق الإصدار الأول (MVP)**:

  1. واجهة ويب بسيطة (Dashboard) لإرسال المهام ومتابعة التنفيذ.
  2. Orchestrator يستدعي وكلاء فرعيين: NLP, Vision, Audio, Data.
  3. ذاكرة (RAG) مبسّطة + قاعدة بيانات للمهام والجلسات والأحداث.
  4. تكامل صوتي أساسي (STT/TTS) + معالجة صور أساسية.
  5. نظام صلاحيات بسيط + مفاتيح API مشفرة.

* **خارج النطاق الأول** (لاحقاً): روبوتات فيزيائية، حوسبة موزعة متقدمة، لوغاريتمات Reinforcement Learning.

---

## 2) حالات استخدام أساسية

1. **وكيل كتابة/ترجمة/تلخيص**: إدخال نص → مخرجات ملخص/ترجمة/مقال.
2. **وكيل رؤية حاسوب بسيط**: رفع صورة → وصف/وسوم/استخراج نص (OCR).
3. **وكيل صوت**: رفع مقطع صوتي → تفريغ نص + توليد صوت لرد مختصر.
4. **وكيل تحليل بيانات**: CSV → إحصاءات سريعة + توصيات.
5. **مهام مركّبة**: سيناريو «ابحث-حلّل-لخص-انشر» بتكليف واحد.

---

## 3) معمارية النظام (High-Level)

```
[Client (Web/UI)] → [API Gateway] → [Orchestrator]
                                ├─[Task Queue]
                                ├─[Agent: NLP]
                                ├─[Agent: Vision]
                                ├─[Agent: Audio]
                                ├─[Agent: Data]
                                ├─[Toolbox: HTTP, Files, OCR, Translators]
                                ├─[RAG: Vector DB + Embeddings]
                                └─[DB: SQL + Object Storage]
```

**المكوّنات:**

* **UI (Next.js/React)**: إرسال مهام + مراقبة حالة/سجل.
* **API Gateway (FastAPI/Express)**: توحيد المصادقة وتحديد المعدل (Rate Limit).
* **Orchestrator (Python)**: Graph/State Machine لإسناد الخطوات للوكلاء.
* **Agents**: وحدات معيارية تُغلف نماذج وخدمات (OpenAI, local LLM, Whisper, TTS...).
* **RAG**: تخزين سياق طويل المدى، استرجاع مقتطفات ذات صلة.
* **DB**: PostgreSQL (بيانات مهيكلة) + MinIO/S3 (الملفات) + Redis (صف مهام/كاش).
* **Observability**: OpenTelemetry + Loki/Promtail + Grafana.

---

## 4) تكنولوجيا مقترحة (Stack)

* **Frontend**: Next.js 14, React 18, TailwindCSS, shadcn/ui.
* **Backend**: Python 3.11, FastAPI, Pydantic v2, Celery/RQ، SQLAlchemy.
* **LLM**: واجهة OpenAI/Anthropic + خيار Local (Ollama) للتكلفة.
* **STT/TTS**: Whisper/NeMo للـ STT، وTTS (OpenAI/Coqui/Tortoise تبعاً للأداء).
* **Vision**: OCR (Tesseract/EasyOCR)، تصنيف/وصف (CLIP/BLIP/VLM عبر API).
* **DB**: PostgreSQL 15/16 + Redis 7 + MinIO (س3-compatible).
* **Vector DB**: Qdrant أو Weaviate أو pgvector.
* **Auth**: JWT + OAuth2 Password Flow.
* **Infra**: Docker Compose (MVP) ثم Kubernetes لاحقاً.
* **CI/CD**: GitHub Actions.

---

## 5) تصميم الـ Orchestrator

* **نموذج تنفيذ**: Directed Acyclic Graph (DAG) أو State Machine (XState-like) بانتقالات واضحة.
* **محرّك قواعد (Policy/Guardrails)**: YAML/JSON يحدد:

  * ما الأدوات المسموح بها لكل دور (Role/Agent).
  * حد التكلفة والوقت لكل مهمة.
  * خطوات التحقق/المراجعة (self-check, critique, retry).
* **بروتوكول المهام**:

  * `Task` يحتوي: id, type, input, context, constraints, owner.
  * `Step` يحتوي: name, agent, tool, input_ref, output_ref, status.
  * `Event` لكل انتقال/ناتج/خطأ.

### مثال تدفق «ابحث-حلّل-لخص»

1. Orchestrator يستدعي Tool:HTTP لجلب 3 مصادر.
2. Agent:NLP يُلخّص كل مصدر على حدة.
3. Agent:NLP يُدمج الملخصات ويولد تقرير.
4. RAG يحدّث الذاكرة بملفات/مقتطفات مهمة.
5. النتيجة تُخزن وتُعرض في الـ UI.

---

## 6) نموذج الصلاحيات والأمان

* **Users**: Admin, Editor, Viewer.
* **AuthN/AuthZ**: JWT + RBAC (قائمة صلاحيات لكل دور، تُحفظ في DB).
* **سرية المفاتيح**: مخزن أسرار (Dotenv محلي ثم Vault لاحقاً).
* **تقييد الأدوات**: سياسة تمنع استدعاء أدوات حساسة إلا بموافقة (Feature Flags).
* **تقييد التكلفة**: حصة يومية لكل مستخدم + مراقبة فواتير.
* **سجلات تدقيق (Audit Log)**: لكل عملية نموذج/أداة + المدخلات والمخرجات المختصرة.

---

## 7) مخزن البيانات (Data Model)

**جداول SQL (مقترح):**

* `users(id, email, hashed_password, role, created_at)`
* `api_keys(id, user_id, name, encrypted_key, created_at)`
* `projects(id, owner_id, name, settings, created_at)`
* `tasks(id, project_id, type, status, params, cost_est, created_at)`
* `task_steps(id, task_id, name, agent, tool, status, started_at, finished_at, output_ref)`
* `events(id, task_id, step_id, level, message, payload, created_at)`
* `artifacts(id, task_id, kind, uri, metadata, created_at)`
* `vectors(id, project_id, doc_id, embedding, meta)` (إن استخدمنا pgvector)

**Object Storage (S3/MinIO):**

* `/artifacts/<task_id>/<filename>` (صور، صوت، PDF، JSON).

---

## 8) RAG والذاكرة

* **تجزئة المستندات**: Recursive/Markdown splitter (512-1024 توكن).
* **تضمين (Embeddings)**: text-embedding-3-large (أو محلي عبر BGE/MiniLM).
* **سياسة الاسترجاع**: Top-K (3-8) + re-ranking بسيط.
* **حقن السياق**: Prompt templating يضيف مقتطفات مع مصادرها.

---

## 9) واجهات داخلية (API Contract)

**Base URL**: `/api/v1`

* `POST /auth/login` → {token}
* `GET /me` → بيانات المستخدم
* `POST /tasks` → إنشاء مهمة `{type, input, options}`
* `GET /tasks/:id` → حالة ونتيجة
* `GET /tasks/:id/events` → سجل الأحداث
* `POST /upload` → رفع ملفات (S3)
* `POST /rag/index` → فهرسة ملف/نص إلى Vector DB
* `POST /agents/nlp/summarize` → ملخص مباشر
* `POST /agents/vision/describe` → وصف صورة
* `POST /agents/audio/transcribe` → تفريغ صوت
* `POST /agents/audio/tts` → توليد صوت

**نموذج `POST /tasks` (مثال):**

```json
{
  "type": "research_summarize",
  "input": {
    "query": "Iraqi education policy 2024",
    "sources": ["https://..."]
  },
  "options": {"deadline_s": 120, "budget_usd": 0.5}
}
```

**استجابة عامة للمهمة:**

```json
{
  "id": "tsk_...",
  "status": "running|succeeded|failed",
  "result": {"summary": "...", "artifacts": ["s3://..."]}
}
```

---

## 10) هيكل المجلدات (Monorepo بسيط)

```
repo/
  apps/
    web/                # Next.js UI
    api/                # FastAPI service
  packages/
    orchestrator/       # محرك الحالات/الجراف
    agents/             # NLP, Vision, Audio, Data
    tools/              # HTTP, Files, OCR, Translators...
    rag/                # تغليف لـ vector DB
    common/             # نماذج Pydantic و Utils
  infra/
    docker/             # Dockerfiles + docker-compose.yml
    k8s/                # YAML لاحقاً
    migrations/         # Alembic
  docs/
    specs/              # هذه الوثيقة + مخططات
```

---

## 11) ملف إعدادات (.env) — نموذج

```
# API
API_HOST=0.0.0.0
API_PORT=8080
JWT_SECRET=change_me
JWT_EXPIRES_MIN=120

# OpenAI
OPENAI_API_KEY=sk-...
OPENAI_BASE_URL=https://api.openai.com/v1

# STT/TTS
WHISPER_MODE=api   # local|api
TTS_PROVIDER=openai  # openai|coqui

# Databases
POSTGRES_URL=postgresql+psycopg2://user:pass@db:5432/superai
REDIS_URL=redis://redis:6379/0
S3_ENDPOINT=http://minio:9000
S3_ACCESS_KEY=minio
S3_SECRET_KEY=minio123
S3_BUCKET=artifacts

# VectorDB
VECTOR_DB=pgvector # qdrant|weaviate|pgvector
```

---

## 12) نماذج Prompt Templates (مختصرة)

**تحليل نص عام**

```
System: أنت وكيل NLP احترافي. التزم بالتنسيق المطلوب.
User Goal: {goal}
Context (RAG): {snippets}
Instruction: لخص المحتوى بدقة مع نقاط رئيسية ومراجع.
Output JSON Schema: {schema}
```

**وصف صورة**

```
System: أنت وكيل رؤية حاسوب. قدم وصفاً واضحاً وقصيراً للصورة.
Constraints: 80-120 كلمة، لا تخمينات.
Return: { "objects":[], "caption":"..." }
```

**تفريغ صوت**

```
System: حوّل الكلام المنطوق إلى نص، مع فواصل زمنية إذا أمكن.
Quality: حسّن علامات الترقيم، لا تغيّر المعنى.
```

---

## 13) خوارزميات أساسية (شيفرة كاذبة)

**Orchestrator (Pseudo-Python)**

```
class Orchestrator:
    def run_task(self, task: Task):
        graph = self.build_graph(task.type)
        for step in graph.topo_order():
            result = self.execute_step(step, task)
            self.record_event(task.id, step, result)
            if result.failed and step.can_retry:
                self.retry(step)
        return self.finalize(task)
```

**Agent واجهة عامة**

```
class Agent(Protocol):
    name: str
    def handle(self, payload: dict, context: Context) -> AgentResult:
        ...
```

---

## 14) المراقبة والمقاييس

* **أساسيات**: request_latency_ms, token_usage, cost_usd_per_task, agent_success_rate.
* **لوحات**: Grafana dashboards للمهام، الأخطاء، التكلفة.
* **تنبيهات**: Slack/Telegram عند ارتفاع الأخطاء/التكلفة.

---

## 15) اختبارات وضمان الجودة

* **Unit Tests** للوكلاء والأدوات.
* **Contract Tests** للـ API.
* **E2E** لسيناريوهات: نص→ملخص، صورة→وصف، صوت→تفريغ.
* **Load Test**: 50-200 مهمة متوازية (Locust/K6).

---

## 16) خطة النشر (MVP)

* **Docker Compose** بخدمات: api, web, db, redis, minio.

* **Seed**: مستخدم Admin افتراضي + مفاتيح وهمية للاختبار.

* **حجم الموارد**:

  * API: 2 vCPU / 4GB RAM
  * DB: 2 vCPU / 4-8GB RAM
  * Redis/MinIO: 1 vCPU / 1-2GB RAM
  * (اختياري GPU لخدمات محلية)

* **ترقية مستقبلية**: انتقال إلى Kubernetes + Autoscaling.

---

## 17) تقدير تكلفة أولي (شهري)

* سحابة (VPS متوسط): 20–60$
* DB مُدارة (اختياري): 30–80$
* مخزن كائنات: 5–20$
* فواتير نماذج (OpenAI/…): حسب الاستعمال (ابدأ بسقف 50–200$)

---

## 18) خارطة طريق (Milestones)

1. **M0 — القاعدة**: هيكل المستودع + Docker + Auth + CRUD بسيط للمهام.
2. **M1 — NLP**: تلخيص/ترجمة + RAG فهرسة/استرجاع.
3. **M2 — Vision**: OCR + وصف صور.
4. **M3 — Audio**: تفريغ صوت + TTS.
5. **M4 — مهام مركبة**: Graph أوركسترا + مراقبة تكلفة.
6. **M5 — Observability**: تتبع موزع + لوحات.

---

## 19) ما الذي نطلبه من Codex تحديداً؟

اطلب من Codex تنفيذ ما يلي بالترتيب:

1. **توليد Monorepo** بالهيكلة المذكورة.
2. إنشاء **FastAPI** مع المسارات (Auth, Tasks, Upload, Agents...).
3. ربط **PostgreSQL** و **SQLAlchemy/Alembic** مع نماذج الجداول.
4. تكوين **Redis** لصف المهام (RQ/Celery) وتنفيذ عامل Worker.
5. إنشاء **RAG Package** بسيط (pgvector أو Qdrant client) مع دوال index/search.
6. إنشاء **Agents Package**: NLP/Vision/Audio/Data كأصناف منفصلة تلتزم بواجهة موحدة.
7. **Orchestrator** بسيط (State Machine/DAG) يدير خطوات «ابحث-حلّل-لخص».
8. واجهة **Next.js** بصفحات: تسجيل الدخول، إنشاء مهمة، لوحة مهام، تفاصيل مهمة.
9. **رفع ملفات** إلى MinIO عبر Signed URLs.
10. **اختبارات** أساسية + GitHub Actions (lint/test/build).

> بعد نجاح الـ MVP، نطلب من Codex تحسين الحماية، إضافة مراقبة تكلفة، وتبويب السياسات (Policy DSL) للأدوات.

---

## 20) ملاحق

### 20.1 Schemas (Pydantic)

* `TaskCreate { type: Literal["nlp_summarize"|"vision_describe"|...], input: dict, options?: dict }`
* `Task { id, status, result?: dict, created_at }`
* `Event { id, task_id, step_id?, level, message, payload, created_at }`

### 20.2 قيود الأداء (MVP)

* زمن استجابة واجهة API < 300ms (بدون تنفيذ مهام ثقيلة).
* تنفيذ مهمة NLP بسيطة < 5s.
* حجم ملف مرفوع ≤ 20MB.

### 20.3 سياسات الاستخدام المقبول

* منع المحتوى المخالف/الضار.
* تعقيم المدخلات (Sanitization) قبل تمريرها للأدوات الخارجية.

---
