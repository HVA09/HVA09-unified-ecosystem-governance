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
