# خطة التنفيذ المتوازي الموحدة — 50 مهمة عبر المراحل الخمس

- **معرّف البرنامج:** `U50-2026-11-16-01`
- **الإصدار:** 1.0 — 16 نوفمبر 2026
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
| Lumen | `108c2eabdc4a180bc5efd5469c368ab3c3f15199` | PR #158 مدمج لتهيئة أول مدير؛ PR #154 Draft ويحتاج مطابقة مع main؛ Issue #134 و#150 بوابتان مفتوحتان حسب آخر فحص. |
| AI Chat App | `17c9868fc17a37a1743c4a49cdb378d93160650c` | PR #717 دمج مصالحة A–G وH–T؛ PR #665 لا يدمج ككتلة. |
| ERP | `7bb01d23328a68aae67c659fd6c32b6db033637d` | SQLite v8 نشطة؛ Stage25C على نسخة فعلية مستعادة غير مثبت؛ TENANT_DATA_SCOPING_ENABLED=False. |
| AI Core | لا يوجد خط أساس برمجي معتمد | يبدأ بجرد القدرات والمتطلبات، بلا شراء GPU أو تكرار قدرات. |
| Unified Ecosystem | المستودع المركزي | حوكمة وعقود مشتركة فقط، لا كود منتجات أو DBs. |

هذه نقاط استئناف مؤرخة وليست بديلاً عن قراءة SHA الحي قبل كل تغيير.


## المرحلة الأولى — Lumen: 10 مهام

| المعرّف | حزمة العمل | الحالة الأولية | الاعتماديات / معيار القبول |
|---|---|---|---|
| S1-01 | تهيئة أول مدير ومسار صلاحيات التشغيل المقيد | **Done — merged + PR CI 4/4** | PR #158 مدمج في main عند 108c2eab؛ CI #43957299993 نجح 4/4 على رأس PR؛ لم يتم تشغيل أمر ترقية مدير فعلياً ولا الوصول إلى قاعدة إنتاج. |
| S1-02 | مصفوفة الحسابات والجلسات وأدوار الطالب والمعلم والمدير | Audit / Ready | RBAC لكل API وواجهة، IDOR ورفض الوصول المتبادل وإبطال الجلسات؛ لا إعادة بناء auth الموجود. |
| S1-03 | دورة المقرر والتسجيل وإدارة الصفوف | Audit / Ready | إنشاء/نشر/أرشفة وتسجيل/إلغاء التسجيل؛ المعلم يرى صفوفه فقط؛ Backend وBrowser E2E. |
| S1-04 | دورة الدروس والمحتوى والتقدم | Audit / Ready | نشر وإخفاء الدرس، منع الطالب من رؤية غير المنشور، استمرار completion بعد reload. |
| S1-05 | الواجبات والاختبارات والتسليم والتصحيح | Audit / Ready | تصحيح خادمي، محاولات ومهل وبوابات، حفظ التسليم، عدم كشف answer keys، اختبارات نجاح وفشل. |
| S1-06 | تقدم المتعلم والتقارير ولوحات المتابعة | Audit / Ready | اتساق completion والدرجات والتجميعات حسب الطالب/المعلم/المدير، مع اختبارات صلاحيات. |
| S1-07 | رفع الملفات والمرفقات والفحص والحذف والاستعادة | Audit / Ready | رحلة ملف كاملة، فحص ضار ورفض آمن، لا orphan records، واختبار backup/restore للملفات والبيانات. |
| S1-08 | قبول الهاتف وRTL وإمكانية الوصول | Open acceptance | قبول يدوي على هاتف: الدخول والتنقل والمقرر والدرس والواجب والملف؛ RTL/LTR والتركيز ولوحة المفاتيح. |
| S1-09 | عزل مختبر Python وحدود التنفيذ والموارد | In Progress / Browser acceptance | Issue #134: opaque origin وCSP ورسائل آمنة وإلغاء وحد زمني وقيود الموارد؛ لا ادعاء OS sandbox. |
| S1-10 | LTI 1.3 وAGS وربط الدرجة بالمورد الأصلي | Blocked on external gates | PR #154 وIssue #150؛ resource A/B وnon-LTI، Moodle وCanvas Sandbox، same-origin، retention لـlti_subject، clock skew/workers. |

## المرحلة الثانية — AI Chat App: 10 مهام

| المعرّف | حزمة العمل | الحالة الأولية | الاعتماديات / معيار القبول |
|---|---|---|---|
| S2-01 | إغلاق بوابة الاستقرار التشغيلي H | Open acceptance | إعادة فحص health/ready والنسخ والاستعادة وحماية main وCI/CodeQL وAuto Deploy مقابل render.yaml. |
| S2-02 | قبول الهاتف وإمكانية الوصول I | Open manual acceptance | اختبار حقيقي للدخول والقائمة وتبديل المحادثة والإرسال والملفات والميكروفون ولوحة المفاتيح وRTL. |
| S2-03 | صيانة معمارية الواجهة J | Ready for audit | قياس App.jsx/hooks ودورة الطلبات والإلغاء؛ فصل مسؤوليات محدود واختبارات سلوكية، بلا إعادة كتابة واسعة. |
| S2-04 | الأداء والاستجابة K | Ready for baseline | قياس bundle والتحميل والتفاعل وإعادة التصيير والقوائم والمحادثات وطلبات API؛ مقارنة قبل/بعد. |
| S2-05 | دورة المحادثة والرسائل والوسائط L | Ready for audit | إرسال/streaming/إلغاء/retry/edit/regenerate/delete/share وصوت/صور؛ لا ازدواج ولا فقد رسائل عند الفشل. |
| S2-06 | الملفات والمعرفة وRAG M | In Progress — gated | دورة ملف وفهرسة وحذف ومنع التكرار وعزل Workspace؛ تقييم retrieval؛ restore إلى staging قبل backfill/reindex. |
| S2-07 | Agent وProject Builder N | Audit / Ready | مشروع end-to-end وcheckpoints/retry/cancel/resume وفشل build آمن وحدود time/cost/tools وعزل preview. |
| S2-08 | Workspace والتعاون والصلاحيات O | Audit / Ready | دعوات وأدوار وسحب وصول وقراءة فقط وتعاون؛ cross-user/role-based backend tests. |
| S2-09 | Developer API وSDKs وWebhooks P | Audit / Ready | توافق API v1 مع Python/JS SDK، versioning وrequest IDs وrate limits وretry/webhooks/OAuth. |
| S2-10 | الأمن والخصوصية والاعتمادية والتوثيق والإصدار Q–T | Parallel audit | IDOR/BOLA والأسرار والاحتفاظ والحذف والمراقبة والحوادث والنسخ والrollback وREADME والترخيص؛ release بعد قبول وCI/Smoke. |

## المرحلة الثالثة — ERP Global Pro: 10 مهام

| المعرّف | حزمة العمل | الحالة الأولية | الاعتماديات / معيار القبول |
|---|---|---|---|
| S3-01 | حفظ النسخة الأصلية وخط الأساس القابل للاستعادة | Ready / Verify | SHA-256 للأرشيف، dependencies، اختبارات نظيفة، مؤشرات مالية/مخزون، نسخة احتياطية مستعادة؛ لا تعديل للأصل. |
| S3-02 | Stage25C على نسخة SQLite v8 فعلية مستعادة | Blocked — needs user backup | نسخة مستقلة ومصرح بها مع SHA/time/source وintegrity_check وforeign_key_check وuniqueness preflight read-only. |
| S3-03 | مصفوفة التفويض وعضويات الشركة والسياق الموقّع | In Progress / Gated | membership/roles/suspension/expiry/replay/tampering؛ لا اعتبار GUC العميل هوية موثوقة. |
| S3-04 | تدقيق عزل CRUD والمراجع والمسارات المخصصة | In Progress | اختبارات شركتين تمنع list/get/update/delete/reference/IDOR المتبادل عبر resources وcallbacks. |
| S3-05 | عزل الخدمات التشغيلية والمخزون والمبيعات والمشتريات والإعدادات | Ready after audit | company context في callbacks وinventory/accounting/settings/sequences/POS/purchasing؛ rollback واختبارات شركتين. |
| S3-06 | سلامة المالية والضرائب والأصول والمدفوعات والتسوية | Ready after data baseline | company-scoped journals/periods/payments/tax/assets/currencies؛ تسوية Trial Balance وP&L والميزانية والتدفق والمخزون. |
| S3-07 | التقارير والاستيراد والتصدير والتدقيق والمهام الخلفية | Audit / In Progress | مراجعة Stage14 ثم HTTP tests فعلية للـdashboard/import/export/sequences/audit/webhooks/jobs. |
| S3-08 | الملفات الخاصة والفحص والحصص والنسخ الذري | Ready | quarantine وAV قبل التنزيل، tenant quotas/retention، atomic DB+file backup/restore وorphan dry-run. |
| S3-09 | PostgreSQL schema وETL وRLS والقيود المركبة | Deferred until preflight | Disposable PG فقط؛ migrations/ledger وcomposite FK/unique وETL/reconciliation/rollback؛ لا dual-write ولا app cutover. |
| S3-10 | بوابة إطلاق تعدد الشركات وحزم القطاعات | Feature gate closed | لا تفعيل TENANT_DATA_SCOPING_ENABLED قبل اكتمال isolation/restore/security؛ Industry Packs اختيارية ولا ادعاء امتثال دون مراجعة. |

## المرحلة الرابعة — AI Core: 10 مهام

| المعرّف | حزمة العمل | الحالة الأولية | الاعتماديات / معيار القبول |
|---|---|---|---|
| S4-01 | جرد قدرات AI الحالية ومنع التكرار | **In Progress — initial inventory** | أُنشئ `docs/AI_CORE_CAPABILITY_INVENTORY.md` من خارطة AI Chat App الحالية والمراجع المركزية؛ يبقى تدقيق code-level لكل قدرة وربطها بملف/اختبار/PR قبل اعتماد إعادة الاستخدام. |
| S4-02 | حالات الاستخدام ومتطلبات الوكلاء المتخصصين | Start in parallel | معايير قبول للبرمجة وLinux والشبكات وCloud/DevOps والأمن وتحليل الملفات؛ فصل المتطلبات عن اختيار النموذج. |
| S4-03 | تقييم نماذج مفتوحة الأوزان منخفضة التكلفة | Design / No paid GPU | مقارنة الجودة والسياق والخصوصية والسرعة والترخيص والعتاد؛ benchmark على موارد متاحة قبل الاختيار. |
| S4-04 | عقد Model Gateway وسجل النماذج والمزودين | Design in parallel | API موحد وregistry وtimeouts/retries/fallback وversioning وحدود استخدام؛ لا اعتماد إلزامي على مزود واحد. |
| S4-05 | Orchestrator ودورة حياة المهمة والوكيل | Design in parallel | job states/checkpoints/cancel/resume/queues/idempotency/time-cost limits؛ لا تكرار runtime الموجود دون مبرر. |
| S4-06 | ملفات الوكلاء المتخصصين ومصفوفة الأدوات | Design in parallel | صلاحيات وأدوات ومعرفة واختبارات لكل تخصص؛ least privilege وعدم مشاركة السياق افتراضياً. |
| S4-07 | تكامل MCP/Tools وطبقة الموافقات | Requires S4-01 | محولات مع allowlists/scopes/approvals للأفعال الحساسة وسجل تدقيق؛ إعادة استخدام MCP الحالي حيث يصلح. |
| S4-08 | RAG والذاكرة والمعرفة وعزل النطاقات | Design / data map required | ingestion/retrieval/deletion/retention عبر user/project/company؛ اختبارات منع تسرب البيانات. |
| S4-09 | Benchmark وتقييم الجودة والسلامة والحقائق | Design in parallel | مهام قابلة للتكرار وgrounding/tool-use/security red-team/latency/cost؛ معايير قبول لكل نسخة. |
| S4-10 | تشغيل النواة ومراقبتها وتكلفتها والتعافي | Design / gated implementation | نشر مستقل وhealth/tracing/limits/circuit breakers/fallback/secrets/backup/kill switch؛ فشل النواة لا يعطل المنتجات. |

## المرحلة الخامسة — Unified Ecosystem: 10 مهام

| المعرّف | حزمة العمل | الحالة الأولية | الاعتماديات / معيار القبول |
|---|---|---|---|
| S5-01 | سجل ملكية المنتجات ومصادر الحقيقة وحدود البيانات | **Done — architecture register** | `docs/PRODUCT_OWNERSHIP_AND_INTERFACE_CATALOG.md` يوثق المالك والمستودع ومصدر بيانات كل منتج وحدود الهوية والبيانات وعقود التكامل المرشحة؛ التنفيذ البرمجي للعقود يبقى ضمن S5-02 وما بعدها. |
| S5-02 | كتالوج عقود API وإصداراتها | Design in parallel | OpenAPI/JSON schemas/versioning/compatibility/deprecation/errors/owners؛ لا استيراد كود أو اتصال DB مباشر. |
| S5-03 | هوية الخدمات والثقة بين المنتجات | Design only | threat model لخدمة-إلى-خدمة وtoken audience/scopes/rotation؛ لا SSO شامل أو نقل مفاتيح جلسات. |
| S5-04 | نموذج التفويض RBAC/ABAC ونطاق الموارد | Design in parallel | مصفوفة actor/action/resource/tenant وleast privilege/delegation/revocation/negative tests. |
| S5-05 | تصنيف البيانات وتدفقها والاحتفاظ والخصوصية | Design in parallel | data inventory/flow وpurpose/minimization/consent/retention/deletion؛ منع نقل بيانات بلا غرض مصرح. |
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
