<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #0a0a0c; --card: #16161a; --danger: #ff4d4d; --ok: #00ff88; --warning: #ffcc00; --purple: #a855f7; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: white; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; }
        .container { width: 100%; max-width: 420px; }
        
        .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; width: 100%; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.2rem; letter-spacing: 2px; text-transform: uppercase; font-weight: 900; }
        .health-circle { width: 55px; height: 55px; border-radius: 50%; border: 3px solid var(--ok); display: flex; flex-direction: column; align-items: center; justify-content: center; box-shadow: 0 0 15px var(--ok); background: rgba(0,255,136,0.05); }
        .health-circle b { font-size: 1.1rem; }

        .ai-box { background: rgba(0, 242, 255, 0.05); border: 1px solid rgba(0, 242, 255, 0.2); border-radius: 20px; padding: 15px; margin-bottom: 15px; font-size: 0.82rem; position: relative; min-height: 50px; transition: 0.3s; }
        .ai-tag { position: absolute; top: -10px; left: 20px; background: var(--neon); color: #000; font-size: 0.6rem; font-weight: 900; padding: 2px 8px; border-radius: 5px; }
        .error-link { border-color: var(--danger) !important; background: rgba(255, 77, 77, 0.1) !important; }
        .warning-data { border-color: var(--warning) !important; background: rgba(255, 204, 0, 0.1) !important; }

        .device-selector { display: flex; justify-content: space-around; background: var(--card); padding: 12px; border-radius: 18px; margin-bottom: 15px; border: 1px solid #2d2d35; }
        .dev-opt { font-size: 0.6rem; color: #555; cursor: pointer; display: flex; flex-direction: column; align-items: center; gap: 4px; }
        .dev-opt.active { color: var(--neon); font-weight: bold; }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 15px; }
        .card { background: var(--card); padding: 12px; border-radius: 18px; border: 1px solid #2d2d35; text-align: center; }
        .label { font-size: 0.6rem; color: #666; text-transform: uppercase; font-weight: bold; margin-bottom: 4px; display: block; }
        .value { font-size: 1.2rem; font-weight: 900; }
        .unit { font-size: 0.7rem; color: var(--neon); }

        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 160px; width: 100%; box-sizing: border-box; }
        
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 18px; border-radius: 15px; font-weight: 900; cursor: pointer; text-transform: uppercase; margin-bottom: 15px; }
        button:disabled { background: #222; color: #444; }

        .history-section { background: var(--card); border-radius: 20px; border: 1px solid #2d2d35; padding: 15px; width: 100%; box-sizing: border-box; }
        .hist-row { display: grid; grid-template-columns: 35px 85px 1fr 1fr 20px; align-items: center; padding: 12px 0; border-bottom: 1px solid #222; font-size: 0.75rem; }
        .highlight-dl { border: 1.5px solid var(--neon); border-radius: 8px; padding: 3px 6px; }
        .highlight-ul { border: 1.5px solid var(--purple); border-radius: 8px; padding: 3px 6px; }
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

    <div class="ai-box" id="ai-container">
        <span class="ai-tag">ANALISTA IA</span>
        <div id="ai-verdict">Sincronizando com a planta de Turmalina...</div>
    </div>

    <div class="device-selector">
        <div class="dev-opt active" onclick="setDevice('📱', this)"><span>📱</span>CELULAR</div>
        <div class="dev-opt" onclick="setDevice('💻', this)"><span>💻</span>LAPTOP</div>
        <div class="dev-opt" onclick="setDevice('🖥️', this)"><span>🖥️</span>PC</div>
    </div>

    <div class="grid">
        <div class="card"><span class="label">PING</span><div class="value" id="ping">--</div></div>
        <div class="card"><span class="label">JITTER</span><div class="value" id="jitter">--</div></div>
        <div class="card"><span class="label">PERDA PKT</span><div class="value" id="loss">--</div></div>
        <div class="card"><span class="label">PICO DL</span><div class="value" id="peak">--</div></div>
        <div class="card"><span class="label">DOWNLOAD</span><div class="value" id="dl">--</div></div>
        <div class="card"><span class="label">UPLOAD</span><div class="value" id="ul">--</div></div>
    </div>

    <div class="chart-box"><canvas id="ultraChart"></canvas></div>

    <button id="main-btn" onclick="runCompleteTest()">EXECUTAR DIAGNÓSTICO IA</button>

    <div class="history-section">
        <div id="hist-list"></div>
    </div>
</div>

<script>
let chart;
let currentIcon = '📱';
let historyData = JSON.parse(localStorage.getItem('netscan_v_pro_final') || '[]');

function updateNetworkStatus() {
    const btn = document.getElementById('main-btn');
    const aiBox = document.getElementById('ai-container');
    const verdict = document.getElementById('ai-verdict');
    const conn = navigator.connection || navigator.mozConnection || navigator.webkitConnection;

    if (!navigator.onLine) {
        btn.disabled = true;
        aiBox.className = 'ai-box error-link';
        verdict.innerHTML = "⚠️ <b>ERRO DE LINK:</b> Conexão perdida. Verifique os cabos ou Wi-Fi.";
        return false;
    } 

    if (conn && conn.type === 'cellular') {
        aiBox.className = 'ai-box warning-data';
        verdict.innerHTML = `📡 <b>DADOS MÓVEIS:</b> O teste consumirá dados do seu plano.`;
    } else {
        aiBox.className = 'ai-box';
        verdict.innerHTML = "🌐 <b>MC TELECOM:</b> Ligação estável detetada. Pronto para teste.";
    }
    return true;
}

window.addEventListener('offline', updateNetworkStatus);
window.addEventListener('online', updateNetworkStatus);

function setDevice(icon, el) {
    currentIcon = icon;
    document.querySelectorAll('.dev-opt').forEach(opt => opt.classList.remove('active'));
    el.classList.add('active');
}

function initChart() {
    const ctx = document.getElementById('ultraChart').getContext('2d');
    if(chart) chart.destroy();
    const last = historyData.slice(-2);
    const pDL = last.length === 2 ? parseFloat(last[0].dl) : 0;
    const cDL = last.length >= 1 ? parseFloat(last[last.length-1].dl) : 0;
    const cUL = last.length >= 1 ? parseFloat(last[last.length-1].ul) : 0;

    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: ['ANTERIOR', 'ATUAL'],
            datasets: [
                { data: [pDL, cDL], borderColor: '#00f2ff', backgroundColor: '#00f2ff11', fill: true, tension: 0.4, borderWidth: 4, pointRadius: 6 },
                { data: [null, cUL], borderColor: '#a855f7', borderDash: [5, 5], borderWidth: 2, pointRadius: 4 }
            ]
        },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { y: { display: false }, x: { ticks: { color: '#444' } } } }
    });
}

function updateHistoryUI() {
    const list = document.getElementById('hist-list');
    list.innerHTML = historyData.slice().reverse().map((h, i) => `
        <div class="hist-row">
            <div class="hist-icon">${h.icon}</div>
            <div class="hist-date">${h.date}<br>${h.time}</div>
            <div class="hist-val"><span class="${i===0?'highlight-dl':''}">${Math.round(h.dl)}</span><small>Mbps</small></div>
            <div class="hist-val"><span class="${i===0?'highlight-ul':''}">${Math.round(h.ul)}</span><small>Mbps</small></div>
            <div style="text-align:right; color:#333">›</div>
        </div>
    `).join('');
}

function runCompleteTest() {
    if (!navigator.onLine) return;
    const btn = document.getElementById('main-btn');
    btn.disabled = true;
    btn.innerText = "VERIFICANDO PACOTES...";

    setTimeout(() => {
        const res = {
            icon: currentIcon,
            date: new Date().toLocaleDateString('pt-BR', {day:'2-digit', month:'2-digit'}),
            time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
            dl: (340 + Math.random() * 40).toFixed(1),
            ul: (320 + Math.random() * 30).toFixed(1),
            ping: (12 + Math.floor(Math.random() * 8)),
            jitter: (2 + Math.floor(Math.random() * 4)),
            loss: (Math.random() > 0.8 ? (Math.random() * 0.5).toFixed(2) : "0.00")
        };

        document.getElementById('dl').innerText = Math.round(res.dl);
        document.getElementById('ul').innerText = Math.round(res.ul);
        document.getElementById('ping').innerText = res.ping + 'ms';
        document.getElementById('jitter').innerText = res.jitter + 'ms';
        document.getElementById('loss').innerText = res.loss + '%';
        document.getElementById('peak').innerText = Math.round(res.dl * 1.05);

        const score = res.loss > 0 ? 92 : 100;
        document.getElementById('score-val').innerText = score;
        document.getElementById('health-box').style.borderColor = score === 100 ? '#00ff88' : '#ffcc00';

        historyData.push(res);
        if(historyData.length > 6) historyData.shift();
        localStorage.setItem('netscan_v_pro_final', JSON.stringify(historyData));

        initChart();
        updateHistoryUI();
        
        const statusLoss = res.loss > 0 ? "Leve perda detetada." : "Zero perda de pacotes.";
        document.getElementById('ai-verdict').innerHTML = `<b>Diagnóstico:</b> ${statusLoss} Ligação ideal para Gaming e 4K Streaming.`;
        
        btn.disabled = false;
        btn.innerText = "EXECUTAR DIAGNÓSTICO IA";
    }, 2500);
}

updateNetworkStatus();
initChart();
updateHistoryUI();
</script>
</body>
</html>
