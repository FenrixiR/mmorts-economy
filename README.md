<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game Economy Curve Visualizer</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: system-ui, -apple-system, sans-serif; background: #121214; color: #e4e4e7; margin: 0; padding: 24px; }
        .container { max-width: 1100px; margin: 0 auto; }
        .grid { display: grid; grid-template-columns: 300px 1fr; gap: 24px; }
        .card { background: #18181b; border: 1px solid #27272a; border-radius: 8px; padding: 20px; }
        .control-group { margin-bottom: 16px; }
        label { display: block; font-size: 0.85rem; color: #a1a1aa; margin-bottom: 6px; }
        input[type="range"] { width: 100%; accent-color: #6366f1; }
        .val-display { float: right; color: #38bdf8; font-weight: 600; }
        .stats-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin-top: 16px; }
        .stat-box { background: #27272a; padding: 12px; border-radius: 6px; text-align: center; }
        .stat-box .title { font-size: 0.75rem; color: #a1a1aa; }
        .stat-box .num { font-size: 1.1rem; font-weight: bold; color: #f4f4f5; margin-top: 4px; }
        canvas { max-height: 500px; }
    </style>
</head>
<body>

<div class="container">
    <h2>Game Progression Curve Inspector</h2>
    <div class="grid">
        <div class="card">
            <h3>Parameters</h3>
            
            <div class="control-group">
                <label>Resource Exponent (&alpha;<sub>cost</sub>) <span id="resExpVal" class="val-display">2.5</span></label>
                <input type="range" id="resExp" min="1.0" max="4.0" step="0.1" value="2.5">
            </div>

            <div class="control-group">
                <label>Time Exponent (&alpha;<sub>time</sub>) <span id="timeExpVal" class="val-display">2.0</span></label>
                <input type="range" id="timeExp" min="1.0" max="3.0" step="0.1" value="2.0">
            </div>

            <div class="control-group">
                <label>Max Cap Build Time (Days) <span id="maxDaysVal" class="val-display">7</span></label>
                <input type="range" id="maxDays" min="1" max="30" step="1" value="7">
            </div>

            <hr style="border-color: #27272a; margin: 20px 0;">

            <h3>Level Inspector</h3>
            <div class="control-group">
                <label>Target Level <span id="inspectLvlVal" class="val-display">50</span></label>
                <input type="range" id="inspectLvl" min="1" max="100" step="1" value="50">
            </div>

            <div class="stats-grid">
                <div class="stat-box">
                    <div class="title">Wood/Stone</div>
                    <div class="num" id="inspectWood">0</div>
                </div>
                <div class="stat-box">
                    <div class="title">Gold</div>
                    <div class="num" id="inspectGold">0</div>
                </div>
                <div class="stat-box">
                    <div class="title">Build Time</div>
                    <div class="num" id="inspectTime">0s</div>
                </div>
            </div>
        </div>

        <div class="card">
            <canvas id="progressionChart"></canvas>
        </div>
    </div>
</div>

<script>
function dynamicRound(val) {
    const rawInt = Math.round(val);
    if (rawInt < 1000) return rawInt;
    if (rawInt < 10000) return Math.round(val / 10) * 10;
    if (rawInt < 100000) return Math.round(val / 100) * 100;
    if (rawInt < 1000000) return Math.round(val / 1000) * 1000;
    return Math.round(val / 10000) * 10000;
}

function formatTime(seconds) {
    let sec = Math.round(seconds);
    if (sec < 60) {}
    else if (sec < 3600) sec = Math.round(sec / 15) * 15;
    else if (sec < 86400) sec = Math.round(sec / 300) * 300;
    else sec = Math.round(sec / 1800) * 1800;

    const d = Math.floor(sec / 86400);
    const h = Math.floor((sec % 86400) / 3600);
    const m = Math.floor((sec % 3600) / 60);
    const s = sec % 60;

    let parts = [];
    if (d > 0) parts.push(`${d}d`);
    if (h > 0) parts.push(`${h}h`);
    if (m > 0) parts.push(`${m}m`);
    if (s > 0 || parts.length === 0) parts.push(`${s}s`);
    return parts.join(' ');
}

function calculateData() {
    const resExp = parseFloat(document.getElementById('resExp').value);
    const timeExp = parseFloat(document.getElementById('timeExp').value);
    const maxDays = parseFloat(document.getElementById('maxDays').value);
    const maxSec = maxDays * 86400;

    const labels = [];
    const woodData = [];
    const goldData = [];
    const timeData = [];
    const rawTimeSecs = [];

    for (let lvl = 1; lvl <= 100; lvl++) {
        labels.push(`Lvl ${lvl}`);
        const progress = (lvl - 1) / 99.0;

        const wood = dynamicRound(100 + (100000 - 100) * Math.pow(progress, resExp));
        const gold = dynamicRound(500 + (6000000 - 500) * Math.pow(progress, resExp));
        const tSec = 10 + (maxSec - 10) * Math.pow(progress, timeExp);

        woodData.push(wood);
        goldData.push(gold);
        timeData.push(tSec / 3600); // Display in hours on chart
        rawTimeSecs.push(tSec);
    }

    return { labels, woodData, goldData, timeData, rawTimeSecs };
}

const ctx = document.getElementById('progressionChart').getContext('2d');
let data = calculateData();

const chart = new Chart(ctx, {
    type: 'line',
    data: {
        labels: data.labels,
        datasets: [
            { label: 'Gold Cost', data: data.goldData, borderColor: '#eab308', yAxisID: 'y' },
            { label: 'Wood/Stone Cost', data: data.woodData, borderColor: '#22c55e', yAxisID: 'y' },
            { label: 'Build Time (Hours)', data: data.timeData, borderColor: '#38bdf8', yAxisID: 'y1' }
        ]
    },
    options: {
        responsive: true,
        interaction: { mode: 'index', intersect: false },
        scales: {
            y: { type: 'linear', position: 'left', grid: { color: '#27272a' }, ticks: { color: '#a1a1aa' } },
            y1: { type: 'linear', position: 'right', grid: { drawOnChartArea: false }, ticks: { color: '#38bdf8' } }
        },
        plugins: { legend: { labels: { color: '#e4e4e7' } } }
    }
});

function update() {
    document.getElementById('resExpVal').innerText = document.getElementById('resExp').value;
    document.getElementById('timeExpVal').innerText = document.getElementById('timeExp').value;
    document.getElementById('maxDaysVal').innerText = document.getElementById('maxDays').value;
    document.getElementById('inspectLvlVal').innerText = document.getElementById('inspectLvl').value;

    data = calculateData();
    chart.data.datasets[0].data = data.goldData;
    chart.data.datasets[1].data = data.woodData;
    chart.data.datasets[2].data = data.timeData;
    chart.update();

    const lvlIdx = parseInt(document.getElementById('inspectLvl').value) - 1;
    document.getElementById('inspectWood').innerText = data.woodData[lvlIdx].toLocaleString();
    document.getElementById('inspectGold').innerText = data.goldData[lvlIdx].toLocaleString();
    document.getElementById('inspectTime').innerText = formatTime(data.rawTimeSecs[lvlIdx]);
}

['resExp', 'timeExp', 'maxDays', 'inspectLvl'].forEach(id => {
    document.getElementById(id).addEventListener('input', update);
});
update();
</script>

</body>
</html>
