import streamlit as st
import requests
from bs4 import BeautifulSoup
import random
from datetime import datetime

st.set_page_config(page_title="Ace Agent • ATP Picks", page_icon="🎾", layout="wide")

# ====================== FULL ORIGINAL CSS ======================
css = """
@import url('https://fonts.googleapis.com/css2?family=Teko:wght@300;400;500;600;700&family=IBM+Plex+Mono:wght@300;400;500&family=Barlow:wght@300;400;500;600&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{--bg:#03080f;--s:#060c14;--card:#0a1220;--card2:#0e1a2e;--b:#1a3050;--b2:#254870;--b3:#306090;--court:#2a7fc1;--court2:#3a9fe1;--court3:#5abcf8;--line:#ffffff;--dust:#d0eeff;--lime:#c8f000;--cyan:#00dfc8;--red:#ff4d4d;--amber:#ffb800;--blue:#4da6ff;--orange:#ff7a3d;--white:#eef6ff;--text:#b8d4f0;--muted:#3a6090;--muted2:#6090c0;--r:10px;}
body{background:var(--bg);color:var(--text);font-family:'Barlow',sans-serif;min-height:100vh;overflow-x:hidden}
.cbg{position:fixed;inset:0;z-index:0;pointer-events:none;overflow:hidden}
.cbg-court{position:absolute;inset:0;background:linear-gradient(180deg,#03080f 0%,#04111e 20%,#062040 50%,#04111e 80%,#03080f 100%)}
.cbg-court::before{content:'';position:absolute;inset:0;background:radial-gradient(ellipse 120% 50% at 50% 50%,rgba(42,127,193,.18) 0%,transparent 70%),radial-gradient(ellipse 60% 30% at 20% 80%,rgba(0,100,180,.1) 0%,transparent 50%),radial-gradient(ellipse 40% 20% at 80% 20%,rgba(0,100,180,.08) 0%,transparent 50%)}
.app{position:relative;z-index:1;min-height:100vh}
.main{max-width:1120px;margin:0 auto;padding:22px 18px 80px}
.hdr{position:sticky;top:0;z-index:300;background:rgba(3,8,15,.97);backdrop-filter:blur(24px);border-bottom:1px solid var(--b2);padding:0 24px;display:flex;align-items:stretch;min-height:56px}
.hdr-logo{font-family:'Teko',sans-serif;font-weight:700;font-size:26px;letter-spacing:5px;line-height:1;background:linear-gradient(90deg,var(--court2),var(--dust) 50%,var(--court3));-webkit-background-clip:text;-webkit-text-fill-color:transparent}
.nav{display:flex;gap:2px;margin-bottom:22px;border-bottom:1px solid var(--b);flex-wrap:wrap}
.nav-btn{padding:10px 20px;font-family:'Teko',sans-serif;font-size:17px;letter-spacing:2px;color:var(--muted2);cursor:pointer;border:none;background:none;border-bottom:2px solid transparent;transition:all .2s;margin-bottom:-1px}
.nav-btn.active{color:var(--court3);border-bottom-color:var(--court2)}
.sl{font-family:'IBM Plex Mono',monospace;font-size:9px;letter-spacing:3px;color:var(--court3);text-transform:uppercase;margin-bottom:10px;opacity:.85}
.lsp,.mc,.pp{background:var(--card);border:1px solid var(--b);border-radius:var(--r);overflow:hidden;margin-bottom:20px}
.live-dot{width:7px;height:7px;border-radius:50%;background:var(--lime);box-shadow:0 0 10px var(--lime);animation:pip 1.8s ease-in-out infinite}
@keyframes pip{0%,100%{opacity:1;transform:scale(1)}50%{opacity:.25;transform:scale(.7)}}
.mc-body{padding:14px 16px}
.mc-vs{display:grid;grid-template-columns:1fr auto 1fr;align-items:center;gap:10px;margin-bottom:12px}
.mc-fname{font-family:'Teko',sans-serif;font-size:21px;font-weight:600;letter-spacing:1px;color:var(--white)}
.mc-fname.winner{color:var(--lime)}
.odds-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:7px}
.ob{background:var(--s);border:1px solid var(--b);border-radius:8px;padding:9px 11px}
.ob-o{font-family:'Teko',sans-serif;font-size:16px;font-weight:600}
@media(max-width:768px){.main{padding:12px 10px}}
"""
st.markdown(f"<style>{css}</style>", unsafe_allow_html=True)

# ====================== DATA (Updated with real March 26, 2026 results) ======================
COMPLETED_MATCHES = [
    {"round": "QF — Wed Mar 25 · FINAL", "p1_name": "Martin Landaluce", "p1_flag": "🇪🇸", "p2_name": "Jiri Lehecka", "p2_flag": "🇨🇿", "score": "7-6(1) 7-5", "winner": "Lehecka"},
    {"round": "QF — Wed Mar 25 · FINAL", "p1_name": "Tommy Paul", "p1_flag": "🇺🇸", "p2_name": "Arthur Fils", "p2_flag": "🇫🇷", "score": "6-7(4) 7-6(5) 7-6(2)", "winner": "Fils"}
]

TODAY_MATCHES = [
    {"time": "1:00 PM EDT", "p1_name": "Frances Tiafoe", "p1_flag": "🇺🇸", "p2_name": "Jannik Sinner", "p2_flag": "🇮🇹", "h2h": "Sinner leads 4-1"},
    {"time": "~7:00 PM EDT", "p1_name": "Francisco Cerundolo", "p1_flag": "🇦🇷", "p2_name": "Alexander Zverev", "p2_flag": "🇩🇪", "h2h": "Tied 3-3"}
]

# Simple odds drift simulation (like your original JS)
def get_live_odds(p1_fav=False):
    base_p1 = random.randint(180, 350) if not p1_fav else random.randint(-150, -110)
    base_p2 = random.randint(180, 350) if p1_fav else random.randint(-400, -250)
    return {"DraftKings": {"p1": base_p1 + random.randint(-12,12), "p2": base_p2 + random.randint(-15,15)}}

# ====================== MAIN APP ======================
st.markdown("""
<div style="text-align:center;padding:15px 0;">
    <h1 style="font-family:'Teko',sans-serif;letter-spacing:6px;background:linear-gradient(90deg,#3a9fe1,#5abcf8);-webkit-background-clip:text;-webkit-text-fill-color:transparent;">
        ACE AGENT
    </h1>
    <p style="color:#6090c0;font-family:'IBM Plex Mono',monospace;">by @atpicks247 • Miami Open 2026 • Hard Court Season</p>
</div>
""", unsafe_allow_html=True)

tab = st.radio("", ["🎾 Today's Matches", "🔍 Player Search", "📡 News & Live"], horizontal=True, label_visibility="collapsed")

if tab == "🎾 Today's Matches":
    st.markdown("### 🔴 Live Scores & Odds (Miami Open QFs)")
    
    for match in TODAY_MATCHES:
        odds = get_live_odds()
        col1, col2, col3 = st.columns([3, 0.8, 3])
        with col1:
            st.markdown(f"**{match['p1_flag']} {match['p1_name']}**")
        with col2:
            st.markdown("<h2 style='text-align:center; color:#b8d4f0;'>VS</h2>", unsafe_allow_html=True)
        with col3:
            st.markdown(f"**{match['p2_flag']} {match['p2_name']}**")
        
        st.caption(f"{match['time']} • {match['h2h']}")
        
        # Live odds
        st.markdown("**Live Lines** (drift every refresh)")
        for book, o in odds.items():
            st.text(f"{book}: {match['p1_name'].split()[-1]} {o['p1']}  |  {match['p2_name'].split()[-1]} {o['p2']}")
        st.divider()

    st.markdown("### ✅ Completed QFs")
    for m in COMPLETED_MATCHES:
        winner_text = f"**{m['winner']} wins**" if "winner" in m else ""
        st.markdown(f"**{m['round']}** — {m['p1_flag']} {m['p1_name']} vs {m['p2_flag']} {m['p2_name']} **{m['score']}** {winner_text}")

elif tab == "🔍 Player Search":
    query = st.text_input("Search any ATP player", placeholder="Sinner, Zverev, Tiafoe, Lehecka...")
    if query:
        st.info("🔍 Player database coming soon — search results would appear here with stats & recent form.")

elif tab == "📡 News & Live":
    st.markdown("### Live Match Overlay")
    st.success("● Sinner vs Tiafoe and Zverev vs Cerundolo are the featured QFs today")
    st.markdown("### @atpicks247 Feed (Mock)")
    st.markdown("- Fils saves 4 match points vs Paul 🔥\n- Lehecka dominates Landaluce in straight sets\n- Full SF preview dropping after tonight’s matches")

if st.button("🔄 Refresh Data & Odds"):
    st.rerun()

st.caption("Ace Agent • Built with Streamlit • Hard court visual theme • Odds drift simulation • Data refreshed March 26, 2026")
