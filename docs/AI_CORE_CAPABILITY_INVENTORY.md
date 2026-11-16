# جرد قدرات الذكاء الاصطناعي الحالية — مدخل AI Core

- **الحالة:** جرد معماري أولي موثق؛ يحتاج تدقيقاً برمجياً على `main` قبل إعادة استخدام أي مكوّن.
- **المرجع التنفيذي:** `docs/UNIFIED_50_TASK_EXECUTION_PLAN.md` — S4-01.
- **AI Chat App main عند خط أساس الجرد:** `17c9868fc17a37a1743c4a49cdb378d93160650c`.
- **المصدر الأساسي:** `HVA09/ai-chat-app/docs/AI_PLATFORM_ROADMAP_AR.md`، مع سجل التفكيك `docs/PR-665_SPLIT_TRACKER_AR.md`.

## 1. قاعدة الجرد

وجود القدرة في خارطة منتج لا يثبت وحده أنها متاحة أو مكتملة على `main` أو منشورة. تذكر هذه الوثيقة ما تصفه وثائق المنتج، وتفرض فحص الملفات والـPRs والاختبارات على رأس `main` قبل اعتماد أي مكوّن في AI Core. لا نكرر الميزة لمجرد أن بناء النواة المشتركة يبدو أنظف.

## 2. قدرات AI Chat App الموثقة في خارطته

| المجال | القدرات الموثقة | القرار الأولي لـAI Core |
|---|---|---|
| Tooling | Tool Registry وMCP Integration | إعادة الاستخدام أو بناء adapters على العقد الحالي؛ لا Tool Registry ثانٍ قبل فحص الكود |
| Agent Runtime | Runtime للوكلاء، Tool Permissions، حالات تشغيل الأدوات | اعتباره مرجع المنتج الحالي؛ استخرج فقط واجهات مشتركة مثبتة الحاجة |
| Execution | Sandboxed Code Execution، Long-running Agent Jobs، Scheduled Agents | إعادة استخدام حدود التنفيذ والمهام إن أثبت الفحص صلاحيتها؛ لا نقل عشوائي للكود |
| Agent Safety | Prompt-injection وTool-abuse defenses | جرد الضوابط والاختبارات الحالية ثم تحديد الثغرات لا إعادة بناء حماية مكررة |
| Developer Platform | API Keys، API Versioning، Webhooks، OAuth/Connectors، Python/JS SDKs | واجهات المنتج تظل مملوكة لـAI Chat App؛ قد يصمم AI Core عقداً داخلياً متوافقاً ولا يستبدلها |
| Enterprise Controls | Usage/Cost Controls وEnterprise RBAC | لا تنسخ سياسة الصلاحيات أو الاستهلاك قبل تحديد النطاقات ومالك القرار |
| Agent Workspace | Agent UX، Project Workspace، Validation/Preview، Collaboration/Import/Export، Advanced Workflows | تعامل معها كقدرات قائمة في المنتج؛ استخرج العقود فقط عند الحاجة المشتركة |

تصف خارطة `AI_PLATFORM_ROADMAP_AR.md` مراحل D1–D8 وE1–E8 وF1–F5 بأنها مكتملة أو متحققة في مواضع متعددة. هذه حالة وثائقية تاريخية يجب مطابقتها مع الكود والاختبارات الحالية، ولا تُستخدم وحدها كدليل قبول حي.



## 2.1 فحص مسارات الكود في main

تم فحص شجرة ملفات AI Chat App عند `main = 17c9868fc17a37a1743c4a49cdb378d93160650c`. وجود المسار يثبت وجود نقطة تنفيذ في المصدر، لكنه لا يثبت وحده صحة كل سلوك أو نتيجة CI أو نشر حي.

| القدرة | مسارات المصدر المرصودة | اختبارات/مراجع مرتبطة | استنتاج الجرد |
|---|---|---|---|
| Agent Runtime | `ai-backend/app/services/agent_runtime.py`، `app/services/ai_agent.py` | `tests/test_agent_runtime.py`، `tests/test_agent.py` | Runtime موجود داخل المنتج؛ المرشح هو توثيق/استخراج عقد مشترك لا بناء runtime جديد. |
| Tool Registry وdispatch | `app/services/tools/registry.py`، `tools/calculator.py`، `tools/code_execution.py`، `tools/data_analysis.py`، `tools/web_search.py` | `tests/test_agent.py`، `tests/test_agent_runtime.py` | registry مركزي وToolContext/ToolSpec موجودان؛ افحص حدود الصلاحيات قبل إعادة الاستخدام. |
| Tool security | `app/services/tool_security.py`، `app/services/agent_runtime.py` | اختبارات runtime والأدوات | توجد دوال schema validation وفحص مخرجات غير موثوقة؛ يلزم مراجعة threat model وتغطية حالات الحقن. |
| MCP | `app/services/mcp_client.py` | `tests/test_mcp_client.py` | تكامل MCP داخل AI Chat App؛ لا تنشئ اتصال MCP ثانياً داخل AI Core قبل فحص العقد الحالي. |
| Agent jobs/checkpoints | `app/routers/agent_jobs.py`، `models/agent_job.py`، `models/agent_workflow_step.py` | `tests/test_agent_jobs.py`، migrations 0066 و0076 و0078 و0079 | واجهات job وpause/resume/cancel موجودة؛ التوافق والحدود والملكية تحتاج فحصاً. |
| Scheduled agents | `app/routers/scheduled_tasks.py`، `models/scheduled_task.py`، `models/scheduled_task_run.py` | `tests/test_scheduled_tasks.py`، `test_scheduled_task_runs.py`، `test_scheduled_task_retry.py` | جدولة المهام موجودة في المنتج؛ لا تُكرر scheduler في النواة قبل تحديد الحاجة المشتركة. |
| Knowledge/RAG/Embeddings | `app/services/rag.py`، `app/services/embeddings.py`، `app/services/image_rag.py`، `models/file_chunk.py` | اختبارات RAG/embeddings في المستودع؛ PR #665 وسجل التفكيك للمراحل المتبقية | خط المعالجة موجود ومملوك للمنتج؛ جودة retrieval والعزل والنسخ والاستعادة تبقى مهام قبول مستقلة. |
| Project Builder/Preview | `app/services/project_preview_builder.py`، `services/project_archive.py`، `routers/projects.py`، `routers/project_files.py` | اختبارات project preview/archive ضمن مستودع AI Chat App | builder/preview موجودان؛ preview builder منفصل ويفشل مغلقاً إذا لم يُضبط endpoint آمن. |
| Workspace/RBAC | `app/services/workspace_rbac.py`، `routers/workspaces.py`، `routers/workspace_rbac.py`، `routers/workspace_members.py` | `tests/test_workspace_usage.py` وملفات workspace access tests | RBAC وعضوية Workspace موجودة؛ لا تستبدل بنموذج صلاحيات مركزي قبل توافق المستهلكين. |
| Webhooks/Developer API | `app/routers/webhooks.py`، `services/webhook_service.py`، `routers/api_keys.py`، `sdk/python/`، `sdk/javascript/` | `tests/test_api_keys.py`، SDK tests، webhook tests | مفاتيح API وWebhooks وSDKs موجودة؛ يجب الحفاظ على عقد API الحالي وإصداراته. |
| Assistant versions/prompts | `models/assistant_version.py`، `models/saved_prompt_version.py`، `routers/assistants.py`، `routers/saved_prompts.py` | `tests/test_assistants.py`، `tests/test_saved_prompts.py` | إدارة المساعدين والموجهات موجودة داخل المنتج؛ لا تضف مخزناً عاماً قبل تحديد النطاق. |

**حد التحقق:** هذا فحص مسارات وشواهد مصدرية على رأس `main`، وليس إعادة تشغيل للاختبارات أو مراجعة لكل ملف. يلزم في S4-01 ربط كل قدرة بتفاصيل API/permission/tests والـPRs المفتوحة وحالة النشر قبل اختيار ما يُستخرج. يجب ألا تُعامل عناوين roadmap بوصفها إثبات تنفيذ منفردة.

## 3. قدرات المنتجات الأخرى وحدودها

### Lumen
- المنتج يملك المجال التعليمي: المتعلمون والمعلمون والمديرون والمقررات والدروس والواجبات والاختبارات والتقدم والتقارير.
- مختبر Python وبيئة تنفيذ المتعلم من مسؤولية Lumen. Issue #134 مفتوحة وتؤكد أن Pyodide/WebAssembly وحدهما ليسا حد عزل أمنياً عن origin التطبيق؛ لا نستخرج هذا المختبر إلى AI Core قبل إثبات العزل والموارد.
- LTI 1.3/AGS تكامل تعليمي منفصل. PR #154 ما زال Draft، وMoodle/Canvas والخصوصية والتشغيل لها بوابات مستقلة.
- لا يجب أن يتطلب عمل Lumen توافر AI Core.

### ERP Global Pro
- ERP يملك بيانات الشركات والهوية العضوية والعمليات والمالية والملفات والتقارير.
- وثائق Stage 23–25 تشرح نماذج signed tenant context وRLS وPostgreSQL، لكن SQLite v8 ما زالت قاعدة التطبيق النشطة، و`TENANT_DATA_SCOPING_ENABLED=False`.
- لا تُمرر بيانات ERP إلى ذاكرة أو RAG مشتركة افتراضياً، ولا يستخدم AI Core سياق شركة يقدمه العميل وحده كدليل صلاحية.

## 4. ما يجب أن يكون مسؤولية AI Core الجديدة

هذه ليست قدرات معتمدة أو منفذة بعد؛ إنها مساحات مرشحة للدراسة بعد إكمال الجرد:
1. Model Gateway موحد مع adapters اختيارية لمزودين ونماذج مفتوحة الأوزان.
2. سجل إصدارات النماذج وتقييمات قابلة للتكرار واختيار النموذج وفق المهمة والموارد.
3. عقود وكلاء متخصصة للبرمجة وهندسة البرمجيات وLinux والشبكات وCloud/DevOps والأمن السيبراني.
4. توجيه مهام الوكلاء وسياسة أدوات وموافقات، مع إعادة استخدام Tool/MCP حيث يصلح.
5. سياسة معرفة وذاكرة منفصلة النطاقات للمستخدم والمشروع والمؤسسة.
6. تقييم الجودة والسلامة والكمون والتكلفة قبل اعتبار أي نموذج أو وكيل جاهزاً.

## 5. قرارات منع التكرار والحدود

- AI Core لا يستبدل Agent Runtime أو MCP أو Workspace أو Project Builder الموجود في AI Chat App لمجرد توحيد الاسم.
- لا نقل لجداول أو ملفات أو بيانات المحادثات أو الطلاب أو الشركات بين المنتجات.
- لا SSO أو جلسة مركزية أو مفتاح موحد قبل نموذج تهديد وعقد تفويض منفصل.
- لا تشغيل GPU أو خدمة مدفوعة؛ يبدأ التقييم بما يتوفر مجاناً ومحلياً، ولا التزام مالي دون موافقة.
- كل استخراج مشترك يجب أن يثبت منفعة صريحة ومالكاً واضحاً واختبارات توافق وخطة rollback.

## 6. الخطوات التالية لإغلاق الجرد

1. تدقيق كل قدرة D/E/F على ملفات `main` الحالية وPRs المفتوحة وعدم الاعتماد على عناوين خارطة الطريق فقط.
2. ربط كل قدرة بمالكها وواجهتها واختباراتها وصلاحياتها وحالة نشرها.
3. تصنيفها: Product-owned، Reusable contract، Shared service candidate، أو Not implemented.
4. تحديث هذا الجرد فقط مع SHA/PR/CI واضح.
5. بعد ذلك يبدأ تصميم AI Core التفصيلي، لا قبله.
