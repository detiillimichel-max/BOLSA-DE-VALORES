
const CORES = [
  {id:'vermelha', label:'Ação Vermelha', class:'vermelha'},
  {id:'azul', label:'Ação Azul', class:'azul'},
  {id:'amarela', label:'Ação Amarela', class:'amarela'},
  {id:'verde', label:'Ação Verde', class:'verde'},
];
const COMMODITIES = [
  {id:'cafe', label:'CAFÉ', class:'commodity'},
  {id:'petroleo', label:'PETRÓLEO', class:'commodity'},
  {id:'energia', label:'ENERGIA', class:'commodity'},
  {id:'ouro', label:'OURO', class:'commodity'},
];
const ESCADA = [10,20,30,50,75,100,150,200,300,500];
let casas = [];
let players = [];
let cotacao = {};
let bolsaDinheiro = 645;
let turno = 0;
let dice = 0;
let gameOver = false;

function buildCasas(){
  casas = [];
  casas.push({tipo:'start', nome:'PARTIDA'});
  const ordem = [
    'vermelha','azul','amarela','verde','cafe','petroleo',
    'vermelha','azul','energia','ouro','vermelha','amarela',
    'verde','cafe','azul','vermelha','petroleo','amarela',
    'verde','energia','ouro','azul','vermelha','cafe'
  ];
  ordem.forEach(t=>{
    if(['vermelha','azul','amarela','verde'].includes(t)){
      const c = CORES.find(x=>x.id===t);
      casas.push({tipo:t, nome:c.label, cor:c});
    } else {
      const c = COMMODITIES.find(x=>x.id===t);
      casas.push({tipo:t, nome:c.label, cor:c, commodity:true});
    }
  });
}

function initCotacao(){
  cotacao = {};
  [...CORES, ...COMMODITIES].forEach(c=>cotacao[c.id]=3); // index 3 = 50
}

function initPlayers(n){
  const coresP = ['#e53935','#1e88e5','#fdd835','#43a047'];
  players = Array.from({length:n},(_,i)=>({
    id:i, nome:`Jogador ${i+1}`, dinheiro:500, pos:0, acoes:{}, eliminado:false, cor:coresP[i]
  }));
}

function renderBoard(){
  const board = document.getElementById('board');
  board.innerHTML='';
  casas.forEach((casa, idx)=>{
    const div=document.createElement('div');
    div.className=`cell ${casa.tipo==='start'?'start':casa.cor?.class||''}`;
    div.dataset.idx=idx;
    if(casa.tipo==='start') div.innerHTML='<span>PARTIDA</span>';
    else div.innerHTML=`<span>${casa.nome}</span><small>${casa.commodity?'COMMODITY':'AÇÃO'}</small>`;
    // pawns
    players.filter(p=>p.pos===idx && !p.eliminado).forEach((p,pi)=>{
      const pawn=document.createElement('div');
      pawn.className='pawn';
      pawn.style.background=p.cor;
      pawn.style.left=(6+pi*20)+'px';
      pawn.style.bottom='6px';
      div.appendChild(pawn);
    });
    if(players[turno]?.pos===idx) div.classList.add('active');
    board.appendChild(div);
  });
}

function renderCotacao(){
  const el=document.getElementById('cotacao');
  el.innerHTML='';
  [...CORES, ...COMMODITIES].forEach(c=>{
    const idx=cotacao[c.id];
    const valor=ESCADA[idx];
    const row=document.createElement('div');
    row.className='quote';
    row.innerHTML=`<span><span class="tag" style="background:${c.class==='commodity'?'#6d4c41':c.id==='vermelha'?'#e53935':c.id==='azul'?'#1e88e5':c.id==='amarela'?'#f9a825':'#43a047'}">${c.label}</span></span><strong>R$ ${valor}</strong>`;
    el.appendChild(row);
  });
}

function renderPlayers(){
  const el=document.getElementById('players');
  el.innerHTML='';
  const grid=document.createElement('div');
  grid.className='players-grid';
  players.forEach((p,i)=>{
    const totalAcoes=Object.values(p.acoes).reduce((s,q)=>s+q,0);
    const div=document.createElement('div');
    div.className=`player-card ${i===turno && !gameOver?'turn':''} ${p.eliminado?'eliminated':''}`;
    const acoesStr=Object.entries(p.acoes).map(([k,v])=>`${k}:${v}`).join(' ');
    div.innerHTML=`<div class="row"><strong style="color:${p.cor}">● ${p.nome}</strong><span>R$ ${p.dinheiro}</span></div>
    <div class="row"><small>${acoesStr||'sem ações'}</small><small>${totalAcoes} ações</small></div>`;
    grid.appendChild(div);
  });
  el.appendChild(grid);
  document.getElementById('bolsaDinheiro').textContent=`R$ ${bolsaDinheiro}`;
}

function log(msg){
  const l=document.getElementById('log');
  l.innerHTML=`<div>> ${msg}</div>`+l.innerHTML;
}

function getValor(id){return ESCADA[cotacao[id]];}

function updateCotacao(id, subiu){
  let idx=cotacao[id];
  if(subiu){ if(idx<ESCADA.length-1) cotacao[id]++; }
  else { if(idx>0) cotacao[id]--; }
}

function showModal(title,text,actions){
  document.getElementById('modalTitle').textContent=title;
  document.getElementById('modalText').textContent=text;
  const act=document.getElementById('modalActions');
  act.innerHTML='';
  actions.forEach(a=>{
    const b=document.createElement('button');
    b.textContent=a.label;
    b.className=a.primary?'btn-primary':'btn-secondary';
    b.onclick=()=>{document.getElementById('modal').classList.add('hidden'); a.onClick();};
    act.appendChild(b);
  });
  document.getElementById('modal').classList.remove('hidden');
}

function proximoTurno(){
  if(gameOver) return;
  let ativos=players.filter(p=>!p.eliminado);
  if(ativos.length<=1){
    gameOver=true;
    const vencedor=ativos[0];
    log(`FIM DE JOGO! Vencedor: ${vencedor?.nome}`);
    showModal('Fim de jogo', vencedor? `${vencedor.nome} venceu com R$ ${vencedor.dinheiro}!` : 'Todos faliram!', [{label:'Novo jogo',primary:true,onClick:()=>startGame()}]);
    return;
  }
  do{ turno=(turno+1)%players.length; } while(players[turno].eliminado);
  document.getElementById('turnInfo').textContent=`Vez de ${players[turno].nome}`;
  document.getElementById('rollBtn').disabled=false;
  renderBoard(); renderPlayers(); renderCotacao();
}

function startGame(){
  const n=parseInt(document.getElementById('numPlayers').value);
  buildCasas(); initCotacao(); initPlayers(n);
  bolsaDinheiro=645; turno=0; gameOver=false;
  document.getElementById('log').innerHTML='';
  document.getElementById('diceResult').textContent='-';
  document.getElementById('turnInfo').textContent=`Vez de ${players[0].nome}`;
  document.getElementById('rollBtn').disabled=false;
  log('Novo jogo iniciado - Bolsa em R$ 50 para todas as ações');
  renderBoard(); renderCotacao(); renderPlayers();
}

function comprarAcao(player, tipo, obrigatoria){
  const valor=getValor(tipo);
  if(player.dinheiro < valor){
    if(obrigatoria){
      // IMPORTANTE regra: vender ações para a Bolsa por max R$50
      const totalAcoes=Object.keys(player.acoes).length>0;
      if(!totalAcoes){
        player.eliminado=true;
        log(`${player.nome} ELIMINADO por falta de dinheiro para compra obrigatória`);
        showModal('Eliminado', `${player.nome} não tem dinheiro para comprar ${tipo} e não tem ações para vender.`, [{label:'Continuar',primary:true,onClick:()=>proximoTurno()}]);
        return;
      }
      showModal('Sem dinheiro!', `${player.nome} precisa vender ações. A Bolsa paga no máximo R$50 por ação.`, [
        {label:'Vender tudo por R$50 cada',primary:true,onClick:()=>{
          let ganho=0;
          Object.entries(player.acoes).forEach(([k,q])=>{ ganho+=q*50; bolsaDinheiro-=q*50; });
          player.dinheiro+=ganho; player.acoes={};
          log(`${player.nome} vendeu tudo para Bolsa por R$${ganho}`);
          if(player.dinheiro>=valor){ comprarAcao(player,tipo,obrigatoria); } else { proximoTurno(); }
          renderPlayers();
        }},
        {label:'Falir',onClick:()=>{player.eliminado=true; proximoTurno();}}
      ]);
      return;
    } else {
      log(`${player.nome} não comprou ${tipo} - sem dinheiro, mas pode passar`);
      proximoTurno(); return;
    }
  }
  player.dinheiro-=valor;
  bolsaDinheiro+=valor;
  player.acoes[tipo]=(player.acoes[tipo]||0)+1;
  log(`${player.nome} comprou 1x ${tipo.toUpperCase()} por R$${valor}`);
  renderPlayers();
  proximoTurno();
}

function cairNaCasa(){
  const player=players[turno];
  const casa=casas[player.pos];
  if(casa.tipo==='start'){ log(`${player.nome} na PARTIDA`); proximoTurno(); return; }

  // COMMODITY
  if(casa.commodity){
    const dono = players.find(p=>!p.eliminado && p.id!==player.id && (p.acoes[casa.tipo]||0)>0);
    const qtdDono = players.filter(p=>!p.eliminado && (p.acoes[casa.tipo]||0)>0);
    if(qtdDono.length>0){
      // pagar R$100 para cada proprietário
      const totalPagar = qtdDono.length*100;
      if(player.dinheiro < totalPagar){
        player.eliminado=true;
        log(`${player.nome} não tem R$${totalPagar} para pagar commodity e foi eliminado`);
        showModal('Eliminado', `${player.nome} parou em ${casa.nome} já comprada e não tem R$${totalPagar} para pagar aos donos.`, [{label:'Ok',primary:true,onClick:()=>proximoTurno()}]);
        return;
      }
      player.dinheiro-=totalPagar;
      qtdDono.forEach(d=>d.dinheiro+=100);
      log(`${player.nome} pagou R$${totalPagar} (R$100 para cada dono de ${casa.nome})`);
      renderPlayers(); proximoTurno(); return;
    }
    // comprar opcional
    showModal(`Commodity: ${casa.nome}`, `Deseja comprar ${casa.nome} por R$${getValor(casa.tipo)}? Compra não é obrigatória. Cotação atual: R$${getValor(casa.tipo)}`, [
      {label:`Comprar R$${getValor(casa.tipo)}`,primary:true,onClick:()=>comprarAcao(player,casa.tipo,false)},
      {label:'Passar a vez',onClick:()=>{log(`${player.nome} não quis comprar ${casa.nome}`); proximoTurno();}}
    ]);
    return;
  }

  // AÇÃO COLORIDA - compra obrigatória
  const tipo=casa.tipo;
  showModal(`Ação ${tipo.toUpperCase()}`, `Você parou na casa ${casa.nome}. Compra OBRIGATÓRIA por R$${getValor(tipo)}. Após comprar, a cotação vai ${dice<=3?'CAIR':'SUBIR'} (dado ${dice}).`, [
    {label:`Comprar por R$${getValor(tipo)}`,primary:true,onClick:()=>{
      // altera cotação após compra conforme regra VALOR DAS AÇÕES
      const subiu = dice>=4;
      const antes=getValor(tipo);
      updateCotacao(tipo, subiu);
      const depois=getValor(tipo);
      log(`Cotação ${tipo}: R$${antes} -> R$${depois} (${subiu?'SUBIU':'CAIU'} dado ${dice})`);
      comprarAcao(player,tipo,true);
    }}
  ]);
}

document.getElementById('rollBtn').addEventListener('click',()=>{
  const player=players[turno];
  if(player.eliminado) {proximoTurno(); return;}
  dice=Math.floor(Math.random()*6)+1;
  document.getElementById('diceResult').textContent=dice;
  document.getElementById('rollBtn').disabled=true;
  log(`${player.nome} tirou ${dice}`);
  // anda
  player.pos=(player.pos+dice)%casas.length;
  renderBoard();
  setTimeout(cairNaCasa,400);
});

document.getElementById('startBtn').addEventListener('click',startGame);
startGame();
