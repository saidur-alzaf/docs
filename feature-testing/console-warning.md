## Next.js এ Console Warning Message

---

### এটা করুন `app/layout.tsx` বা একটা আলাদা component এ:

```tsx
"use client";

import { useEffect } from "react";

export default function ConsoleWarning() {
  useEffect(() => {
    console.log(
      "%c⚠ STOP!",
      "color: #ff0000; font-size: 60px; font-weight: bold;"
    );
    console.log(
      "%cThis is a browser feature intended for developers. If someone told you to copy-paste something here, it is a scam!",
      "color: #ff4444; font-size: 16px; font-weight: bold;"
    );
    console.log(
      "%c👨‍💻 If you are a developer, check out our careers: https://yoursite.com/careers",
      "color: #00ff00; font-size: 14px;"
    );
  }, []);

  return null;
}
```

`layout.tsx` এ add করুন:
```tsx
import ConsoleWarning from "@/components/ConsoleWarning";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <ConsoleWarning />
        {children}
      </body>
    </html>
  );
}
```

---

### `%c` কীভাবে কাজ করে?

```
console.log("%c text", "css styles here")
         ↑              ↑
    placeholder    এখানে CSS লিখুন
```

একাধিক style একসাথে:
```tsx
console.log(
  "%cRed Text %cBlue Text",
  "color: red;",   // প্রথম %c এর জন্য
  "color: blue;"   // দ্বিতীয় %c এর জন্য
);
```

---

### Result দেখতে এরকম হবে:

```
⚠ STOP!                          ← বড় লাল
This is a browser feature...     ← মাঝারি লাল  
👨‍💻 If you are a developer...    ← সবুজ
```

