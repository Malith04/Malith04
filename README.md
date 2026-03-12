import { useState, useEffect, useRef } from "react";

const INITIAL_DATA = {
  name: "Alex Mercer",
  username: "alexmercer",
  bio: "Full Stack Developer · Open Source Enthusiast · Building the future one commit at a time 🚀",
  location: "San Francisco, CA",
  website: "https://alexmercer.dev",
  company: "@OpenSource Labs",
  stats: { repos: 142, followers: 3800, following: 210, stars: 12400 },
  skills: ["React", "Node.js", "TypeScript", "Rust", "Go", "Python", "Docker", "AWS"],
  pinnedRepos: [
    { name: "quantum-ui", desc: "Next-gen UI components with physics-based animations", lang: "TypeScript", stars: 4200, forks: 318, color: "#3178c6" },
    { name: "rust-engine", desc: "Blazing fast HTTP server built in pure Rust", lang: "Rust", stars: 3100, forks: 241, color: "#dea584" },
    { name: "ml-toolkit", desc: "Machine learning utilities for JavaScript developers", lang: "Python", stars: 2800, forks: 195, color: "#3572A5" },
    { name: "devops-kit", desc: "One-click deployment scripts for modern cloud infra", lang: "Go", stars: 1900, forks: 142, color: "#00ADD8" },
  ],
  contributions: Array.from({ length: 365 }, () => Math.floor(Math.random() * 5)),
  avatar: null,
};

function ParticleField() {
  const canvasRef = useRef(null);
  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext("2d");
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    const particles = Array.from({ length: 80 }, () => ({
      x: Math.random() * canvas.width,
      y: Math.random() * canvas.height,
      vx: (Math.random() - 0.5) * 0.4,
      vy: (Math.random() - 0.5) * 0.4,
      r: Math.random() * 1.5 + 0.5,
      alpha: Math.random() * 0.5 + 0.1,
    }));
    let raf;
    const draw = () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      particles.forEach((p) => {
        p.x += p.vx; p.y += p.vy;
        if (p.x < 0 || p.x > canvas.width) p.vx *= -1;
        if (p.y < 0 || p.y > canvas.height) p.vy *= -1;
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
        ctx.fillStyle = `rgba(0,255,180,${p.alpha})`;
        ctx.fill();
      });
      particles.forEach((a, i) => particles.slice(i + 1).forEach((b) => {
        const dx = a.x - b.x, dy = a.y - b.y, dist = Math.sqrt(dx * dx + dy * dy);
        if (dist < 120) {
          ctx.beginPath();
          ctx.moveTo(a.x, a.y); ctx.lineTo(b.x, b.y);
          ctx.strokeStyle = `rgba(0,255,180,${0.08 * (1 - dist / 120)})`;
          ctx.lineWidth = 0.5; ctx.stroke();
        }
      }));
      raf = requestAnimationFrame(draw);
    };
    draw();
    const resize = () => { canvas.width = window.innerWidth; canvas.height = window.innerHeight; };
    window.addEventListener("resize", resize);
    return () => { cancelAnimationFrame(raf); window.removeEventListener("resize", resize); };
  }, []);
  return <canvas ref={canvasRef} style={{ position: "fixed", inset: 0, zIndex: 0, pointerEvents: "none" }} />;
}

function GlitchText({ text }) {
  return (
    <span className="glitch" data-text={text} style={{ position: "relative" }}>
      {text}
    </span>
  );
}

function ContributionGrid({ data }) {
  const max = Math.max(...data);
  const colors = ["#0d1117", "#0e4429", "#006d32", "#26a641", "#39d353"];
  return (
    <div style={{ overflowX: "auto", paddingBottom: 4 }}>
      <div style={{ display: "grid", gridTemplateRows: "repeat(7, 12px)", gridAutoFlow: "column", gap: 3, width: "max-content" }}>
        {data.map((v, i) => (
          <div key={i} title={`${v} contributions`} style={{
            width: 12, height: 12, borderRadius: 2,
            background: colors[Math.round((v / max) * 4)] || colors[0],
            transition: "transform 0.15s",
            cursor: "pointer",
          }}
            onMouseEnter={e => e.target.style.transform = "scale(1.5)"}
            onMouseLeave={e => e.target.style.transform = "scale(1)"}
          />
        ))}
      </div>
    </div>
  );
}

function StatCard({ label, value }) {
  const [display, setDisplay] = useState(0);
  useEffect(() => {
    let start = 0; const target = value;
    const step = Math.ceil(target / 60);
    const t = setInterval(() => {
      start += step;
      if (start >= target) { setDisplay(target); clearInterval(t); }
      else setDisplay(start);
    }, 16);
    return () => clearInterval(t);
  }, [value]);
  const fmt = n => n >= 1000 ? (n / 1000).toFixed(1) + "k" : n;
  return (
    <div style={{
      background: "rgba(0,255,180,0.04)", border: "1px solid rgba(0,255,180,0.15)",
      borderRadius: 12, padding: "16px 20px", textAlign: "center",
      backdropFilter: "blur(8px)", transition: "all 0.3s",
      cursor: "default",
    }}
      onMouseEnter={e => { e.currentTarget.style.background = "rgba(0,255,180,0.1)"; e.currentTarget.style.borderColor = "rgba(0,255,180,0.5)"; e.currentTarget.style.transform = "translateY(-4px)"; }}
      onMouseLeave={e => { e.currentTarget.style.background = "rgba(0,255,180,0.04)"; e.currentTarget.style.borderColor = "rgba(0,255,180,0.15)"; e.currentTarget.style.transform = "translateY(0)"; }}
    >
      <div style={{ fontSize: 28, fontWeight: 800, color: "#00ffb4", fontFamily: "'Space Mono', monospace" }}>{fmt(display)}</div>
      <div style={{ fontSize: 11, color: "#8b949e", textTransform: "uppercase", letterSpacing: 1.5, marginTop: 4 }}>{label}</div>
    </div>
  );
}

function RepoCard({ repo }) {
  const [hovered, setHovered] = useState(false);
  return (
    <div onMouseEnter={() => setHovered(true)} onMouseLeave={() => setHovered(false)}
      style={{
        background: hovered ? "rgba(0,255,180,0.07)" : "rgba(13,17,23,0.8)",
        border: `1px solid ${hovered ? "rgba(0,255,180,0.4)" : "rgba(48,54,61,0.8)"}`,
        borderRadius: 14, padding: "20px 22px",
        transition: "all 0.3s cubic-bezier(0.175,0.885,0.32,1.275)",
        transform: hovered ? "translateY(-6px) scale(1.01)" : "none",
        cursor: "pointer", backdropFilter: "blur(12px)",
        boxShadow: hovered ? `0 20px 40px rgba(0,255,180,0.1)` : "none",
      }}>
      <div style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 8 }}>
        <svg width="16" height="16" fill="none" stroke="#8b949e" strokeWidth="1.5" viewBox="0 0 24 24"><path d="M3 3h18v18H3z" rx="2" /><path d="M3 9h18M9 21V9" /></svg>
        <span style={{ color: "#58a6ff", fontWeight: 700, fontFamily: "'Space Mono', monospace", fontSize: 14 }}>{repo.name}</span>
      </div>
      <p style={{ color: "#8b949e", fontSize: 12, margin: "0 0 16px", lineHeight: 1.6 }}>{repo.desc}</p>
      <div style={{ display: "flex", gap: 16, fontSize: 12, color: "#8b949e" }}>
        <span style={{ display: "flex", alignItems: "center", gap: 5 }}>
          <span style={{ width: 10, height: 10, borderRadius: "50%", background: repo.color, display: "inline-block" }} />
          {repo.lang}
        </span>
        <span>⭐ {(repo.stars / 1000).toFixed(1)}k</span>
        <span>🍴 {repo.forks}</span>
      </div>
    </div>
  );
}

function EditModal({ data, onSave, onClose }) {
  const [form, setForm] = useState({ ...data });
  const set = (k, v) => setForm(f => ({ ...f, [k]: v }));
  return (
    <div style={{ position: "fixed", inset: 0, zIndex: 1000, display: "flex", alignItems: "center", justifyContent: "center", background: "rgba(0,0,0,0.85)", backdropFilter: "blur(8px)" }}>
      <div style={{
        background: "#0d1117", border: "1px solid rgba(0,255,180,0.3)", borderRadius: 20,
        padding: 36, width: "min(540px, 95vw)", maxHeight: "88vh", overflowY: "auto",
        boxShadow: "0 0 80px rgba(0,255,180,0.1)",
        animation: "slideUp 0.3s cubic-bezier(0.175,0.885,0.32,1.275)",
      }}>
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 28 }}>
          <h2 style={{ color: "#00ffb4", fontFamily: "'Space Mono', monospace", margin: 0, fontSize: 18 }}>✏️ Edit Profile</h2>
          <button onClick={onClose} style={{ background: "none", border: "none", color: "#8b949e", fontSize: 22, cursor: "pointer", padding: 4 }}>✕</button>
        </div>
        {[
          { label: "Display Name", key: "name" },
          { label: "Username", key: "username" },
          { label: "Bio", key: "bio", multi: true },
          { label: "Location", key: "location" },
          { label: "Website", key: "website" },
          { label: "Company", key: "company" },
        ].map(({ label, key, multi }) => (
          <div key={key} style={{ marginBottom: 18 }}>
            <label style={{ display: "block", color: "#8b949e", fontSize: 12, marginBottom: 6, textTransform: "uppercase", letterSpacing: 1 }}>{label}</label>
            {multi
              ? <textarea value={form[key]} onChange={e => set(key, e.target.value)} rows={3} style={inputStyle} />
              : <input value={form[key]} onChange={e => set(key, e.target.value)} style={inputStyle} />
            }
          </div>
        ))}
        <div style={{ marginBottom: 18 }}>
          <label style={{ display: "block", color: "#8b949e", fontSize: 12, marginBottom: 6, textTransform: "uppercase", letterSpacing: 1 }}>Skills (comma separated)</label>
          <input value={form.skills.join(", ")} onChange={e => set("skills", e.target.value.split(",").map(s => s.trim()).filter(Boolean))} style={inputStyle} />
        </div>
        <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 14, marginBottom: 24 }}>
          {Object.entries(form.stats).map(([k, v]) => (
            <div key={k}>
              <label style={{ display: "block", color: "#8b949e", fontSize: 11, marginBottom: 5, textTransform: "capitalize", letterSpacing: 0.5 }}>{k}</label>
              <input type="number" value={v} onChange={e => setForm(f => ({ ...f, stats: { ...f.stats, [k]: Number(e.target.value) } }))} style={{ ...inputStyle, padding: "8px 12px" }} />
            </div>
          ))}
        </div>
        <div style={{ display: "flex", gap: 12 }}>
          <button onClick={() => { onSave(form); onClose(); }} style={{
            flex: 1, background: "linear-gradient(135deg, #00ffb4, #00c8ff)", border: "none",
            borderRadius: 10, padding: "12px", color: "#000", fontWeight: 800,
            fontFamily: "'Space Mono', monospace", fontSize: 14, cursor: "pointer",
            transition: "opacity 0.2s",
          }}
            onMouseEnter={e => e.target.style.opacity = 0.85}
            onMouseLeave={e => e.target.style.opacity = 1}
          >Save Changes</button>
          <button onClick={onClose} style={{ flex: 1, background: "rgba(48,54,61,0.6)", border: "1px solid rgba(48,54,61,0.8)", borderRadius: 10, padding: "12px", color: "#8b949e", fontFamily: "'Space Mono', monospace", fontSize: 14, cursor: "pointer" }}>Cancel</button>
        </div>
      </div>
    </div>
  );
}

const inputStyle = {
  width: "100%", background: "rgba(22,27,34,0.9)", border: "1px solid rgba(48,54,61,0.8)",
  borderRadius: 8, padding: "10px 14px", color: "#e6edf3", fontSize: 14,
  fontFamily: "inherit", outline: "none", boxSizing: "border-box",
  transition: "border-color 0.2s",
};

export default function GitHubProfile() {
  const [data, setData] = useState(INITIAL_DATA);
  const [editing, setEditing] = useState(false);
  const [activeTab, setActiveTab] = useState("overview");
  const [scanLine, setScanLine] = useState(0);
  const [typed, setTyped] = useState("");
  const fullText = "$ git status → all systems nominal ✓";

  useEffect(() => {
    let i = 0;
    const t = setInterval(() => {
      if (i <= fullText.length) { setTyped(fullText.slice(0, i)); i++; }
      else clearInterval(t);
    }, 60);
    return () => clearInterval(t);
  }, []);

  useEffect(() => {
    const t = setInterval(() => setScanLine(s => (s + 1) % 100), 30);
    return () => clearInterval(t);
  }, []);

  const tabs = ["overview", "repositories", "activity", "achievements"];

  return (
    <>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@400;700;800&display=swap');
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { background: #010409; }
        ::-webkit-scrollbar { width: 6px; } ::-webkit-scrollbar-track { background: #0d1117; } ::-webkit-scrollbar-thumb { background: #21262d; border-radius: 3px; }
        .glitch::before, .glitch::after {
          content: attr(data-text); position: absolute; top: 0; left: 0; width: 100%;
        }
        .glitch::before { left: 2px; text-shadow: -1px 0 #00c8ff; animation: glitch1 3s infinite; clip-path: polygon(0 0,100% 0,100% 45%,0 45%); }
        .glitch::after { left: -2px; text-shadow: 2px 0 #ff0055; animation: glitch2 3s infinite; clip-path: polygon(0 55%,100% 55%,100% 100%,0 100%); }
        @keyframes glitch1 { 0%,90%,100%{transform:translateX(0)} 92%{transform:translateX(-3px)} 96%{transform:translateX(3px)} }
        @keyframes glitch2 { 0%,90%,100%{transform:translateX(0)} 91%{transform:translateX(3px)} 95%{transform:translateX(-3px)} }
        @keyframes slideUp { from{transform:translateY(30px);opacity:0} to{transform:translateY(0);opacity:1} }
        @keyframes fadeIn { from{opacity:0;transform:translateY(20px)} to{opacity:1;transform:translateY(0)} }
        @keyframes pulse { 0%,100%{box-shadow:0 0 0 0 rgba(0,255,180,0.4)} 50%{box-shadow:0 0 0 12px rgba(0,255,180,0)} }
        @keyframes spin { to{transform:rotate(360deg)} }
        @keyframes float { 0%,100%{transform:translateY(0)} 50%{transform:translateY(-8px)} }
        @keyframes scanAnim { from{transform:translateY(0)} to{transform:translateY(100vh)} }
        .avatar-ring { animation: pulse 2.5s infinite; }
        .float-card { animation: float 4s ease-in-out infinite; }
        .section-in { animation: fadeIn 0.6s ease both; }
        .skill-tag:hover { background: rgba(0,255,180,0.2) !important; border-color: rgba(0,255,180,0.6) !important; transform: scale(1.08) !important; }
        .tab-btn { transition: all 0.2s; }
        .tab-btn:hover { color: #00ffb4 !important; }
      `}</style>

      <ParticleField />

      {/* Scan line */}
      <div style={{
        position: "fixed", left: 0, right: 0, height: 2, zIndex: 1,
        background: "linear-gradient(90deg, transparent, rgba(0,255,180,0.3), transparent)",
        top: `${scanLine}%`, pointerEvents: "none", transition: "top 0.05s linear",
      }} />

      <div style={{
        minHeight: "100vh", fontFamily: "'Syne', sans-serif", color: "#e6edf3",
        position: "relative", zIndex: 2,
      }}>
        {/* Header Bar */}
        <div style={{
          borderBottom: "1px solid rgba(48,54,61,0.6)", padding: "12px 32px",
          display: "flex", alignItems: "center", justifyContent: "space-between",
          backdropFilter: "blur(20px)", background: "rgba(13,17,23,0.8)",
          position: "sticky", top: 0, zIndex: 100,
        }}>
          <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
            <svg height="28" viewBox="0 0 16 16" fill="#e6edf3"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z" /></svg>
            <span style={{ fontFamily: "'Space Mono', monospace", fontSize: 13, color: "#00ffb4" }}>
              {typed}<span style={{ animation: "pulse 1s infinite", opacity: 0.7 }}>|</span>
            </span>
          </div>
          <button onClick={() => setEditing(true)} style={{
            background: "linear-gradient(135deg, rgba(0,255,180,0.15), rgba(0,200,255,0.1))",
            border: "1px solid rgba(0,255,180,0.35)", borderRadius: 8, padding: "8px 18px",
            color: "#00ffb4", fontFamily: "'Space Mono', monospace", fontSize: 12,
            cursor: "pointer", display: "flex", alignItems: "center", gap: 8,
            transition: "all 0.2s",
          }}
            onMouseEnter={e => { e.currentTarget.style.background = "rgba(0,255,180,0.2)"; e.currentTarget.style.transform = "scale(1.05)"; }}
            onMouseLeave={e => { e.currentTarget.style.background = "linear-gradient(135deg, rgba(0,255,180,0.15), rgba(0,200,255,0.1))"; e.currentTarget.style.transform = "scale(1)"; }}
          >✏️ Edit Profile</button>
        </div>

        <div style={{ maxWidth: 1100, margin: "0 auto", padding: "40px 24px" }}>
          {/* Profile Hero */}
          <div className="section-in" style={{
            display: "grid", gridTemplateColumns: "auto 1fr", gap: 40, marginBottom: 40,
            background: "rgba(13,17,23,0.7)", border: "1px solid rgba(48,54,61,0.6)",
            borderRadius: 20, padding: 36, backdropFilter: "blur(16px)",
            boxShadow: "0 0 60px rgba(0,255,180,0.04)",
          }}>
            {/* Avatar */}
            <div style={{ position: "relative" }}>
              <div className="avatar-ring" style={{
                width: 130, height: 130, borderRadius: "50%",
                background: "linear-gradient(135deg, #00ffb4, #00c8ff, #7c3aed)",
                padding: 3, flexShrink: 0,
              }}>
                <div style={{
                  width: "100%", height: "100%", borderRadius: "50%",
                  background: "linear-gradient(135deg, #1a2332, #0d1117)",
                  display: "flex", alignItems: "center", justifyContent: "center",
                  fontSize: 48, border: "3px solid #0d1117",
                }}>
                  {data.avatar ? <img src={data.avatar} style={{ width: "100%", height: "100%", borderRadius: "50%", objectFit: "cover" }} /> : "👨‍💻"}
                </div>
              </div>
              <div style={{
                position: "absolute", bottom: 4, right: 4, width: 22, height: 22,
                background: "#1f6feb", borderRadius: "50%", border: "3px solid #0d1117",
                display: "flex", alignItems: "center", justifyContent: "center", fontSize: 10,
              }}>✓</div>
            </div>

            {/* Info */}
            <div>
              <div style={{ display: "flex", alignItems: "center", gap: 14, flexWrap: "wrap", marginBottom: 6 }}>
                <h1 style={{ fontSize: 32, fontWeight: 800, color: "#e6edf3", lineHeight: 1 }}>
                  <GlitchText text={data.name} />
                </h1>
                <span style={{
                  background: "linear-gradient(135deg, rgba(0,255,180,0.15), rgba(0,200,255,0.1))",
                  border: "1px solid rgba(0,255,180,0.3)", borderRadius: 20, padding: "3px 12px",
                  fontSize: 11, color: "#00ffb4", fontFamily: "'Space Mono', monospace",
                }}>PRO</span>
              </div>
              <div style={{ color: "#8b949e", fontFamily: "'Space Mono', monospace", fontSize: 13, marginBottom: 12 }}>@{data.username}</div>
              <p style={{ color: "#c9d1d9", fontSize: 15, lineHeight: 1.7, marginBottom: 18, maxWidth: 520 }}>{data.bio}</p>
              <div style={{ display: "flex", flexWrap: "wrap", gap: 16, fontSize: 13, color: "#8b949e" }}>
                {data.location && <span>📍 {data.location}</span>}
                {data.website && <a href={data.website} style={{ color: "#58a6ff", textDecoration: "none" }}>🔗 {data.website}</a>}
                {data.company && <span>🏢 {data.company}</span>}
              </div>
            </div>
          </div>

          {/* Stats */}
          <div className="section-in" style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 14, marginBottom: 36 }}>
            <StatCard label="Repositories" value={data.stats.repos} />
            <StatCard label="Followers" value={data.stats.followers} />
            <StatCard label="Following" value={data.stats.following} />
            <StatCard label="Stars Earned" value={data.stats.stars} />
          </div>

          {/* Tabs */}
          <div style={{ display: "flex", gap: 4, marginBottom: 28, borderBottom: "1px solid rgba(48,54,61,0.5)", paddingBottom: 0 }}>
            {tabs.map(tab => (
              <button key={tab} className="tab-btn" onClick={() => setActiveTab(tab)} style={{
                background: "none", border: "none", borderBottom: `2px solid ${activeTab === tab ? "#f78166" : "transparent"}`,
                color: activeTab === tab ? "#e6edf3" : "#8b949e",
                fontFamily: "'Syne', sans-serif", fontWeight: 600, fontSize: 14,
                padding: "10px 18px", cursor: "pointer", textTransform: "capitalize",
                transition: "all 0.2s", marginBottom: -1,
              }}>{tab}</button>
            ))}
          </div>

          {/* Skills */}
          <div className="section-in" style={{ marginBottom: 32 }}>
            <h3 style={{ color: "#8b949e", fontSize: 12, textTransform: "uppercase", letterSpacing: 2, marginBottom: 14, fontFamily: "'Space Mono', monospace" }}>⚡ Tech Stack</h3>
            <div style={{ display: "flex", flexWrap: "wrap", gap: 8 }}>
              {data.skills.map((s, i) => (
                <span key={i} className="skill-tag" style={{
                  background: "rgba(0,255,180,0.06)", border: "1px solid rgba(0,255,180,0.2)",
                  borderRadius: 8, padding: "6px 14px", fontSize: 13, color: "#00ffb4",
                  fontFamily: "'Space Mono', monospace", cursor: "default",
                  transition: "all 0.2s", animationDelay: `${i * 0.1}s`,
                }}>{s}</span>
              ))}
            </div>
          </div>

          {/* Pinned Repos */}
          <div className="section-in" style={{ marginBottom: 36 }}>
            <h3 style={{ color: "#8b949e", fontSize: 12, textTransform: "uppercase", letterSpacing: 2, marginBottom: 18, fontFamily: "'Space Mono', monospace" }}>📌 Pinned Repositories</h3>
            <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fill, minmax(280px, 1fr))", gap: 16 }}>
              {data.pinnedRepos.map((repo, i) => <RepoCard key={i} repo={repo} />)}
            </div>
          </div>

          {/* Contribution Graph */}
          <div className="section-in" style={{
            background: "rgba(13,17,23,0.7)", border: "1px solid rgba(48,54,61,0.6)",
            borderRadius: 16, padding: "28px 28px 20px", backdropFilter: "blur(12px)",
          }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 20 }}>
              <h3 style={{ color: "#8b949e", fontSize: 12, textTransform: "uppercase", letterSpacing: 2, fontFamily: "'Space Mono', monospace" }}>
                🌿 Contribution Activity
              </h3>
              <span style={{ color: "#39d353", fontSize: 12, fontFamily: "'Space Mono', monospace" }}>
                {data.contributions.reduce((a, b) => a + b, 0)} contributions this year
              </span>
            </div>
            <ContributionGrid data={data.contributions} />
            <div style={{ display: "flex", gap: 6, alignItems: "center", justifyContent: "flex-end", marginTop: 12, fontSize: 11, color: "#8b949e" }}>
              <span>Less</span>
              {["#0d1117", "#0e4429", "#006d32", "#26a641", "#39d353"].map((c, i) => (
                <span key={i} style={{ width: 12, height: 12, borderRadius: 2, background: c, display: "inline-block" }} />
              ))}
              <span>More</span>
            </div>
          </div>

          {/* Footer */}
          <div style={{ textAlign: "center", marginTop: 40, color: "#484f58", fontSize: 12, fontFamily: "'Space Mono', monospace" }}>
            <span style={{ color: "#00ffb4" }}>●</span> All systems operational · Built with ❤️ & too much ☕
          </div>
        </div>
      </div>

      {editing && <EditModal data={data} onSave={setData} onClose={() => setEditing(false)} />}
    </>
  );
}
