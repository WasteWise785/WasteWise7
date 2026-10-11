# WasteWise7
Create a user friendly app of wastewise
import os, zipfile, json, textwrap, pathlib, shutil
from PIL import Image, ImageDraw
base = "/mnt/data/WasteWise"
if os.path.exists(base): shutil.rmtree(base)
os.makedirs(base, exist_ok=True)
files = {}

files["index.html"] = r'''<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
  <meta name="theme-color" content="#176b45">
  <meta name="description" content="Give everyday objects a second life with creative reuse ideas and eco challenges.">
  <title>WasteWise — Give Every Object a Second Life</title>
  <link rel="manifest" href="manifest.webmanifest">
  <link rel="icon" href="icons/icon-192.png">
  <link rel="apple-touch-icon" href="icons/icon-192.png">
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <header class="topbar">
    <a class="brand" href="#home" aria-label="WasteWise home"><span class="brand-icon">🌱</span><span>WasteWise<small>Give Every Object a Second Life</small></span></a>
    <button id="installBtn" class="btn btn-light install-btn hidden" type="button">＋ Install app</button>
  </header>
  <main>
    <section id="home" class="hero section">
      <div class="hero-copy">
        <span class="eyebrow">SMALL CHOICES. BIGGER POSSIBILITIES.</span>
        <h1>Don't Waste It.<br><em>Reimagine It!</em></h1>
        <p>Turn everyday discards into useful, creative things. Pick an object, find an idea, and give it a second life.</p>
        <div class="hero-actions">
          <button class="btn btn-primary" data-go="explorer">📷 Scan waste</button>
          <button class="btn btn-outline" data-go="ideas">✦ Explore ideas</button>
        </div>
        <div class="quick-stats"><div><strong id="homePoints">0</strong><span>eco-points</span></div><div><strong id="homeDone">0</strong><span>challenges done</span></div><div><strong id="homeBadges">0</strong><span>badges earned</span></div></div>
      </div>
      <div class="hero-art" aria-hidden="true"><div class="sun"></div><div class="plant-pot">🪴</div><div class="floating f1">♻️</div><div class="floating f2">🌼</div><div class="floating f3">📦</div><div class="art-caption">Your next idea is<br>already around you.</div></div>
    </section>

    <section class="section" aria-labelledby="start-title">
      <div class="section-heading"><div><span class="eyebrow">START HERE</span><h2 id="start-title">Make a little difference</h2></div><p>Choose your next step</p></div>
      <div class="action-grid">
        <button class="action-card" data-go="explorer"><span class="action-icon mint">📸</span><strong>Scan waste</strong><span>Choose a photo and category</span><b>Start exploring →</b></button>
        <button class="action-card" data-go="ideas"><span class="action-icon peach">🧩</span><strong>Explore ideas</strong><span>Find a creative second life</span><b>Browse projects →</b></button>
        <button class="action-card" data-go="challenges"><span class="action-icon yellow">🏆</span><strong>Eco challenges</strong><span>Try a small green habit</span><b>Take a challenge →</b></button>
        <button class="action-card" data-go="impact"><span class="action-icon blue">🌍</span><strong>My eco impact</strong><span>See your progress and badges</span><b>View progress →</b></button>
      </div>
    </section>

    <section id="explorer" class="section panel-section">
      <div class="section-heading"><div><span class="eyebrow">WASTE EXPLORER</span><h2>What have you got?</h2></div><span class="section-number">01 / EXPLORE</span></div>
      <p class="muted">Add a photo if you like, then choose the category yourself. WasteWise does not automatically identify objects.</p>
      <div class="explorer-layout">
        <div class="upload-card">
          <label class="upload-zone" for="photoInput"><span class="upload-symbol">＋</span><strong>Choose or take a photo</strong><span>Photos stay on this device; no upload is made.</span><span class="btn btn-outline">Choose photo</span></label>
          <input id="photoInput" type="file" accept="image/*" capture="environment" class="visually-hidden">
          <div id="previewWrap" class="preview-wrap hidden"><img id="photoPreview" alt="Preview of selected object"><button id="removePhoto" class="text-button" type="button">Remove photo</button></div>
          <p id="photoStatus" class="small muted" aria-live="polite">No photo selected.</p>
        </div>
        <div class="category-card"><h3>Choose a waste category</h3><div id="categoryChoices" class="category-choices"></div><p id="selectedCategory" class="small muted">Select a category to see matching ideas.</p><button id="showCategoryIdeas" class="btn btn-primary" type="button">Find reuse ideas →</button></div>
      </div>
      <div class="safety-note"><strong>⚠ Safety first</strong><span>Never handle sharp or broken items, unknown chemicals, swollen/leaking batteries, or damaged electronics. Ask a trusted adult and use an appropriate collection point.</span></div>
    </section>

    <section id="ideas" class="section">
      <div class="section-heading"><div><span class="eyebrow">THE IDEA LIBRARY</span><h2>Give it a second life</h2></div><span class="section-number">02 / CREATE</span></div>
      <div class="filters"><label class="search-box"><span>⌕</span><input id="ideaSearch" type="search" placeholder="Search projects or materials…" aria-label="Search reuse ideas"></label><label class="filter-label">Category <select id="ideaCategory"><option value="all">All categories</option></select></label></div>
      <div id="ideaCount" class="small muted"></div>
      <div id="ideasGrid" class="ideas-grid"></div>
    </section>

    <section id="challenges" class="section panel-section">
      <div class="section-heading"><div><span class="eyebrow">TINY ACTIONS, REAL HABITS</span><h2>Eco challenges</h2></div><span class="section-number">03 / TAKE ACTION</span></div>
      <p class="muted">Complete a challenge to earn 10 points. Each challenge can award points only once on this device.</p>
      <div class="challenge-summary"><div><span>Today's progress</span><strong id="challengeProgress">0 / 5</strong></div><div class="progress-track"><span id="challengeProgressBar"></span></div><div class="small muted" id="challengeMessage">Choose one small action to begin.</div></div>
      <div id="challengesGrid" class="challenges-grid"></div>
      <div class="badge-strip"><span>🏅</span><div><strong>Badges unlock as you go</strong><p class="small muted">Earn your first badge after one challenge, then keep building your collection.</p></div><button class="btn btn-outline" data-go="impact">View badges</button></div>
    </section>

    <section id="impact" class="section">
      <div class="section-heading"><div><span class="eyebrow">YOUR GREEN JOURNEY</span><h2>My eco impact</h2></div><span class="section-number">04 / YOUR PROGRESS</span></div>
      <div class="impact-grid"><div class="impact-card impact-main"><span>🌱</span><small>TOTAL ECO-POINTS</small><strong id="impactPoints">0</strong><p>Every completed challenge is a step toward a thoughtful habit.</p></div><div class="impact-card"><span>✅</span><small>CHALLENGES COMPLETED</small><strong id="impactDone">0</strong></div><div class="impact-card"><span>♻️</span><small>ITEMS REUSED</small><strong id="impactItems">0</strong><p class="small muted">Counted when you log a reuse project.</p></div></div>
      <div class="subsection-heading"><h3>Your badges</h3><span class="small muted" id="badgeCount">0 earned</span></div><div id="badgesGrid" class="badges-grid"></div>
      <div class="log-card"><div><h3>Did you reuse something?</h3><p class="small muted">Log a completed project to count one reused item. This is a personal tally, not a measured environmental impact.</p></div><button id="logReuse" class="btn btn-primary">＋ Log reused item</button></div>
      <div class="privacy-mini"><strong>🔒 Your data stays on this device</strong><p>Progress is stored in this browser's local storage. It does not automatically sync to other devices. Clearing browser data may erase it.</p><button id="resetBtn" class="text-button danger" type="button">Reset all my progress</button></div>
    </section>

    <section id="privacy" class="section privacy-section">
      <div class="section-heading"><div><span class="eyebrow">YOUR DATA, YOUR CHOICE</span><h2>Privacy & safety</h2></div></div>
      <div class="privacy-grid"><article><h3>🔐 Privacy</h3><p>WasteWise stores points, completed challenge IDs, earned badges (calculated from progress), and your reused-item count in localStorage on this device. Photos are previewed locally and are not sent to a server. There are no accounts, trackers, or external AI calls in this version.</p><p>Anyone using the same browser profile on this device may be able to access the saved progress. Browser storage can be removed by clearing site data. Progress does not automatically sync between devices.</p></article><article><h3>🧤 Responsible reuse</h3><p>Ask an adult before using tools, heat, adhesives, or containers. Never reuse chemical, medicine, or pesticide containers for crafts or food. Do not cut glass or metal, open electronics, or handle batteries. Keep projects away from food preparation surfaces.</p><p>For e-waste, batteries, chemicals, and broken glass, follow your local municipal rules or use an authorised collection point. Local rules differ, so this app does not claim a specific collection service is available everywhere.</p></article></div>
    </section>
  </main>
  <footer><a class="brand footer-brand" href="#home">🌱 WasteWise</a><span>Made for curious minds. Reuse thoughtfully.</span><a href="#privacy">Privacy & safety</a></footer>
  <div id="toast" class="toast" role="status" aria-live="polite"></div>
  <script src="app.js" defer></script>
</body>
</html>'''

files["styles.css"] = r''':root{--green:#176b45;--deep:#104b34;--lime:#dff4df;--mint:#eff9f0;--ink:#19352a;--muted:#65786d;--line:#dce9df;--white:#fff;--cream:#fbfdf9;--orange:#f5a65b;--shadow:0 12px 35px rgba(28,79,52,.08)}
*{box-sizing:border-box}html{scroll-behavior:smooth;scroll-padding-top:82px}body{margin:0;background:var(--cream);color:var(--ink);font-family:system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif;line-height:1.55}button,input,select{font:inherit}button,a,select{touch-action:manipulation}button{cursor:pointer}a{color:var(--green)}.topbar{height:72px;display:flex;align-items:center;justify-content:space-between;padding:0 max(20px,calc((100vw - 1160px)/2));background:rgba(255,255,255,.94);border-bottom:1px solid var(--line);position:sticky;top:0;z-index:10;backdrop-filter:blur(12px)}.brand{display:flex;align-items:center;gap:10px;text-decoration:none;color:var(--deep);font-weight:850;font-size:1.12rem;line-height:1.15}.brand-icon{font-size:1.8rem}.brand small{display:block;font-size:.68rem;color:var(--muted);font-weight:500;margin-top:4px}.btn{border:0;border-radius:14px;padding:13px 18px;font-weight:750;min-height:46px;display:inline-flex;align-items:center;justify-content:center;gap:8px;transition:transform .15s,box-shadow .15s}.btn:active,.action-card:active{transform:scale(.98)}.btn-primary{background:var(--green);color:#fff;box-shadow:0 5px 13px #176b4520}.btn-primary:hover{background:var(--deep)}.btn-outline{border:1px solid #9abfa6;color:var(--green);background:#fff}.btn-light{background:var(--lime);color:var(--deep)}.hidden{display:none!important}.section{max-width:1160px;margin:auto;padding:64px 24px}.hero{display:grid;grid-template-columns:1.1fr .9fr;gap:32px;align-items:center;padding-top:60px;padding-bottom:66px}.eyebrow{font-size:.72rem;font-weight:850;letter-spacing:.13em;color:var(--green)}h1,h2,h3,p{margin-top:0}h1{font-size:clamp(2.4rem,5.4vw,4.35rem);letter-spacing:-.055em;line-height:1.04;margin:15px 0 20px}h1 em{font-style:normal;color:var(--green)}h2{font-size:clamp(1.7rem,3vw,2.35rem);letter-spacing:-.04em;line-height:1.15;margin:8px 0 0}h3{font-size:1.08rem;margin-bottom:10px}.hero-copy>p{max-width:530px;color:var(--muted);font-size:1.08rem}.hero-actions{display:flex;flex-wrap:wrap;gap:12px;margin:26px 0 28px}.quick-stats{display:flex;gap:28px;flex-wrap:wrap}.quick-stats div{display:grid;gap:1px}.quick-stats strong{font-size:1.4rem;color:var(--deep)}.quick-stats span,.small{font-size:.82rem;color:var(--muted)}.hero-art{min-height:350px;border-radius:34px;background:linear-gradient(145deg,#e9f8e8,#cdebd1);position:relative;display:flex;align-items:center;justify-content:center;overflow:hidden}.sun{position:absolute;width:190px;height:190px;border-radius:50%;background:#ffdb8b;right:12%;top:9%;opacity:.9}.plant-pot{position:relative;z-index:2;font-size:clamp(8rem,17vw,13rem);filter:drop-shadow(0 14px 10px #184b2920);transform:translateY(12px)}.floating{position:absolute;display:grid;place-items:center;border-radius:20px;background:#ffffffd9;box-shadow:var(--shadow);font-size:2rem;width:68px;height:68px;animation:float 4s ease-in-out infinite}.f1{left:13%;top:19%}.f2{right:13%;bottom:23%;animation-delay:.8s}.f3{left:22%;bottom:13%;animation-delay:1.4s}.art-caption{position:absolute;right:25px;top:28px;font-size:.78rem;color:var(--deep);font-weight:700;text-align:right}@keyframes float{0%,100%{transform:translateY(0)}50%{transform:translateY(-8px)}}.section-heading{display:flex;align-items:end;justify-content:space-between;gap:18px;margin-bottom:24px}.section-heading>p{color:var(--muted);margin:0}.action-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:16px}.action-card{border:1px solid var(--line);background:#fff;border-radius:20px;padding:22px;text-align:left;display:flex;flex-direction:column;align-items:flex-start;min-height:210px;box-shadow:0 4px 16px #184b2905;transition:.2s}.action-card:hover{box-shadow:var(--shadow);transform:translateY(-3px)}.action-icon{height:48px;width:48px;border-radius:15px;display:grid;place-items:center;font-size:1.5rem;margin-bottom:18px}.mint{background:#e2f6e6}.peach{background:#fff0e2}.yellow{background:#fff5ce}.blue{background:#e4f1ff}.action-card strong{font-size:1.08rem}.action-card>span:not(.action-icon){font-size:.84rem;color:var(--muted);margin:5px 0 18px}.action-card b{color:var(--green);font-size:.84rem;margin-top:auto}.panel-section{max-width:none;background:#f0f8f0}.panel-section>*{max-width:1112px;margin-left:auto;margin-right:auto}.section-number{font-size:.73rem;letter-spacing:.1em;color:#789182;font-weight:800}.muted{color:var(--muted)}.explorer-layout{display:grid;grid-template-columns:1fr 1fr;gap:18px;margin-top:24px}.upload-card,.category-card{border:1px solid var(--line);background:#fff;border-radius:22px;padding:22px}.upload-zone{min-height:230px;border:2px dashed #b8d5bf;border-radius:17px;background:#fbfefb;display:flex;align-items:center;justify-content:center;flex-direction:column;gap:9px;text-align:center;padding:18px;cursor:pointer}.upload-zone>span:not(.upload-symbol){font-size:.83rem;color:var(--muted)}.upload-symbol{font-size:2rem;display:grid;place-items:center;width:48px;height:48px;border-radius:50%;background:var(--lime);color:var(--green)}.visually-hidden{position:absolute;width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0,0,0,0);white-space:nowrap;border:0}.preview-wrap{margin-top:14px;text-align:center}.preview-wrap img{max-height:220px;max-width:100%;border-radius:14px;object-fit:contain}.text-button{border:0;background:transparent;color:var(--green);text-decoration:underline;padding:10px 6px;font-weight:700}.danger{color:#a32e2e}.category-choices{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:9px;margin:16px 0}.category-choice{border:1px solid var(--line);background:#fff;border-radius:12px;padding:11px 10px;text-align:left;color:var(--ink);font-size:.86rem;min-height:48px}.category-choice[aria-pressed=true]{border-color:var(--green);background:#eaf7ec;box-shadow:inset 0 0 0 1px var(--green);font-weight:750}.safety-note{margin-top:18px;background:#fff5df;border:1px solid #f3d9a7;border-radius:16px;padding:15px 18px;display:flex;gap:12px;align-items:flex-start;font-size:.86rem}.safety-note strong{white-space:nowrap;color:#7e5010}.safety-note span{color:#6c5737}.filters{display:flex;gap:12px;margin-bottom:15px}.search-box{display:flex;align-items:center;gap:10px;flex:1;border:1px solid var(--line);border-radius:14px;background:#fff;padding:0 14px;min-height:48px}.search-box span{font-size:1.5rem;color:var(--green)}.search-box input{width:100%;border:0;outline:0;background:transparent;min-width:0}.filter-label{display:flex;align-items:center;gap:8px;color:var(--muted);font-size:.85rem}.filter-label select{border:1px solid var(--line);border-radius:12px;padding:12px;background:#fff;color:var(--ink);max-width:190px}.ideas-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:16px;margin-top:12px}.idea-card{background:#fff;border:1px solid var(--line);border-radius:19px;padding:19px;display:flex;flex-direction:column;min-width:0}.idea-top{display:flex;align-items:center;justify-content:space-between;gap:10px;margin-bottom:10px}.idea-emoji{font-size:1.8rem}.pill{border-radius:100px;background:#edf7ed;color:var(--green);padding:5px 9px;font-size:.68rem;font-weight:800}.idea-card h3{margin-bottom:5px}.idea-meta{font-size:.77rem;color:var(--muted);margin-bottom:12px}.idea-card details{border-top:1px solid #edf1ed;padding-top:11px;margin-top:5px}.idea-card summary{font-weight:750;color:var(--green);cursor:pointer;padding:5px 0}.idea-card ul,.idea-card ol{padding-left:21px;font-size:.86rem}.idea-card li{margin:6px 0}.safety-text{background:#fff5df;border-radius:10px;padding:10px;font-size:.8rem;color:#6c5737;margin-top:auto}.challenge-summary{background:#fff;border:1px solid var(--line);border-radius:18px;padding:19px;display:grid;grid-template-columns:auto 1fr;gap:12px 20px;align-items:center;margin:20px 0}.challenge-summary>div:first-child{display:grid;gap:2px}.challenge-summary strong{font-size:1.35rem;color:var(--green)}.progress-track{height:10px;background:#e6eee7;border-radius:100px;overflow:hidden}.progress-track span{display:block;height:100%;width:0;background:var(--green);border-radius:100px;transition:width .3s}.challenge-summary .small{grid-column:1/-1}.challenges-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:15px}.challenge-card{background:#fff;border:1px solid var(--line);border-radius:18px;padding:19px;display:flex;flex-direction:column;min-height:210px}.challenge-card .challenge-emoji{font-size:1.7rem}.challenge-card h3{margin:9px 0 7px}.challenge-card p{font-size:.87rem;color:var(--muted)}.challenge-card .btn{margin-top:auto;width:100%}.challenge-card.done{background:#f1faf1;border-color:#acd3b4}.badge-strip{display:flex;align-items:center;gap:15px;margin-top:18px;background:#fff;border:1px solid var(--line);border-radius:17px;padding:16px}.badge-strip>span{font-size:2rem}.badge-strip p{margin:0}.badge-strip .btn{margin-left:auto;white-space:nowrap}.impact-grid{display:grid;grid-template-columns:1.3fr 1fr 1fr;gap:15px}.impact-card{background:#fff;border:1px solid var(--line);border-radius:20px;padding:22px;display:flex;flex-direction:column;align-items:flex-start}.impact-card>span{font-size:1.6rem;margin-bottom:14px}.impact-card small{font-size:.68rem;font-weight:850;letter-spacing:.07em;color:var(--muted)}.impact-card strong{font-size:2.3rem;line-height:1.2;margin:8px 0;color:var(--deep)}.impact-card p{font-size:.82rem;color:var(--muted);margin:7px 0 0}.impact-main{background:linear-gradient(135deg,#e2f6e4,#f5fbf3)}.subsection-heading{display:flex;align-items:center;justify-content:space-between;margin:32px 0 14px}.subsection-heading h3{margin:0}.badges-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:12px}.badge-card{border:1px solid var(--line);border-radius:16px;background:#fff;padding:16px;text-align:center}.badge-card .badge-emoji{font-size:2rem;filter:grayscale(1);opacity:.45}.badge-card.earned{background:#f0f9ef;border-color:#b6d9ba
