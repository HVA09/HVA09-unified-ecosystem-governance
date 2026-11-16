# كتالوج عقود API بين المنتجات

- **المعرف:** S5-02 ضمن خطة الخمسين مهمة.
- **الحالة:** مسودة كتالوج وتصميم عقود؛ لا يصف endpoints منفذة ما لم يذكر لها دليل مستقل.
- **المراجع:** `docs/PRODUCT_OWNERSHIP_AND_INTERFACE_CATALOG.md` و`docs/UNIFIED_ENGINEERING_STANDARDS.md`.
- **القاعدة:** مالك البيانات يظل مالك API ومصدر الحقيقة؛ لا وصول مباشر إلى قاعدة بيانات منتج آخر.

## 1. الهدف والنطاق

توحيد طريقة توثيق العقود وتطورها بين Lumen وAI Chat App وERP وAI Core عند إنشائه، مع بقاء التطبيقات مستقلة في النشر والبيانات والهوية. لا تفرض هذه الوثيقة API Gateway مركزياً، ولا SSO شاملاً، ولا مشاركة جلسات أو مفاتيح سرية.

## 2. سجل العقود المرشحة

| العقد | المنتج المالك | المستهلك المحتمل | البيانات المسموح بها مبدئياً | حالة العقد |
|---|---|---|---|---|
| Learning / LMS API | Lumen | تطبيق تعليمي أو موصل LMS مصرح | بيانات تعليمية لازمة للمهمة وبحسب تفويض Lumen | Candidate — يلزم جرد endpoints |
| AI Conversation / Agent API | AI Chat App | AI Core adapter أو تطبيق مصرح | سياق محادثة/وكيل وفق صلاحية المنتج؛ لا تصدير افتراضي | Candidate — المنتج يحتفظ بملكية الجلسات |
| ERP Company / Business API | ERP | موصل أو واجهة مصرح بها | بيانات شركة بعد تحقق عضوية موثوق وعزل tenant | Candidate — محجوب حتى اجتياز preflight والعزل |
| Model / Agent API | AI Core بعد إنشائه | AI Chat App وربما منتجات أخرى | طلب ومخرجات محددة النطاق؛ لا مصدر بيانات أعمال | Proposed — لا يوجد مستودع أو endpoint معتمد |
| Tool / Workspace Adapter | المنتج المالك للأداة | AI Core أو وكيل منتج | أفعال محددة وفق scopes وموافقة | Candidate — يتطلب مراجعة tool registry |
| Events / Webhooks | كل منتج مصدر | مستهلك مصرح ومشترك | أصغر payload ممكن ومعرفات مرجعية | Candidate — لا production subscriptions |

كل صف يعبر عن اتجاه تصميمي، وليس إعلان تكامل قائم. تضاف روابط OpenAPI/JSON Schema والـPR والاختبارات عندما يتم التحقق من التنفيذ على commit معلوم.

## 3. بطاقة تعريف إلزامية لكل عقد

يجب أن يوثق كل endpoint أو event الحقول التالية:

| الحقل | المتطلب |
|---|---|
| Contract ID | معرف ثابت لا يعاد استخدامه |
| Owner / Consumer | خدمة المالك والمستهلك وفرق المسؤولية |
| Purpose | غرض تجاري/تقني محدد |
| Version | إصدار العقد ونطاق التوافق |
| Operation | method + path أو event name |
| Schema | request/response أو event schema، مع required/optional/nullability |
| Identity | consumer identity وaudience وcredential type |
| Authorization | scopes وresource ownership وtenant boundary |
| Data classification | عام/داخلي/حساس/شديد الحساسية |
| Idempotency | المفتاح وسلوك التكرار ومدة الاحتفاظ به عند انطباقه |
| Limits | rate limit وpayload size وtimeout وpagination |
| Failure model | errors وretryability وbackoff وcircuit-breaker behavior |
| Observability | request/correlation ID دون محتوى حساس |
| Retention | سياسة تخزين السجلات والبيانات المنقولة والحذف |
| Compatibility | consumer/provider tests وخطة deprecation |
| Recovery | owner وrollback/fallback وincident contact |

## 4. قواعد الإصدار والتوافق

1. العقود الجديدة الخارجية توضع تحت إصدار واضح، مثل `/api/v1/` عندما يتفق مالك المنتج على ذلك؛ لا يعاد تشكيل مسارات موجودة تلقائياً لمجرد توحيد النمط.
2. التغيير الإضافي المتوافق يمكن إدخاله ضمن نفس الإصدار بعد إثبات عدم كسر المستهلكين.
3. إزالة حقل أو تغيير نوعه أو معناه أو صلاحياته أو دلالات الخطأ تغيير غير متوافق ويحتاج إصداراً جديداً أو مسار انتقال موثقاً.
4. لا تفترض أن إضافة حقل آمنة إذا كان المستهلكون يستخدمون strict schema validation؛ تشترط اختبارات التوافق.
5. يحدد مالك العقد فترة deprecation وإشعار المستهلكين وبيانات استخدام الإصدار قبل الإيقاف.
6. لا يستخدم المستهلك endpoint غير موثق أو يعتمد على جداول داخلية للمنتج المصدر.

## 5. المصادقة والتفويض

- لكل منتج هوية خدمة منفصلة؛ لا مشاركة access tokens أو session cookies بين المنتجات.
- يجب التحقق من issuer وaudience والمدة والصلاحية ومصدر الاعتماد، مع تدوير وإلغاء موثقين.
- يحدد scope الفعل والموارد والنطاق، ولا يكفي امتلاك token صالح.
- يتحقق المنتج المالك من user membership أو workspace/company ownership على الخادم.
- لا يقبل `user_id` أو `workspace_id` أو `company_id` الوارد من العميل كدليل تفويض.
- تبقى جلسات Lumen وAI Chat App وERP مستقلة. لا SSO شامل ضمن هذه المهمة.
- لا يمرر AI Core بيانات شركة أو طالب أو محادثة إلى نموذج/أداة إلا عبر إذن محدد ومنتج مالك موثوق.

## 6. شكل الطلب والاستجابة والأخطاء

### Metadata مبدئية

يُنظر في استخدام الحقول التالية حسب نوع العقد:
- `request_id`: معرف الطلب.
- `correlation_id`: ربط خطوات workflow عبر الخدمات دون كشف هوية المستخدم.
- `idempotency_key`: للعمليات التي تغير الحالة ويمكن إعادة محاولتها بأمان.
- `occurred_at`: وقت الحدث بصيغة UTC عند تبادل الأحداث.
- `schema_version`: إصدار payload في تدفقات الأحداث عند الحاجة.

لا تضاف هذه الحقول إلى API قائم قبل فحص التوافق. لا يوضع نص المحادثة أو محتوى ملف أو secret في header أو metadata غير مخصصة لذلك.

### فئات الأخطاء

| الفئة | أمثلة | سياسة العميل |
|---|---|---|
| Validation | حقل مفقود أو صيغة غير صحيحة | لا retry قبل تعديل الطلب |
| Authentication | اعتماد مفقود/منتهي/غير صالح | refresh وفق تدفق الهوية فقط؛ لا تكرر بلا حد |
| Authorization | scope أو ملكية غير كافية | deny، ولا محاولة الالتفاف بمُعرف آخر |
| Not found | مورد غير موجود أو غير مرئي للمستهلك | لا تسرب وجود مورد محجوب |
| Conflict | تكرار أو حالة متعارضة | اقرأ الحالة الحالية أو استخدم idempotency |
| Rate limited | تجاوز الحد | احترام Retry-After/backoff وحدود التكرار |
| Transient | timeout أو unavailable | bounded retry للعمليات الآمنة أو المحمية بـidempotency |
| Internal | خطأ غير متوقع | request ID للتتبع، دون تفاصيل داخلية حساسة |

يحدد كل عقد status codes والرموز والرسائل العامة والحقول القابلة للعرض. لا تخرج stack traces أو SQL أو أسراراً للعميل.

## 7. الأحداث وWebhooks

- يجب توقيع الرسالة والتحقق من timestamp ومنع replay.
- استخدام event ID فريد، وidempotent consumer، وbounded retries وdead-letter handling.
- توثيق الترتيب: هل هو مضمون أم best-effort، وكيفية التعامل مع duplicate/out-of-order.
- payload يحتوي أقل قدر لازم ويفضل مرجعاً لمورد بدلاً من نسخه كاملاً.
- يتضمن العقد سياسة إعادة الإرسال والإلغاء وتدوير السر وتسجيل التسليم.
- لا يفتح اشتراك إنتاجي قبل اختبار التوقيع وإعادة التشغيل والفشل والعزل.

## 8. الاعتمادية والأداء

- يحدد المالك timeout وحدود حجم payload وpagination وrate limit استناداً إلى القياس لا التخمين.
- لا retries غير محدودة؛ تستخدم backoff وjitter للعمليات العابرة الآمنة.
- تتطلب العمليات غير idempotent مفتاح منع التكرار أو تصميماً يمنع تكرار الأثر.
- يحدد كل مستهلك fallback واضحاً، ويجب ألا يجعل فشل API مركزي منتجاً مستقلاً غير قابل للاستخدام.
- لا توضع SLOs رقمية حتى توفر قياسات baseline ومالكي الخدمة.

## 9. اختبارات العقد المطلوبة

- provider schema validation وconsumer contract tests.
- توافق الإصدارات وتغير الحقول وnullability.
- رفض الوصول عبر user/workspace/company IDOR.
- صلاحيات صحيحة وخاطئة وscopes منتهية أو غير ملائمة.
- replay وduplicate وordering في الأحداث.
- timeout و429 و5xx وbounded retry وcircuit breaker.
- payload limits وpagination وvalidation errors.
- حذف البيانات والاحتفاظ ومنع تسرب المحتوى الحساس إلى logs.
- انقطاع AI Core أو طبقة التكامل دون تعطيل العمليات الأساسية لكل منتج.

## 10. مراحل الاعتماد

1. حصر endpoints والـschemas الفعلية في كل منتج من main، مع commit SHA ومالك الخدمة.
2. تصنيف كل واجهة: موجودة ومستخدمة، موجودة وغير مستخدمة، candidate، أو غير منفذة.
3. إنشاء OpenAPI/JSON Schema لكل عقد معني داخل مستودع مالكه؛ هذه الوثيقة لا تنقل المصدر التنفيذي إلى المستودع المركزي.
4. بناء consumer/provider contract tests قبل أي تكامل حي.
5. اعتماد threat model وسياسة الهوية والتفويض والبيانات لكل ربط.
6. تجربة التكامل في staging/بيئة disposable مع rollback واضح.
7. الانتقال للإنتاج بقرار مالك المنتج وسجل تشغيل مستقل.

## 11. خارج النطاق

- بناء API Gateway موحد أو خدمة هوية مركزية.
- تغيير endpoints القائمة أو جداول قواعد البيانات.
- نقل بيانات Lumen أو AI Chat App أو ERP إلى مستودع الحوكمة.
- تشغيل SSO أو مشاركة session secrets.
- تفعيل multi-company في ERP أو تجاوز `TENANT_DATA_SCOPING_ENABLED=False`.
- إعلان أن أي تكامل مرشح في الجدول أعلاه منفذ أو جاهز للإنتاج.

**النتيجة:** كتالوج S5-02 يحدد الحد الأدنى الذي يجب أن يحتويه كل عقد API ومراحل اعتماده. تبقى المهمة قيد التصميم حتى جرد واجهات المنتجات الفعلية وربطها باختبارات توافق وأمن.


## 12. جرد واجهات المنتجات من المصدر — 2026-11-17

الحالة التالية دليل على وجود مسارات داخلية في المنتجات، وليست بعد عقود service-to-service معتمدة. لم تُجر أي اتصالات بين الخدمات ولم تُنقل بيانات.

### 12.1 Lumen — main commit ed162b0e0af497abc5710d185fa357357baa6ecc

| مجموعة الواجهة | دليل المصدر | مالك البيانات | الحالة |
|---|---|---|---|
| Auth / User / Two-factor | backend/app/auth/routes.py، backend/app/dependencies.py، backend/app/main.py | Lumen | Existing product API؛ يتطلب consumer contract منفصلاً إن استهلكته خدمة أخرى |
| Learning / Assessment | curriculum، quiz، content، assignments، SRS، gamification، reports | Lumen | Existing product routes؛ لا يوجد في هذه الجولة OpenAPI contract خارجي معتمد |
| Classes | backend/app/routers/classes.py وmodels/class_section.py | Lumen | Existing teacher-owned class/enrollment scope؛ ليس organization-wide tenant |
| LTI | backend/app/routers/lti.py وmodels/lti.py | Lumen | LTI 1.3 login/launch ومنصات؛ AGS resource-scoped passback المتقدم في PR #154 غير المدمج فقط |
| Files | backend/app/routers/files.py | Lumen | User-owned uploads؛ ملف المستخدم لا يصبح مورداً مشتركاً مع AI Core تلقائياً |
| Admin | backend/app/routers/admin.py | Lumen | Admin-only operations؛ لا تمنح AI Core أو مستخدم خارجي دور admin |

**Authorization contract:** Lumen يستخدم أدوار user/teacher/admin على حساب المستخدم، مع Bearer أو browser session cookie، والتحقق من CSRF لطلبات cookie غير الآمنة. لا تفترض أن role string أو user_id من منتج آخر له المعنى نفسه في Lumen.

**PR #154 gate:** Draft/Open/Unmerged، HEAD 6c93b49c976245b778800ed63046a2aef321fbb9، 46 ahead / 4 behind main الحالي، mergeable=false. CI run 43779761820 نجح على هذا head، لكنه لا يثبت قبول Moodle/Canvas Sandbox أو الخصوصية والساعة؛ لا يعتمد AGS candidate كعقد production حتى إغلاق تلك البوابات.

### 12.2 ERP Global Pro — main commit 7bb01d23328a68aae67c659fd6c32b6db033637d

| مجموعة الواجهة | دليل المصدر | مالك البيانات | الحالة |
|---|---|---|---|
| Generic CRUD | backend/erp/crud.py يضيف /api/{resource} لكل Resource | ERP / الشركة النشطة | Existing؛ list/get/create/update/delete مربوط بـcompany_id في محرك Resource |
| Auth / Company selection | backend/erp/api/auth.py وservices/tenancy.py وsecurity.py | ERP | Existing؛ JWT company claims cid/mid تُعاد مطابقتها مع عضوية نشطة |
| Reports | backend/erp/api/reports.py | ERP / الشركة النشطة | Sampled routes تمرر company context إلى report services؛ يلزم استكمال audit لجميع SQL والخدمات |
| Import / Export | backend/erp/api/data.py | ERP / الشركة النشطة | Sampled handlers تمرر company_id إلى عمليات الاستيراد والتصدير |
| File Objects | backend/erp/api/file_objects.py وservices/file_storage.py | ERP / الشركة النشطة | Sampled list/upload/download/delete scoping per company |
| Other business APIs | accounting، inventory، trade، HR، ops، admin | ERP | لا تُقبل كعقود خارجية حتى إكمال route/service/tenant audit واختبارات IDOR والعزل |

**Authorization contract:** ERP يعمل حالياً على Flask + SQLite v8. JWT يتضمن cid/mid عند اختيار الشركة بعد التحقق من العضوية؛ middleware يضع g.company ويعيد التحقق في كل طلب. الأدوار الفعالة من company membership، لكن role catalog ما زال مشتركاً. إعداد TENANT_DATA_SCOPING_ENABLED=False، والتطبيق يرفض العمليات التشغيلية عندما يتجاوز عدد الشركات واحدة والحاجز مغلق. ولذلك كل ERP Company/Business API يبقى Candidate/Blocked للتكامل التشغيلي متعدد الشركات إلى أن تُغلق Stage25C وPostgreSQL/RLS والـrestore gates.

### 12.3 AI Chat App — current main commit 6645b0502daa6922040ee4c6bb5c281ed59ef366

جرد المصدر السابق في هذا الكتالوج/AI_CORE_CAPABILITY_INVENTORY.md أثبت وجود Agent Runtime وTool Registry وMCP وAgent Jobs وRAG وWorkspace/Project authorization. إصلاحات التفويض PR #721/#722/#723 مدمجة، واختبارات CI وCodeQL نجحت على رؤوسها. هذا لا يحول داخلياً مسارات المنتج إلى API مشتركة؛ يجب توثيق endpoint/service consumer contract لكل capability يراد استهلاكها خارج التطبيق.

### 12.4 قواعد تحويل واجهة منتج إلى عقد service-to-service

1. لا تستخدم جلسة متصفح Lumen أو AI Chat App كـservice credential، ولا ترسل ERP bearer JWT إلى منتج آخر.
2. يجب إصدار service principal مستقل لكل consumer مع issuer/audience/scopes/expiry/revocation، وربطه بهوية المستخدم/الغرض عند الحاجة دون نسخ session token.
3. لا تعتبر user_id أو student_id أو class_id أو company_id أو workspace_id قابلاً للنقل بين قواعد المنتجات؛ استخدم mapping موثقاً بمعرفات namespace أو issuer + subject.
4. لكل طلب يتحقق المالك من resource ownership والعضوية الحالية؛ service token يثبت هوية المستهلك ولا يثبت تلقائياً حق الوصول إلى بيانات مستخدم أو طالب أو شركة.
5. ERP API لا يتيح cross-company business operations في الوضع الحالي. أي AI action على سجلات ERP يجب أن يفشل مغلقاً إلى حين إتمام release gate، حتى لو نجح توثيق المستخدم.
6. أي Lumen content/grade/student data يجب أن يبقى تحت تفويض Lumen؛ لا يسجل prompt أو grade أو محتوى ملف في logs مشتركة.
7. لا endpoint candidate يعد جاهزاً حتى يوجد OpenAPI/JSON Schema، owner، version، consumer/provider tests، negative authorization tests، idempotency/failure model وrollback.

**تقدم S5-02:** تم جرد مجموعات الواجهات الفعلية وحدود auth/tenant/role في المنتجات الثلاثة على مستوى المصدر. تبقى مطابقة signatures وschemas لجميع endpoints، تصنيف البيانات، decision on identity mapping، وبناء consumer/provider contract tests؛ لذلك المهمة لا تزال In Progress.
