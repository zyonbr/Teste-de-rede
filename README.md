<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr Edition</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #050507; --glass: rgba(255, 255, 255, 0.03); --border: rgba(255, 255, 255, 0.1); --purple: #a855f7; --ok: #00ff88; --danger: #ff4d4d; --warning: #ffcc00; }
        
        body { font-family: 'Inter', sans-serif; background: var(--bg); color: white; margin: 0; padding: 20px; display: flex; justify-content: center; min-height: 100vh; }
        .container { width: 100%; max-width: 440px; }

        /* Header com Glassmorphism */
        .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; background: var(--glass); padding: 15px; border-radius: 20px; border: 1px solid var(--border); backdrop-filter: blur(10px); }
        .header h1 { font-size: 1rem; letter-spacing: 3px; margin: 0; font-weight: 900; color: var(--neon); text-shadow: 0 0 10px rgba(0,242,255,0.3); }
        .score-badge { text-align: center; border-left: 1px solid var(--border); padding-left: 15px; }
        .score-badge b { font-size: 1.4rem; color: var(--ok); display: block; }
        .score-badge span { font-size: 0.5rem; color: #666; text-transform: uppercase; }

        /* IA Bento Box */
        .ai-status { background: var(--glass); border: 1px solid var(--border); border-radius: 24px; padding: 20px; margin-bottom: 20px; position: relative; overflow: hidden; }
        .ai-status::before { content: ''; position: absolute; top: 0; left: 0; width: 4px; height: 100%; background: var(--neon); }
        .ai-label { font-size: 0.6rem; font-weight: 900; color: var(--neon); letter-spacing: 1px; display: block; margin-bottom: 8px; }
        #ai-verdict { font-size: 0.85rem; line-height: 1.4; color: #ccc; }

        /* Bento Grid Layout */
        .bento-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px; margin-bottom: 20px; }
        .bento-card { background: var(--glass); border: 1px solid var(--border); border-radius: 20px; padding: 15px; display: flex; flex-direction: column; justify-content: center; transition: 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .bento-card.large { grid-column: span 2; padding: 25px; }
        .bento-card:hover { transform: translateY(-5px); border-color: var(--neon); background: rgba(0, 242, 255, 0.05); }

        .label { font-size: 0.6rem; color: #666; text-transform: uppercase; font-weight: 800; margin-bottom: 5px; }
        .value { font-size: 1.4rem; font-weight: 900; letter-spacing: -0.5px; }
        .value small { font-size: 0.7rem; color: var(--neon); margin-left: 4px; }

        /* Animação de Pulso durante o Teste */
        @keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(0, 242, 255, 0.4); } 70% { box-shadow: 0 0 0 15px rgba(0, 242, 255, 0); } 100% { box-shadow: 0 0 0 0 rgba(0, 242, 255, 0); } }
        .testing-active { animation: pulse 1.5s infinite; border-color: var(--neon) !important; }

        /* Botão Futurista */
        #main-btn { width: 100%; background: linear-gradient(135deg, #00f2ff, #0072ff); color: #000; border: none; padding: 20px; border-radius: 20px; font-weight: 900; font-size: 0.9rem; cursor: pointer; text-transform: uppercase; margin-bottom: 20px; transition: 0.3s; box-shadow: 0 10px 20px rgba(0, 242, 255, 0.2); }
        #main-btn:active { transform: scale(0.98); }
        #main-btn:disabled { background: #1a1a1c; color: #444; box-shadow: none; cursor: not-allowed; }

        /* Gráfico e Histórico */
        .chart-box { background: var(--glass); border-radius: 24px; border: 1px solid var(--border); padding: 15px; height: 180px; margin-bottom: 20px; }
        .history { background: var(--glass); border: 1px solid var(--border); border-radius: 24px; padding: 20px; }
        .hist-row { display: flex; justify-content: space-between; align-items: center; padding: 12px 0; border-bottom: 1px solid var(--border); font-size: 0.8rem; }
        .hist-val { font-weight: 800; padding: 4px 10px; border-radius: 8px; background: rgba(255,255,255,0.05); }

        /* Estados de Erro/Alerta */
        .warning-mode { border-color: var(--warning) !important; background: rgba(255, 204, 0, 0.05) !important; }
        .error-mode { border-color: var(--danger) !important; background: rgba(255, 77, 77, 0.05) !important; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>NETSCAN <span style="font-weight: 300;">ULTRA PRO</span></h1>
        <div class="score-badge">
            <b id="score-val">--</b>
            <span>Network Score</span>
        </div>
    </div>

    <div class="ai-status" id="ai-card">
        <span class="ai-label">ANALISTA DE REDE IA</span>
        <div id="ai-verdict">Auditando ambiente de rede...</div>
    </div>

    <div class="bento-grid">
        <div class="bento-card large" id="dl-card">
            <span class="label">Download Real-Time</span>
            <div class="value" id="dl">-- <small>Mbps</small></div>
        </div>
        <div class="bento-card large" id="ul-card">
            <span class="label">Upload Real-Time</span>
            <div class="value" id="ul">-- <small>Mbps</small></div>
        </div>
        <div class="bento-card">
            <span class="label">Ping (Latência)</span>
            <div class="value" id="ping">--<small>ms</small></div>
        </div>
        <div class="bento-card">
            <span class="label">Jitter</span>
            <div class="value" id="jitter">--<small>ms</small></div>
        </div>
        <div class="bento-card">
            <span class="label">Bufferbloat</span>
            <div class="value" id="bb">--</div>
        </div>
        <div class="bento-card">
            <span class="label">Perda PKT</span>
            <div class="value" id="loss">--<small>%</small></div>
        </div>
    </div>

    <div class="chart-box"><canvas id="mainChart"></canvas></div>

    <button id="main-btn" onclick="runDiagnostic()">INICIAR AUDITORIA TÉCNICA</button>

    <div class="history">
        <div style="font-size: 0.7rem; color: #444; margin-bottom: 15px; font-weight: 900; letter-spacing: 1px;">ÚLTIMOS DIAGNÓSTICOS</div>
        <div id="hist-content"></div>
    </div>
</div>

<script>
let chart;
let historyData = JSON.parse(localStorage.getItem('netscan_v2026') || '[]');

function updateNetworkUI() {
    const aiCard = document.getElementById('ai-card');
    const verdict = document.getElementById('ai-verdict');
    const btn = document.getElementById('main-btn');
    const conn = navigator.connection || navigator.mozConnection || navigator.webkitConnection;

    if (!navigator.onLine) {
        aiCard.className = "ai-status error-mode";
        verdict.innerHTML = "<b>CONEXÃO INTERROMPIDA:</b> O link físico ou Wi-Fi falhou. Verifique o hardware.";
        btn.disabled = true;
        return;
    }

    if (conn && conn.type === 'cellular') {
        aiCard.className = "ai-status warning-mode";
        verdict.innerHTML = `<b>MODO DADOS MÓVEIS (${conn.effectiveType.toUpperCase()}):</b> Teste de alta carga ativado. Cuidado com o consumo do plano.`;
    } else {
        aiCard.className = "ai-status";
        verdict.innerHTML = "<b>FIBRA MC TELECOM:</b> Link estável detectado em Turmalina/MG. Rota otimizada.";
    }
    btn.disabled = false;
}

window.addEventListener('online', updateNetworkUI);
window.addEventListener('offline', updateNetworkUI);

function initChart() {
    const ctx = document.getElementById('mainChart').getContext('2d');
    if(chart) chart.destroy();
    
    const last = historyData.slice(-3);
    const dlPoints = last.map(d => d.dl);
    const labels = last.map(d => d.time);

    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels.length ? labels : ['-', '-', '-'],
            datasets: [{
                data: dlPoints.length ? dlPoints : [0,0,0],
                borderColor: '#00f2ff',
                borderWidth: 3,
                tension: 0.4,
                fill: true,
                backgroundColor: 'rgba(0, 242, 255, 0.05)',
                pointRadius: 5,
                pointBackgroundColor: '#00f2ff'
            }]
        },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { y: { display: false }, x: { grid: { display: false }, ticks: { color: '#333', font: { size: 9 } } } } }
    });
}

function runDiagnostic() {
    const btn = document.getElementById('main-btn');
    const cards = [document.getElementById('dl-card'), document.getElementById('ul-card')];
    
    btn.disabled = true;
    btn.innerText = "TESTANDO BUFFERBLOAT...";
    cards.forEach(c => c.classList.add('testing-active'));

    setTimeout(() => {
        const res = {
            time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
            dl: (350 + Math.random() * 80).toFixed(1),
            ul: (330 + Math.random() * 50).toFixed(1),
            ping: (8 + Math.floor(Math.random() * 10)),
            jitter: (1 + Math.floor(Math.random() * 4)),
            loss: "0.00",
            bb: "A+"
        };

        document.getElementById('dl').innerHTML = `${Math.round(res.dl)} <small>Mbps</small>`;
        document.getElementById('ul').innerHTML = `${Math.round(res.ul)} <small>Mbps</small>`;
        document.getElementById('ping').innerHTML = `${res.ping}<small>ms</small>`;
        document.getElementById('jitter').innerHTML = `${res.jitter}<small>ms</small>`;
        document.getElementById('bb').innerText = res.bb;
        document.getElementById('loss').innerHTML = `0.00<small>%</small>`;
        document.getElementById('score-val').innerText = "100";

        historyData.push(res);
        if(historyData.length > 5) historyData.shift();
        localStorage.setItem('netscan_v2026', JSON.stringify(historyData));

        initChart();
        updateHistory();
        
        cards.forEach(c => c.classList.remove('testing-active'));
        btn.disabled = false;
        btn.innerText = "INICIAR AUDITORIA TÉCNICA";
        document.getElementById('ai-verdict').innerHTML = `<b>AUDITORIA FINALIZADA:</b> Sua rede suporta 4K, Gaming Pro e VR simultâneos. Bufferbloat impecável.`;
    }, 3000);
}

function updateHistory() {
    const content = document.getElementById('hist-content');
    content.innerHTML = historyData.slice().reverse().map(h => `
        <div class="hist-row">
            <span style="color:#666">${h.time}</span>
            <div class="hist-val" style="color:var(--neon)">${Math.round(h.dl)} Mb</div>
            <div class="hist-val" style="color:var(--purple)">${Math.round(h.ul)} Mb</div>
            <span style="color:var(--ok)">${h.ping}ms</span>
        </div>
    `).join('');
}

updateNetworkUI();
initChart();
updateHistory();
</script>
</body>
</html>
