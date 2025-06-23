let tela = "menu";
let tempoDeEspera = 15000;
let botaoInformacoes, botaoIniciar, botaoVoltar;

let tratorX = -100, tratorY = 460, tratorAndando = false;
let frameHistoria = 0;
let tempoProximoFrame = 0;

let faseNova = false;
let jogador;
let sementesColetadas = 0;
let produtosEntregues = 0;
let obstaculos = [];
let animProdutos = [];
let faseAtual = 1;
let colisao = false;
let tempoColisao = 0;
let fogos = [];

function setup() {
  createCanvas(800, 600);
  criarBotoes();
}

function draw() {
  if (tela === "menu") {
    background('#E592AE');
    mostraMenu();
  } else if (tela === "instrucoes") {
    mostraInstrucoes();
  } else if (tela === "historia") {
    mostraHistoriaSequencial();
  } else if (tela === "jogo") {
    novoJogoInterativo();
  } else if (tela === "final") {
    mostraFinal();
  }
}

function mostraMenu() {
  fill(255);
  textAlign(CENTER, CENTER);
  textSize(36);
  textFont('georgia');
  text("Festejando a Conexão Campo e Cidade", width / 2, height / 2 - 100);
  mostrarBotoes();
}

function mostraInstrucoes() {
  background(255);
  fill('#E592AE');
  textAlign(LEFT, TOP);
  textSize(18);
  textFont('verdana');
  text("Como Jogar:\n\n" +
       "Você vai controlar um trator que coleta sementes no campo e entrega produtos na cidade.\n\n" +
       "Mova o trator com o mouse até o lado esquerdo para coletar sementes\n" +
       "e leve-as até o lado direito para entregá-las na cidade.\n\n" +
       "Cuidado com os obstáculos em movimento que aumentam de tamanho\n" +
       "e se multiplicam conforme você coleta sementes!",
       50, 50, 700, 500);
  botaoVoltar.show();
}

function mostraHistoriaSequencial() {
  let fundos = ['#A3DE8F', '#D0E6A5', '#FFDD94', '#FFC3A0', '#C6A49A', '#8DC7B0', '#8FBC8F'];
  background(fundos[frameHistoria % fundos.length]);
  fill('#ffffff');
  textAlign(CENTER, CENTER);
  textSize(18);
  textFont('Trebuchet MS');

  let textos = [
    "Era uma vez um vilarejo com vastas plantações e uma cidade cheia de tecnologia...",
    "Com o tempo, os dois mundos se afastaram, cada um com suas rotinas...",
    "Até que um trator carregado de sementes partiu da cidade rumo ao campo...",
    "Ele começou a plantar novas esperanças...",
    "O plano era colher e levar à cidade...",
    "E trazer inovações urbanas para ajudar o campo...",
    "Agora é com você: plante, colha e conecte os dois mundos!"
  ];

  text(textos[frameHistoria], width / 2, height / 2);
  desenhaTrator();

  if (tratorAndando) moverTrator();

  if (tratorX >= width + 80 && millis() > tempoProximoFrame) {
    frameHistoria++;
    tratorX = -100;
    tempoProximoFrame = millis() + 5000;
    if (frameHistoria >= textos.length) {
      tela = "jogo";
      jogador = { x: width / 2, y: height / 2 - 30, temSemente: false };
      faseNova = true;
      faseAtual = 1;
      sementesColetadas = 0;
      produtosEntregues = 0;
      obstaculos = gerarObstaculos(1);
    }
  }
}

function moverTrator() {
  if (tratorX <= width + 80) tratorX += 6;
}

function desenhaTrator() {
  fill(200);
  rect(tratorX, tratorY, 80, 40);
  fill(100);
  ellipse(tratorX + 15, tratorY + 40, 30);
  ellipse(tratorX + 60, tratorY + 40, 30);
}

function novoJogoInterativo() {
  background(faseAtual === 1 ? '#A3DE8F' : '#87CEFA');
  fill('#654321');
  rect(0, height / 2 - 40, width, 80);

  fill('#87CEEB');
  rect(width - 100, 0, 100, height);
  fill('#228B22');
  rect(0, 0, 100, height);

  fill(255);
  textSize(16);
  textAlign(LEFT);
  text(`Sementes: ${sementesColetadas}`, 20, 20);
  text(`Produtos: ${produtosEntregues}`, 20, 40);

  if (colisao && millis() < tempoColisao + 1000) {
    fill('#ff0000');
    textSize(24);
    textAlign(CENTER);
    text("Cuidado com os obstáculos!", width / 2, height / 2 - 100);
    return;
  }

  jogador.x = lerp(jogador.x, mouseX, 0.1);
  jogador.y = lerp(jogador.y, mouseY, 0.1);

  fill('#FFD700');
  rect(jogador.x, jogador.y, 50, 30);
  fill('#444');
  ellipse(jogador.x + 10, jogador.y + 30, 20);
  ellipse(jogador.x + 40, jogador.y + 30, 20);

  for (let obs of obstaculos) {
    fill('#8B0000');
    obs.x += obs.vx;
    obs.y += obs.vy;

    if (obs.x < 100 || obs.x + obs.w > width - 100) obs.vx *= -1;
    if (obs.y < height / 2 - 40 || obs.y + obs.h > height / 2 + 40) obs.vy *= -1;

    rect(obs.x, obs.y, obs.w, obs.h);

    if (
      jogador.x + 50 > obs.x && jogador.x < obs.x + obs.w &&
      jogador.y + 30 > obs.y && jogador.y < obs.y + obs.h
    ) {
      colisao = true;
      tempoColisao = millis();
      jogador.x = jogador.temSemente ? width - 130 : 130;
      jogador.y = height / 2 - 30;
      return;
    }
  }

  for (let p of animProdutos) {
    fill('#FFFF00');
    ellipse(p.x, p.y + sin(frameCount * 0.1 + p.offset) * 5, 10);
  }

  if (jogador.x < 100 && !jogador.temSemente) {
    jogador.temSemente = true;
    sementesColetadas++;
    animProdutos.push({ x: jogador.x + 25, y: jogador.y, offset: random(0, TWO_PI) });
    obstaculos.push({ x: random(150, width - 200), y: random(height / 2 - 40, height / 2 + 40), w: 30, h: 80, vx: random([-2, 2]), vy: random([-2, 2]) });
  }

  if (jogador.x > width - 100 && jogador.temSemente) {
    jogador.temSemente = false;
    produtosEntregues++;
    animProdutos.push({ x: jogador.x + 25, y: jogador.y, offset: random(0, TWO_PI) });
    for (let obs of obstaculos) { obs.w += 5; obs.h += 5; }
  }

  if (sementesColetadas >= 5 && produtosEntregues >= 5 && faseAtual === 1) {
    faseAtual = 2;
    jogador.x = width / 2;
    jogador.y = height / 2 - 30;
    sementesColetadas = 0;
    produtosEntregues = 0;
    obstaculos = gerarObstaculos(2);
  }

  if (faseAtual === 2 && sementesColetadas >= 5 && produtosEntregues >= 5) {
    tela = "final";
    for (let i = 0; i < 50; i++) {
      fogos.push(new FogoAleatorio());
    }
  }
}

function gerarObstaculos(fase) {
  let base = fase === 1 ? 2 : 5;
  let lista = [];
  for (let i = 0; i < base; i++) {
    lista.push({ 
      x: random(150, width - 200), 
      y: random(height / 2 - 40, height / 2 + 40), 
      w: 30, 
      h: 80, 
      vx: random([-2, 2]),
      vy: random([-2, 2]) 
    });
  }
  return lista;
}

function mostraFinal() {
  background('#FFD1DC');
  fill('#222');
  textSize(28);
  textAlign(CENTER, CENTER);
  text("Parabéns! Você completou todas as fases!", width / 2, height / 2 - 40);
  textSize(18);
  text("A conexão entre o campo e a cidade ganhou!", width / 2, height / 2);

  for (let f of fogos) {
    f.atualizar();
    f.exibir();
  }
}

function criarBotoes() {
  botaoInformacoes = createButton("Informações");
  botaoInformacoes.position(width / 2 - 60, height / 2);
  estilizarBotao(botaoInformacoes);
  botaoInformacoes.mousePressed(() => {
    tela = "instrucoes";
    esconderBotoes();
    botaoVoltar.show();
  });

  botaoIniciar = createButton("Iniciar");
  botaoIniciar.position(width / 2 - 45, height / 2 + 60);
  estilizarBotao(botaoIniciar);
  botaoIniciar.mousePressed(() => {
    tela = "historia";
    esconderBotoes();
    frameHistoria = 0;
    tratorX = -100;
    tratorAndando = true;
    tempoProximoFrame = millis() + 5000;
  });

  botaoVoltar = createButton("Voltar ao Menu");
  botaoVoltar.position(width / 2 - 65, height - 80);
  estilizarBotao(botaoVoltar);
  botaoVoltar.mousePressed(() => {
    tela = "menu";
    mostrarBotoes();
    botaoVoltar.hide();
  });
  botaoVoltar.hide();
}

function estilizarBotao(botao) {
  botao.style('background-color', '#ffffff');
  botao.style('color', '#E592AE');
  botao.style('font-size', '18px');
  botao.style('padding', '10px 20px');
  botao.style('border-radius', '10px');
  botao.style('border', 'none');
}

function mostrarBotoes() {
  botaoInformacoes.show();
  botaoIniciar.show();
}

function esconderBotoes() {
  botaoInformacoes.hide();
  botaoIniciar.hide();
}

class FogoAleatorio {
  constructor() {
    this.x = random(width);
    this.y = height;
    this.vy = random(-5, -8);
    this.cor = color(random(255), random(255), random(255));
    this.t = random(10, 30);
  }
  atualizar() {
    this.y += this.vy;
    this.vy += 0.2;
  }
  exibir() {
    noStroke();
    fill(this.cor);
    ellipse(this.x, this.y, this.t);
  }
}
