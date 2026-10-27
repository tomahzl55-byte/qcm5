# نشر البوت على Render (خطوة بخطوة)

هذا الدليل يشرح كيف تشغّل البوت على **Render** (خطة مجانية) بنظام **Webhook**.
ملفات النشر جاهزة في المشروع: `render.yaml`، `requirements.txt`، `runtime.txt`،
`render_app.py`.

> **المستودع جاهز على GitHub:** https://github.com/tomahzl55-byte/qcm5
>
> **نشر بضغطة واحدة:** https://render.com/deploy?repo=https://github.com/tomahzl55-byte/qcm5
> (سجّل بحساب GitHub → اسمح لـ Render بالوصول للمستودع → الصق `BOT_TOKEN` → **Apply**).

---

## 0. قبل البدء (مهم)

- **جدّد التوكن** من @BotFather إن كان قد تسرّب في أي مكان:
  `/mybots` → بوتك → **API Token** → **Revoke current token**.
- **لا تكتب التوكن في أي ملف داخل المشروع.** يوضع فقط في
  Render → Environment.
- البوت يجب أن يكون **مشرفًا (Admin)** في القناة `@qcMAX_batna`، وإلا لن
  يستطيع التحقق من اشتراك المستخدمين.

---

## 1. رفع الكود إلى GitHub (تمّ مسبقًا)

المشروع مرفوع على https://github.com/tomahzl55-byte/qcm5 — لا تحتاج لتكرار هذه
الخطوة. إن أردت رفعه لمستودع آخر:

```bash
git init
git add .
git commit -m "Prepare bot for Render deployment"
```

أنشئ مستودعًا **فارغًا** (بدون README) على GitHub باسم مثل `qcm-bot`، ثم:

```bash
git branch -M main
git remote add origin https://github.com/<USERNAME>/qcm5.git
git push -u origin main
```

> تحقّق قبل الرفع أن `bot_users.json` و`reported_questions.json` غير موجودين
> في الـ commit (موجودان في `.gitignore`).

---

## 2. إنشاء الخدمة على Render

### الطريقة الأسهل: Blueprint (يقرأ `render.yaml` تلقائيًا)

1. سجّل في https://render.com (يمكن الدخول بحساب GitHub).
2. افتح الرابط الجاهز: https://render.com/deploy?repo=https://github.com/tomahzl55-byte/qcm5
   (أو **New +** → **Blueprint** → اختر المستودع `qcm5` → **Connect**).
3. سيعرض Render الخدمة `qcm-bot` وكل المتغيرات. الصق `BOT_TOKEN` ثم اضغط **Apply**.

### أو يدويًا: New + → Web Service

| الإعداد | القيمة |
|---|---|
| Repository | `qcm5` |
| Runtime | Python 3 |
| Build Command | `pip install -r requirements.txt` |
| Start Command | `python render_app.py` |
| Plan | Free |
| Health Check Path | اتركه فارغًا |

---

## 3. المتغيرات (Environment)

من صفحة الخدمة → **Environment** → أضف:

| المتغير | القيمة |
|---|---|
| `BOT_TOKEN` | التوكن الجديد من @BotFather (**إلزامي**) |
| `REQUIRED_CHANNEL` | `@qcMAX_batna` (أو اتركه فارغًا لتعطيل شرط الاشتراك) |
| `ADMIN_USER_IDS` | `5696844039` (عدة أرقام مفصولة بفواصل) |
| `WEBHOOK_SECRET` | يُولّده Render تلقائيًا (لا تلمسه) |

`PORT` و`RENDER_EXTERNAL_URL` يضيفهما Render تلقائيًا.

اضغط **Save** → سيُعاد النشر تلقائيًا.

---

## 4. التحقق

1. افتح رابط الخدمة `https://qcm-bot-xxxx.onrender.com` — يجب أن ترى:
   `✅ Le bot QCM est en ligne (mode webhook).`
2. اذهب إلى البوت في تلجرام وأرسل `/start`.
3. في سجلّات Render (**Logs**) يجب أن ترى:
   ```
   Webhook Telegram configuré : https://.../webhook/...
   ```
4. للتأكد من ربط الويب هوك، افتح في المتصفح:
   `https://api.telegram.org/bot<TOKEN>/getWebhookInfo`
   يجب أن يظهر `"url"` الخاص بخدمتك و`"pending_update_count": 0`.

> **مهم:** لا تشغّل البوت بنظام polling (محليًا أو على استضافة أخرى) بنفس
> التوكن في نفس الوقت — سيسبب تعارضًا مع الويب هوك.

---

## 5. منع النوم (Keep-alive)

الخدمة المجانية على Render تنام بعد **15 دقيقة** بلا زيارات، وتستغرق أول
طلبة ~30–60 ثانية للاستيقاظ (قد تتأخر رسائل تلجرام). الحل المجاني:

1. اذهب إلى https://cron-job.org (أو UptimeRobot) وأنشئ حسابًا.
2. أضف مهمة URL = `https://qcm-bot-xxxx.onrender.com/healthz` كل **10 دقائق**.

الخطة المجانية تمنح **750 ساعة/شهر** لكل حساب، وهي كافية لخدمة واحدة تعمل
24/7 (الشهر ≈ 744 ساعة).

---

## 6. ملاحظة عن حفظ البيانات (متى تُفقد؟)

نظام ملفات Render **مؤقت**، وليس هناك مدّة محدّدة: الملفات `bot_users.json`
(قائمة المستخدمين) و`reported_questions.json` (الإبلاغات) تُفقد **فورًا** عند
أي من هذه الأحداث:

| الحدث | متى يحدث |
|---|---|
| إعادة نشر (deploy) | كل `git push` أو Deploy يدوي |
| إعادة تشغيل (restart) | يدويًا أو من Render تلقائيًا (قد يحدث في أي وقت) |
| نوم الخدمة (spin-down) | بعد **15 دقيقة** بلا أي طلب وارد |

> هذا مهم: في وضع Webhook لا يصل أي طلب إلى الخدمة إن لم يراسل أحد البوت،
> لذا بدون Keep-alive تنام الخدمة بعد ~15 دقيقة من آخر رسالة وتُفقد الملفات.
> مع ping كل 10 دقائق تبقى الخدمة مستيقظة وتحفظ الملفات حتى إعادة النشر التالية.

- قائمة الأسئلة نفسها في `data/*.json` محفوظة في Git، فلا مشكلة (تُعاد مع كل نشر).
- **الأقراص الدائمة (Persistent Disks) غير متاحة في الخطة المجانية** — تحتاج
  خطة مدفوعة (Starter). ولأن مسارات الملفات في الكود ثابتة حاليًا، ستحتاج
  تعديلًا صغيرًا ليدعم متغير `DATA_DIR` قبل استخدام القرص.
- البديل المجاني المستقر: قاعدة بيانات خارجية (مثل Neon أو Supabase) وتعديل
  الكود ليحفظ المستخدمين والإبلاغات فيها بدل الملفات.
- Render Postgres المجاني متاح لكنه **ينتهي بعد 30 يومًا** من إنشائه، فلا يصلح
  للحفظ الدائم.

---

## 7. التحديثات المستقبلية

كل `git push` على فرع `main` يُعيد النشر تلقائيًا (`autoDeploy: true`).
لتعديل الأسئلة: عدّل `data/*.json` ثم:

```bash
git add data
git commit -m "Update questions"
git push
```

---

## 8. استكشاف الأخطاء

| المشكلة | الحل |
|---|---|
| `BOT_TOKEN manquant` | أضف المتغير في Environment وأعد النشر |
| البوت لا يردّ | افحص **Logs**؛ تحقق من `getWebhookInfo` |
| `409 Conflict` | البوت يعمل بمكان آخر (polling) بنفس التوكن — أوقفه |
| لا يتحقق من الاشتراك | البوت ليس مشرفًا في القناة، أو `REQUIRED_CHANNEL` خاطئ |
| `Unauthorized` | التوكن قديم/تم إبطاله — جدّده من @BotFather |
