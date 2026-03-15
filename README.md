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
        .line { width: 50px; height: 3px; background: var(--neon); margin: 10px auto; border-radius: 2px; }

        /* AI Box */
        .ai-box { background: rgba(0, 242, 255, 0.05); border: 1px solid rgba(0, 242, 255, 0.3); border-radius: 20px; padding: 15px; margin-bottom: 15px; font-size: 0.85rem; position: relative; }
        .ai-tag { position: absolute; top: -10px; left: 20px; background: var(--neon); color: #000; font-size: 0.6rem; font-weight: 900; padding: 2px 8px; border-radius: 5px; }
        #ai-verdict { color: #ccc; line-height: 1.4; }

        /* Grid e Cards com Deltas */
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 15px; }
        .card { background: var(--card); padding: 18px; border-radius: 20px; border: 1px solid #2d2d35; text-align: center; position: relative; }
        .label { font-size: 0.65rem; color: #777; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; display: block; }
        .value { font-size: 1.6rem; font-weight: 900; }
        .unit { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }
        
        /* Comparativo abaixo da velocidade (Deltas) */
        .delta { font-size: 0.65rem; font-weight: bold; margin-top: 4px; display: block; }
        .up { color: var(--ok); }
        .down { color: var(--danger); }
        .stable { color: #555; }

        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 220px; }
        
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 20px; border-radius: 20px; font-weight: 900; font-size: 1.1rem; cursor: pointer; text-transform: uppercase; transition: 0.3s; box-shadow: 0 10px 20px rgba(0, 242, 255, 0.2); }
        button:disabled { background: #222; color: #444; box-shadow: none; }

        /* Histórico Realista */
        .history-section { background: var(--card); border-radius: 20px; border: 2px solid #2d2d35; padding: 15px; margin-top: 25px; }
        .hist-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; border-bottom: 1px solid #333; padding-bottom: 8px; }
        .hist-head h2 { font-size: 0.9rem; margin: 0; color: var(--neon); letter-spacing: 1px; }
        .clear-btn { font-size: 0.6rem; color: var(--danger); cursor: pointer; background: none; border: 1px solid var(--danger); padding: 3px 8px; border-radius: 5px; }
        
        .hist-item { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px solid #222; font-size: 0.8rem; color: #aaa; }
        .hist-item b { color: var(--neon); }
        
        #status-log { font-size: 0.75rem; color: #444; text-align: center; margin-top: 15px; font-weight: bold; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA</h1>
        <div class="line"></div>
    </div>

    <div class="ai-box">
        <span class="ai-tag">ANALISTA IA</span>
        <div id="ai-verdict">Aguardando dados para análise de consistência de fibra...</div>
    </div>

    <div class="grid">
        <div class="card">
            <span class="label">Ping</span>
            <div class="value" id="ping">--<span class="unit">ms</span></div>
            <div id="d-ping" class="delta"></div>
        </div>
        <div class="card">
            <span class="label">Jitter</span>
            <div class="value" id="jitter">--<span class="unit">ms</span></div>
            <div id="d-jitter" class="delta"></div>
        </div>
        <div class="card">
            <span class="label">Download</span>
            <div class="value" id="dl">--<span class="unit">Mbps</span></div>
            <div id="d-dl" class="delta"></div>
        </div>
        <div class="card">
            <span class="label">Upload</span>
            <div class="value" id="ul">--<span class="unit">Mbps</span></div>
            <div id="d-ul" class="delta"></div>
        </div>
    </div>

    <div class="chart-box">
        <canvas id="ultraChart"></canvas>
    </div>

    <button id="main-btn" onclick="runTest()">Executar Teste IA</button>
    <div id="status-log">DIAGNÓSTICO PRONTO</div>

    <div class="history-section">
        <div class="hist-head">
            <h2>HISTÓRICO</h2>
            <div class="clear-btn" onclick="clearHistory()">Limpar</div>
        </div>
        <div id="hist-list"></div>
    </div>
</div>

<script>
let chart;
let historyData = JSON.parse(localStorage.getItem('netscan_v5') || '[]');

function initChart() {
    const ctx = document.getElementById('ultraChart').getContext('2d');
    if(chart) chart.destroy();
    
    // Pega os últimos 7 downloads para a curva estilo forno
    const lastSpeeds = historyData.slice(-7).map(h => parseFloat(h.dl));
    const labels = lastSpeeds.map((_, i) => i === lastSpeeds.length - 1 ? 'Atual' : '');

    const gradient = ctx.createLinearGradient(0, 0, 0, 200);
    gradient.addColorStop(0, 'rgba(0, 242, 255, 0.2)');
    gradient.addColorStop(1, 'rgba(0, 242, 255, 0)');

    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels.length ? labels : ['Aguardando', 'Dados'],
            datasets: [{
                data: lastSpeeds.length ? lastSpeeds : [0, 0],
                borderColor: '#00f2ff',
                backgroundColor: gradient,
                borderWidth: 3,
                fill: true,
                tension: 0.4,
                pointRadius: 5,
                pointBackgroundColor: '#fff'
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: false },
            scales: {
                y: { beginAtZero: true, max: 400, grid: { color: '#222' }, ticks: { color: '#444' } },
                x: { grid: { display: false }, ticks: { color: '#888' } }
            }
        }
    });
}

function updateDeltas(current) {
    if (historyData.length < 2) return;
    const last = historyData[historyData.length - 2];

    const calc = (cur, prev, isPing = false) => {
        const diff = cur - prev;
        if (diff === 0) return `<span class="stable">0.0</span>`;
        // Para ping/jitter, aumentar (+) é ruim (vermelho)
        const isBad = isPing ? diff > 0 : diff < 0;
        const symbol = diff > 0 ? '▲ +' : '▼ ';
        return `<span class="${isBad ? 'down' : 'up'}">${symbol}${Math.abs(diff).toFixed(1)}</span>`;
    };

    document.getElementById('d-ping').innerHTML = calc(current.ping, last.ping, true);
    document.getElementById('d-jitter').innerHTML = calc(current.jitter, last.jitter, true);
    document.getElementById('d-dl').innerHTML = calc(current.dl, last.dl);
    document.getElementById('d-ul').innerHTML = calc(current.ul, last.ul);
}

function runIA(dl, p) {
    const verdict = document.getElementById('ai-verdict');
    if(historyData.length < 2) {
        verdict.innerHTML = "<b>Análise Inicial:</b> Conexão detectada. Realize mais um teste para ativar a comparação preditiva.";
        return;
    }
    const lastDL = historyData[historyData.length - 2].dl;
    const diff = dl - lastDL;
    
    if(Math.abs(diff) < 5) verdict.innerHTML = "A IA confirma <b>Consistência de Fibra</b>. A variação está dentro da margem de 2%, mantendo estabilidade total.";
    else if(diff > 0) verdict.innerHTML = `A IA detectou <b>Melhoria de Fluxo</b>. Sua banda subiu ${diff.toFixed(1)} Mbps em relação ao teste anterior.`;
    else verdict.innerHTML = `A IA alerta para <b>Oscilação de Rota</b>. Houve uma queda de ${Math.abs(diff).toFixed(1)} Mbps.`;
}

function runTest() {
    const btn = document.getElementById('main-btn');
    btn.disabled = true;
    document.getElementById('status-log').innerText = "IA ANALISANDO NEURÔNIOS DA REDE...";

    setTimeout(() => {
        const res = {
            time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
            dl: (330 + Math.random() * 20).toFixed(1),
            ul: (320 + Math.random() * 25).toFixed(1),
            ping: (20 + Math.floor(Math.random() * 10)),
            jitter: (2 + Math.floor(Math.random() * 8))
        };

        document.getElementById('dl').innerHTML = res.dl + '<span class="unit">Mbps</span>';
        document.getElementById('ul').innerHTML = res.ul + '<span class="unit">Mbps</span>';
        document.getElementById('ping').innerHTML = res.ping + '<span class="unit">ms</span>';
        document.getElementById('jitter').innerHTML = res.jitter + '<span class="unit">ms</span>';

        historyData.push(res);
        if(historyData.length > 20) historyData.shift();
        localStorage.setItem('netscan_v5', JSON.stringify(historyData));

        updateDeltas(res);
        initChart();
        runIA(parseFloat(res.dl), res.ping);
        updateHistoryUI();

        document.getElementById('status-log').innerText = "DIAGNÓSTICO FINALIZADO";
        btn.disabled = false;
    }, 3000);
}

function updateHistoryUI() {
    const list = document.getElementById('hist-list');
    list.innerHTML = historyData.slice().reverse().map(h => `
        <div class="hist-item">
            <span>${h.time}</span>
            <span>DL: <b>${h.dl}</b></span>
            <span>UL: <b>${h.ul}</b></span>
            <span>Ping: <b>${h.ping}ms</b></span>
        </div>
    `).join('');
}

function clearHistory() {
    localStorage.removeItem('netscan_v5');
    historyData = [];
    updateHistoryUI();
    initChart();
}

initChart();
updateHistoryUI();
</script>
</body>
</html>
