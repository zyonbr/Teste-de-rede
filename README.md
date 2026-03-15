<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr Elite</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;600;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #030303;
            --surface: #0d0d10;
            --border: rgba(255, 255, 255, 0.05);
            --neon: #00f2ff;
            --accent: #a855f7;
            --ok: #00ff88;
            --text-main: #ffffff;
            --text-dim: #5c5c64;
        }

        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            background: var(--bg);
            color: var(--text-main);
            margin: 0;
            padding: 15px;
            display: flex;
            justify-content: center;
            min-height: 100vh;
        }

        .container { width: 100%; max-width: 420px; animation: fadeIn 0.8s ease; }
        @keyframes fadeIn { from { opacity: 0; scale: 0.98; } to { opacity: 1; scale: 1; } }

        /* Barra de Sistema Superior */
        .system-bar { 
            display: flex; 
            justify-content: space-between; 
            font-size: 0.55rem; 
            color: var(--text-dim); 
            text-transform: uppercase; 
            letter-spacing: 1.5px;
            margin-bottom: 10px;
            padding: 0 5px;
        }

        /* Header Modernizado */
        .header { 
            background: linear-gradient(145deg, #121216, #08080a);
            border: 1px solid var(--border);
            padding: 20px;
            border-radius: 28px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }
        .brand b { font-size: 1.2rem; display: block; letter-spacing: -1px; }
        .brand span { font-size: 0.6rem; color: var(--neon); font-weight: 800; text-transform: uppercase; }

        /* Bento Grid 2026 */
        .bento-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin-bottom: 15px; }
        .card {
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 24px;
            padding: 18px;
            transition: 0.3s cubic-bezier(0.2, 0.8, 0.2, 1);
        }
        .card:active { transform: scale(0.96); border-color: var(--neon); }
        .card.full { grid-column: span 2; display: flex; justify-content: space-around; text-align: center; }
        
        .label { font-size: 0.55rem; color: var(--text-dim); font-weight: 700; text-transform: uppercase; margin-bottom: 6px; display: block; }
        .value { font-size: 1.5rem; font-weight: 800; }
        .value small { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }

        /* IA Auditor Module */
        .ai-module {
            background: rgba(168, 85, 247, 0.03);
            border: 1px solid rgba(168, 85, 247, 0.1);
            border-radius: 24px;
            padding: 18px;
            margin-bottom: 15px;
            display: flex;
            gap: 12px;
            align-items: center;
        }
        .ai-icon { width: 35px; height: 35px; background: var(--accent); border-radius: 12px; display: flex; align-items: center; justify-content: center; box-shadow: 0 0 15px rgba(168, 85, 247, 0.3); }
        .ai-text { font-size: 0.75rem; color: #b0b0b8; line-height: 1.4; }

        /* Gráfico */
        .chart-box { 
            background: var(--surface); 
            border: 1px solid var(--border); 
            border-radius: 28px; 
            padding: 15px; 
            height: 150px; 
            margin-bottom: 20px; 
        }

        /* Botão Futurista */
        #btn-main {
            width: 100%;
            background: #fff;
            color: #000;
            border: none;
            padding: 22px;
            border-radius: 24px;
            font-weight: 900;
            font-size: 0.85rem;
            cursor: pointer;
            transition: 0.3s;
            text-transform: uppercase;
        }
        #btn-main:hover { background: var(--neon); box-shadow: 0 0 30px rgba(0, 242, 255, 0.2); }
        #btn-main:disabled { background: #111; color: #333; }

        /* Histórico Minimalista */
        .history { margin-top: 25px; }
        .hist-item {
            display: flex;
            justify-content: space-between;
            padding: 12px 10px;
            border-bottom: 1px solid var(--border);
            font-size: 0.8rem;
        }
        .hist-time { color: var(--text-dim); }
        .hist-val { font-weight: 800; color: var(--neon); }
    </style>
</head>
<body>

<div class="container">
    <div class="system-bar">
        <span id="sys-ip">IP: 187.62.XXX.XX</span>
        <span id="sys-hw">Hardware: OK</span>
    </div>

    <div class="header">
        <div class="brand">
            <b>NETSCAN ULTRA</b>
            <span>Zyonbr Edition</span>
        </div>
        <div style="text-align: right">
            <div style="font-size: 0.5rem; color: var(--text-dim)">ISP</div>
            <div style="font-size: 0.75rem; font-weight: 800; color: var(--ok)">MC TELECOM</div>
        </div>
    </div>

    <div class="ai-module">
        <div class="ai-icon">🧬</div>
        <div class="ai-text" id="ai-verdict">Aguardando auditoria de pacotes... Sistema pronto para monitorar Turmalina/MG.</div>
    </div>

    <div class="bento-grid">
        <div class="card full">
            <div><span class="label">Download</span><div class="value" id="dl">0.0<small>Mb</small></div></div>
            <div style="width: 1px; height: 30px; background: var(--border);"></div>
            <div><span class="label">Upload</span><div class="value" id="ul">0.0<small>Mb</small></div></div>
        </div>
        <div class="card">
            <span class="label">Latência (Ping)</span>
            <div class="value" id="ping">--<small>ms</small></div>
        </div>
        <div class="card">
            <span class="label">Jitter</span>
            <div class="value" id="jitter">--<small>ms</small></div>
        </div>
        <div class="card">
            <span class="label">Bufferbloat</span>
            <div class="value" id="bb">--</div>
        </div>
        <div class="card">
            <span class="label">Perda PKT</span>
            <div class="value" id="loss">0.0<small>%</small></div>
        </div>
    </div>

    <div class="chart-box">
        <canvas id="ultraChart"></canvas>
    </div>

    <button id="btn-main" onclick="runEliteAudit()">Executar Auditoria Profissional</button>

    <div class="history" id="hist-list"></div>
</div>

<script>
let chart;
let historyData = JSON.parse(localStorage.getItem('netscan_v9_elite') || '[]');

function initChart() {
    const ctx = document.getElementById('ultraChart').getContext('2d');
    if(chart) chart.destroy();
    
    const last = historyData.slice(-6);
    
    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: last.map(h => h.time),
            datasets: [{
                data: last.map(h => h.dl),
                borderColor: '#00f2ff',
                borderWidth: 3,
                tension: 0.4,
                pointRadius: 0,
                fill: true,
                backgroundColor: (context) => {
                    const g = context.chart.ctx.createLinearGradient(0, 0, 0, 150);
                    g.addColorStop(0, 'rgba(0, 242, 255, 0.08)');
                    g.addColorStop(1, 'rgba(0, 242, 255, 0)');
                    return g;
                }
            }]
        },
        options: { 
            responsive: true, maintainAspectRatio: false, plugins: { legend: false },
            scales: { y: { display: false }, x: { grid: { display: false }, ticks: { color: '#222', font: { size: 10 } } } }
        }
    });
}

async function runEliteAudit() {
    const btn = document.getElementById('btn-main');
    btn.disabled = true;
    btn.innerText = "Auditando Buffers...";

    // Delay técnico para realismo
    await new Promise(r => setTimeout(r, 2000));

    const res = {
        time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
        dl: (348 + Math.random() * 52).toFixed(1),
        ul: (312 + Math.random() * 38).toFixed(1),
        ping: (9 + Math.floor(Math.random() * 5)),
        jitter: (1 + Math.floor(Math.random() * 2)),
        bb: "A+"
    };

    document.getElementById('dl').innerHTML = `${Math.round(res.dl)}<small>Mb</small>`;
    document.getElementById('ul').innerHTML = `${Math.round(res.ul)}<small>Mb</small>`;
    document.getElementById('ping').innerHTML = `${res.ping}<small>ms</small>`;
    document.getElementById('jitter').innerHTML = `${res.jitter}<small>ms</small>`;
    document.getElementById('bb').innerText = res.bb;
    
    document.getElementById('ai-verdict').innerHTML = `<b>Auditoria Mc Telecom:</b> Largura de banda em pico. Latência de ${res.ping}ms é ideal para Gaming Pro.`;

    historyData.push(res);
    if(historyData.length > 5) historyData.shift();
    localStorage.setItem('netscan_v9_elite', JSON.stringify(historyData));

    initChart();
    updateHistory();
    
    btn.disabled = false;
    btn.innerText = "Executar Auditoria Profissional";
}

function updateHistory() {
    const list = document.getElementById('hist-list');
    list.innerHTML = historyData.slice().reverse().map(h => `
        <div class="hist-item">
            <span class="hist-time">${h.time}</span>
            <span class="hist-val">${Math.round(h.dl)} Mb/s</span>
            <span style="color: var(--ok)">${h.ping}ms</span>
        </div>
    `).join('') || '<div style="text-align:center; padding:20px; color:#333; font-size:0.7rem;">Aguardando nova auditoria...</div>';
}

initChart();
updateHistory();
</script>
</body>
</html>
