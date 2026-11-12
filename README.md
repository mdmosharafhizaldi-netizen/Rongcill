
<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>রঙচিল | Rongcill Online Shop</title>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:#faf7f2;color:#292929}
header{background:#fff;padding:16px;text-align:center;box-shadow:0 2px 8px #ddd}
.logo{font-size:30px;font-weight:bold;color:#a33c65}
header p{margin:5px;color:#777}
nav{display:flex;justify-content:center;gap:10px;padding:12px;flex-wrap:wrap}
button{border:0;border-radius:8px;padding:11px 16px;background:#a33c65;color:white;font-weight:bold;cursor:pointer}
button.secondary{background:#555}
button.danger{background:#c0392b}
input,textarea,select{width:100%;padding:12px;margin:6px 0 12px;border:1px solid #ddd;border-radius:8px;font-size:15px}
.container{max-width:1050px;margin:auto;padding:15px}
.hero{background:#f1dce5;padding:30px 18px;text-align:center;border-radius:14px;margin:10px 0 20px}
.hero h1{color:#8d3156}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:15px}
.card,.panel{background:white;border-radius:12px;padding:14px;box-shadow:0 2px 9px #0000000c}
.card img{width:100%;height:190px;object-fit:cover;border-radius:9px}
.card video{width:100%;max-height:190px;border-radius:9px}
.price{font-size:20px;color:#a33c65;font-weight:bold}
.small{font-size:13px;color:#777}
.hidden{display:none!important}
#admin{background:#fff;border-radius:12px;padding:18px;margin-top:25px}
#admin h2{color:#a33c65}
.order{border:1px solid #eee;padding:12px;border-radius:9px;margin:10px 0}
footer{text-align:center;padding:25px;color:#777}
.actions{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px}
</style>
</head>
<body>

<header>
 <div class="logo">রঙচিল | Rongcill</div>
 <p>আপনার পছন্দ, আপনার অনলাইন শপ</p>
</header>

<nav>
 <button onclick="showShop()">হোম</button>
 <button onclick="document.getElementById('admin').classList.toggle('hidden')">Admin Panel</button>
 <button onclick="window.open('https://wa.me/8801611385067','_blank')">WhatsApp</button>
</nav>

<main class="container">
 <section class="hero">
  <h1>রঙচিলে আপনাকে স্বাগতম</h1>
  <p>পছন্দের পণ্য কিনুন সহজেই।</p>
  <button onclick="document.getElementById('products').scrollIntoView()">পণ্য দেখুন</button>
 </section>

 <h2>আমাদের পণ্য</h2>
 <input id="search" placeholder="পণ্যের নাম দিয়ে খুঁজুন..." oninput="renderProducts()">
 <div id="products" class="grid">পণ্য লোড হচ্ছে...</div>

 <section id="admin" class="hidden">
  <h2>রঙচিল Admin Panel</h2>

  <div id="loginBox">
   <h3>Admin Login</h3>
   <input id="email" type="email" placeholder="আপনার Gmail">
   <input id="password" type="password" placeholder="Password">
   <button onclick="login()">লগইন করুন</button>
   <p id="loginMsg"></p>
  </div>

  <div id="dashboard" class="hidden">
   <p>অ্যাডমিন হিসেবে লগইন করা হয়েছে।</p>
   <button class="secondary" onclick="logout()">Logout</button>
   <hr>
   <h3 id="formTitle">নতুন পণ্য যোগ করুন</h3>
   <input id="pname" placeholder="পণ্যের নাম">
   <textarea id="pdesc" placeholder="পণ্যের বিবরণ"></textarea>
   <input id="pprice" type="number" min="0" placeholder="পণ্যের দাম">
   <input id="pstock" type="number" min="0" placeholder="স্টক সংখ্যা">
   <input id="pcat" placeholder="ক্যাটাগরি">
   <label>পণ্যের ছবি</label>
   <input id="pimage" type="file" accept="image/*">
   <label>পণ্যের ভিডিও</label>
   <input id="pvideo" type="file" accept="video/*">
   <button onclick="saveProduct()">পণ্য সেভ করুন</button>
   <button class="secondary" onclick="resetForm()">নতুন পণ্য</button>
   <p id="productMsg"></p>

   <hr>
   <h3>পণ্য পরিচালনা</h3>
   <div id="adminProducts"></div>
   <hr>
   <h3>ক্রেতাদের অর্ডার</h3>
   <button onclick="loadOrders()">অর্ডার রিফ্রেশ করুন</button>
   <div id="orders"></div>
  </div>
 </section>
</main>

<footer>© 2026 Rongcill | All Rights Reserved</footer>

<script>
const SUPABASE_URL = 'https://fstxhzsvvzthqqcisfdz.supabase.co';
const SUPABASE_KEY = 'sb_publishable_k8cAOGYerAOOFsf_UghhyQ_WfVu8buP';
const ADMIN_UID = 'd2e25a48-3ecf-4878-b935-3e56db23b8b0';
const BUCKET = 'product-media';
const db = window.supabase.createClient(SUPABASE_URL,SUPABASE_KEY);

let products = [];
let editingId = null;
let isAdmin = false;

function safe(value){
 return String(value ?? '').replace(/[&<>"']/g,c=>({
  '&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'
 }[c]));
}

async function loadProducts(){
 const {data,error}=await db.from('products')
  .select('*').eq('is_active',true).order('created_at',{ascending:false});
 if(error){
  document.getElementById('products').textContent='পণ্য লোড করা যায়নি।';
  console.error(error);
  return;
 }
 products=data||[];
 renderProducts();
 if(isAdmin) renderAdminProducts();
}

function renderProducts(){
 const q=document.getElementById('search').value.toLowerCase();
 const list=products.filter(p=>
  (p.name||'').toLowerCase().includes(q)||
  (p.category||'').toLowerCase().includes(q)
 );
 const box=document.getElementById('products');
 if(!list.length){
  box.textContent='এখনো কোনো পণ্য যোগ করা হয়নি।';
  return;
 }
 box.innerHTML=list.map(p=>`
 <article class="card">
  ${p.image_url?`<img src="${safe(p.image_url)}" alt="${safe(p.name)}">`:''}
  ${p.video_url?`<video src="${safe(p.video_url)}" controls></video>`:''}
  <h3>${safe(p.name)}</h3>
  <p>${safe(p.description)}</p>
  <div class="price">৳ ${Number(p.price).toLocaleString('en-BD')}</div>
  <p class="small">স্টক: ${Number(p.stock)}</p>
  <button onclick="placeOrder(${p.id})">অর্ডার করুন</button>
  <button class="secondary" onclick="whatsappOrder(${p.id})">WhatsApp</button>
 </article>`).join('');
}

function whatsappOrder(id){
 const p=products.find(x=>x.id===id);
 if(!p)return;
 const msg=`আমি ${p.name} অর্ডার করতে চাই। দাম: ${p.price} টাকা।`;
 window.open('https://wa.me/8801611385067?text='+encodeURIComponent(msg),'_blank');
}

async function placeOrder(id){
 const p=products.find(x=>x.id===id);
 if(!p)return;
 if(Number(p.stock)<=0){alert('দুঃখিত, পণ্যটি স্টকে নেই।');return;}
 const name=prompt('আপনার নাম লিখুন:');
 if(!name)return;
 const phone=prompt('আপনার মোবাইল নম্বর লিখুন:');
 if(!phone)return;
 const address=prompt('আপনার সম্পূর্ণ ঠিকানা লিখুন:');
 if(!address)return;
 const quantityText=prompt('কতটি পণ্য নিতে চান?', '1');
 const quantity=Number(quantityText);
 if(!Number.isInteger(quantity)||quantity<1||quantity>Number(p.stock)){
  alert('সঠিক পরিমাণ লিখুন।');
  return;
 }
 const total=Number(p.price)*quantity;
 const {error}=await db.from('orders').insert({
  customer_name:name.trim(),
  phone:phone.trim(),
  address:address.trim(),
  product_id:p.id,
  quantity:quantity,
  total_amount:total,
  status:'Pending'
 });
 if(error){
  alert('অর্ডার জমা হয়নি: '+error.message);
  return;
 }
 alert('আপনার অর্ডার গ্রহণ করা হয়েছে। ধন্যবাদ!');
}

async function login(){
 const email=document.getElementById('email').value.trim();
 const password=document.getElementById('password').value;
 const msg=document.getElementById('loginMsg');
 msg.textContent='লগইন হচ্ছে...';
 const {data,error}=await db.auth.signInWithPassword({email,password});
 if(error){msg.textContent='লগইন হয়নি: '+error.message;return;}
 if(data.user.id!==ADMIN_UID){
  await db.auth.signOut();
  msg.textContent='এই অ্যাকাউন্ট অ্যাডমিন নয়।';
  return;
 }
 await checkSession();
}

async function logout(){
 await db.auth.signOut();
 isAdmin=false;
 document.getElementById('dashboard').classList.add('hidden');
 document.getElementById('loginBox').classList.remove('hidden');
}

async function checkSession(){
 const {data}=await db.auth.getSession();
 const user=data.session?.user;
 isAdmin=!!user&&user.id===ADMIN_UID;
 document.getElementById('dashboard').classList.toggle('hidden',!isAdmin);
 document.getElementById('loginBox').classList.toggle('hidden',isAdmin);
 if(isAdmin){
  await loadProducts();
  await loadOrders();
  renderAdminProducts();
 }
}

async function uploadFile(file,kind){
 if(!file)return null;
 const ext=file.name.split('.').pop().replace(/[^a-zA-Z0-9]/g,'');
 const path=kind+'/'+Date.now()+'-'+Math.random().toString(36).slice(2)+'.'+ext;
 const {error}=await db.storage.from(BUCKET).upload(path,file,{upsert:false});
 if(error)throw error;
 return db.storage.from(BUCKET).getPublicUrl(path).data.publicUrl;
}

async function saveProduct(){
 if(!isAdmin)return;
 const msg=document.getElementById('productMsg');
 msg.textContent='সেভ হচ্ছে...';
 try{
  const name=document.getElementById('pname').value.trim();
  const price=Number(document.getElementById('pprice').value);
  const stock=Number(document.getElementById('pstock').value);
  if(!name||!Number.isFinite(price)||price<0||!Number.isInteger(stock)||stock<0){
   msg.textContent='নাম, সঠিক দাম ও স্টক লিখুন।';return;
  }
  const imageFile=document.getElementById('pimage').files[0];
  const videoFile=document.getElementById('pvideo').files[0];
  let image_url=null,video_url=null;
  if(imageFile)image_url=await uploadFile(imageFile,'images');
  if(videoFile)video_url=await uploadFile(videoFile,'videos');

  const item={
   name:name,
   description:document.getElementById('pdesc').value.trim(),
   price:price,
   stock:stock,
   category:document.getElementById('pcat').value.trim(),
   is_active:true
  };
  if(image_url)item.image_url=image_url;
  if(video_url)item.video_url=video_url;

  let result;
  if(editingId){
   result=await db.from('products').update(item).eq('id',editingId);
  }else{
   result=await db.from('products').insert(item);
  }
  if(result.error)throw result.error;
  msg.textContent='পণ্য সফলভাবে সেভ হয়েছে।';
  resetForm();
  await loadProducts();
 }catch(e){
  msg.textContent='সমস্যা হয়েছে: '+e.message;
 }
}

function resetForm(){
 editingId=null;
 ['pname','pdesc','pprice','pstock','pcat','pimage','pvideo'].forEach(id=>{
  document.getElementById(id).value='';
 });
 document.getElementById('formTitle').textContent='নতুন পণ্য যোগ করুন';
}

function renderAdminProducts(){
 const box=document.getElementById('adminProducts');
 if(!isAdmin)return;
 box.innerHTML=products.map(p=>`
 <div class="order">
 <b>${safe(p.name)}</b> — ৳${safe(p.price)}
 <p class="small">স্টক: ${safe(p.stock)}</p>
 <div class="actions">
 <button onclick="editProduct(${p.id})">Edit</button>
 <button class="danger" onclick="deleteProduct(${p.id})">Delete</button>
 </div></div>`).join('')||'কোনো পণ্য নেই।';
}

function editProduct(id){
 const p=products.find(x=>x.id===id);
 if(!p)return;
 editingId=id;
 document.getElementById('pname').value=p.name||'';
 document.getElementById('pdesc').value=p.description||'';
 document.getElementById('pprice').value=p.price;
 document.getElementById('pstock').value=p.stock;
 document.getElementById('pcat').value=p.category||'';
 document.getElementById('formTitle').textContent='পণ্য সম্পাদনা';
 document.getElementById('admin').scrollIntoView();
}

async function deleteProduct(id){
 if(!isAdmin||!confirm('এই পণ্যটি মুছে ফেলবেন?'))return;
 const {error}=await db.from('products').delete().eq('id',id);
 if(error){alert('মুছতে সমস্যা: '+error.message);return;}
 await loadProducts();
}

async function loadOrders(){
 if(!isAdmin)return;
 const box=document.getElementById('orders');
 box.textContent='অর্ডার লোড হচ্ছে...';
 const {data,error}=await db.from('orders').select('*').order('created_at',{ascending:false});
 if(error){box.textContent='অর্ডার লোড হয়নি: '+error.message;return;}
 if(!data.length){box.textContent='এখনো কোনো অর্ডার নেই।';return;}
 box.innerHTML=data.map(o=>`
 <div class="order">
 <b>অর্ডার #${safe(o.id)}</b>
 <p>ক্রেতা: ${safe(o.customer_name)}</p>
 <p>ফোন: ${safe(o.phone)}</p>
 <p>ঠিকানা: ${safe(o.address)}</p>
 <p>পণ্য ID: ${safe(o.product_id)} | পরিমাণ: ${safe(o.quantity)}</p>
 <p>মোট: ৳${safe(o.total_amount)}</p>
 <select id="status-${o.id}">
 <option ${o.status==='Pending'?'selected':''}>Pending</option>
 <option ${o.status==='Confirmed'?'selected':''}>Confirmed</option>
 <option ${o.status==='Shipped'?'selected':''}>Shipped</option>
 <option ${o.status==='Delivered'?'selected':''}>Delivered</option>
 <option ${o.status==='Cancelled'?'selected':''}>Cancelled</option>
 </select>
 <button onclick="updateOrder(${o.id})">স্ট্যাটাস সেভ</button>
 </div>`).join('');
}

async function updateOrder(id){
 if(!isAdmin)return;
 const status=document.getElementById('status-'+id).value;
 const {error}=await db.from('orders').update({status}).eq('id',id);
 if(error){alert('আপডেট হয়নি: '+error.message);return;}
 alert('অর্ডার আপডেট হয়েছে।');
 await loadOrders();
}

function showShop(){
 document.getElementById('products').scrollIntoView({behavior:'smooth'});
}

db.auth.onAuthStateChange(()=>{
 setTimeout(checkSession,0);
});

checkSession();
loadProducts();
</script>
</body>
</html>
