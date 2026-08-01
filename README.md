Bolsa de Valores - Jogo Digital
Recriação digital do clássico jogo de tabuleiro Bolsa de Valores baseado  instruções originais.

📜 Regras Implementadas (fiel ao manual)
INÍCIO

Cada jogador escolhe um cone colorido (aqui: cor do jogador) e coloca no ponto de partida. Dado decide ordem de largada.
A Bolsa atua como banco, pagando e recebendo pela compra e venda das ações.
Marcadores coloridos: vermelhas, azuis, amarelas ou verdes = ações. Marrom = Commodities (café, petróleo, energia e ouro). Todos começam em R$ 50,00 na prancha "Cotação da Bolsa".
Ficarão na Bolsa: 3 notas de R5, 3 de R$10, 4 de R$50 e 4 de R$100 = **R 645**. O restante é dividido entre jogadores (aqui simplificado: R$500 cada).
COMO JOGAR

Lança dado e anda número de casas.
Será obrigatório comprar uma ação da cor correspondente à casa que parou: da Bolsa pelo preço da Cotação ou de outro jogador pelo preço negociado.
Commodity: compra NÃO obrigatória.
VALOR DAS AÇÕES

Depois de andar e comprar a ação da cor da casa:
Se dado foi 1,2 ou 3 o valor cai (Ex: 50 → 30, 100 → 75)
Se dado foi 4,5 ou 6 o valor sobe (Ex: 50 → 75, 100 → 150)
Implementado com escada: [10,20,30,50,75,100,150,200,300,500]
IMPORTANTE

Sem dinheiro para comprar → vender suas ações para a Bolsa que paga máximo R$50 por ação, mesmo que a Cotação seja maior, ou vender para outro jogador.
COMODITIES

Café, Petróleo, Energia e Ouro não são compra obrigatória. Pode passar sem comprar. Vendidas para Bolsa pelo preço da Cotação (diferente das ações comuns que valem max 50).
Ao parar em Commodity já comprada, pagar R$100 para cada proprietário. Eliminado quem não tiver dinheiro.
🗂️ Estrutura do projeto
/index.html
/style.css
/script.js
/README.md
▶️ Como rodar
Basta abrir index.html no navegador. Sem dependências.

🎮 Controles
Selecione 2-4 jogadores e clique Novo Jogo
Clique em Lançar Dado
Siga os modais de compra obrigatória / opcional
Log inferior mostra histórico da Bolsa
Feito para preservar o clássico brasileiro!
