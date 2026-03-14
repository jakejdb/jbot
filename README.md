import { useState, useEffect, useRef } from "react";
import { AreaChart, Area, XAxis, YAxis, Tooltip, ResponsiveContainer } from "recharts";

const MARKETS = ["NFL","NBA","BTC","ETH","SPY","SOL","MLB","FOREX","NHL","AAPL"];
const ACTIONS = ["LONG","SHORT","BET","HEDGE","SCALP"];
const OUTCOMES = ["WIN","WIN","WIN","LOSS","PENDING"];
const TARGETS = [
  "Chiefs -3.5","Lakers ML","BTC/USD","ETH/BTC","SPY 500C",
  "SOL/USDT","Yankees -1.5","EUR/USD","BNB/ETH","Celtics +2",
  "Broncos +7","Heat -4.5","AAPL 200C","XRP/USDT","GBP/JPY"
];

function genTrade(id) {
  const outcome = OUTCOMES[Math.floor(Math.random() * OUTCOMES.length)];
  const stake = (Math.random() * 400 + 50).toFixed(0);
  const pnl = outcome === "WIN" ? +(Math.random() * 300 + 20).toFixed(2)
    : outcome === "LOSS" ? -(Math.random() * 200 + 20).toFixed(2) : 0;
  const market = MARKETS[Math.floor(Math.random() * MARKETS.length)];
  return {
    id, market, outcome, stake, pnl,
    action: ACTIONS[Math.floor(Math.random() * ACTIONS.length)],
    target: TARGETS[Math.floor(Math.random() * TARGETS.length)],
    edge: `${(Math.random() * 8 + 1).toFixed(1)}%`,
    confidence: Math.floor(Math.random() * 30 + 65),
    time: new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" }),
    category: ["BTC","ETH","SOL","XRP"].includes(market) ? "crypto"
      : ["SPY","AAPL"].includes(market) ? "stocks"
      : market === "FOREX" || market === "EUR/USD" ? "forex" : "sports"
  };
}

function genHistory() {
  let bal = 10000;
  return Array.from({ length: 30 }, (_, i) => {
    bal += (Math.random() - 0.38) * 350;
    return { d: `${i + 1}`, bal: +bal.toFixed(2) };
  });
}

const history = genHistory();
const initTrades = Array.from({ length: 15 }, (_, i) => genTrade(i));

const NAV = ["home", "trades", "stats", "fund"];
const NAV_ICONS = {
  home: "⬡", trades: "↕", stats: "◈", fund: "◎"
};

export default function JDBBot() {
  const [screen, setScreen] = useState("home");
  const [trades, setTrades] = useState(initTrades);
  const [balance, setBalance] = useState(11243.88);
  const [botOn, setBotOn] = useState(true);
  const [log, setLog] = useState([]);
  const [depositAmt, setDepositAmt] = useState("");
  const [newTrade, setNewTrade] = useState(null);
  const idRef = useRef(16);

  const wins = trades.filter(t => t.outcome === "WIN").length;
  const losses = trades.filter(t => t.outcome === "LOSS").length;
  const winRate = ((wins / Math.max(wins + losses, 1)) * 100).toFixed(1);
  const totalPnL = trades.reduce((s, t) => s + t.pnl, 0);
  const roi = ((totalPnL / 10000) * 100).toFixed(2);
  const pending = trades.filter(t => t.outcome === "PENDING").length;

  useEffect(() => {
    if (!botOn) return;
    const iv = setInterval(() => {
      const t = genTrade(idRef.current++);
      setNewTrade(t.id);
      setTrades(p => [t, ...p.slice(0, 29)]);
      setBalance(p => +(p + t.pnl * 0.8).toFixed(2));
      setLog(p => [{
        msg: `${t.market} · ${t.action} → ${t.target}`,
        sub: `Edge ${t.edge} · ${t.confidence}% conf`,
        type: t.outcome,
        time: t.time
      }, ...p.slice(0, 29)]);
      setTimeout(() => setNewTrade(null), 800);
    }, 3000);
    return () => clearInterval(iv);
  }, [botOn]);

  const catColor = c => c === "crypto" ? "#60a5fa" : c === "stocks" ? "#a78bfa" : c === "forex" ? "#34d399" : "#e6b450";

  return (
    <div style={{
      minHeight: "100vh", minHeight: "100dvh",
      background: "#08080d",
      color: "#e2dfd8",
      fontFamily: "'DM Mono', 'Fira Code', 'Courier New', monospace",
      display: "flex", flexDirection: "column",
      maxWidth: 480, margin: "0 auto",
      position: "relative", overflow: "hidden",
    }}>
      {/* Ambient glow */}
      <div style={{
        position: "fixed", top: -100, right: -80, width: 300, height: 300,
        borderRadius: "50%", background: "radial-gradient(circle, rgba(230,180,80,0.08) 0%, transparent 70%)",
        pointerEvents: "none", zIndex: 0,
      }} />
      <div style={{
        position: "fixed", bottom: 80, left: -80, width: 250, height: 250,
        borderRadius: "50%", background: "radial-gradient(circle, rgba(96,165,250,0.05) 0%, transparent 70%)",
        pointerEvents: "none", zIndex: 0,
      }} />

      {/* Status Bar Spacer */}
      <div style={{ height: "env(safe-area-inset-top, 12px)", background: "#08080d", position: "relative", zIndex: 10 }} />

      {/* Top Bar */}
      <header style={{
        display: "flex", alignItems: "center", justifyContent: "space-between",
        padding: "10px 20px 10px",
        position: "relative", zIndex: 10,
        borderBottom: "1px solid rgba(255,255,255,0.05)"
      }}>
        <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
          <div style={{
            width: 32, height: 32, borderRadius: 9,
            background: "linear-gradient(135deg, #e6b450, #b8721e)",
            display: "flex", alignItems: "center", justifyContent: "center",
            fontSize: 11, fontWeight: 800, color: "#08080d", letterSpacing: "-0.5px",
            boxShadow: "0 0 16px rgba(230,180,80,0.35)"
          }}>JDB</div>
          <div>
            <div style={{ fontSize: 13, fontWeight: 700, letterSpacing: "0.3px" }}>JDB BOT</div>
            <div style={{ fontSize: 9, color: "#555", letterSpacing: "1.5px" }}>MULTI-MARKET ALGO</div>
          </div>
        </div>
        <button onClick={() => setBotOn(p => !p)} style={{
          background: botOn ? "rgba(74,222,128,0.1)" : "rgba(248,113,113,0.1)",
          border: `1px solid ${botOn ? "rgba(74,222,128,0.25)" : "rgba(248,113,113,0.25)"}`,
          color: botOn ? "#4ade80" : "#f87171",
          padding: "6px 12px", borderRadius: 20, cursor: "pointer",
          fontSize: 10, letterSpacing: "1.5px", display: "flex", alignItems: "center", gap: 5,
          fontFamily: "inherit",
        }}>
          <span style={{
            width: 5, height: 5, borderRadius: "50%",
            background: botOn ? "#4ade80" : "#f87171",
            display: "inline-block",
            animation: botOn ? "blink 1.4s infinite" : "none"
          }} />
          {botOn ? "LIVE" : "OFF"}
        </button>
      </header>

      {/* Scrollable Content */}
      <main style={{ flex: 1, overflowY: "auto", position: "relative", zIndex: 5, paddingBottom: 90 }}>

        {/* HOME */}
        {screen === "home" && (
          <div>
            {/* Balance Hero */}
            <div style={{ padding: "24px 20px 16px", textAlign: "center" }}>
              <div style={{ fontSize: 10, color: "#555", letterSpacing: "2.5px", marginBottom: 6 }}>CURRENT BALANCE</div>
              <div style={{
                fontSize: 38, fontWeight: 800, letterSpacing: "-2px",
                color: "#e2dfd8",
                textShadow: "0 0 40px rgba(230,180,80,0.2)"
              }}>
                ${balance.toLocaleString("en-US", { minimumFractionDigits: 2 })}
              </div>
              <div style={{
                display: "inline-flex", alignItems: "center", gap: 5, marginTop: 6,
                fontSize: 12, color: totalPnL >= 0 ? "#4ade80" : "#f87171",
                background: totalPnL >= 0 ? "rgba(74,222,128,0.08)" : "rgba(248,113,113,0.08)",
                padding: "4px 12px", borderRadius: 20,
                border: `1px solid ${totalPnL >= 0 ? "rgba(74,222,128,0.2)" : "rgba(248,113,113,0.2)"}`
              }}>
                {totalPnL >= 0 ? "▲" : "▼"} ${Math.abs(totalPnL).toFixed(2)} &nbsp;·&nbsp; {roi}% ROI
              </div>
            </div>

            {/* Mini Stats */}
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 8, padding: "0 16px 16px" }}>
              {[
                { l: "WIN RATE", v: `${winRate}%`, c: "#e6b450" },
                { l: "WINS", v: wins, c: "#4ade80" },
                { l: "LOSSES", v: losses, c: "#f87171" },
              ].map(s => (
                <div key={s.l} style={{
                  background: "rgba(255,255,255,0.03)", border: "1px solid rgba(255,255,255,0.06)",
                  borderRadius: 12, padding: "12px 10px", textAlign: "center"
                }}>
                  <div style={{ fontSize: 18, fontWeight: 700, color: s.c }}>{s.v}</div>
                  <div style={{ fontSize: 8, color: "#444", letterSpacing: "1.5px", marginTop: 2 }}>{s.l}</div>
                </div>
              ))}
            </div>

            {/* P&L Chart */}
            <div style={{
              margin: "0 16px 16px",
              background: "rgba(255,255,255,0.03)", border: "1px solid rgba(255,255,255,0.06)",
              borderRadius: 14, padding: "16px 8px 8px"
            }}>
              <div style={{ paddingLeft: 10, marginBottom: 10 }}>
                <div style={{ fontSize: 9, color: "#555", letterSpacing: "2px" }}>30-DAY CURVE</div>
                <div style={{ fontSize: 12, color: "#e2dfd8", marginTop: 2 }}>Portfolio Performance</div>
              </div>
              <ResponsiveContainer width="100%" height={130}>
                <AreaChart data={history} margin={{ top: 0, right: 4, left: -28, bottom: 0 }}>
                  <defs>
                    <linearGradient id="g" x1="0" y1="0" x2="0" y2="1">
                      <stop offset="5%" stopColor="#e6b450" stopOpacity={0.25} />
                      <stop offset="95%" stopColor="#e6b450" stopOpacity={0} />
                    </linearGradient>
                  </defs>
                  <XAxis dataKey="d" stroke="#222" tick={{ fontSize: 8, fill: "#444" }} interval={4} />
                  <YAxis stroke="#222" tick={{ fontSize: 8, fill: "#444" }} />
                  <Tooltip
                    contentStyle={{ background: "#111", border: "1px solid #222", borderRadius: 8, fontSize: 10 }}
                    labelStyle={{ color: "#666" }} itemStyle={{ color: "#e6b450" }}
                    formatter={v => [`$${v.toLocaleString()}`, "Balance"]}
                  />
                  <Area type="monotone" dataKey="bal" stroke="#e6b450" strokeWidth={2} fill="url(#g)" dot={false} />
                </AreaChart>
              </ResponsiveContainer>
            </div>

            {/* Market Allocation */}
            <div style={{
              margin: "0 16px 16px",
              background: "rgba(255,255,255,0.03)", border: "1px solid rgba(255,255,255,0.06)",
              borderRadius: 14, padding: "16px"
            }}>
              <div style={{ fontSize: 9, color: "#555", letterSpacing: "2px", marginBottom: 14 }}>MARKET ALLOCATION</div>
              {[
                { l: "Sports", pct: 34, c: "#e6b450" },
                { l: "Crypto", pct: 28, c: "#60a5fa" },
                { l: "Stocks", pct: 22, c: "#a78bfa" },
                { l: "Forex", pct: 16, c: "#34d399" },
              ].map(m => (
                <div key={m.l} style={{ marginBottom: 10 }}>
                  <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 5 }}>
                    <span style={{ fontSize: 11, color: "#888" }}>{m.l}</span>
                    <span style={{ fontSize: 11, color: m.c, fontWeight: 600 }}>{m.pct}%</span>
                  </div>
                  <div style={{ height: 3, background: "rgba(255,255,255,0.06)", borderRadius: 3 }}>
                    <div style={{ height: "100%", width: `${m.pct}%`, background: m.c, borderRadius: 3, opacity: 0.85 }} />
                  </div>
                </div>
              ))}
            </div>

            {/* Recent Activity - top 3 */}
            <div style={{ margin: "0 16px", background: "rgba(255,255,255,0.03)", border: "1px solid rgba(255,255,255,0.06)", borderRadius: 14, overflow: "hidden" }}>
              <div style={{ padding: "14px 16px", borderBottom: "1px solid rgba(255,255,255,0.05)", display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                <div style={{ fontSize: 9, color: "#555", letterSpacing: "2px" }}>RECENT ACTIVITY</div>
                <button onClick={() => setScreen("trades")} style={{ background: "none", border: "none", color: "#e6b450", fontSize: 10, cursor: "pointer", fontFamily: "inherit", letterSpacing: "1px" }}>SEE ALL →</button>
              </div>
              {trades.slice(0, 4).map((t, i) => (
                <div key={t.id} style={{
                  display: "flex", alignItems: "center", justifyContent: "space-between",
                  padding: "12px 16px",
                  borderBottom: i < 3 ? "1px solid rgba(255,255,255,0.03)" : "none",
                  background: t.id === newTrade ? "rgba(230,180,80,0.04)" : "transparent",
                  transition: "background 0.4s"
                }}>
                  <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
                    <div style={{
                      width: 32, height: 32, borderRadius: 8, flexShrink: 0,
                      background: `rgba(${catColor(t.category).slice(1).match(/../g).map(h => parseInt(h, 16)).join(",")}, 0.1)`,
                      border: `1px solid rgba(${catColor(t.category).slice(1).match(/../g).map(h => parseInt(h, 16)).join(",")}, 0.2)`,
                      display: "flex", alignItems: "center", justifyContent: "center",
                      fontSize: 9, color: catColor(t.category), fontWeight: 700, letterSpacing: "0.3px"
                    }}>{t.market}</div>
                    <div>
                      <div style={{ fontSize: 12, color: "#ccc" }}>{t.target}</div>
                      <div style={{ fontSize: 9, color: "#444", marginTop: 1 }}>{t.time} · {t.edge} edge</div>
                    </div>
                  </div>
                  <div style={{ textAlign: "right" }}>
                    <div style={{ fontSize: 13, fontWeight: 700, color: t.pnl > 0 ? "#4ade80" : t.pnl < 0 ? "#f87171" : "#60a5fa" }}>
                      {t.pnl === 0 ? "—" : `${t.pnl > 0 ? "+" : ""}$${t.pnl.toFixed(2)}`}
                    </div>
                    <div style={{
                      fontSize: 8, letterSpacing: "1px", marginTop: 2,
                      color: t.outcome === "WIN" ? "#4ade80" : t.outcome === "LOSS" ? "#f87171" : "#60a5fa"
                    }}>{t.outcome}</div>
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* TRADES */}
        {screen === "trades" && (
          <div style={{ padding: "16px" }}>
            <div style={{ fontSize: 9, color: "#555", letterSpacing: "2px", marginBottom: 4 }}>ALL TRADES</div>
            <div style={{ fontSize: 18, fontWeight: 700, marginBottom: 16 }}>{trades.length} Positions</div>
            <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
              {trades.map((t, i) => (
                <div key={t.id} style={{
                  background: "rgba(255,255,255,0.03)", border: "1px solid rgba(255,255,255,0.06)",
                  borderRadius: 12, padding: "14px 14px",
                  display: "flex", alignItems: "center", gap: 12,
                  borderLeft: `3px solid ${t.outcome === "WIN" ? "#4ade80" : t.outcome === "LOSS" ? "#f87171" : "#60a5fa"}`,
                  animation: i === 0 && t.id === newTrade ? "slideIn 0.3s ease" : "none"
                }}>
                  <div style={{
                    width: 36, height: 36, borderRadius: 9, flexShrink: 0,
                    background: `rgba(${catColor(t.category).slice(1).match(/../g).map(h => parseInt(h, 16)).join(",")}, 0.12)`,
                    display: "flex", alignItems: "center", justifyContent: "center",
                    fontSize: 9, color: catColor(t.category), fontWeight: 800, letterSpacing: "0.2px"
                  }}>{t.market}</div>
                  <div style={{ flex: 1, minWidth: 0 }}>
                    <div style={{ fontSize: 12, color: "#ddd", whiteSpace: "nowrap", overflow: "hidden", textOverflow: "ellipsis" }}>{t.target}</div>
                    <div style={{ fontSize: 9, color: "#555", marginTop: 2 }}>{t.action} · {t.stake} · {t.edge} edge · {t.time}</div>
                  </div>
                  <div style={{ textAlign: "right", flexShrink: 0 }}>
                    <div style={{ fontSize: 14, fontWeight: 700, color: t.pnl > 0 ? "#4ade80" : t.pnl < 0 ? "#f87171" : "#60a5fa" }}>
                      {t.pnl === 0 ? "LIVE" : `${t.pnl > 0 ? "+" : ""}$${t.pnl.toFixed(2)}`}
                    </div>
                    <div style={{
                      fontSize: 8, letterSpacing: "1.2px", marginTop: 2,
                      color: t.outcome === "WIN" ? "#4ade80" : t.outcome === "LOSS" ? "#f87171" : "#60a5fa"
                    }}>{t.outcome}</div>
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* STATS */}
        {screen === "stats" && (
          <div style={{ padding: "16px" }}>
            <div style={{ fontSize: 9, color: "#555", letterSpacing: "2px", marginBottom: 4 }}>ANALYTICS</div>
            <div style={{ fontSize: 18, fontWeight: 700, marginBottom: 16 }}>Performance Stats</div>

            {/* Big stats */}
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10, marginBottom: 14 }}>
              {[
                { l: "Total P&L", v: `${totalPnL >= 0 ? "+" : ""}$${totalPnL.toFixed(2)}`, c: totalPnL >= 0 ? "#4ade80" : "#f87171" },
                { l: "ROI", v: `${roi}%`, c: roi >= 0 ? "#4ade80" : "#f87171" },
                { l: "Win Rate", v: `${winRate}%`, c: "#e6b450" },
                { l: "Active", v: pending, c: "#60a5fa" },
                { l: "Total Bets", v: trades.length, c: "#e2dfd8" },
                { l: "Best Win", v: `+$${Math.max(...trades.filter(t => t.pnl > 0).map(t => t.pnl)).toFixed(2)}`, c: "#4ade80" },
              ].map(s => (
                <div key={s.l} style={{
                  background: "rgba(255,255,255,0.03)", border: "1px solid rgba(255,255,255,0.06)",
                  borderRadius: 12, padding: "16px 14px"
                }}>
                  <div style={{ fontSize: 9, color: "#555", letterSpacing: "1.5px", marginBottom: 6 }}>{s.l}</div>
                  <div style={{ fontSize: 22, fontWeight: 700, color: s.c, letterSpacing: "-1px" }}>{s.v}</div>
                </div>
              ))}
            </div>

            {/* Bot Log */}
            <div style={{ background: "rgba(255,255,255,0.03)", border: "1px solid rgba(255,255,255,0.06)", borderRadius: 14, overflow: "hidden" }}>
              <div style={{ padding: "14px 16px", borderBottom: "1px solid rgba(255,255,255,0.05)", display: "flex", alignItems: "center", gap: 8 }}>
                <span style={{ width: 6, height: 6, borderRadius: "50%", background: botOn ? "#4ade80" : "#555", display: "inline-block", animation: botOn ? "blink 1.4s infinite" : "none" }} />
                <div style={{ fontSize: 9, color: "#555", letterSpacing: "2px" }}>BOT ACTIVITY LOG</div>
              </div>
              <div style={{ maxHeight: 300, overflowY: "auto" }}>
                {log.length === 0 && <div style={{ padding: 20, fontSize: 11, color: "#444", textAlign: "center" }}>Waiting for signals…</div>}
                {log.map((l, i) => (
                  <div key={i} style={{
                    padding: "10px 14px",
                    borderLeft: `2px solid ${l.type === "WIN" ? "#4ade80" : l.type === "LOSS" ? "#f87171" : "#60a5fa"}`,
                    marginLeft: 10, marginBottom: 2,
                    opacity: i === 0 ? 1 : Math.max(0.3, 1 - i * 0.04),
                    animation: i === 0 ? "slideIn 0.3s ease" : "none"
                  }}>
                    <div style={{ fontSize: 11, color: i === 0 ? "#ddd" : "#666" }}>{l.msg}</div>
                    <div style={{ fontSize: 9, color: "#444", marginTop: 2 }}>{l.time} · {l.sub}</div>
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}

        {/* FUND */}
        {screen === "fund" && (
          <div style={{ padding: "16px" }}>
            <div style={{ fontSize: 9, color: "#555", letterSpacing: "2px", marginBottom: 4 }}>DEPOSITS</div>
            <div style={{ fontSize: 18, fontWeight: 700, marginBottom: 6 }}>Fund Your Bot</div>
            <div style={{ fontSize: 11, color: "#555", marginBottom: 20, lineHeight: 1.5 }}>Deposit funds to be deployed across active strategies. Returns compound automatically.</div>

            {/* Balance card */}
            <div style={{
              background: "linear-gradient(135deg, rgba(230,180,80,0.1), rgba(200,134,42,0.06))",
              border: "1px solid rgba(230,180,80,0.2)",
              borderRadius: 16, padding: "20px", marginBottom: 16, textAlign: "center"
            }}>
              <div style={{ fontSize: 9, color: "#e6b450", letterSpacing: "2.5px", marginBottom: 8 }}>BOT BALANCE</div>
              <div style={{ fontSize: 32, fontWeight: 800, letterSpacing: "-1.5px", color: "#e2dfd8" }}>
                ${balance.toLocaleString("en-US", { minimumFractionDigits: 2 })}
              </div>
              <div style={{ fontSize: 11, color: totalPnL >= 0 ? "#4ade80" : "#f87171", marginTop: 6 }}>
                {totalPnL >= 0 ? "+" : ""}${totalPnL.toFixed(2)} earned since start
              </div>
            </div>

            {/* Amount input */}
            <div style={{ background: "rgba(255,255,255,0.03)", border: "1px solid rgba(255,255,255,0.08)", borderRadius: 14, padding: "16px", marginBottom: 12 }}>
              <div style={{ fontSize: 9, color: "#555", letterSpacing: "2px", marginBottom: 12 }}>DEPOSIT AMOUNT</div>
              <div style={{
                display: "flex", alignItems: "center", gap: 0,
                background: "rgba(0,0,0,0.4)", border: "1px solid rgba(255,255,255,0.1)", borderRadius: 10, overflow: "hidden"
              }}>
                <span style={{ padding: "12px 14px", color: "#555", fontSize: 14 }}>$</span>
                <input type="number" placeholder="0.00" value={depositAmt}
                  onChange={e => setDepositAmt(e.target.value)}
                  style={{
                    flex: 1, background: "none", border: "none", color: "#e2dfd8",
                    padding: "12px 0", fontSize: 18, fontFamily: "inherit", outline: "none",
                    letterSpacing: "-0.5px"
                  }} />
              </div>
              <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 8, marginTop: 10 }}>
                {["100", "500", "1000", "5000"].map(a => (
                  <button key={a} onClick={() => setDepositAmt(a)} style={{
                    background: depositAmt === a ? "rgba(230,180,80,0.15)" : "rgba(255,255,255,0.04)",
                    border: `1px solid ${depositAmt === a ? "rgba(230,180,80,0.4)" : "rgba(255,255,255,0.07)"}`,
                    color: depositAmt === a ? "#e6b450" : "#666",
                    padding: "8px 0", borderRadius: 8, cursor: "pointer",
                    fontSize: 11, fontFamily: "inherit",
                  }}>${+a >= 1000 ? `${+a / 1000}K` : a}</button>
                ))}
              </div>
            </div>

            {/* Payment methods */}
            <div style={{ background: "rgba(255,255,255,0.03)", border: "1px solid rgba(255,255,255,0.08)", borderRadius: 14, padding: "16px", marginBottom: 14 }}>
              <div style={{ fontSize: 9, color: "#555", letterSpacing: "2px", marginBottom: 12 }}>PAYMENT METHOD</div>
              {[
                { l: "Credit / Debit Card", sub: "Visa, Mastercard, Amex", icon: "▣" },
                { l: "Crypto Wallet", sub: "BTC, ETH, USDC, SOL", icon: "◎" },
                { l: "Bank Transfer", sub: "ACH / Wire", icon: "⊟" },
              ].map((m, i) => (
                <div key={m.l} style={{
                  display: "flex", alignItems: "center", gap: 12,
                  padding: "12px 0",
                  borderBottom: i < 2 ? "1px solid rgba(255,255,255,0.04)" : "none",
                  cursor: "pointer"
                }}>
                  <div style={{
                    width: 36, height: 36, borderRadius: 9, flexShrink: 0,
                    background: "rgba(230,180,80,0.08)", border: "1px solid rgba(230,180,80,0.15)",
                    display: "flex", alignItems: "center", justifyContent: "center",
                    fontSize: 14, color: "#e6b450"
                  }}>{m.icon}</div>
                  <div>
                    <div style={{ fontSize: 12, color: "#ccc" }}>{m.l}</div>
                    <div style={{ fontSize: 10, color: "#444", marginTop: 1 }}>{m.sub}</div>
                  </div>
                  <div style={{ marginLeft: "auto", color: "#444", fontSize: 12 }}>›</div>
                </div>
              ))}
            </div>

            <button style={{
              width: "100%", padding: "16px",
              background: depositAmt ? "linear-gradient(135deg, #e6b450, #b8721e)" : "rgba(255,255,255,0.05)",
              border: "none", borderRadius: 14, cursor: depositAmt ? "pointer" : "default",
              color: depositAmt ? "#08080d" : "#444",
              fontSize: 13, fontWeight: 800, letterSpacing: "1.5px",
              fontFamily: "inherit",
              boxShadow: depositAmt ? "0 0 24px rgba(230,180,80,0.25)" : "none",
              transition: "all 0.25s"
            }}>
              DEPOSIT {depositAmt ? `$${(+depositAmt).toLocaleString()}` : ""}
            </button>
          </div>
        )}

      </main>

      {/* Bottom Nav */}
      <nav style={{
        position: "fixed", bottom: 0, left: "50%", transform: "translateX(-50%)",
        width: "100%", maxWidth: 480,
        background: "rgba(8,8,13,0.95)",
        backdropFilter: "blur(20px)",
        borderTop: "1px solid rgba(255,255,255,0.07)",
        display: "grid", gridTemplateColumns: "repeat(4, 1fr)",
        paddingBottom: "env(safe-area-inset-bottom, 8px)",
        zIndex: 100,
      }}>
        {NAV.map(n => (
          <button key={n} onClick={() => setScreen(n)} style={{
            background: "none", border: "none", cursor: "pointer",
            padding: "12px 0 8px",
            display: "flex", flexDirection: "column", alignItems: "center", gap: 4,
            color: screen === n ? "#e6b450" : "#444",
            transition: "color 0.2s",
            fontFamily: "inherit",
          }}>
            <span style={{ fontSize: 18, lineHeight: 1 }}>{NAV_ICONS[n]}</span>
            <span style={{
              fontSize: 8, letterSpacing: "1.5px", textTransform: "uppercase",
              fontWeight: screen === n ? 700 : 400
            }}>{n}</span>
            {screen === n && <span style={{ width: 4, height: 4, borderRadius: "50%", background: "#e6b450", marginTop: -2 }} />}
          </button>
        ))}
      </nav>

      <style>{`
        @keyframes blink { 0%,100%{opacity:1;} 50%{opacity:0.3;} }
        @keyframes slideIn { from{opacity:0;transform:translateY(-6px);} to{opacity:1;transform:translateY(0);} }
        * { -webkit-tap-highlight-color: transparent; box-sizing: border-box; }
        input[type=number]::-webkit-inner-spin-button { -webkit-appearance: none; }
        ::-webkit-scrollbar { display: none; }
      `}</style>
    </div>
  );
}
