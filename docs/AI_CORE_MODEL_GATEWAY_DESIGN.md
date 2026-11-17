# تصميم Model Gateway وكتالوج النماذج — AI Core

- **المعرف:** S4-04 ضمن خارطة المراحل الخمس.
- **الحالة:** Design baseline documented — لا خدمة أو API Gateway منفذة.
- **التاريخ:** 2026-11-17.
- **المبدأ:** إعادة استخدام طبقة مزودي AI الموجودة في AI Chat App كمرجع تطبيقي، دون نسخ Runtime أو جعله اعتماداً إلزامياً لـLumen أو ERP.
- **قرار النموذج:** لا يوجد نموذج محدد أو benchmark مقبول في هذه الوثيقة.

## 1. الهدف وحدود المسؤولية

يحدد هذا التصميم عقداً مستقلاً لتوجيه طلبات الاستدلال إلى مزود/نموذج ملائم، مع واجهة موحدة وقدرات معلنة وسياسة موحدة للمهل والفشل والتكلفة والتدقيق.

يكون Model Gateway في AI Core مسؤولاً عن:
- تسجيل نماذج/مزودين مع alias داخلي ثابت وقدرات وإصدار وحالة اعتماد.
- اختيار المزود adapter وفق alias وسياسة المستهلك وتصنيف البيانات.
- توحيد عمليات النص والبث والرؤية وtool-call envelopes حيث يدعمها المزود.
- تطبيق حدود الاتصال والـtimeouts والـretry/fallback وcircuit breaker بطريقة قابلة للتدقيق.
- إعادة usage/latency/error metadata دون تسجيل محتوى prompt أو أسرار.
- إظهار النماذج المسموح بها لكل service consumer عبر catalog محدود.

ولا يكون مسؤولاً عن:
- هوية المستخدم العامة أو تسجيل الدخول أو SSO.
- امتلاك ملفات أو محادثات أو بيانات طالب أو شركة.
- تنفيذ أدوات Agent أو MCP أو أوامر Linux/Cloud أو أفعال مالية.
- إدارة RAG/Memory أو vector stores؛ تلك تبقى خدمات يملكها المنتج أو عقداً منفصلاً.
- احتساب الفاتورة التجارية للمستخدم النهائي أو تجاوز حصص المنتج.
- توحيد قواعد بيانات Lumen وAI Chat App وERP.

**قاعدة الاستقلال:** يستطيع كل منتج الاستمرار في عملياته الأساسية عند تعطل AI Core. ويظل AI Chat App قادراً على استخدام provider factory المحلية حتى يعتمد مسار انتقال اختياري ومثبت.

## 2. خط الأساس الفعلي في AI Chat App

**مصدر التدقيق:** `HVA09/ai-chat-app`، main commit `6645b0502daa6922040ee4c6bb5c281ed59ef366`.

| المكون الحالي | الدليل | الملاحظة المعمارية |
|---|---|---|
| Provider factory | `ai-backend/app/services/ai_providers/factory.py` | `get_provider()` يختار openai/deepseek/anthropic/gemini؛ OpenAI/DeepSeek/Gemini يمرون عبر OpenAI-compatible adapter، وAnthropic عبر adapter مستقل. |
| Provider abstraction | `ai-backend/app/services/ai_providers/base.py` | `AIProvider` يفرض حالياً `get_reply` و`stream_reply` فقط؛ capabilities الخاصة بالرؤية وtool-calls ليست ضمن العقد الأساسي. |
| Provider adapters | `openai_provider.py`, `anthropic_provider.py`, `gemini_provider.py` | يوجد اختلاف في البروتوكولات والرؤوس وتنسيق الردود والاستخدام؛ لا يجوز افتراض تكافؤ capability لمجرد أن كل adapter يعيد نصاً. |
| Routing/failover | `ai-backend/app/services/ai_service.py` | non-stream fallback للأخطاء العابرة؛ streaming fallback قبل أول chunk فقط؛ لا تبديل بعد إرسال جزء من الرد. |
| Model allowlist | `app/config.py`, `app/dependencies.py`, `app/routers/chat.py` | إعدادات عالمية مع plan-level entitlement للمستخدم؛ يجب الحفاظ على منع اختيار نموذج غير مسموح. |
| Agent tool-calls | `app/services/ai_agent.py` | Agent mode الحالي يتطلب OpenAI-compatible provider؛ capability يجب أن تُفحص صراحة، لا أن تُفترض. |
| Timeouts | `app/config.py` وprovider adapters | non-stream timeout قابل للتهيئة؛ stream/tool/vision لها حدود تشغيلية مختلفة في بعض adapters. مطلوب توحيد سياسة per-operation قبل نقلها إلى Gateway. |
| Cost/usage | `app/services/ai_cost.py`, `UsageLog`, admin/workspace usage | pricing config اختياري؛ إذا لم يوجد سعر معرف فلا تُخمن التكلفة. حصص المستخدم/Workspace تبقى لدى المنتج. |
| Regression tests | `tests/test_ai_providers.py`, `tests/test_ai_failover.py`, `tests/test_provider_analytics.py` | تغطي provider selection وallowlist وtimeout وfallback والتمييز بين الفشل قبل/بعد بدء stream. |

### 2.1 قيود يجب أن يحافظ عليها التصميم

1. `get_provider()` يقبل داخلياً overrides للمزود والمفتاح وbase URL. لا يجب كشف هذه الخيارات في request schema أو السماح للمستهلك بتقديم URL؛ ذلك يفتح مجال تجاوز catalog أو SSRF أو تسريب credentials.
2. `AIProvider` الحالي لا يضمن دعم vision أو structured tool-calls. يجب الإعلان عن capability per model/adapter، ورفض العملية بوضوح عند عدم الدعم.
3. Gemini يمر عبر OpenAI-compatible adapter في factory الحالية رغم وجود adapter Gemini مستقل. لا تعتبر صيغة adapter أو الاسم وحدهما دليلاً على equivalence بين الميزات.
4. fallback في streaming لا يغير المزود بعد أول chunk حتى لا ينتج رد مركب من مزودين. يجب تثبيت هذا السلوك في أي Gateway.
5. pricing metadata ليست إلزامية حالياً؛ التكلفة غير المعروفة يجب أن تبقى `unpriced`، ولا تُحسب بسعر افتراضي.
6. Model Gateway لا يملك صلاحية لتغيير model entitlement أو monthly budget أو workspace limits المملوكة لـAI Chat App.

## 3. الطبقات المقترحة

```text
Product Adapter (Lumen / AI Chat App / ERP)
  ├─ authenticates end user in its own product
  ├─ enforces resource ownership + product quotas
  ├─ constructs minimum required context
  └─ obtains short-lived service/delegation credential
             |
             v
AI Core Model Gateway
  ├─ Service Authentication (issuer / audience / expiry / revocation)
  ├─ Consumer Policy (product, operation, data classification, scopes)
  ├─ Model Catalog + Capability Resolver
  ├─ Routing + bounded retry/fallback/circuit breaker
  ├─ Provider Adapters (OpenAI-compatible / Anthropic / future local)
  ├─ Usage + latency + failure metadata
  └─ Secret-free audit/trace metadata
             |
             v
Approved Provider Endpoint / Isolated Local Runtime
```

- لا يضع Product Adapter secrets أو provider URL أو provider API key داخل body.
- لا يتصل Gateway مباشرة بقاعدة أي منتج؛ المنتج المالك يقرر ما يجوز تمريره.
- لا يملك Gateway جدولاً موحداً للمستخدمين أو الشركات أو الطلاب؛ هوية المستهلك تُفهم عبر issuer/audience، وهوية المستخدم المفوضة عبر claims موثوقة ومحدودة.
- يمكن أن يبقى Gateway مكتبة/خدمة داخل AI Core في البداية؛ لا يُنشأ نشر مستقل قبل قياس الحاجة التشغيلية.

## 4. نموذج عقد API مقترح

هذه أسماء تصميمية وليست endpoints قائمة أو معتمدة للنشر.

### 4.1 Catalog

`GET /v1/models`

يرجع aliases المسموح بها لهذا service consumer، مع metadata آمنة فقط:
- `model_alias`
- `catalog_version`
- `capabilities`: text, streaming, vision, tool_calls, structured_output, embeddings
- `max_context_tokens` و`max_output_tokens` عند توفر قيم مؤكدة
- `data_classification_max`
- `status`: active / preview / deprecated
- `consumer_allowlist` أو policy reference عند عدم كشف تفاصيل القائمة

لا يكشف API أسماء أسرار المزود أو API keys أو internal base URLs أو بيانات حساب الفوترة.

### 4.2 Non-stream generation

`POST /v1/responses`

شكل طلب مفاهيمي:

```json
{
  "contract_version": "1",
  "request_id": "opaque-request-id",
  "model": "approved-alias",
  "operation": "text.generate",
  "input": {
    "messages": [
      {"role": "user", "content": "task content"}
    ]
  },
  "limits": {
    "max_output_tokens": 1200,
    "deadline_ms": 30000
  },
  "metadata": {
    "correlation_id": "opaque-correlation-id",
    "data_classification": "internal"
  }
}
```

- `consumer_id` و`actor` و`scopes` تُستخرج من credential موثوق أو claims موقعة؛ لا تقبل كحقول موثوقة من body.
- `model` alias معتمد وليس اسم مزود أو arbitrary model string.
- Gateway يطبق سقفاً خادمياً للـdeadline/output tokens ولا يثق بالقيم الأعلى التي يرسلها المستهلك.
- لا يرسل المستهلك provider API key أو base_url أو provider_name.
- إذا أرفق سياقاً حساساً، يجب أن تكون data classification ضمن policy المسموح لذلك alias/provider.

استجابة مفاهيمية:

```json
{
  "request_id": "opaque-request-id",
  "model": "approved-alias",
  "status": "completed",
  "output": {"text": "normalized response"},
  "usage": {
    "input_tokens": 120,
    "output_tokens": 280,
    "cost_status": "unpriced"
  },
  "provider_metadata": {
    "provider_id": "internal-provider-reference",
    "latency_ms": 1840
  }
}
```

### 4.3 Streaming

`POST /v1/responses/stream`

يستخدم SSE أحداثاً محددة مثل:
- `response.started`
- `response.delta`
- `response.completed`
- `response.error`

لا يبدأ failover إلى مزود آخر بعد أول delta. عند فشل primary قبل أول delta، يسمح فقط بمحاولة fallback واحدة أو عدد صغير bounded وفق policy، بشرط أن يدعم المزود البديل نفس العملية والتصنيف والحدود. إلغاء العميل يوقف stream ويغلق provider iterator.

### 4.4 Capabilities منفصلة

- vision: لا تُقبل الصور إلا مع model alias معلن بدعم الرؤية وحد payload/type validation.
- tool-calls: Gateway يطبع tool call envelope فقط ولا ينفذ الأدوات؛ Agent Runtime والمنتج المالك يتحققان من registry/ACL/approval قبل التنفيذ.
- embeddings: عقد منفصل `/v1/embeddings` عند اعتماد الحاجة؛ لا يخلط embedding-space identity مع chat model alias ولا يغير dimensions/model دون reindex strategy.

## 5. Model Registry

كل سجل نموذج داخلي يجب أن يحتوي على:

| الحقل | الاستخدام |
|---|---|
| `model_alias` + `catalog_version` | اسم ثابت للمستهلك مع إصدار تغييرات القواعد |
| `provider_id` + `provider_model_id` | mapping داخلي لا يظهر فيه secret |
| `adapter_type` | بروتوكول الاتصال/تحويل الرسائل |
| `capabilities` | text / stream / vision / tool-calls / structured output / embeddings |
| `context_limit` و`output_limit` | حدود موثقة ومتحقق منها؛ لا قيم تخمينية |
| `consumer_allowlist` و`operation_scopes` | من يستطيع استعمال النموذج ولأي عملية |
| `data_classification_max` | أعلى تصنيف مسموح بإرساله لهذا المزود |
| `region_or_residency_policy` | قيد إقامة البيانات إن كان مدعوماً ومثبتاً |
| `timeout_policy_id` و`retry_policy_id` | حدود مركزية قابلة للمراجعة |
| `pricing_reference` | مصدر سعر وتاريخ سريانه أو `unpriced` |
| `status` | active / preview / deprecated / disabled |
| `fallback_group` | مجموعة بدائل متوافقة policy/capability وليست fallback عشوائياً |

**عدم افتراض نموذج:** لا نسجل model alias فعلياً أو provider default حتى تنتهي S4-03 من مراجعة الترخيص والعتاد والجودة والخصوصية والbenchmark.

## 6. Service identity والتفويض

1. لكل منتج service principal مستقل، مثل `lumen-adapter` و`ai-chat-adapter` و`erp-adapter`، مع issuer/audience/scopes قصيرة العمر وإلغاء قابل للتحقق.
2. credential الخاص بالمستهلك يثبت هوية الخدمة فقط؛ لا يمنحها حق الوصول إلى أي student/project/company record تلقائياً.
3. المنتج المالك يتحقق من المستخدم والموارد قبل طلب الاستدلال، ويمرر أقل سياق ممكن. عند الحاجة إلى delegated actor، يُمرر عبر claims موقعة من issuer معروف، لا عبر `user_id` في JSON.
4. scopes تكون على العمليات مثل `model.generate` و`model.stream` و`model.vision` و`model.tools.propose`؛ لا تمنح scope التنفيذ `tools.execute`.
5. ERP لا يرسل سجلات تجارية/مالية إلى Gateway حتى إغلاق Stage25C وtenant isolation وrestore/rollback gates؛ يبقى `TENANT_DATA_SCOPING_ENABLED=False`.
6. Lumen لا يرسل بيانات طالب/درجات/محتوى مقرر إلا بموافقة سياسة Lumen وتصنيف البيانات والغرض والتفويض؛ لا تستخدم صلاحية admin للمنصة كاعتماد خدمة.
7. AI Chat App يحتفظ بالجلسة وWorkspace quota وملكية الملفات وAgent execution؛ لا تتحول service identity إلى بديل عن user authorization.

## 7. Routing وRetries وFallback

- اختيار alias deterministic بحسب policy؛ لا اختيار عشوائي أو بناء على حقل provider يرسله العميل.
- لا fallback عند أخطاء المصادقة/التفويض/التحقق/عدم دعم capability أو تجاوز quota/budget.
- retry/fallback للأخطاء العابرة فقط، مثل timeouts وnetwork failures و429 و5xx محددة، وبحد أقصى ومحاولة وdeadline إجمالي معلوم.
- fallback يجب أن يطابق operation capability وdata classification وconsumer policy وoutput limit.
- في streaming لا fallback بعد أول chunk؛ قبل أول chunk يمكن محاولة بديل ضمن نافذة المهلة.
- لا يوجد ضمان exactly-once لاستدعاء المزود: قد يقبل المزود الطلب ثم ينقطع الرد. `request_id/idempotency_key` يساعد في dedup داخل Gateway، لكنه لا يضمن عدم احتساب المزود للطلب الملتبس مرتين ما لم يدعم المزود ذلك.
- circuit breaker per provider/model يمنع تكرار استدعاءات مزود متعطل، مع half-open probe محدود.
- لا retries غير محدودة ولا retry متزامن عاصف؛ تستخدم bounded backoff مع jitter.
- عند انقطاع مزود بعد بدء stream، يعاد خطأ واضح ويحتفظ المنتج بسياسة عدم حفظ partial output ما لم يقرر مالك المنتج خلاف ذلك.

## 8. Limits وCost وObservability

### Limits
- connect timeout، first-token timeout، read/idle timeout، total deadline، max prompt bytes/tokens، max output tokens، concurrent requests، queue depth، per-consumer request rate.
- إلغاء الطلب يجب أن يوقف انتظار provider ويحرر الموارد.
- لا SLOs رقمية قبل جمع baseline؛ لا تنقل timeout الحالي إلى Gateway بوصفه قيمة مثالية دون قياس.

### Cost/Usage
- Gateway يعيد usage metadata الفعلية عندما يوفرها المزود.
- تكلفة الطلب تحسب فقط عند وجود price metadata موثقة بتاريخ سريان؛ وإلا `unpriced`.
- المنتج المستهلك يظل مسؤولاً عن user/workspace quotas والميزانية والفوترة المحلية.
- Gateway يستطيع تطبيق technical fair-use limits على consumer principal، ولا يتجاوز حدود المنتج المالك.

### Audit/Tracing
- سجّل: request_id، correlation_id، consumer_id، model_alias، provider reference، operation، input/output tokens، latency، retry/failover count، error class، status، cost_status.
- لا تسجل: prompt، تاريخ المحادثة، مخرجات النموذج، محتوى الملف/الصورة، bearer token، provider key، signed URL أو raw PII.
- توحيد correlation IDs لا يعني تجميع محادثات أو بيانات المستخدمين في منصة مركزية.

## 9. Failure semantics

| الحالة | السلوك |
|---|---|
| consumer token invalid/expired | 401؛ لا اتصال بالمزود |
| consumer scope denied | 403؛ لا اتصال بالمزود |
| model alias غير موجود أو غير مسموح | 404/403 وفق سياسة عدم كشف catalog |
| capability unsupported | 422؛ لا تحويل خفي إلى نموذج مختلف |
| request invalid/too large | 400/413؛ لا retry |
| rate/quota/budget exceeded | 429 مع Retry-After عند توفره؛ لا fallback |
| provider timeout/temporary 5xx | bounded retry/fallback وفق policy |
| provider auth/config error | 502/503 عام؛ لا retry على credential خاطئ ولا كشف تفاصيل سرية |
| failure after stream delta | SSE error/stream termination؛ لا مزج مخرجات provider آخر |
| ambiguous provider timeout | يسجل حالة completion unknown، ولا يدعي exactly-once |

## 10. خطة انتقال تدريجية

| المرحلة | المطلوب | بوابة الخروج |
|---|---|---|
| G0 — baseline | توثيق provider factory الحالي وcapability gaps والtimeouts/usage/fallback | مراجعة المصدر والاختبارات؛ لا كود جديد |
| G1 — contract | اعتماد `AIProvider` v2 capability contract وrequest/response schemas داخل AI Chat App أولاً | provider contract tests لكل adapter + compatibility مع chat/agent |
| G2 — catalog | إعداد model aliases وcapability/consumer/data policy دون كشف provider secrets | model authorization tests + admin-only configuration |
| G3 — optional Gateway | تنفيذ adapter داخلي اختياري أو خدمة مستقلة صغيرة داخل AI Core بعد إثبات حاجة الفصل | staging/disposable tests، CI/CodeQL، failure isolation |
| G4 — consumer pilot | تجربة مستهلك واحد بعد اعتماد service principal والبيانات والcontract tests | opt-in rollback؛ المنتج يعمل دون Gateway |
| G5 — expansion | إضافة Lumen أو ERP فقط بموافقة المالك وبعد data/tenant gates | provider/consumer tests وفحوص الخصوصية والعزل والفشل |

## 11. اختبارات القبول الإلزامية

- provider conformance tests: text generation، stream، usage mapping، error mapping، cancellation، operation capabilities.
- factory/catalog tests: aliases، unsupported provider/model، no client-supplied key/base URL، provider secret redaction.
- authZ tests: consumer audience/scope، expired/revoked service principal، product-level actor delegation، denied model/data classification.
- fallback tests: 429/5xx/network/timeout، no fallback on 401/403/validation/budget، first chunk failover، no failover after emitted chunk.
- reliability tests: total deadline، queue limits، concurrency bound، circuit breaker، cancellation، duplicate request IDs، ambiguous completion.
- privacy tests: no prompt/file/PII/token/key in logs; provider endpoint allowlist; no cross-product DB access.
- cost tests: real token usage mapping where available; unpriced model stays unpriced; product budgets remain authoritative.
- integration tests in disposable environment with fake provider; no live provider calls in ordinary CI.
- acceptance tests for each consumer product remain in its own repository; central repository stores contract references/evidence only.

## 12. المخاطر المفتوحة وقرارات لم تُتخذ

1. لم يُحسم اختيار نموذج مفتوح الأوزان أو مزود مستضاف؛ ذلك S4-03 وليس S4-04.
2. لم يُحسم هل Gateway مكتبة داخل AI Core أو خدمة منفصلة؛ لا تنشئ service deployment قبل قياس الحاجة.
3. لم تُعتمد service principals أو issuer/audience أو key rotation؛ يتطلب ذلك S5-03.
4. لم تُعتمد data classification/residency لكل provider؛ يتطلب ذلك S5-05 وموافقة مالكي المنتجات.
5. capability parity بين Anthropic/OpenAI-compatible/Gemini غير مثبت لكل tool/vision/structured operation.
6. تكلفة بعض النماذج قد تبقى مجهولة؛ لا تسعير تقديري.
7. لا توجد consumer/provider contract tests بين المنتجات حتى الآن.

## 13. المراجع التنفيذية

**AI Chat App main:** `6645b0502daa6922040ee4c6bb5c281ed59ef366`.

- `ai-backend/app/services/ai_providers/factory.py`
- `ai-backend/app/services/ai_providers/base.py`
- `ai-backend/app/services/ai_providers/openai_provider.py`
- `ai-backend/app/services/ai_providers/anthropic_provider.py`
- `ai-backend/app/services/ai_providers/gemini_provider.py`
- `ai-backend/app/services/ai_service.py`
- `ai-backend/app/config.py`
- `ai-backend/app/dependencies.py`
- `ai-backend/app/services/ai_cost.py`
- `ai-backend/app/services/ai_agent.py`
- `ai-backend/tests/test_ai_providers.py`
- `ai-backend/tests/test_ai_failover.py`

**وثائق المنظومة ذات الصلة:**
- `AI_CORE_SPECIALIZED_AGENT_REQUIREMENTS.md`
- `AI_CORE_AGENT_TOOL_MATRIX.md`
- `AI_CORE_CAPABILITY_INVENTORY.md`
- `PRODUCT_API_CONTRACT_CATALOG.md`
- `PRODUCT_OWNERSHIP_AND_INTERFACE_CATALOG.md`
- `DECISION_REGISTER.md` (خصوصاً DEC-005 وDEC-008 وDEC-009 وDEC-010 وDEC-015)

**الخلاصة:** يوجد اليوم provider abstraction وfactory وfallback داخل AI Chat App؛ المطلوب في AI Core هو capability-aware contract/catalog وسياسة ثقة وفشل وحدود موحدة. لا تعِد بناء provider clients أو Agent Runtime قبل إثبات فجوة لا تغطيها الموجودات الحالية.
