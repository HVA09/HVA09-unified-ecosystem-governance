# سجل المناقشات والملاحظات الهندسية المرتبطة

هذه الوثيقة تلخص نقاط المناقشة المهمة التي تؤثر في القرارات الحالية. تظل مناقشات GitHub الأصلية مصدراً تفصيلياً؛ لم تُنسخ كل التعليقات الفردية إلى المستودع المركزي.

## Lumen

### PR #6 — حزمة وثائق معمارية مقترحة
[PR #6](https://github.com/HVA09/lumen-learning-platform/pull/6)

هذا PR مفتوح وفي حالة Draft وغير مدمج، ويحتوي خمس وثائق معمارية وتطويرية. حُفظت نسخته في `archive/source-snapshots/2026-11-16/lumen-learning-platform/pr-6-open-docs/`. تُستخدم الوثائق للمقارنة مع `main`، ولا تُعامل كخطة نافذة قبل مراجعة الفروق.

### PR #5 — تأمين إنشاء أول مدير

[PR #5](https://github.com/HVA09/lumen-learning-platform/pull/5)

- الحالة عند الجرد: Open + Draft + Unmerged.
- Head: `96f2b6117d5c86abeb6a53e8b4259a8692a34bcb`؛ Base: `f15c8270110b90bc18f9d814c2e07c5285063b22` (أقدم من main وقت الجرد).
- الهدف: منع التسجيل العام من منح admin لأول مسجل، وتوفير أمر operator-side لترقية حساب مالك موجود ومؤكد مرة واحدة.
- ملف الوثائق المتغير: `backend/README.md`، محفوظ تحت `pr-5-open-admin-bootstrap/`.
- الوصف عند الفحص ذكر أن الاختبارات وDocker build لم يكونا قد شُغّلا في بيئته؛ يجب مراجعة Actions على HEAD الحالي قبل أي قرار. لا دمج قبل rebase/review وCI.

### PR #16 — وثائق فرع LTI/AGS القديم

[PR #16](https://github.com/HVA09/lumen-learning-platform/pull/16)

- الحالة: Open + Draft + Unmerged، HEAD `e125643f2bf75f0ede15cdbf6c68fb15eabc14e0`، Base `7d3c6a71f72a9d59115a0c2aba0ee0ecd4a03d41`.
- حُفظ ملفا `backend/README.md` و`backend/LTI_AGS_SANDBOX_RUNBOOK.md` في لقطة منفصلة.
- يتقاطع معظم PR #16 مع PR #154 في الشيفرة؛ لا تعتمد نسخته أو تدمجها بجانب #154. راجع سجل المصالحة لاختبارات AGS الفريدة قبل أي إغلاق.

### Issues مفتوحة في Lumen

- [Issue #150 — resource-scoped AGS](https://github.com/HVA09/lumen-learning-platform/issues/150): لا تنشأ درجة لكل mapping بناءً على platform user link فقط؛ يجب ربط submission بمورد LTI الذي أطلقه الطالب. الفكرة موجودة في فرع #154 غير المدمج؛ issue بلا تعليقات عند الفحص.
- [Issue #134 — Python Lab isolation](https://github.com/HVA09/lumen-learning-platform/issues/134): concern بشأن تشغيل كود المتعلم في أصل التطبيق. main يحتوي iframe opaque-origin/CSP/worker، لكن حدود الذاكرة وكونه ليس OS sandbox تتطلب توثيق قبول متبقٍ؛ issue بلا تعليقات عند الفحص.

### PR #154 — LTI 1.3 / AGS
[PR #154](https://github.com/HVA09/lumen-learning-platform/pull/154)

أبرز الملاحظات الهندسية التي ظهرت في المراجعة:
- يجب ألا تحتفظ عملية إرسال الدرجة بمعاملة أو قفل DB أثناء OAuth/HTTP إلى LMS؛ استخدم lease وحجوزات قصيرة واختبارات تزامن وإثبات عدم استنزاف pool.
- يجب حماية ترتيب الدرجات عند re-grade والتعامل مع نتيجة خارجية غير مؤكدة.
- CI ناجح على رأس المسودة الحالي، لكنه لا يغلق Moodle/Canvas Sandbox.
- تبقى at-least-once semantics، clock skew، retention لـlti_subject، وsame-origin rule نقاطاً يجب توثيقها/اختبارها.
- PR يظل Draft وغير مدمج؛ لا يُعتبر أي جزء منه على main قبل تحقق الدمج.

### PRs #16–#23 — تداخل LTI/AGS مع #154

[طلبات LTI المفتوحة](https://github.com/HVA09/lumen-learning-platform/pulls?q=is%3Apr+is%3Aopen)

تمت مقارنة قائمة الملفات المتغيرة لكل PR مع قائمة ملفات #154 (31 ملفاً). نتائج تقاطع المسارات:
- #16: 15 من 16 مساراً مشتركة؛ المسار المتبقي `backend/tests/test_lti_ags.py`.
- #17: 4/4 مسارات مشتركة.
- #18: 1/2 مشتركة؛ ملف اختبار AGS القديم مستقل.
- #19 و#20 و#21 و#22: كل مسارين في كل PR مشتركان مع #154.
- #23: لا يوجد تقاطع مسارات؛ يغير `backend/tests/test_lti_ags.py` فقط.

هذا تقاطع مسارات وليس برهاناً على تكرار الاختبارات أو منقولية كل السلوك. يجب مقارنة الـdiffs والسيناريوهات قبل أي إغلاق. أولوية المراجعة هي الاختبارات غير المشتركة، خصوصاً `test_lti_ags.py` في #16/#18/#23. لا تُغلق الطلبات تلقائياً ولا تنقل commits عشوائياً.

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

### PR #714 — دمج سجل التفكيك وتعارض آخر ملحق

[PR #714](https://github.com/HVA09/ai-chat-app/pull/714) مغلق ومُدمج في 15 نوفمبر 2026، وMerge commit يساوي `main` الحالي وقت المراجعة: `5d4c46af18838f7cbcebd3063f30279fd6146ad5`.

**ملاحظة تصالحية:** رغم دمج #714، ما زالت آخر كتلة في `docs/PR-665_SPLIT_TRACKER_AR.md` تصف #714 بأنها 7/8 checks وتطلب انتظار Backend ثم الدمج، كما تعرض Base SHA أقدم وتكرر أرقام القسم 51. لذلك هذا السجل يحتاج تحديث ما بعد الدمج؛ راجع `CHAT-004` وسجل التعارضات. PR #665 نفسه ما زال Open + Draft ولا يُدمج ككتلة.

### Issues مفتوحة في AI Chat App

- [Issue #683 — UX/stability](https://github.com/HVA09/ai-chat-app/issues/683): مفتوحة وبها 82 تعليقاً حتى آخر نشاط ظاهر في 12 نوفمبر. تشمل تنظيم sidebar والقائمة، contextual message actions، mobile/RTL، provider failures، وstructured tool invocation. PR #687 عالج سلامة retry والآثار الجانبية، و#664/#676 عالجا تحرير DB connections قبل SSE والانتظار الخارجي؛ لا يعني ذلك أن كل معايير القبول اليدوية في #683 أُغلقت. آخر evidence 503 من Gemini كان على instance قديم، ولا يحدد وحده سبب غياب fallback.
- [Issue #674 — QueuePool external waits](https://github.com/HVA09/ai-chat-app/issues/674): مفتوحة رغم دمج #664 و#676. الإصلاح البرمجي موجود في main؛ لا يزال التحقق من Stage QueuePool telemetry وقبول Android/دورة الملفات بوابات تشغيلية مذكورة في سجل issue. لا تُكرر helper أو الحدود المدمجة.

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


## مراجعة سجلات المراجعات والتعليقات — 16 نوفمبر 2026

### Lumen — PR #154

- أظهرت بيانات GitHub خمسة submissions بحالة `COMMENTED`، جميعها من مالك المستودع `HVA09`؛ لم يظهر أي submission بحالة `APPROVED` من مراجع مستقل، كما لم تظهر inline review threads مفتوحة.
- الملاحظة الأولى (15 نوفمبر) حددت احتفاظ `dispatch_lti_grade_event` بمعاملة وقفل DB أثناء OAuth/HTTP؛ طلبت lease دائمًا ومحدوداً واختبارات ترتيب الدرجات وre-grade وlost response وعدم استنزاف pool.
- الملاحظات اللاحقة سجلت إضافة lease مع owner token، وcommit قبل I/O الخارجي، ومعاملة قصيرة بعده، واختبارات متعددة الجلسات والتزامن. كما راجعت استخدام `join_transaction_mode="create_savepoint"` والتنظيف الصريح للجلسات المستقلة.
- آخر تعليق مراجعة في 15 نوفمبر كان لقطة تحقق تاريخية عند رأس أقدم وCI #43744257767. لا يُستخدم بدلاً من CI الأحدث #43779761820 على HEAD `6c93b49...`.
- الخلاصة: الملاحظات التقنية الأساسية عولجت في فرع المسودة حسب الاختبارات الموثقة، لكن لا توجد موافقة دمج مستقلة ولا اختبار LMS خارجي؛ لذلك يظل #154 Draft.

### Lumen — PR #6 و#16–#23

- فحص سجل المراجعات والـinline threads أعاد قوائم فارغة لـPR #6 و#16–#23. لا تعليقات مراجعة/موافقات مسجلة عبر هذين المسارين في بيانات GitHub التي فُحصت.
- غياب التعليقات لا يثبت أن كل سيناريو فريد من PRs القديمة موجود في #154؛ تُحسم التغطية بمقارنة الاختبارات والسلوك، لا بعدد التعليقات أو تداخل أسماء الملفات.

### AI Chat App — PR #665 و#671

- PR #665 لديه inline CodeQL threads اثنان، وكلاهما معلّم `resolved` و`outdated` على الفرع القديم: ملاحظة Incomplete URL substring sanitization في `ai-backend/app/services/embeddings.py`، وملاحظة مماثلة في `ai-backend/tests/test_web_search.py`. الروابط الأصلية: [CodeQL #13](https://github.com/HVA09/ai-chat-app/security/code-scanning/13) و[CodeQL #14](https://github.com/HVA09/ai-chat-app/security/code-scanning/14).
- لا تعني حالة `outdated` أن الفحص الأمني الحالي على `main` لديه المشكلة أو أنه سليم؛ المرجع هو CodeQL على HEAD الحالي لكل تغيير.
- PR #671 لم يُظهر review submissions أو inline review threads عند الفحص، وما زال Open وغير Draft وعلى Base قديم.

### AI Chat App — PR #714 والطلبات الفرعية

- PR #714 مغلق ومُدمج، وMerge commit يساوي `main` الحالي عند جرد 16 نوفمبر. لذلك فإن نص tracker الذي يطلب انتظار اكتمال #714 ودمجه أصبح تاريخياً ويحتاج تحديثاً في مستودع AI Chat App عبر PR مستقل.
- لم تظهر inline review threads في PRs #714 و#690 و#692 و#694 و#695 و#696 التي فُحصت؛ هذه الملاحظة تخص سجلات المراجعة المتاحة ولا تستبدل نتائج CI أو مراجعة الكود.

**حدود الفحص:** هذا سجل للـPRs المحددة أعلاه. لا يدّعي أنه نسخ كل تعليقات Issues أو كل أحداث GitHub، ولا يحوّل التعليقات إلى موافقات رسمية.
