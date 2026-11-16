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


## 7. تدقيق تنفيذ وصلاحيات على main — 2026-11-17

**خط الأساس:** `HVA09/ai-chat-app` عند `main = 17c9868fc17a37a1743c4a49cdb378d93160650c`. هذا فحص قراءة للمصدر والاختبارات؛ لم تُشغّل اختبارات محلياً ولم يتم التحقق من نشر حي.

### Agent Runtime وTool Registry

| المجال | دليل main | اختبارات main | حالة القبول |
|---|---|---|---|
| Runtime limits | `ai-backend/app/services/agent_runtime.py`: 3 rounds، 4 calls/round، 8,000 chars، 20 history messages، 30 sec/tool | `tests/test_agent_runtime.py` | Runtime موجود ومحدود؛ الأرقام قيود المنتج وليست SLO أو ضماناً أمنياً مستقلاً. |
| Global tool allowlist | `app/config.py` و`services/agent_runtime.py` و`services/tools/registry.py` | `tests/test_agent_runtime.py::test_runtime_exposes_only_globally_allowed_tools`، `tests/test_tool_registry.py::test_scoped_registry_enforces_explicit_tool_allowlist` | موجودة على main؛ السماح العالمي لا يغني عن التحقق من صلاحية مورد محدد داخل handler. |
| Argument validation | `services/tool_security.py` ثم `ToolRegistry.execute` | اختبارات `test_tool_registry.py` | يوجد تحقق schema وحجم وعمق؛ يجب استمرار اختبار schemas القادمة من MCP كمصدر غير موثوق. |
| Untrusted outputs | `ToolResult.untrusted` و`wrap_untrusted_tool_output` ووقف tool follow-ups بعد نتيجة غير موثوقة | `tests/test_agent_runtime.py::test_runtime_wraps_untrusted_output_and_blocks_followup_tools`، `tests/test_tool_registry.py::test_registry_marks_untrusted_output_and_detects_injection` | أساس موجود؛ كشف injection heuristic وليس تصنيفاً كاملاً، ولذلك تبقى كل مخرجات المصدر الخارجي غير موثوقة حتى إن لم تتطابق مع النمط. |
| DB pool release | Runtime يحرر اتصال DB قبل MCP/provider/tool waits | `tests/test_agent_runtime.py::test_runtime_releases_database_pool_during_mcp_provider_and_tool_waits` | اختبار موجود؛ لا تعاد إصلاحات QueuePool السابقة دون telemetry جديدة. |

### MCP — فجوة تفويض حالية

- في main، `AgentRuntime.run()` ينفذ `self.mcp_discoverer()` بعد بناء registry دون فحص دور المستخدم.
- `MCP_ENABLED` افتراضياً `False`، وعند تعطيله ترجع discovery قائمة فارغة؛ لكن عند تفعيله، لا يوجد في main gate حسب الدور قبل الاتصال بخوادم MCP.
- توجد طبقتا allowlist: `AGENT_ALLOWED_TOOLS_JSON` العامة و`allowed_tools` لكل MCP server، إضافة إلى HTTPS في production. هذه الطبقات لا تعوض تفويض المستخدم.
- `get_agent_tool_policy_snapshot()` في main يقرأ إعدادات MCP العالمية ويعرض أسماء الخوادم وقوائم الأدوات ضمن snapshot دون redaction بحسب الدور.
- اختبارات main في `tests/test_mcp_client.py` تغطي default-disabled، إعداد JSON، per-server allowlist، وHTTPS في production؛ لا يظهر اختبار يثبت أن المستخدم العادي لا يبدأ MCP discovery.
- **Issue مفتوحة:** [AI Chat App #719 — restrict globally configured MCP tools to authorized users](https://github.com/HVA09/ai-chat-app/issues/719).
- **مرشح غير مدمج:** PR #665 على الفرع `ai/embedding-provider-separation-20261025` يضيف gate لدور `UserRole.admin`، ويخفي معلومات MCP من snapshots للمستخدمين غير المخولين، ويضيف اختباراً يمنع discovery للمستخدم العادي. PR #665 ما زال Open/Draft/Unmerged؛ هذه التغييرات ليست في main ولا تعد إصلاحاً منجزاً.

### أدوات ملفات المشاريع — إعادة تحقق ACL غير موجودة في main

- `services/tools/registry.py` في main يحتوي `_list_project_files` و`_read_project_file` و`_search_project_files` و`_edit_project_file` و`_create_project_archive`.
- handlers تعتمد على `conversation.project_id` وبيانات المشروع، لكن لا يظهر في main فحص داخل handler عبر `can_read_project()` أو `can_edit_project()` قبل القراءة/التعديل.
- يوجد في المنتج service منفصل `services/project_access.py` يوفر `can_read_project` و`can_edit_project`; استخدامه في هذه الحدود مطلوب لضمان إعادة التحقق بعد إلغاء صلاحية المستخدم أو مشاركة المشروع.
- **Issue مفتوحة:** [AI Chat App #720 — recheck project ACLs inside Agent project-file tools](https://github.com/HVA09/ai-chat-app/issues/720).
- **مرشح غير مدمج:** PR #665 يضيف `_project_permission_denial()` إلى handlers ويضيف اختبار viewer: يمكنه القراءة والبحث، ولا يستطيع تعديل الملف أو استبداله عبر archive tool. لا تحتسب هذه الحماية ضمن main قبل PR مستقل واختبارات CI ناجحة.

### جدولة المهام — كتالوج RBAC لا يطابق المسارات

- `services/workspace_rbac.py` يعرف `scheduled_tasks.manage`، و`routers/workspace_rbac.py` يصفها بأنها إدارة المهام المجدولة.
- `routers/scheduled_tasks.py` في main يتحقق من العضوية أو ملكية سجل المهمة، لكن لا يظهر استدعاء `has_workspace_permission` أو `require_workspace_permission` عبر مسارات create/update/delete/run/retry.
- اختبارات `tests/test_scheduled_tasks.py` تغطي CRUD والتحقق من وقت المستقبل ورفض غير العضو وتفرد run slot؛ ولا تغطي عضواً يملك custom RBAC role لا يتضمن `scheduled_tasks.manage`.
- **Issue مفتوحة:** [AI Chat App #718 — enforce workspace RBAC for scheduled-task operations](https://github.com/HVA09/ai-chat-app/issues/718).
- هذا تعارض بين permission catalog وسلوك المسارات بحسب مراجعة المصدر؛ يلزم حسم ما إذا كانت المهام workspace-governed أو user-owned ثم جعل العقد والاختبارات متوافقة. لا نعلن exploit مثبتاً ولا نغير الكود ضمن هذه الجولة.

### مفاتيح API وWebhooks — دلالات الملكية تحتاج قراراً

- `RBAC_PERMISSIONS` يحتوي `api_keys.manage` و`webhooks.manage`، مع وصفهما بأنهما صلاحيات مرتبطة بمساحة العمل.
- routers الحالية في `api_keys.py` و`webhooks.py` تستخدم سجلات `user_id` وملكية المستخدم، ولا يظهر استدعاء لهاتين الصلاحيتين في المسارات التي تمت مراجعتها.
- توجد اختبارات owner-scoping لمفاتيح API وWebhooks، وفحص URL يمنع العناوين الخاصة في Webhooks، وتوقيع HMAC-SHA256.
- لم تُسجل هذه الملاحظة كفجوة استغلال مؤكدة: قد يكون السلوك الحالي مقصوداً كنطاق user-owned. يلزم قرار واضح: إما إزالة/إعادة توصيف صلاحيات Workspace غير المستخدمة، أو اعتماد ملكية workspace وإضافة authorization tests. لا تحولها إلى صلاحيات مشتركة تلقائياً.

### Python execution — حد العزل بدقة

- `services/tools/code_execution.py` ينفذ كوداً محدوداً بعد AST validation داخل subprocess مستقلة بخيارات Python isolated، ومهلة 2 ثانية، وقيود موارد Linux على CPU/memory/file size/open files/processes، ويحد الكود والناتج.
- اختبارات `tests/test_code_execution.py` تغطي منع import/attribute access/builtins/dynamic evaluation، والمهلة وحجم الناتج.
- **لا توصف هذه الآلية بأنها OS/container sandbox أو عزلاً كاملاً عن kernel/host.** إنها مفسر مقيد + عملية منفصلة + حدود موارد؛ إذا احتاجت منتجات المنظومة تنفيذ كود غير موثوق على نطاق أوسع، فيلزم sandbox مستقل متعدد الطبقات مع عزل filesystem/network/identity قبل التفعيل.

### الوظائف التي ثبت أنها مدمجة بالفعل — لا تكررها من PR #665

وفق سجل التفكيك المدمج في main، section 52، وسجل PRs:
- RAG authorization: PR #694 merged؛ تحقق ملكية conversation وworkspace membership وproject/assistant share. لا تفتح نسخة مكررة.
- Embedding provider configuration: PR #692 merged.
- File-index fingerprints/schema: PR #695 merged (migration `0082_file_index_fingerprints`، بعد 0081 الحالي).
- Safe single-file reindex behavior: PR #696 merged؛ تجهيز embeddings قبل استبدال الفهرس المحفوظ.
- إصلاح تحرير DB connection قبل external waits في المسارات التي سبق إصلاحها موجود في main؛ لا تكرر إصلاح QueuePool دون دليل telemetry جديد.
- PR #665 يبقى Open/Draft/Unmerged كمصدر تاريخي لاستخراج تغييرات محددة فقط؛ لا دمج جماعي ولا cherry-pick عشوائي.

**الاستنتاج:** AI Chat App يوفر بالفعل Runtime وأدوات وMCP وjobs وscheduler وRAG وRBAC. أولويات التدقيق ليست بناء بدائل داخل AI Core، بل معالجة حدود التفويض الفعلية (Issues #718–#720) وتوثيق عقود آمنة قابلة لإعادة الاستخدام.


## 8. متابعة إصلاح MCP داخل المستودع المالك — 2026-11-17

- تم فتح PR [HVA09/ai-chat-app #721](https://github.com/HVA09/ai-chat-app/pull/721) من الفرع `security/mcp-role-gate-20261117` على `main` عند base SHA `17c9868fc17a37a1743c4a49cdb378d93160650c`.
- Head SHA الحالي وقت هذا التحديث: `45cfab03669093511397619f6ac33d31fa536454`.
- النطاق: gate لاكتشاف MCP بحسب `UserRole.admin` مع اشتراط `MCP_ENABLED`، redaction لبيانات خوادم MCP في snapshots القديمة والجديدة للمستخدمين غير المخولين، وتمرير الدور عند إنشاء snapshots للمهام المجدولة.
- أضيفت اختبارات للمستخدم العادي والإداري، ولقطع discovery للمستخدم العادي، وإخفاء metadata القديمة في استجابات Agent Jobs.
- **حالة الفحوص على الرأس الحالي:** CI run `43972899224` وCodeQL run `43972899294`؛ backend وCodeQL كانا قيد التنفيذ عند آخر تحقق، وSDK نجح. وظائف frontend وpreview-builder وproduction-compose لم تكن قد اكتملت بعد في هذا التشغيل. التشغيل الأسبق `43972151077` أصبح قديماً بعد تحديث الرأس، ولا يُستخدم لقبول الإصدار الحالي.
- لا يُغلق Issue #719 ولا توصف الفجوة بأنها معالجة قبل نجاح كل الفحوص المطلوبة ومراجعة التغيير ودمجه. لم يُتصل بأي MCP server ولم تُغيّر إعدادات الإنتاج.


## 9. نتيجة إصلاح تفويض MCP — 2026-11-17

- **تم الدمج:** [AI Chat App PR #721](https://github.com/HVA09/ai-chat-app/pull/721)، head `45cfab03669093511397619f6ac33d31fa536454`، merge commit `59ea8043b51c1abedf80906c999cc4e64fa251d9`.
- **الفحوص على الرأس المدمج:** CI run `43972899224` نجح في backend وfrontend وSDK وpreview-builder وproduction-compose؛ CodeQL run `43972899294` نجح في Python وJavaScript/TypeScript.
- **تحقق main بعد الدمج:** `agent_runtime.py` يحتوي Runtime version 2 وrole gate على MCP؛ `agent_jobs.py` يحتوي redaction للـglobal MCP metadata من استجابات jobs. ظهرت التغييرات على main.
- **Issue #719:** أُغلقت بعد الدمج والتحقق.
- **قيود النطاق:** لم يتم تشغيل أو تفعيل MCP server، ولم تتغير إعدادات الإنتاج؛ الوصول يقتصر على application admins ما دام إعداد MCP مفعلاً، إلى أن يوجد عقد connector grants لكل مستخدم/Workspace.
- **المتبقي:** Issue #718 لصلاحية المهام المجدولة لا تزال مفتوحة. Issue #720 لها PR مستقل #722 لإعادة فحص Project ACLs؛ لا تحتسب منجزة قبل انتهاء فحوصه ومراجعته ودمجه.


## 10. نتيجة إصلاح إعادة تحقق صلاحيات المشروع — 2026-11-17

- **تم الدمج:** [AI Chat App PR #722](https://github.com/HVA09/ai-chat-app/pull/722)، الفرع `security/project-agent-acl-20261117`، head `5ae8595a05dc411901ec0e9a53b4a5670183c624`، merge commit `25a9958efd48de06b96157d47d0e6304d1c71878`.
- **الفحوص على head المدمج:** CI run `43974707209` نجح، وCodeQL run `43974707148` نجح.
- أصبحت إعادة التحقق داخل `_list_project_files` و`_read_project_file` و`_search_project_files` و`_edit_project_file` و`_create_project_archive` باستخدام `can_read_project` و`can_edit_project`.
- اختبارات regression تثبت أن viewer يستطيع القراءة والبحث ولا يستطيع التعديل أو archive overwrite، وأن سحب المشاركة يمنع الاستدعاءات اللاحقة، وأن سحبها أثناء رفع الأرشيف يمنع حفظه في شجرة المشروع ويؤدي إلى تنظيف الملف الذي تم رفعه.
- **Issue #720:** أُغلقت بعد الدمج والتحقق.
- لا migration، ولا نقل بيانات، ولا تغيير إعدادات الإنتاج أو التخزين.

### الحالة الأمنية المتبقية من هذا التدقيق

- #719 / MCP global role gate: PR #721 merged وIssue مغلقة.
- #720 / Project tool ACL rechecks: PR #722 merged وIssue مغلقة.
- #718 / scheduled-task RBAC: لا تزال مفتوحة؛ تحتاج قرار ملكية الموارد واختبارات custom roles ثم PR منفصل.
- لا تُعتبر بقية منظومة الصلاحيات مقبولة بالكامل بسبب إغلاق هذه المسائل الثلاث؛ يظل تدقيق Lumen وERP وعقود AI Core مطلوباً.



## 11. متابعة RBAC للمهام المجدولة — 2026-11-17

- Issue [AI Chat App #718](https://github.com/HVA09/ai-chat-app/issues/718) حُسمت دلالتها: السجل يظل مملوكاً لمنشئه داخل Workspace؛ صلاحية `scheduled_tasks.manage` تسمح للعضو بإدارة مهامه الخاصة فقط ولا تمنح وصولاً إلى مهام أعضاء آخرين أو تشغيلها بهوياتهم.
- PR [AI Chat App #723](https://github.com/HVA09/ai-chat-app/pull/723) مفتوح على الفرع `security/scheduled-task-rbac-20261117`، head `642edc74e08960ed5bf5c01fcf8313275c496ad0`، مبني على main `25a9958efd48de06b96157d47d0e6304d1c71878`.
- النطاق يشمل فحوص permission على create/list/update/delete/run/retry/run-history، مع استمرار owner scope، وإخفاء المهام من listing غير المفلتر عند غياب الصلاحية.
- العامل الخلفي يعيد التحقق قبل أي AI call؛ عند سحب الصلاحية يعطل المهمة ويسجل run فاشلاً دون استهلاك AI quota أو استدعاء المزود.
- اختبارات regression تغطي العضو بلا صلاحية، custom role، عدم عبور ملكية المستخدم، سحب الصلاحية أثناء API requests، وسحبها قبل التنفيذ الخلفي.
- **حالة الفحوص عند آخر تحقق:** CI run `43977627635` وCodeQL run `43977627473` كانا قيد التنفيذ على الرأس نفسه؛ SDK كان ناجحاً وبقية الوظائف لم تكتمل بعد. لا تعتبر #718 أو CHAT-007 مكتملة قبل نجاح الفحوص والمراجعة والدمج.
- لا migrations ولا تغييرات قواعد بيانات/إنتاج، ولا تم تشغيل أي مهمة مجدولة أو AI provider.



## 12. نتيجة إصلاح Workspace RBAC للمهام المجدولة — 2026-11-17

- **تم الدمج:** [AI Chat App PR #723](https://github.com/HVA09/ai-chat-app/pull/723)، head `642edc74e08960ed5bf5c01fcf8313275c496ad0`، merge commit `6645b0502daa6922040ee4c6bb5c281ed59ef366`.
- **الفحوص على الرأس المدمج:** CI run `43977627635` نجح في Backend وFrontend وSDK وPreview Builder وProduction Compose؛ CodeQL run `43977627473` نجح.
- ثبتت دلالة الصلاحية: المهام المجدولة يملكها منشئها؛ `scheduled_tasks.manage` يتيح للعضو إدارة مهامه هو داخل مساحة العمل، ولا يمنح القدرة على الوصول إلى مهام أعضاء آخرين.
- جميع مسارات create/list/update/delete/run/retry/run-history تتحقق من الصلاحية، مع الإبقاء على 404 لغير الأعضاء وعلى عزل `user_id`.
- العامل الخلفي يعيد التحقق قبل أي AI provider call؛ عند سحب الصلاحية، يعطّل المهمة ويسجل run فاشلاً قبل استهلاك AI quota أو استدعاء المزود.
- اختبارات regression تغطي custom roles، منع الوصول إلى مهام الغير، السحب الفوري للصلاحية، وإيقاف worker قبل استدعاء AI.
- **Issue #718:** أُغلقت بعد الدمج والتحقق.
- لم تتم أي migration أو معالجة سجلات جداول أو تغيير إعدادات الإنتاج؛ لم يتم تشغيل أي schedule حقيقي أو استدعاء مزود AI.

### حالة نتائج تدقيق AI Chat App

- #719 / MCP global role gate: PR #721 مدمج، Issue مغلقة.
- #720 / Project tool ACL rechecks: PR #722 مدمج، Issue مغلقة.
- #718 / Scheduled task RBAC: PR #723 مدمج، Issue مغلقة.
- هذه النتيجة تغلق قضايا التدقيق الثلاث المحددة فقط؛ ولا تعني اكتمال تدقيق Lumen أو ERP أو كل الصلاحيات في AI Chat App.


## 13. تدقيق حدود Lumen وERP — 2026-11-17

هذا تدقيق مصدر موجه لحدود الهوية والبيانات وواجهات الخدمة، وليس فحصاً شاملاً لكل endpoint أو كل استعلام. تمت قراءة الملفات المشار إليها على main؛ لا اختبارات محلية أو نشر أو فحص قواعد إنتاج ضمن هذه الجولة.

### Lumen — هوية تعليمية وواجهات تعلم مستقلة

**خط الأساس:** أحدث main commit surfaced في GitHub وقت الفحص: `ed162b0e0af497abc5710d185fa357357baa6ecc`.

- `backend/app/main.py` يجمع routers مستقلة: auth، two-factor، users، curriculum، quiz، content، classes، SRS، assignments، gamification، reports، LTI، files، billing، notifications، admin.
- `backend/app/dependencies.py` يدعم Bearer token أو session cookie، ويتحقق من CSRF لطلبات cookie غير الآمنة، مع `require_admin` و`require_teacher`.
- `backend/app/models/user.py` يستخدم أدواراً عامة على حساب المنصة: `user` و`teacher` و`admin`. هذا ليس نموذج شركة متعدد المستأجرين.
- نطاق الصفوف التعليمية واضح في `ClassSection.teacher_user_id` و`ClassEnrollment.student_user_id`؛ ملكية الفصل للمعلم وتسجيل الطلاب لا تعادل tenant school/company-wide boundary.
- `backend/app/routers/files.py` يقيّد الملف بصاحبه عبر `FileAttachment.user_id`، مع أنواع ملفات وتواقيع محددة، حدود حجم وحصص، وفحص ClamAV عند الإعداد التشغيلي اللازم.
- main يحتوي LTI 1.3 login/launch ومنصات مسجلة تحت `/lti`، وإدارة تسجيل المنصة محمية بـ`require_admin`. لا تُحسب واجهات AGS resource-scoped الجديدة الموجودة في PR #154 ضمن main.
- لم يظهر في مسارات المنتج التي تمت مراجعتها Model Gateway أو Agent Runtime عام مماثل لما هو موجود في AI Chat App؛ لا ينبغي افتراض أن Lumen يملك واجهة AI Core جاهزة دون جرد services إضافي.
- **PR #154:** ما زال Open/Draft/Unmerged، HEAD `6c93b49c976245b778800ed63046a2aef321fbb9`، وBase المسجل `5a065c00ff739c21cad06d551c7e3aa51a3d604b`. مقارنة HEAD الفرع بـmain الحالي: **46 commits ahead / 4 behind**، و`mergeable=false`. CI run `43779761820` نجح 4/4 على ذلك HEAD، لكن هذا لا يغطي تغييرات main اللاحقة ولا يثبت تكاملاً فعلياً مع Moodle/Canvas. البوابات المفتوحة كما يسجل PR: at-least-once/uncertain delivery، clock skew بين العمال، retention/encryption لـ`lti_subject` الخام، إثبات same-origin، وMoodle وCanvas Sandbox كل على حدة.
- **قرار reuse:** احتفظ بملكية Lumen للبيانات التعليمية وLTI؛ أي Agent أو خدمة AI مستقبلية تتلقى فقط سياقاً تعليمياً محدوداً عبر عقد يوافق عليه Lumen، ولا تستخدم اتصالاً مباشراً بقاعدة البيانات أو صلاحيات admin الخاصة بالمنصة.

**مراجع المصدر:**
- [main.py](https://github.com/HVA09/lumen-learning-platform/blob/ed162b0e0af497abc5710d185fa357357baa6ecc/backend/app/main.py)
- [dependencies.py](https://github.com/HVA09/lumen-learning-platform/blob/ed162b0e0af497abc5710d185fa357357baa6ecc/backend/app/dependencies.py)
- [user.py](https://github.com/HVA09/lumen-learning-platform/blob/ed162b0e0af497abc5710d185fa357357baa6ecc/backend/app/models/user.py)
- [class_section.py](https://github.com/HVA09/lumen-learning-platform/blob/ed162b0e0af497abc5710d185fa357357baa6ecc/backend/app/models/class_section.py)
- [lti.py on main](https://github.com/HVA09/lumen-learning-platform/blob/ed162b0e0af497abc5710d185fa357357baa6ecc/backend/app/routers/lti.py)
- [files.py](https://github.com/HVA09/lumen-learning-platform/blob/ed162b0e0af497abc5710d185fa357357baa6ecc/backend/app/routers/files.py)
- [PR #154](https://github.com/HVA09/lumen-learning-platform/pull/154)

### ERP Global Pro — سياق شركة موثوق، لكن تشغيل تعدد الشركات محجوب

**خط الأساس:** أحدث main commit surfaced في GitHub وقت الفحص: `7bb01d23328a68aae67c659fd6c32b6db033637d`. التطبيق الحالي Flask + SQLite v8؛ PostgreSQL هو هدف انتقال مستقبلي وليس قاعدة التشغيل الحالية.

- `backend/erp/config.py` يثبت `TENANT_DATA_SCOPING_ENABLED=False`.
- `backend/erp/__init__.py` يضع `g.company` من هوية موثقة، ويمنع المسارات التشغيلية عندما يزيد عدد الشركات عن واحدة والحاجز مغلقاً؛ يرجع 503 برسالة `tenant_isolation_pending`. هذا fail-closed مقصود وليس دليلاً على جاهزية multi-company.
- `backend/erp/security.py` يضمّن `cid` و`mid` في JWT عند اختيار شركة، ويعيد التحقق من العضوية النشطة والشركة في كل طلب؛ الدور الفعال والصلاحيات يؤخذان من العضوية، لا من الدور العالمي القديم وحده.
- `backend/erp/services/tenancy.py` يوفر `current_company_id()`، ويطابق أي company context صريح مع `g.company` في HTTP requests؛ خارج الطلب يحتاج سياقاً صريحاً أو حالة شركة واحدة فقط.
- محرك CRUD العام في `backend/erp/crud.py` يفرض `company_id` في list/get/update/delete، يحقنه من الخادم عند create، ويرفض client-supplied company_id ويقيد التحقق من المراجع بالشركة نفسها.
- تمت مراجعة عينات من custom APIs: `api/reports.py` يمرر الشركة إلى خدمات التقارير، و`api/data.py` يمررها إلى export/import، و`api/file_objects.py` يستخدم file storage منفصلاً لكل شركة ويقيد list/download/delete بالـcompany context. هذه عينات إيجابية وليست تغطية لكل مسارات SQL والخدمات.
- لا يزال `roles` catalog مشتركاً/عالمياً وفق `ERP_SECURITY_BASELINE.md` و`ERP_MULTI_TENANT_ARCHITECTURE.md`؛ ويجب جعله tenant-aware قبل تعدد الشركات. كما يلزم تدقيق جميع الخدمات والتقارير وjobs وimports/exports وwebhooks والعمليات المالية والقيود المركبة وRLS. وجود generic CRUD scoped لا يثبت أن كل custom SQL scoped.
- `backend/scripts/runtime_preflight.py` يسجل صراحة أن multi-company production غير معتمد وأن `TENANT_DATA_SCOPING_ENABLED` يجب أن يبقى False.
- لم يثبت بعد تشغيل Stage25C preflight على نسخة SQLite v8 فعلية مستعادة من المستخدم؛ لا ETL ولا cutover ولا dual-write ولا تغيير PostgreSQL في هذه الجولة.
- لم يظهر في البحث الموجه لمسارات main AI Model Gateway أو Agent Runtime داخل ERP. لذلك يظل ERP مالك البيانات المالية والتجارية، ويكون أي استدعاء AI من خلال adapter محدود بعد موافقة عقد الخدمة وعزل الشركة، لا عبر وصول مباشر إلى SQLite.

**مراجع المصدر:**
- [application factory](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/__init__.py)
- [config.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/config.py)
- [security.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/security.py)
- [services/tenancy.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/services/tenancy.py)
- [crud.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/crud.py)
- [api/reports.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/api/reports.py)
- [api/data.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/api/data.py)
- [api/file_objects.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/api/file_objects.py)
- [runtime_preflight.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/scripts/runtime_preflight.py)
- [ERP Security Baseline](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/docs/ERP_SECURITY_BASELINE.md)
- [ERP Multi-Tenant Architecture](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/docs/ERP_MULTI_TENANT_ARCHITECTURE.md)

### قرار معماري لاستخدام النتائج

1. AI Chat App هو المنتج الوحيد من الثلاثة الذي ثبت في هذا التدقيق أن main لديه Agent Runtime وTool Registry وMCP وJobs وRAG جاهزة لإعادة الاستخدام بعد تفويض مالك المورد.
2. Lumen يظل مالك الهوية التعليمية والبيانات الدراسية؛ واجهته التعليمية وLTI الحالية لا تُستبدل بـAI Core.
3. ERP يظل مالك سياق الشركة وبيانات الأعمال؛ يبقى التكامل التجاري/المالي محجوباً حتى Stage25C وبوابات العزل/الاستعادة/المراجعة، حتى لو كان هناك بعض tenant-aware APIs الآن.
4. لا توحيد لرقم user ID أو company ID بين المنتجات، ولا مشاركة JWT/session. عند بناء العقود، يكون مرجع الهوية (issuer + subject) منفصلاً عن معرّفات الموارد المحلية، ويُفوض المنتج المالك كل طلب على حدة.
5. لا يُنشأ مستودع AI Core الآن؛ يمكن أن تكون طبقة adapter/Model API مرحلة لاحقة، لكن CORE-001 يبقى In Progress حتى اكتمال الجرد المتبقي واختبار العقود الفعلية.
