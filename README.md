<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>BTC 15m - 专业定制版</title>
    <style>
        :root { --green: #3fb950; --red: #f85149; --gold: #f2c94c; --bg: #090c10; --card: #161b22; }
        /* 核心修正：禁止长按选择和蓝框 */
        * { 
            box-sizing: border-box; 
            -webkit-tap-highlight-color: transparent; 
            margin: 0; 
            padding: 0; 
            -webkit-user-select: none; /* 禁止选中文本 */
            user-select: none;
            -webkit-touch-callout: none; /* 禁止长按弹出系统菜单 */
        }
        /* 允许输入框正常操作 */
        input { -webkit-user-select: text; user-select: text; }

        body { background: var(--bg); color: #c9d1d9; font-family: -apple-system, sans-serif; padding: 10px 12px; height: 100vh; display: flex; flex-direction: column; overflow: hidden; }
        body.landscape-mode { position: fixed; top: 0; left: 0; width: 100vh; height: 100vw; transform: rotate(90deg); transform-origin: 0 0; left: 100vw; padding: 0; z-index: 9999; background: #000; }
        body.landscape-mode .header, body.landscape-mode .strategy-panel, body.landscape-mode .calc-panel, body.landscape-mode .controls { display: none !important; }
        body.landscape-mode #chartCanvas { width: 100vh !important; height: 100vw !important; max-height: none !important; border: none; border-radius: 0; }
        .land-back-btn { display: none; position: absolute; bottom: 20px; right: 20px; z-index: 10001; background: rgba(255,255,255,0.15); color: #fff; padding: 12px 24px; border-radius: 30px; border: 1px solid rgba(255,255,255,0.3); font-size: 14px; backdrop-filter: blur(5px); }
        body.landscape-mode .land-back-btn { display: block; }
        #chartCanvas { width: 100%; flex: 1; max-height: 52vh; background: #010409; border: 1px solid #30363d; border-radius: 12px; touch-action: none; display: block; outline: none; }
        .header { display: flex; justify-content: space-between; align-items: flex-end; margin-bottom: 5px; }
        .price { font-size: 28px; font-weight: 800; color: var(--green); font-family: monospace; }
        .strategy-panel { margin-top: 8px; display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
        .card { background: var(--card); border: 1px solid #30363d; border-radius: 10px; padding: 8px; position: relative; }
        .label { font-size: 10px; font-weight: bold; margin-bottom: 3px; display: block; opacity: 0.8; }
        .val { font-size: 11px; font-family: monospace; color: #f0f6fc; line-height: 1.4; }
        .lev-tag { position: absolute; right: 8px; bottom: 8px; font-size: 16px; font-weight: 900; color: var(--gold); }
        .controls { margin-top: 10px; display: grid; grid-template-columns: repeat(4, 1fr); gap: 6px; }
        .btn { padding: 12px 2px; border: none; border-radius: 8px; font-weight: bold; font-size: 11px; background: #21262d; color: #c9d1d9; border: 1px solid #30363d; }
        .btn-active { background: #238636 !important; }
        .btn-edit { background: #1f6feb !important; }
        .calc-panel { margin-top: 10px; background: var(--card); border: 1px solid #30363d; border-radius: 10px; padding: 10px; }
        .calc-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 6px; }
        .calc-panel input { background: #0d1117; border: 1px solid #30363d; color: #fff; padding: 8px; border-radius: 6px; font-size: 13px; width: 100%; outline: none; }
        #alertOverlay { position: absolute; top: 15%; left: 50%; transform: translateX(-50%); padding: 15px 30px; border-radius: 25px; font-weight: bold; display: none; z-index: 10000; text-align: center; cursor: pointer; box-shadow: 0 4px 15px rgba(0,0,0,0.5); pointer-events: auto; min-width: 200px; }
        .alert-long { background: var(--green); color: white; }
        .alert-short { background: var(--red); color: white; }
        .alert-break { background: var(--gold); color: #000; }
    </style>
</head>
<body>
    <button class="land-back-btn" onclick="toggleLandscape()">退出看盘</button>
    <div id="alertOverlay" onclick="stopAlarm()">信号提示 (点击关闭声音)</div>
    
    <div class="header">
        <div><div id="curPrice" class="price">--.--</div><div style="font-size:10px; color:var(--gold)">OKX BTC-15m (修复版)</div></div>
        <div style="text-align: right"><div id="countdown" style="font-size:10px; color:#8b949e">收线 --:--</div></div>
    </div>
    <canvas id="chartCanvas"></canvas>
    
    <div class="strategy-panel">
        <div class="card">
            <span class="label" style="color:var(--green)">看涨 OB (极值支撑)</span>
            <div class="val">支撑: <span id="lE">--</span></div>
            <div class="val">止损: <span id="lS">--</span></div>
            <div class="lev-tag" id="lLev">--x</div>
        </div>
        <div class="card">
            <span class="label" style="color:var(--red)">看跌 OB (极值阻力)</span>
            <div class="val">压力: <span id="sE">--</span></div>
            <div class="val">止损: <span id="sS">--</span></div>
            <div class="lev-tag" id="sLev">--x</div>
        </div>
    </div>

    <div class="controls">
        <button class="btn" style="background:var(--gold);color:#000" onclick="toggleLandscape()">看盘</button>
        <button id="editBtn" class="btn" onclick="toggleEdit()">调线</button>
        <button id="alertBtn" class="btn btn-active" onclick="toggleAlert()">守护中</button>
        <button class="btn" onclick="document.getElementById('audioFile').click()">音效</button>
        <input type="file" id="audioFile" accept="audio/*,video/*,.mp3,.m4a" hidden>
    </div>

    <div class="calc-panel">
        <div class="calc-grid">
            <input type="number" id="calcE" placeholder="入场价">
            <input type="number" id="calcS" placeholder="止损价">
            <input type="number" id="calcL" placeholder="预亏$">
            <input type="number" id="calcM" placeholder="保证金$">
        </div>
        <div style="margin-top:8px; display:flex; justify-content:space-between; align-items:center">
            <span id="audioNameDisp" style="font-size:9px; color:#444">修复长按/实时K线颜色</span>
            <span id="calcResVal" style="font-size:18px; color:var(--gold); font-weight:900;">-- x</span>
        </div>
    </div>

<script>
    let rawData = [], isMonitoring = true, isEditMode = false, isLandscape = false;
    // 默认提供一个简单的外部提示音
    let alertSound = new Audio('https://actions.google.com/sounds/v1/alarms/beep_short.ogg');
    let crosshair = { x: 0, y: 0, active: false };
    let lastAlertMsg = "", lastAlertBarIdx = -999; 
    const trendline = JSON.parse(localStorage.getItem('btc_trendline') || '{"p1_idx":450,"p1_val":65000,"p2_idx":480,"p2_val":66000,"active":null}');
    const savedCalc = JSON.parse(localStorage.getItem('btc_calc_data') || '{"e":"","s":"","l":"","m":""}');
    let viewOffset = -25, visibleBars = 60, lastProcessedTime = 0, lastX = 0, lastDist = 0;
    const canvas = document.getElementById('chartCanvas'), ctx = canvas.getContext('2d'), dpr = window.devicePixelRatio || 1;

    window.onload = () => {
        document.getElementById('calcE').value = savedCalc.e; document.getElementById('calcS').value = savedCalc.s;
        document.getElementById('calcL').value = savedCalc.l; document.getElementById('calcM').value = savedCalc.m;
        const name = localStorage.getItem('btc_audio_name'); if(name) document.getElementById('audioNameDisp').innerText = "音效: " + name;
        init();
    };

    function stopAlarm() { alertSound.pause(); alertSound.currentTime = 0; document.getElementById('alertOverlay').style.display = "none"; }
    function saveCalcData() { const data = { e: document.getElementById('calcE').value, s: document.getElementById('calcS').value, l: document.getElementById('calcL').value, m: document.getElementById('calcM').value }; localStorage.setItem('btc_calc_data', JSON.stringify(data)); updateAllLevs(); }
    document.querySelectorAll('.calc-panel input').forEach(el => el.addEventListener('input', saveCalcData));
    function coreCalc(e, s, loss, margin) { if (!e || !s || !loss || !margin || Math.abs(e - s) < 1) return "--"; return ((loss * e) / (Math.abs(e - s) * margin)).toFixed(1); }
    function updateAllLevs() { const loss = parseFloat(document.getElementById('calcL').value), margin = parseFloat(document.getElementById('calcM').value); document.getElementById('calcResVal').innerText = coreCalc(parseFloat(document.getElementById('calcE').value), parseFloat(document.getElementById('calcS').value), loss, margin) + " x"; document.getElementById('lLev').innerText = coreCalc(parseFloat(document.getElementById('lE').innerText), parseFloat(document.getElementById('lS').innerText), loss, margin) + "x"; document.getElementById('sLev').innerText = coreCalc(parseFloat(document.getElementById('sE').innerText), parseFloat(document.getElementById('sS').innerText), loss, margin) + "x"; }

    function draw() {
        if (!rawData.length) return;
        const w = canvas.width / dpr, h = canvas.height / dpr, data = [...rawData].reverse();
        ctx.clearRect(0, 0, w, h);
        const endIdx = data.length - 1 - Math.floor(viewOffset), startIdx = Math.max(-100, endIdx - Math.round(visibleBars)); 
        const visibleData = data.slice(Math.max(0, startIdx), Math.min(data.length, endIdx + 1));
        const high = Math.max(...visibleData.map(d => d.h)), low = Math.min(...visibleData.map(d => d.l)), priceRange = (high - low) || 1;
        const padding = h * 0.15;
        const getY = (v) => h - padding - ((v - low) / priceRange) * (h - padding * 2);
        const getX = (idx) => 15 + (idx - startIdx) * ((w - 30) / (endIdx - startIdx));
        const x1 = getX(trendline.p1_idx), y1 = getY(trendline.p1_val), x2 = getX(trendline.p2_idx), y2 = getY(trendline.p2_val);
        const slope = (y2 - y1) / (x2 - x1 || 1), intercept = y1 - slope * x1;
        
        ctx.strokeStyle = "rgba(242,201,76,0.8)"; ctx.lineWidth = 2; ctx.setLineDash([6, 4]);
        ctx.beginPath(); ctx.moveTo(0, slope * 0 + intercept); ctx.lineTo(w, slope * w + intercept); ctx.stroke(); ctx.setLineDash([]);

        let bullOB = null, bearOB = null, highestP = -Infinity, lowestP = Infinity, highIdx = -1, lowIdx = -1;
        for(let i = Math.min(trendline.p1_idx, trendline.p2_idx); i <= Math.max(trendline.p1_idx, trendline.p2_idx); i++) {
            if(!data[i]) continue;
            if(data[i].h > highestP) { highestP = data[i].h; highIdx = i; }
            if(data[i].l < lowestP) { lowestP = data[i].l; lowIdx = i; }
        }
        if(highIdx !== -1) bearOB = data[highIdx];
        if(lowIdx !== -1) bullOB = data[lowIdx];

        if(bullOB) { ctx.fillStyle="rgba(63,185,80,0.18)"; ctx.fillRect(0, getY(bullOB.h), w, Math.abs(getY(bullOB.l)-getY(bullOB.h))); document.getElementById('lE').innerText = bullOB.h.toFixed(0); document.getElementById('lS').innerText = bullOB.l.toFixed(0); }
        if(bearOB) { ctx.fillStyle="rgba(248,81,73,0.18)"; ctx.fillRect(0, getY(bearOB.h), w, Math.abs(getY(bearOB.l)-getY(bearOB.h))); document.getElementById('sE').innerText = bearOB.l.toFixed(0); document.getElementById('sS').innerText = bearOB.h.toFixed(0); }

        if(isEditMode) { ctx.fillStyle="#1f6feb"; ctx.beginPath(); ctx.arc(x1,y1,22,0,7); ctx.fill(); ctx.beginPath(); ctx.arc(x2,y2,22,0,7); ctx.fill(); }
        
        data.forEach((d,i) => { 
            if(i<startIdx || i > endIdx) return; 
            const x=getX(i);
            const date = new Date(d.t); 
            const prev = data[i-1] ? new Date(data[i-1].t) : null; 
            const next = data[i+1] ? new Date(data[i+1].t) : null;
            
            // 基础颜色逻辑
            let color = d.c >= d.o ? "#3fb950" : "#f85149";
            
            // 核心修正：仅当它是历史收盘且为一天最后一根时显示紫色；实时K线保持红/绿
            if(!prev || date.getDate() !== prev.getDate()) {
                color = "#007aff"; // 开盘蓝
            } else if(next && date.getDate() !== next.getDate()) {
                color = "#a371f7"; // 收盘紫 (必须有下一根证明这根已收盘且是最后一根)
            }

            ctx.strokeStyle=color; ctx.beginPath(); ctx.moveTo(x,getY(d.h)); ctx.lineTo(x,getY(d.l)); ctx.stroke(); 
            const bw=Math.max(1.8,((w-30)/visibleBars)*0.8); ctx.fillStyle=color; ctx.fillRect(x-bw/2, getY(Math.max(d.o,d.c)), bw, Math.max(0.5,Math.abs(getY(d.o)-getY(d.c)))); 
        });

        if(crosshair.active) {
            ctx.strokeStyle = "rgba(255,255,255,0.4)"; ctx.lineWidth = 1; ctx.setLineDash([4, 4]);
            ctx.beginPath(); ctx.moveTo(0, crosshair.y); ctx.lineTo(w, crosshair.y); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(crosshair.x, 0); ctx.lineTo(crosshair.x, h); ctx.stroke(); ctx.setLineDash([]);
            const hoveredPrice = low + (h - padding - crosshair.y) / (h - padding * 2) * priceRange;
            ctx.fillStyle = "rgba(242,201,76,0.9)"; ctx.fillRect(w - 75, crosshair.y - 10, 70, 20);
            ctx.fillStyle = "#000"; ctx.font = "bold 12px monospace"; ctx.fillText(hoveredPrice.toFixed(1), w - 70, crosshair.y + 4);
        }
        checkSignals(data[data.length-1], bullOB, bearOB, slope, intercept, getY, getX, data.length-1, low, h, padding, priceRange);
    }

    function checkSignals(last, bull, bear, slope, intercept, getY, getX, currentIdx, low, h, padding, priceRange) {
        const overlay = document.getElementById('alertOverlay');
        const r = 900 - (Math.floor(Date.now()/1000) % 900);
        document.getElementById('curPrice').innerText = last.c.toFixed(1);
        document.getElementById('countdown').innerText = `收线 ${Math.floor(r/60)}:${(r%60).toString().padStart(2,'0')}`;
        const isClosingWindow = r < 180; 
        if (isMonitoring && isClosingWindow) {
            let msg = "", cls = "";
            const lineY = slope * getX(currentIdx) + intercept;
            const bodyMax = getY(Math.max(last.o, last.c)), bodyMin = getY(Math.min(last.o, last.c));
            if (slope < 0) { 
                if (bodyMin > lineY) { msg = "⚠️ 趋势线实体跌破!"; cls = "alert-break"; }
                else if (getY(last.l) > lineY && bodyMin <= lineY) { msg = "🔔 趋势线支撑回踩"; cls = "alert-long"; }
            } else { 
                if (bodyMax < lineY) { msg = "⚠️ 趋势线实体涨破!"; cls = "alert-break"; }
                else if (getY(last.h) < lineY && bodyMax >= lineY) { msg = "🔔 趋势线阻力回踩"; cls = "alert-short"; }
            }
            if(!msg) {
                if(bull && last.c <= bull.h && last.c >= bull.l) { msg = "🔔 价格进入看涨 OB"; cls = "alert-long"; }
                if(bear && last.c >= bear.l && last.c <= bear.h) { msg = "🔔 价格进入看跌 OB"; cls = "alert-short"; }
            }
            if(msg && (msg !== lastAlertMsg || currentIdx > lastAlertBarIdx + 3)) {
                overlay.innerText = msg + " (点击静音)"; overlay.className = cls; overlay.style.display = "block";
                alertSound.loop = true; alertSound.play().catch(()=>{});
                lastAlertMsg = msg; lastAlertBarIdx = currentIdx;
            }
        }
        if (!isClosingWindow && alertSound.paused) overlay.style.display = "none";
        updateAllLevs();
    }

    function getTouchPos(e) { const rect = canvas.getBoundingClientRect(), t = e.touches[0]; if(!isLandscape) return { x: (t.clientX - rect.left), y: (t.clientY - rect.top) }; return { x: (t.clientY - rect.top), y: rect.width - (t.clientX - rect.left) }; }
    canvas.addEventListener('touchstart', e => {
        const pos = getTouchPos(e); crosshair.active = true; crosshair.x = pos.x; crosshair.y = pos.y;
        if(e.touches.length===1) {
            if(isEditMode) {
                const w=canvas.width/dpr, h=canvas.height/dpr, data=[...rawData].reverse();
                const endIdx=data.length-1-Math.floor(viewOffset), startIdx=Math.max(-100,endIdx-Math.round(visibleBars));
                const vData = data.slice(Math.max(0,startIdx),Math.min(data.length,endIdx+1));
                const high=Math.max(...vData.map(d=>d.h)), low=Math.min(...vData.map(d=>d.l));
                const getY = (v) => h - (h*0.15) - ((v - low) / ((high-low)||1)) * (h - (h*0.15)*2);
                const getX = (idx) => 15 + (idx - startIdx) * ((w - 30) / (endIdx - startIdx));
                if(Math.hypot(pos.x-getX(trendline.p1_idx),pos.y-getY(trendline.p1_val))<50) trendline.active='p1';
                else if(Math.hypot(pos.x-getX(trendline.p2_idx),pos.y-getY(trendline.p2_val))<50) trendline.active='p2';
                else { lastX=pos.x; trendline.active=null; }
            } else { lastX=pos.x; }
        } else if(e.touches.length===2) { lastDist=Math.hypot(e.touches[0].clientX-e.touches[1].clientX, e.touches[0].clientY-e.touches[1].clientY); }
    });
    canvas.addEventListener('touchmove', e => {
        e.preventDefault(); const pos = getTouchPos(e); crosshair.x = pos.x; crosshair.y = pos.y;
        if(e.touches.length===1) {
            if(!isEditMode || (isEditMode && !trendline.active)) { viewOffset=Math.max(-visibleBars, Math.min(rawData.length, viewOffset-(pos.x-lastX)/(canvas.width/dpr/visibleBars))); lastX=pos.x; 
            } else if(trendline.active) {
                const data=[...rawData].reverse(), endIdx=data.length-1-Math.floor(viewOffset), startIdx=Math.max(-100,endIdx-Math.round(visibleBars));
                const vData = data.slice(Math.max(0,startIdx),Math.min(data.length,endIdx+1));
                const high=Math.max(...vData.map(d=>d.h)), low=Math.min(...vData.map(d=>d.l));
                trendline[trendline.active+'_idx']=Math.round(startIdx+(pos.x-15)/((canvas.width/dpr-30)/(endIdx-startIdx)));
                const h = canvas.height/dpr, padding = h * 0.15; trendline[trendline.active+'_val']=low+(h-padding-pos.y)/(h-padding*2)*(high-low);
            }
        } else if(e.touches.length===2) { const dist=Math.hypot(e.touches[0].clientX-e.touches[1].clientX, e.touches[0].clientY-e.touches[1].clientY); if(lastDist>0) { visibleBars=Math.max(10,Math.min(250,visibleBars*(lastDist/dist))); lastDist=dist; } }
        requestAnimationFrame(draw);
    }, {passive:false});
    canvas.addEventListener('touchend', () => { crosshair.active = false; trendline.active=null; if(isEditMode) localStorage.setItem('btc_trendline', JSON.stringify(trendline)); requestAnimationFrame(draw); });
    function toggleLandscape() { isLandscape = !isLandscape; document.body.classList.toggle('landscape-mode', isLandscape); setTimeout(init, 100); }
    async function fetchData() { try { const res=await fetch(`https://www.okx.com/api/v5/market/candles?instId=BTC-USDT-SWAP&bar=15m&limit=500`); const json=await res.json(); if(json.data) { rawData=json.data.map(d=>({t:+d[0],o:+d[1],h:+d[2],l:+d[3],c:+d[4]})); requestAnimationFrame(draw); } } catch(e){} }
    function toggleEdit() { isEditMode = !isEditMode; document.getElementById('editBtn').classList.toggle('btn-edit', isEditMode); draw(); }
    function toggleAlert() { isMonitoring = !isMonitoring; document.getElementById('alertBtn').innerText = isMonitoring ? "守护中" : "监控暂停"; }
    document.getElementById('audioFile').addEventListener('change', e => { if (e.target.files[0]) { alertSound.src = URL.createObjectURL(e.target.files[0]); localStorage.setItem('btc_audio_name', e.target.files[0].name); alertSound.loop = true; } });
    function init() { canvas.width = canvas.clientWidth * dpr; canvas.height = canvas.clientHeight * dpr; ctx.setTransform(dpr,0,0,dpr,0,0); fetchData(); }
    setInterval(fetchData, 5000);
</script>
</body>
</html>
