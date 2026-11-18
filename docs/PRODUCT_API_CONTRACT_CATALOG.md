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

مرجع تصميم service principals والتفويض والـaudience/scopes/rotation: `SERVICE_IDENTITY_AND_DELEGATION_DESIGN.md`؛ هذا المرجع تصميم فقط ولا يوجد issuer مشترك منفذ.


- لكل منتج هوية خدمة منفصلة؛ لا مشاركة access tokens أو session cookies بين المنتجات.
- يجب التحقق من issuer وaudience والمدة والصلاحية ومصدر الاعتماد، مع تدوير وإلغاء موثقين.
- يحدد scope الفعل والموارد والنطاق، ولا يكفي امتلاك token صالح.
- يتحقق المنتج المالك من user membership أو workspace/company ownership على الخادم.
- لا يقبل `user_id` أو `workspace_id` أو `company_id` الوارد من العميل كدليل تفويض.
- تبقى جلسات Lumen وAI Chat App وERP مستقلة. لا SSO شامل ضمن هذه المهمة.
- لا يمرر AI Core بيانات شركة أو طالب أو محادثة إلى نموذج/أداة إلا عبر إذن محدد ومنتج مالك موثوق.

مرجع تصنيف البيانات وتدفقها والاحتفاظ والحذف: `DATA_CLASSIFICATION_AND_RETENTION_DESIGN.md`؛ التصنيفات المقترحة غير مطبقة بعد ويجب اعتمادها من مالكي المنتجات.

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

### 12.1 Lumen — main commit fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8

| مجموعة الواجهة | دليل المصدر | مالك البيانات | الحالة |
|---|---|---|---|
| Auth / User / Two-factor | backend/app/auth/routes.py، backend/app/dependencies.py، backend/app/main.py | Lumen | Existing product API؛ يتطلب consumer contract منفصلاً إن استهلكته خدمة أخرى |
| Learning / Assessment | curriculum، quiz، content، assignments، SRS، gamification، reports | Lumen | Existing product routes؛ لا يوجد في هذه الجولة OpenAPI contract خارجي معتمد |
| Classes | backend/app/routers/classes.py وmodels/class_section.py | Lumen | Existing teacher-owned class/enrollment scope؛ `POST /classes/join` يقبل `UserRole.user` فقط بعد PR #162؛ ليس organization-wide tenant |
| LTI | backend/app/routers/lti.py وmodels/lti.py | Lumen | LTI 1.3 login/launch ومنصات؛ AGS resource-scoped passback المتقدم في PR #154 غير المدمج فقط |
| Files | backend/app/routers/files.py | Lumen | User-owned uploads؛ ملف المستخدم لا يصبح مورداً مشتركاً مع AI Core تلقائياً |
| Admin | backend/app/routers/admin.py | Lumen | Admin-only operations؛ حماية آخر مدير نشط عبر PR #164؛ لا تمنح AI Core أو مستخدم خارجي دور admin |

**Authorization contract:** Lumen يستخدم أدوار user/teacher/admin على حساب المستخدم، مع Bearer أو browser session cookie، والتحقق من CSRF لطلبات cookie غير الآمنة. `POST /classes/join` محصور بدور `user` (#162)، و`POST /assignments/{assignment_id}/submit` محصور بالطالب (#163)؛ كما تحمي #164 آخر مدير نشط عبر قفل roster معاملاتي مشترك. لا تفترض أن role string أو user_id من منتج آخر له المعنى نفسه في Lumen.

**PR #154 gate:** Draft/Open/Unmerged، HEAD 6c93b49c976245b778800ed63046a2aef321fbb9، 46 ahead / 7 behind Lumen main `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`، `mergeable=false`. CI run 43779761820 نجح على ذلك الرأس القديم، لكنه لا يثبت قبول Moodle/Canvas Sandbox أو الخصوصية والساعة؛ لا يعتمد AGS candidate كعقد production حتى إغلاق تلك البوابات.

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


### 12.5 أمثلة endpoints فعلية تم التحقق منها من route decorators

الأمثلة التالية تثبت وجود route في main فقط؛ لا تعني أن endpoint معتمد للاستهلاك بين المنتجات أو أن OpenAPI consumer contract قد تم اعتماده. المسارات مبنية على router prefix وعمليات `include_router/register_blueprint` في commit المالك.

| المنتج / SHA | Method + path | الدليل | المصادقة/التحكم المرصود | حالة العقد |
|---|---|---|---|---|
| Lumen `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8` | `POST /auth/register` | `backend/app/auth/routes.py` + router prefix `/auth` | إنشاء حساب ذاتي؛ يجب ألا يُستخدم كـservice provisioning | Existing product route — لا consumer contract |
| Lumen | `GET /curriculum/subjects` | `backend/app/routers/curriculum.py` + prefix `/curriculum` | سياق مستخدم Lumen عبر dependencies حيث ينطبق | Existing product route — بيانات تعليمية |
| Lumen | `POST /classes`, `POST /classes/join` | `backend/app/routers/classes.py` | إنشاء الفصل يتطلب teacher؛ الانضمام عبر دعوة؛ endpoint الانضمام يفرض `UserRole.user` فقط (#162)؛ الفصل مملوك للمعلم | Existing product route — لا organization tenant |
| Lumen | `POST /assignments/{assignment_id}/submit` | `backend/app/routers/assignments.py` | يتطلب حساب الطالب `UserRole.user`؛ teacher/admin يرفضان بـ403 دون إنشاء submission (#163) | Existing product route — لا consumer contract خارجي |
| Lumen | `GET /files`, `GET /files/{file_id}`, `DELETE /files/{file_id}` | `backend/app/routers/files.py` | كل عمليات الملفات تحت `get_current_user` وملكية `user_id` | Existing product route — user-owned data |
| Lumen | `GET /lti/jwks`, `GET /lti/login`, `POST /lti/login`, `POST /lti/launch`, `POST /lti/exchange-code` | `backend/app/routers/lti.py` | بروتوكول LTI/OIDC؛ إدارة المنصات تتطلب admin؛ JWKS/login/launch ليست APIs عامة لتسريب درجات | Existing LTI protocol routes؛ AGS passback advanced يبقى PR #154 فقط |
| AI Chat App `6645b0502daa6922040ee4c6bb5c281ed59ef366` | `POST /agent-jobs` | `ai-backend/app/routers/agent_jobs.py` | `get_current_user`; 202 Accepted؛ سياق المستخدم/Workspace والتحقق من ملكية job | Existing internal product API — لا S2S principal |
| AI Chat App | `GET /agent-jobs`, `GET /agent-jobs/{job_id}` | `ai-backend/app/routers/agent_jobs.py` | قائمة المستخدم وسجلات jobs المملوكة له فقط؛ MCP metadata منقح للمستخدم غير المخول | Existing owner-scoped product API |
| AI Chat App | `POST /agent-jobs/{job_id}/pause`, `POST /agent-jobs/{job_id}/resume` | `ai-backend/app/routers/agent_jobs.py` | تحقق مالك job وحالته قبل تعديل workflow | Existing user action; external invocation غير معتمد |
| ERP `7bb01d23328a68aae67c659fd6c32b6db033637d` | `POST /api/auth/login`, `GET /api/auth/me` | `backend/erp/api/auth.py` | Bearer JWT؛ عند اختيار الشركة يتحقق `cid/mid` من عضوية نشطة | Existing product route — لا ترسل token إلى منتج آخر |
| ERP | `GET /api/auth/companies`, `POST /api/auth/switch-company` | `backend/erp/api/auth.py` | يدرج memberships الخاصة بالمستخدم؛ switch يثبت العضوية ويصدر سياقاً جديداً | Existing company-context routes؛ لا multi-company operations |
| ERP | `GET /api/reports/dashboard`, `GET /api/reports/sales-summary` | `backend/erp/api/reports.py` | `reports.read` مع تمرير company context للخدمات | Existing product route sample؛ محجوب للتكامل التجاري الخارجي حتى release gates |
| ERP | `GET /api/{resource}`, `POST /api/{resource}`, `GET /api/{resource}/{id}`, `PATCH /api/{resource}/{id}`, `DELETE /api/{resource}/{id}` | `backend/erp/crud.py`، Blueprint لكل Resource | permission لكل Resource، و`company_id` من السياق الموثوق فقط، لا يقبل client-supplied company_id | Existing generic CRUD pattern؛ لا تعمم هذا الضمان على custom SQL غير المدقق |
| ERP | `GET /api/file-objects`, `POST /api/file-objects/upload`, `GET /api/file-objects/{id}/download`, `DELETE /api/file-objects/{id}` | `backend/erp/api/file_objects.py` | `documents.read/write` وcompany-scoped private file storage | Existing sample; يخضع للحاجز العام متعدد الشركات |

### 12.6 نتيجة فحص route prefixes ووجود الواجهات

- Lumen وAI Chat App يسجلان FastAPI routers مباشرة من `main.py` دون prefix شامل إضافي في موضع التسجيل الذي تمت مراجعته؛ لذلك أمثلة المسارات أعلاه هي المسارات المحلية للمنتج، وليست paths موحدة تبدأ بـ`/api/v1`.
- ERP يسجل Flask Blueprints تحت `/api` أو `/api/{resource}`، لذلك تختلف بنية المسار بوضوح عن المنتجين الآخرين.
- أنواع الواجهات غير متجانسة: FastAPI/Pydantic في Lumen وAI Chat App، وFlask/JSON في ERP. لا تُنشأ client SDK مشتركة أو افتراضات response envelopes قبل اعتماد schemas وعقود توافق صريحة.
- مسارات Lumen وERP وAI Chat المذكورة هنا عينة مثبتة من ملفات route محددة؛ لم يكتمل بعد استخراج جميع endpoints أو schemas ولا اختبارات كل consumer/provider. تبقى `S5-02 / ECO-001` In Progress.

### 12.7 متطلبات الانتقال من route موجود إلى عقد مقبول

لكل واجهة مرشحة في الجدول أعلاه يجب إضافة ملف OpenAPI/JSON Schema في مستودع المالك يتضمن: request/response models، status codes، auth audience/scopes، resource ownership، data classification، rate/payload limits، idempotency/retry semantics، trace metadata، retention، consumer/provider tests، وسيناريوهات deny/IDOR/revocation. لا يُنسخ تعريف endpoint إلى المستودع المركزي بوصفه مصدراً تنفيذياً، ولا يفتح الاتصال قبل موافقة مالك المنتج.

**تحديث الحكم:** اكتمل جرد مجموعات الواجهات وأضيفت عينات method/path موثقة من المصدر. لا يوجد حتى الآن عقد service-to-service معتمد أو consumer/provider tests أو service principal مشترك. تظل S5-02 In Progress.


## 13. خريطة المسارات الفعلية الموثقة — 2026-11-17

هذه خريطة لمجموعات المسارات التي تمت مراجعتها في source، وليست نسخة OpenAPI كاملة. وجود مسار في الجدول لا يعني وجود consumer service principal أو اعتماد تكامل من منتج آخر.

### 13.1 Lumen — FastAPI

**المصدر:** main `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`، مع مراجعة `backend/app/main.py` وملفات routers ذات الصلة.

| Prefix / المجموعة | العمليات المرصودة | نموذج التفويض |
|---|---|---|
| `/auth` | CSRF، التسجيل، الدخول، refresh | Bearer/session؛ عمليات الويب تتطلب CSRF حسب طريقة الطلب |
| `/users` و`/admin` | الملف الشخصي والإدارة | `get_current_user` أو `require_admin` حسب المسار؛ PR #164 يحمي آخر مدير نشط عبر قفل roster معاملاتي مشترك |
| `/curriculum`، `/quiz`، `/content`، `/srs`، `/assignments`، `/reports` | التعلم والتقييم والتقدم | نطاق الطالب الحالي والمعلم/admin في العمليات المحددة؛ `POST /assignments/{assignment_id}/submit` للطالب فقط (#163)؛ ليس سياق شركة عام |
| `/classes` | إنشاء الصفوف، قائمة الصفوف، تفاصيلها وتعديلها، رموز الدعوة والانضمام وإدارة الطلاب | إدارة الفصل للمعلم المالك أو admin؛ `POST /classes/join` للطالب فقط (#162)، وenrollment يربط student_user_id بالصف |
| `/lti` | platforms، JWKS، login، launch، exchange-code | إدارة المنصة admin-only؛ login/launch تدفق LTI منفصل |
| `/files` | upload/list/get/delete | ملفات مملوكة بـuser_id مع فحص النوع والحجم والحصة وفحص malware حسب إعداد الإنتاج |
| `/billing` و`/notifications` | الدفع والإشعارات | مسارات المنتج؛ لا تُمنح صلاحياتها تلقائياً لأي مستهلك service |

`backend/app/main.py` يسجل هذه routers مباشرة؛ لا نضيف بادئة `/api` إلى Lumen في وثائق العقد من دون دليل من نشره أو إعداد reverse proxy. لا يوجد في الجرد الحالي OpenAPI contract خارجي منشور ومعتمد لاستهلاك هذه الموارد من منتج آخر.

### 13.2 AI Chat App — FastAPI

**المصدر:** main `6645b0502daa6922040ee4c6bb5c281ed59ef366`.

| Prefix / المجموعة | العمليات المرصودة | نموذج التفويض |
|---|---|---|
| `/chat` | compare/models/vision/chat/stream/feedback/bookmark/delete/regenerate/edit | مستخدم المصادقة وملكية المحادثة/الملفات/مساحة العمل بحسب المسار |
| `/conversations` | list/get/update/pin/archive/folder/tags/assistant/branch/duplicate/import/export/trash/delete | ملكية المستخدم والمشاركات المسموح بها |
| `/files` | upload/list/attach/detach/get/reindex/index-image/delete | مالك الملف أو صلاحية workspace/project التي يعيد الخادم التحقق منها |
| `/workspaces` | list/create/update/model/limit/budget/usage/audit-logs | عضوية workspace وصلاحياتها |
| `/agent-jobs` | create/list/get/pause/resume/cancel | ملكية المستخدم وعضوية workspace؛ snapshots تخفي بيانات MCP العامة عن غير admin |
| `/projects` | create/list/import/export/update/members/validate/preview/artifacts/delete | project ACL؛ file tools تعيد التحقق من ACL عند كل عملية |
| `/api-keys` | create/list/revoke/usage | إدارة مفتاح المستخدم؛ لا تستخدمه خدمة أخرى كدليل هوية مستخدم نهائي |
| `/webhooks` | create/list/update/delete/deliveries/test | endpoints مملوكة للمستخدم؛ HMAC-SHA256 وتحقق SSRF وbounded retries |
| `/v1` | `GET /v1` discovery و`POST /v1/chat` | `X-API-Key`، الإصدار في `X-API-Version: v1`؛ التغيير الكاسر يتطلب major URL version جديداً |

- `AgentRuntime` نفسه خدمة داخلية bounded وليست endpoint عام مستقل؛ الواجهة التشغيلية الحالية للـbackground agent runs هي `/agent-jobs`. لا تعرض Runtime كـservice-to-service API حتى يضاف endpoint وعقد مستقلان.
- SDKs الرسمية Python وJavaScript تستعمل `POST /v1/chat`، لكنها لا تثبت وجود contract بين المنتجات الثلاثة تلقائياً؛ يلزم service identity وconsumer scope خاص.
- لا يُفوض MCP application-global إلا لمسار admin الحالي، ولا تُعتبر allowlist العامة بديلاً عن resource ACL.

### 13.3 ERP Global Pro — Flask

**المصدر:** main `7bb01d23328a68aae67c659fd6c32b6db033637d`.

| Prefix / المجموعة | العمليات المرصودة | نموذج التفويض |
|---|---|---|
| `/api/auth` | setup-status/setup/login/me/companies/switch-company/change-password | JWT مع `cid` و`mid` بعد تحقق membership؛ إعداد النظام فقط public |
| `/api/{resource}` | list/get/create/update/delete للموارد العامة | generic Resource يفرض company_id على كل CRUD ويرفض company_id من العميل |
| `/api/reports` | dashboard/sales-summary/top-products/top-customers/profitability/forecast/reorder/statement/ageing | permission checks مع company context؛ مراجعة المصدر شملت هذه المجموعة |
| `/api` custom business | payments/cancel/credit-applications/payment-accounts/POS checkout وغيرها | كل endpoint يحتاج مراجعة مخصصة للخدمة والاستعلامات والقيود |
| `/api/file-objects` | list/usage/upload/get/download/delete | company-scoped file metadata/storage مع private tenant directories |

**قيد تكامل ملزم:** التطبيق SQLite v8؛ `TENANT_DATA_SCOPING_ENABLED=False`. middleware يمنع العمليات التشغيلية عند وجود أكثر من شركة، كما أن role definitions لا تزال مشتركة وتبقى custom SQL/services/constraints بحاجة تدقيق شامل. لذلك هذه المسارات Existing داخل المنتج لكنها **Blocked** كعقود multi-company service-to-service حتى Stage25C وإثبات العزل والنسخ/الاستعادة وPostgreSQL/RLS.

### 13.4 مخاطر نطاق بيانات يجب أن تبقى ظاهرة

- AI Chat App Issue [#724](https://github.com/HVA09/ai-chat-app/issues/724) مفتوحة. PR [#725](https://github.com/HVA09/ai-chat-app/pull/725) يقترح Retain privately: نقل مرفقات المشروع إلى `workspace_id=NULL` مع الحفاظ على المالك والبايتات/Chunks، وحذف روابطها من المساعدين المشتركين؛ أضيفت اختبارات تمنع list/download/RAG عبر عضو Workspace. PR #725 ما زال Draft، CodeQL نجح 2/2؛ CI run `43990553917` لا يزال في Backend tests بينما Frontend/SDK/Preview Builder/Production Compose نجحت على HEAD `f75ceefe790538b96be071c4c86b4e072f25e626`؛ لا تعتمد السياسة أو تغلق Issue حتى اكتمال CI والمراجعة واعتماد المالك.
- Lumen PR #154 لا يحول AGS candidate إلى عقد LMS production: ما زال Draft، متأخراً 4 commits عن main في آخر comparison، وmergeable=false؛ CI 4/4 القديم لا يغلق Moodle/Canvas Sandbox وprivacy/clock gates.
- لا تعتبر مسارات ERP القائمة عقداً قابلاً للاستخدام من AI Core في بيئة تشغيل متعددة الشركات طالما release gate مغلق.

### 13.5 إغلاق جرد المسارات وما بقي

<badge color="success">Source map recorded</badge>

تم تحديد مجموعات الـrouters ومسارات الخدمة الممثلة ورؤوس main وملاك الهوية والبيانات. بقيت الخطوات التالية قبل اعتبار S5-02 Done:
1. استخراج OpenAPI/route schemas لكل endpoint مع request/response models والحقول nullable وحدود payload.
2. مراجعة كل controller/service/query في ERP قبل اعتماد أي custom business route خارجياً.
3. تحديد service principals منفصلة وissuer/audience/scopes/expiry/revocation لكل consumer.
4. إضافة provider/consumer contract tests وnegative authorization/IDOR tests، ثم اختبار في staging أو بيئة disposable.
5. لا بناء أو نشر Gateway مركزي كشرط مسبق؛ يمكن للعقود أن تعمل بين خدمات مستقلة مباشرة بعد موافقة المالك.

## 14. بطاقة عقد Developer API v1 — AI Chat App — 2026-11-17

**مرجع المصدر:** AI Chat App main `6645b0502daa6922040ee4c6bb5c281ed59ef366`.

| البند | العقد المثبت |
|---|---|
| Contract ID | `AIC-DEV-CHAT-001` |
| Owner | AI Chat App — Developer API |
| Consumer | مطور خارجي يستخدم API key شخصية؛ لا يوجد service principal بين المنتجات مثبت |
| Discovery | `GET /v1` |
| Operation | `POST /v1/chat` |
| Version | URL version `v1`، ورد ناجح يحمل `X-API-Version: v1` |
| Authentication | `X-API-Key: ak_live_...` |
| Request schema | `APIChatRequest` في `ai-backend/app/schemas/api_keys.py` |
| Response schema | `APIChatResponse` في الملف نفسه |
| Provider operation | يستدعي AI Chat App داخلياً `get_ai_reply`؛ لا يعرض endpoint provider مباشر |
| Persistence | ينشئ/يستخدم Conversation ويكتب user/assistant Messages وUsageLog تحت المستخدم المرتبط بالمفتاح |
| Rate limit | اختياري لكل مفتاح `daily_request_limit` ضمن نافذة متحركة 24h؛ بالإضافة إلى user-plan وworkspace AI limits |
| Compatibility | التغيير الكاسر في request/response/auth/semantics يتطلب major URL version جديداً مثل `/v2` |

### 14.1 Request / Response

**Request fields:**

| الحقل | النوع | القيد |
|---|---|---|
| `message` | string | مطلوب، 1–4000 حرف، trim، غير فارغ |
| `conversation_id` | integer | null | اختياري؛ يجب أن تكون المحادثة مملوكة للمستخدم المرتبط بالمفتاح وفي مساحة العمل التي حُلت للطلب |
| `workspace_id` | integer | null | اختياري؛ يحل الخادم Workspace المملوكة للمستخدم ويتحقق من الوصول |
| `assistant_id` | integer | null | اختياري؛ يحل الخادم المساعد المسموح استخدامه في مساحة العمل |
| `model` | string | null | اختياري، بحد أقصى 100 حرف؛ يمر عبر قائمة النماذج المتاحة للمستخدم |

**Important omission:** `APIChatRequest` لا يحتوي `project_id` أو `file_ids` أو `tools` أو `stream`. لا توحّد هذا العقد مع `POST /chat` الداخلي أو `/agent-jobs`؛ حقول المشروع/الملفات/الأدوات ليست جزءاً من هذه النسخة العامة.

**Response fields:**

| الحقل | النوع |
|---|---|
| `conversation_id` | integer |
| `reply` | string |
| `model` | string |

### 14.2 Authentication, ownership and denial semantics

- `APIKey` مرتبط بـ`user_id` عبر FK، وليس باسم تطبيق أو service principal. المفتاح يُنشأ بواسطة مستخدم مصادق عليه في `POST /api-keys`، ويُخزن `key_hash` مع `key_prefix`؛ السر الخام يعاد مرة واحدة عند الإنشاء.
- المصادقة في `_get_api_key_auth` تتحقق من وجود المفتاح، hash، عدم الإلغاء، عدم انتهاء `expires_at`، وكون حساب المالك فعالاً.
- إدارة `GET /api-keys` و`POST /api-keys` و`DELETE /api-keys/{key_id}` و`GET /api-keys/{key_id}/usage` تتطلب جلسة المستخدم، وكل عمليات الإدارة scoped إلى `current_user.id`.
- `conversation_id` يجب أن يطابق `Conversation.user_id` وworkspace الذي تم حله؛ الموارد غير المرئية ترجع 404 وفق مسار المحادثة.
- الوصول إلى Workspace وAssistant يعاد التحقق منه في خدمة المنتج. الـAPI key يحدد مالك الاستخدام ولا يثبت وحده عضوية Workspace أخرى.
- المستخدم ومفتاحه يخضعان أيضاً إلى `enforce_daily_ai_limit` و`enforce_workspace_daily_ai_limit`؛ لذلك عدم تعيين `daily_request_limit` للمفتاح لا يعفي من حدود خطة المستخدم/مساحة العمل.

### 14.3 Rate limit / failures

- `daily_request_limit` اختياري، وعند ضبطه يجب أن يكون 1–10000 عند إنشاء المفتاح.
- يستخدم التطبيق نافذة متحركة مدتها 24 ساعة ويصدر `X-RateLimit-Limit` و`X-RateLimit-Remaining` و`X-RateLimit-Reset` عند انطباق الحد.
- عند تجاوز حد المفتاح: HTTP 429 مع `Retry-After` وترويسات RateLimit.
- إذا تعذر limiter في production مع وجود limit: HTTP 503 و`Retry-After: 30` (fail closed).
- حسب السبب: 401 لمفتاح مفقود/غير صالح/ملغى/منتهي أو حساب غير فعال؛ 422 لمدخل غير مطابق للمخطط أو بيانات إنشاء المفتاح غير صالحة؛ 404 لمورد/مفتاح إدارة غير مملوك؛ 429 لتجاوز المعدل؛ 503 لتعذر خدمة تحديد المعدل في production.
- اختبارات المصدر: `ai-backend/tests/test_api_keys.py` تغطي الإنشاء/list/revoke، chat auth، owner scoping، invalid/revoked/expired keys، rate-limit headers/429 وmetadata/version header.

### 14.4 قرار إعادة الاستخدام بين المنتجات

<badge color="warning">Not a service identity</badge>

هذا عقد Developer API صالح لمطورين يستخدمون مفتاحاً شخصياً؛ **ليس** عقد ثقة service-to-service بين Lumen وERP وAI Core، للأسباب التالية:
1. لا يوجد في نموذج `APIKey` حقل service principal أو `audience` أو scopes مستقلة أو consumer identity.
2. المفتاح مرتبط بحساب شخص محدد ويستهلك حصته وحدوده وملكياته؛ تمريره من منتج آخر يخلط هوية المنتج بهوية المستخدم ويمنح المستهلك حدود المستخدم نفسه.
3. لا يوجد delegated-user grant مستقل أو token exchange موثق بين issuers.
4. لا يقبل العقد project/file/tool context المطلوب لبعض سيناريوهات المنتجات؛ لا تمدده ضمن v1 دون مراجعة التوافق.

**النتيجة:** يمكن استخدام هذا API من تكامل مطور خارجي بعد إصدار مفتاحه الشخصي، لكنه لا يؤهل AI Core أو Lumen أو ERP كـservice consumer. يتطلب الربط بين المنتجات عقداً منفصلاً مع service principal وissuer/audience/scopes، وربط delegated subject، وresource owner checks وrevocation واختبارات negative. لا مشاركة JWT أو API keys الشخصية بين المنتجات.

**روابط المصدر:**
- [api_keys.py](https://github.com/HVA09/ai-chat-app/blob/6645b0502daa6922040ee4c6bb5c281ed59ef366/ai-backend/app/routers/api_keys.py)
- [schemas/api_keys.py](https://github.com/HVA09/ai-chat-app/blob/6645b0502daa6922040ee4c6bb5c281ed59ef366/ai-backend/app/schemas/api_keys.py)
- [api_versioning.py](https://github.com/HVA09/ai-chat-app/blob/6645b0502daa6922040ee4c6bb5c281ed59ef366/ai-backend/app/services/api_versioning.py)
- [DEVELOPER_API.md](https://github.com/HVA09/ai-chat-app/blob/6645b0502daa6922040ee4c6bb5c281ed59ef366/ai-backend/docs/DEVELOPER_API.md)

**حالة S5-02:** تمت إضافة أول بطاقة عقد مفصلة لمجموعة عامة ثابتة. يبقى جرد schema لكل واجهات Lumen وERP وواجهات AI Chat الأخرى، وعقد service identity منفصل، واختبارات provider/consumer وstaging قبل اعتماد التكامل بين المنتجات.
## 15. بطاقة Learning API candidate — Lumen — 2026-11-17

**مرجع المصدر:** Lumen main `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`؛ `backend/app/routers/curriculum.py` و`backend/app/schemas/curriculum.py` و`backend/app/dependencies.py`.

| البند | العقد المثبت على main |
|---|---|
| Contract ID | `LUM-LEARN-001` (candidate / internal product API group) |
| Owner | Lumen — Curriculum service |
| Routes | `GET /curriculum/subjects`, `GET /curriculum/path`, `GET /curriculum/topics/{topic_id}`, `GET /curriculum/topics/{topic_id}/mastery`, `GET /curriculum/topics/{topic_id}/lessons` |
| Auth | المحتوى العام عبر بعض GET routes؛ البيانات الشخصية في `/path` و`/mastery` و`/lessons` تستخدم `get_current_user` |
| Data owner | Lumen user/student identity, curriculum, lessons, mastery and progress |
| Versioning | المسارات الحالية لا تظهر تحت `/v1` ولا يوجد contract-version header موثق لها في المصدر الذي تمت مراجعته |
| Consumer readiness | Existing routes inside Lumen; no service principal or approved cross-product consumer contract demonstrated |

### 15.1 Public curriculum catalog vs authenticated progress

- `GET /curriculum/subjects` يرجع subjects النشطة وموضوعاتها عبر `SubjectWithTopicsOut`؛ هذا محتوى المنهج وليس بيانات تقدم مستخدم.
- `GET /curriculum/topics/{topic_id}` يرجع `TopicOut`؛ route لا يضيف `get_current_user` في التنفيذ الحالي، لذلك يصنف كمحتوى موضوع عام وفق سياسة التطبيق الحالية.
- `GET /curriculum/path` يتطلب `get_current_user` ويرجع `LearningPathOut`: subjects/topics مع recommended_topic_id وtopics_total وtopics_mastered وoverall_mastery.
- `GET /curriculum/topics/{topic_id}/mastery` يتطلب current user ويقيد الاستعلام بـ`TopicMastery.user_id == current_user.id`; إن لم توجد محاولة يعيد mastery=0 وattempts_count=0 بدلاً من كشف سجل مستخدم آخر.
- `GET /curriculum/topics/{topic_id}/lessons` يتطلب current user ويرجع الدروس المنشورة مع completed flag المحسوب من `LessonProgress.user_id == current_user.id`.

### 15.2 Response schema boundaries

**`LearningPathOut`:**
- `subjects: PathSubjectOut[]`
- `recommended_topic_id: integer | null`
- `topics_total: integer`
- `topics_mastered: integer`
- `overall_mastery: number`

**`PathSubjectOut`:** `id`, `slug`, `name_ar`, `name_en`, `topics: PathTopicOut[]`.

**`PathTopicOut`:** `id`, `slug`, `name_ar`, `name_en`, `learning_outcomes_ar`, `simulation_type`, `prerequisite_topic_id`, `prerequisite_name_ar`, `status` (`locked|available|in_progress|mastered`), `mastery_percentage`, `gateway_quiz_passed`, `lessons_total`, `lessons_completed`, `questions_count`.

**`MasteryOut`:** `topic_id`, `mastery_percentage`, `gateway_quiz_passed`, `attempts_count`, `last_practiced_at`.

**`LessonOut`:** `id`, `topic_id`, `slug`, `title_ar`, `body_md`, `estimated_minutes`, `order_index`, `completed`.

### 15.3 Privacy and authorization contract

1. Subject/topic catalog fields can be considered public curriculum content only according to Lumen's own publication rules.
2. LearningPath/mastery/lesson completion fields are student-specific progress data; do not send them to AI Core, ERP, or another product merely because a valid Lumen browser token exists.
3. Any external consumer requires a Lumen-issued service identity, explicit audience and scopes, a delegated subject mapping, user consent/legal basis where applicable, and server-side authorization for the requested student/resource.
4. A caller-supplied `user_id` must never replace the current principal; current routes derive mastery/progress ownership from authenticated `current_user.id`.
5. Teacher/class APIs have separate role/ownership rules and may return student identifiers, display names, email addresses and mastery aggregates. They are outside this learning-path read contract and require separate class-scope authorization review.
6. Do not expose admin-only content or LTI platform credentials to AI Core or external consumers.

### 15.4 Cross-product reuse decision

<badge color="warning">Candidate only — no service contract</badge>

These routes establish that Lumen already owns the learning-path and progress services; they do not establish a service-to-service interface. A future AI tutor may request narrowly scoped curriculum content or an authorized student's progress only through a Lumen-owned adapter. It must not query Lumen's database, reuse its browser cookie, or gain `teacher`/`admin` role. Lumen remains authoritative for lesson mastery and progression; AI Core may return explanations or suggestions but must not write mastery or quiz outcomes unless a separately reviewed Lumen operation exists.

**Source links:**
- [curriculum.py](https://github.com/HVA09/lumen-learning-platform/blob/fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8/backend/app/routers/curriculum.py)
- [schemas/curriculum.py](https://github.com/HVA09/lumen-learning-platform/blob/fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8/backend/app/schemas/curriculum.py)
- [dependencies.py](https://github.com/HVA09/lumen-learning-platform/blob/fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8/backend/app/dependencies.py)

**حالة العقد:** route/schema map مثبت للمسارات المذكورة فقط. ليس OpenAPI external contract ولا permission grant ولا integration. تبقى service identity, delegation, student-consent/data-retention, consumer/provider tests وموافقة Lumen prerequisites.
## 16. بطاقة سياق الشركة وتبديلها — ERP Global Pro — 2026-11-17

**مرجع المصدر:** ERP main `7bb01d23328a68aae67c659fd6c32b6db033637d`؛ `backend/erp/api/auth.py` و`backend/erp/security.py` و`backend/erp/services/tenancy.py` و`backend/erp/config.py` و`backend/erp/__init__.py`.

| البند | العقد المثبت على main |
|---|---|
| Contract ID | `ERP-COMPANY-CONTEXT-001` (existing interactive auth flow / external integration blocked) |
| Owner | ERP Global Pro — Authentication / Tenant Context |
| Base route group | `/api/auth` |
| Routes | `GET /api/auth/setup-status`, `POST /api/auth/setup`, `POST /api/auth/login`, `GET /api/auth/me`, `GET /api/auth/companies`, `POST /api/auth/switch-company`, `POST /api/auth/change-password` |
| Auth | Bearer JWT for authenticated routes; `/setup-status`, first-run `/setup`, and `/login` are public bootstrap/auth routes |
| Company context | JWT claims `sub`, `iat`, `exp`, plus `cid` and `mid` once a company membership is selected |
| Effective authorization | current active membership's role and permissions, revalidated on each request |
| Active DB | Flask + SQLite v8; PostgreSQL is a future cutover target, not the application DB today |
| Multi-company gate | `TENANT_DATA_SCOPING_ENABLED=False`; operations fail closed if company_count > 1 |

### 16.1 Company-selection flow

- `POST /api/auth/login` accepts JSON `email` and `password`; successful login returns `token` and `user` (including role/permissions and any unambiguous default active_company).
- `GET /api/auth/companies` requires a valid user token and returns active memberships, `active_company`, `company_count`, and `can_use_operations`.
- `POST /api/auth/switch-company` accepts a positive integer `company_id` in JSON. The server resolves an active membership for the authenticated `g.user.id`; the client-provided ID is a selection request, not proof of access.
- If the membership is valid, the server issues a new signed JWT carrying `sub`, `iat`, `exp`, `cid`, and `mid`, returns the user permission payload and `active_company`, and records an audit event.
- On every request, `security.load_user()` verifies JWT signature/expiry, user active state, and re-resolves the membership by user + company + membership ID. Revoked membership or suspended company invalidates the company-bound context.
- `current_company_id()` uses trusted `g.company` during HTTP requests and rejects an explicitly supplied `company_id` if it differs from the authenticated company.

### 16.2 Example response contract and errors

**`GET /api/auth/companies` response fields:**
- `items`: active memberships with `membership_id`, `company_id`, `company_name`, country, currency, industry, role and status.
- `active_company`: `{id, name, country, currency, industry, role, membership_id}` or null.
- `company_count`: integer.
- `can_use_operations`: boolean; does not assert that multi-company production is enabled.

**`POST /api/auth/switch-company` response fields:**
- `token`: newly issued company-bound JWT.
- `user`: `{id, email, name, role, permissions, active_company}`.
- `active_company`: selected company's metadata and membership identity.

**Observed denial/fail-closed behavior:** 401 when auth is missing/invalid; 403 when the user lacks active membership or attempts a company-context mismatch; 409 when active company context is absent or an ambiguous legacy membership cannot be resolved; 422 for invalid `company_id` input; 503 when operational access is blocked because more than one company exists while tenant isolation is disabled.

### 16.3 Cross-product trust boundary

<badge color="warning">Existing product API — blocked for multi-company external use</badge>

1. These JWTs are interactive ERP user tokens, not service-to-service credentials. The inspected token claims include `sub/cid/mid` but no dedicated service principal, issuer/audience contract or consumer scopes.
2. Do not send the ERP bearer token to AI Chat App, Lumen or AI Core; do not accept a `company_id` from an external consumer as proof of membership.
3. `roles` and permission templates are still shared/global in the current baseline and require tenant-aware design before multiple-company operations can be enabled.
4. `TENANT_DATA_SCOPING_ENABLED=False` is a hard release gate. The Flask app factory refuses operational requests when multiple companies exist. Generic CRUD and sampled reports/import-export/file APIs have scoped code, but complete SQL/service/report/job/constraint audit and verified restore/preflight are still outstanding.
5. No AI service may read or mutate ERP business records through direct SQLite access. A future connector needs a separate service identity with explicit delegation to one user and one verified company, plus per-resource scopes, audit and negative IDOR tests after the ERP release gates close.

**Source links:**
- [auth.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/api/auth.py)
- [security.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/security.py)
- [services/tenancy.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/services/tenancy.py)
- [config.py](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/config.py)
- [application factory](https://github.com/HVA09/erp-global-pro/blob/7bb01d23328a68aae67c659fd6c32b6db033637d/backend/erp/__init__.py)

**حالة العقد:** current-company context and membership validation exist inside ERP. This card is not an approval for ERP multi-company operation or an external AI consumer. Both remain blocked until Stage25C actual restored-copy preflight, full tenant isolation, PostgreSQL/RLS and backup/restore/rollback gates are accepted.