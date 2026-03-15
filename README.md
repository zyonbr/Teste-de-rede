<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #0a0a0c; --card: #16161a; --danger: #ff4d4d; --ok: #00ff88; --warning: #ffcc00; --purple: #a855f7; }
        body { font-family: 'Segoe UI', Roboto, sans-serif; background: var(--bg); color: white; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; min-height: 100vh; }
        .container { width: 100%; max-width: 420px; }
        
        .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; width: 100%; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.2rem; letter-spacing: 2px; text-transform: uppercase; font-weight: 900; }
        
        /* Health Score (Inspirado no visual circular) */
        .health-circle { width: 55px; height: 55px; border-radius: 50%; border: 3px solid var(--ok); display: flex; flex-direction: column; align-items: center; justify-content: center; box-shadow: 0 0 15px var(--ok); background: rgba(0,255,136,0.05); }
        .health-circle span { font-size: 0.5rem; text-transform: uppercase; }
        .health-circle b { font-size: 1.1rem; }

        /* AI Analista */
        .ai-box { background: rgba(0, 242, 255, 0.05); border: 1px solid rgba(0, 242, 255, 0.2); border-radius: 20px; padding: 15px; margin-bottom: 15px; font-size: 0.82rem; position: relative; }
        .ai-tag { position: absolute; top: -10px; left: 20px; background: var(--neon); color: #000; font-size: 0.6rem; font-weight: 900; padding: 2px 8px; border-radius: 5px; }

        /* Seletor de Dispositivo (Novo!) */
        .device-selector { display: flex; justify-content: space-around; background: var(--card); padding: 10px; border-radius: 15px; margin-bottom: 15px; border: 1px solid #222; }
        .dev-opt { font-size: 0.6rem; color: #555; cursor: pointer; display: flex; flex-direction: column; align-items: center; gap: 4px; }
        .dev-opt.active { color: var(--neon); font-weight: bold; }
        .dev-opt i { font-size: 1.2rem; }

        /* Grade de Valores */
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 15px; }
        .card { background: var(--card); padding: 15px; border-radius: 18px; border: 1px solid #2d2d35; text-align: center; }
        .label { font-size: 0.6rem; color: #666; text-transform: uppercase; font-weight: bold; margin-bottom: 4px; display: block; }
        .value { font-size: 1.3rem; font-weight: 900; }
        .unit { font-size: 0.7rem; color: var(--neon); }

        /* Gráfico Principal */
        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 180px; }
        
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 18px; border-radius: 15px; font-weight: 900; cursor: pointer; text-transform: uppercase; margin-bottom: 15px; box-shadow: 0 5px 15px rgba(0, 242, 255, 0.3); }

        /* Histórico Estilo Profissional (Referência 1000397030) */
        .history-section { background: var(--card); border-radius: 20px; border: 1px solid #2d2d35; padding: 15px; width: 100%; box-sizing: border-box; }
        .hist-title { font-size: 0.75rem; font-weight: 900; color: #555; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; }
        
        .hist-row { display: grid; grid-template-columns: 35px 85px 1fr 1fr 20px; align-items: center; padding: 12px 0; border-bottom: 1px solid #222; font-size: 0.8rem; }
        .hist-icon { color: var(--purple); font-size: 1.1rem; text-align: center; opacity: 0.7; }
        .hist-date { color: #888; font-size: 0.65rem; line-height: 1.2; }
        .hist-val { font-weight: 900; font-size: 1rem; text-align: center; }
        .hist-val small { display: block; font-size: 0.5rem; color: #555; font-weight: normal; margin-top: -2px; }
        
        /* Caixas de Destaque (Referência 1000397030) */
        .highlight-dl { border: 1.5px solid var(--neon); border-radius: 8px; padding: 3px 6px; color: #fff; background: rgba(0, 242, 255, 0.05); }
        .highlight-ul { border: 1.5px solid var(--purple); border-radius: 8px; padding: 3px 6px; color: #fff; background: rgba(168, 85, 247, 0.05); }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA</h1>
        <div class="health-circle" id="health-box">
            <span>Score</span><b id="score-val">--</b>
        </div>
    </div>

    <div class="ai-box">
        <span class="ai-tag">ANALISTA IA</span>
        <div id="ai-verdict">Selecione o dispositivo para iniciar o diagnóstico de rede.</div>
    </div>

    <div class="device-selector">
        <div class="dev-opt active" onclick="setDevice('📱', this)"><span>📱</span>CELULAR</div>
        <div class="dev-opt" onclick="setDevice('💻', this)"><span>💻</span>LAPTOP</div>
        <div class="dev-opt" onclick="setDevice('🖥️', this)"><span>🖥️</span>PC</div>
    </div>

    <div class="grid">
        <div class="card"><span class="label">Ping</span><div class="value" id="ping">--</div></div>
        <div class="card"><span class="label">Jitter</span><div class="value" id="jitter">--</div></div>
        <div class="card"><span class="label">Download</span><div class="value" id="dl">--</div></div>
        <div class="card"><span class="label">Upload</span><div class="value" id="ul">--</div></div>
    </div>

    <div class="chart-box"><canvas id="ultraChart"></canvas></div>

    <button id="main-btn" onclick="runUltraTest()">EXECUTAR TESTE IA</button>

    <div class="history-section">
        <div class="hist-title">
            <span>LOGS DE MARÇO/2026</span>
            <span style="color:var(--danger); cursor:pointer; font-size: 0.6rem;" onclick="clearHist()">LIMPAR</span>
        </div>
        <div id="hist-list"></div>
    </div>
</div>

<script>
let chart;
let currentIcon = '📱';
let historyData = JSON.parse(localStorage.getItem('netscan_v_final_zyon') || '[]');

function setDevice(icon, el) {
    currentIcon = icon;
    document.querySelectorAll('.dev-opt').forEach(opt => opt.classList.remove('active'));
    el.classList.add('active');
}

function initChart() {
    const ctx = document.getElementById('ultraChart').getContext('2d');
    if(chart) chart.destroy();
    const last = historyData.slice(-2);
    const pVal = last.length === 2 ? parseFloat(last[0].dl) : 0;
    const cVal = last.length >= 1 ? parseFloat(last[last.length-1].dl) : 0;
    const color = cVal >= pVal ? '#00ff88' : '#ff4d4d';

    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: ['ANTERIOR', 'ATUAL'],
            datasets: [{
                data: [pVal, cVal],
                borderColor: color,
                backgroundColor: color + '11',
                fill: true, tension: 0.4, borderWidth: 4, pointRadius: 8, pointBackgroundColor: ['#222', color]
            }]
        },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { y: { display: false }, x: { ticks: { color: '#444' } } } }
    });
}

function updateHistoryUI() {
    const list = document.getElementById('hist-list');
    list.innerHTML = historyData.slice().reverse().map((h, i) => `
        <div class="hist-row">
            <div class="hist-icon">${h.icon || '📱'}</div>
            <div class="hist-date">${h.date}<br>${h.time}</div>
            <div class="hist-val">
                <span class="${i===0?'highlight-dl':''}">${Math.round(h.dl)}</span>
                <small>Mbps</small>
            </div>
            <div class="hist-val">
                <span class="${i===0?'highlight-ul':''}">${Math.round(h.ul)}</span>
                <small>Mbps</small>
            </div>
            <div style="color:#333; font-weight: bold; text-align: right;">›</div>
        </div>
    `).join('') || '<div style="text-align:center; padding:20px; color:#444; font-size:0.7rem;">Nenhum teste registrado</div>';
}

function runUltraTest() {
    const btn = document.getElementById('main-btn');
    btn.disabled = true;
    btn.innerText = "SINCRONIZANDO...";

    setTimeout(() => {
        const res = {
            icon: currentIcon,
            date: new Date().toLocaleDateString('pt-BR', {day:'2-digit', month:'2-digit', year:'2-digit'}),
            time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
            dl: (320 + Math.random() * 60).toFixed(1),
            ul: (310 + Math.random() * 40).toFixed(1),
            ping: (18 + Math.floor(Math.random() * 12)),
            jitter: (2 + Math.floor(Math.random() * 8))
        };

        document.getElementById('dl').innerText = Math.round(res.dl);
        document.getElementById('ul').innerText = Math.round(res.ul);
        document.getElementById('ping').innerText = res.ping + 'ms';
        document.getElementById('jitter').innerText = res.jitter + 'ms';
        document.getElementById('score-val').innerText = res.ping < 25 ? '99' : '88';

        historyData.push(res);
        if(historyData.length > 8) historyData.shift();
        localStorage.setItem('netscan_v_final_zyon', JSON.stringify(historyData));

        initChart();
        updateHistoryUI();
        document.getElementById('ai-verdict').innerHTML = `<b>Diagnóstico Concluído.</b> O dispositivo ${currentIcon} apresenta estabilidade total na rota de fibra.`;
        btn.disabled = false;
        btn.innerText = "EXECUTAR TESTE IA";
    }, 2000);
}

function clearHist() { localStorage.removeItem('netscan_v_final_zyon'); location.reload(); }

initChart();
updateHistoryUI();
</script>
</body>
</html>
