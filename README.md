<!doctype html><html lang="en"><head><meta charset="utf-8"/><meta name="viewport" content="width=device-width,initial-scale=1"/><title>The Darpan Wears</title>
<style>
:root{--bg:#f6f7f9;--card:#fff;--muted:#6b7280;--accent:#ff6b00;--text:#0f1724;--shadow:0 8px 22px rgba(15,23,36,0.06)}
*{box-sizing:border-box}body{margin:0;font-family:Inter,system-ui,Roboto,-apple-system;background:var(--bg);color:var(--text);-webkit-font-smoothing:antialiased}
.wrap{max-width:480px;margin:0 auto;padding:12px}
.header{display:flex;justify-content:space-between;align-items:center;gap:8px}
.brand{display:flex;gap:10px;align-items:center}
.logo{width:48px;height:48px;border-radius:10px;background:linear-gradient(135deg,var(--accent),#06b6d4);display:flex;align-items:center;justify-content:center;color:#fff;font-weight:800}
.title{font-weight:800;font-size:18px}
.sub{font-size:12px;color:var(--muted)}
.search{margin-top:12px;background:var(--card);padding:10px;border-radius:12px;box-shadow:var(--shadow);display:flex;gap:8px;align-items:center}
.chips{display:flex;gap:8px;margin-top:12px;flex-wrap:wrap}
.chip{padding:8px 12px;border-radius:999px;background:#fff;border:1px solid rgba(0,0,0,0.04);font-weight:700;font-size:13px;color:var(--muted)}
.grid{display:grid;grid-template-columns:1fr;gap:12px;margin-top:12px}
.card{background:var(--card);border-radius:12px;padding:10px;box-shadow:var(--shadow);display:flex;gap:10px;align-items:flex-start}
.card img{width:110px;height:110px;object-fit:cover;border-radius:8px}
.info{flex:1}
.titleP{font-weight:800;font-size:15px}
.desc{font-size:13px;color:var(--muted);margin-top:6px}
.price{margin-top:8px;font-weight:800}
.price .special{color:var(--accent);margin-right:8px}
.row{display:flex;gap:8px;margin-top:8px;align-items:center}
.btn{background:var(--accent);color:#fff;border:0;padding:9px 12px;border-radius:10px;font-weight:700}
.btn.out{background:#fff;color:var(--text);border:1px solid rgba(0,0,0,0.06)}
.footer{margin-top:18px;text-align:center;color:var(--muted);font-size:13px}
.modal{position:fixed;inset:0;background:rgba(0,0,0,0.45);display:none;align-items:center;justify-content:center;padding:12px}
.panel{width:100%;max-width:480px;background:#fff;border-radius:12px;padding:12px;max-height:88vh;overflow:auto}
.input,textarea,select{width:100%;padding:10px;border-radius:8px;border:1px solid rgba(0,0,0,0.06);font-size:14px;margin-top:8px}
.file{border:2px dashed rgba(0,0,0,0.06);padding:10px;border-radius:8px;text-align:center;margin-top:8px}
.canvas{width:100%;height:140px;background:#fff;border-radius:8px;padding:6px;margin-top:8px}
.small{font-size:13px;color:var(--muted)}
@media(min-width:640px){.wrap{margin-top:18px}}
</style>
</head><body>
<div class="wrap">
  <div class="header">
    <div class="brand">
      <div class="logo">DW</div>
      <div><div class="title">The Darpan Wears</div><div class="sub">T-Shirts • Jeans • Jackets • Shoes • Electronic Gadgets</div></div>
    </div>
    <div><button id="adminBtn" class="btn out">Admin</button></div>
  </div>

  <div class="search">
    <div style="flex:1"><input id="q" class="input" placeholder="Search product or category"/></div>
    <div><button id="searchBtn" class="btn out">Search</button></div>
  </div>

  <div class="chips" id="cats"></div>
  <div class="grid" id="grid"></div>

  <div style="margin-top:12px;display:flex;justify-content:space-between;align-items:center">
    <div class="small">Orders: will not be stored permanently on device</div>
    <div class="small">Payments: COD / Online</div>
  </div>

  <div class="footer">© <span id="year"></span> The Darpan Wears</div>
</div>

<!-- Order modal (no localstore saving) -->
<div id="orderModal" class="modal"><div class="panel">
  <div style="display:flex;justify-content:space-between;align-items:center"><strong>Place Order</strong><button id="closeOrder" class="btn out">Close</button></div>
  <div id="orderProd" style="margin-top:8px"></div>
  <input id="o_name" class="input" placeholder="Name"/>
  <input id="o_phone" class="input" placeholder="Phone number"/>
  <input id="o_state" class="input" placeholder="State"/>
  <input id="o_city" class="input" placeholder="City"/>
  <input id="o_village" class="input" placeholder="Village"/>
  <input id="o_pin" class="input" placeholder="Pincode"/>
  <label class="small" style="margin-top:8px">Payment</label>
  <select id="o_pay" class="input"><option value="COD">Cash on Delivery</option><option value="Online">Online Payment</option></select>
  <div style="display:flex;gap:8px;margin-top:12px">
    <button id="sendWhats" class="btn">Send on WhatsApp</button>
    <button id="placeOrder" class="btn out">Place (Save Session)</button>
  </div>
  <div class="small" style="margin-top:8px">Orders are kept only in this session (in memory) for admin viewing; they are not stored in localStorage. Use Send on WhatsApp to forward order to +91 9332307996.</div>
</div></div>

<!-- Admin modal -->
<div id="adminModal" class="modal"><div class="panel">
  <div style="display:flex;justify-content:space-between;align-items:center"><strong>Admin Panel</strong><div><button id="closeAdmin" class="btn out">Close</button></div></div>
  <div id="adminArea" style="margin-top:8px"></div>
</div></div>

<script>
/*
 Single-file compact app:
 - Products persisted in localStorage with base64 images (permanent)
 - Orders NOT persisted in localStorage; stored in memory for current session only
 - After placing order user may click "Send on WhatsApp" which opens WhatsApp chat prefilled to +91 9332307996
 - Admin login uses SHA-256 hash of 'darpan2025' (hash stored)
*/
const ADMIN_SHA256 = '0412d1616b1bfa094eef7a24818416487283351f88cc2675eb7177ffdf75cf19'; // sha256('darpan2025')
const WHATSAPP_NUMBER = '919332307996'; // international format without plus
const CATS = ['T-Shirts','Jeans','Jackets','Shoes','Electronic Gadgets'];
const yearEl = document.getElementById('year'); yearEl.textContent = new Date().getFullYear();

/* product storage: localStorage key dw_products */
function read(key){ try{return JSON.parse(localStorage.getItem(key)||'[]')}catch(e){return[]}}
function write(key,val){ localStorage.setItem(key,JSON.stringify(val)) }

/* seed products if none */
if(!localStorage.getItem('dw_products')){
  write('dw_products',[
    {id:'p_a',name:'Classic Tee',desc:'Comfort cotton tee',sell:499,special:349,size:'M',material:'Cotton',category:'T-Shirts',img:'',sold:0},
    {id:'p_b',name:'Blue Jeans',desc:'Slim fit denim',sell:1299,special:'',size:'32',material:'Denim',category:'Jeans',img:'',sold:0}
  ]);
}

/* orders in-memory only */
let sessionOrders = []; // each order: {id,product,customer,payment,date,status}

/* UI refs */
const grid = document.getElementById('grid'), catsWrap = document.getElementById('cats');
const adminBtn = document.getElementById('adminBtn'), adminModal = document.getElementById('adminModal');
const orderModal = document.getElementById('orderModal'), orderProd = document.getElementById('orderProd');
const closeOrderBtn = document.getElementById('closeOrder'), sendWhats = document.getElementById('sendWhats'), placeOrderBtn = document.getElementById('placeOrder');
const adminArea = document.getElementById('adminArea');

/* render categories */
CATS.forEach(c => { const d=document.createElement('div'); d.className='chip'; d.textContent=c; d.onclick=()=>renderProducts(c); catsWrap.appendChild(d); });

/* search */
document.getElementById('searchBtn').onclick = ()=> {
  const q = document.getElementById('q').value.trim().toLowerCase();
  renderProducts(null,q);
}

/* render products */
function renderProducts(filterCat=null, search=null){
  const prods = read('dw_products');
  let list = prods.slice();
  if(filterCat) list = list.filter(p=>p.category===filterCat);
  if(search) list = list.filter(p=>(p.name+p.desc+p.category).toLowerCase().includes(search));
  grid.innerHTML = '';
  list.forEach(p=>{
    const c = document.createElement('div'); c.className='card';
    const img = document.createElement('img'); img.src = p.img || placeholderData(); img.alt = p.name;
    const info = document.createElement('div'); info.className='info';
    const t = document.createElement('div'); t.className='titleP'; t.innerText = p.name;
    const d = document.createElement('div'); d.className='desc'; d.innerText = p.desc;
    const pr = document.createElement('div'); pr.className='price'; pr.innerHTML = p.special ? `<span class="special">₹${p.special}</span> <span class="small" style="text-decoration:line-through">₹${p.sell}</span>` : `₹${p.sell}`;
    const meta = document.createElement('div'); meta.className='small'; meta.innerText = `Size: ${p.size||'-'} • Material: ${p.material||'-'}`;
    const row = document.createElement('div'); row.className='row';
    const orderBtn = document.createElement('button'); orderBtn.className='btn'; orderBtn.innerText='Order'; orderBtn.onclick=()=>openOrder(p.id);
    const viewBtn = document.createElement('button'); viewBtn.className='btn out'; viewBtn.innerText='View'; viewBtn.onclick=()=>viewProduct(p.id);
    row.appendChild(orderBtn); row.appendChild(viewBtn);
    info.appendChild(t); info.appendChild(d); info.appendChild(pr); info.appendChild(meta); info.appendChild(row);
    c.appendChild(img); c.appendChild(info); grid.appendChild(c);
  });
}

/* placeholder image data */
function placeholderData(){ return 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAAAQCAYAAAB49lVfAAAAF0lEQVR4nGNgGAXUBwYGBgYGBgYGAAAk4A+5k5tJYAAAAASUVORK5CYII=' }

/* order flow (no localStorage saving) */
let currentProduct = null;
function openOrder(pid){
  const p = read('dw_products').find(x=>x.id===pid); if(!p){ alert('Product not found'); return; }
  currentProduct = p;
  orderProd.innerHTML = `<div style="display:flex;gap:8px;align-items:center"><img src="${p.img||placeholderData()}" style="width:72px;height:72px;object-fit:cover;border-radius:8px"><div><div style="font-weight:800">${p.name}</div><div class="small">₹${p.special||p.sell}</div></div></div>`;
  orderModal.style.display='flex';
}
closeOrderBtn.onclick = ()=> { orderModal.style.display='none'; clearOrderForm(); }

/* build whatsapp message and open chat (manual send) */
function buildWhatsText(order){
  let txt = `New order from The Darpan Wears:%0A`;
  txt += `Product: ${order.product.name} (₹${order.product.price})%0A`;
  txt += `Name: ${order.customer.name}%0APhone: ${order.customer.phone}%0A`;
  txt += `State: ${order.customer.state}, City: ${order.customer.city}, Village: ${order.customer.village}, Pincode: ${order.customer.pin}%0A`;
  txt += `Payment: ${order.payment}%0AOrder ID: ${order.id}`;
  return txt;
}
sendWhats.onclick = ()=>{
  // gather form values and open WhatsApp directly (no saving)
  const name = document.getElementById('o_name').value.trim();
  const phone = document.getElementById('o_phone').value.trim();
  const state = document.getElementById('o_state').value.trim();
  const city = document.getElementById('o_city').value.trim();
  const village = document.getElementById('o_village').value.trim();
  const pin = document.getElementById('o_pin').value.trim();
  const payment = document.getElementById('o_pay').value;
  if(!name||!phone||!state||!city||!pin){ alert('Please fill required fields'); return; }
  const order = { id:'tmp_'+Date.now().toString(36), product:{id:currentProduct.id,name:currentProduct.name,price:currentProduct.special||currentProduct.sell}, customer:{name,phone,state,city,village,pin}, payment, date:new Date().toISOString(), status:'pending' };
  const msg = buildWhatsText(order);
  const url = `https://wa.me/${WHATSAPP_NUMBER}?text=${msg}`;
  window.open(url,'_blank');
}

/* place order into sessionOrders (memory) for admin view (not localStorage) */
placeOrderBtn.onclick = ()=>{
  const name = document.getElementById('o_name').value.trim();
  const phone = document.getElementById('o_phone').value.trim();
  const state = document.getElementById('o_state').value.trim();
  const city = document.getElementById('o_city').value.trim();
  const village = document.getElementById('o_village').value.trim();
  const pin = document.getElementById('o_pin').value.trim();
  const payment = document.getElementById('o_pay').value;
  if(!name||!phone||!state||!city||!pin){ alert('Please fill required fields'); return; }
  const order = { id:'ord_'+Date.now().toString(36), product:{id:currentProduct.id,name:currentProduct.name,price:currentProduct.special||currentProduct.sell}, customer:{name,phone,state,city,village,pin}, payment, date:new Date().toISOString(), status:'pending' };
  sessionOrders.unshift(order);
  alert('Order placed (session only). Use Send on WhatsApp to forward to your WhatsApp.');
  orderModal.style.display='none';
  clearOrderForm();
  if(adminModal.style.display==='flex'){ renderAdminContent(); }
}

function clearOrderForm(){ ['o_name','o_phone','o_state','o_city','o_village','o_pin'].forEach(id=>document.getElementById(id).value=''); document.getElementById('o_pay').value='COD'; }

/* admin login */
async function sha256hex(msg){ const enc = new TextEncoder().encode(msg); const buf = await crypto.subtle.digest('SHA-256',enc); return [...new Uint8Array(buf)].map(b=>b.toString(16).padStart(2,'0')).join(''); }
adminBtn.onclick = async ()=>{
  const pwd = prompt('Enter admin password'); if(!pwd) return;
  const h = await sha256hex(pwd);
  if(h !== ADMIN_SHA256){ alert('Incorrect password'); return; }
  openAdmin();
}

/* admin modal behavior */
function openAdmin(){ adminModal.style.display='flex'; renderAdminContent(); }
document.getElementById('closeAdmin').onclick = ()=> adminModal.style.display='none';

/* admin UI: add/edit/delete products (images saved as base64 in localStorage), view session orders, approve/reject */
function renderAdminContent(){
  adminArea.innerHTML = `
    <div><input id="ap_name" class="input" placeholder="Product name"/></div>
    <div><textarea id="ap_desc" class="input" placeholder="Description"></textarea></div>
    <div style="display:flex;gap:8px"><input id="ap_sell" class="input" placeholder="Selling Price"/><input id="ap_special" class="input" placeholder="Special Price"/></div>
    <div style="display:flex;gap:8px"><input id="ap_size" class="input" placeholder="Size"/><input id="ap_material" class="input" placeholder="Material"/></div>
    <div style="margin-top:8px"><select id="ap_cat" class="input">${CATS.map(c=>`<option>${c}</option>`).join('')}</select></div>
    <div class="file"><input id="ap_file" type="file" accept="image/*"/></div>
    <div id="ap_preview" style="margin-top:8px"></div>
    <div style="margin-top:8px"><button id="ap_add" class="btn">Add Product</button></div>
    <hr/><div><strong>Products</strong><div id="ap_list"></div></div><hr/>
    <div><strong>Session Orders</strong><div id="ap_orders"></div></div>
    <div class="canvas"><canvas id="sales" width="360" height="120"></canvas></div>
  `;
  const file = document.getElementById('ap_file');
  file.onchange = ()=>{ const f=file.files[0]; if(!f) return; const r=new FileReader(); r.onload=e=>document.getElementById('ap_preview').innerHTML=`<img src="${e.target.result}" style="max-width:120px;border-radius:8px">`; r.readAsDataURL(f); };
  document.getElementById('ap_add').onclick = async ()=>{
    const name = document.getElementById('ap_name').value.trim(); if(!name){ alert('Name required'); return; }
    const desc = document.getElementById('ap_desc').value||''; const sell = Number(document.getElementById('ap_sell').value)||0;
    const special = document.getElementById('ap_special').value||''; const size = document.getElementById('ap_size').value||''; const material = document.getElementById('ap_material').value||''; const category = document.getElementById('ap_cat').value;
    let img = '';
    if(file.files[0]) img = await new Promise(r=>{ const fr=new FileReader(); fr.onload=e=>r(e.target.result); fr.readAsDataURL(file.files[0]); });
    const prods = read('dw_products'); prods.unshift({ id:'p_'+Date.now().toString(36), name, desc, sell, special, size, material, category, img, sold:0 }); write('dw_products',prods);
    renderAdminContent(); renderProducts();
  };
  renderAdminProducts(); renderAdminOrders(); renderChart();
}

function renderAdminProducts(){
  const list = document.getElementById('ap_list'); list.innerHTML=''; const prods = read('dw_products');
  prods.forEach(p=>{
    const row = document.createElement('div'); row.style.display='flex'; row.style.justifyContent='space-between'; row.style.alignItems='center'; row.style.marginTop='8px';
    row.innerHTML = `<div><strong>${p.name}</strong><div class="small">₹${p.sell} • sold:${p.sold||0}</div></div>`;
    const right = document.createElement('div');
    const edit = document.createElement('button'); edit.className='btn out'; edit.innerText='Edit'; edit.onclick=()=>adminEdit(p.id);
    const del = document.createElement('button'); del.className='btn out'; del.style.marginLeft='8px'; del.innerText='Delete'; del.onclick=()=>{ if(confirm('Delete?')){ write('dw_products', read('dw_products').filter(x=>x.id!==p.id)); renderAdminContent(); renderProducts(); } };
    right.appendChild(edit); right.appendChild(del); row.appendChild(right); list.appendChild(row);
  });
}

function adminEdit(id){
  const prods = read('dw_products'); const p = prods.find(x=>x.id===id); if(!p) return;
  const name = prompt('Name',p.name); if(!name) return; const desc = prompt('Description',p.desc||''); const sell = prompt('Selling price',p.sell); const special = prompt('Special price',p.special||''); const size = prompt('Size',p.size||''); const material = prompt('Material',p.material||'');
  p.name = name; p.desc = desc; p.sell = Number(sell)||0; p.special = special; p.size = size; p.material = material;
  write('dw_products', prods.map(x=>x.id===id? p : x)); renderAdminContent(); renderProducts();
}

/* session orders: render and allow approve/reject/send */
function renderAdminOrders(){
  const out = document.getElementById('ap_orders'); out.innerHTML=''; sessionOrders.forEach(o=>{
    const d = document.createElement('div'); d.style.marginTop='8px'; d.innerHTML = `<div><strong>${o.customer.name}</strong> <span class="small">${new Date(o.date).toLocaleString()}</span></div><div class="small">Phone: ${o.customer.phone} • ${o.payment} • ₹${o.product.price} • <strong>Status: ${o.status}</strong></div><div class="small">Address: ${o.customer.village||''}, ${o.customer.city}, ${o.customer.state} - ${o.customer.pin}</div>`;
    const actions = document.createElement('div'); actions.style.marginTop='6px';
    const approve = document.createElement('button'); approve.className='btn'; approve.innerText='Approve'; approve.onclick=()=>{ o.status='approved'; renderAdminOrders(); renderChart(); };
    const reject = document.createElement('button'); reject.className='btn out'; reject.style.marginLeft='8px'; reject.innerText='Reject'; reject.onclick=()=>{ o.status='rejected'; renderAdminOrders(); renderChart(); };
    const send = document.createElement('button'); send.className='btn out'; send.style.marginLeft='8px'; send.innerText='Send WhatsApp'; send.onclick=()=>{ const msg=buildWhatsText(o); window.open(`https://wa.me/${WHATSAPP_NUMBER}?text=${msg}`,'_blank'); };
    actions.appendChild(approve); actions.appendChild(reject); actions.appendChild(send);
    d.appendChild(actions); out.appendChild(d);
  });
}

/* chart: counts from localStorage products' sold (persistent) plus sessionOrders statuses */
function renderChart(){
  const canvas = document.getElementById('sales'); if(!canvas) return; const ctx = canvas.getContext('2d'); ctx.clearRect(0,0,canvas.width,canvas.height);
  const prods = read('dw_products'); const labels = prods.map(p=>p.name.slice(0,8)); const vals = prods.map(p=>p.sold||0);
  const max = Math.max(1,...vals); const barW=26; const gap=12; canvas.width = Math.max(360,(barW+gap)*vals.length+20); const H=canvas.height;
  vals.forEach((v,i)=>{ const x=10+i*(barW+gap); const h=Math.round((v/max)*(H-30)); ctx.fillStyle='#ff6b00'; ctx.fillRect(x,H-20-h,barW,h); ctx.fillStyle='#6b7280'; ctx.font='10px sans-serif'; ctx.fillText(labels[i],x,H-4); ctx.fillText(String(v),x,H-24-h); });
}

/* view single product quick */
function viewProduct(id){ const p = read('dw_products').find(x=>x.id===id); if(!p) return alert('Not found'); alert(`${p.name}\\n\\n${p.desc}\\n\\nPrice: ₹${p.special||p.sell}\\nSize: ${p.size}\\nMaterial: ${p.material}`) }

/* utility: build whatsapp text for order */
function buildWhatsText(o){
  const encode = s=>encodeURIComponent(s);
  let txt = `New order from The Darpan Wears:%0A`;
  txt += `Product: ${o.product.name} (₹${o.product.price})%0A`;
  txt += `Name: ${o.customer.name}%0APhone: ${o.customer.phone}%0A`;
  txt += `Address: ${o.customer.village||''}, ${o.customer.city}, ${o.customer.state} - ${o.customer.pin}%0A`;
  txt += `Payment: ${o.payment}%0AOrder ID: ${o.id}`;
  return txt;
}

/* initial render */
renderProducts();
</script>
</body></html>
