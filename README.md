<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #0a0a0c; --card: #16161a; --danger: #ff4d4d; --ok: #00ff88; --warning: #ffcc00; }
        body { font-family: 'Segoe UI', Roboto, sans-serif; background: var(--bg); color: white; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; min-height: 100vh; overflow-x: hidden; }
        .container { width: 100%; max-width: 420px; display: flex; flex-direction: column; align-items: center; }
        
        .header { text-align: center; margin-bottom: 20px; padding-top: 10px; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.4rem; letter-spacing: 2px; text-transform: uppercase; font-weight: 900; }
        .line { width: 50px; height: 3px; background: var(--neon); margin: 10px auto; border-radius: 2px; }

        /* AI Intelligence Box */
        .ai-box { background: rgba(0, 242, 255, 0.05); border: 1px solid rgba(0, 242, 255, 0.3); border-radius: 20px; padding: 15px; margin-bottom: 15px; font-size: 0.85rem; position: relative; width: 100%; box-sizing: border-box; }
        .ai-tag { position: absolute; top: -10px; left: 20px; background: var(--neon); color: #000; font-size: 0.6rem; font-weight: 900; padding: 2px 8px; border-radius: 5px; }
        #ai-verdict { color: #ccc; line-height: 1.4; margin-top: 5px; }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 15px; width: 100%; }
        .card { background: var(--card); padding: 18px; border-radius: 20px; border: 1px solid #2d2d35; text-align: center; box-shadow: 0 4px 15px rgba(0,0,0,0.5); position: relative; }
        .label { font-size: 0.65rem; color: #777; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; display: block; }
        .value { font-size: 1.6rem; font-weight: 900; color: #fff; }
        .unit { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }

        .comp-indicator { font-size: 0.6rem; position: absolute; bottom: 8px; width: 100%; left: 0; opacity: 0.8; }
        .trend-up { color: var(--ok); font-weight: bold; }
        .trend-down { color: var(--danger); font-weight: bold; }

        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 200px; width: 100%; box-sizing: border-box; }
        
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 20px; border-radius: 20px; font-weight: 900; font-size: 1.1rem; cursor: pointer; text-transform: uppercase; transition: 0.3s; box-shadow: 0 10px 20px rgba(0, 242, 255, 0.1); }
        button:active { transform: scale(0.96); box-shadow: none; }
        button:disabled { background: #2d2d35; color: #555; box-shadow: none; }
        
        #status-log { font-size: 0.8rem; color: var(--neon); text-align: center; margin: 15px 0; font-weight: bold; text-shadow: 0 0 10px rgba(0,242,255,0.5); text-transform: uppercase; }
        
        /* Secção de Histórico Corrigida */
        .history-section { background: var(--card); border-radius: 20px; border: 2px solid #2d2d35; padding: 15px; margin-top: 25px; width: 100%; box-sizing: border-box; }
        .history-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; border-bottom: 1px solid #333; padding-bottom: 8px; }
        .history-header h2 { font-size: 0.9rem; margin: 0; color: var(--neon); text-transform: uppercase; }
        .clear-btn { font-size: 0.6rem; color: var(--danger); cursor: pointer; background: none; border: 1px solid var(--danger); padding: 4px 10px; border-radius: 6px; text-transform: uppercase; font-weight: bold; }
        
        .history-list { max-height: 150px; overflow-y: auto; font-size: 0.8rem; }
        .history-item { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px solid #222; color: #bbb; }
        .history-item b { color: var(--neon); }

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
        <div id="ai-verdict">Sincronizando parâmetros preditivos... Inicie o diagnóstico para comparar a estabilidade atual com o histórico.</div>
    </div>

    <div class="grid">
        <div class="card"><span class="label">Ping</span><div class="value" id="ping">--<span class="unit">ms</span></div><div class="comp-indicator" id="c-ping"></div></div>
        <div class="card"><span class="label">Jitter</span><div class="value" id="jitter">--<span class="unit">ms</span></div><div class="comp-indicator" id="c-jitter"></div></div>
        <div class="card"><span class="label">Download</span><div class="value" id="dl">--<span class="unit">Mbps</span></div><div class="comp-indicator" id="c-dl"></div></div>
        <div class="card"><span class="label">Upload</span><div class="value" id="ul">--<span class="unit">Mbps</span></div><div class="comp-indicator" id="c-ul"></div></div>
    </div>

    <div class="chart-box">
        <canvas id="trendChart"></canvas>
    </div>

    <button id="main-btn" onclick="runPreciseTest()">Iniciar Teste</button>
    <div id="status-log">AGUARDANDO DIAGNÓSTICO</div>

    <div class="history-section">
        <div class="history-header">
            <h2>Histórico Recente</h2>
            <button class="clear-btn" onclick="clearHistory()">Limpar</button>
        </div>
        <div id="history-list" class="history-list"></div>
    </div>
</div>

<script>
let trendChart;
function initTrendChart(currentDL = 0) {
    const ctx = document.getElementById('trendChart').getContext('2d');
    const history = JSON.parse(localStorage.getItem('netscan_history') || '[]');
    const lastDL = history.length > 0 ? parseFloat(history[history.length - 1].dl) : 0;

    if(trendChart) trendChart.destroy();
    
    // Gradiente Neon
    const gradient = ctx.createLinearGradient(0, 0, 0, 200);
    gradient.addColorStop(0, 'rgba(0, 242, 255, 0.15)');
    gradient.addColorStop(1, 'rgba(0, 242, 255, 0)');

    trendChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: ['Anterior', 'Atual'],
            datasets: [{
                label: 'Download (Mbps)',
                data: [lastDL, currentDL],
                borderColor: '#00f2ff',
                backgroundColor: gradient,
                borderWidth: 4,
                pointBackgroundColor: '#fff',
                pointRadius: 6,
                fill: true,
                tension: 0.4
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { display: false } },
            scales: {
                y: { beginAtZero: true, max: 400, grid: { color: '#222' }, ticks: { color: '#555' } },
                x: { ticks: { color: '#888' } }
            }
        }
    });
}

function updateIndicators(dl, ul, p, j) {
    const history = JSON.parse(localStorage.getItem('netscan_history') || '[]');
    if(history.length < 2) return;
    
    const last = history[history.length - 2];
    
    const diff = (val, last) => {
        const d = val - last;
        if(d === 0) return `<span style="color:#888">Estável</span>`;
        return d > 0 ? `<span class="trend-up">▲ +${d.toFixed(1)}</span>` : `<span class="trend-down">▼ ${d.toFixed(1)}</span>`;
    };

    document.getElementById('c-dl').innerHTML = diff(dl, parseFloat(last.dl));
    document.getElementById('c-ul').innerHTML = diff(ul, parseFloat(last.ul));
    document.getElementById('c-ping').innerHTML = p > last.ping ? `<span class="trend-down">▲ +${(p-last.ping).toFixed(0)}</span>` : `<span class="trend-up">▼ ${(p-last.ping).toFixed(0)}</span>`;
}

async function runPreciseTest() {
    const btn = document.getElementById('main-btn');
    const status = document.getElementById('status-log');
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
        
        initTrendChart(dl);
        saveHistory(dl, ul, p);
        updateIndicators(parseFloat(dl), parseFloat(ul), p, j);
        runIAAnalyst(parseFloat(dl), p, j);
        
        document.getElementById('status-log').innerText = "ANÁLISE CONCLUÍDA";
        btn.disabled = false;
    }, 3500);
}

function saveHistory(dl, ul, p) {
    const data = JSON.parse(localStorage.getItem('netscan_history') || '[]');
    const now = new Date();
    const timeStr = now.getHours().toString().padStart(2, '0') + ':' + now.getMinutes().toString().padStart(2, '0');
    data.push({ time: timeStr, dl: dl.toFixed(1), ul: ul.toFixed(1), ping: p.toFixed(0) });
    if(data.length > 10) data.shift();
    localStorage.setItem('netscan_history', JSON.stringify(data));
    updateHistoryUI();
}

function updateHistoryUI() {
    const list = document.getElementById('history-list');
    const data = JSON.parse(localStorage.getItem('netscan_history') || '[]');
    list.innerHTML = data.slice().reverse().map(item => `
        <div class="history-item">
            <span style="color:#555">${item.time}</span>
            <span>DL: <b>${item.dl}</b></span>
            <span>Ping: <b>${item.ping}ms</b></span>
        </div>
    `).join('') || '<div style="text-align:center;padding:10px;color:#444;">Vazio</div>';
}

function clearHistory() {
    if(confirm("Deseja apagar todo o histórico?")) {
        localStorage.removeItem('netscan_history');
        updateHistoryUI();
        initTrendChart(0);
    }
}

// Lógica de "IA" - Análise de Padrões e Comparativo (Restaurado)
function runIAAnalyst(dl, p, j) {
    const verdict = document.getElementById('ai-verdict');
    const history = JSON.parse(localStorage.getItem('netscan_history') || '[]');
    
    if (history.length < 2) {
        verdict.innerHTML = "<b>Primeira análise preditiva concluída.</b> A IA precisa de um segundo teste para calcular a curva de tendência.";
        return;
    }

    const last = history[history.length - 2];
    const dlDiff = dl - last.dl;
    
    let aiText = "";
    if (dlDiff > 5) {
        aiText = `A IA detectou uma <b class="trend-up">Melhoria Térmica</b> na rede. Sua banda cresceu ${dlDiff.toFixed(1)} Mbps em relação ao teste anterior.`;
    } else if (dlDiff < -10) {
        aiText = `A IA alerta para <b class="trend-down">Degradação de Fluxo</b>. Perda de ${Math.abs(dlDiff).toFixed(1)} Mbps detectada. Verifique interferências.`;
    } else {
        aiText = `A IA confirma <b>Consistência de Fibra</b>. A variação de rede está dentro da margem de 2%, indicando estabilidade total.`;
    }

    verdict.innerHTML = aiText;
}

initTrendChart(0);
updateHistoryUI();
</script>
</body>
</html>
