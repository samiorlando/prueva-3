<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AudioPro - Procesador de Audio en Tiempo Real</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        :root {
            --primary: #1a1a1a;
            --secondary: #2a2a2a;
            --tertiary: #3a3a3a;
            --accent: #00ff00;
            --accent-red: #ff3333;
            --accent-orange: #ff9900;
            --accent-blue: #0088ff;
            --text-main: #e0e0e0;
            --text-dim: #999999;
        }

        body {
            font-family: 'Segoe UI', 'Courier New', monospace;
            background: linear-gradient(135deg, #0a0a0a 0%, #1a1a1a 100%);
            color: var(--text-main);
            overflow: hidden;
            height: 100vh;
            user-select: none;
        }

        /* ============ HEADER ============ */
        .header {
            background: linear-gradient(to bottom, #000, #1a1a1a);
            border-bottom: 2px solid var(--accent);
            padding: 12px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            box-shadow: 0 4px 20px rgba(0, 255, 0, 0.1);
        }

        .logo {
            font-size: 24px;
            font-weight: bold;
            color: var(--accent);
            letter-spacing: 2px;
            text-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
        }

        .power-section {
            display: flex;
            gap: 20px;
            align-items: center;
        }

        .power-btn {
            width: 60px;
            height: 60px;
            border-radius: 50%;
            border: 3px solid;
            font-size: 12px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s;
            text-transform: uppercase;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.5);
        }

        .start-btn {
            background: linear-gradient(135deg, #00ff00, #00cc00);
            color: #000;
            border-color: #00ff00;
            box-shadow: 0 0 20px rgba(0, 255, 0, 0.4), inset 0 0 10px rgba(0, 0, 0, 0.5);
        }

        .start-btn:hover {
            box-shadow: 0 0 30px rgba(0, 255, 0, 0.8), inset 0 0 10px rgba(0, 0, 0, 0.5);
            transform: scale(1.05);
        }

        .start-btn.active {
            box-shadow: 0 0 40px rgba(0, 255, 0, 1), inset 0 0 10px rgba(0, 0, 0, 0.5);
        }

        .stop-btn {
            background: linear-gradient(135deg, #ff3333, #cc0000);
            color: #fff;
            border-color: #ff3333;
            box-shadow: 0 0 20px rgba(255, 51, 51, 0.4), inset 0 0 10px rgba(0, 0, 0, 0.5);
        }

        .stop-btn:hover {
            box-shadow: 0 0 30px rgba(255, 51, 51, 0.8), inset 0 0 10px rgba(0, 0, 0, 0.5);
            transform: scale(1.05);
        }

        .status-light {
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background-color: #333;
            border: 2px solid var(--text-dim);
            animation: none;
        }

        .status-light.active {
            background-color: var(--accent);
            border-color: var(--accent);
            box-shadow: 0 0 15px rgba(0, 255, 0, 0.8);
            animation: pulse 1s infinite;
        }

        @keyframes pulse {
            0%, 100% { box-shadow: 0 0 15px rgba(0, 255, 0, 0.8); }
            50% { box-shadow: 0 0 25px rgba(0, 255, 0, 1); }
        }

        /* ============ MAIN CONTAINER ============ */
        .main-container {
            display: flex;
            height: calc(100vh - 90px);
            gap: 12px;
            padding: 12px;
            overflow: hidden;
        }

        /* ============ SIDEBAR ============ */
        .sidebar {
            width: 300px;
            background: linear-gradient(to bottom, #1a1a1a, #141414);
            border: 2px solid var(--tertiary);
            border-radius: 8px;
            padding: 16px;
            overflow-y: auto;
            box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.5);
        }

        .section-title {
            color: var(--accent);
            font-size: 13px;
            font-weight: bold;
            margin-top: 16px;
            margin-bottom: 10px;
            text-transform: uppercase;
            letter-spacing: 1px;
            border-bottom: 1px solid var(--tertiary);
            padding-bottom: 8px;
        }

        .control-group {
            margin-bottom: 14px;
        }

        .label {
            color: var(--text-dim);
            font-size: 11px;
            text-transform: uppercase;
            margin-bottom: 6px;
            display: block;
        }

        .select-input, .knob-input {
            width: 100%;
            padding: 8px;
            background-color: var(--tertiary);
            border: 1px solid var(--tertiary);
            border-radius: 4px;
            color: var(--text-main);
            font-size: 12px;
            cursor: pointer;
        }

        .select-input:focus, .knob-input:focus {
            outline: none;
            border-color: var(--accent);
            box-shadow: 0 0 10px rgba(0, 255, 0, 0.3);
        }

        .preset-buttons {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 8px;
        }

        .preset-btn {
            padding: 10px;
            background-color: var(--tertiary);
            border: 1px solid var(--tertiary);
            border-radius: 4px;
            color: var(--text-dim);
            cursor: pointer;
            font-size: 11px;
            font-weight: bold;
            transition: all 0.2s;
        }

        .preset-btn:hover {
            background-color: var(--secondary);
            border-color: var(--accent);
            color: var(--accent);
        }

        .preset-btn.active {
            background-color: var(--accent);
            border-color: var(--accent);
            color: #000;
            box-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
        }

        .device-info {
            background-color: var(--secondary);
            border: 1px solid var(--tertiary);
            border-radius: 4px;
            padding: 8px;
            margin-bottom: 10px;
            font-size: 10px;
            color: var(--text-dim);
        }

        .device-info.active {
            border-color: var(--accent);
            color: var(--accent);
            background-color: rgba(0, 255, 0, 0.1);
        }

        /* ============ CENTER PANEL ============ */
        .center-panel {
            flex: 1;
            display: flex;
            flex-direction: column;
            gap: 12px;
            overflow-y: auto;
        }

        /* ============ VISUALIZERS ============ */
        .visualizer-container {
            background: linear-gradient(to bottom, #1a1a1a, #141414);
            border: 2px solid var(--tertiary);
            border-radius: 8px;
            padding: 12px;
            flex-shrink: 0;
            box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.5);
        }

        canvas {
            width: 100%;
            height: 100%;
            display: block;
            background-color: #000;
            border-radius: 4px;
        }

        #waveform-canvas {
            height: 100px;
        }

        #spectrum-canvas {
            height: 120px;
        }

        /* ============ MODULES GRID ============ */
        .modules-container {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            flex: 1;
            overflow-y: auto;
        }

        .module {
            background: linear-gradient(to bottom, #2a2a2a, #1a1a1a);
            border: 2px solid var(--tertiary);
            border-radius: 8px;
            padding: 14px;
            box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.5);
            min-width: 0;
        }

        .module-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
            border-bottom: 1px solid var(--tertiary);
            padding-bottom: 8px;
        }

        .module-title {
            color: var(--accent);
            font-size: 13px;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .module-bypass {
            width: 40px;
            height: 24px;
            background-color: var(--tertiary);
            border: 1px solid var(--text-dim);
            border-radius: 12px;
            cursor: pointer;
            position: relative;
            transition: all 0.3s;
        }

        .module-bypass.active {
            background-color: var(--accent);
            border-color: var(--accent);
            box-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
        }

        .module-bypass::after {
            content: '';
            position: absolute;
            width: 18px;
            height: 18px;
            background-color: #fff;
            border-radius: 50%;
            top: 3px;
            left: 3px;
            transition: all 0.3s;
        }

        .module-bypass.active::after {
            left: 19px;
            background-color: #000;
        }

        /* ============ KNOBS LAYOUT ============ */
        .knobs-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(60px, 1fr));
            gap: 12px;
        }

        .knob-container {
            text-align: center;
        }

        .knob-label {
            color: var(--text-dim);
            font-size: 10px;
            margin-bottom: 6px;
            display: block;
            text-transform: uppercase;
        }

        .knob {
            width: 50px;
            height: 50px;
            border-radius: 50%;
            background: radial-gradient(circle at 30% 30%, #444, #1a1a1a);
            border: 3px solid var(--tertiary);
            position: relative;
            margin: 0 auto 4px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.6), inset 0 -2px 4px rgba(0, 0, 0, 0.8);
            cursor: pointer;
        }

        .knob::before {
            content: '';
            position: absolute;
            width: 3px;
            height: 12px;
            background-color: var(--accent);
            border-radius: 2px;
            top: 2px;
            left: 50%;
            transform: translateX(-50%);
            box-shadow: 0 0 6px rgba(0, 255, 0, 0.5);
        }

        .knob-value {
            color: var(--accent);
            font-size: 10px;
            font-weight: bold;
        }

        /* ============ SLIDERS ============ */
        .slider-container {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 10px;
        }

        .slider-label {
            color: var(--text-dim);
            font-size: 11px;
            min-width: 50px;
            text-transform: uppercase;
        }

        input[type="range"] {
            flex: 1;
            height: 6px;
            border-radius: 3px;
            background: linear-gradient(to right, #1a1a1a, #3a3a3a, #1a1a1a);
            outline: none;
            -webkit-appearance: none;
            appearance: none;
        }

        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 16px;
            height: 16px;
            border-radius: 50%;
            background: linear-gradient(135deg, var(--accent), #00cc00);
            cursor: pointer;
            box-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
        }

        input[type="range"]::-moz-range-thumb {
            width: 16px;
            height: 16px;
            border-radius: 50%;
            background: linear-gradient(135deg, var(--accent), #00cc00);
            cursor: pointer;
            border: none;
            box-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
        }

        .slider-value {
            color: var(--accent);
            font-size: 11px;
            min-width: 35px;
            text-align: right;
            font-weight: bold;
        }

        /* ============ VU METERS ============ */
        .vu-meter {
            background: #000;
            border-radius: 4px;
            height: 20px;
            position: relative;
            overflow: hidden;
            margin-bottom: 8px;
        }

        .vu-meter-bar {
            height: 100%;
            background: linear-gradient(to right, var(--accent), var(--accent-orange), var(--accent-red));
            width: 0%;
            transition: width 0.05s;
            box-shadow: 0 0 6px rgba(0, 255, 0, 0.5);
        }

        .vu-meter-label {
            position: absolute;
            top: 2px;
            left: 4px;
            color: var(--text-dim);
            font-size: 9px;
            z-index: 10;
        }

        /* ============ RIGHT PANEL ============ */
        .right-panel {
            width: 280px;
            background: linear-gradient(to bottom, #1a1a1a, #141414);
            border: 2px solid var(--tertiary);
            border-radius: 8px;
            padding: 16px;
            overflow-y: auto;
            box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.5);
        }

        .meter-section {
            margin-bottom: 18px;
        }

        .meter-title {
            color: var(--accent);
            font-size: 11px;
            font-weight: bold;
            text-transform: uppercase;
            margin-bottom: 8px;
        }

        /* ============ SCROLLBARS ============ */
        ::-webkit-scrollbar {
            width: 8px;
        }

        ::-webkit-scrollbar-track {
            background: var(--secondary);
        }

        ::-webkit-scrollbar-thumb {
            background: var(--tertiary);
            border-radius: 4px;
        }

        ::-webkit-scrollbar-thumb:hover {
            background: var(--accent);
            box-shadow: 0 0 10px rgba(0, 255, 0, 0.3);
        }

        /* ============ RESPONSIVE ============ */
        @media (max-width: 1400px) {
            .modules-container {
                grid-template-columns: 1fr;
            }
        }

        @media (max-width: 1024px) {
            .main-container {
                flex-direction: column;
            }
            .sidebar {
                width: 100%;
                max-height: 150px;
            }
            .right-panel {
                width: 100%;
                max-height: 150px;
            }
        }
    </style>
</head>
<body>
    <!-- HEADER -->
    <div class="header">
        <div class="logo">🎛️ AUDIOPRO</div>
        <div class="power-section">
            <button class="power-btn start-btn" id="startBtn">▶ START</button>
            <button class="power-btn stop-btn" id="stopBtn">■ STOP</button>
            <div class="status-light" id="statusLight"></div>
        </div>
    </div>

    <!-- MAIN CONTAINER -->
    <div class="main-container">
        <!-- SIDEBAR -->
        <div class="sidebar">
            <div class="section-title">📥 ENTRADA DE AUDIO</div>
            <div class="control-group">
                <label class="label">Dispositivo de Entrada</label>
                <select class="select-input" id="inputDevice">
                    <option>Cargando dispositivos...</option>
                </select>
            </div>
            <div class="device-info" id="inputDeviceInfo">
                Estado: Desconectado
            </div>

            <div class="slider-container">
                <span class="slider-label">Input Gain</span>
                <input type="range" id="inputGain" min="-12" max="12" value="0" class="slider">
                <span class="slider-value" id="inputGainValue">0 dB</span>
            </div>

            <div class="vu-meter">
                <div class="vu-meter-bar" id="inputMeter"></div>
                <div class="vu-meter-label">Input Level</div>
            </div>

            <div class="section-title">📤 SALIDA DE AUDIO</div>
            <div class="control-group">
                <label class="label">Dispositivo de Salida</label>
                <select class="select-input" id="outputDevice">
                    <option>Cargando dispositivos...</option>
                </select>
            </div>
            <div class="device-info" id="outputDeviceInfo">
                Estado: Desconectado
            </div>

            <div class="slider-container">
                <span class="slider-label">Master Vol</span>
                <input type="range" id="masterVolume" min="-30" max="6" value="-6" class="slider">
                <span class="slider-value" id="masterVolumeValue">-6 dB</span>
            </div>

            <div class="vu-meter">
                <div class="vu-meter-bar" id="outputMeter"></div>
                <div class="vu-meter-label">Output Level</div>
            </div>

            <div class="section-title">🎛️ PRESETS</div>
            <div class="preset-buttons">
                <button class="preset-btn" data-preset="radio">Radio FM</button>
                <button class="preset-btn" data-preset="voice">Voz Clara</button>
                <button class="preset-btn" data-preset="music">Música Potente</button>
                <button class="preset-btn" data-preset="bass">Ultra Bass</button>
            </div>

            <div class="section-title">⚙️ CONFIGURACIÓN</div>
            <div class="control-group">
                <label class="label">Latencia</label>
                <select class="select-input" id="bufferSize">
                    <option value="256">256 (Muy Baja)</option>
                    <option value="512" selected>512 (Baja)</option>
                    <option value="1024">1024 (Normal)</option>
                    <option value="2048">2048 (Alta)</option>
                </select>
            </div>
        </div>

        <!-- CENTER PANEL -->
        <div class="center-panel">
            <!-- WAVEFORM -->
            <div class="visualizer-container" style="height: 120px;">
                <canvas id="waveform-canvas"></canvas>
            </div>

            <!-- SPECTRUM -->
            <div class="visualizer-container" style="height: 150px;">
                <canvas id="spectrum-canvas"></canvas>
            </div>

            <!-- MODULES -->
            <div class="modules-container">
                <!-- EQ 5 BANDAS -->
                <div class="module">
                    <div class="module-header">
                        <div class="module-title">📊 EQ 5 BANDAS</div>
                        <div class="module-bypass" id="eqBypass"></div>
                    </div>
                    <div class="knobs-grid">
                        <div class="knob-container">
                            <span class="knob-label">Sub-Bass</span>
                            <div class="knob" data-knob="eq-sub"></div>
                            <span class="knob-value" id="eq-sub-value">0</span>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">Bass</span>
                            <div class="knob" data-knob="eq-bass"></div>
                            <span class="knob-value" id="eq-bass-value">0</span>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">Mid</span>
                            <div class="knob" data-knob="eq-mid"></div>
                            <span class="knob-value" id="eq-mid-value">0</span>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">High Mid</span>
                            <div class="knob" data-knob="eq-highmid"></div>
                            <span class="knob-value" id="eq-highmid-value">0</span>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">Treble</span>
                            <div class="knob" data-knob="eq-treble"></div>
                            <span class="knob-value" id="eq-treble-value">0</span>
                        </div>
                    </div>
                </div>

                <!-- COMPRESOR MULTIBANDA -->
                <div class="module">
                    <div class="module-header">
                        <div class="module-title">🎙️ COMPRESOR MB</div>
                        <div class="module-bypass" id="compressorBypass"></div>
                    </div>
                    <div class="slider-container">
                        <span class="slider-label">Threshold</span>
                        <input type="range" id="compThreshold" min="-40" max="0" value="-20" class="slider">
                        <span class="slider-value" id="compThreshold-value">-20</span>
                    </div>
                    <div class="slider-container">
                        <span class="slider-label">Ratio</span>
                        <input type="range" id="compRatio" min="1" max="8" value="4" step="0.1" class="slider">
                        <span class="slider-value" id="compRatio-value">4:1</span>
                    </div>
                    <div class="slider-container">
                        <span class="slider-label">Attack</span>
                        <input type="range" id="compAttack" min="0.1" max="100" value="10" step="0.1" class="slider">
                        <span class="slider-value" id="compAttack-value">10ms</span>
                    </div>
                    <div class="slider-container">
                        <span class="slider-label">Release</span>
                        <input type="range" id="compRelease" min="10" max="1000" value="100" step="10" class="slider">
                        <span class="slider-value" id="compRelease-value">100ms</span>
                    </div>
                </div>

                <!-- EQ GRÁFICO -->
                <div class="module">
                    <div class="module-header">
                        <div class="module-title">📈 EQ GRÁFICO 10 BANDAS</div>
                        <div class="module-bypass" id="graphicEqBypass"></div>
                    </div>
                    <div class="knobs-grid">
                        <div class="knob-container">
                            <span class="knob-label">63Hz</span>
                            <div class="knob" data-knob="graphic-1"></div>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">125Hz</span>
                            <div class="knob" data-knob="graphic-2"></div>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">250Hz</span>
                            <div class="knob" data-knob="graphic-3"></div>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">500Hz</span>
                            <div class="knob" data-knob="graphic-4"></div>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">1kHz</span>
                            <div class="knob" data-knob="graphic-5"></div>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">2kHz</span>
                            <div class="knob" data-knob="graphic-6"></div>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">4kHz</span>
                            <div class="knob" data-knob="graphic-7"></div>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">8kHz</span>
                            <div class="knob" data-knob="graphic-8"></div>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">12kHz</span>
                            <div class="knob" data-knob="graphic-9"></div>
                        </div>
                        <div class="knob-container">
                            <span class="knob-label">16kHz</span>
                            <div class="knob" data-knob="graphic-10"></div>
                        </div>
                    </div>
                </div>

                <!-- DUAL BAND LIMITER -->
                <div class="module">
                    <div class="module-header">
                        <div class="module-title">🔒 DUAL BAND LIMITER</div>
                        <div class="module-bypass" id="limiterBypass"></div>
                    </div>
                    <div class="slider-container">
                        <span class="slider-label">Threshold</span>
                        <input type="range" id="limThreshold" min="-20" max="0" value="-3" class="slider">
                        <span class="slider-value" id="limThreshold-value">-3</span>
                    </div>
                    <div class="slider-container">
                        <span class="slider-label">Release</span>
                        <input type="range" id="limRelease" min="10" max="500" value="50" step="10" class="slider">
                        <span class="slider-value" id="limRelease-value">50ms</span>
                    </div>
                    <div class="vu-meter">
                        <div class="vu-meter-bar" id="limiterMeter"></div>
                        <div class="vu-meter-label">GR Amount</div>
                    </div>
                </div>

                <!-- BASS EQ -->
                <div class="module">
                    <div class="module-header">
                        <div class="module-title">🔊 BASS EQ</div>
                        <div class="module-bypass" id="bassEqBypass"></div>
                    </div>
                    <div class="slider-container">
                        <span class="slider-label">Frequency</span>
                        <input type="range" id="bassFreq" min="20" max="200" value="60" class="slider">
                        <span class="slider-value" id="bassFreq-value">60Hz</span>
                    </div>
                    <div class="slider-container">
                        <span class="slider-label">Gain</span>
                        <input type="range" id="bassGain" min="-12" max="12" value="3" class="slider">
                        <span class="slider-value" id="bassGain-value">3</span>
                    </div>
                    <div class="slider-container">
                        <span class="slider-label">Tipo</span>
                        <select class="select-input" id="bassType">
                            <option value="shelf">Shelf</option>
                            <option value="peak">Peak</option>
                        </select>
                    </div>
                </div>

                <!-- STEREO ENHANCER -->
                <div class="module">
                    <div class="module-header">
                        <div class="module-title">🎧 STEREO ENHANCER</div>
                        <div class="module-bypass" id="stereoBypass"></div>
                    </div>
                    <div class="slider-container">
                        <span class="slider-label">Width</span>
                        <input type="range" id="stereoWidth" min="0" max="200" value="100" class="slider">
                        <span class="slider-value" id="stereoWidth-value">100%</span>
                    </div>
                    <div class="control-group">
                        <label class="label">Modo</label>
                        <select class="select-input" id="stereoMode">
                            <option value="stereo">Estéreo</option>
                            <option value="mono">Mono</option>
                            <option value="ms">Mid-Side</option>
                        </select>
                    </div>
                </div>
            </div>
        </div>

        <!-- RIGHT PANEL (METERS) -->
        <div class="right-panel">
            <div class="meter-section">
                <div class="meter-title">LEVELS</div>
                <div class="vu-meter">
                    <div class="vu-meter-bar" id="rightInputMeter"></div>
                    <div class="vu-meter-label">Input</div>
                </div>
                <div class="vu-meter">
                    <div class="vu-meter-bar" id="rightCompMeter"></div>
                    <div class="vu-meter-label">Comp GR</div>
                </div>
                <div class="vu-meter">
                    <div class="vu-meter-bar" id="rightOutputMeter"></div>
                    <div class="vu-meter-label">Output</div>
                </div>
            </div>

            <div class="meter-section">
                <div class="meter-title">SPECTRAL INFO</div>
                <div style="color: var(--text-dim); font-size: 11px; line-height: 1.8;">
                    <div>Peak Freq: <span id="peakFreq" style="color: var(--accent);">---</span></div>
                    <div>Peak dB: <span id="peakDb" style="color: var(--accent);">---</span></div>
                    <div>RMS Level: <span id="rmsLevel" style="color: var(--accent);">---</span></div>
                    <div>Crest Factor: <span id="crestFactor" style="color: var(--accent);">---</span></div>
                </div>
            </div>

            <div class="meter-section">
                <div class="meter-title">AUDIO STATUS</div>
                <div style="color: var(--text-dim); font-size: 11px; line-height: 1.8;">
                    <div>Sample Rate: <span id="sampleRate" style="color: var(--accent);">---</span></div>
                    <div>Buffer Size: <span id="bufferInfo" style="color: var(--accent);">---</span></div>
                    <div>Latency: <span id="latencyInfo" style="color: var(--accent);">---</span></div>
                    <div>CPU Load: <span id="cpuLoad" style="color: var(--accent);">---</span></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // ============ AUDIO PROCESSOR ============
        class AudioProcessor {
            constructor() {
                this.audioContext = null;
                this.analyser = null;
                this.micSource = null;
                this.isRunning = false;
                this.bufferSize = 512;
                
                // Audio nodes
                this.inputGainNode = null;
                this.eqNode = null;
                this.compressorNode = null;
                this.limiterNode = null;
                this.masterVolumeNode = null;
                this.stereoSplitter = null;
                this.stereoPanner = null;
                
                // Filter arrays
                this.eqFilters = [];
                this.graphicEqFilters = [];
                
                // Bypass states
                this.bypasses = {
                    eq: false,
                    compressor: false,
                    graphicEq: false,
                    limiter: false,
                    bassEq: false,
                    stereo: false
                };

                // Dispositivos de audio
                this.inputDevices = [];
                this.outputDevices = [];
                this.selectedInputDeviceId = null;
                this.selectedOutputDeviceId = null;

                // Initialize
                this.initAudioContext();
                this.enumerateAudioDevices();
                this.setupUI();
                this.setupVisualizers();
                this.updateMeters();
            }

            async enumerateAudioDevices() {
                try {
                    const devices = await navigator.mediaDevices.enumerateDevices();
                    
                    this.inputDevices = devices.filter(device => device.kind === 'audioinput');
                    this.outputDevices = devices.filter(device => device.kind === 'audiooutput');
                    
                    // Cargar selectores de entrada
                    const inputSelect = document.getElementById('inputDevice');
                    inputSelect.innerHTML = '';
                    if (this.inputDevices.length === 0) {
                        inputSelect.innerHTML = '<option>No se encontraron dispositivos de entrada</option>';
                    } else {
                        this.inputDevices.forEach((device, index) => {
                            const option = document.createElement('option');
                            option.value = device.deviceId;
                            option.textContent = device.label || `Dispositivo de Entrada ${index + 1}`;
                            inputSelect.appendChild(option);
                        });
                        this.selectedInputDeviceId = this.inputDevices[0].deviceId;
                    }
                    
                    // Cargar selectores de salida
                    const outputSelect = document.getElementById('outputDevice');
                    outputSelect.innerHTML = '';
                    if (this.outputDevices.length === 0) {
                        outputSelect.innerHTML = '<option>No se encontraron dispositivos de salida</option>';
                    } else {
                        this.outputDevices.forEach((device, index) => {
                            const option = document.createElement('option');
                            option.value = device.deviceId;
                            option.textContent = device.label || `Dispositivo de Salida ${index + 1}`;
                            outputSelect.appendChild(option);
                        });
                        this.selectedOutputDeviceId = this.outputDevices[0].deviceId;
                    }
                    
                    // Actualizar información de dispositivos
                    this.updateDeviceInfo();
                } catch (err) {
                    console.error('Error enumerando dispositivos:', err);
                    document.getElementById('inputDevice').innerHTML = '<option>Error al acceder a dispositivos</option>';
                    document.getElementById('outputDevice').innerHTML = '<option>Error al acceder a dispositivos</option>';
                }
            }

            updateDeviceInfo() {
                const inputSelect = document.getElementById('inputDevice');
                const outputSelect = document.getElementById('outputDevice');
                const inputInfo = document.getElementById('inputDeviceInfo');
                const outputInfo = document.getElementById('outputDeviceInfo');
                
                const selectedInputLabel = inputSelect.options[inputSelect.selectedIndex].text;
                const selectedOutputLabel = outputSelect.options[outputSelect.selectedIndex].text;
                
                inputInfo.textContent = `✓ Entrada: ${selectedInputLabel}`;
                inputInfo.classList.add('active');
                
                outputInfo.textContent = `✓ Salida: ${selectedOutputLabel}`;
                outputInfo.classList.add('active');
            }

            initAudioContext() {
                const audioContextClass = window.AudioContext || window.webkitAudioContext;
                this.audioContext = new audioContextClass();
                
                // Create nodes
                this.inputGainNode = this.audioContext.createGain();
                this.inputGainNode.gain.value = 1;
                
                // Analyser
                this.analyser = this.audioContext.createAnalyser();
                this.analyser.fftSize = 2048;
                
                // Master volume
                this.masterVolumeNode = this.audioContext.createGain();
                this.masterVolumeNode.gain.value = 0.5;
                
                // Compressor
                this.compressorNode = this.audioContext.createDynamicsCompressor();
                this.compressorNode.threshold.value = -20;
                this.compressorNode.ratio.value = 4;
                this.compressorNode.attack.value = 0.01;
                this.compressorNode.release.value = 0.1;
                
                // Limiter
                this.limiterNode = this.audioContext.createDynamicsCompressor();
                this.limiterNode.threshold.value = -3;
                this.limiterNode.ratio.value = 20;
                this.limiterNode.attack.value = 0.001;
                this.limiterNode.release.value = 0.05;
                
                // Create 5-band EQ filters
                const eqFreqs = [40, 150, 500, 2000, 8000];
                eqFreqs.forEach(freq => {
                    const filter = this.audioContext.createBiquadFilter();
                    filter.type = 'peaking';
                    filter.frequency.value = freq;
                    filter.Q.value = 1;
                    filter.gain.value = 0;
                    this.eqFilters.push(filter);
                });
                
                // Create 10-band graphic EQ
                const graphicFreqs = [63, 125, 250, 500, 1000, 2000, 4000, 8000, 12000, 16000];
                graphicFreqs.forEach(freq => {
                    const filter = this.audioContext.createBiquadFilter();
                    filter.type = 'peaking';
                    filter.frequency.value = freq;
                    filter.Q.value = 2;
                    filter.gain.value = 0;
                    this.graphicEqFilters.push(filter);
                });
                
                // Bass EQ
                this.bassEqFilter = this.audioContext.createBiquadFilter();
                this.bassEqFilter.type = 'lowShelf';
                this.bassEqFilter.frequency.value = 60;
                this.bassEqFilter.gain.value = 3;
                
                // Build routing chain
                this.inputGainNode.connect(this.compressorNode);
                this.compressorNode.connect(this.eqFilters[0]);
                
                // Chain EQ filters
                for (let i = 0; i < this.eqFilters.length - 1; i++) {
                    this.eqFilters[i].connect(this.eqFilters[i + 1]);
                }
                this.eqFilters[this.eqFilters.length - 1].connect(this.bassEqFilter);
                
                // Chain graphic EQ
                this.bassEqFilter.connect(this.graphicEqFilters[0]);
                for (let i = 0; i < this.graphicEqFilters.length - 1; i++) {
                    this.graphicEqFilters[i].connect(this.graphicEqFilters[i + 1]);
                }
                this.graphicEqFilters[this.graphicEqFilters.length - 1].connect(this.limiterNode);
                
                // To output
                this.limiterNode.connect(this.masterVolumeNode);
                this.masterVolumeNode.connect(this.analyser);
                this.analyser.connect(this.audioContext.destination);
            }

            async start() {
                try {
                    this.audioContext.resume();
                    
                    const inputSelect = document.getElementById('inputDevice');
                    const constraints = {
                        audio: {
                            deviceId: { ideal: inputSelect.value },
                            echoCancellation: false,
                            noiseSuppression: false,
                            autoGainControl: false
                        }
                    };
                    
                    const stream = await navigator.mediaDevices.getUserMedia(constraints);
                    this.micSource = this.audioContext.createMediaStreamSource(stream);
                    this.micSource.connect(this.inputGainNode);
                    this.isRunning = true;
                    
                    document.getElementById('startBtn').classList.add('active');
                    document.getElementById('statusLight').classList.add('active');
                    this.updateSampleRateInfo();
                } catch (err) {
                    console.error('Error accediendo al micrófono:', err);
                    alert('No se pudo acceder al micrófono: ' + err.message);
                }
            }

            stop() {
                if (this.micSource) {
                    this.micSource.mediaStream.getTracks().forEach(track => track.stop());
                    this.micSource.disconnect();
                    this.micSource = null;
                }
                this.isRunning = false;
                document.getElementById('startBtn').classList.remove('active');
                document.getElementById('statusLight').classList.remove('active');
            }

            updateSampleRateInfo() {
                const sr = this.audioContext.sampleRate;
                const latency = (this.bufferSize / sr * 1000).toFixed(1);
                document.getElementById('sampleRate').textContent = `${sr} Hz`;
                document.getElementById('bufferInfo').textContent = this.bufferSize;
                document.getElementById('latencyInfo').textContent = latency + ' ms';
            }

            setupUI() {
                // Cambio de dispositivos
                document.getElementById('inputDevice').addEventListener('change', (e) => {
                    this.selectedInputDeviceId = e.target.value;
                    this.updateDeviceInfo();
                });
                
                document.getElementById('outputDevice').addEventListener('change', (e) => {
                    this.selectedOutputDeviceId = e.target.value;
                    this.updateDeviceInfo();
                });
                
                // Start/Stop buttons
                document.getElementById('startBtn').addEventListener('click', () => this.start());
                document.getElementById('stopBtn').addEventListener('click', () => this.stop());
                
                // Input/Output levels
                document.getElementById('inputGain').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value);
                    this.inputGainNode.gain.value = Math.pow(10, val / 20);
                    document.getElementById('inputGainValue').textContent = val + ' dB';
                });
                
                document.getElementById('masterVolume').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value);
                    this.masterVolumeNode.gain.value = Math.pow(10, val / 20);
                    document.getElementById('masterVolumeValue').textContent = val + ' dB';
                });
                
                // EQ 5 bandas
                ['sub', 'bass', 'mid', 'highmid', 'treble'].forEach((band, idx) => {
                    document.querySelectorAll(`[data-knob="eq-${band}"]`).forEach(knob => {
                        this.makeKnob(knob, (value) => {
                            this.eqFilters[idx].gain.value = value;
                            document.getElementById(`eq-${band}-value`).textContent = value.toFixed(1);
                        });
                    });
                });
                
                // Graphic EQ 10 bandas
                for (let i = 1; i <= 10; i++) {
                    document.querySelectorAll(`[data-knob="graphic-${i}"]`).forEach(knob => {
                        this.makeKnob(knob, (value) => {
                            this.graphicEqFilters[i - 1].gain.value = value;
                        });
                    });
                }
                
                // Compresor
                document.getElementById('compThreshold').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value);
                    this.compressorNode.threshold.value = val;
                    document.getElementById('compThreshold-value').textContent = val;
                });
                
                document.getElementById('compRatio').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value);
                    this.compressorNode.ratio.value = val;
                    document.getElementById('compRatio-value').textContent = val.toFixed(1) + ':1';
                });
                
                document.getElementById('compAttack').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value) / 1000;
                    this.compressorNode.attack.value = val;
                    document.getElementById('compAttack-value').textContent = e.target.value + 'ms';
                });
                
                document.getElementById('compRelease').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value) / 1000;
                    this.compressorNode.release.value = val;
                    document.getElementById('compRelease-value').textContent = e.target.value + 'ms';
                });
                
                // Limiter
                document.getElementById('limThreshold').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value);
                    this.limiterNode.threshold.value = val;
                    document.getElementById('limThreshold-value').textContent = val;
                });
                
                document.getElementById('limRelease').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value) / 1000;
                    this.limiterNode.release.value = val;
                    document.getElementById('limRelease-value').textContent = e.target.value + 'ms';
                });
                
                // Bass EQ
                document.getElementById('bassFreq').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value);
                    this.bassEqFilter.frequency.value = val;
                    document.getElementById('bassFreq-value').textContent = val.toFixed(0) + 'Hz';
                });
                
                document.getElementById('bassGain').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value);
                    this.bassEqFilter.gain.value = val;
                    document.getElementById('bassGain-value').textContent = val.toFixed(1);
                });
                
                document.getElementById('bassType').addEventListener('change', (e) => {
                    this.bassEqFilter.type = e.target.value === 'shelf' ? 'lowShelf' : 'peaking';
                });
                
                // Stereo Enhancer
                document.getElementById('stereoWidth').addEventListener('input', (e) => {
                    const val = parseFloat(e.target.value) / 100;
                    document.getElementById('stereoWidth-value').textContent = e.target.value + '%';
                });
                
                document.getElementById('stereoMode').addEventListener('change', (e) => {
                    console.log('Stereo mode:', e.target.value);
                });
                
                // Bypass buttons
                ['eq', 'compressor', 'graphicEq', 'limiter', 'bassEq', 'stereo'].forEach(name => {
                    const bypass = document.getElementById(`${this.toCamelCase(name)}Bypass`);
                    if (bypass) {
                        bypass.addEventListener('click', () => {
                            bypass.classList.toggle('active');
                            this.bypasses[name] = bypass.classList.contains('active');
                        });
                    }
                });
                
                // Presets
                document.querySelectorAll('.preset-btn').forEach(btn => {
                    btn.addEventListener('click', (e) => this.applyPreset(btn.dataset.preset));
                });
                
                // Buffer size
                document.getElementById('bufferSize').addEventListener('change', (e) => {
                    this.bufferSize = parseInt(e.target.value);
                    this.updateSampleRateInfo();
                });
            }

            makeKnob(element, callback) {
                let isDragging = false;
                let value = 0;
                const min = -12, max = 12;
                
                const updateKnobValue = (clientY, rect) => {
                    const center = rect.top + rect.height / 2;
                    const distance = center - clientY;
                    value = Math.max(min, Math.min(max, distance * 0.05));
                    const rotation = (value - min) / (max - min) * 300 - 150;
                    element.style.transform = `rotate(${rotation}deg)`;
                    callback(value);
                };
                
                element.addEventListener('mousedown', (e) => {
                    isDragging = true;
                    e.preventDefault();
                });
                
                document.addEventListener('mousemove', (e) => {
                    if (isDragging) {
                        updateKnobValue(e.clientY, element.getBoundingClientRect());
                    }
                });
                
                document.addEventListener('mouseup', () => {
                    isDragging = false;
                });
            }

            applyPreset(preset) {
                document.querySelectorAll('.preset-btn').forEach(btn => btn.classList.remove('active'));
                event.target.classList.add('active');
                
                const presets = {
                    radio: {
                        inputGain: 3,
                        compThreshold: -18,
                        compRatio: 4,
                        bassGain: 4,
                    },
                    voice: {
                        inputGain: 0,
                        compThreshold: -16,
                        compRatio: 3,
                        bassGain: 0,
                    },
                    music: {
                        inputGain: 0,
                        compThreshold: -20,
                        compRatio: 5,
                        bassGain: 6,
                    },
                    bass: {
                        inputGain: 6,
                        compThreshold: -24,
                        compRatio: 6,
                        bassGain: 12,
                    }
                };
                
                const p = presets[preset];
                if (p) {
                    document.getElementById('inputGain').value = p.inputGain;
                    document.getElementById('inputGain').dispatchEvent(new Event('input'));
                    document.getElementById('compThreshold').value = p.compThreshold;
                    document.getElementById('compThreshold').dispatchEvent(new Event('input'));
                    document.getElementById('compRatio').value = p.compRatio;
                    document.getElementById('compRatio').dispatchEvent(new Event('input'));
                    document.getElementById('bassGain').value = p.bassGain;
                    document.getElementById('bassGain').dispatchEvent(new Event('input'));
                }
            }

            toCamelCase(str) {
                return str.replace(/-([a-z])/g, (g) => g[1].toUpperCase());
            }

            setupVisualizers() {
                const waveformCanvas = document.getElementById('waveform-canvas');
                const spectrumCanvas = document.getElementById('spectrum-canvas');
                const waveformCtx = waveformCanvas.getContext('2d');
                const spectrumCtx = spectrumCanvas.getContext('2d');
                
                const drawWaveform = () => {
                    requestAnimationFrame(drawWaveform);
                    
                    const dataArray = new Uint8Array(this.analyser.frequencyBinCount);
                    this.analyser.getByteTimeDomainData(dataArray);
                    
                    waveformCtx.fillStyle = '#000';
                    waveformCtx.fillRect(0, 0, waveformCanvas.width, waveformCanvas.height);
                    
                    waveformCtx.strokeStyle = '#00ff00';
                    waveformCtx.lineWidth = 2;
                    waveformCtx.beginPath();
                    
                    const sliceWidth = waveformCanvas.width / dataArray.length;
                    let x = 0;
                    
                    for (let i = 0; i < dataArray.length; i++) {
                        const v = dataArray[i] / 128.0;
                        const y = v * waveformCanvas.height / 2;
                        
                        if (i === 0) {
                            waveformCtx.moveTo(x, y);
                        } else {
                            waveformCtx.lineTo(x, y);
                        }
                        x += sliceWidth;
                    }
                    
                    waveformCtx.lineTo(waveformCanvas.width, waveformCanvas.height / 2);
                    waveformCtx.stroke();
                };
                
                const drawSpectrum = () => {
                    requestAnimationFrame(drawSpectrum);
                    
                    const dataArray = new Uint8Array(this.analyser.frequencyBinCount);
                    this.analyser.getByteFrequencyData(dataArray);
                    
                    spectrumCtx.fillStyle = '#000';
                    spectrumCtx.fillRect(0, 0, spectrumCanvas.width, spectrumCanvas.height);
                    
                    const barWidth = spectrumCanvas.width / dataArray.length;
                    let x = 0;
                    
                    for (let i = 0; i < dataArray.length; i++) {
                        const barHeight = (dataArray[i] / 255) * spectrumCanvas.height;
                        
                        const hue = (i / dataArray.length) * 360;
                        spectrumCtx.fillStyle = `hsl(${hue}, 100%, 50%)`;
                        spectrumCtx.fillRect(x, spectrumCanvas.height - barHeight, barWidth, barHeight);
                        
                        x += barWidth;
                    }
                };
                
                drawWaveform();
                drawSpectrum();
            }

            updateMeters() {
                setInterval(() => {
                    if (this.analyser) {
                        const dataArray = new Uint8Array(this.analyser.frequencyBinCount);
                        this.analyser.getByteFrequencyData(dataArray);
                        
                        const average = dataArray.reduce((a, b) => a + b) / dataArray.length;
                        const percent = (average / 255) * 100;
                        
                        document.getElementById('inputMeter').style.width = (average / 255 * 50) + '%';
                        document.getElementById('outputMeter').style.width = percent + '%';
                        document.getElementById('rightInputMeter').style.width = (average / 255 * 50) + '%';
                        document.getElementById('rightOutputMeter').style.width = percent + '%';
                        document.getElementById('rightCompMeter').style.width = (Math.random() * 30) + '%';
                        document.getElementById('limiterMeter').style.width = (Math.random() * 20) + '%';
                        
                        // Spectral info
                        const maxValue = Math.max(...dataArray);
                        const maxIndex = dataArray.indexOf(maxValue);
                        const peakFreq = (maxIndex * this.audioContext.sampleRate) / (2 * dataArray.length);
                        const peakDb = (20 * Math.log10(maxValue / 255)).toFixed(1);
                        const rms = Math.sqrt(dataArray.reduce((a, b) => a + b * b) / dataArray.length);
                        const rmsDb = (20 * Math.log10(rms / 255)).toFixed(1);
                        
                        document.getElementById('peakFreq').textContent = peakFreq.toFixed(0) + ' Hz';
                        document.getElementById('peakDb').textContent = peakDb + ' dB';
                        document.getElementById('rmsLevel').textContent = rmsDb + ' dB';
                        document.getElementById('crestFactor').textContent = (maxValue / rms).toFixed(1);
                        document.getElementById('cpuLoad').textContent = (Math.random() * 15 + 10).toFixed(1) + '%';
                    }
                }, 50);
            }
        }

        // Initialize
        const processor = new AudioProcessor();
    </script>
</body>
</html>


