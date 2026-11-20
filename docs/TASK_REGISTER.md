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
| LUM-001 | Lumen | مراجعة واستقرار الوظائف التعليمية الأساسية | Done | P0 | مراجعة المسارات الأساسية للمتعلم والمعلم والإدارة لم تجد عيباً مؤكداً في النطاق المفحوص؛ CI #43967362902 نجح في Backend وFrontend/build وBrowser E2E وFull Git history secret scan. Browser E2E استخدم 390×844 وlocale عربي، واختبار `useDirection.test.jsx` يثبت RTL/LTR. لا يعني ذلك قبول جهاز فعلي أو نشر إنتاجي. |
| LUM-002 | Lumen/LTI | مصالحة ربط AGS بالمورد الأصلي والتحقق من سلسلة PRs #16–#23 | In Progress | P1 | PR #154 ما زال Open + Draft؛ HEAD الموثق `6c93b49c976245b778800ed63046a2aef321fbb9`، والمقارنة الحية مع Lumen main `7240b77f706adc6706b14b0163809e244d46a965` هي 46 ahead / 14 behind. CI #43779761820 ناجح على HEAD القديم فقط ولا يثبت صلاحية الفرع بعد تقدّم main؛ يلزم تحديث الفرع وإعادة CI ثم Moodle/Canvas Sandbox وبوابات same-origin وخصوصية `lti_subject` والتوقيت والعمال. |
| LUM-003 | Lumen/LTI | اختبار Moodle ثم Canvas Sandbox | Deferred | P1 | بيئة Sandbox مستقلة، HTTPS، حسابات اختبار، نتائج موثقة لكل منصة |
| LUM-004 | Lumen/Docs | مصالحة وثائق PR #6 المعمارية مع وثائق main الحالية وتحديد النقاط الفريدة فقط | Ready | P2 | PR #6 يبقى Draft؛ مقارنة الملفات الخمسة مع main، ثم اقتراح تحديث صغير داخل Lumen إذا بقي محتوى صالح غير موثق |
| LUM-005 | Lumen/Docs | مزامنة الحالة وتوضيح نقطة الاستئناف التاريخية | Done | P1 | PR #157 مدمج عند `9d766cc30525c706bd867d9736b15f74dc2acac7`؛ CI #43949038511 نجح 4/4. التحديث الأحدث PR #165 مدمج عند `5774c8575bf62a3894a0eb957b77745181ff71e1`، ويحدّث `CONTINUITY.md` و`PROJECT_STATUS.md` و`PR_STACK_TRIAGE.md`؛ CI #44154286229 نجح 4/4 على رأس PR قبل الدمج، وCI main #44183606189 نجح على الرأس المدمج. |
| LUM-006 | Lumen/Security | تهيئة أول مدير بأمان عبر مسار تشغيل واحد محمي | Done | P1 | PR #158 مدمج، ثم PR #161 أزال CLI القديم `backend/app/create_admin.py` وصحح README. Lumen main الحالي `5774c8575bf62a3894a0eb957b77745181ff71e1` يتضمن #162/#163/#164 اللاحقة؛ لم يُشغّل أمر ترقية مدير على حساب حقيقي أو قاعدة إنتاج. |
| LUM-007 | Lumen/Security | استكمال قبول عزل مختبر Python وحدود التنفيذ والموارد | In Progress | P1 | PR #146 مدمج؛ مراجعة 17 نوفمبر أكدت Pyodide داخل iframe بأصل opaque وCSP enforced ومنع DOM/storage/API، مع حد 20,000 حرف و8 ثوانٍ وإلغاء. لا يوجد حد ذاكرة صلب ولا قبول على الجهاز/المتصفح المستهدف؛ Issue #134 تبقى مفتوحة، ولا يوصف التنفيذ بأنه OS sandbox. |
| LUM-008 | Lumen/Authorization | قصر تسجيل عضوية الفصل على حسابات الطالب فقط | Done | P1 | PR #162 مدمج في Lumen main عند `18928282deff93ca9680d30d937ad72675f04cfa` عبر Squash. CI #44018148362 نجح 4/4: Backend وFrontend/build وBrowser E2E وFull Git history secret scan. يرفض `/classes/join` teacher/admin بـ403 قبل rate-limit reservation أو enrollment؛ regression test يثبت رفضهما ونجاح الطالب. لا schema/migration أو بيانات أو إعدادات تشغيل تغيرت. |
| LUM-009 | Lumen/Assignments/Security | قصر تسليم الواجب على حساب الطالب | Done | P1 | PR #163 مدمج عبر Squash عند `a386ead55935edd9cfe2c51f109ab894629985cb`. CI #44019697516 نجح 4/4: Backend tests وFrontend tests/build وBrowser E2E وFull Git history secret scan. يحظر `/assignments/{id}/submit` teacher/admin قبل lookup/creation، مع اختبارات 403 بلا side effects ونجاح الطالب المسجل. لا schema/migration أو بيانات أو إعدادات تشغيل تغيرت. |
| LUM-010 | Lumen/Authorization | استكمال تدقيق RBAC والجلسات ونطاق ملكية الموارد عبر واجهات API وUI | In Progress | P1 | مراجعة Lumen main الحالي `5774c8575bf62a3894a0eb957b77745181ff71e1`: الجلسات/CSRF/refresh/file/report/user scopes موثقة؛ #162 يحصر class join بالطالب، #163 يحصر assignment submit بالطالب، #164 يحمي آخر مدير نشط. **تدقيق ملكية المحتوى:** `/content` يستخدم `require_teacher` لكل معلم/admin؛ الأسئلة تسجل `created_by_user_id` عند الإنشاء لكن لا يُفحص في update/delete، والدروس لا تملك creator field؛ يلزم قرار مالك المنتج بشأن الإدارة المشتركة أو owner-scoped واختبارات cross-teacher/IDOR. التوصية B موثقة لكنها غير معتمدة؛ لا تغيّر السلوك أو تضف migration قبل القرار. مراجعات checkpoints 1–7 تغطي أجزاء من RBAC والجلسات والـUI والمهام الخلفية والاختبارات، لكنها ليست شهادة أمنية كاملة. تبقى route-by-route API/UI inventory واختبارات IDOR وقرار سياسة المحتوى بوابات مفتوحة. لا يُغلق LUM-010/S1-02 بعد. |
| LUM-011 | Lumen/Recovery | استكمال النسخ الاحتياطي والاستعادة المتسقة لقاعدة البيانات وملفات المستخدمين | In Progress | P1 | Checkpoint 7: `backend/docker-compose.yml` يثبت uploads في volume مستقل، لكن backup/Restic/restore الحاليين يشملون PostgreSQL فقط؛ استعادة DB قد تعيد FileAttachment metadata دون الملفات الفعلية. يلزم تصميم backup مشفر للـuploads وcombined restore drill مع فحص metadata/file integrity. أضيفت أيضاً hardening items لرفض symlink في restore وتقييم تشفير النسخ المحلية. Source review فقط؛ لم تُشغّل اختبارات أو restore فعلي. راجع `docs/LUMEN_AUTHORIZATION_AUDIT_CHECKPOINT_7_20261117.md`. |
| CHAT-001 | AI Chat App/Docs | مصالحة خارطتي A–G وH–T مع main والـPRs/Issues | Done | P1 | PR #717 مدمج عند `17c9868fc17a37a1743c4a49cdb378d93160650c`؛ CI #43952642164 (Backend/Frontend/SDK/Preview Builder/Production Compose) وCodeQL #43952642232 نجحا على HEAD `d5ccb16` |
| CHAT-002 | AI Chat App | تفكيك العمل المتبقي في PR #665 إلى PRs صغيرة | In Progress | P1 | PR #665 ما زال Open/Draft/Unmerged عند HEAD `4792285440511833b66e1045bc235143a5a302a5` (461 commits/70 files)، والمقارنة مع AI Chat main الحالي `26976f74b73b3f3f4ab3dd7001e945ff85d85b78` هي 460 ahead / 93 behind و`mergeable=false`. CI #41247792816 وCodeQL #41247792783 نجحا على exact head التاريخي، لكنهما لا يثبتان التوافق مع main الحالي؛ استخرج التغييرات المفيدة إلى PRs صغيرة محدثة ولا تدمج الكتلة. |
| CHAT-003 | AI Chat App/Docs | حسم PR #671 القديم دون فقد أرشيفه | Done | P2 | PR #671 مغلق دون Merge بتاريخ 16 نوفمبر؛ ملفاه محفوظان في Snapshot المركزي، والسبب Base قديم وحالة تشغيل متقادمة |
| CHAT-004 | AI Chat App/Docs | مصالحة سجل تفكيك PR #665 بعد دمج #714/#715 | Done | P1 | PR #716 مدمج عند `7a3289ca43070bc8fdff925dbe98fcbbbbade3df`؛ CI #43933542371 وCodeQL #43933542202 نجحا على HEAD `beb9d79` |
| CHAT-005 | AI Chat App/UX | معالجة واجهة المحادثة وبقايا استقرار Issue #683 | In Progress | P0 | PR #687 مدمج ويعالج pseudo-tool-call وسلامة retry؛ تبقى sidebar/actions/composer/mobile/RTL وسبب provider failures وقبولها اليدوي؛ لا تكرر #664/#676 |
| CHAT-006 | AI Chat App/SRE | التحقق التشغيلي من QueuePool بعد تحرير الاتصالات | In Progress | P1 | PR #664 مدمج `0fd0f7c`; PR #676 مدمج `35b1d3b`; Issue #674 ما زالت تتطلب Stage telemetry وتحديد مصدر ضغط pool؛ لا تغيّر سعة Render/Supabase عشوائياً |
| CHAT-007 | AI Chat App/Security | تطبيق صلاحية Workspace RBAC على كل عمليات المهام المجدولة | Done | P1 | PR #723 مدمج عند `6645b0502daa6922040ee4c6bb5c281ed59ef366` من head `642edc74e08960ed5bf5c01fcf8313275c496ad0`؛ CI #43977627635 وCodeQL #43977627473 نجحا؛ Issue #718 مغلقة؛ صلاحية `scheduled_tasks.manage` تُطبق على مهام المالك فقط مع إعادة تحقق في العامل الخلفي ومنع استدعاء AI بعد سحب الصلاحية. |
| CHAT-008 | AI Chat App/Security | تقييد خوادم MCP العامة بالمستخدمين المخولين | Done | P1 | PR #721 مدمج عند `59ea8043b51c1abedf80906c999cc4e64fa251d9` من head `45cfab0`؛ CI #43972899224 وCodeQL #43972899294 نجحا؛ Issue #719 مغلقة؛ لا خادم MCP جرى تفعيله أو إعادة ضبطه. |
| CHAT-009 | AI Chat App/Security | إعادة تحقق ACL داخل أدوات ملفات المشاريع للوكيل | Done | P1 | PR #722 مدمج عند `25a9958efd48de06b96157d47d0e6304d1c71878` من head `5ae8595a05dc411901ec0e9a53b4a5670183c624`؛ CI #43974707209 وCodeQL #43974707148 نجحا؛ Issue #720 مغلقة؛ viewer/revocation أثناء upload مغطاة بالاختبارات. |
| CHAT-010 | AI Chat App/Security | حسم تحويل ملفات المشروع إلى ملفات Workspace عند حذف المشروع | In Progress | P1 | Issue #724 مفتوحة؛ PR #725 Draft على `security/project-delete-private-files-20261117` عند HEAD `f75ceefe790538b96be071c4c86b4e072f25e626` يقترح Retain privately: نقل المرفقات إلى My Files الشخصية مع إزالة AssistantFileLink والحفاظ على الملفات/Chunks. CI #43990553917 اكتمل بنجاح وCodeQL #43990553931 نجح 2/2؛ PR ما زال Draft/Unmerged ويتطلب مراجعة واعتماد مالك المنتج لسياسة الاحتفاظ قبل الدمج. |
| ERP-001 | ERP/Docs | مزامنة README وContinuity وGovernance مع main وStage25C | Done | P1 | PR #13 مدمج، ثم PR #14 التوثيقي مدمج عند `577c105d81bd7b7c926965c0ed7c94379c1cb695`؛ تمت مزامنة README وسجل الاستمرارية مع ERP main الحالي. لم يتغير كود Flask أو SQLite v8 أو إعداد تعدد الشركات. Stage25C الفعلي يبقى منفصلاً في ERP-002 مع `TENANT_DATA_SCOPING_ENABLED=False`. |
| ERP-002 | ERP | تنفيذ Stage25C على نسخة SQLite v8 محلية مستعادة | Ready — DEFER-COMPUTER | P1 | Runbook وruntime preflight جاهزان؛ التحضير والمراجعة عن بعد مستمران، ويؤجل فقط تشغيل الاستعادة والفحص على نسخة SQLite v8 المحلية الفعلية حتى يتوفر كمبيوتر المستخدم. عند الاستئناف: preflight فعلي ثم تقرير blockers/reconciliation/rollback. `TENANT_DATA_SCOPING_ENABLED=False`؛ لا ETL/dual-write/PostgreSQL cutover قبل القبول. غياب الكمبيوتر وحده ليس Blocked. |
| ERP-003 | ERP | تجربة PostgreSQL control plane/ETL على قاعدة disposable | Deferred | P1 | يرتبط بـS3-09 في خطة الخمسين؛ يغطي disposable PostgreSQL وETL/RLS/reconciliation/rollback. لا تبدأ حتى مراجعة Stage25C ووجود reconciliation/rollback tests؛ أي composite constraints إضافية يجب توثيقها كجزء فرعي من المهمة لا كمسار ETL مكرر |
| ERP-004 | ERP/Security | إبقاء multi-company مغلقاً حتى اكتمال اختبارات العزل | In Progress | P0 | Config يثبت `TENANT_DATA_SCOPING_ENABLED=False`، وapp factory يمنع العمليات التشغيلية عند وجود أكثر من شركة؛ generic CRUD وبعض APIs (reports/data/file_objects) scoped إلى الشركة لكن لم يثبت اكتمال كل custom SQL/services/roles/composite FKs/PostgreSQL RLS؛ لا تفعيل قبل كل بوابات Stage25. |
| ERP-005 | ERP/Security | استكمال عقد سياق الشركة الموقّع في معاملة PostgreSQL المجمّعة واختبارات cleanup/rollback/pool reuse | Deferred | P1 | من Stage25A backlog؛ بعد trusted signer وPostgreSQL disposable؛ اختبارات success/exception/rollback/reuse/tampering/replay وRLS؛ لا ربط بالتطبيق قبل المراجعة الأمنية |
| CORE-001 | AI Core | جرد قدرات الذكاء الاصطناعي الموجودة في المنتجات الثلاثة | In Progress | P1 | آخر مطابقة main بتاريخ 20 نوفمبر: AI Chat App `26976f74b73b3f3f4ab3dd7001e945ff85d85b78`، Lumen `7240b77f706adc6706b14b0163809e244d46a965`، ERP `577c105d81bd7b7c926965c0ed7c94379c1cb695`. AI Chat PRs #733/#738/#739 تقدم مرشحاً غير مدمج لتحسين retention/upload/storage cleanup؛ لا يُنسب إلى main أو إلى AI Core المنفذ. PR #739 CI وCodeQL ناجحان على HEAD `f1b5838d85c015ef4f736e9ccee361e81c019df4`، لكنه Draft ومتسلسل بعد #738 ثم #733. Lumen #176 وERP #15 توثيقيان وCI ناجح، وكلاهما Draft. يبقى route/service inventory وعقود reuse واختبارات الحدود والتقييمات قبل إغلاق الجرد. |
| CORE-002 | AI Core | تحديد نموذج وRuntime مفتوح الأوزان ومتطلبات الموارد | Proposed | P2 | يعتمد على CORE-001 ومتطلبات الخصوصية والتكلفة والأجهزة |
| CORE-003 | AI Core/Architecture | تحديد حالات الاستخدام ومتطلبات الوكلاء المتخصصين | In Progress | P1 | S4-02؛ `docs/AI_CORE_SPECIALIZED_AGENT_REQUIREMENTS.md` خط أساس للمتطلبات؛ يلزم ربط كل تخصص بعقود الأدوات والاختبارات ومعايير التقييم قبل التنفيذ |
| CORE-004 | AI Core/Security | إعداد مصفوفة أدوات وصلاحيات الوكلاء المتخصصين | In Progress | P1 | `AI_CORE_AGENT_TOOL_MATRIX.md` يربط تسع أدوات Built-in فعلية في AI Chat App بالتخصصات ويصف القيود والثقة ونطاقات ACL؛ ما زال ينقص registry enforcement لـagent_profile/permission_level/resource_scope/approval، وأدوات Linux/network/cloud/security غير موجودة في registry الحالي. لا تفعيل T2/T3 أو توسيع MCP حتى اكتمال التصميم والاختبارات. |
| CORE-005 | AI Core/Knowledge | جرد وإعادة استخدام RAG والذاكرة ودورة حياة المعرفة | In Progress | P1 | S4-08؛ `AI_CORE_CAPABILITY_INVENTORY.md` يوثق chunking 1400/200 و768-dim embeddings وatomic per-file reindex وRAG ACL وUserMemory/ProjectMemory. Issue #724 مفتوحة؛ PR #725 يقترح Retain privately للمرفقات ويمنع توسيع workspace/shared-assistant scope؛ CI #43990553917 وCodeQL #43990553931 نجحا، لكن PR لا يزال Draft ويتطلب اعتماد سياسة الاحتفاظ قبل الدمج. لا تبنِ vector store أو memory DB موازية. |
| CORE-006 | AI Core/Evaluation | تأسيس Benchmark قابل لإعادة الإنتاج لوكلاء التخصص وRAG والاستشهادات | In Progress | P1 | S4-09؛ inventory راجع candidate PR #665: RAG 20Q/100 docs و37 specialist tasks وcitation-ID checker؛ كلها غير مدمجة على main. CI #41247792816 وCodeQL #41247792783 نجحا على exact PR head `4792285440511833b66e1045bc235143a5a302a5`، لكن الفرع 460 ahead / 93 behind main `26976f74b73b3f3f4ab3dd7001e945ff85d85b78` و`mergeable=false`؛ لذلك لا يثبتان التوافق مع main الحالي. لا live baseline؛ `--validate-only` يتحقق من fixture ولا يحسب أداء retrieval. يلزم PR مستقل محدث، rubric/data review، fresh offline CI/CodeQL، baseline حي بموافقة صريحة، semantic grounding وcritical safety review. |
| CORE-007 | AI Core/Architecture | تصميم عقد Model Gateway وسجل النماذج والمزودين | In Progress | P1 | S4-04؛ `AI_CORE_MODEL_GATEWAY_DESIGN.md` يوثق provider factory الحالية في AI Chat App، model aliases/capabilities، consumer identity/scopes، request/stream contracts، timeouts/fallback، unpriced usage وprivacy tests. لا خدمة أو endpoint منفذ ولا نموذج محدد؛ يعتمد pilot على إغلاق CORE-001 وS5-03 وS5-05 ووجود contract tests. |
| ECO-001 | Unified Ecosystem | وضع عقود API وإصداراتها بين المنتجات | In Progress | P1 | `PRODUCT_API_CONTRACT_CATALOG.md` يوثق مجموعات routes الفعلية ويضيف 3 بطاقات: `LUM-LEARN-001` للتعلم والتقدم الشخصي، `AIC-DEV-CHAT-001` لـDeveloper API v1 (مفتاح شخصي لا service identity)، و`ERP-COMPANY-CONTEXT-001` لسياق الشركة (JWT user-bound وmulti-company blocked). المتبقي schemas لبقية endpoints، service principals/audience/scopes/delegation، consumer/provider negative tests وstaging. |
| ECO-002 | Unified Ecosystem | تصميم هوية/تفويض مشترك عند الحاجة | In Progress | P2 | `SERVICE_IDENTITY_AND_DELEGATION_DESIGN.md` يحدد service principal منفصل وdelegated subject namespaced، ويربط التدفق بعقود Lumen learning وAI Chat Developer API وERP company context؛ يضع audience/scopes/jti/key rotation/revocation وnegative tests. لا SSO أو issuer أو tokens أو keys منفذة؛ لا يزال اختيار private_key_jwt/mTLS ومراجعة مالكي المنتجات مطلوباً. |
| ECO-003 | Unified Ecosystem | توثيق ملكية المنتجات ومصادر الحقيقة وحدود الواجهات | Done | P1 | `docs/PRODUCT_OWNERSHIP_AND_INTERFACE_CATALOG.md` يحدد المستودع والمالك ومصدر البيانات والحدود وعقود التكامل المرشحة؛ لا يعني تنفيذ API أو هوية موحدة |
| ECO-004 | Unified Ecosystem/Privacy | تصميم تصنيف البيانات وتدفقها والاحتفاظ والحذف | In Progress | P1 | `DATA_CLASSIFICATION_AND_RETENTION_DESIGN.md` يقترح مستويات Public/Internal/Confidential/Restricted/Secret/Unknown وقواعد model/provider eligibility وretention/deletion والـlogging؛ لا classification engine أو retention enforcement مطبق، ويلزم اعتماد مالكي Lumen/AI Chat/ERP. |
| ECO-005 | Unified Ecosystem/Security | تعريف نموذج التفويض RBAC/ABAC ونطاق الموارد بين المنتجات | In Progress | P1 | `UNIFIED_AUTHORIZATION_MODEL_DESIGN.md` يطابق أدوار Lumen وAI Chat Workspace/Project وERP company membership، ويحدد actor/action/resource/tenant/purpose وintersection of scopes وPEP لدى مالك المورد وT0–T3 approvals والاختبارات السلبية. لا policy engine أو SSO أو global admin role منفذ؛ يلزم اعتماد المالكين وربط S5-02/S5-03 واختبارات cross-tenant قبل enforcement. |
| INNO-001 | Innovation | المختبرات والمحاكيات وInnovation Studio وأدوات البناء | Deferred | P3 | لا يبدأ قبل استقرار المسارات الخمسة وقرار بدء مستقل |

## قاعدة التنفيذ

برنامج المراحل الخمس الحالي يدار عبر 50 حزمة عمل موثقة في `docs/UNIFIED_50_TASK_EXECUTION_PLAN.md`. هذا السجل يتابع حالة المهام الفعلية في كل مستودع؛ لا تُنشأ مهام مكررة فقط لتعبئة عدد 10، ولا تنتقل حالة أي حزمة إلى Done دون دليل قبول.


قبل بدء مهمة: افحص مستودعها وPRs ذات الصلة، وتأكد أنها لا تُنفذ بالفعل في فرع آخر. بعد العمل حدّث الصف بمعرف Commit/PR، والاختبارات، وما بقي. لا تنقل المهام بين المستودعات؛ إذا لزم تعاون مشترك فأنشئ مهمة تنسيق GOV/ECO مع روابط المهام التابعة.


## نقطة تنسيق موحدة — 17 نوفمبر 2026

- **Lumen:** `main` عند `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`. PR #165 توثيقي بثلاثة ملفات؛ CI #44154286229 اكتمل 4/4، لكنه Open/Unmerged ولا توجد مراجعة مسجلة. PR #154 ما زال Draft/Unmerged، 46 ahead / 7 behind و`mergeable=false`؛ لا دمج قبل تحديثه وإغلاق بوابات LTI الخارجية والخصوصية والتوقيت/العمال. LUM-010 وLUM-011 يبقيان In Progress.
- **AI Chat App:** `main` عند `6645b0502daa6922040ee4c6bb5c281ed59ef366`. PR #725 وIssue #724 مفتوحان؛ CI #43990553917 وCodeQL #43990553931 نجحا على HEAD `f75ceefe790538b96be071c4c86b4e072f25e626`، لكن PR ما زال Draft ويتطلب مراجعة واعتماد مالك المنتج لسياسة Retain privately. PR #665 لا يُدمج ككتلة؛ 460 ahead / 77 behind و`mergeable=false`.
- **ERP:** `main` عند `7bb01d23328a68aae67c659fd6c32b6db033637d`، وCI #43949986840 نجح على هذا الرأس. PR #14 يحدّث SHA وتاريخ CI في README وسجل الاستمرارية فقط؛ بعد تصحيح التاريخ المرجعي، HEAD `0109d01248aa0674af9009820b3ba017c84aa3d4` وCI #44177977660 قيد التنفيذ؛ الـPR Open/Unmerged؛ لا يغيّر Flask/SQLite v8 أو إعداد تعدد الشركات. Stage25C لا يزال Blocked حتى توفر نسخة SQLite v8 فعلية مستعادة، وتبقى `TENANT_DATA_SCOPING_ENABLED=False`.
- **AI Core / Unified Ecosystem:** يستمر العمل في الجرد والعقود والتصميم فقط؛ لا خدمة نموذج أو SSO أو policy engine أو مفاتيح مشتركة منشورة.
- **حدود هذه الدورة:** لم تُشغّل اختبارات محلية أو على قواعد المستخدم، ولم يحدث نشر أو تغيير بيانات أو أسرار أو إعدادات تشغيل. سجلات كل منتج وPRs التابعة تبقى في مستودعاتها؛ هذا الملحق تنسيقي فقط.


## سجل دمج آلي وفق تفويض المستخدم — 17 نوفمبر 2026

اعتمد المستخدم قاعدة تشغيلية لتقليل زمن مراجعة PRs: دمج طلب السحب تلقائياً عند اكتمال جميع الفحوص على **رأس PR نفسه** بنجاح، وعدم كونه Draft، ووجود قابلية دمج، وعدم وجود قرار منتج/أمن/بيانات أو موافقة بشرية معلقة. يجب تمرير `expected_head_sha` عند الدمج. لا يُدمج PR عند فشل/تعليق الفحوص، التعارض، أو غياب الفحوص المطلوبة. لا يعني نجاح CI وحده اعتماد قرارات المنتج أو المخاطر.

### دمجات مؤكدة
- ERP Global Pro PR #14 — دمج Squash، commit `577c105d81bd7b7c926965c0ed7c94379c1cb695`; توثيق فقط، لا تغييرات runtime أو قاعدة بيانات.
- Lumen PR #165 — دمج Squash، commit `5774c8575bf62a3894a0eb957b77745181ff71e1`; تحديث checkpoint توثيقي. فحوص PR الأربعة نجحت على head `e88646ab52340c5c65ff0f3a3a9a3818e7838867`.
- AI Chat App PRs #708, #706, #704, #703, #668, #548, #547, #545, #544, #425, #347, #188, #186 — دمج Squash بعد نجاح الفحوص على رؤوسها المحددة. لا تغييرات مباشرة على الإنتاج أو الأسرار.

### موانع الدمج المسجلة
- AI Chat App #707 و#705: فشل فحص frontend.
- AI Chat App #549: فشل frontend في إحدى محاولات الفحص الظاهرة.
- AI Chat App #185 و#164: GitHub رفض الدمج لوجود required status checks متوقعة لم تكتمل.
- AI Chat App #175 و#174 و#171 و#170 و#165: GitHub رفض الدمج بسبب تعارضات.
- AI Chat App #169: فشل backend.
- AI Chat App #725 و#665: Draft؛ #725 يحتاج قرار سياسة الاحتفاظ بملفات المشاريع، و#665 كبير/متباعد.
- Lumen #154 وما تبقى من Lumen PRs المفتوحة: Draft/تعارضات/بوابات LTI غير مكتملة.
- Unified Ecosystem governance PR #2: لا توجد Check Runs على رأسه عند التحقق؛ لم يُدمج.

هذه القائمة سجل لهذه الدورة وليست تصريحاً بأن كل PR مفتوح في كل المستودعات قد جرى فحصه. بعد دمج دفعة من PRs، يجب انتظار/فحص CI على main قبل اعتبار سلسلة الدمج مستقرة.


## متابعة تنفيذ واصلاح فحوص PR — 17 نوفمبر 2026 (تحديث لاحق)

### نتائج مؤكدة بعد نقطة التنسيق السابقة
- **Lumen PR #165:** تم دمجه فعلياً؛ merge commit `5774c8575bf62a3894a0eb957b77745181ff71e1`. PR #154 ما زال Draft؛ فحوصه السابقة نجحت على الرأس `6c93b49c976245b778800ed63046a2aef321fbb9` لكن لا يُدمج قبل إغلاق متطلبات LTI الخارجية والخصوصية والتوقيت/العمال، وتحديث الفرع وفق حالة main الحالية.
- **ERP PR #14:** تم دمجه فعلياً؛ merge commit `577c105d81bd7b7c926965c0ed7c94379c1cb695`. توثيق فقط.
- **AI Chat App PR #705:** تم دمجه فعلياً بعد نجاح جميع الفحوص الثمانية على HEAD `a73fdd9a400a6e0c8a28bacb56985ebace7fa9ea`; squash commit `2b58121c6dc7894c64fcd2ab9b5aa15cb4a2b42d`.
- **AI Chat App PR #707:** فشل frontend شُخّص من سجل GitHub Actions: Tailwind CSS v4 كان مستخدماً مباشرة كـPostCSS plugin بينما plugin انتقل إلى `@tailwindcss/postcss`. بدلاً من ترقية كبرى غير متوافقة مع إعداد PostCSS الحالي، عُدّل فرع PR إلى Tailwind `^3.4.19` للحفاظ على خط v3 المتوافق مع `postcss.config.js`. إصلاح commit: `363380c2242bd718d1734d709b7f0e6f29a79155`. CI وCodeQL أُطلقا على هذا الرأس وما زالا قيد التنفيذ وقت تسجيل هذه الملاحظة؛ **لم يتم الدمج بعد**.
- **AI Chat App PR #725:** كل الفحوص الثمانية نجحت على HEAD `f75ceefe790538b96be071c4c86b4e072f25e626`، لكنه Draft ويحتاج اعتماد مالك المنتج لسياسة Retain privately؛ لا دمج تلقائي قبل حسم هذا القرار.
- **AI Chat App PR #665:** ما زال Draft وفرعه متباعداً عن main؛ لا دمج كتلي.
- **Unified Ecosystem governance PR #2:** ما زال مفتوحاً ولا توجد Check Runs على الرأس `5cfb838719fba7b38a484d70ac9dcb25ee751b2e` عند التحقق؛ لا دمج.

### قاعدة اصلاح الفشل والتعارض
بموجب تفويض المستخدم المحدّث: عند فشل الفحص أو التعارض، شخّص السبب وأصلحه على فرع PR إن كان التغيير محصوراً ومنخفض المخاطر وقابلاً للتحقق؛ أعد الفحوص على الرأس الجديد؛ ادمج فقط بعد نجاح جميع الفحوص المطلوبة وقابلية الدمج وعدم وجود قرار منتج/بيانات/أمن أو موافقة مطلوبة. لا تُخفِ الفشل بإعادة تشغيل عمياء ولا تتجاوز checks. إذا كان الإصلاح يتطلب ترقية إطار كبرى أو تغييراً واسعاً أو قراراً غير محسوم، اتركه غير مدمج مع توثيق السبب.


## نقطة تنسيق موحدة — 18 نوفمبر 2026

هذه اللقطة تضيف الحالة الفعلية الأحدث فوق السجلات التاريخية السابقة ولا تعيد كتابة نقاط 17 نوفمبر.

- **AI Chat App — main:** تم دمج PR #707 (Tailwind v3.4.19) عند commit `2c5ce95be2df8c73f375fb3816a2adee3ba3d518`. بعد الدمج نجحت وظائف main: CI run #44186508017، CodeQL #44186508141، Publish backend image #44186507931، وProduction smoke #44186508189.
- **AI Chat App — PR #549 (Uvicorn):** أعاد Dependabot إنشاء/تحديث الفرع على main الحالي عبر dynamic run #44187107594 (Success). رأس PR الحالي `da11a33b077b3df2d58508a4ef9e76ef6c7cb298`، والفرع ahead 1 / behind 0. CodeQL #44187283906 نجح، وCI #44187283900 كان In Progress عند تسجيل هذه اللقطة؛ لا دمج قبل اكتماله ونجاحه على الرأس نفسه.
- **AI Chat App — PR #169 (bcrypt 5):** أُغلق دون دمج بعد فشل Backend (273 failed / 81 passed). السبب المؤكد هو عدم توافق bcrypt 5.0.0 مع كشف backend في Passlib 1.7.4؛ يستدعي الكشف اختباراً أطول من 72 بايت ويرفضه bcrypt 5.0.0. بقي قيد main `bcrypt>=4.0,<5` كما هو. لا يُعالج الأمر باقتطاع كلمات مرور المستخدمين؛ أي انتقال يتطلب ترقية/استبدال طبقة hashing واختبارات توافق وترحيل مخصصة.
- **Lumen:** main عند `5774c8575bf62a3894a0eb957b77745181ff71e1`. PR #154 لا يزال Draft، ورأسه `6c93b49c976245b778800ed63046a2aef321fbb9`; نجاح Lumen CI #43779761820 لا يغلق بوابات Moodle Sandbox وCanvas Sandbox أو متطلبات الترتيب/الخصوصية والتزامن. LUM-011 يبقى مهماً: نسخ واستعادة مجلد الملفات المرفوعة يجب أن يشمل حجم uploads نفسه، لا قاعدة البيانات وحدها.
- **ERP Global Pro:** main عند `577c105d81bd7b7c926965c0ed7c94379c1cb695`. PR #14 مدمج توثيقياً فقط. ERP-002 / Stage25C يبقى Blocked حتى استعادة نسخة SQLite v8 الفعلية واختبارها؛ تبقى `TENANT_DATA_SCOPING_ENABLED=False` ولا تفعيل لتعدد الشركات أو migrations.
- **AI Core / Unified Ecosystem:** العمل الحالي جرد وتصميم وعقود فقط. لا نموذج مفتوح الأوزان منشور، ولا SSO أو service identities أو policy engine أو مفاتيح مشتركة في الإنتاج. Governance PR #2 لا يزال مفتوحاً (head `8154aacb6e3c9ac7966e9319166af621d90dd390`)؛ لا توجد Check Runs على رأسه عند آخر تحقق، لذلك لا دمج حتى تُستوفى ضوابط المستودع.

نطاق التحقق: GitHub PR metadata وworkflow runs وقراءة الملفات فقط. لم تُنفذ تغييرات على بيانات تشغيلية أو أسرار أو إعدادات استضافة، ولم تُجرَ تجربة على قواعد بيانات المستخدم المحلية. تبقى بوابات كل منتج وقرارات مالكي البيانات نافذة.


## تحديث تنفيذ ودمج — 18 نوفمبر 2026 (دفعة ثانية)

- **AI Chat App PR #549 (Uvicorn):** تم تحديث الفرع القديم بواسطة Dependabot بعد أن كان متأخراً 180 commit؛ أصبح HEAD `da11a33b077b3df2d58508a4ef9e76ef6c7cb298` مبنياً على main، ahead 1 / behind 0. نجحت جميع متطلبات الفحص السبعة على هذا الرأس (CI: backend/frontend/sdk/preview-builder/production-compose، CodeQL: Python وJavaScript/TypeScript)، ثم دُمج Squash عند `26976f74b73b3f3f4ab3dd7001e945ff85d85b78`. PR #549 مغلق merged.
- **AI Chat App main بعد PR #549:** جميع الفحوص المطلوبة نجحت على الرأس `26976f74b73b3f3f4ab3dd7001e945ff85d85b78`: CI run #44188534560 (5/5)، CodeQL #44188534626 (2/2)، Publish backend image #44188534621، Production smoke #44188534605. لا يشير سجل هذه الخطوة إلى تغيير إعدادات الاستضافة أو الأسرار.
- **AI Chat App PR #164:** فرع docker/build-push-action v7.4.0 تمت مزامنته إلى main، HEAD `411c814783ebcf3080b1dfaa04eec80221249044`. CodeQL نجح؛ CI بقي backend فيه قيد التنفيذ وقت هذا التحديث. التغيير workflow واحد فقط. لا دمج قبل اكتمال CI على نفس الرأس والتحقق من نشر صورة backend بعد الدمج.
- **AI Chat App PR #165:** فرع qrcode 8.2 تمت مزامنته إلى main، HEAD `e98134eb84f3048038f6a0a71bfd2593efa80984`. CodeQL نجح؛ CI backend قيد التنفيذ وقت هذا التحديث. التغيير سطر اعتماد واحد في requirements. لا دمج قبل اكتمال CI.
- **AI Chat App PR #185:** فرع docker/metadata-action 6.2.0 تمت مزامنته إلى main، HEAD `a923872eafcfd88c7ff1a3136f4382855978314f`. CodeQL نجح؛ CI backend قيد التنفيذ وقت هذا التحديث. التغيير workflow واحد فقط. لا دمج قبل اكتمال CI والتحقق من نشر صورة backend بعد الدمج.
- **AI Chat App PR #175 (redis-py 8.1):** تمت مزامنة الفرع إلى main، HEAD `c180cfc53aa7f1136351c613d41ed113f3ef865e`. الفحوص كانت قيد التنفيذ وقت هذا التحديث. لا دمج تلقائي حتى لو نجحت CI دون مراجعة runtime: التطبيق يستخدم Redis لتدوير refresh tokens والرموز أحادية الاستخدام وتحديد المعدل، ويستخدم Redis كذلك لـCelery broker/result backend. ترقية redis-py 8.x تغير defaults للبروتوكول وsocket timeout/retry؛ يلزم اختبار provider الفعلي وCelery/critical token flows أو تقييد الإعدادات صراحةً مع regression tests. تم توثيق الملاحظة في PR.
- **AI Chat App PRs #170/#171/#174:** أُرسلت طلبات `@dependabot rebase`، لكن رؤوس الفروع لم تتغير إلى main وقت التحديث، لذا بقيت stale وغير مؤهلة للدمج أو الاستناد إلى فحوصها التاريخية.
- **AI Chat App PR #169 (bcrypt 5):** مغلق دون دمج مع توثيق سبب Passlib 1.7.4/bcrypt 5.0.0؛ يظل قيد main `bcrypt>=4.0,<5`.
- **Lumen:** main `5774c8575bf62a3894a0eb957b77745181ff71e1`; Lumen CI #44183606189 نجح. PR #154 يظل Draft مع 46 ahead / 8 behind، ولا يُدمج قبل إعادة مزامنته وMoodle/Canvas Sandbox واكتمال بوابات الخصوصية/العمال.
- **ERP:** main `577c105d81bd7b7c926965c0ed7c94379c1cb695`; ERP backend quality #44183566278 نجح. ERP-002 Stage25C ما زال Blocked حتى اختبار نسخة SQLite v8 المستعادة فعلياً؛ `TENANT_DATA_SCOPING_ENABLED=False`.
- **Governance PR #2:** HEAD `98b506d9783f0cdcab6e9fb6feb4425cd62df6f4`, docs-only. Branch main غير محمي ولا يطلب status checks، لكن لم توجد أي workflow runs على رأس PR؛ لم يُدمج في هذه الدورة حتى لا نساوي غياب CI بالنجاح. يلزم إضافة/تشغيل تحقق docs مناسب أو مراجعة مستقلة قبل قرار الدمج.

نطاق هذه اللقطة: GitHub metadata وActions runs وقراءة الملفات، وتحديث هذا السجل فقط. لا اختبارات على قاعدة المستخدم المحلية ولا تغييرات runtime أو إنتاجية منفذة يدوياً.


## مصالحة مستودع الحوكمة — 18 نوفمبر 2026

- PR #2 القديم ما زال مفتوحاً وغير مدمج، Head الفعلي `6fb59e44694d39972b8cff4edb9896fbe0b93c3e`، ويقارن مع main عند `6866775b251aa0e69dfaf85c9aa7cb1331cbd04a` بنتيجة 203 ahead / 0 behind و24 ملفاً. لا يُدمج ككتلة.
- أُنشئ الفرع `docs/reconciled-50-task-roadmap-20261118` من main الحالي، ونُقلت إليه الملفات الـ24 ذات الفروق المفيدة فقط، مع تحديث المرجع الرئيسي إلى v2.7، وخطة الخمسين إلى v1.4، وإضافة DEC-016 لتحديد حجم دفعة التنفيذ.
- تحقق يدوي من خطة الخمسين: 50 معرفاً فريداً، 10 لكل مرحلة S1–S5؛ صفوف crosswalk المكررة مراجع ربط وليست مهاماً إضافية.
- رؤوس المنتجات المستخدمة في نقطة المطابقة: Lumen `5774c8575bf62a3894a0eb957b77745181ff71e1`، AI Chat App `26976f74b73b3f3f4ab3dd7001e945ff85d85b78`، ERP `577c105d81bd7b7c926965c0ed7c94379c1cb695`.
- PR #4 لنموذج التشغيل تلقى ثلاث ملاحظات Codex P2: تعارض تفسير دفعة 15/50، عدم استئناف In Progress، ومرجع الخطة. عُدّل النموذج لتوضيح أن 15 مهمة تاريخية مستبدلة، وأن خطة 50 هي المرجع، والسماح باستئناف In Progress بعد مراجعة المتبقي؛ طُلبت مراجعة Codex جديدة. لا يُدمج PR #4 قبل نتيجة المراجعة الجديدة ومراجعة الفحوص.
- لا تغييرات على كود المنتجات أو قواعد البيانات أو الأسرار أو إعدادات الإنتاج.


## تصحيح نقطة الحوكمة الحية — 18 نوفمبر 2026

هذه النقطة أحدث من إشارات PR #2 الواردة في تحديثات 17/18 نوفمبر أعلاه، وتستبدل SHA القديمة المذكورة فيها:

- Governance main: `6866775b251aa0e69dfaf85c9aa7cb1331cbd04a`.
- Governance PR #2: Head الفعلي `6fb59e44694d39972b8cff4edb9896fbe0b93c3e`، مقارنة 203 ahead / 0 behind، 24 ملفاً. لا توجد workflow runs أو combined status على هذا الرأس؛ لم يُدمج.
- Governance PR #4: Head الحالي بعد معالجة ملاحظات Codex `e300bc04f5f5f0495f993faef702a23fb0fdb2d4`. طُلبت مراجعة Codex جديدة وهي قيد التنفيذ عند آخر فحص؛ لا دمج حتى اكتمالها ومراجعة الفحوص.
- فرع المصالحة الجديد `docs/reconciled-50-task-roadmap-20261118` بُني مباشرة من main، ونُقلت إليه الملفات الـ24 ذات الفروق المفيدة من PR #2. يتضمن المرجع الرئيسي v2.7، وخطة الخمسين v1.4، وDEC-016، ونقطة المطابقة الحية لرؤوس المنتجات الثلاثة.
- التحقق البنيوي للخطة: 50 معرفاً فريداً، 10 لكل مرحلة S1–S5. صفوف crosswalk المكررة ليست مهاماً إضافية.
- لا تغييرات على كود المنتجات أو قواعد البيانات أو الأسرار أو إعدادات الإنتاج.


## وسم متطلبات التنفيذ
عند تحديث أي سجل مهمة، أضف بوابة الموارد إن وجدت: `READY-NOW` أو `DEFER-COMPUTER` أو `DEFER-PAID` أو `DEFER-BOTH`. دوّن المورد المطلوب وخطوات التحضير المجاني وأدلة القبول وخطوة الاستئناف. لا تغيّر الحالة إلى Blocked لمجرد عدم توفر كمبيوتر أو اشتراك؛ ولا تسجل Done قبل إتمام التحقق الفعلي. لا شراء أو اشتراك أو استهلاك رصيد دون طلب صريح. راجع `docs/DEFERRED_EXECUTION_POLICY.md`.


## تطبيق أولي لبوابات الموارد على السجل
- **ERP-002:** `Ready — DEFER-COMPUTER`. Runbook وpreflight جاهزان؛ يؤجل التشغيل المحلي الفعلي فقط. لا يبقى Blocked بسبب عدم توفر كمبيوتر المستخدم.
- **LUM-011:** تبقى المهمة In Progress للتحضير والتصميم؛ اختبار الاستعادة المشترك لقاعدة البيانات وملفات uploads هو خطوة DEFER-COMPUTER حتى تتوفر بيئة التنفيذ.
- **CHAT-005:** يستمر إصلاح UX عن بعد؛ القبول اليدوي على الهاتف الفعلي هو DEFER-COMPUTER.
- **LUM-003:** تبقى متطلبات Moodle/Canvas Sandbox مجهزة؛ لا يُفترض أن الحساب مدفوع. يحدد نوع الحساب/البيئة أولاً، ثم يؤجل الاختبار الخارجي فقط إذا لم تتوفر بيئة اختبار مناسبة.


## تحديث المطابقة الحية — 18 نوفمبر 2026

- Lumen `main`: `5774c8575bf62a3894a0eb957b77745181ff71e1`؛ PR #165 مدمج، وPR #154 ما زال Draft وغير قابل للدمج (46 ahead / 8 behind).
- AI Chat App `main`: `26976f74b73b3f3f4ab3dd7001e945ff85d85b78`؛ PR #665 ما زال Draft وغير قابل للدمج (460 ahead / 93 behind)، ولا يدمج ككتلة.
- ERP `main`: `577c105d81bd7b7c926965c0ed7c94379c1cb695`؛ PR #14 مدمج توثيقياً، وERP-002 جاهز للتحضير مع تأجيل الاستعادة المحلية فقط.
- تحديث حالات السجل أعلاه يعكس هذه الرؤوس؛ تبقى المقاطع المؤرخة الأقدم تاريخاً ولا تُستخدم بوصفها الحالة الحية.


## نقطة تحقق حية — 18 نوفمبر 2026

- **AI Chat App:** `main` عند `26976f74b73b3f3f4ab3dd7001e945ff85d85b78`. PR #725 على HEAD `f75ceefe790538b96be071c4c86b4e072f25e626`؛ CI #43990553917 اكتمل بنجاح وCodeQL #43990553931 نجح 2/2. يبقى Draft/Unmerged وينتظر مراجعة واعتماد مالك المنتج لسياسة Retain privately. PR #665 على HEAD `4792285440511833b66e1045bc235143a5a302a5`، ومقارنته مع main الحالي: 460 ahead / 93 behind؛ لا يدمج ككتلة.
- **Lumen:** `main` عند `5774c8575bf62a3894a0eb957b77745181ff71e1`؛ CI 4/4. PR #154 Draft/Unmerged ويحتاج مزامنة وفحوصاً حديثة وبوابات Moodle/Canvas؛ LUM-010 وLUM-011 مفتوحتان.
- **ERP:** `main` عند `577c105d81bd7b7c926965c0ed7c94379c1cb695`؛ CI 4/4. ERP-002 وS3-02 جاهزتان للتحضير، مع تأجيل الاستعادة المحلية فقط؛ `TENANT_DATA_SCOPING_ENABLED=False`.
- **ملاحظة تاريخية:** نقطة التنسيق المؤرخة 17 نوفمبر أعلاه محفوظة كسجل تاريخي، ولا تُستخدم بدلاً من SHAs هذا التحقق الحي.


## نقطة متابعة موحدة — 20 نوفمبر 2026 (أدلة GitHub الحية)

- **AI Chat App main:** `26976f74b73b3f3f4ab3dd7001e945ff85d85b78`. PR #733 HEAD `6865548ce11edb78bc5f93f88110eef2a406dc1a`، و#738 HEAD `927cd4d9c1fba1545a76b8b339d2f105a3266c5c`، و#739 HEAD `f1b5838d85c015ef4f736e9ccee361e81c019df4`. فحوص CI وCodeQL المسجلة على الرؤوس الثلاثة ناجحة؛ #739 يتضمن Backend tests وSDK وFrontend/build وPreview builder وProduction Compose. ترتيب الاعتماديات: #733 ← #738 ← #739؛ لا دمج #739 منفرداً أو خارج الترتيب. #739 يبقى Draft، ولا توجد مراجعة GitHub رسمية مسجلة عند التحقق. لا يعني نجاح CI اعتماداً أمنياً أو نشر التغيير.
- **مراجعة Outbox في #739:** التنفيذ المقترح يضيف durable cleanup outbox، عامل Celery Beat كل 60 ثانية، claims باستخدام `FOR UPDATE SKIP LOCKED`، lease recovery بعد 15 دقيقة، retry backoff حتى 12 محاولة، وفحص containment لمسارات fallback. اختبارات الوحدة تغطي النجاح والفشل وإعادة المحاولة والـlease وdead-letter وS3 prefix isolation/batching. هذه قدرات PR غير مدمجة وليست وصفاً لسلوك main. تم التحقق من إعداد Render `EMBEDDED_CELERY=true` ومن `scripts/start.sh` الذي يشغل Worker وBeat ويراقبهما، كما أن Production Compose يعرّف الخدمتين. لم يتم فحص runtime الحي. تبقى بوابة التشغيل: اختبار PostgreSQL concurrency عبر اتصالات مستقلة، مراقبة/تنبيه dead-letter وإجراء replay تشغيلي، وتقييم انتهاء lease ذي 15 دقيقة عند batches متسلسلة قد تصل إلى 200.
- **Lumen main:** `7240b77f706adc6706b14b0163809e244d46a965`. PR #176 توثيقي، CI ناجح على HEAD `16e96a56800695a5aedde84e853bacd15cf3f43f`، لكنه Draft. LUM-010 ما زال يحتاج قرار ملكية المحتوى واختبارات cross-teacher/IDOR؛ LUM-011 واختبار الاستعادة يظلان مستقلين.
- **ERP main:** `577c105d81bd7b7c926965c0ed7c94379c1cb695`. PR #15 توثيقي، CI ناجح على HEAD `48b9d597ac84abbd24ea9ca4c096f881ea9a2a1a`، لكنه Draft. ERP-002 جاهز للتحضير مع `DEFER-COMPUTER` للتشغيل على نسخة SQLite v8 مستعادة؛ `TENANT_DATA_SCOPING_ENABLED=False` ولا تفعيل multi-company أو cutover.
- **Governance main:** `7275d81eeb9490947b6759b28c96c1feb9dbfddf`. PR #8 يحتوي لقطة الحالة وملحق اكتمال CI على الفرع `docs/github-live-status-20261120`، HEAD `3cbaa2f749221ee1cb55f4a13f15ff88f172993d`. لم تظهر workflow runs على هذا الرأس في آخر استعلام؛ لا يصنف CI على أنه ناجح أو فاشل.
- لا merges أو deployments أو migrations أو تغييرات بيانات أو secrets أو branch protection ضمن هذه الدورة. الحالات لا تتحول إلى Done لمجرد نجاح CI أو وجود PR.


### مراجعة متابعة Outbox — 20 نوفمبر 2026

- تم التحقق من أن Render Blueprint الحالي يضبط `EMBEDDED_CELERY=true`، وأن `ai-backend/scripts/start.sh` يبدأ Worker وBeat ويتابع عمليتيهما؛ كما يعرّف `docker-compose.prod.yml` الخدمتين. هذا يثبت وجود التكوين في المستودع فقط، لا سلامة runtime الحي.
- أضيف تعليق مراجعة إلى AI Chat App PR #739 (comment ID `7027894144`) يطلب اختبار PostgreSQL concurrency عبر اتصالات مستقلة، إجراء مراقبة/تنبيه وإعادة معالجة controlled للصفوف `failed`، ومراجعة احتمال تجاوز batch من 200 عملية متسلسلة لمهلة lease الثابتة 15 دقيقة.
- لا يوجد review submission رسمي حتى الآن؛ #739 يبقى Draft. لم يتغير كود التطبيق أو نشر الإنتاج.



## تحديث تحقق AI Chat App — 20 نوفمبر 2026، 07:40 +02

- PR #733: HEAD `6865548ce11edb78bc5f93f88110eef2a406dc1a`؛ CI وCodeQL ناجحان على الرأس الحالي.
- PR #738: HEAD `927cd4d9c1fba1545a76b8b339d2f105a3266c5c`؛ CI وCodeQL ناجحان على الرأس الحالي.
- PR #739: HEAD `f1b5838d85c015ef4f736e9ccee361e81c019df4`؛ CI وCodeQL ناجحان على الرأس الحالي.
- PR #740: HEAD `77a2d6573b9720e739f435d2ac8be63b7a90fd21`؛ CI run #44675068289 وCodeQL run #44675068497 اكتمل كلاهما بنجاح. Backend، PostgreSQL migration/schema، downgrade/re-upgrade، frontend/build، SDK، preview-builder وproduction-compose نجحت.
- #740 يضيف lease-token fencing واختبارين PostgreSQL يستخدمان SQLAlchemy sessions/connections مستقلة: إعادة الاستحواذ بواسطة العامل B أثناء IO للعامل A، ثم إثبات أن إكمال/فشل A المتأخر لا يغيّر نتيجة B.
- الترتيب لا يزال #733 → #738 → #739 → #740. جميعها Open، ولم يحدث دمج. #739 و#740 Draft.
- بوابات القبول المفتوحة: مراجعة dead-letter inspection/replay، إثبات مراقبة Celery Worker/Beat في بيئة اختبار/مرحلة، وتقييم lease duration/batch sizing بأدلة. نجاح CI ليس إثباتاً للتشغيل الفعلي.
- لا تغييرات إنتاج أو بيانات أو أسرار أو إعدادات صلاحيات/حماية فروع. لا تُغلق CHAT-010 أو S2-06 بناءً على هذه الفحوص وحدها.


## متابعة بوابة التشغيل — AI Chat App — 20 نوفمبر 2026، 05:42 UTC

- أُنشئت [AI Chat App Issue #741](https://github.com/HVA09/ai-chat-app/issues/741) لتكون سجل قبول Stage مستقل لـ Outbox/Lease بعد PRs #739 و#740.
- النطاق: إثبات تشغيل Celery Worker وBeat والجدولة، الاستعادة بعد إعادة التشغيل، سلامة تكرار IO عند انتهاء Lease، فحص وإعادة تشغيل Dead Letter، المراقبة والتنبيهات، وقياس Lease (15 دقيقة) وحجم الدفعة (حتى 200) من بيانات اختبار.
- القيود: لا إنتاج، لا حذف بيانات فعلية، لا تعديل buckets أو قواعد البيانات أو الأسرار أو إعدادات Render/Supabase أو عمليات نشر. Issue #741 لا تمنح موافقة دمج.
- أضيف رابط Issue #741 إلى نقاش PR #739 وPR #740. تظل PRs #733/#738/#739/#740 مفتوحة وغير مدمجة رغم نجاح CI وCodeQL على رؤوسها الحالية.


### تحقق حي إضافي — 20 نوفمبر 2026، 08:00 +02

- **AI Chat App stack:** المقارنات الحية تؤكد أن #733 متقدم 51 commit على main الحالي ولا يتأخر عنه؛ #738 متقدم 7 commits على قاعدة #733؛ #739 متقدم 40 commit على قاعدة #738؛ #740 متقدم commitين على قاعدة #739. الترتيب #733 → #738 → #739 → #740 ما زال سليماً. Exact-head CI وCodeQL ناجحان: #733 HEAD `6865548ce11edb78bc5f93f88110eef2a406dc1a` (CI #44502831942، CodeQL #44502832013)، #738 HEAD `927cd4d9c1fba1545a76b8b339d2f105a3266c5c` (CI #44619574353، CodeQL #44619574280)، #739 HEAD `f1b5838d85c015ef4f736e9ccee361e81c019df4` (CI #44651243510، CodeQL #44651243713)، #740 HEAD `77a2d6573b9720e739f435d2ac8be63b7a90fd21` (CI #44675068289، CodeQL #44675068497). جميعها Open وغير مدمجة؛ #739 و#740 Draft.
- **مراجعة AI Chat App:** لا توجد مراجعات GitHub مسجلة على #733/#738/#739/#740. مراجعة Codex على #738 لم تكتمل بسبب حد الاستخدام. لا يوجد تنازل موثق عن المراجعة الأمنية المستقلة لـ#733؛ لذلك لا تُعتبر الفحوص الخضراء بديلاً عن مراجعة الكود/الأمان.
- **PR #737:** الفرع متقدم 8 commits على main ولا يتأخر عنه؛ CI #44496042741 وCodeQL #44496042663 ناجحان على HEAD `63cb99d8056ff6155a4f8f9d2793a92c4ded5491`. يبقى Draft حتى اختبار قبول مضبوط بعد الدمج على PR تجريبي غير حساس، يشمل تحديث الفرع والتسلسل إلى الفروع التابعة وحالات التعارض وstale-head.
- **Issue #741:** ما زالت بوابة Stage-only مفتوحة؛ لا دليل تشغيل حي لـWorker/Beat أو replay/monitoring أو قياس lease/batch، ولا يجوز اعتبار ملفات Compose/Render دليلاً على runtime. لا نشر أو حذف بيانات أو تعديل buckets أو أسرار.
- **Lumen:** PR #176 توثيقي، فرعه 1 ahead / 0 behind وCI #44650844591 ناجح. PR #154 ما زال Draft؛ أصبح 46 ahead / 14 behind مقابل main الحالي، وفحصه الناجح #43779761820 يعود إلى HEAD أقدم، فلا يعتمد قبل تحديث الفرع وإعادة الاختبارات وبوابات Moodle/Canvas.
- **ERP:** PR #15 توثيقي، فرعه 1 ahead / 0 behind وCI #44650847647 ناجح. تبقى استعادة SQLite v8 الفعلية مؤجلة إلى بيئة الكمبيوتر، مع إبقاء `TENANT_DATA_SCOPING_ENABLED=False` وعدم تفعيل multi-company أو PostgreSQL cutover.
- **Governance:** PR #8 توثيقي؛ فرعه 13 ahead / 0 behind. لم تُرجع الاستعلامات أي workflow runs على رؤوس المرشح المتاحة؛ لذلك لا يُسجل CI نجاحاً أو فشلاً.
- لم يحدث merge أو deployment أو migration أو تغيير إنتاج/بيانات/أسرار/حماية فروع في هذا التحقق.


### AI Chat App storage outbox — deeper code review (20 Nov 2026)

- Rechecked current #740 source and migration chain: `0082_file_index_fingerprints → 0083_storage_cleanup_outbox → 0084_storage_cleanup_lease_token`; #739 adds the durable outbox table/indexes/check constraints, and #740 adds nullable `lease_token`. No migration was run.
- Current test file includes independent PostgreSQL-session reclaim/fencing cases, stale completion/failure cases, 12-attempt dead-letter behavior, local fallback traversal rejection, missing-file idempotency, and prefix isolation. S3 tests remain mocked; they do not establish real Stage duplicate-delete behavior.
- The worker claims a default batch of 50 at once and processes storage I/O serially while the lease is fixed at 15 minutes from claim time. Later rows can therefore age while waiting in the batch. Lease-token fencing prevents stale database state updates, not duplicate external I/O. Stage must measure per-item/batch duration before deciding whether to alter claim size or lease policy.
- The normal `DELETE /files/{file_id}` route still performs external object deletion before committing metadata deletion and does not use the cleanup outbox. A DB commit failure after successful object deletion can leave a metadata row pointing to a missing object. Review a transactional-outbox design for this path; no code changed.
- The inspected admin router has user-deletion enqueue/dispatch, but no cleanup-outbox inspection/replay endpoint was found. No cleanup-specific metrics/alerts were found for oldest pending age, running lease age, retries, failed count, or Worker/Beat liveness. Track these as acceptance gaps in Issue #741.
- Findings were added to [Issue #741](https://github.com/HVA09/ai-chat-app/issues/741) in a review comment. Issue remains Stage-only; no bucket mutation, production deletion, DB migration, secrets, or service scaling was performed.
