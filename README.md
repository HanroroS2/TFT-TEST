<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>TFT Set 16 · 조합 탐색기</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@400;600;700&family=Barlow:wght@300;400;500;600&family=Barlow+Condensed:wght@400;600;700&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #060a14; --bg2: #0d1422; --bg3: #131d2e; --panel: #0f1a2a; --border: #1e3050;
    --gold: #c89b3c; --gold2: #f0c060; --teal: #0bc4b0;
    --orange: #f97316; --blue: #60a5fa; --red: #ef4444;
    --text: #d4c9b0; --muted: #7a8a9a; --white: #f0ede8;
  }
  *,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
  body{background:var(--bg);color:var(--text);font-family:'Barlow',sans-serif;min-height:100vh;overflow-x:hidden}
  body::before{content:'';position:fixed;inset:0;background-image:radial-gradient(ellipse at 20% 0%,rgba(200,155,60,.07) 0%,transparent 50%),radial-gradient(ellipse at 80% 100%,rgba(11,196,176,.05) 0%,transparent 50%),url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='56' height='100'%3E%3Cpolygon points='28,2 54,16 54,44 28,58 2,44 2,16' fill='none' stroke='%231e3050' stroke-width='0.5' opacity='0.35'/%3E%3Cpolygon points='28,52 54,66 54,94 28,108 2,94 2,66' fill='none' stroke='%231e3050' stroke-width='0.5' opacity='0.35'/%3E%3C/svg%3E");pointer-events:none;z-index:0}
  .container{position:relative;z-index:1;max-width:1120px;margin:0 auto;padding:0 20px 80px}

  /* Header */
  header{text-align:center;padding:44px 0 30px}
  header::after{content:'';display:block;width:180px;height:1px;background:linear-gradient(90deg,transparent,var(--gold),transparent);margin:18px auto 0}
  .logo-tag{font-family:'Barlow Condensed',sans-serif;font-size:11px;letter-spacing:4px;text-transform:uppercase;color:var(--teal);margin-bottom:10px}
  h1{font-family:'Cinzel',serif;font-size:clamp(24px,4vw,42px);font-weight:700;color:var(--white);letter-spacing:2px}
  h1 span{color:var(--gold)}
  .header-sub{margin-top:7px;font-size:13px;color:var(--muted);font-weight:300;letter-spacing:.5px}

  /* Search Panel */
  .search-panel{background:var(--panel);border:1px solid var(--border);border-radius:12px;padding:24px 26px;margin-bottom:26px;position:relative;overflow:hidden}
  .search-panel::before{content:'';position:absolute;top:0;left:0;right:0;height:2px;background:linear-gradient(90deg,transparent,var(--gold),transparent)}
  .search-label{font-family:'Barlow Condensed',sans-serif;font-size:11px;letter-spacing:3px;text-transform:uppercase;color:var(--gold);margin-bottom:8px;display:block}

  /* Selected Chips */
  .selected-chips{display:flex;flex-wrap:wrap;gap:6px;margin-bottom:9px;min-height:0}
  .selected-chip{display:inline-flex;align-items:center;gap:5px;padding:4px 9px;border-radius:6px;background:rgba(200,155,60,.13);border:1px solid var(--gold);color:var(--gold2);font-size:12px;font-weight:600;animation:popIn .16s ease}
  @keyframes popIn{from{transform:scale(.82);opacity:0}to{transform:scale(1);opacity:1}}
  .remove-btn{cursor:pointer;opacity:.55;font-size:14px;line-height:1;transition:opacity .15s,color .15s}
  .remove-btn:hover{opacity:1;color:var(--red)}

  /* Search row */
  .search-row{display:flex;gap:12px;align-items:flex-start;flex-wrap:wrap}
  .search-input-wrap{position:relative;flex:1;min-width:200px}
  .search-input{width:100%;background:var(--bg2);border:1px solid var(--border);border-radius:8px;padding:10px 15px;color:var(--white);font-family:'Barlow',sans-serif;font-size:14px;outline:none;transition:border-color .2s}
  .search-input:focus{border-color:var(--gold)}
  .search-input::placeholder{color:var(--muted)}

  .autocomplete-list{position:absolute;top:calc(100% + 4px);left:0;right:0;background:var(--bg3);border:1px solid var(--border);border-radius:8px;max-height:260px;overflow-y:auto;z-index:200;display:none;box-shadow:0 16px 40px rgba(0,0,0,.6)}
  .autocomplete-list.open{display:block}
  .ac-item{padding:8px 13px;cursor:pointer;display:flex;align-items:center;gap:9px;transition:background .1s;border-bottom:1px solid rgba(30,48,80,.4)}
  .ac-item:last-child{border-bottom:none}
  .ac-item:hover,.ac-item.active-ac{background:rgba(200,155,60,.1)}
  .ac-item.already-sel{opacity:.35;pointer-events:none}
  .ac-name{color:var(--white);font-weight:500;font-size:13px}
  .ac-traits{font-size:11px;color:var(--muted)}
  .ac-range{font-size:10px;padding:2px 6px;border-radius:10px;font-weight:700;letter-spacing:.5px;margin-left:auto;flex-shrink:0}
  .r-front{background:rgba(249,115,22,.15);color:var(--orange);border:1px solid rgba(249,115,22,.3)}
  .r-back{background:rgba(96,165,250,.15);color:var(--blue);border:1px solid rgba(96,165,250,.3)}

  .cost-badge{width:17px;height:17px;border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:8px;font-weight:700;flex-shrink:0}
  .cost-1{background:#3a3a3a;color:#bbb}.cost-2{background:#1a5c2a;color:#5edd82}.cost-3{background:#1a3e7a;color:#60a8f0}.cost-4{background:#4a1a7a;color:#b880f0}.cost-5{background:#7a4a00;color:#f0c060}.cost-7{background:#7a0a0a;color:#f06060}

  .btn-search{padding:10px 24px;background:linear-gradient(135deg,var(--gold),#a87820);border:none;border-radius:8px;color:var(--bg);font-family:'Barlow Condensed',sans-serif;font-size:13px;font-weight:700;letter-spacing:2px;text-transform:uppercase;cursor:pointer;transition:all .2s;white-space:nowrap;flex-shrink:0}
  .btn-search:hover{background:linear-gradient(135deg,var(--gold2),var(--gold));transform:translateY(-1px)}
  .btn-search:disabled{opacity:.35;cursor:not-allowed;transform:none}
  .btn-clear{padding:10px 14px;background:transparent;border:1px solid var(--border);border-radius:8px;color:var(--muted);font-family:'Barlow Condensed',sans-serif;font-size:12px;letter-spacing:1px;cursor:pointer;transition:all .2s;flex-shrink:0}
  .btn-clear:hover{border-color:var(--red);color:var(--red)}

  .options-row{display:flex;gap:16px;align-items:center;margin-top:12px;flex-wrap:wrap}
  .option-label{font-size:11px;color:var(--muted)}
  .toggle-wrap{display:flex;gap:5px}
  .toggle-btn{padding:3px 12px;border-radius:20px;border:1px solid var(--border);background:transparent;color:var(--muted);font-family:'Barlow Condensed',sans-serif;font-size:11px;font-weight:600;letter-spacing:1px;cursor:pointer;transition:all .2s}
  .toggle-btn.active{background:rgba(200,155,60,.15);border-color:var(--gold);color:var(--gold)}

  /* Results */
  .results-header{font-family:'Cinzel',serif;font-size:11px;letter-spacing:2px;color:var(--muted);text-transform:uppercase;margin-bottom:16px;padding-bottom:9px;border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between;flex-wrap:wrap;gap:8px}
  .results-header .rh-left span{color:var(--gold)}
  .results-legend{display:flex;gap:14px;font-family:'Barlow',sans-serif;font-size:11px;letter-spacing:0;text-transform:none}
  .legend-item{display:flex;align-items:center;gap:4px}
  .ldot{width:7px;height:7px;border-radius:50%}
  .ldot-f{background:var(--orange)}.ldot-b{background:var(--blue)}

  .level-section{margin-bottom:28px}
  .level-title{font-family:'Barlow Condensed',sans-serif;font-size:17px;font-weight:700;letter-spacing:2px;color:var(--teal);margin-bottom:11px;display:flex;align-items:center;gap:10px}
  .level-title::after{content:'';flex:1;height:1px;background:linear-gradient(90deg,var(--border),transparent)}

  .comp-cards{display:grid;grid-template-columns:repeat(auto-fill,minmax(320px,1fr));gap:13px}
  .comp-card{background:var(--panel);border:1px solid var(--border);border-radius:10px;overflow:hidden;position:relative;transition:border-color .2s,transform .2s;animation:fadeIn .35s ease forwards;opacity:0}
  .comp-card:hover{border-color:rgba(200,155,60,.4);transform:translateY(-2px)}
  @keyframes fadeIn{from{opacity:0;transform:translateY(7px)}to{opacity:1;transform:translateY(0)}}

  .rank-badge{position:absolute;top:9px;right:9px;width:22px;height:22px;border-radius:50%;display:flex;align-items:center;justify-content:center;font-family:'Cinzel',serif;font-size:10px;font-weight:700}
  .rk1{background:rgba(255,215,0,.12);border:1px solid gold;color:gold}
  .rk2{background:rgba(192,192,192,.1);border:1px solid silver;color:silver}
  .rk3{background:rgba(205,127,50,.1);border:1px solid #cd7f32;color:#cd7f32}

  /* Card header */
  .card-header{padding:11px 13px 9px;border-bottom:1px solid var(--border)}
  .header-top{display:flex;align-items:flex-start;justify-content:space-between;gap:10px;margin-bottom:9px}
  .score-row{display:flex;align-items:baseline;gap:3px}
  .score-num{font-family:'Cinzel',serif;font-size:21px;font-weight:700;color:var(--gold)}
  .score-sub{font-size:10px;color:var(--muted)}
  .stats-mini{text-align:right;font-size:11px;color:var(--muted);line-height:1.7}
  .stats-mini strong{color:var(--text)}

  /* Position bar */
  .pos-wrap{display:flex;align-items:center;gap:7px}
  .pos-bar{flex:1;height:5px;border-radius:3px;background:var(--bg2);overflow:hidden;display:flex}
  .bar-f{background:var(--orange);height:100%;transition:width .4s ease}
  .bar-b{background:var(--blue);height:100%;transition:width .4s ease}
  .pos-lbl{font-size:10px;font-weight:700;white-space:nowrap;letter-spacing:.3px}
  .plf{color:var(--orange)}.plb{color:var(--blue)}

  /* Champion chips */
  .sec-label{font-size:10px;letter-spacing:1.5px;text-transform:uppercase;color:var(--muted);padding:7px 13px 2px;font-family:'Barlow Condensed',sans-serif}
  .champ-list{padding:3px 13px 9px;display:flex;flex-wrap:wrap;gap:5px}
  .chip{padding:3px 8px;border-radius:4px;font-size:11px;font-weight:500;background:var(--bg2);border:1px solid var(--border);color:var(--text);display:flex;align-items:center;gap:4px;white-space:nowrap}
  .chip.anchor{background:rgba(200,155,60,.15);border-color:var(--gold);color:var(--gold2);font-weight:700}
  .chip.cfr{border-left:2px solid var(--orange)}
  .chip.cbk{border-left:2px solid var(--blue)}

  /* Synergies */
  .syn-list{padding:7px 13px 11px;display:flex;flex-wrap:wrap;gap:4px}
  .syn{display:inline-flex;align-items:center;gap:3px;padding:2px 8px;border-radius:20px;font-size:10px;font-family:'Barlow Condensed',sans-serif;font-weight:600;letter-spacing:.5px;border:1px solid}
  .st1{background:rgba(205,127,50,.12);border-color:rgba(205,127,50,.5);color:#cd7f32}
  .st2{background:rgba(160,160,160,.1);border-color:rgba(160,160,160,.4);color:#aaa}
  .st3{background:rgba(255,215,0,.12);border-color:rgba(255,215,0,.5);color:#ffd700}
  .st4{background:rgba(0,204,255,.12);border-color:rgba(0,204,255,.5);color:#00ccff}
  .sc{background:rgba(255,255,255,.1);padding:1px 4px;border-radius:10px;font-size:9px}

  /* States */
  .loading{text-align:center;padding:56px 0;color:var(--muted)}
  .spinner{width:34px;height:34px;border:2px solid var(--border);border-top-color:var(--gold);border-radius:50%;animation:spin .8s linear infinite;margin:0 auto 12px}
  @keyframes spin{to{transform:rotate(360deg)}}
  .empty-state{text-align:center;padding:72px 20px;color:var(--muted)}
  .empty-icon{font-size:42px;margin-bottom:12px;opacity:.35}
  .empty-state h3{font-family:'Cinzel',serif;font-size:16px;color:var(--text);margin-bottom:6px}
  .err{background:rgba(239,68,68,.1);border:1px solid rgba(239,68,68,.3);border-radius:8px;padding:11px 15px;color:#f87171;font-size:13px;margin-top:10px}

  ::-webkit-scrollbar{width:5px;height:5px}::-webkit-scrollbar-track{background:var(--bg2)}::-webkit-scrollbar-thumb{background:var(--border);border-radius:3px}::-webkit-scrollbar-thumb:hover{background:var(--gold)}
  @media(max-width:600px){.search-panel{padding:16px 14px}.comp-cards{grid-template-columns:1fr}}
</style>
</head>
<body>
<div class="container">
  <header>
    <div class="logo-tag">Teamfight Tactics · Set 16 · Lore &amp; Legends</div>
    <h1>SYNERGY <span>FINDER</span></h1>
    <div class="header-sub">기물을 1개 이상 선택하면 레벨별 최적 시너지 조합을 탐색합니다</div>
    <div style="margin-top:8px;font-size:12px;color:var(--gold);font-family:'Barlow',sans-serif;letter-spacing:1px;opacity:0.75">롤체의 왕 이종성 등장이오</div>
  </header>

  <div class="search-panel">
    <span class="search-label">🔍 기물 선택 (복수 선택 가능)</span>
    <div class="selected-chips" id="selectedChips"></div>
    <div class="search-row">
      <div class="search-input-wrap">
        <input type="text" class="search-input" id="champSearch" placeholder="기물 이름 입력 (예: 아리, 야스오, Ahri...)" autocomplete="off">
        <div class="autocomplete-list" id="acList"></div>
      </div>
      <button class="btn-search" id="searchBtn" disabled>조합 탐색</button>
      <button class="btn-clear" id="clearBtn" style="display:none">초기화</button>
    </div>
    <div class="options-row">
      <span class="option-label">레벨 필터:</span>
      <div class="toggle-wrap" id="levelFilters">
        <button class="toggle-btn active" data-level="all">전체</button>
        <button class="toggle-btn" data-level="6">6</button>
        <button class="toggle-btn" data-level="7">7</button>
        <button class="toggle-btn" data-level="8">8</button>
        <button class="toggle-btn" data-level="9">9</button>
      </div>
      <span class="option-label" style="margin-left:8px">언락 챔피언:</span>
      <div class="toggle-wrap">
        <button class="toggle-btn active" id="includeUnlocks">포함</button>
      </div>
    </div>
  </div>

  <div id="resultsArea">
    <div class="empty-state">
      <div class="empty-icon">⬡</div>
      <h3>기물을 선택해주세요</h3>
      <p style="font-size:13px;margin-top:6px">기물을 하나 이상 고르면<br>해당 기물이 모두 포함된 레벨별 최적 시너지 조합을 계산합니다</p>
    </div>
  </div>
</div>

<script>
// ============================================================
// DATA
// ============================================================
const TRAITS = {
  Bilgewater:{bp:[3,5,7,10]}, Demacia:{bp:[3,5,7,11]}, Freljord:{bp:[3,5,7]},
  Ionia:{bp:[3,5,7]}, Ixtal:{bp:[3,5,7]}, Noxus:{bp:[3,5,7]},
  Piltover:{bp:[3,5,7]}, ShadowIsles:{bp:[2,4,6]}, Shurima:{bp:[2,4]},
  Targon:{bp:[3,5]}, Void:{bp:[3,6,8]}, Yordle:{bp:[3,6]}, Zaun:{bp:[3,5,7]},
  Darkin:{bp:[1,2,3]},
  Arcanist:{bp:[2,4,6,8]}, Bruiser:{bp:[2,4,6]}, Defender:{bp:[2,4,6,8]},
  Disruptor:{bp:[2,4]}, Gunslinger:{bp:[2,4,6]}, Invoker:{bp:[2,4,6]},
  Juggernaut:{bp:[2,4,6]}, Longshot:{bp:[2,4]}, Quickstriker:{bp:[2,4,6]},
  Slayer:{bp:[2,4,6]}, Vanquisher:{bp:[2,4]}, Warden:{bp:[2,4,6]},
};

// pos: 'front'(사거리 1~2) | 'back'(사거리 3+)
const CHAMPS = [
  // 1코스트
  {id:'Anivia',     kr:'아니비아',       cost:1, pos:'back',  traits:['Freljord','Invoker'],               u:false},
  {id:'Blitzcrank', kr:'블리츠크랭크',   cost:1, pos:'front', traits:['Zaun','Juggernaut'],                u:false},
  {id:'Briar',      kr:'브라이어',       cost:1, pos:'front', traits:['Noxus','Slayer','Juggernaut'],      u:false},
  {id:'Caitlyn',    kr:'케이틀린',       cost:1, pos:'back',  traits:['Piltover','Longshot'],              u:false},
  {id:'Illaoi',     kr:'일라오이',       cost:1, pos:'front', traits:['Bilgewater','Bruiser'],             u:false},
  {id:'JarvanIV',   kr:'자르반 4세',     cost:1, pos:'front', traits:['Demacia','Defender'],               u:false},
  {id:'Jhin',       kr:'진',             cost:1, pos:'back',  traits:['Ionia','Gunslinger'],               u:false},
  {id:'KogMaw',     kr:'콕 모',          cost:1, pos:'back',  traits:['Void','Arcanist','Longshot'],       u:false},
  {id:'Lulu',       kr:'룰루',           cost:1, pos:'back',  traits:['Yordle','Arcanist'],                u:false},
  {id:'Qiyana',     kr:'큐이야나',       cost:1, pos:'front', traits:['Ixtal','Slayer'],                   u:false},
  {id:'Rumble',     kr:'럼블',           cost:1, pos:'front', traits:['Yordle','Defender'],                u:false},
  {id:'Shen',       kr:'쉔',             cost:1, pos:'front', traits:['Ionia','Bruiser'],                  u:false},
  {id:'Sona',       kr:'소나',           cost:1, pos:'back',  traits:['Demacia','Invoker'],                u:false},
  {id:'Viego',      kr:'비에고',         cost:1, pos:'front', traits:['ShadowIsles','Quickstriker'],       u:false},
  // 2코스트
  {id:'Aphelios',   kr:'아펠리오스',     cost:2, pos:'back',  traits:['Targon'],                           u:false},
  {id:'Ashe',       kr:'애쉬',           cost:2, pos:'back',  traits:['Freljord','Quickstriker'],          u:false},
  {id:'Bard',       kr:'바드',           cost:2, pos:'back',  traits:['Bilgewater','Arcanist'],            u:true },
  {id:'ChoGath',    kr:'초가스',         cost:2, pos:'front', traits:['Void','Juggernaut'],                u:false},
  {id:'Ekko',       kr:'에코',           cost:2, pos:'front', traits:['Zaun','Disruptor'],                 u:false},
  {id:'Graves',     kr:'그레이브즈',     cost:2, pos:'back',  traits:['Bilgewater','Gunslinger'],          u:true },
  {id:'Neeko',      kr:'니코',           cost:2, pos:'back',  traits:['Ixtal','Arcanist','Defender'],      u:false},
  {id:'Orianna',    kr:'오리아나',       cost:2, pos:'back',  traits:['Piltover','Invoker'],               u:false},
  {id:'Poppy',      kr:'뽀삐',           cost:2, pos:'front', traits:['Demacia','Yordle','Juggernaut'],    u:true },
  {id:'RekSai',     kr:'렉사이',         cost:2, pos:'front', traits:['Void','Vanquisher'],                u:false},
  {id:'Sion',       kr:'사이온',         cost:2, pos:'front', traits:['Noxus','Bruiser'],                  u:false},
  {id:'Teemo',      kr:'티모',           cost:2, pos:'back',  traits:['Yordle','Longshot'],                u:false},
  {id:'Tristana',   kr:'트리스타나',     cost:2, pos:'back',  traits:['Yordle','Gunslinger'],              u:false},
  {id:'Tryndamere', kr:'트린다미어',     cost:2, pos:'front', traits:['Freljord','Slayer'],                u:true },
  {id:'TwistedFate',kr:'트위스티드 페이트',cost:2,pos:'back', traits:['Bilgewater','Quickstriker'],        u:false},
  {id:'Vi',         kr:'바이',           cost:2, pos:'front', traits:['Piltover','Zaun','Defender'],       u:false},
  {id:'XinZhao',    kr:'신짜오',         cost:2, pos:'front', traits:['Demacia','Ionia','Warden'],         u:false},
  {id:'Yasuo',      kr:'야스오',         cost:2, pos:'front', traits:['Ionia','Slayer'],                   u:false},
  {id:'Yorick',     kr:'요릭',           cost:2, pos:'front', traits:['ShadowIsles','Warden'],             u:false},
  // 3코스트
  {id:'Ahri',       kr:'아리',           cost:3, pos:'back',  traits:['Ionia','Arcanist'],                 u:false},
  {id:'Darius',     kr:'다리우스',       cost:3, pos:'front', traits:['Noxus','Defender'],                 u:false},
  {id:'DrMundo',    kr:'문도 박사',      cost:3, pos:'front', traits:['Zaun','Bruiser'],                   u:false},
  {id:'Draven',     kr:'드레이븐',       cost:3, pos:'back',  traits:['Noxus','Quickstriker'],             u:false},
  {id:'Gangplank',  kr:'갱플랭크',       cost:3, pos:'back',  traits:['Bilgewater','Slayer','Vanquisher'], u:false},
  {id:'Gwen',       kr:'그웬',           cost:3, pos:'front', traits:['ShadowIsles','Disruptor'],          u:false},
  {id:'Jinx',       kr:'징크스',         cost:3, pos:'back',  traits:['Zaun','Gunslinger'],                u:false},
  {id:'Kennen',     kr:'케넨',           cost:3, pos:'front', traits:['Ionia','Yordle','Defender'],        u:true },
  {id:'KobukoYuumi',kr:'코부코&유미',    cost:3, pos:'back',  traits:['Yordle','Bruiser','Invoker'],       u:false},
  {id:'LeBlanc',    kr:'르블랑',         cost:3, pos:'back',  traits:['Noxus','Invoker'],                  u:false},
  {id:'Leona',      kr:'레오나',         cost:3, pos:'front', traits:['Targon'],                           u:false},
  {id:'Loris',      kr:'로리스',         cost:3, pos:'front', traits:['Piltover','Warden'],                u:false},
  {id:'Malzahar',   kr:'말자하르',       cost:3, pos:'back',  traits:['Void','Disruptor'],                 u:false},
  {id:'Milio',      kr:'밀리오',         cost:3, pos:'back',  traits:['Ixtal','Invoker'],                  u:false},
  {id:'Nautilus',   kr:'노틸러스',       cost:3, pos:'front', traits:['Bilgewater','Juggernaut','Warden'], u:false},
  {id:'Sejuani',    kr:'세주아니',       cost:3, pos:'front', traits:['Freljord','Defender'],              u:false},
  {id:'Vayne',      kr:'베인',           cost:3, pos:'back',  traits:['Demacia','Longshot'],               u:false},
  {id:'Zoe',        kr:'조이',           cost:3, pos:'back',  traits:['Targon'],                           u:false},
  // 4코스트
  {id:'Ambessa',    kr:'암베사',         cost:4, pos:'front', traits:['Noxus','Vanquisher'],               u:false},
  {id:'BelVeth',    kr:'벨베스',         cost:4, pos:'front', traits:['Void','Slayer'],                    u:false},
  {id:'Braum',      kr:'브라움',         cost:4, pos:'front', traits:['Freljord','Warden'],                u:false},
  {id:'Diana',      kr:'다이애나',       cost:4, pos:'front', traits:['Targon'],                           u:false},
  {id:'Fizz',       kr:'피즈',           cost:4, pos:'front', traits:['Bilgewater','Yordle'],              u:true },
  {id:'Garen',      kr:'가렌',           cost:4, pos:'front', traits:['Demacia','Defender'],               u:false},
  {id:'KaiSa',      kr:'카이사',         cost:4, pos:'back',  traits:['Void','Longshot'],                  u:true },
  {id:'Kalista',    kr:'칼리스타',       cost:4, pos:'back',  traits:['ShadowIsles','Vanquisher'],         u:false},
  {id:'Lissandra',  kr:'리산드라',       cost:4, pos:'back',  traits:['Freljord','Invoker'],               u:false},
  {id:'Lux',        kr:'럭스',           cost:4, pos:'back',  traits:['Demacia','Arcanist'],               u:false},
  {id:'MissFortune',kr:'미스 포춘',      cost:4, pos:'back',  traits:['Bilgewater','Gunslinger'],          u:false},
  {id:'Nasus',      kr:'나서스',         cost:4, pos:'front', traits:['Shurima'],                          u:false},
  {id:'Nidalee',    kr:'니달리',         cost:4, pos:'back',  traits:['Ixtal'],                            u:true },
  {id:'Renekton',   kr:'레넥톤',         cost:4, pos:'front', traits:['Shurima','Juggernaut'],             u:false},
  {id:'RiftHerald', kr:'균열 선구자',    cost:4, pos:'front', traits:['Void','Bruiser'],                   u:false},
  {id:'Seraphine',  kr:'세라핀',         cost:4, pos:'back',  traits:['Piltover','Disruptor'],             u:false},
  {id:'Singed',     kr:'신지드',         cost:4, pos:'front', traits:['Zaun','Juggernaut'],                u:false},
  {id:'Skarner',    kr:'스카너',         cost:4, pos:'front', traits:['Ixtal'],                            u:false},
  {id:'Swain',      kr:'스웨인',         cost:4, pos:'front', traits:['Noxus','Arcanist','Juggernaut'],    u:false},
  {id:'Taric',      kr:'타릭',           cost:4, pos:'front', traits:['Targon'],                           u:false},
  {id:'Veigar',     kr:'베이가',         cost:4, pos:'back',  traits:['Yordle','Arcanist'],                u:false},
  {id:'Warwick',    kr:'워윅',           cost:4, pos:'front', traits:['Zaun','Quickstriker'],              u:false},
  {id:'Wukong',     kr:'우콩',           cost:4, pos:'front', traits:['Ionia','Bruiser'],                  u:false},
  {id:'Yone',       kr:'요네',           cost:4, pos:'front', traits:['Ionia','Slayer'],                   u:true },
  {id:'Yunara',     kr:'유나라',         cost:4, pos:'back',  traits:['Ionia','Quickstriker'],             u:false},
  // 5코스트
  {id:'Aatrox',     kr:'아트록스',       cost:5, pos:'front', traits:['Darkin','Slayer'],                  u:true },
  {id:'Annie',      kr:'애니',           cost:5, pos:'back',  traits:['Arcanist'],                         u:false},
  {id:'Azir',       kr:'아지르',         cost:5, pos:'back',  traits:['Shurima','Disruptor'],              u:false},
  {id:'Fiddlesticks',kr:'피들스틱',      cost:5, pos:'back',  traits:['Vanquisher'],                       u:false},
  {id:'Galio',      kr:'갈리오',         cost:5, pos:'front', traits:['Demacia'],                          u:true },
  {id:'Kindred',    kr:'킨드레드',       cost:5, pos:'back',  traits:['Quickstriker'],                     u:false},
  {id:'LucianSenna',kr:'루시안&세나',    cost:5, pos:'back',  traits:['Gunslinger'],                       u:false},
  {id:'Mel',        kr:'멜',             cost:5, pos:'back',  traits:['Noxus','Disruptor'],                u:false},
  {id:'Ornn',       kr:'오른',           cost:5, pos:'front', traits:['Warden'],                           u:false},
  {id:'Sett',       kr:'세트',           cost:5, pos:'front', traits:['Ionia'],                            u:true },
  {id:'Shyvana',    kr:'쉬바나',         cost:5, pos:'front', traits:['Juggernaut'],                       u:false},
  {id:'THex',       kr:'T-헥스',         cost:5, pos:'back',  traits:['Piltover','Gunslinger'],            u:true },
  {id:'TahmKench',  kr:'탐 켄치',        cost:5, pos:'front', traits:['Bilgewater','Bruiser'],             u:true },
  {id:'Thresh',     kr:'쓰레쉬',         cost:5, pos:'front', traits:['ShadowIsles','Warden'],             u:false},
  {id:'Tibbers',    kr:'티버스',         cost:5, pos:'front', traits:['Arcanist'],                         u:false},
  {id:'Volibear',   kr:'볼리베어',       cost:5, pos:'front', traits:['Freljord','Bruiser'],               u:true },
  {id:'Xerath',     kr:'제라스',         cost:5, pos:'back',  traits:['Shurima'],                          u:true },
  {id:'Ziggs',      kr:'직스',           cost:5, pos:'back',  traits:['Zaun','Yordle','Longshot'],         u:false},
  {id:'Zilean',     kr:'질리언',         cost:5, pos:'back',  traits:['Invoker'],                          u:false},
  // 7코스트
  {id:'AurelionSol',kr:'아우렐리온 솔',  cost:7, pos:'back',  traits:['Targon'],                           u:true },
  {id:'BaronNashor',kr:'바론 내셔',      cost:7, pos:'front', traits:['Void'],                             u:true },
  {id:'Brock',      kr:'브록',           cost:7, pos:'front', traits:['Ixtal'],                            u:true },
  {id:'Ryze',       kr:'라이즈',         cost:7, pos:'back',  traits:[],                                   u:true },
  {id:'Sylas',      kr:'사일러스',       cost:7, pos:'front', traits:['Arcanist','Defender'],              u:true },
  {id:'Zaahen',     kr:'자아헨',         cost:7, pos:'front', traits:['Darkin'],                           u:true },
];

// ============================================================
// SCORE
// ============================================================
function getTier(traitId, count) {
  const t = TRAITS[traitId]; if(!t) return 0;
  let tier=0;
  for(const bp of t.bp){ if(count>=bp) tier++; else break; }
  return tier;
}
function scoreComp(ids) {
  const tc={};
  for(const id of ids){ const c=CHAMPS.find(x=>x.id===id); if(!c) continue; for(const t of c.traits){ if(TRAITS[t]) tc[t]=(tc[t]||0)+1; } }
  let activeTiers=0,totalTiers=0; const activeTraits=[];
  for(const [trait,count] of Object.entries(tc)){ const tier=getTier(trait,count); if(tier>0){activeTiers++;totalTiers+=tier;activeTraits.push({trait,count,tier});} }
  return {score:activeTiers+totalTiers, activeTiers, totalTiers, activeTraits};
}
function getPositions(ids) {
  let f=0,b=0;
  for(const id of ids){ const c=CHAMPS.find(x=>x.id===id); if(!c) continue; if(c.pos==='front') f++; else b++; }
  return {front:f,back:b};
}

// ============================================================
// ALGORITHM
// ============================================================
function findBestComps(anchorIds, size, pool, topN=3, runs=340) {
  if(anchorIds.length > size) return [];
  for(const a of anchorIds) if(!pool.find(c=>c.id===a)) return [];
  const others = pool.filter(c=>!anchorIds.includes(c.id));
  const seen=new Set(), results=[];
  for(let r=0;r<runs;r++){
    const comp=[...anchorIds], avail=[...others];
    for(let i=avail.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[avail[i],avail[j]]=[avail[j],avail[i]];}
    while(comp.length<size && avail.length>0){
      let best=-1,bi=-1;
      const n=Math.min(avail.length,24);
      for(let i=0;i<n;i++){const {score}=scoreComp([...comp,avail[i].id]);if(score>best){best=score;bi=i;}}
      if(bi===-1) break;
      comp.push(avail[bi].id); avail.splice(bi,1);
    }
    if(comp.length<size) continue;
    const key=[...comp].sort().join(',');
    if(seen.has(key)) continue; seen.add(key);
    const sd=scoreComp(comp);
    results.push({comp,...sd});
  }
  results.sort((a,b)=>b.score-a.score);
  const top=[];
  for(const r of results){
    if(top.length>=topN) break;
    const ok=top.every(ex=>r.comp.filter(id=>ex.comp.includes(id)).length/r.comp.length<0.78);
    if(ok||top.length<topN) top.push(r);
  }
  return top.slice(0,topN);
}

// ============================================================
// UI
// ============================================================
let selectedChamps=[], includeUnlocks=true, levelFilter='all', acIdx=-1;
const $=id=>document.getElementById(id);
const champSearchEl=$('champSearch'), acListEl=$('acList'), searchBtnEl=$('searchBtn'),
      clearBtnEl=$('clearBtn'), chipsEl=$('selectedChips'), resultsEl=$('resultsArea'),
      inclBtn=$('includeUnlocks');

function norm(s){return s.toLowerCase().replace(/[\s'\.\·\&]+/g,'');}

function renderChips(){
  chipsEl.innerHTML=selectedChamps.map(c=>`
    <span class="selected-chip">
      <span class="cost-badge cost-${c.cost}">${c.cost}</span>
      ${c.kr}
      <span class="remove-btn" data-id="${c.id}">×</span>
    </span>`).join('');
  searchBtnEl.disabled=selectedChamps.length===0;
  clearBtnEl.style.display=selectedChamps.length>0?'':'none';
}

chipsEl.addEventListener('click',e=>{
  const btn=e.target.closest('.remove-btn'); if(!btn) return;
  selectedChamps=selectedChamps.filter(c=>c.id!==btn.dataset.id);
  renderChips(); refreshAc();
});
clearBtnEl.addEventListener('click',()=>{
  selectedChamps=[]; champSearchEl.value=''; renderChips(); acListEl.classList.remove('open');
});

function refreshAc(){
  const q=norm(champSearchEl.value.trim());
  if(!q){acListEl.classList.remove('open');return;}
  const matches=CHAMPS.filter(c=>norm(c.kr).includes(q)||norm(c.id).includes(q));
  if(!matches.length){acListEl.classList.remove('open');return;}
  const selSet=new Set(selectedChamps.map(c=>c.id));
  acIdx=-1;
  acListEl.innerHTML=matches.slice(0,15).map(c=>{
    const dis=selSet.has(c.id)?' already-sel':'';
    const rl=c.pos==='front'?'<span class="ac-range r-front">전방</span>':'<span class="ac-range r-back">후방</span>';
    return `<div class="ac-item${dis}" data-id="${c.id}">
      <span class="cost-badge cost-${c.cost}">${c.cost}</span>
      <div style="flex:1"><div class="ac-name">${c.kr}${c.u?' <span style="color:var(--teal);font-size:10px">🔓</span>':''}</div>
      <div class="ac-traits">${c.traits.join(' · ')||'—'}</div></div>${rl}</div>`;
  }).join('');
  acListEl.classList.add('open');
}

champSearchEl.addEventListener('input',refreshAc);
champSearchEl.addEventListener('keydown',e=>{
  const items=[...acListEl.querySelectorAll('.ac-item:not(.already-sel)')];
  if(e.key==='ArrowDown'){acIdx=Math.min(acIdx+1,items.length-1);items.forEach((el,i)=>el.classList.toggle('active-ac',i===acIdx));e.preventDefault();}
  else if(e.key==='ArrowUp'){acIdx=Math.max(acIdx-1,-1);items.forEach((el,i)=>el.classList.toggle('active-ac',i===acIdx));e.preventDefault();}
  else if(e.key==='Enter'){if(acIdx>=0&&items[acIdx])pickChamp(items[acIdx].dataset.id);else if(selectedChamps.length>0)runSearch();e.preventDefault();}
  else if(e.key==='Escape')acListEl.classList.remove('open');
});
acListEl.addEventListener('click',e=>{
  const item=e.target.closest('.ac-item:not(.already-sel)');if(item)pickChamp(item.dataset.id);
});
document.addEventListener('click',e=>{if(!acListEl.contains(e.target)&&e.target!==champSearchEl)acListEl.classList.remove('open');});

function pickChamp(id){
  const c=CHAMPS.find(x=>x.id===id);
  if(!c||selectedChamps.find(x=>x.id===id))return;
  selectedChamps.push(c); champSearchEl.value=''; acListEl.classList.remove('open'); renderChips();
}

document.querySelectorAll('[data-level]').forEach(btn=>{
  btn.addEventListener('click',()=>{
    document.querySelectorAll('[data-level]').forEach(b=>b.classList.remove('active'));
    btn.classList.add('active'); levelFilter=btn.dataset.level;
  });
});
inclBtn.addEventListener('click',()=>{
  includeUnlocks=!includeUnlocks;
  inclBtn.textContent=includeUnlocks?'포함':'제외';
  inclBtn.classList.toggle('active',includeUnlocks);
});
searchBtnEl.addEventListener('click',runSearch);

function runSearch(){
  if(!selectedChamps.length)return;
  const pool=CHAMPS.filter(c=>includeUnlocks||!c.u);
  const anchorIds=selectedChamps.map(c=>c.id);
  const missing=anchorIds.filter(id=>!pool.find(c=>c.id===id));
  if(missing.length){
    const names=missing.map(id=>CHAMPS.find(c=>c.id===id)?.kr||id).join(', ');
    resultsEl.innerHTML=`<div class="err">⚠️ <b>${names}</b> 은(는) 언락 챔피언입니다. 언락 포함으로 설정해주세요.</div>`;return;
  }
  const anchorNames=selectedChamps.map(c=>c.kr).join(' + ');
  resultsEl.innerHTML=`<div class="loading"><div class="spinner"></div><p>${anchorNames} 포함 조합 탐색 중…</p></div>`;
  const levels=levelFilter==='all'?[4,5,6,7,8,9,10]:[parseInt(levelFilter)];
  setTimeout(()=>{
    try{
      let html=`<div class="results-header">
        <div class="rh-left"><span style="color:var(--white)">${anchorNames}</span>&nbsp;포함 <span>레벨별 최적 조합</span></div>
        <div class="results-legend">
          <span class="legend-item"><span class="ldot ldot-f"></span><span style="color:var(--orange)">전방</span></span>
          <span class="legend-item"><span class="ldot ldot-b"></span><span style="color:var(--blue)">후방</span></span>
          <span style="font-size:10px;color:var(--muted)">점수=활성시너지+티어합산</span>
        </div>
      </div>`;
      let hasResult=false;
      for(const lv of levels){
        if(pool.length<lv||anchorIds.length>lv) continue;
        const top3=findBestComps(anchorIds,lv,pool,3,330);
        if(!top3.length) continue;
        hasResult=true;
        html+=`<div class="level-section"><div class="level-title">LEVEL ${lv}</div><div class="comp-cards">`;
        top3.forEach((r,i)=>{ html+=renderCard(r,i+1,anchorIds,i*70); });
        html+=`</div></div>`;
      }
      if(!hasResult) html+=`<div class="err">조합을 찾지 못했습니다. 앵커 수가 레벨보다 많거나 풀에 기물이 부족합니다.</div>`;
      resultsEl.innerHTML=html;
    }catch(e){resultsEl.innerHTML=`<div class="err">오류: ${e.message}</div>`;}
  },20);
}

// ── Render card ──
const tierCls=['','st1','st2','st3','st4'];
function renderCard(result, rank, anchorIds, delay){
  const {front,back}=getPositions(result.comp);
  const total=result.comp.length;
  const fp=Math.round(front/total*100), bp=100-fp;

  const frontIds=result.comp.filter(id=>CHAMPS.find(c=>c.id===id)?.pos==='front');
  const backIds =result.comp.filter(id=>CHAMPS.find(c=>c.id===id)?.pos==='back');

  function chip(id){
    const c=CHAMPS.find(x=>x.id===id); if(!c) return '';
    const isAnc=anchorIds.includes(id);
    const pc=isAnc?'':(c.pos==='front'?'cfr':'cbk');
    return `<span class="chip ${isAnc?'anchor':''} ${pc}">
      <span class="cost-badge cost-${c.cost}" style="width:14px;height:14px;font-size:7px">${c.cost}</span>${c.kr}</span>`;
  }

  const synBadges=[...result.activeTraits]
    .sort((a,b)=>b.tier-a.tier||b.count-a.count)
    .map(({trait,count,tier})=>`<span class="syn ${tierCls[Math.min(tier,4)]}">${trait}<span class="sc">${count}</span></span>`)
    .join('');

  return `<div class="comp-card" style="animation-delay:${delay}ms">
    <span class="rank-badge rk${rank}">${rank}</span>
    <div class="card-header">
      <div class="header-top">
        <div class="score-row"><span class="score-num">${result.score}</span><span class="score-sub">pts</span></div>
        <div class="stats-mini">
          <div>활성 시너지 <strong>${result.activeTiers}개</strong></div>
          <div>티어 합산 <strong>${result.totalTiers}</strong></div>
        </div>
      </div>
      <div class="pos-wrap">
        <span class="pos-lbl plf">전방 ${front}</span>
        <div class="pos-bar"><div class="bar-f" style="width:${fp}%"></div><div class="bar-b" style="width:${bp}%"></div></div>
        <span class="pos-lbl plb">${back} 후방</span>
      </div>
    </div>
    ${frontIds.length?`<div class="sec-label">⚔ 전방</div><div class="champ-list">${frontIds.map(chip).join('')}</div>`:''}
    ${backIds.length?`<div class="sec-label">🏹 후방</div><div class="champ-list">${backIds.map(chip).join('')}</div>`:''}
    <div class="syn-list">${synBadges}</div>
  </div>`;
}
</script>
</body>
</html>
