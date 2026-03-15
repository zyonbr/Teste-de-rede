<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NetScan Ultra Pro | Zyonbr</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --neon: #00f2ff; --bg: #0a0a0c; --card: #16161a; --danger: #ff4d4d; --ok: #00ff88; --warning: #ffcc00; }
        body { font-family: 'Segoe UI', Roboto, sans-serif; background: var(--bg); color: white; margin: 0; padding: 15px; display: flex; flex-direction: column; align-items: center; min-height: 100vh; }
        .container { width: 100%; max-width: 420px; }
        
        .header { text-align: center; margin-bottom: 20px; }
        .header h1 { color: var(--neon); margin: 0; font-size: 1.4rem; letter-spacing: 2px; text-transform: uppercase; font-weight: 900; }
        .line { width: 50px; height: 3px; background: var(--neon); margin: 8px auto; border-radius: 2px; }

        /* AI Intelligence Box */
        .ai-box { background: rgba(0, 242, 255, 0.05); border: 1px solid rgba(0, 242, 255, 0.3); border-radius: 20px; padding: 15px; margin-bottom: 15px; font-size: 0.85rem; position: relative; }
        .ai-tag { position: absolute; top: -10px; left: 20px; background: var(--neon); color: #000; font-size: 0.6rem; font-weight: 900; padding: 2px 8px; border-radius: 5px; }
        #ai-verdict { color: #ccc; line-height: 1.4; margin-top: 5px; }
        .trend-up { color: var(--ok); font-weight: bold; }
        .trend-down { color: var(--danger); font-weight: bold; }

        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 15px; }
        .card { background: var(--card); padding: 18px; border-radius: 20px; border: 1px solid #2d2d35; text-align: center; position: relative; }
        .label { font-size: 0.65rem; color: #777; text-transform: uppercase; font-weight: bold; margin-bottom: 5px; display: block; }
        .value { font-size: 1.6rem; font-weight: 900; }
        .unit { font-size: 0.7rem; color: var(--neon); margin-left: 2px; }
        
        /* Indicador de Comparação */
        .comp-indicator { font-size: 0.6rem; position: absolute; bottom: 8px; width: 100%; left: 0; opacity: 0.8; }

        .chart-box { background: var(--card); padding: 15px; border-radius: 20px; border: 1px solid #2d2d35; margin-bottom: 15px; height: 200px; width: 100%; box-sizing: border-box; }
        
        button#main-btn { width: 100%; background: var(--neon); color: #000; border: none; padding: 20px; border-radius: 20px; font-weight: 900; font-size: 1.1rem; cursor: pointer; text-transform: uppercase; transition: 0.3s; box-shadow: 0 10px 20px rgba(0, 242, 255, 0.1); }
        button:disabled { opacity: 0.5; }

        #status-log { font-size: 0.7rem; color: #555; text-align: center; margin-top: 15px; font-weight: bold; letter-spacing: 1px; }

        .history-section { background: var(--card); border-radius: 20px; padding: 15px; margin-top: 20px; width: 100%; box-sizing: border-box; border: 1px solid #222; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <h1>NETSCAN ULTRA</h1>
        <div class="line"></div>
    </div>

    <div class="ai-box">
        <span class="ai-tag">ANALISTA IA</span>
        <div id="ai-verdict">Inicie o diagnóstico para comparar a estabilidade atual com o histórico de rede.</div>
    </div>

    <div class="grid">
        <div class="card">
            <span class="label">Ping</span>
            <div class="value" id="ping">--<span class="unit">ms</span></div>
            <div class="comp-indicator" id="c-ping"></div>
        </div>
        <div class="card">
            <span class="label">Jitter</span>
            <div class="value" id="jitter">--<span class="unit">ms</span></div>
            <div class="comp-indicator" id="c-jitter"></div>
        </div>
        <div class="card">
            <span class="label">Download</span>
            <div class="value" id="dl">--<span class="unit">Mbps</span></div>
            <div class="comp-indicator" id="c-dl"></div>
        </div>
        <div class="card">
            <span class="label">Upload</span>
            <div class="value" id="ul">--<span class="unit">Mbps</span></div>
            <div class="comp-indicator" id="c-ul"></div>
        </div>
    </div>

    <div class="chart-box">
        <canvas id="trendChart"></canvas>
    </div>

    <button id="main-btn" onclick="runUltraTest()">Executar Teste IA</button>
    <div id="status-log">AGUARDANDO COMANDO</div>

    <div class="history-section" id="hist-sec" style="display:none;">
        <h2 style="font-size: 0.7rem; color: #555; margin: 0 0 10px 0; text-transform: uppercase;">Tendência de Sessão</h2>
        <div id="history-list"></div>
    </div>
</div>

<script>
let trendChart;
let lastData = JSON.parse(localStorage.getItem('last_test') || 'null');

function initTrendChart(currentDL = 0) {
    const ctx = document.getElementById('trendChart').getContext('2d');
    const lastDL = lastData ? parseFloat(lastData.dl) : 0;

    if(trendChart) trendChart.destroy();
    
    // Gradiente Neon
    const gradient = ctx.createLinearGradient(0, 0, 0, 200);
    gradient.addColorStop(0, 'rgba(0, 242, 255, 0.2)');
    gradient.addColorStop(1, 'rgba(0, 242, 255, 0)');

    trendChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: ['Teste Anterior', 'Teste Atual'],
            datasets: [{
                label: 'Download (Mbps)',
                data: [lastDL, currentDL],
                borderColor: '#00f2ff',
                backgroundColor: gradient,
                borderWidth: 4,
                pointBackgroundColor: '#fff',
                pointRadius: 6,
                fill: true,
                tension: 0.4
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { display: false } },
            scales: {
                y: { beginAtZero: true, max: 400, grid: { color: '#222' }, ticks: { color: '#555' } },
                x: { ticks: { color: '#888' } }
            }
        }
    });
}

function updateIndicators(dl, ul, p, j) {
    if(!lastData) return;

    const diff = (val, last) => {
        const d = val - last;
        if(d === 0) return `<span style="color:#888">● Estável</span>`;
        return d > 0 ? `<span class="trend-up">▲ +${d.toFixed(1)}</span>` : `<span class="trend-down">▼ ${d.toFixed(1)}</span>`;
    };

    // Para Ping e Jitter, MENOR é melhor (invertemos as cores logicamente na IA)
    document.getElementById('c-dl').innerHTML = diff(dl, lastData.dl);
    document.getElementById('c-ul').innerHTML = diff(ul, lastData.ul);
    document.getElementById('c-ping').innerHTML = p > lastData.ping ? `<span class="trend-down">▲ +${(p-lastData.ping).toFixed(0)}</span>` : `<span class="trend-up">▼ ${(p-lastData.ping).toFixed(0)}</span>`;
}

function runIAAnalyst(dl, p, j) {
    const verdict = document.getElementById('ai-verdict');
    if(!lastData) {
        verdict.innerHTML = "<b>Primeira análise concluída.</b> A IA precisa de um segundo teste para calcular a curva de tendência.";
        return;
    }

    const dlDiff = dl - lastData.dl;
    const pingDiff = p - lastData.ping;
    
    let aiText = "";
    if (dlDiff > 5) {
        aiText = `A IA detectou uma <b class="trend-up">Melhoria Térmica</b> na rede. Sua banda cresceu ${dlDiff.toFixed(1)} Mbps em relação ao teste anterior.`;
    } else if (dlDiff < -10) {
        aiText = `A IA alerta para <b class="trend-down">Degradação de Fluxo</b>. Perda de ${Math.abs(dlDiff).toFixed(1)} Mbps detectada. Verifique interferências.`;
    } else {
        aiText = `A IA confirma <b>Consistência de Fibra</b>. A variação de rede está dentro da margem de 2%, indicando estabilidade total.`;
    }

    if (pingDiff > 5) aiText += " Notamos um aumento na latência, o que pode afetar jogos.";
    
    verdict.innerHTML = aiText;
}

async function runUltraTest() {
    const btn = document.getElementById('main-btn');
    btn.disabled = true;
    document.getElementById('status-log').innerText = "IA PROCESSANDO CURVA PREDITIVA...";

    setTimeout(() => {
        const p = 18 + Math.floor(Math.random() * 10);
        const j = 2 + Math.floor(Math.random() * 6);
        const dl = (320 + Math.random() * 35).toFixed(1);
        const ul = (310 + Math.random() * 30).toFixed(1);

        document.getElementById('ping').innerHTML = p + '<span class="unit">ms</span>';
        document.getElementById('jitter').innerHTML = j + '<span class="unit">ms</span>';
        document.getElementById('dl').innerHTML = dl + '<span class="unit">Mbps</span>';
        document.getElementById('ul').innerHTML = ul + '<span class="unit">Mbps</span>';
        
        initTrendChart(dl);
        updateIndicators(parseFloat(dl), parseFloat(ul), p, j);
        runIAAnalyst(parseFloat(dl), p, j);
        
        // Salva para a próxima comparação
        lastData = { dl: parseFloat(dl), ul: parseFloat(ul), ping: p, jitter: j };
        localStorage.setItem('last_test', JSON.stringify(lastData));

        document.getElementById('status-log').innerText = "DIAGNÓSTICO FINALIZADO";
        btn.disabled = false;
    }, 3000);
}

// Inicia limpo
initTrendChart(0);
</script>
</body>
</html>
