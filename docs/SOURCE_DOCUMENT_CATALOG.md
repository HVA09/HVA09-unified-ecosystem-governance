# فهرس وثائق المشاريع ومصادرها

- **تاريخ الجرد:** 16 نوفمبر 2026
- **الغرض:** حفظ نسخ مرجعية قابلة للتتبع للخطط والمواثيق وسجلات الاستمرارية والقرارات والتشغيل والتحقق.
- **موقع الأرشيف:** `archive/source-snapshots/2026-11-16/`
- **الإجمالي المرفوع:** **163 ملفاً**: 156 وثيقة Markdown/نصية + 7 تقارير JSON تقنية.

## قواعد قراءة الفهرس

- كل ملف محفوظ تحت اسم المشروع والفرع/PR والـمسار الأصلي؛ لا تُخلط نسخة `main` مع نسخة فرع مفتوح.
- الأرشيف Snapshot غير قابل للتحرير مكان المصدر. عند التحديث، أنشئ Snapshot جديداً بتاريخ وSHA جديدين.
- ملف المصدر داخل مستودع المشروع يظل المرجع التشغيلي. الروابط أدناه للتتبع والرجوع، لا تعني أن الملف أصبح مدمجاً أو معتمداً في `main`.
- لم تُنسخ الشيفرة أو ملفات الإعداد أو قواعد البيانات أو الملفات الثنائية. كما لم تُنسخ جميع تعليقات GitHub أو سجلات Actions؛ انظر [سجل المناقشات](EXTERNAL_DISCUSSION_REGISTER.md).

## ملخص الأرشفة

| المشروع | الفرع/المصدر | عدد النسخ | المصدر |
|---|---|---:|---|
| Lumen | `main` عند `5a065c00ff739c21cad06d551c7e3aa51a3d604b` | 16 | [الأرشيف](../archive/source-snapshots/2026-11-16/lumen-learning-platform/main/) |
| Lumen | PR #154 Draft، HEAD `6c93b49c976245b778800ed63046a2aef321fbb9` | 3 | [لقطة المسودة](../archive/source-snapshots/2026-11-16/lumen-learning-platform/pr-154-draft/) |
| AI Chat App | `main` عند `5d4c46af18838f7cbcebd3063f30279fd6146ad5` | 19 | [الأرشيف](../archive/source-snapshots/2026-11-16/ai-chat-app/main/) |
| AI Chat App | PR #665 Draft، HEAD `4792285440511833b66e1045bc235143a5a302a5` | 8 | [لقطة المسودة](../archive/source-snapshots/2026-11-16/ai-chat-app/pr-665-draft/) |
| AI Chat App | PR #671 مفتوح وغير Draft، HEAD `5bbdd5a6ab6e9dbc1c82c4db4ed238ba68d924b6` | 2 | [لقطة PR](../archive/source-snapshots/2026-11-16/ai-chat-app/pr-671-open-docs/) |
| ERP | `main` عند `946b8c3bfa684143f233df519f075253a15ca621` | 115 | [الأرشيف](../archive/source-snapshots/2026-11-16/erp-global-pro/main/) |
| **الإجمالي** |  | **163** |  |

## أبرز محتويات Lumen

### نسخة main (16 ملفاً)

تشمل:
- `README.md` و`backend/README.md` و`frontend/README.md`.
- `docs/ARCHITECTURE_PLAN.md` و`docs/PROJECT_CHARTER.md`.
- `docs/DECISIONS.md` و`docs/CONTINUITY.md` و`docs/PROJECT_STATUS.md`.
- `docs/PROJECT_GOVERNANCE_AR.md` و`docs/ENGINEERING_QUALITY_COVENANT_AR.md`.
- `docs/QUALITY_BASELINE.md` و`docs/PR_STACK_TRIAGE.md`.
- `docs/FRONTEND_ARCHITECTURE.md` و`docs/HOSTING_FEASIBILITY.md` و`docs/PYTHON_LAB_SECURITY.md` و`imports/README.md`.

### PR #154 Draft (3 ملفات)

- `backend/LTI_AGS_SANDBOX_RUNBOOK.md`
- `backend/README.md`
- `docs/CONTINUITY.md`

هذه النسخة منفصلة عن main ولا تعني دمج PR #154.

## أبرز محتويات AI Chat App

### نسخة main (19 ملفاً)

تشمل خارطة المنصة وخارطة ما بعد الإصدار H–T وسجل تقسيم PR #665 والحوكمة وميثاق الجودة، أدلة الأمان والنسخ الاحتياطي والتخزين، README للخلفية والواجهة وSDKs وPreview Builder، وأدلة إعداد Embeddings وProduction وDeveloper API وDeployment.

### PR #665 Draft (8 ملفات)

تشمل نسخة من `README.md` و`ROADMAP.md` وملفات backend/RAG evaluation وRAG index migration plan وتقييم AI Specialist ونسخة H–T من فرع المسودة. هذه مواد فرع اقتراح واسع وليست قراراً منفذاً؛ راجع سجل المصالحة قبل الاعتماد عليها.

### PR #671 (ملفان)

- `docs/AI_CHAT_APP_HANDOFF_AR.md`
- `docs/STABILITY_AUDIT_AR_2026-11-08.md`

هذه لقطة توثيقية من فرع PR مبني على Base أقدم، وليست الحالة الحية الحالية.

## أبرز محتويات ERP

108 وثائق Markdown/نصية تتضمن:
- Architecture Decisions، Multi-Tenant Architecture، Implementation Backlog.
- Baseline/Migration Runbook، Security Baseline، Recovery Note، Continuity وGovernance.
- Stage 1–25C changelogs/runbooks/validation/test logs.
- Tenant Ownership Matrix.
- README وRelease Notes.

إضافة إلى 7 تقارير JSON تقنية من Preflight/PostgreSQL محفوظة تحت مجلد `evidence/` داخل لقطة ERP:
- Stage17 runtime preflight.
- Stage22/23/24 PostgreSQL preflight.
- تقارير backend compatibility preflight الخاصة بالمراحل 21 و22 والحالية.

## ما لم يُنسخ

هذا الجرد خاص بملفات الوثائق والسجلات ذات الامتدادات Markdown/MDX/TXT/RST، وبـ7 تقارير JSON المحددة لـERP. لم تُنسخ:
- الشيفرة أو migrations أو ملفات البناء أو إعدادات النشر.
- قواعد البيانات أو النسخ الاحتياطية أو ملفات `.env` أو الأسرار.
- الملفات الثنائية والصور والمرفقات.
- جميع التعليقات والمراجعات في Issues/PRs، ولا جميع سجلات CI/Actions.
- الملفات المحلية أو غير الموجودة في GitHub.

للنقاشات الهندسية المهمة راجع [EXTERNAL_DISCUSSION_REGISTER.md](EXTERNAL_DISCUSSION_REGISTER.md)، وللتعارضات راجع [CONFLICTS_AND_RECONCILIATION.md](CONFLICTS_AND_RECONCILIATION.md).
