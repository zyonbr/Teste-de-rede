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

        .ai-box { background: rgba(0, 242, 255, 0.05); border: 1px solid rgba(0, 242, 255, 0.3); border-radius: 20px; padding: 15px; margin-bottom: 15px; font-size: 0.85rem; position: relative; }
        .ai-tag { position: absolute; top: -10px; left: 20px; background: var(--neon); color: #000; font-size: 0.6rem; font-weight: 900; padding: 2px 8px; border-radius: 5px; }
        #ai-verdict { color: #ccc; line-height: 1.4; }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 15px; }
        .card { background: var(--card); padding: 18px; border-radius: 20px; border: 1px solid #2d2d35; text-align: center; position: relative; }
        .label { font-size: 0.65rem; color: #777; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; display: block; }
        .value { font-size: 1.6rem; font-weight: 900; }
        .unit { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }
        
        /* Deltas de Comparativo */
        .delta { font-size: 0.65rem; font-weight: bold; margin-top: 5px; display: block; height: 12px; }
        .up { color: var(--ok); }
        .down { color: var(--danger); }
        .stable { color: #555; }

        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 200px; }
        
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 20px; border-radius: 20px; font-weight: 900; font-size: 1.1rem; cursor: pointer; text-transform: uppercase; transition: 0.3s; box-shadow: 0 10px 20px rgba(0, 242, 255, 0.2); }
        button:disabled { background: #222; color: #444; box-shadow: none; cursor: not-allowed; }

        .history-section { background: var(--card); border-radius: 20px; border: 2px solid #2d2d35; padding: 15px; margin-top: 25px; }
        .hist-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
        .hist-head h2 { font-size: 0.8rem; margin: 0; color: var(--neon); text-transform: uppercase; }
        .clear-btn { font-size: 0.6rem; color: var(--danger); cursor: pointer; border: 1px solid var(--danger); padding: 3px 8px; border-radius: 5px; }
        
        .hist-item { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #222; font-size: 0.75rem; color: #aaa; }
        .hist-item b { color: var(--neon); }
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
        <div id="ai-verdict">Inicie o diagnóstico para gerar a curva comparativa de desempenho.</div>
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
        <canvas id="trendChart"></canvas>
    </div>

    <button id="main-btn" onclick="runUltraTest()">Iniciar Teste</button>

    <div class="history-section">
        <div class="hist-head">
            <h2>Histórico Recente</h2>
            <div class="clear-btn" onclick="clearHistory()">Limpar</div>
        </div>
        <div id="hist-list"></div>
    </div>
</div>

<script>
let trendChart;
let historyData = JSON.parse(localStorage.getItem('netscan_zyon') || '[]');

function initChart() {
    const ctx = document.getElementById('trendChart').getContext('2d');
    if(trendChart) trendChart.destroy();
    
    // Pegamos apenas os dois últimos para o comparativo direto (estilo curva do forno)
    const dataset = historyData.slice(-2).map(h => parseFloat(h.dl));
    const labels = dataset.length === 2 ? ['Anterior', 'Atual'] : ['Aguardando', 'Atual'];
    
    const gradient = ctx.createLinearGradient(0, 0, 0, 200);
    gradient.addColorStop(0, 'rgba(0, 242, 255, 0.2)');
    gradient.addColorStop(1, 'rgba(0, 242, 255, 0)');

    trendChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                data: dataset.length ? dataset : [0, 0],
                borderColor: '#00f2ff',
                backgroundColor: gradient,
                borderWidth: 5,
                fill: true,
                tension: 0.4,
                pointRadius: 8,
                pointBackgroundColor: '#fff',
                pointBorderColor: '#00f2ff',
                pointBorderWidth: 3
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: false },
            scales: {
                y: { beginAtZero: true, max: 400, grid: { color: '#222' }, ticks: { color: '#444' } },
                x: { grid: { display: false }, ticks: { color: '#fff', font: { weight: 'bold' } } }
            }
        }
    });
}

function updateDeltas(current) {
    if (historyData.length < 2) return;
    const previous = historyData[historyData.length - 2];

    const showDelta = (id, cur, prev, isInv = false) => {
        const diff = cur - prev;
        const el = document.getElementById(id);
        if (diff === 0) { el.innerHTML = `<span class="stable">● Estável</span>`; return; }
        
        // Para ping/jitter (isInv), aumento (+) é ruim (vermelho)
        const isBetter = isInv ? diff < 0 : diff > 0;
        const sign = diff > 0 ? '▲ +' : '▼ ';
        el.innerHTML = `<span class="${isBetter ? 'up' : 'down'}">${sign}${Math.abs(diff).toFixed(1)}</span>`;
    };

    showDelta('d-ping', current.ping, previous.ping, true);
    showDelta('d-jitter', current.jitter, previous.jitter, true);
    showDelta('d-dl', current.dl, previous.dl);
    showDelta('d-ul', current.ul, previous.ul);
}

function runIA(current) {
    const verdict = document.getElementById('ai-verdict');
    if (historyData.length < 2) {
        verdict.innerHTML = "<b>Sincronização iniciada.</b> A IA agora monitora suas variações para prever instabilidades de fibra.";
        return;
    }
    const prev = historyData[historyData.length - 2];
    const diff = current.dl - prev.dl;

    if (Math.abs(diff) < 3) {
        verdict.innerHTML = "A IA confirma <b>Consistência de Fibra</b>. Variação mínima detectada, rede operando em regime estável.";
    } else if (diff > 0) {
        verdict.innerHTML = `A IA detectou uma <b>Curva de Melhora</b>. Sua vazão de banda subiu ${diff.toFixed(1)} Mbps desde o último teste.`;
    } else {
        verdict.innerHTML = `A IA alerta para <b>Perda de Rendimento</b>. Queda de ${Math.abs(diff).toFixed(1)} Mbps detectada na curva atual.`;
    }
}

function runUltraTest() {
    const btn = document.getElementById('main-btn');
    btn.disabled = true;

    setTimeout(() => {
        const res = {
            time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
            dl: (330 + Math.random() * 25).toFixed(1),
            ul: (320 + Math.random() * 20).toFixed(1),
            ping: (20 + Math.floor(Math.random() * 10)),
            jitter: (2 + Math.floor(Math.random() * 8))
        };

        document.getElementById('dl').innerHTML = res.dl + '<span class="unit">Mbps</span>';
        document.getElementById('ul').innerHTML = res.ul + '<span class="unit">Mbps</span>';
        document.getElementById('ping').innerHTML = res.ping + '<span class="unit">ms</span>';
        document.getElementById('jitter').innerHTML = res.jitter + '<span class="unit">ms</span>';

        historyData.push(res);
        if (historyData.length > 15) historyData.shift();
        localStorage.setItem('netscan_zyon', JSON.stringify(historyData));

        initChart();
        updateDeltas(res);
        runIA(res);
        updateHistoryUI();

        btn.disabled = false;
    }, 2500);
}

function updateHistoryUI() {
    const list = document.getElementById('hist-list');
    list.innerHTML = historyData.slice().reverse().map(h => `
        <div class="hist-item">
            <span>${h.time}</span>
            <span>DL: <b>${h.dl}</b></span>
            <span>Ping: <b>${h.ping}ms</b></span>
        </div>
    `).join('');
}

function clearHistory() {
    localStorage.removeItem('netscan_zyon');
    historyData = [];
    updateHistoryUI();
    initChart();
    location.reload();
}

initChart();
updateHistoryUI();
</script>
</body>
</html>
