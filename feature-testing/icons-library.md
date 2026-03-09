"use client";

import { useState, useMemo } from "react";
import * as Icons from "@/components/icons";

type IconEntry = {
  name: string;
  Component: React.FC;
};

export default function IconsPage() {
  const [search, setSearch] = useState("");
  const [copied, setCopied] = useState<string | null>(null);

  const iconList: IconEntry[] = useMemo(() => {
    return Object.entries(Icons)
      .filter(([, val]) => typeof val === "function")
      .map(([name, Component]) => ({
        name,
        Component: Component as React.FC,
      }));
  }, []);

  const filtered = useMemo(() => {
    if (!search.trim()) return iconList;
    return iconList.filter((i) =>
      i.name.toLowerCase().includes(search.toLowerCase()),
    );
  }, [search, iconList]);

  // ✅ Fixed: async + fallback for non-secure context / older browsers
  const handleCopy = async (name: string) => {
    const text = `<${name} />`;
    try {
      if (navigator?.clipboard?.writeText) {
        await navigator.clipboard.writeText(text);
      } else {
        // Fallback: hidden textarea trick
        const textarea = document.createElement("textarea");
        textarea.value = text;
        textarea.style.position = "fixed";
        textarea.style.opacity = "0";
        document.body.appendChild(textarea);
        textarea.focus();
        textarea.select();
        document.execCommand("copy");
        document.body.removeChild(textarea);
      }
      setCopied(name);
      setTimeout(() => setCopied(null), 2000);
    } catch (err) {
      console.error("Copy failed:", err);
    }
  };

  return (
    <main
      style={{ fontFamily: "'DM Mono', 'Fira Code', monospace" }}
      className="min-h-screen bg-white text-zinc-900 px-6 py-12"
    >
      {/* Header */}
      <div className="sm:container mx-auto mb-12">
        <p className="text-xs tracking-[0.3em] uppercase text-zinc-400 mb-2">
          Component Library
        </p>
        <h1 className="text-6xl font-bold tracking-tight text-zinc-900 mb-1">
          Icons
        </h1>
        <div className="h-px bg-gradient-to-r from-zinc-300 to-transparent mt-4 mb-6" />
        <p className="text-zinc-400 text-sm">
          {iconList.length} icons available · Click any icon to copy its usage
        </p>
      </div>

      <div className="max-w-5xl mx-auto mb-10">
        <div className="relative">
          <svg
            className="absolute left-4 top-1/2 -translate-y-1/2 text-zinc-400"
            width="16"
            height="16"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            strokeWidth="2"
          >
            <circle cx="11" cy="11" r="8" />
            <path d="m21 21-4.35-4.35" />
          </svg>

          <input
            type="text"
            placeholder="Search icons..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="w-full bg-zinc-50 border border-zinc-200 rounded-xl pl-11 pr-4 py-3 text-sm text-zinc-900 placeholder:text-zinc-400 focus:outline-none focus:border-zinc-400 focus:bg-white transition-all"
          />

          {search && (
            <button
              onClick={() => setSearch("")}
              className="absolute right-4 top-1/2 -translate-y-1/2 text-zinc-400 hover:text-zinc-700 transition-colors"
            >
              ✕
            </button>
          )}
        </div>

        {search && (
          <p className="text-xs text-zinc-400 mt-2 pl-1">
            {filtered.length} result{filtered.length !== 1 ? "s" : ""} for
            &quot;
            {search}&quot;
          </p>
        )}
      </div>

      <div className="max-w-5xl mx-auto">
        {filtered.length === 0 ? (
          <div className="text-center py-24 text-zinc-400">
            <p className="text-4xl mb-3">∅</p>
            <p className="text-sm">No icons found for &quot;{search}&quot;</p>
          </div>
        ) : (
          <div className="grid grid-cols-3 sm:grid-cols-4 md:grid-cols-5 lg:grid-cols-6 xl:grid-cols-8 gap-3">
            {filtered.map(({ name, Component }) => {
              const isCopied = copied === name;
              return (
                <button
                  key={name}
                  onClick={() => handleCopy(name)}
                  title={`Click to copy <${name} />`}
                  className={[
                    "group relative flex flex-col items-center justify-center gap-2.5",
                    "rounded-xl p-4 border transition-all duration-200 cursor-pointer",
                    isCopied
                      ? "bg-emerald-50 border-emerald-300 scale-95"
                      : "bg-zinc-50 border-zinc-200 hover:bg-zinc-100 hover:border-zinc-300 hover:scale-105",
                  ].join(" ")}
                >
                  <div className="w-8 h-8 flex items-center justify-center">
                    {isCopied ? (
                      <svg
                        width="20"
                        height="20"
                        viewBox="0 0 24 24"
                        fill="none"
                        stroke="#10b981"
                        strokeWidth="2.5"
                      >
                        <polyline points="20 6 9 17 4 12" />
                      </svg>
                    ) : (
                      <Component />
                    )}
                  </div>

                  <span
                    className={[
                      "text-[10px] leading-tight text-center break-all transition-colors",
                      isCopied
                        ? "text-emerald-600"
                        : "text-zinc-400 group-hover:text-zinc-700",
                    ].join(" ")}
                  >
                    {isCopied ? "Copied!" : name}
                  </span>

                  {!isCopied && (
                    <div className="absolute -top-9 left-1/2 -translate-x-1/2 opacity-0 group-hover:opacity-100 transition-opacity pointer-events-none z-10">
                      <div className="bg-zinc-800 border border-zinc-700 rounded-md px-2 py-1 text-[10px] whitespace-nowrap text-white shadow-lg">
                        {`<${name} />`}
                      </div>
                    </div>
                  )}
                </button>
              );
            })}
          </div>
        )}
      </div>

      <div className="max-w-5xl mx-auto mt-16 pt-8 border-t border-zinc-100 flex justify-between items-center text-xs text-zinc-400">
        <span>{iconList.length} total icons</span>
        <span>Click icon → copies JSX tag</span>
      </div>
    </main>
  );
}
