# 📚 الدليل الموحد — Round 2 (.NET Level 2 — DonationHub)

النسخة V10 — 17/10/2026. المشروع التطبيقي: DonationHub — Tawjeh Mentorship Program.

**الإصدار المعتمد:** .NET 10 (`net10.0` مع ملف الحل `.slnx` و `Program.cs`). الدليل يعمل أيضاً على .NET 8 و .NET 9 — راجع قسم التوافق أدناه.

## 🧭 كيف تقرأ هذا الدليل؟

- ⭐ = المطلوب الأساسي لتنفيذ التاسك. 🔄 = بديل أو استزادة (اختياري).
- العربي أولاً؛ الإنجليزي ⭐ فقط إذا لم يتوفر شرح عربي مباشر أو كان أقصر وأدق لإصدار .NET 8/9/10.
- حين يتكون التاسك من عدة متطلبات في الـ SRS يُقسم إلى ⭐1 و ⭐2 حتى لا يسقط متطلب في خانة الاختياري.

## ✅ منهجية التحقق

- عناوين وقنوات الـ 9 معرّفات الجديدة (الملحق 2) أُعيد التحقق منها عبر YouTube oEmbed بتاريخ 17/10/2026 وتطابقت كلها.
- المدد وتواريخ النشر **استرشادية وغير مؤكدة بشكل مستقل**؛ أُكّدت صفحياً 4 مدد فقط: `v9fRFyg414k` و `lsP5ZjcFtMM` و `29x5WmcUNtM` و `tXUtSGvHMeg`.
- روابط التوثيق (Microsoft Learn / Hangfire / Stripe / Mailpit) أُضيفت بصيغتها الرسمية المعروفة؛ افتحها مرة للتأكد. رابط Mailpit لم يُفتح من جهتي.
- التوقيتات الفرعية استرشادية، ويراجعها المنتور بصرياً قبل التوزيع.

## ⛔ بوابة الإطلاق (لكل Sprint قبل التوزيع)

1. **مراجعة التوقيتات بصرياً:** يفتح المنسق التوقيت المذكور في ملحق المنتور ويتأكد من بدايته ونهايته؛ وإن لم يتحقق يرسل البديل القصير 🔄 في نفس الخطوة.
2. **Reference Solution على .NET 10:** يبنيه المنسق قبل التوزيع بأسبوع ويوثق الفروق عن الفيديوهات الأقدم (Startup.cs ← Program.cs، وSwashbuckle ← OpenAPI/Scalar في الخطوة 7ب).
3. **فيديوهات طويلة تتطلب تحديد مقطع:** [`v9fRFyg414k`](https://youtu.be/v9fRFyg414k) (خطوات 1 و 8 و 12 — 1:31:41)، [`fb8j6CjyWWw`](https://youtu.be/fb8j6CjyWWw) (خطوتا 16 و 30 — 1:11:58)، [`6EEltKS8AwA`](https://youtu.be/6EEltKS8AwA) (بديل في 7أ/7ب — 1:13:38)، [`WZu0lHvNYjI`](https://youtu.be/WZu0lHvNYjI) (خطوة 27 — 10:58).

## 📋 القرارات الهندسية والتراخيص

| البند | القرار | الأمر / الملاحظة |
|---|---|---|
| إصدار .NET | .NET 10 (`net10.0`) | الفيديوهات على .NET 5/6/7 موسومة (قديم)؛ الكود الداخلي للمكتبات متطابق ويُكتب في `Program.cs`. في .NET 9/10 أُزيل Swashbuckle من القالب ويُستخدم `builder.Services.AddOpenApi();` و `app.MapOpenApi();` مع `Scalar.AspNetCore` (راجع [Nick Chapsas](https://youtu.be/8yI4gD1HruY) و [Israel Quiroz](https://youtu.be/J8i1RWowjwc) في 7ب). |
| التوافق مع .NET 8 / 9 | كل الخطوات تعمل مع الفروق التالية | `.slnx` يتطلب SDK حديثاً (9.0.200+) — على .NET 8 استخدم `dotnet new sln`. `AddOpenApi/MapOpenApi` من .NET 9؛ على .NET 8 يعمل Swagger الافتراضي. `MapIdentityApi` و `IExceptionHandler` و `TimeProvider` من .NET 8، والـ Rate Limiting من .NET 7. غيّر `TargetFramework` وحزم Microsoft.* و EF Core إلى الإصدار الرئيسي نفسه ولا تخلط الإصدارات. |
| MediatR | تثبيت 12.5.0 | `<PackageReference Include="MediatR" Version="[12.5.0]" />` — الأقواس تمنع الترقية التلقائية للإصدار 13+ التجاري. بديل مستقبلي: مفتاح Community أو Wolverine. |
| FluentAssertions | تثبيت 7.0.0 (Apache 2.0) | `<PackageReference Include="FluentAssertions" Version="[7.0.0]" />` — الإصدار 8+ تجاري. البديل: Shouldly. |
| Moq | تثبيت 4.18.4 | `<PackageReference Include="Moq" Version="[4.18.4]" />` — يتجنب جدل SponsorLink في 4.20.0. البديل: NSubstitute. |
| QuestPDF | رخصة Community أو Evaluation | `QuestPDF.Settings.License = LicenseType.Community;` — وفق الصفحة الرسمية: مجانية للأفراد والشركات بدخل أقل من مليون دولار، والتعليم والتجريب، والمؤسسات غير الربحية، والمصادر المفتوحة؛ ولا تشمل الجهات الحكومية ولا الشركات المدرجة. |
| Hangfire | الحزم الأساسية المجانية (LGPL v3) | `Hangfire.Core` و `Hangfire.AspNetCore` و `Hangfire.SqlServer`. تجنب حزم Pro المدفوعة. |
| Redis | Docker محلياً | `docker run -d --name redis -p 6379:6379 redis:alpine` — البديل المفتوح: `valkey/valkey:alpine`. |
| Mailpit | حاوية `axllent/mailpit` | `docker run -d --name mailpit -p 8025:8025 -p 1025:1025 axllent/mailpit` — SMTP على 1025 (`localhost`، بدون تشفير في التطوير)، والواجهة على http://localhost:8025. |
| أخطاء الـ API | RFC 9457 ProblemDetails | `builder.Services.AddProblemDetails();` — المحاضرون يذكرون RFC 7807 والكود واحد. |

## 🗂️ فهرس الـ Sprints

- [🏁 Sprint 0 — المرحلة التحضيرية (قبل بدء الـ Sprints)](sprint-0.md) — 5 خطوات
- [🚀 Sprint 1: Clean Architecture, Domain Modeling, EF Core, Generic Repository/UoW, Soft Delete & Identity/JWT/Swagger (الخطوات 1–7ب)](sprint-1.md) — 8 خطوات
- [⚡ Sprint 2: Campaigns, CQRS/MediatR, CancellationToken, TimeProvider, Concurrency, Result Pattern, Validation, Exception Handling & Unit Testing](sprint-2.md) — 7 خطوات
- [🛡️ Sprint 3: Donations Core, Transactions, Pagination/Filtering, Resource-Based Auth, Redis, Localization, Versioning, Rate Limiting & CORS](sprint-3.md) — 7 خطوات
- [🔔 Sprint 4: Hangfire, MailKit + Mailpit Local SMTP, QuestPDF, File Uploads (Magic Bytes) & Serilog/Seq](sprint-4.md) — 5 خطوات
- [💳 Sprint 5: Stripe Checkout, Webhook Signature, Stripe CLI, Idempotency & Integration Testing](sprint-5.md) — 3 خطوات
- [🐳 Sprint 6: EF Core Aggregations, Multi-stage Dockerfile, Docker Compose, Health Checks, CI/CD & README](sprint-6.md) — 6 خطوات
- [ملحق المنتور (1): التوقيتات الفرعية](mentor-appendix-1-timestamps.md)
- [ملحق المنتور (2): مطابقة المعرّفات](mentor-appendix-2-id-mapping.md)

## 📖 ما بعد الدورة (Post-Course Reading)

- SignalR: tutorialsEU [`LEImpa77irI`](https://youtu.be/LEImpa77irI) · Mohamad Lawand [`pvi_ZS_PrSc`](https://youtu.be/pvi_ZS_PrSc) · قائمة أحمد محمدي بالعربي
- .NET Aspire & Microservices: Aspire — Julio Casal [`pk6FJfHhfq8`](https://youtu.be/pk6FJfHhfq8) · Microservices — Les Jackson [`DgVjEo3OGBI`](https://youtu.be/DgVjEo3OGBI) · Julio Casal [`ByYyk8eMG6c`](https://youtu.be/ByYyk8eMG6c) · DotNetMastery [`Nw4AZs1kLAs`](https://youtu.be/Nw4AZs1kLAs)
- كتب مقترحة: Clean Architecture، و Dependency Injection Principles, Practices, and Patterns، و Domain-Driven Design، و Software Architecture with C# 12 and .NET 8، و Designing Data-Intensive Applications.
