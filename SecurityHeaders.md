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
