# jarvis
Questo repository serve per creare un asistente online totalmente autonomo
import { useState, useRef, useEffect } from "react";

const MOCK_USERS = [
  { id: 1, name: "Sofia R.", avatar: "SR", role: "Designer", online: true },
  { id: 2, name: "Marco V.", avatar: "MV", role: "Developer", online: true },
  { id: 3, name: "Giulia T.", avatar: "GT", role: "Writer", online: false },
  { id: 4, name: "Luca N.", avatar: "LN", role: "Photographer", online: true },
];

const MOCK_POSTS = [
  {
    id: 1, user: MOCK_USERS[0],
    content: "Ho appena finito un nuovo progetto di branding! Che ne pensate del minimalismo nel design moderno?",
    likes: 24, time: "2h fa", liked: false,
  },
  {
    id: 2, user: MOCK_USERS[1],
    content: "Qualcuno ha esperienza con React Server Components in produzione? Sto cercando best practices.",
    likes: 17, time: "4h fa", liked: false,
  },
  {
    id: 3, user: MOCK_USERS[3],
    content: "Nuova serie fotografica in lavorazione: street art di Napoli 🎨 Prima preview questa settimana!",
    likes: 41, time: "6h fa", liked: false,
  },
];

const EXPLORE_CARDS = [
  { icon: "🎨", title: "Design & Art", desc: "Condividi progetti creativi e ispirazioni visive", count: "1.2k membri" },
  { icon: "💻", title: "Tech & Dev", desc: "Discussioni su codice, tools e innovazione", count: "890 membri" },
  { icon: "📸", title: "Fotografia", desc: "Gallery, feedback e sfide fotografiche", count: "654 membri" },
  { icon: "✍️", title: "Scrittura", desc: "Narrativa, copywriting e storytelling", count: "432 membri" },
  { icon: "🎵", title: "Musica", desc: "Produzione, ascolti e collaborazioni", count: "378 membri" },
  { icon: "🌱", title: "Sostenibilità", desc: "Idee e progetti per un futuro migliore", count: "290 membri" },
];

const TABS = ["Feed", "Esplora", "AI Assistant"];

const css = `
  @import url('https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Sans:ital,wght@0,300;0,400;0,500;1,300&display=swap');
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  :root {
    --bg: #0a0a0f; --surface: #13131a; --surface2: #1c1c28; --border: #2a2a3a;
    --accent: #c8f05a; --accent2: #7c5cfc; --text: #f0f0f5; --muted: #6b6b85; --danger: #ff4d6d;
  }
  body { background: var(--bg); color: var(--text); font-family: 'DM Sans', sans-serif; }
  .app { min-height: 100vh; max-width: 900px; margin: 0 auto; padding: 0 16px; }
  .header {
    display: flex; align-items: center; justify-content: space-between;
    padding: 20px 0 16px; border-bottom: 1px solid var(--border);
    position: sticky; top: 0; background: var(--bg); z-index: 100;
  }
  .logo { font-family: 'Syne', sans-serif; font-weight: 800; font-size: 1.4rem; letter-spacing: -0.04em; display: flex; align-items: center; gap: 8px; }
  .logo-dot { width: 10px; height: 10px; border-radius: 50%; background: var(--accent); box-shadow: 0 0 12px var(--accent); animation: pulse 2s ease-in-out infinite; }
  @keyframes pulse { 0%,100%{opacity:1;transform:scale(1)} 50%{opacity:.6;transform:scale(.8)} }
  .online-count { font-size: .75rem; color: var(--muted); display: flex; align-items: center; gap: 5px; }
  .online-dot { width: 6px; height: 6px; border-radius: 50%; background: var(--accent); }
  .tabs { display: flex; gap: 4px; padding: 16px 0 12px; border-bottom: 1px solid var(--border); position: sticky; top: 61px; background: var(--bg); z-index: 99; }
  .tab-btn { padding: 7px 18px; border-radius: 100px; border: 1px solid var(--border); background: transparent; color: var(--muted); font-family: 'DM Sans',sans-serif; font-size: .85rem; font-weight: 500; cursor: pointer; transition: all .2s; }
  .tab-btn:hover { border-color: var(--accent2); color: var(--text); }
  .tab-btn.active { background: var(--accent); border-color: var(--accent); color: #0a0a0f; font-weight: 700; box-shadow: 0 0 20px rgba(200,240,90,.25); }
  .main { display: grid; grid-template-columns: 1fr 260px; gap: 24px; padding: 20px 0 40px; align-items: start; }
  @media(max-width:640px){ .main{grid-template-columns:1fr} .sidebar{display:none} }
  .composer { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 20px; margin-bottom: 20px; }
  .composer-top { display: flex; gap: 12px; align-items: flex-start; }
  .avatar { width: 38px; height: 38px; border-radius: 50%; background: linear-gradient(135deg,var(--accent2),var(--accent)); display: flex; align-items: center; justify-content: center; font-family: 'Syne',sans-serif; font-size: .75rem; font-weight: 700; color: white; flex-shrink: 0; }
  .avatar.sm { width: 32px; height: 32px; font-size: .7rem; }
  .avatar.online { box-shadow: 0 0 0 2px var(--bg),0 0 0 4px var(--accent); }
  .composer textarea { width: 100%; background: transparent; border: none; outline: none; color: var(--text); font-family: 'DM Sans',sans-serif; font-size: .95rem; resize: none; min-height: 70px; line-height: 1.6; }
  .composer textarea::placeholder { color: var(--muted); }
  .composer-actions { display: flex; justify-content: flex-end; gap: 8px; margin-top: 12px; padding-top: 12px; border-top: 1px solid var(--border); }
  .btn-ghost { padding: 7px 16px; border-radius: 8px; border: 1px solid var(--border); background: transparent; color: var(--muted); font-family: 'DM Sans',sans-serif; font-size: .82rem; cursor: pointer; transition: all .2s; }
  .btn-ghost:hover { border-color: var(--accent2); color: var(--text); }
  .btn-primary { padding: 7px 20px; border-radius: 8px; border: none; background: var(--accent); color: #0a0a0f; font-family: 'Syne',sans-serif; font-weight: 700; font-size: .85rem; cursor: pointer; transition: all .2s; }
  .btn-primary:hover { filter: brightness(1.1); box-shadow: 0 4px 15px rgba(200,240,90,.3); }
  .btn-primary:disabled { opacity: .4; cursor: not-allowed; }
  .post-card { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 20px; margin-bottom: 14px; animation: slideIn .3s ease; }
  @keyframes slideIn { from{opacity:0;transform:translateY(-10px)} to{opacity:1;transform:translateY(0)} }
  .post-header { display: flex; gap: 12px; align-items: center; margin-bottom: 14px; }
  .post-user-name { font-family: 'Syne',sans-serif; font-weight: 700; font-size: .9rem; }
  .post-meta { display: flex; gap: 8px; align-items: center; margin-top: 2px; }
  .post-role { font-size: .75rem; color: var(--accent2); font-weight: 500; }
  .post-time { font-size: .75rem; color: var(--muted); }
  .post-content { font-size: .93rem; line-height: 1.65; color: #d0d0e0; margin-bottom: 16px; }
  .post-actions { display: flex; gap: 16px; }
  .action-btn { background: none; border: none; cursor: pointer; color: var(--muted); font-family: 'DM Sans',sans-serif; font-size: .82rem; display: flex; align-items: center; gap: 5px; padding: 5px 10px; border-radius: 8px; transition: all .2s; }
  .action-btn:hover { background: var(--surface2); color: var(--text); }
  .action-btn.liked { color: var(--danger); }
  .action-btn.ai-btn { color: var(--accent2); }
  .action-btn.ai-btn:hover { background: rgba(124,92,252,.12); }
  .ai-reply { background: var(--surface2); border: 1px solid rgba(124,92,252,.3); border-radius: 12px; padding: 14px 16px; margin-top: 14px; font-size: .87rem; line-height: 1.65; color: #c8c8e0; animation: slideIn .3s ease; }
  .ai-reply-label { font-family: 'Syne',sans-serif; font-size: .72rem; font-weight: 700; color: var(--accent2); text-transform: uppercase; letter-spacing: .08em; margin-bottom: 8px; display: flex; align-items: center; gap: 6px; }
  .typing-dots span { display: inline-block; width: 6px; height: 6px; border-radius: 50%; background: var(--accent2); margin: 0 2px; animation: bounce 1.2s ease-in-out infinite; }
  .typing-dots span:nth-child(2){animation-delay:.2s} .typing-dots span:nth-child(3){animation-delay:.4s}
  @keyframes bounce{0%,80%,100%{transform:translateY(0);opacity:.5}40%{transform:translateY(-6px);opacity:1}}
  .sidebar { display: flex; flex-direction: column; gap: 16px; }
  .sidebar-card { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; padding: 18px; }
  .sidebar-title { font-family: 'Syne',sans-serif; font-weight: 700; font-size: .85rem; color: var(--muted); text-transform: uppercase; letter-spacing: .08em; margin-bottom: 14px; }
  .member-row { display: flex; gap: 10px; align-items: center; margin-bottom: 12px; }
  .member-info { flex: 1; }
  .member-name { font-size: .87rem; font-weight: 500; }
  .member-role { font-size: .74rem; color: var(--muted); }
  .online-badge { width: 8px; height: 8px; border-radius: 50%; background: var(--accent); box-shadow: 0 0 6px var(--accent); flex-shrink: 0; }
  .offline-badge { width: 8px; height: 8px; border-radius: 50%; background: var(--border); flex-shrink: 0; }
  .tag-list { display: flex; flex-wrap: wrap; gap: 6px; }
  .tag { padding: 4px 12px; border-radius: 100px; background: var(--surface2); border: 1px solid var(--border); font-size: .78rem; color: var(--muted); cursor: pointer; transition: all .2s; }
  .tag:hover { border-color: var(--accent2); color: var(--accent2); }
  .ai-panel { background: var(--surface); border: 1px solid var(--border); border-radius: 16px; height: 520px; display: flex; flex-direction: column; overflow: hidden; }
  .ai-panel-header { padding: 18px 20px; border-bottom: 1px solid var(--border); display: flex; align-items: center; gap: 10px; }
  .ai-avatar { width: 36px; height: 36px; border-radius: 12px; background: linear-gradient(135deg,var(--accent2),#ff6b9d); display: flex; align-items: center; justify-content: center; font-size: 1rem; box-shadow: 0 0 20px rgba(124,92,252,.4); }
  .ai-name { font-family: 'Syne',sans-serif; font-weight: 700; font-size: .92rem; }
  .ai-status { font-size: .75rem; color: var(--accent); }
  .ai-messages { flex: 1; overflow-y: auto; padding: 16px; display: flex; flex-direction: column; gap: 12px; }
  .ai-messages::-webkit-scrollbar{width:4px} .ai-messages::-webkit-scrollbar-track{background:transparent} .ai-messages::-webkit-scrollbar-thumb{background:var(--border);border-radius:4px}
  .msg { max-width: 78%; padding: 10px 14px; border-radius: 12px; font-size: .88rem; line-height: 1.6; animation: slideIn .2s ease; }
  .msg.user { background: var(--accent2); color: white; align-self: flex-end; border-bottom-right-radius: 4px; }
  .msg.ai { background: var(--surface2); color: #d0d0e0; align-self: flex-start; border-bottom-left-radius: 4px; border: 1px solid var(--border); }
  .ai-input-row { padding: 14px 16px; border-top: 1px solid var(--border); display: flex; gap: 8px; align-items: center; }
  .ai-input { flex: 1; background: var(--surface2); border: 1px solid var(--border); border-radius: 10px; padding: 9px 14px; color: var(--text); font-family: 'DM Sans',sans-serif; font-size: .88rem; outline: none; transition: border-color .2s; }
  .ai-input::placeholder{color:var(--muted)} .ai-input:focus{border-color:var(--accent2)}
  .send-btn { width: 38px; height: 38px; border-radius: 10px; background: var(--accent2); border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 1rem; transition: all .2s; flex-shrink: 0; }
  .send-btn:hover { filter: brightness(1.2); box-shadow: 0 4px 12px rgba(124,92,252,.4); }
  .send-btn:disabled { opacity: .4; cursor: not-allowed; }
  .explore-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
  .explore-card { background: var(--surface); border: 1px solid var(--border); border-radius: 14px; padding: 18px; cursor: pointer; transition: all .2s; }
  .explore-card:hover { border-color: var(--accent2); transform: translateY(-2px); }
  .explore-icon { font-size: 1.6rem; margin-bottom: 8px; }
  .explore-title { font-family: 'Syne',sans-serif; font-weight: 700; font-size: .9rem; margin-bottom: 4px; }
  .explore-desc { font-size: .78rem; color: var(--muted); line-height: 1.5; }
  .explore-count { font-size: .75rem; color: var(--accent2); margin-top: 8px; font-weight: 500; }
`;

export default function App() {
  const [activeTab, setActiveTab] = useState("Feed");
  const [posts, setPosts] = useState(MOCK_POSTS);
  const [newPost, setNewPost] = useState("");
  const [aiLoading, setAiLoading] = useState({});
  const [aiReplies, setAiReplies] = useState({});
  const [chatMessages, setChatMessages] = useState([
    { role: "ai", text: "Ciao! 👋 Sono l'assistente AI di Nexus. Posso aiutarti a scrivere post migliori, trovare persone con interessi simili o rispondere alle tue domande. Come posso aiutarti?" }
  ]);
  const [chatInput, setChatInput] = useState("");
  const [chatLoading, setChatLoading] = useState(false);
  const messagesEndRef = useRef(null);

  useEffect(() => { messagesEndRef.current?.scrollIntoView({ behavior: "smooth" }); }, [chatMessages]);

  const handlePost = () => {
    if (!newPost.trim()) return;
    setPosts([{
      id: Date.now(), user: { id: 0, name: "Tu", avatar: "TU", role: "Membro", online: true },
      content: newPost, likes: 0, time: "adesso", liked: false,
    }, ...posts]);
    setNewPost("");
  };

  const toggleLike = (id) => {
    setPosts(posts.map(p => p.id === id ? { ...p, liked: !p.liked, likes: p.liked ? p.likes - 1 : p.likes + 1 } : p));
  };

  const getAiComment = async (post) => {
    setAiLoading(prev => ({ ...prev, [post.id]: true }));
    setAiReplies(prev => ({ ...prev, [post.id]: null }));
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514", max_tokens: 1000,
          system: "Sei un membro intelligente e coinvolgente di una community creativa italiana. Rispondi ai post in modo naturale, stimolante e amichevole in italiano. Usa 2-3 frasi. Puoi fare domande per stimolare la discussione.",
          messages: [{ role: "user", content: `Commenta questo post di ${post.user.name} (${post.user.role}): "${post.content}"` }]
        })
      });
      const data = await res.json();
      setAiReplies(prev => ({ ...prev, [post.id]: data.content?.map(b => b.text || "").join("") || "Ottimo spunto!" }));
    } catch {
      setAiReplies(prev => ({ ...prev, [post.id]: "Errore nel caricamento della risposta AI." }));
    }
    setAiLoading(prev => ({ ...prev, [post.id]: false }));
  };

  const sendChat = async () => {
    if (!chatInput.trim() || chatLoading) return;
    const userMsg = { role: "user", text: chatInput };
    const history = [...chatMessages, userMsg];
    setChatMessages(history);
    setChatInput("");
    setChatLoading(true);
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514", max_tokens: 1000,
          system: "Sei l'assistente AI di 'Nexus', una community creativa italiana. Aiuti i membri a scrivere, trovare ispirazioni e connettersi. Sei amichevole e conciso. Rispondi sempre in italiano.",
          messages: history.map(m => ({ role: m.role === "user" ? "user" : "assistant", content: m.text }))
        })
      });
      const data = await res.json();
      setChatMessages(prev => [...prev, { role: "ai", text: data.content?.map(b => b.text || "").join("") || "..." }]);
    } catch {
      setChatMessages(prev => [...prev, { role: "ai", text: "Errore di connessione. Riprova." }]);
    }
    setChatLoading(false);
  };

  return (
    <>
      <style>{css}</style>
      <div className="app">
        <header className="header">
          <div className="logo"><div className="logo-dot" />Nexus</div>
          <div className="online-count"><div className="online-dot" />{MOCK_USERS.filter(u => u.online).length} online ora</div>
        </header>

        <div className="tabs">
          {TABS.map(t => <button key={t} className={`tab-btn${activeTab === t ? " active" : ""}`} onClick={() => setActiveTab(t)}>{t}</button>)}
        </div>

        <div className="main">
          <div>
            {activeTab === "Feed" && (
              <>
                <div className="composer">
                  <div className="composer-top">
                    <div className="avatar online">TU</div>
                    <textarea placeholder="Condividi qualcosa con la community..." value={newPost}
                      onChange={e => setNewPost(e.target.value)}
                      onKeyDown={e => { if (e.key === "Enter" && e.metaKey) handlePost(); }} />
                  </div>
                  <div className="composer-actions">
                    <button className="btn-ghost">🔗 Link</button>
                    <button className="btn-primary" onClick={handlePost} disabled={!newPost.trim()}>Pubblica</button>
                  </div>
                </div>

                {posts.map(post => (
                  <div key={post.id} className="post-card">
                    <div className="post-header">
                      <div className={`avatar${post.user.online ? " online" : ""}`}>{post.user.avatar}</div>
                      <div style={{ flex: 1 }}>
                        <div className="post-user-name">{post.user.name}</div>
                        <div className="post-meta">
                          <span className="post-role">{post.user.role}</span>
                          <span className="post-time">{post.time}</span>
                        </div>
                      </div>
                    </div>
                    <div className="post-content">{post.content}</div>
                    <div className="post-actions">
                      <button className={`action-btn${post.liked ? " liked" : ""}`} onClick={() => toggleLike(post.id)}>
                        {post.liked ? "❤️" : "🤍"} {post.likes}
                      </button>
                      <button className="action-btn">💬 Commenta</button>
                      <button className="action-btn ai-btn" onClick={() => getAiComment(post)} disabled={aiLoading[post.id]}>
                        ✦ AI risponde
                      </button>
                    </div>
                    {aiLoading[post.id] && (
                      <div className="ai-reply">
                        <div className="ai-reply-label">✦ Nexus AI</div>
                        <div className="typing-dots"><span /><span /><span /></div>
                      </div>
                    )}
                    {aiReplies[post.id] && !aiLoading[post.id] && (
                      <div className="ai-reply">
                        <div className="ai-reply-label">✦ Nexus AI</div>
                        {aiReplies[post.id]}
                      </div>
                    )}
                  </div>
                ))}
              </>
            )}

            {activeTab === "Esplora" && (
              <div className="explore-grid">
                {EXPLORE_CARDS.map((c, i) => (
                  <div key={i} className="explore-card">
                    <div className="explore-icon">{c.icon}</div>
                    <div className="explore-title">{c.title}</div>
                    <div className="explore-desc">{c.desc}</div>
                    <div className="explore-count">👥 {c.count}</div>
                  </div>
                ))}
              </div>
            )}

            {activeTab === "AI Assistant" && (
              <div className="ai-panel">
                <div className="ai-panel-header">
                  <div className="ai-avatar">✦</div>
                  <div>
                    <div className="ai-name">Nexus AI</div>
                    <div className="ai-status">● Online</div>
                  </div>
                </div>
                <div className="ai-messages">
                  {chatMessages.map((m, i) => <div key={i} className={`msg ${m.role}`}>{m.text}</div>)}
                  {chatLoading && <div className="msg ai"><div className="typing-dots"><span /><span /><span /></div></div>}
                  <div ref={messagesEndRef} />
                </div>
                <div className="ai-input-row">
                  <input className="ai-input" placeholder="Chiedi qualcosa..." value={chatInput}
                    onChange={e => setChatInput(e.target.value)}
                    onKeyDown={e => { if (e.key === "Enter") sendChat(); }}
                    disabled={chatLoading} />
                  <button className="send-btn" onClick={sendChat} disabled={!chatInput.trim() || chatLoading}>➤</button>
                </div>
              </div>
            )}
          </div>

          <aside className="sidebar">
            <div className="sidebar-card">
              <div className="sidebar-title">Membri Attivi</div>
              {MOCK_USERS.map(u => (
                <div key={u.id} className="member-row">
                  <div className={`avatar sm${u.online ? " online" : ""}`}>{u.avatar}</div>
                  <div className="member-info">
                    <div className="member-name">{u.name}</div>
                    <div className="member-role">{u.role}</div>
                  </div>
                  <div className={u.online ? "online-badge" : "offline-badge"} />
                </div>
              ))}
            </div>
            <div className="sidebar-card">
              <div className="sidebar-title">Tag Popolari</div>
              <div className="tag-list">
                {["#design", "#react", "#AI", "#napoli", "#foto", "#ux", "#startup", "#creatività"].map(t => (
                  <span key={t} className="tag">{t}</span>
                ))}
              </div>
            </div>
          </aside>
        </div>
      </div>
    </>
  );
}

