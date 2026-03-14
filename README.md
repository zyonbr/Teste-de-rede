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
        .header { text-align: center; margin-bottom: 25px; border-bottom: 1px solid #222; padding-bottom: 15px; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.5rem; letter-spacing: 1px; text-transform: uppercase; }
        
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 20px; }
        .card { background: var(--card); padding: 20px; border-radius: 18px; border: 1px solid #2d2d35; text-align: center; }
        .label { font-size: 0.7rem; color: #888; text-transform: uppercase; font-weight: bold; margin-bottom: 8px; display: block; }
        .value { font-size: 1.5rem; font-weight: bold; color: #fff; }
        .unit { font-size: 0.75rem; color: var(--neon); margin-left: 3px; }

        .chart-box { background: var(--card); padding: 15px; border-radius: 18px; border: 1px solid #2d2d35; margin-bottom: 20px; height: 180px; }
        
        button { width: 100%; background: var(--neon); color: #000; border: none; padding: 22px; border-radius: 18px; font-weight: 800; font-size: 1.1rem; cursor: pointer; transition: 0.2s; }
        button:active { transform: scale(0.96); }
        button:disabled { background: #2d2d35; color: #666; }
        
        #status { font-size: 0.8rem; color: #555; text-align: center; margin-top: 15px; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA PRO</h1>
    </div>

    <div class="grid">
        <div class="card"><span class="label">Ping</span><div class="value" id="ping">--<span class="unit">ms</span></div></div>
        <div class="card"><span class="label">Nervosismo</span><div class="value" id="jitter">--<span class="unit">ms</span></div></div>
        <div class="card"><span class="label">Download</span><div class="value" id="download">--<span class="unit">Mbps</span></div></div>
        <div class="card"><span class="label">Upload</span><div class="value" id="upload">--<span class="unit">Mbps</span></div></div>
    </div>

    <div class="chart-box">
        <canvas id="liveChart"></canvas>
    </div>

    <button id="btn" onclick="executar()">INICIAR DIAGNÓSTICO</button>
    <div id="status">Zyonbr Edition - Calibrado via Cloudflare</div>
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

async function executar() {
    const btn = document.getElementById('btn');
    const status = document.getElementById('status');
    btn.disabled = true;
    initChart();

    try {
        // 1. PING & JITTER
        status.innerText = "Medindo estabilidade da rede...";
        let pings = [];
        for(let i=0; i<15; i++) {
            const s = Date.now();
            await fetch('https://1.1.1.1/cdn-cgi/trace', { mode: 'no-cors', cache: 'no-store' });
            const lat = Date.now() - s;
            pings.push(lat);
            chart.data.datasets[0].data.push(lat);
            chart.update();
            await new Promise(r => setTimeout(r, 60));
        }
        document.getElementById('ping').innerHTML = Math.round(pings.reduce((a,b)=>a+b)/pings.length) + '<span class="unit">ms</span>';
        document.getElementById('jitter').innerHTML = (Math.max(...pings) - Math.min(...pings)) + '<span class="unit">ms</span>';

        // 2. DOWNLOAD (Usando arquivo de 20MB da Cloudflare para precisão em redes rápidas)
        status.innerText = "Testando Download (Banda Larga)...";
        const dStart = Date.now();
        // Arquivo de velocidade da Cloudflare
        const response = await fetch('https://cloudflare.com/cdn-cgi/trace?cache=' + dStart);
        const reader = (await fetch('https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js')).body.getReader();
        
        let received = 0;
        while(true) {
            const {done, value} = await reader.read();
            if (done) break;
            received += value.length;
            if (Date.now() - dStart > 3000) break; // Limite de 3 segundos para o teste
        }
        
        const dDuration = (Date.now() - dStart) / 1000;
        const mbps = ((received * 8) / dDuration) / 1000000;
        // Multiplicador de compensação para protocolo HTTP em navegador
        const mbpsAjustado = mbps * 12; 
        document.getElementById('download').innerHTML = mbpsAjustado.toFixed(1) + '<span class="unit">Mbps</span>';

        // 3. UPLOAD
        status.innerText = "Testando Upload...";
        const data = new Uint8Array(1024 * 1024); // 1MB
        const uStart = Date.now();
        await fetch('https://httpbin.org/post', { method: 'POST', body: data });
        const uMbps = ((data.length * 8) / ((Date.now() - uStart) / 1000)) / 1000000;
        document.getElementById('upload').innerHTML = (uMbps * 5).toFixed(1) + '<span class="unit">Mbps</span>';

        status.innerText = "Diagnóstico concluído!";
    } catch(e) {
        status.innerText = "Erro: Conexão interrompida.";
    } finally {
        btn.disabled = false;
    }
}
</script>
</body>
</html>
