
<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>রঙচিল Admin Panel</title>
<style>
body{font-family:Arial;background:#f5f0fa;padding:20px;color:#222}
.container{max-width:500px;margin:auto;background:white;padding:20px;border-radius:12px}
h2{text-align:center;color:#7027a0}
input,textarea,button{width:100%;padding:12px;margin:8px 0;box-sizing:border-box}
button{background:#7027a0;color:white;border:0;border-radius:6px}
.hidden{display:none}
.product{border:1px solid #ddd;padding:12px;margin:10px 0}
.product img{max-width:100%;max-height:150px}
</style>
</head>
<body>
<div class="container">
<h2>রঙচিল Admin Panel</h2>

<div id="login">
<h3>Admin Login</h3>
<input id="email" type="email" placeholder="Gmail">
<input id="password" type="password" placeholder="Password">
<button id="loginBtn">লগইন করুন</button>
<p id="loginMsg"></p>
</div>

<div id="panel" class="hidden">
<p>Admin হিসেবে লগইন সফল হয়েছে।</p>
<button id="logoutBtn">লগআউট</button>
<hr>
<h3>নতুন পণ্য যোগ করুন</h3>
<input id="name" placeholder="পণ্যের নাম">
<textarea id="description" placeholder="বিবরণ"></textarea>
<input id="price" type="number" placeholder="দাম">
<input id="category" placeholder="ক্যাটাগরি">
<input id="photo" type="file" accept="image/*">
<button id="addBtn">পণ্য যোগ করুন</button>
<p id="msg"></p>
<hr>
<h3>পণ্যের তালিকা</h3>
<div id="products"></div>
</div>
</div>

<script type="module">
import { createClient } from
'https://esm.sh/@supabase/supabase-js@2';

const SUPABASE_URL =
'https://yjzqtdnpgamuivxxrukv.supabase.co';

const SUPABASE_KEY = 'sb_publishable_HkzSNDAO1Hpqenbnqah0Kg_0lFSJKu8';

const loginMsg = document.getElementById('loginMsg');
const msg = document.getElementById('msg');

const supabase = createClient(SUPABASE_URL, SUPABASE_KEY);

document.getElementById('loginBtn').onclick = async () => {
  loginMsg.textContent = 'লগইন হচ্ছে...';
  try {
    const email = document.getElementById('email').value.trim();
    const password = document.getElementById('password').value;

    const { error } = await supabase.auth.signInWithPassword({
      email, password
    });

    if (error) throw error;

    document.getElementById('login').classList.add('hidden');
    document.getElementById('panel').classList.remove('hidden');
    await loadProducts();
  } catch (e) {
    loginMsg.textContent = 'লগইন হয়নি: ' + e.message;
  }
};

document.getElementById('logoutBtn').onclick = async () => {
  await supabase.auth.signOut();
  document.getElementById('panel').classList.add('hidden');
  document.getElementById('login').classList.remove('hidden');
};

document.getElementById('addBtn').onclick = async () => {
  msg.textContent = 'পণ্য যোগ হচ্ছে...';
  try {
    const name = document.getElementById('name').value.trim();
    const description = document.getElementById('description').value.trim();
    const price = Number(document.getElementById('price').value || 0);
    const category = document.getElementById('category').value.trim();
    const photo = document.getElementById('photo').files[0];

    if (!name) throw new Error('পণ্যের নাম লিখুন।');

    let image_url = null;

    if (photo) {
      const filename = Date.now() + '-' +
        photo.name.replace(/[^a-zA-Z0-9._-]/g, '_');

      const { error } = await supabase.storage
        .from('product-images').upload(filename, photo);

      if (error) throw error;

      image_url = supabase.storage
        .from('product-images').getPublicUrl(filename).data.publicUrl;
    }

    const { error } = await supabase.from('products').insert([{
      name, description, price, category, image_url
    }]);

    if (error) throw error;

    msg.textContent = 'পণ্য যোগ হয়েছে!';
    await loadProducts();
  } catch (e) {
    msg.textContent = 'সমস্যা: ' + e.message;
  }
};

async function loadProducts() {
  const box = document.getElementById('products');
  box.textContent = 'লোড হচ্ছে...';

  try {
    const { data, error } = await supabase.from('products')
      .select('*').order('created_at', { ascending: false });

    if (error) throw error;

    box.innerHTML = '';

    if (!data.length) {
      box.textContent = 'এখনো কোনো পণ্য নেই।';
      return;
    }

    data.forEach(p => {
      const div = document.createElement('div');
      div.className = 'product';

      const title = document.createElement('h4');
      title.textContent = p.name;
      div.appendChild(title);

      if (p.image_url) {
        const img = document.createElement('img');
        img.src = p.image_url;
        div.appendChild(img);
      }

      const details = document.createElement('p');
      details.textContent = 'দাম: ' + p.price + ' টাকা';
      div.appendChild(details);
      box.appendChild(div);
    });
  } catch (e) {
    box.textContent = 'পণ্য লোড হয়নি: ' + e.message;
  }
}
</script>
</body>
</html>
