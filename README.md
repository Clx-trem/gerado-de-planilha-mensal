<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Gerador de Planilha Personalizada - CLX</title>

  <style>
    /* TEMA A - Elegante (cinza + azul neon) */
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
    body {
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
    th { background:#f1f5f9; }
    #colunasContainer input { display:block; width:100%; margin-top:6px; }
    .modelos { display:flex; gap:10px; align-items:center; flex-wrap:wrap; margin-top:10px; }
    footer { text-align:center; margin-top:28px; font-size:13px; color:#555; }

    /* Tema escuro (opção A aplicada por padrão quando ativado) */
    body.dark {
      background: var(--bg);
      color: var(--white);
    }
    body.dark .preview { background: var(--card); border-color: var(--border); box-shadow: 0 6px 18px var(--shadow); }
    body.dark input, body.dark select, body.dark textarea {
      background: var(--input-bg); color: var(--white); border:1px solid var(--border);
    }
    body.dark th { background: rgba(255,255,255,0.04); }
    body.dark table { color: var(--white); }
    body.dark button { background: var(--accent); color: #fff; border:none; }

    /* pequenos ajustes para responsividade */
    @media (max-width:700px){
      .months { grid-template-columns: repeat(2, 1fr); }
      body { padding:14px; margin:12px; }
    }
  </style>

  <!-- XLSX com suporte a estilos (full) -->
  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
</head>
<body>
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

  <script>
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
      alert(`Modelo "${nome}" salvo!`);
    });

    document.getElementById("carregarModeloBtn").addEventListener("click", () => {
      const nome = modeloSelect.value; if(!nome) return alert("Escolha um modelo.");
      const modelos = obterModelos(); const colunas = modelos[nome];
      const container = document.getElementById("colunasContainer"); container.innerHTML = "";
      colunas.forEach(c => { const i = document.createElement("input"); i.type="text"; i.value=c; container.appendChild(i); });
      alert(`Modelo "${nome}" carregado!`);
    });

    document.getElementById("duplicarModeloBtn").addEventListener("click", () => {
      const nome = modeloSelect.value; if(!nome) return alert("Escolha um modelo.");
      const novo = prompt("Digite o novo nome:"); if(!novo) return;
      const modelos = obterModelos(); modelos[novo] = [...modelos[nome]]; salvarModelos(modelos);
      alert("Modelo duplicado!");
    });

    document.getElementById("excluirModeloBtn").addEventListener("click", () => {
      const nome = modeloSelect.value; if(!nome) return alert("Escolha um modelo.");
      if(!confirm(`Excluir modelo "${nome}"?`)) return;
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

    /* ---------- Estilos e utilitários XLSX ---------- */
    function autoAjustarColunas(ws, colunas){
      ws['!cols'] = colunas.map(c => ({ wch: Math.max(12, c.length + 2) }));
    }

    function aplicarBordasNaPlanilha(ws){
      if(!ws['!ref']) return;
      const range = XLSX.utils.decode_range(ws['!ref']);
      for(let R = range.s.r; R <= range.e.r; R++){
        for(let C = range.s.c; C <= range.e.c; C++){
          const addr = XLSX.utils.encode_cell({ r: R, c: C });
          if(!ws[addr]) continue;
          // preservar estilo existente se houver
          ws[addr].s = Object.assign({}, ws[addr].s || {}, {
            border: {
              top: { style:"thin", color:{ rgb:"000000" } },
              bottom: { style:"thin", color:{ rgb:"000000" } },
              left: { style:"thin", color:{ rgb:"000000" } },
              right: { style:"thin", color:{ rgb:"000000" } }
            }
          });
        }
      }
    }

    function congelarPrimeiraLinha(ws){
      // nota: XLSX pode aceitar ws['!freeze'] ou ws['!pane'] dependendo da versão;
      // vamos setar ambos para maior compatibilidade:
      ws['!freeze'] = { xSplit: 0, ySplit: 1 };
      ws['!pane'] = { xSplit: 0, ySplit: 1, activePane: "bottomLeft", frozen: true };
    }

    /* Flags controláveis por botões */
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

    /* Tema */
    document.getElementById("temaBtn").addEventListener("click", () => {
      darkMode = !darkMode;
      document.body.classList.toggle("dark", darkMode);
      document.getElementById("temaBtn").textContent = darkMode ? "☀️ Tema Claro" : "🌙 Tema Escuro";
    });

    /* ---------- Gerar XLSX ---------- */
    function gerarPlanilha(ano, meses, qtdAbas){
      const colunas = pegarColunas();
      if(colunas.length === 0) return alert("Adicione ao menos uma coluna antes de gerar.");

      const wb = XLSX.utils.book_new();

      // limitar qtdAbas ao número de meses selecionados
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

        // aplicar ajustes e estilos antes de adicionar a workbook
        autoAjustarColunas(ws, colunas);
        if(aplicarBordasFlag) aplicarBordasNaPlanilha(ws);
        if(congelarCabecalhoFlag) congelarPrimeiraLinha(ws);

        XLSX.utils.book_append_sheet(wb, ws, nomesMeses[mes].substring(0,31)); // nomes limitados a 31 chars
      }

      // salvar arquivo
      XLSX.writeFile(wb, `planejamento_${ano}.xlsx`);
      alert("Arquivo XLSX gerado!");
    }

    /* ---------- Preview + Grade (visual) + PDF ---------- */
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
      tabela.querySelectorAll("th, td").forEach(c => { c.style.border = "1px solid #000"; c.style.padding = "6px"; });
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
      // Abre diálogo de impressão — o usuário escolhe "Salvar como PDF"
      win.print();
    });

    /* ---------- Botões principais ---------- */
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

    /* Inicializa estado visual do botão (opcional) */
    document.getElementById("bordasBtn").textContent = "Bordas Excel";
    document.getElementById("congelarBtn").textContent = "Congelar Cabeçalho";
    document.getElementById("temaBtn").textContent = "🌙 Tema Escuro";
  </script>
</body>
</html>
