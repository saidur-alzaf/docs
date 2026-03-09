Sound আর instant (real-time) notification — সম্পূর্ণ গাইড
---
এটা ২টা আলাদা জিনিস মিলিয়ে করতে হবে:

```
Real-time = WebSocket / Pusher / Firebase  (server থেকে instant পাঠায়)
Sound     = use-sound / Web Audio API      (notification-এ শব্দ যোগ করে)
Toast UI  = Sonner                         (সুন্দর করে দেখায়)
```

---

## 🏆 Best Combination — Sonner + use-sound + Pusher

```bash
npm i sonner use-sound pusher-js
```

---

## Step 1 — Sound যোগ করো

```ts
// hooks/useNotification.ts
import { toast } from "sonner";
import useSound from "use-sound";
import notifSound from "@/public/sounds/notification.mp3";

export function useNotification() {
  const [play] = useSound(notifSound, { volume: 0.5 });

  const notify = {
    success: (msg: string) => {
      play(); // 🔊 Sound বাজাও
      toast.success(msg);
    },
    error: (msg: string) => {
      play();
      toast.error(msg);
    },
    message: (title: string, body: string) => {
      play();
      toast(title, {
        description: body,
        icon: "💬",
        duration: 5000,
      });
    },
  };

  return notify;
}
```

---

## Step 2 — Real-time (Pusher দিয়ে)

```bash
npm i pusher pusher-js
```

```ts
// hooks/useRealTimeNotification.ts
"use client";
import { useEffect } from "react";
import Pusher from "pusher-js";
import { useNotification } from "./useNotification";

export function useRealTimeNotification(userId: string) {
  const notify = useNotification();

  useEffect(() => {
    // Pusher connect করো
    const pusher = new Pusher(process.env.NEXT_PUBLIC_PUSHER_KEY!, {
      cluster: "ap2",
    });

    // User-এর channel-এ subscribe করো
    const channel = pusher.subscribe(`user-${userId}`);

    // নতুন message এলে
    channel.bind("new-message", (data: { title: string; body: string }) => {
      notify.message(data.title, data.body); // 🔊 Sound + Toast একসাথে!
    });

    // নতুন order এলে
    channel.bind("new-order", (data: { amount: number }) => {
      notify.success(`নতুন অর্ডার! ৳${data.amount}`);
    });

    return () => {
      channel.unbind_all();
      pusher.unsubscribe(`user-${userId}`);
    };
  }, [userId]);
}
```

---

## Step 3 — Server থেকে Notification পাঠাও

```ts
// app/api/notify/route.ts
import Pusher from "pusher";

const pusher = new Pusher({
  appId: process.env.PUSHER_APP_ID!,
  key: process.env.PUSHER_KEY!,
  secret: process.env.PUSHER_SECRET!,
  cluster: "ap2",
  useTLS: true,
});

export async function POST(req: Request) {
  const { userId, title, body } = await req.json();

  // Real-time পাঠাও → সাথে সাথে client-এ পৌঁছাবে!
  await pusher.trigger(`user-${userId}`, "new-message", { title, body });

  return Response.json({ sent: true });
}
```

---

## Step 4 — Layout-এ Setup করো

```tsx
// app/layout.tsx
import { Toaster } from "sonner";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Toaster
          position="top-right"
          richColors
          expand={true}
          visibleToasts={5}
        />
      </body>
    </html>
  );
}
```

```tsx
// app/dashboard/page.tsx
"use client";
import { useRealTimeNotification } from "@/hooks/useRealTimeNotification";

export default function Dashboard() {
  useRealTimeNotification("user-123"); // ✅ Real-time চালু!

  return <div>Dashboard</div>;
}
```

---

## Browser Push Notification (ব্রাউজার বন্ধ থাকলেও!)

```ts
// lib/push-notification.ts
export async function requestPermission() {
  const permission = await Notification.requestPermission();
  return permission === "granted";
}

export function sendBrowserNotification(title: string, body: string) {
  if (Notification.permission === "granted") {
    new Notification(title, {
      body,
      icon: "/icon.png",
      badge: "/badge.png",
      // sound সরাসরি support নেই, কিন্তু Service Worker দিয়ে করা যায়
    });
  }
}
```

---

## সব Option তুলনা

| Solution | Real-time | Sound | Browser বন্ধেও | কঠিনতা |
|---------|-----------|-------|----------------|---------|
| **Sonner + use-sound + Pusher** | ✅ | ✅ | ❌ | 🟡 মাঝারি |
| **Novu** | ✅ | ✅ | ✅ | 🟢 সহজ |
| **Firebase FCM** | ✅ | ✅ | ✅ | 🔴 কঠিন |
| **Socket.io** | ✅ | ✅ | ❌ | 🔴 কঠিন |

---

## 🏆 Recommendation

| আপনার project | ব্যবহার করুন |
|--------------|-------------|
| ছোট app (chat, dashboard) | **Sonner + use-sound + Pusher** |
| বড় app (multi-channel) | **[Novu](https://novu.co)** — email + SMS + push সব একসাথে |
| Mobile-like push | **Firebase FCM** |

> **সবচেয়ে সহজ শুরু:** `Sonner + use-sound` দিয়ে শুরু করুন, পরে Pusher যোগ করুন 🚀
