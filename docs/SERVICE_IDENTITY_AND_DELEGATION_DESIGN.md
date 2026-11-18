# تصميم هوية الخدمات والتفويض بين المنتجات

- **المعرف:** S5-03 ضمن خارطة المراحل الخمس.
- **الحالة:** Design baseline — لا service principal أو token issuer أو SSO مشترك منفذ.
- **التاريخ:** 2026-11-17.
- **القاعدة:** هوية الخدمة تثبت هوية التطبيق المستهلك، ولا تمنحه تلقائياً صلاحية الوصول إلى موارد المستخدم أو الطالب أو الشركة.

## 1. المشكلة والهدف

تحتاج المنظومة عند بدء التكامل إلى طريقة آمنة لتمييز:
1. **المستخدم النهائي** الذي قام بالفعل داخل منتجه الأصلي.
2. **الخدمة المستهلكة** التي تطلب عملية من منتج آخر.
3. **المورد المملوك** للمنتج المصدر، مثل محادثة أو ملف أو صف أو شركة.
4. **الصلاحية المفوضة** التي يسمح بها مالك المورد لهذه العملية المحددة.

يجب ألا تخلط العقود بين هذه الهويات. لا تتطابق أرقام `user_id` أو `student_id` أو `workspace_id` أو `company_id` بين قواعد المنتجات، ولا يصلح أي منها كـprincipal identity مشترك.

## 2. خط الأساس الفعلي

| المنتج | آلية المستخدم الحالية | ما لا يثبت وجوده |
|---|---|---|
| Lumen | Bearer access token أو browser session cookie؛ CSRF لطلبات cookie غير الآمنة؛ أدوار user/teacher/admin | لا يوجد في المسارات التي تمت مراجعتها service principal مخصص للاستهلاك بين المنتجات |
| AI Chat App | جلسات المستخدم وJWT، API keys لواجهة Developer API، OAuth connections لموصلات المستخدم | Developer API key المرتبط بمستخدم ليس بديلاً عن هوية خدمة بين المنتجات، ولا يمنح وصولاً تلقائياً إلى بيانات منتج آخر |
| ERP Global Pro | JWT مع claims مرتبطة بالشركة/العضوية `cid/mid` بعد التحقق من membership؛ الدور والصلاحيات مرتبطان بعضوية الشركة | لا يوجد حتى الآن service identity مشترك؛ لا يرسل JWT المستخدم إلى منتج آخر |

هذه الآليات تظل محلية لكل منتج. لا مشاركة لمفاتيح توقيع الجلسات أو Cookies أو refresh tokens.

## 3. نوعا الاعتماد المطلوبان

### 3.1 Service principal

لكل منتج مستهلك معرف خدمة مستقل، مثلاً:
- `lumen-adapter`
- `ai-chat-adapter`
- `erp-adapter`
- `ai-core-gateway` عند إنشائه

سجل الهوية المقترح:
- `client_id` ثابت وغير حساس.
- مالك الخدمة والمستودع والبيئة (dev/staging/prod).
- `issuer` موثوق و`audience` واحد لكل API مستهدف.
- public key set مع `kid`، وحالة المفتاح ووقت التدوير.
- scopes محددة بدقة، وسجل إصدار/إلغاء.
- تاريخ انتهاء الاعتماد ومالك التدوير والاستجابة للحوادث.

لا تستخدم secret مشتركاً بين الخدمات أو مفتاح JWT نفسه عبر المنتجات.

### 3.2 Delegated user context

عندما تعتمد العملية على طلب مستخدم، يمكن للمنتج الذي يملك جلسة المستخدم إصدار تفويض قصير العمر ومحدد الوجهة، يربط:
- `sub`: subject namespaced من issuer المنتج الأصلي، وليس user ID خاماً مشتركاً.
- `act`: service principal الذي ينفذ الطلب نيابة عن المستخدم.
- `aud`: API المنتج المالك للمورد.
- `scope`: العملية والمورد المسموح بهما.
- `iat/nbf/exp/jti`: صلاحية زمنية ومعرف يمنع إعادة الاستخدام.
- `purpose` أو `delegation_ref`: غرض موثق عند الحاجة، دون نصوص أو بيانات حساسة.

لا يقبل المنتج المالك `user_id` أو `company_id` الوارد في JSON كإثبات تفويض. عليه التحقق من الاعتماد والـaudience والـscope، ثم إعادة التحقق من العضوية وملكية المورد الحالية.

## 4. شكل claims مقترح

هذا مثال تصميمي وليس token فعلياً ولا يحدد خوارزمية توقيع معتمدة. يجب وضع `kid` في **Protected JOSE Header** وفق RFC 7515 §4.1.4، وليس كـclaim داخل payload. المثالان التاليان توضيحيان فقط:

Protected JOSE Header:
```json
{
  "typ": "JWT",
  "alg": "ES256",
  "kid": "key-version-identifier"
}
```

JWT Claims Payload:
```json
{
  "iss": "https://identity.lumen.example",
  "sub": "urn:lumen:user:opaque-subject",
  "aud": "erp-business-api",
  "azp": "lumen-adapter",
  "act": {"sub": "lumen-adapter"},
  "scope": "business.read.limited",
  "purpose": "approved-learning-workflow",
  "iat": 0,
  "nbf": 0,
  "exp": 0,
  "jti": "unique-token-id"
}
```

قواعد التحقق:
1. خوارزميات توقيع غير متماثلة allowlisted فقط؛ يمنع قبول `alg=none` أو تغيير الخوارزمية من token.
2. `iss` و`aud` exact-match مع قائمة موثوقة؛ لا wildcard audience.
3. عمر قصير قابل للتهيئة، ورفض `exp/nbf` غير صالحين، مع tolerance محدود لانحراف الساعة.
4. اقرأ `kid` من Protected JOSE Header، ثم تحقق منه داخل key set موثوق مع تدوير وإلغاء؛ لا تقبل قيمة `kid` من payload بديلاً عن header، ولا تنزّل JWKS من URL يقدمه العميل.
5. `jti` لمنع replay للعمليات الحساسة أو إضافة nonce/receipt حيث يلزم.
6. لا scopes عامة مثل `*` ولا تفويضاً يتجاوز الدور أو policy الخاصة بالمنتج المصدر.

## 5. آلية النقل المرشحة

خيارات تحتاج قراراً أمنياً منفصلاً:
- OAuth 2.0 Client Credentials مع `private_key_jwt` أو credential غير متماثل.
- mTLS للخدمات التي تستطيع إدارة شهادات الخدمة والتدوير.
- Signed delegated token صادر من المنتج المالك عند الحاجة إلى user-on-behalf-of context.

لا يعتمد هذا التصميم خوارزمية أو بروتوكولاً نهائياً قبل مراجعة library support وkey lifecycle وdeployment topology. يمكن الجمع بين client authentication وmTLS كدفاع إضافي عند الحاجة، لا كشرط مبكر دون قدرة تشغيلية.

## 6. Scopes ونطاق الموارد

يجب أن يكون لكل scope:
- فعل واضح (read/list/create/update/execute).
- نوع مورد واضح.
- مستهلك محدد.
- حدود بيانات/tenant.
- مستوى خطر ومطلب موافقة عند انطباقه.

أمثلة تصميمية فقط:
- `model.generate`
- `model.stream`
- `model.vision`
- `learning.content.read.limited`
- `workspace.agent.invoke`
- `erp.business.read.limited`

لا تمنح scope التنفيذ العام لـTools أو MCP لمستهلك خارجي. أفعال T2/T3 تحتاج موافقة ونطاقاً محدداً وسجل تدقيق من المنتج المالك.

## 7. الإلغاء والتدوير والاستجابة للحوادث

- مفاتيح منفصلة لكل consumer وبيئة؛ لا dev key في الإنتاج.
- تدوير بالمبدأ overlap قصير: مفتاح جديد يصبح active، ثم يلغى القديم بعد انتقال المستهلكين.
- عند إلغاء خدمة أو اختراق مفتاح: تعطيل client_id أو kid، إبطال/denylist للـjti عند الحاجة، ورفض فوري للطلبات الجديدة.
- access tokens قصيرة العمر تقلل نافذة الضرر؛ لا تخزن refresh tokens لخدمات machine-to-machine ما لم يبررها البروتوكول.
- تسجيل issuer/client_id/audience/scope/request_id ونتيجة التفويض؛ لا تسجل token أو private key أو بيانات المستخدم الخام.
- فصل مفاتيح التوقيع عن مفاتيح مزودي AI وقواعد بيانات المنتجات.

## 8. حدود صلاحية المنتج المالك

- Lumen هو الوحيد الذي يقرر وصولاً تعليمياً إلى مقرر أو واجب أو طالب أو درجة؛ service principal لا يتحول إلى teacher/admin.
- AI Chat App هو الذي يقرر ملكية المحادثة والملف والـWorkspace والـProject وحقوق تشغيل Agent/Tool.
- ERP هو الذي يتحقق من عضوية الشركة وصلاحية العملية في سياق الشركة؛ أي بيانات تشغيل متعددة الشركات تبقى محجوبة حتى إغلاق release gates.
- AI Core لا يحتفظ بهوية أعمال أو user database مشتركة؛ يتحقق من هوية المستهلك ويطلب من مالك المورد صلاحية العملية.
- لا قراءة مباشرة لقواعد المنتجات، ولا impersonation غير مسجل، ولا SSO شامل ضمن S5-03.

## 9. اختبارات القبول

- رفض issuer/audience/algorithm/key id غير صحيح.
- رفض scope مفقود أو زائد أو لمورد خارج النطاق.
- انتهاء token، clock skew، replay، token revoked، key rotation.
- رفض service principal من بيئة مختلفة (staging مقابل production).
- إثبات أن user subject من Lumen لا يُفسر كـuser ID في ERP أو AI Chat App.
- إلغاء العضوية أو تغيير role يؤثر في العملية التالية وفق سياسة المنتج المصدر.
- محاولة cross-workspace/cross-project/cross-company IDOR.
- منع استخدام service principal كبديل عن owner/admin أو لتشغيل أداة بلا موافقة.
- عدم ظهور bearer tokens أو claims الحساسة أو محتوى الموارد في logs.

## 10. مراحل الاعتماد

1. اعتماد threat model والمستهلكين والموارد المطلوبة.
2. توثيق audiences/scopes وعقد إصدار/إلغاء/تدوير لكل خدمة.
3. اختيار آلية client authentication والتوقيع بعد مراجعة المكتبات والبيئة.
4. تنفيذ verifier داخل API المالك، مع اختبارات سلبية.
5. بناء consumer/provider contract tests في مستودعات المنتجات.
6. تجربة disposable/staging بهوية واحدة ونطاق قراءة محدود.
7. توسيع المستهلكين تدريجياً بقرار مالك كل منتج.

**خارج النطاق:** لا SSO، ولا توحيد user IDs، ولا نقل session secrets، ولا إصدار مفاتيح فعلية، ولا فتح API إنتاجية في هذه المهمة.

## 11. المراجع

- `PRODUCT_OWNERSHIP_AND_INTERFACE_CATALOG.md`
- `PRODUCT_API_CONTRACT_CATALOG.md`
- `AI_CORE_MODEL_GATEWAY_DESIGN.md`
- `AI_CORE_CAPABILITY_INVENTORY.md`
- DEC-005 وDEC-008 وDEC-010 وDEC-015

**الخلاصة:** المطلوب فصل هوية الخدمة عن هوية المستخدم وعن ملكية المورد. service token يثبت من هو المستهلك؛ المنتج المالك وحده يثبت هل يجوز له الوصول إلى مورد معين وما العملية المسموحة.

## 12. ربط التصميم بعقود المنتجات التي تم التحقق منها — 2026-11-17

هذه أمثلة flow مرشحة لتوجيه التنفيذ والاختبارات؛ لا تنشئ صلاحيات فعلية ولا تعني أن المستهلكين الحاليين يستطيعون استخدام المسارات.

### 12.1 Lumen — LUM-LEARN-001

المسارات الحالية: `/curriculum/subjects`, `/curriculum/path`, `/curriculum/topics/{topic_id}`, `/curriculum/topics/{topic_id}/mastery`, `/curriculum/topics/{topic_id}/lessons`.

- `/curriculum/subjects` وبعض محتوى الموضوعات لا يطلب هوية المستخدم في التنفيذ الحالي؛ يمكن أن يبقى محتوى منهجياً عاماً حسب سياسة النشر في Lumen.
- `/curriculum/path`, `/mastery` و`/lessons` تعتمد `get_current_user` وتعيد معلومات تقدم الطالب أو إتمام الدروس الخاصة بالمستخدم المصادق عليه.
- لا يرسل AI Core `user_id` رقمياً من AI Chat App على أنه هوية Lumen. يحدد Lumen آلية mapping بين `issuer + subject` والطالب المحلي، أو يطلب تفويضاً من Lumen نفسه.
- **Flow مقترح:** خدمة تعليمية مسجلة في Lumen → طلب delegated token محدود إلى audience Lumen → تحقق Lumen من issuer/audience/scope/expiry → يحدد الطالب المرتبط بالـsubject من قاعدة Lumen → يعيد فقط الحقول المسموحة لمسار/موضوع محدد.
- scopes candidate: `learning.catalog.read`, `learning.path.read.self`, `learning.mastery.read.self`, `learning.lessons.read`. هذه أسماء تصميمية وليست صلاحيات معرفة في كود Lumen حالياً.
- لا يسمح هذا التفويض بإدارة الصفوف أو كشف بيانات زملاء الفصل أو الوصول إلى `teacher/admin` APIs. تفاصيل المعلم والصف تحتاج scope منفصل وموافقة Lumen.

### 12.2 AI Chat App — AIC-DEV-CHAT-001

المسار الحالي: `POST /v1/chat`, authentication عبر `X-API-Key: ak_live_...`.

- نموذج `APIKey` الحالي يحمل `user_id` ويخضع لحصص المستخدم ومساحات العمل؛ لا يوجد `client_id`, `audience`, service principal أو delegation grant في نموذج المفتاح.
- **ممنوع:** تخزين مفتاح مطور شخصي خاص بمستخدم داخل AI Core أو Lumen أو ERP واستخدامه كاعتماد خدمة دائم. هذا يخلط هوية المستخدم بهوية المستهلك ويجعل استهلاك AI باسم مالك المفتاح.
- **Flow مقترح منفصل:** AI Core service principal موثوق → token قصير العمر audience=`ai-chat-internal-api` وscope لعملية واحدة → AI Chat API يتحقق من client identity ثم delegated user subject عند الحاجة → AI Chat يعيد التحقق من Conversation/Workspace/Project ownership والصلاحيات الحالية → ينفذ العملية ضمن الحدود المسموحة.
- scopes candidate: `chat.invoke`, `agent.job.create`, `workspace.resource.read.limited`. لا تعطِ `tools.execute.*` أو MCP server access عام أو صلاحيات إدارة المستخدم.
- إذا لم يتوفر delegated subject موثوق أو لم تثبت ملكية المحادثة/Workspace/Project، يفشل الطلب مغلقاً. لا يُنشأ conversation تحت مالك الخدمة إلا لسيناريو machine-owned صريح ومراجع.

### 12.3 ERP Global Pro — ERP-COMPANY-CONTEXT-001

المسارات الحالية: `/api/auth/login`, `/api/auth/companies`, `/api/auth/switch-company`, ثم business APIs تحت `/api/...`.

- JWT الحالي هو JWT مستخدم تفاعلي يحمل `sub` و`cid` و`mid` عند اختيار الشركة؛ لا يثبت هوية خدمة خارجية ولا يملك audience خاصاً بمستهلك.
- **ممنوع:** تمرير bearer JWT الخاص بمستخدم ERP إلى AI Core أو اعتبار `company_id` في body/header إثبات عضوية.
- **Flow المقترح بعد إغلاق release gate فقط:** service principal يحصل على delegated request موجه إلى ERP API → ERP يثبت service audience/scope → ERP يربط subject بالـuser mapping المعتمد → ERP يتحقق من عضوية الشركة والصلاحية الحالية على سجلاته → كل query تتم داخل company context موثوق وتحت العزل المعتمد.
- scopes candidate: `erp.customer.read.limited`, `erp.inventory.read.limited`, `erp.report.read.limited`; لا scopes مالية/كتابية عامة ضمن pilot.
- **Release gate مغلق حالياً:** SQLite v8 هي DB النشطة و`TENANT_DATA_SCOPING_ENABLED=False`؛ app factory يمنع العمليات التشغيلية عند وجود أكثر من شركة. لذلك لا ينفذ AI Core أي read/write تشغيلي على ERP حتى إثبات Stage25C وtenant isolation وPostgreSQL/RLS والنسخ/الاستعادة والrollback.

### 12.4 قواعد claims والـcontext المشتركة

| المعلومة | المصدر الموثوق | هل يرسلها المستهلك مباشرة؟ | قرار التحقق |
|---|---|---|---|
| service identity | issuer / key registry لدى مالك API | لا كـ`client_id` ادعائي فقط؛ التوقيع يثبتها | تحقق signature + issuer + audience + active client |
| end user subject | issuer المنتج الأصلي أو delegation grant | لا كـraw numeric `user_id` | تحقق mapping داخل المنتج المالك |
| workspace | AI Chat App | لا كدليل عضوية | إعادة lookup للعضوية وresource ACL في AI Chat |
| project | AI Chat App | يمكن أن يكون locator فقط | إعادة فحص project ACL داخل handler |
| company | ERP | لا كدليل تفويض | ERP يتحقق من cid/mid/membership وtenant context |
| student/class/grade | Lumen | لا كدليل ملكية أو access | Lumen يتحقق من user/class enrollment/teacher ownership |

### 12.5 قرار آلية التنفيذ — ما زال مفتوحاً

الخيارات المرشحة هي OAuth 2.0 Client Credentials مع `private_key_jwt`، أو mTLS، مع signed delegated token محدود عند الحاجة إلى سياق المستخدم. الاختيار النهائي يجب أن يقارن دعم المكتبات الحالية، إدارة المفاتيح/JWKS، التدوير والإلغاء، trust boundaries، مراقبة replay، تعقيد التشغيل، وإمكان الاختبار في بيئة staging.

لا تفترض أن وجود OpenID Connect أو OAuth user connectors في AI Chat App يعني وجود OAuth authorization server بين المنتجات. لا يوجد حتى هذا التحديث أي service principal أو issuer مركزي أو token exchange أو mTLS deployment منفذ.

**Acceptance gate:** لا يصبح S5-03 Verified قبل اختيار آلية موثقة، واعتماد مالكي المنتجات، ونشر verifier داخل المنتج المالك، واختبارات cross-issuer/audience/scope/tenant/subject وrevocation/replay على بيئة disposable، مع إثبات عدم تسرب مفاتيح أو بيانات موارد إلى السجلات.