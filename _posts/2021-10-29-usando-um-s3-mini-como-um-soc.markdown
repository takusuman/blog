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

 <StYlE>
  body, pre { margin: 0 }
  #vt100 {
    float: left;
    font-size: 12pt;
    border: 0px solid rgb(0.00%, 0.00%, 0.00%);
    padding: 2px;
    background: rgb(0.00%, 0.00%, 0.00%)
  }
  .ul { text-decoration: underline }
  .bd { font-weight: bold }
  .it { font-style: italic }
  .st { text-decoration: line-through }
  .lu { text-decoration: line-through underline }
  </styLe>
 
 
  <diV id='vt100'>
   <pRE><spaN class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>G450%; git clone https://aur.archlinux.org/heimdall-git                                                                       </sPan>
<span class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>Cloning into 'heimdall-git'...                                                                                                </Span>
<span class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>remote: Enumerating objects: 24, done.                                                                                        </spaN>
<SPAN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>remote: Counting objects: 100% (24/24), done.                                                                                 </SPaN>
<Span class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>remote: Compressing objects: 100% (24/24), done.                                                                              </spaN>
<sPaN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>remote: Total 24 (delta 0), reused 24 (delta 0), pack-reused 0                                                                </sPan>
<sPaN class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>Unpacking objects: 100% (24/24), 8.95 KiB | 763.00 KiB/s, done.                                                               </SPAn>
<Span class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>G450%; cd heimdall-git                                                                                                        </span>
<spAn class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>G450%; # Instale o Heimdall no /usr/local ao invés do /usr                                                                    </SPaN>
<SPAN class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>G450%; sed -i -e 's@/usr/bin@/usr/local/bin@g' -e 's@/usr/share@/usr/local/share@g' -e 's@-DCMAKE_INSTALL_PREFIX="/usr"@-DCMAK</SPAn>
<SPAN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>E_INSTALL_PREFIX="/usr/local"@g' PKGBUILD                                                                                     </sPAn>
<sPAn class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>G450%; # Cheque se tudo foi trocado corretamente dentro do PKGBUILD                                                           </SPan>
<SPaN class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>G450%; grep '/usr/local' PKGBUILD --color=always                                                                              </SpAN>
<SpaN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>        cmake . -DCMAKE_INSTALL_PREFIX="</sPan><SPan class=' ev bd ' style='color: rgb(100.00%, 0.00%, 0.00%); background: rgb(0.00%, 0.00%, 0.00%)'>/usr/local</SPaN><SPaN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>"                                                                           </sPan>
<sPAn class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>        install -m644 -D "$srcdir/heimdall.desktop" "$pkgdir</spAn><SpAN class=' od bd ' style='color: rgb(100.00%, 0.00%, 0.00%); background: rgb(0.00%, 0.00%, 0.00%)'>/usr/local</SPan><Span class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>/share/applications/heimdall.desktop"                   </sPAn>
<SpaN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>        install -m644 -D LICENSE "$pkgdir</SpAN><spAn class=' ev bd ' style='color: rgb(100.00%, 0.00%, 0.00%); background: rgb(0.00%, 0.00%, 0.00%)'>/usr/local</SpAN><SPAn class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>/share/licenses/$pkgname/LICENSE"                                          </sPAN>
<sPaN class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>        install -d "$pkgdir"</sPan><spAN class=' od bd ' style='color: rgb(100.00%, 0.00%, 0.00%); background: rgb(0.00%, 0.00%, 0.00%)'>/usr/local</sPAN><spAn class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>/bin                                                                                    </sPaN>
<SpAn class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>        install -Dm755 bin/* "$pkgdir"</sPan><sPAn class=' ev bd ' style='color: rgb(100.00%, 0.00%, 0.00%); background: rgb(0.00%, 0.00%, 0.00%)'>/usr/local</SpaN><spaN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>/bin/                                                                         </sPan>
<sPAn class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>G450%; # Agora só compile o pacote usando o makepkg                                                                           </SpAN>
<SpAN class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>G450%; makepkg -si                                                                                                            </Span>
<SPaN class=' ev bd ' style='color: rgb(100.00%, 100.00%, 0.00%); background: rgb(0.00%, 0.00%, 0.00%)'>==&gt; WARNING:</sPaN><SPAN class=' ev bd ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'> Cannot find the sudo binary. Will use su to acquire root privileges.</sPAN><SPAN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>                                             </sPAn>
<SPAn class=' od bd ' style='color: rgb(0.00%, 100.00%, 0.00%); background: rgb(0.00%, 0.00%, 0.00%)'>==&gt;</SPan><sPan class=' od bd ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'> Making package: heimdall-git 1.4.2.r10.g3997d5c-1 (Mon Nov  1 07:26:31 2021)</spAn><sPAn class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>                                              </SpAn>
<SPAN class=' ev bd ' style='color: rgb(0.00%, 100.00%, 0.00%); background: rgb(0.00%, 0.00%, 0.00%)'>==&gt;</sPAn><SpAn class=' ev bd ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'> Checking runtime dependencies...</sPan><SpAN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>                                                                                          </Span>
<spaN class=' od bd ' style='color: rgb(0.00%, 100.00%, 0.00%); background: rgb(0.00%, 0.00%, 0.00%)'>==&gt;</spAn><SPAn class=' od bd ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'> Checking buildtime dependencies...</sPaN><Span class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>                                                                                        </sPAN>
<SPaN class=' ev bd ' style='color: rgb(0.00%, 100.00%, 0.00%); background: rgb(0.00%, 0.00%, 0.00%)'>==&gt;</SPAn><SPAN class=' ev bd ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'> Retrieving sources...</spaN><spAN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>                                                                                                     </spAN>
<sPan class=' od bd ' style='color: rgb(36.08%, 36.08%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>  -&gt;</sPAN><SPaN class=' od bd ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'> Cloning Heimdall git repo...</SpAn><SpAN class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>                                                                                             </Span>
<spAN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>Cloning into bare repository '/var/tmp/heimdall-git/Heimdall'...                                                              </sPAN>
<sPan class=' od ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>                                                                                                                              </span>
<sPAN class=' ev ' style='color: rgb(100.00%, 100.00%, 100.00%); background: rgb(0.00%, 0.00%, 0.00%)'>                                                                                                                              </spAn>
<sPAN class=' od ' style='color: rgb(0.00%, 0.00%, 0.00%); background: rgb(61.96%, 61.96%, 61.96%)'>[0] 0:bash  1:bash* 2:zsh  3:heimdall-frontend  4:vim-                                                  "G450" 07:26 01-Nov-21</sPaN>
</prE>
  </DIV>



