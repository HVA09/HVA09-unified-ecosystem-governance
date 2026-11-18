# نموذج التفويض الموحد بين المنتجات — RBAC/ABAC

- **المعرف:** S5-04 ضمن خارطة المراحل الخمس.
- **الحالة:** Design baseline — لا policy engine مركزي أو enforcement مشترك منفذ.
- **التاريخ:** 2026-11-17.
- **الهدف:** توحيد طريقة وصف قرار التفويض، لا توحيد أدوار التطبيقات أو نقل سلطة القرار من مالك المورد.

## 1. المبدأ الحاكم

المنظومة تستخدم عقد تقييم مشتركاً، لكن كل منتج يبقى Policy Enforcement Point (PEP) وصاحب القرار النهائي على موارده. لا توجد رتبة `admin` عابرة للمنتجات، ولا يعني قبول هوية الخدمة منحها صلاحية على بيانات المستخدم أو الطالب أو الشركة.

قرار السماح يجب أن يثبت جميع الحدود المطلوبة:

`ALLOW = valid_service_identity ∩ valid_delegation ∩ service_scope ∩ user_permission ∩ resource_acl ∩ tenant_boundary ∩ purpose_policy`

فشل أي شرط ينتج deny أو approval_required؛ لا تُجمع الصلاحيات بالاتحاد ولا تُرفع صلاحية الخدمة فوق صلاحية المستخدم أو مالك المورد.

## 2. الحقائق الفعلية لكل منتج

### 2.1 Lumen

**المصدر:** Lumen main الحالي بعد PR #164: `fe3a87591a6a6c5f909b0dba162cdd8b4ba47ce8`.

- `UserRole` على مستوى حساب المنصة: `user`, `teacher`, `admin`.
- `get_current_user` يقبل Bearer أو browser session cookie، ويتحقق من session binding وCSRF لطلبات cookie غير الآمنة.
- `require_teacher` يسمح teacher/admin؛ `require_admin` محصور بـadmin.
- `POST /classes/join` يفرض `UserRole.user` فقط؛ teacher/admin يحصلان على 403 قبل إنشاء enrollment (PR #162).
- `POST /assignments/{assignment_id}/submit` يفرض دور الطالب فقط ويرفض teacher/admin قبل lookup/creation للـsubmission (PR #163).
- حماية آخر مدير نشط تشترك فيها مسارات حذف الحساب الذاتي وتغيير الدور/التفعيل والحذف الإداري وbootstrap/recovery عبر transaction-scoped PostgreSQL advisory lock وإعادة قراءة actor/target بعد القفل (PR #164). هذا لا يعني أن recovery نُفّذ على حساب أو قاعدة إنتاج.
- الفصول مملوكة عبر `ClassSection.teacher_user_id`; إدارة الفصل للمعلم المالك أو admin؛ الطالب يكتسب نطاق الفصل من `ClassEnrollment.student_user_id`.
- mastery وlesson progress مملوكان للمستخدم الحالي؛ لا يقبل الخادم user_id مرسلاً من العميل كبديل لهوية current_user في المسارات التي تمت مراجعتها.
- هذه أدوار تعليمية محلية وليست company roles أو workspace roles؛ لا تُحوّل `teacher` إلى دور مدير عام في AI Core.

### 2.2 AI Chat App

**المصدر:** main `6645b0502daa6922040ee4c6bb5c281ed59ef366`.

- Workspace roles القديمة: `owner`, `admin`, `member`.
- `owner` يملك مجموعة RBAC كاملة؛ `admin` يملك الأذونات باستثناء `rbac.manage`؛ `member` الافتراضي محدود إلى `workspace.read`, `members.read`, `audit.read`.
- Workspace custom RBAC roles تحمل قائمة permissions محددة من `RBAC_PERMISSIONS`، والتحقق يتم عبر `has_workspace_permission()`.
- Project roles مستقلة: `viewer`, `editor`, `manager`. مالك المشروع له صلاحية الملكية؛ membership في workspace وحدها لا تكفي لقراءة المشروع؛ `projects.manage` يرفع effective project role إلى manager.
- `can_read_project`, `can_edit_project`, `can_manage_project` تحدد نطاق المشروع؛ أدوات الملفات تعيد فحص ACL عند التنفيذ.
- global MCP مقيد حالياً لـapplication admins؛ هذه قاعدة connector عامة وليست دوراً ينتقل إلى المنتجات الأخرى.

### 2.3 ERP Global Pro

**المصدر:** main `7bb01d23328a68aae67c659fd6c32b6db033637d`.

- الدور الفعال والصلاحيات يُحمّلان من `company_memberships.role` للعضوية النشطة، وليس من global user role وحده.
- سجل تعريفات `roles` والـpermission templates ما زال مشتركاً على مستوى التطبيق، حتى لو كان assignment للدور داخل membership؛ يجب جعله tenant-aware قبل multi-company.
- أدوار seed الحالية تشمل `admin`, `accountant`, `sales`, `warehouse`, `hr`, `viewer`، وصلاحياتها نصوص module/action مثل `sales.*`, `reports.read`, `inventory.*`؛ `admin` يملك wildcard داخل ERP فقط، لا عبر المنظومة.
- `cid` و`mid` يثبتان سياق العضوية في JWT ERP التفاعلي، ويعاد التحقق منهما على كل request.
- `TENANT_DATA_SCOPING_ENABLED=False`; العمليات التشغيلية ترفض fail-closed عندما يوجد أكثر من شركة. لا اعتماد على ERP scopes في تكامل خارجي قبل إغلاق بوابات Stage25.

## 3. عقد قرار موحد

يُقيّم أي طلب وفق الحقول التالية، سواء كان داخلياً أو بين المنتجات:

| البعد | الوصف | مصدر الحقيقة |
|---|---|---|
| `principal` | service principal أو user subject موثوق مع issuer namespace | issuer + registry لدى مالك API |
| `actor` | الخدمة التي تنفذ الطلب، وقد تعمل نيابة عن مستخدم | `azp`/`act` أو delegation record |
| `action` | فعل محدد: read/list/create/update/delete/execute/approve | API/operation registry |
| `resource` | المنتج المالك، نوع المورد، معرف opaque، ومالك المورد إن وجد | المنتج المصدر |
| `tenant_context` | workspace/project/class/company أو personal scope | المنتج الذي يملك النطاق |
| `attributes` | role/membership status/resource status/data class/environment/time/purpose | مصادر موثوقة server-side |
| `risk` | درجة أثر العملية T0–T3 | policy catalog |
| `decision` | allow/deny/approval_required/not_found | PEP لدى مالك المورد |

لا يُقبل معرف resource أو tenant من body/header كدليل تفويض. يجوز للعميل تمرير locator، لكن على الخادم إثبات الملكية والنطاق من قاعدة بيانات المنتج.

## 4. مستويات نطاق المورد

| النطاق | مثال | القاعدة |
|---|---|---|
| Public product content | قائمة موضوعات Lumen المنشورة | يمكن السماح بالقراءة دون user context إذا قرر المالك أنها عامة |
| User-private | ملف أو محادثة شخصية | المالك فقط، أو تفويض صريح يراجعه المنتج المالك |
| Class | صف Lumen | teacher owner/admin لإدارة الصف؛ الطالب عبر enrollment؛ لا تسرب بيانات زملاء الصف |
| Workspace | Workspace AI Chat | membership + workspace permission؛ لا يكفي امتلاك user token فقط |
| Project | Workspace Project | owner أو ProjectMember ACL أو `projects.manage`؛ لا ترث صلاحية Workspace تلقائياً دون سياسة المنتج |
| Company | ERP records | active membership + company context + resource permission؛ لا تُفتح عمليات multi-company حالياً |
| Platform-admin | إعدادات النظام/الموصلات العامة | صلاحية محلية للتطبيق؛ لا تمتد عبر المنتجات |

## 5. خوارزمية قرار مقترحة

1. **Authenticate:** تحقق من توقيع الاعتماد، issuer، audience، expiry/nbf، active client/key، وenvironment.
2. **Resolve actor:** حدد service principal ونطاق التفويض إن وجد؛ افصل actor عن delegated subject.
3. **Resolve tenant:** اشتق workspace/project/class/company من السجل المالك أو context موثوق؛ لا تثق بقيمة يرسلها المستهلك.
4. **Resolve membership:** أعد التحقق من user membership/status/role في المنتج المصدر في كل طلب حساس.
5. **Check service scope:** هل consumer مسموح له بهذه العملية على هذا النوع من المورد؟
6. **Check user rights:** هل delegated user يملك الفعل نفسه ضمن المنتج المصدر؟
7. **Check resource ACL:** هل المستخدم أو الخدمة يملك حق الوصول إلى هذا المورد بعينه؟
8. **Check attributes/purpose:** تصنيف البيانات، غرض الاستخدام، البيئة، حالة الحساب/الشركة، قيود retention، والوقت عند الحاجة.
9. **Check risk/approval:** اطلب موافقة إذا كان الفعل خارج القراءة منخفضة المخاطر.
10. **Audit and decide:** سجّل القرار الأدنى اللازم ثم allow/deny/approval_required. لا تسجل token أو محتوى المورد.

## 6. مستويات المخاطر واشتراط الموافقة

| المستوى | أمثلة | القاعدة المقترحة |
|---|---|---|
| T0 — public/read-only | قراءة دليل موضوعات عام أو وثائق عامة | لا تعديل؛ يظل التحقق من audience/consumer مطلوباً عند API خارجي |
| T1 — scoped private read | قراءة mastery طالب واحد أو ملف خاص مفوض | owner/delegation + resource ACL + data-purpose check |
| T2 — reversible write | إنشاء مسودة أو تحديث إعداد محدود | scope ضيق، idempotency، audit، وموافقة حسب المنتج/السياق |
| T3 — high impact | ترحيل مالي، حذف واسع، production change، إدارة عضويات/أسرار | deny by default؛ موافقة بشرية صريحة وstep-up authorization وسجل تدقيق؛ لا ينفذه AI agent منفرداً |

تتبع هذه المستويات تصميم `AI_CORE_AGENT_TOOL_MATRIX.md`؛ لا تفعّل T2/T3 بمجرد توثيق المستوى. يحدد مالك المنتج متى تكون الموافقة مطلوبة.

## 7. قواعد التفويض بين الخدمات

- service scope يحدد السقف الأعلى لما يمكن للخدمة طلبه؛ delegated user rights تضيق السقف ولا توسّعه.
- service token بلا delegated subject يصلح فقط لعمليات machine-owned صريحة ومراجعة، ولا ينشئ هوية مستخدم بديلة.
- delegated subject يجب أن يكون namespaced بـissuer/subject؛ لا تُنسخ أرقام user_id بين قواعد البيانات.
- عند غياب subject mapping أو موافقة/غرض مطلوب، يكون القرار deny.
- لا impersonation ولا on-behalf-of claims يرسلها العميل بلا توقيع من issuer موثوق.
- لا تعمم صلاحية admin أو wildcard أو workspace owner عبر التطبيقات.
- عند revoke membership/role/share، تؤثر الصلاحية في الطلب التالي؛ لا تعتمد على grants مخزنة طويلة العمر بلا revalidation.

## 8. سلوك الرفض وإخفاء الموارد

- `401`: principal/token غير صالح أو منتهي أو ملغى.
- `403`: الهوية معروفة والعملية ممنوعة ضمن نطاق معلوم.
- `404`: يستخدمه المنتج المالك عندما تكون سياسة عدم كشف وجود المورد المحجوب مطلوبة.
- `409`: سياق tenant مفقود/ملتبس أو انتقال حالة يمنع العملية.
- `approval_required`: قرار داخلي/واجهة تدفق يوقف الفعل حتى موافقة حقيقية؛ لا يحوّل إلى allow تلقائي عند timeout.
- `429`: rate/quota boundary مع Retry-After إذا توفر.
- `503`: verifier أو policy dependency غير متاح في بيئة production؛ fail closed.

يجب أن يتبع الرد semantics المنتج المالك؛ لا تفرض envelope أو status code موحداً على واجهة قائمة دون اختبار التوافق.

## 9. سجل القرار والتدقيق

الحد الأدنى المقترح لكل قرار حساس:
- `request_id` و`correlation_id`.
- `consumer_client_id` وissuer/audience بعد التحقق.
- namespaced subject أو delegation reference دون PII غير لازمة.
- owner product وresource type وopaque resource reference.
- action/scope وtenant boundary type دون نسخ محتوى السجل.
- decision/reason code/risk tier/approval reference.
- policy version وtime وenvironment.

لا تسجل bearer tokens، API keys، كلمات المرور، نصوص prompts، درجات الطلاب، المستندات أو بيانات ERP المالية في سجل التفويض المركزي.

## 10. مصفوفة قبول سلبية إلزامية

| الاختبار | النتيجة المطلوبة |
|---|---|
| Lumen teacher token إلى AI Chat workspace API | deny ما لم يوجد service identity وdelegation صالحان منفصلان |
| AI Chat personal API key إلى ERP | deny كاعتماد خدمة؛ المفتاح لا يمثل service principal |
| ERP user JWT إلى Lumen | deny كهوية عبر منتج؛ JWT لا يُعاد تفسيره خارج ERP |
| تبديل user_id أو company_id أو workspace_id في body | لا يغيّر principal أو النطاق؛ server-side lookup يحسم |
| نفس الرقم في قاعدتين مختلفتين | لا تطابق تلقائي بسبب اختلاف issuer/namespace |
| عضو workspace بلا project membership | لا وصول project إلا وفق ACL صريح أو projects.manage |
| Lumen student خارج enrollment | لا بيانات صف أو تقدم طالب آخر |
| ERP membership revoked أو suspended company | deny في الطلب التالي |
| service scope صحيح مع user permission مفقود | deny |
| user permission صحيح لكن service audience خطأ | deny |
| T3 action دون approval reference | approval_required/deny، لا تنفيذ |
| verifier/policy service غير متاح في production | fail closed |

## 11. ما لا ننفذه في هذه المرحلة

- لا policy engine مركزي أو SSO أو global admin role.
- لا دمج RBAC catalogs بين المنتجات أو نقل user/company IDs.
- لا token issuer أو signing keys أو service principals فعلية.
- لا تجاوز ERP `TENANT_DATA_SCOPING_ENABLED=False`.
- لا منح AI Core وصولاً مباشراً إلى قواعد Lumen أو AI Chat App أو ERP.
- لا تفعيل أدوات T2/T3 أو MCP خارجي عبر منتجات أخرى.

## 12. بوابات إتمام S5-04

1. موافقة مالكي المنتجات على mapping المحلي للـroles/resources/actions.
2. ربط service/delegation contract بـS5-03 وAPI contract IDs في S5-02.
3. توثيق ABAC attributes ومصادر موثوقيتها لكل scope.
4. PEP داخل كل منتج مالك مع revalidation/revocation.
5. contract tests وcross-tenant/IDOR negative tests في المستودعات المالكة.
6. اختبار disposable/staging مع replay/expiry/key rotation ومراجعة بشرية للأفعال الحساسة.

**الخلاصة:** نموذج التفويض الموحد هو تنسيق للمعايير وتبادل القرار، وليس سلطة مركزية تتجاوز تطبيقات المنتجات. كل منتج يقرر حق الوصول إلى موارده، مع إبقاء service identity وuser delegation وresource ownership منفصلة.