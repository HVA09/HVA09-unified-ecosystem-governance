# فهرس وثائق المشاريع ومصادرها

- **تاريخ الجرد:** 16 نوفمبر 2026
- **الغرض:** حفظ نسخ مؤرخة قابلة للتتبع للخطط والمعمارية والمواثيق وسجلات القرارات والاستمرارية والتشغيل والتحقق والملاحظات الهندسية.
- **الإجمالي:** **171 ملفاً**: 164 ملف Markdown/نصي + 7 تقارير JSON تقنية.
- **التوزيع:** 150 ملفاً من `main`، و21 ملفاً من PRs غير مدمجة.

## قواعد قراءة الفهرس

- كل ملف محفوظ تحت اسم المشروع والفرع/PR والمسار الأصلي؛ لا تُخلط نسخة `main` مع نسخة فرع مفتوح.
- الأرشيف Snapshot مؤرخ؛ لا يحل محل المصدر ولا يتحدث تلقائياً. عند التحديث أنشئ Snapshot بتاريخ وSHA جديدين.
- ملف المصدر في مستودع المنتج يظل المرجع التشغيلي. لا تعني اللقطة أن PR دُمج أو أن محتواه معتمد.
- لم تُنسخ الشيفرة أو migrations أو ملفات البناء أو إعدادات النشر أو ملفات الاعتماديات/lock أو قواعد البيانات أو النسخ الاحتياطية أو `.env` أو الأسرار أو الملفات الثنائية.
- لم تُنسخ جميع تعليقات GitHub أو سجلات Actions؛ لخصت النقاط ذات الصلة وروابطها في [سجل المناقشات](EXTERNAL_DISCUSSION_REGISTER.md).

## ملخص الأرشفة

| المشروع | المصدر | عدد الملفات | HEAD المصدر | مسار الأرشيف |
|---|---|---:|---|---|
| Lumen | `main` | 16 | `5a065c00ff739c21cad06d551c7e3aa51a3d604b` | [main](../archive/source-snapshots/2026-11-16/lumen-learning-platform/main/) |
| Lumen | PR #5 Open + Draft | 1 | `96f2b6117d5c86abeb6a53e8b4259a8692a34bcb` | [PR #5](../archive/source-snapshots/2026-11-16/lumen-learning-platform/pr-5-open-admin-bootstrap/) |
| Lumen | PR #16 Open + Draft | 2 | `e125643f2bf75f0ede15cdbf6c68fb15eabc14e0` | [PR #16](../archive/source-snapshots/2026-11-16/lumen-learning-platform/pr-16-draft/) |
| Lumen | PR #154 Open + Draft | 3 | `6c93b49c976245b778800ed63046a2aef321fbb9` | [PR #154](../archive/source-snapshots/2026-11-16/lumen-learning-platform/pr-154-draft/) |
| Lumen | PR #6 Open + Draft | 5 | `ec44daec83b5c42d20cc91c0cd738d7307fccb24` | [PR #6](../archive/source-snapshots/2026-11-16/lumen-learning-platform/pr-6-open-docs/) |
| AI Chat App | `main` | 19 | `5d4c46af18838f7cbcebd3063f30279fd6146ad5` | [main](../archive/source-snapshots/2026-11-16/ai-chat-app/main/) |
| AI Chat App | PR #665 Open + Draft | 8 | `4792285440511833b66e1045bc235143a5a302a5` | [PR #665](../archive/source-snapshots/2026-11-16/ai-chat-app/pr-665-draft/) |
| AI Chat App | PR #671 Open, non-Draft | 2 | `5bbdd5a6ab6e9dbc1c82c4db4ed238ba68d924b6` | [PR #671](../archive/source-snapshots/2026-11-16/ai-chat-app/pr-671-open-docs/) |
| ERP Global Pro | `main` | 115 | `946b8c3bfa684143f233df519f075253a15ca621` | [main](../archive/source-snapshots/2026-11-16/erp-global-pro/main/) |
| **الإجمالي** |  | **171** | **150 main + 21 PR** |  |

## أبرز محتويات Lumen

### نسخة main — 16 ملفاً

تشمل:
- `README.md` و`backend/README.md` و`frontend/README.md` و`imports/README.md`.
- `docs/ARCHITECTURE_PLAN.md` و`docs/PROJECT_CHARTER.md` و`docs/FRONTEND_ARCHITECTURE.md`.
- `docs/DECISIONS.md` و`docs/CONTINUITY.md` و`docs/PROJECT_STATUS.md` و`docs/PROJECT_GOVERNANCE_AR.md`.
- `docs/ENGINEERING_QUALITY_COVENANT_AR.md` و`docs/QUALITY_BASELINE.md` و`docs/PR_STACK_TRIAGE.md`.
- `docs/HOSTING_FEASIBILITY.md` و`docs/PYTHON_LAB_SECURITY.md`.

### PR #5 — ملف واحد

- `backend/README.md`: تعليمات تشغيل `promote_initial_admin.py` لترقية حساب المالك إلى أول مدير بعد التحقق من الحساب.
- هذه لقطة توثيقية لمسودة قديمة؛ لا تثبت أن PR اجتاز CI أو دُمج.

### PR #16 — ملفان

- `backend/README.md`
- `backend/LTI_AGS_SANDBOX_RUNBOOK.md`

هذه نسخة فرع LTI قديم مفتوح. تتقاطع وثائقه مع #154، لذا تحفظ منفصلة لأغراض المقارنة ولا تُعامل على أنها أحدث من #154.

### PR #154 — ثلاثة ملفات

- `backend/LTI_AGS_SANDBOX_RUNBOOK.md`
- `backend/README.md`
- `docs/CONTINUITY.md`

هذه نسخة الفرع المفتوح للمراجعة؛ لا تعني الدمج إلى `main`.

### PR #6 — خمس وثائق

- `docs/ARCHITECTURE_BASELINE_AR.md`
- `docs/ARCHITECTURE_ROADMAP_AR.md`
- `docs/DEVELOPMENT_STATUS_AR.md`
- `docs/DOMAIN_BOUNDARIES_PROPOSAL_AR.md`
- `docs/PR4_TECHNICAL_REVIEW_AR.md`

وثائق مقترحة غير مدمجة؛ يجب مقارنة النقاط الفريدة مع وثائق main قبل نقل أي شيء.

## أبرز محتويات AI Chat App

### نسخة main — 19 ملفاً

تشمل خارطتي المنصة وما بعد الإصدار، سجل تفكيك PR #665، الحوكمة وميثاق الجودة، أدلة أمان الأدوات والنسخ الاحتياطي والتخزين، README للخلفية والواجهة وSDKs وPreview Builder، وأدلة Embeddings وProduction وDeveloper API وDeployment.

### PR #665 — ثمانية ملفات

نسخ من README وROADMAP وأدلة AI Specialist وRAG وRAG index migration وProduction. هذه مواد فرع واسع ومقترح، وليست دليلاً على أن التغييرات مدمجة.

### PR #671 — ملفان

- `docs/AI_CHAT_APP_HANDOFF_AR.md`
- `docs/STABILITY_AUDIT_AR_2026-11-08.md`

لقطة توثيقية من PR مبني على Base قديم؛ لا تمثل الحالة الحية.

## أبرز محتويات ERP Global Pro

115 ملفاً، منها 108 وثائق Markdown/نصية و7 تقارير JSON تقنية.

تشمل:
- Architecture Decisions، Multi-Tenant Architecture، Implementation Backlog.
- Baseline/Migration Runbook، Security Baseline، Recovery Note، Continuity وGovernance.
- Stage 1–25C changelogs/runbooks/validation/test logs.
- Tenant Ownership Matrix وREADME وRelease Notes.
- 7 تقارير Preflight/PostgreSQL JSON: أربعة ملفات على الجذر وثلاثة تحت `backend/artifacts/`.

## ما لم يُنسخ

- الشيفرة والمigrations وملفات البناء وملفات الاعتماديات وملفات lock وإعدادات النشر.
- قواعد البيانات والنسخ الاحتياطية وملفات `.env` والأسرار.
- الملفات الثنائية والصور والمرفقات.
- جميع تعليقات Issues/PRs ومراجعات GitHub وكل سجلات Actions.
- الملفات المحلية غير الموجودة على GitHub.

راجع [سجل المناقشات والملاحظات](EXTERNAL_DISCUSSION_REGISTER.md) للروابط والملخصات، و[سجل التعارضات والمصالحة](CONFLICTS_AND_RECONCILIATION.md) لحالات التداخل والوثائق القديمة.

## تحقق سلامة النسخ — 16 نوفمبر 2026

- تمت مطابقة بصمات Git Blob لجميع الملفات المؤرشفة مع مصادرها الأصلية: **171/171** تطابق.
- التوزيع: **150/150** من `main` و**21/21** من PRs غير المدمجة.
- ملفات PR المؤرشفة: Lumen #5 (1)، #16 (2)، #154 (3)، #6 (5)، وAI Chat App #665 (8)، #671 (2).
- تقارير ERP JSON السبعة صالحة بنيوياً ولم تظهر فيها أنماط مفاتيح/أسرار معروفة في فحص آلي محدود. هذا ليس فحصاً شاملاً ولا ضماناً مطلقاً لعدم وجود بيانات حساسة.
- الإجمالي: **171 ملفاً**، 164 Markdown/نص + 7 JSON.

## حدود اختيار الملفات

اختيرت خطط المشروع ومواثيق الجودة والمعمارية وسجلات القرار والاستمرارية والتشغيل والتحقق وملفات README والأدلة النصية المرتبطة بالمراحل. الأرشيف مرجع للوثائق، وليس نسخة كاملة من المستودعات ولا بديلاً عنها.


## ملحق مصالحة المرجع المعماري — 16 نوفمبر 2026

- توجد في مكتبة المستخدم نسخة `PLATFORM_MASTER_ARCHITECTURE_AR_v0.11.md`، وهي مستند تصدير/عمل خارج مستودع GitHub وليست فرعاً أو ملف مصدر داخل أي منتج.
- تمت مقارنة نطاقاتها المعمارية مع `docs/UNIFIED_ECOSYSTEM_MASTER_PLAN.md`. تم استكمال التفاصيل الفريدة عن Research & Discovery وInnovation Studio وProject Forge وVirtual Labs وSmart Technology Operations وOpen Innovation، إضافة إلى الأجهزة الفعلية واختيار التقنية وقابلية التوسع، ضمن الإصدار المركزي 1.5.
- من الآن فصاعداً المرجع الحاكم للتنسيق والرؤية المشتركة هو المستودع `HVA09/HVA09-unified-ecosystem-governance` والملف `docs/UNIFIED_ECOSYSTEM_MASTER_PLAN.md`. احتفظ بملف Library v0.11 بوصفه لقطة تاريخية؛ لا تحدّث النسختين بشكل مستقل.
- إجمالي 171 ملفاً في هذا الفهرس هو Snapshot أرشيفي محفوظ كما التُقط؛ لا يُزاد أو يُعدّل ضمن هذه الدفعة. رؤوس `main` الأحدث بعد PRs #715 و#156 و#12 موثقة في الإصدار 1.5، وPRs التوثيقية #716 و#157 و#13 غير مدمجة عند إعداد هذا الملحق.

| `docs/AI_CORE_MODEL_GATEWAY_DESIGN.md` | S4-04 Model Gateway design baseline | Central governance design; no service implementation or model selection | 2026-11-17 |