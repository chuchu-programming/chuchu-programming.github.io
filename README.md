<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mimi★ — Virtual Broadcaster / VT-002</title>
<script src="https://cdn.tailwindcss.com"></script>
<link href="https://fonts.googleapis.com/css2?family=Audiowide&family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
<style>
  :root{
    --white:#FFFFFF;
    --cyan:#07BBF4;
    --pink:#F34D8E;
    --orange:#FF4600;
    --yellow:#E3FF00;
    --navy:#262851;
    --r-sm:12px;
    --r-md:18px;
    --r-lg:28px;
  }
  *{-webkit-font-smoothing:antialiased;box-sizing:border-box}
  html,body{margin:0;padding:0;background:var(--navy)}
  body{font-family:'Space Grotesk',sans-serif;color:var(--navy);overflow-x:hidden}
  .font-display{font-family:'Audiowide',cursive;letter-spacing:.01em}
  .font-mono{font-family:'JetBrains Mono',monospace}

  /* === STRUCTURE === */
  #pin-stack{position:fixed;top:0;left:0;width:100vw;height:100vh;z-index:2;overflow:hidden}
  #pin-stack .section { position:absolute; inset:0; width:100vw; height:100vh; overflow:hidden; will-change:transform,opacity; opacity:0; pointer-events:none }
  #pin-stack .section.active { opacity:1; pointer-events:auto }
  
  .section .inner-scroll{width:100%;height:100%;overflow-y:auto;overflow-x:hidden;scrollbar-width:none;-ms-overflow-style:none}
  .section .inner-scroll::-webkit-scrollbar{display:none}

  /* === SCROLL PROGRESS === */
  #scroll-progress{position:fixed;top:0;left:0;height:4px;width:100%;background:var(--cyan);z-index:100;transform-origin:left center;transform:scaleX(0);will-change:transform}

  /* === WIPE OVERLAY === */
  #wipe-overlay{position:fixed;inset:0;pointer-events:none;z-index:50}
  .wipe-rect{position:absolute;left:-100%;width:200%;height:calc(160vh/3 + 20px);will-change:transform;border-radius:var(--r-lg)}
  .wipe-rect-1{top:0;background:var(--cyan)}
  .wipe-rect-2{top:calc(160vh/3);background:var(--navy)}
  .wipe-rect-3{top:calc(160vh/3*2);background:var(--pink)}

  /* === GRID BG === */
  .grid-bg{background-image:linear-gradient(rgba(38,40,81,.05) 1px,transparent 1px),linear-gradient(90deg,rgba(38,40,81,.05) 1px,transparent 1px);background-size:80px 80px}
  .grid-bg-dark{background-image:linear-gradient(rgba(255,255,255,.08) 1px,transparent 1px),linear-gradient(90deg,rgba(255,255,255,.08) 1px,transparent 1px);background-size:60px 60px}

  /* === NOISE === */
  .noise-overlay{position:fixed;inset:0;pointer-events:none;z-index:60;opacity:.03;mix-blend-mode:multiply;background-image:url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E")}

  /* === PULSE === */
  .pulse-dot{width:8px;height:8px;background:var(--orange);border-radius:50%;box-shadow:0 0 0 0 rgba(255,70,0,.7);animation:pulse 2s infinite}
  @keyframes pulse{0%{box-shadow:0 0 0 0 rgba(255,70,0,.7)}70%{box-shadow:0 0 0 10px rgba(255,70,0,0)}100%{box-shadow:0 0 0 0 rgba(255,70,0,0)}}

  /* === MARQUEE === */
  .marquee-track{display:inline-flex;animation:marquee 30s linear infinite;white-space:nowrap}
  @keyframes marquee{0%{transform:translateX(0)}100%{transform:translateX(-50%)}}

  /* === FLOAT === */
  @keyframes float{0%,100%{transform:translateY(0) rotate(0deg)}50%{transform:translateY(-15px) rotate(2deg)}}
  .float{animation:float 6s ease-in-out infinite}
  .float-slow{animation:float 9s ease-in-out infinite}

  /* === ROTATION === */
  @keyframes spin-cw{from{transform:rotate(0)}to{transform:rotate(360deg)}}
  @keyframes spin-ccw{from{transform:rotate(0)}to{transform:rotate(-360deg)}}
  .spin-cw-18{animation:spin-cw 18s linear infinite}
  .spin-cw-25{animation:spin-cw 25s linear infinite}
  .spin-cw-40{animation:spin-cw 40s linear infinite}
  .spin-ccw-30{animation:spin-ccw 30s linear infinite}
  .spin-ccw-22{animation:spin-ccw 22s linear infinite}

  .mega-text{font-family:'Audiowide',cursive;line-height:.95;letter-spacing:.005em}

  .content-card{transition:all .5s cubic-bezier(.7,0,.3,1);overflow:hidden;position:relative;cursor:pointer;border-radius:var(--r-lg)}
  .content-card .card-bg{position:absolute;inset:0;transform:translateY(101%);transition:transform .5s cubic-bezier(.7,0,.3,1);z-index:0;border-radius:var(--r-lg)}
  .content-card:hover .card-bg{transform:translateY(0)}
  .content-card:hover .card-inner{color:var(--white)}
  .content-card .card-inner{position:relative;z-index:1;transition:color .3s ease}

  .schedule-slot{transition:all .3s ease;position:relative;border-radius:var(--r-md)}
  .schedule-slot:hover{transform:translate(-3px,-3px);box-shadow:6px 6px 0 var(--navy)}

  .stat-card{transition:transform .4s cubic-bezier(.7,0,.3,1);border-radius:var(--r-md)}
  .stat-card:hover{transform:translateY(-8px) rotate(-1deg)}

  .social-link{transition:all .4s cubic-bezier(.7,0,.3,1);position:relative;overflow:hidden;border-radius:var(--r-md)}
  .social-link .link-bg{position:absolute;inset:0;transform:translateX(-101%);transition:transform .4s cubic-bezier(.7,0,.3,1);z-index:0;border-radius:var(--r-md)}
  .social-link:hover .link-bg{transform:translateX(0)}
  .social-link:hover .link-content{color:var(--white)}
  .social-link .link-content{position:relative;z-index:1;transition:color .3s ease}

  .cta-fill{position:absolute;inset:0;background:var(--pink);transform:translateX(-101%);transition:transform .4s cubic-bezier(.7,0,.3,1);z-index:0;border-radius:inherit}
  .cta-btn:hover .cta-fill{transform:translateX(0)}
  .cta-btn{border-radius:999px}

  .bar{transition:all .3s ease;transform-origin:bottom;border-radius:4px 4px 0 0}
  .bar:hover{transform:scaleY(1.05);filter:brightness(1.1)}

  @media (prefers-reduced-motion:reduce){
    *,*::before,*::after{animation-duration:.01ms!important;animation-iteration-count:1!important;transition-duration:.01ms!important}
  }

  ::-webkit-scrollbar{width:0;height:0}
  ::selection{background:var(--yellow);color:var(--navy)}

  .tag{display:inline-flex;align-items:center;gap:6px;padding:4px 12px;border:1px solid currentColor;font-family:'JetBrains Mono',monospace;font-size:11px;text-transform:uppercase;letter-spacing:.1em;border-radius:999px}
  .scanlines{background:repeating-linear-gradient(0deg,transparent,transparent 2px,rgba(0,0,0,.12) 2px,rgba(0,0,0,.12) 3px)}
  .badge-text{font-family:'JetBrains Mono',monospace;font-size:9px;letter-spacing:.15em;text-transform:uppercase;fill:#262851}
  .marquee-wrap{overflow:hidden}
  a:focus-visible,button:focus-visible{outline:2px solid var(--orange);outline-offset:3px}
  .ornament{position:absolute;pointer-events:none;z-index:2;will-change:transform}
  .rounded-card{border-radius:var(--r-md)}
  .rounded-lg-card{border-radius:var(--r-lg)}
  .rounded-badge{border-radius:999px}

  /* === NAV === */
  #nav{position:fixed;top:0;left:0;right:0;z-index:80;backdrop-filter:blur(10px)}
  .nav-link{transition:color .2s ease;cursor:pointer}
  .nav-link:hover{color:var(--pink)}
  .nav-dot{width:6px;height:6px;border-radius:50%;background:var(--navy);opacity:.25;transition:all .3s ease;cursor:pointer}
  .nav-dot.active{opacity:1;background:var(--pink);transform:scale(1.6)}

  /* Content fit */
  .section .inner-scroll{display:flex;flex-direction:column;justify-content:center;}
</style>
</head>
<body>
  <div class="noise-overlay"></div>

  <!-- SCROLL PROGRESS -->
  <div id="scroll-progress"></div>

  <!-- NAV -->
  <nav id="nav" class="px-6 md:px-10 py-4 flex items-center justify-between bg-white/70 border-b border-[#262851]/10">
    <div class="flex items-center gap-2.5 group cursor-pointer" data-jump="0">
      <div class="w-9 h-9 bg-[#07BBF4] flex items-center justify-center group-hover:bg-[#F34D8E] transition-colors rounded-badge">
        <span class="text-white font-display">M</span>
      </div>
      <span class="font-display text-sm tracking-wider">118R.cl</span>
    </div>
    <div class="hidden md:flex items-center gap-7 font-mono text-xs uppercase tracking-widest">
      <a class="nav-link" data-jump="1">/01 about</a>
      <a class="nav-link" data-jump="2">/02 stats</a>
      <a class="nav-link" data-jump="3">/03 schedule</a>
      <a class="nav-link" data-jump="4">/04 content</a>
      <a class="nav-link" data-jump="5">/05 connect</a>
    </div>
    <div class="flex items-center gap-1.5 md:hidden">
      <span class="nav-dot active" data-jump="0"></span>
      <span class="nav-dot" data-jump="1"></span>
      <span class="nav-dot" data-jump="2"></span>
      <span class="nav-dot" data-jump="3"></span>
      <span class="nav-dot" data-jump="4"></span>
      <span class="nav-dot" data-jump="5"></span>
    </div>
    <div class="flex items-center gap-2 px-3 py-1.5 bg-[#262851] text-white rounded-badge">
      <span class="pulse-dot"></span>
      <span class="font-mono text-[10px] uppercase tracking-wider hidden sm:inline">LIVE NOW</span>
    </div>
  </nav>

  <!-- FIXED STACK -->
  <div id="pin-stack">

    <!-- =================== SECTION 0: HERO =================== -->
    <section id="top" class="section active grid-bg bg-[#FFFFFF]" data-index="0">
      <div class="inner-scroll pt-20 px-6 md:px-10">
        <!-- Ornaments -->
        <div class="absolute top-32 right-[20%] w-28 h-28 bg-[#E3FF00] rounded-full opacity-90 float-slow hidden lg:block"></div>
        <div class="absolute bottom-40 left-[8%] float hidden lg:block"><div class="w-16 h-16 border-4 border-[#FF4600]" style="border-radius:8px"></div></div>
        <div class="absolute top-[55%] left-[40%] w-3 h-3 bg-[#07BBF4] float hidden lg:block rounded-full"></div>
        <div class="absolute top-[30%] left-[5%] w-2 h-2 bg-[#F34D8E] float-slow hidden lg:block rounded-full"></div>

        <div class="ornament top-[15%] right-[8%] hidden lg:block">
          <svg width="90" height="90" viewBox="0 0 100 100" class="spin-cw-25"><polygon points="50,8 92,85 8,85" fill="none" stroke="#07BBF4" stroke-width="3"/></svg>
        </div>
        <div class="ornament top-[60%] left-[3%] hidden lg:block">
          <svg width="60" height="60" viewBox="0 0 100 100" class="spin-ccw-22"><polygon points="50,10 90,85 10,85" fill="#F34D8E" opacity="0.85"/></svg>
        </div>
        <div class="ornament bottom-[18%] right-[28%] hidden lg:block">
          <svg width="70" height="70" viewBox="0 0 100 100" class="spin-cw-40"><circle cx="50" cy="50" r="44" fill="none" stroke="#E3FF00" stroke-width="4" stroke-dasharray="10 6"/></svg>
        </div>
        <div class="ornament top-[18%] left-[33%] hidden lg:block">
          <div class="relative w-24 h-24">
            <svg width="96" height="96" viewBox="0 0 100 100" class="absolute inset-0 spin-ccw-30"><circle cx="50" cy="50" r="44" fill="none" stroke="#FF4600" stroke-width="3"/><circle cx="50" cy="6" r="4" fill="#FF4600"/></svg>
            <svg width="96" height="96" viewBox="0 0 100 100" class="absolute inset-0 spin-cw-18"><circle cx="50" cy="50" r="20" fill="none" stroke="#262851" stroke-width="2" stroke-dasharray="4 4"/></svg>
          </div>
        </div>
        <div class="ornament bottom-[12%] left-[30%] hidden lg:block">
          <svg width="40" height="40" viewBox="0 0 100 100" class="spin-cw-18"><polygon points="50,12 88,80 12,80" fill="none" stroke="#262851" stroke-width="4"/></svg>
        </div>

        <div class="relative z-10 grid grid-cols-1 lg:grid-cols-12 gap-8 items-center max-w-7xl mx-auto w-full py-8">
          <div class="lg:col-span-7">
            <div class="flex flex-wrap items-center gap-3 mb-4">
              <span class="tag text-[#262851]"><span class="w-1.5 h-1.5 bg-[#FF4600] rounded-full"></span>VT-002 / 2026</span>
              <span class="font-mono text-xs uppercase tracking-widest text-[#262851]/60">Independent Virtual Broadcaster</span>
            </div>

            <h1 class="mega-text text-[16vw] sm:text-[14vw] lg:text-[9rem] xl:text-[11rem] text-[#262851]">
              MIMI<span class="text-[#F34D8E]"> AI </span> VTuber
            </h1>

            <div class="mt-4 mb-6 flex items-center gap-4">
              <div class="h-1 w-16 bg-[#07BBF4] rounded-full"></div>
              <p class="font-mono text-xs sm:text-sm uppercase tracking-widest text-[#262851]/80">
                Streaming from sector 7 · since 2021
              </p>
            </div>

            <p class="text-base md:text-lg text-[#262851]/80 max-w-xl leading-relaxed mb-8">
              Broadcasting from a satellite somewhere between Andromeda and your screen. I play games, sing songs, and pretend I'm not three raccoons in a digital trenchcoat.
            </p>

            <div class="flex flex-wrap items-center gap-4">
              <button data-jump="5" class="cta-btn group relative inline-flex items-center gap-3 bg-[#262851] text-white px-7 py-4 overflow-hidden cursor-pointer">
                <span class="cta-fill"></span>
                <span class="relative font-mono text-sm uppercase tracking-wider z-10">Join Stream</span>
                <i class="fa-solid fa-arrow-right relative z-10"></i>
              </button>
              <button data-jump="1" class="inline-flex items-center gap-3 px-7 py-4 border-2 border-[#262851] hover:bg-[#07BBF4] hover:border-[#07BBF4] hover:text-white transition-colors font-mono text-sm uppercase tracking-wider rounded-badge cursor-pointer">
                <i class="fa-solid fa-circle-info"></i>
                About Me
              </button>
            </div>

            <div class="mt-10 grid grid-cols-3 gap-6 max-w-md">
              <div>
                <div class="font-display text-2xl md:text-3xl text-[#F34D8E]">847K</div>
                <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mt-1">Followers</div>
              </div>
              <div>
                <div class="font-display text-2xl md:text-3xl text-[#07BBF4]">2.3K</div>
                <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mt-1">Streamed Hrs</div>
              </div>
              <div>
                <div class="font-display text-2xl md:text-3xl text-[#FF4600]">42</div>
                <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mt-1">Songs Covered</div>
              </div>
            </div>
          </div>

          <div class="lg:col-span-5 relative hidden lg:block">
            <div class="relative portrait-container max-w-md mx-auto">
              <div class="absolute -top-4 -right-4 w-full h-full border-2 border-[#07BBF4] rounded-lg-card"></div>
              <div class="absolute -bottom-4 -left-4 w-full h-full bg-[#F34D8E] rounded-lg-card"></div>

              <div class="relative bg-[#262851] aspect-[3/4] overflow-hidden rounded-lg-card">
                <img src="[/chuchu-programming.github.io/a00.png]" alt="Mimi portrait" class="w-full h-full object-cover" style="mix-blend-mode:luminosity;filter:contrast(1.3) brightness(.85)"/>
                <div class="absolute inset-0 bg-gradient-to-br from-[#07BBF4]/50 via-transparent to-[#F34D8E]/50 mix-blend-color"></div>
                <div class="absolute inset-0 bg-[#262851]/15"></div>
                <div class="absolute inset-0 scanlines pointer-events-none opacity-60"></div>

                <div class="absolute top-4 left-4 right-4 flex justify-between items-start">
                  <div class="bg-[#E3FF00] px-2 py-1 font-mono text-[10px] uppercase tracking-wider text-[#262851] flex items-center gap-1.5 rounded-badge">
                    <span class="w-1.5 h-1.5 bg-[#FF4600] rounded-full animate-pulse"></span>REC 00:42:17
                  </div>
                  <div class="bg-[#262851]/80 backdrop-blur px-2 py-1 font-mono text-[10px] uppercase tracking-wider text-white rounded-badge">CAM 01</div>
                </div>

                <div class="absolute left-2 top-1/2 -translate-y-1/2 flex flex-col gap-2">
                  <div class="w-1 h-4 bg-[#E3FF00] rounded-full"></div>
                  <div class="w-1 h-2 bg-white/40 rounded-full"></div>
                  <div class="w-1 h-2 bg-white/40 rounded-full"></div>
                  <div class="w-1 h-4 bg-[#07BBF4] rounded-full"></div>
                  <div class="w-1 h-2 bg-white/40 rounded-full"></div>
                  <div class="w-1 h-2 bg-white/40 rounded-full"></div>
                  <div class="w-1 h-4 bg-[#F34D8E] rounded-full"></div>
                </div>

                <div class="absolute bottom-0 left-0 right-0 p-4 bg-gradient-to-t from-[#262851] via-[#262851]/80 to-transparent">
                  <div class="flex items-end justify-between">
                    <div>
                      <div class="font-mono text-[10px] uppercase tracking-widest text-white/60">Model ID</div>
                      <div class="font-display text-white text-lg">MIMI-VT002</div>
                    </div>
                    <div class="text-right">
                      <div class="font-mono text-[10px] uppercase tracking-widest text-white/60">Status</div>
                      <div class="font-display text-[#E3FF00] text-lg">ONLINE</div>
                    </div>
                  </div>
                  <div class="mt-3 flex items-center gap-1">
                    <div class="w-1 h-2 bg-[#E3FF00] rounded-full"></div>
                    <div class="w-1 h-3 bg-[#E3FF00] rounded-full"></div>
                    <div class="w-1 h-4 bg-[#E3FF00] rounded-full"></div>
                    <div class="w-1 h-5 bg-[#E3FF00] rounded-full"></div>
                    <span class="ml-2 font-mono text-[9px] uppercase tracking-widest text-white/60">Signal · Strong</span>
                  </div>
                </div>
              </div>

              <div class="absolute -top-6 -left-6 w-20 h-20 spin-cw-18">
                <svg viewBox="0 0 100 100" class="w-full h-full">
                  <defs><path id="circle" d="M 50,50 m -34,0 a 34,34 0 1,1 68,0 a 34,34 0 1,1 -68,0"/></defs>
                  <text class="badge-text"><textPath href="#circle">★ INDEPENDENT VTUBER ★ MODEL 02</textPath></text>
                </svg>
                <div class="absolute inset-0 flex items-center justify-center">
                  <div class="w-3 h-3 bg-[#FF4600] rounded-full"></div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- =================== SECTION 1: ABOUT =================== -->
    <section id="about" class="section bg-[#FFFFFF] relative overflow-hidden" data-index="1">
      <div class="inner-scroll pt-20 pb-12 px-6 md:px-10">
        <div class="ornament top-[10%] right-[6%] hidden lg:block">
          <svg width="70" height="70" viewBox="0 0 100 100" class="spin-ccw-30"><polygon points="50,10 90,85 10,85" fill="none" stroke="#F34D8E" stroke-width="3"/></svg>
        </div>
        <div class="ornament bottom-[14%] left-[2%] hidden lg:block">
          <div class="relative w-20 h-20">
            <svg width="80" height="80" viewBox="0 0 100 100" class="absolute inset-0 spin-cw-25"><circle cx="50" cy="50" r="44" fill="none" stroke="#07BBF4" stroke-width="3"/><circle cx="50" cy="6" r="5" fill="#07BBF4"/></svg>
            <svg width="80" height="80" viewBox="0 0 100 100" class="absolute inset-0 spin-ccw-22"><polygon points="50,20 78,70 22,70" fill="none" stroke="#262851" stroke-width="2"/></svg>
          </div>
        </div>
        <div class="ornament top-[45%] right-[3%] hidden lg:block">
          <svg width="50" height="50" viewBox="0 0 100 100" class="spin-cw-18"><circle cx="50" cy="50" r="40" fill="none" stroke="#E3FF00" stroke-width="3" stroke-dasharray="6 4"/></svg>
        </div>

        <div class="max-w-7xl mx-auto relative z-10 w-full">
          <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
            <div class="lg:col-span-4">
              <div class="font-mono text-xs uppercase tracking-widest text-[#F34D8E] mb-4">/01 — About</div>
              <h2 class="mega-text text-4xl md:text-6xl text-[#262851]">
                Not your<br/>average<br/><span class="text-[#F34D8E]">pixel.</span>
              </h2>
            </div>

            <div class="lg:col-span-7 lg:col-start-6 space-y-8">
              <div>
                <div class="font-mono text-xs uppercase tracking-widest text-[#F34D8E] mb-2">/01.a — Origin</div>
                <p class="text-xl md:text-2xl leading-snug text-[#262851] font-light">
                  I'm <span class="font-display text-[#F34D8E]">MIMI</span> — a virtual broadcaster beaming in from a satellite somewhere between Andromeda and your screen. My model runs on coffee, code, and the collective chaos of a thousand chat messages per second.
                </p>
              </div>

              <div>
                <div class="font-mono text-xs uppercase tracking-widest text-[#07BBF4] mb-2">/01.b — Journey</div>
                <p class="text-base md:text-lg leading-relaxed text-[#262851]/80">
                  Three years ago I pressed "go live" for the first time on a borrowed laptop and a $20 microphone. Today I host daily streams, original music, and a community that calls itself <span class="font-mono text-[#07BBF4] font-bold">the constellation</span> — because every single star matters.
                </p>
              </div>

              <div>
                <div class="font-mono text-xs uppercase tracking-widest text-[#FF4600] mb-2">/01.c — Vibe</div>
                <p class="text-base md:text-lg leading-relaxed text-[#262851]/80">
                  You'll find me speedrunning obscure PS2 games at 3AM, writing sad songs on a digital piano, or having a two-hour conversation with chat about which pasta shape has the most chaotic energy. <span class="font-bold text-[#262851]">It's all part of the show.</span>
                </p>
              </div>

              <div class="pt-6 border-t-2 border-[#262851]">
                <div class="font-mono text-xs uppercase tracking-widest text-[#262851]/60 mb-4">// Quick facts</div>
                <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                  <div>
                    <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mb-1">Debut</div>
                    <div class="font-display text-lg text-[#262851]">11.07.21</div>
                  </div>
                  <div>
                    <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mb-1">Zodiac</div>
                    <div class="font-display text-lg text-[#F34D8E]">SCORPIO</div>
                  </div>
                  <div>
                    <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mb-1">Height</div>
                    <div class="font-display text-lg text-[#07BBF4]">157CM</div>
                  </div>
                  <div>
                    <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mb-1">Fav Snack</div>
                    <div class="font-display text-lg text-[#FF4600]">POCKY</div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- =================== SECTION 2: STATS =================== -->
    <section id="stats" class="section bg-[#07BBF4] text-white relative overflow-hidden" data-index="2">
      <div class="absolute inset-0 grid-bg-dark opacity-50"></div>
      <div class="inner-scroll pt-20 pb-12 px-6 md:px-10">
        <div class="absolute top-20 right-10 w-32 h-32 border-4 border-white/20 rounded-full hidden lg:block spin-cw-25"></div>
        <div class="absolute bottom-20 left-10 hidden lg:block">
          <svg width="80" height="80" viewBox="0 0 100 100" class="spin-ccw-22"><polygon points="50,10 90,85 10,85" fill="#E3FF00" opacity="0.3"/></svg>
        </div>
        <div class="ornament top-[50%] right-[5%] hidden lg:block">
          <svg width="60" height="60" viewBox="0 0 100 100" class="spin-cw-40"><circle cx="50" cy="50" r="40" fill="none" stroke="#E3FF00" stroke-width="2" stroke-dasharray="4 6"/></svg>
        </div>
        <div class="ornament top-[15%] left-[8%] hidden lg:block">
          <svg width="44" height="44" viewBox="0 0 100 100" class="spin-ccw-30"><polygon points="50,12 88,80 12,80" fill="none" stroke="white" stroke-width="3" opacity="0.6"/></svg>
        </div>

        <div class="max-w-7xl mx-auto relative z-10 w-full">
          <div class="flex flex-col md:flex-row md:items-end md:justify-between mb-8 gap-4">
            <div>
              <div class="font-mono text-xs uppercase tracking-widest text-[#E3FF00] mb-3">/02 — By the numbers</div>
              <h2 class="mega-text text-4xl md:text-6xl">
                Constellation<br/><span class="text-[#E3FF00]">in figures.</span>
              </h2>
            </div>
            <p class="font-mono text-sm text-white/70 max-w-sm">
              // Real-time data from across the MIMI network. Numbers update every stream cycle.
            </p>
          </div>

          <div class="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
            <div class="stat-card bg-white p-5 text-[#262851]">
              <div class="flex items-start justify-between mb-2">
                <div class="font-mono text-xs uppercase tracking-widest opacity-70">Followers</div>
                <i class="fa-solid fa-users text-[#F34D8E]"></i>
              </div>
              <div class="font-display text-4xl mb-1" data-counter="847" data-suffix="K">0K</div>
              <div class="font-mono text-[10px] uppercase tracking-wider opacity-70">+12.4K this month</div>
            </div>
            <div class="stat-card bg-[#F34D8E] p-5 text-white">
              <div class="flex items-start justify-between mb-2">
                <div class="font-mono text-xs uppercase tracking-widest opacity-90">Hours Streamed</div>
                <i class="fa-solid fa-clock"></i>
              </div>
              <div class="font-display text-4xl mb-1" data-counter="2314">0</div>
              <div class="font-mono text-[10px] uppercase tracking-wider opacity-90">avg 4.2 hrs/session</div>
            </div>
            <div class="stat-card bg-[#E3FF00] p-5 text-[#262851]">
              <div class="flex items-start justify-between mb-2">
                <div class="font-mono text-xs uppercase tracking-widest opacity-70">Songs Covered</div>
                <i class="fa-solid fa-music text-[#FF4600]"></i>
              </div>
              <div class="font-display text-4xl mb-1" data-counter="42">0</div>
              <div class="font-mono text-[10px] uppercase tracking-wider opacity-70">3 originals released</div>
            </div>
            <div class="stat-card bg-[#262851] p-5 text-white">
              <div class="flex items-start justify-between mb-2">
                <div class="font-mono text-xs uppercase tracking-widest opacity-70">Chat Messages</div>
                <i class="fa-solid fa-comments text-[#E3FF00]"></i>
              </div>
              <div class="font-display text-4xl mb-1" data-counter="8.4" data-suffix="M">0M</div>
              <div class="font-mono text-[10px] uppercase tracking-wider opacity-70">all-time, yes really</div>
            </div>
          </div>

          <div class="grid grid-cols-1 lg:grid-cols-3 gap-4 items-end">
            <div class="lg:col-span-2 bg-[#262851] p-5 rounded-card">
              <div class="flex justify-between items-end mb-4">
                <div>
                  <div class="font-mono text-xs uppercase tracking-widest text-white/60 mb-1">Weekly Activity</div>
                  <div class="font-display text-xl">Streams per day</div>
                </div>
                <div class="font-mono text-xs text-[#E3FF00]">↗ +18% vs last week</div>
              </div>
              <div class="flex items-end justify-between gap-2 h-24">
                <div class="bar flex-1 bg-[#07BBF4]" style="height:60%"></div>
                <div class="bar flex-1 bg-[#07BBF4]" style="height:80%"></div>
                <div class="bar flex-1 bg-[#F34D8E]" style="height:45%"></div>
                <div class="bar flex-1 bg-[#07BBF4]" style="height:90%"></div>
                <div class="bar flex-1 bg-[#F34D8E]" style="height:70%"></div>
                <div class="bar flex-1 bg-[#E3FF00]" style="height:95%"></div>
                <div class="bar flex-1 bg-[#07BBF4]" style="height:55%"></div>
              </div>
              <div class="flex justify-between mt-2 font-mono text-[10px] uppercase tracking-widest text-white/40">
                <span>MON</span><span>TUE</span><span>WED</span><span>THU</span><span>FRI</span><span>SAT</span><span>SUN</span>
              </div>
            </div>
            <div class="bg-white p-5 text-[#262851] rounded-card">
              <div class="font-mono text-xs uppercase tracking-widest text-[#FF4600] mb-2">Peak Concurrent</div>
              <div class="font-display text-4xl text-[#262851]">14,207</div>
              <div class="font-mono text-xs text-[#262851]/60 mt-2">viewers · sat 21:00 JST</div>
              <div class="mt-4 pt-4 border-t border-[#262851]/10">
                <div class="font-mono text-xs uppercase tracking-widest text-[#07BBF4] mb-2">Avg Retention</div>
                <div class="flex items-center gap-2">
                  <div class="flex-1 h-2 bg-[#262851]/10 rounded-full overflow-hidden">
                    <div class="h-full bg-[#07BBF4] rounded-full" style="width:78%"></div>
                  </div>
                  <span class="font-mono text-sm font-bold">78%</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- =================== SECTION 3: SCHEDULE =================== -->
    <section id="schedule" class="section bg-[#FFFFFF] relative overflow-hidden" data-index="3">
      <div class="inner-scroll pt-20 pb-12 px-6 md:px-10">
        <div class="ornament top-[12%] right-[5%] hidden lg:block">
          <div class="relative w-24 h-24">
            <svg width="96" height="96" viewBox="0 0 100 100" class="absolute inset-0 spin-cw-25"><circle cx="50" cy="50" r="44" fill="none" stroke="#07BBF4" stroke-width="2" stroke-dasharray="8 4"/></svg>
            <svg width="96" height="96" viewBox="0 0 100 100" class="absolute inset-0 spin-ccw-30"><polygon points="50,15 85,80 15,80" fill="none" stroke="#F34D8E" stroke-width="2"/></svg>
          </div>
        </div>
        <div class="ornament bottom-[8%] left-[4%] hidden lg:block">
          <svg width="55" height="55" viewBox="0 0 100 100" class="spin-cw-18"><circle cx="50" cy="50" r="40" fill="none" stroke="#FF4600" stroke-width="3"/><circle cx="50" cy="10" r="4" fill="#FF4600"/></svg>
        </div>

        <div class="max-w-7xl mx-auto relative z-10 w-full">
          <div class="mb-8 flex flex-col md:flex-row md:items-end md:justify-between gap-4">
            <div>
              <div class="font-mono text-xs uppercase tracking-widest text-[#07BBF4] mb-3">/03 — Weekly Schedule</div>
              <h2 class="mega-text text-4xl md:text-6xl text-[#262851]">
                Seven days,<br/><span class="text-[#07BBF4]">seven signals.</span>
              </h2>
            </div>
            <p class="text-sm text-[#262851]/70 max-w-md">
              All times JST. Streams may run late because I have no concept of time. Or boundaries. Mostly time.
            </p>
          </div>

          <div class="grid grid-cols-2 sm:grid-cols-4 lg:grid-cols-7 gap-3">
            <div class="schedule-slot bg-white border-2 border-[#262851] p-4">
              <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mb-1">Day 01</div>
              <div class="font-display text-xl mb-3">MON</div>
              <div class="space-y-2">
                <div><div class="font-mono text-[10px] text-[#F34D8E]">19:00</div><div class="font-bold text-xs">Speedrun Pt.2</div></div>
                <div><div class="font-mono text-[10px] text-[#07BBF4]">22:00</div><div class="font-bold text-xs">Late Zatsudan</div></div>
              </div>
            </div>
            <div class="schedule-slot bg-[#07BBF4] text-white p-4">
              <div class="font-mono text-[10px] uppercase tracking-widest opacity-80 mb-1">Day 02</div>
              <div class="font-display text-xl mb-3">TUE</div>
              <div class="space-y-2">
                <div><div class="font-mono text-[10px] opacity-80">20:00</div><div class="font-bold text-xs">RPG Story</div></div>
                <div><div class="font-mono text-[10px] opacity-80">23:00</div><div class="font-bold text-xs">Watchalong</div></div>
              </div>
            </div>
            <div class="schedule-slot bg-white border-2 border-[#262851] p-4">
              <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mb-1">Day 03</div>
              <div class="font-display text-xl mb-3">WED</div>
              <div class="space-y-2">
                <div><div class="font-mono text-[10px] text-[#F34D8E]">21:00</div><div class="font-bold text-xs">Karaoke Night</div></div>
              </div>
            </div>
            <div class="schedule-slot bg-[#F34D8E] text-white p-4">
              <div class="font-mono text-[10px] uppercase tracking-widest opacity-80 mb-1">Day 04</div>
              <div class="font-display text-xl mb-3">THU</div>
              <div class="space-y-2">
                <div><div class="font-mono text-[10px] opacity-80">19:00</div><div class="font-bold text-xs">Horror Game</div></div>
                <div><div class="font-mono text-[10px] opacity-80">22:00</div><div class="font-bold text-xs">Art Stream</div></div>
              </div>
            </div>
            <div class="schedule-slot bg-white border-2 border-[#262851] p-4">
              <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mb-1">Day 05</div>
              <div class="font-display text-xl mb-3">FRI</div>
              <div class="space-y-2">
                <div><div class="font-mono text-[10px] text-[#07BBF4]">20:00</div><div class="font-bold text-xs">Co-op Chaos</div></div>
              </div>
            </div>
            <div class="schedule-slot bg-[#E3FF00] p-4">
              <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/70 mb-1">Day 06</div>
              <div class="font-display text-xl mb-3 text-[#262851]">SAT</div>
              <div class="space-y-2">
                <div><div class="font-mono text-[10px] text-[#FF4600]">18:00</div><div class="font-bold text-xs text-[#262851]">Big Event</div></div>
                <div><div class="font-mono text-[10px] text-[#FF4600]">21:00</div><div class="font-bold text-xs text-[#262851]">Mbr Karaoke</div></div>
              </div>
            </div>
            <div class="schedule-slot bg-white border-2 border-[#262851] p-4">
              <div class="font-mono text-[10px] uppercase tracking-widest text-[#262851]/60 mb-1">Day 07</div>
              <div class="font-display text-xl mb-3">SUN</div>
              <div class="space-y-2">
                <div><div class="font-mono text-[10px] text-[#F34D8E]">15:00</div><div class="font-bold text-xs">Speedrun Sun</div></div>
              </div>
            </div>
          </div>

          <div class="mt-8 flex flex-wrap items-center gap-4 text-xs text-[#262851]/60 font-mono">
            <div class="flex items-center gap-2"><span class="w-3 h-3 bg-[#07BBF4] rounded-sm"></span>Gaming</div>
            <div class="flex items-center gap-2"><span class="w-3 h-3 bg-[#F34D8E] rounded-sm"></span>Music</div>
            <div class="flex items-center gap-2"><span class="w-3 h-3 bg-[#E3FF00] rounded-sm"></span>Special Event</div>
            <div class="flex items-center gap-2"><i class="fa-solid fa-circle-info text-[#FF4600]"></i>Schedule subject to change · watch socials</div>
          </div>
        </div>
      </div>
    </section>

    <!-- =================== SECTION 4: CONTENT =================== -->
    <section id="content" class="section bg-[#FFFFFF] relative overflow-hidden" data-index="4">
      <div class="inner-scroll pt-20 pb-12 px-6 md:px-10">
        <div class="ornament top-[14%] left-[5%] hidden lg:block">
          <svg width="65" height="65" viewBox="0 0 100 100" class="spin-ccw-22"><polygon points="50,12 88,80 12,80" fill="none" stroke="#FF4600" stroke-width="3"/></svg>
        </div>
        <div class="ornament bottom-[12%] right-[6%] hidden lg:block">
          <div class="relative w-24 h-24">
            <svg width="96" height="96" viewBox="0 0 100 100" class="absolute inset-0 spin-cw-40"><circle cx="50" cy="50" r="44" fill="none" stroke="#262851" stroke-width="2" stroke-dasharray="4 4"/></svg>
            <svg width="96" height="96" viewBox="0 0 100 100" class="absolute inset-0 spin-ccw-30"><circle cx="50" cy="50" r="28" fill="none" stroke="#F34D8E" stroke-width="2"/><circle cx="50" cy="22" r="3" fill="#F34D8E"/></svg>
          </div>
        </div>
        <div class="ornament top-[55%] left-[3%] hidden lg:block">
          <svg width="40" height="40" viewBox="0 0 100 100" class="spin-cw-18"><polygon points="50,15 85,80 15,80" fill="#E3FF00" opacity="0.7"/></svg>
        </div>

        <div class="max-w-7xl mx-auto relative z-10 w-full">
          <div class="mb-8">
            <div class="font-mono text-xs uppercase tracking-widest text-[#FF4600] mb-3">/04 — Content</div>
            <h2 class="mega-text text-4xl md:text-6xl text-[#262851]">
              Four channels,<br/>one <span class="text-[#FF4600]">frequency.</span>
            </h2>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <article class="content-card bg-[#07BBF4] p-6 md:p-8 min-h-[220px] flex flex-col justify-between">
              <div class="card-bg" style="background:#262851"></div>
              <div class="card-inner">
                <div class="flex items-start justify-between mb-4">
                  <div class="font-mono text-xs uppercase tracking-widest opacity-80">/ Category 01</div>
                  <i class="fa-solid fa-gamepad text-2xl"></i>
                </div>
                <div class="font-display text-3xl md:text-4xl mb-2">Gaming</div>
                <p class="opacity-90 max-w-md text-sm">Speedruns, horror, RPG deep-dives, and the occasional rage-quit. I play everything that has a start screen.</p>
              </div>
              <div class="card-inner flex items-center justify-between mt-4 pt-4 border-t border-current/20">
                <span class="font-mono text-xs uppercase tracking-wider">218 streams</span>
                <span class="font-mono text-xs uppercase tracking-wider">View Library <i class="fa-solid fa-arrow-right ml-1"></i></span>
              </div>
            </article>

            <article class="content-card bg-[#F34D8E] p-6 md:p-8 min-h-[220px] flex flex-col justify-between">
              <div class="card-bg" style="background:#262851"></div>
              <div class="card-inner">
                <div class="flex items-start justify-between mb-4">
                  <div class="font-mono text-xs uppercase tracking-widest opacity-80">/ Category 02</div>
                  <i class="fa-solid fa-music text-2xl"></i>
                </div>
                <div class="font-display text-3xl md:text-4xl mb-2">Music</div>
                <p class="opacity-90 max-w-md text-sm">Weekly karaoke, original songs, covers of tracks you forgot existed. From city pop to math rock.</p>
              </div>
              <div class="card-inner flex items-center justify-between mt-4 pt-4 border-t border-current/20">
                <span class="font-mono text-xs uppercase tracking-wider">42 covers · 3 originals</span>
                <span class="font-mono text-xs uppercase tracking-wider">Listen <i class="fa-solid fa-arrow-right ml-1"></i></span>
              </div>
            </article>

            <article class="content-card bg-[#E3FF00] p-6 md:p-8 min-h-[220px] flex flex-col justify-between">
              <div class="card-bg" style="background:#FF4600"></div>
              <div class="card-inner">
                <div class="flex items-start justify-between mb-4">
                  <div class="font-mono text-xs uppercase tracking-widest opacity-70">/ Category 03</div>
                  <i class="fa-solid fa-paintbrush text-2xl"></i>
                </div>
                <div class="font-display text-3xl md:text-4xl mb-2">Art</div>
                <p class="opacity-90 max-w-md text-sm">Live drawing streams, model rigging experiments, and fan-art showcases. Sometimes I make a mess.</p>
              </div>
              <div class="card-inner flex items-center justify-between mt-4 pt-4 border-t border-current/30">
                <span class="font-mono text-xs uppercase tracking-wider">67 sessions</span>
                <span class="font-mono text-xs uppercase tracking-wider">Browse Gallery <i class="fa-solid fa-arrow-right ml-1"></i></span>
              </div>
            </article>

            <article class="content-card bg-[#262851] text-white p-6 md:p-8 min-h-[220px] flex flex-col justify-between">
              <div class="card-bg" style="background:#07BBF4"></div>
              <div class="card-inner">
                <div class="flex items-start justify-between mb-4">
                  <div class="font-mono text-xs uppercase tracking-widest opacity-80">/ Category 04</div>
                  <i class="fa-solid fa-comments text-2xl"></i>
                </div>
                <div class="font-display text-3xl md:text-4xl mb-2">Chat</div>
                <p class="opacity-90 max-w-md text-sm">Zatsudan streams, watchalongs, ASMR experiments, and unscripted conversations about nothing in particular.</p>
              </div>
              <div class="card-inner flex items-center justify-between mt-4 pt-4 border-t border-current/20">
                <span class="font-mono text-xs uppercase tracking-wider">341 episodes</span>
                <span class="font-mono text-xs uppercase tracking-wider">Tune In <i class="fa-solid fa-arrow-right ml-1"></i></span>
              </div>
            </article>
          </div>
        </div>
      </div>
    </section>

    <!-- =================== SECTION 5: CONNECT =================== -->
    <section id="connect" class="section bg-[#FFFFFF] relative overflow-hidden" data-index="5">
      <div class="inner-scroll pt-20 pb-10 px-6 md:px-10">
        <div class="ornament top-[10%] right-[6%] hidden lg:block">
          <div class="relative w-24 h-24">
            <svg width="96" height="96" viewBox="0 0 100 100" class="absolute inset-0 spin-cw-25"><circle cx="50" cy="50" r="44" fill="none" stroke="#07BBF4" stroke-width="2" stroke-dasharray="6 6"/></svg>
            <svg width="96" height="96" viewBox="0 0 100 100" class="absolute inset-0 spin-ccw-22"><polygon points="50,15 85,80 15,80" fill="none" stroke="#F34D8E" stroke-width="2"/></svg>
          </div>
        </div>
        <div class="ornament bottom-[16%] left-[5%] hidden lg:block">
          <svg width="55" height="55" viewBox="0 0 100 100" class="spin-cw-40"><circle cx="50" cy="50" r="40" fill="none" stroke="#FF4600" stroke-width="3" stroke-dasharray="10 5"/></svg>
        </div>

        <div class="max-w-7xl mx-auto relative z-10 w-full">
          <div class="grid grid-cols-1 lg:grid-cols-12 gap-8 mb-8">
            <div class="lg:col-span-7">
              <div class="font-mono text-xs uppercase tracking-widest text-[#07BBF4] mb-3">/05 — Connect</div>
              <h2 class="mega-text text-4xl md:text-6xl text-[#262851]">
                Join the<br/><span class="text-[#07BBF4]">constellation.</span>
              </h2>
            </div>
            <div class="lg:col-span-5 lg:pt-10">
              <p class="text-base text-[#262851]/80 leading-relaxed">
                Pick your platform of choice. I'm everywhere — like a digital ghost with better hair. Drop a follow, leave a comment, or just lurk quietly. All welcome.
              </p>
            </div>
          </div>

          <div class="grid grid-cols-2 lg:grid-cols-4 gap-3 mb-6">
            <a href="http://www.twitch.com/chuchu_programming" class="social-link group bg-white border-2 border-[#262851] p-4 block min-h-[150px]">
              <div class="link-bg" style="background:#40286d"></div>
              <div class="link-content flex flex-col h-full">
                <i class="fa-brands fa-twitch text-2xl mb-4 text-[#9146FF]"></i>
                <div class="font-mono text-[10px] uppercase tracking-widest opacity-60 mb-1">Twitch</div>
                <div  class="font-display text-lg mb-1">/chuchu_programming</div>
                <div class="font-mono text-[10px] opacity-60 mt-auto pt-4">847K followers</div>
              </div>
            </a>
            <a href="http://www.youtube.com/@chuchu_programming" class="social-link group bg-white border-2 border-[#262851] p-4 block min-h-[150px]">
              <div class="link-bg" style="background:#75020a"></div>
              <div class="link-content flex flex-col h-full">
                <i class="fa-brands fa-youtube text-2xl mb-4 text-[#FF0000]"></i>
                <div class="font-mono text-[10px] uppercase tracking-widest opacity-60 mb-1">YouTube</div>
                <div class="font-display text-lg mb-1">@chuchu_programming</div>
                <div class="font-mono text-[10px] opacity-60 mt-auto pt-4">1.2M subscribers</div>
              </div>
            </a>
            <a href="http://x.com/chuchu_programming" class="social-link group bg-white border-2 border-[#262851] p-4 block min-h-[150px]">
              <div class="link-bg" style="background:#3a3a3a"></div>
              <div class="link-content flex flex-col h-full">
                <i class="fa-brands fa-x-twitter text-2xl mb-4 text-[#262851]"></i>
                <div class="font-mono text-[10px] uppercase tracking-widest opacity-60 mb-1">Twitter / X</div>
                <div class="font-display text-lg mb-1">@chuchu_programming</div>
                <div class="font-mono text-[10px] opacity-60 mt-auto pt-4">389K followers</div>
              </div>
            </a>
            <a href="#" class="social-link group bg-white border-2 border-[#262851] p-4 block min-h-[150px]">
              <div class="link-bg" style="background:#262851"></div>
              <div class="link-content flex flex-col h-full">
                <i class="fa-brands fa-discord text-2xl mb-4 text-[#5865F2]"></i>
                <div class="font-mono text-[10px] uppercase tracking-widest opacity-60 mb-1">Discord</div>
                <div class="font-display text-lg mb-1">Constellation</div>
                <div class="font-mono text-[10px] opacity-60 mt-auto pt-4">24K members</div>
              </div>
            </a>
          </div>

          <div class="bg-[#262851] text-white p-6 md:p-8 relative overflow-hidden rounded-lg-card">
            <div class="absolute top-0 right-0 w-24 h-24 bg-[#E3FF00] rounded-full -translate-y-1/2 translate-x-1/2 opacity-90"></div>
            <div class="absolute bottom-0 left-0 w-20 h-20 border-4 border-[#F34D8E] translate-y-1/2 -translate-x-1/2 rounded-sm"></div>
            <div class="relative grid grid-cols-1 md:grid-cols-3 gap-6 items-center">
              <div class="md:col-span-2">
                <div class="font-mono text-xs uppercase tracking-widest text-[#E3FF00] mb-2">★ Membership Open</div>
                <h3 class="mega-text text-2xl md:text-3xl mb-2">Become a Star in the Constellation.</h3>
                <p class="text-white/70 max-w-xl text-sm">Members get exclusive emotes, behind-the-scenes content, monthly karaoke, and the warm fuzzy feeling of supporting an independent creator.</p>
              </div>
              <div class="md:text-right">
                <button class="cta-btn group inline-flex items-center gap-3 bg-[#F34D8E] text-white px-6 py-3 overflow-hidden relative hover:opacity-95">
                  <span class="cta-fill" style="background:#07BBF4"></span>
                  <span class="relative font-mono text-sm uppercase tracking-wider z-10">Join Now</span>
                  <i class="fa-solid fa-arrow-right relative z-10"></i>
                </button>
                <div class="mt-2 font-mono text-[10px] uppercase tracking-widest text-white/50">from $4.99/mo</div>
              </div>
            </div>
          </div>

          <footer class="mt-8 pt-6 border-t-2 border-[#262851]">
            <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
              <div class="font-mono text-xs text-[#262851]/50">
                © 2026 118R · All signals reserved · Made with code & coffee
              </div>
              <div class="flex items-center gap-4 font-mono text-xs text-[#262851]/50">
                <span>v2.4.7</span>
                <span>·</span>
                <span>JST <span data-clock>--:--</span></span>
                <span>·</span>
                <span class="text-[#FF4600]">● STABLE</span>
              </div>
            </div>
          </footer>
        </div>
      </div>
    </section>

  </div>

  <!-- WIPE OVERLAY -->
  <div id="wipe-overlay">
    <div class="wipe-rect wipe-rect-1"></div>
    <div class="wipe-rect wipe-rect-2"></div>
    <div class="wipe-rect wipe-rect-3"></div>
  </div>

  <script>
    // ============================================
    // SCROLL-DRIVEN HIDDEN TRANSITION
    // ============================================
    const NUM_SECTIONS = 6;
    const sections = Array.from(document.querySelectorAll('.section'));
    const progressBar = document.getElementById('scroll-progress');
    const wipeOverlay = document.getElementById('wipe-overlay');
    const wipeRects = Array.from(wipeOverlay.querySelectorAll('.wipe-rect'));
    const navDots = Array.from(document.querySelectorAll('.nav-dot'));
    const navLinks = Array.from(document.querySelectorAll('.nav-link'));

    // Set body height to create scrollable space for 5 transitions
    document.body.style.height = (NUM_SECTIONS * 100) + 'vh';

    let currentVisible = 0;

    function updateWipe(progress) {
      // progress: 0 (hidden) -> 0.5 (covered) -> 1 (passed)
      // We want: 0 -> -100%, 0.5 -> 0%, 1 -> +100%
      const p = Math.max(0, Math.min(1, progress));

      // Hide overlay entirely when not transitioning
      if (p <= 0.001 || p >= 0.999) {
        wipeOverlay.style.opacity = '0';
      } else {
        wipeOverlay.style.opacity = '1';
      }

      // Stagger bands slightly for layered feel
      wipeRects.forEach((r, i) => {
        const stagger = (i - 1) * 0.06;
        const tp = Math.max(0, Math.min(1, p + stagger));
        const t = (-1 + tp * 2) * 200;
        r.style.transform = `rotate(-15deg) translateX(${t}%)`;
      });
    }

    function setActiveSection(idx) {
      if (idx === currentVisible) return;
      currentVisible = idx;
      sections.forEach((s, i) => s.classList.toggle('active', i === idx));
      navDots.forEach((d, i) => d.classList.toggle('active', i === idx));
      navLinks.forEach((l, i) => l.style.color = (i === idx - 1) ? 'var(--pink)' : '');
    }

    function onScroll() {
      const scrollY = window.scrollY;
      const vh = window.innerHeight;
      const totalScrollable = (NUM_SECTIONS - 1) * vh;
      const globalProgress = Math.max(0, Math.min(1, scrollY / totalScrollable));

      progressBar.style.transform = `scaleX(${globalProgress})`;

      const sectionFloat = scrollY / vh;

      // Determine current section
      const currentIdx = Math.round(sectionFloat);
      setActiveSection(Math.max(0, Math.min(NUM_SECTIONS - 1, currentIdx)));

      // Determine transition progress
      const transitionStart = Math.floor(sectionFloat) * vh;
      const transitionProgress = (scrollY - transitionStart) / vh;

      // If we're exactly on a section (no transition), hide wipe
      if (transitionProgress <= 0 || transitionProgress >= 1) {
        updateWipe(0);
      } else {
        updateWipe(transitionProgress);
      }
    }

    let rafId = null;
    function scheduleUpdate() {
      if (rafId) return;
      rafId = requestAnimationFrame(() => {
        rafId = null;
        onScroll();
      });
    }

    window.addEventListener('scroll', scheduleUpdate, { passive: true });
    window.addEventListener('resize', scheduleUpdate);

    // Initial paint
    onScroll();

    // === NAVIGATION ===
    function jumpToSection(idx) {
      idx = Math.max(0, Math.min(NUM_SECTIONS - 1, idx));
      window.scrollTo({ top: idx * window.innerHeight, behavior: 'smooth' });
    }

    document.querySelectorAll('[data-jump]').forEach(el => {
      el.addEventListener('click', (e) => {
        e.preventDefault();
        jumpToSection(parseInt(el.dataset.jump, 10));
      });
    });

    // Keyboard nav
    document.addEventListener('keydown', (e) => {
      if (e.target.matches('input, textarea')) return;
      if (e.key === 'ArrowDown' || e.key === 'PageDown') {
        e.preventDefault();
        jumpToSection(currentVisible + 1);
      } else if (e.key === 'ArrowUp' || e.key === 'PageUp') {
        e.preventDefault();
        jumpToSection(currentVisible - 1);
      }
    });

    // Touch swipe
    let touchStartY = null;
    window.addEventListener('touchstart', (e) => {
      touchStartY = e.touches[0].clientY;
    }, { passive: true });
    window.addEventListener('touchend', (e) => {
      if (touchStartY === null) return;
      const dy = e.changedTouches[0].clientY - touchStartY;
      if (Math.abs(dy) > 80) {
        if (dy < 0) jumpToSection(currentVisible + 1);
        else jumpToSection(currentVisible - 1);
      }
      touchStartY = null;
    }, { passive: true });

    // === ANIMATED COUNTERS ===
    const counters = document.querySelectorAll('[data-counter]');
    const counterObserver = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const el = entry.target;
          const target = parseFloat(el.dataset.counter);
          const suffix = el.dataset.suffix || '';
          const isDecimal = target % 1 !== 0;
          const duration = 1800;
          const start = performance.now();

          function tick(now) {
            const elapsed = now - start;
            const progress = Math.min(elapsed / duration, 1);
            const eased = 1 - Math.pow(1 - progress, 3);
            const value = target * eased;
            el.textContent = (isDecimal ? value.toFixed(1) : Math.floor(value).toLocaleString()) + suffix;
            if (progress < 1) requestAnimationFrame(tick);
          }
          requestAnimationFrame(tick);
          counterObserver.unobserve(el);
        }
      });
    }, { threshold: 0.4 });
    counters.forEach(c => counterObserver.observe(c));

    // === LIVE CLOCK (JST) ===
    const clockEl = document.querySelector('[data-clock]');
    function updateClock() {
      if (!clockEl) return;
      const now = new Date();
      const jst = new Date(now.getTime() + (now.getTimezoneOffset() + 540) * 60000);
      const hh = String(jst.getHours()).padStart(2, '0');
      const mm = String(jst.getMinutes()).padStart(2, '0');
      clockEl.textContent = `${hh}:${mm}`;
    }
    updateClock();
    setInterval(updateClock, 30000);
  </script>
</body>
</html>
