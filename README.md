<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Liberação Mod Menu — @NAUTYSX</title>
  <style>
    :root{
      --bg-dark:#000000;
      --accent:#00ff99;
      --muted:#cfeee0;
      --red-1: #ff4b4b;
      --red-2: #d92b2b;
      --green-1: #37d06f;
      --green-2: #28b05a;
    }

    html,body{height:100%;margin:0;font-family:Inter,system-ui,-apple-system,"Segoe UI",Roboto,Arial;background:var(--bg-dark);color:var(--muted);-webkit-font-smoothing:antialiased}
    canvas#network{position:fixed;inset:0;width:100%;height:100%;z-index:0;display:block;pointer-events:none}

    .center{position:relative;z-index:10;min-height:100vh;display:flex;align-items:center;justify-content:center;padding:24px}
    .card{width:360px;max-width:94vw;border-radius:14px;padding:20px;box-sizing:border-box;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.02);box-shadow:0 12px 36px rgba(0,0,0,0.6);text-align:center}

    .title{font-weight:900;font-size:20px;color:var(--accent);margin:0 0 8px 0;letter-spacing:0.6px}
    .big{font-weight:800;font-size: clamp(20px,5.5vw,38px);margin:8px 0 6px;text-transform:uppercase}
    .sub{margin:0 0 16px;color:rgba(255,255,255,0.65);font-size:13px}

    .controls{display:flex;gap:12px;justify-content:center;align-items:center;margin-top:6px}
    .btn{padding:12px 16px;border-radius:12px;font-weight:800;cursor:pointer;border:1px solid rgba(255,255,255,0.04);background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));color:inherit;transition:transform 160ms,box-shadow 160ms,opacity 160ms;display:inline-flex;gap:10px;align-items:center;justify-content:center}
    .btn:active{transform:translateY(2px)}
    .btn.disabled{opacity:0.6;cursor:not-allowed}

    /* Lock button default: red closed */
    .btn.lock.red{ background: linear-gradient(180deg, var(--red-1), var(--red-2)); border:1px solid rgba(255,80,80,0.18); color:#fff; box-shadow: 0 10px 30px rgba(217,43,43,0.12); }
    /* When unlocked -> green */
    .btn.lock.unlocked{ background: linear-gradient(180deg, var(--green-1), var(--green-2)); border:1px solid rgba(40,176,90,0.18); color:#fff; box-shadow: 0 12px 34px rgba(40,176,90,0.12); }

    /* SVG icon sizing inside button */
    .icon{width:20px;height:20px;flex:0 0 20px;display:inline-block}

    /* Lock SVG: shackle will animate */
    .shackle{ transform-box: fill-box; transform-origin: 50% 46%; transition: transform 120ms ease; }
    /* final open position when unlocked (applied immediately if 'unlocked' present) */
    .btn.lock.unlocked .shackle{ transform: rotate(-40deg) translateY(-6px) translateX(-2px); }

    /* animation that runs only when 'animate' class present */
    @keyframes lockOpenAnim {
      0% { transform: rotate(0deg) translateY(0) translateX(0); }
      45% { transform: rotate(-50deg) translateY(-10px) translateX(-6px); }
      70% { transform: rotate(-30deg) translateY(-6px) translateX(-2px); }
      100% { transform: rotate(-40deg) translateY(-6px) translateX(-2px); }
    }
    .btn.lock.unlocked.animate .shackle {
      animation: lockOpenAnim 700ms cubic-bezier(.2,.9,.2,1) forwards;
    }

    /* small bounce of lock body when opening */
    @keyframes bodyBounce {
      0% { transform: translateY(0) scale(1); }
      40% { transform: translateY(-4px) scale(1.01); }
      100% { transform: translateY(0) scale(1); }
    }
    .btn.lock.unlocked.animate .lock-body {
      animation: bodyBounce 560ms cubic-bezier(.2,.9,.2,1);
    }

    .state{margin-top:12px;font-size:13px;color:rgba(255,255,255,0.7)}

    /* notice and quick links (same as before, animated reveal) */
    .notice{position:fixed;left:50%;transform:translateX(-50%) translateY(18px) scale(0.98);z-index:24;background:linear-gradient(180deg, rgba(3,20,10,0.92), rgba(3,20,10,0.85));color:#eafff3;padding:14px 18px;border-radius:12px;border:1px solid rgba(0,255,153,0.12);box-shadow:0 12px 40px rgba(0,0,0,0.6);opacity:0;pointer-events:none;width:min(780px, calc(100% - 48px));display:flex;flex-direction:column;gap:10px;align-items:center}
    .notice.show{animation:noticeIn 600ms cubic-bezier(.2,.9,.2,1) forwards;pointer-events:auto}
    @keyframes noticeIn{0%{transform:translateX(-50%) translateY(18px) scale(0.98);opacity:0}60%{transform:translateX(-50%) translateY(-6px) scale(1.02);opacity:1}100%{transform:translateX(-50%) translateY(0) scale(1);opacity:1}}
    .links{display:flex;gap:12px;justify-content:center;align-items:center;flex-wrap:wrap}
    .link-btn{padding:10px 14px;border-radius:10px;background:rgba(255,255,255,0.02);color:inherit;font-weight:800;border:1px solid rgba(255,255,255,0.04);text-decoration:none;transform:translateY(10px) scale(0.98);opacity:0;transition:transform 360ms cubic-bezier(.2,.9,.2,1),opacity 360ms;display:inline-flex;gap:8px;align-items:center}
    .link-btn.show{opacity:1;transform:translateY(0) scale(1)}
    .quick-links{margin-top:16px;display:flex;gap:10px;justify-content:center;align-items:center;visibility:hidden;opacity:0;transform:translateY(8px);transition:opacity 320ms,transform 320ms,visibility 0s 320ms}
    .quick-links.visible{visibility:visible;opacity:1;transform:translateY(0)}
    .quick-btn{padding:8px 12px;border-radius:10px;background:rgba(255,255,255,0.02);border:1px solid rgba(255,255,255,0.04);color:inherit;font-weight:800;text-decoration:none;display:inline-flex;gap:8px;align-items:center}

    @media (max-width:820px){.notice{left:12px;transform:none;width:calc(100% - 24px)}}
  </style>
</head>
<body>
  <canvas id="network" aria-hidden="true"></canvas>

  <main class="center" role="main">
    <div class="card" role="region" aria-label="Liberar mod">
      <div class="title">@NAUTYSX</div>
      <div class="big" id="mainBig">MOD MENU</div>
      <div class="sub" id="mainSub">Clique para liberar por 24 horas</div>

      <div class="controls" role="group" aria-label="Ações">
        <!-- Lock button: starts red closed, becomes green unlocked -->
        <button id="unlockBtn" class="btn lock red" type="button" aria-pressed="false" aria-label="Liberar mod">
          <!-- SVG lock: shackle (id=shackle) and body (class=lock-body) -->
          <svg class="icon" viewBox="0 0 24 24" fill="none" aria-hidden="true" focusable="false">
            <!-- shackle -->
            <g id="shackle" class="shackle" stroke="currentColor" stroke-width="1.4" stroke-linecap="round" stroke-linejoin="round" fill="none">
              <path d="M7 10V7a5 5 0 0110 0v3"></path>
            </g>
            <!-- body -->
            <rect class="lock-body" x="4" y="10" width="16" height="10" rx="2" stroke="currentColor" stroke-width="1.4" fill="none"></rect>
          </svg>
          <span id="unlockLabel">Liberar Mod</span>
        </button>
      </div>

      <div class="state" id="stateLine">Estado: <strong id="stateText">Bloqueado</strong></div>

      <div id="quickLinks" class="quick-links" aria-hidden="true">
        <a id="quickShop" class="quick-btn" href="https://lojinhahirnaises.mginex.site/" target="_blank" rel="noopener noreferrer">🛍️ Lojinha HG</a>
        <a id="quickDisc" class="quick-btn" href="https://discord.gg/38Spd2NH" target="_blank" rel="noopener noreferrer">💬 Discord</a>
        <a id="quickYT" class="quick-btn" href="https://youtube.com/@texltbff?si=mzV31xecLuirKzgi" target="_blank" rel="noopener noreferrer">▶️ YouTube</a>
      </div>
    </div>
  </main>

  <div id="notice" class="notice" role="status" aria-live="polite" aria-atomic="true">
    <div class="msg" id="noticeMsg">MOD MENU @NAUTYSX LIBERADO POR 24 HORAS — APROVEITE</div>
    <div class="details">Abaixo os links oficiais — obrigado pelo apoio!</div>
    <div class="links" id="linksArea" aria-hidden="true">
      <a id="linkShop" class="link-btn" href="https://lojinhahirnaises.mginex.site/" target="_blank" rel="noopener noreferrer">
        🛍️ Nossa Lojinha HG
      </a>
      <a id="linkDiscord" class="link-btn" href="https://discord.gg/38Spd2NH" target="_blank" rel="noopener noreferrer">
        💬 Nosso Discord
      </a>
      <a id="linkYouTube" class="link-btn" href="https://youtube.com/@texltbff?si=mzV31xecLuirKzgi" target="_blank" rel="noopener noreferrer">
        ▶️ Nosso YouTube
      </a>
    </div>
  </div>

  <script>
    // Background (connected nodes) - minimal
    (function networkCanvas(){
      const canvas = document.getElementById('network');
      const ctx = canvas.getContext('2d');
      const DPR = Math.max(1, window.devicePixelRatio || 1);
      let W = innerWidth, H = innerHeight;
      function resize(){
        W = innerWidth; H = innerHeight;
        canvas.width = W * DPR; canvas.height = H * DPR;
        canvas.style.width = W + 'px'; canvas.style.height = H + 'px';
        ctx.setTransform(DPR,0,0,DPR,0,0);
        initParticles();
      }
      addEventListener('resize', resize);

      const cfg = { speed: 0.28, connectDist: 120, baseCount: 80 };
      let particles = [];
      function rand(a,b){ return Math.random()*(b-a)+a; }
      function initParticles(){
        particles = [];
        const count = Math.max(28, Math.round(cfg.baseCount * Math.min(1, (W*H)/(1280*720))));
        for (let i=0;i<count;i++){
          particles.push({ x: rand(0,W), y: rand(0,H), vx: rand(-cfg.speed,cfg.speed), vy: rand(-cfg.speed,cfg.speed), r: rand(1.1,2.6), phase: Math.random()*Math.PI*2 });
        }
      }
      initParticles();

      function step(){
        ctx.clearRect(0,0,W,H);
        ctx.fillStyle = 'rgba(0,0,0,0.06)';
        ctx.fillRect(0,0,W,H);
        for (let i=0;i<particles.length;i++){
          const a = particles[i];
          for (let j=i+1;j<particles.length;j++){
            const b = particles[j];
            const dx = a.x - b.x, dy = a.y - b.y;
            const d2 = dx*dx+dy*dy;
            if (d2 < cfg.connectDist*cfg.connectDist){
              const d = Math.sqrt(d2);
              const alpha = 1 - d/cfg.connectDist;
              ctx.strokeStyle = `rgba(0,255,153,${0.06 * alpha})`;
              ctx.lineWidth = 1 * alpha;
              ctx.beginPath(); ctx.moveTo(a.x,a.y); ctx.lineTo(b.x,b.y); ctx.stroke();
            }
          }
        }
        for (let p of particles){
          p.x += p.vx; p.y += p.vy; p.phase += 0.01;
          if (p.x < 0 || p.x > W) p.vx *= -1;
          if (p.y < 0 || p.y > H) p.vy *= -1;
          ctx.beginPath();
          ctx.fillStyle = 'rgba(0,255,153,0.9)';
          ctx.arc(p.x, p.y, p.r + Math.sin(p.phase)*0.18, 0, Math.PI*2);
          ctx.fill();
        }
        requestAnimationFrame(step);
      }
      requestAnimationFrame(step);
    })();

    // Unlock button behavior with animated lock opening
    (function lockButton(){
      const UNLOCK_KEY = 'mod_exp_ts_v3';
      const btn = document.getElementById('unlockBtn');
      const label = document.getElementById('unlockLabel');
      const stateText = document.getElementById('stateText');
      const mainBig = document.getElementById('mainBig');
      const mainSub = document.getElementById('mainSub');
      const notice = document.getElementById('notice');
      const linksArea = document.getElementById('linksArea');

      function now(){ return Date.now(); }
      function setExp(ts){ try { localStorage.setItem(UNLOCK_KEY, String(ts)); } catch(e) {} }
      function getExp(){ try { return Number(localStorage.getItem(UNLOCK_KEY)) || 0; } catch(e){ return 0; } }
      function clearExp(){ try { localStorage.removeItem(UNLOCK_KEY); } catch(e) {} }

      function activate(expTs, animate=false){
        const expDate = new Date(expTs);
        stateText.textContent = 'Ativo';
        mainBig.textContent = 'MOD ATIVO';
        mainSub.textContent = 'Disponível até ' + expDate.toLocaleString();
        // update button styles: remove red, add unlocked (green)
        btn.classList.remove('red');
        btn.classList.add('unlocked');
        btn.setAttribute('aria-pressed','true');
        label.textContent = 'Liberado';
        // play open animation only when requested
        if (animate) {
          btn.classList.add('animate');
          // remove animate class after animation ends to keep final state
          setTimeout(()=> btn.classList.remove('animate'), 800);
        }
        // show notice and quick links
        showNotice();
        document.getElementById('quickLinks').classList.add('visible');
      }

      function deactivate(){
        stateText.textContent = 'Bloqueado';
        mainBig.textContent = 'MOD MENU';
        mainSub.textContent = 'Clique para liberar por 24 horas';
        btn.classList.remove('unlocked','animate');
        btn.classList.add('red');
        btn.setAttribute('aria-pressed','false');
        label.textContent = 'Liberar Mod';
        hideNotice();
        document.getElementById('quickLinks').classList.remove('visible');
      }

      function showNotice(){
        notice.classList.add('show');
        const children = Array.from(linksArea.querySelectorAll('.link-btn'));
        children.forEach((el,i)=> setTimeout(()=> el.classList.add('show'), i*120));
        // after 9s, hide the notice (links remain available in quick links)
        setTimeout(()=> {
          notice.classList.remove('show');
          children.forEach(el=> el.classList.remove('show'));
        }, 9000);
      }

      function hideNotice(){
        notice.classList.remove('show');
        linksArea.querySelectorAll('.link-btn').forEach(el=> el.classList.remove('show'));
      }

      // init from storage
      (function init(){
        const exp = getExp();
        if (exp && exp > now()){
          // already active -> show unlocked state (no animation)
          activate(exp, false);
          btn.disabled = true;
          btn.classList.add('disabled');
        } else {
          clearExp();
          deactivate();
          btn.disabled = false;
          btn.classList.remove('disabled');
        }
      })();

      // click -> set expiration 24h, animate lock opening
      btn.addEventListener('click', ()=>{
        const expiration = now() + 24*60*60*1000;
        setExp(expiration);
        activate(expiration, true);
        // disable button after unlocking
        btn.disabled = true;
        btn.classList.add('disabled');
      });

      // expire handling (update remaining time)
      setInterval(()=>{
        const exp = getExp();
        if (exp && exp > now()){
          const rem = exp - now();
          const mins = Math.floor(rem / 60000);
          const secs = Math.floor((rem % 60000) / 1000);
          mainSub.textContent = `Tempo restante: ${mins}m ${secs}s`;
          stateText.textContent = 'Ativo';
        } else {
          // expired -> reset UI
          clearExp();
          deactivate();
          btn.disabled = false;
          btn.classList.remove('disabled');
        }
      }, 1000);
    })();

    // small accessibility: allow Enter on button
    document.getElementById('unlockBtn').addEventListener('keydown', (e) => {
      if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        e.target.click();
      }
    });
  </script>
</body>
</html>
