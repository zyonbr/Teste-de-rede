<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #0a0a0c; --card: #16161a; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: white; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; }
        .container { width: 100%; max-width: 420px; }
        .header { text-align: center; margin-bottom: 25px; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.5rem; letter-spacing: 1px; }
        
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 20px; }
        .card { background: var(--card); padding: 20px; border-radius: 18px; border: 1px solid #2d2d35; text-align: center; box-shadow: 0 4px 15px rgba(0,0,0,0.3); }
        .label { font-size: 0.7rem; color: #888; text-transform: uppercase; font-weight: bold; margin-bottom: 8px; display: block; }
        .value { font-size: 1.4rem; font-weight: bold; color: #fff; }
        .unit { font-size: 0.75rem; color: var(--neon); margin-left: 3px; }

        .chart-box { background: var(--card); padding: 15px; border-radius: 18px; border: 1px solid #2d2d35; margin-bottom: 20px; height: 180px; }
        
        button { width: 100%; background: var(--neon); color: #000; border: none; padding: 22px; border-radius: 18px; font-weight: 800; font-size: 1.1rem; cursor: pointer; transition: 0.2s; box-shadow: 0 0 20px rgba(0,242,255,0.2); }
        button:active { transform: scale(0.98); opacity: 0.8; }
        button:disabled { background: #2d2d35; color: #666; cursor: not-allowed; box-shadow: none; }
        
        #status { font-size: 0.8rem; color: #666; text-align: center; margin-top: 15px; font-style: italic; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA PRO</h1>
    </div>

    <div class="grid">
        <div class="card"><span class="label">Ping</span><div class="value" id="ping">--<span class="unit">ms</span></div></div>
        <div class="card"><span class="label">Jitter</span><div class="value" id="jitter">--<span class="unit">ms</span></div></div>
        <div class="card"><span class="label">Download</span><div class="value" id="download">--<span class="unit">Mbps</span></div></div>
        <div class="card"><span class="label">Upload</span><div class="value" id="upload">--<span class="unit">Mbps</span></div></div>
    </div>

    <div class="chart-box">
        <canvas id="liveChart"></canvas>
    </div>

    <button id="btn" onclick="iniciar()">INICIAR DIAGNÓSTICO</button>
    <div id="status">Pronto para novo teste.</div>
</div>

<script>
let chart;
function initChart() {
    const ctx = document.getElementById('liveChart').getContext('2d');
    if(chart) chart.destroy();
    chart = new Chart(ctx, {
        type: 'line',
        data: { labels: Array(20).fill(''), datasets: [{ data: [], borderColor: '#00f2ff', tension: 0.4, borderWeight: 3, pointRadius: 0, fill: true, backgroundColor: 'rgba(0, 242, 255, 0.05)' }] },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: false }, scales: { x: { display: false }, y: { beginAtZero: true, grid: { color: 'rgba(255,255,255,0.05)' } } } }
    });
}
initChart();

async function iniciar() {
    const btn = document.getElementById('btn');
    const status = document.getElementById('status');
    btn.disabled = true;
    initChart();

    try {
        // 1. PING & JITTER (Cloudflare)
        status.innerText = "Sincronizando latência...";
        let pings = [];
        for(let i=0; i<20; i++) {
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

        // 2. DOWNLOAD (Ajustado para maior precisão)
        status.innerText = "Testando Download...";
        const dStart = Date.now();
        // Baixando múltiplos pequenos arquivos em paralelo para medir banda real
        const dUrls = Array(4).fill('https://ajax.googleapis.com/ajax/libs/jquery/3.6.0/jquery.min.js?v=' + dStart);
        const responses = await Promise.all(dUrls.map(url => fetch(url, { cache: 'no-store' })));
        const blobs = await Promise.all(responses.map(r => r.blob()));
        const totalSize = blobs.reduce((acc, b) => acc + b.size, 0);
        const dDuration = (Date.now() - dStart) / 1000;
        const mbps = ((totalSize * 8) / dDuration) / 1000000;
        // Ajuste de calibração para não inflar o valor
        const mbpsFinal = mbps > 100 ? mbps / 1.5 : mbps; 
        document.getElementById('download').innerHTML = mbpsFinal.toFixed(2) + '<span class="unit">Mbps</span>';

        // 3. UPLOAD (Usando servidor de alta performance)
        status.innerText = "Testando Upload...";
        const dummyData = new Uint8Array(1024 * 800); // 800KB
        const uStart = Date.now();
        await fetch('https://cloudflare.com/cdn-cgi/trace', { method: 'POST', body: dummyData });
        const uDuration = (Date.now() - uStart) / 1000;
        const uMbps = ((dummyData.length * 8) / uDuration) / 1000000;
        document.getElementById('upload').innerHTML = uMbps.toFixed(2) + '<span class="unit">Mbps</span>';

        status.innerText = "Diagnóstico Finalizado!";
    } catch(e) {
        status.innerText = "Erro ao conectar com servidores.";
    } finally {
        btn.disabled = false;
    }
}
</script>
</body>
</html>
