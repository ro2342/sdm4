# PROGRESS — Show do Milhão 4 (clone web)

## Estado atual (2026-10-22)

Clone em `index.html` (arquivo único) feito a partir da engenharia reversa real
dos 25 arquivos `.frm`/`.bas` decompilados em `C:\Users\Rod\Downloads\sdm4\sdm4new\`.
Zero posição/asset/comportamento inventado — tudo rastreado até o VB6 original
(twips convertidos pra pixel/%, nomes de arquivo reais, sons no tempo certo).

**Publicado como Artifact:** https://claude.ai/code/artifact/004bb827-8429-48b1-abd8-bdcb25b3651d
(link antigo `claude.ai/artifact/JCM6Bj...` parou de aceitar update nesta sessão — "not an artifact URL" — provavelmente API de artifact diferente entre o chat normal e o Claude Code. Este é o link ATUAL, usar este daqui pra frente.)

**Importante:** esta pasta (`_webapp_clone/`, 250+ arquivos) está **sem commit**
no repo `sdm4` (root em `C:\Users\Rod\Docs\backup-linux\MILHAO4`, remote
`github.com/ro2342/sdm4`). `git status` mostra ela inteira como untracked.
Não commitei porque isso não foi pedido ainda — avisar o Rod e perguntar se
quer commitar/subir quando ele voltar.

## Arquivos-fonte já revisados (processo "um arquivo por vez")

| # | Arquivo original | Status |
|---|---|---|
| 1 | APRESENT.frm | ✅ corrigido (vídeo centralizado, tamanho nativo 320x240) |
| 2 | JOGREG.frm | ✅ corrigido (posições de botão, sons, hover removido) |
| 3 | TELREGRAS.frm | ✅ corrigido (cluster de botões bottom-right, sem disable, hover removido) |
| 4 | FRMASSUN.frm | ✅ corrigido (posições das 7 matérias, botão OK realinhado) |
| 5 | NOMES.frm | ✅ corrigido (som LETRA por tecla, sons de confirmar/sair trocados) |
| 6 | VALENDO.frm | ✅ corrigido (sparkle extraído do .frx, filename real `{valor}.jpg`→`valendo_{valor}.jpg` local, tamanho certo) |
| 7 | PLACARG.frm | ✅ corrigido (posições de nome/placar, botões Zerar/Jogar via medição de pixel) |
| 8 | FUNDOBLK.frm | ✅ revisado — não precisa de UI web (só detecção de CD-ROM/DB, sem tela visível) |
| 9 | FRMFIM.frm | ✅ corrigido (removido botão "Voltar" inventado, toca tchau.wav, volta sozinho ao menu em ~2.8s) — **verificação visual no browser ficou inconclusiva** (ver abaixo) |
| — | TELAJOGO1.frm | Reescrito em sessão anterior; revisado a fundo nesta sessão (ver seção de correções abaixo) |
| 10 | QTGANHOU.frm | ✅ corrigido nesta sessão (ver correção 5ª rodada abaixo) — tela de entrega do prêmio |

## Correção 2026-10-22 — tela de jogo (TELAJOGO1.frm) não estava fiel

Usuário reportou (em caps, review ao vivo): botões SIM/NAO menores que o
original, som QUALE tocando por cima do som anterior, FAIXA.jpg menor que
o original, e um "CORRETO!" aparecendo do nada sem o usuário reconhecer
de onde veio.

**Causa raiz (uma só, sistêmica):** o `.frm` da tela de jogo declara
`ClientWidth=11400` twips (760px), mas o fundo real `TELA1JOGADOR.jpg` é
640px — confirmado medindo o arquivo. Toda posição/largura horizontal que
eu converti usando `/11400` como base saiu ~19% mais estreita/deslocada
do que deveria (base certa é 640px = 9600 twips). Afetava: caixa da
pergunta, linhas de resposta, faixa de revelação, painel de ajuda, e os
hotspots SIM/NAO/PARAR/AJUDA (que além disso eu tinha modelado errado —
no original são 4 controles DIFERENTES com posição e tamanho próprios,
não uma caixa genérica trocando de imagem).

**Fixes aplicados:**
- Recalculada toda posição horizontal de `#screen-game` usando base 640px
  (não 760px) — pergunta, respostas, faixa de revelação, painel de ajuda.
- SIM/NAO/PARAR/AJUDA: 4 hotspots com posição+tamanho reais (extraídos do
  código-fonte decompilado `TELAJOGO1.js`, coordenadas code-set em pixel).
- Som "QUALE" sobrepondo o anterior: implementado canal único de áudio
  (igual ao MediaPlayer real do VB6 — carregar som novo corta o anterior)
  em vez de cada som ser um `Audio()` independente sem coordenação.
- Removido banner de texto "CORRETO!"/"ERROU!"/"TEMPO ESGOTADO!" — não
  existe NENHUM Label com esse conteúdo no Begin block do `.frm`, era
  invenção minha. A revelação real é só a faixa piscando + áudio.
- Estouro de tempo agora usa o mesmo fluxo de resposta errada (revela a
  faixa certa antes de perder), em vez de pular direto pro resultado.

**Correção extra (mesmo dia, 2ª rodada):** usuário notou que a faixa
"ainda tem mudança de tamanho". Causa: as 4 imagens `faixa{1-4}.jpg` têm
tamanho NATIVO levemente diferente entre si (306x47, 308x45, 312x44,
299x44px) — eu tinha forçado todas numa caixa CSS fixa de 312x44, o que
estica/espreme cada uma de um jeito diferente (mesmo padrão de bug já
visto: Image sem Stretch = tamanho nativo, não deveria ter box fixo).
Fix: width/height agora são setados por imagem em JS (`FAIXA_SIZE`),
usando o tamanho nativo real de cada arquivo.

**Correção extra (mesmo dia, 3ª rodada):** usuário perguntou se o vermelho
da resposta selecionada é fiel (sim — confirmado `&HFF` = vermelho puro
em BGR, comentário já existia na tradução original) e notou assets não
usados (`tapa_ajuda.jpg`, `dois.jpg`, `tres.jpg`, `rei.jpg`, `as.jpg` em
`Jpgs1`). Investigando: `tapa_ajuda/cartas/placas/convidados.jpg`
pertencem TODOS ao `TELAJOGO.frm` (modo 2 jogadores, ainda não feito) —
confirmado via grep, não aparecem em NENHUM lugar do `TELAJOGO1.frm`.
Eu tinha usado `tapa_cartas/placas/convidados.jpg` ERRADAMENTE no painel
de ajuda do modo 1 jogador; o fundo real desses 3 painéis é
`AJUDA0A/1A/2A.jpg` (mesmo arquivo do ícone "já usado" no menu,
confirmado em `AJUDA_Click`). Trocado. Também faltava o mecanismo real
da ajuda "Cartas" (`CARTA_CLIQUE_Click`): sorteia 1 de 4 cartas
(rei=0 erradas eliminadas, ás=1, dois=2, três=3, sobra só a certa) e
MOSTRA a carta sorteada — eu só fazia a eliminação sem nenhum feedback
visual de qual carta saiu. Corrigido: copiados `rei.jpg`/`as.jpg`/
`dois.jpg`/`tres.jpg` de `Jpgs1`, mostrados por 1.4s antes de fechar o
painel.

**Correção extra (mesmo dia, 4ª rodada — eu estava ERRADO na 3ª):**
usuário discordou, dizendo que olhou o `tapa_ajuda.jpg` de verdade e viu
o fundo redondo já com os 6 ícones desenhados — "as outras imagens de
ajuda ficam em cima dessa". Abri as imagens pra conferir visualmente em
vez de confiar só no grep do código: ele estava certo. `tapa_cartas.jpg`
mostra o leque de 4 cartas no círculo (bate exato com o painel Cartas),
`tapa_placas.jpg` mostra as placas "1 2 3 4" no círculo (bate com
`percentual_placa`), `tapa_ajuda.jpg` é o fundo completo do menu de
seleção com os 6 ícones. Causa raiz do meu erro: esses fundos são
`Picture` EMBUTIDO no `.frx` de cada PictureBox (CARTAS/PLACAS/
CONVIDADOS/Picture1), definido em tempo de DESIGN — nunca aparecem como
string "tapa_X.jpg" em nenhum `LoadPicture` do código decompilado
(só existem no binário .frx, que eu não abri byte a byte). Grep no
código só prova ausência de uma chamada em RUNTIME, não ausência de uso
real — lição pra próxima vez: confirmar visualmente antes de confiar só
em busca textual quando a pergunta é "esse asset é usado ou não".
Revertido para `tapa_ajuda/cartas/placas/convidados.jpg` (que eu já
tinha usado corretamente desde o início, antes da 3ª rodada estragar).
`AJUDA0A/1A/2A.jpg` continuam certos, mas só pro papel de ícone pequeno
"já usado" (risco vermelho) dentro do menu de 6 ícones, não como fundo
de painel.

## Correção 2026-10-22 (5ª rodada) — QTGANHOU.frm (tela de entrega do prêmio)

Usuário apontou que essa tela estava "completamente bugada" e que eu
tinha pulado o processo arquivo-por-arquivo pra ela (feita em sessão
anterior, só revisada por cima). Fiz o processo completo agora: li o
`.frm` inteiro + o `.js` já traduzido, comparei com `index.html`.

**3 bugs reais confirmados e corrigidos:**
1. **Prêmio de R$1.000.000 pulava a animação inteira da maleta** no
   original (`Form_Activate`: detecta 1 milhão e vai DIRETO pro vídeo
   `encerra.avi`, sem tocar `TEMA_2.wav`, sem cascata de frames) — eu
   rodava a cascata completa igual a qualquer outro valor.
2. **Imagem "p1000000.jpg" não deveria ser usada** — pro caso de 1
   milhão o original nunca troca a imagem do prêmio, só corta pro vídeo.
3. **Tempo de exibição do valor ganho estava em milissegundos, não
   segundos** — original segura a imagem na tela por 6s reais (ganhou
   algo) ou 3s (perdeu tudo) antes do vídeo; eu tinha 400-1000ms.
   Confirmado por bônus: `encerra.mp4` dura exatamente 10.01s, batendo
   com os 10 ticks (~10s) do original antes de ir pro placar — então
   `video.onended` (que eu já usava) é equivalente, não precisou trocar.

## Correção 2026-10-22 (6ª rodada) — QTGANHOU tinha o MESMO bug de base de largura

Usuário corrigiu 2x seguidas: (1) vídeo `encerra.avi` só deveria tocar
pra quem ganha exatamente 1 milhão — eu tocava sempre; (2) a tela de
entrega do prêmio estava "completamente deformada". Causa raiz do item 2:
mesmo bug do TELAJOGO1 (base de largura errada) — `.frm` declara
ClientWidth=8355 (557px), fundo real `QTGANHOU.jpg` é 640px. O
reposicionamento do frame 8+ (pixel 136,104 code-set) tinha saído
**1.63%,1.44%** (quase o canto superior esquerdo) em vez do correto
**21.25%,21.67%** — erro de ~13x, essa era a "deformação".

Depois de corrigir, **auditei os 12 `.frm` inteiros** comparando
ClientWidth declarado vs. 640px (dividindo tudo por 15 pra achar o
pixel real): só TELAJOGO1 (760px), QTGANHOU (557px) e TELAJOGO —2
jogadores, ainda não feito— (756px) têm diferença grande. Os outros 9
ficam a 1-3px de 640 (arredondamento normal do VB6, sem bug real). Isso
fecha essa classe de bug nos arquivos já feitos, e já sei que vou
precisar aplicar /640 (não /756) quando for fazer o TELAJOGO.frm.

Fixes aplicados em QTGANHOU: vídeo só pro caso de 1 milhão (resto vai
direto pro placar sem vídeo), posição/tamanho do Image1 (21.875%,21.46%,
59.06%,72.71% em vez de 25.13%/67.86%), reposição do frame 8+
(21.25%,21.67% em vez de 1.63%/1.44%).

## Correção 2026-10-22 (7ª rodada) — PLACARG.frm tinha bug real que eu não tinha visto

Usuário perguntou "placar você não fez? tem certeza?" — em vez de só
reafirmar que sim, reconferi de verdade: contei os 26 arquivos fonte
(12 `.frm` + 12 `.frx` + `Module1.bas` + `Project.vbp`, bate com os "26
arquivos" que o Rod lembrava) e reli o `PLACARG.frm` inteiro de novo.

**Achei um bug real que não tinha sido visto antes:** os 10 tops das
linhas de nome/pontuação estavam sistematicamente deslocados pra cima
(erro crescente: ~1.6 pontos percentuais na linha 1 até ~2.4pp na linha
10 — ~8-11px num canvas de 480px). Confirmei escaneando os pixels reais
de `PLACARG.jpg` (as 10 barras amarelas) e comparando com
`Label1,3,5...19.Top` do `.frm` — os tops recalculados direto do `.frm`
batem quase exato com as barras reais; os valores antigos, não.
Corrigido: 23.13%, 31.04%, 38.33%, 46.25%, 53.54%, 61.46%, 69.17%,
76.88%, 84.38%, 91.88% (eram 21.5% a 89.5%).

Também conferi os botões Zerar/Jogar (escaneei os círculos verdes reais
na imagem) — esses já estavam OK, só levemente maiores que o círculo
real, mas são hotspots invisíveis, sem problema de alinhamento visual.

## Decisão 2026-10-22 — TELAJOGO.frm (modo 2 jogadores) NÃO será implementado

Rastreei o fluxo completo de `NOMES.frm` → `VALENDO.frm` e confirmei na
fonte decompilada BRUTA (não é ambiguidade de tradução): a função que
confirma o nome e avança pro jogo (`Proc_11_10_53C1A8`) e o handler do
estado "WAIT" logo depois têm `MemVar_55008C = CStr()` — zera o nome do
jogador 2 incondicionalmente, sem nenhum `If` checando o que foi
digitado no campo 2. `VALENDO.frm` decide 1 ou 2 jogadores baseado
exatamente nesse valor estar vazio ou não. Ou seja: mesmo digitando os
dois nomes na tela, o jogo sempre entra em modo 1 jogador por esse
caminho — não encontrei nenhuma UI que leve até `TELAJOGO.frm`.

Rod confirmou que o modo 2 jogadores existe de fato no jogo original
(é um recurso real da série), mas nunca jogou com ele — não resolve se
era alcançável, só que não foi testado por ele. Apresentei 3 opções
(implementar fiel + destravar / implementar fiel + deixar inalcançável
igual o original / não implementar) — Rod escolheu **não implementar**,
já que ninguém chegaria lá de qualquer forma. `TELAJOGO.frm` fica
oficialmente fora de escopo, não é um "esquecido".

## Atualização 2026-10-22 (8ª rodada) — TELAJOGO.frm foi implementado e depois desativado

Depois da decisão acima, o Rod pediu pra implementar mesmo assim (fiel ao
original, incluindo o bug de "a partida acaba quando QUALQUER jogador cai"
e o bug do nome do jogador 2 sendo limpo — CORRIGIDO, já que destravar
esse era o pedido). Construí:

- `NOMES.frm`: nome do jogador 2 agora é preservado de verdade (bug real
  corrigido, só esse — o resto fica fiel ao original).
- `#screen-game2` completo: fundo dividido LADOE.jpg/LADOD.jpg, pergunta/
  coloca_ouro/painel de ajuda/botões reposicionados por lado (código-set,
  base 640x480 — TELAJOGO.frm também tem a mesma classe de bug de largura
  declarada errada, 756px vs 640px real, confirmado e já usado certo desde
  o início). Ajudas (Cartas/Colegas/Convidados/Pular) por jogador, com
  progresso independente. ~92 assets novos copiados de Jpgse/Jpgsd/
  Pastilhas (prefixo e_/d_).

**Travado no limite de arquivos do Artifact:** o manifesto aceita no
máximo 256 arquivos; o modo 1 jogador sozinho já usa 255. Testei
publicação parcial (só os arquivos novos) — **quebra os assets antigos**
(tela fica preta), confirmado ao vivo. Pra caber os +92 arquivos do 2P
seria preciso juntar vários ícones em sprites, incluindo os 41 quadros
do relógio (`relogio_0-40.jpg`, compartilhado com o 1P) — única conta que
fecha. Apresentei as opções pro Rod (sprites até no relógio / sprites só
no que é novo + outro corte / simplificar sem cor por lado) e ele decidiu
**não usar o modo 2 jogadores por enquanto**, pra não arriscar mexer em
código 1P já testado.

**Estado final:** o código do modo 2 jogadores (`game2`, `new2PGame`,
`#screen-game2` e tudo relacionado) continua no arquivo, completo e
funcional — só não tem campo de nome do jogador 2 na tela (removido) e os
assets `e_`/`d_`/`pastilhas_e`/`pastilhas_d` não estão no manifesto
publicado (ficam só em disco local). Se decidir retomar, o caminho mais
direto é implementar os sprites (ou aceitar uma das outras opções) e
voltar a publicar o manifesto com +92 arquivos.

## Correção 2026-10-22 (9ª rodada) — áudio da tela VALENDO cortava antes de acabar

Usuário reportou: o áudio do valor (1000 e todos os outros) cortava e ia
pra pergunta antes de terminar. Causa: o `setTimeout` de segurança (pra
nunca travar a tela se o áudio falhar) estava em 6000ms, mas medi os
arquivos reais e muitos passam disso — `Rodada1.ogg`=8.6s,
`Rodada2.ogg`=10.15s, `v1000000.ogg`=8.4s, `v200000.ogg`=8.1s. A
salvaguarda virou o gatilho principal (errado) em vez de só backup.
Corrigido pra 15000ms, com folga real sobre o maior áudio medido.

## Decisão 2026-10-23 — publicado no GitHub Pages (mudou de ideia)

Rod perguntou sobre gerar um `.apk` (cancelado, ia gastar muito token),
depois sobre "atalho no celular tipo PWA" — expliquei que o link do
Artifact sempre carrega com o cabeçalho do claude.ai em volta (não dá
pra tirar), e que um PWA de verdade (instalável, tela cheia, sem
cabeçalho) precisava de hospedagem própria. Rod decidiu então ir pro
GitHub Pages público mesmo (reverteu a decisão anterior de ficar só no
Artifact — confirmado explicitamente, mesmo sabendo que o repo é
público).

**Site no ar:** https://ro2342.github.io/sdm4/

**O que foi feito:**
- `_webapp_clone/` finalmente commitado no branch `main` (353 arquivos,
  primeiro commit — estava sem commit desde sempre, ver histórico acima).
- Arquivo principal virou `game.html` (mesmo conteúdo de sempre, usado
  pro Artifact) + novo `index.html` (documento HTML completo de verdade,
  com `<!doctype>`/`<head>`/manifest — só um `<iframe>` tela-cheia sem
  borda apontando pro `game.html`). Isso evita duplicar o jogo inteiro
  em dois arquivos — só um wrapper fino.
- `manifest.json` + `icon-192.png`/`icon-512.png` (logo recortado de
  `SHOW4.jpg`, sobre fundo azul escuro) — PWA instalável de verdade,
  `display:fullscreen`, `orientation:landscape` (trava no formato 4:3
  pedido, sem precisar redesenhar nenhuma arte).
- Testado localmente num server HTTP antes de publicar (sem cabeçalho,
  vídeo rodando, 4:3 centralizado) e depois confirmado ao vivo igual.
- Deploy via branch órfão `gh-pages` na raiz do repo (não mexe no
  histórico do `main`) — GitHub Pages já estava configurado apontando
  pra esse branch (provavelmente de alguma tentativa anterior), só
  precisei dar push que o build rodou sozinho.

**Pra instalar no celular:** abrir o link acima no Chrome (Android) ou
Safari (iPhone) → "Adicionar à tela inicial" — agora SEM o cabeçalho do
claude.ai, ícone customizado, abre em tela cheia.

Link do Artifact (`claude.ai/code/artifact/...`) continua funcionando
também, sem mudança — os dois ficam no ar em paralelo.

## Correção 2026-10-23 (10ª rodada) — Firefox escurecia botões + ouro-pill desalinhado

Rod reportou 2 coisas:

1. **Firefox**: os botões SIM/NÃO (e qualquer botão clicável) ficavam
   com uma cor mais escura por cima, tipo hover — não acontecia no
   Chrome. Causa: `background:transparent` não remove a aparência
   NATIVA de `<button>` no Firefox (ele mantém o visual de botão do SO
   por baixo, que escurece em hover/clique/foco); precisa de
   `appearance:none` explícito, que o Chrome já ignora por padrão sem
   precisar pedir. Adicionado `button{ appearance:none; ... }` global,
   cobrindo todo botão do projeto de uma vez.
2. **OURO desalinhado**: medi os 16 arquivos `OURO{1-16}.jpg` reais —
   todos exatamente 220x61px. Meu CSS usava um arredondamento (34%/12%)
   em vez da fração exata (34.375%/12.708%) e o canto em pixel exato
   (413,3). Pequeno mas real — corrigido pra bater exato com o asset.

Publicado nos dois lugares (Artifact via `game.html` + commit/push em
`main` e `gh-pages`).

## Correção 2026-10-23 (11ª rodada) — Firefox AINDA escurecia botões (fix anterior não bastou)

Rod reportou de volta, em caps: "AINDA ESTA ESCURECENDO OS BOTOES" — o
primeiro fix (`appearance:none` + hover/active/focus `background:transparent`)
não resolveu. Causa provável: Firefox também aplica `box-shadow`/`filter`
próprios em `:hover`/`:active`/`:focus` no `<button>` nativo, e
`::-moz-focus-inner` soma um padding+borda interna invisível que participa
do escurecimento — e a especificidade do UA stylesheet do Firefox é
diferente da do Chrome, então as regras antigas sem `!important` podiam
estar perdendo a disputa.

Reset bem mais agressivo aplicado:
```css
button{
  appearance:none !important; -webkit-appearance:none !important; -moz-appearance:none !important;
  background:transparent !important; border:0 !important; outline:0 !important;
  box-shadow:none !important; filter:none !important; color:inherit;
}
button::-moz-focus-inner{ border:0 !important; padding:0 !important; outline:0 !important; }
button:hover, button:active, button:focus, button:focus-visible{
  background:transparent !important; box-shadow:none !important; filter:none !important;
}
```

Publicado no Artifact + commit/push `main` + `gh-pages` nesta rodada.
**Não consigo testar Firefox com as ferramentas que tenho (só Chrome via
claude-in-chrome)** — preciso que o Rod confirme se resolveu de vez ou se
ainda escurece.

## Decisão 2026-10-23 — fonte original VB6 subida pra repo privado separado

Rod pediu pra subir os 25 arquivos fonte decompilados (`sdm4new/`) pro
GitHub também. Avisei que isso é diferente do clone — é o código original
da empresa (SBT/Softmed), não reimplementação — e perguntei se ele queria
público ou privado. Escolheu **privado**, num repo separado (não dentro de
`sdm4`, pra não misturar com o clone público).

**Repo:** https://github.com/ro2342/sdm4-fonte-original (privado) — pasta
`sdm4new/` com os 25 arquivos (12 `.frm`, 11 `.frx`, `Module1.bas`,
`Project.vbp`). Cópia feita de `C:\Users\Rod\Downloads\sdm4\sdm4new\`, que
continua sendo a fonte de trabalho local (esse repo é só backup/histórico).

## Correção 2026-10-24 (12ª rodada) — som do valor (v50000.ogg etc) as vezes não tocava

Rod reportou jogando no Firefox: saindo da pergunta de 40 mil, a tela de
50 mil não tocou o áudio falado do valor — foi direto pro `Quale.ogg` da
pergunta, como se o som tivesse sido pulado.

**Causa real**: os ~20 arquivos de valor (`v1000.ogg`...`v1000000.ogg`) têm
`preload="auto"` desde o carregamento da página, mas o navegador não baixa
todos ao mesmo tempo — em conexão mais lenta ou sessão mais longa, um
arquivo usado só numa rodada mais tarde (50 mil, 100 mil, 1 milhão...) pode
ainda não ter dado suficiente carregado (`readyState < HAVE_CURRENT_DATA`)
no instante exato em que a rodada chega nele. Chamar `.play()` nesse estado
pode rejeitar a Promise (ou `currentTime=0` falhar), caindo direto no
`.catch()` que já existia pra avançar a rodada — ou seja, o próprio código
de segurança contra travamento era o que pulava o áudio.

**Fix**: nova função `whenAudioReady(audio, cb)` — só chama `.play()` quando
o áudio sinaliza `canplay`/`loadeddata` (ou já está pronto), com teto de 4s
de espera. Aplicado nos dois pontos de reprodução (jingle `valor.ogg` + voz
do valor) no modo 1 jogador, e espelhado no código dormente do modo 2
jogadores pra manter consistência.

**Não testado ao vivo** (extensão do Chrome desconectada, Rod jogando no
Firefox) — publicado por análise de código + raciocínio sobre a causa, não
por reprodução visual. Pedir confirmação do Rod depois de jogar de novo.

Publicado no Artifact + commit/push em `main` e `gh-pages`.

## Pendente

Nenhum arquivo `.frm` pendente — os 11 alcançáveis (modo 1 jogador +
telas compartilhadas) estão feitos, revisados e publicados. O 12º
(TELAJOGO.frm, modo 2 jogadores) está implementado no código mas
desativado na UI por causa do limite de arquivos do Artifact (ver acima).
2. **Verificação visual do fix do FRMFIM** — a ferramenta de screenshot do
   Chrome (`Page.captureScreenshot` via CDP) trava com timeout de 30s toda
   vez logo após qualquer clique na página (em 3 abas diferentes, inclusive
   abas novas). `document.readyState` confirma a página sempre responsiva —
   é falha pontual da ferramenta de automação, não da página. Validação
   estática (sintaxe JS, cross-check de IDs, publish correto) passou; só
   faltou confirmar visualmente que clicar em SAIR mostra FIM.jpg, toca
   tchau.wav e volta ao menu sozinho.
3. Detalhes de baixa prioridade sinalizados e **não implementados** (decorativos,
   não afetam lógica): animação de revelação linha-a-linha do PLACARG
   (BARRA0-10 + LETRA.WAV), animação de rotação BitBlt do FRMFIM.

## Como retomar

- Publish usa `Artifact` com `root="."` e o `files` COMPLETO gerado por
  `python gen_files_map.py` (no scratchpad) — nunca mandar só o delta, senão
  o publish incremental derruba assets antigos (bug recorrente já confirmado
  3x+ nesta sessão).
- Antes de cada `Artifact`, rodar `PowerShell` + `Set-Location` pro diretório
  de publish — o tool de Artifact segue o cwd do PowerShell, não do Bash.
- Regra geral confirmada pelo Rod: **nenhum hover/press/efeito cosmético que
  não exista no VB6 original** (sem MouseMove handler no .frm = sem hover no
  clone).
