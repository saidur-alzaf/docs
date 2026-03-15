## Open Source Contribution — Step by Step 🔥

এটা **Pull Request (PR)** এর মাধ্যমে হয়। তুমি সরাসরি original repo তে push করতে পারবে না — PR পাঠাতে হবে।

---

### 🔄 পুরো Flow এক নজরে

```
Original Repo (upstream)
       ↓ fork
Your GitHub Repo (origin)
       ↓ clone
Your Local Machine
       ↓ code fix
       ↓ push → Your GitHub Repo
       ↓ Pull Request
Original Repo ✅
```

---

### 📋 Step-by-Step Commands

---

**① Fork করো** — GitHub এ গিয়ে "Fork" বাটন চাপো
> তোমার account এ একটা copy হবে

---

**② Clone করো তোমার forked repo**
```bash
git clone https://github.com/YOUR_USERNAME/REPO_NAME.git
cd REPO_NAME
```

---

**③ Original repo কে upstream হিসেবে add করো**
```bash
git remote add upstream https://github.com/ORIGINAL_OWNER/REPO_NAME.git

# check করো
git remote -v
# origin   → তোমার fork
# upstream → original repo
```

---

**④ নতুন branch বানাও** (সরাসরি main এ কাজ করো না!)
```bash
git checkout -b fix/issue-123
# অথবা
git checkout -b feature/add-login
```

---

**⑤ Code fix করো, তারপর commit করো**
```bash
git add .
git commit -m "fix: solve issue #123 - user login bug"
```

> 💡 **Commit message ভালো লেখো** — `fix:`, `feat:`, `docs:` দিয়ে শুরু করো

---

**⑥ তোমার forked repo তে push করো**
```bash
git push origin fix/issue-123
```

---

**⑦ Pull Request (PR) খোলো**

GitHub এ যাও → তোমার forked repo তে যাবে → একটা banner দেখবে **"Compare & pull request"** — সেটায় click করো।

```
Title: fix: solve login bug (issue #123)

Description:
## What does this PR do?
- Fixed the login bug mentioned in issue #123
- Added input validation

## How to test?
1. npm install
2. npm run test

Closes #123   ← এটা লিখলে issue auto close হয়
```

---

**⑧ PR Submit করার পর**

```
তুমি PR পাঠালে
    ↓
Maintainer review করবে
    ↓
হয়তো changes চাইবে → তুমি আবার commit করো → auto update হবে
    ↓
Approve হলে → Merge হবে ✅
```

---

### 🔄 পরের বার কাজ করার আগে — Sync করো

Original repo এ নতুন changes এসে থাকতে পারে, তাই sync রাখো:

```bash
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

---

### ⚠️ Common Mistakes যা করবে না

| ভুল | সঠিক কাজ |
|---|---|
| সরাসরি `main` branch এ কাজ করা | সবসময় নতুন branch বানাও |
| Vague commit message | `fix: user login null pointer issue #123` |
| PR এ issue number না দেওয়া | `Closes #123` লেখো |
| Sync না করে কাজ শুরু | upstream থেকে আগে pull করো |

---

### 💡 Quick Cheat Sheet

```bash
# ১. Fork → Clone
git clone https://github.com/YOUR_USERNAME/REPO.git

# ২. Upstream add
git remote add upstream https://github.com/ORIGINAL/REPO.git

# ৩. Sync (প্রতিবার শুরুর আগে)
git fetch upstream && git merge upstream/main

# ৪. Branch বানাও
git checkout -b fix/issue-123

# ৫. Code করো → Commit
git add . && git commit -m "fix: issue #123"

# ৬. Push
git push origin fix/issue-123

# ৭. GitHub এ গিয়ে Pull Request খোলো ✅
```

এখন যাও, প্রথম PR পাঠাও! 🎯 কোনো সমস্যা হলে জানাও।