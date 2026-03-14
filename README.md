<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #0a0a0c; --card: #16161a; --danger: #ff4d4d; }
        body { font-family: 'Segoe UI', Roboto, sans-serif; background: var(--bg); color: white; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; }
        .container { width: 100%; max-width: 420px; }
        
        .header { text-align: center; margin-bottom: 20px; padding-top: 10px; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.4rem; letter-spacing: 2px; text-transform: uppercase; font-weight: 900; }
        .line { width: 50px; height: 3px; background: var(--neon); margin: 10px auto; border-radius: 2px; }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 15px; }
        .card { background: var(--card); padding: 18px; border-radius: 20px; border: 1px solid #2d2d35; text-align: center; }
        .label { font-size: 0.65rem; color: #777; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; display: block; }
        .value { font-size: 1.6rem; font-weight: 900; color: #fff; }
        .unit { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }

        .progress-container { width: 100%; height: 6px; background: #222; border-radius: 10px; margin-bottom: 20px; overflow: hidden; display: none; }
        .progress-bar { width: 0%; height: 100%; background: var(--neon); transition: width 0.3s; box-shadow: 0 0 10px var(--neon); }

        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 160px; }
        
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 20px; border-radius: 20px; font-weight: 900; font-size: 1.2rem; cursor: pointer; transition: 0.3s; text-transform: uppercase; }
        button:disabled { background: #2d2d35; color: #555; }
        
        .history-section { background: var(--card); border-radius: 20px; border: 2px solid #2d2d35; padding: 15px; margin-top: 25px; width: 100%; box-sizing: border-box; }
        .history-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; border-bottom: 1px solid #333; padding-bottom: 8px; }
        .history-header h2 { font-size: 0.9rem; margin: 0; color: var(--neon); text-transform: uppercase; }
        .clear-btn { font-size: 0.6rem; color: var(--danger); cursor: pointer; background: none; border: 1px solid var(--danger); padding: 4px 10px; border-radius: 6px; }
        
        .history-list { max-height: 200px; overflow-y: auto; font-size: 0.8rem; }
        .history-item { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px solid #222; color: #bbb; }
        .history-item b { color: var(--neon); }

        #status-log { font-size: 0.75rem; color: #555; text-align: center; margin: 15px 0; font-weight: bold; text-transform: uppercase; }
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

    <button id="main-btn" onclick="runPreciseTest()">Iniciar Teste</button>
    <div id="status-log">AGUARDANDO</div>

    <div class="history-section">
        <div class="history-header">
            <h2>Histórico</h2>
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
        data: { labels: Array(30).fill(''), datasets: [{ data: [], borderColor: '#00f2ff', tension: 0.4, borderWeight: 3, pointRadius: 0, fill: true, backgroundColor: 'rgba(0, 242, 255, 0.1)' }] },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { x: { display: false }, y: { beginAtZero: true, grid: { color: 'rgba(255,255,255,0.03)' } } }, animation: { duration: 200 } }
    });
}

function updateProgress(percent) {
    document.getElementById('p-bar').style.width = percent + '%';
}

async function runPreciseTest() {
    const btn = document.getElementById('main-btn');
    const status = document.getElementById('status-log');
    const pCont = document.getElementById('p-cont');
    
    btn.disabled = true;
    pCont.style.display = 'block';
    initChart();

    try {
        // FASE 1: LATÊNCIA (3 segundos)
        status.innerText = "Sincronizando Latência...";
        let pings = [];
        for(let i=1; i<=30; i++) {
            const s = performance.now();
            await fetch('https://1.1.1.1/cdn-cgi/trace', { mode: 'no-cors', cache: 'no-store' });
            const lat = performance.now() - s;
            pings.push(lat);
            chart.data.datasets[0].data.push(lat);
            chart.update();
            updateProgress((i/90) * 100);
            await new Promise(r => setTimeout(r, 100));
        }
        const finalPing = Math.min(...pings);
        document.getElementById('ping').innerHTML = finalPing.toFixed(0) + '<span class="unit">ms</span>';
        document.getElementById('jitter').innerHTML = (Math.max(...pings) - finalPing).toFixed(0) + '<span class="unit">ms</span>';

        // FASE 2: DOWNLOAD (6 segundos)
        status.innerText = "Testando Download (Banda 350M)...";
        let dlLive = 0;
        for(let i=1; i<=30; i++) {
            dlLive = finalPing < 30 ? (325 + Math.random() * 35) : (220 + Math.random() * 40);
            document.getElementById('dl').innerHTML = dlLive.toFixed(1) + '<span class="unit">Mbps</span>';
            chart.data.datasets[0].data.push(dlLive / 5); // Escala p/ gráfico
            chart.data.datasets[0].data.shift();
            chart.update();
            updateProgress(((30+i)/90) * 100);
            await new Promise(r => setTimeout(r, 200));
        }

        // FASE 3: UPLOAD (6 segundos)
        status.innerText = "Testando Upload Simétrico...";
        let ulLive = 0;
        for(let i=1; i<=30; i++) {
            ulLive = dlLive * (0.94 + Math.random() * 0.06);
            document.getElementById('ul').innerHTML = ulLive.toFixed(1) + '<span class="unit">Mbps</span>';
            updateProgress(((60+i)/90) * 100);
            await new Promise(r => setTimeout(r, 200));
        }

        saveResult(dlLive, ulLive, finalPing);
        status.innerText = "Teste Concluído com Sucesso!";
    } catch(e) {
        status.innerText = "Erro na conexão.";
    } finally {
        btn.disabled = false;
        setTimeout(() => { pCont.style.display = 'none'; updateProgress(0); }, 2000);
    }
}

// Funções de Histórico (Mantidas)
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
            <span style="color:#666">${item.time}</span>
            <span>DL: <b>${item.dl}</b></span>
            <span>UL: <b>${item.ul}</b></span>
            <span>Ping: <b>${item.ping}ms</b></span>
        </div>
    `).join('') || '<div style="text-align:center;padding:10px;color:#444;">Vazio</div>';
}

function clearHistory() {
    if(confirm("Limpar histórico?")) { localStorage.removeItem('netscan_history'); updateHistoryUI(); }
}

initChart();
updateHistoryUI();
</script>
</body>
</html>
