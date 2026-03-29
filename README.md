<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#8b0000">
<title>AS DONAS DA BRASA</title>

<style>
body {
  margin:0;
  font-family:Arial;
  background:linear-gradient(135deg,#1f1f1f,#3b0a0a);
  color:#fff;
}

header {
  background:#8b0000;
  padding:15px;
  text-align:center;
  font-size:22px;
  font-weight:bold;
}

.tabs {
  display:flex;
  overflow-x:auto;
  background:#111;
}

.tab {
  padding:10px 15px;
  cursor:pointer;
  white-space:nowrap;
}

.tab.active {
  border-bottom:3px solid #ff4d4d;
  color:#ff4d4d;
}

.tab.ocupada {
  color:#ff4d4d;
}

.nomeCliente {
  width:90%;
  margin:10px auto;
  display:block;
  padding:12px;
  border-radius:10px;
  border:none;
  font-size:16px;
}

.container { padding:10px; }

.card {
  background:#2a2a2a;
  padding:10px;
  margin-bottom:10px;
  border-radius:12px;
  box-shadow:0 0 10px rgba(0,0,0,0.5);
}

input, textarea {
  width:45%;
  padding:10px;
  margin:5px 2%;
  border:none;
  border-radius:8px;
  font-size:15px;
}

textarea {
  width:95%;
  height:50px;
}

.total { text-align:right; color:#00ff99; font-weight:bold; }

.totalMesa {
  text-align:center;
  font-size:20px;
  color:#00ff99;
  margin-top:10px;
}

button {
  width:100%;
  padding:14px;
  margin-top:8px;
  border:none;
  border-radius:10px;
  background:#ff4d4d;
  color:#fff;
  font-size:16px;
}

.btn-sec { background:#444; }

.totalDia {
  background:#000;
  padding:10px;
  margin-top:10px;
  border-radius:10px;
  text-align:center;
  color:#00ff99;
}
</style>
</head>

<body>

<header>🔥 AS DONAS DA BRASA 🔥</header>

<div class="tabs" id="tabs"></div>

<input class="nomeCliente" id="nomeCliente" placeholder="Nome do cliente">

<div class="container" id="conteudo"></div>

<script>
const lista = [
"Espetinhos","Petiscos","Caldos","Água",
"Refrigerante lata","Refrigerante litro",
"Cerveja lata","Cachaça dose"
];

let mesas = 15;

let dados = JSON.parse(localStorage.getItem('dados')) || {};
let totalDia = parseFloat(localStorage.getItem('totalDia')) || 0;

if (dados && Array.isArray(dados[1])) dados = {};

for (let i=1;i<=mesas;i++){
  if(!dados[i]){
    dados[i]={nome:"",itens:lista.map(()=>({q:0,p:0,obs:""}))};
  }
}

let atual = 1;

const tabs = document.getElementById("tabs");
const conteudo = document.getElementById("conteudo");
const nomeInput = document.getElementById("nomeCliente");

for (let i=1;i<=mesas;i++){
  let t=document.createElement("div");
  t.className="tab"+(i===1?" active":"");
  t.onclick=()=>trocarMesa(i);
  tabs.appendChild(t);
}

function salvar(){
  localStorage.setItem('dados',JSON.stringify(dados));
  localStorage.setItem('totalDia',totalDia);
}

function trocarMesa(n){
  atual=n;

  document.querySelectorAll('.tab').forEach((el,i)=>{
    el.classList.toggle('active',i===n-1);
  });

  nomeInput.value=dados[atual].nome||"";
  atualizarAbas();
  render();
}

nomeInput.oninput=function(){
  dados[atual].nome=this.value;
  salvar();
  atualizarAbas();
};

function atualizarAbas(){
  document.querySelectorAll('.tab').forEach((el,i)=>{
    let mesa=i+1;
    let nome=dados[mesa].nome;
    let total=dados[mesa].itens.reduce((s,x)=>s+(x.q*x.p),0);

    el.innerText = nome ? `Mesa ${mesa} - ${nome}` : `Mesa ${mesa}`;
    el.classList.toggle("ocupada", total>0);
  });
}

function render(){
  conteudo.innerHTML="";
  let total=0;

  lista.forEach((p,i)=>{
    let item=dados[atual].itens[i];
    let t=item.q*item.p;
    total+=t;

    conteudo.innerHTML+=`
    <div class="card">
      <b>${p}</b><br>

      <input type="number" value="${item.q}"
        oninput="update(${i},this.value,'q')">

      <input type="number" value="${item.p}"
        oninput="update(${i},this.value,'p')">

      <textarea oninput="update(${i},this.value,'obs')">${item.obs}</textarea>

      <div class="total">Total: R$ ${t.toFixed(2)}</div>
    </div>`;
  });

  conteudo.innerHTML+=`
  <div class="totalMesa">
    ${(dados[atual].nome||"Mesa "+atual)}: R$ ${total.toFixed(2)}
  </div>

  <button onclick="limparMesa()">🧹 Limpar Mesa</button>
  <button onclick="fecharMesa()">💳 Fechar Conta</button>
  <button onclick="window.print()">🖨️ Imprimir</button>
  <button class="btn-sec" onclick="relatorio()">📊 Relatório</button>
  <button class="btn-sec" onclick="resetDia()">♻️ Zerar Dia</button>

  <div class="totalDia">💰 Total do Dia: R$ ${totalDia.toFixed(2)}</div>
  `;
}

function update(i,val,tipo){
  if(tipo==="obs"){
    dados[atual].itens[i][tipo]=val;
  } else {
    dados[atual].itens[i][tipo]=parseFloat(val)||0;
  }

  salvar();
  atualizarAbas();
  render();
}

function limparMesa(){
  if(confirm("Limpar mesa?")){
    dados[atual]={nome:"",itens:lista.map(()=>({q:0,p:0,obs:""}))};
    nomeInput.value="";
    salvar();
    atualizarAbas();
    render();
  }
}

function fecharMesa(){
  let total=dados[atual].itens.reduce((s,x)=>s+(x.q*x.p),0);
  totalDia+=total;

  alert("Conta fechada!");
  limparMesa();
}

function relatorio(){
  let texto="RELATÓRIO DO DIA\n\n";

  for(let i=1;i<=mesas;i++){
    let total=dados[i].itens.reduce((s,x)=>s+(x.q*x.p),0);
    if(total>0){
      texto+=`${dados[i].nome||"Mesa "+i}: R$ ${total.toFixed(2)}\n`;
    }
  }

  texto+=`\nTOTAL: R$ ${totalDia.toFixed(2)}`;
  alert(texto);
}

function resetDia(){
  if(confirm("Zerar tudo?")){
    totalDia=0;
    salvar();
    render();
  }
}

trocarMesa(1);
atualizarAbas();
</script>

</body>
</html>