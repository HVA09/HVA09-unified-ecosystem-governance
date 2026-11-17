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


### 12.5 أمثلة endpoints فعلية تم التحقق منها من route decorators

الأمثلة التالية تثبت وجود route في main فقط؛ لا تعني أن endpoint معتمد للاستهلاك بين المنتجات أو أن OpenAPI consumer contract قد تم اعتماده. المسارات مبنية على router prefix وعمليات `include_router/register_blueprint` في commit المالك.

| المنتج / SHA | Method + path | الدليل | المصادقة/التحكم المرصود | حالة العقد |
|---|---|---|---|---|
| Lumen `ed162b0e0af497abc5710d185fa357357baa6ecc` | `POST /auth/register` | `backend/app/auth/routes.py` + router prefix `/auth` | إنشاء حساب ذاتي؛ يجب ألا يُستخدم كـservice provisioning | Existing product route — لا consumer contract |
| Lumen | `GET /curriculum/subjects` | `backend/app/routers/curriculum.py` + prefix `/curriculum` | سياق مستخدم Lumen عبر dependencies حيث ينطبق | Existing product route — بيانات تعليمية |
| Lumen | `POST /classes`, `POST /classes/join` | `backend/app/routers/classes.py` | إنشاء الفصل يتطلب teacher؛ الانضمام عبر دعوة؛ الفصل مملوك للمعلم | Existing product route — لا organization tenant |
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

**المصدر:** main `ed162b0e0af497abc5710d185fa357357baa6ecc`، مع مراجعة `backend/app/main.py` وملفات routers ذات الصلة.

| Prefix / المجموعة | العمليات المرصودة | نموذج التفويض |
|---|---|---|
| `/auth` | CSRF، التسجيل، الدخول، refresh | Bearer/session؛ عمليات الويب تتطلب CSRF حسب طريقة الطلب |
| `/users` و`/admin` | الملف الشخصي والإدارة | `get_current_user` أو `require_admin` حسب المسار |
| `/curriculum`، `/quiz`، `/content`، `/srs`، `/assignments`، `/reports` | التعلم والتقييم والتقدم | نطاق الطالب الحالي والمعلم/admin في العمليات المحددة؛ ليس سياق شركة عام |
| `/classes` | إنشاء الصفوف، قائمة الصفوف، تفاصيلها وتعديلها، رموز الدعوة والانضمام وإدارة الطلاب | المعلم المالك أو admin؛ enrollment يربط student_user_id بالصف |
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
