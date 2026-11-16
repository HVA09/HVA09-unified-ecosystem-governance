# سجل المهام الموحد

يُستخدم هذا السجل لمنع فقدان العمل أو تكراره أو تنفيذه في المستودع الخطأ. تحديث الحالة يتطلب دليلاً. «Done» لا تعني أن المهمة خُطط لها أو وُثقت فقط.

هذا السجل قائمة أعمال حية لكل مشروع. لا توجد قاعدة خمس مهام لكل مستودع أو دفعة 15 مهمة. البرنامج التنسيقي الحالي يضم 10 حزم لكل مرحلة من المراحل الخمس (50 إجمالاً) وفق `docs/UNIFIED_50_TASK_EXECUTION_PLAN.md`؛ معرفات LUM/CHAT/ERP تبقى مهاماً في مصادرها ولا تُنقل ملكيتها إلى هذا السجل.

## قاموس الحالات

- **Done:** اكتملت وفق معيار القبول وبالدليل المحدد.
- **Verified:** تم التحقق من النتيجة، وقد يتبقى إجراء اعتماد أو دمج.
- **Ready:** جاهزة للتنفيذ عند بلوغ المرحلة المناسبة واكتمال التبعيات.
- **In Progress:** يجري تنفيذها.
- **Blocked:** لا يمكن المتابعة حتى إزالة عائق محدد.
- **Deferred:** مؤجلة بقرار.
- **Proposed:** اقتراح لم يعتمد كعمل نشط.
- **Review:** تنتظر مراجعة.
- **Rejected:** ألغيت مع سبب مسجل.

## سجل المهام
| ID | المسار | المهمة | الحالة | الأولوية | التبعية/معيار القبول |
|---|---|---|---|---|---|
| GOV-001 | Governance | اعتماد المرجع الرئيسي ذي المراحل الخمس | Done | P0 | قرار المستخدم محفوظ؛ مرجع v1.6 موجود في main، وتصحيح v1.7 موثق في PR مستقل |
| GOV-002 | Governance | جرد وثائق المشاريع وحفظ نسخ مؤرخة قابلة للتتبع | Done | P0 | 171 ملفاً محفوظاً ومطابقاً 171/171 على Git Blob؛ 150 من main و21 من PRs غير المدمجة |
| GOV-003 | Governance | بناء سجل التعارضات وسياسة أولوية المصدر | Done | P0 | `CONFLICTS_AND_RECONCILIATION.md` يبين الاختلافات المفتوحة وحدود الحسم |
| GOV-004 | Governance | مزامنة الفهرس والحالة عند كل دفعة كبيرة أو تغير معماري | Ready | P1 | تحديث Snapshot جديد وSHA ودليل المصدر؛ لا تعديل اللقطات القديمة |
| GOV-005 | Governance | مراجعة نقاشات Issues/PR comments كاملة عند الحاجة إلى قرار تاريخي | Proposed | P2 | حفظ الروابط والملخصات؛ لا تُنسخ التعليقات الشخصية عشوائياً |
| GOV-006 | Governance | اعتماد الميثاق الهندسي المشترك كمرجع واحد مع إبقاء ملاحق المشاريع | Done | P1 | `docs/UNIFIED_ENGINEERING_STANDARDS.md` يضم الأقسام المشتركة المتطابقة؛ الملاحق المحلية محفوظة في كل منتج |
| GOV-007 | Governance | دمج التفاصيل المعمارية الفريدة من نسخة Library v0.11 في المرجع المركزي ومنع ازدواج المصدر | Done | P1 | المحتوى الفريد مدمج في المرجع المركزي؛ نسخة Library v0.11 لقطة تاريخية وليست مصدراً موازياً |
| GOV-008 | Governance | إعداد وصيانة خارطة الطريق الموحدة ذات المراحل الخمس والمرحلة السادسة المؤجلة | In Progress | P0 | `docs/UNIFIED_ROADMAP_5_STAGES.md`؛ المراحل والنطاق والبوابات موثقة في PR #2؛ تبقى حالة PR والقبول بعد الدمج مفتوحة |
| GOV-009 | Governance | تنسيق برنامج التنفيذ المتوازي ذي 50 مهمة (10 لكل مرحلة) | In Progress | P0 | `docs/UNIFIED_50_TASK_EXECUTION_PLAN.md`؛ 50 حزمة بمصادر واعتماديات ومعايير قبول؛ تحديث الحالة بالأدلة وعدم اعتبار التوازي تجاوزاً للبوابات |
| LUM-001 | Lumen | مواصلة تحسين الوظائف التعليمية الأساسية دون انتظار LMS Sandbox | Ready | P0 | خطة عمل مفصلة من main واختبارات على الفرع المستهدف |
| LUM-002 | Lumen/LTI | مصالحة ربط AGS بالمورد الأصلي والتحقق من سلسلة PRs #16–#23 | In Progress | P1 | PR #154 يحتوي ربطاً دقيقاً واختبارات regression لـresource A/B وnon-LTI؛ تبقى مزامنة main وMoodle/Canvas Sandbox والخصوصية والساعة/العمال قبل إغلاق Issue #150 |
| LUM-003 | Lumen/LTI | اختبار Moodle ثم Canvas Sandbox | Deferred | P1 | بيئة Sandbox مستقلة، HTTPS، حسابات اختبار، نتائج موثقة لكل منصة |
| LUM-004 | Lumen/Docs | مصالحة وثائق PR #6 المعمارية مع وثائق main الحالية وتحديد النقاط الفريدة فقط | Ready | P2 | PR #6 يبقى Draft؛ مقارنة الملفات الخمسة مع main، ثم اقتراح تحديث صغير داخل Lumen إذا بقي محتوى صالح غير موثق |
| LUM-005 | Lumen/Docs | مزامنة الحالة وتوضيح نقطة الاستئناف التاريخية | Done | P1 | PR #157 مدمج عند `9d766cc30525c706bd867d9736b15f74dc2acac7`؛ CI #43949038511 نجح 4/4 على HEAD `7529266`؛ تم وسم نقطة 7 نوفمبر تاريخية دون حذفها |
| LUM-006 | Lumen/Security | تهيئة أول مدير بأمان عبر أمر تشغيل مستقل | Done | P1 | PR #158 مدمج في main عند `108c2eabdc4a180bc5efd5469c368ab3c3f15199`؛ CI #43957299993 نجح 4/4 على رأس PR `10e4a726`؛ لم يُشغّل أمر الترقية على حساب حقيقي أو قاعدة إنتاج |
| LUM-007 | Lumen/Security | استكمال قبول عزل مختبر Python وحدود الذاكرة والمتصفح | In Progress | P1 | PR #146 مدمج؛ CI #42405345735 أثبت opaque origin وCSP/DOM/storage؛ Issue #134 ما زالت مفتوحة لقبول المتصفح المستهدف وتأكيد قيود Worker memory وعدم كونه OS sandbox |
| CHAT-001 | AI Chat App/Docs | مصالحة خارطتي A–G وH–T مع main والـPRs/Issues | Done | P1 | PR #717 مدمج عند `17c9868fc17a37a1743c4a49cdb378d93160650c`؛ CI #43952642164 (Backend/Frontend/SDK/Preview Builder/Production Compose) وCodeQL #43952642232 نجحا على HEAD `d5ccb16` |
| CHAT-002 | AI Chat App | تفكيك العمل المتبقي في PR #665 إلى PRs صغيرة | In Progress | P1 | اتباع tracker المدمج عبر #714؛ لا دمج #665 ككتلة |
| CHAT-003 | AI Chat App/Docs | حسم PR #671 القديم دون فقد أرشيفه | Done | P2 | PR #671 مغلق دون Merge بتاريخ 16 نوفمبر؛ ملفاه محفوظان في Snapshot المركزي، والسبب Base قديم وحالة تشغيل متقادمة |
| CHAT-004 | AI Chat App/Docs | مصالحة سجل تفكيك PR #665 بعد دمج #714/#715 | Done | P1 | PR #716 مدمج عند `7a3289ca43070bc8fdff925dbe98fcbbbbade3df`؛ CI #43933542371 وCodeQL #43933542202 نجحا على HEAD `beb9d79` |
| CHAT-005 | AI Chat App/UX | معالجة واجهة المحادثة وبقايا استقرار Issue #683 | In Progress | P0 | PR #687 مدمج ويعالج pseudo-tool-call وسلامة retry؛ تبقى sidebar/actions/composer/mobile/RTL وسبب provider failures وقبولها اليدوي؛ لا تكرر #664/#676 |
| CHAT-006 | AI Chat App/SRE | التحقق التشغيلي من QueuePool بعد تحرير الاتصالات | In Progress | P1 | PR #664 مدمج `0fd0f7c`; PR #676 مدمج `35b1d3b`; Issue #674 ما زالت تتطلب Stage telemetry وتحديد مصدر ضغط pool؛ لا تغيّر سعة Render/Supabase عشوائياً |
| CHAT-007 | AI Chat App/Security | تطبيق صلاحية Workspace RBAC على كل عمليات المهام المجدولة | In Progress | P1 | Issue #718 مفتوحة؛ `scheduled_tasks.manage` معرفة في RBAC ولا يظهر فحصها في مسارات create/update/delete/run/retry على main؛ يلزم قرار ملكية workspace/user واختبارات custom role و403/404 قبل PR صغير. |
| CHAT-008 | AI Chat App/Security | تقييد خوادم MCP العامة بالمستخدمين المخولين | Done | P1 | PR #721 مدمج عند `59ea8043b51c1abedf80906c999cc4e64fa251d9` من head `45cfab0`؛ CI #43972899224 نجح 5/5 وCodeQL #43972899294 نجح 2/2؛ Issue #719 مغلقة بعد التحقق من main؛ لم يتم تفعيل أي خادم MCP أو تغيير إعدادات الإنتاج. |
| CHAT-009 | AI Chat App/Security | إعادة تحقق ACL داخل أدوات ملفات المشاريع للوكيل | In Progress | P1 | Issue #720 مفتوحة؛ PR #722 مفتوح على `security/project-agent-acl-20261117` عند head `0ca3322`؛ يضيف `can_read_project/can_edit_project` داخل tool handlers واختبار viewer/revocation؛ CI backend جارٍ وCodeQL وfrontend وSDK وpreview-builder وproduction-compose نجحت عند آخر تحقق؛ لا دمج قبل اكتمال backend والمراجعة. |
| ERP-001 | ERP/Docs | مزامنة README وContinuity وGovernance مع main وStage25C | Done | P1 | PR #13 مدمج عند `7bb01d23328a68aae67c659fd6c32b6db033637d`؛ ERP backend quality #43933811836 نجح على HEAD `d4ada44`؛ Stage25C الفعلي ما زال Blocked في ERP-002 |
| ERP-002 | ERP | تنفيذ Stage25C على نسخة SQLite v8 محلية مستعادة | Blocked | P1 | يحتاج نسخة محلية مستعادة على جهاز المستخدم؛ التقرير المنقح فقط يُراجع |
| ERP-003 | ERP | تجربة PostgreSQL control plane/ETL على قاعدة disposable | Deferred | P1 | لا تبدأ حتى مراجعة Stage25C ووجود reconciliation/rollback tests |
| ERP-004 | ERP | إبقاء multi-company مغلقاً حتى اكتمال اختبارات العزل | In Progress | P0 | `TENANT_DATA_SCOPING_ENABLED=False` يستمر حتى الموافقة على جميع البوابات |
| ERP-005 | ERP/Security | استكمال عقد سياق الشركة الموقّع في معاملة PostgreSQL المجمّعة واختبارات cleanup/rollback/pool reuse | Deferred | P1 | من Stage25A backlog؛ بعد trusted signer وPostgreSQL disposable؛ اختبارات success/exception/rollback/reuse/tampering/replay وRLS؛ لا ربط بالتطبيق قبل المراجعة الأمنية |
| CORE-001 | AI Core | جرد قدرات الذكاء الاصطناعي الموجودة في المنتجات الثلاثة | In Progress | P1 | `docs/AI_CORE_CAPABILITY_INVENTORY.md` يتضمن مراجعة main SHA `17c9868fc17a37a1743c4a49cdb378d93160650c` ومسارات واختبارات فعلية؛ PR #721 عالج MCP global role gate ودمج بنجاح وأُغلقت Issue #719؛ PR #692/#694/#695/#696 مدمجة؛ Issues #718 و#720 لا تزالان مفتوحتين؛ يلزم استكمال تدقيق Lumen وERP وبقية القدرات قبل إغلاق الجرد. |
| CORE-002 | AI Core | تحديد نموذج وRuntime مفتوح الأوزان ومتطلبات الموارد | Proposed | P2 | يعتمد على CORE-001 ومتطلبات الخصوصية والتكلفة والأجهزة |
| CORE-003 | AI Core/Architecture | تحديد حالات الاستخدام ومتطلبات الوكلاء المتخصصين | In Progress | P1 | S4-02؛ `docs/AI_CORE_SPECIALIZED_AGENT_REQUIREMENTS.md` خط أساس للمتطلبات؛ يلزم ربط كل تخصص بعقود الأدوات والاختبارات ومعايير التقييم قبل التنفيذ |
| CORE-004 | AI Core/Security | إعداد مصفوفة أدوات وصلاحيات الوكلاء المتخصصين | In Progress | P1 | S4-06؛ `docs/AI_CORE_AGENT_TOOL_MATRIX.md` يحدد مستويات الصلاحية وعقد الموافقة؛ يلزم تدقيق أدوات المنتج والـallowlists والاختبارات قبل أي تفعيل |
| ECO-001 | Unified Ecosystem | وضع عقود API وإصداراتها بين المنتجات | In Progress | P1 | S5-02؛ `docs/PRODUCT_API_CONTRACT_CATALOG.md` يحدد قالب العقد وسياسة الإصدار والهوية والتفويض والأخطاء والاختبارات؛ يلزم جرد الواجهات الفعلية داخل كل منتج |
| ECO-002 | Unified Ecosystem | تصميم هوية/تفويض مشترك عند الحاجة | Proposed | P2 | لا SSO أو مشاركة مفاتيح قبل threat model وقرار مستقل |
| ECO-003 | Unified Ecosystem | توثيق ملكية المنتجات ومصادر الحقيقة وحدود الواجهات | Done | P1 | `docs/PRODUCT_OWNERSHIP_AND_INTERFACE_CATALOG.md` يحدد المستودع والمالك ومصدر البيانات والحدود وعقود التكامل المرشحة؛ لا يعني تنفيذ API أو هوية موحدة |
| INNO-001 | Innovation | المختبرات والمحاكيات وInnovation Studio وأدوات البناء | Deferred | P3 | لا يبدأ قبل استقرار المسارات الخمسة وقرار بدء مستقل |

## قاعدة التنفيذ

برنامج المراحل الخمس الحالي يدار عبر 50 حزمة عمل موثقة في `docs/UNIFIED_50_TASK_EXECUTION_PLAN.md`. هذا السجل يتابع حالة المهام الفعلية في كل مستودع؛ لا تُنشأ مهام مكررة فقط لتعبئة عدد 10، ولا تنتقل حالة أي حزمة إلى Done دون دليل قبول.


قبل بدء مهمة: افحص مستودعها وPRs ذات الصلة، وتأكد أنها لا تُنفذ بالفعل في فرع آخر. بعد العمل حدّث الصف بمعرف Commit/PR، والاختبارات، وما بقي. لا تنقل المهام بين المستودعات؛ إذا لزم تعاون مشترك فأنشئ مهمة تنسيق GOV/ECO مع روابط المهام التابعة.
