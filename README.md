
<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>রঙচিল Admin Panel</title>
<style>
body{font-family:Arial,sans-serif;background:#f5f0fa;margin:0;padding:20px;color:#222}
.container{max-width:500px;margin:auto;background:white;padding:20px;border-radius:12px}
h2{color:#7027a0;text-align:center}
input,textarea,button{width:100%;padding:12px;margin:8px 0;box-sizing:border-box;border:1px solid #ccc;border-radius:6px}
button{background:#7027a0;color:white;border:0;font-size:16px}
.product{padding:12px;border:1px solid #ddd;margin:12px 0;border-radius:8px}
.product img{max-width:100%;max-height:150px;object-fit:contain}
.hidden{display:none}
</style>
</head>
<body>
<div class="container">
<h2>রঙচিল Admin Panel</h2>

<div id="login">
<h3>Admin Login</h3>
<input id="email" type="email" placeholder="আপনার Gmail">
<input id="password" type="password" placeholder="পাসওয়ার্ড">
<button id="loginBtn">লগইন করুন</button>
<p id="loginMsg"></p>
</div>

<div id="panel" class="hidden">
<p>আপনি Admin হিসেবে লগইন করেছেন।</p>
<button id="logoutBtn">লগআউট</button>
<hr>
<h3>নতুন পণ্য যোগ করুন</h3>
<input id="name" placeholder="পণ্যের নাম">
<textarea id="description" placeholder="পণ্যের বিবরণ"></textarea>
<input id="price" type="number" placeholder="পণ্যের দাম">
<input id="category" placeholder="ক্যাটাগরি">
<label>পণ্যের ছবি নির্বাচন করুন</label>
<input id="photo" type="file" accept="image/*">
<button id="addBtn">পণ্য যোগ করুন</button>
<p id="msg"></p>
<hr>
<h3>পণ্যের তালিকা</h3>
<div id="products">লোড হচ্ছে...</div>
</div>
</div>

<script type="module">
import { createClient } from
'https://esm.sh/@supabase/supabase-js@2';

const SUPABASE_URL =
'https://yjzqtdnpgamuivxxrukv.supabase.co';

// আপনার আগের Publishable Key এখানে বসান
const SUPABASE_KEY = 'এখানে_আপনার_আগের_Key';

const loginMsg = document.getElementById('loginMsg');
const msg = document.getElementById('msg');
const loginBtn = document.getElementById('loginBtn');

let supabase;

try {
  if (!/^sb_publishable_[A-Za-z0-9_-]+$/.test(SUPABASE_KEY)) {
    throw new Error('Publishable Key সঠিকভাবে বসান।');
  }
const diagnosticFetch = async (url, options = {}) => {
  const headers = options.headers || {};
  const entries = headers instanceof Headers
    ? [...headers.entries()]
    : Object.entries(headers);

  const bad = entries.filter(([key, value]) =>
    /[^\x00-\xFF]/.test(String(value))
  );

  if (bad.length) {
    throw new Error(
      'সমস্যাযুক্ত Header: ' + bad.map(item => item[0]).join(', ')
    );
  }

  return fetch(url, options);
};

supabase = createClient(SUPABASE_URL, SUPABASE_KEY, {
  global: { fetch: diagnosticFetch }
});
} catch (error) {
  loginMsg.textContent = error.message;
  loginBtn.disabled = true;
}

async function showPanel() {
  document.getElementById('login').classList.add('hidden');
  document.getElementById('panel').classList.remove('hidden');
  await loadProducts();
}

loginBtn.addEventListener('click', async () => {
  if (!supabase) return;

  const email = document.getElementById('email').value.trim();
  const password = document.getElementById('password').value;

  if (!email || !password) {
    loginMsg.textContent = 'Gmail ও Password লিখুন।';
    return;
  }

  loginMsg.textContent = 'লগইন হচ্ছে...';

  try {
    const { error } = await supabase.auth.signInWithPassword({
      email: email,
      password: password
    });

    if (error) throw error;
    loginMsg.textContent = '';
    await showPanel();
  } catch (error) {
    loginMsg.textContent = 'লগইন হয়নি: ' + error.message;
  }
});

document.getElementById('logoutBtn').addEventListener('click', async () => {
  try {
    const { error } = await supabase.auth.signOut();
    if (error) throw error;

    document.getElementById('panel').classList.add('hidden');
    document.getElementById('login').classList.remove('hidden');
    loginMsg.textContent = 'লগআউট হয়েছে।';
  } catch (error) {
    loginMsg.textContent = error.message;
  }
});

document.getElementById('addBtn').addEventListener('click', async () => {
  if (!supabase) return;
  msg.textContent = 'পণ্য যোগ হচ্ছে...';

  try {
    const name = document.getElementById('name').value.trim();
    const description = document.getElementById('description').value.trim();
    const price = Number(document.getElementById('price').value || 0);
    const category = document.getElementById('category').value.trim();
    const photo = document.getElementById('photo').files[0];

    if (!name) {
      msg.textContent = 'পণ্যের নাম লিখুন।';
      return;
    }

    let image_url = null;

    if (photo) {
      const safeName = Date.now() + '-' +
        photo.name.replace(/[^a-zA-Z0-9._-]/g, '_');

      const { error } = await supabase.storage
        .from('product-images').upload(safeName, photo);

      if (error) throw error;

      image_url = supabase.storage
        .from('product-images')
        .getPublicUrl(safeName).data.publicUrl;
    }

    const { error } = await supabase.from('products').insert([{
      name, description, price, category, image_url
    }]);

    if (error) throw error;

    msg.textContent = 'পণ্য সফলভাবে যোগ হয়েছে!';
    document.getElementById('name').value = '';
    document.getElementById('description').value = '';
    document.getElementById('price').value = '';
    document.getElementById('category').value = '';
    document.getElementById('photo').value = '';

    await loadProducts();
  } catch (error) {
    msg.textContent = 'পণ্য যোগ হয়নি: ' + error.message;
  }
});

async function loadProducts() {
  const box = document.getElementById('products');
  box.textContent = 'পণ্য লোড হচ্ছে...';

  try {
    const { data, error } = await supabase
      .from('products')
      .select('*')
      .order('created_at', { ascending: false });

    if (error) throw error;

    box.innerHTML = '';

    if (!data.length) {
      box.textContent = 'এখনো কোনো পণ্য যোগ করা হয়নি।';
      return;
    }

    data.forEach(p => {
      const item = document.createElement('div');
      item.className = 'product';

      const title = document.createElement('h4');
      title.textContent = p.name;
      item.appendChild(title);

      if (p.image_url) {
        const img = document.createElement('img');
        img.src = p.image_url;
        img.alt = p.name;
        item.appendChild(img);
      }

      const details = document.createElement('p');
      details.textContent = 'দাম: ' + p.price + ' টাকা';
      item.appendChild(details);

      const desc = document.createElement('p');
      desc.textContent = p.description || '';
      item.appendChild(desc);

      box.appendChild(item);
    });
  } catch (error) {
    box.textContent = 'পণ্য লোড হয়নি: ' + error.message;
  }
}

if (supabase) {
  try {
    const { data, error } = await supabase.auth.getSession();
    if (error) throw error;
    if (data.session) await showPanel();
  } catch (error) {
    loginMsg.textContent = error.message;
  }
}
</script>
</body>
</html>
