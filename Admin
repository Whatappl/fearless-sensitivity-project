import { FormEvent, useEffect, useMemo, useState } from "react";

type Content = { id: number; kind: string; slug: string; title: string; body?: string | null; data?: string | null; published: number };
type Feedback = { id: number; message: string; rating?: number | null; status: "new" | "reviewed" | "resolved"; createdAt: string };
const emptyDraft = { kind: "device-preset", slug: "", title: "", body: "", data: "", published: false };

export default function Admin() {
  const [authenticated, setAuthenticated] = useState<boolean | null>(null);
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [content, setContent] = useState<Content[]>([]);
  const [feedback, setFeedback] = useState<Feedback[]>([]);
  const [draft, setDraft] = useState<typeof emptyDraft & { id?: number }>(emptyDraft);
  const [notice, setNotice] = useState("");
  const [menuOpen, setMenuOpen] = useState(false);
  const [search, setSearch] = useState("");
  const [contentFilter, setContentFilter] = useState("all");

  const load = async () => {
    const me = await fetch("/api/admin/me", { credentials: "include" }).then(r => r.json());
    setAuthenticated(Boolean(me.authenticated));
    if (me.authenticated) {
      const data = await fetch("/api/admin/content", { credentials: "include" }).then(r => r.json());
      setContent(data.content ?? []); setFeedback(data.feedback ?? []);
    }
  };
  useEffect(() => { void load(); }, []);

  const login = async (event: FormEvent) => {
    event.preventDefault(); setError("");
    const response = await fetch("/api/admin/login", { method: "POST", headers: { "Content-Type": "application/json" }, credentials: "include", body: JSON.stringify({ username, password }) });
    if (!response.ok) { setError("That username or password was not recognised."); return; }
    setPassword(""); await load();
  };
  const logout = async () => { await fetch("/api/admin/logout", { method: "POST", credentials: "include" }); setAuthenticated(false); setMenuOpen(false); };
  const save = async (event: FormEvent) => {
    event.preventDefault(); setNotice("");
    const response = await fetch("/api/admin/content", { method: "POST", headers: { "Content-Type": "application/json" }, credentials: "include", body: JSON.stringify(draft) });
    if (!response.ok) { setNotice("I couldn't save that yet. Please check the fields and try again."); return; }
    setDraft(emptyDraft); setNotice("Saved. It will be visible publicly when you publish it."); await load();
  };
  const remove = async (id: number) => { if (!window.confirm("Delete this item permanently?")) return; await fetch(`/api/admin/content/${id}`, { method: "DELETE", credentials: "include" }); await load(); };
  const edit = (item: Content) => { setDraft({ id: item.id, kind: item.kind, slug: item.slug, title: item.title, body: item.body ?? "", data: item.data ?? "", published: Boolean(item.published) }); window.scrollTo({ top: 0, behavior: "smooth" }); };
  const updateFeedback = async (id: number, patch: { status?: string; rating?: number | null }) => { await fetch(`/api/admin/feedback/${id}`, { method: "PATCH", headers: { "Content-Type": "application/json" }, credentials: "include", body: JSON.stringify(patch) }); await load(); };
  const removeFeedback = async (id: number) => { if (!window.confirm("Delete this feedback permanently?")) return; await fetch(`/api/admin/feedback/${id}`, { method: "DELETE", credentials: "include" }); await load(); };
  const filteredContent = useMemo(() => content.filter(item => (contentFilter === "all" || item.kind === contentFilter) && `${item.title} ${item.slug}`.toLowerCase().includes(search.toLowerCase())), [content, contentFilter, search]);
  const publishedCount = content.filter(item => item.published).length;
  const newFeedbackCount = feedback.filter(item => item.status === "new").length;

  if (authenticated === null) return <div className="admin-loading">Checking your secure session…</div>;
  if (!authenticated) return <main className="admin-login"><section className="admin-login-card"><div className="admin-kicker">FEARLESS FFH4X / PRIVATE AREA</div><h1>Welcome back</h1><p>Sign in to manage presets, updates, links, and follower feedback. Your password is checked on the server and never exposed in the browser.</p><form onSubmit={login}><label>Username<input value={username} onChange={e => setUsername(e.target.value)} autoComplete="username" placeholder="Enter your username" required /></label><label>Password<input type="password" value={password} onChange={e => setPassword(e.target.value)} autoComplete="current-password" placeholder="Enter your password" required /></label>{error && <div className="admin-error">{error}</div>}<button className="admin-primary">Sign in securely</button></form><a className="admin-back" href="/">← Back to the generator</a></section></main>;

  return <main className="admin-page"><header className="admin-header"><div><div className="admin-kicker">FEARLESS FFH4X / CONTROL ROOM</div><h1>Good to see you</h1><p>Keep the generator fresh, helpful, and ready for your followers.</p></div><div className="admin-menu-wrap"><button className="menu-button" onClick={() => setMenuOpen(value => !value)} aria-expanded={menuOpen}>Menu <span>☰</span></button>{menuOpen && <div className="admin-menu"><a href="#content" onClick={() => setMenuOpen(false)}>Manage content</a><a href="#feedback" onClick={() => setMenuOpen(false)}>Review feedback{newFeedbackCount ? ` (${newFeedbackCount} new)` : ""}</a><a href="/api/admin/export">Download backup</a><a href="/" target="_blank" rel="noreferrer">Open public generator</a><button onClick={() => void load()}>Refresh dashboard</button><button className="menu-danger" onClick={() => void logout()}>Sign out</button></div>}</div></header><section className="stats-row"><div><strong>{content.length}</strong><span>Total content items</span></div><div><strong>{publishedCount}</strong><span>Currently published</span></div><div><strong>{newFeedbackCount}</strong><span>New feedback</span></div><div><strong>{feedback.length}</strong><span>All feedback</span></div></section><section className="admin-grid"><div className="admin-panel" id="content"><div className="panel-heading"><div><span className="status-dot" /> Your dashboard is protected</div><a className="export-link" href="/api/admin/export">Download JSON backup</a></div><form className="content-form" onSubmit={save}><h2>{draft.id ? "Update content" : "Add something new"}</h2><p className="form-help">Create a device preset, daily tip, social link, featured device, or news update. Drafts stay private until you publish them.</p><div className="form-row"><label>What are you adding?<select value={draft.kind} onChange={e => setDraft({ ...draft, kind: e.target.value })}><option value="device-preset">Device preset</option><option value="daily-tip">Daily tip</option><option value="featured-device">Featured device</option><option value="social-link">Social link</option><option value="changelog">News or changelog</option><option value="setting">General setting</option></select></label><label>Short name<input value={draft.slug} onChange={e => setDraft({ ...draft, slug: e.target.value })} placeholder="for example, galaxy-a15" required /></label></div><label>Title<input value={draft.title} onChange={e => setDraft({ ...draft, title: e.target.value })} placeholder="Give this update a clear title" required /></label><label>Details<textarea value={draft.body} onChange={e => setDraft({ ...draft, body: e.target.value })} rows={4} placeholder="Write it the way you would explain it to a follower." /></label><label>Extra settings (optional JSON)<textarea value={draft.data} onChange={e => setDraft({ ...draft, data: e.target.value })} rows={4} placeholder='Only needed for structured preset or link data' /></label><label className="check-row"><input type="checkbox" checked={draft.published} onChange={e => setDraft({ ...draft, published: e.target.checked })} /> Show this to visitors now</label><div className="form-actions"><button className="admin-primary">{draft.id ? "Save update" : "Add to library"}</button>{draft.id && <button type="button" onClick={() => setDraft(emptyDraft)}>Start over</button>}</div>{notice && <div className="admin-notice">{notice}</div>}</form></div><div className="admin-panel"><div className="panel-heading"><h2>Content library</h2><span>{filteredContent.length} shown</span></div><div className="library-tools"><input value={search} onChange={e => setSearch(e.target.value)} placeholder="Search your content" /><select value={contentFilter} onChange={e => setContentFilter(e.target.value)}><option value="all">All types</option><option value="device-preset">Device presets</option><option value="daily-tip">Daily tips</option><option value="featured-device">Featured devices</option><option value="social-link">Social links</option><option value="changelog">News</option></select></div>{filteredContent.length === 0 ? <p className="empty-state">Nothing matches that search yet.</p> : <div className="content-list">{filteredContent.map(item => <article className="content-item" key={item.id}><div><span className="content-kind">{item.kind}</span><h3>{item.title}</h3><p>{item.slug} · {item.published ? "Visible to visitors" : "Private draft"}</p></div><div className="item-actions"><button onClick={() => edit(item)}>Edit</button><button className="danger" onClick={() => void remove(item.id)}>Delete</button></div></article>)}</div>}<div className="feedback-heading" id="feedback"><h2>Follower feedback</h2><span>{feedback.length} messages</span></div>{feedback.length === 0 ? <p className="empty-state">Messages sent through the public feedback button will appear here.</p> : feedback.map(item => <article className="feedback-item" key={item.id}><div className="feedback-copy"><strong>{item.rating ? `${item.rating}/5` : "No rating yet"}</strong><span>{item.message}</span><small>{new Date(item.createdAt).toLocaleString()}</small></div><div className="feedback-actions"><select value={item.status} onChange={e => void updateFeedback(item.id, { status: e.target.value })}><option value="new">New</option><option value="reviewed">Reviewed</option><option value="resolved">Resolved</option></select><button className="danger" onClick={() => void removeFeedback(item.id)}>Delete</button></div></article>)}</div></section></main>;
}
