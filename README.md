[index.html](https://github.com/user-attachments/files/27585055/index.html)
<img width="512" height="512" alt="icon-maskable-512" src="https://github.com/user-attachments/assets/2207e59c-d93c-4ab3-9222-8a91b43ba77a" />
<img width="512" height="512" alt="icon-512" src="https://github.com/user-attachments/assets/f2726c48-bddc-4b4e-8c2b-542b8dd02eaa" />
<img width="192" height="192" alt="icon-192" src="https://github.com/user-attachments/assets/fec1a485-55b6-4e53-ac67-e65e2a57933e" />
<img width="32" height="32" alt="favicon-32" src="https://github.com/user-attachments/assets/d60211ed-296f-4bbb-bd49-0e7433c9643a" />
<img width="180" height="180" alt="apple-touch-icon" src="https://github.com/user-attachments/assets/9c8cba81-eb47-4e25-a379-ece61451313a" />
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
  <title>The Daily Ledger</title>
  <meta name="description" content="A personal daily journal — notes, highlights, todos, tags, and a remark corner." />
  <meta name="theme-color" content="#f3ecdd" />
  <link rel="manifest" href="manifest.json" />
  <link rel="icon" type="image/png" sizes="32x32" href="favicon-32.png" />
  <link rel="icon" type="image/png" sizes="192x192" href="icon-192.png" />
  <link rel="apple-touch-icon" href="apple-touch-icon.png" />
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-status-bar-style" content="default" />
  <meta name="apple-mobile-web-app-title" content="Daily Ledger" />

  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,400;0,9..144,600;0,9..144,800;1,9..144,800&family=DM+Sans:wght@400;500;600&family=Caveat:wght@400;600&display=swap" rel="stylesheet" />

  <script src="https://cdn.tailwindcss.com"></script>
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <style>
    html, body { margin: 0; padding: 0; }
    body {
      font-family: 'DM Sans', system-ui, sans-serif;
      background: #f3ecdd;
      color: #2a221b;
    }
    #root:empty::before {
      content: '';
      display: block;
      min-height: 100vh;
    }
    /* nicer date input look */
    input[type="date"]::-webkit-calendar-picker-indicator {
      cursor: pointer;
      opacity: 0.6;
    }
  </style>
</head>
<body>
  <div id="root"></div>

  <script type="text/babel" data-presets="react">
    const { useState, useEffect, useRef } = React;

    /* ---------- localStorage wrapper (mirrors Claude's storage API) ---------- */
    const storage = {
      async get(key) {
        try {
          const v = localStorage.getItem(key);
          return v != null ? { key, value: v } : null;
        } catch { return null; }
      },
      async set(key, value) {
        try {
          localStorage.setItem(key, value);
          return { key, value };
        } catch { return null; }
      },
    };

    /* ---------- inline SVG icons (lucide) ---------- */
    const Icon = ({ size = 16, children, ...rest }) => (
      <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size}
           viewBox="0 0 24 24" fill="none" stroke="currentColor"
           strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" {...rest}>
        {children}
      </svg>
    );
    const ChevronLeft = (p) => <Icon {...p}><path d="m15 18-6-6 6-6" /></Icon>;
    const ChevronRight = (p) => <Icon {...p}><path d="m9 18 6-6-6-6" /></Icon>;
    const Plus = (p) => <Icon {...p}><path d="M5 12h14" /><path d="M12 5v14" /></Icon>;
    const X = (p) => <Icon {...p}><path d="M18 6 6 18" /><path d="m6 6 12 12" /></Icon>;
    const Sparkles = (p) => <Icon {...p}>
      <path d="M9.937 15.5A2 2 0 0 0 8.5 14.063l-6.135-1.582a.5.5 0 0 1 0-.962L8.5 9.936A2 2 0 0 0 9.937 8.5l1.582-6.135a.5.5 0 0 1 .963 0L14.063 8.5A2 2 0 0 0 15.5 9.937l6.135 1.581a.5.5 0 0 1 0 .964L15.5 14.063a2 2 0 0 0-1.437 1.437l-1.582 6.135a.5.5 0 0 1-.963 0z" />
      <path d="M20 3v4" /><path d="M22 5h-4" /><path d="M4 17v2" /><path d="M5 18H3" />
    </Icon>;
    const CloudRain = (p) => <Icon {...p}>
      <path d="M4 14.899A7 7 0 1 1 15.71 8h1.79a4.5 4.5 0 0 1 2.5 8.242" />
      <path d="M16 14v6" /><path d="M8 14v6" /><path d="M12 16v6" />
    </Icon>;
    const ListChecks = (p) => <Icon {...p}>
      <path d="m3 17 2 2 4-4" /><path d="m3 7 2 2 4-4" />
      <path d="M13 6h8" /><path d="M13 12h8" /><path d="M13 18h8" />
    </Icon>;
    const TagIcon = (p) => <Icon {...p}>
      <path d="M12.586 2.586A2 2 0 0 0 11.172 2H4a2 2 0 0 0-2 2v7.172a2 2 0 0 0 .586 1.414l8.704 8.704a2.426 2.426 0 0 0 3.42 0l6.58-6.58a2.426 2.426 0 0 0 0-3.42z" />
      <circle cx="7.5" cy="7.5" r=".5" fill="currentColor" />
    </Icon>;
    const PenLine = (p) => <Icon {...p}>
      <path d="M12 20h9" />
      <path d="M16.376 3.622a1 1 0 0 1 3.002 3.002L7.368 18.635a2 2 0 0 1-.855.506l-2.872.838a.5.5 0 0 1-.62-.62l.838-2.872a2 2 0 0 1 .506-.854z" />
    </Icon>;
    const CalendarIcon = (p) => <Icon {...p}>
      <path d="M8 2v4" /><path d="M16 2v4" />
      <rect width="18" height="18" x="3" y="4" rx="2" />
      <path d="M3 10h18" />
    </Icon>;
    const Trash2 = (p) => <Icon {...p}>
      <path d="M3 6h18" />
      <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6" />
      <path d="M8 6V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2" />
      <line x1="10" x2="10" y1="11" y2="17" />
      <line x1="14" x2="14" y1="11" y2="17" />
    </Icon>;

    /* ---------- palette ---------- */
    const C = {
      bg: "#f3ecdd",
      paper: "#fbf6ea",
      ink: "#2a221b",
      inkSoft: "#6b5e4f",
      inkFaint: "#a39684",
      rule: "rgba(42, 34, 27, 0.12)",
      amber: "#b9842c",
      amberSoft: "#f4e3bf",
      clay: "#a8543c",
      claySoft: "#f1d4c8",
      sage: "#5b6b4a",
      sageSoft: "#dde4cf",
      sticky: "#f7d96a",
      stickyEdge: "#e6c247",
    };

    /* ---------- date helpers ---------- */
    const isoToday = () => {
      const d = new Date();
      return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, "0")}-${String(d.getDate()).padStart(2, "0")}`;
    };
    const shift = (iso, days) => {
      const d = new Date(iso + "T00:00:00");
      d.setDate(d.getDate() + days);
      return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, "0")}-${String(d.getDate()).padStart(2, "0")}`;
    };
    const longDate = (iso) =>
      new Date(iso + "T00:00:00").toLocaleDateString("en-US", {
        weekday: "long", month: "long", day: "numeric", year: "numeric",
      });

    const EMPTY = { freeNote: "", highlights: [], lowlights: [], todos: [], tags: [], remark: "" };
    const uid = () => Math.random().toString(36).slice(2, 9);

    /* ============================================================ */
    function DailyLedger() {
      const [date, setDate] = useState(isoToday());
      const [entry, setEntry] = useState(EMPTY);
      const [loading, setLoading] = useState(true);
      const [saveStatus, setSaveStatus] = useState("idle");
      const [newTag, setNewTag] = useState("");
      const [showJump, setShowJump] = useState(false);
      const saveTimerRef = useRef(null);
      const skipSaveRef = useRef(true);

      useEffect(() => {
        let cancelled = false;
        skipSaveRef.current = true;
        setLoading(true);
        (async () => {
          try {
            const res = await storage.get(`entry:${date}`);
            if (cancelled) return;
            if (res && res.value) {
              setEntry({ ...EMPTY, ...JSON.parse(res.value) });
            } else {
              setEntry(EMPTY);
            }
          } catch {
            if (!cancelled) setEntry(EMPTY);
          } finally {
            if (!cancelled) {
              setLoading(false);
              setTimeout(() => { skipSaveRef.current = false; }, 120);
            }
          }
        })();
        return () => { cancelled = true; };
      }, [date]);

      useEffect(() => {
        if (loading || skipSaveRef.current) return;
        if (saveTimerRef.current) clearTimeout(saveTimerRef.current);
        setSaveStatus("saving");
        saveTimerRef.current = setTimeout(async () => {
          try {
            await storage.set(`entry:${date}`, JSON.stringify(entry));
            setSaveStatus("saved");
            setTimeout(() => setSaveStatus("idle"), 1400);
          } catch { setSaveStatus("idle"); }
        }, 500);
        return () => saveTimerRef.current && clearTimeout(saveTimerRef.current);
      }, [entry, date, loading]);

      const patch = (p) => setEntry((prev) => ({ ...prev, ...p }));
      const addLine = (key) => patch({ [key]: [...entry[key], { id: uid(), text: "" }] });
      const editLine = (key, id, text) => patch({ [key]: entry[key].map(l => l.id === id ? { ...l, text } : l) });
      const removeLine = (key, id) => patch({ [key]: entry[key].filter(l => l.id !== id) });
      const addTodo = () => patch({ todos: [...entry.todos, { id: uid(), text: "", done: false }] });
      const editTodo = (id, text) => patch({ todos: entry.todos.map(t => t.id === id ? { ...t, text } : t) });
      const toggleTodo = (id) => patch({ todos: entry.todos.map(t => t.id === id ? { ...t, done: !t.done } : t) });
      const removeTodo = (id) => patch({ todos: entry.todos.filter(t => t.id !== id) });
      const addTag = () => {
        const t = newTag.trim();
        if (!t || entry.tags.includes(t)) return;
        patch({ tags: [...entry.tags, t] });
        setNewTag("");
      };
      const removeTag = (t) => patch({ tags: entry.tags.filter(x => x !== t) });
      const clearDay = () => {
        if (window.confirm("Clear all entries for this day? This cannot be undone.")) {
          setEntry(EMPTY);
        }
      };
      const isToday = date === isoToday();

      return (
        <div className="min-h-screen w-full"
          style={{
            background: C.bg,
            backgroundImage:
              "radial-gradient(circle at 20% 10%, rgba(255,255,255,0.55), transparent 55%), radial-gradient(circle at 85% 85%, rgba(180,150,100,0.10), transparent 60%)",
            fontFamily: "'DM Sans', system-ui, sans-serif",
            color: C.ink,
          }}
        >
          <div className="max-w-6xl mx-auto px-6 py-8 md:py-12">
            {/* Masthead */}
            <header className="flex items-end justify-between pb-4 mb-8 border-b" style={{ borderColor: C.rule }}>
              <div>
                <div className="text-xs uppercase" style={{ color: C.inkFaint, letterSpacing: "0.3em" }}>
                  Vol. I &middot; A personal ledger
                </div>
                <h1 className="leading-none mt-1"
                  style={{
                    fontFamily: "'Fraunces', serif",
                    fontWeight: 800,
                    fontSize: "clamp(2.2rem, 5vw, 3.4rem)",
                    fontStyle: "italic",
                    letterSpacing: "-0.02em",
                  }}>
                  The Daily Ledger
                </h1>
              </div>
              <SaveBadge status={saveStatus} />
            </header>

            {/* Date Navigator */}
            <div className="flex items-center justify-between gap-4 mb-8 flex-wrap">
              <div className="flex items-center gap-3">
                <NavBtn onClick={() => setDate(shift(date, -1))} aria-label="Previous day">
                  <ChevronLeft size={18} />
                </NavBtn>
                <div className="text-center">
                  <div style={{
                    fontFamily: "'Fraunces', serif",
                    fontSize: "1.55rem",
                    fontWeight: 600,
                    letterSpacing: "-0.01em",
                  }}>
                    {longDate(date)}
                  </div>
                  {!isToday && (
                    <button onClick={() => setDate(isoToday())}
                      className="text-xs underline mt-0.5 hover:opacity-70 transition-opacity"
                      style={{ color: C.inkSoft }}>
                      return to today
                    </button>
                  )}
                </div>
                <NavBtn onClick={() => setDate(shift(date, 1))} aria-label="Next day">
                  <ChevronRight size={18} />
                </NavBtn>
              </div>

              <div className="flex items-center gap-2 relative">
                <button onClick={() => setShowJump(s => !s)}
                  className="flex items-center gap-1.5 px-3 py-1.5 text-sm rounded-full transition-all hover:scale-105"
                  style={{ background: C.paper, border: `1px solid ${C.rule}`, color: C.inkSoft }}>
                  <CalendarIcon size={14} />
                  Jump to date
                </button>
                {showJump && (
                  <input type="date" value={date}
                    onChange={(e) => { setDate(e.target.value); setShowJump(false); }}
                    className="absolute right-0 top-full mt-2 px-3 py-2 rounded-lg text-sm z-10"
                    style={{ background: C.paper, border: `1px solid ${C.rule}`, color: C.ink, fontFamily: "'DM Sans', sans-serif" }}
                    autoFocus
                    onBlur={() => setTimeout(() => setShowJump(false), 200)}
                  />
                )}
                <button onClick={clearDay}
                  className="flex items-center gap-1.5 px-3 py-1.5 text-sm rounded-full transition-all hover:scale-105"
                  style={{ background: "transparent", border: `1px solid ${C.rule}`, color: C.inkFaint }}
                  title="Clear this day's entries">
                  <Trash2 size={14} />
                  Clear
                </button>
              </div>
            </div>

            {/* Main Grid */}
            <div className="grid grid-cols-1 lg:grid-cols-5 gap-6">
              <section className="lg:col-span-3 rounded-2xl p-6 md:p-8 relative"
                style={{
                  background: C.paper,
                  border: `1px solid ${C.rule}`,
                  boxShadow: "0 1px 0 rgba(255,255,255,0.7) inset, 0 10px 30px -20px rgba(42,34,27,0.25)",
                }}>
                <SectionLabel icon={<PenLine size={14} />} text="Today's notes" />
                <textarea value={entry.freeNote}
                  onChange={(e) => patch({ freeNote: e.target.value })}
                  placeholder="What happened today? Anything on your mind…"
                  className="w-full bg-transparent border-0 outline-none resize-none mt-3 leading-relaxed relative z-10"
                  style={{
                    minHeight: "320px",
                    fontFamily: "'Fraunces', serif",
                    fontSize: "1.05rem",
                    color: C.ink,
                    lineHeight: "1.75",
                  }}
                />
                <div aria-hidden className="absolute inset-x-8 bottom-6 top-20 pointer-events-none"
                  style={{
                    opacity: 0.06,
                    backgroundImage: `repeating-linear-gradient(to bottom, transparent 0, transparent 27px, ${C.ink} 27px, ${C.ink} 28px)`,
                  }}
                />
              </section>

              <aside className="lg:col-span-2 flex flex-col gap-6">
                <Panel icon={<Sparkles size={14} />} title="Highlights" accent={C.amber} bg={C.paper}>
                  <LineList items={entry.highlights} accent={C.amber} bullet="✦"
                    onEdit={(id, t) => editLine("highlights", id, t)}
                    onRemove={(id) => removeLine("highlights", id)}
                    placeholder="A small win, a warm moment…" />
                  <AddBtn onClick={() => addLine("highlights")} accent={C.amber}>add highlight</AddBtn>
                </Panel>

                <Panel icon={<CloudRain size={14} />} title="Lowlights" accent={C.clay} bg={C.paper}>
                  <LineList items={entry.lowlights} accent={C.clay} bullet="◦"
                    onEdit={(id, t) => editLine("lowlights", id, t)}
                    onRemove={(id) => removeLine("lowlights", id)}
                    placeholder="A friction, a frustration, a lesson…" />
                  <AddBtn onClick={() => addLine("lowlights")} accent={C.clay}>add lowlight</AddBtn>
                </Panel>

                <Panel icon={<ListChecks size={14} />} title="To-do" accent={C.sage} bg={C.paper}>
                  <ul className="space-y-1.5">
                    {entry.todos.map((t) => (
                      <li key={t.id} className="flex items-center gap-2 group">
                        <button onClick={() => toggleTodo(t.id)}
                          className="flex-shrink-0 w-4 h-4 rounded border flex items-center justify-center transition-all"
                          style={{
                            borderColor: t.done ? C.sage : C.inkFaint,
                            background: t.done ? C.sage : "transparent",
                          }}
                          aria-label={t.done ? "Mark undone" : "Mark done"}>
                          {t.done && (
                            <svg width="10" height="10" viewBox="0 0 12 12" fill="none">
                              <path d="M2 6.5L5 9.5L10 3" stroke="white" strokeWidth="2"
                                strokeLinecap="round" strokeLinejoin="round" />
                            </svg>
                          )}
                        </button>
                        <input value={t.text}
                          onChange={(e) => editTodo(t.id, e.target.value)}
                          placeholder="something to do…"
                          className="flex-1 bg-transparent border-0 outline-none text-sm py-0.5"
                          style={{
                            color: t.done ? C.inkFaint : C.ink,
                            textDecoration: t.done ? "line-through" : "none",
                            fontFamily: "'DM Sans', sans-serif",
                          }}
                        />
                        <button onClick={() => removeTodo(t.id)}
                          className="opacity-0 group-hover:opacity-100 transition-opacity"
                          style={{ color: C.inkFaint }} aria-label="Remove">
                          <X size={13} />
                        </button>
                      </li>
                    ))}
                  </ul>
                  <AddBtn onClick={addTodo} accent={C.sage}>add task</AddBtn>
                </Panel>

                <Panel icon={<TagIcon size={14} />} title="Tags" accent={C.inkSoft} bg={C.paper}>
                  <div className="flex flex-wrap gap-1.5 items-center">
                    {entry.tags.map((t) => (
                      <span key={t}
                        className="inline-flex items-center gap-1 text-xs px-2.5 py-1 rounded-full group"
                        style={{ background: C.sageSoft, color: C.sage, fontFamily: "'DM Sans', sans-serif", fontWeight: 500 }}>
                        {t}
                        <button onClick={() => removeTag(t)}
                          className="opacity-50 hover:opacity-100 transition-opacity"
                          aria-label="Remove tag">
                          <X size={11} />
                        </button>
                      </span>
                    ))}
                    <input value={newTag}
                      onChange={(e) => setNewTag(e.target.value)}
                      onKeyDown={(e) => { if (e.key === "Enter") { e.preventDefault(); addTag(); } }}
                      placeholder={entry.tags.length ? "+ tag" : "+ add tag"}
                      className="bg-transparent border-0 outline-none text-xs px-2 py-1"
                      style={{ color: C.ink, minWidth: "80px", fontFamily: "'DM Sans', sans-serif" }}
                    />
                  </div>
                </Panel>
              </aside>
            </div>

            {/* Remark Corner */}
            <div className="flex justify-end mt-10 mb-6 pr-2">
              <div className="relative"
                style={{ transform: "rotate(-2.5deg)", transition: "transform 0.3s ease" }}
                onMouseEnter={(e) => e.currentTarget.style.transform = "rotate(-1deg) scale(1.02)"}
                onMouseLeave={(e) => e.currentTarget.style.transform = "rotate(-2.5deg)"}>
                <div className="p-5 pb-6 w-72 md:w-80"
                  style={{
                    background: `linear-gradient(135deg, ${C.sticky} 0%, ${C.stickyEdge} 100%)`,
                    boxShadow: "0 1px 0 rgba(255,255,255,0.5) inset, 0 14px 28px -10px rgba(80,60,20,0.35), 0 4px 8px -4px rgba(80,60,20,0.2)",
                    clipPath: "polygon(0 0, 100% 0, 100% 88%, 88% 100%, 0 100%)",
                  }}>
                  <div className="flex items-center gap-1.5 mb-2" style={{ color: "#6b5320" }}>
                    <PenLine size={13} />
                    <span className="uppercase" style={{ fontWeight: 600, fontSize: "10px", letterSpacing: "0.25em" }}>
                      Remark Corner
                    </span>
                  </div>
                  <textarea value={entry.remark}
                    onChange={(e) => patch({ remark: e.target.value })}
                    placeholder="A scribble in the margin…"
                    className="w-full bg-transparent border-0 outline-none resize-none"
                    rows={5}
                    style={{
                      fontFamily: "'Caveat', cursive",
                      fontSize: "1.25rem",
                      lineHeight: "1.4",
                      color: "#3a2c10",
                    }}
                  />
                </div>
              </div>
            </div>

            <footer className="text-center text-xs pt-6 mt-4 border-t"
              style={{ borderColor: C.rule, color: C.inkFaint }}>
              Saved locally on this device &middot; entries persist across visits
            </footer>
          </div>
        </div>
      );
    }

    function SaveBadge({ status }) {
      const map = {
        idle:   { text: "saved",    dot: C.inkFaint },
        saving: { text: "saving…",  dot: C.amber },
        saved:  { text: "saved",    dot: C.sage },
      };
      const cur = map[status] || map.idle;
      return (
        <div className="flex items-center gap-2 text-xs" style={{ color: C.inkSoft }}>
          <span className="inline-block w-1.5 h-1.5 rounded-full transition-colors" style={{ background: cur.dot }} />
          {cur.text}
        </div>
      );
    }

    function NavBtn({ children, ...props }) {
      return (
        <button {...props}
          className="w-9 h-9 rounded-full flex items-center justify-center transition-all hover:scale-110"
          style={{ background: C.paper, border: `1px solid ${C.rule}`, color: C.inkSoft }}>
          {children}
        </button>
      );
    }

    function SectionLabel({ icon, text, accent }) {
      return (
        <div className="flex items-center gap-1.5 uppercase"
          style={{
            color: accent || C.inkFaint,
            fontWeight: 600, fontSize: "10px", letterSpacing: "0.28em",
          }}>
          {icon}<span>{text}</span>
        </div>
      );
    }

    function Panel({ icon, title, accent, bg, children }) {
      return (
        <section className="rounded-2xl p-5"
          style={{
            background: bg,
            border: `1px solid ${C.rule}`,
            boxShadow: "0 1px 0 rgba(255,255,255,0.7) inset, 0 6px 20px -16px rgba(42,34,27,0.2)",
          }}>
          <div className="flex items-center justify-between mb-3">
            <SectionLabel icon={icon} text={title} accent={accent} />
          </div>
          {children}
        </section>
      );
    }

    function LineList({ items, accent, bullet, onEdit, onRemove, placeholder }) {
      return (
        <ul className="space-y-1.5">
          {items.map((it) => (
            <li key={it.id} className="flex items-start gap-2 group">
              <span className="flex-shrink-0 mt-1.5" style={{ color: accent, fontSize: "11px" }}>{bullet}</span>
              <input value={it.text}
                onChange={(e) => onEdit(it.id, e.target.value)}
                placeholder={placeholder}
                className="flex-1 bg-transparent border-0 outline-none text-sm py-0.5"
                style={{ color: C.ink, fontFamily: "'DM Sans', sans-serif" }}
              />
              <button onClick={() => onRemove(it.id)}
                className="opacity-0 group-hover:opacity-100 transition-opacity mt-1"
                style={{ color: C.inkFaint }} aria-label="Remove">
                <X size={13} />
              </button>
            </li>
          ))}
        </ul>
      );
    }

    function AddBtn({ onClick, accent, children }) {
      return (
        <button onClick={onClick}
          className="flex items-center gap-1 mt-2.5 text-xs transition-opacity hover:opacity-70"
          style={{ color: accent }}>
          <Plus size={12} />
          {children}
        </button>
      );
    }

    ReactDOM.createRoot(document.getElementById('root')).render(<DailyLedger />);
  </script>

  <script>
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        navigator.serviceWorker.register('sw.js').catch(err => console.warn('SW failed:', err));
      });
    }
  </script>
</body>
</html>
[manifest.json](https://github.com/user-attachments/files/27585064/manifest.json)
{
  "name": "The Daily Ledger",
  "short_name": "Ledger",
  "description": "A personal daily journal with notes, highlights, todos, tags, and a remark corner.",
  "start_url": "./index.html",
  "scope": "./",
  "display": "standalone",
  "orientation": "any",
  "background_color": "#f3ecdd",
  "theme_color": "#f3ecdd",
  "categories": ["productivity", "lifestyle"],
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "icon-maskable-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ]
}
[README.md](https://github.com/user-attachments/files/27585069/README.md)
# The Daily Ledger — PWA

A personal daily journal that installs to your phone, tablet, or desktop like a native app. Notes, highlights, lowlights, todos, tags, and a sticky-note remark corner. All data stays on your device.

## What's in this folder

```
daily-ledger-pwa/
├── index.html              the app
├── manifest.json           PWA install metadata
├── sw.js                   service worker (offline support)
├── icon-192.png            app icon (192×192)
├── icon-512.png            app icon (512×512)
├── icon-maskable-512.png   adaptive Android icon
├── apple-touch-icon.png    iOS home-screen icon
└── favicon-32.png          browser tab icon
```

## Why you need to host it (can't just double-click)

PWAs require HTTPS and a real URL — service workers won't register on `file://`. Pick one option below.

---

## Option 1 — Netlify Drop (easiest, ~30 seconds)

1. Go to **https://app.netlify.com/drop**
2. Drag this entire `daily-ledger-pwa` folder onto the page
3. Netlify gives you a URL like `https://random-name-12345.netlify.app`
4. Open it on your phone or desktop and install (see "Installing" below)

Free, no account needed for a temporary site. Sign up if you want to keep the URL forever.

---

## Option 2 — GitHub Pages (free, permanent URL)

1. Create a new GitHub repo (e.g. `my-daily-ledger`), public.
2. Upload all files from this folder to the repo root.
3. Repo → **Settings** → **Pages** → **Source: Deploy from a branch** → pick `main`, folder `/ (root)` → Save.
4. Wait ~1 minute. Your URL will be `https://YOUR-USERNAME.github.io/my-daily-ledger/`
5. Open it and install.

---

## Option 3 — Run locally for testing

You need a tiny local web server (just opening the file won't work for the service worker).

**With Python (already installed on Mac/Linux):**
```bash
cd daily-ledger-pwa
python3 -m http.server 8080
```
Then open `http://localhost:8080` in a browser.

**With Node:**
```bash
npx serve daily-ledger-pwa
```

Note: PWA install prompts only appear on HTTPS or `localhost`, so local testing works fine.

---

## Installing the app

Once the URL is open in a browser:

- **iPhone / iPad (Safari):** Share button → **Add to Home Screen**
- **Android (Chrome):** Three-dot menu → **Install app** (or it may prompt automatically)
- **Mac / Windows (Chrome / Edge):** Look for the install icon in the address bar (a small monitor with an arrow) → **Install**

After install you'll have a "Ledger" app icon. Tapping it opens the app full-screen with no browser chrome.

## Data & privacy

All entries save to your device's `localStorage`, scoped to the URL the app is installed from. **Different URL = different data.** If you switch hosts later, your old entries won't carry over automatically (they're not synced anywhere — this is a feature, not a bug).

To export: open browser DevTools → Application → Local Storage → copy the keys starting with `entry:`.

## Updating the app

If you change the code, bump the `CACHE` version in `sw.js` (e.g. `'daily-ledger-v1'` → `'daily-ledger-v2'`) so installed users get the new version on next open.

## Offline

After first visit the service worker caches everything (including the React/Tailwind/Babel CDN scripts), so the app works fully offline from then on.
[sw.js](https://github.com/user-attachments/files/27585075/sw.js)
/* Daily Ledger — service worker */
const CACHE = 'daily-ledger-v1';

const APP_SHELL = [
  './',
  './index.html',
  './manifest.json',
  './icon-192.png',
  './icon-512.png',
  './apple-touch-icon.png',
];

const CDN = [
  'https://cdn.tailwindcss.com',
  'https://unpkg.com/react@18/umd/react.production.min.js',
  'https://unpkg.com/react-dom@18/umd/react-dom.production.min.js',
  'https://unpkg.com/@babel/standalone/babel.min.js',
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE).then(async (cache) => {
      // Cache app shell (must succeed)
      await cache.addAll(APP_SHELL);
      // Try to cache CDN assets (best-effort)
      await Promise.all(
        CDN.map((url) =>
          fetch(url, { mode: 'no-cors' })
            .then((res) => cache.put(url, res))
            .catch(() => {})
        )
      );
    })
  );
  self.skipWaiting();
});

self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((keys) =>
      Promise.all(keys.filter((k) => k !== CACHE).map((k) => caches.delete(k)))
    )
  );
  self.clients.claim();
});

self.addEventListener('fetch', (event) => {
  const { request } = event;
  if (request.method !== 'GET') return;

  // Network-first for the HTML doc (so updates show up), cache fallback when offline
  if (request.mode === 'navigate') {
    event.respondWith(
      fetch(request)
        .then((res) => {
          const copy = res.clone();
          caches.open(CACHE).then((c) => c.put(request, copy));
          return res;
        })
        .catch(() => caches.match(request).then((r) => r || caches.match('./index.html')))
    );
    return;
  }

  // Cache-first for everything else (assets, fonts, CDN scripts)
  event.respondWith(
    caches.match(request).then(
      (cached) =>
        cached ||
        fetch(request)
          .then((res) => {
            // Only cache successful, basic/cors responses
            if (res && (res.status === 200 || res.type === 'opaque')) {
              const copy = res.clone();
              caches.open(CACHE).then((c) => c.put(request, copy));
            }
            return res;
          })
          .catch(() => cached)
    )
  );
});
