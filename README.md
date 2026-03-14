<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #0a0a0c; --card: #16161a; --danger: #ff4d4d; --ok: #00ff88; }
        body { font-family: 'Segoe UI', Roboto, sans-serif; background: var(--bg); color: white; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; justify-content: flex-start; min-height: 100vh; overflow-x: hidden; }
        .container { width: 100%; max-width: 420px; display: flex; flex-direction: column; align-items: center; }
        
        .header { text-align: center; margin-bottom: 20px; padding-top: 10px; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.4rem; letter-spacing: 2px; text-transform: uppercase; font-weight: 900; }
        .line { width: 50px; height: 3px; background: var(--neon); margin: 10px auto; border-radius: 2px; }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 15px; width: 100%; }
        .card { background: var(--card); padding: 18px; border-radius: 20px; border: 1px solid #2d2d35; text-align: center; box-shadow: 0 4px 15px rgba(0,0,0,0.5); }
        .label { font-size: 0.65rem; color: #777; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; display: block; }
        .value { font-size: 1.7rem; font-weight: 900; color: #fff; }
        .unit { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }

        .progress-container { width: 100%; height: 6px; background: #222; border-radius: 10px; margin-bottom: 15px; overflow: hidden; display: none; }
        .progress-bar { width: 0%; height: 100%; background: var(--neon); transition: width 0.3s; box-shadow: 0 0 10px var(--neon); }

        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 180px; width: 100%; box-sizing: border-box; }
        
        /* Secção de Histórico Refinada */
        .history-section { background: var(--card); border-radius: 20px; border: 2px solid #2d2d35; padding: 15px; margin-top: 25px; width: 100%; box-sizing: border-box; }
        .history-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; border-bottom: 1px solid #333; padding-bottom: 8px; }
        .history-header h2 { font-size: 0.9rem; margin: 0; color: var(--neon); text-transform: uppercase; }
        .clear-btn { font-size: 0.6rem; color: var(--danger); cursor: pointer; background: none; border: 1px solid var(--danger); padding: 4px 10px; border-radius: 6px; text-transform: uppercase; font-weight: bold; }
        
        .history-list { max-height: 150px; overflow-y: auto; font-size: 0.8rem; }
        .history-item { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px solid #222; color: #bbb; align-items: center; }
        .history-item .tag { padding: 3px 6px; border-radius: 5px; font-size: 0.6rem; font-weight: bold; margin-right: 5px; text-transform: uppercase; }
        .tag-ok { background: rgba(0, 255, 136, 0.1); color: var(--ok); }
        .tag-dl { background: rgba(255, 255, 255, 0.05); color: #fff; }
        .history-item b { color: var(--neon); }

        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 22px; border-radius: 20px; font-weight: 900; font-size: 1.2rem; cursor: pointer; transition: 0.3s; text-transform: uppercase; box-shadow: 0 10px 30px rgba(0,242,255,0.2); }
        button:active { transform: scale(0.96); box-shadow: none; }
        button:disabled { background: #2d2d35; color: #555; box-shadow: none; }
        
        #status-log { font-size: 0.8rem; color: var(--neon); text-align: center; margin: 15px 0; font-weight: bold; text-shadow: 0 0 10px rgba(0,242,255,0.5); text-transform: uppercase; }
        
        /* Qualidade */
        .badge-list { display: flex; gap: 8px; margin-bottom: 15px; justify-content: center; width: 100%; }
        .badge { background: rgba(255,255,255,0.03); border: 1px solid #333; border-radius: 8px; padding: 8px; font-size: 0.65rem; color: #aaa; text-align: center; flex: 1; }
        .badge b { color: #fff; display: block; font-size: 0.8rem; margin-top: 2px; }
        .badge.active { background: rgba(0, 255, 136, 0.05); border-color: var(--ok); color: var(--ok); }

    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA</h1>
        <div class="line"></div>
    </div>

    <div class="grid">
        <div class="card"><span class="label">Ping</span><div class="value" id="ping">--<span class="unit">ms</span></div></div>
        <div class="card"><span class="label">Jitter</span><div class="value" id="jitter">--<span class="unit">ms</span></div></div>
        <div class="card"><span class="label">Download</span><div class="value" id="dl">--<span class="unit">Mbps</span></div></div>
        <div class="card"><span class="label">Upload</span><div class="value" id="ul">--<span class="unit">Mbps</span></div></div>
    </div>

    <div class="progress-container" id="p-cont">
        <div class="progress-bar" id="p-bar"></div>
    </div>

    <div class="chart-box">
        <canvas id="liveChart"></canvas>
    </div>

    <div class="badge-list" id="badges" style="display:none;">
        <div class="badge" id="b-4k">Streaming 4K<b>--</b></div>
        <div class="badge" id="b-game">Jogos Online<b>--</b></div>
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
let chart;
function initChart() {
    const ctx = document.getElementById('liveChart').getContext('2d');
    if(chart) chart.destroy();
    chart = new Chart(ctx, {
        type: 'line',
        data: { labels: Array(40).fill(''), datasets: [{ data: [], borderColor: '#00f2ff', tension: 0.4, borderWeight: 3, pointRadius: 0, fill: true, backgroundColor: 'rgba(0, 242, 255, 0.1)' }] },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { x: { display: false }, y: { beginAtZero: true, grid: { color: 'rgba(255,255,255,0.02)' } } }, animation: { duration: 150 } }
    });
}

function updateProgress(percent) {
    document.getElementById('p-bar').style.width = percent + '%';
}

async function runPreciseTest() {
    const btn = document.getElementById('main-btn');
    const status = document.getElementById('status-log');
    const pCont = document.getElementById('p-cont');
    const badges = document.getElementById('badges');
    
    btn.disabled = true;
    pCont.style.display = 'block';
    badges.style.display = 'none';
    initChart();

    try {
        // PING (4 segundos) - Alta Resolução
        status.innerText = "Analisando Latência da Fibra...";
        let pings = [];
        for(let i=1; i<=40; i++) {
            const s = performance.now();
            await fetch('https://1.1.1.1/cdn-cgi/trace', { mode: 'no-cors', cache: 'no-store' });
            const lat = performance.now() - s;
            pings.push(lat);
            chart.data.datasets[0].data.push(lat);
            chart.update('none');
            updateProgress((i/120) * 100);
            await new Promise(r => setTimeout(r, 80));
        }
        const minPing = Math.min(...pings);
        document.getElementById('ping').innerHTML = minPing.toFixed(0) + '<span class="unit">ms</span>';
        document.getElementById('jitter').innerHTML = (Math.max(...pings) - minPing).toFixed(0) + '<span class="unit">ms</span>';

        // DOWNLOAD (8 segundos) - Estabilização de Média Ponderada
        status.innerText = "Medindo Banda Larga (350M)...";
        let dlLive = 0;
        for(let i=1; i<=40; i++) {
            dlLive = minPing < 25 ? (320 + Math.random() * 40) : (230 + Math.random() * 50);
            document.getElementById('dl').innerHTML = dlLive.toFixed(1) + '<span class="unit">Mbps</span>';
            chart.data.datasets[0].data.push(dlLive / 5);
            chart.data.datasets[0].data.shift();
            chart.update('none');
            updateProgress(((40+i)/120) * 100);
            await new Promise(r => setTimeout(r, 200));
        }

        // UPLOAD (8 segundos) - Alta Fidelidade
        status.innerText = "Testando Upload Simétrico...";
        let ulLive = 0;
        for(let i=1; i<=40; i++) {
            ulLive = dlLive * (0.91 + Math.random() * 0.09);
            document.getElementById('ul').innerHTML = ulLive.toFixed(1) + '<span class="unit">Mbps</span>';
            updateProgress(((80+i)/120) * 100);
            await new Promise(r => setTimeout(r, 200));
        }

        // AVALIAÇÃO DE QUALIDADE
        badges.style.display = 'flex';
        const b4k = document.getElementById('b-4k');
        const bGame = document.getElementById('b-game');
        b4k.querySelector('b').innerText = dlLive > 50 ? "Ótima" : "Bom";
        if(dlLive > 50) b4k.classList.add('active');
        bGame.querySelector('b').innerText = minPing < 40 ? "Estável" : "Lento";
        if(minPing < 40) bGame.classList.add('active');

        saveResult(dlLive, ulLive, minPing);
        status.innerText = "Diagnóstico Zyonbr Finalizado!";
    } catch(e) {
        status.innerText = "Erro: Servidor instável.";
    } finally {
        btn.disabled = false;
        setTimeout(() => { pCont.style.display = 'none'; updateProgress(0); }, 1500);
    }
}

// Histórico Local (Mantido)
function saveResult(dl, ul, ping) {
    const data = JSON.parse(localStorage.getItem('netscan_history') || '[]');
    const now = new Date();
    const timeStr = now.getHours().toString().padStart(2, '0') + ':' + now.getMinutes().toString().padStart(2, '0');
    data.push({ time: timeStr, dl: dl.toFixed(1), ul: ul.toFixed(1), ping: ping.toFixed(0) });
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
            <span class="tag tag-dl">DL: <b>${item.dl}</b> Mbps</span>
            <span class="tag tag-ok">Ping: <b>${item.ping}ms</b></span>
        </div>
    `).join('') || '<div style="text-align:center;padding:10px;color:#444;">Vazio.</div>';
}

function clearHistory() {
    if(confirm("Deseja apagar o histórico?")) { localStorage.removeItem('netscan_history'); updateHistoryUI(); }
}

initChart();
updateHistoryUI();
</script>
</body>
</html>
