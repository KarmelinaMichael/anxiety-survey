# استبيان القلق للشباب — نسخة Vercel

صفحة واحدة + دالتين serverless. مفيش build ولا dependencies.

```
anxiety-survey/
├── index.html        الاستبيان والنتائج (كل حاجة في ملف واحد)
├── api/submit.js     يستقبل إجابة واحدة ويخزّنها
├── api/stats.js      يرجّع الإحصائيات المجمَّعة فقط
├── package.json
└── README.md
```

## ١. قاعدة البيانات (Supabase — مجاني)

اعملي مشروع جديد على [supabase.com](https://supabase.com)، وافتحي **SQL Editor** ونفّذي:

```sql
create table responses (
  id          bigint generated always as identity primary key,
  respondent  text unique not null,
  answers     smallint[] not null,
  total       smallint not null,
  level       smallint not null,
  created_at  timestamptz not null default now()
);

alter table responses enable row level security;
-- من غير أي policy: مفيش حد يقدر يقرا أو يكتب إلا الـ service key
-- اللي بيستخدمه السيرفر بتاعنا. المتصفح عمره ما بيوصل لـ Supabase مباشرة.
```

من **Project Settings → API** انسخي:
- `Project URL`
- `service_role` key (ده سري تمامًا، ميتحطش في أي ملف في الريبو)

## ٢. الرفع على Vercel

الريبو على GitHub ← **Import Project** على [vercel.com](https://vercel.com)، وسيبي كل الإعدادات زي ما هي (Framework Preset: Other).

قبل الـ Deploy ضيفي **Environment Variables**:

| الاسم | القيمة |
| --- | --- |
| `SUPABASE_URL` | الـ Project URL |
| `SUPABASE_SERVICE_KEY` | الـ service_role key |

اعملي Deploy، وهيطلعلك لينك زي `https://anxiety-survey.vercel.app` — ده اللي يتبعت للشباب. أي حد يفتحه من أي موبايل من غير تسجيل دخول.

> لو غيّرتي أي Environment Variable بعد الـ deploy، لازم تعملي Redeploy عشان تشتغل.

## ٣. التجربة

1. افتحي اللينك وجاوبي من الموبايل.
2. افتحيه من جهاز تاني وجاوبي.
3. دوسي **عرض النتائج المجمَّعة** — المفروض تلاقي عدد المشاركين اتنين والأرقام اتحدّثت.

## ملاحظات

- **الخصوصية:** مفيش اسم ولا إيميل ولا IP بيتخزن. كل جهاز بياخد معرّف عشوائي في الـ localStorage عشان لو الشخص جاوب تاني تتحدّث إجابته بدل ما تتكرر. الـ API بيرجّع أرقام مجمَّعة بس — مفيش طريقة من الصفحة تشوف بيها إجابة شخص بعينه.
- **التحديث اللايف:** الصفحة بتسأل السيرفر كل ٥ ثواني وهي مفتوحة على شاشة النتائج.
- **السعة:** الخطة المجانية في Supabase و Vercel تكفي آلاف الإجابات بسهولة.
- **تصفير النتائج قبل اجتماع جديد:** في Supabase SQL Editor نفّذي `truncate table responses;`
- **سحب البيانات للتحليل:** من Supabase → Table Editor → Export CSV.
- **تعديل الأسئلة:** كلها في مصفوفة `Q` في أول السكريبت في `index.html`. لو غيّرتي عدد الأسئلة، غيّري الرقم `15` في `api/submit.js` و `api/stats.js` وفي حساب الدرجات والمستويات.
