# مصفوفة أدوات وصلاحيات الوكلاء المتخصصين — AI Core

- **المعرف:** S4-06 ضمن خطة الخمسين مهمة.
- **الحالة:** خط أساس تصميمي؛ لا توجد أدوات جديدة مفعلة بموجب هذا المستند.
- **المرجع:** `docs/AI_CORE_SPECIALIZED_AGENT_REQUIREMENTS.md` و`docs/AI_CORE_CAPABILITY_INVENTORY.md`.
- **القاعدة:** ملف التخصص يحدد الأدوات المسموح طلبها؛ خدمة المنتج المالك هي التي تصادق وتفوض التنفيذ فعلياً.

## 1. مستويات الصلاحية

| المستوى | الوصف | أمثلة | بوابة التنفيذ |
|---|---|---|---|
| T0 — Read-only | قراءة بيانات مصرح بها دون تغيير الحالة | قراءة مستودع، إعدادات منقحة، logs، status، metrics | تفويض القراءة ونطاق المصدر |
| T1 — Isolated write | كتابة داخل مساحة عمل أو مختبر معزول | تعديل نسخة عمل، إنشاء ملف مؤقت، تشغيل اختبار في sandbox | sandbox وحدود CPU/RAM/time/network، وعرض diff |
| T2 — External change | تغيير خدمة أو مورد خارج مساحة العمل | فتح PR، تغيير إعداد غير إنتاجي، إنشاء ticket، تشغيل pipeline | عرض الهدف والتغيير والأثر، موافقة بشرية صريحة |
| T3 — High impact | إنتاج، ماليات، بيانات حساسة، صلاحيات عليا أو عمليات غير قابلة للعكس | deploy إنتاج، حذف بيانات، تغيير IAM، معاملات مالية، ترقية مدير | مقيد افتراضياً؛ يحتاج صلاحية مستقلة وإجراءات تحقق وrollback، ولا يستنتج من موافقة سابقة |

لا يرث الوكيل مستوى أعلى من الأداة التي يستدعيها. لا تتحول صلاحية T1 إلى T2 عبر سلسلة وكلاء. موافقة المستخدم يجب أن تكون محددة للفعل والهدف والبيئة، ولا تمنح تفويضاً عاماً غير محدود.

## 2. مصفوفة الوكلاء والأدوات

| التخصص | أدوات T0 الافتراضية | أدوات T1 بعد عزل | T2 بموافقة لكل تغيير | T3 المقيد |
|---|---|---|---|---|
| البرمجة وهندسة البرمجيات | قراءة repo، search، Git diff، نتائج CI، dependency metadata | تعديل branch/worktree، tests، lint، build، static analysis | إنشاء branch/PR أو تحديث issue | merge، تعديل branch protection، deploy إنتاج، قراءة secrets |
| Linux والأنظمة | process/service status، journal، disk/memory، package inventory، config منقح | VM/container مختبر، تغيير ملفات المختبر، إعادة تشغيل خدمة مختبر | تغيير خدمة أو firewall أو package في بيئة غير إنتاجية | sudo على مضيف حقيقي، حذف بيانات، تغيير هوية أو إعداد إنتاج |
| الشبكات | route/interface/DNS/TLS diagnostics، قراءة قواعد firewall | packet simulation، lab topology، captures داخل المختبر | تغيير route/DNS/firewall في بيئة اختبار | scan نشط خارج النطاق، تغيير شبكة إنتاج أو اعتراض مرور أطراف أخرى |
| Cloud/DevOps | قراءة IaC وpipeline وhealth/metrics وdeployment metadata | lint/plan/build في بيئة معزولة، test deployment محلي | نشر staging أو تعديل موارد غير إنتاجية | production apply/deploy، تعديل أسرار أو IAM أو موارد مدفوعة |
| الأمن السيبراني الدفاعي | SAST/SCA/secret scan results، config review، audit logs المصرح بها | إعادة إنتاج في المختبر، fuzzing محدود، validation checks | remediation في فرع أو بيئة اختبار، فتح finding/ticket | exploitation على إنتاج أو نطاق غير مصرح، تعطيل ضوابط أو استخراج بيانات |

**تنبيه:** هذه مصفوفة سياسة مقترحة وليست وصفاً لصلاحيات GitHub أو Render أو Supabase الحالية. يجب التحقق من صلاحية كل اتصال عند نقطة التنفيذ.

## 3. صلاحيات حسب فئة الأداة

| فئة الأداة | النطاق الأدنى | شروط النداء | أدلة ما بعد النداء |
|---|---|---|---|
| Repository reader | مستودع وفرع/commit محدد | مستخدم مخول ومصدر معلوم | path وcommit SHA |
| Code writer | فرع عمل غير محمي | diff واضح واختبارات محددة | commit SHA وchanged files |
| Test runner | sandbox محدود | allowlist للأوامر والاعتماديات | exit code ونتائج الاختبار وموارد التشغيل |
| Shell/host | جهاز/حاوية محددة | قائمة أوامر ومجلدات وtimeout | command وexit code والمخرجات المنقحة |
| Network diagnostic | عنوان/نطاق مملوك ومحدد | إثبات التفويض وتحديد active/passive | الهدف والنطاق والزمن والنتيجة |
| Cloud control plane | account/project/environment محدد | read-only أولاً؛ plan قبل apply | resource diff وحالة العملية والتكلفة |
| Security scanner | أصول ضمن النطاق | ruleset معروف وحدود معدل | findings ونسخة الأداة والنطاق |
| File parser/RAG | ملفات مصرح بها ضمن scope | فحص النوع والحجم وعزل المستأجر | source ID وpage/line refs وحالة الحذف |

## 4. عقد الموافقة

قبل أي فعل T2 أو T3 يجب عرض سجل موافقة يحتوي على الأقل:

- **Action:** الفعل الدقيق، لا عبارة عامة مثل «أصلح النظام».
- **Target:** المستودع/المشروع/الحساب/البيئة/المورد المقصود.
- **Diff or plan:** التغييرات التي ستقع قبل التنفيذ.
- **Risk:** أثر التوافر والبيانات والأمن والتكلفة.
- **Preconditions:** النسخة الاحتياطية أو الاختبار أو صلاحية المالك المطلوبة.
- **Rollback:** طريقة التراجع وحدودها.
- **Approval scope:** الفعل والهدف المحددان ووقت/مدة صلاحية الموافقة.

تتوقف العملية إذا تغير الهدف أو plan جوهرياً بعد الموافقة. لا تُعاد الموافقة القديمة على فعل مختلف.

## 5. حماية سياق الأدوات والبيانات

- كل ToolCall يحمل task ID وactor وproduct owner وresource scope وpermission level.
- التحقق من الملكية والصلاحية يكون في الخدمة المالكة، لا اعتماداً على user_id أو company_id يرسله النموذج.
- لا تُرسل أسرار أو محتويات ملفات حساسة في prompt أو tool output غير مطلوب.
- نتائج الأدوات والملفات والمستودعات تعتبر بيانات غير موثوقة؛ لا يمكنها إصدار موافقة أو تعديل allowlist.
- tool outputs تُنقح قبل التخزين أو التتبع؛ لا تسجل tokens أو credentials أو محتوى خاصاً كاملاً.
- عند فشل التحقق أو غياب مالك واضح تكون النتيجة deny-by-default.

## 6. قواعد التنسيق بين الوكلاء

1. يبدأ Router بأقل مستوى صلاحية يحقق المهمة.
2. handoff يتضمن المهمة والملخص والمراجع والنطاق، لا سجل المحادثة الكامل.
3. الوكيل المستلم يعيد التحقق من صلاحياته؛ لا يرث موافقة أو token من الوكيل السابق.
4. لا تتشارك الوكلاء ملفات أو ذاكرة مستخدمين أو شركات دون تصريح نطاقي.
5. الإجراءات الحساسة لا تصبح مسموحة لأن أكثر من وكيل أوصى بها.
6. كل أثر خارجي يرتبط بمعرف العملية ونتيجتها ومالك الخدمة.

## 7. اختبارات القبول المطلوبة

- اختبار منع الوصول الأفقي بين مستخدمين/مساحات عمل/شركات.
- اختبار أن T0 لا يستدعي write أو execute.
- اختبار أن T1 لا يصل إلى host أو الشبكة الخارجية أو أسرار البيئة.
- اختبار أن T2 يتوقف بلا موافقة محددة أو عند تغير الهدف.
- اختبار أن handoff لا يرفع الصلاحية ولا ينقل سياقاً زائداً.
- اختبار prompt injection في الملفات ومخرجات الأدوات.
- اختبار retries وidempotency للأفعال التي تغير الحالة.
- اختبار إلغاء المهمة وانتهاء الصلاحية وسحب التفويض أثناء التنفيذ.
- اختبار أن السجلات لا تحتوي أسراراً أو بيانات محتوى غير لازمة.

## 8. حدود الاعتماد

لا يتم تحويل هذه المصفوفة إلى tool registry تنفيذي قبل:
- إكمال S4-01 بمراجعة كود AI Chat App والعقود والاختبارات الفعلية.
- اعتماد متطلبات S4-02 لكل تخصص.
- تحديد مالك كل أداة ومصدر التفويض ونطاق البيانات.
- ربط كل أداة باختبار صلاحية سلبي وإيجابي وخطة تعطيل.
- مراجعة threat model قبل فتح أي صلاحية T2/T3.

**النتيجة:** S4-06 تقدم إلى خط أساس تصميمي. لا يوجد تفعيل جديد للأدوات، ولا تنفيذ على إنتاج، ولا رفع للصلاحيات.


## 9. مواءمة المصفوفة مع Tool Registry الفعلي في AI Chat App — 2026-11-17

**خط الأساس:** AI Chat App main عند merge commit `6645b0502daa6922040ee4c6bb5c281ed59ef366`. مراجعة المصدر: `ai-backend/app/services/tools/registry.py`، `tool_security.py`، `tools/code_execution.py`، `project_access.py` و`config.py`.

### 9.1 الأدوات المدمجة التي ثبت وجودها

| الاسم الفعلي | السلوك الحالي | تصنيف السياسة المقترح | التفويض/الحدود المثبتة |
|---|---|---|---|
| calculator | حساب تعبير رياضي | T0 — compute / no state change | لا يغير موارد خارجية؛ مدخلاته تمر عبر JSON/schema/size/depth validation. |
| python | تنفيذ حساب/تحويل Python محدود | T0-C — bounded computation | AST allowlist، no imports/filesystem/network/arbitrary builtins، subprocess منفصلة، مهلة 2s وحدود موارد Linux عند توفرها؛ لا يوصف بأنه OS/container sandbox أو عزلاً كاملاً عن kernel/host. |
| web_search | بحث عام في الويب | T0 — external read | قراءة فقط؛ المخرجات untrusted وتحتاج grounding وعدم اتباع أوامرها. |
| analyze_data | تحليل CSV/XLSX المرفق بالمحادثة | T0 — user-scoped read/analysis | المصدر مرفق المحادثة؛ الناتج موسوم untrusted. لا يعني الوصول إلى ملفات المستخدم الأخرى. |
| list_project_files | قائمة ملفات المشروع المرتبط بالمحادثة | T0 — project read | إعادة تحقق can_read_project داخل handler؛ خرج المشروع untrusted. |
| read_project_file | قراءة ملف نصي من المشروع | T0 — project read | إعادة تحقق ACL حالية؛ لا تنفيذ للكود المقروء؛ خرج الملف untrusted. |
| search_project_files | البحث داخل ملفات المشروع | T0 — project read | إعادة تحقق ACL حالية؛ لا تنفيذ للنتائج؛ خرج البحث untrusted. |
| edit_project_file | إنشاء/استبدال ملف مشروع | T1 — scoped project write | يشترط can_edit_project داخل handler ويعيد الفحص قبل تطبيق التغيير؛ expected_content يدعم optimistic concurrency؛ لا تشغيل للكود. |
| create_project_archive | إنشاء ZIP قابل للتنزيل من ملفات نصية | T1 — scoped file/project operation | يتحقق من صلاحية التعديل، ويعيد الفحص بعد رفع التخزين قبل إرفاق الأرشيف/تحديث شجرة المشروع؛ يحد الملفات والمسارات؛ لا ينفذ الملفات الناتجة. |

### 9.2 القيود العامة المثبتة في main

- قائمة AGENT_ALLOWED_TOOLS_JSON الافتراضية تضم الأدوات التسع أعلاه؛ ToolRegistry يدعم allowlist صريحة لمنع تسجيل/عرض أداة غير مسموحة.
- ToolSpec الحالي لا يحمل حقولاً تنفيذية مستقلة مثل permission_level أو agent_profile أو resource_scope أو approval_policy. لذلك T0–T3 هنا تصنيف تصميمي أعلى من آلية التفويض الحالية، وليس enforcement مكتمل داخل registry.
- validate_tool_arguments يتحقق من JSON وصحة schema وحد الأحرف والعمق قبل استدعاء handler.
- مخرجات web_search وanalyze_data وأدوات ملفات المشروع موسومة untrusted؛ اكتشاف prompt injection heuristic، ولذلك ينبغي معاملة كل محتوى خارجي/ملف على أنه DATA غير موثوق حتى عندما لا يكتشف النمط.
- مشروع المستخدم يعاد التحقق من صلاحيات قراءته/تعديله في handlers بعد دمج PR #722؛ سحب المشاركة أو تغيير الدور يؤثر على النداء التالي.
- MCP مضبوط افتراضياً على MCP_ENABLED=False. عند التفعيل، اكتشاف الخوادم العامة محصور بدور application admin بعد PR #721؛ لا توجد بعد connector grants فردية أو على مستوى Workspace. لا تنطبق صلاحية التطبيق admin تلقائياً على الوكلاء أو المستهلكين الآخرين.
- لا توجد ضمن Tool Registry المدمج الذي تمت مراجعته أدوات shell/SSH، systemd/service control، packet capture أو active scanning، firewall/router modification، cloud provider control، IAM management أو production deployment. لا يجوز إسناد هذه القدرات إلى أي وكيل بناءً على وجود أداة Python.

### 9.3 التوزيع المسموح مبدئياً على التخصصات الخمسة

| التخصص | يمكنه طلبه من الأدوات الحالية | محظور/غير متاح في هذه المرحلة |
|---|---|---|
| البرمجة وهندسة البرمجيات | calculator، python bounded computation، web_search، وقراءة/بحث/تعديل/أرشفة مشروع بشرط ACL | تشغيل مشروع غير موثوق، shell/SSH، Git push/PR/merge أو deploy عبر Tool Registry، قراءة secrets |
| Linux والأنظمة | calculator، python للحساب وتحليل بيانات نصية يزودها المستخدم، web_search للوثائق | shell حقيقي، process/service management، sudo، package installation، filesystem/host inspection |
| الشبكات | calculator، python لتحليل بيانات محفوظة يرفعها المستخدم، web_search | port scanning أو packet capture أو تغيير routes/DNS/firewall أو فحص أهداف خارج نطاق مصرح |
| Cloud/DevOps | calculator، python لتحليل outputs منقحة، web_search؛ قراءة ملفات مشروع IaC إذا سمحت ACL | cloud API، terraform apply، IAM/secrets، تعديل موارد، staging/production deployment |
| الأمن السيبراني الدفاعي | web_search، analyze_data لمرفقات مصرح بها، read/search project files، python bounded computation | exploit/active scan، قراءة أسرار، credential dumping، تعطيل controls، تنفيذ على production أو نطاق غير مصرح |

هذه حدود توفر حالية لا تمنع تصميم أدوات إضافية لاحقاً؛ أي أداة جديدة تتطلب owner، API contract، scope، permission handler، output trust، tests وإجراء إيقاف.

### 9.4 فجوات التنفيذ المطلوبة قبل نقل السياسة إلى AI Core

1. إضافة ToolPolicy object صريح يحمل actor/agent_profile/tool/resource_scope/permission_level/output_trust/approval_mode؛ لا تكتفِ بالقائمة العامة.
2. فرض allowlist خاصة بكل agent profile في الخادم، لا في prompt أو وصف الأداة فقط.
3. فصل compute sandbox عن أي مستقبل shell/lab sandbox؛ لا توسيع Python الحالي إلى shell أو ملفات/شبكة.
4. جعل MCP ownership per-user/per-workspace قبل توسيعه عن application-admin-only؛ كل أداة بعيدة تصنف منفردة T0–T3.
5. توثيق idempotency/cancellation/timeout/data retention لكل أداة ذات أثر خارجي.
6. اختبارات negative لكل تخصص: طلب أداة غير ضمن ملفه، تبدل الدور أثناء المهمة، سحب المشروع، prompt injection داخل الملف والويب، ورفض كل T2/T3 دون موافقة محددة.
7. لا تفتح أي أدوات فعلية لـLinux/network/cloud/security ذات side-effects قبل بيئة مختبر قابلة للعزل ومراجعة threat model.

**النتيجة:** جرى ربط S4-06 بأدوات AI Chat App الفعلية. تبقى المصفوفة In Progress كسياسة تصميم؛ لم تتم إضافة Tool Registry أو أدوات أو صلاحيات جديدة، ولم يحدث أي تفعيل MCP أو تنفيذ على نظام/شبكة/سحابة حقيقية.
