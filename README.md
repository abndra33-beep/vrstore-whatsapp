# VR STORE — WhatsApp Notification Service

خدمة Baileys + Express لإرسال إشعارات طلبات VR STORE إلى رقم واتساب المسؤول (مع صورة إيصال التحويل).

---

## 📦 الملفات المطلوبة (ارفعها كما هي إلى مستودع GitHub جديد)

```
whatsapp-service/
├── server.js         ← الكود الأساسي
├── package.json      ← التبعيات
├── Dockerfile        ← لبناء الصورة على Railway
├── .gitignore
└── README.md
```

فقط هذه الخمسة. لا ترفع `node_modules` ولا `bun.lock`.

---

## 🚀 خطوات النشر على Railway

### 1) أنشئ مستودع GitHub
- افتح https://github.com/new
- اسم مقترح: `vrstore-whatsapp`
- خاص (Private) أو عام — كلاهما يعمل
- ارفع الملفات أعلاه (سحب وإفلات في الواجهة أو `git push`)

### 2) أنشئ مشروع Railway
- افتح https://railway.app → **New Project** → **Deploy from GitHub repo**
- اختر المستودع `vrstore-whatsapp`
- Railway سيكتشف `Dockerfile` تلقائياً ويبدأ البناء

### 3) أضف متغيرات البيئة (Variables)
في تبويب **Variables** لخدمتك:

| المتغير | القيمة |
|--------|--------|
| `SERVICE_TOKEN` | اختَر كلمة سر قوية (مثال: `vr_secret_9k2mLp7X`) — استخدمها لاحقاً في لوحة التحكم |
| `PORT` | `3000` |
| `AUTH_DIR` | `/data/auth` |

### 4) أضف قرص دائم (Volume) — مهم جداً
حتى لا يُفصل واتساب عند كل إعادة تشغيل:
- تبويب **Settings** → **Volumes** → **New Volume**
- **Mount Path:** `/data`
- **Size:** 1 GB يكفي

### 5) فعّل الدومين العام
- تبويب **Settings** → **Networking** → **Generate Domain**
- انسخ الرابط (مثال: `https://vrstore-whatsapp-production.up.railway.app`)

### 6) اربط اللوحة بالسيرفر
في لوحة تحكم VR STORE → الإعدادات → **سيرفرات واتساب (Railway)**:
- **Server URL:** الرابط من الخطوة 5
- **Token:** نفس قيمة `SERVICE_TOKEN`
- اضغط **حفظ السيرفر** ثم **تفعيل** ثم **اختبار**

### 7) اقرن واتساب (QR)
افتح في المتصفح:
```
https://<railway-url>/status
```
- أضف Header: `Authorization: Bearer <SERVICE_TOKEN>` (استخدم إضافة مثل ModHeader، أو `curl`)
- ستحصل على `qr` كصورة Base64 — امسحها من:
  واتساب على جوالك → الإعدادات → الأجهزة المرتبطة → ربط جهاز

الأسرع: افتح الرابط `https://<railway-url>/status` من الطرفية:
```bash
curl -H "Authorization: Bearer <SERVICE_TOKEN>" https://<railway-url>/status
```
انسخ قيمة `qr` والصقها في أي أداة عرض data:image لعرض الـ QR.

بعد المسح، ستصبح الحالة `open` والسيرفر جاهز.

---

## ✅ اختبار سريع
```bash
curl -X POST https://<railway-url>/send \
  -H "Authorization: Bearer <SERVICE_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"to":"96894913233","message":"اختبار من VR STORE"}'
```

---

## 🔒 ملاحظات
- **لا تشارك `SERVICE_TOKEN`** — من يعرفه يمكنه الإرسال باسمك.
- الـ Volume ضروري: بدونه سيُفصل الواتساب عند كل نشر جديد.
- إذا فُصل واتساب: افتح `/logout` ثم أعد المسح.

## 📡 الـ Endpoints
| Method | Path | الوصف |
|--------|------|-------|
| GET  | `/` | فحص حي (بدون auth) |
| GET  | `/status` | الحالة + QR إن وُجد |
| GET  | `/qr` | صورة QR فقط |
| POST | `/send` | إرسال `{to, message, imageUrl?}` |
| POST | `/restart` | إعادة تشغيل الاتصال |
| POST | `/logout` | تسجيل خروج ومسح الجلسة |

جميع الـ endpoints ماعدا `/` تتطلب Header:
`Authorization: Bearer <SERVICE_TOKEN>`
