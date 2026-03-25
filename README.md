<!DOCTYPE html>
<html lang="sv">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>SAVR – Utdelningsförslag vid orderläggning</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<style>
  :root {
    --bg: #0A0A0A;
    --card: #1B1B1B;
    --card-hover: #222;
    --elevated: #252525;
    --input-bg: #1B1B1B;
    --border: rgba(255,255,255,0.08);
    --border-light: rgba(255,255,255,0.12);
    --border-input: rgba(255,255,255,0.15);
    --white: #fff;
    --w80: rgba(255,255,255,0.8);
    --w60: rgba(255,255,255,0.6);
    --w40: rgba(255,255,255,0.4);
    --w20: rgba(255,255,255,0.2);
    --w10: rgba(255,255,255,0.06);
    --muted: #9E9E9E;
    --dim: #666;
    --accent: #C084FC;
    --accent-dim: #A855F7;
    --accent-bg: rgba(168,85,247,0.1);
    --accent-border: rgba(168,85,247,0.25);
    --green: #4ADE80;
    --green-dim: #22C55E;
    --green-bg: rgba(74,222,128,0.08);
    --green-border: rgba(74,222,128,0.15);
    --red: #F87171;
    --red-bg: rgba(248,113,113,0.12);
    --yellow: #FBBF24;
    --yellow-bg: rgba(251,191,36,0.1);
    --blue: #60A5FA;
    --r: 18px;
    --rm: 14px;
    --rs: 10px;
    --pill: 100px;
  }
  * { margin:0; padding:0; box-sizing:border-box; }
  body {
    font-family: 'proxima-nova','Proxima Nova','Inter','Helvetica Neue',-apple-system,sans-serif;
    background: var(--bg); color: var(--white);
    line-height: 1.5; min-height: 100vh;
    -webkit-font-smoothing: antialiased;
  }
  .app { max-width: 480px; margin: 0 auto; min-height: 100vh; position: relative; }

  /* ---- Close button ---- */
  .close-btn {
    position: absolute; top: 16px; left: 16px; z-index: 60;
    width: 36px; height: 36px; border-radius: 50%;
    background: transparent; border: none; color: var(--white);
    font-size: 24px; cursor: pointer; display: flex;
    align-items: center; justify-content: center;
    transition: background 0.15s;
  }
  .close-btn:hover { background: var(--w10); }

  /* ---- Stock Header ---- */
  .stock-header {
    text-align: center; padding: 20px 20px 0;
  }
  .sh-name { font-size: 18px; font-weight: 700; letter-spacing: 0.02em; }
  .sh-flag { display: inline-block; margin-left: 6px; font-size: 14px; }
  .sh-price { font-size: 14px; color: var(--muted); margin-top: 2px; }
  .sh-change { color: var(--green); font-weight: 500; }
  .sh-change.neg { color: var(--red); }

  /* ---- Order Book ---- */
  .orderbook {
    display: flex; align-items: center; justify-content: center;
    gap: 0; padding: 16px 20px 12px; font-size: 13px;
  }
  .ob-label { text-align: center; margin-bottom: 8px; }
  .ob-labels { font-size: 13px; font-weight: 600; color: var(--w60); display: flex; gap: 12px; justify-content: center; margin-bottom: 6px; }
  .ob-row {
    display: flex; align-items: center; justify-content: center; gap: 6px;
  }
  .ob-vol { font-size: 13px; color: var(--muted); min-width: 50px; }
  .ob-vol.left { text-align: right; }
  .ob-vol.right { text-align: left; }
  .ob-bid {
    background: rgba(96,165,250,0.12); color: var(--blue);
    padding: 6px 16px; border-radius: 8px; font-weight: 600; font-size: 14px;
  }
  .ob-ask {
    background: var(--red-bg); color: var(--red);
    padding: 6px 16px; border-radius: 8px; font-weight: 600; font-size: 14px;
    min-width: 90px; text-align: center;
  }
  .ob-ask-vol { font-size: 13px; color: var(--muted); margin-left: 8px; }

  /* ---- Chart Placeholder ---- */
  .chart-placeholder {
    height: 200px; margin: 0 20px;
    background: var(--card); border-radius: var(--rm);
    display: flex; align-items: center; justify-content: center;
    position: relative; overflow: hidden;
  }
  .chart-placeholder canvas { width: 100% !important; height: 100% !important; }
  .chart-dots {
    display: flex; gap: 6px; justify-content: center;
    padding: 10px 0 6px;
  }
  .chart-dot {
    width: 6px; height: 6px; border-radius: 3px;
    background: var(--w20); cursor: pointer;
  }
  .chart-dot.active { background: var(--white); }

  /* ---- Account Selector ---- */
  .account-bar {
    display: flex; align-items: center; justify-content: space-between;
    padding: 14px 20px;
  }
  .account-left { display: flex; align-items: center; gap: 10px; }
  .account-badge {
    background: var(--white); color: var(--bg);
    font-size: 11px; font-weight: 700; padding: 4px 8px;
    border-radius: 6px; letter-spacing: 0.04em;
  }
  .account-name {
    font-size: 14px; font-weight: 600; color: var(--white);
    display: flex; align-items: center; gap: 4px;
  }
  .account-name svg { opacity: 0.5; }
  .account-sub { font-size: 12px; color: var(--dim); }
  .account-balance { font-size: 13px; color: var(--muted); display: flex; align-items: center; gap: 4px; }

  /* ---- Order Inputs (matches SAVR exactly) ---- */
  .order-inputs {
    display: flex; gap: 8px; padding: 0 20px; margin-bottom: 14px;
  }
  .oi-field {
    flex: 1; background: var(--input-bg);
    border: 1px solid var(--border-input);
    border-radius: var(--rs); padding: 10px 14px;
    transition: border-color 0.2s;
  }
  .oi-field:focus-within { border-color: var(--accent); }
  .oi-label { font-size: 11px; color: var(--muted); margin-bottom: 2px; display: block; }
  .oi-value {
    width: 100%; background: transparent; border: none;
    font-size: 17px; font-weight: 700; color: var(--white);
    outline: none; font-family: inherit;
  }
  .oi-value::placeholder { color: var(--w20); }
  .oi-value::-webkit-inner-spin-button { display: none; }

  /* ---- CTA Row ---- */
  .cta-row {
    display: flex; gap: 10px; padding: 0 20px 16px; align-items: stretch;
  }
  .cta-btn {
    flex: 1; padding: 16px 24px;
    background: var(--white); color: var(--bg);
    border: none; border-radius: var(--pill);
    font-size: 15px; font-weight: 700; cursor: pointer;
    font-family: inherit; display: flex;
    align-items: center; justify-content: space-between;
    transition: all 0.2s; gap: 8px;
  }
  .cta-btn:hover { background: rgba(255,255,255,0.9); }
  .cta-btn:active { transform: scale(0.99); }
  .cta-total { font-weight: 600; color: rgba(0,0,0,0.5); display: flex; align-items: center; gap: 4px; }
  .cta-more {
    width: 50px; height: 50px; border-radius: var(--rm);
    background: var(--card); border: 1px solid var(--border);
    color: var(--white); font-size: 20px; cursor: pointer;
    display: flex; align-items: center; justify-content: center;
    transition: background 0.15s; flex-shrink: 0;
  }
  .cta-more:hover { background: var(--card-hover); }

  /* ---- Tabs ---- */
  .tabs-bar {
    display: flex; gap: 0; padding: 0 20px;
    border-top: 1px solid var(--border);
  }
  .tab-btn {
    flex: 1; text-align: center; padding: 14px 0;
    font-size: 14px; font-weight: 500; color: var(--dim);
    background: transparent; border: none; cursor: pointer;
    font-family: inherit; position: relative; transition: color 0.2s;
  }
  .tab-btn.active { color: var(--white); }
  .tab-btn.active::after {
    content: ''; position: absolute; bottom: 0; left: 20%; right: 20%;
    height: 2px; background: var(--white); border-radius: 1px;
  }

  /* ---- Order field sub-info ---- */
  .oi-sub { font-size: 10px; color: var(--dim); margin-top: 3px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  .oi-sub.eur { color: var(--accent); font-weight: 500; }

  /* ===== Dividend Preview + Target ===== */
  .dividend-preview {
    margin: 0 20px 14px; padding: 16px;
    background: linear-gradient(135deg, rgba(74,222,128,0.06), rgba(168,85,247,0.04));
    border: 1px solid var(--green-border);
    border-radius: var(--rm);
  }
  @keyframes fadeSlide {
    from { opacity: 0; transform: translateY(-8px); }
    to { opacity: 1; transform: translateY(0); }
  }
  .dp-row {
    display: flex; align-items: center; justify-content: space-between; gap: 8px;
  }
  .dp-label { font-size: 12px; color: var(--muted); }
  .dp-value { font-size: 15px; font-weight: 700; color: var(--green); }
  .dp-sub { font-size: 11px; color: var(--dim); margin-top: 6px; display: flex; align-items: center; gap: 6px; }
  .dp-yield-badge {
    font-size: 10px; font-weight: 600; padding: 2px 8px;
    border-radius: var(--pill); background: var(--green-bg); color: var(--green);
  }
  .dp-exdate { font-size: 11px; color: var(--yellow); display: flex; align-items: center; gap: 4px; }

  /* Dividend target (inside preview) */
  .dp-target {
    margin-top: 12px; padding-top: 12px;
    border-top: 1px solid var(--border);
  }
  #dpEstimate + .dp-target { margin-top: 12px; }
  .dp-target-label {
    font-size: 12px; font-weight: 600; color: var(--green);
    margin-bottom: 8px;
  }
  .dp-target-inputs {
    display: flex; gap: 6px; align-items: stretch;
  }
  .dp-target-field {
    flex: 1; display: flex; align-items: center; gap: 6px;
    background: rgba(74,222,128,0.04);
    border: 1px solid var(--green-border);
    border-radius: var(--rs); padding: 0 12px;
    transition: border-color 0.2s;
  }
  .dp-target-field:focus-within { border-color: var(--green); }
  .dp-target-input {
    flex: 1; background: transparent; border: none;
    font-size: 16px; font-weight: 700; color: var(--green);
    outline: none; font-family: inherit; padding: 10px 0;
    min-width: 0;
  }
  .dp-target-input::placeholder { color: rgba(74,222,128,0.3); font-weight: 500; }
  .dp-target-input::-webkit-inner-spin-button { display: none; }
  .dp-target-cur { font-size: 12px; color: rgba(74,222,128,0.5); font-weight: 600; flex-shrink: 0; }
  .dp-target-period {
    display: flex; background: var(--w10); border-radius: var(--rs);
    border: 1px solid var(--border); overflow: hidden; flex-shrink: 0;
  }
  .dtp-btn {
    padding: 10px 12px; font-size: 12px; font-weight: 600;
    background: transparent; border: none; color: var(--dim);
    cursor: pointer; font-family: inherit; transition: all 0.15s;
    white-space: nowrap;
  }
  .dtp-btn.active { background: var(--green-bg); color: var(--green); }
  .dtp-btn:hover:not(.active) { color: var(--w80); }
  .dp-target-result {
    margin-top: 8px; font-size: 12px; color: var(--muted);
    display: none; align-items: center; justify-content: space-between;
    gap: 8px; animation: fadeSlide 0.2s ease;
  }
  .dp-target-result.show { display: flex; }
  .dp-target-result span { line-height: 1.4; }
  .dtr-apply {
    padding: 7px 16px; border-radius: var(--pill);
    background: var(--green); border: none; color: var(--bg);
    font-size: 12px; font-weight: 700; cursor: pointer; font-family: inherit;
    transition: opacity 0.15s; white-space: nowrap; flex-shrink: 0;
  }
  .dtr-apply:hover { opacity: 0.85; }

  /* Expand button */
  .dp-expand {
    margin-top: 10px; padding-top: 10px;
    border-top: 1px solid var(--border);
    display: flex; align-items: center; justify-content: space-between;
  }
  .dp-expand-btn {
    font-size: 12px; font-weight: 600; color: var(--accent);
    background: none; border: none; cursor: pointer;
    font-family: inherit; display: flex; align-items: center; gap: 4px;
    transition: opacity 0.15s;
  }
  .dp-expand-btn:hover { opacity: 0.8; }

  /* ===== IMPROVEMENT 2: Quick Amount Buttons ===== */
  .quick-amounts {
    display: flex; gap: 6px; padding: 0 20px; margin-bottom: 12px;
    overflow-x: auto; -webkit-overflow-scrolling: touch;
  }
  .quick-amounts::-webkit-scrollbar { display: none; }
  .qa-btn {
    padding: 8px 16px; border-radius: var(--pill);
    background: var(--w10); border: 1px solid var(--border);
    color: var(--w80); font-size: 13px; font-weight: 600;
    cursor: pointer; font-family: inherit;
    white-space: nowrap; transition: all 0.15s; flex-shrink: 0;
  }
  .qa-btn:hover { background: var(--w20); border-color: var(--w40); }
  .qa-btn.active { background: var(--accent-bg); border-color: var(--accent-border); color: var(--accent); }

  /* ===== FULL ANALYSIS PANEL ===== */
  .analysis-panel {
    padding: 0 20px; display: none;
    animation: fadeSlide 0.35s ease;
  }
  .analysis-panel.open { display: block; }
  .ap-section {
    background: var(--card); border: 1px solid var(--border);
    border-radius: var(--rm); padding: 18px; margin-bottom: 12px;
  }
  .ap-title { font-size: 14px; font-weight: 700; color: var(--white); margin-bottom: 14px; }

  /* Stats grid */
  .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-bottom: 12px; }
  .stat {
    background: var(--elevated); border: 1px solid var(--border);
    border-radius: var(--rs); padding: 14px;
  }
  .stat.hl { border-color: var(--accent-border); background: var(--accent-bg); }
  .stat-lbl { font-size: 10px; color: var(--dim); font-weight: 500; text-transform: uppercase; letter-spacing: 0.05em; margin-bottom: 4px; }
  .stat-val { font-size: 20px; font-weight: 800; color: var(--white); }
  .stat-sub { font-size: 11px; color: var(--dim); margin-top: 2px; }

  /* Estimate highlight */
  .estimate {
    background: linear-gradient(135deg, var(--accent-bg), var(--green-bg));
    border: 1px solid var(--accent-border);
    border-radius: var(--rm); padding: 20px; margin-bottom: 12px;
  }
  .est-title { font-size: 12px; font-weight: 600; color: var(--w60); margin-bottom: 12px; }
  .est-row { display: flex; gap: 24px; flex-wrap: wrap; align-items: flex-end; }
  .est-amount { font-size: 28px; font-weight: 900; color: var(--green); letter-spacing: -0.02em; }
  .est-label { font-size: 11px; color: var(--dim); margin-top: 2px; }
  .est-div { width: 1px; background: var(--border-light); align-self: stretch; }

  /* Chart */
  .chart-wrap { position: relative; height: 180px; }

  /* Calendar items */
  .cal-item {
    display: flex; align-items: center; justify-content: space-between;
    padding: 10px 14px; border-radius: var(--rs);
    background: var(--elevated); border: 1px solid var(--border);
    margin-bottom: 6px;
  }
  .cal-item.up { background: var(--green-bg); border-color: var(--green-border); }
  .cal-left { display: flex; align-items: center; gap: 8px; }
  .cal-dot { width: 7px; height: 7px; border-radius: 50%; background: var(--w20); flex-shrink: 0; }
  .cal-dot.up { background: var(--green); }
  .cal-q { font-size: 13px; font-weight: 600; }
  .cal-ex { font-size: 11px; color: var(--dim); }
  .cal-right { display: flex; align-items: center; gap: 8px; }
  .cal-amt { font-size: 13px; font-weight: 700; color: var(--green); }
  .sbadge { font-size: 9px; font-weight: 600; padding: 3px 8px; border-radius: var(--pill); text-transform: uppercase; letter-spacing: 0.04em; }
  .sb-up { background: var(--green-bg); color: var(--green); }
  .sb-paid { background: var(--w10); color: var(--dim); }

  /* Compare */
  .cmp-item {
    display: flex; align-items: center; justify-content: space-between;
    padding: 12px 14px; border-radius: var(--rs);
    background: var(--elevated); border: 1px solid var(--border);
    margin-bottom: 6px; gap: 8px;
  }
  .cmp-item.me { background: var(--accent-bg); border-color: var(--accent-border); }
  .cmp-name { font-size: 13px; font-weight: 600; }
  .cmp-tag { font-size: 10px; color: var(--accent); font-weight: 600; margin-left: 6px; }
  .cmp-ticker { font-size: 11px; color: var(--dim); margin-left: 4px; }
  .cmp-metrics { display: flex; gap: 12px; flex-shrink: 0; }
  .cmp-m { font-size: 11px; color: var(--w60); white-space: nowrap; }
  .cmp-m strong { color: var(--white); }
  .cmp-m.better strong { color: var(--green); }

  /* Footer */
  .footer-note {
    margin: 20px 20px 0; padding: 14px 16px;
    background: var(--card); border-radius: var(--rm);
    border: 1px solid var(--border);
    font-size: 11px; color: var(--dim); line-height: 1.6;
    display: flex; gap: 10px; align-items: flex-start;
  }

  /* Risk bar */
  .risk-bar {
    position: fixed; bottom: 0; left: 0; right: 0;
    background: rgba(10,10,10,0.9); backdrop-filter: blur(10px);
    padding: 10px 20px; font-size: 11px; color: var(--dim);
    display: flex; align-items: center; gap: 6px; z-index: 50;
    border-top: 1px solid var(--border);
  }

  /* ---- Improvement labels ---- */
  .imp-label {
    display: inline-flex; align-items: center; gap: 4px;
    font-size: 9px; font-weight: 700; padding: 3px 8px;
    border-radius: var(--pill); text-transform: uppercase;
    letter-spacing: 0.08em; margin-bottom: 8px;
  }
  .imp-1 { background: rgba(74,222,128,0.15); color: var(--green); }
  .imp-2 { background: var(--accent-bg); color: var(--accent); }

  /* Confirmed state */
  .order-confirmed {
    text-align: center; padding: 24px 20px;
  }
  .oc-icon { font-size: 48px; margin-bottom: 8px; }
  .oc-title { font-size: 16px; font-weight: 700; color: var(--green); }
  .oc-sub { font-size: 13px; color: var(--muted); margin-top: 4px; }
  .oc-btn {
    margin-top: 14px; padding: 10px 28px;
    background: var(--card); border: 1px solid var(--border);
    border-radius: var(--pill); color: var(--w80);
    font-size: 13px; font-weight: 500; cursor: pointer; font-family: inherit;
  }

  /* Dropdown */
  .dd-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.6); z-index: 70; display: none; }
  .dd-overlay.open { display: block; }
  .dd-panel {
    position: fixed; bottom: 0; left: 0; right: 0;
    background: var(--card); border-radius: var(--r) var(--r) 0 0;
    max-height: 70vh; overflow-y: auto; z-index: 80;
    transform: translateY(100%); transition: transform 0.3s ease;
    padding-bottom: 20px;
  }
  .dd-panel.open { transform: translateY(0); }
  .dd-search-wrap { padding: 16px; position: sticky; top: 0; background: var(--card); z-index: 1; }
  .dd-search {
    width: 100%; background: var(--elevated); border: 1px solid var(--border-light);
    border-radius: var(--rs); padding: 14px 16px 14px 42px;
    font-size: 14px; color: var(--white); outline: none; font-family: inherit;
  }
  .dd-search::placeholder { color: var(--dim); }
  .dd-search-icon { position: absolute; left: 30px; top: 50%; transform: translateY(-50%); color: var(--dim); font-size: 15px; }
  .dd-item {
    padding: 14px 20px; cursor: pointer;
    display: flex; align-items: center; justify-content: space-between;
    border-bottom: 1px solid var(--border); transition: background 0.12s;
  }
  .dd-item:hover { background: var(--card-hover); }
  .dd-item.sel { background: var(--accent-bg); }
  .dd-item-left { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
  .dd-item-name { font-size: 14px; font-weight: 600; }
  .dd-item-ticker { font-size: 11px; color: var(--dim); }
  .dd-item-type { font-size: 10px; color: var(--w60); background: var(--w10); padding: 2px 7px; border-radius: 5px; }
  .dd-item-xetra { font-size: 10px; color: var(--accent); background: var(--accent-bg); padding: 2px 7px; border-radius: 5px; }
  .dd-item-yield { font-size: 13px; font-weight: 700; color: var(--green); }

  @media (max-width: 480px) {
    .stats-grid { grid-template-columns: 1fr 1fr; }
    .est-row { flex-direction: column; gap: 10px; }
    .est-div { width: 100%; height: 1px; }
    .cmp-item { flex-direction: column; align-items: flex-start; }
  }
</style>
</head>
<body>

<div class="app" id="app">

  <!-- Close -->
  <button class="close-btn">&times;</button>

  <!-- Stock Header -->
  <div class="stock-header" onclick="openDD()" style="cursor:pointer">
    <div class="sh-name" id="shName">XACTHDIV <span class="sh-flag">🇸🇪</span></div>
    <div class="sh-price">
      <span id="shPrice">163,66 SEK</span>
      <span class="sh-change" id="shChange">+0,20% (0,32)</span>
    </div>
  </div>

  <!-- Order Book -->
  <div class="ob-labels">Köp&emsp;&emsp;&emsp;Sälj</div>
  <div class="ob-row">
    <span class="ob-vol left" id="obBidVol">1 299 st</span>
    <span class="ob-bid" id="obBid">163,54</span>
    <span class="ob-ask" id="obAsk">163,66</span>
    <span class="ob-vol right" id="obAskVol">2 871 st</span>
  </div>

  <!-- Mini Chart -->
  <div style="padding: 14px 20px 0">
    <div class="chart-placeholder" id="miniChartWrap">
      <canvas id="miniChart"></canvas>
    </div>
  </div>
  <div class="chart-dots">
    <div class="chart-dot active"></div>
    <div class="chart-dot"></div>
  </div>

  <!-- Account Bar -->
  <div class="account-bar">
    <div class="account-left">
      <span class="account-badge">ISK</span>
      <div>
        <div class="account-name">Träning <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M6 9l6 6 6-6"/></svg></div>
        <div class="account-sub">Inget investerat i fonden</div>
      </div>
    </div>
    <div class="account-balance">0 kr tillgängligt <span style="font-size:16px">›</span></div>
  </div>

  <!-- IMPROVEMENT 2: Quick Amount Buttons -->
  <div>
    <div class="quick-amounts" id="quickAmounts">
      <button class="qa-btn" onclick="setAmount(500)">500 kr</button>
      <button class="qa-btn" onclick="setAmount(1000)">1 000 kr</button>
      <button class="qa-btn" onclick="setAmount(2500)">2 500 kr</button>
      <button class="qa-btn" onclick="setAmount(5000)">5 000 kr</button>
      <button class="qa-btn" onclick="setAmount(10000)">10 000 kr</button>
      <button class="qa-btn" onclick="setAmount(25000)">25 000 kr</button>
    </div>
  </div>

  <!-- Order Inputs (Exact SAVR layout) -->
  <div class="order-inputs" id="orderInputs">
    <div class="oi-field">
      <span class="oi-label" id="amtLabel">Belopp (SEK)</span>
      <input class="oi-value" type="text" inputmode="numeric" id="amtInput" placeholder="0" oninput="onAmtInput()" onfocus="onFieldFocus(this)" onblur="onFieldBlur(this,'amt')" />
      <div class="oi-sub" id="amtSub"></div>
    </div>
    <div class="oi-field">
      <span class="oi-label" id="kursLabel">Kurs (SEK)</span>
      <input class="oi-value" type="text" inputmode="decimal" id="kursInput" oninput="onKursInput()" onfocus="onFieldFocus(this)" onblur="onFieldBlur(this,'kurs')" />
      <div class="oi-sub" id="kursSub"></div>
    </div>
    <div class="oi-field">
      <span class="oi-label">Antal</span>
      <input class="oi-value" type="text" inputmode="numeric" id="antalInput" placeholder="0" oninput="onAntalInput()" onfocus="onFieldFocus(this)" onblur="onFieldBlur(this,'antal')" />
      <div class="oi-sub" id="antalSub"></div>
    </div>
  </div>

  <!-- IMPROVEMENT 1: Dividend Preview + Target -->
  <div class="dividend-preview" id="divPreview">
    <!-- Shows current dividend estimate when antal > 0 -->
    <div id="dpEstimate" style="display:none">
      <div class="dp-row">
        <div>
          <div class="dp-label">Uppskattad årlig utdelning</div>
          <div class="dp-value" id="dpAnnual">0,00 SEK</div>
        </div>
        <div style="text-align:right">
          <div class="dp-label">Per månad</div>
          <div class="dp-value" id="dpMonthly" style="font-size:13px">0,00 SEK</div>
        </div>
      </div>
      <div class="dp-sub">
        <span class="dp-yield-badge" id="dpYield">0% direktavkastning</span>
        <span id="dpExInfo" class="dp-exdate"></span>
      </div>
    </div>

    <!-- Dividend target input — always visible -->
    <div class="dp-target" id="dpTarget">
      <div class="dp-target-row">
        <div class="dp-target-label">💰 Eller ange önskad utdelning</div>
        <div class="dp-target-inputs">
          <div class="dp-target-field">
            <input class="dp-target-input" type="number" id="dtInput" placeholder="t.ex. 500" oninput="onDTChange()" />
            <span class="dp-target-cur" id="dtCurLabel">SEK</span>
          </div>
          <div class="dp-target-period">
            <button class="dtp-btn" onclick="setDTPeriod('month')" id="dtpMonth">/ mån</button>
            <button class="dtp-btn active" onclick="setDTPeriod('year')" id="dtpYear">/ år</button>
          </div>
        </div>
      </div>
      <div class="dp-target-result" id="dtResult">
        <span id="dtResultText"></span>
        <button class="dtr-apply" onclick="applyDT()">Fyll i order →</button>
      </div>
    </div>

    <div class="dp-expand">
      <button class="dp-expand-btn" onclick="toggleAnalysis()">
        Visa fullständig analys <span id="expandArrow">↓</span>
      </button>
    </div>
  </div>

  <!-- CTA Row -->
  <div class="cta-row" id="ctaRow">
    <button class="cta-btn" id="ctaBtn" onclick="doReview()">
      <span>Granska köp</span>
      <span class="cta-total" id="ctaTotal">0,00 kr <span>›</span></span>
    </button>
    <button class="cta-more">⋮</button>
  </div>

  <!-- Full Analysis Panel -->
  <div class="analysis-panel" id="analysisPanel">
    <!-- Stats -->
    <div class="stats-grid" id="statsGrid"></div>

    <!-- Estimate -->
    <div class="estimate">
      <div class="est-title">✦ Din uppskattade utdelning</div>
      <div class="est-row">
        <div>
          <div class="est-amount" id="estYear">0,00 SEK</div>
          <div class="est-label" id="estYearL">per år</div>
        </div>
        <div class="est-div"></div>
        <div>
          <div class="est-amount" id="estMonth" style="font-size:22px">0,00 SEK</div>
          <div class="est-label">per månad (snitt)</div>
        </div>
      </div>
    </div>

    <!-- History Chart -->
    <div class="ap-section">
      <div class="ap-title">Utdelningshistorik</div>
      <div class="chart-wrap"><canvas id="divChart"></canvas></div>
    </div>

    <!-- Calendar -->
    <div class="ap-section">
      <div class="ap-title">Utdelningskalender</div>
      <div id="calList"></div>
    </div>

    <!-- Compare -->
    <div class="ap-section">
      <div class="ap-title" id="cmpTitle">Jämför med liknande</div>
      <div id="cmpList"></div>
    </div>
  </div>

  <!-- Tabs -->
  <div class="tabs-bar">
    <button class="tab-btn" onclick="switchTab(this)">Om</button>
    <button class="tab-btn active" onclick="switchTab(this)">Handel</button>
  </div>

  <!-- Footer -->
  <div class="footer-note">
    <span style="flex-shrink:0">ℹ</span>
    <span>Prototyp för SAVRs vibecoding-tävling. All data är simulerad. Investeringar innebär en risk.
    Historisk avkastning är ingen garanti för framtida avkastning. Bidrag av Magnus Ljuvhage.</span>
  </div>

  <div style="height:50px"></div>
</div>

<!-- Bottom Sheet Dropdown -->
<div class="dd-overlay" id="ddOverlay" onclick="closeDD()"></div>
<div class="dd-panel" id="ddPanel">
  <div class="dd-search-wrap">
    <span class="dd-search-icon">&#128269;</span>
    <input class="dd-search" id="ddSearch" type="text" placeholder="Sök aktie eller ETF..." oninput="filterStocks()" />
  </div>
  <div id="ddItems"></div>
</div>

<!-- Risk bar -->
<div class="risk-bar"><span>ⓘ</span> Investeringar innebär en risk</div>

<script>
// ===== DATA =====
const S = {
  "XACT Högutdelande":{name:"XACT Högutdelande",ticker:"XACTHDIV",price:163.66,type:"ETF",sector:"Utdelnings-ETF",exchange:"Stockholm",flag:"🇸🇪",
    yield:5.8,annDiv:9.20,payout:null,growth:4.1,streak:null,
    bid:163.54,ask:163.66,bidVol:"1 299",askVol:"2 871",change:"+0,20%",changeAbs:"0,32",
    exDate:"2026-06-15",payDate:"2026-06-19",
    hist:[{y:"2019",d:7.2},{y:"2020",d:5.8},{y:"2021",d:7.0},{y:"2022",d:7.8},{y:"2023",d:8.2},{y:"2024",d:8.8},{y:"2025",d:9.2}],
    cal:[{q:"Q2 2026",a:2.30,ex:"2026-06-15",s:"up"},{q:"Q1 2026",a:2.30,ex:"2026-03-16",s:"up"},{q:"Q4 2025",a:2.30,ex:"2025-12-15",s:"paid"},{q:"Q3 2025",a:2.30,ex:"2025-09-15",s:"paid"}],
    sim:[{n:"Vanguard FTSE All-World High Div.",t:"VHYL.DE",y:3.4,g:3.2},{n:"iShares STOXX Gl. Select Div. 100",t:"ISPA.DE",y:5.1,g:2.8},{n:"SPDR S&P Euro Div. Aristocrats",t:"EUDV.DE",y:3.8,g:3.5}]},
  "Investor B":{name:"Investor B",ticker:"INVE-B",price:268.40,type:"Aktie",sector:"Investmentbolag",exchange:"Stockholm",flag:"🇸🇪",
    yield:2.8,annDiv:7.50,payout:42,growth:8.2,streak:28,
    bid:268.30,ask:268.40,bidVol:"845",askVol:"1 203",change:"+1,12%",changeAbs:"2,96",
    exDate:"2026-04-08",payDate:"2026-04-14",
    hist:[{y:"2019",d:5.0},{y:"2020",d:4.0},{y:"2021",d:5.5},{y:"2022",d:6.0},{y:"2023",d:6.5},{y:"2024",d:7.0},{y:"2025",d:7.5}],
    cal:[{q:"Q1 2026",a:4.50,ex:"2026-04-08",s:"up"},{q:"Q3 2025",a:3.00,ex:"2025-09-15",s:"paid"},{q:"Q1 2025",a:4.00,ex:"2025-04-09",s:"paid"}],
    sim:[{n:"Latour B",t:"LATO-B.ST",y:1.2,g:12.1},{n:"Industrivärden C",t:"INDU-C.ST",y:3.1,g:6.5},{n:"Lundbergföretagen B",t:"LUND-B.ST",y:2.4,g:9.0}]},
  "Volvo B":{name:"Volvo B",ticker:"VOLV-B",price:285.60,type:"Aktie",sector:"Industri",exchange:"Stockholm",flag:"🇸🇪",
    yield:4.5,annDiv:13.00,payout:55,growth:11.4,streak:22,
    bid:285.50,ask:285.60,bidVol:"2 100",askVol:"3 450",change:"-0,35%",changeAbs:"-1,00",
    exDate:"2026-04-02",payDate:"2026-04-07",
    hist:[{y:"2019",d:6.5},{y:"2020",d:0},{y:"2021",d:9.0},{y:"2022",d:10.0},{y:"2023",d:11.5},{y:"2024",d:12.0},{y:"2025",d:13.0}],
    cal:[{q:"Q1 2026",a:6.50,ex:"2026-04-02",s:"up"},{q:"Q3 2025",a:6.50,ex:"2025-10-01",s:"paid"}],
    sim:[{n:"Sandvik",t:"SAND.ST",y:3.2,g:8.7},{n:"Atlas Copco A",t:"ATCO-A.ST",y:1.8,g:14.2},{n:"Epiroc A",t:"EPI-A.ST",y:2.1,g:10.5}]},
  "Handelsbanken A":{name:"Handelsbanken A",ticker:"SHB-A",price:124.80,type:"Aktie",sector:"Bank",exchange:"Stockholm",flag:"🇸🇪",
    yield:6.1,annDiv:7.60,payout:72,growth:5.8,streak:18,
    bid:124.70,ask:124.80,bidVol:"4 200",askVol:"5 100",change:"+0,48%",changeAbs:"0,60",
    exDate:"2026-03-25",payDate:"2026-03-30",
    hist:[{y:"2019",d:5.5},{y:"2020",d:3.1},{y:"2021",d:4.5},{y:"2022",d:5.5},{y:"2023",d:6.5},{y:"2024",d:7.0},{y:"2025",d:7.6}],
    cal:[{q:"Q1 2026",a:7.60,ex:"2026-03-25",s:"up"},{q:"Q1 2025",a:7.00,ex:"2025-03-26",s:"paid"}],
    sim:[{n:"Nordea",t:"NDA-SE.ST",y:7.2,g:6.3},{n:"SEB A",t:"SEB-A.ST",y:5.8,g:7.1},{n:"Swedbank A",t:"SWED-A.ST",y:6.5,g:4.9}]},
  "Vanguard FTSE All-World High Div.":{name:"Vanguard FTSE All-World High Div.",ticker:"VHYL",price:62.80,type:"ETF",sector:"Global Utdelning",exchange:"Xetra",flag:"🇩🇪",
    yield:3.4,annDiv:2.14,payout:null,growth:3.2,streak:null,
    bid:62.76,ask:62.80,bidVol:"12 500",askVol:"8 300",change:"+0,16%",changeAbs:"0,10",
    exDate:"2026-06-24",payDate:"2026-07-01",
    hist:[{y:"2019",d:1.72},{y:"2020",d:1.48},{y:"2021",d:1.65},{y:"2022",d:1.82},{y:"2023",d:1.90},{y:"2024",d:2.02},{y:"2025",d:2.14}],
    cal:[{q:"Q2 2026",a:0.54,ex:"2026-06-24",s:"up"},{q:"Q1 2026",a:0.52,ex:"2026-03-25",s:"up"},{q:"Q4 2025",a:0.55,ex:"2025-12-17",s:"paid"}],
    sim:[{n:"iShares STOXX Gl. Select Div. 100",t:"ISPA.DE",y:5.1,g:2.8},{n:"SPDR S&P Global Div. Aristocrats",t:"ZPRG.DE",y:4.3,g:3.0},{n:"Fidelity Global Quality Income",t:"FGQI.DE",y:2.8,g:4.5}]},
  "iShares STOXX Global Select Div. 100":{name:"iShares STOXX Global Select Div. 100",ticker:"ISPA",price:32.45,type:"ETF",sector:"Global Utdelning",exchange:"Xetra",flag:"🇩🇪",
    yield:5.1,annDiv:1.65,payout:null,growth:2.8,streak:null,
    bid:32.42,ask:32.45,bidVol:"6 800",askVol:"4 200",change:"-0,09%",changeAbs:"-0,03",
    exDate:"2026-07-15",payDate:"2026-07-22",
    hist:[{y:"2019",d:1.38},{y:"2020",d:1.05},{y:"2021",d:1.22},{y:"2022",d:1.35},{y:"2023",d:1.45},{y:"2024",d:1.55},{y:"2025",d:1.65}],
    cal:[{q:"Q2 2026",a:0.85,ex:"2026-07-15",s:"up"},{q:"Q4 2025",a:0.80,ex:"2025-12-10",s:"paid"}],
    sim:[{n:"Vanguard FTSE All-World High Div.",t:"VHYL.DE",y:3.4,g:3.2},{n:"SPDR S&P Euro Div. Aristocrats",t:"EUDV.DE",y:3.8,g:3.5},{n:"WisdomTree Gl. Quality Div.",t:"GGRP.DE",y:2.2,g:6.1}]},
  "SPDR S&P Euro Div. Aristocrats":{name:"SPDR S&P Euro Div. Aristocrats",ticker:"EUDV",price:24.10,type:"ETF",sector:"Europa Utdelning",exchange:"Xetra",flag:"🇩🇪",
    yield:3.8,annDiv:0.92,payout:null,growth:3.5,streak:null,
    bid:24.08,ask:24.10,bidVol:"9 100",askVol:"7 500",change:"+0,33%",changeAbs:"0,08",
    exDate:"2026-05-20",payDate:"2026-05-27",
    hist:[{y:"2019",d:0.68},{y:"2020",d:0.52},{y:"2021",d:0.65},{y:"2022",d:0.72},{y:"2023",d:0.78},{y:"2024",d:0.85},{y:"2025",d:0.92}],
    cal:[{q:"Q2 2026",a:0.46,ex:"2026-05-20",s:"up"},{q:"Q4 2025",a:0.46,ex:"2025-11-19",s:"paid"}],
    sim:[{n:"iShares EURO STOXX Select Div. 30",t:"EXSH.DE",y:4.6,g:2.9},{n:"Vanguard FTSE All-World High Div.",t:"VHYL.DE",y:3.4,g:3.2},{n:"iShares STOXX Gl. Select Div. 100",t:"ISPA.DE",y:5.1,g:2.8}]},
  "SPDR S&P Global Div. Aristocrats":{name:"SPDR S&P Global Div. Aristocrats",ticker:"ZPRG",price:30.85,type:"ETF",sector:"Global Utdelning",exchange:"Xetra",flag:"🇩🇪",
    yield:4.3,annDiv:1.33,payout:null,growth:3.0,streak:null,
    bid:30.82,ask:30.85,bidVol:"5 600",askVol:"3 900",change:"+0,23%",changeAbs:"0,07",
    exDate:"2026-05-14",payDate:"2026-05-21",
    hist:[{y:"2019",d:1.05},{y:"2020",d:0.82},{y:"2021",d:0.98},{y:"2022",d:1.08},{y:"2023",d:1.15},{y:"2024",d:1.25},{y:"2025",d:1.33}],
    cal:[{q:"Q2 2026",a:0.67,ex:"2026-05-14",s:"up"},{q:"Q4 2025",a:0.66,ex:"2025-11-12",s:"paid"}],
    sim:[{n:"Vanguard FTSE All-World High Div.",t:"VHYL.DE",y:3.4,g:3.2},{n:"iShares STOXX Gl. Select Div. 100",t:"ISPA.DE",y:5.1,g:2.8},{n:"Fidelity Global Quality Income",t:"FGQI.DE",y:2.8,g:4.5}]},
};

// ===== STATE =====
let sel = "XACT Högutdelande";
let analysisOpen = false;
let chart = null, miniC = null;

// ===== HELPERS =====
const EUR_SEK = 11.28; // Approximativt EUR/SEK-kurs
const f = n => n.toLocaleString("sv-SE",{minimumFractionDigits:2,maximumFractionDigits:2});
const fInt = n => n.toLocaleString("sv-SE",{maximumFractionDigits:0});
const isEUR = s => s.exchange === "Xetra";
const toSEK = (v, s) => isEUR(s) ? v * EUR_SEK : v;
// All UI now shows SEK — EUR instruments are converted at EUR_SEK rate
const daysTo = d => Math.ceil((new Date(d)-new Date("2026-03-17"))/864e5);

// ===== STOCK SELECTION =====
function loadStock() {
  const s = S[sel];
  const priceSEK = toSEK(s.price, s), askSEK = toSEK(s.ask, s), bidSEK = toSEK(s.bid, s);
  document.getElementById("shName").innerHTML = s.ticker + ' <span class="sh-flag">' + s.flag + '</span>';
  document.getElementById("shPrice").textContent = f(priceSEK) + " SEK";
  if (isEUR(s)) document.getElementById("shPrice").textContent += " (" + f(s.price) + " EUR)";
  const ch = document.getElementById("shChange");
  ch.textContent = s.change + " (" + s.changeAbs + ")";
  ch.className = "sh-change" + (s.changeAbs.startsWith("-") ? " neg" : "");
  document.getElementById("obBid").textContent = f(bidSEK);
  document.getElementById("obAsk").textContent = f(askSEK);
  document.getElementById("obBidVol").textContent = s.bidVol + " st";
  document.getElementById("obAskVol").textContent = s.askVol + " st";
  document.getElementById("kursInput").value = fmtKurs(askSEK);
  document.getElementById("amtLabel").textContent = "Belopp (SEK)";
  document.getElementById("kursLabel").textContent = "Kurs (SEK)";
  document.getElementById("amtInput").value = "";
  document.getElementById("antalInput").value = "";
  document.getElementById("amtSub").textContent = "";
  document.getElementById("kursSub").textContent = "";
  document.getElementById("antalSub").textContent = "";
  document.querySelectorAll(".qa-btn").forEach(b => b.classList.remove("active"));
  // Reset dividend target
  document.getElementById("dtInput").value = "";
  document.getElementById("dtResult").classList.remove("show");
  document.getElementById("dtCurLabel").textContent = "SEK";
  dtCalcN = 0;
  updateDivPreview();
  updateCTA();
  updateFieldSubs();
  renderMiniChart(s);
  if (analysisOpen) renderAnalysis();
}

// ===== DIVIDEND TARGET (Improvement 3) =====
let dtPeriod = "year"; // "month" or "year"
let dtCalcN = 0; // calculated antal

function setDTPeriod(p) {
  dtPeriod = p;
  document.getElementById("dtpMonth").classList.toggle("active", p === "month");
  document.getElementById("dtpYear").classList.toggle("active", p === "year");
  onDTChange();
}
function onDTChange() {
  const s = S[sel];
  const raw = parseFloat(document.getElementById("dtInput").value) || 0;
  const res = document.getElementById("dtResult");
  if (raw <= 0) { res.classList.remove("show"); dtCalcN = 0; return; }

  // User inputs SEK target — convert dividend per share to SEK
  const annDivSEK = toSEK(s.annDiv, s);
  const annualTarget = dtPeriod === "month" ? raw * 12 : raw;
  const sharesNeeded = Math.ceil(annualTarget / annDivSEK);
  const totalCostSEK = sharesNeeded * toSEK(s.ask, s);
  const actualAnnualSEK = sharesNeeded * annDivSEK;
  const actualMonthlySEK = actualAnnualSEK / 12;
  dtCalcN = sharesNeeded;

  document.getElementById("dtResultText").innerHTML =
    `Du behöver <strong>${fInt(sharesNeeded)} andelar</strong> (${f(totalCostSEK)} SEK) → ${f(actualAnnualSEK)} SEK/år ≈ ${f(actualMonthlySEK)} SEK/mån`;
  res.classList.add("show");
}
function applyDT() {
  if (dtCalcN <= 0) return;
  const s = S[sel], askSEK = toSEK(s.ask, s);
  document.getElementById("antalInput").value = fmtAntal(dtCalcN);
  document.getElementById("amtInput").value = fmtAmt(Math.round(dtCalcN * askSEK));
  updateAll();
  if (analysisOpen) renderAnalysis();
  document.getElementById("orderInputs").scrollIntoView({ behavior: "smooth", block: "center" });
}
// ===== NUMBER FORMATTING HELPERS =====
// Parse a formatted string like "10 000" or "268,40" back to a number
function parseNum(str) {
  if (!str) return 0;
  return parseFloat(String(str).replace(/\s/g, "").replace(",", ".")) || 0;
}
// Format for display in input fields (when not focused)
function fmtAmt(n) { return n > 0 ? fInt(Math.round(n)) : ""; }
function fmtKurs(n) { return n > 0 ? f(n) : ""; }
function fmtAntal(n) { return n > 0 ? fInt(n) : ""; }

// On focus: show raw number for easy editing
function onFieldFocus(el) {
  const raw = parseNum(el.value);
  if (raw > 0) el.value = raw;
}
// On blur: format the value nicely
function onFieldBlur(el, type) {
  const raw = parseNum(el.value);
  if (type === "amt") el.value = fmtAmt(raw);
  else if (type === "kurs") el.value = fmtKurs(raw);
  else if (type === "antal") el.value = fmtAntal(raw);
  updateFieldSubs();
}

// ===== QUICK AMOUNTS =====
function setAmount(v) {
  const s = S[sel], askSEK = toSEK(s.ask, s);
  const antal = Math.floor(v / askSEK);
  document.getElementById("amtInput").value = fmtAmt(v);
  document.getElementById("antalInput").value = fmtAntal(antal);
  document.querySelectorAll(".qa-btn").forEach(b => {
    b.classList.toggle("active", parseInt(b.textContent.replace(/\D/g,"")) === v);
  });
  updateAll();
}

// ===== INPUT HANDLERS =====
function onAmtInput() {
  const s = S[sel], askSEK = toSEK(s.ask, s);
  const amt = parseNum(document.getElementById("amtInput").value);
  document.getElementById("antalInput").value = amt > 0 ? fmtAntal(Math.floor(amt / askSEK)) : "";
  updateAll();
}
function onKursInput() { updateAll(); }
function onAntalInput() {
  const s = S[sel], askSEK = toSEK(s.ask, s);
  const n = parseNum(document.getElementById("antalInput").value);
  document.getElementById("amtInput").value = n > 0 ? fmtAmt(Math.round(n * askSEK)) : "";
  updateAll();
}

function updateAll() {
  updateDivPreview(); updateCTA(); updateFieldSubs();
}

// ===== FIELD SUB-INFO =====
function updateFieldSubs() {
  const s = S[sel];
  const n = parseNum(document.getElementById("antalInput").value);
  const askSEK = toSEK(s.ask, s);
  const totalSEK = n * askSEK;

  // Belopp sub: show total if antal > 0
  const amtSub = document.getElementById("amtSub");
  if (n > 0 && isEUR(s)) {
    amtSub.textContent = f(n * s.ask) + " EUR";
    amtSub.className = "oi-sub eur";
  } else {
    amtSub.textContent = "";
  }

  // Kurs sub: show EUR original price
  const kursSub = document.getElementById("kursSub");
  if (isEUR(s)) {
    kursSub.textContent = f(s.ask) + " EUR";
    kursSub.className = "oi-sub eur";
  } else {
    kursSub.textContent = "";
  }

  // Antal sub: show total cost
  const antalSub = document.getElementById("antalSub");
  if (n > 0) {
    antalSub.textContent = "= " + f(totalSEK) + " SEK";
    antalSub.className = "oi-sub";
  } else {
    antalSub.textContent = "";
  }
}

// ===== DIVIDEND PREVIEW (Improvement 1) =====
function updateDivPreview() {
  const s = S[sel], n = parseNum(document.getElementById("antalInput").value);
  // Always show the preview box (dividend target input lives here)
  document.getElementById("divPreview").style.display = "block";
  const estEl = document.getElementById("dpEstimate");
  if (n > 0) {
    estEl.style.display = "block";
    const annualSEK = toSEK(s.annDiv, s) * n, monthlySEK = annualSEK / 12;
    document.getElementById("dpAnnual").textContent = f(annualSEK) + " SEK";
    document.getElementById("dpMonthly").textContent = f(monthlySEK) + " SEK";
    document.getElementById("dpYield").textContent = s.yield.toFixed(1) + "% direktavkastning";
    const d = daysTo(s.exDate);
    const exEl = document.getElementById("dpExInfo");
    if (d > 0 && d <= 45) {
      exEl.innerHTML = "⚡ Ex-dag om " + d + " dagar";
      exEl.style.display = "";
    } else { exEl.style.display = "none"; }
  } else {
    estEl.style.display = "none";
  }
}

// ===== CTA =====
function updateCTA() {
  const s = S[sel], n = parseNum(document.getElementById("antalInput").value);
  const totalSEK = n * toSEK(s.ask, s);
  document.getElementById("ctaTotal").innerHTML = f(totalSEK) + " sek <span>›</span>";
}
function doReview() {
  const s = S[sel], n = parseNum(document.getElementById("antalInput").value);
  if (n === 0) return;
  const askSEK = toSEK(s.ask, s), totalSEK = n * askSEK;
  document.getElementById("ctaRow").innerHTML = `
    <div class="order-confirmed">
      <div class="oc-icon">✓</div>
      <div class="oc-title">Order skickad (demo)</div>
      <div class="oc-sub">${n} st ${s.ticker} till ${f(askSEK)} SEK = ${f(totalSEK)} SEK</div>
      <button class="oc-btn" onclick="resetCTA()">Ny order</button>
    </div>`;
}
function resetCTA() {
  document.getElementById("ctaRow").innerHTML = `
    <button class="cta-btn" id="ctaBtn" onclick="doReview()">
      <span>Granska köp</span>
      <span class="cta-total" id="ctaTotal">0,00 kr <span>›</span></span>
    </button>
    <button class="cta-more">⋮</button>`;
  updateCTA();
}

// ===== ANALYSIS PANEL =====
function toggleAnalysis() {
  analysisOpen = !analysisOpen;
  document.getElementById("analysisPanel").classList.toggle("open", analysisOpen);
  document.getElementById("expandArrow").textContent = analysisOpen ? "↑" : "↓";
  if (analysisOpen) renderAnalysis();
}
function renderAnalysis() {
  const s = S[sel], n = parseNum(document.getElementById("antalInput").value);
  const annDivSEK = toSEK(s.annDiv, s);
  const ea = annDivSEK * n, em = ea / 12;

  // Stats
  let h = `<div class="stat hl"><div class="stat-lbl">Direktavkastning</div><div class="stat-val">${s.yield.toFixed(1)}%</div><div class="stat-sub">${f(annDivSEK)} SEK/aktie/år</div></div>
  <div class="stat"><div class="stat-lbl">Tillväxt 5 år</div><div class="stat-val">+${s.growth}%</div>${s.streak?`<div class="stat-sub">${s.streak} år i rad</div>`:''}</div>
  <div class="stat"><div class="stat-lbl">Nästa ex-dag</div><div class="stat-val" style="font-size:15px">${s.exDate}</div><div class="stat-sub">Utbetalning: ${s.payDate}</div></div>`;
  if (s.payout) h += `<div class="stat"><div class="stat-lbl">Payout Ratio</div><div class="stat-val">${s.payout}%</div><div class="stat-sub">${s.payout<60?'Hållbar':'Hög'} nivå</div></div>`;
  else h += `<div class="stat"><div class="stat-lbl">Sektor</div><div class="stat-val" style="font-size:14px">${s.sector}</div></div>`;
  document.getElementById("statsGrid").innerHTML = h;

  // Estimate
  document.getElementById("estYear").textContent = f(ea) + " SEK";
  document.getElementById("estYearL").textContent = `per år (${n} andelar)`;
  document.getElementById("estMonth").textContent = f(em) + " SEK";

  renderDivChart(s);

  // Calendar
  document.getElementById("calList").innerHTML = s.cal.map(x => {
    const aSEK = toSEK(x.a, s);
    return `<div class="cal-item ${x.s==='up'?'up':''}">
      <div class="cal-left"><div class="cal-dot ${x.s==='up'?'up':''}"></div><span class="cal-q">${x.q}</span><span class="cal-ex">${x.ex}</span></div>
      <div class="cal-right"><span class="cal-amt">${f(aSEK)} SEK</span><span class="sbadge ${x.s==='up'?'sb-up':'sb-paid'}">${x.s==='up'?'Kommande':'Utbetald'}</span></div>
    </div>`;
  }).join("");

  // Compare
  document.getElementById("cmpTitle").textContent = "Jämför med liknande " + (s.type==="ETF"?"ETF:er":"aktier");
  let cm = `<div class="cmp-item me"><div><span class="cmp-name">${s.name}</span><span class="cmp-tag">Ditt val</span></div>
    <div class="cmp-metrics"><span class="cmp-m">${s.yield.toFixed(1)}%</span><span class="cmp-m">+${s.growth}%</span></div></div>`;
  s.sim.forEach(x => {
    const yb = x.y > s.yield, gb = x.g > s.growth;
    cm += `<div class="cmp-item"><div><span class="cmp-name">${x.n}</span><span class="cmp-ticker">${x.t}</span></div>
      <div class="cmp-metrics"><span class="cmp-m ${yb?'better':''}"><strong>${x.y.toFixed(1)}%</strong>${yb?' ↑':''}</span>
      <span class="cmp-m ${gb?'better':''}"><strong>+${x.g}%</strong>${gb?' ↑':''}</span></div></div>`;
  });
  document.getElementById("cmpList").innerHTML = cm;
}

// ===== CHARTS =====
function renderMiniChart(s) {
  if (miniC) miniC.destroy();
  const ctx = document.getElementById("miniChart").getContext("2d");
  // Simulate price line
  const pts = []; let p = s.price * 0.92;
  for (let i = 0; i < 30; i++) { p += (Math.random()-0.48)*s.price*0.008; pts.push(p); }
  pts.push(s.price);
  miniC = new Chart(ctx, { type:"line", data:{labels:pts.map((_,i)=>i),datasets:[{data:pts,
    borderColor:"rgba(192,132,252,0.6)",borderWidth:1.5,fill:true,
    backgroundColor:ctx.createLinearGradient?(() => {const g=ctx.createLinearGradient(0,0,0,200);g.addColorStop(0,"rgba(192,132,252,0.12)");g.addColorStop(1,"rgba(192,132,252,0)");return g;})():"rgba(192,132,252,0.05)",
    pointRadius:0,tension:0.4}]},
    options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},tooltip:{enabled:false}},
    scales:{x:{display:false},y:{display:false}}} });
}
function renderDivChart(s) {
  if (chart) chart.destroy();
  const ctx = document.getElementById("divChart").getContext("2d");
  const dataSEK = s.hist.map(h => toSEK(h.d, s));
  chart = new Chart(ctx, { type:"bar", data:{labels:s.hist.map(h=>h.y),datasets:[{data:dataSEK,
    backgroundColor:s.hist.map((_,i)=>i===s.hist.length-1?"rgba(192,132,252,0.85)":"rgba(192,132,252,0.25)"),
    borderRadius:5,borderSkipped:false,barPercentage:0.55}]},
    options:{responsive:true,maintainAspectRatio:false,
    plugins:{legend:{display:false},tooltip:{backgroundColor:"#1B1B1B",borderColor:"rgba(255,255,255,0.1)",borderWidth:1,
      titleColor:"#fff",bodyColor:"#C084FC",cornerRadius:10,padding:12,
      callbacks:{label:i=>f(i.parsed.y)+' SEK'}}},
    scales:{x:{grid:{display:false},ticks:{color:"#555",font:{family:"Inter",size:11}},border:{display:false}},
      y:{grid:{color:"rgba(255,255,255,0.04)"},ticks:{color:"#555",font:{family:"Inter",size:11}},border:{display:false}}}} });
}

// ===== DROPDOWN =====
function openDD() {
  document.getElementById("ddOverlay").classList.add("open");
  document.getElementById("ddPanel").classList.add("open");
  document.getElementById("ddSearch").value = "";
  renderDDItems(Object.keys(S));
  setTimeout(() => document.getElementById("ddSearch").focus(), 100);
}
function closeDD() {
  document.getElementById("ddOverlay").classList.remove("open");
  document.getElementById("ddPanel").classList.remove("open");
}
function filterStocks() {
  const q = document.getElementById("ddSearch").value.toLowerCase();
  renderDDItems(Object.keys(S).filter(k => k.toLowerCase().includes(q) || S[k].ticker.toLowerCase().includes(q)));
}
function renderDDItems(keys) {
  document.getElementById("ddItems").innerHTML = keys.map(k => {
    const s = S[k];
    return `<div class="dd-item ${k===sel?'sel':''}" onclick="pickStock(this)" data-key="${k}">
      <div class="dd-item-left">
        <span class="dd-item-name">${s.name}</span>
        <span class="dd-item-ticker">${s.ticker}</span>
        <span class="dd-item-type">${s.type}</span>
        ${s.exchange==="Xetra"?'<span class="dd-item-xetra">Xetra</span>':''}
      </div>
      <span class="dd-item-yield">${s.yield.toFixed(1)}%</span>
    </div>`;
  }).join("");
}
function pickStock(el) {
  sel = el.dataset.key;
  closeDD();
  analysisOpen = false;
  document.getElementById("analysisPanel").classList.remove("open");
  document.getElementById("expandArrow").textContent = "↓";
  loadStock();
}
function switchTab(el) {
  document.querySelectorAll(".tab-btn").forEach(t => t.classList.remove("active"));
  el.classList.add("active");
}

// ===== INIT =====
loadStock();
</script>
</body>
</html>
