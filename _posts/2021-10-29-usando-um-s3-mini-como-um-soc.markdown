---
layout: post
title: Experimento: usando um Samsung Galaxy S3 Mini como um SoC
---

Há umas semanas atrás, eu consegui um celular antigo da Samsung, um S3 Mini, era
de um parente meu que tinha esse aparelho pegando pó há um **bom** tempo.  
A versão do Android dele é bem antiga, ou seja, algumas aplicações essenciais
para alguém que realmente vá usar um smartphone nem sequer tem mais suporte;
então, para não deixar esse aparelho na gaveta de novo, eu tive a ideia de
tentar transformá-lo num microcomputador ARM, para usar como um "estepe de
Raspberry Pi" enquanto não compro um.  

A primeira coisa que vamos precisar fazer de fato é uma troca de ROM no dispositivo,
a fim de removermos bloat que vem instalado por padrão. Se você não é brasileiro,
possivelmente não sabe do que estou falando, então deixa que eu explico: sempre
que você compra um smartphone aqui no Brasil que seja vinculado à alguma
operadora, ele vem com **vários** aplicativos instalados que são **impossíveis**
de remover sem root ao menos --- inclusive, isso é algo que o Ciro Santilli
abordou no site dele em sua página sobre o Brasil --- e esses aplicativos são um
porre.  
Eles geralmente consomem uma boa parte do armazenamento interno e alguns ainda
rodam de fundo, o que é longe do ideal para o nosso caso, onde queremos usar o
máximo do desempenho do aparelho para processamento e afins.  
Além do mais, a própria Samsung botou alguns aplicativos que são úteis (e bons,
como no caso do Samsung E-Mail), mas que para nosso experimento não vão servir
de muita coisa.  

O modelo que tenho em mãos é um `GT-I8200L`, que é o modelo mais vanilla.   
Eu penei um pouco para encontrar os arquivos, mas os encontrei eventualmente,
que são a ROM do CyanogenMod (lembra dele?) e o TWRP.  
Não gosto de deixar links e downloads (afinal, isso não é um blog de pirataria e
downloads, odeio essa estigma), mas nesse caso creio que seria necessário por
conta da dificuldade que você teria de encontrá-los.  
A ROM do CyanogenMod pode ser encontrada nessa thread do XDA Developers:  
https://forum.xda-developers.com/t/cm-rom-for-s-iii-mini-value-edition-gt-i8200.3432885/  

E a imagem do TWRP para o `GT-I8200L` também poderia ser encontrada aqui:  
https://forum.xda-developers.com/t/twrp-for-galaxy-s3-mini-gt-i8200.3595483/  

Pelo o que eu li, o `GT-I8200L` e o `GT-I8200` são o mesmo modelo.  

Depois que eu baixei os arquivos para o meu modelo, eu fui atrás de um programa
para flasheá-los no aparelho --- eu sei que poderia possivelmente fazer isso com
o `adb` + `flashboot`, mas como eu disse antes a documentação é bem escassa e
ninguém falava propriamente sobre o uso do `adb` nessa. Lembro de que o pessoal
usava (e muito) o tal Odin, mas não vou usar ele pois, além de não ter para
Linux, é um programa que foi vazado da Samsung, é bem lento e não é muito
confiável; no lugar estarei usando o Heimdall, um programa de código-aberto e
livre (tá na MIT), criado pela Glass Echidna, que é uma empresa australiana que
produz software e presta consultoria de TI, ele tá disponível no AUR, então
instalar ele no Arch é bem straightforward.

{% include xterm.2021.11.01.07.26.31.html %}


