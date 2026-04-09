<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Broadcast Audio Processor PRO</title>
<style>
  :root {
    --bg-dark: #0a0a0f;
    --bg-rack: #12121a;
    --bg-module: #1a1a25;
    --bg-module-inner: #14141e;
    --border: #2a2a3a;
    --border-light: #3a3a4a;
    --text: #c8c8d4;
    --text-dim: #6a6a7a;
    --text-bright: #e8e8f0;
    --accent-green: #00e676;
    --accent-red: #ff1744;
    --accent-orange: #ff9100;
    --accent-blue: #448aff;
    --accent-cyan: #00e5ff;
    --accent-purple: #b388ff;
    --knob-bg: #22222e;
    --meter-bg: #0d0d14;
    --led-off: #1a1a22;
    --shadow: 0 2px 8px rgba(0,0,0,0.5);
    --shadow-inner: inset 0 1px 3px rgba(0,0,0,0.4);
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: var(--bg-dark);
    color: var(--text);
    font-family: 'Segoe UI', 'Roboto', 'Helvetica Neue', sans-serif;
    min-height: 100vh;
    overflow-x: hidden;
  }

  .header {
    background: linear-gradient(180deg, #16161f 0%, #0e0e16 100%);
    border-bottom: 1px solid var(--border);
    padding: 12px 24px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: sticky;
    top: 0;
    z-index: 100;
  }

  .header-title { display: flex; align-items: center; gap: 12px; }
  .header-title h1 { font-size: 18px; font-weight: 700; color: var(--text-bright); letter-spacing: 2px; text-transform: uppercase; }
  .header-title .badge { background: linear-gradient(135deg, var(--accent-cyan), var(--accent-blue)); color: #000; font-size: 9px; font-weight: 800; padding: 2px 8px; border-radius: 3px; letter-spacing: 1px; }

  .header-status { display: flex; align-items: center; gap: 16px; }
  .status-led { width: 8px; height: 8px; border-radius: 50%; background: var(--led-off); transition: all 0.3s; }
  .status-led.active { background: var(--accent-green); box-shadow: 0 0 8px var(--accent-green), 0 0 16px rgba(0,230,118,0.3); }
  .status-led.error { background: var(--accent-red); box-shadow: 0 0 8px var(--accent-red); }
  .status-text { font-size: 11px; color: var(--text-dim); letter-spacing: 1px; text-transform: uppercase; }

  .main-container { max-width: 1400px; margin: 0 auto; padding: 16px; display: flex; flex-direction: column; gap: 12px; }

  .transport-bar { background: var(--bg-rack); border: 1px solid var(--border); border-radius: 8px; padding: 16px 24px; display: flex; align-items: center; justify-content: space-between; box-shadow: var(--shadow); flex-wrap: wrap; gap: 12px; }
  .transport-controls { display: flex; gap: 12px; align-items: center; }
  .hw-btn { width: 56px; height: 56px; border-radius: 50%; border: 2px solid var(--border-light); cursor: pointer; display: flex; align-items: center; justify-content: center; transition: all 0.2s; position: relative; font-size: 20px; }
  .hw-btn:active { transform: scale(0.95); }
  .btn-start { background: linear-gradient(145deg, #1a3a1a, #0d1f0d); border-color: #2a5a2a; color: var(--accent-green); }
  .btn-start:hover { background: linear-gradient(145deg, #1f4a1f, #122a12); border-color: #3a7a3a; box-shadow: 0 0 20px rgba(0,230,118,0.2); }
  .btn-start.active { background: linear-gradient(145deg, #0d3a0d, #0a2a0a); border-color: var(--accent-green); box-shadow: 0 0 25px rgba(0,230,118,0.3), inset 0 0 15px rgba(0,230,118,0.1); }
  .btn-stop { background: linear-gradient(145deg, #3a1a1a, #1f0d0d); border-color: #5a2a2a; color: var(--accent-red); }
  .btn-stop:hover { background: linear-gradient(145deg, #4a1f1f, #2a1212); border-color: #7a3a3a; box-shadow: 0 0 20px rgba(255,23,68,0.2); }
  .btn-stop.active { background: linear-gradient(145deg, #3a0d0d, #2a0a0a); border-color: var(--accent-red); box-shadow: 0 0 25px rgba(255,23,68,0.3), inset 0 0 15px rgba(255,23,68,0.1); }
  .transport-label { font-size: 9px; letter-spacing: 1px; text-transform: uppercase; color: var(--text-dim); margin-top: 4px; text-align: center; }
  .transport-group { display: flex; flex-direction: column; align-items: center; }

  .mode-selector { display: flex; gap: 4px; background: var(--bg-module-inner); border-radius: 6px; padding: 4px; border: 1px solid var(--border); }
  .mode-btn { padding: 8px 16px; border: none; background: transparent; color: var(--text-dim); font-size: 11px; font-weight: 600; letter-spacing: 1px; text-transform: uppercase; cursor: pointer; border-radius: 4px; transition: all 0.2s; }
  .mode-btn:hover { color: var(--text); }
  .mode-btn.active { background: linear-gradient(145deg, #2a2a3a, #22222e); color: var(--accent-cyan); box-shadow: var(--shadow-inner); }
  .mode-btn[data-mode="hard"].active { color: var(--accent-orange); }
  .mode-btn[data-mode="soft"].active { color: var(--accent-green); }

  .device-row { display: flex; gap: 12px; flex-wrap: wrap; align-items: flex-end; }
  .device-group { display: flex; flex-direction: column; gap: 4px; }
  .device-group label { font-size: 9px; color: var(--text-dim); letter-spacing: 1px; text-transform: uppercase; }
  .device-group select { background: var(--bg-module-inner); border: 1px solid var(--border); color: var(--text); padding: 8px 10px; border-radius: 4px; font-size: 12px; cursor: pointer; min-width: 220px; }
  .device-group select:focus { outline: none; border-color: var(--accent-blue); }

  .sys-audio-btn {
    padding: 8px 16px;
    background: linear-gradient(145deg, #2a1a3a, #1a0d2a);
    border: 1px solid #4a2a5a;
    color: var(--accent-purple);
    font-size: 11px;
    font-weight: 600;
    letter-spacing: 1px;
    text-transform: uppercase;
    cursor: pointer;
    border-radius: 6px;
    transition: all 0.2s;
    align-self: flex-end;
  }
  .sys-audio-btn:hover { background: linear-gradient(145deg, #3a1f4a, #2a123a); box-shadow: 0 0 15px rgba(179,136,255,0.2); }
  .sys-audio-btn.active { background: linear-gradient(145deg, #1a0d2a, #0d0616); border-color: var(--accent-purple); color: #fff; box-shadow: 0 0 20px rgba(179,136,255,0.3); }

  .rack { display: grid; grid-template-columns: repeat(auto-fit, minmax(320px, 1fr)); gap: 12px; }
  .rack-module { background: var(--bg-rack); border: 1px solid var(--border); border-radius: 8px; overflow: hidden; box-shadow: var(--shadow); transition: all 0.3s; }
  .rack-module:hover { border-color: var(--border-light); }
  .module-header { background: linear-gradient(180deg, #1e1e2a 0%, #16161f 100%); padding: 10px 16px; display: flex; align-items: center; justify-content: space-between; border-bottom: 1px solid var(--border); }
  .module-title { font-size: 11px; font-weight: 700; letter-spacing: 2px; text-transform: uppercase; color: var(--text-bright); display: flex; align-items: center; gap: 8px; }
  .module-bypass { display: flex; align-items: center; gap: 6px; }
  .bypass-btn { width: 28px; height: 14px; background: var(--accent-green); border-radius: 7px; position: relative; cursor: pointer; transition: all 0.2s; border: none; }
  .bypass-btn::after { content: ''; position: absolute; width: 10px; height: 10px; background: #fff; border-radius: 50%; top: 2px; right: 2px; transition: all 0.2s; }
  .bypass-btn.off { background: var(--accent-red); }
  .bypass-btn.off::after { right: auto; left: 2px; }
  .bypass-label { font-size: 9px; color: var(--text-dim); letter-spacing: 1px; text-transform: uppercase; }
  .module-body { padding: 16px; }

  .control-row { display: flex; gap: 16px; margin-bottom: 12px; flex-wrap: wrap; }
  .control-group { flex: 1; min-width: 80px; }
  .control-label { font-size: 9px; color: var(--text-dim); letter-spacing: 1px; text-transform: uppercase; margin-bottom: 6px; text-align: center; }
  .control-value { font-size: 10px; color: var(--accent-cyan); text-align: center; margin-top: 4px; font-family: 'Consolas', 'Courier New', monospace; }
  input[type="range"] { -webkit-appearance: none; width: 100%; height: 6px; background: var(--knob-bg); border-radius: 3px; outline: none; border: 1px solid var(--border); }
  input[type="range"]::-webkit-slider-thumb { -webkit-appearance: none; width: 16px; height: 16px; background: linear-gradient(145deg, #3a3a4a, #2a2a3a); border: 2px solid var(--border-light); border-radius: 50%; cursor: pointer; transition: all 0.15s; }
  input[type="range"]::-webkit-slider-thumb:hover { border-color: var(--accent-cyan); box-shadow: 0 0 8px rgba(0,229,255,0.3); }
  input[type="range"]::-moz-range-thumb { width: 16px; height: 16px; background: linear-gradient(145deg, #3a3a4a, #2a2a3a); border: 2px solid var(--border-light); border-radius: 50%; cursor: pointer; }

  .vu-meters { display: flex; gap: 4px; height: 160px; padding: 12px; background: var(--bg-module-inner); border-radius: 4px; border: 1px solid var(--border); }
  .vu-channel { flex: 1; display: flex; flex-direction: column; align-items: center; gap: 4px; }
  .vu-channel-label { font-size: 8px; color: var(--text-dim); letter-spacing: 1px; }
  .vu-bar-container { flex: 1; width: 24px; background: var(--meter-bg); border-radius: 3px; position: relative; overflow: hidden; border: 1px solid var(--border); }
  .vu-bar { position: absolute; bottom: 0; left: 0; right: 0; height: 0%; background: linear-gradient(0deg, var(--accent-green) 0%, var(--accent-green) 60%, var(--accent-orange) 75%, var(--accent-red) 90%, var(--accent-red) 100%); transition: height 0.05s; border-radius: 0 0 2px 2px; }
  .vu-peak { position: absolute; bottom: 0; left: 0; right: 0; height: 2px; background: #fff; transition: bottom 0.1s; z-index: 2; }

  .spectrum-container { height: 100px; background: var(--meter-bg); border-radius: 4px; border: 1px solid var(--border); overflow: hidden; position: relative; }
  .spectrum-canvas { width: 100%; height: 100%; }
  .multiband-section { display: flex; flex-direction: column; gap: 12px; }
  .band-control { background: var(--bg-module-inner); border: 1px solid var(--border); border-radius: 6px; padding: 12px; }
  .band-header { display: flex; align-items: center; justify-content: space-between; margin-bottom: 10px; }
  .band-name { font-size: 10px; font-weight: 700; letter-spacing: 1px; text-transform: uppercase; }
  .band-name.low { color: var(--accent-orange); }
  .band-name.mid { color: var(--accent-cyan); }
  .band-name.high { color: var(--accent-purple); }
  .band-controls { display: flex; gap: 12px; flex-wrap: wrap; }
  .band-control .control-group { min-width: 60px; flex: 0; }
  .band-control input[type="range"] { height: 4px; }
  .output-section { display: flex; gap: 16px; align-items: flex-end; }
  .master-volume { flex: 1; }
  .loudness-info { text-align: right; min-width: 120px; }
  .loudness-value { font-size: 28px; font-weight: 700; font-family: 'Consolas', monospace; color: var(--accent-cyan); text-shadow: 0 0 10px rgba(0,229,255,0.3); }
  .loudness-label { font-size: 9px; color: var(--text-dim); letter-spacing: 1px; text-transform: uppercase; }
  .peak-value { font-size: 14px; font-family: 'Consolas', monospace; color: var(--accent-orange); }
  .full-width { grid-column: 1 / -1; }
  ::-webkit-scrollbar { width: 6px; }
  ::-webkit-scrollbar-track { background: var(--bg-dark); }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }
  .clip-indicator { width: 12px; height: 12px; border-radius: 2px; background: var(--led-off); transition: all 0.1s; }
  .clip-indicator.clipped { background: var(--accent-red); box-shadow: 0 0 8px var(--accent-red); }
  .preset-bar { display: flex; gap: 6px; flex-wrap: wrap; }
  .preset-btn { padding: 6px 14px; background: var(--bg-module-inner); border: 1px solid var(--border); color: var(--text-dim); font-size: 10px; font-weight: 600; letter-spacing: 1px; text-transform: uppercase; cursor: pointer; border-radius: 4px; transition: all 0.2s; }
  .preset-btn:hover { border-color: var(--accent-blue); color: var(--text); }
  .preset-btn.active { border-color: var(--accent-blue); color: var(--accent-blue); background: rgba(68,138,255,0.1); }
  .waveform-container { height: 80px; background: var(--meter-bg); border-radius: 4px; border: 1px solid var(--border); overflow: hidden; }
  .waveform-canvas, .eq-viz { width: 100%; height: 100%; }
  .eq-viz { height: 60px; background: var(--meter-bg); border-radius: 4px; border: 1px solid var(--border); margin-bottom: 12px; }
  @media (max-width: 768px) {
    .rack { grid-template-columns: 1fr; }
    .transport-bar { flex-direction: column; align-items: stretch; }
    .header { flex-direction: column; gap: 8px; }
    .device-row { flex-direction: column; }
    .device-group select { min-width: 100%; }
  }
</style>
</head>
<body>

<!-- HEADER -->
<div class="header">
  <div class="header-title">
    <span style="font-size:24px">🎙️</span>
    <div><h1>Broadcast Processor</h1></div>
    <span class="badge">PRO</span>
  </div>
  <div class="header-status">
    <div class="status-led" id="statusLed"></div>
    <span class="status-text" id="statusText">STANDBY</span>
    <span class="status-text" id="cpuText">CPU: --</span>
    <span class="status-text" id="srText">SR: --</span>
  </div>
</div>

<!-- MAIN -->
<div class="main-container">
  <!-- TRANSPORT -->
  <div class="transport-bar">
    <div class="transport-controls">
      <div class="transport-group">
        <button class="hw-btn btn-start" id="btnStart" title="Iniciar procesamiento">▶</button>
        <span class="transport-label">Start</span>
      </div>
      <div class="transport-group">
        <button class="hw-btn btn-stop" id="btnStop" title="Detener">■</button>
        <span class="transport-label">Stop</span>
      </div>
    </div>

    <div class="mode-selector">
      <button class="mode-btn active" data-mode="soft" onclick="setMode('soft')">Soft</button>
      <button class="mode-btn" data-mode="smart" onclick="setMode('smart')">Smart</button>
      <button class="mode-btn" data-mode="hard" onclick="setMode('hard')">Hard</button>
    </div>

    <div class="device-row">
      <div class="device-group">
        <label>🎤 Entrada</label>
        <select id="inputDevice"><option value="">Cargando...</option></select>
      </div>
      <div class="device-group">
        <label>🔈 Salida</label>
        <select id="outputDevice"><option value="">Predeterminado</option></select>
      </div>
      <button class="sys-audio-btn" id="btnSysAudio" onclick="toggleSystemAudio()">📺 Audio Sistema</button>
    </div>
  </div>

  <!-- RACK -->
  <div class="rack">
    <!-- INPUT GAIN -->
    <div class="rack-module" id="modInput">
      <div class="module-header">
        <div class="module-title"><span class="icon">🎛️</span> Input Gain</div>
        <div class="module-bypass"><span class="clip-indicator" id="inputClip"></span></div>
      </div>
      <div class="module-body">
        <div class="control-row">
          <div class="control-group" style="flex:2">
            <div class="control-label">Ganancia de entrada</div>
            <input type="range" id="inputGain" min="-24" max="24" value="0" step="0.5" oninput="updateInputGain(this.value)">
            <div class="control-value" id="inputGainVal">0.0 dB</div>
          </div>
          <div class="control-group">
            <div class="control-label">Nivel</div>
            <div class="control-value" id="inputLevelVal" style="font-size:16px;color:var(--accent-green)">-- dB</div>
          </div>
        </div>
        <div class="waveform-container"><canvas class="waveform-canvas" id="waveformCanvas"></canvas></div>
      </div>
    </div>

    <!-- GRAPHIC EQ -->
    <div class="rack-module" id="modGEQ">
      <div class="module-header">
        <div class="module-title"><span class="icon">📊</span> EQ Gráfico</div>
        <div class="module-bypass">
          <span class="bypass-label">BYPASS</span>
          <button class="bypass-btn" id="bypassGEQ" onclick="toggleBypass('GEQ')"></button>
        </div>
      </div>
      <div class="module-body">
        <canvas class="eq-viz" id="geqCanvas"></canvas>
        <div class="control-row" id="geqControls"></div>
      </div>
    </div>

    <!-- PARAMETRIC EQ -->
    <div class="rack-module" id="modPEQ">
      <div class="module-header">
        <div class="module-title"><span class="icon">🎚️</span> EQ Paramétrico</div>
        <div class="module-bypass">
          <span class="bypass-label">BYPASS</span>
          <button class="bypass-btn" id="bypassPEQ" onclick="toggleBypass('PEQ')"></button>
        </div>
      </div>
      <div class="module-body">
        <div class="multiband-section">
          <div class="band-control">
            <div class="band-header"><span class="band-name low">Low Shelf</span></div>
            <div class="band-controls">
              <div class="control-group"><div class="control-label">Gain</div><input type="range" id="peqLowGain" min="-12" max="12" value="0" step="0.5" oninput="updatePEQ()"><div class="control-value" id="peqLowGainVal">0 dB</div></div>
              <div class="control-group"><div class="control-label">Freq</div><input type="range" id="peqLowFreq" min="20" max="500" value="100" step="1" oninput="updatePEQ()"><div class="control-value" id="peqLowFreqVal">100 Hz</div></div>
            </div>
          </div>
          <div class="band-control">
            <div class="band-header"><span class="band-name mid">Mid Peak</span></div>
            <div class="band-controls">
              <div class="control-group"><div class="control-label">Gain</div><input type="range" id="peqMidGain" min="-12" max="12" value="0" step="0.5" oninput="updatePEQ()"><div class="control-value" id="peqMidGainVal">0 dB</div></div>
              <div class="control-group"><div class="control-label">Freq</div><input type="range" id="peqMidFreq" min="200" max="8000" value="1000" step="10" oninput="updatePEQ()"><div class="control-value" id="peqMidFreqVal">1000 Hz</div></div>
              <div class="control-group"><div class="control-label">Q</div><input type="range" id="peqMidQ" min="0.1" max="10" value="1" step="0.1" oninput="updatePEQ()"><div class="control-value" id="peqMidQVal">1.0</div></div>
            </div>
          </div>
          <div class="band-control">
            <div class="band-header"><span class="band-name high">High Shelf</span></div>
            <div class="band-controls">
              <div class="control-group"><div class="control-label">Gain</div><input type="range" id="peqHighGain" min="-12" max="12" value="0" step="0.5" oninput="updatePEQ()"><div class="control-value" id="peqHighGainVal">0 dB</div></div>
              <div class="control-group"><div class="control-label">Freq</div><input type="range" id="peqHighFreq" min="2000" max="16000" value="6000" step="100" oninput="updatePEQ()"><div class="control-value" id="peqHighFreqVal">6000 Hz</div></div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- BASS ENHANCER -->
    <div class="rack-module" id="modBass">
      <div class="module-header">
        <div class="module-title"><span class="icon">🔊</span> Bass Enhancer</div>
        <div class="module-bypass">
          <span class="bypass-label">BYPASS</span>
          <button class="bypass-btn" id="bypassBass" onclick="toggleBypass('Bass')"></button>
        </div>
      </div>
      <div class="module-body">
        <div class="control-row">
          <div class="control-group"><div class="control-label">Amount</div><input type="range" id="bassAmount" min="0" max="12" value="3" step="0.5" oninput="updateBass()"><div class="control-value" id="bassAmountVal">3.0 dB</div></div>
          <div class="control-group"><div class="control-label">Freq</div><input type="range" id="bassFreq" min="40" max="200" value="80" step="1" oninput="updateBass()"><div class="control-value" id="bassFreqVal">80 Hz</div></div>
          <div class="control-group"><div class="control-label">Harmonics</div><input type="range" id="bassHarmonics" min="0" max="100" value="30" step="1" oninput="updateBass()"><div class="control-value" id="bassHarmonicsVal">30%</div></div>
        </div>
      </div>
    </div>

    <!-- MULTIBAND COMPRESSOR -->
    <div class="rack-module full-width" id="modMultiband">
      <div class="module-header">
        <div class="module-title"><span class="icon">⚡</span> Compresor Multibanda</div>
        <div class="module-bypass">
          <span class="clip-indicator" id="compClip"></span>
          <span class="clip-indicator" id="limClip"></span>
          <span style="font-size:8px;color:var(--text-dim);margin-left:4px">CLIP</span>
        </div>
      </div>
      <div class="module-body">
        <div class="preset-bar" style="margin-bottom:12px">
          <button class="preset-btn active" onclick="setCompPreset('broadcast')">Broadcast</button>
          <button class="preset-btn" onclick="setCompPreset('voice')">Voice</button>
          <button class="preset-btn" onclick="setCompPreset('music')">Music</button>
          <button class="preset-btn" onclick="setCompPreset('podcast')">Podcast</button>
          <button class="preset-btn" onclick="setCompPreset('heavy')">Heavy</button>
        </div>
        <div class="multiband-section">
          <div class="band-control">
            <div class="band-header"><span class="band-name low">LOW (20–120 Hz)</span></div>
            <div class="band-controls">
              <div class="control-group"><div class="control-label">Thresh</div><input type="range" id="lowThresh" min="-60" max="0" value="-24" step="1" oninput="updateMultiband()"><div class="control-value" id="lowThreshVal">-24 dB</div></div>
              <div class="control-group"><div class="control-label">Ratio</div><input type="range" id="lowRatio" min="1" max="20" value="4" step="0.5" oninput="updateMultiband()"><div class="control-value" id="lowRatioVal">4:1</div></div>
              <div class="control-group"><div class="control-label">Atk</div><input type="range" id="lowAttack" min="5" max="100" value="30" step="1" oninput="updateMultiband()"><div class="control-value" id="lowAttackVal">30 ms</div></div>
              <div class="control-group"><div class="control-label">Rel</div><input type="range" id="lowRelease" min="50" max="1000" value="200" step="10" oninput="updateMultiband()"><div class="control-value" id="lowReleaseVal">200 ms</div></div>
              <div class="control-group"><div class="control-label">Makeup</div><input type="range" id="lowMakeup" min="0" max="18" value="6" step="0.5" oninput="updateMultiband()"><div class="control-value" id="lowMakeupVal">6 dB</div></div>
            </div>
          </div>
          <div class="band-control">
            <div class="band-header"><span class="band-name mid">MID (120–4000 Hz)</span></div>
            <div class="band-controls">
              <div class="control-group"><div class="control-label">Thresh</div><input type="range" id="midThresh" min="-60" max="0" value="-20" step="1" oninput="updateMultiband()"><div class="control-value" id="midThreshVal">-20 dB</div></div>
              <div class="control-group"><div class="control-label">Ratio</div><input type="range" id="midRatio" min="1" max="20" value="3.5" step="0.5" oninput="updateMultiband()"><div class="control-value" id="midRatioVal">3.5:1</div></div>
              <div class="control-group"><div class="control-label">Atk</div><input type="range" id="midAttack" min="1" max="50" value="10" step="1" oninput="updateMultiband()"><div class="control-value" id="midAttackVal">10 ms</div></div>
              <div class="control-group"><div class="control-label">Rel</div><input type="range" id="midRelease" min="20" max="500" value="100" step="10" oninput="updateMultiband()"><div class="control-value" id="midReleaseVal">100 ms</div></div>
              <div class="control-group"><div class="control-label">Makeup</div><input type="range" id="midMakeup" min="0" max="18" value="5" step="0.5" oninput="updateMultiband()"><div class="control-value" id="midMakeupVal">5 dB</div></div>
            </div>
          </div>
          <div class="band-control">
            <div class="band-header"><span class="band-name high">HIGH (4000–16000 Hz)</span></div>
            <div class="band-controls">
              <div class="control-group"><div class="control-label">Thresh</div><input type="range" id="highThresh" min="-60" max="0" value="-18" step="1" oninput="updateMultiband()"><div class="control-value" id="highThreshVal">-18 dB</div></div>
              <div class="control-group"><div class="control-label">Ratio</div><input type="range" id="highRatio" min="1" max="20" value="3" step="0.5" oninput="updateMultiband()"><div class="control-value" id="highRatioVal">3:1</div></div>
              <div class="control-group"><div class="control-label">Atk</div><input type="range" id="highAttack" min="1" max="30" value="3" step="0.5" oninput="updateMultiband()"><div class="control-value" id="highAttackVal">3 ms</div></div>
              <div class="control-group"><div class="control-label">Rel</div><input type="range" id="highRelease" min="10" max="300" value="60" step="5" oninput="updateMultiband()"><div class="control-value" id="highReleaseVal">60 ms</div></div>
              <div class="control-group"><div class="control-label">Makeup</div><input type="range" id="highMakeup" min="0" max="18" value="4" step="0.5" oninput="updateMultiband()"><div class="control-value" id="highMakeupVal">4 dB</div></div>
            </div>
          </div>
        </div>
        <div style="margin-top:12px">
          <div class="control-label" style="margin-bottom:6px">Reducción de Ganancia</div>
          <div class="vu-meters" style="height:40px">
            <div class="vu-channel"><span class="vu-channel-label" style="color:var(--accent-orange)">LOW GR</span><div class="vu-bar-container"><div class="vu-bar" id="lowGrMeter" style="background:var(--accent-orange)"></div></div></div>
            <div class="vu-channel"><span class="vu-channel-label" style="color:var(--accent-cyan)">MID GR</span><div class="vu-bar-container"><div class="vu-bar" id="midGrMeter" style="background:var(--accent-cyan)"></div></div></div>
            <div class="vu-channel"><span class="vu-channel-label" style="color:var(--accent-purple)">HIGH GR</span><div class="vu-bar-container"><div class="vu-bar" id="highGrMeter" style="background:var(--accent-purple)"></div></div></div>
          </div>
        </div>
      </div>
    </div>

    <!-- STEREO ENHANCER -->
    <div class="rack-module" id="modStereo">
      <div class="module-header">
        <div class="module-title"><span class="icon">🎧</span> Stereo Enhancer</div>
        <div class="module-bypass">
          <span class="bypass-label">BYPASS</span>
          <button class="bypass-btn" id="bypassStereo" onclick="toggleBypass('Stereo')"></button>
        </div>
      </div>
      <div class="module-body">
        <div class="control-row">
          <div class="control-group"><div class="control-label">Width</div><input type="range" id="stereoWidth" min="0" max="200" value="120" step="1" oninput="updateStereo()"><div class="control-value" id="stereoWidthVal">120%</div></div>
          <div class="control-group"><div class="control-label">Low Cut</div><input type="range" id="stereoLowCut" min="100" max="500" value="200" step="10" oninput="updateStereo()"><div class="control-value" id="stereoLowCutVal">200 Hz</div></div>
        </div>
      </div>
    </div>

    <!-- FINAL PROCESSOR -->
    <div class="rack-module" id="modFinal">
      <div class="module-header">
        <div class="module-title"><span class="icon">🔥</span> Final Processor</div>
      </div>
      <div class="module-body">
        <div class="control-row">
          <div class="control-group"><div class="control-label">Density</div><input type="range" id="finalDensity" min="0" max="100" value="50" step="1" oninput="updateFinal()"><div class="control-value" id="finalDensityVal">50%</div></div>
          <div class="control-group"><div class="control-label">Punch</div><input type="range" id="finalPunch" min="0" max="100" value="40" step="1" oninput="updateFinal()"><div class="control-value" id="finalPunchVal">40%</div></div>
          <div class="control-group"><div class="control-label">Saturation</div><input type="range" id="finalSat" min="0" max="100" value="20" step="1" oninput="updateFinal()"><div class="control-value" id="finalSatVal">20%</div></div>
        </div>
      </div>
    </div>

    <!-- OUTPUT -->
    <div class="rack-module full-width" id="modOutput">
      <div class="module-header">
        <div class="module-title"><span class="icon">📡</span> Output / Master</div>
        <div class="module-bypass"><span class="clip-indicator" id="outputClip"></span><span style="font-size:8px;color:var(--text-dim);margin-left:4px">CLIP</span></div>
      </div>
      <div class="module-body">
        <div class="output-section">
          <div class="master-volume">
            <div class="control-label">Master Volume</div>
            <input type="range" id="masterVolume" min="-24" max="0" value="-3" step="0.5" oninput="updateMasterVolume(this.value)">
            <div class="control-value" id="masterVolVal">-3.0 dB</div>
          </div>
          <div class="control-group" style="min-width:100px">
            <div class="control-label">Limiter Threshold</div>
            <input type="range" id="limiterThresh" min="-12" max="0" value="-1" step="0.5" oninput="updateLimiter()">
            <div class="control-value" id="limiterThreshVal">-1.0 dB</div>
          </div>
          <div class="loudness-info">
            <div class="loudness-label">Loudness (Est.)</div>
            <div class="loudness-value" id="loudnessVal">--.- LUFS</div>
            <div class="loudness-label" style="margin-top:4px">Peak</div>
            <div class="peak-value" id="peakVal">--.- dBTP</div>
          </div>
        </div>
        <div class="vu-meters" style="margin-top:12px;height:140px">
          <div class="vu-channel"><span class="vu-channel-label">L</span><div class="vu-bar-container"><div class="vu-bar" id="vuL"></div><div class="vu-peak" id="peakL"></div></div></div>
          <div class="vu-channel"><span class="vu-channel-label">R</span><div class="vu-bar-container"><div class="vu-bar" id="vuR"></div><div class="vu-peak" id="peakR"></div></div></div>
          <div class="vu-channel" style="flex:0.5"><span class="vu-channel-label">GR</span><div class="vu-bar-container"><div class="vu-bar" id="limGrMeter" style="background:var(--accent-orange)"></div></div></div>
        </div>
      </div>
    </div>

    <!-- SPECTRUM -->
    <div class="rack-module full-width" id="modSpectrum">
      <div class="module-header">
        <div class="module-title"><span class="icon">📈</span> Spectrum Analyzer</div>
      </div>
      <div class="module-body">
        <div class="spectrum-container"><canvas class="spectrum-canvas" id="spectrumCanvas"></canvas></div>
      </div>
    </div>
  </div>
</div>

<!-- Audio element for output routing -->
<audio id="outputRouter" autoplay playsinline muted style="display:none"></audio>

<script>
// ============================================================
// BROADCAST AUDIO PROCESSOR ENGINE (CORREGIDO & OPTIMIZADO)
// ============================================================

const BAP = {
  ctx: null,
  isRunning: false,
  isSystemAudio: false,
  currentMode: 'soft',
  animFrame: null,
  frameCount: 0,
  lastSmartUpdate: 0,
  loudnessHistory: [],

  // Nodes
  source: null,
  inputGain: null,
  inputAnalyser: null,
  waveformAnalyser: null,

  geqFilters: [],
  geqBypass: false,

  peqLow: null, peqMid: null, peqHigh: null,
  peqBypass: false,

  bassFilter: null, bassGain: null, bassDry: null, bassWet: null, bassMerger: null,
  bassBypass: false,

  lowSplit: null, midSplit: null, highSplit: null,
  lowComp: null, midComp: null, highComp: null,
  lowMakeup: null, midMakeup: null, highMakeup: null,
  lowAnalyser: null, midAnalyser: null, highAnalyser: null,
  mergerNode: null,

  stereoWidthNode: null, stereoBypass: false,

  finalComp: null, finalAnalyser: null,

  limiter: null, limiterAnalyser: null,

  masterGain: null,
  masterAnalyser: null,

  destStream: null,
  outputAudioEl: null
};

const GEQ_FREQS = [60, 170, 350, 1000, 3500, 6000, 12000, 16000];

// ============================================================
// INIT
// ============================================================
async function init() {
  await enumerateDevices();
  buildGEQControls();
  updateAllDisplays();
  setMode('soft');
  setCompPreset('broadcast');
  BAP.outputAudioEl = document.getElementById('outputRouter');
  startVisualization();
}

// ============================================================
// DEVICE ENUMERATION
// ============================================================
async function enumerateDevices() {
  const inSel = document.getElementById('inputDevice');
  const outSel = document.getElementById('outputDevice');

  try {
    // Request permission to get labels
    const temp = await navigator.mediaDevices.getUserMedia({ audio: true });
    temp.getTracks().forEach(t => t.stop());
  } catch(e) { console.warn('Permiso de mic no concedido'); }

  const devices = await navigator.mediaDevices.enumerateDevices();
  
  inSel.innerHTML = '';
  outSel.innerHTML = '<option value="">Predeterminado del sistema</option>';

  devices.filter(d => d.kind === 'audioinput').forEach((d, i) => {
    const opt = document.createElement('option');
    opt.value = d.deviceId;
    opt.textContent = d.label || `Entrada ${i + 1}`;
    inSel.appendChild(opt);
  });

  devices.filter(d => d.kind === 'audiooutput').forEach((d, i) => {
    const opt = document.createElement('option');
    opt.value = d.deviceId;
    opt.textContent = d.label || `Salida ${i + 1}`;
    outSel.appendChild(opt);
  });

  if (!inSel.options.length) inSel.innerHTML = '<option value="">Sin dispositivos</option>';
}

async function changeInputDevice() {
  if (BAP.isRunning) { await stopAudio(); await startAudio(); }
}

async function changeOutputDevice() {
  const deviceId = document.getElementById('outputDevice').value;
  if (BAP.outputAudioEl && deviceId) {
    try {
      if (typeof BAP.outputAudioEl.setSinkId === 'function') {
        await BAP.outputAudioEl.setSinkId(deviceId);
      } else {
        alert('Tu navegador no soporta cambio de salida de audio.');
      }
    } catch(e) { console.error('Error setSinkId:', e); }
  }
}

document.getElementById('outputDevice').addEventListener('change', changeOutputDevice);

// ============================================================
// SYSTEM AUDIO TOGGLE
// ============================================================
async function toggleSystemAudio() {
  BAP.isSystemAudio = !BAP.isSystemAudio;
  const btn = document.getElementById('btnSysAudio');
  btn.classList.toggle('active', BAP.isSystemAudio);
  btn.textContent = BAP.isSystemAudio ? '🎤 Usar Micrófono' : '📺 Audio Sistema';
  
  if (BAP.isRunning) {
    await stopAudio();
    await startAudio();
  }
}

// ============================================================
// START / STOP
// ============================================================
async function startAudio() {
  if (BAP.isRunning) return;

  try {
    BAP.ctx = new (window.AudioContext || window.webkitAudioContext)({
      sampleRate: 48000, latencyHint: 'interactive'
    });

    let stream;
    if (BAP.isSystemAudio) {
      stream = await navigator.mediaDevices.getDisplayMedia({
        audio: { echoCancellation: false, noiseSuppression: false, autoGainControl: false },
        video: false
      });
    } else {
      const deviceId = document.getElementById('inputDevice').value;
      const constraints = {
        audio: {
          echoCancellation: false, noiseSuppression: false, autoGainControl: false, channelCount: 2
        }
      };
      if (deviceId) constraints.audio.deviceId = { exact: deviceId };
      stream = await navigator.mediaDevices.getUserMedia(constraints);
    }

    buildDSPChain();
    BAP.source = BAP.ctx.createMediaStreamSource(stream);
    BAP.inputGain = BAP.ctx.createGain();
    BAP.inputAnalyser = BAP.ctx.createAnalyser();
    BAP.waveformAnalyser = BAP.ctx.createAnalyser();
    BAP.inputAnalyser.fftSize = 2048;
    BAP.waveformAnalyser.fftSize = 4096;
    BAP.inputAnalyser.smoothingTimeConstant = 0.85;
    BAP.waveformAnalyser.smoothingTimeConstant = 0.3;

    BAP.inputGain.gain.value = dbToGain(parseFloat(document.getElementById('inputGain').value));

    BAP.source.connect(BAP.inputGain);
    BAP.inputGain.connect(BAP.waveformAnalyser);
    BAP.inputGain.connect(BAP.inputAnalyser);
    BAP.inputAnalyser.connect(BAP.geqFilters[0]);
    
    routeToOutput();

    BAP.isRunning = true;
    BAP.frameCount = 0;
    BAP.loudnessHistory = [];

    document.getElementById('statusLed').className = 'status-led active';
    document.getElementById('statusText').textContent = 'PROCESSING';
    document.getElementById('btnStart').classList.add('active');
    document.getElementById('btnStop').classList.remove('active');
    document.getElementById('srText').textContent = `SR: ${BAP.ctx.sampleRate}`;

  } catch(e) {
    console.error('Error starting audio:', e);
    document.getElementById('statusLed').className = 'status-led error';
    document.getElementById('statusText').textContent = 'ERROR';
    alert('Error al acceder al audio: ' + e.message);
  }
}

async function stopAudio() {
  if (!BAP.isRunning) return;
  BAP.isRunning = false;
  if (BAP.animFrame) { cancelAnimationFrame(BAP.animFrame); BAP.animFrame = null; }
  if (BAP.source) BAP.source.mediaStream.getTracks().forEach(t => t.stop());
  if (BAP.ctx) { await BAP.ctx.close(); BAP.ctx = null; }
  BAP.source = null;
  document.getElementById('statusLed').className = 'status-led';
  document.getElementById('statusText').textContent = 'STANDBY';
  document.getElementById('btnStart').classList.remove('active');
  document.getElementById('btnStop').classList.add('active');
  setMeter('vuL', 0); setMeter('vuR', 0); setMeter('peakL', 0); setMeter('peakR', 0);
}

function routeToOutput() {
  if (!BAP.ctx) return;
  BAP.destStream = BAP.ctx.createMediaStreamDestination();
  BAP.masterAnalyser.connect(BAP.destStream);
  BAP.outputAudioEl.srcObject = BAP.destStream;
  changeOutputDevice();
}

// ============================================================
// DSP CHAIN
// ============================================================
function buildDSPChain() {
  const c = BAP.ctx;

  // GEQ
  BAP.geqFilters = [];
  let prev = null;
  GEQ_FREQS.forEach((f, i) => {
    const filter = c.createBiquadFilter();
    filter.type = i === 0 ? 'lowshelf' : i === GEQ_FREQS.length - 1 ? 'highshelf' : 'peaking';
    filter.frequency.value = f;
    filter.gain.value = 0;
    if (i === 0) filter.Q.value = 0.7;
    else if (i === GEQ_FREQS.length - 1) filter.Q.value = 0.7;
    else filter.Q.value = 1.4;
    BAP.geqFilters.push(filter);
  });
  for (let i = 0; i < BAP.geqFilters.length - 1; i++) BAP.geqFilters[i].connect(BAP.geqFilters[i+1]);
  prev = BAP.geqFilters[BAP.geqFilters.length - 1];

  // PEQ
  BAP.peqLow = c.createBiquadFilter(); BAP.peqLow.type = 'lowshelf'; BAP.peqLow.frequency.value = 100;
  BAP.peqMid = c.createBiquadFilter(); BAP.peqMid.type = 'peaking'; BAP.peqMid.frequency.value = 1000; BAP.peqMid.Q.value = 1;
  BAP.peqHigh = c.createBiquadFilter(); BAP.peqHigh.type = 'highshelf'; BAP.peqHigh.frequency.value = 6000;
  prev.connect(BAP.peqLow); BAP.peqLow.connect(BAP.peqMid); BAP.peqMid.connect(BAP.peqHigh);
  prev = BAP.peqHigh;

  // Bass Enhancer
  BAP.bassFilter = c.createBiquadFilter(); BAP.bassFilter.type = 'lowpass'; BAP.bassFilter.frequency.value = 150; BAP.bassFilter.Q.value = 1;
  BAP.bassGain = c.createGain(); BAP.bassGain.gain.value = dbToGain(3);
  BAP.bassDry = c.createGain(); BAP.bassDry.gain.value = 1;
  BAP.bassWet = c.createGain(); BAP.bassWet.gain.value = 0.3;
  BAP.bassMerger = c.createGain();
  prev.connect(BAP.bassDry); prev.connect(BAP.bassFilter);
  BAP.bassFilter.connect(BAP.bassGain); BAP.bassGain.connect(BAP.bassWet);
  BAP.bassDry.connect(BAP.bassMerger); BAP.bassWet.connect(BAP.bassMerger);
  prev = BAP.bassMerger;

  // Multiband Split
  BAP.lowSplit = c.createBiquadFilter(); BAP.lowSplit.type = 'lowpass'; BAP.lowSplit.frequency.value = 120; BAP.lowSplit.Q.value = 0.707;
  BAP.midSplit = c.createBiquadFilter(); BAP.midSplit.type = 'bandpass'; BAP.midSplit.frequency.value = 1000; BAP.midSplit.Q.value = 0.707;
  BAP.highSplit = c.createBiquadFilter(); BAP.highSplit.type = 'highpass'; BAP.highSplit.frequency.value = 4000; BAP.highSplit.Q.value = 0.707;

  BAP.lowComp = c.createDynamicsCompressor(); BAP.midComp = c.createDynamicsCompressor(); BAP.highComp = c.createDynamicsCompressor();
  BAP.lowMakeup = c.createGain(); BAP.midMakeup = c.createGain(); BAP.highMakeup = c.createGain();
  BAP.lowAnalyser = c.createAnalyser(); BAP.midAnalyser = c.createAnalyser(); BAP.highAnalyser = c.createAnalyser();

  prev.connect(BAP.lowSplit); prev.connect(BAP.midSplit); prev.connect(BAP.highSplit);
  
  BAP.lowSplit.connect(BAP.lowComp); BAP.lowComp.connect(BAP.lowMakeup); BAP.lowMakeup.connect(BAP.lowAnalyser);
  BAP.midSplit.connect(BAP.midComp); BAP.midComp.connect(BAP.midMakeup); BAP.midMakeup.connect(BAP.midAnalyser);
  BAP.highSplit.connect(BAP.highComp); BAP.highComp.connect(BAP.highMakeup); BAP.highMakeup.connect(BAP.highAnalyser);

  BAP.mergerNode = c.createGain();
  BAP.lowAnalyser.connect(BAP.mergerNode); BAP.midAnalyser.connect(BAP.mergerNode); BAP.highAnalyser.connect(BAP.mergerNode);
  prev = BAP.mergerNode;

  // Stereo (simple width via mid/side simulation is complex, using gain/phase for stability)
  BAP.stereoMerger = c.createGain();
  prev.connect(BAP.stereoMerger); prev = BAP.stereoMerger;

  // Final Processor
  BAP.finalComp = c.createDynamicsCompressor();
  BAP.finalAnalyser = c.createAnalyser();
  prev.connect(BAP.finalComp); BAP.finalComp.connect(BAP.finalAnalyser);
  prev = BAP.finalAnalyser;

  // Limiter
  BAP.limiter = c.createDynamicsCompressor();
  BAP.limiter.threshold.value = -1; BAP.limiter.ratio.value = 20; BAP.limiter.attack.value = 0.001; BAP.limiter.release.value = 0.05; BAP.limiter.knee.value = 0;
  BAP.limiterAnalyser = c.createAnalyser();
  prev.connect(BAP.limiter); BAP.limiter.connect(BAP.limiterAnalyser);
  prev = BAP.limiterAnalyser;

  // Master
  BAP.masterGain = c.createGain(); BAP.masterGain.gain.value = dbToGain(-3);
  BAP.masterAnalyser = c.createAnalyser(); BAP.masterAnalyser.fftSize = 2048; BAP.masterAnalyser.smoothingTimeConstant = 0.8;
  prev.connect(BAP.masterGain); BAP.masterGain.connect(BAP.masterAnalyser);

  updateMultiband(); updateBass(); updatePEQ(); updateFinal(); updateBypassRouting();
}

// ============================================================
// CONTROL UPDATES
// ============================================================
function dbToGain(db) { return Math.pow(10, db / 20); }
function setMeter(id, percent) { const el = document.getElementById(id); if(el) el.style.height = `${Math.max(0, Math.min(100, percent))}%`; }

function updateInputGain(v) { document.getElementById('inputGainVal').textContent = `${parseFloat(v).toFixed(1)} dB`; if(BAP.inputGain) BAP.inputGain.gain.value = dbToGain(parseFloat(v)); }

function updateMultiband() {
  const setComp = (comp, gain, prefix) => {
    comp.threshold.value = parseFloat(document.getElementById(`${prefix}Thresh`).value);
    comp.ratio.value = parseFloat(document.getElementById(`${prefix}Ratio`).value);
    comp.attack.value = parseFloat(document.getElementById(`${prefix}Attack`).value) / 1000;
    comp.release.value = parseFloat(document.getElementById(`${prefix}Release`).value) / 1000;
    comp.knee.value = prefix === 'low' ? 6 : prefix === 'mid' ? 6 : 4;
    gain.gain.value = dbToGain(parseFloat(document.getElementById(`${prefix}Makeup`).value));
    document.getElementById(`${prefix}ThreshVal`).textContent = `${document.getElementById(`${prefix}Thresh`).value} dB`;
    document.getElementById(`${prefix}RatioVal`).textContent = `${document.getElementById(`${prefix}Ratio`).value}:1`;
    document.getElementById(`${prefix}AttackVal`).textContent = `${document.getElementById(`${prefix}Attack`).value} ms`;
    document.getElementById(`${prefix}ReleaseVal`).textContent = `${document.getElementById(`${prefix}Release`).value} ms`;
    document.getElementById(`${prefix}MakeupVal`).textContent = `${document.getElementById(`${prefix}Makeup`).value} dB`;
  };
  setComp(BAP.lowComp, BAP.lowMakeup, 'low');
  setComp(BAP.midComp, BAP.midMakeup, 'mid');
  setComp(BAP.highComp, BAP.highMakeup, 'high');
}

function updateBass() {
  const a = parseFloat(document.getElementById('bassAmount').value);
  const f = parseFloat(document.getElementById('bassFreq').value);
  const h = parseFloat(document.getElementById('bassHarmonics').value);
  if(BAP.bassFilter) BAP.bassFilter.frequency.value = f;
  if(BAP.bassGain) BAP.bassGain.gain.value = dbToGain(a);
  if(BAP.bassWet) BAP.bassWet.gain.value = h / 100;
  document.getElementById('bassAmountVal').textContent = `${a.toFixed(1)} dB`;
  document.getElementById('bassFreqVal').textContent = `${f} Hz`;
  document.getElementById('bassHarmonicsVal').textContent = `${h}%`;
}

function updateStereo() {
  document.getElementById('stereoWidthVal').textContent = `${document.getElementById('stereoWidth').value}%`;
  document.getElementById('stereoLowCutVal').textContent = `${document.getElementById('stereoLowCut').value} Hz`;
}

function updateFinal() {
  const d = parseFloat(document.getElementById('finalDensity').value);
  const p = parseFloat(document.getElementById('finalPunch').value);
  if(BAP.finalComp) {
    BAP.finalComp.threshold.value = -6 - (d/100)*18;
    BAP.finalComp.ratio.value = 1.5 + (d/100)*3;
    BAP.finalComp.attack.value = 0.001 + (1-p/100)*0.02;
    BAP.finalComp.release.value = 0.05 + (1-p/100)*0.2;
  }
  document.getElementById('finalDensityVal').textContent = `${d}%`;
  document.getElementById('finalPunchVal').textContent = `${p}%`;
  document.getElementById('finalSatVal').textContent = `${document.getElementById('finalSat').value}%`;
}

function updateMasterVolume(v) { document.getElementById('masterVolVal').textContent = `${parseFloat(v).toFixed(1)} dB`; if(BAP.masterGain) BAP.masterGain.gain.value = dbToGain(parseFloat(v)); }
function updateLimiter() { const t = parseFloat(document.getElementById('limiterThresh').value); document.getElementById('limiterThreshVal').textContent = `${t.toFixed(1)} dB`; if(BAP.limiter) BAP.limiter.threshold.value = t; }

function updatePEQ() {
  const lg = parseFloat(document.getElementById('peqLowGain').value); const lf = parseFloat(document.getElementById('peqLowFreq').value);
  const mg = parseFloat(document.getElementById('peqMidGain').value); const mf = parseFloat(document.getElementById('peqMidFreq').value); const mq = parseFloat(document.getElementById('peqMidQ').value);
  const hg = parseFloat(document.getElementById('peqHighGain').value); const hf = parseFloat(document.getElementById('peqHighFreq').value);
  
  if(BAP.peqLow) { BAP.peqLow.gain.value = BAP.peqBypass ? 0 : lg; BAP.peqLow.frequency.value = lf; }
  if(BAP.peqMid) { BAP.peqMid.gain.value = BAP.peqBypass ? 0 : mg; BAP.peqMid.frequency.value = mf; BAP.peqMid.Q.value = mq; }
  if(BAP.peqHigh) { BAP.peqHigh.gain.value = BAP.peqBypass ? 0 : hg; BAP.peqHigh.frequency.value = hf; }

  document.getElementById('peqLowGainVal').textContent = `${lg} dB`; document.getElementById('peqLowFreqVal').textContent = `${lf} Hz`;
  document.getElementById('peqMidGainVal').textContent = `${mg} dB`; document.getElementById('peqMidFreqVal').textContent = `${mf} Hz`; document.getElementById('peqMidQVal').textContent = mq.toFixed(1);
  document.getElementById('peqHighGainVal').textContent = `${hg} dB`; document.getElementById('peqHighFreqVal').textContent = `${hf} Hz`;
  drawGEQViz();
}

function toggleBypass(name) {
  const btn = document.getElementById(`bypass${name}`);
  if(!btn) return;
  const isOff = btn.classList.toggle('off');
  if(name==='GEQ') BAP.geqBypass = isOff;
  if(name==='PEQ') { BAP.peqBypass = isOff; updatePEQ(); }
  if(name==='Bass') {
    BAP.bassBypass = isOff;
    if(BAP.bassWet && BAP.bassDry) {
      BAP.bassWet.gain.value = isOff ? 0 : parseFloat(document.getElementById('bassHarmonics').value)/100;
      BAP.bassDry.gain.value = isOff ? 1 : 1;
    }
  }
  updateBypassRouting();
}

function updateBypassRouting() {
  if(!BAP.ctx) return;
  if(BAP.geqBypass) BAP.geqFilters.forEach(f => f.gain.value = 0);
}

function setMode(mode) {
  BAP.currentMode = mode;
  document.querySelectorAll('.mode-btn').forEach(b => b.classList.toggle('active', b.dataset.mode === mode));
  if(BAP.isRunning) applyMode(mode);
}

function applyMode(mode) {
  if(!BAP.lowComp) return;
  const m = {
    soft: { lt:-20, lr:3, mt:-18, mr:2.5, ht:-16, hr:2, ft:-10, fr:1.5, lth:-0.5, lr:15 },
    smart: { lt:-24, lr:4, mt:-20, mr:3.5, ht:-18, hr:3, ft:-12, fr:2, lth:-1, lr:20 },
    hard: { lt:-30, lr:6, mt:-26, mr:5, ht:-24, hr:4, ft:-15, fr:3, lth:-0.3, lr:30 }
  };
  const p = m[mode];
  BAP.lowComp.threshold.value = p.lt; BAP.lowComp.ratio.value = p.lr;
  BAP.midComp.threshold.value = p.mt; BAP.midComp.ratio.value = p.mr;
  BAP.highComp.threshold.value = p.ht; BAP.highComp.ratio.value = p.hr;
  BAP.finalComp.threshold.value = p.ft; BAP.finalComp.ratio.value = p.fr;
  BAP.limiter.threshold.value = p.lth; BAP.limiter.ratio.value = p.lr;
  updateAllDisplays();
}

function setCompPreset(preset) {
  document.querySelectorAll('.preset-btn').forEach(b => b.classList.toggle('active', b.textContent.toLowerCase() === preset));
  const presets = {
    broadcast: { lt:-24, lr:4, la:30, lrel:200, lm:6, mt:-20, mr:3.5, ma:10, mrel:100, mm:5, ht:-18, hr:3, ha:3, hrel:60, hm:4 },
    voice: { lt:-30, lr:5, la:20, lrel:150, lm:8, mt:-22, mr:4, ma:5, mrel:80, mm:7, ht:-16, hr:2.5, ha:2, hrel:50, hm:3 },
    music: { lt:-20, lr:3, la:40, lrel:250, lm:4, mt:-18, mr:2.5, ma:15, mrel:120, mm:3, ht:-14, hr:2, ha:5, hrel:80, hm:2 },
    podcast: { lt:-28, lr:4.5, la:25, lrel:180, lm:7, mt:-24, mr:4, ma:8, mrel:90, mm:6, ht:-16, hr:2.5, ha:2, hrel:50, hm:3 },
    heavy: { lt:-35, lr:8, la:15, lrel:300, lm:10, mt:-30, mr:6, ma:3, mrel:60, mm:8, ht:-26, hr:5, ha:1, hrel:40, hm:6 }
  };
  const p = presets[preset]; if(!p) return;
  ['low','mid','high'].forEach((b,i) => {
    const keys = ['Thresh','Ratio','Attack','Release','Makeup'];
    const vals = [p[b[0]+'t'], p[b[0]+'r'], p[b[0]+'a'], p[b[0]+'rel'], p[b[0]+'m']];
    keys.forEach((k,j) => document.getElementById(`${b}${k}`).value = vals[j]);
  });
  updateMultiband();
}

function updateAllDisplays() {
  ['low','mid','high'].forEach(b => {
    document.getElementById(`${b}ThreshVal`).textContent = `${document.getElementById(`${b}Thresh`).value} dB`;
    document.getElementById(`${b}RatioVal`).textContent = `${document.getElementById(`${b}Ratio`).value}:1`;
    document.getElementById(`${b}AttackVal`).textContent = `${document.getElementById(`${b}Attack`).value} ms`;
    document.getElementById(`${b}ReleaseVal`).textContent = `${document.getElementById(`${b}Release`).value} ms`;
    document.getElementById(`${b}MakeupVal`).textContent = `${document.getElementById(`${b}Makeup`).value} dB`;
  });
}

// ============================================================
// GEQ CONTROLS
// ============================================================
function buildGEQControls() {
  const c = document.getElementById('geqControls'); c.innerHTML = '';
  GEQ_FREQS.forEach((f, i) => {
    const g = document.createElement('div'); g.className = 'control-group';
    const l = document.createElement('div'); l.className = 'control-label'; l.textContent = f>=1000 ? `${f/1000}k` : `${f}`;
    const s = document.createElement('input'); s.type='range'; s.min='-12'; s.max='12'; s.value='0'; s.step='0.5'; s.dataset.index=i; s.oninput = function(){updateGEQ(this);};
    const v = document.createElement('div'); v.className='control-value'; v.id=`geqVal${i}`; v.textContent='0 dB';
    g.append(l, s, v); c.appendChild(g);
  });
  drawGEQViz();
}

function updateGEQ(slider) {
  const i = parseInt(slider.dataset.index); const v = parseFloat(slider.value);
  if(BAP.geqFilters[i] && !BAP.geqBypass) BAP.geqFilters[i].gain.value = v;
  document.getElementById(`geqVal${i}`).textContent = `${v.toFixed(1)} dB`;
  drawGEQViz();
}

function drawGEQViz() {
  const cv = document.getElementById('geqCanvas'); if(!cv) return;
  const ctx = cv.getContext('2d'); const rect = cv.getBoundingClientRect(); cv.width = rect.width*2; cv.height = rect.height*2; ctx.scale(2,2);
  const w = rect.width, h = rect.height, midY = h/2;
  ctx.clearRect(0,0,w,h);
  ctx.strokeStyle = 'rgba(255,255,255,0.05)'; ctx.lineWidth = 0.5;
  for(let i=-12; i<=12; i+=6) { const y = midY - (i/12)*(midY-4); ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(w,y); ctx.stroke(); }
  ctx.strokeStyle = 'rgba(255,255,255,0.15)'; ctx.beginPath(); ctx.moveTo(0,midY); ctx.lineTo(w,midY); ctx.stroke();

  const vals = GEQ_FREQS.map((_,i) => { const s=document.querySelector(`#geqControls input[data-index="${i}"]`); return s?parseFloat(s.value):0; });
  ctx.strokeStyle = 'rgba(0,229,255,0.6)'; ctx.lineWidth = 2; ctx.beginPath();
  vals.forEach((v,i) => {
    const x = (i/(vals.length-1))*w; const y = midY - (v/12)*(midY-4);
    if(i===0) ctx.moveTo(x,y); else {
      const px = ((i-1)/(vals.length-1))*w; const py = midY - (vals[i-1]/12)*(midY-4);
      ctx.quadraticCurveTo(px, py, (px+x)/2, (py+y)/2);
    }
  });
  ctx.stroke();
  vals.forEach((v,i) => {
    const x = (i/(vals.length-1))*w; const y = midY - (v/12)*(midY-4);
    ctx.fillStyle = v>0?'#00e676':v<0?'#ff1744':'#448aff';
    ctx.beginPath(); ctx.arc(x,y,3,0,Math.PI*2); ctx.fill();
  });
}

// ============================================================
// VISUALIZATION LOOP
// ============================================================
function startVisualization() {
  function loop() {
    BAP.animFrame = requestAnimationFrame(loop);
    if(!BAP.isRunning || !BAP.ctx) return;
    
    BAP.frameCount++;
    if(BAP.frameCount % 60 === 0) {
      document.getElementById('cpuText').textContent = `CPU: ${Math.min(99, (performance.now()/BAP.frameCount)*0.05).toFixed(1)}%`;
    }

    // Input Level & Clip
    if(BAP.inputAnalyser) {
      const d = new Uint8Array(BAP.inputAnalyser.frequencyBinCount);
      BAP.inputAnalyser.getByteTimeDomainData(d);
      let rms = 0, clipped = false;
      for(let i=0; i<d.length; i++) { const v=(d[i]-128)/128; rms+=v*v; if(d[i]===0||d[i]===255) clipped=true; }
      rms = Math.sqrt(rms/d.length);
      document.getElementById('inputLevelVal').textContent = `${(20*Math.log10(rms)).toFixed(1)} dB`;
      document.getElementById('inputClip').classList.toggle('clipped', clipped);
    }

    // Output Meters
    if(BAP.masterAnalyser) {
      const d = new Uint8Array(BAP.masterAnalyser.fftSize);
      BAP.masterAnalyser.getByteTimeDomainData(d);
      let sL=0, sR=0, c=0, pL=0, pR=0;
      for(let i=0; i<d.length; i+=2) {
        const l=(d[i]-128)/128, r=(d[i+1]-128)/128;
        sL+=l*l; sR+=r*r; c++; pL=Math.max(pL,Math.abs(l)); pR=Math.max(pR,Math.abs(r));
      }
      const rL=Math.sqrt(sL/c), rR=Math.sqrt(sR/c);
      const dbL=20*Math.log10(rL||0.0001), dbR=20*Math.log10(rR||0.0001);
      setMeter('vuL', Math.max(0,Math.min(100,((dbL+60)/60)*100)));
      setMeter('vuR', Math.max(0,Math.min(100,((dbR+60)/60)*100)));
      document.getElementById('peakL').style.bottom = `${Math.max(0,Math.min(100,((Math.log10(pL||0.0001)*20+60)/60)*100))}%`;
      document.getElementById('peakR').style.bottom = `${Math.max(0,Math.min(100,((Math.log10(pR||0.0001)*20+60)/60)*100))}%`;
      document.getElementById('outputClip').classList.toggle('clipped', pL>0.95||pR>0.95);

      const avg = (rL+rR)/2;
      if(avg>0) {
        const lu = 20*Math.log10(avg)+12;
        BAP.loudnessHistory.push(lu); if(BAP.loudnessHistory.length>100) BAP.loudnessHistory.shift();
        document.getElementById('loudnessVal').textContent = `${(BAP.loudnessHistory.reduce((a,b)=>a+b,0)/BAP.loudnessHistory.length).toFixed(1)} LUFS`;
      }
      document.getElementById('peakVal').textContent = `${Math.max(Math.log10(pL||0.0001)*20, Math.log10(pR||0.0001)*20).toFixed(1)} dBTP`;
    }

    // GR Meters
    const gr = (comp) => { try { return Math.max(0, comp.reduction*3) } catch { return 0; } };
    if(BAP.lowComp) {
      setMeter('lowGrMeter', gr(BAP.lowComp)); setMeter('midGrMeter', gr(BAP.midComp)); setMeter('highGrMeter', gr(BAP.highComp));
      setMeter('limGrMeter', gr(BAP.limiter));
      document.getElementById('compClip').classList.toggle('clipped', gr(BAP.lowComp)>15||gr(BAP.midComp)>15||gr(BAP.highComp)>15);
    }

    drawWaveform(); drawSpectrum();

    // Smart Mode Throttled
    if(BAP.currentMode==='smart' && performance.now() - BAP.lastSmartUpdate > 100) {
      BAP.lastSmartUpdate = performance.now();
      adaptiveProcessing();
    }
  }
  loop();
}

function drawWaveform() {
  const cv = document.getElementById('waveformCanvas'); if(!cv||!BAP.waveformAnalyser) return;
  const ctx = cv.getContext('2d'); const rect = cv.getBoundingClientRect(); cv.width = rect.width*2; cv.height = rect.height*2; ctx.scale(2,2);
  const w = rect.width, h = rect.height;
  ctx.fillStyle = '#0d0d14'; ctx.fillRect(0,0,w,h);
  const d = new Uint8Array(BAP.waveformAnalyser.fftSize); BAP.waveformAnalyser.getByteTimeDomainData(d);
  ctx.strokeStyle = 'rgba(0,229,255,0.4)'; ctx.lineWidth = 0.5; ctx.beginPath(); ctx.moveTo(0,h/2); ctx.lineTo(w,h/2); ctx.stroke();
  ctx.lineWidth = 1.5; ctx.strokeStyle = '#00e5ff'; ctx.shadowColor = 'rgba(0,229,255,0.3)'; ctx.shadowBlur = 4; ctx.beginPath();
  const sl = w/d.length; let x = 0;
  for(let i=0; i<d.length; i++) { const v=d[i]/128; const y=v*h/2; i===0?ctx.moveTo(x,y):ctx.lineTo(x,y); x+=sl; }
  ctx.stroke(); ctx.shadowBlur = 0;
}

function drawSpectrum() {
  const cv = document.getElementById('spectrumCanvas'); if(!cv||!BAP.masterAnalyser) return;
  const ctx = cv.getContext('2d'); const rect = cv.getBoundingClientRect(); cv.width = rect.width*2; cv.height = rect.height*2; ctx.scale(2,2);
  const w = rect.width, h = rect.height;
  ctx.fillStyle = '#0d0d14'; ctx.fillRect(0,0,w,h);
  const d = new Uint8Array(BAP.masterAnalyser.frequencyBinCount); BAP.masterAnalyser.getByteFrequencyData(d);
  const bars = 128, bw = (w/bars)-1;
  for(let i=0; i<bars; i++) {
    const idx = Math.floor(Math.pow(i/bars, 2)*d.length);
    const v = d[idx]/255;
    const bh = v*h;
    const r = v<0.5?0:v<0.75?Math.floor((v-0.5)*4*255):255;
    const g = v<0.5?Math.floor(v*2*230):230;
    const b = v<0.5?255:v<0.75?Math.floor(255-(v-0.5)*4*255):0;
    ctx.fillStyle = `rgb(${r},${g},${b})`; ctx.fillRect(i*(bw+1), h-bh, bw, bh);
    if(v>0.7) { ctx.fillStyle = `rgba(${r},${g},${b},0.3)`; ctx.fillRect(i*(bw+1), h-bh-2, bw, 2); }
  }
  ctx.fillStyle = 'rgba(255,255,255,0.3)'; ctx.font = '8px Consolas';
  ['100','500','1k','5k','10k','16k'].forEach((l,i) => ctx.fillText(l, (i/5)*w, h-4));
}

function adaptiveProcessing() {
  if(!BAP.midAnalyser || !BAP.midComp) return;
  const d = new Uint8Array(BAP.midAnalyser.frequencyBinCount); BAP.midAnalyser.getByteFrequencyData(d);
  let ws=0, te=0;
  for(let i=0; i<d.length; i++) { ws+=i*d[i]; te+=d[i]; }
  const cent = te>0 ? ws/te : 0;
  const avg = d.reduce((a,b)=>a+b,0)/d.length;

  BAP.highComp.threshold.value = Math.max(-30, BAP.highComp.threshold.value + (cent>50?-0.05:0.02));
  BAP.midComp.release.value = Math.max(0.05, Math.min(0.3, BAP.midComp.release.value + (avg>180?-0.002:0.001)));
}

// ============================================================
// EVENTS
// ============================================================
document.getElementById('btnStart').addEventListener('click', () => { if(!BAP.isRunning) startAudio(); });
document.getElementById('btnStop').addEventListener('click', () => { if(BAP.isRunning) stopAudio(); });
document.getElementById('inputDevice').addEventListener('change', changeInputDevice);
document.addEventListener('keydown', e => { if(e.code==='Space') { e.preventDefault(); BAP.isRunning ? stopAudio() : startAudio(); } });

window.addEventListener('load', init);
window.addEventListener('resize', drawGEQViz);
</script>
</body>
</html>
