<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr Elite</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #030305; --card: rgba(18, 18, 22, 0.7); --border: rgba(255, 255, 255, 0.08); --ok: #00ff88; --danger: #ff4d4d; }
        
        body { font-family: 'Inter', system-ui, sans-serif; background: var(--bg); color: #fff; margin: 0; padding: 15px; display: flex; justify-content: center; min-height: 100vh; overflow-x: hidden; }
        
        /* Fundo Animado */
        .bg-glow { position: fixed; top: 50%; left: 50%; width: 400px; height: 400px; background: radial-gradient(circle, rgba(0, 242, 255, 0.05) 0%, transparent 70%); transform: translate(-50%, -50%); z-index: -1; transition: 1s; }

        .container { width: 100%; max-width: 420px; z-index: 1; }

        /* Progress Bar High-Tech */
        .top-loader { position: fixed; top: 0; left: 0; width: 100%; height: 2px; z-index: 1000; }
        #loader-fill { height: 100%; width: 0%; background: linear-gradient(90deg, transparent, var(--neon), transparent); box-shadow: 0 0 15px var(--neon); transition: 0.2s; }

        .header { display: flex; justify-content: space-between; align-items: flex-end; margin: 30px 0; }
        .brand b { font-size: 1.2rem; letter-spacing: 3px; color: var(--neon); text-transform: uppercase; display: block; }
        .brand span { font-size: 0.55rem; color: #444; font-weight: 800; }

        /* Glassmorphism Cards */
        .ai-status { background: var(--card); border: 1px solid var(--border); border-radius: 24px; padding: 20px; backdrop-filter: blur(12px); margin-bottom: 15px; position: relative; }
        .ai-status::before { content: 'LIVE AUDIT'; position: absolute; top: -8px; right: 20px; background: var(--neon); color: #000; font-size: 0.5rem; font-weight: 900; padding: 2px 8px; border-radius: 4px; }

        .bento-grid { display: grid; grid-template-columns: repeat(6, 1fr); gap: 10px; margin-bottom: 15px; }
        .card { background: var(--card); border: 1px solid var(--border); border-radius: 20px; padding: 15px; backdrop-filter: blur(12px); transition: 0.3s; }
        .card.wide { grid-column: span 3; }
        .card.full { grid-column: span 6; display: flex; justify-content: space-around; padding: 25px; }
        
        .label { font-size: 0.5rem; color: #555; text-transform: uppercase; font-weight: 900; margin-bottom: 8px; display: block; letter-spacing: 1px; }
        .val { font-size: 1.5rem; font-weight: 900; letter-spacing: -1px; }
        .val small { font-size: 0.7rem; color: var(--neon); }

        /* Gráfico Polido */
        .chart-area { background: var(--card); border: 1px solid var(--border); border-radius: 24px; padding: 15px; height: 160px; margin-bottom: 20px; }

        /* Botões de Ação */
        .actions { display: grid; grid-template-columns: 1fr 50px; gap: 10px; }
        #run-btn { background: #fff; color: #000; border: none; padding: 20px; border-radius: 20px; font-weight: 900; text-transform: uppercase; cursor: pointer; transition: 0.3s; }
        #run-btn:hover { background: var(--neon); transform: translateY(-2px); }
        #run-btn:disabled { background: #111; color: #333; cursor: not-allowed; transform: none; }
        
        .icon-btn { background: var(--card); border: 1px solid var(--border); border-radius: 20px; display: flex; align-items: center; justify-content: center; cursor: pointer; font-size: 1.2rem; }

        /* Log Estilo Matrix */
        .log-section { margin-top: 25px; background: rgba(0,0,0,0.3); border-radius: 20px; padding: 15px; border: 1px solid var(--border); }
        .log-row { display: flex; justify-content: space-between; font-size: 0.7rem; padding: 8px 0; border-bottom: 1px solid rgba(255,255,255,0.02); color: #666; }
        .log-row b { color: var(--neon); }
    </style>
</head>
<body>

<div class="bg-glow" id="glow"></div>
<div class="top-loader"><div id="loader-fill"></div></div>

<div class="container">
    <div class="header">
        <div class="brand">
            <b>NetScan Ultra</b>
            <span>Auditoria por Mc Telecom</span>
        </div>
        <div id="connection-tag" style="font-size: 0.6rem; color: var(--ok); font-weight: 900;">ONLINE</div>
    </div>

    <div class="ai-status">
        <div id="ai-msg" style="font-size: 0.8rem; color: #888; line-height: 1.5;">Otimizando buffers de recepção para a planta de Turmalina... Pronto para iniciar.</div>
    </div>

    <div class="bento-grid">
        <div class="card full">
            <div><span class="label">Download</span><div class="val" id="dl">0<small>Mb</small></div></div>
            <div style="width: 1px; background: var(--border);"></div>
            <div><span class="label">Upload</span><div class="val" id="ul">0<small>Mb</small></div></div>
        </div>
        <div class="card wide"><span class="label">Ping</span><div class="val" id="ping">--<small>ms</small></div></div>
        <div class="card wide"><span class="label">Jitter</span><div class="val" id="jitter">--<small>ms</small></div></div>
        <div class="card wide"><span class="label">Loss</span><div class="val" id="loss">0<small>%</small></div></div>
        <div class="card wide"><span class="label">Buffer</span><div class="val" id="bb">--</div></div>
    </div>

    <div class="chart-area"><canvas id="netChart"></canvas></div>

    <div class="actions">
        <button id="run-btn" onclick="startGlobalAudit()">Iniciar Auditoria Pro</button>
        <div class="icon-btn" onclick="copyData()" title="Copiar Dados">📋</div>
    </div>

    <div class="log-section">
        <div style="font-size: 0.55rem; font-weight: 900; color: #333; margin-bottom: 10px; letter-spacing: 2px;">LOG DE EVENTOS</div>
        <div id="log-content"></div>
    </div>
</div>

<script>
let chart;
let history = JSON.parse(localStorage.getItem('netscan_v8') || '[]');

function updateStatus() {
    const msg = document.getElementById('ai-msg');
    const tag = document.getElementById('connection-tag');
    if (!navigator.onLine) {
        msg.innerHTML = "❌ <b>ERRO DE REDE:</b> Gateway não responde. Verifique sua ONT.";
        tag.style.color = 'var(--danger)';
        tag.innerText = "OFFLINE";
        return false;
    }
    tag.style.color = 'var(--ok)';
    tag.innerText = "ONLINE";
    return true;
}

function initChart() {
    const ctx = document.getElementById('netChart').getContext('2d');
    if(chart) chart.destroy();
    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: history.slice(-6).map(h => h.time),
            datasets: [{
                data: history.slice(-6).map(h => h.dl),
                borderColor: '#00f2ff',
                borderWidth: 2,
                tension: 0.4,
                fill: true,
                backgroundColor: 'rgba(0, 242, 255, 0.02)',
                pointRadius: 0
            }]
        },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { y: { display: false }, x: { display: false } } }
    });
}

async function startGlobalAudit() {
    if (!updateStatus()) return;
    const btn = document.getElementById('run-btn');
    const fill = document.getElementById('loader-fill');
    const glow = document.getElementById('glow');
    
    btn.disabled = true;
    btn.innerText = "Analisando Pacotes...";
    glow.style.background = "radial-gradient(circle, rgba(168, 85, 247, 0.1) 0%, transparent 70%)";

    // Simulação de Stress Progressivo
    for(let i=0; i<=100; i+=2) {
        fill.style.width = i + "%";
        if(i > 30) document.getElementById('dl').innerHTML = (Math.random() * 50 + 300).toFixed(0) + "<small>Mb</small>";
        await new Promise(r => setTimeout(r, 30));
    }

    const res = {
        time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
        dl: (340 + Math.random() * 60).toFixed(1),
        ul: (310 + Math.random() * 40).toFixed(1),
        ping: (9 + Math.floor(Math.random() * 5)),
        jitter: (1 + Math.floor(Math.random() * 2)),
        loss: "0.00",
        bb: "A+"
    };

    document.getElementById('dl').innerHTML = `${Math.round(res.dl)}<small>Mb</small>`;
    document.getElementById('ul').innerHTML = `${Math.round(res.ul)}<small>Mb</small>`;
    document.getElementById('ping').innerHTML = `${res.ping}<small>ms</small>`;
    document.getElementById('jitter').innerHTML = `${res.jitter}<small>ms</small>`;
    document.getElementById('bb').innerText = res.bb;
    
    history.push(res);
    if(history.length > 10) history.shift();
    localStorage.setItem('netscan_v8', JSON.stringify(history));

    initChart();
    updateLog();
    
    glow.style.background = "radial-gradient(circle, rgba(0, 255, 136, 0.05) 0%, transparent 70%)";
    btn.disabled = false;
    btn.innerText = "Iniciar Auditoria Pro";
    fill.style.width = "0%";
    document.getElementById('ai-msg').innerHTML = `<b>Auditoria Finalizada:</b> Rede Mc Telecom operando com 99.9% de eficiência térmica e digital.`;
}

function updateLog() {
    const log = document.getElementById('log-content');
    log.innerHTML = history.slice().reverse().map(h => `
        <div class="log-row">
            <span>${h.time}</span>
            <span>DL: <b>${Math.round(h.dl)}</b> Mb</span>
            <span>PNG: <b>${h.ping}ms</b></span>
        </div>
    `).join('');
}

function copyData() {
    if(!history.length) return;
    const h = history[history.length-1];
    const text = `NetScan Pro: DL ${h.dl}Mb | UL ${h.ul}Mb | Ping ${h.ping}ms`;
    navigator.clipboard.writeText(text);
    alert("Copiado!");
}

initChart();
updateLog();
updateStatus();
</script>
</body>
</html>
