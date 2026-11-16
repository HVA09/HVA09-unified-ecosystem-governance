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
