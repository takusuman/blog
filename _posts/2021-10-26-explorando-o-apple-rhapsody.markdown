---
layout: post
title: "Explorando o Apple(R) Rhapsody"
---

*Nota: texto W.I.P (work in progress), coisas podem mudar ao longo do tempo*  

# O que é esse Rhapsody?
O Apple(R) Rhapsody é um sistema operacional UNIX-compatível, foi uma prova de conceito
por parte da Apple antes de apostar na prática no macOS atual; é, de certa forma, a
transição do velho e limitado MacOS de kernel monolítico (ou nanokernel para PowerPC) para
os novos macOSes baseados em kernel híbrido e UNIX-compatíveis.  
A Apple resolveu usar como base o kernel Mach da Universidade Carnegie Mellon com 
**várias** modificações --- como, por exemplo, a ideia de servidores de kernel carregáveis
e o servidor Bootstrap que permitiria processos abrirem soquetes que poderiam ser
conectados por outros processos para mandarem mensagens e afins (como um FIFO, me corrija
se eu estiver errado).
Possivelmente o time de desenvolvimento do Rhapsody decidiram optar pelo kernel Mach e
pelo uso de servidores ao invés de módulos por conta da má-experiência que tiveram com o
kernel monolítico no MacOS Classic --- o que, além de carecer de fontes neste artigo, é
questionável, a partir do momento em que você têm diversos sistemas monolíticos que são
extremamente confiáveis, como o Plan 9 --- onde se uma aplicação travasse, o sistema ia
junto.     

## Por que é importante, além dos motivos históricos?
TODO

    

## Por que você quer tanto fazer isso?
Há tempos venho querendo fazer isso, aproximadamente desde os meus 13 anos, quando comecei
a me interessar mais pela computação e por sistemas operacionais --- por mais que, hoje em
dia, eu flerte mais com a eletrônica --- mas acabei nunca conseguindo fazer por falta de
conhecimento na época, tanto em inglês para pesquisar e ler documentação quanto em
computação por si só.  

Também venho querendo estudar a hierarquia padrão do sistema de arquivos (FHS) empregada
nele, afinal sempre me interessou também saber o que a Apple realmente fez em cima da
hierarquia padrão BSD que o Mach normalmente seguiria, e para entender isso bem eu teria
de pôr as mãos no sistema; e, ligando nisso, eu estou querendo aplicar o que eu aprender
aqui futuramente num projeto experimental, mas por hora é surpresa. :shushing_face:  

     