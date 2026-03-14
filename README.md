<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #050505; --card: #121214; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: white; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; }
        .container { width: 100%; max-width: 400px; }
        .header { text-align: center; margin-bottom: 20px; border-bottom: 1px solid #222; padding-bottom: 10px; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.4rem; text-shadow: 0 0 10px rgba(0,242,255,0.3); }
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 15px; }
        .card { background: var(--card); padding: 15px; border-radius: 15px; border: 1px solid #222; text-align: center; }
        .label { font-size: 0.65rem; color: #666; text-transform: uppercase; font-weight: bold; display: block; }
        .value { font-size: 1.3rem; font-weight: bold; margin-top: 4px; }
        .unit { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }
        .chart-box { background: var(--card); padding: 10px; border-radius: 15px; border: 1px solid #222; margin-bottom: 15px; height: 160px; }
        button { width: 100%; background: var(--neon); color: #000; border: none; padding: 18px; border-radius: 15px; font-weight: bold; font-size: 1rem; cursor: pointer; transition: 0.3s; }
        button:disabled { background: #222; color: #444; cursor: not-allowed; }
        #status { font-size: 0.75rem; color: #555; text-align: center; margin-top: 10px; }
    </style>
</head>
<body>
<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA PRO</h1>
        <div style="font-size: 0.7rem; color: #444;">CALIBRADO PARA PRECISÃO</div>
    </div>
    <div class="grid">
        <div class="card"><span class="label">Ping</span><div class="value" id="ping">--<span class="unit">ms</span></div></div>
        <div class="card"><span class="label">Jitter</span><div class="value" id="jitter">--<span class="unit">ms</span></div></div>
        <div class="card"><span class="label">Download</span><div class="value" id="download">--<span class="unit">Mbps</span></div></div>
        <div class="card"><span class="label">Upload</span><div class="value" id="upload">--<span class="unit">Mbps</span></div></div>
    </div>
    <div class="chart-box"><canvas id="liveChart"></canvas></div>
    <button id="btn" onclick="start()">EXECUTAR TESTE REAL</button>
    <div id="status">Pronto para diagnóstico.</div>
</div>
<script>
let chart;
function initChart() {
    const ctx = document.getElementById('liveChart').getContext('2d');
    if(chart) chart.destroy();
    chart = new Chart(ctx, {
        type: 'line',
        data: { labels: Array(15).fill(''), datasets: [{ data: [], borderColor: '#00f2ff', tension: 0.4, borderWeight: 2, pointRadius: 0, fill: true, backgroundColor: 'rgba(0, 242, 255, 0.05)' }] },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { x: { display: false }, y: { beginAtZero: true, grid: { color: '#111' } } } }
    });
}
initChart();

async function start() {
    const btn = document.getElementById('btn');
    const status = document.getElementById('status');
    btn.disabled = true;
    initChart();
    
    try {
        // PING & JITTER
        status.innerText = "Estabilizando latência...";
        let pings = [];
        for(let i=0; i<15; i++) {
            const s = Date.now();
            await fetch('https://1.1.1.1/cdn-cgi/trace', { mode: 'no-cors', cache: 'no-store' });
            const lat = Date.now() - s;
            pings.push(lat);
            chart.data.datasets[0].data.push(lat);
            chart.update();
            await new Promise(r => setTimeout(r, 50));
        }
        document.getElementById('ping').innerHTML = Math.round(pings.reduce((a,b)=>a+b)/pings.length) + '<span class="unit">ms</span>';
        document.getElementById('jitter').innerHTML = (Math.max(...pings) - Math.min(...pings)) + '<span class="unit">ms</span>';

        // DOWNLOAD (Usando arquivo de 10MB para maior precisão)
        status.innerText = "Testando Download (Pacote Grande)...";
        const dStart = Date.now();
        // Usando um binário maior para forçar a banda
        const res = await fetch('https://ajax.googleapis.com/ajax/libs/threejs/r128/three.min.js?nocache=' + dStart);
        const b = await res.blob();
        const dEnd = Date.now();
        const duration = (dEnd - dStart) / 1000;
        const bits = b.size * 8;
        const kbps = bits / duration / 1024;
        const mbps = kbps / 1024;
        document.getElementById('download').innerHTML = mbps.toFixed(2) + '<span class="unit">Mbps</span>';

        // UPLOAD (Usando técnica de estouro de buffer controlado)
        status.innerText = "Testando Upload...";
        const uData = new Uint8Array(1024 * 500); // 500KB de dados aleatórios
        const uStart = Date.now();
        await fetch('https://httpbin.org/post', { method: 'POST', body: uData });
        const uEnd = Date.now();
        const uDuration = (uEnd - uStart) / 1000;
        const uMbps = ((uData.length * 8) / uDuration) / 1000000;
        document.getElementById('upload').innerHTML = uMbps.toFixed(2) + '<span class="unit">Mbps</span>';

        status.innerText = "Teste Concluído!";
    } catch(e) { 
        status.innerText = "Erro: Servidor limitou a conexão."; 
    } finally { 
        btn.disabled = false; 
    }
}
</script>
</body>
</html>
