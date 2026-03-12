```
"use client";

import { boom } from "boom-toaster";
import { useState, useRef, useEffect } from "react";

type ToastPosition =
  | "top-left"
  | "top-center"
  | "top-right"
  | "bottom-left"
  | "bottom-center"
  | "bottom-right";
type ToastType = "success" | "error" | "warning" | "info" | "loading";

// ─── Global styles ────────────────────────────────────────────────────────────
const styles = `
  @import url('https://fonts.googleapis.com/css2?family=Syne:wght@700;800&family=DM+Mono:wght@400;500&family=DM+Sans:wght@300;400;500&display=swap');

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg:      #060608;
    --surface: #0e0e14;
    --border:  rgba(255,255,255,0.07);
    --accent1: #7c6bff;
    --accent2: #ff5caa;
    --accent3: #00e5c8;
    --text:    #e8e8f0;
    --muted:   #6b6b80;
    --glow1:   rgba(124,107,255,0.35);
    --glow2:   rgba(255,92,170,0.35);
  }

  html { scroll-behavior: smooth; }
  body {
    background: var(--bg); color: var(--text);
    font-family: 'DM Sans', sans-serif;
    -webkit-font-smoothing: antialiased; overflow-x: hidden;
  }

  /* ── Blobs ── */
  .blob-wrap { position: fixed; inset: 0; pointer-events: none; z-index: 0; overflow: hidden; }
  .blob { position: absolute; border-radius: 50%; filter: blur(110px); opacity: 0.15; animation: blobFloat 14s ease-in-out infinite alternate; }
  .blob-a { width: 520px; height: 520px; background: var(--accent1); top: -180px; left: -120px; animation-delay: 0s; }
  .blob-b { width: 420px; height: 420px; background: var(--accent2); top: 30%; right: -150px; animation-delay: -5s; }
  .blob-c { width: 360px; height: 360px; background: var(--accent3); bottom: 5%; left: 20%; animation-delay: -9s; }
  @keyframes blobFloat {
    0%   { transform: translate(0,0) scale(1); }
    50%  { transform: translate(40px,-60px) scale(1.08); }
    100% { transform: translate(-30px,40px) scale(0.95); }
  }

  /* ── Grain ── */
  body::after {
    content: ''; position: fixed; inset: 0;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
    background-repeat: repeat; background-size: 180px;
    pointer-events: none; z-index: 9999; opacity: 0.5;
  }

  .font-display { font-family: 'Syne', sans-serif; }
  .font-mono    { font-family: 'DM Mono', monospace; }

  /* ── Shimmer ── */
  .shimmer-text {
    background: linear-gradient(100deg, var(--accent1) 0%, var(--accent2) 35%, var(--accent3) 60%, var(--accent1) 100%);
    background-size: 200% auto;
    -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
    animation: shimmer 4s linear infinite;
  }
  @keyframes shimmer { from { background-position: 0% center; } to { background-position: 200% center; } }

  /* ── Hero ── */
  .hero { position: relative; padding: 140px 20px 100px; text-align: center; z-index: 1; }
  .hero-badge {
    display: inline-flex; align-items: center; gap: 6px;
    padding: 6px 16px; border-radius: 999px;
    border: 1px solid rgba(124,107,255,0.4); background: rgba(124,107,255,0.08);
    color: var(--accent1); font-size: 0.78rem; letter-spacing: 0.12em;
    text-transform: uppercase; font-weight: 500; margin-bottom: 28px;
    animation: fadeUp 0.8s ease both;
  }
  .hero-title {
    font-size: clamp(3rem, 10vw, 7.5rem); line-height: 1;
    font-weight: 800; letter-spacing: -0.03em;
    margin-bottom: 20px; animation: fadeUp 0.9s 0.1s ease both;
  }
  @media (max-width: 680px) { .hero-title { font-size: clamp(1rem, 8vw, 6.5rem); } }
  .hero-sub {
    font-size: clamp(0.95rem, 2.5vw, 1.2rem); color: var(--muted);
    max-width: 500px; margin: 0 auto 40px; line-height: 1.7;
    font-weight: 300; animation: fadeUp 1s 0.2s ease both;
  }
  .hero-actions {
    display: flex; gap: 12px; justify-content: center;
    flex-wrap: wrap; animation: fadeUp 1s 0.3s ease both;
  }

  /* ── Buttons ── */
  .btn-primary {
    display: inline-flex; align-items: center; gap: 8px;
    padding: 13px 28px; border-radius: 14px;
    background: linear-gradient(135deg, var(--accent1), var(--accent2));
    color: #fff; font-weight: 600; font-size: 0.92rem;
    border: none; cursor: pointer; position: relative; overflow: hidden;
    transition: transform 0.2s, box-shadow 0.2s;
    box-shadow: 0 0 28px var(--glow1); font-family: 'DM Sans', sans-serif;
  }
  .btn-primary::before {
    content:''; position:absolute; inset:0;
    background: linear-gradient(135deg, var(--accent2), var(--accent1));
    opacity:0; transition:opacity 0.3s;
  }
  .btn-primary:hover { transform: translateY(-2px); box-shadow: 0 0 44px var(--glow2); }
  .btn-primary:hover::before { opacity:1; }
  .btn-primary span { position:relative; z-index:1; }

  .btn-ghost {
    display: inline-flex; align-items: center; gap: 8px;
    padding: 12px 24px; border-radius: 14px;
    border: 1px solid var(--border); background: rgba(255,255,255,0.03);
    color: var(--text); font-size: 0.92rem; font-weight: 500;
    cursor: pointer; text-decoration: none;
    transition: border-color 0.2s, background 0.2s, transform 0.2s;
  }
  .btn-ghost:hover { border-color: rgba(124,107,255,0.5); background: rgba(124,107,255,0.08); transform: translateY(-2px); }

  /* ── Sections ── */
  .section { position: relative; z-index: 1; padding: 80px 20px; }
  .section-title {
    font-size: clamp(1.8rem, 4vw, 2.8rem); font-weight: 800;
    text-align: center; margin-bottom: 52px; letter-spacing: -0.02em;
  }
  .divider {
    height: 1px; max-width: 860px; margin: 0 auto;
    background: linear-gradient(90deg, transparent, var(--border) 20%, var(--accent1) 50%, var(--border) 80%, transparent);
  }
  .bg-alt { background: linear-gradient(180deg, transparent, rgba(124,107,255,0.025) 30%, rgba(255,92,170,0.025) 70%, transparent); }

  /* ── Playground ── */
  .playground-card {
    max-width: 860px; min-height: 200px; margin: 0 auto;
    background: var(--surface); border: 1px solid var(--border);
    border-radius: 22px; padding: 40px; position: relative; overflow: visible;
    transition: border-color 0.3s, box-shadow 0.3s;
  }
  /* Top shimmer line via box-shadow instead of ::before, so overflow:visible works */
  .playground-card::before {
    content:''; position:absolute; top:0; left:10%; right:10%; height:1px;
    background: linear-gradient(90deg, transparent, var(--accent1) 40%, var(--accent2) 60%, transparent);
    opacity:0.6; border-radius: 1px;
    pointer-events: none;
  }
  .playground-card:hover { border-color: rgba(124,107,255,0.25); box-shadow: 0 0 50px rgba(124,107,255,0.07); }

  .playground-grid { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 20px; }
  @media (max-width: 680px) {
    .playground-grid { grid-template-columns: 1fr; }
    .playground-card { padding: 22px 16px; }
  }

  .field-label {
    font-size: 0.72rem; color: var(--muted); text-transform: uppercase;
    letter-spacing: 0.1em; margin-bottom: 8px; display: block;
  }
  .field-input {
    width: 100%; padding: 11px 14px; border-radius: 11px;
    background: rgba(255,255,255,0.04); border: 1px solid var(--border);
    color: var(--text); font-family: 'DM Sans', sans-serif; font-size: 0.88rem;
    transition: border-color 0.2s, box-shadow 0.2s; outline: none;
  }
  .field-input:focus { border-color: rgba(124,107,255,0.5); box-shadow: 0 0 0 3px rgba(124,107,255,0.1); }

  /* ── Custom Select ── */
  .cs-wrap { position: relative; user-select: none; }
  .cs-trigger {
    width: 100%; padding: 11px 14px; border-radius: 11px;
    background: rgba(255,255,255,0.04); border: 1px solid var(--border);
    color: var(--text); font-family: 'DM Sans', sans-serif; font-size: 0.88rem;
    cursor: pointer; display: flex; align-items: center; justify-content: space-between;
    transition: border-color 0.2s, box-shadow 0.2s, background 0.2s;
  }
  .cs-trigger:hover, .cs-trigger.open {
    border-color: rgba(124,107,255,0.5);
    background: rgba(124,107,255,0.06);
    box-shadow: 0 0 0 3px rgba(124,107,255,0.1);
  }
  .cs-arrow { display: flex; align-items: center; color: var(--muted); transition: transform 0.2s, color 0.2s; }
  .cs-arrow.open { transform: rotate(180deg); color: var(--accent1); }

  .cs-menu {
    position: absolute; top: calc(100% + 6px); left: 0; right: 0; z-index: 200;
    background: #12121c; border: 1px solid rgba(124,107,255,0.22);
    border-radius: 12px; overflow: hidden;
    box-shadow: 0 16px 48px rgba(0,0,0,0.65), 0 0 0 1px rgba(124,107,255,0.06);
    animation: menuOpen 0.14s ease both;
  }
  @keyframes menuOpen {
    from { opacity:0; transform: translateY(-6px) scale(0.98); }
    to   { opacity:1; transform: translateY(0) scale(1); }
  }
  .cs-option {
    padding: 10px 14px; font-size: 0.87rem; cursor: pointer;
    display: flex; align-items: center; gap: 10px;
    transition: background 0.13s, color 0.13s; color: var(--muted);
  }
  .cs-option:hover { background: rgba(124,107,255,0.1); color: var(--text); }
  .cs-option.sel   { background: rgba(124,107,255,0.14); color: var(--accent1); font-weight: 500; }
  .cs-dot { width: 6px; height: 6px; border-radius: 50%; background: currentColor; opacity: 0.5; flex-shrink: 0; }
  .cs-option.sel .cs-dot { opacity: 1; background: var(--accent1); }
  .cs-icon { font-size: 1rem; line-height: 1; width: 20px; text-align: center; }

  .run-btn-wrap { text-align: center; margin-top: 36px; }

  /* ── Features ── */
  .features-grid { display: grid; grid-template-columns: repeat(3,1fr); gap: 18px; max-width: 1060px; margin: 0 auto; }
  @media (max-width: 860px) { .features-grid { grid-template-columns: repeat(2,1fr); } }
  @media (max-width: 520px) { .features-grid { grid-template-columns: 1fr; } }

  .feature-card {
    background: var(--surface); border: 1px solid var(--border);
    border-radius: 18px; padding: 28px; overflow: hidden;
    transition: border-color 0.3s, transform 0.3s, box-shadow 0.3s;
  }
  .feature-card:hover { border-color: rgba(124,107,255,0.28); transform: translateY(-4px); box-shadow: 0 20px 44px rgba(0,0,0,0.4); }
  .feature-icon {
    width: 42px; height: 42px; border-radius: 11px;
    background: linear-gradient(135deg, rgba(124,107,255,0.18), rgba(255,92,170,0.1));
    border: 1px solid rgba(124,107,255,0.2);
    display: flex; align-items: center; justify-content: center;
    font-size: 1.1rem; margin-bottom: 16px;
  }
  .feature-title { font-weight: 700; font-size: 1rem; margin-bottom: 7px; }
  .feature-desc  { color: var(--muted); font-size: 0.87rem; line-height: 1.6; }

  /* ── Install ── */
  .install-wrap { max-width: 740px; margin: 0 auto; }
  .install-cmd {
    background: var(--surface); border: 1px solid var(--border);
    border-radius: 14px; padding: 17px 22px;
    font-family: 'DM Mono', monospace; font-size: 0.93rem;
    color: var(--accent3); display: flex; align-items: center; gap: 10px;
    margin-bottom: 16px; overflow-x: auto;
  }
  .install-cmd::before { content:'$'; color: var(--muted); }

  /* ── Code blocks ── */
  .code-card {
    position: relative; background: var(--surface);
    border: 1px solid var(--border); border-radius: 15px; overflow: hidden;
    transition: border-color 0.2s;
  }
  .code-card:hover { border-color: rgba(124,107,255,0.22); }
  .code-header {
    display: flex; align-items: center; gap: 6px;
    padding: 11px 16px; border-bottom: 1px solid var(--border);
    background: rgba(255,255,255,0.02);
  }
  .dot { width: 10px; height: 10px; border-radius: 50%; }
  .dot-r { background: #ff5f57; } .dot-y { background: #febc2e; } .dot-g { background: #28c840; }
  .code-body {
    padding: 20px; overflow-x: auto;
    font-family: 'DM Mono', monospace; font-size: 0.83rem;
    line-height: 1.7; color: #bdbdd8;
  }
  .copy-btn {
    position: absolute; top: 8px; right: 12px;
    padding: 4px 12px; border-radius: 7px;
    background: rgba(124,107,255,0.12); border: 1px solid rgba(124,107,255,0.26);
    color: var(--accent1); font-size: 0.75rem; font-family: 'DM Sans', sans-serif;
    cursor: pointer; transition: background 0.2s, color 0.2s, border-color 0.2s; z-index: 10;
  }
  .copy-btn:hover  { background: rgba(124,107,255,0.28); color: #fff; }
  .copy-btn.copied { background: rgba(0,229,200,0.1); border-color: var(--accent3); color: var(--accent3); }

  /* ── Footer ── */
  .site-footer {
    position: relative; z-index: 1; padding: 36px 20px;
    text-align: center; color: var(--muted); font-size: 0.86rem;
    border-top: 1px solid var(--border);
  }
  .site-footer strong { color: var(--accent2); }

  /* ── Animations ── */
  @keyframes fadeUp {
    from { opacity:0; transform: translateY(20px); }
    to   { opacity:1; transform: translateY(0); }
  }
  .stagger-1 { animation: fadeUp 0.65s 0.05s ease both; }
  .stagger-2 { animation: fadeUp 0.65s 0.15s ease both; }
  .stagger-3 { animation: fadeUp 0.65s 0.25s ease both; }
  .stagger-4 { animation: fadeUp 0.65s 0.35s ease both; }
  .stagger-5 { animation: fadeUp 0.65s 0.45s ease both; }
  .stagger-6 { animation: fadeUp 0.65s 0.55s ease both; }

  /* ── Ping ── */
  @keyframes ping { 75%,100% { transform:scale(2); opacity:0; } }
  .ping-dot { position:relative; display:inline-flex; align-items:center; justify-content:center; width:8px; height:8px; }
  .ping-dot::before { content:''; position:absolute; inset:0; border-radius:50%; background:var(--accent3); animation:ping 1.4s ease-in-out infinite; }
  .ping-dot::after  { content:''; width:8px; height:8px; border-radius:50%; background:var(--accent3); position:relative; z-index:1; }

  /* ── Scrollbars ── */
  ::-webkit-scrollbar { width:5px; height:5px; }
  ::-webkit-scrollbar-track { background:transparent; }
  ::-webkit-scrollbar-thumb { background:rgba(255,255,255,0.1); border-radius:99px; }
  .hidden-scrollbar::-webkit-scrollbar { display:none; }
  .hidden-scrollbar { -ms-overflow-style:none; scrollbar-width:none; }

  /* ── Mobile ── */
  @media (max-width: 480px) {
    .hero { padding: 100px 16px 70px; }
    .section { padding: 56px 16px; }
    .hero-actions { flex-direction: column; align-items: stretch; }
    .btn-primary, .btn-ghost { width: 100%; justify-content: center; }
    .install-cmd { font-size: 0.8rem; }
    .code-body { font-size: 0.78rem; padding: 16px; }
  }
`;

// ─── Style injector ───────────────────────────────────────────────────────────
function StyleInjector() {
  if (typeof document !== "undefined") {
    if (!document.getElementById("boom-styles")) {
      const el = document.createElement("style");
      el.id = "boom-styles";
      el.textContent = styles;
      document.head.appendChild(el);
    }
  }
  return null;
}

// ─── Safe clipboard copy (with textarea fallback) ────────────────────────────
function safeCopy(text: string): Promise<void> {
  if (
    typeof navigator !== "undefined" &&
    navigator.clipboard &&
    typeof navigator.clipboard.writeText === "function"
  ) {
    return navigator.clipboard.writeText(text);
  }
  return new Promise((resolve, reject) => {
    try {
      const ta = document.createElement("textarea");
      ta.value = text;
      ta.style.cssText = "position:fixed;top:-9999px;left:-9999px;opacity:0";
      document.body.appendChild(ta);
      ta.focus();
      ta.select();
      const ok = document.execCommand("copy");
      document.body.removeChild(ta);
      ok ? resolve() : reject(new Error("execCommand copy failed"));
    } catch (e) {
      reject(e);
    }
  });
}

// ─── Metadata for custom selects ─────────────────────────────────────────────
const TYPE_META: Record<string, { icon: string; label: string }> = {
  success: { icon: "", label: "Success" },
  error: { icon: "", label: "Error" },
  warning: { icon: "", label: "Warning" },
  info: { icon: "", label: "Info" },
  loading: { icon: "", label: "Loading" },
};

const POS_META: Record<string, { icon: string; label: string }> = {
  "top-right": { icon: "↗", label: "Top Right" },
  "top-left": { icon: "↖", label: "Top Left" },
  "top-center": { icon: "⬆", label: "Top Center" },
  "bottom-right": { icon: "↘", label: "Bottom Right" },
  "bottom-left": { icon: "↙", label: "Bottom Left" },
  "bottom-center": { icon: "⬇", label: "Bottom Center" },
};

// ─── Custom Select component ──────────────────────────────────────────────────
function CustomSelect({
  label,
  value,
  onChange,
  options,
  meta,
}: {
  label: string;
  value: string;
  onChange: (v: string) => void;
  options: string[];
  meta: Record<string, { icon: string; label: string }>;
}) {
  const [open, setOpen] = useState(false);
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    function onOutside(e: MouseEvent) {
      if (ref.current && !ref.current.contains(e.target as Node))
        setOpen(false);
    }
    document.addEventListener("mousedown", onOutside);
    return () => document.removeEventListener("mousedown", onOutside);
  }, []);

  const current = meta[value];

  return (
    <div>
      <label className="field-label">{label}</label>
      <div className="cs-wrap" ref={ref}>
        <div
          className={`cs-trigger${open ? " open" : ""}`}
          onClick={() => setOpen((o) => !o)}
        >
          <span style={{ display: "flex", alignItems: "center", gap: 8 }}>
            <span className="cs-icon">{current?.icon}</span>
            <span>{current?.label ?? value}</span>
          </span>
          <span className={`cs-arrow${open ? " open" : ""}`}>
            <svg width="13" height="13" viewBox="0 0 13 13" fill="none">
              <path
                d="M2.5 4.5l4 4 4-4"
                stroke="currentColor"
                strokeWidth="1.5"
                strokeLinecap="round"
                strokeLinejoin="round"
              />
            </svg>
          </span>
        </div>

        {open && (
          <div className="cs-menu">
            {options.map((o) => (
              <div
                key={o}
                className={`cs-option${value === o ? " sel" : ""}`}
                onClick={() => {
                  onChange(o);
                  setOpen(false);
                }}
              >
                <span className="cs-dot" />
                <span className="cs-icon">{meta[o]?.icon}</span>
                <span>{meta[o]?.label ?? o}</span>
              </div>
            ))}
          </div>
        )}
      </div>
    </div>
  );
}

// ─── Main page ────────────────────────────────────────────────────────────────
export default function ToastTest() {
  const [type, setType] = useState<ToastType>("success");
  const [position, setPosition] = useState<ToastPosition>("top-right");
  const [duration, setDuration] = useState(4000);

  function showToast() {
    const message = "Hello from BoomToaster 🚀";
    const options = { position, duration, description: "This is a demo toast" };
    boom[type](message, options);
  }

  return (
    <>
      <StyleInjector />

      <div className="blob-wrap">
        <div className="blob blob-a" />
        <div className="blob blob-b" />
        <div className="blob blob-c" />
      </div>

      <main style={{ position: "relative", zIndex: 1 }}>
        {/* ── HERO ── */}
        <section className="hero">
          <div className="hero-badge">
            <span className="ping-dot" />
            Open Source · Zero Dependencies
          </div>
          <h1 className="hero-title font-display">
            <span className="shimmer-text">BoomToaster</span>
          </h1>
          <p className="hero-sub">
            Lightweight, beautiful toast notifications for React.
            Zero&nbsp;dependencies. Blazing fast. Production&nbsp;ready.
          </p>
          <div className="hero-actions">
            <button
              className="btn-primary"
              onClick={() => boom.success("Boom! It works 🎉")}
            >
              <span>✦ Show Toast</span>
            </button>
            <a
              href="https://www.npmjs.com/package/boom-toaster"
              target="_blank"
              rel="noreferrer"
              className="btn-ghost"
            >
              NPM Package ↗
            </a>
          </div>
        </section>

        <div className="divider" />

        {/* ── PLAYGROUND ── */}
        <section className="section">
          <h2 className="section-title font-display">
            Interactive <span className="shimmer-text">Playground</span>
          </h2>
          <div className="playground-card">
            <div className="playground-grid">
              <CustomSelect
                label="Toast Type"
                value={type}
                onChange={(v) => setType(v as ToastType)}
                options={["success", "error", "warning", "info", "loading"]}
                meta={TYPE_META}
              />
              <CustomSelect
                label="Position"
                value={position}
                onChange={(v) => setPosition(v as ToastPosition)}
                options={[
                  "top-right",
                  "top-left",
                  "top-center",
                  "bottom-right",
                  "bottom-left",
                  "bottom-center",
                ]}
                meta={POS_META}
              />
              <div>
                <label className="field-label">Duration (ms)</label>
                <input
                  type="number"
                  value={duration}
                  onChange={(e) => setDuration(Number(e.target.value))}
                  className="field-input"
                />
              </div>
            </div>
            <div className="run-btn-wrap">
              <button className="btn-primary" onClick={showToast}>
                <span>⚡ Launch Toast</span>
              </button>
            </div>
          </div>
        </section>

        <div className="divider" />

        {/* ── FEATURES ── */}
        <section className="section bg-alt">
          <h2 className="section-title font-display">
            Why <span className="shimmer-text">BoomToaster</span>
          </h2>
          <div className="features-grid">
            {[
              {
                icon: "⚡",
                title: "Zero Dependencies",
                desc: "Only React required. No extra libraries shipped.",
                delay: "stagger-1",
              },
              {
                icon: "📦",
                title: "Tiny Bundle",
                desc: "Extremely lightweight and blazing fast.",
                delay: "stagger-2",
              },
              {
                icon: "✨",
                title: "Beautiful UI",
                desc: "Modern toast design with smooth, fluid animations.",
                delay: "stagger-3",
              },
              {
                icon: "🧭",
                title: "Multiple Positions",
                desc: "Top, bottom, and center placements all supported.",
                delay: "stagger-4",
              },
              {
                icon: "🔄",
                title: "Promise Support",
                desc: "Automatically handle async success and error states.",
                delay: "stagger-5",
              },
              {
                icon: "🎛️",
                title: "Fully Customizable",
                desc: "Duration, description, dismiss control and more.",
                delay: "stagger-6",
              },
            ].map(({ icon, title, desc, delay }) => (
              <Feature
                key={title}
                icon={icon}
                title={title}
                desc={desc}
                delay={delay}
              />
            ))}
          </div>
        </section>

        <div className="divider" />

        {/* ── INSTALLATION ── */}
        <section className="section">
          <h2 className="section-title font-display">
            Get <span className="shimmer-text">Started</span>
          </h2>
          <div className="install-wrap">
            <div className="install-cmd font-mono">
              npm install boom-toaster
            </div>
            <CodeBlock
              code={`import { BoomToaster } from "boom-toaster";

export default function Layout() {
  return (
    <>
      <BoomToaster position="top-right" />
    </>
  );
}`}
            />
          </div>
        </section>

        <div className="divider" />

        {/* ── EXAMPLES ── */}
        <section className="section bg-alt">
          <h2 className="section-title font-display">
            <span className="shimmer-text">Examples</span>
          </h2>
          <div
            style={{
              display: "flex",
              flexDirection: "column",
              gap: "16px",
              maxWidth: "780px",
              margin: "0 auto",
            }}
          >
            <CodeBlock
              code={`boom.success("Order placed!", { description: "Your order #1234 is confirmed" });`}
            />
            <CodeBlock
              code={`const id = boom.loading("Uploading...", { duration: 0 });\n\n// later\nboom.dismiss(id);`}
            />
            <CodeBlock
              code={`boom.promise(fetch("/api/save"), {\n  loading: "Saving...",\n  success: "Saved!",\n  error:   "Failed to save.",\n});`}
            />
          </div>
        </section>

        {/* ── FOOTER ── */}
        <footer className="site-footer">
          Built with ❤️ for <strong>Md Saidul Basar</strong>
        </footer>
      </main>
    </>
  );
}

// ─── Feature card ─────────────────────────────────────────────────────────────
function Feature({
  icon,
  title,
  desc,
  delay,
}: {
  icon: string;
  title: string;
  desc: string;
  delay: string;
}) {
  return (
    <div className={`feature-card ${delay}`}>
      <div className="feature-icon">{icon}</div>
      <div className="feature-title">{title}</div>
      <p className="feature-desc">{desc}</p>
    </div>
  );
}

// ─── Code block ───────────────────────────────────────────────────────────────
function CodeBlock({ code }: { code: string }) {
  const [copied, setCopied] = useState(false);

  const handleCopy = () => {
    safeCopy(code)
      .then(() => {
        setCopied(true);
        setTimeout(() => setCopied(false), 2000);
      })
      .catch(() => {
        // copy unavailable in this context — silently ignore
      });
  };

  return (
    <div className="code-card">
      <div className="code-header">
        <span className="dot dot-r" />
        <span className="dot dot-y" />
        <span className="dot dot-g" />
      </div>
      <button
        onClick={handleCopy}
        className={`copy-btn${copied ? " copied" : ""}`}
      >
        {copied ? "✓ Copied" : "Copy"}
      </button>
      <pre className="code-body hidden-scrollbar">
        <code className="select-all">{code}</code>
      </pre>
    </div>
  );
}
```
