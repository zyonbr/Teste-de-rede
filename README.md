<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr Elite V10</title>
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

        .bento-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin-bottom: 15px; }
        .card {
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 24px;
            padding: 18px;
            transition: 0.3s;
        }
        .card.full { grid-column: span 2; display: flex; justify-content: space-around; text-align: center; }
        .label { font-size: 0.55rem; color: var(--text-dim); font-weight: 700; text-transform: uppercase; margin-bottom: 6px; display: block; }
        .value { font-size: 1.5rem; font-weight: 800; }
        .value small { font-size: 0.7rem; color: var(--neon); }

        /* Módulo de Dispositivos Conectados */
        .device-card {
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 28px;
            padding: 20px;
            margin-bottom: 15px;
        }
        .device-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
        .device-count { background: var(--neon); color: #000; font-size: 0.6rem; font-weight: 900; padding: 2px 8px; border-radius: 5px; }
        
        .device-list { display: flex; flex-direction: column; gap: 10px; }
        .device-item { 
            display: flex; 
            align-items: center; 
            gap: 12px; 
            padding: 10px; 
            background: rgba(255,255,255,0.02); 
            border-radius: 15px;
            font-size: 0.75rem;
        }
        .dev-icon { font-size: 1.2rem; }
        .dev-info { display: flex; flex-direction: column; }
        .dev-name { font-weight: 700; color: #eee; }
        .dev-ip { color: var(--text-dim); font-size: 0.65rem; }

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

        .chart-box { background: var(--surface); border: 1px solid var(--border); border-radius: 28px; padding: 15px; height: 130px; margin-bottom: 15px; }
    </style>
</head>
<body>

<div class="container">
    <div class="system-bar">
        <span>Network ID: McTelecom_GPON</span>
        <span id="sys-time">22:45:00</span>
    </div>

    <div class="header">
        <div class="brand">
            <b>NETSCAN ULTRA</b>
            <span>Zyonbr Edition</span>
        </div>
        <div style="text-align: right">
            <div style="font-size: 0.75rem; font-weight: 800; color: var(--ok)">ONLINE</div>
        </div>
    </div>

    <div class="bento-grid">
        <div class="card full">
            <div><span class="label">Download</span><div class="value" id="dl">0.0<small>Mb</small></div></div>
            <div style="width: 1px; height: 30px; background: var(--border);"></div>
            <div><span class="label">Upload</span><div class="value" id="ul">0.0<small>Mb</small></div></div>
        </div>
        <div class="card"><span class="label">Ping</span><div class="value" id="ping">--<small>ms</small></div></div>
        <div class="card"><span class="label">Jitter</span><div class="value" id="jitter">--<small>ms</small></div></div>
    </div>

    <div class="device-card">
        <div class="device-header">
            <span class="label" style="margin:0">Dispositivos na Rede</span>
            <span class="device-count" id="dev-count">4</span>
        </div>
        <div class="device-list" id="device-list">
            <div class="device-item">
                <span class="dev-icon">📱</span>
                <div class="dev-info"><span class="dev-name">iPhone de Zyon</span><span class="dev-ip">192.168.1.15</span></div>
            </div>
            <div class="device-item">
                <span class="dev-icon">💻</span>
                <div class="dev-info"><span class="dev-name">Desktop-Workstation</span><span class="dev-ip">192.168.1.10</span></div>
            </div>
            <div class="device-item">
                <span class="dev-icon">📺</span>
                <div class="dev-info"><span class="dev-name">Smart TV 4K</span><span class="dev-ip">192.168.1.22</span></div>
            </div>
            <div class="device-item">
                <span class="dev-icon">🌐</span>
                <div class="dev-info"><span class="dev-name">Gateway Mc Telecom</span><span class="dev-ip">192.168.1.1</span></div>
            </div>
        </div>
    </div>

    <div class="chart-box"><canvas id="ultraChart"></canvas></div>

    <button id="btn-main" onclick="runEliteAudit()">Executar Auditoria Profissional</button>
</div>

<script>
let chart;
let historyData = [];

function initChart() {
    const ctx = document.getElementById('ultraChart').getContext('2d');
    if(chart) chart.destroy();
    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: historyData.map(h => h.time),
            datasets: [{
                data: historyData.map(h => h.dl),
                borderColor: '#00f2ff',
                borderWidth: 3,
                tension: 0.4,
                pointRadius: 0,
                fill: true,
                backgroundColor: 'rgba(0, 242, 255, 0.05)'
            }]
        },
        options: { 
            responsive: true, maintainAspectRatio: false, plugins: { legend: false },
            scales: { y: { display: false }, x: { display: false } }
        }
    });
}

async function runEliteAudit() {
    const btn = document.getElementById('btn-main');
    btn.disabled = true;
    btn.innerText = "Sincronizando IPs...";

    // Simulação de scan de rede
    await new Promise(r => setTimeout(r, 2000));

    const res = {
        time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit', second:'2-digit'}),
        dl: (348 + Math.random() * 52).toFixed(1),
        ul: (312 + Math.random() * 38).toFixed(1),
        ping: (9 + Math.floor(Math.random() * 5)),
        jitter: (1 + Math.floor(Math.random() * 2))
    };

    document.getElementById('dl').innerHTML = `${Math.round(res.dl)}<small>Mb</small>`;
    document.getElementById('ul').innerHTML = `${Math.round(res.ul)}<small>Mb</small>`;
    document.getElementById('ping').innerHTML = `${res.ping}<small>ms</small>`;
    document.getElementById('jitter').innerHTML = `${res.jitter}<small>ms</small>`;

    historyData.push(res);
    if(historyData.length > 10) historyData.shift();
    
    initChart();
    btn.disabled = false;
    btn.innerText = "Executar Auditoria Profissional";
}

// Relógio do Sistema
setInterval(() => {
    document.getElementById('sys-time').innerText = new Date().toLocaleTimeString();
}, 1000);

initChart();
</script>
</body>
</html>
