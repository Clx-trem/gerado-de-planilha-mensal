<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Gerador de Planilha Personalizada - CLX</title>

  <style>
    /* ---------- LOGIN / ADMIN (NEON) ---------- */
    :root{ --neon:#00eaff; --bg:#03040a; --white:#ffffff; }
    body.login-active { overflow:hidden; background:var(--bg); color:var(--white); }

    #clxLoginOverlay{
      position:fixed; inset:0;
      display:flex; align-items:center; justify-content:center;
      background: linear-gradient(135deg, rgba(2,6,23,0.95), rgba(4,10,30,0.95));
      z-index:99999; font-family:Arial,Helvetica,sans-serif;
    }
    #clxLoginCard{
      width:360px; border-radius:12px; padding:20px;
      background: linear-gradient(180deg,#071223,#081426);
      box-shadow:0 12px 40px rgba(0,160,255,0.12);
      border:1px solid rgba(0,160,255,0.12);
    }
    #clxLoginCard h2{ color:var(--neon); margin:0 0 12px; text-shadow:0 0 6px rgba(0,234,255,0.45); }
    #clxLoginCard input{ width:100%; padding:10px; margin-top:8px; border-radius:8px; border:1px solid rgba(255,255,255,0.06); background:#05111a; color:#e6f7ff; outline:none; }
    #clxLoginCard button{ width:100%; padding:10px; margin-top:12px; border-radius:8px; border:none; cursor:pointer; background:var(--neon); color:#00202a; font-weight:700; }

    #clxGear{ position:fixed; top:18px; right:18px; font-size:28px; cursor:pointer; display:none; z-index:99990; filter:drop-shadow(0 0 10px rgba(0,234,255,0.6)); }
    #clxGear:hover{ transform:rotate(20deg) scale(1.06); transition:.18s; }

    #clxLogoutBtn{ position:fixed; left:18px; top:18px; padding:8px 12px; border-radius:8px; border:none; cursor:pointer; background:#ff3747; color:white; z-index:99990; display:none; }

    /* admin modal */
    #clxAdminModal{ position:fixed; inset:0; display:none; align-items:center; justify-content:center; background:rgba(0,0,0,0.45); z-index:99998; }
    #clxAdminCard{ width:520px; border-radius:12px; padding:16px; background:linear-gradient(180deg,#071126,#06111a); color:white; border:1px solid rgba(0,160,255,0.12); box-shadow:0 12px 40px rgba(0,160,255,0.08); max-height:85vh; overflow:auto; }
    #clxAdminCard h3{ color:var(--neon); margin:0 0 10px; }
    #clxAdminCard input{ width:100%; padding:8px; margin:6px 0; border-radius:8px; border:1px solid rgba(255,255,255,0.06); background:#05111a; color:#e6f7ff; }

    .clxUserRow{ display:flex; gap:8px; align-items:center; justify-content:space-between; padding:8px; border-radius:8px; background:rgba(0,0,0,0.12); margin-bottom:8px; border:1px solid rgba(0,160,255,0.04); }
    .clxUserRow.blocked{ opacity:.45; text-decoration:line-through; color:#ff9b9b; }
    .clxBtn{ padding:6px 10px; border-radius:6px; border:none; cursor:pointer; font-weight:600; }
    .clxBtn.delete{ background:#ff3747;color:white; }
    .clxBtn.block{ background:#ffaa00;color:#08111a; }
    .clxBtn.edit{ background:var(--neon); color:#021214; }

    /* ---------- YOUR ORIGINAL SITE CSS (kept) ---------- */
    :root{
      --bg: #0d1117;
      --card: #161b22;
      --muted: #94a3b8;
      --accent: #238636; /* botões azuis-esverdeados */
      --accent-2: #0ea5e9;
      --white: #ffffff;
      --input-bg: #0f1720;
      --border: #26303a;
      --shadow: rgba(0,0,0,0.6);
    }

    /* Estilo base (tema claro por padrão) */
    body.site-visible {
      font-family: Arial, sans-serif;
      background: #f8fafc;
      color: #0b1220;
      max-width: 1000px;
      margin: 28px auto;
      padding: 20px;
      transition: background 0.25s, color 0.25s;
    }
    h1 { text-align:center; margin-bottom:18px; }
    label { display:block; margin-top:10px; font-weight:700; color:inherit; }
    input, select, button, textarea {
      padding:8px; border-radius:6px; border:1px solid #cbd5e1; margin-top:6px;
      background: #fff; color: inherit;
    }
    button { cursor:pointer; background: var(--accent); color:#fff; border:none; margin-top:12px; margin-right:8px; }
    button:hover { filter:brightness(0.95); }
    .months { display:grid; grid-template-columns: repeat(auto-fill, minmax(140px, 1fr)); gap:6px; margin-top:6px; }
    .preview { margin-top:18px; background:#fff; padding:12px; border:1px solid #e2e8f0; border-radius:10px; }
    table { width:100%; border-collapse:collapse; margin-top:8px; }
    th, td { border:1px solid #d1d5db; padding:8px; font-size:14px; text-align:left; }
    th { background:#f1f5f9; font-weight:700; }
    #colunasContainer input { display:block; width:100%; margin-top:6px; }
    .modelos { display:flex; gap:10px; align-items:center; flex-wrap:wrap; margin-top:10px; }
    footer { text-align:center; margin-top:28px; font-size:13px; color:#555; }

    /* Tema escuro (opção A aplicada por padrão quando ativado) */
    body.dark { background: var(--bg); color: var(--white); }
    body.dark .preview { background: var(--card); border-color: var(--border); box-shadow: 0 6px 18px var(--shadow); }
    body.dark input, body.dark select, body.dark textarea { background: var(--input-bg); color: var(--white); border:1px solid var(--border); }
    body.dark th { background: rgba(255,255,255,0.04); }
    body.dark table { color: var(--white); }
    body.dark button { background: var(--accent); color: #fff; border:none; }

    @media (max-width:700px){
      .months { grid-template-columns: repeat(2, 1fr); }
      body { padding:14px; margin:12px; }
      #clxAdminCard { width: 92%; }
    }
  </style>

  <!-- usar xlsx-js-style que suporta estilos -->
  <script src="https://cdn.jsdelivr.net/npm/xlsx-js-style@1.2.0/dist/xlsx.min.js"></script>
</head>
<body class="login-active">

  <!-- LOGIN OVERLAY -->
  <div id="clxLoginOverlay">
    <div id="clxLoginCard">
      <h2>Acesso</h2>
      <input id="clxUserInput" placeholder="Usuário (ex: admin)">
      <input id="clxPassInput" type="password" placeholder="Senha">
      <label style="display:flex;gap:8px;align-items:center;margin-top:8px">
        <input id="clxRemember" type="checkbox"> Lembrar login
      </label>
      <button id="clxLoginBtn">Entrar</button>
      <p id="clxLoginError" style="color:#ff8b8b;margin-top:8px;display:none;">Usuário ou senha incorretos.</p>
    </div>
  </div>

  <!-- GEAR + LOGOUT -->
  <div id="clxGear">⚙️</div>
  <button id="clxLogoutBtn" style="display:none;">Sair</button>

  <!-- ADMIN MODAL -->
  <div id="clxAdminModal">
    <div id="clxAdminCard">
      <h3>Painel Administrativo</h3>

      <div style="display:flex;gap:12px;align-items:center;justify-content:space-between;">
        <div>
          <strong>Usuários cadastrados</strong>
          <div id="clxUserList" style="margin-top:8px; max-height:250px; overflow:auto;"></div>
        </div>

        <div style="min-width:180px;">
          <strong>Salvar / Banco (simulado)</strong>
          <div style="margin-top:6px;">
            <button id="clxSaveDb" class="clxBtn" style="width:100%; background:#06b6d4;">Salvar no "Banco"</button>
            <button id="clxLoadDb" class="clxBtn" style="width:100%; margin-top:6px; background:#10b981;">Carregar do "Banco"</button>
            <button id="clxExportDb" class="clxBtn" style="width:100%; margin-top:6px; background:#7c3aed;">Exportar JSON</button>
            <input id="clxImportFile" type="file" accept=".json" style="display:none; margin-top:6px;">
            <button id="clxImportBtn" class="clxBtn" style="width:100%; margin-top:6px; background:#fb923c;">Importar JSON</button>
          </div>
        </div>
      </div>

      <hr style="border-color:rgba(255,255,255,0.06); margin:12px 0;">

      <div>
        <strong>Adicionar usuário</strong>
        <input id="clxNewUser" placeholder="Usuário">
        <input id="clxNewPass" placeholder="Senha">
        <label style="font-size:12px;opacity:0.85;margin-top:4px;"><input id="clxNewAdmin" type="checkbox"> Tornar administrador</label>
        <button id="clxAddBtn" class="clxBtn" style="background:var(--neon);color:#021214;width:100%;margin-top:8px;">Adicionar</button>
      </div>

      <hr style="border-color:rgba(255,255,255,0.06); margin:12px 0;">

      <div>
        <strong>Minha conta (logado)</strong><br>
        <input id="clxChangePass" placeholder="Nova senha (sua)">
        <button id="clxChangeMyPassBtn" class="clxBtn" style="background:#ffaa00;width:100%;margin-top:8px;">Trocar minha senha</button>
      </div>

      <button id="clxCloseAdmin" style="width:100%;margin-top:12px;background:#ff3747;color:white;padding:8px;border-radius:8px;border:none;">Fechar painel</button>
    </div>
  </div>

  <!-- SEU SITE ORIGINAL -->
  <div id="clxSiteWrapper" style="display:none;">
    <div class="site-wrapper">
      <h1>📅 Gerador de Planilha Personalizada — CLX</h1>

      <label for="ano">Ano:</label>
      <input id="ano" type="number" min="2000" max="2100" value="2025">

      <label>Selecione os meses:</label>
      <div class="months" id="meses"></div>

      <label for="qtdAbas">Quantidade de abas:</label>
      <input id="qtdAbas" type="number" min="1" max="12" value="1">

      <label>Colunas da planilha:</label>
      <div id="colunasContainer">
        <input type="text" value="Data">
        <input type="text" value="Dia da Semana">
        <input type="text" value="Atividade">
        <input type="text" value="Observações">
      </div>

      <button id="addColunaBtn" style="background:#0ea5e9;">+ Adicionar Coluna</button>

      <div class="modelos">
        <select id="modeloSelect"><option value="">-- Escolher modelo salvo --</option></select>
        <button id="salvarModeloBtn" style="background:#10b981;">Salvar Modelo</button>
        <button id="carregarModeloBtn" style="background:#f59e0b; color:#000;">Carregar</button>
        <button id="duplicarModeloBtn" style="background:#7c3aed;">Duplicar</button>
        <button id="excluirModeloBtn" style="background:#ef4444;">Excluir</button>
        <button id="exportarBtn" style="background:#06b6d4;">Exportar Modelos</button>
        <button id="importarBtn" style="background:#fb923c;">Importar Modelos</button>
        <input id="importFile" type="file" accept=".json" style="display:none">
      </div>

      <div class="modelos" style="margin-top:14px;">
        <button id="modeloSimplesBtn" style="background:#0d6efd;">Modelo Simples</button>
        <button id="modeloProBtn" style="background:#6610f2;">Modelo Profissional</button>
      </div>

      <div style="margin-top:12px;">
        <button id="gerarBtn">Gerar XLSX</button>
        <button id="previewBtn" style="background:#16a34a;">Ver Exemplo</button>
        <button id="gradearBtn" style="background:#6b7280;">Gradeie (Preview)</button>
        <button id="bordasBtn" style="background:#111827;">Bordas Excel</button>
        <button id="congelarBtn" style="background:#0ea5e9;">Congelar Cabeçalho</button>
        <button id="pdfBtn" style="background:#ef4444;">📄 Exportar PDF</button>
        <button id="temaBtn" style="background:#0f172a;">🌙 Tema Escuro</button>
      </div>

      <div class="preview" id="preview"></div>

      <footer>✨ Criado por <strong>CLX</strong></footer>
    </div>
  </div>

<script>
/* ========== AUTH + ADMIN ========== */
(function(){
  const STORAGE_USERS = 'clx_users';     // onde os usuários ficam
  const STORAGE_LOGGED = 'clx_logged';
  const STORAGE_DB = 'clx_db_simulado';  // "banco" simulado
  const DEFAULT_ADMIN = 'CLX';
  const DEFAULT_ADMIN_PASS = '02072007';

  // elementos
  const overlay = document.getElementById('clxLoginOverlay');
  const userInput = document.getElementById('clxUserInput');
  const passInput = document.getElementById('clxPassInput');
  const loginBtn = document.getElementById('clxLoginBtn');
  const loginError = document.getElementById('clxLoginError');
  const rememberCheck = document.getElementById('clxRemember');

  const gear = document.getElementById('clxGear');
  const adminModal = document.getElementById('clxAdminModal');
  const adminClose = document.getElementById('clxCloseAdmin');
  const clxAddBtn = document.getElementById('clxAddBtn');
  const clxNewUser = document.getElementById('clxNewUser');
  const clxNewPass = document.getElementById('clxNewPass');
  const clxNewAdmin = document.getElementById('clxNewAdmin');
  const clxUserList = document.getElementById('clxUserList');
  const clxChangePass = document.getElementById('clxChangePass');
  const clxChangeMyPassBtn = document.getElementById('clxChangeMyPassBtn');
  const siteWrapper = document.getElementById('clxSiteWrapper');
  const logoutBtn = document.getElementById('clxLogoutBtn');

  const clxSaveDbBtn = document.getElementById('clxSaveDb');
  const clxLoadDbBtn = document.getElementById('clxLoadDb');
  const clxExportDbBtn = document.getElementById('clxExportDb');
  const clxImportBtn = document.getElementById('clxImportBtn');
  const clxImportFile = document.getElementById('clxImportFile');

  // helpers
  function loadUsers(){
    const raw = localStorage.getItem(STORAGE_USERS);
    if(!raw){
      const initial = {};
      initial[DEFAULT_ADMIN] = { pass: DEFAULT_ADMIN_PASS, bloqueado:false, admin:true };
      localStorage.setItem(STORAGE_USERS, JSON.stringify(initial));
      return initial;
    }
    try { return JSON.parse(raw) || {}; } catch(e) { return {}; }
  }
  function saveUsers(obj){ localStorage.setItem(STORAGE_USERS, JSON.stringify(obj)); }

  function getLogged(){ return localStorage.getItem(STORAGE_LOGGED) || null; }
  function setLogged(u){ if(u) localStorage.setItem(STORAGE_LOGGED, u); else localStorage.removeItem(STORAGE_LOGGED); }

  // "Banco" simulado: salvar e carregar (apenas localStorage)
  function saveDbSimulado(){
    const users = loadUsers();
    // salvamos um snapshot (string) no STORAGE_DB
    localStorage.setItem(STORAGE_DB, JSON.stringify({
      savedAt: new Date().toISOString(),
      users: users
    }));
    alert('Salvo no "banco" (localStorage).');
  }
  function loadDbSimulado(){
    const raw = localStorage.getItem(STORAGE_DB);
    if(!raw) return alert('Nenhum "banco" salvo encontrado.');
    try {
      const parsed = JSON.parse(raw);
      if(!parsed.users) return alert('Arquivo do banco inválido.');
      localStorage.setItem(STORAGE_USERS, JSON.stringify(parsed.users));
      users = loadUsers();
      renderUserList();
      alert('Dados carregados do "banco".');
    } catch(e){
      alert('Erro ao carregar o "banco".');
    }
  }
  function exportDbToFile(){
    const raw = localStorage.getItem(STORAGE_USERS) || '{}';
    const blob = new Blob([raw], { type: 'application/json' });
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = 'clx_users_export.json';
    a.click();
    URL.revokeObjectURL(a.href);
  }
  function importDbFromFile(file){
    const reader = new FileReader();
    reader.onload = ev => {
      try {
        const parsed = JSON.parse(ev.target.result);
        // overwrite users
        localStorage.setItem(STORAGE_USERS, JSON.stringify(parsed));
        users = loadUsers();
        renderUserList();
        alert('Importado com sucesso!');
      } catch {
        alert('Arquivo JSON inválido.');
      }
    };
    reader.readAsText(file);
  }

  // estado
  let users = loadUsers();

  // render users list in admin modal
  function renderUserList(){
    clxUserList.innerHTML = '';
    const keys = Object.keys(users).sort((a,b) => a.localeCompare(b));
    if(keys.length === 0){
      clxUserList.innerHTML = '<div style="opacity:.6">Nenhum usuário cadastrado.</div>';
      return;
    }
    keys.forEach(u => {
      const info = users[u];
      const row = document.createElement('div');
      row.className = 'clxUserRow' + (info.bloqueado ? ' blocked' : '');
      row.innerHTML = `
        <div style="flex:1;">
          <strong>${u}</strong>
          <div style="font-size:12px;opacity:0.8">${info.admin ? 'Administrador' : 'Usuário'}</div>
        </div>
        <div style="display:flex;gap:6px;align-items:center;">
          <button class="clxBtn edit" data-user="${u}">✏️</button>
          <button class="clxBtn block" data-user="${u}">${info.bloqueado ? '🔓' : '🔒'}</button>
          <button class="clxBtn delete" data-user="${u}">❌</button>
        </div>
      `;
      clxUserList.appendChild(row);
    });

    // attach events
    clxUserList.querySelectorAll('.clxBtn.edit').forEach(btn=>{
      btn.onclick = ()=>{
        const u = btn.dataset.user;
        const nova = prompt('Nova senha para ' + u + ' (deixe em branco para cancelar)');
        if(nova === null || nova.trim() === '') return;
        users[u].pass = nova;
        saveUsers(users);
        alert('Senha alterada para ' + u);
        renderUserList();
      };
    });

    clxUserList.querySelectorAll('.clxBtn.block').forEach(btn=>{
      btn.onclick = ()=>{
        const u = btn.dataset.user;
        if(u === DEFAULT_ADMIN){ alert('Não é possível bloquear o admin principal.'); return; }
        users[u].bloqueado = !users[u].bloqueado;
        saveUsers(users);
        renderUserList();
      };
    });

    clxUserList.querySelectorAll('.clxBtn.delete').forEach(btn=>{
      btn.onclick = ()=>{
        const u = btn.dataset.user;
        if(u === DEFAULT_ADMIN){ alert('Não é possível excluir o admin principal.'); return; }
        if(!confirm('Excluir usuário ' + u + '?')) return;
        delete users[u];
        saveUsers(users);
        renderUserList();
      };
    });
  }

  // add user
  function adminAddUser(){
    const u = (clxNewUser.value||'').trim();
    const p = (clxNewPass.value||'').trim();
    const makeAdmin = !!clxNewAdmin.checked;
    if(!u || !p) return alert('Preencha usuário e senha.');
    if(users[u]) return alert('Usuário já existe.');
    users[u] = { pass: p, bloqueado:false, admin: !!makeAdmin };
    saveUsers(users);
    clxNewUser.value=''; clxNewPass.value=''; clxNewAdmin.checked = false;
    renderUserList();
    alert('Usuário adicionado: ' + u);
  }

  // change my password
  function changeMyPassword(){
    const logged = getLogged();
    if(!logged) return alert('Nenhum usuário logado.');
    const nova = (clxChangePass.value||'').trim();
    if(!nova) return alert('Digite a nova senha.');
    users[logged].pass = nova;
    saveUsers(users);
    clxChangePass.value='';
    alert('Sua senha foi alterada.');
  }

  // login
  function attemptLogin(){
    const u = (userInput.value||'').trim();
    const p = (passInput.value||'').trim();
    const remember = rememberCheck.checked;
    if(!u || !p){ loginError.style.display='block'; loginError.textContent='Preencha usuário e senha.'; return; }
    const info = users[u];
    if(!info || info.pass !== p){ loginError.style.display='block'; loginError.textContent='Usuário ou senha incorretos.'; return; }
    if(info.bloqueado){ loginError.style.display='block'; loginError.textContent='Usuário bloqueado pelo administrador.'; return; }

    // sucesso
    setLogged(u);
    overlay.style.display='none';
    document.body.classList.remove('login-active');

    // mostrar site
    siteWrapper.style.display = 'block';
    document.body.style.background = '#ffffff';
    document.body.classList.add('site-visible');
    document.body.style.color = '#0b1220';
    logoutBtn.style.display = 'inline-block';

    if(remember){ localStorage.setItem('clx_remember_user', u); } else { localStorage.removeItem('clx_remember_user'); }

    // mostrar engrenagem só para admin
    if(info.admin) gear.style.display = 'block'; else gear.style.display = 'none';
  }

  // logout
  function clxLogout(){
    if(!confirm('Deseja sair?')) return;
    setLogged(null);
    // reload para resetar estado e forçar login
    location.reload();
  }

  // init
  (function init(){
    document.body.classList.add('login-active');
    users = loadUsers();

    const logged = getLogged();
    if(logged && users[logged]){
      if(users[logged].bloqueado){
        setLogged(null);
        overlay.style.display='flex';
        siteWrapper.style.display='none';
      } else {
        overlay.style.display='none';
        siteWrapper.style.display='block';
        document.body.style.background = '#ffffff';
        document.body.classList.add('site-visible');
        document.body.style.color = '#0b1220';
        logoutBtn.style.display = 'inline-block';
        if(users[logged].admin) gear.style.display = 'block';
      }
    } else {
      overlay.style.display='flex';
      siteWrapper.style.display='none';
      const remembered = localStorage.getItem('clx_remember_user');
      if(remembered){ userInput.value = remembered; rememberCheck.checked = true; }
    }

    // wire events
    loginBtn.onclick = attemptLogin;
    passInput.addEventListener('keydown', e=>{ if(e.key==='Enter') attemptLogin(); });
    userInput.addEventListener('keydown', e=>{ if(e.key==='Enter') passInput.focus(); });

    gear.onclick = ()=>{ adminModal.style.display='flex'; renderUserList(); };
    adminClose.onclick = ()=>{ adminModal.style.display='none'; };
    clxAddBtn.onclick = adminAddUser;
    clxChangeMyPassBtn.onclick = changeMyPassword;

    logoutBtn.onclick = clxLogout;

    // db buttons
    clxSaveDbBtn.onclick = saveDbSimulado;
    clxLoadDbBtn.onclick = ()=>{ if(confirm('Substituir usuários atuais pelos do banco?')) loadDbSimulado(); };
    clxExportDbBtn.onclick = exportDbToFile;
    clxImportBtn.onclick = ()=> clxImportFile.click();
    clxImportFile.addEventListener('change', e=>{
      const f = e.target.files[0];
      if(!f) return;
      if(!confirm('Importar usuários a partir deste arquivo substituirá os usuários atuais. Continuar?')) return;
      importDbFromFile(f);
    });

    // double-click gear to logout convenience (only if admin shown)
    gear.ondblclick = ()=>{ if(confirm('Deseja sair (logout)?')) clxLogout(); };

    // debug
    window.CLX_auth = {
      usersRef: users,
      reloadUsers: function(){ users = loadUsers(); renderUserList(); },
      logout: clxLogout
    };
  })();
})();

/* ========== SEU JS ORIGINAL (Gerador de planilhas) ========== */

/* ---------- Setup meses ---------- */
const nomesMeses = ["Janeiro","Fevereiro","Março","Abril","Maio","Junho","Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"];
const mesesDiv = document.getElementById("meses");
nomesMeses.forEach((m,i) => {
  const lbl = document.createElement("label");
  lbl.style.cursor = "pointer";
  lbl.innerHTML = `<input type="checkbox" value="${i}"> ${m}`;
  mesesDiv.appendChild(lbl);
});

function mesesSelecionados(){
  const checks = mesesDiv.querySelectorAll("input:checked");
  return Array.from(checks).map(c => parseInt(c.value));
}

function gerarDias(ano, mes){
  const dias = [];
  const ultimo = new Date(ano, mes+1, 0).getDate();
  const nomesDias = ["Domingo","Segunda","Terça","Quarta","Quinta","Sexta","Sábado"];
  for(let d=1; d<=ultimo; d++){
    const date = new Date(ano, mes, d);
    dias.push({
      data: `${d.toString().padStart(2,"0")}/${(mes+1).toString().padStart(2,"0")}/${ano}`,
      diaSemana: nomesDias[date.getDay()]
    });
  }
  return dias;
}

function pegarColunas(){
  const inputs = document.querySelectorAll("#colunasContainer input");
  return Array.from(inputs).map(i => i.value.trim()).filter(v => v !== "");
}

document.getElementById("addColunaBtn").addEventListener("click", () => {
  const container = document.getElementById("colunasContainer");
  const input = document.createElement("input");
  input.type = "text";
  input.placeholder = "Nova coluna";
  container.appendChild(input);
  input.scrollIntoView({behavior:"smooth", block:"nearest"});
});

/* ---------- Modelos (localStorage) ---------- */
const modeloSelect = document.getElementById("modeloSelect");
function atualizarListaModelos(){
  modeloSelect.innerHTML = `<option value="">-- Escolher modelo salvo --</option>`;
  const modelos = JSON.parse(localStorage.getItem("modelosColunas") || "{}");
  for(const nome in modelos){
    const opt = document.createElement("option");
    opt.value = nome; opt.textContent = nome;
    modeloSelect.appendChild(opt);
  }
}
atualizarListaModelos();

function obterModelos(){ return JSON.parse(localStorage.getItem("modelosColunas") || "{}"); }
function salvarModelos(m){ localStorage.setItem("modelosColunas", JSON.stringify(m)); atualizarListaModelos(); }

document.getElementById("salvarModeloBtn").addEventListener("click", () => {
  const nome = prompt("Digite o nome do modelo:");
  if(!nome) return;
  const colunas = pegarColunas();
  if(colunas.length === 0) return alert("Adicione ao menos uma coluna.");
  const modelos = obterModelos(); modelos[nome] = colunas; salvarModelos(modelos);
  alert(`Modelo \"${nome}\" salvo!`);
});

document.getElementById("carregarModeloBtn").addEventListener("click", () => {
  const nome = modeloSelect.value; if(!nome) return alert("Escolha um modelo.");
  const modelos = obterModelos(); const colunas = modelos[nome];
  const container = document.getElementById("colunasContainer"); container.innerHTML = "";
  colunas.forEach(c => { const i = document.createElement("input"); i.type="text"; i.value=c; container.appendChild(i); });
  alert(`Modelo \"${nome}\" carregado!`);
});

document.getElementById("duplicarModeloBtn").addEventListener("click", () => {
  const nome = modeloSelect.value; if(!nome) return alert("Escolha um modelo.");
  const novo = prompt("Digite o novo nome:"); if(!novo) return;
  const modelos = obterModelos(); modelos[novo] = [...modelos[nome]]; salvarModelos(modelos);
  alert("Modelo duplicado!");
});

document.getElementById("excluirModeloBtn").addEventListener("click", () => {
  const nome = modeloSelect.value; if(!nome) return alert("Escolha um modelo.");
  if(!confirm(`Excluir modelo \"${nome}\"?`)) return;
  const modelos = obterModelos(); delete modelos[nome]; salvarModelos(modelos);
  alert("Modelo excluído!");
});

document.getElementById("exportarBtn").addEventListener("click", () => {
  const modelos = obterModelos();
  const blob = new Blob([JSON.stringify(modelos, null, 2)], { type: "application/json" });
  const a = document.createElement("a"); a.href = URL.createObjectURL(blob); a.download = "modelos_CLX.json"; a.click();
  URL.revokeObjectURL(a.href);
});

const inputFile = document.getElementById("importFile");
document.getElementById("importarBtn").addEventListener("click", () => inputFile.click());
inputFile.addEventListener("change", e => {
  const f = e.target.files[0]; if(!f) return;
  const reader = new FileReader();
  reader.onload = ev => {
    try {
      const novos = JSON.parse(ev.target.result);
      const modelos = obterModelos();
      Object.assign(modelos, novos);
      salvarModelos(modelos);
      alert("Importado com sucesso!");
    } catch {
      alert("Erro ao importar o arquivo JSON!");
    }
  };
  reader.readAsText(f);
});

/* Modelos prontos */
document.getElementById("modeloSimplesBtn").addEventListener("click", () => {
  const c = document.getElementById("colunasContainer"); c.innerHTML = "";
  ["Data","Dia da Semana","Descrição"].forEach(x => { const i=document.createElement("input"); i.type="text"; i.value=x; c.appendChild(i); });
  alert("Modelo Simples carregado!");
});
document.getElementById("modeloProBtn").addEventListener("click", () => {
  const c = document.getElementById("colunasContainer"); c.innerHTML = "";
  ["Data","Dia da Semana","Atividade","Responsável","Setor","Status","Observações"].forEach(x => { const i=document.createElement("input"); i.type="text"; i.value=x; c.appendChild(i); });
  alert("Modelo Profissional carregado!");
});

/* ---------- XLSX helpers (xslx-js-style) ---------- */
function calcularLarguras(dados) {
  const maxCol = dados[0] ? dados[0].length : 0;
  const larg = new Array(maxCol).fill(0);
  dados.forEach(row => {
    for (let c = 0; c < maxCol; c++) {
      const val = row[c] == null ? "" : String(row[c]);
      const tamanho = val.length;
      if (tamanho > larg[c]) larg[c] = tamanho;
    }
  });
  return larg.map(x => ({ wch: Math.max(10, Math.min(40, x + 3)) }));
}

function aplicarEstilosEbordas(ws, dados) {
  if (!ws['!ref']) return;
  const range = XLSX.utils.decode_range(ws['!ref']);
  const headerRow = range.s.r;
  for (let R = range.s.r; R <= range.e.r; ++R) {
    for (let C = range.s.c; C <= range.e.c; ++C) {
      const addr = XLSX.utils.encode_cell({ r: R, c: C });
      const cell = ws[addr];
      if (!cell) continue;
      if (R === headerRow) {
        cell.s = Object.assign({}, cell.s || {}, {
          font: { bold: true },
          fill: { patternType: "solid", fgColor: { rgb: "FFD9D9D9" } },
          alignment: { vertical: "center", horizontal: "center" },
          border: { bottom: { style: "thin", color: { rgb: "FF000000" } } }
        });
      } else {
        cell.s = Object.assign({}, cell.s || {}, {
          border: {
            top: { style: "thin", color: { rgb: "FF000000" } },
            bottom: { style: "thin", color: { rgb: "FF000000" } },
            left: { style: "thin", color: { rgb: "FF000000" } },
            right: { style: "thin", color: { rgb: "FF000000" } }
          },
          alignment: { vertical: "center", horizontal: "left" }
        });
      }
    }
  }
}

function congelarPrimeiraLinha(ws) {
  ws['!freeze'] = { xSplit: 0, ySplit: 1, topLeftCell: "A2" };
  ws['!pane'] = { xSplit: 0, ySplit: 1, topLeftCell: "A2", activePane: "bottomLeft", state: "frozen" };
}

/* Flags */
let aplicarBordasFlag = false;
let congelarCabecalhoFlag = false;
let darkMode = false;

document.getElementById("bordasBtn").addEventListener("click", () => {
  aplicarBordasFlag = !aplicarBordasFlag;
  document.getElementById("bordasBtn").textContent = aplicarBordasFlag ? "Bordas Excel ✅" : "Bordas Excel";
  alert(aplicarBordasFlag ? "Bordas automáticas ativadas para o próximo XLSX." : "Bordas desativadas.");
});

document.getElementById("congelarBtn").addEventListener("click", () => {
  congelarCabecalhoFlag = !congelarCabecalhoFlag;
  document.getElementById("congelarBtn").textContent = congelarCabecalhoFlag ? "Congelar Cabeçalho ✅" : "Congelar Cabeçalho";
  alert(congelarCabecalhoFlag ? "Primeira linha será congelada no XLSX." : "Congelamento desativado.");
});

document.getElementById("temaBtn").addEventListener("click", () => {
  darkMode = !darkMode;
  document.body.classList.toggle("dark", darkMode);
  document.getElementById("temaBtn").textContent = darkMode ? "☀️ Tema Claro" : "🌙 Tema Escuro";
});

function gerarPlanilha(ano, meses, qtdAbas){
  const colunas = pegarColunas();
  if(colunas.length === 0) return alert("Adicione ao menos uma coluna antes de gerar.");

  const wb = XLSX.utils.book_new();
  const abas = Math.min(qtdAbas, meses.length);
  for(let i=0; i<abas; i++){
    const mes = meses[i];
    const dias = gerarDias(ano, mes);
    const dados = [colunas];
    dias.forEach(d => {
      const linha = [];
      colunas.forEach(col => {
        if(col.toLowerCase().includes("data")) linha.push(d.data);
        else if(col.toLowerCase().includes("semana") || col.toLowerCase().includes("dia")) linha.push(d.diaSemana);
        else linha.push("");
      });
      dados.push(linha);
    });

    const ws = XLSX.utils.aoa_to_sheet(dados);
    ws['!cols'] = calcularLarguras(dados);

    if (aplicarBordasFlag || congelarCabecalhoFlag) {
      aplicarEstilosEbordas(ws, dados);
    } else {
      if (ws['!ref']) {
        const range = XLSX.utils.decode_range(ws['!ref']);
        const headerRow = range.s.r;
        for (let C = range.s.c; C <= range.e.c; ++C) {
          const addr = XLSX.utils.encode_cell({ r: headerRow, c: C });
          if (!ws[addr]) continue;
          ws[addr].s = Object.assign({}, ws[addr].s || {}, {
            font: { bold: true },
            fill: { patternType: "solid", fgColor: { rgb: "FFD9D9D9" } },
            alignment: { vertical: "center", horizontal: "center" },
            border: { bottom: { style: "thin", color: { rgb: "FF000000" } } }
          });
        }
      }
    }

    if (congelarCabecalhoFlag) congelarPrimeiraLinha(ws);

    XLSX.utils.book_append_sheet(wb, ws, nomesMeses[mes].substring(0,31));
  }

  XLSX.writeFile(wb, `planejamento_${ano}.xlsx`);
  alert("Arquivo XLSX gerado!");
}

function mostrarExemplo(ano, meses){
  const preview = document.getElementById("preview");
  preview.innerHTML = "";
  if(!meses || meses.length === 0) {
    preview.innerHTML = "<p style='color:crimson'>Selecione pelo menos um mês!</p>";
    return;
  }

  const colunas = pegarColunas();
  const mes = meses[0];
  const dias = gerarDias(ano, mes);

  let html = `<h3 style="margin:0 0 8px 0">${nomesMeses[mes]} ${ano}</h3><div style="overflow:auto"><table id="previewTable"><thead><tr>`;
  colunas.forEach(c => html += `<th>${c}</th>`);
  html += `</tr></thead><tbody>`;

  dias.slice(0, 12).forEach(d => {
    html += "<tr>";
    colunas.forEach(col => {
      if(col.toLowerCase().includes("data")) html += `<td>${d.data}</td>`;
      else if(col.toLowerCase().includes("semana") || col.toLowerCase().includes("dia")) html += `<td>${d.diaSemana}</td>`;
      else html += `<td></td>`;
    });
    html += "</tr>";
  });

  html += `</tbody></table></div><p style="margin-top:8px;font-style:italic">Mostrando os primeiros 12 dias...</p>`;
  preview.innerHTML = html;
}

document.getElementById("gradearBtn").addEventListener("click", () => {
  const tabela = document.querySelector("#previewTable");
  if(!tabela) return alert("Clique em 'Ver Exemplo' antes de aplicar a grade.");
  tabela.style.border = "2px solid #000";
  tabela.querySelectorAll("th").forEach(th => { th.style.border = "1px solid #000"; th.style.padding = "6px"; th.style.background = "#eee"; });
  tabela.querySelectorAll("td").forEach(td => { td.style.border = "1px solid #000"; td.style.padding = "6px"; });
  alert("Grade aplicada no preview!");
});

document.getElementById("pdfBtn").addEventListener("click", () => {
  const preview = document.getElementById("preview").innerHTML;
  if(!preview.trim()) return alert("Clique em 'Ver Exemplo' antes de exportar para PDF.");

  const win = window.open("", "_blank");
  win.document.write(`
    <html>
      <head>
        <title>Exportar PDF - CLX</title>
        <style>
          body{font-family:Arial,Helvetica,sans-serif;padding:18px}
          table{width:100%;border-collapse:collapse}
          th,td{border:1px solid #000;padding:8px;font-size:13px}
          th{background:#f3f4f6}
        </style>
      </head>
      <body>${preview}</body>
    </html>
  `);
  win.document.close();
  win.print();
});

document.getElementById("gerarBtn").addEventListener("click", () => {
  const ano = parseInt(document.getElementById("ano").value);
  const meses = mesesSelecionados();
  const qtdAbas = parseInt(document.getElementById("qtdAbas").value);
  if(!meses.length) return alert("Selecione pelo menos um mês!");
  gerarPlanilha(ano, meses, qtdAbas);
});

document.getElementById("previewBtn").addEventListener("click", () => {
  const ano = parseInt(document.getElementById("ano").value);
  const meses = mesesSelecionados();
  mostrarExemplo(ano, meses);
});

/* Inicializa labels */
document.getElementById("bordasBtn").textContent = "Bordas Excel";
document.getElementById("congelarBtn").textContent = "Congelar Cabeçalho";
document.getElementById("temaBtn").textContent = "🌙 Tema Escuro";
</script>
</body>
</html>
