# 🚀 Sprint 1: Clean Architecture, Domain Modeling, EF Core, Generic Repository/UoW, Soft Delete & Identity/JWT/Swagger (الخطوات 1–7ب)

## الخطوة 1 — Clean Architecture وإنشاء المشاريع الأربعة

**⭐ المطلوب الأساسي**

- Clean Architecture 10,000ft View — عصام عبدالنبي [`DRSZ6V-eKUY`](https://youtu.be/DRSZ6V-eKUY) (19:37)

**🔄 بديل / استزادة**

- Beginner's Guide to Clean Architecture — Milan [`TQdLgzVk2T8`](https://youtu.be/TQdLgzVk2T8)
- Clean Architecture & CQRS in Action — عصام (.NET 10) [`v9fRFyg414k`](https://youtu.be/v9fRFyg414k) (1:31:41 — جزء إنشاء المشاريع وملف .slnx من 09:18 إلى 20:25 فقط)

**✅ تم؟** `[ ]`

## الخطوة 2 — بناء Entity و ValueObject (لحل Primitive Obsession)

**⭐ المطلوب الأساسي**

- DDD 06: Tactical Design - Getting Started — محمد المهدي [`bHEDV62ATNM`](https://youtu.be/bHEDV62ATNM) (33:40 — يبني Entity في 09:00 و ValueObject في 19:10 و Aggregate Root في 24:55)

**🔄 بديل / استزادة**

- Value Objects — Milan [`P5CRea21R2E`](https://youtu.be/P5CRea21R2E) (13:54)
- What Is An Entity — Milan [`00tCda35Bvk`](https://youtu.be/00tCda35Bvk)
- #003 Immutable Vs. Mutable — عصام [`bbdmknnwcP8`](https://youtu.be/bbdmknnwcP8)

**✅ تم؟** `[ ]`

## الخطوة 3 — إعداد AppDbContext و IEntityTypeConfiguration و Migrations

**⭐ المطلوب الأساسي**

- #010 EF Migration بالعربي — عصام عبدالنبي [`WqiGu_MQ-xw`](https://youtu.be/WqiGu_MQ-xw) (46:32)

**🔄 بديل / استزادة**

- Correct Way to Run Database Migrations — Nick Chapsas [`StBf3-M5WdM`](https://youtu.be/StBf3-M5WdM)

**✅ تم؟** `[ ]`

## الخطوة 4 — بناء Generic Repository و Unit of Work

**⭐ المطلوب الأساسي**

- ⭐1 3. Add The Base (Generic) Repository — محمد الهلالي [`7Lc9NRaPMr0`](https://youtu.be/7Lc9NRaPMr0) (11:57 — .NET 6)
- ⭐2 11. Use Complete Method (UnitOfWork) — محمد الهلالي [`-72HfsPv9Xs`](https://youtu.be/-72HfsPv9Xs) (03:37 — .NET 6)

**🔄 بديل / استزادة**

- Transactions & UoW — Milan [`25H84BXcr9M`](https://youtu.be/25H84BXcr9M)
- قائمة Repository & UoW الكاملة — الهلالي

**✅ تم؟** `[ ]`

## الخطوة 5 — تطبيق Soft Delete و Global Query Filters

**⭐ المطلوب الأساسي**

- #019 Interceptors EF Core — عصام عبدالنبي [`hEMCDxqURr8`](https://youtu.be/hEMCDxqURr8) (22:02 — يبني ISoftDeletable و SoftDeleteInterceptor و HasQueryFilter)

**🔄 بديل / استزادة**

- Soft Deletes With EF Core — Milan [`zwBaWiu-rF4`](https://youtu.be/zwBaWiu-rF4) (12:39 — يشرح الكود الكامل ثم يختم برأي نقدي في 11:25)
- Global Query Filters — Milan [`q09fw5OCa_w`](https://youtu.be/q09fw5OCa_w)

**✅ تم؟** `[ ]`

## الخطوة 6 — تطبيق Options Pattern (IOptions, IOptionsSnapshot, IOptionsMonitor)

**⭐ المطلوب الأساسي**

- #11 Options Pattern في ASP.NET Core 9 — CodingClass [`S3ygXq94dLo`](https://youtu.be/S3ygXq94dLo) (13:55)

**🔄 بديل / استزادة**

- Options Pattern — Milan [`wxYt0motww0`](https://youtu.be/wxYt0motww0)

**✅ تم؟** `[ ]`

## الخطوة 7أ — إعداد ASP.NET Core Identity و JWT (التسجيل Registration وتسجيل الدخول Login)

**⭐ المطلوب الأساسي**

- ⭐1 إعداد Identity وجداولها بالعربي: 03 Create Database and Identity Tables — الهلالي [`zshypyVIVFU`](https://youtu.be/zshypyVIVFU) (07:13 — قديم .NET 5: يُكتب الكود في Program.cs بدلاً من Startup.cs). للطلاب على .NET 10 يمكن بدلاً منه: Auth made easy with ASP.NET Core Identity (.NET 10) — Remigiusz Zalewski [`g9YSB4bXhes`](https://youtu.be/g9YSB4bXhes) (14:50)
- ⭐2 إعداد JWT والـ Registration/Login بالعربي: 05 Add JWT Configuration [`B7hQY1YckR0`](https://youtu.be/B7hQY1YckR0) (06:35) + 06 Prepare Registration EndPoint [`FqQjESC4Gvs`](https://youtu.be/FqQjESC4Gvs) (12:48) + 07 Registration EndPoint Part 2 [`PXKrGK4zjuM`](https://youtu.be/PXKrGK4zjuM) (06:26) — الهلالي

**🔄 بديل / استزادة**

- [Identity API endpoints — Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-api-authorization) (مرجع نصي لـ MapIdentityApi على .NET 8+)
- ASP.NET Web API 8 Identity — Fullstack Dev [`tXUtSGvHMeg`](https://youtu.be/tXUtSGvHMeg) (2:23:36 — الدقيقة 12:15 لـ AddIdentityApiEndpoints)

**✅ تم؟** `[ ]`

## الخطوة 7ب — تدوير Refresh Token وتفعيل JWT في أداة التوثيق (OpenAPI / Scalar / Swagger)

**⭐ المطلوب الأساسي**

- ⭐1 توليد وتدوير Refresh Token بالعربي: 4. Generate Refresh Token [`5g0aq4g17yg`](https://youtu.be/5g0aq4g17yg) (10:18) + 7. Generate New JWT Using Refresh Token [`jefxQ2xFM8I`](https://youtu.be/jefxQ2xFM8I) (05:55) — الهلالي
- ⭐2 تفعيل خانة التوكن في أداة التوثيق حسب إصدار .NET: على .NET 9/10 (بدون Swagger افتراضي): No Swagger in .NET 9? Replace it with SCALAR! — Israel Quiroz [`J8i1RWowjwc`](https://youtu.be/J8i1RWowjwc) (12:15 — BearerSecuritySchemeTransformer في الدقيقة 09:50) أو Swagger is Gone in .NET 9! — Nick Chapsas [`8yI4gD1HruY`](https://youtu.be/8yI4gD1HruY) (09:34 — لإعادة SwaggerUI في 02:35). على .NET 8: الـ Swagger الافتراضي يعمل، ويكفي فيديو الهلالي Swagger JWT [`LGPiZcacG5g`](https://youtu.be/LGPiZcacG5g) (10:44)

**🔄 بديل / استزادة**

- بديل شامل على .NET 9 (2025) يجمع JWT + Roles + Refresh Token + Scalar: JWT Authentication with .NET 9 Full Course — Patrick God [`6EEltKS8AwA`](https://youtu.be/6EEltKS8AwA) (1:13:38 — مقسم بالتوقيت في الملحق)
- Master Refresh Tokens — Milan (.NET 8/9) [`kR_9gRBeRMQ`](https://youtu.be/kR_9gRBeRMQ) (17:19)

**✅ تم؟** `[ ]`

---

**إثرائي (بعد تسليم Sprint 1): Clean vs Vertical Slice [`Az4Z0HJYl-U`](https://youtu.be/Az4Z0HJYl-U) · Simplest Vertical Slice [`dQdXHRkePr8`](https://youtu.be/dQdXHRkePr8) · Project Setup [`tLk4pZZtiDY`](https://youtu.be/tLk4pZZtiDY) · DDD 01 [`AV3yzj6Ck7k`](https://youtu.be/AV3yzj6Ck7k) · Bounded Contexts [`kLLsVT_53bw`](https://youtu.be/kLLsVT_53bw) · Specification Pattern [`bEyBtpnCqVY`](https://youtu.be/bEyBtpnCqVY) · Dapper [`C763K-VGkfc`](https://youtu.be/C763K-VGkfc) · Validate Options [`I0YPTeCYvrE`](https://youtu.be/I0YPTeCYvrE) · What Is JWT [`7Q17ubqLfaM`](https://youtu.be/7Q17ubqLfaM) · OAuth/OIDC [`t18YB3xDfXI`](https://youtu.be/t18YB3xDfXI)**
