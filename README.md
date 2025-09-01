<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>ระบบจัดการสต็อก Production</title>
  <style>
    :root{
      --bg:#0f172a; --panel:#111827; --muted:#1f2937; --card:#0b1220;
      --text:#e5e7eb; --sub:#9ca3af; --brand:#22c55e; --accent:#f59e0b; --danger:#ef4444;
      --ring: 0 0 0 3px rgba(34,197,94,.25);
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{margin:0;font-family:ui-sans-serif,system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial,"Noto Sans Thai","Noto Sans";background:radial-gradient(1000px 500px at 10% -10%, #133, transparent),radial-gradient(1200px 800px at 90% -20%, #24134a, transparent),var(--bg);color:var(--text)}
    .container{max-width:1400px;margin:32px auto;padding:0 16px}
    header{display:flex;gap:16px;align-items:center;justify-content:space-between;margin-bottom:16px; position:relative; z-index:2;}
    h1{font-size:clamp(1.3rem,2.2vw,1.9rem);margin:0;font-weight:700;letter-spacing:.3px}
    .hint{color:var(--sub);font-size:.92rem}
    .panel{background:linear-gradient(180deg,rgba(255,255,255,.03),rgba(255,255,255,.01));border:1px solid rgba(255,255,255,.07);border-radius:16px;padding:16px;box-shadow:0 10px 24px rgba(0,0,0,.35)}
    form#addForm{display:grid;grid-template-columns:2fr 1fr 1.6fr auto;gap:12px;align-items:end}
    label{display:block;font-size:.9rem;color:var(--sub);margin:0 0 6px}
    input[type="text"],input[type="number"],input[type="file"],.search{width:100%;padding:12px;background:var(--panel);color:var(--text);border:1px solid rgba(255,255,255,.08);border-radius:12px;outline:none}
    input:focus,.search:focus{box-shadow:var(--ring);border-color:rgba(255,255,255,.18)}
    input[type="file"]{padding:10px}

    /* Buttons (compact) */
    .btn{ cursor:pointer;border:0;padding:7px 10px;border-radius:10px;font-weight:700;font-size:.74rem;line-height:1.15;letter-spacing:.1px;transition:transform .08s ease, opacity .2s ease;white-space:nowrap;overflow:visible;text-overflow:clip }
    .btn:active{transform:translateY(1px)}
    .btn-primary{background:var(--brand);color:#052e16}
    .btn-danger{background:var(--danger);color:#fff}
    .btn-ghost{background:transparent;color:var(--sub)}

    .toolbar{display:flex;gap:12px;align-items:center;flex-wrap:wrap;margin:16px 0}
    .searchWrap{position:relative;flex:1;min-width:260px}
    .search{padding-left:40px}
    .searchIcon{position:absolute;left:12px;top:50%;transform:translateY(-50%);opacity:.6}
    .clearBtn{position:absolute;right:8px;top:50%;transform:translateY(-50%);font-size:.85rem;background:transparent;border:0;color:var(--sub);cursor:pointer}
    .badge{display:inline-flex;align-items:center;gap:6px;background:#122033;border:1px solid rgba(255,255,255,.08);padding:6px 10px;border-radius:999px;color:#cbd5e1;font-size:.85rem}

    /* Grid responsive */
    .grid{ display:grid; grid-template-columns: repeat(10, 1fr); gap:12px; }
    @media (max-width:1400px){ .grid{ grid-template-columns: repeat(8, 1fr); } }
    @media (max-width:1200px){ .grid{ grid-template-columns: repeat(6, 1fr); } }
    @media (max-width:900px){ .grid{ grid-template-columns: repeat(4, 1fr); } }
    @media (max-width:600px){ .grid{ grid-template-columns: repeat(2, 1fr); } }

    .card{background:linear-gradient(180deg,rgba(255,255,255,.03),rgba(255,255,255,.015));border:1px solid rgba(255,255,255,.07);border-radius:16px;overflow:hidden;display:flex;flex-direction:column}
    .thumb{width:100%;aspect-ratio:4/3;background:var(--muted);display:grid;place-items:center;cursor:pointer}
    .thumb img{width:100%;height:100%;object-fit:contain;display:block}
    .thumb svg{opacity:.35}
    .cardBody{padding:10px 10px 12px;display:flex;flex-direction:column;align-items:stretch;gap:6px}
    .title{font-weight:600;font-size:.85rem;line-height:1.15;letter-spacing:.2px;margin:2px 0 2px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
    .title.editing{display:none}
    .editRow{display:none;gap:6px}
    .editRow.active{display:flex}
    .editRow .editInput{font-size:.85rem;padding:8px;background:#0e1726;border:1px solid rgba(255,255,255,.12);border-radius:10px;color:var(--text)}
    .qty{font-size:.85rem;color:var(--sub)}

    /* Actions: expand FRAME to fit all buttons (wrap lines, no scroll) */
    .actions{ display:flex; flex-wrap:wrap; align-items:center; gap:8px 8px; width:100%; border-top:1px solid rgba(255,255,255,.08); padding-top:8px; overflow:visible }
    .actions .btn{ flex:0 0 auto }

    .footer{display:flex;justify-content:space-between;align-items:center;margin-top:14px;color:var(--sub);font-size:.92rem}
    .muted{color:var(--sub)}
    .empty{opacity:.8;text-align:center;padding:24px}
    .stat{display:flex;gap:12px;align-items:center;flex-wrap:wrap}

    /* Preview modal */
    .modal{position:fixed;inset:0;display:none;align-items:center;justify-content:center;background:rgba(0,0,0,.7);backdrop-filter:blur(2px);z-index:9999}
    .modal.open{display:flex}
    .modal .box{max-width:min(90vw,1200px);max-height:90vh;background:#000;border:1px solid rgba(255,255,255,.15);border-radius:12px;overflow:hidden;position:relative}
    .modal img{display:block;max-width:100%;max-height:90vh;object-fit:contain;background:#000}
    .modal .close{position:absolute;top:8px;right:8px;background:rgba(255,255,255,.15);color:#fff;border:0;border-radius:10px;padding:6px 10px;cursor:pointer}
    .modal .caption{position:absolute;left:0;right:0;bottom:0;background:linear-gradient(180deg,transparent,rgba(0,0,0,.75));color:#e5e7eb;padding:10px 12px;font-size:.92rem}

    /* Settings modal */
    .settings{position:fixed;inset:0;display:none;align-items:center;justify-content:center;background:rgba(0,0,0,.55);backdrop-filter:blur(2px);z-index:9998}
    .settings.open{display:flex}
    .settings .box{width:min(560px,92vw);background:#0b1020;border:1px solid rgba(255,255,255,.08);border-radius:16px;padding:16px}
    .row{display:grid;grid-template-columns:140px 1fr;gap:12px;align-items:center;margin:8px 0}
    @media (max-width:800px){form#addForm{grid-template-columns:1fr 1fr}form#addForm .full{grid-column:1/-1}.row{grid-template-columns:1fr}}
  </style>
</head>
<body>
<div class="container">
  <header>
    <div>
      <h1>ระบบจัดการสต็อก Production</h1>
      <div class="hint">โฮสต์บน napfamily.w3spaces.com • ใส่ Endpoint เป็น URL ของ Cloudflare Worker หลัง Deploy</div>
    </div>
    <div style="display:flex;gap:8px">
      <button id="btnSettings" type="button" class="btn">⚙️ ตั้งค่า</button>
      <button id="btnSync" type="button" class="btn btn-primary" title="ส่งข้อมูลออกไปยังปลายทางที่ตั้งค่าไว้">ซิงก์ตอนนี้</button>
    </div>
  </header>

  <section class="panel" aria-labelledby="addLabel" style="margin-bottom:16px;">
    <form id="addForm">
      <div>
        <label id="addLabel" for="name">ชื่อสินค้า<span class="muted"> *</span></label>
        <input id="name" type="text" placeholder="เช่น กาแฟคั่วบด 250g" required />
      </div>
      <div>
        <label for="qty">จำนวนคงเหลือ</label>
        <input id="qty" type="number" min="0" step="1" value="0" />
      </div>
      <div class="full">
        <label for="image">รูปสินค้า (อัปโหลดได้)</label>
        <input id="image" type="file" accept="image/*" />
      </div>
      <div>
        <button class="btn btn-primary" type="submit">➕ เพิ่มสินค้า</button>
      </div>
    </form>

    <div class="toolbar">
      <div class="searchWrap">
        <input id="search" class="search" type="text" placeholder="ค้นหาสินค้าด้วยชื่อ…" autocomplete="off" />
        <svg class="searchIcon" width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true"><path d="M21 21l-4.3-4.3" stroke="currentColor" stroke-width="2" stroke-linecap="round"/><circle cx="11" cy="11" r="7" stroke="currentColor" stroke-width="2"/></svg>
        <button id="clearSearch" class="clearBtn" aria-label="ล้างการค้นหา">ล้าง</button>
      </div>
      <div class="stat">
        <span class="badge" id="countBadge">รายการทั้งหมด: 0</span>
        <span class="badge" id="sumBadge">ยอดรวมสต็อก: 0 ชิ้น</span>
      </div>
    </div>
  </section>

  <section class="panel">
    <div id="grid" class="grid" aria-live="polite"></div>
    <div id="empty" class="empty" hidden>ยังไม่มีสินค้า – เพิ่มรายการแรกของคุณด้านบนได้เลย</div>
  </section>

  <div class="footer">
    <span>Made with ❤️ • ใช้งานออฟไลน์</span>
    <div style="display:flex;gap:8px;align-items:center">
      <button id="resetAll" class="btn btn-ghost" title="ล้างข้อมูลทั้งหมด" type="button">ล้างข้อมูลทั้งหมด</button>
      <button id="btnPull" class="btn" type="button">โหลดจากเซิร์ฟเวอร์</button>
    </div>
  </div>
</div>

<!-- Image Preview Modal -->
<div id="imgPreview" class="modal" role="dialog" aria-modal="true" aria-labelledby="imgCaption">
  <div class="box">
    <button class="close" id="imgClose" type="button">ปิด</button>
    <img id="imgLarge" alt="ตัวอย่างรูปสินค้า" />
    <div id="imgCaption" class="caption"></div>
  </div>
</div>

<!-- Settings Modal -->
<div id="settingsModal" class="settings" role="dialog" aria-modal="true" aria-labelledby="settingsTitle">
  <div class="box">
    <h2 id="settingsTitle" style="margin:0 0 8px">ตั้งค่าการเชื่อมต่อ</h2>
    <p class="muted" style="margin-top:0">เชื่อมต่อผ่าน Cloudflare Worker → Google Apps Script</p>
    <div class="row">
      <label for="endpoint">Endpoint URL</label>
      <input id="endpoint" type="text" placeholder="วาง URL ของ Worker เช่น https://xxxx.workers.dev" />
    </div>
    <div class="row">
      <label for="apiKey">API Key (ถ้ามี)</label>
      <input id="apiKey" type="text" placeholder="ใส่ถ้ามีการตรวจสอบสิทธิ์" />
    </div>
    <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:12px">
      <button id="closeSettings" type="button" class="btn">ปิด</button>
      <button id="saveSettings" type="button" class="btn btn-primary">บันทึก</button>
    </div>
  </div>
</div>

<script>
(function(){
  const LS_KEY = 'inventoryProducts_v1';
  const LS_SETTINGS = 'inventorySettings_v1';
  const $ = sel => document.querySelector(sel);
  const grid = $('#grid');
  const empty = $('#empty');
  const nameEl = $('#name');
  const qtyEl = $('#qty');
  const imgEl = $('#image');
  const form = $('#addForm');
  const searchEl = $('#search');
  const clearBtn = $('#clearSearch');
  const resetAllBtn = $('#resetAll');
  const countBadge = $('#countBadge');
  const sumBadge = $('#sumBadge');

  const btnSettings = $('#btnSettings');
  const btnSync = $('#btnSync');
  const btnPull = $('#btnPull');
  const settingsModal = $('#settingsModal');
  const endpointEl = $('#endpoint');
  const apiKeyEl = $('#apiKey');
  const closeSettingsBtn = $('#closeSettings');
  const saveSettingsBtn = $('#saveSettings');

  const imgPreview = $('#imgPreview');
  const imgLarge = $('#imgLarge');
  const imgClose = $('#imgClose');
  const imgCaption = $('#imgCaption');

  let products = load();
  let settings = loadSettings();
  let q = '';

  function uid(){ return Date.now().toString(36) + Math.random().toString(36).slice(2,7); }
  function load(){ try{ const raw = localStorage.getItem(LS_KEY); return raw ? JSON.parse(raw) : []; }catch(e){ return []; } }
  function save(){ localStorage.setItem(LS_KEY, JSON.stringify(products)); }
  function loadSettings(){ try{ const raw = localStorage.getItem(LS_SETTINGS); return raw ? JSON.parse(raw) : {endpoint:'',apiKey:''}; }catch(e){ return {endpoint:'',apiKey:''}; } }
  function saveSettings(){ localStorage.setItem(LS_SETTINGS, JSON.stringify(settings)); }

  function normalize(str){ return (str||'').toString().toLowerCase().trim(); }
  function stats(){ countBadge.textContent = `รายการทั้งหมด: ${products.length}`; const sum = products.reduce((a,b)=> a + (Number(b.qty)||0), 0); sumBadge.textContent = `ยอดรวมสต็อก: ${sum} ชิ้น`; }

  function placeholderSVG(){ return '<svg width="64" height="64" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><rect x="3" y="3" width="18" height="18" rx="2" stroke="white" opacity=".25"/><path d="M7 16l3-3 3 3 4-4 2 2v3H7z" fill="white" opacity=".25"/><circle cx="9" cy="9" r="2" fill="white" opacity=".25"/></svg>'; }
  function escapeHtml(str){ return (str||'').replace(/[&<>"]/g, s => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[s])); }

  function cardTemplate(p){
    const img = p.image ? '<img alt="'+escapeHtml(p.name)+'" src="'+p.image+'"/>' : placeholderSVG();
    return '<article class="card" data-id="'+p.id+'">\
      <div class="thumb" data-action="preview">'+ img +'</div>\
      <div class="cardBody">\
        <div class="title" data-field="title">'+escapeHtml(p.name)+'</div>\
        <div class="editRow" data-field="editRow">\
          <input type="text" class="editInput" value="'+escapeHtml(p.name)+'"/>\
          <button class="btn btn-primary" data-action="saveName" type="button">บันทึก</button>\
          <button class="btn" data-action="cancelEdit" type="button">ยกเลิก</button>\
        </div>\
        <div class="qty">คงเหลือ <strong>'+ (Number(p.qty)||0) +'</strong> ชิ้น</div>\
        <div class="actions" aria-label="ปุ่มดำเนินการ">\
          <button class="btn" data-action="edit" title="แก้ไขชื่อ" type="button">แก้ไขชื่อ</button>\
          <button class="btn" data-action="inc" title="เพิ่มจำนวน" type="button">เพิ่มจำนวน</button>\
          <button class="btn" data-action="dec" title="ลดจำนวน" type="button">ลดจำนวน</button>\
          <button class="btn btn-danger" data-action="delete" title="ลบสินค้า" type="button">ลบสินค้า</button>\
        </div>\
      </div>\
    </article>';
  }

  function render(list){ if(!list.length){ grid.innerHTML=''; empty.hidden=false; } else { grid.innerHTML=list.map(cardTemplate).join(''); empty.hidden=true; } stats(); }
  function filter(){ const query = normalize(q); if(!query) return products; return products.filter(p => normalize(p.name).includes(query)); }

  async function readImageToDataURL(file){
    if(!file) return null;
    const maxEdge=800;
    const blobURL = URL.createObjectURL(file);
    const img = new Image(); img.src = blobURL;
    await img.decode();
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    const ratio = Math.min(1, maxEdge/Math.max(img.width, img.height));
    canvas.width = Math.round(img.width*ratio);
    canvas.height = Math.round(img.height*ratio);
    ctx.drawImage(img,0,0,canvas.width,canvas.height);
    URL.revokeObjectURL(blobURL);
    const type = (file.type||'').includes('png') ? 'image/png' : 'image/jpeg';
    const quality = type==='image/jpeg' ? 0.85 : 0.92;
    return canvas.toDataURL(type, quality);
  }

  // Add product
  form.addEventListener('submit', async (e)=>{ e.preventDefault();
    const name = nameEl.value.trim(); const qty = Math.max(0, parseInt(qtyEl.value || '0', 10));
    if(!name){ nameEl.focus(); nameEl.style.boxShadow='var(--ring)'; setTimeout(()=>nameEl.style.boxShadow='',400); return; }
    let imageData=null; try{ imageData = await readImageToDataURL(imgEl.files[0]); }catch(_){} const p = { id: uid(), name, qty, image: imageData, createdAt: Date.now() }; products.unshift(p); save(); form.reset(); qtyEl.value = 0; nameEl.focus(); render(filter()); });

  // Search
  let t; searchEl.addEventListener('input', (e)=>{ q = e.target.value || ''; clearTimeout(t); t = setTimeout(()=> render(filter()), 120); });
  clearBtn.addEventListener('click', ()=>{ q=''; searchEl.value=''; render(filter()); searchEl.focus(); });

  // Card actions
  grid.addEventListener('click', (e)=>{
    const card = e.target.closest('.card');
    if(!card) return;
    const id = card.getAttribute('data-id');
    const item = products.find(p=> p.id===id);
    if(!item) return;

    const previewEl = e.target.closest('[data-action="preview"]');
    if(previewEl){
      if(item.image){
        imgLarge.src = item.image;
        imgCaption.textContent = item.name + (typeof item.qty!=='undefined' ? ' • คงเหลือ ' + (Number(item.qty)||0) + ' ชิ้น' : '');
        imgPreview.classList.add('open');
      }
      return;
    }

    const btn = e.target.closest('button'); if(!btn) return;
    const action = btn.getAttribute('data-action');
    const titleEl = card.querySelector('[data-field="title"]');
    const editRow = card.querySelector('[data-field="editRow"]');
    const inputEl = editRow && editRow.querySelector('.editInput');

    if(action==='edit'){
      titleEl.classList.add('editing'); editRow.classList.add('active'); inputEl.value = item.name; inputEl.focus(); inputEl.select(); return;
    }
    if(action==='saveName'){
      const newName = (inputEl.value||'').trim(); if(!newName){ alert('กรุณากรอกชื่อสินค้า'); return; } item.name = newName; save(); render(filter()); return;
    }
    if(action==='cancelEdit'){ titleEl.classList.remove('editing'); editRow.classList.remove('active'); return; }
    if(action==='delete'){
      const code = prompt('กรอกรหัสลบ (Delete Code):');
      if(code === null) return;
      if(code !== 'NAP'){ alert('รหัสลบไม่ถูกต้อง'); return; }
      sessionStorage.setItem('lastDeleteCode', 'NAP'); // จำไว้สำหรับซิงก์
      if(confirm('ยืนยันการลบ "'+item.name+'"?')) products = products.filter(p=> p.id!==id);
      save(); render(filter()); return;
    }
    else if(action==='inc'){ item.qty = (Number(item.qty)||0) + 1; }
    else if(action==='dec'){ item.qty = Math.max(0, (Number(item.qty)||0) - 1); }
    save(); render(filter());
  });

  // Image preview events
  $('#imgClose').addEventListener('click', ()=> imgPreview.classList.remove('open'));
  imgPreview.addEventListener('click', (e)=>{ if(e.target === imgPreview) imgPreview.classList.remove('open'); });
  window.addEventListener('keydown', (e)=>{ if(e.key === 'Escape') imgPreview.classList.remove('open'); });

  // Reset all
  resetAllBtn.addEventListener('click', ()=>{ if(!products.length) return alert('ยังไม่มีข้อมูลให้ลบ'); if(confirm('ต้องการล้างข้อมูลสินค้าทั้งหมดหรือไม่?')){ products = []; save(); render(filter()); } });

  // ===== Settings & Sync =====
  function openSettings(){
    const s = loadSettings();
    if (!s.endpoint){
      endpointEl.placeholder = 'วาง Worker URL ที่ได้ เช่น https://<ชื่อ>.workers.dev';
    }
    endpointEl.value = s.endpoint || '';
    apiKeyEl.value = s.apiKey || '';
    settingsModal.classList.add('open');
  }
  function closeSettings(){ settingsModal.classList.remove('open'); }
  btnSettings.addEventListener('click', openSettings);
  closeSettingsBtn.addEventListener('click', closeSettings);
  settingsModal.addEventListener('click', (e)=>{ if(e.target===settingsModal) closeSettings(); });
  saveSettingsBtn.addEventListener('click', ()=>{
    settings = {
      endpoint: (endpointEl.value||'').trim(),
      apiKey: (apiKeyEl.value||'').trim()
    };
    localStorage.setItem(LS_SETTINGS, JSON.stringify(settings));
    alert('บันทึกการตั้งค่าแล้ว');
    closeSettings();
  });
  if(location.hash === '#settings'){ openSettings(); }

  // Pull from server
  btnPull.addEventListener('click', async ()=>{
    if(!settings.endpoint){ openSettings(); alert('กรุณาตั้งค่า Endpoint URL ก่อน'); return; }
    try{
      const res = await fetch(settings.endpoint, { method:'GET', headers:{'Accept':'application/json'} });
      const data = await res.json();
      if(!data.ok) throw new Error(data.error || 'โหลดไม่สำเร็จ');
      products = data.products || [];
      save(); render(filter());
      alert('โหลดข้อมูลแล้ว: ' + (products.length||0) + ' รายการ');
    }catch(err){
      alert('โหลดไม่สำเร็จ: ' + err.message);
    }
  });

  // Sync to server
  btnSync.addEventListener('click', async ()=>{
    if(!settings.endpoint){ openSettings(); alert('กรุณาตั้งค่า Endpoint URL ก่อน'); return; }
    if(!/^https?:\/\//i.test(settings.endpoint)){ alert('Endpoint URL ไม่ถูกต้อง กรุณาใส่ลิงก์ที่ขึ้นต้นด้วย https://'); return; }

    const controller = new AbortController();
    const timer = setTimeout(()=>controller.abort(), 20000);

    try{
      const res = await fetch(settings.endpoint, {
        method:'POST',
        headers: {
          'Content-Type':'application/json',
          'Accept': 'application/json',
          ...(settings.apiKey?{'x-api-key':settings.apiKey}:{}) 
        },
        body: JSON.stringify({
          source:'inventory-app',
          timestamp: Date.now(),
          deleteCode: sessionStorage.getItem('lastDeleteCode') || '',
          products
        }),
        signal: controller.signal
      });
      clearTimeout(timer);
      let data = null;
      try { data = await res.json(); } catch(_) { data = { rawText: await res.text().catch(()=>'(no body)') }; }
      if(!res.ok){
        throw new Error((data && (data.message||data.error)) || ('HTTP ' + res.status + ' ' + res.statusText));
      }
      if(data && data.ok){ sessionStorage.removeItem('lastDeleteCode'); }
      alert('ซิงก์สำเร็จ\n' + JSON.stringify(data, null, 2));
    }catch(err){
      clearTimeout(timer);
      console.error(err);
      alert('ซิงก์ไม่สำเร็จ: ' + (err && err.message ? err.message : err) +
        '\n\nตรวจสอบเบื้องต้น:\n• Endpoint URL ถูกต้องและเป็น HTTPS\n• หากใช้ Google Apps Script: Deploy เป็น Web app และตั้งสิทธิ์ Anyone with the link\n• ปลายทางตอบกลับ JSON และตั้ง CORS header');
    }
  });

  render(filter());
})();
</script>
</body>
</html>
