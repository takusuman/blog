---
layout: post
title: Recuperando uma instalação do Arch Linux
---

**Nota**: essa é a segunda vez que estou tendo que escrever isso, pois
na primeira, por algum motivo muito específico, o Fatdog não escreveu o
arquivo no meu disco; então talvez eu esteja um pouco menos animado para
escrever e explicar isso.  

Não sei você, mas comigo o meu sistema sempre quebra em algum momento, seja
sozinho, como no caso do Arch, seja por algo que eu faça de errado, como já foi
no Slackware há uns meses atrás.  
Recentemente --- na real anteontem --- minha instalação do Arch Linux
simplesmente quebrou depois de uma atualização. Eu atualizei depois que o
Firefox simplesmente tinha parado de reproduzir vídeos e áudio e a atualização
em si correu normal.  
Nessas horas apenas duas coisas vêm à cabeça: "Como eu queria ter uma máquina
decente para rodar NixOS" e "F*deu"  
Dentre essas duas, a que ecoou mais alto foi a segunda, afinal o sistema quebrou
apenas há duas semanas antes que eu pudesse finalizar meu trabalho de física
que, como estou fazendo utilizando um programa praticamente exclusivo para
Linux (e que estou sem saco de tentar fazer funcionar no Windows), eu precisaria
do meu sistema funcional o mais rápido o possível.  

A causa para esse erro, possivelmente, é porque o `pacman` não atualiza o LiLo
automaticamente, logo quando você vai bootar ele sempre redireciona para a
imagem antiga do kernel, o que fica num loop infernal.  

Como isso já tinha acontecido comigo antes, há umas semanas atrás, eu sei o que
fazer.
Para reparar, estarei utilizando um CD com Fatdog Linux 64-bits apenas, pois ele
contém os drivers para minha placa de rede wireless, que será essencial para
conseguir reparar o sistema (afinal, vou reinstalar todos os pacotes).  

Primeiramente, caso você não tenha a imagem de disco do Fatdog, vá no seu site
hospedado em https://distro.ibiblio.org/fatdog/web/ e faça o download da ISO,
isso vai requerer conhecimento mínimo em inglês para que você entenda as
instruções iniciais de download.
Após isso, apenas grave a imagem em um CD ou em uma pendrive, não estarei
falando sobre essa parte aqui pois não é o objetivo principal desse texto.  

Normalmente eu bootaria no modo sem o servidor gráfico X habilitado (na tela de
boot, ele é chamado "Fatdog64 without graphical desktop"), mas para fins de
demonstração gráfica eu subirei o X para que eu possa anexar as screenshots
neste artigo; demora um pouco mais que o normal, mas nada que seja doloroso de
esperar.  

Após subir o X no Fatdog, caso você tenha o feito, a interface que irá
subir será o jwm; não é de muita importância fala sobre isso aqui, a
interface é análoga ao Windows 2000 e só precisamos abrir o terminal,
que tem um ícone próprio na barra de tarefas.  

O terminal vai abrir já logado no usuário root (assim como toda a sessão
do Fatdog em si), então não teremos a preocupação de procurar pela senha
na documentação.  

## Montando os discos
A primeira coisa que vamos fazer de fato aqui é montar as partições
necessárias, vamos precisar apenas da partição de root do sistema (`/`)
e da partição de boot (`/boot`).   
Primeiramente, liste todas suas partições usando o `fdisk -l`:  
**Nota**: O Fatdog tem **várias** partições `/dev/ram` que acabam por
aparecer no `fdisk`, mas podemos ignorá-las para o que estamos fazendo
aqui.  

```
# fdisk -l

Disk /dev/sda: 465.8 GiB, 500107862016 bytes, 976773168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0xa2109934

Device     Boot     Start       End   Sectors   Size Id Type
/dev/sda1  *         2048    411647    409600   200M 83 Linux
/dev/sda2          411648 234786815 234375168 111.8G 83 Linux
/dev/sda3       234786816 910626815 675840000 322.3G 83 Linux
/dev/sda4       910626816 976771071  66144256  31.6G  7 HPFS/NTFS/exFAT

Disk /dev/sdc: 298.1 GiB, 320072933376 bytes, 625142448 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdf7fdf7f

Device     Boot Start       End   Sectors   Size Id Type
/dev/sdc1        2048 625139711 625137664 298.1G  7 HPFS/NTFS/exFAT
```  

Eu localizei minhas partições de boot e root, que são `/dev/sda1` e
`/dev/sda2`, respectivamente. Só irei montá-las normalmente com o 
`mount` em `/mnt`:  

```
# mount /dev/sda2 /mnt # disco root (/)
# mount /dev/sda1 /mnt/boot # disco boot ( /boot)
```  

Também precisamos espelhar os sistemas de arquivos virtuais `/sys`,
`/proc` e `/dev`, usando o `mount` com a opção rbind (`-R`).  
Isso porque sem eles montados no chroot, o LiLo não consegue ler a lista
de partições legíveis (ou não) criadas pelo kernel a tempo de execução
no `/proc` e o `pacman` não consegue operar sem o `/sys` e o `/proc`
montados.  

```
# mount -R /proc /mnt/proc # espelhando o /proc no chroot
# mount -R /sys /mnt/sys # espelhando o /sys no chroot
# mount -R /dev /mnt/dev # espelhando o /dev/ no chroot
```  

## Conectando à Internet
Para que a gente consiga reinstalar os pacotes pelo pacman, nós
iremos precisar de conexão com a internet. Se você usa cabo de rede,
pode pular todos esses passos.  

Por padrão, no Fatdog, a placa de rede está sempre "baixa", para subi-la
usaremos o comando `ip`, que ficaria assim:  

```
# ip a # listando as interfaces
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 00:23:5a:62:08:66 brd ff:ff:ff:ff:ff:ff
3: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 78:e4:00:49:b5:93 brd ff:ff:ff:ff:ff:ff
# ip link set wlan0 up
```  

Feito isso, agora só precisamos conectar o Fatdog na internet usando o
`wpa_supplicant`. É necessário que você saiba o identificador (nome
bonito para o literal nome da rede) e a senha. Estarei usando um
redirecionamento em Shell script que nos poupa de criar um arquivo só
com essas informações:

```
# wpa_supplicant -Dnl80211 -iwlan0 -c<(wpa_passphrase 'nome da sua rede' senha4lph4numeric4)
```  

Feita a conexão básica ao roteador, nós precisamos subir a daemon DHCP.  
É ela que determina o IP que sua máquina terá na rede.
Como possivelmente seu terminal estará com o log do `wpa_supplicant`
na tela, você precisará abrir um novo. Abra-o normalmente e digite
`dhcpcd`:  

```
# dhcpcd
dev: loaded udev
eth0: adding address fe80::d13:9adf:d62a:433e
DUID 00:01:00:01:29:0c:1d:cb:78:e4:00:49:b5:93
wlan0: IAID 00:49:b5:93
eth0: waiting for carrier
wlan0: rebinding lease of 192.168.100.80
wlan0: leased 192.168.100.80 for 86400 seconds
wlan0: changing route to 192.168.100.0/24
wlan0: changing default route via 192.168.100.1
forked to background, child pid 4929
```

E, por último, mas extremamente importante, copie o arquivo `/etc/resolv.conf`
para o nosso chroot. Sem esse arquivo no nosso chroot, ele estará sem um
IP válido (que, deve ser compartilhado tanto entre o Fatdog quanto entre
o chroot):  

```
# cp -v /etc/resolv.conf /mnt/etc/resolv.conf 
‘/etc/resolv.conf’ -> ‘/mnt/etc/resolv.conf’
```

Pronto, agora estamos conectados.  

## Entrando no chroot e consertando o sistema
Entre normalmente no chroot.  
É mais um capricho meu, mas eu sempre mudo o meu `$PS1` quando uso
chroot, no caso sempre coloco o nome da minha máquina (no caso, G450)
seguido de um sinal de porcentagem, um ponto-e-vírgula e um espaço, para
lembrar que estou num chroot e não no Fatdog. Isso é feito de tal forma:  

```sh
PS1='G450%; '
```  

Após configurarmos, opcionalmente, nosso `$PS1`, apenas rodamos o
seguinte comando do `pacman`:  

```
# pacman -Syyyu linux linux-firmware
```  

Ele irá reinstalar o kernel por completo; caso você, assim como eu, use
o kernel LTS, só adicione `-lts` após o nome.  

```
G450%; pacman -Syyyu linux-lts linux-firmware
:: Synchronizing package databases...
 core
 extra
 community
warning: linux-lts-5.10.75-1 is up to date -- reinstalling
warning: linux-firmware-20210919.d526e04-1 is up to date -- reinstalling
:: Starting full system upgrade...
resolving dependencies...
looking for conflicting packages...

Packages (3) yt-dlp-2021.10.22-3  linux-firmware-20210919.d526e04-1
             linux-lts-5.10.75-1

Total Download Size:     2.69 MiB
Total Installed Size:  810.94 MiB
Net Upgrade Size:       13.51 MiB

:: Proceed with installation? [Y/n] Y
```  
Dependendo da velocidade da sua rede (e do seu processador, para
descompactar os pacotes), talvez demore *um pouco*. Recomendo que
levante um pouco da cadeira e prepare um café ou sirva um copo d'água,
olhe o céu e talvez até tire uma foto.    

Depois que o kernel for (re-)instalado, apenas rode o binário do LiLo
usando o caminho completo, para que ele possa gerar novamente o seu
arquivo de configuração e carregar a imagem nova do kernel ao invés da
antiga agora inexistente:  

```
G450%; /sbin/lilo
Added arch  +  *
Added arch-fallback  +
Added windows
```  

Feito, finalmente.  
Agora podemos sair do chroot e reiniciar. Cruzem os dedos para que
boote.  

No meu caso, bootou com sucesso. Missão cumprida!
