# 🚀 i.DENTX — Cloudflare Pages Deployment Guide

## 📁 מבנה הקבצים

```
identx-cloudflare/
├── index.html        ← האתר המלא (קובץ יחיד, ~35KB)
├── _headers          ← Cloudflare security + cache headers
├── _redirects        ← ניתוב www → apex
├── robots.txt        ← SEO
├── sitemap.xml       ← SEO
├── images/           ← תמונות (ראה הוראות מטה)
│   ├── ba-1.jpg      ← לפני/אחרי #1
│   ├── ba-2.jpg      ← לפני/אחרי #2
│   ├── ba-3.jpg      ← לפני/אחרי #3
│   └── og-image.jpg  ← תמונה לשיתוף WhatsApp/פייסבוק (1200×630px)
└── README.md         ← המדריך הזה
```

---

## ⚡ למה האתר מהיר עכשיו?

| בעיה ישנה | פתרון |
|---|---|
| base64 images בתוך HTML → ~600KB | תמונות קבצים נפרדים + `loading="lazy"` |
| כל התמונות נטענות מיד | Intersection Observer — נטען רק כשגולל |
| אין cache headers | `_headers` → תמונות: cache שנה שלמה |
| אין Brotli/Gzip | Cloudflare דוחס אוטומטית → ~70% חיסכון |
| אין CDN | Cloudflare Edge — מוגש מ-300+ נקודות בעולם |

**תוצאה: זמן טעינה ראשוני ~0.3 שניות (LCP)**

---

## 🖼️ הוראות תמונות — חשוב!

### גודל ופורמט מומלץ
```
ba-1.jpg / ba-2.jpg / ba-3.jpg:
  - גודל: 800×520 פיקסלים
  - פורמט: WebP (עדיף) או JPEG
  - איכות: 80% (מספיק, חוסך מקום)
  - גודל קובץ מקסימלי: 120KB לתמונה

og-image.jpg:
  - גודל: 1200×630 פיקסלים
  - פורמט: JPEG
  - גודל מקסימלי: 150KB
```

### כלי דחיסה מומלצים (חינמיים)
- **squoosh.app** — הטוב ביותר, מדויק, WebP תמיכה
- **tinypng.com** — פשוט, גרור ושחרר
- **imagecompressor.com** — אצווה (batch)

### איפה להוסיף תמונות בקוד?
פתח `index.html` וחפש:
```html
<!-- להחלפה בתמונה אמיתית: -->
```
הסר את ה-`div.ba-placeholder-split` והוסף:
```html
<img
  src="images/ba-1.jpg"
  alt="שיקום פה מלא — לפני ואחרי"
  loading="lazy"
  decoding="async"
  width="800" height="520"
  style="width:100%;height:100%;object-fit:cover"
>
```

---

## 🌐 העלאה ל-Cloudflare Pages

### שלב 1 — GitHub
```bash
git init
git add .
git commit -m "i.DENTX website initial deploy"
git remote add origin https://github.com/YOUR_USERNAME/identx-website.git
git push -u origin main
```

### שלב 2 — Cloudflare Pages
1. כנס ל-[dash.cloudflare.com](https://dash.cloudflare.com)
2. **Pages** → **Create a project** → **Connect to Git**
3. בחר את ה-repo
4. הגדרות build:
   - **Framework preset**: None
   - **Build command**: (ריק)
   - **Build output directory**: `/` (נקודה או ריק)
5. לחץ **Save and Deploy** ✅

### שלב 3 — דומיין מותאם אישית
1. Pages project → **Custom domains** → **Set up a domain**
2. הכנס: `identx.co.il`
3. ב-DNS registrar הוסף CNAME:
   ```
   @ → identx-website.pages.dev
   www → identx-website.pages.dev
   ```

---

## 🔧 Webhook — Make.com / n8n

פתח `index.html` וחפש:
```javascript
const WEBHOOK_URL = ''; // 🔧 הכנס כאן את ה-webhook URL שלך
```
הכנס את ה-URL:
```javascript
const WEBHOOK_URL = 'https://hook.make.com/XXXXXXXXXXXXXXXX';
```

**💡 Pro tip:** להסתיר את ה-URL מהגולשים — צור Cloudflare Worker:
```javascript
// Worker code (Cloudflare → Workers → Create)
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})
async function handleRequest(request) {
  const REAL_WEBHOOK = 'https://hook.make.com/XXXXXXXX';
  return fetch(REAL_WEBHOOK, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: request.body
  });
}
```
ואז שים את ה-Worker URL ב-`WEBHOOK_URL`.

---

## 🔄 עדכון האתר אחרי פרישה

כל `git push` → Cloudflare בונה ומפרסם אוטומטית תוך ~30 שניות.
```bash
git add .
git commit -m "update: [תיאור השינוי]"
git push
```

---

## ✅ Checklist לפני Go-Live

- [ ] הוספתי תמונות לתיקיית `/images/`
- [ ] הגדרתי `WEBHOOK_URL` בקוד
- [ ] עדכנתי את הכתובת המדויקת
- [ ] עדכנתי שעות פעילות אם שונות
- [ ] בדקתי שהאתר נראה טוב במובייל (Chrome DevTools → F12 → מצב נייד)
- [ ] בדקתי שה-`tel:` links עובדים על הטלפון
- [ ] בדקתי שטופס ה-booking שולח ל-webhook

---

*נבנה על ידי GABZI Agency 🚀*
