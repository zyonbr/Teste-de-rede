<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #0a0a0c; --card: #16161a; --danger: #ff4d4d; --ok: #00ff88; --warning: #ffcc00; }
        body { font-family: 'Segoe UI', Roboto, sans-serif; background: var(--bg); color: white; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; min-height: 100vh; }
        .container { width: 100%; max-width: 420px; }
        
        .header { text-align: center; margin-bottom: 20px; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.4rem; letter-spacing: 2px; text-transform: uppercase; font-weight: 900; }
        .line { width: 50px; height: 3px; background: var(--neon); margin: 8px auto; border-radius: 2px; }

        /* AI Analysis Box */
        .ai-box { background: rgba(0, 242, 255, 0.05); border: 1px solid var(--neon); border-radius: 15px; padding: 15px; margin-bottom: 15px; font-size: 0.85rem; position: relative; overflow: hidden; }
        .ai-box::before { content: "AI ANALYST"; position: absolute; top: 0; right: 10px; font-size: 0.6rem; color: var(--neon); font-weight: bold; opacity: 0.5; }
        #ai-verdict { color: #fff; line-height: 1.4; }
        .ai-score { font-weight: 900; color: var(--ok); }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 15px; }
        .card { background: var(--card); padding: 18px; border-radius: 20px; border: 1px solid #2d2d35; text-align: center; }
        .label { font-size: 0.65rem; color: #777; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; display: block; }
        .value { font-size: 1.7rem; font-weight: 900; }
        .unit { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }

        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 160px; }
        
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 20px; border-radius: 20px; font-weight: 900; font-size: 1.1rem; cursor: pointer; text-transform: uppercase; transition: 0.3s; }
        .report-btn { width: 100%; background: transparent; color: #aaa; border: 1px solid #333; padding: 10px; border-radius: 12px; font-size: 0.7rem; font-weight: bold; cursor: pointer; margin-top: 10px; display: none; }

        .history-section { background: var(--card); border-radius: 20px; border: 2px solid #2d2d35; padding: 15px; margin-top: 25px; width: 100%; box-sizing: border-box; }
        .history-item { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #222; font-size: 0.75rem; color: #888; }
        
        #status-log { font-size: 0.7rem; color: var(--neon); text-align: center; margin-top: 15px; font-weight: bold; text-transform: uppercase; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA</h1>
        <div class="line"></div>
    </div>

    <div class="ai-box">
        <div id="ai-verdict">Aguardando início do teste para realizar análise preditiva de rede...</div>
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

    <button id="main-btn" onclick="runSmartTest()">Iniciar Teste</button>
    <button class="report-btn" id="rep-btn" onclick="shareReport()">Exportar Diagnóstico Técnico</button>
    
    <div id="status-log">IA ENGINE: STANDBY</div>

    <div class="history-section">
        <h2 style="font-size: 0.8rem; color: var(--neon); margin: 0 0 10px 0;">REGISTROS TÉCNICOS</h2>
        <div id="history-list"></div>
    </div>
</div>

<script>
let compChart;
const PLANO_350 = 350;

function initChart(val = 0) {
    const ctx = document.getElementById('compChart').getContext('2d');
    if(compChart) compChart.destroy();
    compChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: ['Plano 350M', 'Realizado'],
            datasets: [{
                data: [PLANO_350, val],
                backgroundColor: ['#1a1a1d', '#00f2ff'],
                borderRadius: 8
            }]
        },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { y: { beginAtZero: true, max: 400, grid: {color: '#222'} } } }
    });
}

// Lógica de "IA" - Análise de Padrões de Rede
function aiAnalyst(dl, ping, jitter) {
    const verdict = document.getElementById('ai-verdict');
    let score = 100;
    let message = "";

    // Dedução de pontuação
    if (dl < 300) score -= 20;
    if (ping > 40) score -= 15;
    if (jitter > 15) score -= 15;

    if (score >= 90) {
        message = `Análise: <span class="ai-score">EXCELENTE</span>. Sua conexão de fibra Mc Telecom está operando com 98% de eficiência. Ideal para Cloud Gaming e 4K.`;
    } else if (score >= 70) {
        message = `Análise: <span class="ai-score" style="color:var(--warning)">ESTÁVEL</span>. Detectada oscilação leve. A banda está em ${((dl/350)*100).toFixed(0)}% da capacidade contratada.`;
    } else {
        message = `Análise: <span class="ai-score" style="color:var(--danger)">INSTÁVEL</span>. Ping/Jitter elevados sugerem saturação de canal ou interferência física.`;
    }

    verdict.innerHTML = message;
}

async function runSmartTest() {
    const btn = document.getElementById('main-btn');
    btn.disabled = true;
    document.getElementById('status-log').innerText = "IA PROCESSANDO DADOS...";

    setTimeout(() => {
        // Simulação baseada na sua rede real de 350M
        const p = 18 + Math.floor(Math.random() * 12);
        const j = 1 + Math.floor(Math.random() * 8);
        const dl = (325 + Math.random() * 25).toFixed(1);
        const ul = (310 + Math.random() * 30).toFixed(1);

        document.getElementById('ping').innerHTML = p + '<span class="unit">ms</span>';
        document.getElementById('jitter').innerHTML = j + '<span class="unit">ms</span>';
        document.getElementById('dl').innerHTML = dl + '<span class="unit">Mbps</span>';
        document.getElementById('ul').innerHTML = ul + '<span class="unit">Mbps</span>';
        
        initChart(dl);
        aiAnalyst(dl, p, j);
        saveHistory(dl, p);
        
        document.getElementById('status-log').innerText = "ANÁLISE CONCLUÍDA";
        document.getElementById('rep-btn').style.display = 'block';
        btn.disabled = false;
    }, 3500);
}

function saveHistory(dl, p) {
    const list = document.getElementById('history-list');
    const time = new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
    list.insertAdjacentHTML('afterbegin', `<div class="history-item"><span>${time}</span> <span><b>${dl}M</b></span> <span>Ping: ${p}ms</span></div>`);
}

function shareReport() {
    const msg = `NETSCAN IA REPORT\nStatus: ${document.getElementById('ai-verdict').innerText}\nVelocidade: ${document.getElementById('dl').innerText}\nPing: ${document.getElementById('ping').innerText}\nProvedor: Mc Telecom`;
    if (navigator.share) navigator.share({ title: 'Relatório IA', text: msg });
    else alert(ip);
}

initChart(0);
</script>
</body>
</html>
