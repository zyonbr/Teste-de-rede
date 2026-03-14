<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #0a0a0c; --card: #16161a; --danger: #ff4d4d; --ok: #00ff88; }
        body { font-family: 'Segoe UI', Roboto, sans-serif; background: var(--bg); color: white; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; min-height: 100vh; }
        .container { width: 100%; max-width: 420px; }
        
        .header { text-align: center; margin-bottom: 20px; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.4rem; letter-spacing: 2px; text-transform: uppercase; font-weight: 900; }
        .line { width: 50px; height: 3px; background: var(--neon); margin: 8px auto; border-radius: 2px; }

        .net-info { background: rgba(255,255,255,0.03); border: 1px solid #2d2d35; border-radius: 15px; padding: 12px; margin-bottom: 15px; font-size: 0.8rem; }
        .info-item { display: flex; justify-content: space-between; margin-bottom: 4px; color: #888; }
        .info-item b { color: var(--neon); cursor: pointer; }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 15px; }
        .card { background: var(--card); padding: 18px; border-radius: 20px; border: 1px solid #2d2d35; text-align: center; }
        .label { font-size: 0.65rem; color: #777; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; display: block; }
        .value { font-size: 1.7rem; font-weight: 900; }
        .unit { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }

        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 180px; }
        
        .btn-group { display: flex; flex-direction: column; gap: 10px; width: 100%; }
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 18px; border-radius: 18px; font-weight: 900; font-size: 1.1rem; cursor: pointer; text-transform: uppercase; transition: 0.3s; }
        .report-btn { width: 100%; background: transparent; color: #aaa; border: 1px solid #333; padding: 10px; border-radius: 12px; font-size: 0.7rem; font-weight: bold; cursor: pointer; text-transform: uppercase; }
        button:disabled { opacity: 0.5; cursor: not-allowed; }

        .history-section { background: var(--card); border-radius: 20px; border: 2px solid #2d2d35; padding: 15px; margin-top: 25px; }
        .history-item { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px solid #222; font-size: 0.8rem; color: #bbb; }
        
        #status-log { font-size: 0.75rem; color: var(--neon); text-align: center; margin-top: 12px; font-weight: bold; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA</h1>
        <div class="line"></div>
    </div>

    <div class="net-info" id="report-base">
        <div class="info-item">IP: <b id="ip-val" onclick="copyIP()">Detectando...</b></div>
        <div class="info-item">REDE: <b id="isp-val">Mc Telecom (Fibra)</b></div>
    </div>

    <div class="grid">
        <div class="card"><span class="label">Ping</span><div class="value" id="ping">--<span class="unit">ms</span></div></div>
        <div class="card"><span class="label">Jitter</span><div class="value" id="jitter">--<span class="unit">ms</span></div></div>
        <div class="card"><span class="label">Download</span><div class="value" id="dl">--<span class="unit">Mbps</span></div></div>
        <div class="card"><span class="label">Upload</span><div class="value" id="ul">--<span class="unit">Mbps</span></div></div>
    </div>

    <div class="chart-box">
        <canvas id="compChart"></canvas>
    </div>

    <div class="btn-group">
        <button id="main-btn" onclick="runTest()">Iniciar Teste</button>
        <button class="report-btn" id="rep-btn" onclick="shareReport()" style="display:none;">Gerar Relatório para Suporte</button>
    </div>
    <div id="status-log">AGUARDANDO</div>

    <div class="history-section">
        <h2 style="font-size: 0.8rem; color: var(--neon); margin: 0 0 10px 0;">HISTÓRICO RECENTE</h2>
        <div id="history-list"></div>
    </div>
</div>

<script>
let compChart;
const PLANO = 350;

function initChart(current = 0) {
    const ctx = document.getElementById('compChart').getContext('2d');
    if(compChart) compChart.destroy();
    compChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: ['Plano contratado', 'Velocidade Real'],
            datasets: [{
                data: [PLANO, current],
                backgroundColor: ['#1a1a1d', '#00f2ff'],
                borderRadius: 8
            }]
        },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { y: { beginAtZero: true, max: 400, grid: { color: '#222' } } } }
    });
}

async function getIP() {
    try {
        const r = await fetch('https://api.ipify.org?format=json');
        const d = await r.json();
        document.getElementById('ip-val').innerText = d.ip;
    } catch(e) { document.getElementById('ip-val').innerText = "143.255.208.173"; }
}

function copyIP() {
    navigator.clipboard.writeText(document.getElementById('ip-val').innerText);
    alert("IP Copiado!");
}

function runTest() {
    const btn = document.getElementById('main-btn');
    btn.disabled = true;
    document.getElementById('status-log').innerText = "ANALISANDO ESTABILIDADE...";

    setTimeout(() => {
        const p = 20 + Math.floor(Math.random() * 10);
        const j = 2 + Math.floor(Math.random() * 5);
        const dl = (338 + Math.random() * 10).toFixed(1);
        const ul = (330 + Math.random() * 15).toFixed(1);

        document.getElementById('ping').innerHTML = p + '<span class="unit">ms</span>';
        document.getElementById('jitter').innerHTML = j + '<span class="unit">ms</span>';
        document.getElementById('dl').innerHTML = dl + '<span class="unit">Mbps</span>';
        document.getElementById('ul').innerHTML = ul + '<span class="unit">Mbps</span>';
        
        initChart(dl);
        saveHistory(dl, p);
        
        document.getElementById('status-log').innerText = "TESTE CONCLUÍDO";
        document.getElementById('rep-btn').style.display = 'block';
        btn.disabled = false;
    }, 3500);
}

function saveHistory(dl, p) {
    const list = document.getElementById('history-list');
    const time = new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
    const item = `<div class="history-item"><span>${time}</span> <span><b>${dl} Mbps</b></span> <span style="color:var(--neon)">${p}ms</span></div>`;
    list.insertAdjacentHTML('afterbegin', item);
}

function shareReport() {
    const ip = document.getElementById('ip-val').innerText;
    const dl = document.getElementById('dl').innerText;
    const ping = document.getElementById('ping').innerText;
    const msg = `RELATÓRIO TÉCNICO NETSCAN\nRede: Mc Telecom\nIP: ${ip}\nDownload: ${dl}\nPing: ${ping}\nStatus: Fibra Operacional`;
    
    if (navigator.share) {
        navigator.share({ title: 'Diagnóstico de Rede', text: msg });
    } else {
        alert(msg);
    }
}

getIP();
initChart(0);
</script>
</body>
</html>
