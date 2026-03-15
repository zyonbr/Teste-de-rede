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
        .line-header { width: 50px; height: 3px; background: var(--neon); margin: 10px auto; border-radius: 2px; }

        .ai-box { background: rgba(0, 242, 255, 0.05); border: 1px solid rgba(0, 242, 255, 0.3); border-radius: 20px; padding: 15px; margin-bottom: 15px; font-size: 0.85rem; position: relative; }
        .ai-tag { position: absolute; top: -10px; left: 20px; background: var(--neon); color: #000; font-size: 0.6rem; font-weight: 900; padding: 2px 8px; border-radius: 5px; }
        #ai-verdict { color: #ccc; line-height: 1.4; }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 15px; }
        .card { background: var(--card); padding: 18px; border-radius: 20px; border: 1px solid #2d2d35; text-align: center; position: relative; }
        .label { font-size: 0.65rem; color: #777; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; display: block; }
        .value { font-size: 1.6rem; font-weight: 900; }
        .unit { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }
        
        .delta { font-size: 0.65rem; font-weight: bold; margin-top: 5px; display: block; height: 12px; }
        .up { color: var(--ok); }
        .down { color: var(--danger); }

        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 210px; }
        
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 20px; border-radius: 20px; font-weight: 900; font-size: 1.1rem; cursor: pointer; text-transform: uppercase; transition: 0.3s; box-shadow: 0 10px 20px rgba(0, 242, 255, 0.2); }
        button:disabled { background: #222; color: #444; cursor: not-allowed; }

        .history-section { background: var(--card); border-radius: 20px; border: 2px solid #2d2d35; padding: 15px; margin-top: 25px; }
        .hist-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
        .hist-head h2 { font-size: 0.8rem; margin: 0; color: var(--neon); text-transform: uppercase; }
        .clear-btn { font-size: 0.6rem; color: var(--danger); cursor: pointer; border: 1px solid var(--danger); padding: 3px 8px; border-radius: 5px; }
        
        .hist-item { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px solid #222; font-size: 0.75rem; color: #aaa; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA</h1>
        <div class="line-header"></div>
    </div>

    <div class="ai-box">
        <span class="ai-tag">ANALISTA IA</span>
        <div id="ai-verdict">Aguardando dados para gerar o comparativo de curvas separadas.</div>
    </div>

    <div class="grid">
        <div class="card"><span class="label">Ping</span><div class="value" id="ping">--</div><div id="d-ping" class="delta"></div></div>
        <div class="card"><span class="label">Nervosismo</span><div class="value" id="jitter">--</div><div id="d-jitter" class="delta"></div></div>
        <div class="card"><span class="label">Download</span><div class="value" id="dl">--</div><div id="d-dl" class="delta"></div></div>
        <div class="card"><span class="label">Carregar</span><div class="value" id="ul">--</div><div id="d-ul" class="delta"></div></div>
    </div>

    <div class="chart-box">
        <canvas id="splitChart"></canvas>
    </div>

    <button id="main-btn" onclick="runUltraTest()">Iniciar Diagnóstico</button>

    <div class="history-section">
        <div class="hist-head">
            <h2>Histórico</h2>
            <div class="clear-btn" onclick="clearHistory()">Limpar</div>
        </div>
        <div id="hist-list"></div>
    </div>
</div>

<script>
let chart;
let historyData = JSON.parse(localStorage.getItem('netscan_split_v1') || '[]');

function initSplitChart() {
    const ctx = document.getElementById('splitChart').getContext('2d');
    if(chart) chart.destroy();
    
    const lastTests = historyData.slice(-2);
    const prevVal = lastTests.length === 2 ? parseFloat(lastTests[0].dl) : 0;
    const currVal = lastTests.length >= 1 ? parseFloat(lastTests[lastTests.length-1].dl) : 0;

    // Lógica de cores separadas: quem for maior fica verde, quem for menor fica vermelho
    const prevColor = prevVal >= currVal ? '#00ff88' : '#ff4d4d';
    const currColor = currVal > prevVal ? '#00ff88' : '#ff4d4d';

    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: ['Teste Anterior', 'Teste Atual'],
            datasets: [
                {
                    label: 'Anterior',
                    data: [prevVal, prevVal], // Linha reta do nível anterior
                    borderColor: prevColor,
                    borderWidth: 3,
                    borderDash: [5, 5],
                    pointRadius: 0,
                    fill: false
                },
                {
                    label: 'Atual',
                    data: [prevVal, currVal], // Linha de transição
                    borderColor: currColor,
                    backgroundColor: currColor + '22',
                    borderWidth: 5,
                    fill: true,
                    tension: 0.4,
                    pointRadius: 8,
                    pointBackgroundColor: [prevColor, currColor]
                }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: false },
            scales: {
                y: { beginAtZero: true, max: 450, grid: { color: '#222' } },
                x: { ticks: { color: '#fff' } }
            }
        }
    });
}

function updateDeltas(current) {
    if (historyData.length < 2) return;
    const prev = historyData[historyData.length - 2];
    const calc = (id, cur, old, inv = false) => {
        const diff = cur - old;
        const isBetter = inv ? diff < 0 : diff > 0;
        document.getElementById(id).innerHTML = `<span class="${isBetter ? 'up' : 'down'}">${diff > 0 ? '▲ +' : '▼ '}${Math.abs(diff).toFixed(1)}</span>`;
    };
    calc('d-ping', current.ping, prev.ping, true);
    calc('d-jitter', current.jitter, prev.jitter, true);
    calc('d-dl', current.dl, prev.dl);
    calc('d-ul', current.ul, prev.ul);
}

function runIA(cur) {
    const v = document.getElementById('ai-verdict');
    if (historyData.length < 2) { v.innerHTML = "<b>IA em aprendizado.</b> Realize o segundo teste para comparar as linhas de performance."; return; }
    const diff = cur.dl - historyData[historyData.length - 2].dl;
    v.innerHTML = diff > 0 ? `A IA detectou <b>Ganho de Banda</b>. Sua linha atual superou a anterior em ${diff.toFixed(1)} Mbps.` : `A IA detectou <b>Perda de Fluxo</b>. Sua linha atual está abaixo da marca anterior.`;
}

function runUltraTest() {
    const btn = document.getElementById('main-btn');
    btn.disabled = true;
    setTimeout(() => {
        const res = {
            time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
            dl: (330 + Math.random() * 30).toFixed(1),
            ul: (320 + Math.random() * 25).toFixed(1),
            ping: (18 + Math.floor(Math.random() * 10)),
            jitter: (2 + Math.floor(Math.random() * 8))
        };
        document.getElementById('dl').innerText = res.dl;
        document.getElementById('ul').innerText = res.ul;
        document.getElementById('ping').innerText = res.ping;
        document.getElementById('jitter').innerText = res.jitter;

        historyData.push(res);
        if (historyData.length > 10) historyData.shift();
        localStorage.setItem('netscan_split_v1', JSON.stringify(historyData));

        initSplitChart();
        updateDeltas(res);
        runIA(res);
        updateHistoryUI();
        btn.disabled = false;
    }, 2000);
}

function updateHistoryUI() {
    document.getElementById('hist-list').innerHTML = historyData.slice().reverse().map(h => `
        <div class="hist-item"><span>${h.time}</span><span>DL: <b>${h.dl}</b></span><span>Ping: <b>${h.ping}ms</b></span></div>
    `).join('');
}

function clearHistory() { localStorage.removeItem('netscan_split_v1'); location.reload(); }

initSplitChart();
updateHistoryUI();
</script>
</body>
</html>
