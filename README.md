# حوكمة المنظومة التقنية المتكاملة

هذا المستودع هو المرجع المركزي لخارطة موحدة من خمس مراحل أساسية: **Lumen، AI Chat App، ERP، AI Core، وUnified Ecosystem**، مع تأجيل المرحلة السادسة للابتكار والتوسع. لا يحتوي هذا المستودع على كود المنتجات أو قواعد بياناتها، ولا يحل محل وثائق كل منتج.

## ابدأ من هنا

1. [المرجع الرئيسي وخارطة البرنامج](docs/UNIFIED_ECOSYSTEM_MASTER_PLAN.md)
2. [فهرس الوثائق والنسخ المؤرشفة ومصادرها](docs/SOURCE_DOCUMENT_CATALOG.md)
3. [التعارضات المكتشفة وكيفية التعامل معها](docs/CONFLICTS_AND_RECONCILIATION.md)
4. [سجل المهام والتبعيات وحالات الإنجاز](docs/TASK_REGISTER.md)
5. [المعايير الهندسية المشتركة](docs/UNIFIED_ENGINEERING_STANDARDS.md)
6. [سجل القرارات المعتمدة](docs/DECISION_REGISTER.md)
7. [ملاحظات PRs والنقاشات الهندسية المهمة](docs/EXTERNAL_DISCUSSION_REGISTER.md)
8. [شرح الأرشيف وقواعد النسخ](archive/README.md)
9. [خارطة الطريق الموحدة — المراحل الخمس والمرحلة السادسة المؤجلة](docs/UNIFIED_ROADMAP_5_STAGES.md)
10. [خطة التنفيذ المتوازي — 50 مهمة، 10 لكل مرحلة](docs/UNIFIED_50_TASK_EXECUTION_PLAN.md)
11. [جرد قدرات AI الحالية قبل بناء AI Core](docs/AI_CORE_CAPABILITY_INVENTORY.md)
12. [متطلبات الوكلاء المتخصصين في AI Core — S4-02](docs/AI_CORE_SPECIALIZED_AGENT_REQUIREMENTS.md)
13. [مصفوفة أدوات وصلاحيات الوكلاء — S4-06](docs/AI_CORE_AGENT_TOOL_MATRIX.md)
14. [سجل ملكية المنتجات وحدود التكامل](docs/PRODUCT_OWNERSHIP_AND_INTERFACE_CATALOG.md)
15. [كتالوج عقود API بين المنتجات — S5-02](docs/PRODUCT_API_CONTRACT_CATALOG.md)
16. [تصميم Model Gateway وسجل النماذج — S4-04](docs/AI_CORE_MODEL_GATEWAY_DESIGN.md)
17. [تصميم هوية الخدمات والتفويض — S5-03](docs/SERVICE_IDENTITY_AND_DELEGATION_DESIGN.md)
18. [تصميم تصنيف البيانات والاحتفاظ والخصوصية — S5-05](docs/DATA_CLASSIFICATION_AND_RETENTION_DESIGN.md)
19. [السجل التاريخي لمقترح دفعة 15 مهمة — مستبدل](docs/UNIFIED_BATCH_EXECUTION_PLAN.md)
20. [نموذج التفويض الموحد RBAC/ABAC ونطاق الموارد — S5-04](docs/UNIFIED_AUTHORIZATION_MODEL_DESIGN.md)
21. [مذكرة قرار ملكية الأسئلة والدروس في Lumen — LUM-010](docs/LUMEN_CONTENT_OWNERSHIP_DECISION_BRIEF.md)
22. [نقطة تدقيق الصلاحيات وIDOR في Lumen — LUM-010](docs/LUMEN_AUTHORIZATION_AUDIT_CHECKPOINT_20261117.md)
23. [نقطة تدقيق الصلاحيات الثانية في Lumen — LUM-010](docs/LUMEN_AUTHORIZATION_AUDIT_CHECKPOINT_2_20261117.md)

## قاعدة مصدر الحقيقة

- كود كل منتج: `main` في مستودعه، بعد التحقق من SHA الحالي.
- العمل غير المدمج: الفرع/PR الخاص به، مع إبقائه منفصلاً عن حالة `main`.
- الاختبارات: CI على نفس Commit المراد اعتماده.
- التشغيل: دليل فعلي مؤرخ من البيئة المستهدفة.
- التكامل بين المنتجات: عقود API موثقة وإصدارات واضحة.
- هذا المستودع: حوكمة مشتركة، سجلات وفهرس ولقطات أرشيفية.

## المنتجات

- [Lumen](https://github.com/HVA09/lumen-learning-platform)
- [AI Chat App](https://github.com/HVA09/ai-chat-app)
- [ERP Global Pro](https://github.com/HVA09/erp-global-pro)

## قواعد لا تُكسر

لا تدمج المستودعات، ولا تشارك قواعد البيانات، ولا تنقل أسراراً أو بيانات حقيقية إلى الأرشيف. لا تعتبر Snapshot الأرشيف نسخة حية. لا تحذف أو تعدّل الملفات التاريخية لإخفاء تعارض؛ أضف قرار مصالحة موثقاً. راجع سجل المهام قبل إنشاء عمل جديد حتى لا يتكرر أو يتعارض.

**آخر مراجعة للمصادر:** 17 نوفمبر 2026. الحالة التاريخية مفهرسة في [سجل المصالحة](docs/CONFLICTS_AND_RECONCILIATION.md).
