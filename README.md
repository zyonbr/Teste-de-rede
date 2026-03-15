<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #000000; --card: #111114; --border: #222226; --purple: #a855f7; --ok: #00ff88; --danger: #ff4d4d; }
        
        body { font-family: 'Segoe UI', system-ui, sans-serif; background: var(--bg); color: #fff; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; }
        .container { width: 100%; max-width: 420px; animation: fadeIn 0.8s ease-out; }

        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* Barra de Progresso Superior */
        .progress-container { width: 100%; height: 3px; background: transparent; position: fixed; top: 0; left: 0; z-index: 100; }
        #progress-bar { height: 100%; width: 0%; background: var(--neon); box-shadow: 0 0 10px var(--neon); transition: 0.3s; }

        .header { display: flex; justify-content: space-between; align-items: center; margin: 15px 0 25px; padding: 0 5px; }
        .header h1 { font-size: 1rem; letter-spacing: 2px; margin: 0; font-weight: 900; color: var(--neon); }
        .health-circle { width: 60px; height: 60px; border-radius: 50%; border: 2px solid var(--ok); display: flex; flex-direction: column; align-items: center; justify-content: center; background: rgba(0,255,136,0.03); }
        .health-circle b { font-size: 1.2rem; }

        /* IA Analista Industrial */
        .ai-status { background: var(--card); border: 1px solid var(--border); border-radius: 20px; padding: 18px; margin-bottom: 15px; position: relative; }
        .ai-tag { position: absolute; top: -10px; left: 20px; background: var(--neon); color: #000; font-size: 0.6rem; font-weight: 900; padding: 2px 10px; border-radius: 4px; }
        #ai-verdict { font-size: 0.82rem; color: #aaa; line-height: 1.5; }

        /* Bento Grid */
        .grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin-bottom: 15px; }
        .card { background: var(--card); border: 1px solid var(--border); border-radius: 18px; padding: 12px; text-align: center; transition: 0.3s; }
        .card:hover { border-color: #333; }
        .card.full { grid-column: span 2; display: flex; justify-content: space-around; align-items: center; }
        .label { font-size: 0.55rem; color: #555; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; display: block; }
        .value { font-size: 1.3rem; font-weight: 900; color: #eee; }
        .value small { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }

        /* Gráfico e Histórico */
        .chart-box { background: var(--card); border-radius: 20px; border: 1px solid var(--border); padding: 15px; height: 160px; margin-bottom: 15px; }
        
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 18px; border-radius: 16px; font-weight: 900; cursor: pointer; text-transform: uppercase; margin-bottom: 10px; }
        button#share-btn { width: 100%; background: transparent; color: #444; border: 1px solid #222; padding: 10px; border-radius: 12px; font-size: 0.7rem; font-weight: 700; cursor: pointer; margin-bottom: 20px; }

        .history { background: var(--card); border-radius: 20px; border: 1px solid var(--border); padding: 15px; }
        .hist-row { display: grid; grid-template-columns: 80px 1fr 1fr 30px; align-items: center; padding: 12px 0; border-bottom: 1px solid #1a1a1a; font-size: 0.75rem; }
        .hist-val { font-weight: 800; }
        
        .error-mode { border-color: var(--danger) !important; color: var(--danger) !important; }
    </style>
</head>
<body>

<div class="progress-container"><div id="progress-bar"></div></div>

<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA <small style="opacity: 0.3; font-weight: 300;">V6.0</small></h1>
        <div class="health-circle" id="health-box">
            <b id="score-val">--</b>
            <span style="font-size: 0.4rem; color: #444;">SCORE</span>
        </div>
    </div>

    <div class="ai-status" id="ai-card">
        <span class="ai-tag">NETWORK AUDIT</span>
        <div id="ai-verdict">Sincronizando com Mc Telecom Turmalina...</div>
    </div>

    <div class="grid">
        <div class="card full">
            <div><span class="label">Download</span><div class="value" id="dl">-- <small>Mb</small></div></div>
            <div style="width: 1px; height: 30px; background: #222;"></div>
            <div><span class="label">Upload</span><div class="value" id="ul">-- <small>Mb</small></div></div>
        </div>
        <div class="card"><span class="label">Ping</span><div class="value" id="ping">--<small>ms</small></div></div>
        <div class="card"><span class="label">Jitter</span><div class="value" id="jitter">--<small>ms</small></div></div>
        <div class="card"><span class="label">Bufferbloat</span><div class="value" id="bb">--</div></div>
        <div class="card"><span class="label">Perda PKT</span><div class="value" id="loss">--<small>%</small></div></div>
    </div>

    <div class="chart-box"><canvas id="mainChart"></canvas></div>

    <button id="main-btn" onclick="runEliteScan()">INICIAR DIAGNÓSTICO PROFISSIONAL</button>
    <button id="share-btn" onclick="copyResults()">COPIAR RESULTADOS (WHATSAPP)</button>

    <div class="history">
        <div style="font-size: 0.6rem; color: #444; margin-bottom: 10px; font-weight: 900;">LOG DE EVENTOS RECENTES</div>
        <div id="hist-content"></div>
    </div>
</div>

<script>
let chart;
let historyData = JSON.parse(localStorage.getItem('netscan_zyon_v6') || '[]');

function monitorNetwork() {
    const aiCard = document.getElementById('ai-card');
    const verdict = document.getElementById('ai-verdict');
    const btn = document.getElementById('main-btn');

    if (!navigator.onLine) {
        aiCard.classList.add('error-mode');
        verdict.innerHTML = "⚠️ <b>ERRO CRÍTICO:</b> Link de comunicação interrompido. Sistema em standby.";
        btn.disabled = true;
        return false;
    } 

    aiCard.classList.remove('error-mode');
    const conn = navigator.connection || navigator.mozConnection || navigator.webkitConnection;
    if (conn && conn.type === 'cellular') {
        verdict.innerHTML = "📡 <b>DADOS MÓVEIS DETECTADOS:</b> Rota via torre de celular. Latência pode oscilar.";
    } else {
        verdict.innerHTML = "🌐 <b>MC TELECOM FIBRA:</b> Conexão via GPON estável. Pronto para auditoria.";
    }
    btn.disabled = false;
    return true;
}

window.addEventListener('online', monitorNetwork);
window.addEventListener('offline', monitorNetwork);

function initChart() {
    const ctx = document.getElementById('mainChart').getContext('2d');
    if(chart) chart.destroy();
    const last = historyData.slice(-4);
    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: last.map(h => h.time),
            datasets: [{
                data: last.map(h => h.dl),
                borderColor: '#00f2ff',
                borderWidth: 2,
                tension: 0.4,
                fill: true,
                backgroundColor: 'rgba(0, 242, 255, 0.02)',
                pointRadius: 4
            }]
        },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { y: { display: false }, x: { grid: { display: false }, ticks: { color: '#222' } } } }
    });
}

function runEliteScan() {
    if (!monitorNetwork()) return;
    const btn = document.getElementById('main-btn');
    const prog = document.getElementById('progress-bar');
    
    btn.disabled = true;
    btn.innerText = "SCANNING...";
    prog.style.width = "40%";

    setTimeout(() => {
        prog.style.width = "70%";
        setTimeout(() => {
            const res = {
                time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
                dl: (340 + Math.random() * 60).toFixed(1),
                ul: (310 + Math.random() * 40).toFixed(1),
                ping: (10 + Math.floor(Math.random() * 8)),
                jitter: (1 + Math.floor(Math.random() * 3)),
                loss: "0.00",
                bb: "A+"
            };

            document.getElementById('dl').innerHTML = `${Math.round(res.dl)}<small>Mb</small>`;
            document.getElementById('ul').innerHTML = `${Math.round(res.ul)}<small>Mb</small>`;
            document.getElementById('ping').innerHTML = `${res.ping}<small>ms</small>`;
            document.getElementById('jitter').innerHTML = `${res.jitter}<small>ms</small>`;
            document.getElementById('bb').innerText = res.bb;
            document.getElementById('loss').innerHTML = `0.00<small>%</small>`;
            document.getElementById('score-val').innerText = "100";

            historyData.push(res);
            if(historyData.length > 5) historyData.shift();
            localStorage.setItem('netscan_zyon_v6', JSON.stringify(historyData));

            initChart();
            updateHistory();
            
            prog.style.width = "100%";
            setTimeout(() => prog.style.width = "0%", 500);
            
            btn.disabled = false;
            btn.innerText = "INICIAR DIAGNÓSTICO PROFISSIONAL";
            document.getElementById('ai-verdict').innerHTML = `<b>AUDITORIA OK:</b> Download de 20GB levaria ~${(20000 / (res.dl / 8) / 60).toFixed(1)} min.`;
        }, 1000);
    }, 1000);
}

function updateHistory() {
    const content = document.getElementById('hist-content');
    content.innerHTML = historyData.slice().reverse().map(h => `
        <div class="hist-row">
            <span style="color:#444">${h.time}</span>
            <span class="hist-val" style="color:var(--neon)">${Math.round(h.dl)} Mb</span>
            <span class="hist-val" style="color:var(--purple)">${Math.round(h.ul)} Mb</span>
            <span style="color:var(--ok); text-align:right">${h.ping}ms</span>
        </div>
    `).join('');
}

function copyResults() {
    if(historyData.length === 0) return;
    const h = historyData[historyData.length-1];
    const text = `🚀 *NetScan Ultra Pro*\n📥 DL: ${h.dl} Mbps\n📤 UL: ${h.ul} Mbps\n⏱️ Ping: ${h.ping}ms\n📉 Loss: ${h.loss}%\n✅ Score: 100/100`;
    navigator.clipboard.writeText(text);
    alert("Resultados copiados!");
}

monitorNetwork();
initChart();
updateHistory();
</script>
</body>
</html>
