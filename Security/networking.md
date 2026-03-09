
# Networking Security 

---

## ১. HTTP vs HTTPS

```
❌ HTTP (বিপজ্জনক):
Browser ──── "password=1234" ────► Server
         (সবাই দেখতে পাচ্ছে!)

✅ HTTPS (নিরাপদ):
Browser ──── "x7$#@!kP9q" ────► Server
         (Encrypted, কেউ বুঝতে পারবে না)
```

**Next.js-এ force HTTPS:**
```js
// next.config.js
module.exports = {
  async redirects() {
    return [
      {
        source: "/:path*",
        has: [{ type: "header", key: "x-forwarded-proto", value: "http" }],
        destination: "https://yourdomain.com/:path*",
        permanent: true,
      },
    ];
  },
};
```

---

## ২. CORS (Cross-Origin Resource Sharing)

**সমস্যা কী?**
```
evil.com থেকে কেউ আপনার API call করতে পারে!

❌ ছাড়া:
https://evil.com ──── fetch("https://yourapi.com/users") ────► ✅ Success!

✅ CORS দিলে:
https://evil.com ──── fetch("https://yourapi.com/users") ────► ❌ Blocked!
```

**Next.js API Route-এ CORS:**
```js
// app/api/data/route.ts
export async function GET(req: Request) {
  const allowedOrigins = ["https://myapp.com", "https://admin.myapp.com"];
  const origin = req.headers.get("origin") ?? "";

  const isAllowed = allowedOrigins.includes(origin);

  return new Response(JSON.stringify({ data: "secure" }), {
    headers: {
      "Access-Control-Allow-Origin": isAllowed ? origin : "null",
      "Access-Control-Allow-Methods": "GET, POST",
      "Access-Control-Allow-Headers": "Content-Type, Authorization",
    },
  });
}
```

---

## ৩. Rate Limiting (Brute Force আটকানো)

**সমস্যা কী?**
```
❌ Rate Limit ছাড়া:
হ্যাকার: /api/login এ 10,000 বার try করে password বের করে ফেলে!

✅ Rate Limit দিলে:
5 বারের বেশি try করলে → 15 মিনিট blocked!
```

**Upstash দিয়ে Rate Limiting:**
```ts
// middleware.ts
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";
import { NextRequest, NextResponse } from "next/server";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, "15 m"), // ১৫ মিনিটে ৫ বার
});

export async function middleware(req: NextRequest) {
  const ip = req.ip ?? "anonymous";
  const { success } = await ratelimit.limit(ip);

  if (!success) {
    return new NextResponse("Too Many Requests", { status: 429 });
  }
  return NextResponse.next();
}

export const config = { matcher: ["/api/login"] };
```

---

## ৪. SSL/TLS — কীভাবে কাজ করে

```
Client                          Server
  |                               |
  |──── "Hello, কথা বলব?" ──────►|
  |                               |
  |◄─── "এই আমার Certificate" ───|
  |     (Public Key সহ)           |
  |                               |
  |──── Secret Key তৈরি করে ────►|
  |     (Public Key দিয়ে encrypt) |
  |                               |
  |◄════ Encrypted Channel ══════►|
  |      (এখন নিরাপদ!)            |
```

**Certificate Pinning** — নিশ্চিত করে যে আপনি সঠিক সার্ভারের সাথেই কথা বলছেন।

---

## ৫. DNS Security

### DNS Spoofing আক্রমণ:
```
❌ Normal DNS (বিপজ্জনক):
আপনি টাইপ করলেন: mybank.com
হ্যাকার DNS পরিবর্তন করে দিল
আপনি গেলেন: fake-bank.com (হুবহু একই দেখতে!)

✅ DNSSEC দিলে:
DNS response-এ digital signature থাকে
Fake হলে ধরা পড়ে যায়
```

**Cloudflare DNS ব্যবহার করুন:**
```
Primary DNS:   1.1.1.1
Secondary DNS: 1.0.0.1
(Fast + Secure + DNSSEC supported)
```

---

## ৬. Firewall & DDoS Protection

```
❌ DDoS Attack:
১০ লক্ষ fake request ──────────────► আপনার সার্ভার DOWN!

✅ Firewall/CDN দিলে:
১০ লক্ষ fake request ──► Cloudflare Filter ──► শুধু real traffic যায়
```

**Next.js Middleware দিয়ে basic firewall:**
```ts
// middleware.ts
import { NextRequest, NextResponse } from "next/server";

// Blocked IP list
const BLOCKED_IPS = ["192.168.1.100", "10.0.0.5"];

// Blocked countries (Cloudflare header)
const BLOCKED_COUNTRIES = ["XX", "YY"];

export function middleware(req: NextRequest) {
  const ip = req.ip ?? "";
  const country = req.geo?.country ?? "";

  if (BLOCKED_IPS.includes(ip)) {
    return new NextResponse("Forbidden", { status: 403 });
  }

  if (BLOCKED_COUNTRIES.includes(country)) {
    return new NextResponse("Not Available", { status: 451 });
  }

  return NextResponse.next();
}
```

---

## ৭. Secure Cookies

```ts
// Cookie সঠিকভাবে set করুন
import { cookies } from "next/headers";

cookies().set("session", token, {
  httpOnly: true,   // JS দিয়ে access করা যাবে না
  secure: true,     // শুধু HTTPS-এ যাবে
  sameSite: "lax",  // CSRF আটকাবে
  maxAge: 60 * 60 * 24, // ২৪ ঘণ্টা
  path: "/",
});
```

| Option | কাজ |
|--------|-----|
| `httpOnly` | XSS থেকে cookie চুরি আটকায় |
| `secure` | HTTP-তে cookie যাবে না |
| `sameSite` | CSRF attack আটকায় |
| `maxAge` | নির্দিষ্ট সময় পর expire হয় |

---

## ৮. Man-in-the-Middle (MITM) Attack

```
❌ MITM Attack:
আপনি ──── data ────► [হ্যাকার পড়ছে/পরিবর্তন করছে] ────► Server

✅ সুরক্ষা:
HTTPS + HSTS Header যোগ করুন:

{ 
  key: "Strict-Transport-Security",
  value: "max-age=63072000; includeSubDomains; preload"
}
```
এই header বলে: **"এই সাইটে সবসময় HTTPS ব্যবহার করো, কখনো HTTP না"**

---

## ৯. Network Security চেকলিস্ট

```
✅ সম্পূর্ণ নিরাপত্তার জন্য:

□ HTTPS enforce করা আছে
□ HSTS header যোগ করা আছে  
□ CORS সঠিকভাবে configure করা
□ Rate Limiting চালু আছে
□ Cookies httpOnly + secure + sameSite
□ Cloudflare বা CDN ব্যবহার করছি (DDoS protection)
□ DNS হিসেবে 1.1.1.1 বা 8.8.8.8 ব্যবহার
□ API-তে Input Validation আছে
□ সব dependency আপডেট (npm audit)
□ Sensitive data log করছি না
```

---

## সহজ ভাষায় সারসংক্ষেপ

| আক্রমণ | সমাধান |
|--------|--------|
| Data চুরি | HTTPS + TLS |
| Fake সাইট | DNSSEC + SSL Certificate |
| সার্ভার বন্ধ | Cloudflare DDoS Protection |
| Password guess | Rate Limiting |
| Cookie চুরি | httpOnly + Secure Cookie |
| অন্য সাইট থেকে API call | CORS Policy |
| Traffic intercept | HSTS Header |
| Brute Force | Rate Limit + Account Lockout |

> **মূল কথা:** Networking Security মানে শুধু code নয় — **সার্ভার, DNS, CDN, Certificate** সব মিলিয়ে একটা সুরক্ষার দেওয়াল তৈরি করতে হয়। একটা দুর্বল জায়গাই হ্যাকারের জন্য যথেষ্ট!