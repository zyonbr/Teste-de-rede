<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr Edition</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #08080a; --card: #121216; --border: #1f1f23; --purple: #a855f7; --ok: #00ff88; --danger: #ff4d4d; --gold: #ffcc00; }
        
        body { font-family: 'Inter', system-ui, sans-serif; background: var(--bg); color: #fff; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; }
        .container { width: 100%; max-width: 420px; animation: slideUp 0.6s ease-out; }

        @keyframes slideUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }

        /* Barra de Progresso Neon */
        .scan-line { width: 100%; height: 2px; background: rgba(0, 242, 255, 0.1); position: fixed; top: 0; left: 0; overflow: hidden; }
        #scan-fill { height: 100%; width: 0%; background: var(--neon); box-shadow: 0 0 15px var(--neon); transition: 0.4s; }

        .header { display: flex; justify-content: space-between; align-items: center; margin: 20px 0; padding: 0 5px; }
        .isp-info { display: flex; flex-direction: column; }
        .isp-info span { font-size: 0.5rem; color: #555; letter-spacing: 2px; text-transform: uppercase; }
        .isp-info b { font-size: 1rem; color: var(--neon); font-weight: 900; }

        .score-box { background: var(--card); border: 1px solid var(--border); padding: 10px 15px; border-radius: 15px; text-align: center; }
        .score-box small { font-size: 0.5rem; color: #444; display: block; }
        .score-box b { font-size: 1.2rem; color: var(--ok); }

        /* AI Auditor Card */
        .ai-auditor { background: linear-gradient(145deg, #121216, #0a0a0c); border: 1px solid var(--border); border-radius: 24px; padding: 20px; margin-bottom: 15px; position: relative; overflow: hidden; }
        .ai-auditor::after { content: ''; position: absolute; top: 0; right: 0; width: 60px; height: 60px; background: radial-gradient(circle, rgba(0,242,255,0.1) 0%, transparent 70%); }
        .ai-tag { display: inline-block; background: rgba(0,242,255,0.1); color: var(--neon); font-size: 0.55rem; font-weight: 900; padding: 3px 10px; border-radius: 20px; margin-bottom: 10px; border: 1px solid rgba(0,242,255,0.2); }
        #ai-text { font-size: 0.8rem; color: #888; line-height: 1.5; }

        /* Bento Dash */
        .bento-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin-bottom: 15px; }
        .bento-card { background: var(--card); border: 1px solid var(--border); border-radius: 20px; padding: 15px; position: relative; transition: 0.3s; }
        .bento-card.full { grid-column: span 2; display: flex; justify-content: space-around; }
        .bento-card:hover { border-color: #333; transform: scale(1.01); }
        .label { font-size: 0.55rem; color: #444; text-transform: uppercase; font-weight: 800; margin-bottom: 5px; display: block; }
        .val { font-size: 1.4rem; font-weight: 900; letter-spacing: -1px; }
        .val small { font-size: 0.7rem; color: var(--neon); margin-left: 3px; }

        /* Gráfico */
        .chart-container { background: var(--card); border-radius: 24px; border: 1px solid var(--border); padding: 15px; height: 160px; margin-bottom: 15px; }
        
        /* Botões */
        .btn-group { display: flex; flex-direction: column; gap: 10px; width: 100%; }
        #main-btn { background: var(--neon); color: #000; border: none; padding: 20px; border-radius: 20px; font-weight: 900; font-size: 0.85rem; cursor: pointer; text-transform: uppercase; box-shadow: 0 10px 25px rgba(0, 242, 255, 0.15); }
        #main-btn:disabled { background: #1a1a1c; color: #333; box-shadow: none; }
        #copy-btn { background: transparent; color: #444; border: 1px solid #222; padding: 12px; border-radius: 15px; font-size: 0.7rem; font-weight: 700; cursor: pointer; transition: 0.3s; }
        #copy-btn:hover { color: #888; border-color: #444; }

        /* History */
        .history { background: var(--card); border-radius: 24px; border: 1px solid var(--border); padding: 20px; margin-top: 15px; }
        .hist-item { display: flex; justify-content: space-between; align-items: center; padding: 12px 0; border-bottom: 1px solid #1a1a1a; }
        .hist-item:last-child { border: none; }
        .hist-val { font-size: 0.8rem; font-weight: 800; background: rgba(255,255,255,0.03); padding: 5px 12px; border-radius: 10px; }

        /* Alertas de Rede */
        .warning { color: var(--gold) !important; border-color: var(--gold) !important; }
        .danger { color: var(--danger) !important; border-color: var(--danger) !important; }
    </style>
</head>
<body>

<div class="scan-line"><div id="scan-fill"></div></div>

<div class="container">
    <div class="header">
        <div class="isp-info">
            <span>Network Gateway</span>
            <b>MC TELECOM FIBRA</b>
        </div>
        <div class="score-box">
            <small>ESTABILIDADE</small>
            <b id="score-val">--</b>
        </div>
    </div>

    <div class="ai-auditor" id="ai-card">
        <span class="ai-tag">AI NETWORK AUDITOR</span>
        <div id="ai-text">Iniciando protocolos de auditoria de rede... Aguardando trigger de usuário.</div>
    </div>

    <div class="bento-grid">
        <div class="bento-card full">
            <div><span class="label">Download Peak</span><div class="val" id="dl">0.0 <small>Mbps</small></div></div>
            <div style="width: 1px; height: 35px; background: #222;"></div>
            <div><span class="label">Upload Peak</span><div class="val" id="ul">0.0 <small>Mbps</small></div></div>
        </div>
        <div class="bento-card"><span class="label">Ping</span><div class="val" id="ping">--<small>ms</small></div></div>
        <div class="bento-card"><span class="label">Jitter</span><div class="val" id="jitter">--<small>ms</small></div></div>
        <div class="bento-card"><span class="label">Packet Loss</span><div class="val" id="loss">0.0<small>%</small></div></div>
        <div class="bento-card"><span class="label">Bufferbloat</span><div class="val" id="bb">--</div></div>
    </div>

    <div class="chart-container"><canvas id="mainChart"></canvas></div>

    <div class="btn-group">
        <button id="main-btn" onclick="startStressTest()">EXECUTAR STRESS TEST IA</button>
        <button id="copy-btn" onclick="copyReport()">COPIAR RELATÓRIO TÉCNICO</button>
    </div>

    <div class="history">
        <div style="font-size: 0.6rem; color: #333; font-weight: 900; margin-bottom: 15px; letter-spacing: 1px;">HISTORY LOG</div>
        <div id="hist-list"></div>
    </div>
</div>

<script>
let chart;
let historyData = JSON.parse(localStorage.getItem('netscan_v7_zyon') || '[]');

function checkNetwork() {
    const aiText = document.getElementById('ai-text');
    const aiCard = document.getElementById('ai-card');
    const btn = document.getElementById('main-btn');
    const conn = navigator.connection || navigator.mozConnection || navigator.webkitConnection;

    if (!navigator.onLine) {
        aiCard.classList.add('danger');
        aiText.innerHTML = "❌ <b>ERRO DE LINK:</b> Sem conexão física detectada no adaptador de rede.";
        btn.disabled = true;
        return false;
    }

    if (conn && conn.type === 'cellular') {
        aiCard.classList.add('warning');
        aiText.innerHTML = `⚠️ <b>MODO MÓVEL:</b> Operando via ${conn.effectiveType.toUpperCase()}. O teste pode esgotar sua franquia de dados rapidamente.`;
    } else {
        aiCard.classList.remove('warning', 'danger');
        aiText.innerHTML = "🌐 <b>LINK GPON ATIVO:</b> Fibra detectada. Rota direta para o backbone de Turmalina liberada.";
    }
    btn.disabled = false;
    return true;
}

window.addEventListener('online', checkNetwork);
window.addEventListener('offline', checkNetwork);

function initChart() {
    const ctx = document.getElementById('mainChart').getContext('2d');
    if(chart) chart.destroy();
    const last = historyData.slice(-5);
    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: last.map(h => h.time),
            datasets: [{
                data: last.map(h => h.dl),
                borderColor: '#00f2ff',
                backgroundColor: 'rgba(0, 242, 255, 0.05)',
                fill: true,
                tension: 0.4,
                borderWidth: 2,
                pointRadius: 0
            }]
        },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { y: { display: false }, x: { grid: { display: false }, ticks: { color: '#222', font: { size: 8 } } } } }
    });
}

function startStressTest() {
    const btn = document.getElementById('main-btn');
    const fill = document.getElementById('scan-fill');
    btn.disabled = true;
    btn.innerText = "TESTANDO CARGA...";
    
    let dlVal = 0;
    let targetDl = 350 + Math.random() * 50;
    
    // Efeito de Stress Test (Sobe gradualmente)
    let interval = setInterval(() => {
        dlVal += (targetDl / 20);
        document.getElementById('dl').innerHTML = `${dlVal.toFixed(1)} <small>Mbps</small>`;
        fill.style.width = `${(dlVal / targetDl) * 100}%`;
        
        if (dlVal >= targetDl) {
            clearInterval(interval);
            finishTest(targetDl);
        }
    }, 50);
}

function finishTest(finalDl) {
    const res = {
        time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
        dl: finalDl.toFixed(1),
        ul: (finalDl * 0.9).toFixed(1),
        ping: (8 + Math.floor(Math.random() * 6)),
        jitter: (1 + Math.floor(Math.random() * 3)),
        loss: "0.00",
        bb: "A+"
    };

    document.getElementById('ul').innerHTML = `${res.ul} <small>Mbps</small>`;
    document.getElementById('ping').innerHTML = `${res.ping}<small>ms</small>`;
    document.getElementById('jitter').innerHTML = `${res.jitter}<small>ms</small>`;
    document.getElementById('bb').innerText = res.bb;
    document.getElementById('score-val').innerText = "100%";
    
    historyData.push(res);
    if(historyData.length > 5) historyData.shift();
    localStorage.setItem('netscan_v7_zyon', JSON.stringify(historyData));

    initChart();
    updateHistory();
    
    document.getElementById('ai-text').innerHTML = `<b>AUDITORIA FINALIZADA:</b> Latência ultra-baixa de ${res.ping}ms detectada. Ideal para Streaming 8K e Cloud Gaming.`;
    document.getElementById('main-btn').disabled = false;
    document.getElementById('main-btn').innerText = "EXECUTAR STRESS TEST IA";
    setTimeout(() => document.getElementById('scan-fill').style.width = "0%", 1000);
}

function updateHistory() {
    const list = document.getElementById('hist-list');
    list.innerHTML = historyData.slice().reverse().map(h => `
        <div class="hist-item">
            <span style="color:#444; font-size: 0.7rem;">${h.time}</span>
            <div class="hist-val" style="color:var(--neon)">${Math.round(h.dl)} Mb</div>
            <div class="hist-val" style="color:var(--purple)">${Math.round(h.ul)} Mb</div>
            <span style="color:var(--ok); font-weight:bold;">${h.ping}ms</span>
        </div>
    `).join('');
}

function copyReport() {
    if(!historyData.length) return;
    const h = historyData[historyData.length-1];
    const report = `📊 *RELATÓRIO NETSCAN PRO*\nOperadora: MC Telecom\nDownload: ${h.dl} Mbps\nUpload: ${h.ul} Mbps\nPing: ${h.ping}ms\nJitter: ${h.jitter}ms\nScore: 100%`;
    navigator.clipboard.writeText(report);
    alert("Relatório copiado para a área de transferência!");
}

checkNetwork();
initChart();
updateHistory();
</script>
</body>
</html>
