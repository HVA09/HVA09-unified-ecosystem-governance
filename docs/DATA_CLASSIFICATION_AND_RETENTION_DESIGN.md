# تصميم تصنيف البيانات والتدفق والاحتفاظ — المنظومة المتكاملة

- **المعرف:** S5-05 ضمن خارطة المراحل الخمس.
- **الحالة:** Design baseline — لا سياسة مركزية مطبقة على المنتجات بعد.
- **التاريخ:** 2026-11-17.
- **القاعدة:** المنتج الذي يملك البيانات يصنفها ويحدد غرض استخدامها والاحتفاظ بها؛ لا يحق لـAI Core أو Gateway تخفيف التصنيف أو توسيع جمهور البيانات.

## 1. الهدف

تحديد آلية مشتركة لتصنيف البيانات قبل انتقالها إلى AI Core أو موصل أو مزود خارجي، مع احترام استقلال قواعد المنتجات وهوياتها وسياسات الحذف والاحتفاظ.

يركز التصميم على:
- أقل قدر لازم من البيانات لغرض واحد محدد.
- تصنيف يرافق البيانات عند النقل والمخرجات والمشتقات.
- التحقق من تفويض المورد قبل استخراجه وقبل تنفيذ أي فعل لاحق.
- منع تسرب محتوى prompts/files/grades/financial rows إلى logs أو telemetry.
- الحذف المتسلسل للنسخ والمرفقات والفهارس وembeddings والcache عندما يفرضه مالك المصدر.
- fail-closed عند غياب تصنيف أو غرض أو سياسة مزود موثقة.

## 2. مبادئ الملكية

1. Lumen يملك بيانات الحساب التعليمي والصفوف والمقررات والتقدم والواجبات والدرجات وملفات الطالب.
2. AI Chat App يملك المستخدمين والمحادثات والرسائل والذاكرة والملفات والمشاريع ومساحات العمل وأدواتها.
3. ERP يملك الشركات والعضويات والمالية والمخزون والموارد والوثائق؛ ولا يفتح بيانات multi-company حتى اجتياز release gates.
4. AI Core لا يملك نسخة أعمال من هذه البيانات؛ يتعامل مع payload مؤقت ومصرح به عبر عقد.
5. Unified Ecosystem يحتفظ ببيانات الحوكمة ومراجع العقود وحالات الفحص، ولا يخزن محتوى المستخدم أو الملف أو prompt.
6. أي snapshot أو logs مركزية تخزن الحد الأدنى من metadata وتخضع لسياسة منفصلة.

## 3. مستويات التصنيف المقترحة

| التصنيف | أمثلة | قواعد الاستخدام |
|---|---|---|
| Public | صفحات عامة أو مواد منشورة صراحة للعامة | يمكن إرسالها إلى نموذج approved وفق شروط الاستخدام وسياسة المصدر |
| Internal | إعدادات تشغيل غير سرية، مخططات عامة، محتوى داخلي غير شخصي | يسمح فقط لمزود/نموذج معتمد لهذا التصنيف؛ لا يدرج secrets |
| Confidential | محادثات خاصة، ملفات شخصية، معرفات حساب، مواد عمل غير عامة | لا خروج إلى مزود خارجي إلا بعد موافقة مالك البيانات وتوثيق processor/retention/region |
| Restricted | درجات ومعلومات تعلم فردية، سجلات مالية/رواتب/ضرائب، ملفات حساسة أو بيانات شخصية عالية الأثر | افتراضياً لا تُرسل إلى مزود خارجي؛ تحتاج معالجة محلية معتمدة أو تفويض قانوني/تعاقدي وموافقة المالك |
| Secret | كلمات مرور، access/refresh tokens، private keys، provider API keys، recovery codes | يمنع إدخالها إلى prompt أو RAG أو tool output أو telemetry؛ ليست مادة تحليل للنموذج |
| Unknown | بيانات لا تحمل classification أو مصدرها/غرضها مجهول | تعامل كـRestricted وارفض النقل حتى يحدد مالكها التصنيف والغرض |

هذه مستويات حوكمة مقترحة وليست بديلاً عن تقييم قانوني أو تعاقدي بحسب البلد والقطاع والعميل. لا يعلن المستودع امتثالاً تنظيمياً أو قانونياً بمجرد وجود هذه الوثيقة.

## 4. خريطة أولية للبيانات

| نطاق المنتج | مجموعات البيانات | التصنيف الافتراضي المقترح | مالك القرار |
|---|---|---|---|
| Lumen | هوية الطالب، enrollment، mastery، submissions، grades | Restricted | Lumen |
| Lumen | محتوى منشور للعامة أو مادة تعليمية مرخصة للنشر | Public أو Internal بحسب المصدر | Lumen |
| Lumen | ملفات المستخدم أو المدرس غير المنشورة | Confidential، وترتفع إلى Restricted حسب المحتوى | Lumen |
| AI Chat App | الرسائل والمحادثات والذاكرة الشخصية | Confidential | AI Chat App |
| AI Chat App | ملفات المشاريع وRAG chunks/embeddings | ترث أعلى تصنيف للمصدر، والافتراضي Confidential | AI Chat App |
| AI Chat App | MCP/OAuth/API credentials وsecrets | Secret | AI Chat App |
| ERP | فواتير وقيود محاسبية ورواتب ومخزون وبيانات الشريك/العميل | Restricted أو Confidential وفق الحقل | ERP |
| ERP | ملفات الشركة والعقود والتقارير المصدرة | Confidential أو Restricted وفق المحتوى | ERP |
| Unified Ecosystem | IDs opaque وروابط PR/commit وحالات CI | Internal | الحوكمة |
| Unified Ecosystem | prompts أو نصوص ملفات أو بيانات حساب حقيقية | ممنوع تخزينها مركزياً افتراضياً | مالك المصدر |

التصنيف يجب أن يكون على مستوى المورد/الحقل عند الحاجة؛ لا يكفي تصنيف المنتج كله بمستوى واحد. ويجب أن يكون لكل انتقال purpose محدد.

## 5. عقد نقل البيانات

كل طلب إلى AI Core/Gateway أو API آخر يجب أن يرفق metadata موثوقة:
- `source_product` و`source_resource_type` وopaque resource reference.
- `data_classification` بعد تحديدها من المالك.
- `purpose_code` ووصف مهمة محدود لا يحتوي معلومات أكثر من اللازم.
- `consumer_id` و`audience` وscopes من اعتماد موثوق.
- `retention_policy_id` ومدة صلاحية السياق/النتيجة.
- `correlation_id` دون كتابة محتوى حساس.
- `consent_or_legal_basis_ref` عند طلب مالك البيانات ذلك.

لا يحدد المستهلك التصنيف بنفسه في body لتخفيفه؛ يمكنه طلب تصنيف أكثر تشدداً فقط. لا تقبل Gateway رفع مستوى الثقة أو تغيير `source_product` من قيمة يرسلها المستخدم.

## 6. سياسة النماذج والمزودين

لكل provider/model alias يجب أن يحدد owner:
- أقصى تصنيف يمكن معالجته.
- هل تُرسل البيانات إلى خدمة خارجية أو local/on-prem.
- retention الفعلي لدى المزود وشروط استخدامها/التدريب إن كانت موثقة.
- region/residency عند وجود إثبات.
- قدرات الحذف والـlogging والتحكم في abuse monitoring حيثما تتيح الخدمة ذلك.
- modalities المسموحة (text/image/audio) وحد payload.
- هل يسمح بالـtools أو structured output.

السياسة:
- Unknown/Secret: deny.
- Restricted: deny افتراضياً على المزود الخارجي حتى approval محدد أو مسار local معتمد.
- Confidential: لا خروج خارجي دون اعتماد processor/retention/region وعقد مالك المصدر.
- Internal/Public: لا يعني السماح التلقائي؛ يبقى model allowlist وconsumer scope مطلوبين.
- لا يُستنتج عدم تدريب المزود أو موقع التخزين أو مدة الاحتفاظ من اسم المنتج؛ يلزم دليل تعاقدي/وثائقي من مالك المزود.

## 7. الاحتفاظ والحذف ودورة حياة RAG

يجب تعريف lifecycle لكل مصدر:
1. Source object: الملف أو المورد الأصلي لدى المنتج.
2. Derived content: extracted text، chunks، OCR/metadata، embeddings.
3. Cache: prompt cache، tool cache، retrieval cache، response cache إن وجدت.
4. Model/provider copies: البيانات التي قد يحتفظ بها المزود وفق عقده الفعلي.
5. Audit/telemetry: metadata اللازمة فقط مع retention قصير ومحدد.
6. Export/backup: يخضع لسياسة backup/restore منفصلة ولا يتجاوز قرار الحذف النهائي بلا مبرر.

عند الحذف/سحب المشاركة:
- يمنع استرجاع المورد فوراً عبر access check، حتى لو لم يكتمل الحذف الفيزيائي.
- يسجل deletion event داخلياً ويزيل chunks/embeddings/cache المشتقة وفق policy.
- يحترم retention legal hold إن وجد بقرار owner موثق.
- لا يغير نطاق المورد من project إلى workspace تلقائياً دون اختيار واضح من المالك وصاحب الصلاحية وإبلاغ المستخدم.
- Issue #724 في AI Chat App توضح أن حذف مشروع يضع `project_id=NULL` مع بقاء `workspace_id` فيحوّل بعض الملفات من Project scope إلى Workspace scope. يجب حسم private/archive/delete/promote policy واختبارها قبل قبول دورة حياة RAG كمكوّن عابر للمنتجات.
- إعادة فهرسة الملفات أو نقلها لا تعني تغيير جمهور الوصول؛ ACL تسبق retrieval دائماً.

## 8. الخصوصية في prompts والأدوات والـoutputs

- تنقيح أو حذف الحقول غير اللازمة قبل إرسالها للنموذج.
- لا تمرر password/token/provider secret/cookies/private key إلى النموذج أو tool result.
- كل مخرجات Provider وMCP وWeb search والملفات تعامل كـuntrusted input؛ لا تمنحها تعليمات نظام أو scopes أو صلاحيات تنفيذ.
- لا ينفذ Agent tool call قبل تحقق policy/ACL/approval في المنتج المالك.
- المخرجات ترث أعلى تصنيف من inputs افتراضياً، ولا تصبح Public لمجرد أن النموذج أعاد صياغتها.
- يتطلب نشر نتيجة أو مشاركتها تصنيفاً ومراجعة صريحة وفق سياسة المنتج.

## 9. السجلات والمراقبة

المسموح افتراضياً:
- request_id/correlation_id، product/consumer، operation، model alias/provider reference، latency، token counts، response status، error class، classification label، retention ID.
- counts أو digests غير قابلة للرجوع للمحتوى، إذا لم تسمح باستخلاص البيانات الحساسة.

الممنوع افتراضياً:
- prompt/response كامل، نص ملف أو درجة طالب أو سجل مالي، bearer token أو provider secret، raw lineitem/OAuth assertion، private URL أو signed URL، أسماء أو emails عندما لا تكون ضرورية.
- نسخ محتوى المحادثة إلى logs مركزية بغرض debugging دون تصريح وعملية redaction.

إتاحة debug payload تكون مؤقتة ومقيدة بصلاحيات عالية وموافقة owner، مع سجل تدقيق وانتهاء تلقائي.

## 10. اختبارات القبول

- Unknown/Secret classification يرفض إرسال البيانات.
- Restricted data لا يذهب إلى external provider دون policy approval.
- Model alias غير مسموح لتصنيف/consumer معيّن يرجع deny قبل الاتصال.
- prompt/response/file content لا يظهر في logs أو telemetry.
- cross-product user/company/workspace IDs لا تُعامل كهوية أو تفويض.
- تغيير role أو سحب membership/share يمنع retrieval التالي.
- project/file delete يزيل الوصول ويعالج chunks/cache ولا يوسع scope بصمت.
- ملفات Lumen والطلاب والدرجات لا تتسرب عبر RAG/AI Chat إلى مستهلك آخر.
- ERP company A لا يمكن أن تقرأ بيانات company B أو ترسلها إلى AI service، وتظل عمليات multi-company blocked حتى gates.
- backups/export/restore تحافظ على التصنيف والحدود والسياسات.
- مخرجات الأدوات غير الموثوقة لا تتجاوز صلاحيات النظام.

## 11. مراحل الاعتماد

1. تعيين data owner وتصنيف أولي لكل resource family في كل منتج.
2. إكمال data-flow maps وتحديد purpose/consumer/provider لكل انتقال.
3. مراجعة retention/region/processor من أصحاب المنتجات والمزودين.
4. اعتماد Model Registry policy وربطها بتصنيف البيانات.
5. اختبار deny/allow وdeletion/retention في بيئات disposable.
6. تجربة انتقال بيانات دنيا مع مستهلك واحد وrollback.
7. توسيع الاستخدام فقط بعد موافقات مالكي المنتجات.

## 12. القرارات غير المحسومة

- لا يوجد حالياً معيار مركزي قابل للتطبيق آلياً للـclassification labels.
- لا توجد service principals أو contracts بين المنتجات.
- مدة retention لدى كل external provider لم تُثبت في هذا الجرد.
- لم يُحسم أسلوب التشفير/إقامة البيانات لكل قطاع أو بلد.
- لم تُعتمد سياسة معالجة بيانات Lumen الحساسة عبر نماذج خارجية.
- ERP multi-company محجوب، ولا تُنقل بياناته إلى AI Core.
- AI Chat App Issue #724 تحتاج قراراً واضحاً حول سلوك حذف المشروع واتساع scope.

## 13. المراجع

- `AI_CORE_CAPABILITY_INVENTORY.md`
- `AI_CORE_MODEL_GATEWAY_DESIGN.md`
- `PRODUCT_OWNERSHIP_AND_INTERFACE_CATALOG.md`
- `PRODUCT_API_CONTRACT_CATALOG.md`
- `SERVICE_IDENTITY_AND_DELEGATION_DESIGN.md`
- AI Chat App Issue #724
- DEC-005 وDEC-008 وDEC-010 وDEC-015

**الخلاصة:** كل منتج يظل مالك بياناته. لا تنتقل بيانات إلى AI Core أو مزود خارجي دون تصنيف وغرض وتفويض وretention policy موثقة. الغياب أو الشك يعني deny، لا افتراض السماح.
