# Authentication & JWT — সম্পূর্ণ গাইড (বাংলায়)

---

## প্রথমে বুঝি — JWT কী?

```
JWT = JSON Web Token

এটা একটা "ডিজিটাল পরিচয়পত্র" যা ৩ ভাগে বিভক্ত:

eyJhbGciOiJIUzI1NiJ9  ←  Header  (Algorithm কী?)
.
eyJ1c2VySWQiOiIxMjMifQ  ←  Payload  (User এর data)
.
SflKxwRJSMeKKF2QT4fwpM  ←  Signature  (এটা আসল কিনা যাচাই)
```

**JWT কীভাবে কাজ করে:**
```
1. User → Login করল (email + password)
2. Server → Password মিললে JWT Token তৈরি করল
3. Server → Token পাঠাল User-কে
4. User → পরের প্রতিটি request-এ Token পাঠায়
5. Server → Token যাচাই করে, তারপর response দেয়
```

---

## হাতে-কলমে শুরু করি

### Step 1 — Package Install
```bash
npm install next-auth@beta
npm install @auth/prisma-adapter prisma @prisma/client
npm install bcryptjs
npm install -D @types/bcryptjs
```

---

### Step 2 — Folder Structure
```
app/
├── api/
│   └── auth/
│       └── [...nextauth]/
│           └── route.ts       ← Auth API
├── login/
│   └── page.tsx               ← Login Page
├── dashboard/
│   └── page.tsx               ← Protected Page
├── middleware.ts               ← Route Protection
lib/
├── auth.ts                    ← Auth Config
├── db.ts                      ← Database
```

---

### Step 3 — Database Setup (Prisma)
```prisma
// prisma/schema.prisma
model User {
  id       String   @id @default(cuid())
  email    String   @unique
  password String
  name     String?
  role     Role     @default(USER)
  createdAt DateTime @default(now())
}

enum Role {
  USER
  ADMIN
}
```
```bash
npx prisma migrate dev --name init
```

---

### Step 4 — Auth Configuration
```ts
// lib/auth.ts
import NextAuth from "next-auth";
import Credentials from "next-auth/providers/credentials";
import Google from "next-auth/providers/google";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { prisma } from "./db";
import bcrypt from "bcryptjs";

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  
  providers: [
    // ✅ Email + Password Login
    Credentials({
      async authorize(credentials) {
        const { email, password } = credentials as {
          email: string;
          password: string;
        };

        // Database থেকে user খোঁজো
        const user = await prisma.user.findUnique({
          where: { email },
        });

        if (!user) throw new Error("User not found!");

        // Password মেলাও
        const isValid = await bcrypt.compare(password, user.password);
        if (!isValid) throw new Error("Wrong password!");

        return user; // ✅ Login সফল
      },
    }),

    // ✅ Google Login
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],

  session: { strategy: "jwt" }, // JWT use করব

  callbacks: {
    // Token-এ extra data যোগ করি
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
        token.role = user.role;
      }
      return token;
    },

    // Session-এ সেই data পাঠাই
    async session({ session, token }) {
      session.user.id = token.id as string;
      session.user.role = token.role as string;
      return session;
    },
  },

  pages: {
    signIn: "/login", // Custom login page
  },
});
```

---

### Step 5 — API Route
```ts
// app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/lib/auth";
export const { GET, POST } = handlers;
```

---

### Step 6 — Register API
```ts
// app/api/register/route.ts
import { prisma } from "@/lib/db";
import bcrypt from "bcryptjs";
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const { name, email, password } = await req.json();

  // ইতিমধ্যে আছে কিনা চেক
  const existing = await prisma.user.findUnique({ where: { email } });
  if (existing) {
    return NextResponse.json(
      { error: "Email already exists!" },
      { status: 400 }
    );
  }

  // Password hash করো (কখনো plain text সেভ করবে না!)
  const hashedPassword = await bcrypt.hash(password, 12);

  const user = await prisma.user.create({
    data: { name, email, password: hashedPassword },
  });

  return NextResponse.json({ message: "Account created!", userId: user.id });
}
```

---

### Step 7 — Login Page
```tsx
// app/login/page.tsx
"use client";
import { signIn } from "next-auth/react";
import { useState } from "react";
import { useRouter } from "next/navigation";

export default function LoginPage() {
  const router = useRouter();
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setLoading(true);
    const form = e.currentTarget;

    const result = await signIn("credentials", {
      email: form.email.value,
      password: form.password.value,
      redirect: false, // নিজে redirect করব
    });

    if (result?.error) {
      setError("Email অথবা Password ভুল!");
    } else {
      router.push("/dashboard"); // ✅ সফল হলে dashboard-এ যাও
    }
    setLoading(false);
  }

  return (
    <div className="flex items-center justify-center min-h-screen">
      <form onSubmit={handleSubmit} className="space-y-4 w-80">
        <h1 className="text-2xl font-bold">Login</h1>

        {error && <p className="text-red-500">{error}</p>}

        <input name="email" type="email" placeholder="Email" required
          className="w-full border p-2 rounded" />

        <input name="password" type="password" placeholder="Password" required
          className="w-full border p-2 rounded" />

        <button type="submit" disabled={loading}
          className="w-full bg-blue-600 text-white p-2 rounded">
          {loading ? "Loading..." : "Login"}
        </button>

        {/* Google Login */}
        <button type="button" onClick={() => signIn("google")}
          className="w-full border p-2 rounded flex items-center justify-center gap-2">
          Google দিয়ে Login
        </button>
      </form>
    </div>
  );
}
```

---

### Step 8 — Protected Page
```tsx
// app/dashboard/page.tsx
import { auth } from "@/lib/auth";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const session = await auth();

  // Login না থাকলে পাঠিয়ে দাও
  if (!session) redirect("/login");

  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold">
        স্বাগতম, {session.user.name}! 👋
      </h1>
      <p>Email: {session.user.email}</p>
      <p>Role: {session.user.role}</p>
    </div>
  );
}
```

---

### Step 9 — Middleware (সব Route রক্ষা করো)
```ts
// middleware.ts
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export default auth((req) => {
  const isLoggedIn = !!req.auth;
  const isProtected = req.nextUrl.pathname.startsWith("/dashboard");
  const isAdminRoute = req.nextUrl.pathname.startsWith("/admin");

  // Login না থাকলে redirect
  if (isProtected && !isLoggedIn) {
    return NextResponse.redirect(new URL("/login", req.url));
  }

  // Admin না হলে block
  if (isAdminRoute && req.auth?.user?.role !== "ADMIN") {
    return NextResponse.redirect(new URL("/unauthorized", req.url));
  }

  return NextResponse.next();
});

export const config = {
  matcher: ["/dashboard/:path*", "/admin/:path*"],
};
```

---

### Step 10 — Environment Variables
```bash
# .env.local
DATABASE_URL="postgresql://..."

NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-super-secret-key-here"  # openssl rand -base64 32

GOOGLE_CLIENT_ID="..."
GOOGLE_CLIENT_SECRET="..."
```

---

## সম্পূর্ণ Flow একনজরে

```
Register:
User ──► /api/register ──► Password Hash ──► Database Save

Login:
User ──► /login ──► Credentials Check ──► JWT Token তৈরি ──► Cookie-তে Save

Protected Route:
Request ──► middleware.ts ──► Token যাচাই ──► ✅ Allow / ❌ Redirect

API Call:
fetch('/api/data') ──► Token পাঠাও ──► Server যাচাই করে ──► Response
```

---

## Security Best Practices

| ✅ করবেন | ❌ করবেন না |
|---------|------------|
| bcrypt দিয়ে hash করুন | Plain text password সেভ করবেন না |
| httpOnly cookie ব্যবহার করুন | localStorage-এ token রাখবেন না |
| Token expiry সেট করুন | Token কখনো expire না করা |
| Server-side session check করুন | শুধু client-side check করবেন না |
| HTTPS ব্যবহার করুন | HTTP-তে token পাঠাবেন না |

---
