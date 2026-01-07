---
layout: post
title:  "Jornal dos Pindoramas #4: Hoje é dia de Santos Reis!"
date:   2026-01-06 22:43:00 +0300
categories: jdp copacabana
---

**Note for non-lusophones: If you can't read any Portuguese, [click here](https://takusuman-github-io.translate.goog/blog/jdp/copacabana/2026/01/06/JdP-4-santos-reis.html?_x_tr_sl=pt&_x_tr_tl=en&_x_tr_hl=pt-BR&_x_tr_pto=wapp).**

Cheguei meio tarde, mas hoje é dia de Santos Reis!  
Escrevo hoje uma edição mais corriqueira em complementação à [última edição do
JdP](https://takusuman.github.io/blog/jdp/copacabana/2025/12/31/JdP-3-reveillon.html)
e, também, para comemorar os 3 anos da primeira versão de desenvolvimento do
Copacabana --- e sim, já faz esse tempo todo.  
Eu desejava originalmente fazer um lançamento de um arquivo do sistema-base
_hoje_, mas algumas coisas aconteceram e atrasaram ~~(mais do que já estava)~~
o passo da versão 0.5. No entanto, eu pretendo conseguir lançar até dia 15.  
Não é um _devlog_ sem o _dev_, então cá a atualização sobre código em si: ainda
não fiz um commit formal, mas comecei a implementar a seção de ``chroot`` do
sistema de montagem do Copacabana e que é essencial para automatizar o resto da
compilação do sistema.  
Como gerenciamento de pacotes provisório, considerei usar a
[pkgsuite](https://github.com/mamccollum/pkgsuite), que é basicamente a
implementação original, vinda do OpenSolaris, do gerenciamento de pacotes
estilo-SVR4 que estamos implementando no
[pacote](https://github.com/Projeto-Pindorama/pacote).
O que atrasa o desenvolvimento do pacote é a falta de gente para mão-de-obra em
Go, mas compreendo que isso deva ser conquistado por notoriedade.  
Trabalho no pacote significa, também, uma boa recauchutada na libcmon, afinal,
as funções para lidar com cpio virão de lá.  
Também preciso colocar a Silicon Tabula de volta no ar, mas tenho de ver como
configurar o Hugo (ainda). 

No fim, creio que tenha sido mais uma lista de afazeres do que um devlog, mas
vai ser importante para me guiar nesse mês.  
E, diga-se de passagem, eu provavelmente vou fazer mais um devlog em
complemento a este nos próximos dias.
