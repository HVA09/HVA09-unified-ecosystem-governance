# خطة التنفيذ المتوازي الموحدة — 50 مهمة عبر المراحل الخمس

- **معرّف البرنامج:** `U50-2026-11-16-01`
- **الإصدار:** 1.3 — 17 نوفمبر 2026 (مطابقة Lumen بعد PR #164 وCI main)
- **الحالة:** خطة تنسيق للتنفيذ المتوازي المشروط بالتبعيات؛ لا تعني بدء أو اكتمال جميع المهام.
- **التركيب:** 10 مهام لكل مرحلة × 5 مراحل = **50 حزمة عمل**.
- **القرار:** DEC-015.
- **المرجع الاستراتيجي:** `docs/UNIFIED_ROADMAP_5_STAGES.md`
- **سجل الحالة:** `docs/TASK_REGISTER.md`

## 1. معنى التنفيذ المتوازي

تُفتح حزم العمل الخمسون للتدقيق والتخطيط وتحديد المسؤوليات والتبعيات ضمن موجة تنسيق واحدة. تبدأ الأعمال المستقلة بالتوازي لتقليل التأخير، لكن ذلك لا يعني دمج 50 تغييراً في `main` أو نشرها دفعة واحدة، ولا تجاوز المدخلات أو بوابات الأمن والقبول.

تبدأ أعمال التحليل والتدقيق والتصميم والاختبارات غير المدمرة عبر المراحل الخمس بالتوازي. أما تغييرات الكود والبيانات والنشر فتبدأ بالتوازي فقط عندما تكون مستقلة ولا تتعارض، ولكل منها فرع وPR واختبارات منفصلة. المهمة التابعة تنتظر شرطها المحدد. تبقى المراحل الخمس وبوابات الانتقال كما هي، والمرحلة السادسة مؤجلة.

## 2. قواعد التشغيل

1. عشر حزم ثابتة لكل مرحلة؛ لا إضافة أو حذف لتعبئة العدد.
2. افحص أحدث main وPRs وIssues وCI قبل التعديل؛ لا تكرر عملاً مدمجاً.
3. لا يعمل فرعان على الملف أو الخدمة أو المخطط نفسه دون تنسيق.
4. توازٍ مشروط بالتبعيات؛ تحضير AI Core وعقود التكامل يمكن أن يبدأ مبكراً، لكن لا تنفيذ تابع قبل مدخلاته.
5. لكل منتج مستودعه وقاعدة بياناته ونشره المستقل؛ لا نقطة فشل مركزية.
6. لا قواعد إنتاج أو أسرار أو بيانات طلاب/عملاء حقيقية في الاختبارات؛ لا migrations أو cutover قبل restore وrollback.
7. لا خدمة مدفوعة أو GPU مدفوع دون موافقة صريحة.
8. لا Done دون SHA/PR/CI أو تقرير اختبار أو قبول يدوي واضح.
9. «اكمل» يدفع المسارات المؤهلة في البرنامج نفسه، ولا يعني إكمال 50 مهمة في رد واحد.
10. المرحلة السادسة للابتكار والتوسع لا تدخل في هذه الخمسين.

## 3. خط الأساس

| المسار | رأس main عند الإعداد | نقطة الاستئناف |
|---|---|---|
| Lumen | `18928282deff93ca9680d30d937ad72675f04cfa` | PR #158/#161/#162 مدمجة؛ PR #154 ما زال Draft/Unmerged عند `6c93b49`، 46 ahead / 5 behind وmergeable=false؛ يحتاج rebase وإعادة CI ثم بوابات Moodle/Canvas والخصوصية/clock/worker. |
| AI Chat App | `6645b0502daa6922040ee4c6bb5c281ed59ef366` | PR #717 دمج مصالحة A–G وH–T؛ #721/#722/#723 أصلحت MCP role gate وproject ACL وscheduled-task RBAC مع CI/CodeQL ناجحين؛ PR #665 يبقى Draft ولا يدمج ككتلة. |
| ERP | `7bb01d23328a68aae67c659fd6c32b6db033637d` | SQLite v8 نشطة؛ Stage25C على نسخة فعلية مستعادة غير مثبت؛ TENANT_DATA_SCOPING_ENABLED=False. |
| AI Core | لا يوجد خط أساس برمجي معتمد | يبدأ بجرد القدرات والمتطلبات، بلا شراء GPU أو تكرار قدرات. |
| Unified Ecosystem | المستودع المركزي | حوكمة وعقود مشتركة فقط، لا كود منتجات أو DBs. |

هذه نقاط استئناف مؤرخة وليست بديلاً عن قراءة SHA الحي قبل كل تغيير.


## المرحلة الأولى — Lumen: 10 مهام

| المعرّف | حزمة العمل | الحالة الأولية | الاعتماديات / معيار القبول |
|---|---|---|---|
| S1-01 | تهيئة أول مدير ومسار صلاحيات التشغيل المقيد | **Done — merged + CI evidence** | PR #158 أضاف bootstrap المحروس وCI #43957299993 نجح 4/4؛ PR #161 أزال CLI القديم `backend/app/create_admin.py` وصحح README؛ PR #164 وحّد قفل bootstrap/recovery مع roster النشط. Lumen main الحالي `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`. لم يُشغّل أمر ترقية/استعادة مدير على حساب حقيقي أو قاعدة إنتاج. **مرجع سجل المنتج: LUM-006 وLUM-010؛ لا يُحسب التغيير مرتين.** |
| S1-02 | مصفوفة الحسابات والجلسات وأدوار الطالب والمعلم والمدير | In Progress — route audit + content-ownership decision | آخر رأس Lumen `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`. PR #162/#163 أغلقا class-join وassignment-submit؛ PR #164 يحمي آخر مدير نشط بقفل PostgreSQL مشترك وre-read للـactor/target بعد القفل واختبارات سباق ودليل recovery؛ CI PR #44149632805 وCI main #44150551971 نجحا 4/4. **الملاحظة غير المحسومة:** هل إدارة محتوى الأسئلة والدروس مشتركة لكل المعلمين أم مقيدة بمنشئ المورد؟ `content.py` لا يفحص owner في update/delete، وLesson لا يملك owner field؛ يلزم اعتماد السياسة وإضافة cross-teacher/IDOR tests قبل تغيير السلوك. تبقى route-by-route API/UI inventory مفتوحة. **مراجع السجل: LUM-008 وLUM-009 وPR #164 مكتملة؛ LUM-010 In Progress.** |
| S1-03 | دورة المقرر والتسجيل وإدارة الصفوف | Audit / Ready | إنشاء/نشر/أرشفة وتسجيل/إلغاء التسجيل؛ المعلم يرى صفوفه فقط؛ Backend وBrowser E2E. |
| S1-04 | دورة الدروس والمحتوى والتقدم | Audit / Ready | نشر وإخفاء الدرس، منع الطالب من رؤية غير المنشور، استمرار completion بعد reload. |
| S1-05 | الواجبات والاختبارات والتسليم والتصحيح | In Progress — submission-role fix merged | PR #163 مدمج عند `a386ead55935edd9cfe2c51f109ab894629985cb`، وCI #44019697516 نجح 4/4. التغيير يفرض `UserRole.user` على `/assignments/{id}/submit` ويختبر رفض teacher/admin دون آثار جانبية ونجاح الطالب المسجل. تبقى مراجعة دورة الواجبات/الاختبارات كاملة ومعايير القبول الأوسع. **مرجع سجل المنتج: LUM-009؛ لا تنشئ إصلاحاً مكرراً.** |
| S1-06 | تقدم المتعلم والتقارير ولوحات المتابعة | Audit / Ready | اتساق completion والدرجات والتجميعات حسب الطالب/المعلم/المدير، مع اختبارات صلاحيات. |
| S1-07 | رفع الملفات والمرفقات والفحص والحذف والاستعادة | Audit / Ready | رحلة ملف كاملة، فحص ضار ورفض آمن، لا orphan records، واختبار backup/restore للملفات والبيانات. |
| S1-08 | قبول الهاتف وRTL وإمكانية الوصول | Open acceptance | يتطلب قبولاً يدوياً على هاتف فعلي: الدخول والتنقل والمقرر والدرس والواجب والملفات وRTL/LTR والتركيز ولوحة المفاتيح. نتائج Browser E2E عند 390×844 لا تثبت قبول جهاز فعلي. `LUM-001` يغطي مراجعة الوظائف وBrowser E2E فقط، ولا يغلق هذا القبول اليدوي. |
| S1-09 | عزل مختبر Python وحدود التنفيذ والموارد | In Progress / Browser acceptance | Issue #134: opaque origin وCSP ورسائل آمنة وإلغاء وحد زمني وقيود الموارد؛ لا ادعاء OS sandbox، ولا حد ذاكرة صلب موثق. **مرجع سجل المنتج: LUM-007؛ نفس مسار القبول.** |
| S1-10 | LTI 1.3 وAGS وربط الدرجة بالمورد الأصلي | Blocked on external gates | PR #154 وIssue #150؛ resource A/B وnon-LTI، Moodle وCanvas Sandbox، same-origin، retention لـ`lti_subject`، clock skew/workers وإعادة مزامنة الفرع مع main. **مراجع سجل المنتج: LUM-002 (AGS) وLUM-003 (Sandbox)؛ لا تُحسب كمهام إضافية.** |

## المرحلة الثانية — AI Chat App: 10 مهام

| المعرّف | حزمة العمل | الحالة الأولية | الاعتماديات / معيار القبول |
|---|---|---|---|
| S2-01 | إغلاق بوابة الاستقرار التشغيلي H | Open acceptance | إعادة فحص health/ready والنسخ والاستعادة وحماية main وCI/CodeQL وAuto Deploy مقابل render.yaml. **مرتبط بـCHAT-005/CHAT-006؛ يبقى السجل المحلي مرجع الحالة.** |
| S2-02 | قبول الهاتف وإمكانية الوصول I | Open manual acceptance | اختبار حقيقي للدخول والقائمة وتبديل المحادثة والإرسال والملفات والميكروفون ولوحة المفاتيح وRTL. **مرتبط بقبول UX/mobile في CHAT-005؛ لم يكتمل القبول اليدوي.** |
| S2-03 | صيانة معمارية الواجهة J | Ready for audit | قياس App.jsx/hooks ودورة الطلبات والإلغاء؛ فصل مسؤوليات محدود واختبارات سلوكية، بلا إعادة كتابة واسعة. |
| S2-04 | الأداء والاستجابة K | Ready for baseline | قياس bundle والتحميل والتفاعل وإعادة التصيير والقوائم والمحادثات وطلبات API؛ مقارنة قبل/بعد. |
| S2-05 | دورة المحادثة والرسائل والوسائط L | Ready for audit | إرسال/streaming/إلغاء/retry/edit/regenerate/delete/share وصوت/صور؛ لا ازدواج ولا فقد رسائل عند الفشل. |
| S2-06 | الملفات والمعرفة وRAG M | In Progress — gated | دورة ملف وفهرسة وحذف ومنع التكرار وعزل Workspace؛ تقييم retrieval؛ restore إلى staging قبل backfill/reindex. **مراجع سجل المنتج: CHAT-010 وCORE-005؛ PR #725 وIssue #724 ما زالا ينتظران قبول سياسة الاحتفاظ.** |
| S2-07 | Agent وProject Builder N | Audit / Ready | مشروع end-to-end وcheckpoints/retry/cancel/resume وفشل build آمن وحدود time/cost/tools وعزل preview. |
| S2-08 | Workspace والتعاون والصلاحيات O | Audit / Ready | دعوات وأدوار وسحب وصول وقراءة فقط وتعاون؛ cross-user/role-based backend tests. PRs #721/#722/#723 أغلقت ثلاث فجوات محددة فقط؛ **مراجعها CHAT-007/CHAT-008/CHAT-009 (Done)، ولا تعني اكتمال قبول Workspace ككل.** |
| S2-09 | Developer API وSDKs وWebhooks P | Audit / Ready | توافق API v1 مع Python/JS SDK، versioning وrequest IDs وrate limits وretry/webhooks/OAuth. |
| S2-10 | الأمن والخصوصية والاعتمادية والتوثيق والإصدار Q–T | Parallel audit | IDOR/BOLA والأسرار والاحتفاظ والحذف والمراقبة والحوادث والنسخ والrollback وREADME والترخيص؛ release بعد قبول وCI/Smoke. |

## المرحلة الثالثة — ERP Global Pro: 10 مهام

| المعرّف | حزمة العمل | الحالة الأولية | الاعتماديات / معيار القبول |
|---|---|---|---|
| S3-01 | حفظ النسخة الأصلية وخط الأساس القابل للاستعادة | Ready / Verify | SHA-256 للأرشيف، dependencies، اختبارات نظيفة، مؤشرات مالية/مخزون، نسخة احتياطية مستعادة؛ لا تعديل للأصل. |
| S3-02 | Stage25C على نسخة SQLite v8 فعلية مستعادة | Blocked — needs user backup | نسخة مستقلة ومصرح بها مع SHA/time/source وintegrity_check وforeign_key_check وuniqueness preflight read-only. **مرجع سجل المنتج: ERP-002؛ الحالة Blocked حتى توفر نسخة فعلية مستعادة.** |
| S3-03 | مصفوفة التفويض وعضويات الشركة والسياق الموقّع | In Progress / Gated | membership/roles/suspension/expiry/replay/tampering؛ لا اعتبار GUC العميل هوية موثوقة. |
| S3-04 | تدقيق عزل CRUD والمراجع والمسارات المخصصة | In Progress | اختبارات شركتين تمنع list/get/update/delete/reference/IDOR المتبادل عبر resources وcallbacks. |
| S3-05 | عزل الخدمات التشغيلية والمخزون والمبيعات والمشتريات والإعدادات | Ready after audit | company context في callbacks وinventory/accounting/settings/sequences/POS/purchasing؛ rollback واختبارات شركتين. |
| S3-06 | سلامة المالية والضرائب والأصول والمدفوعات والتسوية | Ready after data baseline | company-scoped journals/periods/payments/tax/assets/currencies؛ تسوية Trial Balance وP&L والميزانية والتدفق والمخزون. |
| S3-07 | التقارير والاستيراد والتصدير والتدقيق والمهام الخلفية | Audit / In Progress | مراجعة Stage14 ثم HTTP tests فعلية للـdashboard/import/export/sequences/audit/webhooks/jobs. |
| S3-08 | الملفات الخاصة والفحص والحصص والنسخ الذري | Ready | quarantine وAV قبل التنزيل، tenant quotas/retention، atomic DB+file backup/restore وorphan dry-run. |
| S3-09 | PostgreSQL schema وETL وRLS والقيود المركبة | Deferred until preflight | Disposable PG فقط؛ migrations/ledger وcomposite FK/unique وETL/reconciliation/rollback؛ لا dual-write ولا app cutover. |
| S3-10 | بوابة إطلاق تعدد الشركات وحزم القطاعات | Feature gate closed | لا تفعيل `TENANT_DATA_SCOPING_ENABLED` قبل اكتمال isolation/restore/security؛ Industry Packs اختيارية ولا ادعاء امتثال دون مراجعة. **مرجع سجل المنتج: ERP-004؛ يبقى العلم False.** |

## المرحلة الرابعة — AI Core: 10 مهام

| المعرّف | حزمة العمل | الحالة الأولية | الاعتماديات / معيار القبول |
|---|---|---|---|
| S4-01 | جرد قدرات AI الحالية ومنع التكرار | **In Progress — AI Chat authorization fixes merged; Lumen/ERP service-boundary audit recorded** | `AI_CORE_CAPABILITY_INVENTORY.md` يوثق AI Chat App على main `6645b0502daa6922040ee4c6bb5c281ed59ef366` مع دمج #721/#722/#723 وفحوص CI/CodeQL الناجحة، ويضيف جرد الهوية والواجهات وحدود البيانات في Lumen main `ed162b0e0af497abc5710d185fa357357baa6ecc` وERP main `7bb01d23328a68aae67c659fd6c32b6db033637d`. يبقى تدقيق service/route الكامل والاختبارات وعقود إعادة الاستخدام قبل إغلاق CORE-001؛ لا إنشاء بدائل AI runtime. |
| S4-02 | حالات الاستخدام ومتطلبات الوكلاء المتخصصين | **In Progress — requirements baseline documented** | `docs/AI_CORE_SPECIALIZED_AGENT_REQUIREMENTS.md` يحدد ملفات وكلاء البرمجة وLinux والشبكات وCloud/DevOps والأمن، وتحليل الملفات المشترك، وعقود المدخلات/المخرجات والحدود والاختبارات؛ تبقى مراجعة المتطلبات وربطها بعقود الأدوات والـbenchmark. |
| S4-03 | تقييم نماذج مفتوحة الأوزان منخفضة التكلفة | Design / No paid GPU | مقارنة الجودة والسياق والخصوصية والسرعة والترخيص والعتاد؛ benchmark على موارد متاحة قبل الاختيار. |
| S4-04 | عقد Model Gateway وسجل النماذج والمزودين | **In Progress — design baseline documented** | `AI_CORE_MODEL_GATEWAY_DESIGN.md` يحدد scope وحدود reuse من Provider Factory الحالية، Model Registry/capabilities، service identity، request/response/stream contracts، routing/fallback/limits/cost/privacy/tests؛ لا نموذج مختار ولا Gateway service/API منفذ، ويتطلب S4-01/S5-03/S5-05 قبل pilot. |
| S4-05 | Orchestrator ودورة حياة المهمة والوكيل | Design in parallel | job states/checkpoints/cancel/resume/queues/idempotency/time-cost limits؛ لا تكرار runtime الموجود دون مبرر. |
| S4-06 | ملفات الوكلاء المتخصصين ومصفوفة الأدوات | **In Progress — mapped to current AI Chat App Tool Registry** | `AI_CORE_AGENT_TOOL_MATRIX.md` يربط تسع أدوات مدمجة فعلية (calculator/python/web_search/analyze_data وأدوات ملفات المشروع) بالتخصصات الخمسة، ويحدد read/write boundaries وuntrusted outputs وMCP admin-only. ToolSpec لا يفرض بعد permission_level/profile/scope/approval؛ يلزم ToolPolicy enforcement واختبارات negative قبل تفعيل صلاحيات جديدة. |
| S4-07 | تكامل MCP/Tools وطبقة الموافقات | Requires S4-01 | محولات مع allowlists/scopes/approvals للأفعال الحساسة وسجل تدقيق؛ إعادة استخدام MCP الحالي حيث يصلح. |
| S4-08 | RAG والذاكرة والمعرفة وعزل النطاقات | **In Progress — private file-retention proposal passes CI/CodeQL; owner review pending** | `AI_CORE_CAPABILITY_INVENTORY.md` يوثق RAG/embeddings وUserMemory/ProjectMemory وACL وإعادة الفهرسة في AI Chat App؛ Issue #724 مفتوحة وPR #725 يقترح Retain privately للمرفقات مع إزالة روابط المساعدين المشتركين واختبارات منع workspace/RAG access. CI #43990553917 وCodeQL #43990553931 نجحا على HEAD المحدد؛ PR لا يزال Draft ويتطلب مراجعة واعتماد سياسة الاحتفاظ من المالك قبل الدمج. |
| S4-09 | Benchmark وتقييم الجودة والسلامة والحقائق | **In Progress — offline evaluation candidate audited; no live baseline** | `AI_CORE_CAPABILITY_INVENTORY.md` يوثق candidate في PR #665: 20 query/100 doc RAG set وmetrics Recall@k/Precision@k/nDCG@k/MRR ومقارنة case-level، إضافةً إلى 37 specialist cases وrubric 0–4 وفحص Citation ID consistency. هذه الأدوات غير موجودة في main، والـPR Draft/mergeable=false، وCI غير ظاهر على رأسه؛ `--validate-only` يتحقق من fixture فقط ولا يحسب الأداء، و`--run-live` لم يشغل. يلزم فصل/مزامنة PR مستقل، CI/CodeQL، اعتماد rubric، baseline مصرح، وsemantic citation/security/end-to-end tests قبل Verified. |
| S4-10 | تشغيل النواة ومراقبتها وتكلفتها والتعافي | Design / gated implementation | نشر مستقل وhealth/tracing/limits/circuit breakers/fallback/secrets/backup/kill switch؛ فشل النواة لا يعطل المنتجات. |

## المرحلة الخامسة — Unified Ecosystem: 10 مهام

| المعرّف | حزمة العمل | الحالة الأولية | الاعتماديات / معيار القبول |
|---|---|---|---|
| S5-01 | سجل ملكية المنتجات ومصادر الحقيقة وحدود البيانات | **Done — architecture register** | `docs/PRODUCT_OWNERSHIP_AND_INTERFACE_CATALOG.md` يوثق المالك والمستودع ومصدر بيانات كل منتج وحدود الهوية والبيانات وعقود التكامل المرشحة؛ التنفيذ البرمجي للعقود يبقى ضمن S5-02 وما بعدها. |
| S5-02 | كتالوج عقود API وإصداراتها | **In Progress — route groups and three grounded API contract cards documented** | `PRODUCT_API_CONTRACT_CATALOG.md` يضيف `AIC-DEV-CHAT-001` لـDeveloper API v1، و`LUM-LEARN-001` لمسارات curriculum وبيانات التعلم الشخصية، و`ERP-COMPANY-CONTEXT-001` لتبديل الشركة والتحقق من membership. يميز developer API key الشخصية، Lumen student progress، وERP JWT التفاعلي عن service identity. عقد ERP موجود لكنه Blocked للتكامل متعدد الشركات؛ المتبقي schemas لبقية endpoints وservice delegation/scopes وprovider/consumer tests وstaging acceptance. |
| S5-03 | هوية الخدمات والثقة بين المنتجات | **In Progress — delegation flows mapped to three verified product contracts** | `SERVICE_IDENTITY_AND_DELEGATION_DESIGN.md` يربط `LUM-LEARN-001` و`AIC-DEV-CHAT-001` و`ERP-COMPANY-CONTEXT-001` بتدفقات service principal + delegated subject، ويحدد scopes candidate ومصادر الثقة والـowner checks. لا استخدام لمفتاح AI Chat الشخصي أو JWT ERP التفاعلي كاعتماد خدمة؛ اختيار private_key_jwt مقابل mTLS والـverifier/contract tests ما زال مفتوحاً. |
| S5-04 | نموذج التفويض RBAC/ABAC ونطاق الموارد | **In Progress — cross-product authorization model documented** | `UNIFIED_AUTHORIZATION_MODEL_DESIGN.md` maps Lumen user/teacher/admin + class ownership, AI Chat workspace custom RBAC + project viewer/editor/manager ACL, and ERP membership-scoped roles with a global role catalog. Defines actor/action/resource/tenant/purpose, intersected scopes, PEP ownership, T0–T3 approval gates and negative IDOR/revocation tests. No shared policy engine or global admin role is implemented. |
| S5-05 | تصنيف البيانات وتدفقها والاحتفاظ والخصوصية | **In Progress — classification/retention design baseline documented** | `DATA_CLASSIFICATION_AND_RETENTION_DESIGN.md` يقترح Public/Internal/Confidential/Restricted/Secret/Unknown، data-flow purpose, model/provider policy, retention/deletion propagation, logging minimization واختبارات deny/IDOR. لا policy engine أو classification labels مطبقة بعد؛ تحتاج مراجعة مالكي البيانات والقطاع/البلد قبل الاعتماد. |
| S5-06 | بوابة التكامل وموصلات الأحداث وWebhooks | Design / contract required | event/webhook contracts وsignature/idempotency/retries/DLQ/allowlists/rate limits؛ لا production integration قبل الاختبار. |
| S5-07 | تنسيق workflows والمهام العابرة للمنتجات | Design / gated | orchestration boundary وcompensation/idempotency/timeouts؛ تعطل المنسق لا يوقف التطبيقات. |
| S5-08 | المراقبة والتدقيق والتتبع المشترك | Design in parallel | correlation IDs/audit schema/retention/dashboards/alerts/latency-errors؛ لا تجميع محتوى حساس. |
| S5-09 | اختبارات توافق التكامل والفشل والعزل | Requires contracts | consumer/provider contract tests/schema compatibility/fault injection/auth denial/data isolation/fallback. |
| S5-10 | التعافي والحوادث والنسخ والrollback ومفاتيح الإيقاف | Design in parallel | Runbooks/ownership/backup-restore/rollback/kill switch/incident/postmortem؛ استقلال تعافي كل منتج. |

## 4. طريقة تشغيل الموجة

### موجة A — تبدأ بالتوازي
- **Lumen:** تدقيق قبول الهاتف، مصفوفة الأدوار، عزل المختبر وLTI دون تعديل PR #154 عشوائياً.
- **AI Chat App:** إعادة التحقق من H/I والهاتف والـPRs والإعدادات الحية.
- **ERP:** حفظ خط الأساس وتجهيز نسخة SQLite v8 المستعادة لـStage25C؛ القراءة فقط حتى اكتمال النسخة.
- **AI Core:** جرد القدرات الحالية واستخلاص المتطلبات دون موارد مدفوعة.
- **Unified Ecosystem:** جرد الملكية والعقود وتدفق البيانات دون SSO أو ربط إنتاجي.

### موجة B — تطوير مستقل
تُفتح PRs صغيرة في المنتجات التي اجتازت فحص التبعيات. يجوز العمل على المستودعات الثلاثة بالتوازي، مع منع تعديل الملف أو الخدمة نفسها من أكثر من فرع، ومنع تغيير قاعدة أو إعداد إنتاج منتج آخر.

### موجة C — تكامل مشروط
لا تُشغّل AI Core أو Unified Ecosystem كخدمات إنتاجية قبل جرد القدرات والعقود ونموذج التهديد. ولا يحدث ERP multi-company أو PostgreSQL cutover قبل نجاح preflight والاستعادة والعزل والـrollback.

## 5. تعريف الإنجاز والتقرير

لكل حزمة عمل: المعرّف، المالك، المستودع، النطاق المستثنى، الحالة، التبعيات، SHA والفرع وPR، اختبارات القبول ونتائج CI، المراجعة اليدوية، قرار الدمج وخطة rollback. الحالات: Proposed، Ready، In Progress، Blocked، Review، Verified، Done، Deferred، Rejected. لا تستخدم نسبة إنجاز كلية مضللة؛ اعرض المكتمل والجاري والمحجوب بأدلته.

## 6. المراجع المصدرية

- **Lumen:** `PROJECT_STATUS.md`، `PROJECT_CHARTER.md`، `ARCHITECTURE_PLAN.md`، `CONTINUITY.md`، PR #154، Issues #134/#150.
- **AI Chat App:** `AI_PLATFORM_ROADMAP_AR.md`، `POST_RELEASE_ROADMAP_H_TO_T_AR.md`، `PR-665_SPLIT_TRACKER_AR.md`، Issues #683 و#674؛ راجع main الحي قبل أي تعديل.
- **ERP:** `ERP_IMPLEMENTATION_BACKLOG.md`، `ERP_GLOBAL_PRO_CONTINUITY_AR.md`، `ERP_MULTI_TENANT_ARCHITECTURE.md`، Stage25C runbook.
- **AI Core / Unified Ecosystem:** `AI_CORE_CAPABILITY_INVENTORY.md` و`PRODUCT_OWNERSHIP_AND_INTERFACE_CATALOG.md` و`UNIFIED_ENGINEERING_STANDARDS.md` و`DECISION_REGISTER.md` و`CONFLICTS_AND_RECONCILIATION.md`؛ إنشاء مستودعات مستقلة بعد اعتماد التصميم والحدود فقط.

هذه الخطة تنظّم 50 حزمة عمل، ولا تعلن أن المهام بدأت أو اكتملت. تتغير الحالة فقط مع تنفيذ العمل وتسجيل دليله.


## 7. خريطة الربط مع سجلات المشاريع ومنع تكرار العمل

معرّفات `S1–S5` هي معرفات حزم برنامج الخمسين، ومعرّفات `LUM/CHAT/ERP/CORE/ECO` هي سجلات متابعة مرتبطة بالمستودعات والحوكمة. الربط أدناه **مرجعي وليس إنشاءً لمهام إضافية**. عند وجود تطابق أو تداخل، يُحدّث سجل المشروع المحلي ويظل هو مرجع حالة التنفيذ؛ لا يُنشأ PR ثانٍ لنفس التغيير.

| حزم البرنامج | معرفات السجل المرتبطة | قاعدة منع التكرار |
|---|---|---|
| S1-01 | LUM-006 | نفس مسار bootstrap؛ Done وفق PR #158 و#161، مع بقاء قاعدة عدم تشغيل bootstrap على حساب حقيقي دون تفويض تشغيلي مستقل. |
| S1-02 | LUM-008، LUM-009، LUM-010 | PR #162 أغلق class-join (LUM-008 Done)؛ PR #163 أغلق assignment-submit (LUM-009 Done)؛ PR #164 أغلق حماية آخر مدير نشط مع CI كامل وrunbook. LUM-010 يبقى مفتوحاً لقرار ملكية محتوى السؤال/الدرس بين المعلمين وجرد endpoints/UI واختبارات cross-teacher IDOR؛ لا يُغلق S1-02 قبل قبول هذه البوابات. |
| S1-05 | LUM-009 | PR #163 مدمج وCI #44019697516 نجح 4/4 لبوابة student-only submission؛ تبقى دورة الواجبات/التصحيح/الاختبارات الأوسع مفتوحة. |
| S1-08 | LUM-001 | دليل جزئي فقط: LUM-001 يغطي مراجعة الوظائف وBrowser E2E؛ قبول الهاتف الفعلي في S1-08 ما زال مفتوحاً. |
| S1-09 | LUM-007 | نفس مسار قبول مختبر Python؛ يبقى In Progress حتى حد الموارد وقبول الجهاز/المتصفح. |
| S1-10 | LUM-002، LUM-003 | AGS في LUM-002 وSandbox في LUM-003؛ لا دمج قبل البوابات الخارجية. |
| S2-01 | CHAT-005، CHAT-006 | مسارات UX والاستقرار وQueuePool؛ لا تكرر إصلاحاً مدمجاً من #687/#664/#676. |
| S2-02 | CHAT-005 | قبول الهاتف وRTL؛ يتطلب اختباراً يدوياً منفصلاً. |
| S2-06 | CHAT-010، CORE-005 | دورة المرفقات/RAG والاحتفاظ؛ PR #725 يحتاج اعتماد سياسة المالك. |
| S2-08 | CHAT-007، CHAT-008، CHAT-009 | ثلاثة إصلاحات محددة مدمجة؛ لا يعني ذلك إغلاق تدقيق Workspace الأوسع. |
| S3-02 | ERP-002 | نفس بوابة Stage25C؛ تبقى Blocked حتى النسخة المستعادة الفعلية. |
| S3-03، S3-04، S3-10 | ERP-004، ERP-005 | ارتباط بحدود multi-company وسياق الشركة؛ لا تفعيل أو cutover قبل البوابات. |
| S4-01 | CORE-001 | سجل جرد القدرات ومنع التكرار. |
| S4-02 | CORE-003 | متطلبات الوكلاء المتخصصين. |
| S4-04 | CORE-007 | تصميم Model Gateway. |
| S4-06 | CORE-004 | مصفوفة أدوات وصلاحيات الوكلاء. |
| S4-08 | CORE-005 | RAG والذاكرة ودورة حياة المعرفة. |
| S4-09 | CORE-006 | Benchmark والتقييم. |
| S5-01 | ECO-003 | سجل الملكية وحدود الواجهات. |
| S5-02 | ECO-001 | عقود API وإصداراتها. |
| S5-03 | ECO-002 | هوية الخدمات والتفويض. |
| S5-04 | ECO-005 | نموذج التفويض RBAC/ABAC. |
| S5-05 | ECO-004 | تصنيف البيانات والاحتفاظ والخصوصية. |

لا يوجد تطابق حالي معلن لباقي الحزم؛ تظل حزم البرنامج قابلة للتتبع بمعرفها، ويُضاف الربط إلى سجل المشروع عند إنشاء المهمة هناك. لا يُحوّل تداخل جزئي إلى حالة Done تلقائياً.
