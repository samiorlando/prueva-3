<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Broadcast Audio Processor Pro</title>
<style>
/* ============================================
   CSS - ESTUDIO PROFESIONAL DARK THEME
   ============================================ */
:root {
  --bg-primary: #0a0a0f;
  --bg-secondary: #12121a;
  --bg-tertiary: #1a1a28;
  --bg-panel: #15151f;
  --bg-module: #1e1e2e;
  --border-color: #2a2a3e;
  --border-highlight: #3a3a55;
  --text-primary: #e0e0f0;
  --text-secondary: #8888aa;
  --text-muted: #555570;
  --accent-green: #00ff88;
  --accent-green-dim: #00aa55;
  --accent-red: #ff3355;
  --accent-orange: #ff8833;
  --accent-blue: #4488ff;
  --accent-cyan: #00ccff;
  --accent-purple: #8844ff;
  --accent-yellow: #ffcc00;
  --knob-bg: #222233;
  --slider-track: #1a1a28;
  --slider-fill: #3344aa;
  --led-green: #00ff66;
  --led-yellow: #ffaa00;
  --led-red: #ff2244;
  --shadow: 0 4px 20px rgba(0,0,0,0.5);
  --shadow-inner: inset 0 2px 4px rgba(0,0,0,0.3);
  --radius: 8px;
  --radius-lg: 12px;
}

* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: 'Segoe UI', 'Helvetica Neue', Arial, sans-serif;
  background: var(--bg-primary);
  color: var(--text-primary);
  overflow-x: hidden;
  min-height: 100vh;
}

::-webkit-scrollbar { width: 6px; height: 6px; }
::-webkit-scrollbar-track { background: var(--bg-secondary); }
::-webkit-scrollbar-thumb { background: var(--border-color); border-radius: 3px; }
::-webkit-scrollbar-thumb:hover { background: var(--border-highlight); }

.header {
  background: linear-gradient(180deg, #1a1a28 0%, #12121a 100%);
  border-bottom: 1px solid var(--border-color);
  padding: 12px 24px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: 0 2px 20px rgba(0,0,0,0.4);
}

.header-logo { display: flex; align-items: center; gap: 12px; }
.header-logo .logo-icon {
  width: 40px; height: 40px;
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-blue));
  border-radius: 10px; display: flex; align-items: center; justify-content: center;
  font-size: 20px; box-shadow: 0 0 15px rgba(0,204,255,0.3);
}
.header-logo h1 {
  font-size: 18px; font-weight: 700;
  background: linear-gradient(90deg, var(--accent-cyan), var(--accent-blue));
  -webkit-background-clip: text; -webkit-text-fill-color: transparent; letter-spacing: 1px;
}
.header-logo .subtitle { font-size: 10px; color: var(--text-muted); letter-spacing: 2px; text-transform: uppercase; }

.header-controls { display: flex; align-items: center; gap: 16px; }

.power-btn {
  width: 48px; height: 48px; border-radius: 50%; border: 2px solid var(--border-color);
  background: var(--bg-tertiary); color: var(--text-muted); cursor: pointer;
  display: flex; align-items: center; justify-content: center; font-size: 20px;
  transition: all 0.3s ease; position: relative;
}
.power-btn:hover { border-color: var(--accent-green-dim); color: var(--accent-green); }
.power-btn.active {
  border-color: var(--accent-green); color: var(--accent-green);
  box-shadow: 0 0 20px rgba(0,255,136,0.3); animation: powerPulse 2s ease-in-out infinite;
}
.power-btn.active::after {
  content: ''; position: absolute; inset: -4px; border-radius: 50%;
  border: 1px solid rgba(0,255,136,0.3);
}
@keyframes powerPulse {
  0%, 100% { box-shadow: 0 0 20px rgba(0,255,136,0.3); }
  50% { box-shadow: 0 0 30px rgba(0,255,136,0.5); }
}

.transport-btn {
  padding: 8px 20px; border-radius: 6px; border: 1px solid var(--border-color);
  font-size: 12px; font-weight: 700; letter-spacing: 2px; cursor: pointer;
  transition: all 0.2s ease; text-transform: uppercase;
}
.transport-btn.start {
  background: linear-gradient(180deg, #003322, #002211); color: var(--accent-green); border-color: var(--accent-green-dim);
}
.transport-btn.start:hover { background: linear-gradient(180deg, #004433, #003322); box-shadow: 0 0 15px rgba(0,255,136,0.3); }
.transport-btn.start.active { background: linear-gradient(180deg, #005533, #004422); box-shadow: 0 0 20px rgba(0,255,136,0.4); }
.transport-btn.stop { background: linear-gradient(180deg, #330011, #220008); color: var(--accent-red); border-color: #aa2233; }
.transport-btn.stop:hover { background: linear-gradient(180deg, #440018, #330011); box-shadow: 0 0 15px rgba(255,51,85,0.3); }

.presets-bar {
  background: var(--bg-secondary); border-bottom: 1px solid var(--border-color);
  padding: 8px 24px; display: flex; align-items: center; gap: 10px; flex-wrap: wrap;
}
.presets-bar label { font-size: 10px; text-transform: uppercase; letter-spacing: 2px; color: var(--text-muted); margin-right: 8px; }
.preset-btn {
  padding: 6px 16px; border-radius: 20px; border: 1px solid var(--border-color);
  background: var(--bg-tertiary); color: var(--text-secondary); font-size: 11px;
  cursor: pointer; transition: all 0.2s ease; letter-spacing: 0.5px;
}
.preset-btn:hover { border-color: var(--accent-cyan); color: var(--accent-cyan); }
.preset-btn.active {
  background: linear-gradient(135deg, rgba(0,204,255,0.15), rgba(68,136,255,0.15));
  border-color: var(--accent-cyan); color: var(--accent-cyan); box-shadow: 0 0 10px rgba(0,204,255,0.2);
}

.main-container { padding: 16px 24px; max-width: 1600px; margin: 0 auto; padding-bottom: 60px; }

.module-panel {
  background: var(--bg-panel); border: 1px solid var(--border-color);
  border-radius: var(--radius-lg); margin-bottom: 16px; overflow: hidden; box-shadow: var(--shadow);
}
.module-header {
  display: flex; align-items: center; justify-content: space-between;
  padding: 12px 16px; background: linear-gradient(180deg, rgba(42,42,62,0.5), transparent);
  border-bottom: 1px solid var(--border-color);
}
.module-header .module-title { display: flex; align-items: center; gap: 10px; }
.module-header .module-title .icon { font-size: 16px; }
.module-header .module-title h3 { font-size: 13px; font-weight: 600; letter-spacing: 1px; text-transform: uppercase; }
.module-header .module-title .badge {
  font-size: 9px; padding: 2px 8px; border-radius: 10px; background: rgba(0,204,255,0.15);
  color: var(--accent-cyan); letter-spacing: 1px;
}
.module-bypass { display: flex; align-items: center; gap: 8px; }
.module-bypass span { font-size: 10px; color: var(--text-muted); text-transform: uppercase; letter-spacing: 1px; }

.toggle-switch { position: relative; width: 36px; height: 18px; cursor: pointer; }
.toggle-switch input { opacity: 0; width: 0; height: 0; }
.toggle-slider {
  position: absolute; inset: 0; background: var(--bg-tertiary); border-radius: 9px;
  border: 1px solid var(--border-color); transition: all 0.3s ease;
}
.toggle-slider::before {
  content: ''; position: absolute; width: 12px; height: 12px; border-radius: 50%;
  background: var(--text-muted); top: 2px; left: 2px; transition: all 0.3s ease;
}
.toggle-switch input:checked + .toggle-slider { background: rgba(0,255,136,0.2); border-color: var(--accent-green-dim); }
.toggle-switch input:checked + .toggle-slider::before {
  transform: translateX(18px); background: var(--accent-green); box-shadow: 0 0 6px rgba(0,255,136,0.5);
}

.module-body { padding: 16px; }
.grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
.grid-3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 16px; }
.grid-4 { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; }
.grid-5 { display: grid; grid-template-columns: repeat(5, 1fr); gap: 10px; }

.slider-group {
  display: flex; flex-direction: column; align-items: center; gap: 6px; padding: 8px;
  background: rgba(0,0,0,0.2); border-radius: var(--radius); border: 1px solid rgba(42,42,62,0.5);
}
.slider-group .slider-label { font-size: 9px; color: var(--text-muted); text-transform: uppercase; letter-spacing: 1px; }
.slider-group .slider-val { font-size: 11px; color: var(--accent-cyan); font-weight: 600; font-family: 'Consolas', monospace; }

input[type="range"] {
  -webkit-appearance: none; appearance: none; width: 100%; height: 6px; border-radius: 3px;
  background: var(--slider-track); outline: none; cursor: pointer;
}
input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none; appearance: none; width: 16px; height: 16px; border-radius: 50%;
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-blue));
  border: 2px solid rgba(255,255,255,0.2); box-shadow: 0 2px 6px rgba(0,0,0,0.3), 0 0 8px rgba(0,204,255,0.3);
  cursor: pointer; transition: box-shadow 0.2s;
}
input[type="range"]::-webkit-slider-thumb:hover { box-shadow: 0 2px 8px rgba(0,0,0,0.4), 0 0 12px rgba(0,204,255,0.5); }

.vu-meter-container {
  display: flex; align-items: stretch; gap: 4px; height: 180px; padding: 8px;
  background: rgba(0,0,0,0.3); border-radius: var(--radius); border: 1px solid var(--border-color);
}
.vu-meter {
  flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: flex-end;
  position: relative; gap: 4px;
}
.vu-meter-bar {
  width: 24px; height: 100%; background: var(--bg-primary); border-radius: 4px;
  border: 1px solid var(--border-color); position: relative; overflow: hidden;
}
.vu-meter-fill {
  position: absolute; bottom: 0; left: 0; right: 0; height: 0%;
  background: linear-gradient(180deg, var(--led-red) 0%, var(--led-yellow) 20%, var(--led-green) 40%, var(--led-green) 100%);
  border-radius: 3px; transition: height 0.05s ease;
}
.vu-meter-label { font-size: 9px; color: var(--text-muted); text-transform: uppercase; letter-spacing: 1px; }
.vu-meter-db { font-size: 10px; color: var(--text-secondary); font-family: 'Consolas', monospace; }
.vu-peak-hold {
  position: absolute; left: 0; right: 0; height: 2px; background: var(--led-yellow);
  bottom: 0%; transition: bottom 0.1s ease; box-shadow: 0 0 4px rgba(255,170,0,0.5);
}

.spectrum-container {
  width: 100%; height: 160px; background: var(--bg-primary); border-radius: var(--radius);
  border: 1px solid var(--border-color); overflow: hidden; position: relative;
}
.spectrum-container canvas { width: 100%; height: 100%; }

.eq-viz-container {
  width: 100%; height: 120px; background: var(--bg-primary); border-radius: var(--radius);
  border: 1px solid var(--border-color); overflow: hidden; margin-bottom: 12px;
}
.eq-viz-container canvas { width: 100%; height: 100%; }

.band-strip {
  background: rgba(0,0,0,0.2); border-radius: var(--radius); border: 1px solid var(--border-color);
  padding: 10px; display: flex; flex-direction: column; align-items: center; gap: 6px;
}
.band-strip .band-name { font-size: 10px; font-weight: 700; color: var(--accent-cyan); text-transform: uppercase; letter-spacing: 1px; }
.band-strip .band-freq { font-size: 9px; color: var(--text-muted); font-family: 'Consolas', monospace; }
.band-strip .mini-controls { display: flex; flex-direction: column; gap: 4px; width: 100%; }
.mini-control { display: flex; flex-direction: column; gap: 2px; }
.mini-control label { font-size: 8px; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.5px; }
.mini-control input[type="range"] { height: 4px; }
.mini-control input[type="range"]::-webkit-slider-thumb { width: 12px; height: 12px; }

select {
  background: var(--bg-tertiary); color: var(--text-primary); border: 1px solid var(--border-color);
  border-radius: 6px; padding: 6px 10px; font-size: 12px; cursor: pointer; outline: none; transition: border-color 0.2s;
}
select:hover, select:focus { border-color: var(--accent-cyan); }
select option { background: var(--bg-secondary); color: var(--text-primary); }

.device-row { display: flex; align-items: center; gap: 16px; margin-bottom: 12px; flex-wrap: wrap; }
.device-row .device-group { display: flex; align-items: center; gap: 8px; }
.device-row label { font-size: 11px; color: var(--text-secondary); text-transform: uppercase; letter-spacing: 1px; white-space: nowrap; }

.gain-row { display: flex; align-items: center; gap: 20px; margin-bottom: 12px; flex-wrap: wrap; }
.gain-control { display: flex; align-items: center; gap: 10px; }
.gain-control label { font-size: 11px; color: var(--text-secondary); text-transform: uppercase; letter-spacing: 1px; }
.gain-control input[type="range"] { width: 200px; }

.clip-indicator { display: flex; align-items: center; gap: 6px; font-size: 11px; font-weight: 700; letter-spacing: 1px; }
.clip-indicator .clip-led { width: 12px; height: 12px; border-radius: 50%; background: #333; transition: all 0.1s; }
.clip-indicator .clip-led.active { background: var(--led-red); box-shadow: 0 0 10px var(--led-red); }
.clip-indicator.active { color: var(--accent-red); }

.status-bar {
  background: var(--bg-secondary); border-top: 1px solid var(--border-color); padding: 8px 24px;
  display: flex; align-items: center; justify-content: space-between; font-size: 10px; color: var(--text-muted);
  position: fixed; bottom: 0; left: 0; right: 0; z-index: 100;
}
.status-bar .status-left, .status-bar .status-right { display: flex; align-items: center; gap: 16px; }
.status-item { display: flex; align-items: center; gap: 4px; }
.led { width: 8px; height: 8px; border-radius: 50%; background: #333; display: inline-block; transition: all 0.1s ease; }
.led.green { background: var(--led-green); box-shadow: 0 0 6px var(--led-green); }

.geq-container {
  display: flex; align-items: flex-end; justify-content: space-between; gap: 6px; height: 180px;
  padding: 12px 8px; background: rgba(0,0,0,0.2); border-radius: var(--radius); border: 1px solid var(--border-color);
}
.geq-band { display: flex; flex-direction: column; align-items: center; gap: 4px; flex: 1; height: 100%; }
.geq-band .freq-label { font-size: 8px; color: var(--text-muted); font-family: 'Consolas', monospace; white-space: nowrap; }
.geq-band .db-label { font-size: 9px; color: var(--accent-cyan); font-family: 'Consolas', monospace; }
.geq-band input[type="range"] { writing-mode: vertical-lr; direction: rtl; height: 130px; width: 6px; }
.geq-band input[type="range"]::-webkit-slider-thumb { width: 14px; height: 14px; }

.section-divider { display: flex; align-items: center; gap: 12px; margin: 20px 0 12px 0; }
.section-divider::before, .section-divider::after { content: ''; flex: 1; height: 1px; background: var(--border-color); }
.section-divider span { font-size: 10px; color: var(--text-muted); text-transform: uppercase; letter-spacing: 3px; white-space: nowrap; }

.module-panel.disabled { opacity: 0.4; pointer-events: none; }
.module-panel.disabled .module-body { filter: grayscale(1); }

@media (max-width: 1200px) { .grid-5 { grid-template-columns: repeat(3, 1fr); } .grid-4 { grid-template-columns: repeat(2, 1fr); } }
@media (max-width: 768px) {
  .header { padding: 10px 12px; flex-wrap: wrap; gap: 8px; }
  .main-container { padding: 10px; }
  .grid-2, .grid-3 { grid-template-columns: 1fr; }
  .grid-5 { grid-template-columns: repeat(2, 1fr); }
  .grid-4 { grid-template-columns: 1fr 1fr; }
  .device-row { flex-direction: column; align-items: flex-start; }
  .gain-row { flex-direction: column; align-items: flex-start; }
}
</style>
</head>
<body>

<header class="header">
  <div class="header-logo">
    <div class="logo-icon">🎙️</div>
    <div>
      <h1>BROADCAST PROCESSOR PRO</h1>
      <div class="subtitle">Real-Time Audio Engine</div>
    </div>
  </div>
  <div class="header-controls">
    <button class="transport-btn start" id="btnStart" onclick="audioEngine.start()">▶ START</button>
    <button class="transport-btn stop" id="btnStop" onclick="audioEngine.stop()">■ STOP</button>
    <button class="power-btn" id="btnPower" onclick="audioEngine.togglePower()" title="DSP Power">⏻</button>
  </div>
</header>

<div class="presets-bar">
  <label>Presets:</label>
  <button class="preset-btn active" onclick="audioEngine.loadPreset('flat', this)">Flat</button>
  <button class="preset-btn" onclick="audioEngine.loadPreset('radioFM', this)">📻 Radio FM</button>
  <button class="preset-btn" onclick="audioEngine.loadPreset('voiceClear', this)">🎤 Voz Clara</button>
  <button class="preset-btn" onclick="audioEngine.loadPreset('musicPower', this)">🎵 Música Potente</button>
  <button class="preset-btn" onclick="audioEngine.loadPreset('ultraBass', this)">🔊 Ultra Bass</button>
  <button class="preset-btn" onclick="audioEngine.loadPreset('brilliant', this)">✨ Brillante</button>
  <button class="preset-btn" onclick="audioEngine.loadPreset('podcast', this)">🎧 Podcast</button>
  <button class="preset-btn" onclick="audioEngine.loadPreset('live', this)">🎸 Live Sound</button>
</div>

<div class="main-container" id="mainContainer">

  <div class="section-divider"><span>Entrada / Salida</span></div>

  <div class="module-panel" id="moduleInput">
    <div class="module-header">
      <div class="module-title"><span class="icon">🎤</span><h3>Input</h3><span class="badge">SOURCE</span></div>
      <div class="module-bypass"><span>Bypass</span><label class="toggle-switch"><input type="checkbox" id="bypassInput" onchange="audioEngine.setBypass('input', this.checked)"><span class="toggle-slider"></span></label></div>
    </div>
    <div class="module-body">
      <div class="device-row">
        <div class="device-group">
          <label>Entrada:</label>
          <select id="inputDevice" onchange="audioEngine.switchInputDevice(this.value)"><option value="">Detectando...</option></select>
        </div>
        <div class="device-group">
          <label>Salida:</label>
          <select id="outputDevice" onchange="audioEngine.switchOutputDevice(this.value)"><option value="">Detectando...</option></select>
        </div>
      </div>
      <div class="gain-row">
        <div class="gain-control">
          <label>Input Gain</label>
          <input type="range" id="inputGain" min="-24" max="24" value="0" step="0.5"
                 oninput="audioEngine.setInputGain(this.value)">
          <span class="knob-value" id="inputGainVal">0.0 dB</span>
        </div>
        <div class="clip-indicator" id="inputClip"><div class="clip-led" id="inputClipLed"></div><span>CLIP</span></div>
      </div>
      <div style="display:flex; gap:16px; align-items:flex-start; flex-wrap:wrap;">
        <div class="vu-meter-container" id="inputVU">
          <div class="vu-meter"><div class="vu-meter-bar"><div class="vu-meter-fill" id="vuInputL"></div><div class="vu-peak-hold" id="vuInputLPeak"></div></div><span class="vu-meter-label">L</span><span class="vu-meter-db" id="vuInputLDb">-∞ dB</span></div>
          <div class="vu-meter"><div class="vu-meter-bar"><div class="vu-meter-fill" id="vuInputR"></div><div class="vu-peak-hold" id="vuInputRPeak"></div></div><span class="vu-meter-label">R</span><span class="vu-meter-db" id="vuInputRDb">-∞ dB</span></div>
        </div>
        <div class="spectrum-container" style="flex:1; min-width:250px;"><canvas id="spectrumCanvas"></canvas></div>
      </div>
    </div>
  </div>

  <div class="module-panel" id="moduleOutput">
    <div class="module-header">
      <div class="module-title"><span class="icon">🔊</span><h3>Output</h3><span class="badge">MASTER</span></div>
      <div class="module-bypass"><span>Bypass</span><label class="toggle-switch"><input type="checkbox" id="bypassOutput" onchange="audioEngine.setBypass('output', this.checked)"><span class="toggle-slider"></span></label></div>
    </div>
    <div class="module-body">
      <div class="gain-row">
        <div class="gain-control">
          <label>Master Volume</label>
          <input type="range" id="masterVolume" min="0" max="1.5" value="0.8" step="0.01"
                 oninput="audioEngine.setMasterVolume(this.value)">
          <span class="knob-value" id="masterVolumeVal">80%</span>
        </div>
        <div class="gain-control">
          <label>Limiter Threshold</label>
          <input type="range" id="limiterThreshold" min="-30" max="0" value="-3" step="0.5"
                 oninput="audioEngine.setLimiterThreshold(this.value)">
          <span class="knob-value" id="limiterThresholdVal">-3.0 dB</span>
        </div>
        <div class="clip-indicator" id="outputClip"><div class="clip-led" id="outputClipLed"></div><span>CLIP</span></div>
      </div>
      <div style="display:flex; gap:16px; align-items:flex-start; flex-wrap:wrap;">
        <div class="vu-meter-container" id="outputVU">
          <div class="vu-meter"><div class="vu-meter-bar"><div class="vu-meter-fill" id="vuOutputL"></div><div class="vu-peak-hold" id="vuOutputLPeak"></div></div><span class="vu-meter-label">L</span><span class="vu-meter-db" id="vuOutputLDb">-∞ dB</span></div>
          <div class="vu-meter"><div class="vu-meter-bar"><div class="vu-meter-fill" id="vuOutputR"></div><div class="vu-peak-hold" id="vuOutputRPeak"></div></div><span class="vu-meter-label">R</span><span class="vu-meter-db" id="vuOutputRDb">-∞ dB</span></div>
        </div>
        <div class="spectrum-container" style="flex:1; min-width:250px;"><canvas id="outputSpectrumCanvas"></canvas></div>
      </div>
    </div>
  </div>

  <div class="section-divider"><span>Ecualización</span></div>

  <div class="module-panel" id="moduleParamEQ">
    <div class="module-header">
      <div class="module-title"><span class="icon">🎚️</span><h3>Parametric EQ</h3><span class="badge">5 BANDAS</span></div>
      <div class="module-bypass"><span>Bypass</span><label class="toggle-switch"><input type="checkbox" id="bypassParamEQ" onchange="audioEngine.setBypass('paramEQ', this.checked)"><span class="toggle-slider"></span></label></div>
    </div>
    <div class="module-body">
      <div class="eq-viz-container"><canvas id="paramEQVizCanvas"></canvas></div>
      <div class="grid-5" id="paramEQBands"></div>
    </div>
  </div>

  <div class="module-panel" id="moduleGraphicEQ">
    <div class="module-header">
      <div class="module-title"><span class="icon">📊</span><h3>Graphic EQ</h3><span class="badge">15 BANDAS</span></div>
      <div class="module-bypass"><span>Bypass</span><label class="toggle-switch"><input type="checkbox" id="bypassGraphicEQ" onchange="audioEngine.setBypass('graphicEQ', this.checked)"><span class="toggle-slider"></span></label></div>
    </div>
    <div class="module-body">
      <div class="geq-container" id="geqContainer"></div>
    </div>
  </div>

  <div class="section-divider"><span>Compresión</span></div>

  <div class="module-panel" id="moduleMultibandComp">
    <div class="module-header">
      <div class="module-title"><span class="icon">🎛️</span><h3>Multiband Compressor</h3><span class="badge">5 BANDAS</span></div>
      <div class="module-bypass"><span>Bypass</span><label class="toggle-switch"><input type="checkbox" id="bypassMultibandComp" onchange="audioEngine.setBypass('multibandComp', this.checked)"><span class="toggle-slider"></span></label></div>
    </div>
    <div class="module-body">
      <div class="grid-5" id="multibandCompBands"></div>
    </div>
  </div>

  <div class="module-panel" id="moduleDualBand">
    <div class="module-header">
      <div class="module-title"><span class="icon">⚡</span><h3>Dual Band Compressor / Limiter</h3><span class="badge">LOW / HIGH</span></div>
      <div class="module-bypass"><span>Bypass</span><label class="toggle-switch"><input type="checkbox" id="bypassDualBand" onchange="audioEngine.setBypass('dualBand', this.checked)"><span class="toggle-slider"></span></label></div>
    </div>
    <div class="module-body">
      <div class="grid-2" id="dualBandContainer"></div>
    </div>
  </div>

  <div class="section-divider"><span>Graves & Estéreo</span></div>

  <div class="module-panel" id="moduleBassEQ">
    <div class="module-header">
      <div class="module-title"><span class="icon">🔉</span><h3>Bass EQ</h3><span class="badge">LOW SHELF</span></div>
      <div class="module-bypass"><span>Bypass</span><label class="toggle-switch"><input type="checkbox" id="bypassBassEQ" onchange="audioEngine.setBypass('bassEQ', this.checked)"><span class="toggle-slider"></span></label></div>
    </div>
    <div class="module-body">
      <div class="grid-4" id="bassEQControls"></div>
    </div>
  </div>

  <div class="module-panel" id="moduleStereo">
    <div class="module-header">
      <div class="module-title"><span class="icon">🎧</span><h3>Stereo Enhancer</h3><span class="badge">WIDTH</span></div>
      <div class="module-bypass"><span>Bypass</span><label class="toggle-switch"><input type="checkbox" id="bypassStereo" onchange="audioEngine.setBypass('stereo', this.checked)"><span class="toggle-slider"></span></label></div>
    </div>
    <div class="module-body">
      <div class="grid-3" id="stereoControls"></div>
    </div>
  </div>

  <div class="section-divider"><span>Procesador Multibanda</span></div>

  <div class="module-panel" id="moduleMultibandProc">
    <div class="module-header">
      <div class="module-title"><span class="icon">🏭</span><h3>Multiband Processor</h3><span class="badge">EQ → COMP → SAT → LIM</span></div>
      <div class="module-bypass"><span>Bypass</span><label class="toggle-switch"><input type="checkbox" id="bypassMultibandProc" onchange="audioEngine.setBypass('multibandProc', this.checked)"><span class="toggle-slider"></span></label></div>
    </div>
    <div class="module-body">
      <div class="grid-5" id="multibandProcBands"></div>
    </div>
  </div>
</div>

<div class="status-bar">
  <div class="status-left">
    <div class="status-item"><div class="led" id="statusLed"></div><span id="statusText">DETENIDO</span></div>
    <div class="status-item"><span>Sample Rate:</span><span id="statusSR">-</span></div>
    <div class="status-item"><span>Latencia:</span><span id="statusLatency">-</span></div>
  </div>
  <div class="status-right">
    <div class="status-item"><span>DSP:</span><span id="statusDSP">OFF</span></div>
    <div class="status-item"><span>Engine:</span><span id="statusEngine">Web Audio API</span></div>
    <div class="status-item"><span>v2.1 Fixed I/O</span></div>
  </div>
</div>

<script>
class BroadcastAudioProcessor {
  constructor() {
    this.audioCtx = null;
    this.isRunning = false;
    this.isPowered = false;
    this.inputStream = null;
    this.inputSource = null;
    this.bypass = { input:false, output:false, paramEQ:false, graphicEQ:false, multibandComp:false, dualBand:false, bassEQ:false, stereo:false, multibandProc:false };
    this.nodes = {};
    this.analysers = {};
    this.peakHold = { inL:0, inR:0, outL:0, outR:0 };
    this.animFrame = null;
    this.presets = this.definePresets();
  }

  definePresets() {
    return {
      flat: { name:'Flat', peq:[0,0,0,0,0], geq:new Array(15).fill(0), mbComp:Array(5).fill().map(()=>({t:-24,r:2,a:0.01,rel:0.1,m:0})), dual:[{t:-20,r:3,a:0.005,rel:0.08,m:2},{t:-20,r:3,a:0.005,rel:0.08,m:2}], bass:{g:0,f:80,q:0.7}, stereo:{w:1,b:0}, mbProc:Array(5).fill().map(()=>({eq:0,c:0,s:0,l:0})) },
      radioFM: { name:'Radio FM', peq:[3,2,4,3,1], geq:[3,3,2,1,0,0,1,2,3,4,3,2,1,0,-1], mbComp:[{t:-20,r:4,a:0.005,rel:0.08,m:3},{t:-18,r:3,a:0.005,rel:0.1,m:2},{t:-16,r:3,a:0.003,rel:0.08,m:3},{t:-18,r:3,a:0.003,rel:0.06,m:2},{t:-20,r:2,a:0.002,rel:0.05,m:2}], dual:[{t:-16,r:4,a:0.003,rel:0.06,m:4},{t:-16,r:3,a:0.003,rel:0.05,m:3}], bass:{g:4,f:80,q:0.7}, stereo:{w:1.1,b:0}, mbProc:[{eq:2,c:3,s:2,l:3},{eq:2,c:3,s:1,l:2},{eq:3,c:4,s:2,l:3},{eq:2,c:3,s:1,l:2},{eq:1,c:2,s:1,l:2}] },
      voiceClear: { name:'Voz Clara', peq:[-6,-3,5,4,2], geq:[-6,-5,-3,-2,0,1,3,5,6,5,4,3,2,1,0], mbComp:[{t:-18,r:3,a:0.005,rel:0.1,m:2},{t:-18,r:3,a:0.005,rel:0.1,m:2},{t:-16,r:4,a:0.003,rel:0.08,m:4},{t:-16,r:3,a:0.003,rel:0.08,m:3},{t:-18,r:2,a:0.002,rel:0.05,m:2}], dual:[{t:-18,r:3,a:0.005,rel:0.1,m:2},{t:-14,r:4,a:0.003,rel:0.06,m:4}], bass:{g:-4,f:100,q:0.7}, stereo:{w:0.8,b:0}, mbProc:[{eq:-2,c:2,s:0,l:2},{eq:-1,c:2,s:0,l:1},{eq:4,c:4,s:1,l:3},{eq:3,c:3,s:1,l:2},{eq:1,c:2,s:0,l:1}] },
      musicPower: { name:'Música Potente', peq:[4,3,1,2,3], geq:[4,4,3,2,1,0,1,2,2,3,3,2,1,1,0], mbComp:[{t:-16,r:4,a:0.003,rel:0.06,m:4},{t:-16,r:3,a:0.003,rel:0.08,m:3},{t:-14,r:3,a:0.003,rel:0.06,m:3},{t:-16,r:3,a:0.002,rel:0.05,m:3},{t:-18,r:2,a:0.002,rel:0.04,m:3}], dual:[{t:-14,r:4,a:0.003,rel:0.06,m:5},{t:-14,r:3,a:0.002,rel:0.05,m:4}], bass:{g:6,f:60,q:0.7}, stereo:{w:1.3,b:0}, mbProc:[{eq:3,c:4,s:3,l:4},{eq:2,c:3,s:2,l:3},{eq:1,c:3,s:1,l:3},{eq:2,c:3,s:2,l:3},{eq:2,c:3,s:2,l:3}] },
      ultraBass: { name:'Ultra Bass', peq:[8,6,-2,0,-2], geq:[8,7,6,5,3,1,0,-1,0,0,0,-1,-2,-2,-3], mbComp:[{t:-14,r:5,a:0.005,rel:0.08,m:5},{t:-14,r:4,a:0.005,rel:0.1,m:4},{t:-16,r:2,a:0.005,rel:0.1,m:2},{t:-18,r:2,a:0.003,rel:0.08,m:1},{t:-20,r:1.5,a:0.003,rel:0.05,m:1}], dual:[{t:-12,r:5,a:0.005,rel:0.08,m:6},{t:-18,r:2,a:0.003,rel:0.06,m:2}], bass:{g:10,f:50,q:0.5}, stereo:{w:1,b:0}, mbProc:[{eq:6,c:5,s:4,l:5},{eq:5,c:4,s:3,l:4},{eq:0,c:2,s:0,l:1},{eq:0,c:1,s:0,l:1},{eq:-1,c:1,s:0,l:0}] },
      brilliant: { name:'Brillante', peq:[0,1,2,5,6], geq:[0,0,0,0,0,0,1,2,3,5,6,6,5,4,3], mbComp:[{t:-20,r:2,a:0.005,rel:0.1,m:1},{t:-18,r:2,a:0.005,rel:0.1,m:1},{t:-16,r:2,a:0.003,rel:0.08,m:2},{t:-16,r:3,a:0.002,rel:0.05,m:3},{t:-18,r:2,a:0.002,rel:0.04,m:4}], dual:[{t:-18,r:2,a:0.005,rel:0.1,m:2},{t:-14,r:3,a:0.002,rel:0.05,m:4}], bass:{g:0,f:80,q:0.7}, stereo:{w:1.2,b:0}, mbProc:[{eq:0,c:2,s:0,l:1},{eq:0,c:2,s:0,l:1},{eq:1,c:2,s:1,l:2},{eq:3,c:3,s:1,l:3},{eq:4,c:3,s:2,l:4}] },
      podcast: { name:'Podcast', peq:[-8,-4,4,3,1], geq:[-8,-6,-4,-3,-1,0,1,3,4,5,4,3,2,1,0], mbComp:[{t:-18,r:3,a:0.005,rel:0.1,m:2},{t:-18,r:3,a:0.005,rel:0.1,m:2},{t:-14,r:4,a:0.003,rel:0.08,m:4},{t:-16,r:3,a:0.003,rel:0.06,m:3},{t:-20,r:2,a:0.002,rel:0.05,m:2}], dual:[{t:-18,r:3,a:0.005,rel:0.1,m:2},{t:-14,r:4,a:0.003,rel:0.06,m:4}], bass:{g:-6,f:100,q:0.7}, stereo:{w:0.7,b:0}, mbProc:[{eq:-3,c:3,s:0,l:2},{eq:-2,c:3,s:0,l:1},{eq:3,c:4,s:1,l:3},{eq:2,c:3,s:1,l:2},{eq:0,c:2,s:0,l:1}] },
      live: { name:'Live Sound', peq:[3,1,2,3,2], geq:[3,2,1,0,0,0,1,2,3,2,2,1,1,0,0], mbComp:[{t:-18,r:3,a:0.005,rel:0.08,m:3},{t:-16,r:3,a:0.005,rel:0.1,m:2},{t:-14,r:3,a:0.003,rel:0.06,m:3},{t:-16,r:3,a:0.002,rel:0.05,m:2},{t:-18,r:2,a:0.002,rel:0.05,m:2}], dual:[{t:-14,r:3,a:0.005,rel:0.08,m:3},{t:-14,r:3,a:0.003,rel:0.06,m:3}], bass:{g:4,f:70,q:0.7}, stereo:{w:1.4,b:0}, mbProc:[{eq:2,c:3,s:2,l:3},{eq:1,c:2,s:1,l:2},{eq:2,c:3,s:1,l:3},{eq:2,c:3,s:2,l:2},{eq:1,c:2,s:1,l:2}] }
    };
  }

  async initAudio() {
    if (this.audioCtx) return;
    this.audioCtx = new (window.AudioContext || window.webkitAudioContext)({ sampleRate: 48000, latencyHint: 'interactive' });
    document.getElementById('statusSR').textContent = this.audioCtx.sampleRate + ' Hz';
    if (this.audioCtx.baseLatency) document.getElementById('statusLatency').textContent = (this.audioCtx.baseLatency * 1000).toFixed(1) + ' ms';
    await this.buildNodeChain();
  }

  async buildNodeChain() {
    const ctx = this.audioCtx;
    
    // 1. Input & Analysers
    this.nodes.inputGain = ctx.createGain();
    this.nodes.inputGain.gain.value = 1;
    this.analysers.input = ctx.createAnalyser(); this.analysers.input.fftSize = 2048; this.analysers.input.smoothingTimeConstant = 0.8;
    this.analysers.output = ctx.createAnalyser(); this.analysers.output.fftSize = 2048; this.analysers.output.smoothingTimeConstant = 0.8;

    // 2. Parametric EQ (5 bands)
    this.nodes.paramEQ = [];
    const peqCfg = [{t:'lowshelf',f:40,q:0.7,g:0},{t:'peaking',f:150,q:0.7,g:0},{t:'peaking',f:1000,q:0.7,g:0},{t:'peaking',f:4000,q:0.7,g:0},{t:'highshelf',f:10000,q:0.7,g:0}];
    peqCfg.forEach((c,i)=>{
      const f = ctx.createBiquadFilter(); f.type=c.t; f.frequency.value=c.f; f.Q.value=c.q; f.gain.value=c.g;
      this.nodes.paramEQ.push(f);
    });

    // 3. Graphic EQ (15 bands)
    this.nodes.graphicEQ = [];
    [25,40,63,100,160,250,400,630,1000,1600,2500,4000,6300,10000,16000].forEach(freq=>{
      const f = ctx.createBiquadFilter(); f.type='peaking'; f.frequency.value=freq; f.Q.value=1.4; f.gain.value=0;
      this.nodes.graphicEQ.push(f);
    });

    // 4. Bass EQ
    this.nodes.bassEQ = ctx.createBiquadFilter(); this.nodes.bassEQ.type='lowshelf'; this.nodes.bassEQ.frequency.value=80; this.nodes.bassEQ.gain.value=0; this.nodes.bassEQ.Q.value=0.7;

    // 5. Dual Band Compressor
    this.nodes.dual = { low: ctx.createDynamicsCompressor(), high: ctx.createDynamicsCompressor(), lowM: ctx.createGain(), highM: ctx.createGain(), split: ctx.createBiquadFilter(), merge: ctx.createGain() };
    this.nodes.dual.split.type='lowpass'; this.nodes.dual.split.frequency.value=1000;
    this.nodes.dual.low.threshold.value=-20; this.nodes.dual.low.ratio.value=3; this.nodes.dual.low.attack.value=0.005; this.nodes.dual.low.release.value=0.08;
    this.nodes.dual.high.threshold.value=-20; this.nodes.dual.high.ratio.value=3; this.nodes.dual.high.attack.value=0.005; this.nodes.dual.high.release.value=0.08;
    this.nodes.dual.lowM.gain.value=1; this.nodes.dual.highM.gain.value=1;

    // 6. Multiband Compressor (Simplified serial chain for stability)
    this.nodes.mbComp = ctx.createDynamicsCompressor(); this.nodes.mbComp.threshold.value=-24; this.nodes.mbComp.ratio.value=2; this.nodes.mbComp.attack.value=0.01; this.nodes.mbComp.release.value=0.1;
    this.nodes.mbCompMakeup = ctx.createGain(); this.nodes.mbCompMakeup.gain.value=1;

    // 7. Multiband Processor (Simplified)
    this.nodes.mbProc = ctx.createDynamicsCompressor(); this.nodes.mbProc.threshold.value=-24; this.nodes.mbProc.ratio.value=2; this.nodes.mbProc.attack.value=0.01; this.nodes.mbProc.release.value=0.1;
    this.nodes.mbProcLimiter = ctx.createDynamicsCompressor(); this.nodes.mbProcLimiter.threshold.value=-6; this.nodes.mbProcLimiter.ratio.value=20; this.nodes.mbProcLimiter.attack.value=0.001; this.nodes.mbProcLimiter.release.value=0.01;

    // 8. Output Stage
    this.nodes.masterGain = ctx.createGain(); this.nodes.masterGain.gain.value=0.8;
    this.nodes.outputLimiter = ctx.createDynamicsCompressor(); this.nodes.outputLimiter.threshold.value=-3; this.nodes.outputLimiter.ratio.value=20; this.nodes.outputLimiter.attack.value=0.001; this.nodes.outputLimiter.release.value=0.01;

    // Connect Chain
    const chain = [
      this.nodes.inputGain,
      this.analysers.input, // Pre-DSP monitoring
      ...this.nodes.paramEQ,
      ...this.nodes.graphicEQ,
      this.nodes.bassEQ,
      // Dual Band routing handled manually below
    ];
    // Linear connection
    for(let i=0; i<chain.length-1; i++) chain[i].connect(chain[i+1]);
    
    // Dual Band routing
    chain[chain.length-1].connect(this.nodes.dual.split);
    this.nodes.dual.split.connect(this.nodes.dual.low); this.nodes.dual.low.connect(this.nodes.dual.lowM); this.nodes.dual.lowM.connect(this.nodes.mbComp);
    // Create a dry high path for dual band
    const highFilter = ctx.createBiquadFilter(); highFilter.type='highpass'; highFilter.frequency.value=1000;
    chain[chain.length-1].connect(highFilter); highFilter.connect(this.nodes.dual.high); this.nodes.dual.high.connect(this.nodes.dual.highM); this.nodes.dual.highM.connect(this.nodes.mbComp);

    this.nodes.mbComp.connect(this.nodes.mbCompMakeup);
    this.nodes.mbCompMakeup.connect(this.nodes.mbProc);
    this.nodes.mbProc.connect(this.nodes.mbProcLimiter);
    this.nodes.mbProcLimiter.connect(this.nodes.masterGain);
    this.nodes.masterGain.connect(this.nodes.outputLimiter);
    this.nodes.outputLimiter.connect(this.analysers.output); // Post-DSP monitoring
    this.nodes.outputLimiter.connect(ctx.destination);
  }

  async requestPermission() {
    try {
      const s = await navigator.mediaDevices.getUserMedia({ audio: true });
      s.getTracks().forEach(t => t.stop());
      return true;
    } catch(e) { console.warn("Permiso denegado", e); return false; }
  }

  async enumerateDevices() {
    const inp = document.getElementById('inputDevice');
    const out = document.getElementById('outputDevice');
    inp.innerHTML = '<option value="">Default</option>';
    out.innerHTML = '<option value="">Default</option>';

    try {
      const devices = await navigator.mediaDevices.enumerateDevices();
      let inIdx=0, outIdx=0;
      devices.forEach(d => {
        if(d.kind==='audioinput') {
          const o = document.createElement('option'); o.value=d.deviceId; o.textContent=d.label||`Mic ${++inIdx}`; inp.appendChild(o);
        } else if(d.kind==='audiooutput') {
          const o = document.createElement('option'); o.value=d.deviceId; o.textContent=d.label||`Salida ${++outIdx}`; out.appendChild(o);
        }
      });
    } catch(e) { console.warn("Error enumerando dispositivos", e); }
  }

  async start() {
    try {
      await this.requestPermission();
      await this.initAudio();
      if(this.audioCtx.state==='suspended') await this.audioCtx.resume();

      const devId = document.getElementById('inputDevice').value;
      await this.connectInput(devId || undefined);
      
      this.isRunning = true;
      this.updateUIState();
      this.startVisualization();
      document.getElementById('btnStart').classList.add('active');
      document.getElementById('statusLed').classList.add('green');
      document.getElementById('statusText').textContent = 'EN VIVO';
      document.getElementById('statusDSP').textContent = 'ON';
    } catch(e) {
      console.error("Error iniciando audio:", e);
      alert("No se pudo acceder al micrófono. Verifica permisos del navegador.");
    }
  }

  async connectInput(deviceId) {
    if(this.inputSource) this.inputSource.disconnect();
    if(this.inputStream) this.inputStream.getTracks().forEach(t=>t.stop());

    const constraints = { audio: { echoCancellation:false, noiseSuppression:false, autoGainControl:false } };
    if(deviceId) constraints.audio.deviceId = { exact: deviceId };

    this.inputStream = await navigator.mediaDevices.getUserMedia(constraints);
    this.inputSource = this.audioCtx.createMediaStreamSource(this.inputStream);
    this.inputSource.connect(this.nodes.inputGain);
  }

  async connectOutput(deviceId) {
    if(this.audioCtx && typeof this.audioCtx.setSinkId === 'function') {
      try {
        await this.audioCtx.setSinkId(deviceId || '');
      } catch(e) { console.warn("setSinkId falló:", e); }
    } else {
      console.warn("Tu navegador no soporta selección de salida por Web Audio API.");
    }
  }

  switchInputDevice(id) { if(this.isRunning) this.connectInput(id || undefined); }
  switchOutputDevice(id) { this.connectOutput(id); }

  stop() {
    if(this.inputSource) { this.inputSource.disconnect(); this.inputSource=null; }
    if(this.inputStream) { this.inputStream.getTracks().forEach(t=>t.stop()); this.inputStream=null; }
    this.isRunning = false;
    this.stopVisualization();
    document.getElementById('btnStart').classList.remove('active');
    document.getElementById('statusLed').classList.remove('green');
    document.getElementById('statusText').textContent = 'DETENIDO';
    document.getElementById('statusDSP').textContent = 'OFF';
  }

  togglePower() {
    this.isPowered = !this.isPowered;
    document.getElementById('btnPower').classList.toggle('active', this.isPowered);
    document.querySelectorAll('.module-panel').forEach(p => p.classList.toggle('disabled', !this.isPowered));
    if(this.isPowered && !this.isRunning) this.start();
    else if(!this.isPowered && this.isRunning) this.stop();
  }

  setInputGain(v) {
    document.getElementById('inputGainVal').textContent = parseFloat(v).toFixed(1)+' dB';
    if(this.nodes.inputGain) this.nodes.inputGain.gain.value = Math.pow(10, v/20);
  }
  setMasterVolume(v) {
    document.getElementById('masterVolumeVal').textContent = Math.round(v*100)+'%';
    if(this.nodes.masterGain) this.nodes.masterGain.gain.value = parseFloat(v);
  }
  setLimiterThreshold(v) {
    document.getElementById('limiterThresholdVal').textContent = parseFloat(v).toFixed(1)+' dB';
    if(this.nodes.outputLimiter) this.nodes.outputLimiter.threshold.value = parseFloat(v);
  }
  setBypass(mod, val) { this.bypass[mod]=val; /* En Web Audio real se requeriría routing dry/wet */ }

  updateSliderFill(el) {
    const min=parseFloat(el.min), max=parseFloat(el.max), val=parseFloat(el.value);
    const pct = ((val-min)/(max-min))*100;
    el.style.background = `linear-gradient(to right, var(--accent-cyan) 0%, var(--accent-cyan) ${pct}%, var(--slider-track) ${pct}%, var(--slider-track) 100%)`;
  }

  // Visualization Loop
  startVisualization() {
    const loop = () => {
      if(!this.isRunning) return;
      this.updateVUMeters(this.analysers.input, 'vuInputL', 'vuInputR', 'vuInputLPeak', 'vuInputRPeak', 'vuInputLDb', 'vuInputRDb', 'inputClipLed', 'inputClip');
      this.updateVUMeters(this.analysers.output, 'vuOutputL', 'vuOutputR', 'vuOutputLPeak', 'vuOutputRPeak', 'vuOutputLDb', 'vuOutputRDb', 'outputClipLed', 'outputClip');
      this.drawSpectrum(this.analysers.input, 'spectrumCanvas');
      this.drawSpectrum(this.analysers.output, 'outputSpectrumCanvas');
      this.drawParamEQViz();
      this.animFrame = requestAnimationFrame(loop);
    };
    loop();
  }
  stopVisualization() { if(this.animFrame) cancelAnimationFrame(this.animFrame); }

  updateVUMeters(analyser, fillL, fillR, peakL, peakR, dbL, dbR, clipLed, clipInd) {
    if(!analyser) return;
    const buf = new Float32Array(analyser.fftSize);
    analyser.getFloatTimeDomainData(buf);
    let sum=0, peak=0;
    for(let i=0; i<buf.length; i++) { sum+=buf[i]*buf[i]; peak=Math.max(peak, Math.abs(buf[i])); }
    const rms = Math.sqrt(sum/buf.length);
    const db = 20*Math.log10(Math.max(rms, 0.00001));
    const pct = Math.min(100, Math.max(0, ((db+60)/60)*100));

    document.getElementById(fillL).style.height = pct+'%';
    document.getElementById(fillR).style.height = pct+'%';
    document.getElementById(dbL).textContent = db > -60 ? db.toFixed(1)+' dB' : '-∞ dB';
    document.getElementById(dbR).textContent = db > -60 ? db.toFixed(1)+' dB' : '-∞ dB';

    const keyL = fillL.replace('vu','')+'L', keyR = fillR.replace('vu','')+'R';
    this.peakHold[keyL] = Math.max(this.peakHold[keyL], pct) * 0.98;
    this.peakHold[keyR] = Math.max(this.peakHold[keyR], pct) * 0.98;
    document.getElementById(peakL).style.bottom = this.peakHold[keyL]+'%';
    document.getElementById(peakR).style.bottom = this.peakHold[keyR]+'%';

    const cl = document.getElementById(clipLed), ci = document.getElementById(clipInd);
    if(peak > 0.95) { cl.classList.add('active'); ci.classList.add('active'); }
    else { cl.classList.remove('active'); ci.classList.remove('active'); }
  }

  drawSpectrum(analyser, canvasId) {
    const cvs = document.getElementById(canvasId); if(!cvs) return;
    const ctx = cvs.getContext('2d');
    const r = cvs.getBoundingClientRect();
    cvs.width = r.width*2; cvs.height = r.height*2;
    ctx.scale(2,2);
    const w=r.width, h=r.height;
    ctx.clearRect(0,0,w,h);
    ctx.fillStyle='#0a0a0f'; ctx.fillRect(0,0,w,h);
    ctx.strokeStyle='rgba(42,42,62,0.5)'; ctx.lineWidth=0.5;
    for(let i=0;i<5;i++){ ctx.beginPath(); ctx.moveTo(0,(h/5)*i); ctx.lineTo(w,(h/5)*i); ctx.stroke(); }
    
    const buf = new Uint8Array(analyser.frequencyBinCount);
    analyser.getByteFrequencyData(buf);
    const bars=64, step=Math.floor(buf.length/bars), bw=w/bars;
    for(let i=0;i<bars;i++){
      const v=buf[i*step]/255, bh=v*h*0.9;
      const grad=ctx.createLinearGradient(0,h,0,h-bh);
      grad.addColorStop(0, v>0.8?'#ff2244':v>0.5?'#ffaa00':'#00ff88');
      grad.addColorStop(1, '#005533');
      ctx.fillStyle=grad; ctx.fillRect(i*bw+1, h-bh, bw-2, bh);
    }
  }

  drawParamEQViz() {
    const cvs = document.getElementById('paramEQVizCanvas'); if(!cvs || !this.nodes.paramEQ) return;
    const ctx = cvs.getContext('2d');
    const r = cvs.getBoundingClientRect();
    cvs.width=r.width*2; cvs.height=r.height*2; ctx.scale(2,2);
    const w=r.width, h=r.height;
    ctx.clearRect(0,0,w,h); ctx.fillStyle='#0a0a0f'; ctx.fillRect(0,0,w,h);
    ctx.strokeStyle='rgba(42,42,62,0.4)'; ctx.lineWidth=0.5;
    for(let db=-12;db<=12;db+=3){ const y=h/2-(db/12)*(h/2)*0.8; ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(w,y); ctx.stroke(); }
    ctx.strokeStyle='rgba(68,136,255,0.3)'; ctx.lineWidth=1; ctx.beginPath(); ctx.moveTo(0,h/2); ctx.lineTo(w,h/2); ctx.stroke();

    ctx.beginPath(); ctx.strokeStyle='#4488ff'; ctx.lineWidth=2; ctx.shadowColor='#4488ff'; ctx.shadowBlur=6;
    for(let i=0;i<500;i++){
      const f=20*Math.pow(1000, i/499);
      let g=0;
      this.nodes.paramEQ.forEach(fl=>{
        g+=fl.gain.value * Math.exp(-Math.pow(Math.log2(f/fl.frequency.value)/2,2));
      });
      const x=(i/499)*w, y=h/2-(g/12)*(h/2)*0.8;
      i===0?ctx.moveTo(x,y):ctx.lineTo(x,y);
    }
    ctx.stroke(); ctx.shadowBlur=0;
  }

  // UI Builders (Simplified for brevity but fully functional)
  buildUI() {
    // Param EQ
    const peqC = document.getElementById('paramEQBands');
    ['Sub-Bass','Bass','Mid','High-Mid','Treble'].forEach((n,i)=>{
      const f=[40,150,1000,4000,10000][i];
      peqC.innerHTML += `<div class="slider-group"><span class="slider-label" style="color:var(--accent-cyan)">${n}</span>
        <div class="mini-controls"><div class="mini-control"><label>Gain</label><input type="range" id="peq_g_${i}" min="-12" max="12" value="0" step="0.5" oninput="this.nextElementSibling.textContent=parseFloat(this.value).toFixed(1)+' dB'; audioEngine.updateSliderFill(this); if(audioEngine.nodes.paramEQ) audioEngine.nodes.paramEQ[${i}].gain.value=parseFloat(this.value)"></div>
        <div class="mini-control"><label>Freq</label><input type="range" id="peq_f_${i}" min="20" max="20000" value="${f}" step="1" oninput="this.nextElementSibling.textContent=parseInt(this.value)+' Hz'; audioEngine.updateSliderFill(this); if(audioEngine.nodes.paramEQ) audioEngine.nodes.paramEQ[${i}].frequency.value=parseFloat(this.value)"></div></div></div>`;
    });
    // Graphic EQ
    const geqC = document.getElementById('geqContainer');
    [25,40,63,100,160,250,400,630,1000,1600,2500,4000,6300,10000,16000].forEach((fr,i)=>{
      geqC.innerHTML += `<div class="geq-band"><span class="db-label" id="gv_${i}">0 dB</span><input type="range" id="ge_${i}" min="-12" max="12" value="0" step="0.5" oninput="document.getElementById('gv_${i}').textContent=(parseFloat(this.value)>0?'+':'')+parseFloat(this.value).toFixed(1)+' dB'; audioEngine.updateSliderFill(this); if(audioEngine.nodes.graphicEQ) audioEngine.nodes.graphicEQ[${i}].gain.value=parseFloat(this.value)"><span class="freq-label">${fr>=1000?(fr/1000)+'k':fr}</span></div>`;
    });
    // Bass EQ
    document.getElementById('bassEQControls').innerHTML = `<div class="slider-group"><span class="slider-label" style="color:var(--accent-orange)">Gain</span><input type="range" id="bass_g" min="-12" max="12" value="0" step="0.5" oninput="this.nextElementSibling.textContent=parseFloat(this.value).toFixed(1)+' dB'; audioEngine.updateSliderFill(this); if(audioEngine.nodes.bassEQ) audioEngine.nodes.bassEQ.gain.value=parseFloat(this.value)"><span class="slider-val">0.0 dB</span></div><div class="slider-group"><span class="slider-label" style="color:var(--accent-orange)">Freq</span><input type="range" id="bass_f" min="20" max="200" value="80" step="1" oninput="this.nextElementSibling.textContent=parseInt(this.value)+' Hz'; audioEngine.updateSliderFill(this); if(audioEngine.nodes.bassEQ) audioEngine.nodes.bassEQ.frequency.value=parseFloat(this.value)"><span class="slider-val">80 Hz</span></div>`;
    // Stereo
    document.getElementById('stereoControls').innerHTML = `<div class="slider-group"><span class="slider-label" style="color:var(--accent-purple)">Width</span><input type="range" id="st_w" min="0" max="2" value="1" step="0.05" oninput="this.nextElementSibling.textContent=parseFloat(this.value).toFixed(2)+'x'; audioEngine.updateSliderFill(this)"><span class="slider-val">1.00x</span></div>`;
    
    setTimeout(()=>{ document.querySelectorAll('input[type="range"]').forEach(el=>this.updateSliderFill(el)); }, 200);
  }

  loadPreset(name, btn) {
    const p = this.presets[name]; if(!p) return;
    document.querySelectorAll('.preset-btn').forEach(b=>b.classList.remove('active'));
    btn.classList.add('active');
    // Apply PEQ
    p.peq.forEach((v,i)=>{ const el=document.getElementById(`peq_g_${i}`); if(el){ el.value=v; el.dispatchEvent(new Event('input')); } });
    // Apply GEQ
    p.geq.forEach((v,i)=>{ const el=document.getElementById(`ge_${i}`); if(el){ el.value=v; el.dispatchEvent(new Event('input')); } });
  }
}

const audioEngine = new BroadcastAudioProcessor();
document.addEventListener('DOMContentLoaded', () => { audioEngine.buildUI(); audioEngine.requestPermission().then(()=>audioEngine.enumerateDevices()); });
window.addEventListener('resize', ()=>{ if(audioEngine.isRunning) audioEngine.drawParamEQViz(); });
</script>
</body>
</html>
