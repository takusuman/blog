---
layout: post
title: "Adicionando uma suíte de testes a um projeto de 20 anos de idade"
---

# Nota de 5 de Maio de 2024

***As chuvas pioraram mais do que eu esperava, pelo visto São Pedro estourou
um cano enquanto limpava a casa; o estado do Rio Grande do Sul, onde resido,
está passando por um período terrível de chuvas e, pela [falta de
infraestrutura (e até de manutenção) dada à corrupção, incompetência ou até 
mesmo inocência das autoridades
locais](https://www.youtube.com/live/GAZhfNYSySw?si=pKwvtFOX--pvtJYI&t=868),
enchentes e rompimento de estradas.
Se você reside no Brasil e pode ajudar em algo, acesse 
[ParaQuemDoar.com.br](https://www.paraquemdoar.com.br/hub/chuvasRS2024) e, por
favor, tente ajudar de alguma forma.***

# Nota de 12 de Maio de 2024

***Num geral, as coisas aparentam melhora, mas ainda há risco: [o nível do rio
Taquari subiu em seis metros](https://g1.globo.com/rs/rio-grande-do-sul/noticia/2024/05/11/rio-taquari-sobe-6-metros-em-24h-e-ultrapassa-cota-de-inundacao.ghtml)
e [seis barragens estão em alerta de ruptura em virtude de estarem operando em
sobrecarga](https://www.bbc.com/portuguese/articles/c2ley37jl7vo). Felizmente,
os órgãos responsáveis por essas instalações garantiram que ações estão sendo
realizadas para que não haja tanto desgaste na estrutura das instalações.***

# Nota de 21 de Junho de 2024

***Depois de um bom tempo fora desse artigo por várias questões que aconteceram,
resolvi voltar nessa semana a fim de terminá-lo depois de ficar sem abrir o
arquivo desde o dia 5.***

# Nota de 7 de Setembro de 2025

***Santo Deus, passou tempo... As enchentes acabaram, os incompetentes foram
reeleitos --- e alguns competentes perderam a reeleição --- e até mesmo se
"trocou" de Papa. Acho que é hora do artigo ser finalizado, já ficou
"fermentando" um bom tempo. A propósito, feliz dia da Independência.***

Data de começo de escrita: 26 de Abril de 2024

# Introdução

Uns nove dias atrás --- ou, melhor dizendo, cinco, afinal comecei a escrever
este artigo à meia-noite do dia 27 enquanto assistia ao Jornal da Globo mas,
como São Pedro andou arrastando os móveis e lavando o chão, eu tive que parar de
escrever pois, além de faltar luz graças à magnífica infraestrutura da cidade,
entrou água na minha garagem e só estou conseguindo voltar agora, dia 1º de
Maio, sendo que provavelmente irei ~~terminar perto do dia 10~~ demorar mais de
um ano por conta de diversos fatores  ---, fiz [um artigo estudando o sistema
de montagem do Heirloom](https://takusuman.github.io/blog/2024/04/22/tentando-entender-a-montagem-do-heirloom.html).
A partir daquele artigo, aprendi --- ou melhor, de novo, aprendemos --- sobre
como o sistema em si funciona e qual a forma correta de adicionar uma nova
subpasta no projeto, a fim de criar uma suíte de testes que seja totalmente
integrada.  
Em complemento, talvez para tentar despistar um certo
"[Sebastianismo](https://pt.wikipedia.org/wiki/Sebastianismo) amoroso" que
criei depois daquela segunda-feira de doer (e praticar a escrita mais um bocado),
estou escrevendo esse aqui, com maiores detalhes sobre como funcionarão os
testes e algumas escolhas que decidi fazer na última semana desde que [foram
importados _verbatim_ do Toybox
0.8.11](https://github.com/Projeto-Pindorama/heirloom-ng/pull/48/commits/84cf641e2eff26942130ef58b4dc5987e2c7a665).  
Caso você ainda não tenha lido o primeiro artigo, recomendo fortemente que
leia.

## Aqua vitæ: O que descobrimos com a "destilação"?

Certo, tentando condensar o último artigo em uma frase, nós sabemos que a
configuração principal das pastas que fazem parte do projeto é feita no arquivo
``makefile`` e com a alteração da regra ``makefiles`` para incluir uma nova
pasta sem dependência na variável ``SUBDIRS``, além de que cada pasta que for
interagir com o sistema de montagem deve ter o seu próprio arquivo
``Makefile.mk`` com regras definidas (e outros específicos de cada pasta).  
Isso é o suficiente para sabermos como implementar uma suíte de testes no
projeto da forma correta.

## "Água de beber, camará": Implementando a suíte de testes

Tentarei sintetizar aqui o que estamos fazendo no sistema de montagem antes de
falar dos _scripts_ de teste de alguns dos comandos no pacote --- até porque
outros não são possíveis de testar por meio de shell scripts por motivos que
explicarei depois --- pois é a base dos passos a serem tomados a seguir e vai
nos dar um bom indicativo de como seguir para rodar os testes em si.  

A primeira coisa que eu tentei fazer foi adicionar uma regra nova no
``makefile`` para que imprimisse uma mensagem, sem criar uma pasta nova ainda. 

```makefile
tests:
    @echo "Nde 'ara t'i porang."
```

Como esperado, ao executar o Make, temos a saída sem mostrar a chamada do
comando ``echo`` em si --- ao utilizar o ``@``, que diz que o comando deve ser
silenciado ---, que nos deseja, em Tupi-guarani, um belo dia. 

```console
S145% gmake tests 
Nde 'ara t'i porang.
```

Certo, agora seria bacana se conseguíssemos fazer isso por meio de um
``Makefile`` próprio da pasta de testes. Comecemos criando um diretório e um
arquivo ``Makefile.mk``.

```console
S145% mkdir build/test 
S145% > build/test/Makefile.mk
```

Como já sabemos, só isso não basta; nós também precisamos citar a existência da
pasta build/test no ``makefile``, para que o alvo --- vulgo arquivo gerado pela
regra --- ``build/test/Makefile.mk`` esteja na lista ligada de dependências
(``deps``) criada pelo Make.  
A coisa mais intuitiva a se fazer seria adicionar o caminho da pasta na variável
``SUBDIRS``, mas teria um pequeno problema: sempre que tentássemos compilar o
Heirloom, os testes seriam executados mesmo que contra nossa vontade, pois o
alvo padrão executa um laço _for_ que itera sobre todos as pastas em ``SUBDIRS``
e executa o Makefile de cada uma delas, como vimos anteriormente.  
O que fazer?

Bem, podemos adicionar uma nova variável chamada ``TESTSDIR`` e inserir o
caminho da pasta lá, então estaremos independentes da ``SUBDIRS``:

```diff
S145% git diff HEAD makefile 
diff --git a/makefile b/makefile
index 04a0655..9141ae2 100644
--- a/makefile
+++ b/makefile
@@ -17,6 +17,8 @@ SUBDIRS = build libwchar libcommon libuxre _install \
        tabs tail tapecntl tar tcopy tee test time touch tr true tsort tty \
        ul uname uniq units users wc what who whoami whodo xargs yes
 
+TESTSDIR = build/test
+
 dummy: makefiles all
 
 .DEFAULT:
@@ -30,7 +32,7 @@ dummy: makefiles all
 .mk:
        cat build/mk.head build/mk.config $< build/mk.tail >$@
 
-makefiles: Makefile $(SUBDIRS:=/Makefile)
+makefiles: Makefile $(SUBDIRS:=/Makefile) $(TESTSDIR:=/Makefile)
```

Agora podemos tentar escrever um ``Makefile.mk`` e desejar um belo dia de lá:

```console
S145% printf 'all: tests\n\ntests:\n\t@echo "Nde '\''ara t'\''i porang."\n' > build/test/Makefile.mk
S145% cat build/test/Makefile.mk
all: tests

tests:
        @echo "Nde 'ara t'i porang."
```

Ah, temos que mexer no ``makefile`` de novo e dizer que queremos entrar na pasta
de testes e executar o ``Makefile`` de lá na hora de executar os testes, além de
que precisamos adicionar a tal pasta na regra que executa o ``mrproper``.

```diff
@@ -42,7 +44,7 @@ install:
 
 mrproper:
        rm -f .foo .FOO Makefile
-       + for i in $(SUBDIRS) ;\
+       + for i in $(SUBDIRS) $(TESTSDIR) ;\
        do      \
                (cd "$$i" && $(MAKE) $@) || exit ; \
        done
@@ -53,6 +55,9 @@ casecheck: .foo .Foo
 .foo .Foo:
        echo $@ > $@
 
+tests:
+       (cd $(TESTSDIR) && $(MAKE)) || exit
+
```

Se tentarmos executar os testes de primeira, teremos esse erro:

```console
S145% gmake tests 
(cd build/test && gmake) || exit 
gmake[1]: Entering directory '/tmp/heirloom-07/build/test'
gmake[1]: *** No targets specified and no makefile found.  Stop.
gmake[1]: Leaving directory '/tmp/heirloom-07/build/test'
gmake: *** [makefile:59: tests] Error 2
```

Algo similar acontece com o alvo ``mrproper``:

```console
S145% gmake mrproper
rm -f .foo .FOO Makefile
for i in build libwchar libcommon libuxre _install banner basename bc bdiff bfs cal calendar cat chmod chown cksum cmp col comm copy cp cpio csplit cut date dc dd deroff diff diff3 dircmp dirname df du echo ed env expand expr factor file find fmt fmtmsg fold getconf getopt grep groups hd head hostname id join kill line listusers ln logins logname ls mail man mesg mkdir mkfifo mknod more mvdir nawk news nice nl nohup oawk od paste pathchk pg pgrep pr printenv printf priocntl ps psrinfo pwd random renice rm rmdir sdiff sed setpgrp shl sleep sort spell split stty su sum sync tabs tail tapecntl tar tcopy tee test time touch tr true tsort tty ul uname uniq units users wc what who whoami whodo xargs yes build/test ;\
do      \
        (cd "$i" && gmake mrproper) || exit ; \
done
gmake[1]: Entering directory '/tmp/heirloom-07/build'
gmake[1]: *** No rule to make target 'mrproper'.  Stop.
gmake[1]: Leaving directory '/tmp/heirloom-07/build'
gmake: *** [makefile:47: mrproper] Error 2
```

Por mais que não pareça, esse erro não é algo que devemos nos preocupar, já que
ele, claro como o Sol, indica a quem está tentando fazer tal operação que esse
alvo não existe se o resto do pacote não tiver sido compilado antes. Isso também
nos poupa de verificações extras, pois os testes só poderiam ocorrer depois do
Heirloom estar compilado e instalado em alguma localização --- algo que
precisaremos nos incomodar depois --- de qualquer forma.  
Tendo isso em mente, rodaremos o Make até o ponto em que ele gera os
``Makefile``s, só para podermos testar se o ``Makefile`` principal dos testes
funciona corretamente.

```console
S145% gmake                                        
cat build/mk.head build/mk.config Makefile.mk build/mk.tail >Makefile
cat build/mk.head build/mk.config build/Makefile.mk build/mk.tail >build/Makefile
# [...]
cat build/mk.head build/mk.config build/test/Makefile.mk build/mk.tail >build/test/Makefile
# [...]
gmake[1]: Leaving directory '/tmp/heirloom-07/build'
^Cgmake: *** [makefile:25: all] Interrupt
```

E então podemos rodar a regra ``tests`` que, como vimos, entra na pasta de
testes e executa o Makefile.

```console
S145% gmake tests 
(cd build/test && gmake) || exit 
gmake[1]: Entering directory '/tmp/heirloom-07/build/test'
Nde 'ara t'i porang.
gmake[1]: Leaving directory '/tmp/heirloom-07/build/test'
```

Agora é interessante que limpemos o sistema de montagem com a regra ``mrproper``
de novo e...

```console
S145% gmake mrproper
# [...] 
gmake[1]: Entering directory '/tmp/heirloom-07/build/test'
gmake[1]: *** No rule to make target 'clean', needed by 'mrproper'.  Stop.
gmake[1]: Leaving directory '/tmp/heirloom-07/build/test'
gmake: *** [makefile:47: mrproper] Error 2
```

Epa, esquecemos de declarar uma regra ``clean``, que é uma das dependências do
mrproper, como podemos ver em ``build/mk.tail``.  

**Íntegra do [arquivo ``mk.tail``](https://github.com/Projeto-Pindorama/heirloom-ng/blob/master/build/mk.tail):**

```makefile

mrproper: clean
	rm -f Makefile $(MRPROPER)
```

Então basta apenas que adicionemos uma declaração em
``build/test/Makefile.mk`` com a regra fazendo o trabalho de remover os arquivos
que iremos gerar.

```
S145% printf '\nclean:\n\trm -f passed *~\n' >> build/test/Makefile.mk 
S145% cat build/test/Makefile.mk                                                                                                                                      
all: tests

tests:
        @echo "Nde 'ara t'i porang."

clean:
        rm -f passed *~

```

O arquivo "passed" ainda não existe e nem é criado pela regra ``tests``,
mas também não está sendo referenciado ali só para inglês ver, pois é onde
serão salvos os resultados de cada teste. 
Ao contrário dos outros arquivos ``Makefile.mk``, nós não iremos colocar os
arquivos ``core`` e nem ``log`` na lista de arquivos a serem apagados, pois a
aparição deles só faz sentido quando lidamos com um programa em C, não com
uma bateria de shell scripts e Makefiles.
O arquivo "core" é referenciado em todos os arquivos ``Makefile.mk`` do
projeto, é criado pelo núcleo em caso de um despejo de memória (em inglês,
"_core dump_") e se localiza na pasta atual de execução do programa. No Linux,
podemos ver evidência disso no arquivo ``core_pattern``, que fica na pasta
``/proc/sys/kernel``:

```console
S145% cat /proc/sys/kernel/core_pattern
core
```

Sobre o arquivo ``log``, eu não encontrei nenhuma informação sobre. Como esse
artigo é, de certa forma, apenas um rascunho, talvez essa parte seja
atualizada.  
O caractere coringa acompanhado de um til, "\*~", é mencionado para que
qualquer arquivo temporário criado pelo Vim ou pelo Emacs seja apagado.

Agora, após gerar novamente os ``Makefile``s para que as mudanças se apliquem,
vejamos se está tudo funcionando.

```console
S145% gmake tests
(cd build/test && gmake) || exit 
gmake[1]: Entering directory '/tmp/heirloom-07/build/test'
Nde 'ara t'i porang.
gmake[1]: Leaving directory '/tmp/heirloom-07/build/test'
S145% gmake mrproper -C build/test
gmake: Entering directory '/tmp/heirloom-07/build/test'
rm -f passed *~
rm -f Makefile 
gmake: Leaving directory '/tmp/heirloom-07/build/test'
```

Ótimo, conseguimos chegar ao começo do caminho. Já é algo.  

## Nos ombros dos gigantes... ou numa pilha de anões: como implementar testes de forma correta?

Agora algo que, parafraseando novamente Alighieri, rodeia a cabeça dia e noite
é pensar numa forma sã --- e sana, mentalmente falando quanto para quem for
ler tanto para quem for implementar para novos programas --- de implementar
esses testes.  
Pela razão de diminuir o tempo que levaria-se para escrever cada teste,
importamos os testes da última versão estável do Toybox até esse presente
momento, a 0.8.11, pois lá já existem casos a serem testados e precisaremos
fazer poucas alterações nesse sentido. O problema é que esses testes utilizam
três funções, uma chamada ``testing``, outra chamada ``testcmd`` e a ``txpect``,
e elas são muito complexas para sequer pensar em começar a simplificar, além do
estilo de código diferir tanto do nosso que precisaríamos reescrever só para se
encaixar no Heirloom, logo, é preferível que reescrevamos as funções de teste do
zero a fim de se ter código novo e mais simples de manter.   
O primeiro pensamento é tentar buscar uma outra referência no mundo da Sun
Microsystems ou da AT&T, pois casaria melhor com um projeto como o Heirloom, mas
até onde pesquisei, nenhuma dessas empresas fez testes na época em que estavam
trabalhando diretamente na _userspace_ do SunOS e do UNIX, respectivamente,
dependendo num geral de um acordo entre cavaleiros de reportar problemas caso
ocorressem e de pacotes de correção. Padrão.  
Quando faltam referências, o que resta fazer é sintetizar o que se sabe sobre o
que é desejado com alguma referência moderna.  
Muito tempo antes de começar a implementar testes para o Heirloom, eu já tinha
visto os testes do [pigz](https://zlib.net/pigz), que totalmente eram escritos
em Makefile --- o que nos pouparia dor de cabeça com shell, pois não teríamos
de especificar um novo, como o Korn Shell 93, ou escrever algo cheio de
código-porcelana para compatibilidade e lento para rodar na pior
implementação possível ---, e eles são assim:

**Linhas 80 até 97, [arquivo ``Makefile``](https://github.com/madler/pigz/blob/master/Makefile):**

```makefile
test: pigz
	./pigz -kf pigz.c ; ./pigz -t pigz.c.gz
	./pigz -kfb 32 pigz.c ; ./pigz -t pigz.c.gz
	./pigz -kfp 1 pigz.c ; ./pigz -t pigz.c.gz
	./pigz -kfz pigz.c ; ./pigz -t pigz.c.zz
	./pigz -kfK pigz.c ; ./pigz -t pigz.c.zip
	printf "" | ./pigz -cdf | wc -c | test `cat` -eq 0
	printf "x" | ./pigz -cdf | wc -c | test `cat` -eq 1
	printf "xy" | ./pigz -cdf | wc -c | test `cat` -eq 2
	printf "xyz" | ./pigz -cdf | wc -c | test `cat` -eq 3
	(printf "w" | gzip ; printf "x") | ./pigz -cdf | wc -c | test `cat` -eq 2
	(printf "w" | gzip ; printf "xy") | ./pigz -cdf | wc -c | test `cat` -eq 3
	(printf "w" | gzip ; printf "xyz") | ./pigz -cdf | wc -c | test `cat` -eq 4
	-@if test "`which compress | grep /`" != ""; then \
	  echo 'compress -f < pigz.c | ./unpigz | cmp - pigz.c' ;\
	  compress -f < pigz.c | ./unpigz | cmp - pigz.c ;\
	fi
	@rm -f pigz.c.gz pigz.c.zz pigz.c.zip
```

Entretanto, seria impraticável escrever testes mais complexos do que isso para
alguns programas, até porque não estaríamos apenas comparando o comprimento em
caracteres da saída do programa, mas também o valor devolvido. Sem falar que,
considerando que os testes foram originalmente importados do Toybox para que
fossem adaptados, isso também significaria não apenas uma pequena modificação,
mas também uma reescrita completa para algo que, mesmo similar, é outra
linguagem de programação com outras funções e limitações.  
O Leonardo ([``LeonardoCBoar``](https://github.com/LeonardoCBoar)), um amigo
meu que estuda na UFABC e programa em C++, me recomendou uma batida de olho na
introdução do [GoogleTest](https://google.github.io), que é, resumindo, um
_framework_ para se escrever testes para programas em C++.
Isso é uma enorme simplificação do que ele de fato faz, mas meu interesse é
saber como os testes são escritos e o que pode-se importar de lá. Além de que,
ao contrário do que realmente precisamos fazer, que é testar cada programa já
compilado, o GoogleTest funciona como uma biblioteca que é chamada por um
programa independente em C++ (escrito por quem vai fazer o teste) para testar
funções do projeto em si. Escrevendo de forma mais técnica, o GoogleTest é um
_framework_ para testes estruturais, vulgo testes de caixa-branca ("_white box
tests_", em inglês), onde, como o nome diz, se conhece a estrutura do programa
--- ou, no caso de um projeto como o LLVM, dos programas e das bibliotecas do
projeto --- e utiliza-se dessa vantagem para testar o código-fonte desde sua
fundação; enquanto isso, precisamos fazer testes funcionais, vulgo testes de
caixa-preta ("_black box tests_", em inglês), onde, por mais que conheçamos a
estrutura de cada programa do pacote por termos acesso a seu código-fonte
integral, iremos testar a funcionalidade do código-fonte já compilado,
desconhecendo --- ou, melhor, ignorando --- sua estrutura interna, pois assim
acaba por ser mais eficiente para testar se algum programa tem determinado
problema em determinada plataforma, se os valores quebram em determinadas
condições e afins.  
Por mais que pareça loucura (ou idiotice), o objetivo é saber qual os métodos
usados para se escrever os testes em si --- ou seja, como se compara o valor
esperado com o valor real que será devolvido e afins ---, assim criando (ou
melhorando) uma maneira para se escrever os nossos testes, no estilo
sborniense (ou sbørniano? Talvez sbørnio?) de síntese.  
Na [página de introdução do
GoogleTest](https://google.github.io/googletest/primer.html), é apresentado,
além do funcionamento, conceitos interessantes que devem ser anotados para
quando começarmos a implementar a suíte de testes para o Heirloom:

* Organizar os testes assim como o código e, caso haja código que seja repetido
  entre os testes, tentar manter isso em uma subrotina compartilhada;
* Portabilidade entre sistemas --- falaremos disso adiante;
* Liberdade a quem for escrever os testes para que consiga focar no conteúdo dos
  testes em si e não em sua execução. Caso contrário, não estaríamos fazendo uma
  suíte com funções para tal, mas sim testes como os do pigz, onde quem vai
  escrever precisa se preocupar com o método para verificar e não apenas com o
  conteúdo a ser verificado, algo que um projeto grande, como o Heirloom, não
  pode se dar o direito, pois criaria testes enormes, difíceis de ler e
  ineficientes;
* Prover a maior quantidade de informação que tivermos acesso sobre o porquê do
  erro ter ocorrido, pois assim é mais fácil de se identificar o erro e, caso
  venha do nosso lado, corrigi-lo;

Após isso, indo para os conceitos básicos sobre testes, vemos que o GoogleTest
trabalha com asseverações, ou seja, afirmações que são verdadeiras ou falsas
dependendo dos resultados em comparação com o que é esperado. Clássica, os
testes do Toybox já faziam isso. Entretanto, temos a diferença de que não é oito
ou oitenta, mas sim sucesso, falha não-fatal e falha fatal. Essa divisão entre
falha não-fatal e fatal é importante em casos onde, por exemplo, não faria
sentido prosseguir --- por exemplo, se o tamanho da _string_ de saída for
diferente do tamanho da _string_ esperada, podemos parar de imediato o teste
atual e prosseguir para o próximo.  
Essa diferenciação é feita primariamente entre dois "polos": os macros
da família ``EXPECT`` e da família ``ASSERT``, como podemos ver aqui:

> The assertions come in pairs that test the same thing but have different
> effects on the current function. ``ASSERT_*`` versions generate fatal
> failures when they fail, and abort the current function. ``EXPECT_*``
> versions generate nonfatal failures, which don’t abort the current
> function. Usually ``EXPECT_*`` are preferred, as they allow more than one
> failure to be reported in a test. However, you should use ``ASSERT_*`` if
> it doesn’t make sense to continue when the assertion in question fails.

Em suma, ``ASSERT`` funciona como uma afirmação --- "e tenho dito!" --- logo,
ela precisa necessariamente ser verdade, caso contrário, algo realmente está
errado e não pode-se continuar, já ``EXPECT`` é algo que supomos --- "ser ou
não ser" ---, então um erro já não seria fatal, mas sim algo digno de um aviso
para que façamos alguma observação no código para determinar o que causou aquela
diferença.  
Qualquer um que já tenha programado em Go, por exemplo, saberia a diferença,
pois lá há funções que tratam o erro e a função ``panic()``, que para o
programa em caso de um erro grave.  
Quando uma asseveração for falsa, ou seja, diferente do resultado que esperamos,
devemos informar o máximo que soubermos sobre o porquê falhou, como foi dito lá
em cima na nossa lista de objetivos. Temos a limitação de estarmos fazendo
testes de caixa-preta, mas podemos trabalhar bem em cima disso mesmo assim.
Organizando as nossas referências, nós precisamos fazer uma função para
específicar o tipo de teste que será feito, uma função que retorne um erro fatal
e outra que retorne erros não-fatais. Inicialmente, considerei em fazer uma
função inteira que testasse o comando, mas acredito que tenhamos mais liberdade
usando duas funções separadas, como é no GoogleTest. Como em shell, desde sua
concepção, variáveis do tipo _strings_ são "cidadãs de primeira-classe", como
disse Stephen Bourne, e, como eu mesmo costumo dizer, até que se prove o 
contrário, as únicas, nós não iremos ir muito fundo além de comparar valores
enquanto _strings_ --- como, por exemplo, criar uma função específica para
comparar números com vírgula. É necessário criar esse planejamento desde já
pois, na hora de programar, dificilmente adicionaremos algo novo ao que já
temos no papel.  
Algo que descobri enquanto esse artigo "fermentava" é que o ``go test`` funciona
de forma praticamente idêntica, então eu poderia ter utilizado-o de referência
também.

## "_[...] sunt nate res omnes ab hac re una aptatione_": escrevendo novo código e adaptando testes

Programei --- e ainda programo --- em shell por anos, mas andei a fuçar com C e
Go e isso acabou por "corromper" minha prática um bocado, pois passei a buscar
fazer coisas de forma mais manual ao invés de utilizar o que era já seria
provido --- o que é desincentivado em scripts de shell pois acaba por atrapalhar
a portabilidade, afinal, quanto mais controle você tem sobre as operações que o
interpretador realiza, mais complexa a implementação do interpretador precisaria
ser (ao menos em teoria).  
Entretanto, isso se mostra um tanto imprático em shells mais simples do que o
Korn Shell, pois não se tem extensões que permitem que a linguagem se comporte
de forma mais próxima de C, como um macro para ler um caractere único de uma
_string_ por vez.  
A fim de não adicionar mais uma dependência no projeto, decidi que não usarei
nem o Korn Shell 93 e nem o GNU Bash para escrever as funções que serão
chamadas pelos _scripts_ de testes, então vejamos o que o README pede --- ou
ao menos propõe --- que esteja no sistema para ser usado como shell:

> The following prerequisites are optional:
> 
> Bourne shell   It is recommended that the Heirloom Bourne shell from
>                <http://heirloom.sourceforge.net/sh.html> is installed
>                first. The tools will work with any shell, but use of the
>                Bourne shell ensures maximum compatibility with traditional
>                Unix behavior.

Certo, pelo visto usar o [Bourne shell do
Heirloom](https://heirloom.sf.net/sh.html), derivado da implementação 
SVR4 que é obviamente não-POSIX --- lembrem-se disso depois ---, é algo
meramente opicional, certo? Pois bem:

**Linhas 1 até 16, [arquivo ``build/mk.config``](https://github.com/Projeto-Pindorama/heirloom-ng/blob/master/build/mk.config):**

```makefile
#
# This is the shell used for the compilation phase, the execution of most
# installed scripts, and the shell escapes in the traditional command
# versions. It needs not conform to POSIX. The system shell should work
# fine; for maximum compatibility with traditional tools, the Heirloom
# Bourne shell is recommended. It then must obviously be compiled and
# installed first.
#
SHELL = /bin/sh

#
# Specify the path name for a POSIX-conforming shell here. For example,
# Solaris users should insert /usr/xpg4/bin/sh. This shell is used for
# the shell escape in the POSIX variants of the ed utility.
#
POSIX_SHELL = /bin/sh
```

Sim, usar o Bourne shell tradicional é opicional, __mas__ as instruções são
taxativas em dizer que praticamente toda a fase de montagem utilizará o shell
tradicional e não o POSIX. Ou seja, por uma simples formalidade nós --- e por
nós, leia-se eu --- seremos obrigados a escrever os scripts num "_scriptum
francus_" que funcione tanto no Bourne shell quanto no POSIX, logo, não podemos
usar extensões específicas de nenhum dos dois e, por conta disso, iremos
provavelmente depender em vários programas independentes ao invés de usar
implementações embutidas, ao menos para algumas coisas.

### ["Por que São Jorge mora na Lua?"](https://youtu.be/-gBrzeSOXCg): Alguns porquês que encontramos no Bourne shell (e seus "clones")

Comecei por praticar mexer com o Bourne shell tradicional criando algo simples,
um programa que pudesse contar números de 1 até o número que eu passasse.  
É intuitivo, como não existem laços _for_ "chiques" ao estilo-C, usar um
laço condicional _while_ que lide com um valor booleano --- que em shell são
dois binários que retornam um valor de estado de saída, mas isso não deve nos
preocupar muito por ora --- e que pare quando chegar no valor desejado.  
Óbvio que poderíamos simplesmente usar a condicional "``$n -eq $1``", que,
caso você nunca tenha mexido com shell, seria como iterar até que o valor de
_``n``_ seja o mesmo do primeiro argumento passado, _``$1``_, de primeira; no
entanto,  fazer dessa forma nos permite, em tese, que possamos efetuar
movimentos mais ousados mais à diante como, por exemplo, implementar um
contador de caracteres de uma _string_ sem depender de nenhum comando externo.  

```sh
p=false
n=0
while ! $p; do
    n=$(( $n + 1 ))
    printf '%d\n' $n
    [ $n -eq $1 ] && p=true
done
```

Não é a Oitava maravilha, mas parece bom. Vamos testar.

```console
S145% ./sh teste2.sh 10
teste2.sh: syntax error at line 4: `n=$' unexpected
```

De primeira, já temos uma surpresa e algo que eu francamente havia me esquecido
depois de passar bons anos longe do "shell Bourne tradicional" é o fato de que
ele não suporta operações aritméticas nativamente, mas sim depende do comando
``expr``.  
Isso me fez lembrar de uma [palestra feita pelo próprio Stephen Bourne na BSDcan
de 2015, que está disponível no
YouTube](https://www.youtube.com/watch?v=2kEJoWfobpA), onde, mesmo com toda a
evolução que houve desde a primeira versão até a versão do UNIX v7, que é de
onde o shell do Heirloom é derivado, é mostrado que o shell foi originalmente
pensado para prover "apenas" estruturas elementares de controle de fluxo, como
laços e condições, variáveis e substituições das mesmas e gerenciamento de
sinais e processos, e que qualquer outra coisa além disso deveria ser
provida por algum meio externo, seja um módulo, programa, etc. Logo, por essa
ideia, que é o âmago da filosofia UNIX que vigorava na época, não haveria
o porquê de implementar algo no shell que já seria implementado pelo comando
[``expr``(1)](https://heirloom-ng.pindorama.net.br/manual/man1/expr.1.html),
além de que, pelo fato do shell ter apenas _strings_ como tipo (como falei
anteriormente), não se esperaria que se usasse de fato funções aritméticas a
todo momento, apenas filtros de texto como ``awk`` ou ``grep``; então,
finalizando, não haveria o porquê de "martelar" tais funções no binário do
programa, já que seria mais alguma coisa para carregar na memória em uma época
em que recursos de hardware eram realmente escassos.

Enfim, que se resolva o problema...

```diff
4c4
<       n=$(( $n + 1 ))
---
>       n=`expr $n + 1`
```

E que tentemos outra vez:

```console
S145% ./sh teste2.sh 10 
teste2.sh: !: not found
```

Ótimo, parece que o ``!``, que significaria "negar" o status de saída de um
comando, também não está implementado. Estranho, seria algo que veríamos em C
tranquilamente e, por mais que pudéssemos encontrar outro caminho para resolver
isso --- e que seria mais lógico, até ---, não faria sentido deixar de fora.  
Pois bem, segundo o apêndice A do livro "_Learning the Korn Shell 1st. Edition_",
lançado pela editora O'Relly em Junho de 1993, que fala sobre a norma POSIX
1003.2 para shell, diz que __sim, ``!`` é parte do padrão POSIX desde pelo
menos 1992__ --- que foi o ano em que esse padrão foi lançado, depois de seis
anos de discussões acaloradas entre programadores e empresas que forneciam
software para UNIX ---, __mas que isso não significa que o Bourne shell
tenha adotado-o__, até porque, segundo o arquivo ``cmd.c``, que define as
palavras-chave usadas no shell, a última vez que o dito recebeu alguma
alteração por mão da AT&T foi em __1989__, ou seja, coisa de __três anos
antes__ do padrão ser lançado; depois disso, apenas alguns reparos foram
feitos por mão da Sun Microsystems por cerca de 1996 e do Gunnar Ritter --- e
isso já na era do Heirloom por volta de 2005.  
Por mais que para mim e outros programadores de shell seja natural desferir
diversos xingamentos contra a equipe da AT&T ou até mesmo estendê-los para o
comitê do POSIX --- _"por que diabos vocês não implementaram arrays?"_, ou,
caso você esteja do outro lado da força, _"por que tanto bloat se o
``expr``(1) já implementa isso?"_ --- devemos entender, ou simplesmente ler o
bendito apêndice do livro, que um padrão como o POSIX surge em uma época
caótica no mundo da computação --- e mais especificamente no mundo de
aplicação corporativa dessa ---, onde, mesmo com uma linguagem tecnicamente
portável (linguagem C) e com progresso constante no campo do hardware, havia
empresas de software disputando contratos trilhardários a tapa e implementando
funções proprietárias para tentar se destacar no mercado --- ou seja, se um
estagiário implementasse um comando que parecesse interessante, pararia na
versão de produção do sistema, mesmo que ainda com várias falhas a serem
descobertas ---, logo o padrão POSIX tinha a importante missão de estabelecer
o mínimo para que esses sistemas continuassem compatíveis entre si, por isso há
tanto o dito "_bloat_" (contra o qual o pessoal do site ``cat-v.org`` tanto se
revoltou), quanto a abertura de espaço para que cada um implementasse as
extensões desejadas, de forma que ainda houvesse alguma competitividade ---
inclusive, por isso o Korn Shell 93 e, por tabela, o GNU ~~Broken~~Bourne-Again
Shell, tem funções adicionais que vão desde _arrays_ e laços _for_ estilo-C até
pomposos "pseudo-ponteiros" (os ditos ``nameref``) e _namespaces_.  
Lógico que críticas são cabíveis, principalmente ao que se chama de
"_feature creep_", que seria a sobrecarga de novas utilidades e de novas
funções em programas já existentes e que muitas das vezes acabam por ser
redundantes --- algo que eu estranhei no padrão POSIX.1-2024 --- mas que se
reconheça, no caso anterior, o porquê dessas diferenças terem se desenvolvido,
e que também se busque formas de mitigar os problemas sintetizando as soluções
com base na filosofia UNIX e as necessidades de um usuário (e de um programador)
médio.  
Certo, tendo tudo isso em mente, além de uma lista do que __pode__ não estar
presente no Bourne shell, podemos encontrar uma solução para o problema. Então,
que voltemos ao código.  
Bem, nós poderíamos inverter a condição, então ficaria assim:

```sh
p=true
n=0
while $p; do
    n=`expr $n + 1`
    printf '%d\n' $n
	[ $n -eq $1 ] && p=false
done
```

Por mais que isso pareça melhor, afinal seria a solução lógica --- se o número
``n`` se tornou igual ao que queríamos (``$1``), então não precisamos mais somar
nada (``p=false``) ---, lembremos que estamos apenas "brincando" com o shell, ou
seja, isso tudo é um exemplo.
Pensemos que talvez tenhamos de fazer um laço assim utilizando um programa, logo
seria bom ter alguma alternativa mais compacta do que algo como
``("$p" || true)`` --- que, diga-se de passagem, não funciona na prática ---, ou
seja, precisaremos implementar uma função para isso. Pois bem, que tal uma
função com o nome de ``!``? Assim poderíamos ter algo que lembrasse a
palavra-chave original e não precisaríamos mexer no código, apenas fazer uma
condicional para que caso ``!`` não fosse definido usasse nossa função. Certo,
podemos começar a prototipar na forma de um _one-liner_:

```console
$ !() { (exec "$@"); if [ $? -eq 0 ]; then return 1; else return 0; fi; }
!: is not an identifier
$ type !
! not found
``` 

Bem, por essa eu esperaria em um shell que tivesse ``!`` como palavra-chave
reservada, como o Dash ou o Korn Shell:

```console
S145% echo ${.sh.version}
Version AJM 93u+m/1.1.0-alpha+f2bc1f45 2024-03-24
S145% function ! { (exec "$@"); if [ $? -eq 0 ]; then return 1; else return 0; fi; }
/usr/bin/ksh: syntax error: `!' unexpected
S145% !() { (exec "$@"); if [ $? -eq 0 ]; then return 1; else return 0; fi; }
/usr/bin/ksh: syntax error: `(' unexpected
S145% dash
$ !(){ (exec "$@"); if [ $? -eq 0 ]; then return 1; else return 0; fi; }
dash: 10: Syntax error: ")" unexpected
```

Agora o GNU Bash... É, é complicado:

```console
luiz@S145:/run/media/luiz/Novo Volume/heirloom-sh-050706$ echo ${BASH_VERSION}
5.2.26(1)-release
luiz@S145:/run/media/luiz/Novo Volume/heirloom-sh-050706$ !() { (exec "$@"); if [ $? -eq 0 ]; then return 1; else return 0; fi; }
bash: !: event not found
luiz@S145:/run/media/luiz/Novo Volume/heirloom-sh-050706$ function ! { (exec "$@"); if [ $? -eq 0 ]; then return 1; else return 0; fi; }
luiz@S145:/run/media/luiz/Novo Volume/heirloom-sh-050706$ type !
! is a shell keyword
luiz@S145:/run/media/luiz/Novo Volume/heirloom-sh-050706$ type -a !
! is a shell keyword
! is a function
function ! () 
{ 
    ( exec "$@" );
    if [ $? -eq 0 ]; then
        return 1;
    else
        return 0;
    fi
}
```

Bem, o Bash não tem o apelido de "Broken-Again" à toa...  
Confesso que geralmente pego pesado com o Bash, mas esse tipo de comportamento
deveria ser considerado simplesmente inaceitável --- e muito menos deveria ser
contornado com a desculpa de ["a palavra-chave ``function`` é
obsoleta"](https://www.reddit.com/r/bash/comments/sn4wrf/comment/hw0z1b5/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)
dita por alguns "lealistas do Bash". Não, isso não causa nenhum tipo de
vulnerabilidade --- até porque o Bash prioriza suas palavras-chaves antes de
qualquer função (graças a Deus!) --- e, apesar do comando ``type`` nos mostrar
isso ao imprimir o nome de palavras-chave antes da função --- provavelmente
apenas seguindo a ordem da pilha que armazena as palavras-chave e depois
funções definidas pelo usuário ---, decidi "provar"  pela experimentação e
fiz essa pequena brincadeira, dessa vez uma função que imprime uma mensagem e
então mata o processo do shell com um belo ``kill -9``:

```console
luiz@S145:~$ function ! { echo 'Errou!'; kill -9 $$; }
luiz@S145:~$ if ! false; then
> echo 'Glu glu ié ié!'
> fi
Glu glu ié ié!
luiz@S145:~$ type -a !
! is a shell keyword
! is a function
function ! () 
{ 
    echo 'Errou!';
    kill -9 $$
}
```

Digamos que nisso acertaram, [_they ain't doing too
bad_](https://youtu.be/6Ds9Qs6R4YM).

Mas, como diria qualquer professor, do "prézinho" até o Ensino Médio,
"é só elogiar que estraga": esse tipo de brincadeirinha até __pode
acontecer__ caso a nossa função engodo seja declarada com identificador
vulgar, __desde que__ ``!`` __seja associado (``alias``) a tal__ pois, como
podemos ver abaixo, __esse tipo de associação tem prioridade maior que a
própria palavra-chave da linguagem__:

```console
luiz@S145:~/projetos/heirloom-testing$ type -a !
! is a shell keyword
luiz@S145:~/projetos/heirloom-testing$ engodo() { echo 'Errou!'; kill -9 $$; }
luiz@S145:~/projetos/heirloom-testing$ alias !=engodo
luiz@S145:~/projetos/heirloom-testing$ type -a !
! is aliased to `engodo'
! is a shell keyword
luiz@S145:~/projetos/heirloom-testing$ if ! false; then
> echo 'Glu glu ié ié!'
> fi
Errou!
Killed
S145% # De volta ao ksh...
```

Antes que você diga que o alias não tem efeito em modo não-interativo, saiba que
é possível "envenenar" o ``.bashrc`` com esse código:

```console
S145% printf 'engodo() { echo "Errou!"; kill -9 $$; }\nalias !=engodo\n'>~/.bashrc
S145% cat ~/.bashrc                                                                                                  
engodo() { echo "Errou!"; kill -9 $$; }
alias !=engodo
S145% bash
bash-5.2$ type !
! is aliased to `engodo'
```

... Ou seja, ainda é problema.

À primeira vista, pensei que fosse algo do Bourne shell e que eles replicaram
por uma mera questão de compatibilidade, até que testei no Korn Shell ---
afinal, já estava ali mesmo...

```console
S145% type -a !
! is a keyword
S145% engodo() { print 'Errou!'; kill -9 $$; }
S145% alias !=engodo 
```

Bem, não pestanejou com o identificador... Como será que isso está na pilha de
memória?

```console
S145% type -a !
! is a keyword
! is an alias for engodo
```

Promissor, mas testemos mais um bocado para ter a certeza:

```console
S145% if ! false; then
> echo 'Glu glu ié ié!'
> fi
Glu glu ié ié!
```

Sem baixas? Beleza pura! Pelo menos temos a certeza de que o Korn Shell é imune
a esse tipo de ataque --- apesar de não rejeitar o identificador, como faz no
caso de uma função. Mas, como sempre tem um que vai dizer que o Korn Shell não é
verdadeiramente POSIX ou que não está com o ``set -o posix`` "ativado", vamos
fazer o exato mesmo teste no Almquist shell --- ou, mais especificamente, a sua
bifurcação, o ``dash``, mas que é efetivamente a mesma coisa. Vamos usar o mesmo
teste de antes:

```console
S145% dash
$ type !
! is a shell keyword
$ engodo() { echo 'Errou!'; kill -9 $$; }
$ alias !=engodo
```

Certo, até agora temos o exato mesmo comportamento das outras vezes, mas vejam o
que acontece quando eu verifico o que o shell entende por "``!``" de novo
utilizando o comando ``type``:

```console
$ type !
! is a shell keyword
```

Ou seja, ele __nem ao menos considerou o alias__, tanto é que, se tentarmos
executar o ``!`` por si só, teremos a mesma resposta que teríamos antes do alias:

```console
$ !
dash: 5: Syntax error: newline unexpected
```

Antes que pergunte: sim, isso é absolutamente esperado, afinal ele espera pelo
menos um comando para ser executado a fim de obter seu estado de saída,
diferente do Bash e do Korn Shell que eu francamente não sei como é
implementado --- talvez eles apenas ignorem caso não seja passado um comando a
ser executado.

Poderíamos parar por aqui, mas vamos terminar só por desencargo de consciência:

```console
$ if ! false; then
> echo 'Glu glu ié ié!'
> fi
Glu glu ié ié!
```

Touché! Isso não é "do POSIX", é verdadeiramente uma falha do Bash.  
Apesar dos trocadilhos que fiz durante esse artigo até agora, não vou --- e nem
posso --- julgar a equipe do GNU por tal falha, pois não conheço a implementação
por dentro, logo possivelmente eles até já conhecem essa falha, mas não
consertaram-na pois precisaria mexer consideravelmente no código-fonte. Caso não
a conheçam ainda, também não é uma tentativa de tripudiar e nem ao menos criar
pânico para uma transição em direção ao Korn Shell --- apesar de scripts
escritos para o Bash poderem rodar com nenhuma (ou pouquíssima) modificação no
Korn Shell ---, mas sim apenas de reportar; afinal, concorrência foi, é e sempre
será ótima.

Enfim, voltando ao Bourne shell, ``!`` não seria um bom identificador de
qualquer forma... Que tal ``¬``?

```console
$ ¬() { (exec "$@"); if [ $? -eq 0 ]; then return 1; else return 0; fi; }
\302\254: is not an identifier
```

Ué, também não pode? Engraçado o fato de que o caractere foi impresso como
seu código octal, com ``\302`` sendo o início da representação e ``\254`` o
valor do caractere ``¬`` em si, de forma separada tal como se apenas suportasse
caracteres ASCII primitivos --- podemos ser mais específicos e lembrar que
emuladores de terminal modernos como o Konsole emitem caracteres com
codificação UTF-8 para o comando (nesse caso, o shell), sem falar em editores
de texto como o Vim (que são configurados em sua maioria para escrever arquivos
também codificados em UTF-8), o que explicaria o porquê de "¬" não ter sido
inserido em forma ASCII mas sim como uma sequência UTF-8, o que podemos ver
pelo fato de que começa com ``\302``.  
Aproveitando que estamos na árvore de código-fonte, vamos ver o que determina
o que é um identificador válido ou não.

```console
$ pwd
/run/media/luiz/Novo Volume/heirloom-sh-050706
$ ls -c *.c *.h
args.c     echo.c      io.c         name.h     test.c
blok.c     error.c     jobs.c       print.c    timeout.h
bltin.c    expand.c    mac.h        pwd.c      ulimit.c
brkincr.h  fault.c     macro.c      service.c  umask.c
cmd.c      func.c      main.c       setbrk.c   version.c
ctype.c    getopt.c    mapmalloc.c  stak.c     word.c
ctype.h    gmatch.c    mbtowi.h     stak.h     xec.c
defs.c     hash.c      mode.h       string.c
defs.h     hash.h      msg.c        strsig.c
dup.h      hashserv.c  name.c       sym.h
$ find . -type f -name '*.c' -print | xargs grep 'is not an identifier'
./msg.c: const char     notid[]         = "is not an identifier";
```

Agora algo bem bacana: programas antigos em C e Assembly já utilizavam algo
que viria a ser parte do princípio D.R.Y. de hoje em dia. Mesmo já conhecendo
o que é o D.R.Y., eu desconhecia o fato dessa prática ser parte do princípio
e, pensando ter um nome à parte --- até por ser uma prática comum em Java ---,
fui consultar à comunidade [He4rt Developers](https://heartdevs.com) e recebi
uma resposta do André Luís ([``andreluispy``](https://github.com/andreluispy))
sobre e, bem, antes do conceito de D.R.Y. existir como algo unificado, essa
prática já era feita em código Assembly para diminuir a quantidade de coisa
que precisaria ser digitada e repetida ao criar uma constante com a _string_ de
erro em uma época onde não havia muito espaço disponível no terminal para
representar muitos caracteres e, no caso de uma mensagem de erro utilizada em
várias ocasiões, onde os compiladores não eram tão espertos para cortar uma
possível redundância, ou seja, não custava muito estender isso até C.  

Certo, voltando ao código, vejamos em qual arquivo o ``notid`` é mencionado:

```console
$ find . -type f -name '*.c' -print | xargs grep 'notid' 
./msg.c: const char     notid[]         = "is not an identifier";
./name.c:               failed(nam, notid);
``` 

**Linhas 496 até 504, [arquivo ``name.c``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/name.c):**

```c
struct namnod *
lookup(register unsigned char *nam)
{
        register struct namnod *nscan = namep;
        register struct namnod **prev = NULL;
        int             LR;

        if (!chkid(nam))
                failed(nam, notid);
```

Como esperado, o código verifica se o identificador (``char *nam``) é válido
antes de continuar operando em detalhes que eu não vou entrar aqui a fim de
manter esse artigo direto, vejamos o que essa função ``chkid()`` de fato faz.

**Linhas 530 até 546, [arquivo ``name.c``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/name.c):**

```c
BOOL
chkid(unsigned char *nam)
{
        register unsigned char *cp = nam;

        if (!letter(*cp))
                return(FALSE);
        else
        {
                while (*++cp)
                {
                        if (!alphanum(*cp))
                                return(FALSE);
                }
        }
        return(TRUE);
}
```

Cavando mais um pouco, vemos que ``letter()``, na realidade, é um macro definido
no arquivo
[``ctype.h``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/ctype.h):

```console
$ find . -type f -print | xargs egrep 'letter\(.*\)'
./ctype.h: #define      letter(c)       ((c<QUOTE) && sh_ctype2[c]&(T_IDC))
./name.c:       if (letter(*argscan))
./name.c:       if (!letter(*cp))
./hashserv.c:           if (letter(*s))
./macro.c:                      if (letter(c))
./word.c:               if (!letter(arg->argval[0]))
```

Esse macro é, de tão simples, complexo. Mas, fazendo uma aposta segura baseada
no que eu consegui entender, esse macro verifica se o caractere ``c`` é menor do
que ``QUOTE`` --- esse que é definido em
[``mac.h``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/mac.h)
como 0200 --- e, em seguida, se verifica se o valor de ``c`` está presente no
_array_ ``sh_ctype2`` que é, efetivamente, uma tabela de caracteres usados
dentro do shell para nomes de funções, passando por caracteres reservados,
esses que são denotados por macros próprios ou por zeros, até caracteres
válidos para se usar como identificadores, esses que são denotados pelos
macros ``_LPC`` (para letras minúsculas, "*__l__owercase*") e ``_UPC`` (para
letras maíusculas, "*__u__ppercase*"), já números são denotados pelo macro
``_DIG`` (que é uma abreviação clara de "dígito").

**Linhas 87 até 134, [arquivo ``ctype.c``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/ctype.c):**

```c
const unsigned char	sh_ctype2[] =
{
/*	000	001	002	003	004	005	006	007	*/
	0,	0,	0,	0,	0,	0,	0,	0,

/*	bs	ht	nl	vt	np	cr	so	si	*/
	0,	0,	0,	0,	0,	0,	0,	0,

	0,	0,	0,	0,	0,	0,	0,	0,

	0,	0,	0,	0,	0,	0,	0,	0,

/*	sp	!	"	#	$	%	&	'	*/
	0,	_PCS,	0,	_NUM,	_DOL2,	0,	0,	0,

/*	(	)	*	+	,	-	.	/	*/
	0,	0,	_AST,	_PLS,	0,	_MIN,	0,	0,

/*	0	1	2	3	4	5	6	7	*/
	_DIG,	_DIG,	_DIG,	_DIG,	_DIG,	_DIG,	_DIG,	_DIG,

/*	8	9	:	;	<	=	>	?	*/
	_DIG,	_DIG,	0,	0,	0,	_EQ,	0,	_QU,

/*	@	A	B	C	D	E	F	G	*/
	_AT,	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,

/*	H	I	J	K	L	M	N	O	*/
	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,

/*	P	Q	R	S	T	U	V	W	*/
	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,	_UPC,

/*	X	Y	Z	[	\	]	^	_	*/
	_UPC,	_UPC,	_UPC,	0,	0,	0,	0,	_UPC,

/*	`	a	b	c	d	e	f	g	*/
	0,	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,

/*	h	i	j	k	l	m	n	o	*/
	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,

/*	p	q	r	s	t	u	v	w	*/
	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,	_LPC,

/*	x	y	z	{	|	}	~	del	*/
	_LPC,	_LPC,	_LPC,	_CBR,	0,	_CKT,	0,	0
};
```

Ha, touché, _encore_: a minha hipótese de que ele só faz uso de caracteres ASCII
"primitivos" acabou de ser comprovada por essa tabela --- e também pela
``sh_ctype1``, que veremos mais à frente --- já que ela é a tabela ASCII de 1977
esculpida em mármore carrara.  
Em suma, a função ``chkid()`` executa primeiro o macro ``letter()``, que
verifica se o primeiro caractere é uma letra --- algo que equivale à função
``isalpha()``, mas, por mais que essa função tenha aparecido pela primeira vez
no UNIX v7, ela possivelmente não estava disponível quando escreveram essa
seção do código por uma questão de meses, semanas ou dias ou, até mesmo,
tinha uma carga muito grande para o que o shell poderia ter quando comparado
a um simples macro --- e, posteriormente, "caminha" pela _string_ usando um
laço _while_ e, utilizando o ``alphanum()``, também macro --- que equivale à
função ``isalnum()`` --- verifica se a função contém um dígito ou um número,
entretanto não aceita nada diferente disso.
A ``setname()`` faz exatamente a mesma coisa, com o adendo de que, ao contrário
da ``lookup()``, como o nome já explicita, não busca pelo identificador em uma
tabela que relacione com a operação a ser feita, mas sim escreve --- ou
"configura" (_set_) --- o identificador e seu correspondente na tabela.

**Linhas 179 até 191, [arquivo ``name.c``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/name.c):**

```c
void
setname (	/* does parameter assignments */
    unsigned char *argi,
    int xp
)
{
	register unsigned char *argscan = argi;
	register struct namnod *n;

	if (letter(*argscan))
	{
		while (alphanum(*argscan))
			argscan++;
```

Bem, só um adendo antes da conclusão: talvez você me pergunte, _"Luiz, por que
não usaram a ``chkid()`` ao invés de reescrever a mesma coisa, só que com lógica
diferente?"_; a resposta é simples, basta apenas que leiamos o resto da função e
que entendamos que essa função não apenas verifica se um nome é válido e
adiciona-o na tabela, mas sim procura pelo caractere que significa nomear uma
variável, em outros termos o símbolo de igualdade (``=``), procura um espaço
para o tal nome na tabela utilizando a função ``lookup()``, essa que falamos
sobre anteriormente e que já faz a verificação completa, e então atribui o valor
ao nome no espaço da tabela utilizando uma função chamada ``assign()``. 
 
Ademais, seria interessante que descubramos o que é o tal ``_PCS`` associado ao
``!``, não custa nada, não é mesmo? Que procuremos nos arquivos de cabeçalho,
terminados em ``.h``:

```console
$ find . -type f -name '*.h' -print | xargs grep '_PCS' 
./ctype.h: #define _PCS (T_SHN)
```

Certo, é definido como ``T_SHN``, que tem o valor numérico de 040 na tabela 2,
como pode-se ver aqui:

**Linhas 47 até 54, [arquivo ``ctype.h``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/ctype.h):**

```c
/* table 2 */
#define T_BRC	01
#define T_DEF	02
#define T_AST	04
#define	T_DIG	010
#define T_SHN	040
#define	T_IDC	0100
#define T_SET	0200
```

Esse ``T_SHN`` não parece ter uma abreviação muito clara à primeira vista --- e
nem à segunda ---, no entanto, poderia-se dizer que significa que tal caractere
é restrito para uso em quaisquer identificadores.

Sobre a tabela 1, definida no _array_ ``sh_ctype1``, também há uma lista dos
mesmos caracteres, no entanto, são associados com outros macros, esses com
outros nomes, mas mesmos valores:

**Linhas 37 até 45, [arquivo ``ctype.h``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/ctype.h):**

```c
/* table 1 */
#define T_SUB	01
#define T_MET	02
#define	T_SPC	04
#define T_DIP	010
#define T_EOF	020
#define T_EOR	040
#define T_QOT	0100
#define T_ESC	0200
```

**Linhas 38 até 85, [arquivo ``ctype.c``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/ctype.c):**

```c
const unsigned char	sh_ctype1[] =
{
/*	000	001	002	003	004	005	006	007	*/
	_EOF,	0,	0,	0,	0,	0,	0,	0,

/*	bs	ht	nl	vt	np	cr	so	si	*/
	0,	_TAB,	_EOR,	0,	0,	0,	0,	0,

	0,	0,	0,	0,	0,	0,	0,	0,

	0,	0,	0,	0,	0,	0,	0,	0,

/*	sp	!	"	#	$	%	&	'	*/
	_SPC,	0,	_DQU,	0,	_DOL1,	0,	_AMP,	0,

/*	(	)	*	+	,	-	.	/	*/
	_BRA,	_KET,	0,	0,	0,	0,	0,	0,

/*	0	1	2	3	4	5	6	7	*/
	0,	0,	0,	0,	0,	0,	0,	0,

/*	8	9	:	;	<	=	>	?	*/
	0,	0,	0,	_SEM,	_LT,	0,	_GT,	0,

/*	@	A	B	C	D	E	F	G	*/
	0,	0,	0,	0,	0,	0,	0,	0,

/*	H	I	J	K	L	M	N	O	*/
	0,	0,	0,	0,	0,	0,	0,	0,

/*	P	Q	R	S	T	U	V	W	*/
	0,	0,	0,	0,	0,	0,	0,	0,

/*	X	Y	Z	[	\	]	^	_	*/
	0,	0,	0,	0,	_BSL,	0,	_HAT,	0,

/*	`	a	b	c	d	e	f	g	*/
	_LQU,	0,	0,	0,	0,	0,	0,	0,

/*	h	i	j	k	l	m	n	o	*/
	0,	0,	0,	0,	0,	0,	0,	0,

/*	p	q	r	s	t	u	v	w	*/
	0,	0,	0,	0,	0,	0,	0,	0,

/*	x	y	z	{	|	}	~	del	*/
	0,	0,	0,	0,	_BAR,	0,	0,	0
};
```

Essa tabela é utilizada pelos macros ``eofmeta()``, ``qotchar()``, ``eolchar()``,
``dipchar()``, ``subchar()`` e ``eschar()``, todos definidos no nosso agora
famigerado ``ctype.h``:

```console
$ find . -type f -name '*.h' -print | xargs grep 'sh_ctype1'
./ctype.h: extern const unsigned char   sh_ctype1[];
./ctype.h: #define      space(c)        ((c<QUOTE) && sh_ctype1[c]&(T_SPC))
./ctype.h: #define eofmeta(c)   ((c<QUOTE) && sh_ctype1[c]&(_META|T_EOF))
./ctype.h: #define qotchar(c)   ((c<QUOTE) && sh_ctype1[c]&(T_QOT))
./ctype.h: #define eolchar(c)   ((c<QUOTE) && sh_ctype1[c]&(T_EOR|T_EOF))
./ctype.h: #define dipchar(c)   ((c<QUOTE) && sh_ctype1[c]&(T_DIP))
./ctype.h: #define subchar(c)   ((c<QUOTE) && sh_ctype1[c]&(T_SUB|T_QOT))
./ctype.h: #define escchar(c)   ((c<QUOTE) && sh_ctype1[c]&(T_ESC))
```

Quase todos esses macros, com exceção dos ``subchar()`` e ``escchar()``, que
são usados pela função ``copyto()``, são usados pela função ``word()``.
Essas duas funções, ``word()`` e ``copyto()``, são usadas no processamento
dos comandos passados ao shell em si. Logo, a tabela (ou _array_, chame como
preferir) ``sh_ctype1`` é usada para delimitar caracteres que são usados na
sintaxe da linguagem em si, como colchetes, aspas simples e duplas, etc --- já
que é usada por macros que compõem funções que fazem toda a parte de
processamento léxico da linguagem ---, enquanto a ``sh_ctype2`` é específica
para os identificadores que são usados em funções e variáveis --- já que é
usada por macros que compõem a inserção de valores declarados na memória,
como vimos anteriormente na função ``setname()``, que chama uma série de
outras funções para lidar com toda a parte de identificadores.
Por isso --- não pelo adendo, mas sim pela explicação de como as funções operam
para guardar uma variável/função na memória --- um identificador como
``37programa`` falha enquanto ``programa37``, não.   
No entanto, ainda continuei com uma pulga atrás da orelha: será que a pilha de
memória em que as variáveis e as funções são guardadas é a mesma? Pois, se é,
isso explicaria o porquê de "!", "#", "$" e outros caracteres não-permitidos
para uso vulgar estarem na tabela de caracteres para identificadores __e__ com
constantes próprias --- como vimos, ``_PCS`` para o "``!``", ``_NUM`` para
"``#``" e ``_DOL2`` para "``$``". Ao dar uma espiada na página de manual, os
nomes dessas constantes ganham significado:

>  The following parameters are automatically set by the
>  shell.
>
>      #  The number of positional parameters in decimal.
>      -  Options supplied to the shell on invocation or
>         by set.
>      ?  The value returned by the last executed command
>         in decimal.
>      $  The process number of this shell.
>      !  The process number of the last background
>         command invoked.

Vale lembrar, antes de se continuar, que um "parâmetro" nesse manual
corresponde à mesma coisa que uma variável. No manual do Korn Shell 93, que
teve o mesmo lugar de origem desse Bourne shell, os Laboratórios Bell, o termo
"parâmetro" é utilizado também mas, na seção de "Expansão de Parâmetros"
("_Parameter Expansion_"), o manual nos diz que um parâmetro é a mesma coisa
que uma variável. Eu confesso que não tenho ideia imediata de onde esse termo
surgiu, para ser franco.  
Voltando ao manual, agora podemos ver que "``_PCS``" na realidade corresponde
ao número identificador de processo (P.ID.) do último comando que fora executado
em plano de fundo (ou seja, "<i><u>p</u>ro<u>c</u>es<u>s</u> number of the last 
background command invoked</i>") e "``_NUM``" ao número de parâmetros passados a
uma função --- ou ao próprio shell --- (ou seja, "<i><u>num</u>ber of positional
parameters</i>").
Já "``_DOL2``" é apenas o símbolo de dólar, não fizeram nenhum acrônimo em
especial sobre o número identificador de processo do shell, logo presumi que
fosse utilizado em algo além disso, mas não consegui achar referência imediata
no código a essa constante; no entanto, procurei e encontrei um macro chamado
``dolchar()``, que é usada em uma função chamada ``getch()``, essa que é parte
do mecanismo de análise sintática do próprio shell e que lida, justamente,
buscando por caracteres especiais; caso seja encontrado um caractere de dólar
--- não por essa constante, mas sim por uma outra chamada ``DOLLAR`` ---, começa
a ler os caracteres subsequentes e, utilizando-se justamente da ``dolchar()``,
busca caracteres que sejam reconhecidos como válidos para compôr o
identificador de uma variável. Nesse processo, ele também busca por símbolos
reservados como números --- ``$n``, que seria o equivalente de ``argv[n]``,
como pôde-se ver naquele pequeno exemplo que fiz antes de começar toda essa
jornada ---, o asterisco --- que representa todos os parâmetros posicionais,
como se fosse o ``argv`` íntegro, mas sem respeito aos espaços das _strings_
que venham a estar ali --, colchetes --- que indicam uma expansão de parâmetros
---, os cinco caracteres reservados que vimos acima (se lembra do ``T_SHN``?) e,
claro, letras quaisquer --- que são, independente de ``_UPC`` ou ``_LPC``, a
mesma constante (``T_IDC``).

**Linhas 174 até 226, [arquivo ``macro.c``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/macro.c):**

```c
static int 
getch (
    int endch,
    int trimflag /* flag to check if an argument is going to be trimmed, here document
		 output is never trimmed
	 */
)
{
	register unsigned int	d;
	int atflag = 0;  /* flag to check if $@ has already been seen within double 
		        quotes */
retry:
	d = readwc();
	if (!subchar(d))
		return(d);

	if (d == DOLLAR)
	{
		unsigned int c;

		if ((c = readwc(), dolchar(c)))
		{
			struct namnod *n = (struct namnod *)NIL;
			int		dolg = 0;
			BOOL		bra;
			BOOL		nulflg;
			register unsigned char	*argp, *v = NULL;
			unsigned char		idb[2];
			unsigned char		*id = idb;

			if (bra = (c == BRACE))
				c = readwc();
			if (letter(c))
			{
				argp = (unsigned char *)relstak();
				while (alphanum(c))
				{
					if (staktop >= brkend)
						growstak(staktop);
					pushstak(c);
					c = readwc();
				}
				if (staktop >= brkend)
					growstak(staktop);
				zerostak();
				n = lookup(absstak(argp));
				setstak(argp);
				if (n->namflg & N_FUNCTN)
					error(badsub);
				v = n->namval;
				id = (unsigned char *)n->namid;
				peekc = c | MARK;
			}
```

Essa função continua por mais de 100 linhas, é enorme e não é nem ao menos todo
o sistema de análise sintática e execução dos comandos passados ao shell, mas
ainda é admirável como foi bem-construído apesar de todas as limitações
de seu tempo; arrisco dizer, inclusive, que seria uma boa referência para quem
está disposto a escrever um analisador sintático sem precisar depender de
ferramentas como o ``yacc``(1) --- ou que simplesmente não pode utilizá-las.  
Certo, mas e sobre uma variável e uma função não poderem compartilhar o mesmo
identificador por ambos serem guardados na mesma pilha de memória?
Bem, por mais que possamos comprovar isso pelo código-fonte que acabamos de
analisar, é válido que vejamos se estamos corretos:

>       There is only one namespace for both functions and parameters.  A
>       function definition will delete a parameter with the same name and
>       vice-versa.

Algo que achei curioso também, além do uso da terminologia "parâmetro", foi se
utilizar "_namespace_" para a pilha de memória --- o que é engraçado para quem
conhece Korn Shell 93 mais a fundo pois um "namespace" no contexto dele tem o
mesmo significado que em Java, C++ e Go.  
E bem, vamos voltar ao código-fonte mais uma vez pois, mesmo que nós tenhamos
visto funções que trabalham com a estrutura ``namnod``, que contém o
identificador da variável/função e seu conteúdo em si, ainda não vimos como
é essa estrutura e quais dados ela contém. Então que procuremos sua definição.
Sabendo que é uma convenção geral da linguagem C que estruturas de dados sejam
declaradas em cabeçalhos, podemos restringir um bocado a nossa pesquisa:

```console
$ find . -type f -name '*.h' -print | xargs grep 'struct namnod'
# [...]
./name.h: struct namnod
./name.h:       struct namnod   *namlft;
./name.h:       struct namnod   *namrgt;
```

Certo, consegui uma tela cheia de resultados (e que eu obviamente encurtei)
--- principalmente do arquivo ``defs.h``, que contém a definição de funções e
algumas delas já conhecidas nossas, como a ``assign()`` ---, mas pude cortar
praticamente todos e deixar só o arquivo ``name.h`` pois é o único que parece
ter uma declaração. Vejamos o que há:

**Linhas 37 até 53, [arquivo ``name.h``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/name.h):**

```c
#define	N_ENVCHG 0020
#define N_RDONLY 0010
#define N_EXPORT 0004
#define N_ENVNAM 0002
#define N_FUNCTN 0001

#define N_DEFAULT 0

struct namnod
{
	struct namnod	*namlft;
	struct namnod	*namrgt;
	unsigned char	*namid;
	unsigned char	*namval;
	unsigned char	*namenv;
	int	namflg;
};
```

Tem uma série de coisas interessantes para se notar: primeiro, tem uma série de
constantes declaradas para funções que utilizam essa estrutura consigam lidar
com o que venha a ser armazenado na memória: ``N_ENVNAM`` para que uma variável
pertença ao ambiente e que, assim como a ``N_EXPORT``, passe para processos e
subshells, já a ``N_ENVCHG`` indica que aquela variável em questão foi
alterada. Curiosamente a palavra-chave ``readonly`` já existe desde essa época,
e é por causa dela que há a constante ``N_RDONLY``; todas essas constantes são
para variáveis, já, o que diferencia funções de variáveis na pilha de memória é
justamente a ``N_FUNCTN``. A execução da função é dada pela função ``execute()``
no arquivo ``xec.c``, mas não irei entrar muito a fundo pois o intuito disso era
apenas saber como o shell consegue colocar tudo numa pilha de memória só --- o
que, de certa forma, também é uma influência de C, que também não permite essa
traquinagem.  
Também podemos ver que sim, _string_ é o único tipo disponível, o que sim, eu já
sabia, você já sabia, todos nós sabíamos, a questão é que é interessante ver
explicitamente que é do tipo ``unsigned char`` e que usa o identificador
``namval`` --- o que podemos comprovar olhando para a função ``dfault()``, dessa
vez no arquivo ``name.c``, e vendo que o valor da variável, nela indicada como
'``v``', é associado (por meio da função ``assign()``) à ``namval`` da estrutura
informada, que tem o identificador de ``n`` no protótipo da função: 

**Linhas 222 até 227, [arquivo ``name.c``](https://github.com/ryanwoodsmall/heirloom-project/blob/master/heirloom-sh/name.c):**

```c
void
dfault(struct namnod *n, const unsigned char *v)
{
	if (n->namval == 0)
		assign(n, v);
}
```
 
Estou consciente de que minha análise não é tão profunda quanto a que fiz em
cima do GNU Make pois, de fato, não depurei o código do Bourne shell utilizando
o GDB. Entretanto, caso alguém queira me corrigir com base em uma análise mais
profunda, está livre para tal, visto que [o código dele já foi portabilizado para
UNIX-compatíveis modernos e existem n-maneiras de
obtê-lo](https://www.in-ulm.de/~mascheck/bourne/#running).  

Bem, com tudo isso em mente, podemos voltar ao código do começo:


# ["Verum sine mendacio, certum, certissimum"](https://youtu.be/m0Xf5iBLokw): 
