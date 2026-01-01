---
layout: post
title:  "Jornal dos Pindoramas #3: Um relatório de réveillon"
date:   2025-12-31 23:54:00 +0300
categories: jdp copacabana
---

**Note for non-lusophones: If you can't read any Portuguese, [click here](https://takusuman-github-io.translate.goog/blog/jdp/copacabana/2025/12/31/JdP-3-reveillon.html?_x_tr_sl=pt&_x_tr_tl=en&_x_tr_hl=pt-BR&_x_tr_pto=wapp).**

"Salve! Como é que vai?"  
Começo esse devlog, tão demorado (e que eu deveria ter feito antes), citando a
letra da música "Amigo é pra essas coisas", escrita pelo saudoso Aldir Blanc e
do ainda entre nós Sílvio da Silva Júnior, não a fim de remoer mágoas e
desabafar, mas sim para falar, justamente, sobre amizade.  Você, com razão,
deve estar a se perguntar o quê amizade tem a ver com o devlog em questão.  
Sendo franco, eu estava sem ideia de como começar o artigo e decidi fazer essa
citação à obra d'um artista que nos deixou à francesa no começo do agridoce
ano de 2020 e que infelizmente só é citado em abundância quando se fala da lei
de incentivo à cultura criada em seu nome. No entanto, se formos ver a fundo,
sim, a amizade, unir pessoas inspiradas e competentes, move montanhas (ou ao
menos ajuda a mover um pouco mais rápido do que sozinho). O que eu quero dizer
com isso é que novos contribuidores serão bem-vindos para esse próximo ano de
2026, mas me faltou a poesia para ligar a minha citação ao pedido na
cara-de-pau. 

Esse ano que nos deixa hoje à meia-noite foi excelente para o projeto, todavia,
como isso se trata d'um devlog e não d'uma retrospectiva, irei citar apenas o
que aconteceu desde Julho até então, ou seja, cinco meses.  
Primeiramente, depois de bastante bateção de cabeça, [as duas toolchains
necessárias para a compilação do Copacabana estão prontas e são facilmente
reproduzíveis graças ao sistema de
montagem](https://github.com/Projeto-Pindorama/copacabana/commit/43bafa0fc1917180804660b9276c5b1e5f8e93e8)
e, considerando que tudo vá certo --- isso é, não precisemos fazer nenhuma
gambiarra ---, isso também se aplicará ao sistema-base como um todo. Para o
sistema-base em si, falta apenas escrever os scripts para os demais pacotes
e compilar, o que não deve ser nada além de repetitivo. Além disso, o sistema
de montagem, mais especificamente o programa ``cmd/snapshot_stage.ksh``, [agora
suporta salvar os arquivos ``.tar`` em outra máquina por meio do
``ssh``](https://github.com/Projeto-Pindorama/copacabana/commit/ff4e111ce2da8513d7a7d90489b369b81101f629).
Ainda é uma implementação inicial, claro, mas foi útil quando testei se o
Copacabana poderia ser compilado em uma máquina virtual com Debian.  
Ainda sobre as toolchains, e para não dizer que não falei de amigos, descobri
que o [GCC 15 não consegue compilar o LLVM
17.0.5](https://github.com/Projeto-Pindorama/alambiko/commit/d2dccf886d1a70b8a8e6f7b59f884e44be8975cb),
ainda usado pelo Copacabana, após atualizar a mussel. Avisei o Firas sobre o
incidente e, enquanto pesquisava, descobri que isso é [facilmente
resolvido](https://github.com/firasuke/mussel/commit/8c931ddb2f794ee10de65732bef18ef311d2ca2b#commitcomment-170530236).
Ainda creio que a mussel deva ser versionada para evitar esses pequenos
"sustos", mas não sei se isso está nos planos.

No Heirloom, parte da _userspace_ do Copacabana, consertamos diversas falhas
relacionadas ao gerenciamento de memória, com o ``apply`` recebendo código para
fazer alocações mais inteligentes de memória, removemos a função ``afterchar()``
da libcommon, um _hack_ feito por mim em 2024 e, sobre a experiência de usuário
direta, implementamos o uso da função ``system()`` no programa ``watch`` por
padrão ao invés da ``exec()`` (agora habilitada pela opção '``-x``') e
implementamos a opção '``-0``' no ``xargs``, comumente utilizada junto com o
``find`` e, além disso, as páginas de manual para o ``apply`` e para o ``seq``
também receberam um "trato" em nome da clareza e, também, da "corretude", já que
havia exemplos incorretos e informações faltando. Por fim, vale lembrar que
nossa bifurcação do Heirloom foi citada em um artigo da OSNews por Thom Holwerda
no começo do ano, mas só tomei conhecimento disso em Outubro.  

Falando em mídia, também recuperamos o acesso à conta do Twitter,
[``@pindorama_prj``](https://twitter.com/pindorama_prj), depois de pagarmos a
bendita mensalidade que o Bol vinha cobrando, o que foi convenientemente
mais barato do que fazer todo o caminho burocrático para conseguir o acesso
gratuito novamente. Por que será?  
A única parte chata é que ainda recebo ligações da equipe de marketing do UOL me
oferecendo acesso a seguro para carro, casa e celular mas, fora isso, tudo perfeito.
Nós usamos o Outlook agora de qualquer maneira.

E, por fim, esse foi um bom ano, mas o que está por vir há de ser melhor ainda.  
E eu ainda tenho de organizar melhor os calendários do projeto e as metas, esse
ano poderia ter sido bem mais produtivo (e eu espero não dizer isso no último
devlog do ano que vem). 

Feliz Ano Novo!
