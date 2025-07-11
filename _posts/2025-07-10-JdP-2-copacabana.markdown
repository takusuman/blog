---
layout: post
title:  "Jornal dos Pindoramas #2: Copacabana, Superbacana"
date:   2025-7-10 23:49:00 +0300
categories: jdp copacabana
---

**Note for non-lusophones: If you can't read any Portuguese, [click here]().**

Desde a última (e primeira) edição desse jornal --- que é mais um "devlog" ---,
certo progresso vem a ser feito. Apesar de estar a um tempo sem fazer nenhum
teste e nenhuma alteração significativa no repositório principal do Copacabana
--- o que confesso que deveria ser a prioridade quando consideramos a ideia de
um sistema-base em si ---, houve trabalho significativo em relação aos
programas que vão no espaço de usuário e coisas relacionadas a esses.  
Resumindo, mais dois programas serão adicionados: o
[``zipar``](https://github.com/Projeto-Pindorama/zipar), que pretende substituir
os programas ``unzip`` e ``zip`` do pacote Info-ZIP de forma gradual, e o
[``bzip2(2)``](https://github.com/pedroalbanese/bzip2), que é uma reimplementação
do programa ``bzip2`` em Go feita originalmente pelo
[Pedro F. Albanese](http://albanese.atwebpages.com) e que vem a ser desenvolvida
em trabalho conjunto para que seja compatível com a implementação original em C.  
O desenvolvimento do gerenciador de pacotes
[Pacote](https://github.com/Projeto-Pindorama/pacote)
também foi retomado nesse tempo e passará a receber mais atenção durante as
próximas duas semanas, com um "plano de ataque" do que deverá ser implementado, o
que é basicamente um decalque feito em cima da documentação provida pela própria
Sun Microsystems somada ao código-fonte que foi liberado por meio do OpenSolaris.
Sim, esse plano é antigo, mas apenas agora estamos conseguindo colocar em prática. 
Por conta desses projetos, os três escritos em Go, a libcmon precisará receber
mais código (e passar por uma refatoração na biblioteca para discos) e tivemos
que fazer uma bifurcação da biblioteca ``getopt`` escrita pelo Russ Cox, visto
que ele não parece ser muito fã de aceitar contribuições.   
Indo para o Heirloom NG, os testes serão concluídos até o próximo dia 20 e
um artigo mais detalhado, que vem sendo escrito desde o ano passado e que passou
o último ano no fundo da geladeira, será publicado junto. Além disso, algumas
ferramentas estão precisando de uma revisão após uma mudança feita na última
versão, que se tratou de uma mudança da escrita na declaração de um _array_ de
caracteres em C, já que, caso a mudança tenha sido feita de forma errada, essas
podem facilmente ter alguma falha de segmentação escondida prestes a explodir na
fuça de alguém.  
Sendo mais específico sobre o Copacabana, várias ações já foram planejadas e
tiveram suas prioridades levantadas, com a primeira sendo finalizar a compilação
do sistema por meio do sistema de montagem atual e, consequentemente, a segunda
se trata do foco no desenvolvimento de uma ferramenta para gerar o arquivo de
inicialização do sistema, vulgo ``initrd``, o que vai possibilitar, além de uma
instalação padrão com um esquema de particionamento moderno, também a criação de
uma ISO inicializável para a arquitetura AMD64; brevemente após isso, também será
criada uma imagem que funcione em aparelhos ARM do tipo TV Box, com certa base no
[_port_ do Armbian para esse tipo de aparelho que já foi desenvolvido pelo Paolo
Sabatino](https://github.com/paolosabatino/armbian-build). Se tudo correr bem em
relação ao _port_ para ARM, podemos voltar a ter nosso servidor próprio e a
hospedar nossos arquivos novamente, sem depender __apenas__ do GitHub --- ou
ainda do SourceForge, no caso do Heirloom.  
E, por último, no dia 20 do último mês (6), o Copacabana também ganhou logo e
banner com uma identidade visual própria, diferente da anterior que era baseada
inteiramente no "banner-logo", digamos assim, do próprio Pindorama. Foi criada
pelos meus amigos de longa-data [Asriel](https://github.com/asriel8691) e
[Samuel](https://www.youtube.com/channel/UCQVzi7rdAdFPgWSm_DkNUdA).
Dê uma olhada (enquanto eu ainda não coloco no site):

![Banner do Copacabana Linux](https://raw.githubusercontent.com/Projeto-Pindorama/artworks/refs/heads/master/Pindorama%20Copacabana%20Banner/banner%20degrad%C3%AA.png)

Apesar de parecer algo meramente estético, essa mudança vem num momento
importante: apesar do plano geral para o Copacabana ainda ser o mesmo,
as coisas estão consideravelmente mais ligeiras. O jogo é rápido e fazemos
questão de que os rumores do desenvolvimento não se confinem a boatos, bares
e boates. _"Toda essa gente se engana ou então finge que não vê que eu nasci
para ser o Superbacana."_
