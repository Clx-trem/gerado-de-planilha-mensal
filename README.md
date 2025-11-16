<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Gerador de Planilha Personalizada - CLX</title>

  <style>
    body{
      font-family: Arial, sans-serif;
      background:#03040a;
      color:white;
      margin:0;
      padding:0;
      overflow-x:hidden;
    }

    /* Neon */
    .neon{
      text-shadow:0 0 5px #00eaff,0 0 10px #00eaff,0 0 20px #00eaff;
    }

    @keyframes fadeIn{
      from{opacity:0; transform:translateY(10px);}
      to{opacity:1; transform:translateY(0);}
    }

    /* Login */
    #loginBox{
      height:100vh;
      display:flex;
      justify-content:center;
      align-items:center;
      animation:fadeIn .4s;
    }

    .card{
      background:#0b0f19;
      padding:30px;
      width:350px;
      border-radius:12px;
      box-shadow:0 0 20px #00aaff88;
    }

    input,button{
      width:100%;
      padding:12px;
      border:none;
      border-radius:6px;
      margin-top:10px;
      font-size:16px;
      transition:.2s;
    }

    input{
      background:#05060c;
      color:white;
      border:1px solid #333;
    }

    input:focus{
      box-shadow:0 0 10px #00eaff;
      outline:none;
    }

    button{
      background:#00eaff;
      color:black;
      font-weight:bold;
      cursor:pointer;
    }

    button:hover{
      transform:scale(1.03);
      box-shadow:0 0 15px #00eaff;
    }

    /* Conteúdo */
    #siteArea{
      display:none;
      padding:20px;
      animation:fadeIn .5s;
      background:white;
      color:black;
      min-height:100vh;
    }

    /* Engrenagem */
    #gear{
      position:fixed;
      top:20px;
      right:20px;
      font-size:32px;
      cursor:pointer;
      display:none;
      filter:drop-shadow(0 0 10px #00eaff);
      transition:.2s;
    }
    #gear:hover{
      transform:rotate(20deg) scale(1.1);
    }

    /* Painel admin */
    #adminPanel{
      display:none;
      position:fixed;
      right:20px;
      top:70px;
      width:350px;
      background:#0b0f19;
      padding:20px;
      border-radius:14px;
      box-shadow:0 0 25px #00aaffcc;
      animation:fadeIn .3s;
      z-index:9999;
    }

    .btnSmall{
      padding:4px 10px;
      font-size:12px;
      cursor:pointer;
      border:none;
      border-radius:4px;
    }
    .danger{background:#ff4444;color:white;}
    .warning{background:#ffaa00;color:black;}
    .ok{background:#00eaff;color:black;}

    .blocked{
      opacity:0.5;
      text-decoration:line-through;
      color:#ff5a5a;
    }
  </style>
</head>
<body>

<!-- LOGIN -->
<div id="loginBox">
  <div class="card">
    <h2 class="neon">Acesso</h2>
    <input id="usuario" placeholder="Usuário">
    <input id="senha" type="password" placeholder="Senha">
    <label style="margin-top:10px; display:flex; align-items:center; gap:6px;">
      <input type="checkbox" id="lembrar"> Lembrar login
    </label>
    <button onclick="login()">Entrar</button>
  </div>
</div>

<!-- ENGRANAGEM ADMIN -->
<div id="gear" onclick="abrirPainel()">⚙️</div>

<!-- PAINEL ADMIN -->
<div id="adminPanel">
  <h3 class="neon">Gerenciar Logins</h3>

  <h4 class="neon">Adicionar</h4>
  <input id="newUser" placeholder="Novo usuário">
  <input id="newPass" placeholder="Senha">
  <button class="ok" onclick="addUser()">Adicionar</button>

  <h4 class="neon" style="margin-top:15px;">Usuários</h4>
  <ul id="listaUsers"></ul>

  <hr style="margin:15px 0; border:1px solid #333;">

  <h4 class="neon">Trocar minha senha</h4>
  <input id="newMyPass" placeholder="Nova senha">
  <button class="warning" onclick="trocarMinhaSenha()">Trocar</button>

  <button onclick="fecharPainel()" style="margin-top:15px; width:100%; background:#ff4444; color:white;">Fechar</button>
</div>

<!-- ÁREA DO SITE -->
<div id="siteArea">
  <h1 class="neon">Gerador de Planilha Personalizada - CLX</h1>
  <p>Aqui fica TODO o conteúdo do seu site.</p>

  <button id="clxLogoutBtn" style="margin-top:30px; padding:12px 20px;">Sair</button>
</div>

<script>
/* =============================
      BANCO LOCAL
============================= */

let users = [
  {user:"CLX", pass:"02072007", admin:true, blocked:false},
];

// carregar banco salvo
function loadUsers(){
  const d = localStorage.getItem("clx_users");
  if(d) users = JSON.parse(d);
}
loadUsers();

function saveUsers(){
  localStorage.setItem("clx_users", JSON.stringify(users));
}

/* =============================
      LOGIN
============================= */

let loggedUser = null;

// carregar lembrança
const savedLogin = localStorage.getItem("clx_last_login");
if(savedLogin){
  document.getElementById("usuario").value = savedLogin;
  document.getElementById("lembrar").checked = true;
}

// entrar
function login(){
  const u = usuario.value.trim();
  const s = senha.value.trim();

  const user = users.find(x=>x.user === u && x.pass === s);

  if(!user) return alert("Login incorreto!");
  if(user.blocked) return alert("Usuário bloqueado!");

  loggedUser = user;

  // lembrar login
  if(document.getElementById("lembrar").checked){
    localStorage.setItem("clx_last_login", u);
  }else{
    localStorage.removeItem("clx_last_login");
  }

  loginBox.style.display = "none";
  siteArea.style.display = "block";

  // fundo branco
  document.body.style.background = "white";
  document.body.style.color = "black";

  if(user.admin) gear.style.display = "block";
}

/* =============================
      PAINEL ADMIN
============================= */

function abrirPainel(){
  adminPanel.style.display = "block";
  renderUsers();
}

function fecharPainel(){
  adminPanel.style.display = "none";
}

function addUser(){
  const u = newUser.value.trim();
  const p = newPass.value.trim();

  if(!u || !p) return alert("Preencha todos os campos!");

  users.push({user:u, pass:p, admin:false, blocked:false});
  saveUsers();
  renderUsers();

  newUser.value = "";
  newPass.value = "";
}

function remover(i){
  if(!confirm("Remover este usuário?")) return;
  users.splice(i,1);
  saveUsers();
  renderUsers();
}

function bloquear(i){
  users[i].blocked = !users[i].blocked;
  saveUsers();
  renderUsers();
}

function trocarMinhaSenha(){
  const nova = newMyPass.value.trim();
  if(!nova) return alert("Digite nova senha");

  loggedUser.pass = nova;
  saveUsers();
  alert("Senha alterada!");
  newMyPass.value = "";
}

function renderUsers(){
  listaUsers.innerHTML = "";

  users.forEach((u,i)=>{
    listaUsers.innerHTML += `
      <li class="${u.blocked?'blocked':''}">
        ${u.user}
        <button class="btnSmall danger" onclick="remover(${i})">❌</button>
        <button class="btnSmall warning" onclick="bloquear(${i})">${u.blocked?'🔓':'🔒'}</button>
        <button class="btnSmall ok" onclick="editarSenha(${i})">✏️</button>
      </li>
    `;
  });
}

function editarSenha(i){
  const nova = prompt("Nova senha para " + users[i].user);
  if(!nova) return;
  users[i].pass = nova;
  saveUsers();
  renderUsers();
}

/* =============================
      SAIR
============================= */

function clxLogout(){
  if(!confirm("Deseja sair?")) return;
  loggedUser = null;
  loginBox.style.display = "flex";
  siteArea.style.display = "none";
  document.body.style.background = "#03040a";
  document.body.style.color = "white";
}

// EXPOE A FUNÇÃO GLOBALMENTE
window.clxLogout = clxLogout;

</script>

</body>
</html>
