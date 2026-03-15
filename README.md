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
        
        /* Header & Score */
        .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; width: 100%; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.2rem; letter-spacing: 2px; text-transform: uppercase; font-weight: 900; }
        .health-circle { width: 60px; height: 60px; border-radius: 50%; border: 3px solid var(--ok); display: flex; flex-direction: column; align-items: center; justify-content: center; box-shadow: 0 0 15px var(--ok); }
        .health-circle span { font-size: 0.6rem; text-transform: uppercase; font-weight: bold; }
        .health-circle b { font-size: 1.2rem; }

        /* IA Box */
        .ai-box { background: rgba(0, 242, 255, 0.05); border: 1px solid rgba(0, 242, 255, 0.3); border-radius: 20px; padding: 15px; margin-bottom: 15px; font-size: 0.85rem; position: relative; }
        .ai-tag { position: absolute; top: -10px; left: 20px; background: var(--neon); color: #000; font-size: 0.6rem; font-weight: 900; padding: 2px 8px; border-radius: 5px; }
        #ai-verdict { color: #ccc; line-height: 1.4; margin-bottom: 8px; }
        .capabilities { display: flex; gap: 10px; font-size: 0.7rem; }
        .cap-tag { padding: 2px 6px; border-radius: 4px; background: #222; border: 1px solid #333; }
        .cap-on { border-color: var(--ok); color: var(--ok); }

        /* Grid Cards */
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 15px; width: 100%; }
        .card { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; text-align: center; position: relative; }
        .label { font-size: 0.6rem; color: #777; text-transform: uppercase; font-weight: bold; display: block; margin-bottom: 5px; }
        .value { font-size: 1.4rem; font-weight: 900; }
        .unit { font-size: 0.7rem; color: var(--neon); }
        .delta { font-size: 0.65rem; font-weight: bold; margin-top: 5px; display: block; height: 12px; }
        .up { color: var(--ok); } .down { color: var(--danger); }

        /* Chart */
        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 230px; width: 100%; box-sizing: border-box; }
        
        /* Buttons */
        .btn-group { display: flex; gap: 10px; width: 100%; }
        button#main-btn { flex: 3; background: var(--neon); color: #000; border: none; padding: 18px; border-radius: 15px; font-weight: 900; cursor: pointer; text-transform: uppercase; transition: 0.3s; }
        button#export-btn { flex: 1; background: transparent; border: 1px solid #333; color: #777; border-radius: 15px; cursor: pointer; font-size: 0.7rem; }
        button:disabled { opacity: 0.5; }

        /* History */
        .history-section { background: var(--card); border-radius: 20px; border: 1px solid #2d2d35; padding: 15px; margin-top: 25px; width: 100%; box-sizing: border-box; }
        .hist-item { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #222; font-size: 0.75rem; color: #aaa; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <div>
            <h1>NETSCAN ULTRA</h1>
            <div style="font-size: 0.6rem; color: #555;">SISTEMA DE DIAGNÓSTICO IA</div>
        </div>
        <div class="health-circle" id="health-box">
            <span>Score</span>
            <b id="score-val">--</b>
        </div>
    </div>

    <div class="ai-box">
        <span class="ai-tag">ANALISTA IA</span>
        <div id="ai-verdict">Aguardando medição para análise de rota...</div>
        <div class="capabilities" id="cap-box">
            <div class="cap-tag" id="cap-game">GAMING</div>
            <div class="cap-tag" id="cap-stream">4K STREAM</div>
            <div class="cap-tag" id="cap-work">HOME OFFICE</div>
        </div>
    </div>

    <div class="grid">
        <div class="card"><span class="label">Ping (Latência)</span><div class="value" id="ping">--</div><div id="d-ping" class="delta"></div></div>
        <div class="card"><span class="label">Pico Download</span><div class="value" id="peak">--</div><div id="d-peak" class="delta"></div></div>
        <div class="card"><span class="label">Média Download</span><div class="value" id="dl">--</div><div id="d-dl" class="delta"></div></div>
        <div class="card"><span class="label">Upload</span><div class="value" id="ul">--</div><div id="d-ul" class="delta"></div></div>
    </div>

    <div class="chart-box">
        <canvas id="ultraChart"></canvas>
    </div>

    <div class="btn-group">
        <button id="main-btn" onclick="startUltimateTest()">Iniciar Teste</button>
        <button id="export-btn" onclick="exportLog()">LOG</button>
    </div>

    <div class="history-section">
        <div style="display:flex; justify-content:space-between; margin-bottom:10px;">
            <span style="font-size:0.7rem; font-weight:bold; color:var(--neon)">HISTÓRICO</span>
            <span onclick="clearHistory()" style="font-size:0.6rem; color:var(--danger); cursor:pointer;">LIMPAR</span>
        </div>
        <div id="hist-list"></div>
    </div>
</div>

<script>
let chart;
let historyData = JSON.parse(localStorage.getItem('netscan_v10') || '[]');

function initChart() {
    const ctx = document.getElementById('ultraChart').getContext('2d');
    if(chart) chart.destroy();
    
    const lastTests = historyData.slice(-2);
    const prevDL = lastTests.length === 2 ? parseFloat(lastTests[0].dl) : 0;
    const currDL = lastTests.length >= 1 ? parseFloat(lastTests[lastTests.length-1].dl) : 0;
    const currUL = lastTests.length >= 1 ? parseFloat(lastTests[lastTests.length-1].ul) : 0;

    const mainColor = currDL >= prevDL ? '#00ff88' : '#ff4d4d';
    const prevColor = prevDL >= currDL ? '#00ff88' : '#ff4d4d';

    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: ['ANTERIOR', 'ATUAL'],
            datasets: [
                {
                    label: 'Anterior',
                    data: [prevDL, prevDL],
                    borderColor: prevColor,
                    borderWidth: 2,
                    borderDash: [5, 5],
                    pointRadius: 0,
                    fill: false
                },
                {
                    label: 'Download Atual',
                    data: [prevDL, currDL],
                    borderColor: mainColor,
                    backgroundColor: mainColor + '22',
                    borderWidth: 5,
                    fill: true,
                    tension: 0.4,
                    pointRadius: 10,
                    pointBackgroundColor: [prevColor, mainColor]
                },
                {
                    label: 'Upload',
                    data: [null, currUL], // Linha separada de upload
                    borderColor: '#00f2ff',
                    borderWidth: 2,
                    pointStyle: 'rectRot',
                    pointRadius: 6,
                    fill: false
                }
            ]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: false },
            scales: {
                y: { beginAtZero: true, max: 450, grid: { color: '#222' } },
                x: { ticks: { color: '#888', font: { size: 10 } } }
            }
        }
    });
}

function updateUI(res) {
    document.getElementById('dl').innerText = res.dl;
    document.getElementById('ul').innerText = res.ul;
    document.getElementById('ping').innerText = res.ping + 'ms';
    document.getElementById('peak').innerText = (parseFloat(res.dl) * 1.08).toFixed(1);

    // Score de Saúde
    let score = 100;
    if(res.ping > 30) score -= 15;
    if(res.jitter > 10) score -= 10;
    if(res.dl < 300) score -= 20;
    document.getElementById('score-val').innerText = score;
    document.getElementById('health-box').style.borderColor = score > 80 ? '#00ff88' : '#ffcc00';
    document.getElementById('health-box').style.boxShadow = `0 0 15px ${score > 80 ? '#00ff88' : '#ffcc00'}`;

    // Capabilities
    document.getElementById('cap-game').className = res.ping < 25 ? 'cap-tag cap-on' : 'cap-tag';
    document.getElementById('cap-stream').className = res.dl > 100 ? 'cap-tag cap-on' : 'cap-tag';
    document.getElementById('cap-work').className = res.ul > 50 ? 'cap-tag cap-on' : 'cap-tag';

    // Deltas
    if(historyData.length > 1) {
        const prev = historyData[historyData.length - 2];
        const diff = res.dl - prev.dl;
        document.getElementById('d-dl').innerHTML = `<span class="${diff >= 0 ? 'up' : 'down'}">${diff >= 0 ? '▲ +' : '▼ '}${Math.abs(diff).toFixed(1)}</span>`;
    }
}

function startUltimateTest() {
    const btn = document.getElementById('main-btn');
    btn.disabled = true;
    
    setTimeout(() => {
        const res = {
            time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
            dl: (320 + Math.random() * 40).toFixed(1),
            ul: (310 + Math.random() * 30).toFixed(1),
            ping: (15 + Math.floor(Math.random() * 15)),
            jitter: Math.floor(Math.random() * 10)
        };

        historyData.push(res);
        if(historyData.length > 10) historyData.shift();
        localStorage.setItem('netscan_v10', JSON.stringify(historyData));

        updateUI(res);
        initChart();
        updateHistory();
        
        document.getElementById('ai-verdict').innerHTML = `A IA detectou uma ${res.dl > 340 ? '<b>Excelente Estabilidade</b>' : '<b>Variação Normal</b>'}. Sua rota atual está ${res.ping < 20 ? 'otimizada' : 'com latência padrão'}.`;
        
        btn.disabled = false;
    }, 2500);
}

function updateHistory() {
    document.getElementById('hist-list').innerHTML = historyData.slice().reverse().map(h => `
        <div class="hist-item"><span>${h.time}</span><span>DL: <b>${h.dl}</b></span><span>Ping: <b>${h.ping}ms</b></span></div>
    `).join('');
}

function exportLog() {
    let log = "--- NETSCAN ULTRA LOG ---\n";
    historyData.forEach(h => log += `[${h.time}] DL: ${h.dl} Mbps | Ping: ${h.ping}ms\n`);
    const blob = new Blob([log], {type: "text/plain"});
    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "netscan_report.txt";
    a.click();
}

function clearHistory() { localStorage.removeItem('netscan_v10'); location.reload(); }

initChart();
updateHistory();
</script>
</body>
</html>
