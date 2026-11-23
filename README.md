<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
<meta name="theme-color" content="#0b1120" />
<title>TVSam Click - IPTV Player</title>

<!-- Fuentes -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet" />

<!-- Tailwind CSS (Play CDN) -->
<script src="https://cdn.tailwindcss.com"></script>
<script>
  tailwind.config = {
    theme: {
      extend: {
        fontFamily: { sans: ['Inter', 'ui-sans-serif', 'system-ui', 'sans-serif'] },
        colors: { ink: '#0b1120', panel: '#111827' }
      }
    }
  };
</script>

<!-- hls.js: HLS en Chrome/Firefox/Edge. Safari usa reproducción nativa (ver JS). -->
<script src="https://cdn.jsdelivr.net/npm/hls.js@1.5.20/dist/hls.min.js"></script>

<style>
  html, body { min-height: 100%; }
  body {
    background:
      radial-gradient(1100px 600px at 12% -10%, rgba(37, 99, 235, .18), transparent 60%),
      radial-gradient(900px 500px at 100% 0%, rgba(239, 68, 68, .10), transparent 55%),
      #0b1120;
  }
  ::-webkit-scrollbar { width: 8px; height: 8px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: #273449; border-radius: 99px; }
  ::-webkit-scrollbar-thumb:hover { background: #3a4c66; }
  .no-scrollbar::-webkit-scrollbar { display: none; }
  .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
  .glass { backdrop-filter: blur(16px); -webkit-backdrop-filter: blur(16px); }
  .glow { box-shadow: 0 10px 35px -10px rgba(59, 130, 246, .55); }
  @keyframes softpulse { 0%, 100% { opacity: 1 } 50% { opacity: .35 } }
  .pulse-dot { animation: softpulse 1.4s ease-in-out infinite; }
  /* Range de volumen */
  input[type="range"] { accent-color: #60a5fa; cursor: pointer; }

  /* Pantalla completa nativa */
  #player-shell:fullscreen,
  #player-shell:-webkit-full-screen,
  #player-shell:-moz-full-screen {
    width: 100vw !important;
    height: 100vh !important;
    max-width: 100vw !important;
    max-height: 100vh !important;
    aspect-ratio: auto !important;
    border: 0 !important;
    border-radius: 0 !important;
    background: #000 !important;
  }
  #player-shell:fullscreen video,
  #player-shell:-webkit-full-screen video,
  #player-shell:-moz-full-screen video {
    width: 100% !important;
    height: 100% !important;
    max-width: 100% !important;
    max-height: 100% !important;
    object-fit: contain !important;
  }

  /* Pantalla completa de respaldo (cuando el navegador bloquea la API nativa o en iframes) */
  .fake-fullscreen {
    position: fixed !important;
    inset: 0 !important;
    z-index: 9999 !important;
    width: 100vw !important;
    height: 100vh !important;
    max-height: none !important;
    aspect-ratio: auto !important;
    border: 0 !important;
    border-radius: 0 !important;
    background: #000 !important;
  }
  .fake-fullscreen video {
    width: 100% !important;
    height: 100% !important;
    object-fit: contain !important;
  }
  body.fake-fs { overflow: hidden !important; }

  /* Ocultar controles y cursor en pantalla completa tras inactividad */
  #player-shell.hide-controls {
    cursor: none;
  }
  #player-shell.hide-controls .player-ui {
    opacity: 0 !important;
    pointer-events: none !important;
  }

  /* ---------- header TVSam Click ---------- */
  :root {
    --accent: #38bdf8;
    --accent2: #0ea5e9;
    --danger: #ef4444;
    --line: #26334a;
    --panel: #131a26;
    --panel2: #1a2333;
    --muted: #8b9ab4;
  }
  header.tvsam-header {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 10px 18px;
    background: rgba(19, 26, 38, 0.95);
    border-bottom: 1px solid var(--line);
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
    flex-wrap: nowrap;
    scrollbar-width: thin;
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
  }
  header.tvsam-header::-webkit-scrollbar { height: 4px; }
  header.tvsam-header::-webkit-scrollbar-thumb { background: var(--line); border-radius: 4px; }
  header.tvsam-header h1 {
    font-size: 17px;
    margin: 0;
    white-space: nowrap;
    flex-shrink: 0;
    font-weight: 800;
    color: #fff;
  }
  header.tvsam-header h1 span { color: var(--accent); }
  .stats {
    color: var(--muted);
    font-size: 12.5px;
    white-space: nowrap;
    flex-shrink: 0;
  }
  .stats b { color: #f1f5f9; }
  .spacer { flex: 1; min-width: 10px; }
  .header-btns {
    display: flex;
    align-items: center;
    gap: 8px;
    flex-shrink: 0;
  }
  .btn {
    background: var(--panel2);
    color: #e2e8f0;
    border: 1px solid var(--line);
    border-radius: 8px;
    padding: 7px 12px;
    font-size: 12px;
    font-weight: 600;
    white-space: nowrap;
    display: inline-flex;
    align-items: center;
    gap: 6px;
    transition: .15s;
    touch-action: manipulation;
    min-height: 35px;
    cursor: pointer;
  }
  .btn:hover { border-color: var(--accent); color: var(--accent); }
  .btn.primary { background: var(--accent2); border-color: var(--accent2); color: #fff; font-weight: 700; }
  .btn.primary:hover { background: #0284c7; color: #fff; }
  .btn.danger { border-color: rgba(239, 68, 68, 0.4); color: #fca5a5; }
  .btn.danger:hover { border-color: var(--danger); background: rgba(239, 68, 68, 0.15); color: #fff; }
  .btn:disabled { opacity: .45; cursor: not-allowed; }
  @media (max-width: 768px) {
    .stats { display: none; }
  }
</style>
</head>

<body class="font-sans text-slate-200 antialiased selection:bg-blue-500/30">

<div class="relative flex min-h-screen flex-col">

  <!-- ===================== HEADER ===================== -->
  <header class="tvsam-header sticky top-0 z-30">
    <h1>📺 TVSam <span>Click</span></h1>
    <div class="stats" id="stats"><b>160</b> canales · <b>67</b> eliminados · <b>0</b> favoritos</div>
    <div class="spacer"></div>
    <div class="header-btns">
      <button class="btn" id="btn-scan">⚡ Verificar canales</button>
      <button class="btn" id="btn-load">📂 Cargar M3U</button>
      <input type="file" id="file-input" accept=".m3u,.m3u8,.txt" hidden="">
      <button class="btn" id="btn-export">⬇ Descargar M3U limpio</button>
      <button class="btn" id="btn-trash">🗑 Eliminados (<span id="trash-count">67</span>)</button>
      <button class="btn danger" id="btn-reset">♻ Restaurar todo</button>
      <button class="btn xl:hidden" id="btn-menu" title="Canales">📋 Canales</button>
    </div>
  </header>

  <!-- ===================== CONTENIDO ===================== -->
  <main class="mx-auto flex w-full max-w-[1800px] flex-1 flex-col gap-5 p-4 sm:p-6 xl:flex-row">

    <!-- ---------- Columna del reproductor ---------- -->
    <section class="min-w-0 flex-1 space-y-4">

      <!-- Reproductor -->
      <div id="player-shell" class="relative aspect-video w-full overflow-hidden rounded-2xl border border-white/10 bg-black shadow-2xl shadow-black/60">
        <video id="video" class="h-full w-full bg-black object-contain" playsinline webkit-playsinline preload="none"></video>

        <!-- Degradado para legibilidad de controles -->
        <div class="player-ui pointer-events-none absolute inset-0 bg-gradient-to-t from-black/85 via-transparent to-black/45 transition-opacity duration-300"></div>

        <!-- Insignia EN VIVO + marca de canal -->
        <div class="player-ui pointer-events-none absolute inset-x-3 top-3 flex items-start justify-between gap-2 transition-opacity duration-300 sm:inset-x-4 sm:top-4">
          <div id="live-badge" class="flex items-center gap-2 rounded-full bg-red-600/90 px-3 py-1.5 text-[10px] font-extrabold uppercase tracking-widest text-white opacity-0 shadow-lg shadow-red-900/40 transition-opacity duration-300">
            <span class="h-2 w-2 rounded-full bg-white pulse-dot"></span> En vivo
          </div>
          <div id="watermark" class="hidden max-w-[45%] truncate rounded-lg bg-black/45 px-2.5 py-1 text-[11px] font-bold text-white/85 glass"></div>
        </div>

        <!-- ESTADO: cargando -->
        <div id="ov-loading" class="absolute inset-0 grid place-items-center bg-black/55 px-6 text-center">
          <div>
            <div class="mx-auto h-11 w-11 animate-spin rounded-full border-[3px] border-white/15 border-t-blue-400"></div>
            <p id="ov-loading-text" class="mt-4 text-sm font-semibold text-white">Conectando con el canal…</p>
            <p class="mt-1 text-[11px] text-slate-400">Si tarda, es posible que el stream esté congestionado</p>
          </div>
        </div>

        <!-- ESTADO: autoplay bloqueado -->
        <div id="ov-play" class="absolute inset-0 hidden place-items-center bg-black/55">
          <button id="btn-big-play" class="group grid h-16 w-16 place-items-center rounded-full bg-gradient-to-br from-blue-500 to-red-500 text-white shadow-2xl shadow-blue-900/50 transition hover:scale-105" title="Reproducir con sonido">
            <svg viewBox="0 0 24 24" class="ml-1 h-7 w-7" fill="currentColor"><path d="M8 5.5v13l11-6.5z"/></svg>
          </button>
          <p class="pointer-events-none absolute bottom-24 text-xs font-medium text-white/80">Toca para reproducir con sonido</p>
        </div>

        <!-- ESTADO: error / fuera de línea -->
        <div id="ov-offline" class="absolute inset-0 hidden place-items-center bg-[#070b14]/85 px-6 text-center glass">
          <div class="max-w-md">
            <div class="mx-auto grid h-12 w-12 place-items-center rounded-2xl bg-red-500/15 text-red-400 ring-1 ring-red-500/30">
              <svg viewBox="0 0 24 24" class="h-6 w-6" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 3l18 18"/><path d="M10.5 10.5A5 5 0 0 0 8 14.9M2 8.8A15 15 0 0 1 6 6m2.5-2.3A12 12 0 0 1 22 8.8M15.5 5.6A9 9 0 0 1 20 8"/><circle cx="12" cy="19" r="1" fill="currentColor" stroke="none"/></svg>
            </div>
            <h3 class="mt-4 text-base font-bold text-white">Señal no disponible</h3>
            <p id="offline-detail" class="mt-2 text-xs leading-relaxed text-slate-400">El canal puede estar temporalmente fuera de línea o bloquear la reproducción desde tu red.</p>
            <button id="btn-retry" class="mt-5 inline-flex items-center gap-2 rounded-xl bg-blue-600 px-4 py-2.5 text-xs font-bold text-white shadow-lg shadow-blue-900/50 transition hover:bg-blue-500">
              <svg viewBox="0 0 24 24" class="h-4 w-4" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 11a8 8 0 1 0-2.3 5.7"/><path d="M20 5v6h-6"/></svg>
              Reintentar canal
            </button>
          </div>
        </div>

        <!-- Controles -->
        <div class="player-ui pointer-events-auto absolute inset-x-0 bottom-0 z-20 px-2 pb-2 transition-opacity duration-300 sm:px-4 sm:pb-4">
          <div class="flex items-center gap-1 rounded-xl border border-white/10 bg-black/60 px-2 py-2 glass sm:gap-2 sm:px-3">
            <button id="btn-play" type="button" title="Reproducir / Pausar (Espacio)" class="cursor-pointer grid h-8 w-8 place-items-center rounded-lg text-white transition hover:bg-white/15">
              <svg id="icon-play" viewBox="0 0 24 24" class="h-5 w-5" fill="currentColor"><path d="M8 5.5v13l11-6.5z"/></svg>
              <svg id="icon-pause" viewBox="0 0 24 24" class="hidden h-5 w-5" fill="currentColor"><path d="M7 5h3.5v14H7zm6.5 0H17v14h-3.5z"/></svg>
            </button>
            <button id="btn-mute" type="button" title="Silenciar (M)" class="cursor-pointer grid h-8 w-8 place-items-center rounded-lg text-white transition hover:bg-white/15">
              <svg id="icon-vol" viewBox="0 0 24 24" class="h-5 w-5" fill="currentColor"><path d="M4 9v6h4l5 4V5L8 9H4zm12.5 3a4.5 4.5 0 0 0-2.5-4v8a4.5 4.5 0 0 0 2.5-4z"/></svg>
              <svg id="icon-muted" viewBox="0 0 24 24" class="hidden h-5 w-5" fill="currentColor"><path d="M4 9v6h4l5 4V5L8 9H4zm12.6 3 2.2-2.2-1.4-1.4-2.2 2.2-2.2-2.2-1.4 1.4 2.2 2.2-2.2 2.2 1.4 1.4 2.2-2.2 2.2 2.2 1.4-1.4z"/></svg>
            </button>
            <input id="volume" type="range" min="0" max="1" step="0.05" value="0.8" title="Volumen" class="w-14 sm:w-24" />
            <span class="ml-1 hidden items-center gap-1.5 text-[10px] font-extrabold tracking-wider text-red-400 sm:flex"><span class="h-1.5 w-1.5 rounded-full bg-red-500 pulse-dot"></span> LIVE</span>

            <div class="ml-auto flex items-center gap-1">
              <button id="btn-player-channels" type="button" title="Lista de canales en pantalla (C)" class="cursor-pointer flex items-center gap-1.5 h-8 px-2.5 rounded-lg text-white text-xs font-semibold transition bg-white/10 hover:bg-blue-600/80">
                <svg viewBox="0 0 24 24" class="h-4 w-4 text-blue-400" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="5" width="20" height="14" rx="2"></rect><path d="M8 21h8M12 19v2"></path></svg>
                <span class="hidden sm:inline">Canales</span>
              </button>
              <button id="btn-pip" type="button" title="Picture-in-Picture" class="cursor-pointer grid h-8 w-8 place-items-center rounded-lg text-white transition hover:bg-white/15">
                <svg viewBox="0 0 24 24" class="h-[18px] w-[18px]" fill="none" stroke="currentColor" stroke-width="2"><rect x="2.5" y="4.5" width="19" height="15" rx="2"/><rect x="12" y="11" width="8" height="6" rx="1" fill="currentColor" stroke="none"/></svg>
              </button>
              <button id="btn-fs" type="button" title="Pantalla completa (F)" class="cursor-pointer grid h-8 w-8 place-items-center rounded-lg text-white transition hover:bg-white/15">
                <svg id="icon-fs-enter" viewBox="0 0 24 24" class="pointer-events-none h-[18px] w-[18px]" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 9V4h5M20 9V4h-5M4 15v5h5M20 15v5h-5"/></svg>
                <svg id="icon-fs-exit" viewBox="0 0 24 24" class="pointer-events-none hidden h-[18px] w-[18px]" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 14h5v5M20 14h-5v5M4 10h5V5M20 10h-5V5"/></svg>
              </button>
            </div>
          </div>
        </div>

        <!-- Panel OSD Flotante de Canales sobre el Reproductor -->
        <div id="player-osd-channels" class="pointer-events-auto absolute inset-y-0 right-0 z-30 flex w-full max-w-[340px] flex-col bg-slate-950/95 backdrop-blur-md border-l border-white/10 p-3 shadow-2xl transition-transform duration-300 translate-x-full">
          <div class="flex items-center justify-between border-b border-white/10 pb-2.5">
            <h3 class="flex items-center gap-2 text-xs font-bold text-white">
              <svg viewBox="0 0 24 24" class="h-4 w-4 text-blue-400" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="5" width="20" height="14" rx="2"></rect><path d="M8 21h8M12 19v2"></path></svg>
              Canales <span id="osd-count" class="rounded-md bg-white/10 px-1.5 py-0.5 text-[10px] text-slate-300">0</span>
            </h3>
            <button id="btn-close-osd" type="button" class="cursor-pointer rounded-lg p-1 text-slate-400 hover:bg-white/10 hover:text-white transition" title="Cerrar (Esc / C)">
              <svg viewBox="0 0 24 24" class="h-4 w-4" fill="none" stroke="currentColor" stroke-width="2"><path d="M6 6l12 12M18 6 6 18"/></svg>
            </button>
          </div>

          <!-- Buscador en OSD -->
          <div class="relative my-2.5">
            <svg viewBox="0 0 24 24" class="pointer-events-none absolute left-2.5 top-1/2 h-3.5 w-3.5 -translate-y-1/2 text-slate-500" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"></circle><path d="m20 20-3.6-3.6"></path></svg>
            <input id="osd-search" type="search" placeholder="Buscar canal..." autocomplete="off" class="w-full rounded-lg border border-white/10 bg-black/40 py-1.5 pl-8 pr-2.5 text-xs text-white outline-none transition placeholder:text-slate-600 focus:border-blue-500 focus:ring-1 focus:ring-blue-500/20">
          </div>

          <!-- Lista de canales dentro del OSD -->
          <div id="osd-channel-list" class="flex-1 overflow-y-auto space-y-1 pr-0.5">
            <!-- Renderizado dinámico -->
          </div>
        </div>
      </div>

      <!-- Barra del canal actual -->
      <div class="flex flex-wrap items-center gap-3 rounded-2xl border border-white/10 bg-white/[0.04] p-3.5 sm:p-4">
        <div id="np-logo" class="relative grid h-12 w-[72px] shrink-0 place-items-center overflow-hidden rounded-xl border border-white/10 bg-slate-900 text-[11px] font-extrabold text-slate-500 sm:h-14 sm:w-20">
          <span id="np-logo-fallback">—</span>
        </div>
        <div class="min-w-0 flex-1">
          <h2 id="np-title" class="truncate text-base font-bold text-white sm:text-lg">Selecciona un canal</h2>
          <div class="mt-1 flex flex-wrap items-center gap-2 text-[11px] text-slate-400">
            <span id="np-country" class="rounded-md bg-white/5 px-2 py-0.5 font-semibold">—</span>
            <span id="np-category" class="rounded-md bg-blue-500/10 px-2 py-0.5 font-semibold text-blue-300">—</span>
            <span id="np-pos" class="hidden sm:inline">—</span>
          </div>
        </div>
        <div class="flex items-center gap-2">
          <button id="btn-fav" title="Añadir/quitar de favoritos" class="grid h-10 w-10 place-items-center rounded-xl border border-white/10 bg-white/5 text-slate-400 transition hover:border-amber-400/50 hover:text-amber-400">
            <svg viewBox="0 0 24 24" class="h-5 w-5" fill="currentColor"><path d="m12 3 2.8 5.7 6.2.9-4.5 4.4 1.1 6.2L12 17.3 6.4 20.2l1.1-6.2L3 9.6l6.2-.9L12 3z"/></svg>
          </button>
          <button id="btn-prev" title="Canal anterior (←)" class="grid h-10 w-10 place-items-center rounded-xl border border-white/10 bg-white/5 text-slate-300 transition hover:border-blue-400/50 hover:bg-blue-500/10 hover:text-white">
            <svg viewBox="0 0 24 24" class="h-5 w-5" fill="currentColor"><path d="M6 5h2.5v14H6zm12.5 0v14L9.5 12z"/></svg>
          </button>
          <button id="btn-next" title="Canal siguiente (→)" class="grid h-10 w-10 place-items-center rounded-xl border border-white/10 bg-white/5 text-slate-300 transition hover:border-blue-400/50 hover:bg-blue-500/10 hover:text-white">
            <svg viewBox="0 0 24 24" class="h-5 w-5" fill="currentColor"><path d="M15.5 5H18v14h-2.5zM5.5 5v14L14.5 12z"/></svg>
          </button>
        </div>
      </div>

      <p class="hidden px-1 text-[11px] text-slate-600 lg:block">
        Atajos: <span class="text-slate-400">Espacio</span> reproducir/pausar · <span class="text-slate-400">← →</span> cambiar canal · <span class="text-slate-400">F</span> pantalla completa · <span class="text-slate-400">M</span> silenciar · <span class="text-slate-400">/</span> buscar
      </p>
    </section>
  </main>
</div>

<!-- Overlay global de carga de listas -->
<div id="loading" class="fixed inset-0 z-[60] grid place-items-center bg-[#0b1120]/95 px-6 text-center glass">
  <div class="w-full max-w-sm">
    <div class="mx-auto grid h-14 w-14 place-items-center rounded-2xl bg-gradient-to-br from-red-600 via-rose-500 to-red-700 shadow-2xl shadow-red-900/50">
      <svg viewBox="0 0 24 24" class="h-7 w-7 text-white" fill="currentColor"><path d="M8 5.5v13l11-6.5z"/></svg>
    </div>
    <h2 class="mt-5 text-lg font-extrabold text-white">Preparando canales de Perú 🇵🇪</h2>
    <p class="mt-1.5 text-xs text-slate-400" id="loading-detail">Descargando lista oficial (m3u.cl/lista/PE.m3u)</p>
    <div class="mt-5 h-1.5 w-full overflow-hidden rounded-full bg-white/10">
      <div id="loading-bar" class="h-full w-0 rounded-full bg-gradient-to-r from-red-500 to-rose-400 transition-all duration-300"></div>
    </div>
    <p id="loading-progress" class="mt-2 text-[11px] font-medium text-slate-500">Conectando con la lista…</p>
  </div>
</div>

<!-- Modal: Añadir / Editar Canal -->
<div id="modal-channel" class="fixed inset-0 z-50 flex items-center justify-center bg-black/75 p-4 backdrop-blur-sm hidden" role="dialog" aria-modal="true" aria-labelledby="modal-title">
  <div class="relative w-full max-w-md rounded-2xl border border-white/10 bg-[#0d1424] p-6 shadow-2xl shadow-black/80">
    <div class="flex items-center justify-between pb-3 border-b border-white/10">
      <h3 id="modal-title" class="flex items-center gap-2 text-base font-bold text-white">
        <svg viewBox="0 0 24 24" class="h-5 w-5 text-red-500" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 5v14M5 12h14"/></svg>
        <span>Añadir nuevo canal</span>
      </h3>
      <button id="btn-modal-close" type="button" class="cursor-pointer rounded-lg p-1.5 text-slate-400 hover:bg-white/10 hover:text-white transition" title="Cerrar">
        <svg viewBox="0 0 24 24" class="h-5 w-5" fill="none" stroke="currentColor" stroke-width="2"><path d="M6 6l12 12M18 6 6 18"/></svg>
      </button>
    </div>

    <form id="form-channel" class="mt-4 space-y-4">
      <input type="hidden" id="form-original-url" value="" />
      
      <div>
        <label for="form-name" class="block text-xs font-semibold text-slate-300 mb-1.5">Nombre del canal <span class="text-red-400">*</span></label>
        <input id="form-name" type="text" required placeholder="Ej: Latina Televisión HD" class="w-full rounded-xl border border-white/10 bg-black/40 px-3.5 py-2 text-xs text-white placeholder:text-slate-600 outline-none focus:border-red-500/70 focus:ring-2 focus:ring-red-500/20" />
      </div>

      <div>
        <label for="form-url" class="block text-xs font-semibold text-slate-300 mb-1.5">URL de transmisión (.m3u8 o stream) <span class="text-red-400">*</span></label>
        <input id="form-url" type="url" required placeholder="https://servidor.com/live/canal.m3u8" class="w-full rounded-xl border border-white/10 bg-black/40 px-3.5 py-2 text-xs text-white placeholder:text-slate-600 outline-none focus:border-red-500/70 focus:ring-2 focus:ring-red-500/20" />
      </div>

      <div>
        <label for="form-logo" class="block text-xs font-semibold text-slate-300 mb-1.5">Logotipo URL (opcional)</label>
        <div class="flex items-center gap-2.5">
          <div id="form-logo-preview" class="grid h-10 w-14 shrink-0 place-items-center overflow-hidden rounded-lg border border-white/10 bg-slate-900 text-[10px] font-extrabold text-slate-600">
            <span>LOGO</span>
          </div>
          <input id="form-logo" type="url" placeholder="https://servidor.com/logo.png" class="flex-1 rounded-xl border border-white/10 bg-black/40 px-3.5 py-2 text-xs text-white placeholder:text-slate-600 outline-none focus:border-red-500/70 focus:ring-2 focus:ring-red-500/20" />
        </div>
      </div>

      <div>
        <label for="form-cat" class="block text-xs font-semibold text-slate-300 mb-1.5">Categoría</label>
        <input id="form-cat" list="category-suggestions" type="text" placeholder="General, Noticias, Deportes, Música..." class="w-full rounded-xl border border-white/10 bg-black/40 px-3.5 py-2 text-xs text-white placeholder:text-slate-600 outline-none focus:border-red-500/70 focus:ring-2 focus:ring-red-500/20" />
        <datalist id="category-suggestions"></datalist>
      </div>

      <div class="flex items-center justify-end gap-2 pt-3 border-t border-white/10">
        <button id="btn-modal-cancel" type="button" class="cursor-pointer rounded-xl border border-white/10 bg-white/5 px-4 py-2 text-xs font-bold text-slate-300 transition hover:bg-white/10 hover:text-white">Cancelar</button>
        <button id="btn-modal-save" type="submit" class="cursor-pointer rounded-xl bg-red-600 px-5 py-2 text-xs font-bold text-white shadow-lg shadow-red-900/40 transition hover:bg-red-500">Guardar canal</button>
      </div>
    </form>
  </div>
</div>

<!-- Modal: Confirmar Eliminación -->
<div id="modal-delete" class="fixed inset-0 z-50 flex items-center justify-center bg-black/75 p-4 backdrop-blur-sm hidden" role="dialog" aria-modal="true">
  <div class="relative w-full max-w-sm rounded-2xl border border-white/10 bg-[#0d1424] p-6 shadow-2xl shadow-black/80">
    <div class="mx-auto grid h-12 w-12 place-items-center rounded-2xl bg-red-500/15 text-red-400 border border-red-500/30">
      <svg viewBox="0 0 24 24" class="h-6 w-6" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 6h18M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2M10 11v6M14 11v6"/></svg>
    </div>
    <div class="mt-4 text-center">
      <h3 class="text-base font-bold text-white">¿Eliminar canal?</h3>
      <p class="mt-1.5 text-xs text-slate-400">Esta acción removerá el canal de tu lista personalizada.</p>
      <div id="delete-channel-name" class="mt-3 truncate rounded-xl border border-white/10 bg-black/40 px-3 py-2 text-xs font-semibold text-slate-200">Canal</div>
    </div>
    <div class="mt-5 flex items-center justify-center gap-2">
      <button id="btn-delete-cancel" type="button" class="cursor-pointer flex-1 rounded-xl border border-white/10 bg-white/5 py-2.5 text-xs font-bold text-slate-300 transition hover:bg-white/10 hover:text-white">Cancelar</button>
      <button id="btn-delete-confirm" type="button" class="cursor-pointer flex-1 rounded-xl bg-red-600 py-2.5 text-xs font-bold text-white shadow-lg shadow-red-900/40 transition hover:bg-red-500">Sí, eliminar</button>
    </div>
  </div>
</div>

<!-- Modal: Opciones y Gestión de la Lista -->
<div id="modal-manage" class="fixed inset-0 z-50 flex items-center justify-center bg-black/75 p-4 backdrop-blur-sm hidden" role="dialog" aria-modal="true">
  <div class="relative w-full max-w-sm rounded-2xl border border-white/10 bg-[#0d1424] p-6 shadow-2xl shadow-black/80">
    <div class="flex items-center justify-between pb-3 border-b border-white/10">
      <h3 class="flex items-center gap-2 text-sm font-bold text-white">
        <svg viewBox="0 0 24 24" class="h-4 w-4 text-red-500" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 20h9M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/></svg>
        <span>Gestión de Canales</span>
      </h3>
      <button id="btn-manage-close" type="button" class="cursor-pointer rounded-lg p-1.5 text-slate-400 hover:bg-white/10 hover:text-white transition">
        <svg viewBox="0 0 24 24" class="h-5 w-5" fill="none" stroke="currentColor" stroke-width="2"><path d="M6 6l12 12M18 6 6 18"/></svg>
      </button>
    </div>

    <div class="mt-4 space-y-2.5">
      <button id="btn-export-m3u" type="button" class="cursor-pointer flex w-full items-center gap-3 rounded-xl border border-white/10 bg-white/5 p-3 text-left transition hover:border-white/20 hover:bg-white/10">
        <div class="grid h-9 w-9 shrink-0 place-items-center rounded-lg bg-emerald-500/15 text-emerald-400 border border-emerald-500/30">
          <svg viewBox="0 0 24 24" class="h-4 w-4" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M7 10l5 5 5-5M12 15V3"/></svg>
        </div>
        <div>
          <p class="text-xs font-bold text-white">Descargar lista M3U</p>
          <p class="text-[11px] text-slate-400">Guarda una copia de todos tus canales actuales (.m3u)</p>
        </div>
      </button>

      <label class="cursor-pointer flex w-full items-center gap-3 rounded-xl border border-white/10 bg-white/5 p-3 text-left transition hover:border-white/20 hover:bg-white/10">
        <div class="grid h-9 w-9 shrink-0 place-items-center rounded-lg bg-blue-500/15 text-blue-400 border border-blue-500/30">
          <svg viewBox="0 0 24 24" class="h-4 w-4" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M17 8l-5-5-5 5M12 3v12"/></svg>
        </div>
        <div>
          <p class="text-xs font-bold text-white">Importar archivo M3U</p>
          <p class="text-[11px] text-slate-400">Carga canales desde un archivo .m3u de tu PC</p>
        </div>
        <input type="file" id="file-import-m3u" accept=".m3u,.m3u8,text/plain" class="hidden" />
      </label>

      <button id="btn-reset-list" type="button" class="cursor-pointer flex w-full items-center gap-3 rounded-xl border border-white/10 bg-white/5 p-3 text-left transition hover:border-red-500/30 hover:bg-red-500/10">
        <div class="grid h-9 w-9 shrink-0 place-items-center rounded-lg bg-amber-500/15 text-amber-400 border border-amber-500/30">
          <svg viewBox="0 0 24 24" class="h-4 w-4" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 12a9 9 0 0 1 9-9 9.75 9.75 0 0 1 6.74 2.74L21 8M21 3v5h-5M21 12a9 9 0 0 1-9 9 9.75 9.75 0 0 1-6.74-2.74L3 16M3 21v-5h5"/></svg>
        </div>
        <div>
          <p class="text-xs font-bold text-amber-300">Restaurar lista original</p>
          <p class="text-[11px] text-slate-400">Vuelve a la lista predeterminada oficial de Perú</p>
        </div>
      </button>
    </div>
  </div>
</div>

<!-- Modal: Canales Eliminados -->
<div id="modal-trash" class="fixed inset-0 z-50 flex items-center justify-center bg-black/75 p-4 backdrop-blur-sm hidden" role="dialog" aria-modal="true">
  <div class="relative flex w-full max-w-lg max-h-[85vh] flex-col rounded-2xl border border-white/10 bg-[#0d1424] shadow-2xl shadow-black/80">
    <div class="flex items-center justify-between border-b border-white/10 p-4">
      <div class="flex items-center gap-2">
        <div class="grid h-8 w-8 place-items-center rounded-lg bg-red-500/15 text-red-400 border border-red-500/30">
          <svg viewBox="0 0 24 24" class="h-4 w-4" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 6h18M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2M10 11v6M14 11v6"/></svg>
        </div>
        <div>
          <h3 class="text-sm font-bold text-white">Canales Eliminados</h3>
          <p class="text-[11px] text-slate-400">Puedes restaurar canales individualmente o todos juntos</p>
        </div>
      </div>
      <div class="flex items-center gap-2">
        <button id="btn-restore-all" type="button" class="btn text-[11px] font-bold">Restaurar todos</button>
        <button id="btn-trash-close" type="button" class="cursor-pointer rounded-lg p-1.5 text-slate-400 hover:bg-white/10 hover:text-white transition">
          <svg viewBox="0 0 24 24" class="h-5 w-5" fill="none" stroke="currentColor" stroke-width="2"><path d="M6 6l12 12M18 6 6 18"/></svg>
        </button>
      </div>
    </div>
    <div id="trash-body" class="flex-1 overflow-y-auto p-3 space-y-1.5 min-h-[160px]">
      <!-- Canales eliminados renderizados aquí -->
    </div>
  </div>
</div>

<!-- Toasts -->
<div id="toast-host" class="pointer-events-none fixed bottom-5 left-1/2 z-[70] flex w-[calc(100%-2rem)] max-w-sm -translate-x-1/2 flex-col gap-2"></div>

<!-- Lista M3U local incrustada para compatibilidad directa offline y file:// -->
<script id="embedded-pe-m3u" type="text/plain">
#EXTM3U
#EXTINF:-1 tvg-id="36TVHuacho.pe@SD" tvg-logo="https://i.imgur.com/F4NZM3s.jpeg" group-title="General",36 TV Huacho (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/36tvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="ABCTeleshow.pe@SD" tvg-logo="https://i.imgur.com/cF7gJ2Q.jpeg" group-title="General",ABC Teleshow (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/abctv/playlist.m3u8
#EXTINF:-1 tvg-id="ABTelevision.pe@SD" tvg-logo="https://i.imgur.com/tqMDscD.png" group-title="General",ABTelevision (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/abctelevisionSRT/playlist.m3u8
#EXTINF:-1 tvg-id="AgroTV.pe@SD" tvg-logo="https://i.imgur.com/1uLGa3w.png" group-title="Outdoor",Agro TV (Peru) (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/agroSRT/playlist.m3u8
#EXTINF:-1 tvg-id="AlturaTV.pe@SD" tvg-logo="https://i.imgur.com/U6GwIL7.jpeg" group-title="General",Altura TV (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/urbanatv/playlist.m3u8
#EXTINF:-1 tvg-id="AmazoniaTV.pe@SD" tvg-logo="https://i.imgur.com/sSW7d8e.png" group-title="General",Amazonia TV
https://servilive.com:3199/live/amazoniatvlive.m3u8
#EXTINF:-1 tvg-id="AmazonicaTV.pe@SD" tvg-logo="https://i.imgur.com/22ebHAX.jpeg" group-title="General",Amazonica TV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/amazonicatvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="AmericaInternacional.pe@SD" tvg-logo="https://i.imgur.com/4ZTKAGd.png" group-title="General",America Internacional (720p)
https://api-ott.americatv.com.pe/loggingmediaurlpassthrough/a.m3u8?ads.gdpr=0&ap=1&app_name=Whale+TV%2B&app_store=Browser&brand_name=&cb=&content_categories=IAB19-29%2CIAB12&content_custom_1_param=OTTera&content_custom_2_param=F%2CM&content_custom_3_param=18-20%2C21-24%2C25-29%2C30-34%2C35-39%2C40-44%2C45-49%2C50-54%2C55-59&content_episode=&content_genre=News&content_id=&content_livestream=1&content_producer_name=OTTera_AmericaTV_AmericaTelevision&content_season=&content_series=&content_title=&coppa=0&device_make=&device_model=&did=&dnt=&dur=&gdpr=1&gdpr_consent=&h=&ic=IAB1&id=10&ip=&language=&lmt=&max_dur=30&min_dur=5&mute=0&partner=whaletv&placement=1&rating=12%2B&skip=0&tag_id=139913&tv_language=&tv_model=&tv_vendor=&ua=&url=&us_privacy=1YNN&version=12&w=&wtv_channel_id=0470
#EXTINF:-1 tvg-id="AmericaTelevision.pe@SD" tvg-logo="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4c/Am%C3%A9rica_Televisi%C3%B3n.svg/500px-Am%C3%A9rica_Televisi%C3%B3n.svg.png" group-title="General",America Television (1080p)
http://190.93.224.42/AMERICA-TV/index.m3u8
#EXTINF:-1 tvg-id="AndesTelevision.pe@SD" tvg-logo="https://i.imgur.com/CsW4Op8.png" group-title="Religious",Andes Television (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/andestvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="AndinaRTV.pe@SD" tvg-logo="https://i.imgur.com/KoQPpVh.jpeg" group-title="Music",Andina RTV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/andinatvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="AntaresTelevision.pe@SD" tvg-logo="https://i.imgur.com/05gwtk2.png" group-title="Entertainment",Antares Television
https://5949aa132c8fb.streamlock.net:1963/iptvantares/liveantarestv/playlist.m3u8
#EXTINF:-1 tvg-id="694" tvg-name="Antares TV" tvg-logo="https://cdn.m3u.cl/logo/694_Antares_TV.png", Antares TV | PE
https://5c3fb01839654.streamlock.net:1963/iptvantares/liveantarestv/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="Antena1.pe@SD" tvg-logo="https://i.imgur.com/Bt0yGtw.jpeg" http-user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" group-title="General",Antena 1 (Peru) (1080p)
#EXTVLCOPT:http-user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/antenauno.stream/playlist.m3u8
#EXTINF:-1 tvg-id="AntenaSUR.pe@SD" tvg-logo="https://i.imgur.com/4aHrD2l.jpeg" group-title="General",Antena SUR (720p)
https://cloudpro.servidoresdestream.com:8081/8010/index.m3u8
#EXTINF:-1 tvg-id="ArpegioTV.pe@SD" tvg-logo="https://i.imgur.com/mxnKK1a.png" group-title="General",Arpegio TV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/arpegioSRT/playlist.m3u8
#EXTINF:-1 tvg-id="AsiriTV.pe@SD" tvg-logo="https://i.imgur.com/7bGVFBa.png" group-title="Culture",AsiriTV (720p) [Not 24/7]
https://video2.lhdserver.es/asiritv/live.m3u8
#EXTINF:-1 tvg-id="AtlantisRadioTV.pe@SD" tvg-logo="https://i.imgur.com/1eB9roP.png" group-title="General",Atlantis Radio TV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/atlantisSRT/playlist.m3u8
#EXTINF:-1 tvg-id="ATV.pe@SD" tvg-logo="https://upload.wikimedia.org/wikipedia/commons/thumb/3/32/ATV_Peru_2025.svg/1920px-ATV_Peru_2025.svg.png" group-title="General",ATV (Peru) (1080p)
http://190.93.224.42/ATV/index.m3u8
#EXTINF:-1 tvg-id="ATVSur.pe@SD" tvg-logo="https://i.imgur.com/LnyVa5H.png" group-title="General",ATV Sur (1080p)
http://190.93.224.42/ATV-SUR/index.m3u8
#EXTINF:-1 tvg-id="1492" tvg-name="ATV Sur" tvg-logo="https://cdn.m3u.cl/logo/1492_ATV_Sur.png", ATV Sur | PE
https://d3r5ioyjqcy69v.cloudfront.net/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="1065" tvg-name="ATV" tvg-logo="https://cdn.m3u.cl/logo/1065_ATV.png", ATV | PE
https://d19e55ehz2il4i.cloudfront.net/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="ATVPlus.pe@SD" tvg-logo="https://upload.wikimedia.org/wikipedia/commons/5/56/ATV%2B_logo_2020.png" group-title="News",ATV+ (1080p)
http://45.171.108.253:8888/ATV/index.m3u8
#EXTINF:-1 tvg-id="1066" tvg-name="ATV+" tvg-logo="https://cdn.m3u.cl/logo/1066_ATV_.png", ATV+ | PE
https://d1svme839xkbsn.cloudfront.net/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="AutenticaTelevision.pe@SD" tvg-logo="https://i.imgur.com/bEJK3RN.png" group-title="General",Autentica Television (720p) [Not 24/7]
https://live.obslivestream.com/autenticatvmux/index.m3u8
#EXTINF:-1 tvg-id="1089" tvg-name="Autentica TV" tvg-logo="https://cdn.m3u.cl/logo/1089_Autentica_TV.png", Autentica TV | PE
https://live.obslivestream.com/autenticatvmux/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="BellaAbanquinaTV.pe@SD" tvg-logo="https://i.imgur.com/bORM4rt.png" group-title="Undefined",Bella Abanquina TV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/bellaabanquinaSRT/playlist.m3u8
#EXTINF:-1 tvg-id="BethelTV.pe@SD" tvg-logo="https://i.imgur.com/zCVh765.png" group-title="Religious",Bethel TV (720p)
https://alfa.betheltv.tv/srt/3_abr/playlist.m3u8
#EXTINF:-1 tvg-id="695" tvg-name="BHTV" tvg-logo="https://cdn.m3u.cl/logo/695_BHTV.png", BHTV | PE
https://sc1.wasidata.com/Bhtv/index.fmp4.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="BicolorPlus.pe@SD" tvg-logo="https://img-evg3.tv360.bitel.com.pe/peru-image/2026/09/24/00/1790182833200/6a8e1eb7f5d6_640_360.png" group-title="Sports",Bicolor+ (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/bicolorfree_abr/playlist.m3u8
#EXTINF:-1 tvg-id="BicolorPlusPremium2.pe@SD" tvg-logo="https://img-evg3.tv360.bitel.com.pe/peru-image/2026/09/24/03/1790194132371/1e57475a9a6b_640_360.png" group-title="Sports",Bicolor+ Premium 2 (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/bicolorpremium2_abr/playlist.m3u8
#EXTINF:-1 tvg-id="BicolorPlusPremium3.pe@SD" tvg-logo="https://img-evg3.tv360.bitel.com.pe/peru-image/2026/09/24/03/1790194198636/d91910da3254_640_360.png" group-title="Sports",Bicolor+ Premium 3 (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/bicolorpremium3_abr/playlist.m3u8
#EXTINF:-1 tvg-id="BicolorPlusPremium.pe@SD" tvg-logo="https://img-evg3.tv360.bitel.com.pe/peru-image/2026/09/24/00/1790183086681/1dff7002a22a_640_360.png" group-title="Sports",Bicolor+ Premium [Geo-blocked]
https://live-evg10.tv360.bitel.com.pe/bitel/bicolorpremium_abr/bitel/bicolorpremium_720p/chunks.m3u8
#EXTINF:-1 tvg-id="BNBDigital.pe@SD" tvg-logo="https://i.imgur.com/MgEN1F8.png" group-title="General",BNB Digital (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/bnbSRT/playlist.m3u8
#EXTINF:-1 tvg-id="CadenaTV.pe@HD" tvg-logo="https://i.imgur.com/stvzqzd.png" group-title="Undefined",Cadena TV (720p) [Not 24/7]
https://tv.bitstreaming.net:3789/live/cdntvlive.m3u8
#EXTINF:-1 tvg-id="CadenaTV.pe@SD" tvg-logo="https://i.imgur.com/stvzqzd.png" group-title="Undefined",Cadena TV SD
https://live-evg10.tv360.bitel.com.pe/bitel/cadenatv/playlist.m3u8
#EXTINF:-1 tvg-id="CajamarcaTV.pe@SD" tvg-logo="https://i.imgur.com/pC57NYv.png" group-title="General",Cajamarca TV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/cajamarcatvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="Canal21Huancayo.pe@HD" tvg-logo="https://i.imgur.com/Kfdkm06.png" group-title="General",Canal 21 Huancayo (720p)
https://play.agenciastreaming.com:8081/tvwanka/tracks-v1/index.fmp4.m3u8
#EXTINF:-1 tvg-id="Canal25Ayacucho.pe@SD" tvg-logo="https://i.imgur.com/lU02cWc.png" group-title="General",Canal 25 Ayacucho (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/canal25ayacucho/playlist.m3u8
#EXTINF:-1 tvg-id="Canal8Catacaos.pe@SD" tvg-logo="https://i.imgur.com/O5MrFjY.png" group-title="General",Canal 8 Catacaos (720p) [Not 24/7]
https://live.obslivestream.com/canal8/index.m3u8
#EXTINF:-1 tvg-id="1097" tvg-name="Canal 8 Catacaos" tvg-logo="https://cdn.m3u.cl/logo/1097_Canal_8_Catacaos.png", Canal 8 Catacaos | PE
https://live.obslivestream.com/canal8/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="CanalIPe.pe@SD" tvg-logo="https://upload.wikimedia.org/wikipedia/commons/thumb/0/04/Logo_Canal_IPe_2024.png/960px-Logo_Canal_IPe_2024.png" group-title="Family",Canal IPe (720p)
http://187.102.210.46/IPE/index.m3u8
#EXTINF:-1 tvg-id="CANALMTelevision.pe@SD" tvg-logo="https://i.imgur.com/EcdNFuJ.png" group-title="General",CANAL M Television (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/anqaratv/playlist.m3u8
#EXTINF:-1 tvg-id="CanalMasTV.pe@SD" tvg-logo="https://i.imgur.com/rdHygJ1.png" group-title="General",Canal Mas TV (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/mastvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="CanalN.pe@SD" tvg-logo="https://i.imgur.com/Xytvw3J.png" group-title="News",Canal N (1080p)
http://187.102.208.209:8000/play/a0az/index.m3u8
#EXTINF:-1 tvg-id="CasmaTV.pe@SD" tvg-logo="https://i.imgur.com/42aE0nm.jpeg" group-title="General",Casma TV (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/casmatvSRT-TRA/playlist.m3u8
#EXTINF:-1 tvg-id="CharapaTV.pe@SD" tvg-logo="https://i.imgur.com/tFRmMkf.png" group-title="General",Charapa TV (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/charapatvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="Chikitoonz.pe@SD" tvg-logo="https://chikitoonz.com/site/wp-content/uploads/2024/08/logochikitoonz2.png" group-title="Kids",Chikitoonz (1080p)
http://45.70.201.81:8000/play/a00p/index.m3u8
#EXTINF:-1 tvg-id="Conecta2TV.pe@SD" tvg-logo="https://i.imgur.com/W6FDUwt.png" group-title="News",Conecta2TV (720p) [Not 24/7]
https://live.obslivestream.com/conecta2/index.m3u8
#EXTINF:-1 tvg-id="ControversiaTV.pe@SD" tvg-logo="https://i.imgur.com/X7cPyP2.png" group-title="Undefined",Controversia TV (720p) [Not 24/7]
https://live.obslivestream.com/controversiatv/index.m3u8
#EXTINF:-1 tvg-id="1188" tvg-name="Controversia TV" tvg-logo="https://cdn.m3u.cl/logo/1188_Controversia_TV.png", Controversia TV | PE
https://live.obslivestream.com/controversiatv/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="CRTelevision.pe@SD" tvg-logo="https://i.imgur.com/oxycMeA.png" group-title="General",CR Television (720p) [Not 24/7]
https://live.obslivestream.com/crtvmux/index.m3u8
#EXTINF:-1 tvg-id="1180" tvg-name="CR TV" tvg-logo="https://cdn.m3u.cl/logo/1180_CR_TV.png", CR TV | PE
https://live.obslivestream.com/crtvmux/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="Cultura24.pe@SD" tvg-logo="https://i.imgur.com/h0LxXsx.png" group-title="Culture",Cultura 24 (720p)
https://www.opencaster.com/resources/hls_stream/cultura24.m3u8
#EXTINF:-1 tvg-id="696" tvg-name="Cultura 24 TV" tvg-logo="https://cdn.m3u.cl/logo/696_Cultura_24_TV.png", Cultura 24 TV | PE
https://www.opencaster.com/resources/hls_stream/cultura24.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="CVNTV.pe@SD" tvg-logo="https://i.imgur.com/QF5VUcy.png" group-title="General",CVN TV
https://7.innovatestream.pe:19360/cvntumbes/cvntumbes.m3u8
#EXTINF:-1 tvg-id="DacTV.pe@SD" tvg-logo="https://i.imgur.com/g4pZ3ZW.jpeg" group-title="General",Dac TV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/dactvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="DeltaTV.pe@SD" tvg-logo="https://i.imgur.com/crttwAR.png" group-title="Undefined",DeltaTV
https://tv2.bitstreaming.net:3539/live/deltanetlive.m3u8
#EXTINF:-1 tvg-id="DiasporaTV.pe@SD" tvg-logo="https://i.imgur.com/lppmyxD.png" group-title="Culture",Diaspora TV
https://s1.tvdatta.com:3978/live/diasporatvlive.m3u8
#EXTINF:-1 tvg-id="978" tvg-name="DMJ" tvg-logo="https://cdn.m3u.cl/logo/978_DMJ.png", DMJ | PE
https://stmv1.voxhdnet.com/dmjsurtv/dmjsurtv/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="DobleCTelevision.pe@SD" tvg-logo="https://i.imgur.com/2zCD4TB.png" group-title="General",Doble C Television (720p)
https://live20.bozztv.com/giatv/giatv-doblec1/doblec1/playlist.m3u8
#EXTINF:-1 tvg-id="DTV.pe@SD" tvg-logo="https://i.imgur.com/OSvegAq.png" group-title="General",DTV (Peru) (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/sabormixtvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="ECOTelevision.pe@SD" tvg-logo="https://i.imgur.com/ly1KU6w.png" group-title="General",ECO Television (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/ecotv/playlist.m3u8
#EXTINF:-1 tvg-id="1555" tvg-name="Emanuel7 TV" tvg-logo="https://cdn.m3u.cl/logo/1555_Emanuel7_TV.png", Emanuel7 TV | PE
https://stv1.voxbr.com.br/emanuel7tv/emanuel7tv/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="EvolucionTV.pe@SD" tvg-logo="https://i.imgur.com/OUsT530.jpeg" group-title="General",Evolucion TV (576p)
https://edgectc.com/CTC_SAN_RAFAEL/index.m3u8
#EXTINF:-1 tvg-id="ExitosaTV.pe@SD" tvg-logo="https://i.imgur.com/pu0BSB7.png" group-title="News",Exitosa TV (1080p)
http://190.93.224.42/EXITOSA/index.m3u8
#EXTINF:-1 tvg-id="691" tvg-name="Exitosa" tvg-logo="https://cdn.m3u.cl/logo/691_Exitosa.png", Exitosa | PE
https://streamtv.onliv3.online:1936/exito-tv/exito-tv/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="FolkloreTV.pe@SD" tvg-logo="https://i.imgur.com/QdOGJes.png" group-title="Music",Folklore TV (720p)
https://play.folkloretv.pe/StreamSD/index.m3u8
#EXTINF:-1 tvg-id="ForoTV.pe@SD" tvg-logo="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTarTiQHz2Pl0Bpx9g7JV_eaz4V4f61wtGyRw&s" group-title="Undefined",Foro TV (576p)
https://mc.servidor.stream:19360/8274/8274.m3u8
#EXTINF:-1 tvg-id="1659" tvg-name="Foro TV" tvg-logo="https://cdn.m3u.cl/logo/1659_Foro_TV.png", Foro TV | PE
https://mc.servidor.stream:19360/8274/8274.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="FrecuenciaAmazonica.pe@SD" tvg-logo="https://i.imgur.com/jVkzHw0.png" group-title="Undefined",Frecuencia Amazonica
https://vivatv.juanjuiserver.com:3349/live/frecuenciaamazonicalive.m3u8
#EXTINF:-1 tvg-id="FuegoTV.pe@SD" tvg-logo="https://i.imgur.com/1LuNEX4.png" http-user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" group-title="News",FuegoTV (1080p)
#EXTVLCOPT:http-user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/fuegotv.stream/playlist.m3u8
#EXTINF:-1 tvg-id="GalacticaTV.pe@SD" tvg-logo="https://i.imgur.com/byL1zCV.jpeg" group-title="Music",Galactica TV
https://stream.neux.site/memfs/e3f59eec-6b93-45e1-a2ab-c82381c2a9ec.m3u8
#EXTINF:-1 tvg-id="GeniosTV.pe@SD" tvg-logo="https://i.imgur.com/iH2Lx6E.png" group-title="General",Genios TV (1080p)
https://sc3.wasidata.com/Genios2_1902/video.m3u8
#EXTINF:-1 tvg-id="1093" tvg-name="Genios TV" tvg-logo="https://cdn.m3u.cl/logo/1093_Genios_TV.png", Genios TV | PE
https://live.obslivestream.com/geniostvmux/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="GlobalTV.pe@SD" tvg-logo="https://i.imgur.com/yuEou9C.png" group-title="General",Global TV (Peru) (1080p)
http://190.93.224.42/GLOBAL/index.m3u8
#EXTINF:-1 tvg-id="697" tvg-name="Hechicera TV" tvg-logo="https://cdn.m3u.cl/logo/697_Hechicera_TV.png", Hechicera TV | PE
https://video.misistemareseller.com:443/corporationhc/corporationhc/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="HuachoPeruTV.pe@SD" tvg-logo="https://i.imgur.com/PevmfqK.png" group-title="General",Huacho Peru TV
https://tv.huachotv.com/stream.m3u8
#EXTINF:-1 tvg-id="ImpactoTelevision.pe@SD" tvg-logo="https://i.imgur.com/bkpCGP6.png" group-title="Entertainment",Impacto Television (360p) [Not 24/7]
https://cloudvideo.servers10.com:8081/impactotv/index.m3u8
#EXTINF:-1 tvg-id="1740" tvg-name="Impacto TV" tvg-logo="https://cdn.m3u.cl/logo/1740_Impacto_TV.png", Impacto TV | PE
https://cloudpro.servidoresdestream.com:8081/8010/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="ImperialTelevision.pe@SD" tvg-logo="https://i.imgur.com/eYXqkyN.png" group-title="Undefined",Imperial Television (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/imperialSRT/playlist.m3u8
#EXTINF:-1 tvg-id="IntuitivaTV.pe@SD" tvg-logo="https://i.imgur.com/zcehR4M.png" group-title="General",Intuitiva TV (720p)
https://video2.lhdserver.es/cableperu/live.m3u8
#EXTINF:-1 tvg-id="JN19.pe@SD" tvg-logo="https://i.imgur.com/dudxyqk.png" group-title="Religious",JN 19 (720p)
http://187.102.210.46/JN19/index.m3u8
#EXTINF:-1 tvg-id="690" tvg-name="JN19" tvg-logo="https://cdn.m3u.cl/logo/690_JN19.png", JN19 | PE
https://servilive.com:3028/live/jntv19live.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="JNETV.pe@SD" tvg-logo="https://i.imgur.com/b25XfCj.png" group-title="Legislative",JNE TV (720p)
https://lbgo.bozztv.com/ssh101/ssh101/jnetv/playlist.m3u8
#EXTINF:-1 tvg-id="JuntosTV.pe@SD" tvg-logo="https://upload.wikimedia.org/wikipedia/commons/b/bd/Juntos_TV_Logo.png" http-user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/150.0.0.0 Safari/537.36" group-title="General",Juntos TV (Peru) (1080p)
#EXTVLCOPT:http-user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/150.0.0.0 Safari/537.36
http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/juntos_tv.stream/playlist.m3u8
#EXTINF:-1 tvg-id="JusticiaTV.pe@SD" tvg-logo="https://i.imgur.com/VOjLXJd.png" group-title="Legislative",Justicia TV (Peru) (1080p)
http://190.93.224.42/JUSTICIA-TV/index.m3u8
#EXTINF:-1 tvg-id="Karibena.pe@SD" tvg-logo="https://i.imgur.com/cgHjg1e.png" group-title="Music",Karibena (720p)
http://187.102.210.46/KARIBENA/index.m3u8
#EXTINF:-1 tvg-id="KoraTV.pe@SD" tvg-logo="https://i.imgur.com/ahBYv4x.png" group-title="Entertainment",Kora TV
https://s3.tvdatta.com:3700/live/jjrtvlive.m3u8
#EXTINF:-1 tvg-id="L1.pe@SD" tvg-logo="https://i.imgur.com/eRmXneK.png" group-title="Sports",L1 (1080p)
http://45.4.1.201:8000/play/a1ai/index.m3u8
#EXTINF:-1 tvg-id="L1Max.pe@SD" tvg-logo="https://i.imgur.com/2fziU2B.png" group-title="Sports",L1 Max (1080p)
http://190.93.224.42/LIGA-1-MAX/index.m3u8
#EXTINF:-1 tvg-id="LaFabulosaTV.pe@SD" tvg-logo="https://i.imgur.com/GjME5H0.png" group-title="Music",La Fabulosa TV (720p)
https://eu1.servers10.com:8081/8004/index.m3u8
#EXTINF:-1 tvg-id="LaTele.pe@SD" tvg-logo="https://upload.wikimedia.org/wikipedia/commons/thumb/8/80/LaTele_Per%C3%BA_2018_Logo.png/960px-LaTele_Per%C3%BA_2018_Logo.png" group-title="Entertainment",La Tele (1080p)
http://190.93.224.42/LA-TELE/index.m3u8
#EXTINF:-1 tvg-id="LaTremendaTV.pe@TDT62" tvg-logo="https://i.pinimg.com/736x/46/24/d3/4624d356bfdaa6254c2597f686108841.jpg" group-title="General",La Tremenda TV (1080p)
https://iptv.latremenda.pe/LaTremendaTv/tracks-v1a1/mono.m3u8
#EXTINF:-1 tvg-id="Latina.pe@SD" tvg-logo="https://i.imgur.com/MOsWriY.png" group-title="General",Latina (1080p)
http://190.93.224.42/LATINA/index.m3u8
#EXTINF:-1 tvg-id="LatinaClasicos.pe@SD" tvg-logo="https://i.imgur.com/q0dAWyj.png" group-title="Series",Latina Clasicos (1080p)
http://45.171.108.253:8888/LATINA/index.m3u8
#EXTINF:-1 tvg-id="1741" tvg-name="Latina Clasicos" tvg-logo="https://cdn.m3u.cl/logo/1741_Latina_Clasicos.png", Latina Clasicos | PE
https://redirector.rudo.video/hls-video/plus226/latina2/latina2.smil/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="LatinaNoticias247.pe@SD" tvg-logo="https://assets.rudo.video/assets/latina/live/logoApi_4b43aad719f5bbf7aa2813c2199e6e3e1755620584.png" group-title="News",Latina Noticias 24/7 (1080p)
https://dai.google.com/linear/hls/event/oYQGDqEGTmGdHE_fz9oLlg/master.m3u8
#EXTINF:-1 tvg-id="1742" tvg-name="Latina Noticias" tvg-logo="https://cdn.m3u.cl/logo/1742_Latina_Noticias.png", Latina Noticias | PE
https://redirector.rudo.video/hls-video/567ffde3fa319fadf3419efda25619456231dfea/latinanoticias/latinanoticias.smil/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="LatinaPodcast.pe@SD" tvg-logo="https://assets.rudo.video/assets/latina/live/backgroundApi_347d1f0b8f6b597bc08abeb60c975c471769447202.png" group-title="General",Latina Podcast (720p)
https://redirector.rudo.video/hls-video/plus226/latina4/latina4.smil/playlist.m3u8
#EXTINF:-1 tvg-id="1657" tvg-name="Latina" tvg-logo="https://cdn.m3u.cl/logo/1657_Latina.png", Latina | PE
https://redirector.rudo.video/hls-video/567ffde3fa319fadf3419efda25619456231dfea/latina/latina.smil/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="LiderTV.pe@SD" tvg-logo="https://i.imgur.com/naZXUdR.jpeg" group-title="General",Lider TV (Peru) (720p)
https://chankillo.agenciastreaming.com/lidertv/index.m3u8
#EXTINF:-1 tvg-id="LotPlusTV.pe@SD" tvg-logo="https://i.imgur.com/gBbCDCL.png" group-title="General",LotPlus TV (720p)
https://ontvcable.streamottmas.workers.dev/ontvcable.m3u8
#EXTINF:-1 tvg-id="MasterTV.pe@SD" tvg-logo="https://i.imgur.com/K2ZimRv.png" group-title="General",Master TV (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/mastertvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="MediaTV.pe@SD" tvg-logo="https://i.imgur.com/BAaR5T5.png" group-title="General",Media TV (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/mediatv/playlist.m3u8
#EXTINF:-1 tvg-id="689" tvg-name="Mega TV Arequipa" tvg-logo="https://cdn.m3u.cl/logo/689_Mega_TV_Arequipa.png", Mega TV Arequipa | PE
https://mc.servidor.stream:19360/megatv/megatv.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="MegaTVJaen.pe@SD" tvg-logo="https://i.imgur.com/zK1YcpJ.png" group-title="General",Mega TV Jaen (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/megatv/playlist.m3u8
#EXTINF:-1 tvg-id="1491" tvg-name="Mega TV Jaen" tvg-logo="https://cdn.m3u.cl/logo/1491_Mega_TV_Jaen.png", Mega TV Jaen | PE
https://7.innovatestream.pe:19360/megatvjaen/megatvjaen.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="MetropolitanadelCuzco.pe@SD" tvg-logo="https://i.imgur.com/0B04qgM.png" group-title="Music",Metropolitana del Cuzco
https://video1.earthcam.com/myearthcam/075ff02f78c35af55564cf3af3b3f750.flv/playlist.m3u8
#EXTINF:-1 tvg-id="MielTV.pe@SD" tvg-logo="https://i.imgur.com/dPSFrCs.png" group-title="Music",Miel TV (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/mieltvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="MilenialTV.pe@SD" tvg-logo="https://i.imgur.com/8ZiNoEa.png" group-title="General",Milenial TV
https://www.milenial.news/salida/milenial.m3u8
#EXTINF:-1 tvg-id="Millenium109FM.pe@SD" tvg-logo="https://i.imgur.com/sfNL6Bv.png" group-title="Music",Millenium 109 FM
https://videoserver.tmcreativos.com:19360/nqvnhujhrx/nqvnhujhrx.m3u8
#EXTINF:-1 tvg-id="MonterricoTV.pe@SD" tvg-logo="https://i.imgur.com/SuVO9T7.png" group-title="Sports",Monterrico TV
https://h.jcp.live.opencaster.com/play_src/index.m3u8
#EXTINF:-1 tvg-id="MovistarDeportes.pe@SD" tvg-logo="https://i.imgur.com/Zt3iE86.png" group-title="Sports",Movistar Deportes (1080p)
http://195.178.110.110/movistar/index.m3u8
#EXTINF:-1 tvg-id="MovistarPlus.pe@SD" tvg-logo="https://i.imgur.com/yGQGDoi.png" group-title="Entertainment",Movistar Plus (576p)
http://45.4.1.201:8000/play/a19y/index.m3u8
#EXTINF:-1 tvg-id="771" tvg-name="Mundo de la Musica TV" tvg-logo="https://cdn.m3u.cl/logo/771_Mundo_de_la_Musica_TV.png", Mundo de la Musica TV | CL
https://tv.arkeo.cl:1936/mundodelamusica/mundodelamusica/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="NacionalTvPeru.pe@SD" tvg-logo="https://i.imgur.com/ad0GNqa.png" group-title="General",Nacional Tv Peru (720p)
http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/nacionaltv_peru.stream/playlist.m3u8
#EXTINF:-1 tvg-id="Nativa.pe@SD" tvg-logo="https://i.imgur.com/Y9k9XSO.png" group-title="General",Nativa (720p)
http://187.102.210.46/NATIVATV/index.m3u8
#EXTINF:-1 tvg-id="NativaTVSatipo.pe@SD" tvg-logo="https://i.imgur.com/ScCAgGB.png" group-title="General",Nativa TV Satipo (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/nativachanchamayoSRT/playlist.m3u8
#EXTINF:-1 tvg-id="684" tvg-name="Nativa" tvg-logo="https://cdn.m3u.cl/logo/684_Nativa.png", Nativa | PE
https://oneplay.iptvperu.tv:1936/static/nativa1.stream/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="NeoTV.pe@SD" tvg-logo="https://i.imgur.com/bS5JtYX.jpeg" group-title="Entertainment",Neo TV (Peru) (720p)
http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/DigitalTV/playlist.m3u8
#EXTINF:-1 tvg-id="NZKTelevision.pe@SD" tvg-logo="https://i.imgur.com/IqsB6VR.jpeg" group-title="General",NZK Television (720p)
https://eu1.servers10.com:8081/8258/index.m3u8
#EXTINF:-1 tvg-id="1095" tvg-name="OK TeVe" tvg-logo="https://cdn.m3u.cl/logo/1095_OK_TeVe.png", OK TeVe | PE
https://live.obslivestream.com/oktv/video.m3u8?token=Bk-DXAXulO1OV0&PlaylistM3UCL
#EXTINF:-1 tvg-id="ONDADIGITAL.pe@SD" tvg-logo="https://i.imgur.com/3R2emx7.jpg" group-title="General",ONDA DIGITAL (720p)
https://tv.ondadigital.pe/ondadigital/index.m3u8
#EXTINF:-1 tvg-id="1667" tvg-name="Onda Digital TV 2" tvg-logo="https://cdn.m3u.cl/logo/1667_Onda_Digital_TV_2.png", Onda Digital TV 2 | PE
https://v4.tustreaming.cl:443/odtv2/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="969" tvg-name="Onda Digital TV" tvg-logo="https://cdn.m3u.cl/logo/969_Onda_Digital_TV.png", Onda Digital TV | PE
https://v4.tustreaming.cl:443/ondadigitaltv/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="OndaTV.pe@SD" tvg-logo="https://i.imgur.com/gbjdiLW.jpeg" group-title="General",Onda TV (Peru) (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/onda/playlist.m3u8
#EXTINF:-1 tvg-id="OvacionTV.pe@SD" tvg-logo="https://i.imgur.com/7Xndgxk.png" group-title="Sports",Ovacion TV (720p) [Not 24/7]
http://cdn2.ujjina.com:1935/iptvovacion1/liveovacion1tv/playlist.m3u8
#EXTINF:-1 tvg-id="699" tvg-name="Ovacion" tvg-logo="https://cdn.m3u.cl/logo/699_Ovacion.png", Ovacion | PE
https://5c3fb01839654.streamlock.net:1963/iptvovacion1/liveovacion1tv/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="OzonoTV.pe@SD" tvg-logo="https://i.imgur.com/Wkdibpv.png" group-title="General",Ozono TV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/ozonotv/playlist.m3u8
#EXTINF:-1 tvg-id="1290" tvg-name="Palmeras TV" tvg-logo="https://cdn.m3u.cl/logo/1290_Palmeras_TV.png", Palmeras TV | PE
https://play.agenciastreaming.com:8081/palmerastv/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="PanamericanaTV.pe@SD" tvg-logo="https://upload.wikimedia.org/wikipedia/commons/thumb/7/71/Logo_Panamericana_2020.png/1280px-Logo_Panamericana_2020.png" group-title="General",Panamericana TV (1080p)
http://45.171.108.253:8888/PANAMERICANA/index.m3u8
#EXTINF:-1 tvg-id="1578" tvg-name="PANC TV Peru" tvg-logo="https://cdn.m3u.cl/logo/1578_PANC_TV_Peru.png", PANC TV Peru | PE
https://panctv.ddns.net/hls/pe/live.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="PBO.pe@SD" tvg-logo="https://i.imgur.com/vBpxX4S.png" group-title="News",PBO (1080p)
http://190.93.224.42/PBO/index.m3u8
#EXTINF:-1 tvg-id="PequesTV.pe@SD" tvg-logo="https://img-evg3.tv360.bitel.com.pe/peru-image/2026/01/06/04/1767647409377/6123aa5e7fc7.png" group-title="Kids",Peques TV (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/pequestv/playlist.m3u8
#EXTINF:-1 tvg-id="PiuraTV.pe@SD" tvg-logo="https://i.imgur.com/KnJAqbd.png" group-title="General",PiuraTV (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/piuratvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="PlaneaTV.pe@SD" tvg-logo="https://i.imgur.com/EoLhvBN.png" group-title="Undefined",Planea TV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/planeatvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="1092" tvg-name="Planeta TV" tvg-logo="https://cdn.m3u.cl/logo/1092_Planeta_TV.png", Planeta TV | PE
https://live.obslivestream.com/planetatv/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="PrattelTV.pe@SD" tvg-logo="https://pratteltv.lat/wp-content/uploads/2026/05/cropped-V2.jpg" http-referrer="https://www.pratteltv.lat" group-title="Music",Prattel TV (720p)
#EXTVLCOPT:http-referrer=https://www.pratteltv.lat
https://183.bozztv.com/ssh101/ssh101/pratteltvperu/playlist.m3u8
#EXTINF:-1 tvg-id="Primavera15.pe@SD" tvg-logo="https://i.imgur.com/wsJ8Ag2.png" group-title="General",Primavera 15
https://mc.servidor.stream:19360/8244/8244.m3u8
#EXTINF:-1 tvg-id="QollasuyoTV.pe@SD" tvg-logo="https://i.imgur.com/6bi7fGB.jpeg" group-title="General",Qollasuyo TV (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/qollasuyotv/playlist.m3u8
#EXTINF:-1 tvg-id="QosqoTimes.pe@SD" tvg-logo="https://i.imgur.com/oWJauwQ.png" group-title="News",Qosqo Times (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/qosqo/playlist.m3u8
#EXTINF:-1 tvg-id="RadioChalaca.pe@SD" tvg-logo="https://i.imgur.com/9A6DBG7.png" group-title="Music",Radio Chalaca
https://servilive.com:3414/hybrid/play.m3u8
#EXTINF:-1 tvg-id="RadioMaster.pe@HD" tvg-logo="https://i.imgur.com/AUoxyzP.png" group-title="Music",Radio Master (720p)
https://videoserver.tmcreativos.com:19360/radiomaster/radiomaster.m3u8
#EXTINF:-1 tvg-id="RadioOndaDigital.pe@SD" tvg-logo="https://i.imgur.com/Lid5RTv.png" group-title="Music",Radio Onda Digital (1080p)
https://live.ondadigitaltv.com/RADIOODTV/index.m3u8
#EXTINF:-1 tvg-id="RadioRumbaTV.pe@SD" tvg-logo="https://i.imgur.com/BYcxI03.png" group-title="Music",Radio Rumba TV (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/rumba/playlist.m3u8
#EXTINF:-1 tvg-id="RadioSanBorjaTV.pe@SD" tvg-logo="https://i.imgur.com/75dJBtH.png" group-title="Music",Radio San Borja TV (1080p)
https://stream.mediacorp.pe/srt/3/playlist.m3u8
#EXTINF:-1 tvg-id="RadioTropical.pe@SD" tvg-logo="https://i.imgur.com/TBPVOBI.png" group-title="Music",Radio Tropical (480p) [Not 24/7]
https://videoserver.tmcreativos.com:19360/raditropical/raditropical.m3u8
#EXTINF:-1 tvg-id="RadioUnoTacna.pe@SD" tvg-logo="https://i.imgur.com/PswowQP.png" group-title="News",Radio Uno Tacna (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/radiounotv/playlist.m3u8
#EXTINF:-1 tvg-id="RadioTVOriente.pe@SD" tvg-logo="https://i.imgur.com/iHaFLHX.png" group-title="General",RadioTV Oriente (720p)
https://live.obslivestream.com/tvoriente/index.m3u8
#EXTINF:-1 tvg-id="RealTelevision.pe@SD" tvg-logo="https://i.imgur.com/7KECyh5.jpeg" group-title="General",Real Television (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/realtv/playlist.m3u8
#EXTINF:-1 tvg-id="RegionTVCallao.pe@SD" tvg-logo="https://i.imgur.com/I4grcqk.png" group-title="Undefined",Region TV Callao (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/regiontvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="156" tvg-name="Rewind" tvg-logo="https://cdn.m3u.cl/logo/156_Rewind.png", Rewind ✪ | CL
https://www.rewindmusic.cl/senal/m3u/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="RikuTV.pe@SD" tvg-logo="https://i.imgur.com/XIDC4Mv.png" group-title="Entertainment",Riku TV (1080p)
https://amixtv.com:8081/rikutv/index.m3u8
#EXTINF:-1 tvg-id="RNTelevision.pe@SD" tvg-logo="https://i.imgur.com/oFGiW7u.png" group-title="Undefined",RN Television (720p) [Not 24/7]
https://videoserver.tmcreativos.com:19360/rnyurimaguas/rnyurimaguas.m3u8
#EXTINF:-1 tvg-id="RPPTV.pe@SD" tvg-logo="https://upload.wikimedia.org/wikipedia/commons/3/36/RPP_logo.png" group-title="News",RPP TV (1080p)
http://190.93.224.42/RPP/index.m3u8
#EXTINF:-1 tvg-id="RSelvaTV.pe@SD" tvg-logo="https://i.imgur.com/DAK5ZnD.png" group-title="News",RSelvaTV (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/selvaticaSRT/playlist.m3u8
#EXTINF:-1 tvg-id="RTP.pe@SD" tvg-logo="https://i.imgur.com/nHMqAHn.jpeg" group-title="General",RTP (Peru) (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/rtp/playlist.m3u8
#EXTINF:-1 tvg-id="RumbaMixTV.pe@SD" tvg-logo="https://i.imgur.com/rwA3HIp.png" group-title="Music",RumbaMix TV (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/mixtvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="RumboMineroTV.pe@SD" tvg-logo="https://www.rumbominero.com/wp-content/uploads/2021/05/logo_RM_AM_2.png" group-title="General",Rumbo Minero TV (1080p)
http://38.87.224.19:2220/play/a049/index.m3u8
#EXTINF:-1 tvg-id="1686" tvg-name="San Pedro SAT" tvg-logo="https://cdn.m3u.cl/logo/1686_San_Pedro_SAT.png", San Pedro SAT | PE
https://eu1.servers10.com:8081/8156/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="1187" tvg-name="Sanjuanera TV" tvg-logo="https://cdn.m3u.cl/logo/1187_Sanjuanera_TV.png", Sanjuanera TV | PE
https://infosmartpe.store:5443/LiveApp/streams/3zc2BXi5SEys2310139330128741.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="SenalPeruTV.pe@SD" tvg-logo="https://i.imgur.com/1q0R6R9.png" group-title="Undefined",Senal Peru TV (404p) [Not 24/7]
https://livestream.perucast.com/hls/stream.m3u8
#EXTINF:-1 tvg-id="SiscomTV.pe@SD" tvg-logo="https://i.imgur.com/0JUlFZ4.png" group-title="General",Siscom TV
https://video2.lhdserver.es/siscomtv/live.m3u8
#EXTINF:-1 tvg-id="SolarTelevision.pe@SD" tvg-logo="https://i.imgur.com/ufnWpUf.png" group-title="Undefined",Solar Television (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/solartvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="1658" tvg-name="Solar TV Cusco" tvg-logo="https://cdn.m3u.cl/logo/1658_Solar_TV_Cusco.png", Solar TV Cusco | PE
https://stmv1.voxhdnet.com/solartv/solartv/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="SolTV.pe@SD" tvg-logo="https://i.imgur.com/GtNkuAc.png" group-title="Undefined",SolTV (Peru) (720p)
https://5790d294af2dc.streamlock.net/streamtv/streamtv/playlist.m3u8
#EXTINF:-1 tvg-id="SqalaTV.pe@SD" tvg-logo="https://i.imgur.com/DJ05m8s.png" group-title="Sports",Sqala TV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/studiotvhuanta/playlist.m3u8
#EXTINF:-1 tvg-id="StudioPascoTV.pe@SD" tvg-logo="https://i.imgur.com/zgyNuVE.jpeg" group-title="General",Studio Pasco TV (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/studiopasco/playlist.m3u8
#EXTINF:-1 tvg-id="SuperCanal.pe@SD" tvg-logo="https://i.imgur.com/eotfCMd.png" group-title="Undefined",Super Canal (Peru) (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/supercanalSRT/playlist.m3u8
#EXTINF:-1 tvg-id="1546" tvg-name="Tele Selva" tvg-logo="https://cdn.m3u.cl/logo/1546_Tele_Selva.png", Tele Selva | PE
https://7.innovatestream.pe:19360/tvnoticiassatipo/tvnoticiassatipo.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="1189" tvg-name="Telecolor TV" tvg-logo="https://cdn.m3u.cl/logo/1189_Telecolor_TV.png", Telecolor TV | PE
https://live.obslivestream.com/telecolormux/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="TelecolorYurimaguas.pe@SD" tvg-logo="https://i.imgur.com/Ug6XNIU.png" group-title="General",Telecolor Yurimaguas (720p) [Not 24/7]
https://live.obslivestream.com/telecolormux/index.m3u8
#EXTINF:-1 tvg-id="Telelima.pe@SD" tvg-logo="https://telelima.pe/img/logo.png" http-user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" group-title="General",Telelima (720p)
#EXTVLCOPT:http-user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/telelima.stream/playlist.m3u8
#EXTINF:-1 tvg-id="Teleselva.pe@SD" tvg-logo="https://i.imgur.com/cu7hTn8.png" group-title="Undefined",Teleselva (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/teleselva/playlist.m3u8
#EXTINF:-1 tvg-id="TelesystemTV.pe@SD" tvg-logo="https://i.imgur.com/CI4CNgL.jpeg" group-title="General",Telesystem TV (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/telesystemSRT/playlist.m3u8
#EXTINF:-1 tvg-id="TelevisionTarapoto.pe@SD" tvg-logo="https://i.imgur.com/Dzen7sh.png" group-title="General",Television Tarapoto (1080p)
http://45.5.57.58:8000/play/a038/index.m3u8
#EXTINF:-1 tvg-id="TeveEmpresa.pe@SD" tvg-logo="https://img-evg3.tv360.bitel.com.pe/peru-image/2026/04/07/02/1775503150878/c34840061c94_480_270.png" group-title="General",Teve Empresa (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/tvempresaSRT/playlist.m3u8
#EXTINF:-1 tvg-id="Tevesur.pe@SD" tvg-logo="https://i.imgur.com/LcZ2Hhd.jpeg" group-title="General",Tevesur (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/tevesurSRT/playlist.m3u8
#EXTINF:-1 tvg-id="TopLatinoTV.pe@SD" tvg-logo="https://i.imgur.com/VRl8nPZ.png" group-title="Music",Top Latino TV
http://38.87.224.20:2220/play/a02b/index.m3u8
#EXTINF:-1 tvg-id="35" tvg-name="TOP Latino TV" tvg-logo="https://cdn.m3u.cl/logo/35_TOP_Latino_TV.png", TOP Latino TV | PE
https://5cefcbf58ba2e.streamlock.net:543/webtltv/webtltv.stream/playlist.m3u8?wowzatokenstarttime=1752707094&wowzatokenendtime=1752708094&wowzatokenhash=ehhxNAzcNA0F16D7rrGOSaTgY2M5_O38gPj7Ir3Lx2U=&PlaylistM3UCL
#EXTINF:-1 tvg-id="TopTV.pe@SD" tvg-logo="https://i.imgur.com/tiNvihZ.png" group-title="General",Top TV (240p) [Not 24/7]
https://tvdatta.com:3084/live/toptvaguaytialive.m3u8
#EXTINF:-1 tvg-id="Trivu.pe@SD" tvg-logo="https://i.imgur.com/QrUYjOS.png" group-title="General",Trivu (1080p)
http://190.93.224.42/TRIVU-TV/index.m3u8
#EXTINF:-1 tvg-id="1115" tvg-name="Tumpis TV" tvg-logo="https://cdn.m3u.cl/logo/1115_Tumpis_TV.png", Tumpis TV | PE
https://servilive.com:3531/stream/play.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="TurboMixRadioTV.pe@SD" tvg-logo="https://i.imgur.com/AsdBQa3.png" group-title="Music",Turbo Mix Radio TV (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/turbomixSRT/playlist.m3u8
#EXTINF:-1 tvg-id="43" tvg-name="Turbo Mix TV" tvg-logo="https://cdn.m3u.cl/logo/43_Turbo_Mix_TV.png", Turbo Mix TV | PE
https://7.innovatestream.pe:19360/turbomixoficial/turbomixoficial.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="TVBellaAsuncion.pe@SD" tvg-logo="https://i.imgur.com/H69eLFW.png" group-title="Undefined",TV Bella Asuncion
https://s2.tvdatta.com:3687/hybrid/play.m3u8
#EXTINF:-1 tvg-id="TVCosmos.pe@SD" tvg-logo="https://i.imgur.com/nr4ctt4.png" group-title="General",TV Cosmos (720p)
https://videoserver.tmcreativos.com:19360/tvcosmos/tvcosmos.m3u8
#EXTINF:-1 tvg-id="701" tvg-name="TV Cosmos 15.1" tvg-logo="https://cdn.m3u.cl/logo/701_TV_Cosmos_15_1.png", TV Cosmos 15.1 | PE
https://5790d294af2dc.streamlock.net:443/8134/8134/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="1590" tvg-name="TV en linea Cusco" tvg-logo="https://cdn.m3u.cl/logo/1590_TV_en_linea_Cusco.png", TV en linea Cusco | PE
https://s4.tvdatta.com:3450/live/tvenlinealive.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="TVKaliente.pe@SD" tvg-logo="https://i.imgur.com/MFQpLuQ.png" group-title="Entertainment",TV Kaliente (720p)
https://lbgo.bozztv.com/ssh101/ssh101/kalientecusco/playlist.m3u8
#EXTINF:-1 tvg-id="TVMAR.pe@SD" tvg-logo="https://i.imgur.com/VXvbNjT.png" group-title="General",TV MAR (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/tvmarSRT/playlist.m3u8
#EXTINF:-1 tvg-id="TVMAX.pe@SD" tvg-logo="https://i.imgur.com/JFBpDJ1.jpeg" group-title="General",TV MAX (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/nexSRT_TRA/playlist.m3u8
#EXTINF:-1 tvg-id="TVNorteChiclayo.pe@SD" tvg-logo="https://i.imgur.com/0fg6m2L.png" group-title="Undefined",TV Norte Chiclayo (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/tvnorte/playlist.m3u8
#EXTINF:-1 tvg-id="TVPeru.pe@SD" tvg-logo="https://i.imgur.com/6io0IrX.png" group-title="General",TV Peru (1080p)
http://181.224.200.5:2277/play/a0dx/index.m3u8
#EXTINF:-1 tvg-id="TVPeruNoticias.pe@SD" tvg-logo="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d9/TV_Per%C3%BA_Noticias_-_2019_logo.png/960px-TV_Per%C3%BA_Noticias_-_2019_logo.png" group-title="News",TV Peru Noticias (1080p)
http://190.93.224.42/TV-PERU-NOTICIAS/index.m3u8
#EXTINF:-1 tvg-id="705" tvg-name="TV Peru Noticias" tvg-logo="https://cdn.m3u.cl/logo/705_TV_Peru_Noticias.png", TV Peru Noticias | PE
https://cdnhd.iblups.com/hls/902c1a0395264f269f1160efa00660e4.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="TVPeruanisima.pe@SD" tvg-logo="https://i.imgur.com/m50qAVY.png" group-title="Music",TV Peruanisima (720p)
https://k20.usastreams.com:8081/tvperuanisima/index.m3u8
#EXTINF:-1 tvg-id="TVPlusChincha.pe@SD" tvg-logo="https://i.imgur.com/bjQpQsu.jpeg" group-title="General",TV+ Chincha (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/tvchincha/playlist.m3u8
#EXTINF:-1 tvg-id="TVenLinea.pe@SD" tvg-logo="https://i.imgur.com/txV8lw0.png" group-title="General",TVenLinea (1080p)
https://srt.tvdatta.com/tvenlinea/index.m3u8
#EXTINF:-1 tvg-id="1091" tvg-name="TVPE" tvg-logo="https://cdn.m3u.cl/logo/1091_TVPE.png", TVPE | PE
https://cdnhd.iblups.com/hls/777b4d4cc0984575a7d14f6ee57dbcaf.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="686" tvg-name="TVperuanisima" tvg-logo="https://cdn.m3u.cl/logo/686_TVperuanisima.png", TVperuanisima | PE
https://k20.usastreams.com:8081/tvperuanisima/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="TVUNO.pe@SD" tvg-logo="https://i.imgur.com/dht94P4.png" group-title="Music",TVUNO (720p)
https://streamtv.onliv3.online:1936/karibtv/karibtv/playlist.m3u8
#EXTINF:-1 tvg-id="UcayalinadeTelevision.pe@SD" tvg-logo="https://i.imgur.com/jBhfcqy.png" group-title="Undefined",Ucayalina de Television (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/utv/playlist.m3u8
#EXTINF:-1 tvg-id="UchuTV.pe@SD" tvg-logo="https://i.imgur.com/xYGkUR9.png" group-title="General",Uchu TV (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/ucayalitvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="UnicentroTV.pe@SD" tvg-logo="https://i.imgur.com/lhewC3K.png" group-title="General",Unicentro TV (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/unicentroSRT/playlist.m3u8
#EXTINF:-1 tvg-id="Unitel.pe@SD" tvg-logo="https://i.imgur.com/F0PWpyc.png" group-title="General",Unitel (Peru) (1080p)
https://play.agenciastreaming.com:8081/uniteltv/index.m3u8
#EXTINF:-1 tvg-id="UniversitariaTV.pe@SD" tvg-logo="https://i.imgur.com/OcDimHX.png" group-title="Undefined",Universitaria TV (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/uraniotv/playlist.m3u8
#EXTINF:-1 tvg-id="UranioTV.pe@SD" tvg-logo="https://i.imgur.com/EeNpfIr.png" group-title="Undefined",Uranio TV (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/uranioyurimaguasSRT/playlist.m3u8
#EXTINF:-1 tvg-id="1094" tvg-name="Uranio TV" tvg-logo="https://cdn.m3u.cl/logo/1094_Uranio_TV.png", Uranio TV | PE
https://live.obslivestream.com/uraniotv/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="USMPTV.pe@SD" tvg-logo="https://i.imgur.com/9cmIv3q.png" group-title="Culture",USMPTV (720p)
http://187.102.210.46/USMP/index.m3u8
#EXTINF:-1 tvg-id="1191" tvg-name="Via Alto Mayo TV" tvg-logo="https://cdn.m3u.cl/logo/1191_Via_Alto_Mayo_TV.png", Via Alto Mayo TV | PE
https://live.obslivestream.com/viaaltomayomux/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="ViaAltomayo.pe@SD" tvg-logo="https://i.imgur.com/aJ4XExn.png" group-title="General",Via Altomayo (720p) [Not 24/7]
https://live.obslivestream.com/viaaltomayomux/index.m3u8
#EXTINF:-1 tvg-id="ViaTelevision.pe@SD" tvg-logo="https://i.imgur.com/FiptbGQ.png" group-title="General",Via Television (720p) [Not 24/7]
https://videoserver.tmcreativos.com:19360/cablerasvia/cablerasvia.m3u8
#EXTINF:-1 tvg-id="703" tvg-name="Via Television Tarapoto" tvg-logo="https://cdn.m3u.cl/logo/703_Via_Television_Tarapoto.png", Via Television Tarapoto | PE
https://videoserver.tmcreativos.com:19360/cablerasvia/cablerasvia.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="Vision.pe@SD" tvg-logo="https://i.imgur.com/gyuoNZk.png" group-title="Religious",Vision (Peru) (480p)
https://lbgo.bozztv.com/ssh101/ssh101/visiononline/playlist.m3u8
#EXTINF:-1 tvg-id="876" tvg-name="Vision Musica TV" tvg-logo="https://cdn.m3u.cl/logo/876_Vision_Musica_TV.png", Vision Musica TV | PE
https://videoserver.tmcreativos.com:19360/visionmusica/visionmusica.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="VisionSur.pe@SD" tvg-logo="https://i.imgur.com/AbZYWEQ.png" group-title="General",Vision Sur (720p)
https://live-evg8.tv360.bitel.com.pe/bitel/visionsur/playlist.m3u8
#EXTINF:-1 tvg-id="1490" tvg-name="Vision Sur Television" tvg-logo="https://cdn.m3u.cl/logo/1490_Vision_Sur_Television.png", Vision Sur Television | PE
https://ott1.hdlatam.tv/live_abr/webtvVisionSur/playlist.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="VivaTVYurimaguas.pe@SD" tvg-logo="https://i.imgur.com/GkSoxhx.png" group-title="Undefined",Viva TV Yurimaguas (720p)
https://live.obslivestream.com/vivatv/index.m3u8
#EXTINF:-1 tvg-id="VivaTV.pe@SD" tvg-logo="https://i.imgur.com/m44jTZK.png" http-user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" group-title="General",VivaTV (1080p)
#EXTVLCOPT:http-user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/vivatv.stream/playlist.m3u8
#EXTINF:-1 tvg-id="VNP.pe@SD" tvg-logo="https://i.imgur.com/TOnknEW.png" group-title="News",VNP
https://videoserver.tmcreativos.com:19360/visionnoticias/visionnoticias.m3u8
#EXTINF:-1 tvg-id="VRCanal21Nasca.pe@SD" tvg-logo="https://i.imgur.com/rxzz6kD.jpeg" group-title="General",VR Canal 21 Nasca (720p)
https://live-evg10.tv360.bitel.com.pe/bitel/nasca/playlist.m3u8
#EXTINF:-1 tvg-id="1183" tvg-name="VZ TV" tvg-logo="https://cdn.m3u.cl/logo/1183_VZ_TV.png", VZ TV | PE
https://live.obslivestream.com/vztv/index.m3u8?PlaylistM3UCL
#EXTINF:-1 tvg-id="WayraTV.pe@SD" tvg-logo="https://i.imgur.com/eQLRVFp.png" group-title="General",Wayra TV (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/wayratv/playlist.m3u8
#EXTINF:-1 tvg-id="WEST.pe@SD" tvg-logo="https://aptcperu.org/wp-content/uploads/2026/04/logoMesa-de-trabajo-1-copia-5.png" group-title="Entertainment",WEST (1080p)
https://cablered.iptvperu.tv:1936/cablered/west/playlist.m3u8
#EXTINF:-1 tvg-id="WillaxTV.pe@SD" tvg-logo="https://i.imgur.com/bZnDDPH.png" group-title="General",Willax TV (1080p)
http://190.93.224.42/WILLAX/index.m3u8
#EXTINF:-1 tvg-id="Wtv.pe@SD" tvg-logo="https://i.imgur.com/V10iVeh.png" group-title="General",Wtv (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/ncanalSRT/playlist.m3u8
#EXTINF:-1 tvg-id="YurimaguasTV.pe@SD" tvg-logo="https://i.imgur.com/VWC6Pt1.png" group-title="General",Yurimaguas TV (720p)
https://live-evg7.tv360.bitel.com.pe/bitel/yurimaguastvSRT/playlist.m3u8
#EXTINF:-1 tvg-id="Yurivision.pe@SD" tvg-logo="https://i.imgur.com/VhWuaER.jpeg" group-title="General",Yurivision (720p)
https://live-evg5.tv360.bitel.com.pe/bitel/yurivisionSRT/playlist.m3u8
#EXTINF:-1 tvg-id="ZeleveTV.pe@SD" tvg-logo="https://i.imgur.com/CGWaunJ.jpeg" group-title="General",Zeleve TV (720p)
https://live-evg11.tv360.bitel.com.pe/bitel/zeleve/playlist.m3u8
</script>

<script>
(function () {
  "use strict";

  /* ============================================================
     1. CONFIGURACIÓN Y FUENTES DE DATOS (SOLO PERÚ)
     ============================================================ */
  // Lista única: Perú (m3u.cl) con espejo de respaldo (iptv-org).
  const SOURCES = [
    { code: 'PE', name: 'Perú', flag: '🇵🇪', url: 'https://www.m3u.cl/lista/PE.m3u', mirror: 'https://iptv-org.github.io/iptv/countries/pe.m3u' }
  ];
  const COUNTRY = Object.fromEntries(SOURCES.map(s => [s.code, s]));

  // Todas las fuentes se intentan EN PARALELO y gana la primera que responde.
  // Así una fuente bloqueada nunca retrasa a las demás (antes eran 12 s por intento).
  const PROXIES = [
    { name: 'espejo',      build: (u, s) => s.mirror },
    { name: 'directa',     build: u => u },
    { name: 'corsproxy',   build: u => 'https://corsproxy.io/?url=' + encodeURIComponent(u) },
    { name: 'allorigins',  build: u => 'https://api.allorigins.win/raw?url=' + encodeURIComponent(u) },
    { name: 'codetabs',    build: u => 'https://api.codetabs.com/v1/proxy?quest=' + encodeURIComponent(u) }
  ];
  const FETCH_TIMEOUT = 7000;

  const LS_FAVS = 'iptv-latam:favorites:v1';
  const LS_RECENT = 'iptv-latam:recent:v1';
  const LS_VOL = 'iptv-latam:volume:v1';
  const LS_LAST = 'iptv-latam:last-channel:v1';
  const LS_CACHE = 'iptv-peru:playlists-cache:v1';
  const LS_CUSTOM_CHANNELS = 'iptv-peru:custom-channels:v1';
  const LS_DELETED_CHANNELS = 'iptv-peru:deleted-channels:v1';
  try { localStorage.removeItem('iptv-latam:playlists-cache:v1'); } catch (_) {}
  const CACHE_TTL = 6 * 60 * 60 * 1000; // 6 horas
  const MAX_RECENT = 30;
  const PAGE_SIZE = 80; // renderizado incremental: evita crear cientos de tarjetas de golpe
  let renderLimit = PAGE_SIZE;

  /* ============================================================
     2. ESTADO Y UTILIDADES
     ============================================================ */
  const state = {
    channels: [],
    country: 'PE',
    category: 'ALL',
    tab: 'all',
    query: '',
    current: null,
    loading: false,
    sourcesOk: 0,
    mirrors: 0,
    fromCache: 0
  };

  const $ = id => document.getElementById(id);
  const esc = s => String(s ?? '').replace(/[&<>"']/g, c => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c]));
  const initials = name => String(name || '?').replace(/[^\p{L}\p{N} ]/gu, '').trim().split(/\s+/).slice(0, 2).map(w => w[0] || '').join('').toUpperCase() || 'TV';

  function loadJSON(key, fallback) {
    try { const v = JSON.parse(localStorage.getItem(key)); return v ?? fallback; }
    catch (_) { return fallback; }
  }
  function saveJSON(key, value) {
    try { localStorage.setItem(key, JSON.stringify(value)); } catch (_) { /* modo privado */ }
  }

  let favorites = loadJSON(LS_FAVS, []);
  let recents = loadJSON(LS_RECENT, []);
  const DEFAULT_DELETED_URLS = [
  "https://live-evg10.tv360.bitel.com.pe/bitel/abctv/playlist.m3u8",
  "https://live-evg10.tv360.bitel.com.pe/bitel/urbanatv/playlist.m3u8",
  "https://servilive.com:3199/live/amazoniatvlive.m3u8",
  "https://live-evg8.tv360.bitel.com.pe/bitel/andinatvSRT/playlist.m3u8",
  "https://5949aa132c8fb.streamlock.net:1963/iptvantares/liveantarestv/playlist.m3u8",
  "https://5c3fb01839654.streamlock.net:1963/iptvantares/liveantarestv/playlist.m3u8?PlaylistM3UCL",
  "http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/antenauno.stream/playlist.m3u8",
  "https://live-evg8.tv360.bitel.com.pe/bitel/arpegioSRT/playlist.m3u8",
  "https://live-evg8.tv360.bitel.com.pe/bitel/atlantisSRT/playlist.m3u8",
  "https://d3r5ioyjqcy69v.cloudfront.net/index.m3u8?PlaylistM3UCL",
  "https://d19e55ehz2il4i.cloudfront.net/index.m3u8?PlaylistM3UCL",
  "https://d1svme839xkbsn.cloudfront.net/index.m3u8?PlaylistM3UCL",
  "https://sc1.wasidata.com/Bhtv/index.fmp4.m3u8?PlaylistM3UCL",
  "https://live-evg10.tv360.bitel.com.pe/bitel/canal25ayacucho/playlist.m3u8",
  "http://187.102.210.46/IPE/index.m3u8",
  "http://187.102.208.209:8000/play/a0az/index.m3u8",
  "https://s1.tvdatta.com:3978/live/diasporatvlive.m3u8",
  "https://stmv1.voxhdnet.com/dmjsurtv/dmjsurtv/playlist.m3u8?PlaylistM3UCL",
  "https://live20.bozztv.com/giatv/giatv-doblec1/doblec1/playlist.m3u8",
  "https://live-evg7.tv360.bitel.com.pe/bitel/sabormixtvSRT/playlist.m3u8",
  "https://live-evg11.tv360.bitel.com.pe/bitel/ecotv/playlist.m3u8",
  "https://edgectc.com/CTC_SAN_RAFAEL/index.m3u8",
  "https://vivatv.juanjuiserver.com:3349/live/frecuenciaamazonicalive.m3u8",
  "http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/fuegotv.stream/playlist.m3u8",
  "https://stream.neux.site/memfs/e3f59eec-6b93-45e1-a2ab-c82381c2a9ec.m3u8",
  "https://live.obslivestream.com/geniostvmux/index.m3u8?PlaylistM3UCL",
  "https://video.misistemareseller.com:443/corporationhc/corporationhc/playlist.m3u8?PlaylistM3UCL",
  "http://187.102.210.46/JN19/index.m3u8",
  "https://servilive.com:3028/live/jntv19live.m3u8?PlaylistM3UCL",
  "https://lbgo.bozztv.com/ssh101/ssh101/jnetv/playlist.m3u8",
  "http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/juntos_tv.stream/playlist.m3u8",
  "http://187.102.210.46/KARIBENA/index.m3u8",
  "https://s3.tvdatta.com:3700/live/jjrtvlive.m3u8",
  "http://45.4.1.201:8000/play/a1ai/index.m3u8",
  "https://redirector.rudo.video/hls-video/plus226/latina4/latina4.smil/playlist.m3u8",
  "https://ontvcable.streamottmas.workers.dev/ontvcable.m3u8",
  "https://mc.servidor.stream:19360/megatv/megatv.m3u8?PlaylistM3UCL",
  "https://video1.earthcam.com/myearthcam/075ff02f78c35af55564cf3af3b3f750.flv/playlist.m3u8",
  "http://195.178.110.110/movistar/index.m3u8",
  "http://45.4.1.201:8000/play/a19y/index.m3u8",
  "http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/nacionaltv_peru.stream/playlist.m3u8",
  "http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/DigitalTV/playlist.m3u8",
  "https://live.obslivestream.com/oktv/video.m3u8?token=Bk-DXAXulO1OV0&PlaylistM3UCL",
  "https://v4.tustreaming.cl:443/odtv2/index.m3u8?PlaylistM3UCL",
  "https://v4.tustreaming.cl:443/ondadigitaltv/index.m3u8?PlaylistM3UCL",
  "https://live-evg5.tv360.bitel.com.pe/bitel/onda/playlist.m3u8",
  "https://live.obslivestream.com/planetatv/index.m3u8?PlaylistM3UCL",
  "https://183.bozztv.com/ssh101/ssh101/pratteltvperu/playlist.m3u8",
  "https://servilive.com:3414/hybrid/play.m3u8",
  "https://stream.mediacorp.pe/srt/3/playlist.m3u8",
  "https://live-evg7.tv360.bitel.com.pe/bitel/radiounotv/playlist.m3u8",
  "https://amixtv.com:8081/rikutv/index.m3u8",
  "https://live-evg7.tv360.bitel.com.pe/bitel/rtp/playlist.m3u8",
  "https://eu1.servers10.com:8081/8156/index.m3u8?PlaylistM3UCL",
  "https://infosmartpe.store:5443/LiveApp/streams/3zc2BXi5SEys2310139330128741.m3u8?PlaylistM3UCL",
  "https://stmv1.voxhdnet.com/solartv/solartv/playlist.m3u8?PlaylistM3UCL",
  "http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/telelima.stream/playlist.m3u8",
  "https://live-evg5.tv360.bitel.com.pe/bitel/telesystemSRT/playlist.m3u8",
  "https://5cefcbf58ba2e.streamlock.net:543/webtltv/webtltv.stream/playlist.m3u8?wowzatokenstarttime=1752707094&wowzatokenendtime=1752708094&wowzatokenhash=ehhxNAzcNA0F16D7rrGOSaTgY2M5_O38gPj7Ir3Lx2U=&PlaylistM3UCL",
  "https://servilive.com:3531/stream/play.m3u8?PlaylistM3UCL",
  "https://live-evg11.tv360.bitel.com.pe/bitel/turbomixSRT/playlist.m3u8",
  "https://5790d294af2dc.streamlock.net:443/8134/8134/playlist.m3u8?PlaylistM3UCL",
  "https://lbgo.bozztv.com/ssh101/ssh101/kalientecusco/playlist.m3u8",
  "https://live-evg11.tv360.bitel.com.pe/bitel/tvmarSRT/playlist.m3u8",
  "https://cdnhd.iblups.com/hls/902c1a0395264f269f1160efa00660e4.m3u8?PlaylistM3UCL",
  "https://cdnhd.iblups.com/hls/777b4d4cc0984575a7d14f6ee57dbcaf.m3u8?PlaylistM3UCL",
  "https://live-evg11.tv360.bitel.com.pe/bitel/ucayalitvSRT/playlist.m3u8",
  "https://lbgo.bozztv.com/ssh101/ssh101/visiononline/playlist.m3u8",
  "http://bantel-cdn1.iptvperu.tv:1935/btnscrtn/vivatv.stream/playlist.m3u8",
  "https://live.obslivestream.com/vztv/index.m3u8?PlaylistM3UCL",
  "https://live-evg11.tv360.bitel.com.pe/bitel/zeleve/playlist.m3u8"
];
  let deletedUrls = new Set(loadJSON(LS_DELETED_CHANNELS, DEFAULT_DELETED_URLS));
  const masterChannels = new Map();
  if (!Array.isArray(favorites)) favorites = [];
  if (!Array.isArray(recents)) recents = [];
  const favMap = new Map(favorites.map(c => [c.url, c]));

  let toastTimer;
  function toast(msg, kind) {
    const host = $('toast-host');
    const colors = { ok: 'border-emerald-500/40 bg-emerald-950/90 text-emerald-200', warn: 'border-amber-500/40 bg-amber-950/90 text-amber-200', err: 'border-red-500/40 bg-red-950/90 text-red-200', info: 'border-blue-500/40 bg-blue-950/90 text-blue-200' };
    host.innerHTML = '<div class="rounded-xl border px-4 py-2.5 text-xs font-medium shadow-2xl glass ' + (colors[kind] || colors.info) + '">' + esc(msg) + '</div>';
    clearTimeout(toastTimer);
    toastTimer = setTimeout(() => { host.innerHTML = ''; }, 3500);
  }

  /* ============================================================
     3. DESCARGA + PARSEO DE M3U (5 listas unificadas)
     ============================================================ */
  function parseExtinf(line, countryCode) {
    const comma = line.lastIndexOf(',');
    const name = (comma >= 0 ? line.slice(comma + 1) : 'Canal sin nombre').trim() || 'Canal sin nombre';
    const attr = key => {
      const m = line.match(new RegExp(key + '="([^"]*)"', 'i'));
      return m ? m[1].trim() : '';
    };
    return {
      name,
      logo: attr('tvg-logo'),
      category: attr('group-title') || 'General',
      id: attr('tvg-id'),
      country: countryCode,
      url: ''
    };
  }

  function parseM3U(text, countryCode) {
    const lines = text.replace(/^\uFEFF/, '').split(/\r?\n/);
    const out = [];
    let pending = null;
    for (let raw of lines) {
      const line = raw.trim();
      if (!line) continue;
      if (/^#EXTINF/i.test(line)) {
        pending = parseExtinf(line, countryCode);
      } else if (line.startsWith('#EXTGRP:') && pending) {
        pending.category = line.slice(8).trim() || pending.category;
      } else if (!line.startsWith('#') && pending) {
        pending.url = line;
        if (/^https?:\/\//i.test(pending.url)) {
          masterChannels.set(pending.url, pending);
          out.push(pending);
        }
        pending = null;
      }
    }
    return out;
  }

  // Lanza todas las variantes a la vez; la primera respuesta válida gana y
  // cancela el resto. Evita esperar a proxies caídos de forma secuencial.
  function fetchWithFallback(source) {
    const controllers = [];
    const attempts = PROXIES.map(proxy => new Promise((resolve, reject) => {
      const ctrl = new AbortController();
      controllers.push(ctrl);
      const timer = setTimeout(() => { ctrl.abort(); reject(new Error('Tiempo de espera')); }, FETCH_TIMEOUT);
      fetch(proxy.build(source.url, source), {
        signal: ctrl.signal, cache: 'no-store', headers: { 'Accept': 'text/plain,*/*' }
      })
        .then(res => { if (!res.ok) throw new Error('HTTP ' + res.status); return res.text(); })
        .then(text => {
          if (!/#EXTINF/i.test(text)) throw new Error('Contenido sin canales');
          clearTimeout(timer);
          resolve({ text, proxy: proxy.name });
        })
        .catch(err => { clearTimeout(timer); reject(err); });
    }));
    // Evita warnings de promesas rechazadas sin manejar.
    attempts.forEach(p => p.catch(() => {}));
    return Promise.any(attempts).then(winner => {
      controllers.forEach(c => { try { c.abort(); } catch (_) {} });
      return winner;
    }).catch(() => null);
  }

  function readPlaylistCache() {
    const cache = loadJSON(LS_CACHE, null);
    if (!cache || !cache.time || Date.now() - cache.time > CACHE_TTL || !cache.items) return null;
    return cache.items;
  }

  async function loadAll(force) {
    if (state.loading) return;

    // Si existen canales personalizados guardados y no es una recarga forzada, usarlos directamente
    if (!force) {
      // Inicializar masterChannels con los canales embebidos
      const embEl = $('embedded-pe-m3u');
      if (embEl && embEl.textContent) {
        parseM3U(embEl.textContent, 'PE');
      }

      const custom = loadJSON(LS_CUSTOM_CHANNELS, null);
      if (Array.isArray(custom)) {
        custom.forEach(c => masterChannels.set(c.url, c));
        state.channels = custom.filter(c => !deletedUrls.has(c.url));
        state.channels.sort((a, b) => a.name.localeCompare(b.name, 'es', { sensitivity: 'base', numeric: true }));
        state.sourcesOk = 1;
        state.category = 'ALL';
        state.loading = false;
        if ($('btn-reload')) $('btn-reload').disabled = false;
        $('loading').classList.add('hidden');
        updateStatus();
        renderAll();
        const last = loadJSON(LS_LAST, null);
        const restored = last && state.channels.find(c => c.url === last.url);
        if (restored) playChannel(restored, { silent: true });
        else if (!state.current && state.channels.length) playChannel(state.channels[0], { silent: true });
        toast(state.channels.length + ' canales de Perú cargados', 'ok');
        return;
      }
    }

    state.loading = true;
    if ($('btn-reload')) $('btn-reload').disabled = true;
    $('loading').classList.remove('hidden');
    $('loading-bar').style.width = '8%';
    $('loading-progress').textContent = 'Descargando lista de Perú…';

    const cache = force ? null : readPlaylistCache();
    let done = 0;
    const results = await Promise.all(SOURCES.map(async source => {
      const cached = cache && cache[source.code];
      let result;
      if (cached && cached.text && /#EXTINF/i.test(cached.text)) {
        result = { text: cached.text, proxy: 'caché' };
      } else {
        result = await fetchWithFallback(source);
      }
      done++;
      $('loading-bar').style.width = '100%';
      $('loading-progress').textContent = 'Lista de Perú descargada';
      return { source, result };
    }));

    // Guardamos el texto original de cada lista para que la próxima apertura sea instantánea.
    const items = {};
    results.forEach(({ source, result }) => {
      if (result && result.text) items[source.code] = { text: result.text, time: Date.now() };
    });
    if (Object.keys(items).length) saveJSON(LS_CACHE, { time: Date.now(), items });

    // Fusión en una sola lista. El Set garantiza URLs únicas (prevalece la lista personalizada).
    const seen = new Set();
    const merged = [];

    // 1. Canales de la lista personalizada de Perú
    const embeddedEl = $('embedded-pe-m3u');
    if (embeddedEl && embeddedEl.textContent) {
      parseM3U(embeddedEl.textContent, 'PE').forEach(ch => {
        if (!seen.has(ch.url) && !deletedUrls.has(ch.url)) { seen.add(ch.url); merged.push(ch); }
      });
    }

    // 2. Canales remotos de respaldo (solo si la lista local embebida estuviese vacía)
    if (!merged.length) {
      results.forEach(({ source, result }) => {
        if (!result) return;
        parseM3U(result.text, source.code).forEach(ch => {
          if (!seen.has(ch.url) && !deletedUrls.has(ch.url)) { seen.add(ch.url); merged.push(ch); }
        });
      });
    }

    // Orden alfabético (A-Z) para todos los canales
    merged.sort((a, b) => a.name.localeCompare(b.name, 'es', { sensitivity: 'base', numeric: true }));

    state.channels = merged;
    saveJSON(LS_CUSTOM_CHANNELS, merged);
    state.sourcesOk = results.filter(r => !!r.result).length;
    state.mirrors = results.filter(r => r.result && r.result.proxy === 'espejo').length;
    state.fromCache = results.filter(r => r.result && r.result.proxy === 'caché').length;
    state.category = 'ALL';
    state.loading = false;
    if ($('btn-reload')) $('btn-reload').disabled = false;
    $('loading').classList.add('hidden');

    updateStatus();
    renderAll();

    if (!merged.length) {
      if ($('sidebar-status')) $('sidebar-status').textContent = 'No se pudo descargar la lista de Perú. Revisa tu conexión.';
      $('status-text').textContent = 'Sin canales disponibles';
      $('status-dot').className = 'h-2 w-2 rounded-full bg-red-500';
      showOffline('No se pudo descargar la lista de Perú (PE.m3u). Puede que el sitio esté temporalmente inaccesible.');
      toast('Error al descargar la lista de Perú. Prueba «Recargar».', 'err');
    } else {
      const last = loadJSON(LS_LAST, null);
      const restored = last && merged.find(c => c.url === last.url);
      if (restored) playChannel(restored, { silent: true });
      else if (!state.current && merged.length) playChannel(merged[0], { silent: true });
      if (state.fromCache) {
        toast(merged.length + ' canales de Perú listos · cargados desde caché', 'ok');
      } else if (state.mirrors) {
        toast(merged.length + ' canales de Perú listos · vía espejo de respaldo', 'info');
      } else {
        toast(merged.length + ' canales de Perú listos (m3u.cl/lista/PE.m3u)', 'ok');
      }
    }
  }

  /* ============================================================
     4. FILTRADO Y RENDERIZADO DE LA INTERFAZ
     ============================================================ */
  function baseList() {
    let list = state.channels;
    if (state.tab === 'fav') list = list.filter(c => favMap.has(c.url));
    if (state.tab === 'recent') {
      list = recents.map(r => state.channels.find(c => c.url === r.url) || r);
    }
    return list;
  }

  function visibleList() {
    let list = baseList();
    if (state.category !== 'ALL') list = list.filter(c => c.category === state.category);
    const q = state.query.trim().toLowerCase();
    if (q) list = list.filter(c => c.name.toLowerCase().includes(q) || c.category.toLowerCase().includes(q));
    return list;
  }

  function updateStatus() {
    const via = state.fromCache ? ' · desde caché' : (state.mirrors ? ' · vía espejo' : '');
    const statusTextEl = $('status-text');
    if (statusTextEl) statusTextEl.textContent = state.channels.length.toLocaleString('es') + ' canales de Perú' + via;
    const statusDotEl = $('status-dot');
    if (statusDotEl) statusDotEl.className = 'h-2 w-2 rounded-full ' + (state.sourcesOk >= 1 ? 'bg-emerald-400' : 'bg-amber-400');
    const sidebarStatusEl = $('sidebar-status');
    if (sidebarStatusEl) sidebarStatusEl.textContent = state.channels.length + ' canales de Perú disponibles' + via;

    // Estadísticas de cabecera TVSam Click
    const alive = state.channels.length;
    const delCount = deletedUrls.size;
    const favCount = favMap.size;
    const statsEl = $('stats');
    if (statsEl) {
      statsEl.innerHTML = `<b>${alive}</b> canales · <b>${delCount}</b> eliminados · <b>${favCount}</b> favoritos`;
    }
    const trashCountEl = $('trash-count');
    if (trashCountEl) {
      trashCountEl.textContent = delCount;
    }
  }

  function chip(label, value, active, extra) {
    return '<button data-chip="' + esc(value) + '" class="shrink-0 rounded-full border px-3 py-1.5 text-[11px] font-semibold transition ' +
      (active ? 'border-red-500/60 bg-red-600 text-white shadow-lg shadow-red-900/40' : 'border-white/10 bg-white/5 text-slate-400 hover:border-white/25 hover:text-white') +
      '">' + label + (extra ? ' <span class="opacity-60">' + extra + '</span>' : '') + '</button>';
  }

  function renderTabs() {
    document.querySelectorAll('#tabs button').forEach(btn => {
      const active = btn.dataset.tab === state.tab;
      btn.className = 'flex-1 rounded-lg px-2 py-1.5 text-[11px] font-bold transition ' +
        (active ? 'bg-red-600 text-white shadow-lg shadow-red-900/40' : 'text-slate-400 hover:bg-white/5 hover:text-white');
    });
    if ($('cnt-all')) $('cnt-all').textContent = state.channels.length;
    if ($('cnt-fav')) $('cnt-fav').textContent = favMap.size;
    if ($('cnt-recent')) $('cnt-recent').textContent = recents.length;
  }

  function renderChips() {
    const el = $('country-chips');
    if (el) { el.innerHTML = ''; el.classList.add('hidden'); }

    const catCounts = {};
    baseList().forEach(c => { catCounts[c.category] = (catCounts[c.category] || 0) + 1; });
    const cats = Object.keys(catCounts).sort((a, b) => catCounts[b] - catCounts[a]).slice(0, 25);
    if (state.category !== 'ALL' && !cats.includes(state.category)) cats.unshift(state.category);
    let catHtml = chip('Todas', 'ALL', state.category === 'ALL', state.channels.length);
    cats.forEach(c => catHtml += chip(esc(c), 'cat:' + c, state.category === c, catCounts[c]));
    const catEl = $('category-chips');
    if (catEl) catEl.innerHTML = catHtml;
  }

  function logoHTML(ch, sizeClass) {
    const fallback = '<span>' + esc(initials(ch.name)) + '</span>';
    if (!ch.logo) return fallback;
    return fallback +
      '<img src="' + esc(ch.logo) + '" alt="" loading="lazy" class="absolute inset-0 h-full w-full object-contain p-1" ' +
      'onerror="this.style.display=\'none\'" />';
  }

  function cardHTML(ch) {
    const active = state.current && state.current.url === ch.url;
    const c = COUNTRY[ch.country] || { flag: '🇵🇪', name: 'Perú' };
    const isFav = favMap.has(ch.url);
    return '<div data-url="' + esc(ch.url) + '" class="group flex cursor-pointer items-center gap-2 rounded-xl border p-2 transition duration-150 ' +
      (active ? 'border-red-500/60 bg-red-600/15 glow' : 'border-transparent bg-white/[0.03] hover:border-white/15 hover:bg-white/[0.07]') + '">' +
      '<div class="relative grid h-11 w-16 shrink-0 place-items-center overflow-hidden rounded-lg border border-white/10 bg-slate-900 text-[10px] font-extrabold text-slate-600">' + logoHTML(ch) + '</div>' +
      '<div class="min-w-0 flex-1">' +
        '<p class="truncate text-[12.5px] font-semibold ' + (active ? 'text-white' : 'text-slate-200') + '">' + esc(ch.name) + '</p>' +
        '<p class="mt-0.5 truncate text-[10.5px] text-slate-500">' + c.flag + ' ' + esc(c.name) + ' · ' + esc(ch.category) + '</p>' +
      '</div>' +
      '<div class="flex items-center shrink-0 gap-0.5">' +
        '<button type="button" data-edit="' + esc(ch.url) + '" title="Editar canal" class="rounded-lg p-1.5 text-slate-400 opacity-60 transition hover:bg-white/10 hover:text-blue-400 hover:opacity-100 group-hover:opacity-100">' +
          '<svg viewBox="0 0 24 24" class="h-3.5 w-3.5" fill="none" stroke="currentColor" stroke-width="2"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/></svg>' +
        '</button>' +
        '<button type="button" data-delete="' + esc(ch.url) + '" title="Eliminar canal" class="rounded-lg p-1.5 text-slate-400 opacity-60 transition hover:bg-white/10 hover:text-red-400 hover:opacity-100 group-hover:opacity-100">' +
          '<svg viewBox="0 0 24 24" class="h-3.5 w-3.5" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 6h18M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/></svg>' +
        '</button>' +
        '<button type="button" data-fav="' + esc(ch.url) + '" title="Favorito" class="rounded-lg p-1.5 transition ' + (isFav ? 'text-amber-400' : 'text-slate-600 hover:bg-white/10 hover:text-amber-400') + '">' +
          '<svg viewBox="0 0 24 24" class="h-3.5 w-3.5" fill="currentColor"><path d="m12 3 2.8 5.7 6.2.9-4.5 4.4 1.1 6.2L12 17.3 6.4 20.2l1.1-6.2L3 9.6l6.2-.9L12 3z"/></svg>' +
        '</button>' +
      '</div>' +
    '</div>';
  }

  function renderChannels() { renderLimit = PAGE_SIZE; drawChannels(); }

  function drawChannels() {
    const list = visibleList();
    if ($('list-count')) $('list-count').textContent = list.length;
    const cl = $('channel-list');
    if (!cl) return;
    if (!list.length) {
      const msg = state.tab === 'fav' ? 'Aún no tienes favoritos. Pulsa la estrella de un canal.' :
                  state.tab === 'recent' ? 'Todavía no has visto ningún canal.' :
                  'No hay canales con estos filtros. Prueba con otra búsqueda.';
      cl.innerHTML = '<div class="px-4 py-14 text-center"><div class="mx-auto grid h-11 w-11 place-items-center rounded-xl bg-white/5 text-slate-600"><svg viewBox="0 0 24 24" class="h-5 w-5" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="7"/><path d="m20 20-3.6-3.6"/></svg></div><p class="mt-3 text-xs leading-relaxed text-slate-500">' + esc(msg) + '</p></div>';
      return;
    }
    const shown = list.slice(0, renderLimit);
    let html = shown.map(cardHTML).join('');
    if (list.length > shown.length) {
      html += '<button data-more="1" class="mt-2 w-full rounded-xl border border-blue-500/30 bg-blue-500/10 py-2.5 text-[11px] font-bold text-blue-300 transition hover:bg-blue-500/20">Mostrar más canales (' + (list.length - shown.length) + ' restantes)</button>';
    }
    cl.innerHTML = html;
  }

  function renderAll() {
    renderTabs(); renderChips(); renderChannels(); updateStatus();
    if (typeof isOsdOpen === 'function' && isOsdOpen()) renderOsdChannels();
  }

  /* ============================================================
     5. REPRODUCTOR (hls.js / nativo / controles)
     ============================================================ */
  const video = $('video');
  let hls = null;
  let recoveries = 0;
  let streamWatchdog = null;
  video.volume = Math.min(1, Math.max(0, Number(loadJSON(LS_VOL, 0.8))));

  function clearWatchdog() { if (streamWatchdog) { clearTimeout(streamWatchdog); streamWatchdog = null; } }

  function overlay(which) {
    ['loading', 'play', 'offline'].forEach(k => $('ov-' + k).classList.add('hidden'));
    if (which) $('ov-' + which).classList.remove('hidden');
  }
  function showOffline(msg) {
    clearWatchdog();
    $('offline-detail').textContent = msg || 'El canal puede estar temporalmente fuera de línea o bloquear la reproducción desde tu red.';
    $('live-badge').classList.add('opacity-0');
    overlay('offline');
  }
  function syncPlayIcon() {
    $('icon-play').classList.toggle('hidden', !video.paused);
    $('icon-pause').classList.toggle('hidden', video.paused);
  }
  function syncVolumeIcon() {
    const silent = video.muted || video.volume === 0;
    $('icon-vol').classList.toggle('hidden', silent);
    $('icon-muted').classList.toggle('hidden', !silent);
    $('volume').value = silent ? 0 : video.volume;
  }
  function teardown() {
    clearWatchdog();
    if (hls) { try { hls.destroy(); } catch (_) {} hls = null; }
    video.pause();
    video.removeEventListener('error', onDirectError);
    video.removeAttribute('src');
    try { video.load(); } catch (_) {}
  }

  function pushRecent(ch) {
    recents = [ch, ...recents.filter(r => r.url !== ch.url)].slice(0, MAX_RECENT);
    saveJSON(LS_RECENT, recents);
  }

  function updateNowPlaying() {
    const ch = state.current;
    if (typeof isOsdOpen === 'function' && isOsdOpen()) renderOsdChannels();
    if (!ch) return;
    const c = COUNTRY[ch.country] || { flag: '🌎', name: ch.country };
    $('np-title').textContent = ch.name;
    $('np-country').textContent = c.flag + ' ' + c.name;
    $('np-category').textContent = ch.category;
    $('np-logo-fallback').textContent = initials(ch.name);
    const logo = $('np-logo').querySelector('img');
    if (logo) logo.remove();
    if (ch.logo) {
      const img = document.createElement('img');
      img.src = ch.logo; img.alt = ''; img.className = 'pointer-events-none absolute inset-0 h-full w-full object-contain p-1';
      img.onerror = () => img.remove();
      $('np-logo').appendChild(img);
    }
    const list = visibleList();
    const idx = list.findIndex(x => x.url === ch.url);
    $('np-pos').textContent = idx >= 0 ? 'Canal ' + (idx + 1) + ' de ' + list.length + ' en la vista actual' : '';
    const fav = favMap.has(ch.url);
    $('btn-fav').className = 'grid h-10 w-10 place-items-center rounded-xl border transition ' + (fav ? 'border-amber-400/50 bg-amber-400/10 text-amber-400' : 'border-white/10 bg-white/5 text-slate-400 hover:border-amber-400/50 hover:text-amber-400');
    $('watermark').textContent = ch.name;
    $('watermark').classList.remove('hidden');
    document.title = ch.name + ' · IPTV Perú';
  }

  function onDirectError() {
    showOffline('El navegador no pudo cargar este stream. Puede que el canal esté caído o bloqueado por CORS.');
  }

  function onPlayBlocked(err) {
    if (err && err.name === 'AbortError') return;
    if (err && err.name === 'NotAllowedError') { overlay('play'); return; }
    showOffline('No se pudo iniciar la reproducción. ' + (err && err.message ? err.message : ''));
  }

  function onHlsError(data) {
    if (!data.fatal) return;
    if (data.type === Hls.ErrorTypes.NETWORK_ERROR && recoveries < 1) {
      recoveries++;
      toast('Problemas de red: reconectando…', 'warn');
      hls && hls.startLoad();
      return;
    }
    if (data.type === Hls.ErrorTypes.MEDIA_ERROR && recoveries < 3) {
      recoveries++;
      hls && hls.recoverMediaError();
      return;
    }
    showOffline('El stream no responde o está fuera de línea. Código: ' + (data.details || data.type));
  }

  function playChannel(ch, opts) {
    if (!ch || !ch.url) return;
    opts = opts || {};
    state.current = ch;
    recoveries = 0;
    pushRecent(ch);
    teardown();
    updateNowPlaying();
    renderTabs(); renderChips(); renderChannels();
    saveJSON(LS_LAST, { url: ch.url, name: ch.name, logo: ch.logo, category: ch.category, country: ch.country });
    if (window.innerWidth < 1280) toggleSidebar(false);

    overlay('loading');
    // Si el stream nunca entrega video, mostramos un estado claro en vez de un spinner eterno.
    clearWatchdog();
    streamWatchdog = setTimeout(() => {
      if (video.readyState < 2) showOffline('El canal no comenzó a transmitir a tiempo. Puede estar fuera de línea o bloqueado por CORS.');
    }, 15000);
    const isHLS = /\.m3u8(?:\?|#|$)/i.test(ch.url) || /\/hls\//i.test(ch.url);

    try {
      if (isHLS && window.Hls && Hls.isSupported()) {
        hls = new Hls({ enableWorker: true, lowLatencyMode: true, capLevelToPlayerSize: true, backBufferLength: 60 });
        hls.on(Hls.Events.MEDIA_ATTACHED, () => hls && hls.loadSource(ch.url));
        hls.on(Hls.Events.MANIFEST_PARSED, () => { overlay(null); video.play().catch(onPlayBlocked); });
        hls.on(Hls.Events.ERROR, (_e, data) => onHlsError(data));
        hls.attachMedia(video);
      } else {
        // Safari (HLS nativo) o streams MP4 directos.
        video.src = ch.url;
        video.addEventListener('loadedmetadata', () => overlay(null), { once: true });
        video.addEventListener('error', onDirectError, { once: true });
        video.play().catch(onPlayBlocked);
      }
      if (opts.silent) $('live-badge').classList.remove('opacity-0');
    } catch (err) {
      showOffline('Error inesperado al iniciar el canal: ' + err.message);
    }
  }

  function step(dir) {
    const list = visibleList().length ? visibleList() : state.channels;
    if (!list.length) return;
    const i = list.findIndex(c => c.url === (state.current && state.current.url));
    const nextIndex = i < 0 ? 0 : (i + dir + list.length) % list.length;
    const next = list[nextIndex];
    playChannel(next);
    if (nextIndex >= renderLimit) { renderLimit = nextIndex + 1 + PAGE_SIZE; drawChannels(); }
    requestAnimationFrame(() => {
      const el = Array.from(document.querySelectorAll('#channel-list [data-url]')).find(n => n.dataset.url === next.url);
      if (el) el.scrollIntoView({ block: 'center', behavior: 'smooth' });
    });
  }

  /* ============================================================
     6. EVENTOS DE INTERFAZ
     ============================================================ */
  function toggleSidebar(open) {
    const sb = $('sidebar');
    if (!sb) {
      if (typeof toggleOsd === 'function') toggleOsd(open);
      return;
    }
    const next = open === undefined ? sb.style.transform !== 'translateX(0px)' : open;
    sb.style.transform = next ? 'translateX(0px)' : '';
    if ($('backdrop')) $('backdrop').classList.toggle('hidden', !next);
  }

  if ($('btn-menu')) $('btn-menu').addEventListener('click', () => {
    if (typeof toggleOsd === 'function') toggleOsd(true);
    else toggleSidebar(true);
  });
  if ($('btn-close')) $('btn-close').addEventListener('click', () => toggleSidebar(false));
  if ($('backdrop')) $('backdrop').addEventListener('click', () => toggleSidebar(false));
  if ($('btn-reload')) $('btn-reload').addEventListener('click', () => {
    toast('Actualizando canal y lista…', 'info');
    if (state.current) playChannel(state.current);
    renderAll();
    updateStatus();
  });

  if ($('tabs')) $('tabs').addEventListener('click', e => {
    const btn = e.target.closest('button[data-tab]');
    if (!btn) return;
    state.tab = btn.dataset.tab;
    renderAll();
  });

  if ($('country-chips')) $('country-chips').addEventListener('click', e => {
    const btn = e.target.closest('button[data-chip]');
    if (!btn) return;
    state.country = btn.dataset.chip;
    if (state.category !== 'ALL') state.category = 'ALL';
    renderAll();
  });
  if ($('category-chips')) $('category-chips').addEventListener('click', e => {
    const btn = e.target.closest('button[data-chip]');
    if (!btn) return;
    state.category = btn.dataset.chip.startsWith('cat:') ? btn.dataset.chip.slice(4) : 'ALL';
    renderChannels(); renderChips();
  });

  let searchTimer;
  if ($('search')) $('search').addEventListener('input', e => {
    clearTimeout(searchTimer);
    searchTimer = setTimeout(() => { state.query = e.target.value; renderChannels(); updateNowPlaying(); }, 120);
  });

  /* ============================================================
     GESTIÓN DE CANALES: AÑADIR, EDITAR, ELIMINAR, EXPORTAR
     ============================================================ */
  let channelToDelete = null;

  function saveChannels() {
    state.channels = state.channels.filter(c => !deletedUrls.has(c.url));
    state.channels.sort((a, b) => a.name.localeCompare(b.name, 'es', { sensitivity: 'base', numeric: true }));
    saveJSON(LS_CUSTOM_CHANNELS, state.channels);
    renderAll();
  }

  function updateCategorySuggestions() {
    const datalist = $('category-suggestions');
    if (!datalist) return;
    const cats = Array.from(new Set(state.channels.map(c => c.category || 'General'))).sort();
    datalist.innerHTML = cats.map(cat => '<option value="' + esc(cat) + '"></option>').join('');
  }

  function updateLogoPreview(url) {
    const preview = $('form-logo-preview');
    if (!preview) return;
    if (url && url.trim()) {
      preview.innerHTML = '<img src="' + esc(url.trim()) + '" class="h-full w-full object-contain p-1" onerror="this.onerror=null;this.parentElement.innerHTML=\'<span>LOGO</span>\'" />';
    } else {
      preview.innerHTML = '<span>LOGO</span>';
    }
  }

  function openAddModal() {
    $('modal-title').innerHTML = '<svg viewBox="0 0 24 24" class="h-5 w-5 text-red-500" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 5v14M5 12h14"/></svg><span>Añadir nuevo canal</span>';
    $('form-original-url').value = '';
    $('form-name').value = '';
    $('form-url').value = '';
    $('form-logo').value = '';
    $('form-cat').value = 'General';
    updateLogoPreview('');
    updateCategorySuggestions();
    $('modal-channel').classList.remove('hidden');
    $('form-name').focus();
  }

  function openEditModal(url) {
    const ch = state.channels.find(c => c.url === url) || recents.find(c => c.url === url);
    if (!ch) return;
    $('modal-title').innerHTML = '<svg viewBox="0 0 24 24" class="h-5 w-5 text-blue-400" fill="none" stroke="currentColor" stroke-width="2"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"/><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"/></svg><span>Editar canal</span>';
    $('form-original-url').value = ch.url;
    $('form-name').value = ch.name || '';
    $('form-url').value = ch.url || '';
    $('form-logo').value = ch.logo || '';
    $('form-cat').value = ch.category || 'General';
    updateLogoPreview(ch.logo || '');
    updateCategorySuggestions();
    $('modal-channel').classList.remove('hidden');
    $('form-name').focus();
  }

  function closeChannelModal() {
    $('modal-channel').classList.add('hidden');
  }

  function confirmDeleteChannel(url) {
    const ch = state.channels.find(c => c.url === url) || recents.find(c => c.url === url);
    if (!ch) return;
    channelToDelete = ch;
    $('delete-channel-name').textContent = ch.name;
    $('modal-delete').classList.remove('hidden');
  }

  function closeDeleteModal() {
    $('modal-delete').classList.add('hidden');
    channelToDelete = null;
  }

  function deleteChannelConfirmed() {
    if (!channelToDelete) return;
    const targetUrl = channelToDelete.url;
    const targetName = channelToDelete.name;

    deletedUrls.add(targetUrl);
    masterChannels.set(targetUrl, channelToDelete);
    saveJSON(LS_DELETED_CHANNELS, Array.from(deletedUrls));

    state.channels = state.channels.filter(c => c.url !== targetUrl);
    if (favMap.has(targetUrl)) {
      favMap.delete(targetUrl);
      favorites = favorites.filter(c => c.url !== targetUrl);
      saveJSON(LS_FAVS, favorites);
    }
    recents = recents.filter(c => c.url !== targetUrl);
    saveJSON(LS_RECENT, recents);

    if (state.current && state.current.url === targetUrl) {
      teardown();
      if (state.channels.length > 0) {
        playChannel(state.channels[0]);
      } else {
        state.current = null;
        updateNowPlaying();
        showOffline('No hay canales disponibles en la lista.');
      }
    }

    saveChannels();
    closeDeleteModal();
    toast('Canal «' + targetName + '» eliminado', 'ok');
  }

  function exportM3U() {
    let lines = ['#EXTM3U'];
    state.channels.forEach(ch => {
      let ext = '#EXTINF:-1';
      if (ch.tvg && ch.tvg.id) ext += ' tvg-id="' + ch.tvg.id + '"';
      if (ch.logo) ext += ' tvg-logo="' + ch.logo + '"';
      if (ch.category) ext += ' group-title="' + ch.category + '"';
      ext += ',' + (ch.name || 'Canal');
      lines.push(ext);
      lines.push(ch.url);
    });
    const blob = new Blob([lines.join('\n')], { type: 'audio/x-mpegurl;charset=utf-8' });
    const u = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = u;
    a.download = 'canales-peru.m3u';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(u);
    toast('Lista M3U descargada (' + state.channels.length + ' canales)', 'ok');
  }

  function resetDefaultChannels() {
    if (!confirm('¿Restaurar la lista original de canales de Perú?\nSe restablecerán los canales predeterminados (160 activos y 71 caídos en Eliminados).')) {
      return;
    }
    try {
      localStorage.removeItem(LS_CUSTOM_CHANNELS);
      localStorage.removeItem(LS_DELETED_CHANNELS);
    } catch (_) {}
    deletedUrls = new Set(DEFAULT_DELETED_URLS);
    saveJSON(LS_DELETED_CHANNELS, Array.from(deletedUrls));
    if ($('modal-manage')) $('modal-manage').classList.add('hidden');
    loadAll(true);
    renderTrash();
    toast('Lista de canales restaurada a los valores predeterminados', 'info');
  }

  /* ============================================================
     TVSAM CLICK: PAPELERA, ESCANEO Y GESTIÓN DE ARCHIVOS
     ============================================================ */
  function renderTrash() {
    const body = $('trash-body');
    if (!body) return;
    const items = [];
    deletedUrls.forEach(url => {
      const ch = masterChannels.get(url) || { name: 'Canal ' + url.slice(-24), url: url, category: 'General' };
      items.push(ch);
    });
    items.sort((a, b) => a.name.localeCompare(b.name, 'es', { sensitivity: 'base', numeric: true }));

    if (!items.length) {
      body.innerHTML = '<div class="flex flex-col items-center justify-center py-10 text-center text-xs text-slate-500">' +
        '<svg viewBox="0 0 24 24" class="h-8 w-8 mb-2 opacity-50" fill="none" stroke="currentColor" stroke-width="1.5"><path d="M3 6h18M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/></svg>' +
        'No hay canales en la papelera.</div>';
      return;
    }

    body.innerHTML = items.map(ch => {
      const logoHtml = ch.logo
        ? `<div class="relative h-8 w-8 shrink-0 overflow-hidden rounded-lg bg-black/40 border border-white/10">
             <div class="grid h-full w-full place-items-center text-[10px] font-black text-red-300 bg-red-600/20">${esc(initials(ch.name))}</div>
             <img src="${esc(ch.logo)}" alt="${esc(ch.name)}" class="absolute inset-0 h-full w-full object-contain p-0.5" onerror="this.style.display='none'">
           </div>`
        : `<div class="grid h-8 w-8 shrink-0 place-items-center rounded-lg bg-red-600/20 text-[10px] font-black text-red-300 border border-red-500/30">${esc(initials(ch.name))}</div>`;

      return `<div class="flex items-center gap-3 rounded-xl border border-white/5 bg-white/[0.03] p-2.5 transition hover:bg-white/[0.07]">
        ${logoHtml}
        <div class="min-w-0 flex-1">
          <div class="truncate text-xs font-semibold text-white">${esc(ch.name)}</div>
          <div class="truncate text-[10px] text-slate-400">${esc(ch.category || 'General')} · ${esc(ch.id || 'Perú')}</div>
        </div>
        <button data-restore="${esc(ch.url)}" type="button" class="btn text-[11px] shrink-0 font-medium hover:text-white" title="Restaurar canal">
          ↺ Restaurar
        </button>
      </div>`;
    }).join('');
  }

  function restoreChannel(url) {
    if (!deletedUrls.has(url)) return;
    deletedUrls.delete(url);
    saveJSON(LS_DELETED_CHANNELS, Array.from(deletedUrls));

    const ch = masterChannels.get(url) || { name: 'Canal', url: url, category: 'General' };
    if (!state.channels.some(c => c.url === url)) {
      state.channels.push(ch);
      state.channels.sort((a, b) => a.name.localeCompare(b.name, 'es', { sensitivity: 'base', numeric: true }));
      saveChannels();
    }
    renderAll();
    renderTrash();
    toast(`Canal «${ch.name}» restaurado`, 'ok');
  }

  function restoreAllChannels() {
    if (deletedUrls.size === 0) {
      toast('No hay canales eliminados para restaurar', 'info');
      return;
    }
    const count = deletedUrls.size;
    deletedUrls.forEach(url => {
      const ch = masterChannels.get(url) || { name: 'Canal', url: url, category: 'General' };
      if (!state.channels.some(c => c.url === url)) {
        state.channels.push(ch);
      }
    });
    deletedUrls.clear();
    saveJSON(LS_DELETED_CHANNELS, []);
    state.channels.sort((a, b) => a.name.localeCompare(b.name, 'es', { sensitivity: 'base', numeric: true }));
    saveChannels();
    renderAll();
    renderTrash();
    toast(`Se restauraron ${count} canales`, 'ok');
  }

  let scan = null;
  async function scanChannels() {
    if (scan && scan.active) {
      scan.cancelled = true;
      scan.controllers.forEach(c => { try { c.abort(); } catch(e){} });
      return;
    }
    const targets = state.channels.filter(ch => !deletedUrls.has(ch.url));
    if (!targets.length) {
      toast('No hay canales para verificar', 'info');
      return;
    }
    if (!confirm(`Se comprobarán ${targets.length} canales. Los que no respondan se moverán a Eliminados (podrás restaurarlos cuando quieras). ¿Continuar?`)) return;

    scan = { active: true, cancelled: false, controllers: [], done: 0, dead: 0 };
    const btnScan = $('btn-scan');
    if (btnScan) btnScan.textContent = '⏹ Detener verificación';
    toast('Verificando canales en paralelo…', 'info');

    const check = ch => new Promise(resolve => {
      const ctl = new AbortController();
      scan.controllers.push(ctl);
      const timer = setTimeout(() => { try { ctl.abort(); } catch(e){} resolve(false); }, 3500);
      fetch(ch.url, { signal: ctl.signal, cache: 'no-store', redirect: 'follow' })
        .then(res => { clearTimeout(timer); try { ctl.abort(); } catch(e){} resolve(res.ok); })
        .catch(() => { clearTimeout(timer); resolve(false); });
    });

    let idx = 0;
    const worker = async () => {
      while (scan && !scan.cancelled && idx < targets.length) {
        const ch = targets[idx++];
        const ok = await check(ch);
        if (!scan || scan.cancelled) break;
        scan.done++;
        if (!ok) {
          scan.dead++;
          deletedUrls.add(ch.url);
          masterChannels.set(ch.url, ch);
        }
        if (scan.done % 10 === 0 || scan.done === targets.length) {
          saveJSON(LS_DELETED_CHANNELS, Array.from(deletedUrls));
          state.channels = state.channels.filter(c => !deletedUrls.has(c.url));
          renderAll();
          renderTrash();
        }
      }
    };
    await Promise.all(Array.from({ length: 12 }, worker));
    const done = scan.done, dead = scan.dead, stopped = scan.cancelled;
    saveJSON(LS_DELETED_CHANNELS, Array.from(deletedUrls));
    state.channels = state.channels.filter(c => !deletedUrls.has(c.url));
    saveChannels();
    renderAll();
    renderTrash();
    scan = null;
    if (btnScan) btnScan.textContent = '⚡ Verificar canales';
    toast(`Verificación ${stopped ? 'detenida' : 'completada'}: ${done} comprobados, ${dead} movidos a Eliminados.`, dead ? 'warn' : 'ok');
  }

  // Listeners de cabecera TVSam Click
  if ($('btn-scan')) $('btn-scan').addEventListener('click', scanChannels);
  if ($('btn-load')) $('btn-load').addEventListener('click', () => { if ($('file-input')) $('file-input').click(); });
  if ($('file-input')) {
    $('file-input').addEventListener('change', async e => {
      const file = e.target.files && e.target.files[0];
      if (!file) return;
      try {
        const text = await file.text();
        const parsed = parseM3U(text, 'PE');
        if (!parsed.length) {
          toast('No se encontraron canales válidos en el archivo M3U', 'warn');
          return;
        }
        parsed.forEach(ch => masterChannels.set(ch.url, ch));
        const newChannels = parsed.filter(ch => !deletedUrls.has(ch.url));
        const existingUrls = new Set(state.channels.map(c => c.url));
        let added = 0;
        newChannels.forEach(ch => {
          if (!existingUrls.has(ch.url)) {
            state.channels.push(ch);
            existingUrls.add(ch.url);
            added++;
          }
        });
        state.channels.sort((a, b) => a.name.localeCompare(b.name, 'es', { sensitivity: 'base', numeric: true }));
        saveChannels();
        renderAll();
        toast(`Cargados ${parsed.length} canales (${added} nuevos) desde ${file.name}`, 'ok');
      } catch (err) {
        toast('Error al leer el archivo M3U: ' + err.message, 'err');
      } finally {
        e.target.value = '';
      }
    });
  }
  if ($('btn-export')) $('btn-export').addEventListener('click', exportM3U);
  if ($('btn-trash')) {
    $('btn-trash').addEventListener('click', () => {
      renderTrash();
      if ($('modal-trash')) $('modal-trash').classList.remove('hidden');
    });
  }
  if ($('btn-trash-close')) {
    $('btn-trash-close').addEventListener('click', () => {
      if ($('modal-trash')) $('modal-trash').classList.add('hidden');
    });
  }
  if ($('modal-trash')) {
    $('modal-trash').addEventListener('click', e => {
      if (e.target === $('modal-trash')) $('modal-trash').classList.add('hidden');
      const restoreBtn = e.target.closest('[data-restore]');
      if (restoreBtn) restoreChannel(restoreBtn.dataset.restore);
    });
  }
  if ($('btn-restore-all')) $('btn-restore-all').addEventListener('click', restoreAllChannels);
  if ($('btn-reset')) $('btn-reset').addEventListener('click', resetDefaultChannels);


  if ($('channel-list')) {
    $('channel-list').addEventListener('click', e => {
      if (e.target.closest('[data-more]')) { renderLimit += PAGE_SIZE; drawChannels(); return; }

      const editBtn = e.target.closest('[data-edit]');
      if (editBtn) {
        e.stopPropagation();
        openEditModal(editBtn.dataset.edit);
        return;
      }

      const delBtn = e.target.closest('[data-delete]');
      if (delBtn) {
        e.stopPropagation();
        confirmDeleteChannel(delBtn.dataset.delete);
        return;
      }

      const favBtn = e.target.closest('[data-fav]');
      if (favBtn) {
        e.stopPropagation();
        const url = favBtn.dataset.fav;
        const ch = state.channels.find(c => c.url === url) || recents.find(c => c.url === url);
        if (!ch) return;
        if (favMap.has(url)) { favMap.delete(url); favorites = favorites.filter(c => c.url !== url); toast('Quitado de favoritos', 'info'); }
        else { favMap.set(url, ch); favorites = [ch, ...favorites]; toast('Añadido a favoritos', 'ok'); }
        saveJSON(LS_FAVS, favorites);
        renderAll(); updateNowPlaying();
        return;
      }
      const card = e.target.closest('[data-url]');
      if (card) {
        const ch = state.channels.find(c => c.url === card.dataset.url) || recents.find(c => c.url === card.dataset.url);
        if (ch) playChannel(ch);
      }
    });
  }

  $('form-channel').addEventListener('submit', e => {
    e.preventDefault();
    const origUrl = $('form-original-url').value.trim();
    const name = $('form-name').value.trim();
    const url = $('form-url').value.trim();
    const logo = $('form-logo').value.trim();
    const category = $('form-cat').value.trim() || 'General';

    if (!name || !url) {
      toast('Por favor completa el nombre y la URL del canal', 'warn');
      return;
    }

    if (origUrl) {
      const idx = state.channels.findIndex(c => c.url === origUrl);
      if (idx >= 0) {
        state.channels[idx] = {
          ...state.channels[idx],
          name,
          url,
          logo,
          category
        };
      } else {
        state.channels.push({
          name,
          url,
          logo,
          category,
          country: 'PE',
          tvg: { id: '', name: '', logo, group: category }
        });
      }

      if (origUrl !== url) {
        if (favMap.has(origUrl)) {
          favMap.delete(origUrl);
          const updatedFav = { ...(state.channels[idx] || { name, url, logo, category, country: 'PE' }) };
          favMap.set(url, updatedFav);
          favorites = favorites.map(f => f.url === origUrl ? updatedFav : f);
          saveJSON(LS_FAVS, favorites);
        }
        recents = recents.map(r => r.url === origUrl ? { ...r, name, url, logo, category } : r);
        saveJSON(LS_RECENT, recents);

        if (state.current && state.current.url === origUrl) {
          playChannel(state.channels[idx] || { name, url, logo, category, country: 'PE' });
        }
      } else {
        if (favMap.has(url)) {
          const updatedFav = { ...favMap.get(url), name, logo, category };
          favMap.set(url, updatedFav);
          favorites = favorites.map(f => f.url === url ? updatedFav : f);
          saveJSON(LS_FAVS, favorites);
        }
        recents = recents.map(r => r.url === url ? { ...r, name, logo, category } : r);
        saveJSON(LS_RECENT, recents);

        if (state.current && state.current.url === url) {
          state.current.name = name;
          state.current.logo = logo;
          state.current.category = category;
          updateNowPlaying();
        }
      }

      saveChannels();
      closeChannelModal();
      toast('Canal «' + name + '» actualizado con éxito', 'ok');
    } else {
      if (state.channels.some(c => c.url === url)) {
        toast('Ya existe un canal con esa URL en la lista', 'warn');
        return;
      }
      if (deletedUrls.has(url)) {
        deletedUrls.delete(url);
        saveJSON(LS_DELETED_CHANNELS, Array.from(deletedUrls));
      }
      const newCh = {
        name,
        url,
        logo,
        category,
        country: 'PE',
        tvg: { id: '', name: '', logo, group: category }
      };
      state.channels.push(newCh);
      saveChannels();
      closeChannelModal();
      toast('Canal «' + name + '» añadido a la lista', 'ok');

      requestAnimationFrame(() => {
        const el = Array.from(document.querySelectorAll('#channel-list [data-url]')).find(n => n.dataset.url === url);
        if (el) el.scrollIntoView({ block: 'center', behavior: 'smooth' });
      });
    }
  });

  if ($('btn-add-channel')) $('btn-add-channel').addEventListener('click', openAddModal);
  $('btn-modal-close').addEventListener('click', closeChannelModal);
  $('btn-modal-cancel').addEventListener('click', closeChannelModal);
  $('form-logo').addEventListener('input', e => updateLogoPreview(e.target.value));

  $('btn-delete-cancel').addEventListener('click', closeDeleteModal);
  $('btn-delete-confirm').addEventListener('click', deleteChannelConfirmed);

  if ($('btn-manage-menu')) $('btn-manage-menu').addEventListener('click', () => $('modal-manage').classList.remove('hidden'));
  $('btn-manage-close').addEventListener('click', () => $('modal-manage').classList.add('hidden'));
  $('btn-export-m3u').addEventListener('click', exportM3U);
  $('btn-reset-list').addEventListener('click', resetDefaultChannels);

  const fileInput = $('file-import-m3u');
  if (fileInput) {
    fileInput.addEventListener('change', e => {
      const file = e.target.files && e.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = evt => {
        const text = evt.target.result;
        const parsed = parseM3U(text, 'PE');
        if (!parsed.length) {
          toast('No se encontraron canales válidos en el archivo', 'warn');
          return;
        }
        let addedCount = 0;
        const existingUrls = new Set(state.channels.map(c => c.url));
        parsed.forEach(ch => {
          if (!existingUrls.has(ch.url)) {
            existingUrls.add(ch.url);
            state.channels.push(ch);
            addedCount++;
          }
        });
        saveChannels();
        $('modal-manage').classList.add('hidden');
        toast('Se importaron ' + addedCount + ' canales nuevos', 'ok');
        fileInput.value = '';
      };
      reader.readAsText(file);
    });
  }

  ['modal-channel', 'modal-delete', 'modal-manage'].forEach(id => {
    $(id).addEventListener('click', e => {
      if (e.target === $(id)) $(id).classList.add('hidden');
    });
  });

  $('btn-fav').addEventListener('click', () => {
    const ch = state.current;
    if (!ch) return;
    if (favMap.has(ch.url)) { favMap.delete(ch.url); favorites = favorites.filter(c => c.url !== ch.url); toast('Quitado de favoritos', 'info'); }
    else { favMap.set(ch.url, ch); favorites = [ch, ...favorites]; toast('Añadido a favoritos', 'ok'); }
    saveJSON(LS_FAVS, favorites);
    renderAll(); updateNowPlaying();
  });

  $('btn-prev').addEventListener('click', () => step(-1));
  $('btn-next').addEventListener('click', () => step(1));

  $('btn-play').addEventListener('click', () => {
    if (video.paused) video.play().catch(onPlayBlocked); else video.pause();
  });
  $('btn-big-play').addEventListener('click', () => {
    video.play().then(() => overlay(null)).catch(onPlayBlocked);
  });
  $('btn-retry').addEventListener('click', () => { if (state.current) playChannel(state.current); });

  $('btn-mute').addEventListener('click', () => {
    video.muted = !video.muted;
    if (!video.muted && video.volume === 0) video.volume = 0.8;
    saveJSON(LS_VOL, video.volume);
    syncVolumeIcon();
  });
  $('volume').addEventListener('input', e => {
    video.volume = Number(e.target.value);
    video.muted = video.volume === 0;
    saveJSON(LS_VOL, video.volume);
    syncVolumeIcon();
  });

  /* --- Pantalla completa: API nativa robusta + respaldo CSS si el navegador la bloquea --- */
  function isFakeFs() { return $('player-shell').classList.contains('fake-fullscreen'); }
  function isFullscreen() {
    return !!(
      document.fullscreenElement ||
      document.webkitFullscreenElement ||
      document.mozFullScreenElement ||
      document.msFullscreenElement ||
      (video && video.webkitDisplayingFullscreen) ||
      isFakeFs()
    );
  }

  /* ============================================================
     OSD FLOTANTE DE CANALES EN EL PLAYER
     ============================================================ */
  function isOsdOpen() {
    const osd = $('player-osd-channels');
    return osd && !osd.classList.contains('translate-x-full');
  }

  function renderOsdChannels() {
    const container = $('osd-channel-list');
    if (!container) return;
    const cleanStr = s => (s || '').toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, '');
    const q = cleanStr($('osd-search')?.value || '');
    let list = state.channels;
    if (q) {
      list = list.filter(c => cleanStr(c.name).includes(q) || cleanStr(c.category).includes(q));
    }
    const countEl = $('osd-count');
    if (countEl) countEl.textContent = list.length;

    if (!list.length) {
      container.innerHTML = '<div class="py-8 text-center text-xs text-slate-500">No se encontraron canales</div>';
      return;
    }

    container.innerHTML = list.map(ch => {
      const isCurrent = state.current && state.current.url === ch.url;
      const logo = ch.logo
        ? `<div class="relative h-7 w-7 shrink-0 overflow-hidden rounded bg-black/40 border border-white/10">
             <div class="grid h-full w-full place-items-center text-[9px] font-black text-blue-300 bg-blue-600/20">${esc(initials(ch.name))}</div>
             <img src="${esc(ch.logo)}" alt="${esc(ch.name)}" class="absolute inset-0 h-full w-full object-contain p-0.5" onerror="this.style.display='none'">
           </div>`
        : `<div class="grid h-7 w-7 shrink-0 place-items-center rounded bg-blue-600/20 text-[9px] font-black text-blue-300 border border-blue-500/30">${esc(initials(ch.name))}</div>`;

      return `<button data-osd-url="${esc(ch.url)}" type="button" class="group flex w-full items-center gap-2.5 rounded-lg px-2 py-1.5 text-left transition cursor-pointer ${
        isCurrent ? 'bg-blue-600 text-white shadow-md' : 'text-slate-300 hover:bg-white/10 hover:text-white'
      }">
        ${logo}
        <div class="min-w-0 flex-1">
          <div class="truncate text-xs font-semibold ${isCurrent ? 'text-white' : 'text-slate-200'}">${esc(ch.name)}</div>
          <div class="truncate text-[10px] ${isCurrent ? 'text-blue-200' : 'text-slate-400'}">${esc(ch.category || 'General')}</div>
        </div>
        ${isCurrent ? '<span class="h-2 w-2 rounded-full bg-white pulse-dot shrink-0"></span>' : ''}
      </button>`;
    }).join('');

    const activeEl = container.querySelector('[data-osd-url].bg-blue-600');
    if (activeEl) {
      activeEl.scrollIntoView({ block: 'nearest' });
    }
  }

  function toggleOsd(force) {
    const osd = $('player-osd-channels');
    if (!osd) return;
    const shouldOpen = typeof force === 'boolean' ? force : osd.classList.contains('translate-x-full');
    if (shouldOpen) {
      renderOsdChannels();
      osd.classList.remove('translate-x-full');
      resetControlsTimeout();
      $('player-shell').classList.remove('hide-controls');
      setTimeout(() => $('osd-search')?.focus(), 150);
    } else {
      osd.classList.add('translate-x-full');
      $('osd-search')?.blur();
    }
  }

  if ($('btn-player-channels')) {
    $('btn-player-channels').addEventListener('click', e => {
      e.stopPropagation();
      toggleOsd();
    });
  }

  if ($('btn-close-osd')) {
    $('btn-close-osd').addEventListener('click', e => {
      e.stopPropagation();
      toggleOsd(false);
    });
  }

  if ($('osd-search')) {
    $('osd-search').addEventListener('input', () => renderOsdChannels());
  }

  if ($('osd-channel-list')) {
    $('osd-channel-list').addEventListener('click', e => {
      const btn = e.target.closest('[data-osd-url]');
      if (!btn) return;
      const url = btn.dataset.osdUrl;
      const ch = state.channels.find(c => c.url === url);
      if (ch) {
        playChannel(ch);
        renderOsdChannels();
      }
    });
  }

  let fsHideTimer = null;
  function resetControlsTimeout() {
    const shell = $('player-shell');
    shell.classList.remove('hide-controls');
    clearTimeout(fsHideTimer);
    if (isOsdOpen()) return;
    if (isFullscreen() && !video.paused) {
      fsHideTimer = setTimeout(() => {
        if (isFullscreen() && !video.paused && !isOsdOpen()) {
          shell.classList.add('hide-controls');
        }
      }, 2800);
    }
  }

  function fallbackFullscreen(el) {
    el.classList.add('fake-fullscreen');
    document.body.classList.add('fake-fs');
    syncFullscreenUI();
  }

  async function leaveFullscreen() {
    if (isFakeFs()) {
      $('player-shell').classList.remove('fake-fullscreen');
      document.body.classList.remove('fake-fs');
    }
    if (document.fullscreenElement || document.webkitFullscreenElement || document.mozFullScreenElement || document.msFullscreenElement) {
      const exitFn = document.exitFullscreen || document.webkitExitFullscreen || document.mozCancelFullScreen || document.msExitFullscreen;
      if (exitFn) {
        try { await exitFn.call(document); } catch (_) {}
      }
    } else if (video && video.webkitDisplayingFullscreen && video.webkitExitFullscreen) {
      try { video.webkitExitFullscreen(); } catch (_) {}
    }
    syncFullscreenUI();
  }

  async function enterFullscreen() {
    const el = $('player-shell');
    let ok = false;
    try {
      if (el.requestFullscreen) {
        await el.requestFullscreen();
        ok = true;
      } else if (el.webkitRequestFullscreen) {
        await el.webkitRequestFullscreen();
        ok = true;
      } else if (el.mozRequestFullScreen) {
        await el.mozRequestFullScreen();
        ok = true;
      } else if (el.msRequestFullscreen) {
        await el.msRequestFullscreen();
        ok = true;
      }
    } catch (_) {
      ok = false;
    }

    if (!ok) {
      if (video && video.webkitEnterFullscreen) {
        try {
          video.webkitEnterFullscreen();
          return;
        } catch (_) {}
      }
      fallbackFullscreen(el);
    } else {
      syncFullscreenUI();
    }
  }

  function toggleFullscreen() {
    if (isFullscreen()) leaveFullscreen();
    else enterFullscreen();
  }

  function syncFullscreenUI() {
    const fs = isFullscreen();
    const enterIcon = $('icon-fs-enter');
    const exitIcon = $('icon-fs-exit');
    if (enterIcon) enterIcon.classList.toggle('hidden', fs);
    if (exitIcon) exitIcon.classList.toggle('hidden', !fs);
    const btnFs = $('btn-fs');
    if (btnFs) btnFs.title = fs ? 'Salir de pantalla completa (F / Esc)' : 'Pantalla completa (F)';

    if (!fs) {
      $('player-shell').classList.remove('fake-fullscreen', 'hide-controls');
      document.body.classList.remove('fake-fs');
      clearTimeout(fsHideTimer);
    } else {
      resetControlsTimeout();
    }
  }

  // Eventos de cambio de pantalla completa del sistema
  ['fullscreenchange', 'webkitfullscreenchange', 'mozfullscreenchange', 'MSFullscreenChange'].forEach(ev => {
    document.addEventListener(ev, syncFullscreenUI);
  });
  video.addEventListener('webkitbeginfullscreen', syncFullscreenUI);
  video.addEventListener('webkitendfullscreen', syncFullscreenUI);

  $('btn-fs').addEventListener('click', toggleFullscreen);

  $('btn-pip').addEventListener('click', async () => {
    try {
      if (document.pictureInPictureElement) await document.exitPictureInPicture();
      else if (document.pictureInPictureEnabled && video.requestPictureInPicture) await video.requestPictureInPicture();
      else toast('Picture-in-Picture no disponible en este navegador', 'warn');
    } catch (_) { toast('No se pudo activar Picture-in-Picture', 'warn'); }
  });

  // Ocultar controles automáticamente en pantalla completa al dejar de mover el cursor
  const playerShell = $('player-shell');
  playerShell.addEventListener('mousemove', resetControlsTimeout);
  playerShell.addEventListener('touchstart', resetControlsTimeout, { passive: true });
  playerShell.addEventListener('mouseleave', () => {
    if (isFullscreen() && !video.paused) playerShell.classList.add('hide-controls');
  });

  // Clic simple en el video: pausar/reanudar. Doble clic rápido: pantalla completa (sin pausas molestas).
  let videoClickTimer = null;
  video.addEventListener('click', () => {
    if (isOsdOpen()) {
      toggleOsd(false);
      return;
    }
    if (videoClickTimer) {
      clearTimeout(videoClickTimer);
      videoClickTimer = null;
      toggleFullscreen();
    } else {
      videoClickTimer = setTimeout(() => {
        videoClickTimer = null;
        if (video.paused) video.play().catch(onPlayBlocked);
        else video.pause();
      }, 240);
    }
  });
  video.addEventListener('dblclick', e => e.preventDefault());

  video.addEventListener('play', () => { syncPlayIcon(); overlay(null); resetControlsTimeout(); });
  video.addEventListener('pause', () => { syncPlayIcon(); resetControlsTimeout(); });
  video.addEventListener('playing', () => { clearWatchdog(); $('live-badge').classList.remove('opacity-0'); overlay(null); resetControlsTimeout(); });
  video.addEventListener('volumechange', syncVolumeIcon);

  // Atajos de teclado
  document.addEventListener('keydown', e => {
    if (e.key === 'Escape') {
      if (document.activeElement && typeof document.activeElement.blur === 'function') {
        document.activeElement.blur();
      }
      if (typeof isOsdOpen === 'function' && isOsdOpen()) { toggleOsd(false); return; }
      if ($('modal-trash') && !$('modal-trash').classList.contains('hidden')) { $('modal-trash').classList.add('hidden'); return; }
      if (!$('modal-channel').classList.contains('hidden')) { closeChannelModal(); return; }
      if (!$('modal-delete').classList.contains('hidden')) { closeDeleteModal(); return; }
      if (!$('modal-manage').classList.contains('hidden')) { $('modal-manage').classList.add('hidden'); return; }
      if (isFullscreen()) leaveFullscreen(); else toggleSidebar(false);
      return;
    }

    const typing = /INPUT|TEXTAREA|SELECT/.test(document.activeElement.tagName);
    if (e.key === '/' && !typing && $('search')) { e.preventDefault(); $('search').focus(); return; }
    if (typing) return;
    if (e.code === 'Space') { e.preventDefault(); $('btn-play').click(); }
    else if (e.key === 'ArrowRight') { e.preventDefault(); step(1); }
    else if (e.key === 'ArrowLeft') { e.preventDefault(); step(-1); }
    else if (e.key.toLowerCase() === 'c') { e.preventDefault(); toggleOsd(); }
    else if (e.key.toLowerCase() === 'f') toggleFullscreen();
    else if (e.key.toLowerCase() === 'm') $('btn-mute').click();
  });

  /* ============================================================
     7. ARRANQUE
     ============================================================ */
  syncVolumeIcon();
  syncPlayIcon();
  loadAll();
})();
</script>
</body>
</html>



