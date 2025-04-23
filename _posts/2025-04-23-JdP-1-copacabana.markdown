---
layout: post
title:  "Jornal dos Pindoramas #1: O que foi feito pelo Copacabana nos últimos 4 anos?"
date:   2025-4-23 18:52:00 +0300
categories: jdp copacabana
---

**Note for non-lusophones (or people that definitively can not read Portuguese
even per mutual inteligibility): Since this is the first edition of what can be
called a "devlog"/"devjournal", it is sort of reserved for people closer to the
core of the Copacabana project, a group composed per a considerable majority of
lusophones. If you can not read Portuguese or feel more confortable reading in
your first language, a machine translator will work just fine. I intend to write
the next edition also in English.**

Primeiramente, [salve São Jorge!](https://www.google.com/search?q=dia+de+S%C3%A3o+Jorge)  
Segundamente, sim, 4 anos. É o tempo de um mandato à presidência do Brasil
(e dos E.U.A.), e é o tempo que o Copacabana tem de existência, desde a
concepção da ideia em 2021, até o primeiro lançamento entre meados de 2022
e o começo de 2023, passando por uma mudança de compilador e a automatização
do processo de montagem em 2024 até agora. Percebi que, ao invés de recomendar
que leiam a lista de _commits_ e/ou que assinem a lista de observadores do
repositório com o que é efetivamente o arcabouço da
distribuição, seria melhor escrever resumidamente sobre o que foi feito
até agora.  
Por uma questão de simplicidade, é melhor começarmos por hoje: sim, boa parte já
foi feita --- por mais que algumas coisas precisem de certa revisão, como o
[leiaute dos _scripts_ que determinam como um pacote deve ser compilado para o
sistema](https://github.com/Projeto-Pindorama/alambiko?tab=readme-ov-file#templates)
e, por consequência, [as funções que
interpretam-no](https://github.com/Projeto-Pindorama/copacabana/blob/copaclang/build-system/internals/makier.ksh)
---, apenas faltando finalizar a lista de pacotes, o que, por mais que pareça
muita coisa, é facilitado [graças ao manual escrito em 2022 para a versão
0.4](https://github.com/Projeto-Pindorama/Silicon-Tabula/blob/master/docs/copacabana/compila%C3%A7%C3%A3o.md)
e que ainda se aplica com pouca adaptação. Não há muito uma razão para se
entrar em detalhes de cada mudança feita individualmente no sistema de montagem
durante os anos de 2023 e 2024, mas posso citar algumas: um [_parser_ de INI
feito do zero a fim de se poder usar um bom formato para os arquivos de
configuração](https://github.com/Projeto-Pindorama/copacabana/blob/copaclang/build-system/internals/rconfig.shi),
novas funções para [exibir o registro das atividades em
tela](https://github.com/Projeto-Pindorama/copacabana/blob/copaclang/build-system/internals/log.shi),
funções para [manipular a variável
``PATH``](https://github.com/Projeto-Pindorama/copacabana/blob/copaclang/build-system/internals/path.shi),
suporte ao [uso do aria2 no script que transfere os arquivos compactados com
código-fonte, vulgo
``cmd/download_sources.ksh``](https://github.com/Projeto-Pindorama/copacabana/blob/2c984e57a554cef20894a0660668f6c676f98004/cmd/download_sources.ksh#L96-L101),
etc, [a lista vai
longe](https://github.com/Projeto-Pindorama/copacabana/compare/0.4...copaclang).  
Além da parte principal do projeto, do arcabouço da distribuição, também existem
os projetos associados, que vão desde o [projeto
mussel](https://github.com/firasuke/mussel/pulls?q=is%3Apr+is%3Aclosed+author%3Atakusuman),
passando pelo [Heirloom "Esquema Novo"](https://heirloom-ng.pindorama.net.br),
parte integral da _userspace_ do sistema, até coisas que parecem menores, mas
que tomam um tempo considerável de, no mínimo, dois dias para alguma mudança
relevante, como o [gerenciador de
pacotes](https://github.com/Projeto-Pindorama/pacote), que depende de algumas
funções da [libcmon](https://github.com/Projeto-Pindorama/libcmon) ainda não
implementadas e implementações alternativas de programas requisitados ---
como, por exemplo, [o _port_ do programa ``patch`` do
OpenBSD](https://github.com/Projeto-Pindorama/opatch) --- o que, sim, apesar de
parecer dispensável, sempre foi um objetivo não apenas do Copacabana, mas
essencialmente de todo o projeto Pindorama. É difícil encontrar contribuidores
para esses projetos pois, quando não são hobbistas demais ao ponto de ainda
não conseguirem ajudar por falta de conhecimento, ou estão contratados, ou estão
com um burnout para lá de conveniente --- ou, até mesmo, as duas anteriores ---,
então se torna um trabalho de uma pessoa só; o que felizmente não é o caso do
Heirloom pois, por esse ser um projeto mais geral, trouxe maior interesse externo
e, consequentemente, novos contribuidores.
