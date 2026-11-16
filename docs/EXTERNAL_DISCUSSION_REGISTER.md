# سجل المناقشات والملاحظات الهندسية المرتبطة

هذه الوثيقة تلخص نقاط المناقشة المهمة التي تؤثر في القرارات الحالية. تظل مناقشات GitHub الأصلية مصدراً تفصيلياً؛ لم تُنسخ كل التعليقات الفردية إلى المستودع المركزي.

## Lumen

### PR #6 — حزمة وثائق معمارية مقترحة
[PR #6](https://github.com/HVA09/lumen-learning-platform/pull/6)

هذا PR مفتوح وفي حالة Draft وغير مدمج، ويحتوي خمس وثائق معمارية وتطويرية. حُفظت نسخته في `archive/source-snapshots/2026-11-16/lumen-learning-platform/pr-6-open-docs/`. تُستخدم الوثائق للمقارنة مع `main`، ولا تُعامل كخطة نافذة قبل مراجعة الفروق.

### PR #154 — LTI 1.3 / AGS
[PR #154](https://github.com/HVA09/lumen-learning-platform/pull/154)

أبرز الملاحظات الهندسية التي ظهرت في المراجعة:
- يجب ألا تحتفظ عملية إرسال الدرجة بمعاملة أو قفل DB أثناء OAuth/HTTP إلى LMS؛ استخدم lease وحجوزات قصيرة واختبارات تزامن وإثبات عدم استنزاف pool.
- يجب حماية ترتيب الدرجات عند re-grade والتعامل مع نتيجة خارجية غير مؤكدة.
- CI ناجح على رأس المسودة الحالي، لكنه لا يغلق Moodle/Canvas Sandbox.
- تبقى at-least-once semantics، clock skew، retention لـlti_subject، وsame-origin rule نقاطاً يجب توثيقها/اختبارها.
- PR يظل Draft وغير مدمج؛ لا يُعتبر أي جزء منه على main قبل تحقق الدمج.

### PRs القديمة
[قائمة طلبات LTI في المستودع](https://github.com/HVA09/lumen-learning-platform/pulls?q=is%3Apr+is%3Aopen)

توجد طلبات Draft أقدم تتعلق بإدارة LTI وAGS وOIDC والتزامن والخصوصية. يجب مقارنة محتواها بـ#154 قبل بدء إصلاح مكرر.

## AI Chat App

### PR #665 — خارطة التفكيك
[PR #665](https://github.com/HVA09/ai-chat-app/pull/665)

المناقشات سجلت أن PR واسع ومتشعب، وأن إصلاحات محددة استخرجت منه:
- #690 إصلاح parser لـGemini embedContent، دُمج.
- #692 فصل إعداد مزود Embeddings، دُمج.
- #694 ضبط صلاحيات RAG وإعادة التحقق من الوصول، دُمج.
- سجل التفكيك تم تحديثه في #714 ودمجه في main في 15 نوفمبر.
العمل المتبقي يجب أن يبنى على main الحالي عبر PRs صغيرة؛ لا تعتمد على الفرع القديم ولا تدمج #665 دفعة واحدة.

### PR #671 — تدقيق الاستقرار
[PR #671](https://github.com/HVA09/ai-chat-app/pull/671)

هذا PR توثيقي مفتوح وغير Draft عند الفحص، لكنه مبني على Base قديم. وثائقه مفيدة كتاريخ تسليم، لكنها لا تحدد HEAD أو CI الحالي. يحتاج قرار تحديث أو إغلاق توثيقي.

## ERP

لم تُظهر المراجعة PRs مفتوحة للمستودع عند الفحص. ملاحظات Stage25B/25C الأساسية مثبتة في مستندات المشروع:
- لا تستخدم قاعدة حية كمصدر ETL؛ المطلوب نسخة SQLite v8 مستقلة ومستعادة.
- Preflight أداة قراءة فقط، ولا تعني نجاحها الجاهزية للترحيل.
- يجب أن تكون PostgreSQL disposable حتى اجتياز reconciliation وrollback واختبارات الهوية والعزل.
- لا تغيير على SQLite v8 أو تشغيل multi-company قبل اكتمال البوابات.

المصادر:
- [سجل استمرارية ERP](https://github.com/HVA09/erp-global-pro/blob/main/ERP_GLOBAL_PRO_CONTINUITY_AR.md)
- [خطة Stage25C](https://github.com/HVA09/erp-global-pro/blob/main/docs/ERP_STAGE_25C_OFFLINE_PREFLIGHT_RUNBOOK_AR.md)
- [ADR ERP](https://github.com/HVA09/erp-global-pro/blob/main/docs/ERP_ARCHITECTURE_DECISIONS.md)

## حدود السجل

- الملخص لا يحل محل نص المراجعة الأصلي.
- لم تُحفظ كل تعليقات PR أو Issue، ولا كل سجلات Actions.
- عند اتخاذ قرار يعتمد على تعليق محدد، افتح الرابط الأصلي وأثبت التاريخ والـCommit الذي كان التعليق يراجعه.
