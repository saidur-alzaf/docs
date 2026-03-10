# SEO in Next.js — সম্পূর্ণ গাইড (বাংলায়)

---

## SEO কী এবং কেন দরকার?

```
SEO = Search Engine Optimization

আপনি লিখলেন: "best pizza in dhaka"
Google কীভাবে সিদ্ধান্ত নেয় কোন সাইট আগে দেখাবে?
→ SEO ভালো হলে আপনার সাইট উপরে, খারাপ হলে ১০ পেজে!
```

---

## ১. Metadata — সবচেয়ে গুরুত্বপূর্ণ

### Static Metadata
```ts
// app/page.tsx
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "আমার অ্যাপ — Best Service in Bangladesh",
  description: "আমরা সেরা সেবা দিই। ১০,০০০+ সন্তুষ্ট গ্রাহক।",
  keywords: ["next.js", "bangladesh", "service"],
  authors: [{ name: "Your Name" }],
  
  // Open Graph (Facebook, LinkedIn share preview)
  openGraph: {
    title: "আমার অ্যাপ",
    description: "সেরা সেবা",
    url: "https://myapp.com",
    siteName: "MyApp",
    images: [
      {
        url: "https://myapp.com/og-image.jpg", // 1200x630px
        width: 1200,
        height: 630,
        alt: "MyApp Preview",
      },
    ],
    type: "website",
  },

  // Twitter Card
  twitter: {
    card: "summary_large_image",
    title: "আমার অ্যাপ",
    description: "সেরা সেবা",
    images: ["https://myapp.com/og-image.jpg"],
  },

  // Canonical URL (duplicate content আটকায়)
  alternates: {
    canonical: "https://myapp.com",
  },
};
```

### Dynamic Metadata (প্রতিটি page-এর জন্য আলাদা)
```ts
// app/products/[id]/page.tsx
import { Metadata } from "next";

type Props = { params: { id: string } };

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  // Database থেকে product আনো
  const product = await fetch(`/api/products/${params.id}`).then(r => r.json());

  return {
    title: `${product.name} — MyShop`,
    description: product.description,
    openGraph: {
      title: product.name,
      images: [product.image],
    },
  };
}
```

---

## ২. Layout-এ Default Metadata

```ts
// app/layout.tsx
export const metadata: Metadata = {
  // %s = child page-এর title
  title: {
    default: "MyApp",
    template: "%s | MyApp", // → "Products | MyApp"
  },
  description: "Default description",
  
  // Favicon
  icons: {
    icon: "/favicon.ico",
    apple: "/apple-icon.png",
  },

  // Robot instructions
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
    },
  },
};
```

---

## ৩. Sitemap — Google কে সাইট চেনাও

```ts
// app/sitemap.ts
import { MetadataRoute } from "next";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  // Static pages
  const staticPages = [
    { url: "https://myapp.com", lastModified: new Date(), priority: 1 },
    { url: "https://myapp.com/about", lastModified: new Date(), priority: 0.8 },
    { url: "https://myapp.com/contact", lastModified: new Date(), priority: 0.5 },
  ];

  // Dynamic pages (database থেকে)
  const products = await fetch("https://myapp.com/api/products").then(r => r.json());
  const productPages = products.map((p: any) => ({
    url: `https://myapp.com/products/${p.id}`,
    lastModified: new Date(p.updatedAt),
    priority: 0.7,
  }));

  return [...staticPages, ...productPages];
}

// ✅ এখন: https://myapp.com/sitemap.xml এ automatically পাওয়া যাবে
```

---

## ৪. Robots.txt

```ts
// app/robots.ts
import { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: "*",
        allow: "/",
        disallow: ["/admin", "/api", "/private"], // এগুলো index করবে না
      },
    ],
    sitemap: "https://myapp.com/sitemap.xml",
  };
}
```

---

## ৫. Structured Data (JSON-LD) — Rich Results

Google-এ star rating, price, breadcrumb দেখানোর জন্য:

```tsx
// app/products/[id]/page.tsx
export default async function ProductPage({ params }) {
  const product = await getProduct(params.id);

  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "Product",
    name: product.name,
    description: product.description,
    image: product.image,
    offers: {
      "@type": "Offer",
      price: product.price,
      priceCurrency: "BDT",
      availability: "https://schema.org/InStock",
    },
    aggregateRating: {
      "@type": "AggregateRating",
      ratingValue: product.rating,
      reviewCount: product.reviewCount,
    },
  };

  return (
    <>
      {/* JSON-LD inject করো */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <div>{product.name}</div>
    </>
  );
}
```

**Google-এ এভাবে দেখাবে:**
```
MyProduct — MyShop
★★★★☆ (4.2) · 234 reviews · ৳1,200
Best product description here...
```

---

## ৬. Performance = SEO (Core Web Vitals)

Google rank করে performance দিয়েও!

```
LCP (Largest Contentful Paint) → পেজের বড় element লোড সময়  → < 2.5s
FID (First Input Delay)        → প্রথম click এ response     → < 100ms
CLS (Cumulative Layout Shift)  → layout কতটা নড়ে          → < 0.1
```

**Next.js-এ Image Optimization:**
```tsx
import Image from "next/image";

// ✅ SEO-friendly image
<Image
  src="/hero.jpg"
  alt="Hero Image — এই alt text Google পড়ে!"  // ← অবশ্যই দিন
  width={1200}
  height={630}
  priority    // ← LCP image হলে এটা দিন
  placeholder="blur"
/>
```

**Font Optimization:**
```ts
// app/layout.tsx
import { Inter } from "next/font/google";

const font = Inter({
  subsets: ["latin"],
  display: "swap", // ← CLS কমায়
});
```

---

## ৭. SEO-friendly URL Structure

```
❌ খারাপ:
/p?id=123
/page/1/2/3/abc

✅ ভালো:
/products/samsung-galaxy-s24
/blog/how-to-learn-nextjs-2025
/categories/electronics/phones
```

```ts
// app/blog/[slug]/page.tsx  ← slug = seo-friendly URL
export async function generateStaticParams() {
  const posts = await getAllPosts();
  return posts.map((post) => ({ slug: post.slug }));
}
```

---

## ৮. Canonical & Hreflang (বাংলা সাইটের জন্য)

```ts
export const metadata: Metadata = {
  alternates: {
    canonical: "https://myapp.com/products/phone",
    // বাংলা ও ইংরেজি দুই ভাষা থাকলে:
    languages: {
      "en-US": "https://myapp.com/en/products/phone",
      "bn-BD": "https://myapp.com/bn/products/phone",
    },
  },
};
```

---

## SEO Checklist ✅

```
□ প্রতিটি page-এ unique title + description
□ Open Graph image (1200x630px)
□ sitemap.xml আছে ও Google Search Console-এ submit করা
□ robots.txt সঠিক
□ সব image-এ alt text
□ Mobile responsive (Google mobile-first index করে)
□ HTTPS চালু
□ Page speed ভালো (Lighthouse score 90+)
□ Structured Data (JSON-LD) আছে
□ Canonical URL set করা
□ Internal linking ভালো
```

---

## Tools যা ব্যবহার করবেন

| Tool | কাজ |
|------|-----|
| [Google Search Console](https://search.google.com/search-console) | Index status, errors দেখুন |
| [PageSpeed Insights](https://pagespeed.web.dev/) | Performance check |
| [Rich Results Test](https://search.google.com/test/rich-results) | JSON-LD ঠিক আছে কিনা |
| [OG Image Preview](https://www.opengraph.xyz/) | Social share preview দেখুন |

