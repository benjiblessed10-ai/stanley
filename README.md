[Uploimport { useState, useRef, useEffect } from "react";

const SYSTEM_PROMPT = `You are Nova, a friendly, thoughtful, and highly capable AI assistant. You are helpful, honest, and clear. You enjoy engaging conversations and give thorough but concise answers. You have a warm personality and a touch of wit. When you don't know something, you say so. You never make things up.`;

const SUGGESTED = [
  { icon: "💡", text: "Explain quantum computing simply" },
  { icon: "✍️", text: "Write a short poem about the ocean" },
  { icon: "🧮", text: "Help me solve a math problem" },
  { icon: "🌍", text: "Tell me something fascinating about space" },
];

function TypingDots() {
  return (
    <div style={{ display: "flex", gap: 5, alignItems: "center", padding: "4px 0" }}>
      {[0, 1, 2].map((i) => (
        <span key={i} style={{
          width: 7, height: 7, borderRadius: "50%", background: "#A78BFA",
          display: "inline-block",
          animation: `bounce 1.2s ease-in-out ${i * 0.2}s infinite`,
        }} />
      ))}
    </div>
  );
}

function Message({ msg }) {
  const isUser = msg.role === "user";
  return (
    <div style={{
      display: "flex",
      flexDirection: isUser ? "row-reverse" : "row",
      alignItems: "flex-start",
      gap: 12,
      margin: "18px 0",
    }}>
      {/* Avatar */}
      <div style={{
        width: 36, height: 36, borderRadius: "50%", flexShrink: 0,
        background: isUser ? "linear-gradient(135deg,#6366F1,#8B5CF6)" : "linear-gradient(135deg,#0EA5E9,#6366F1)",
        display: "flex", alignItems: "center", justifyContent: "center",
        fontSize: 16, color: "#fff", fontWeight: 700,
        boxShadow: "0 2px 8px rgba(99,102,241,0.3)",
      }}>
        {isUser ? "U" : "N"}
      </div>

      {/* Bubble */}
      <div style={{
        maxWidth: "72%",
        background: isUser ? "linear-gradient(135deg,#6366F1,#7C3AED)" : "#1E293B",
        color: isUser ? "#fff" : "#E2E8F0",
        borderRadius: isUser ? "18px 4px 18px 18px" : "4px 18px 18px 18px",
        padding: "13px 17px",
        fontSize: 14.5,
        lineHeight: 1.75,
        whiteSpace: "pre-wrap",
        boxShadow: isUser
          ? "0 4px 16px rgba(99,102,241,0.25)"
          : "0 2px 8px rgba(0,0,0,0.3)",
      }}>
        {msg.content}
      </div>
    </div>
  );
}

export default function NovaAI() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");
  const bottomRef = useRef(null);
  const textareaRef = useRef(null);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages, loading]);

  const sendMessage = async (text) => {
    const content = (text || input).trim();
    if (!content || loading) return;
    setInput("");
    setError("");

    const userMsg = { role: "user", content };
    const newMessages = [...messages, userMsg];
    setMessages(newMessages);
    setLoading(true);

    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          system: SYSTEM_PROMPT,
          messages: newMessages,
        }),
      });

      const data = await res.json();
      const reply = data.content?.map((b) => b.text || "").join("\n") || "Sorry, I couldn't respond.";
      setMessages((prev) => [...prev, { role: "assistant", content: reply }]);
    } catch (e) {
      setError("Connection error. Please try again.");
    } finally {
      setLoading(false);
    }
  };

  const handleKey = (e) => {
    if (e.key === "Enter" && !e.shiftKey) {
      e.preventDefault();
      sendMessage();
    }
  };

  const clearChat = () => {
    setMessages([]);
    setError("");
  };

  const isEmpty = messages.length === 0;

  return (
    <div style={{
      minHeight: "100vh", background: "#0A0F1E",
      display: "flex", flexDirection: "column",
      fontFamily: "'Segoe UI', system-ui, sans-serif",
    }}>
      {/* Header */}
      <div style={{
        display: "flex", alignItems: "center", justifyContent: "space-between",
        padding: "16px 24px",
        background: "rgba(255,255,255,0.03)",
        borderBottom: "1px solid rgba(255,255,255,0.07)",
        backdropFilter: "blur(10px)",
        position: "sticky", top: 0, zIndex: 10,
      }}>
        <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
          <div style={{
            width: 40, height: 40, borderRadius: 12,
            background: "linear-gradient(135deg,#0EA5E9,#6366F1,#8B5CF6)",
            display: "flex", alignItems: "center", justifyContent: "center",
            fontSize: 20, boxShadow: "0 4px 16px rgba(99,102,241,0.4)",
          }}>✦</div>
          <div>
            <div style={{ color: "#F1F5F9", fontWeight: 700, fontSize: 18, letterSpacing: "-0.3px" }}>Nova</div>
            <div style={{ color: "#22C55E", fontSize: 12, display: "flex", alignItems: "center", gap: 5 }}>
              <span style={{ width: 6, height: 6, borderRadius: "50%", background: "#22C55E", display: "inline-block" }} />
              Online
            </div>
          </div>
        </div>
        {!isEmpty && (
          <button onClick={clearChat} style={{
            background: "rgba(255,255,255,0.07)", border: "1px solid rgba(255,255,255,0.1)",
            color: "#94A3B8", borderRadius: 8, padding: "7px 14px", fontSize: 13,
            cursor: "pointer", fontWeight: 500,
          }}>
            New chat
          </button>
        )}
      </div>

      {/* Chat area */}
      <div style={{ flex: 1, overflowY: "auto", padding: "0 20px" }}>
        <div style={{ maxWidth: 740, margin: "0 auto" }}>
          {isEmpty ? (
            /* Welcome screen */
            <div style={{ textAlign: "center", paddingTop: 80 }}>
              <div style={{
                width: 72, height: 72, borderRadius: 20,
                background: "linear-gradient(135deg,#0EA5E9,#6366F1,#8B5CF6)",
                display: "flex", alignItems: "center", justifyContent: "center",
                fontSize: 36, margin: "0 auto 24px",
                boxShadow: "0 8px 32px rgba(99,102,241,0.4)",
              }}>✦</div>
              <h1 style={{
                color: "#F1F5F9", fontSize: 32, fontWeight: 800,
                letterSpacing: "-0.5px", margin: "0 0 10px",
              }}>Hi, I'm Nova</h1>
              <p style={{ color: "#64748B", fontSize: 16, margin: "0 0 48px", lineHeight: 1.6 }}>
                Your AI assistant — ask me anything.
              </p>

              {/* Suggestions */}
              <div style={{
                display: "grid", gridTemplateColumns: "1fr 1fr",
                gap: 12, maxWidth: 560, margin: "0 auto",
              }}>
                {SUGGESTED.map((s) => (
                  <button
                    key={s.text}
                    onClick={() => sendMessage(s.text)}
                    style={{
                      background: "rgba(255,255,255,0.04)",
                      border: "1px solid rgba(255,255,255,0.09)",
                      borderRadius: 14, padding: "16px",
                      cursor: "pointer", textAlign: "left",
                      color: "#CBD5E1", fontSize: 13.5, lineHeight: 1.5,
                      transition: "all 0.15s",
                    }}
                    onMouseEnter={(e) => {
                      e.currentTarget.style.background = "rgba(99,102,241,0.12)";
                      e.currentTarget.style.borderColor = "rgba(99,102,241,0.35)";
                    }}
                    onMouseLeave={(e) => {
                      e.currentTarget.style.background = "rgba(255,255,255,0.04)";
                      e.currentTarget.style.borderColor = "rgba(255,255,255,0.09)";
                    }}
                  >
                    <span style={{ fontSize: 20, display: "block", marginBottom: 6 }}>{s.icon}</span>
                    {s.text}
                  </button>
                ))}
              </div>
            </div>
          ) : (
            <div style={{ paddingTop: 20, paddingBottom: 20 }}>
              {messages.map((msg, i) => <Message key={i} msg={msg} />)}
              {loading && (
                <div style={{ display: "flex", alignItems: "flex-start", gap: 12, margin: "18px 0" }}>
                  <div style={{
                    width: 36, height: 36, borderRadius: "50%", flexShrink: 0,
                    background: "linear-gradient(135deg,#0EA5E9,#6366F1)",
                    display: "flex", alignItems: "center", justifyContent: "center",
                    fontSize: 16, color: "#fff", fontWeight: 700,
                  }}>N</div>
                  <div style={{
                    background: "#1E293B", borderRadius: "4px 18px 18px 18px",
                    padding: "13px 17px",
                  }}>
                    <TypingDots />
                  </div>
                </div>
              )}
              {error && (
                <div style={{
                  background: "rgba(239,68,68,0.1)", border: "1px solid rgba(239,68,68,0.2)",
                  color: "#FCA5A5", borderRadius: 10, padding: "10px 16px",
                  fontSize: 13, margin: "10px 0",
                }}>⚠ {error}</div>
              )}
              <div ref={bottomRef} />
            </div>
          )}
        </div>
      </div>

      {/* Input area */}
      <div style={{
        padding: "16px 20px 24px",
        background: "rgba(10,15,30,0.95)",
        borderTop: "1px solid rgba(255,255,255,0.06)",
        backdropFilter: "blur(10px)",
      }}>
        <div style={{ maxWidth: 740, margin: "0 auto" }}>
          <div style={{
            display: "flex", alignItems: "flex-end", gap: 10,
            background: "rgba(255,255,255,0.06)",
            border: "1px solid rgba(255,255,255,0.1)",
            borderRadius: 16, padding: "10px 10px 10px 18px",
            transition: "border-color 0.15s",
          }}
            onFocus={() => {}}
          >
            <textarea
              ref={textareaRef}
              value={input}
              onChange={(e) => {
                setInput(e.target.value);
                e.target.style.height = "auto";
                e.target.style.height = Math.min(e.target.scrollHeight, 160) + "px";
              }}
              onKeyDown={handleKey}
              placeholder="Message Nova…"
              rows={1}
              style={{
                flex: 1, background: "transparent", border: "none", outline: "none",
                color: "#F1F5F9", fontSize: 15, lineHeight: 1.6,
                resize: "none", fontFamily: "inherit",
                maxHeight: 160, overflowY: "auto",
              }}
            />
            <button
              onClick={() => sendMessage()}
              disabled={!input.trim() || loading}
              style={{
                width: 40, height: 40, borderRadius: 11, border: "none",
                background: input.trim() && !loading
                  ? "linear-gradient(135deg,#6366F1,#8B5CF6)"
                  : "rgba(255,255,255,0.1)",
                color: input.trim() && !loading ? "#fff" : "#475569",
                cursor: input.trim() && !loading ? "pointer" : "not-allowed",
                display: "flex", alignItems: "center", justifyContent: "center",
                fontSize: 18, flexShrink: 0,
                transition: "all 0.15s",
                boxShadow: input.trim() && !loading ? "0 4px 12px rgba(99,102,241,0.4)" : "none",
              }}
            >
              ↑
            </button>
          </div>
          <div style={{ textAlign: "center", color: "#334155", fontSize: 12, marginTop: 10 }}>
            Nova can make mistakes. Use good judgment.
          </div>
        </div>
      </div>

      <style>{`
        @keyframes bounce {
          0%, 80%, 100% { transform: translateY(0); opacity: 0.4; }
          40% { transform: translateY(-6px); opacity: 1; }
        }
        * { box-sizing: border-box; }
        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.1); border-radius: 4px; }
      `}</style>
    </div>
  );
}
ading nova-ai.jsx…]()
