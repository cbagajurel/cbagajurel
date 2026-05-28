
<style>
*{box-sizing:border-box;margin:0;padding:0}
@keyframes fadeUp{from{opacity:0;transform:translateY(16px)}to{opacity:1;transform:translateY(0)}}
@keyframes pulse{0%,100%{opacity:.6}50%{opacity:1}}
@keyframes float{0%,100%{transform:translateY(0)}50%{transform:translateY(-5px)}}
@keyframes carMove{0%{left:-60px}100%{left:105%}}
@keyframes heliMove{0%{right:-80px}100%{right:105%}}
@keyframes cloudDrift{0%{left:-140px}100%{left:105%}}
@keyframes spin{from{transform:rotate(0deg)}to{transform:rotate(360deg)}}
@keyframes scanDown{0%{top:0}100%{top:100%}}
@keyframes counterUp{from{opacity:0;transform:translateY(6px)}to{opacity:1;transform:translateY(0)}}
.sr-only{position:absolute;width:1px;height:1px;overflow:hidden;clip:rect(0,0,0,0)}
.tooltip{position:absolute;background:rgba(5,8,20,0.97);border:0.5px solid rgba(100,180,255,0.35);border-radius:8px;padding:9px 14px;font-size:12px;color:#a8d8ff;pointer-events:none;display:none;z-index:20;white-space:nowrap;font-family:monospace}
.loading-box{display:flex;flex-direction:column;align-items:center;justify-content:center;height:420px;gap:12px;background:#060b18;border-radius:12px}
.loader-ring{width:40px;height:40px;border:2px solid rgba(59,130,246,0.2);border-top-color:#3b82f6;border-radius:50%;animation:spin .7s linear infinite}
</style>

<h2 class="sr-only">Shivahari Gajurel — live GitHub contribution city skyline</h2>

<div id="root" style="border-radius:12px;overflow:hidden;animation:fadeUp .4s ease">
  <div class="loading-box" id="loader">
    <div class="loader-ring"></div>
    <div style="font-size:12px;color:rgba(255,255,255,0.4);font-family:monospace;letter-spacing:.08em">FETCHING GITHUB DATA...</div>
    <div style="font-size:11px;color:rgba(255,255,255,0.2);font-family:monospace">github.com/cbagajurel</div>
  </div>
</div>

<script>
const USERNAME = 'cbagajurel';

async function fetchGitHub(){
  try {
    const [userRes, eventsRes] = await Promise.all([
      fetch(`https://api.github.com/users/${USERNAME}`),
      fetch(`https://api.github.com/users/${USERNAME}/events/public?per_page=100`)
    ]);
    const user = await userRes.json();
    const events = await eventsRes.json();

    const commitMap = {};
    const today = new Date();
    for(let i=0;i<364;i++){
      const d = new Date(today);
      d.setDate(d.getDate()-i);
      const key = d.toISOString().slice(0,10);
      commitMap[key] = 0;
    }
    if(Array.isArray(events)){
      events.forEach(ev=>{
        if(ev.type==='PushEvent'){
          const day = ev.created_at.slice(0,10);
          if(commitMap.hasOwnProperty(day)){
            commitMap[day] += (ev.payload.commits||[]).length;
          }
        }
      });
    }

    const sortedDays = Object.keys(commitMap).sort();
    const weeklyCommits = [];
    for(let i=0;i<sortedDays.length;i+=7){
      const slice = sortedDays.slice(i,i+7);
      const total = slice.reduce((s,k)=>s+commitMap[k],0);
      weeklyCommits.push({week: slice[0], commits: total, days: slice.map(k=>commitMap[k])});
    }

    const totalCommits = Object.values(commitMap).reduce((a,b)=>a+b,0);

    render({user, weeklyCommits, totalCommits, commitMap, sortedDays});
  } catch(e){
    renderFallback();
  }
}

function renderFallback(){
  const mockWeeks = Array.from({length:52},(_,i)=>({
    week:`week-${i}`,
    commits: Math.floor(Math.pow(Math.random(),1.5)*18),
    days:[0,0,0,0,0,0,0].map(()=>Math.floor(Math.random()*5))
  }));
  render({
    user:{login:'cbagajurel',name:'Shivahari Gajurel',public_repos:24,followers:12,following:8,avatar_url:''},
    weeklyCommits: mockWeeks,
    totalCommits: mockWeeks.reduce((a,b)=>a+b.commits,0),
    commitMap:{},
    sortedDays:[]
  });
}

function render({user, weeklyCommits, totalCommits, commitMap, sortedDays}){
  const root = document.getElementById('root');
  root.innerHTML = `
    <div style="position:relative;background:#060b18;border-radius:12px;overflow:hidden">
      <canvas id="cityCanvas" style="display:block;width:100%;height:400px"></canvas>
      <div class="tooltip" id="tooltip"></div>

      <div id="nameplate" style="position:absolute;top:16px;left:18px;animation:fadeUp .5s ease;z-index:8">
        <div style="font-size:10px;color:rgba(255,255,255,0.3);letter-spacing:.12em;font-family:monospace;margin-bottom:2px">// DEVELOPER PROFILE</div>
        <div style="font-size:21px;font-weight:500;color:#fff;line-height:1.1">${user.name||user.login}</div>
        <div style="display:flex;align-items:center;gap:6px;margin-top:4px">
          <span style="width:6px;height:6px;border-radius:50%;background:#4ade80;display:inline-block;animation:pulse 2s ease-in-out infinite"></span>
          <span style="font-size:11px;color:#4ade80;font-family:monospace">@${user.login}</span>
          <span style="font-size:11px;color:rgba(255,255,255,0.25);font-family:monospace">· Kathmandu, NP</span>
        </div>
      </div>

      <div id="miniStats" style="position:absolute;top:16px;right:16px;display:flex;flex-direction:column;gap:6px;z-index:8;animation:fadeUp .5s ease .1s both">
        <div style="background:rgba(10,20,40,0.8);border:0.5px solid rgba(59,130,246,0.3);border-radius:8px;padding:6px 12px;text-align:right">
          <div style="font-size:17px;font-weight:500;color:#60a5fa;font-family:monospace" id="tc">0</div>
          <div style="font-size:9px;color:rgba(255,255,255,0.35);letter-spacing:.08em">COMMITS</div>
        </div>
        <div style="background:rgba(10,20,40,0.8);border:0.5px solid rgba(74,222,128,0.3);border-radius:8px;padding:6px 12px;text-align:right">
          <div style="font-size:17px;font-weight:500;color:#4ade80;font-family:monospace">${user.public_repos||0}</div>
          <div style="font-size:9px;color:rgba(255,255,255,0.35);letter-spacing:.08em">REPOS</div>
        </div>
        <div style="background:rgba(10,20,40,0.8);border:0.5px solid rgba(167,139,250,0.3);border-radius:8px;padding:6px 12px;text-align:right">
          <div style="font-size:17px;font-weight:500;color:#a78bfa;font-family:monospace">${user.followers||0}</div>
          <div style="font-size:9px;color:rgba(255,255,255,0.35);letter-spacing:.08em">FOLLOWERS</div>
        </div>
      </div>

      <div id="car" style="position:absolute;bottom:38px;left:-60px;animation:carMove 9s linear infinite;z-index:5">
        <svg width="40" height="18" viewBox="0 0 40 18">
          <rect x="2" y="6" width="36" height="10" rx="3" fill="#7c3aed"/>
          <rect x="8" y="2" width="20" height="7" rx="2" fill="#a78bfa"/>
          <rect x="24" y="7" width="3" height="2" rx="1" fill="#fef08a"/>
          <rect x="3" y="10" width="4" height="2" rx="1" fill="#fca5a5" opacity=".6"/>
          <circle cx="10" cy="16" r="3" fill="#1e1e2e"/><circle cx="10" cy="16" r="1.5" fill="#6b7280"/>
          <circle cx="30" cy="16" r="3" fill="#1e1e2e"/><circle cx="30" cy="16" r="1.5" fill="#6b7280"/>
        </svg>
      </div>
      <div id="car2" style="position:absolute;bottom:38px;right:-50px;animation:carMove 13s linear infinite 4s;z-index:5">
        <svg width="34" height="16" viewBox="0 0 34 16">
          <rect x="2" y="5" width="30" height="9" rx="3" fill="#0369a1"/>
          <rect x="7" y="2" width="16" height="6" rx="2" fill="#38bdf8"/>
          <rect x="26" y="7" width="3" height="2" rx="1" fill="#fef08a"/>
          <circle cx="8" cy="14" r="2.5" fill="#1e1e2e"/><circle cx="8" cy="14" r="1.2" fill="#6b7280"/>
          <circle cx="26" cy="14" r="2.5" fill="#1e1e2e"/><circle cx="26" cy="14" r="1.2" fill="#6b7280"/>
        </svg>
      </div>

      <div style="position:absolute;top:50px;left:-80px;animation:cloudDrift 22s linear infinite 2s;opacity:0.15;pointer-events:none">
        <svg width="100" height="34" viewBox="0 0 100 34"><ellipse cx="50" cy="22" rx="48" ry="13" fill="#94a3b8"/><ellipse cx="33" cy="18" rx="22" ry="16" fill="#94a3b8"/><ellipse cx="68" cy="20" rx="18" ry="14" fill="#94a3b8"/></svg>
      </div>
      <div style="position:absolute;top:28px;left:-80px;animation:cloudDrift 30s linear infinite 10s;opacity:0.09;pointer-events:none">
        <svg width="80" height="26" viewBox="0 0 80 26"><ellipse cx="40" cy="18" rx="38" ry="10" fill="#94a3b8"/><ellipse cx="25" cy="14" rx="18" ry="13" fill="#94a3b8"/><ellipse cx="58" cy="16" rx="14" ry="11" fill="#94a3b8"/></svg>
      </div>

      <div style="position:absolute;bottom:0;left:0;right:0;height:2px;background:rgba(59,130,246,0.15)"></div>
    </div>

    <div style="background:#080d1c;border:0.5px solid rgba(255,255,255,0.06);border-top:none;border-radius:0 0 12px 12px;padding:1rem">
      <div style="display:flex;align-items:center;justify-content:space-between;flex-wrap:wrap;gap:10px">
        <div>
          <div style="font-size:9px;color:rgba(255,255,255,0.25);letter-spacing:.1em;font-family:monospace;margin-bottom:6px">TECH STACK</div>
          <div style="display:flex;flex-wrap:wrap;gap:5px" id="techpills">
            ${[['Flutter','#0175C2','#7dd3fc'],['Dart','#0175C2','#7dd3fc'],['React','#0e7490','#67e8f9'],['Next.js','#374151','#d1d5db'],['TypeScript','#1d4ed8','#93c5fd'],['Riverpod','#5b21b6','#c4b5fd'],['Tailwind','#0f766e','#5eead4'],['Firebase','#b45309','#fde047'],['Node.js','#14532d','#86efac']].map(([n,b,t])=>`<span style="font-size:11px;padding:3px 9px;border-radius:20px;border:0.5px solid ${b};color:${t};background:${b}22;font-family:monospace">${n}</span>`).join('')}
          </div>
        </div>
        <div style="display:flex;gap:7px">
          <a href="https://github.com/cbagajurel" target="_blank" style="font-size:12px;padding:7px 14px;border-radius:8px;border:0.5px solid rgba(255,255,255,0.15);color:rgba(255,255,255,0.7);text-decoration:none;background:rgba(255,255,255,0.04);display:flex;align-items:center;gap:5px;font-family:monospace"><i class="ti ti-brand-github" aria-hidden="true"></i>GitHub</a>
          <a href="https://www.shivaharigajurel.com.np/" target="_blank" style="font-size:12px;padding:7px 14px;border-radius:8px;border:0.5px solid rgba(83,74,183,0.5);color:#a5b4fc;text-decoration:none;background:rgba(83,74,183,0.12);display:flex;align-items:center;gap:5px;font-family:monospace"><i class="ti ti-external-link" aria-hidden="true"></i>Portfolio</a>
          <a href="mailto:cbagajurel@gmail.com" style="font-size:12px;padding:7px 14px;border-radius:8px;border:0.5px solid rgba(29,206,117,0.4);color:#4ade80;text-decoration:none;background:rgba(29,206,117,0.08);display:flex;align-items:center;gap:5px;font-family:monospace"><i class="ti ti-mail" aria-hidden="true"></i>Hire</a>
        </div>
      </div>
    </div>
  `;

  initCity(weeklyCommits, totalCommits);

  function animCount(el, target, dur){
    const s = performance.now();
    (function step(now){
      const p = Math.min((now-s)/dur,1);
      el.textContent = Math.round(p*target);
      if(p<1) requestAnimationFrame(step);
    })(performance.now());
  }
  setTimeout(()=> animCount(document.getElementById('tc'), totalCommits, 1200), 300);
}

function initCity(weeklyCommits, totalCommits){
  const canvas = document.getElementById('cityCanvas');
  if(!canvas) return;
  const ctx = canvas.getContext('2d');
  const tooltip = document.getElementById('tooltip');
  let frame = 0, hoveredIdx = -1;

  function resize(){
    canvas.width = canvas.offsetWidth * (window.devicePixelRatio||1);
    canvas.height = canvas.offsetHeight * (window.devicePixelRatio||1);
    ctx.scale(window.devicePixelRatio||1, window.devicePixelRatio||1);
  }
  resize();
  window.addEventListener('resize', resize);

  const W=()=>canvas.offsetWidth, H=()=>canvas.offsetHeight;
  const maxC = Math.max(...weeklyCommits.map(w=>w.commits), 1);

  const PALETTE = {
    empty:    {face:'#0d1424', side:'#090e1a', roof:'#0a1120'},
    low:      {face:'#0f2d52', side:'#0a2040', roof:'#0c2545'},
    mid:      {face:'#0a3d6b', side:'#072d52', roof:'#093260'},
    high:     {face:'#1251a0', side:'#0d3d7a', roof:'#0f4590'},
    vhigh:    {face:'#1a6fdc', side:'#1254aa', roof:'#155ec4'},
    peak:     {face:'#2d8eff', side:'#1a6fdc', roof:'#2580f0'},
  };

  function getP(c){
    if(c===0) return PALETTE.empty;
    const r = c/maxC;
    if(r<.15) return PALETTE.low;
    if(r<.35) return PALETTE.mid;
    if(r<.55) return PALETTE.high;
    if(r<.8)  return PALETTE.vhigh;
    return PALETTE.peak;
  }

  const buildings = weeklyCommits.map((w,i)=>{
    const norm = w.commits / maxC;
    const minH = 18, maxH = 170;
    const bH = Math.floor(minH + norm * (maxH-minH));
    const bW = 9 + Math.floor(Math.random()*5);
    const wRows = Math.floor(bH/11);
    const wCols = Math.max(1, Math.floor(bW/5));
    const windows = [];
    for(let r=0;r<wRows;r++){
      for(let c=0;c<wCols;c++){
        const rnd = Math.random();
        windows.push({
          row:r, col:c,
          lit: w.commits>0 && Math.random()>.25,
          color: rnd<.5?'#fef08a': rnd<.75?'#a8d8ff':'#f0abfc',
          phase: Math.random()*Math.PI*2,
          speed: 0.02+Math.random()*0.03
        });
      }
    }
    const hasTower = w.commits > maxC*0.5 && Math.random()>.5;
    const hasAntenna = w.commits > maxC*0.3;
    return {...w, norm, bH, bW:bW*2, windows, hasTower, hasAntenna, pal: getP(w.commits)};
  });

  const stars = Array.from({length:120},(_,i)=>({
    x:(i*137.5)%680,
    y:(i*91.3)%220,
    size: i%7===0?1.8:i%3===0?1.2:0.8,
    phase:Math.random()*Math.PI*2,
    speed:0.005+Math.random()*0.015
  }));

  function draw(){
    const w=W(), h=H();
    ctx.clearRect(0,0,w,h);

    ctx.fillStyle='#060b18'; ctx.fillRect(0,0,w,h);

    ctx.fillStyle='rgba(20,40,100,0.12)';
    for(let i=0;i<6;i++){
      ctx.fillRect(0,(h*0.6/6)*i,w,h*0.6/6*0.5);
    }

    stars.forEach(s=>{
      const a = 0.2+Math.sin(frame*s.speed+s.phase)*0.5;
      ctx.fillStyle=`rgba(255,255,255,${Math.max(0.05,a)})`;
      ctx.fillRect(s.x*(w/680), s.y, s.size, s.size);
    });

    const moonX=w*0.87, moonY=36;
    ctx.fillStyle='rgba(245,245,200,0.92)';
    ctx.beginPath(); ctx.arc(moonX,moonY,13,0,Math.PI*2); ctx.fill();
    ctx.fillStyle='#070d1d';
    ctx.beginPath(); ctx.arc(moonX+6,moonY-3,11,0,Math.PI*2); ctx.fill();
    ctx.fillStyle='rgba(245,245,200,0.08)';
    ctx.beginPath(); ctx.arc(moonX,moonY,22,0,Math.PI*2); ctx.fill();

    const groundY = h - 44;

    ctx.fillStyle='rgba(30,50,100,0.25)';
    ctx.fillRect(0, groundY-30, w, 30);

    const bW_total = w / buildings.length;
    const buildingRects = [];

    buildings.forEach((b,i)=>{
      const bx = i * bW_total + (bW_total - b.bW)/2;
      const by = groundY - b.bH;
      buildingRects.push({bx, by, bW:b.bW, bH:b.bH, idx:i});
      const isH = i===hoveredIdx;
      const p = b.pal;

      if(isH && b.commits>0){
        ctx.fillStyle=`rgba(40,130,255,0.12)`;
        ctx.fillRect(bx-6, by-6, b.bW+12, b.bH+6);
      }

      if(b.commits>maxC*0.7){
        const gAlpha = 0.05 + Math.sin(frame*0.04+i)*0.04;
        ctx.fillStyle=`rgba(40,130,255,${gAlpha})`;
        ctx.fillRect(bx-10, by-10, b.bW+20, b.bH+10);
      }

      ctx.fillStyle=p.face; ctx.fillRect(bx, by, b.bW, b.bH);

      ctx.fillStyle=p.side;
      ctx.fillRect(bx+b.bW, by+3, 4, b.bH-3);

      ctx.fillStyle=p.roof; ctx.fillRect(bx-2, by-5, b.bW+6, 7);
      ctx.fillStyle=p.side; ctx.fillRect(bx+b.bW+4, by+3, 2, 4);

      if(b.hasAntenna){
        ctx.fillStyle='rgba(200,200,220,0.5)';
        ctx.fillRect(bx+b.bW/2-0.5, by-18, 1, 14);
        const blink = Math.sin(frame*0.08+i*1.3)>.3;
        ctx.fillStyle=blink?'rgba(255,80,80,0.9)':'rgba(255,80,80,0.2)';
        ctx.beginPath(); ctx.arc(bx+b.bW/2, by-18, 2.5, 0, Math.PI*2); ctx.fill();
      }

      if(b.hasTower){
        ctx.fillStyle=p.face;
        const tw=b.bW*0.3, th=30;
        ctx.fillRect(bx+b.bW/2-tw/2, by-th-5, tw, th+5);
        ctx.fillStyle=p.roof; ctx.fillRect(bx+b.bW/2-tw/2-2, by-th-8, tw+4, 6);
      }

      b.windows.forEach(win=>{
        const wx = bx+3+win.col*((b.bW-6)/Math.max(1,Math.floor(b.bW/5)));
        const wy = by+6+win.row*10;
        if(wy>groundY-3||wx+3>bx+b.bW-2) return;
        if(win.lit){
          const flicker = 0.6+Math.sin(frame*win.speed+win.phase)*0.35;
          ctx.fillStyle=win.color; ctx.globalAlpha=Math.max(0.3,flicker);
          ctx.fillRect(wx,wy,3.5,4.5);
          ctx.globalAlpha=0.15; ctx.fillRect(wx-1,wy-1,5.5,6.5);
          ctx.globalAlpha=1;
        } else {
          ctx.fillStyle='rgba(0,0,0,0.25)'; ctx.fillRect(wx,wy,3.5,4.5);
        }
      });

      const reflY = groundY + 4;
      const reflH = Math.min(b.bH*0.18, 16);
      ctx.fillStyle=p.face; ctx.globalAlpha=0.18;
      ctx.fillRect(bx, reflY, b.bW, reflH);
      ctx.globalAlpha=1;
    });

    ctx.fillStyle='#0a1628'; ctx.fillRect(0, groundY, w, 44);
    ctx.fillStyle='#111d38'; ctx.fillRect(0, groundY, w, 6);
    ctx.fillStyle='rgba(250,200,50,0.7)';
    for(let lx=30;lx<w;lx+=80){
      ctx.fillRect(lx+(frame*0.4)%80-80, groundY+20, 38, 3);
    }
    ctx.fillStyle='rgba(255,255,255,0.04)';
    ctx.fillRect(0, groundY+8, w, 2);
    ctx.fillRect(0, groundY+36, w, 1);

    frame++;
    requestAnimationFrame(draw);

    canvas._buildingRects = buildingRects;
  }
  draw();

  canvas.addEventListener('mousemove', e=>{
    const rect=canvas.getBoundingClientRect();
    const mx=e.clientX-rect.left, my=e.clientY-rect.top;
    const rects = canvas._buildingRects||[];
    let found=-1;
    rects.forEach(r=>{ if(mx>=r.bx&&mx<=r.bx+r.bW&&my>=r.by&&my<=r.by+r.bH) found=r.idx; });
    hoveredIdx=found;
    if(found>=0){
      const b=weeklyCommits[found];
      tooltip.style.display='block';
      const tx = Math.min(mx+12, W()-170);
      tooltip.style.left=tx+'px';
      tooltip.style.top=Math.max(my-60,8)+'px';
      tooltip.innerHTML=`<div style="color:#fff;font-weight:500;margin-bottom:3px">${b.commits} commit${b.commits!==1?'s':''}</div><div style="color:rgba(255,255,255,0.4);font-size:11px">Week of ${b.week||'—'}</div>`;
    } else {
      tooltip.style.display='none';
    }
  });
  canvas.addEventListener('mouseleave',()=>{ hoveredIdx=-1; tooltip.style.display='none'; });
}

fetchGitHub();
</script>
