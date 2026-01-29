---
layout: post
title:  "Escrevendo um 'demake' do select do Korn Shell para o Bourne shell original"
date:   2026-01-29 09:10:00 +0300
categories: shell 
---

... e para POSIX também.

Semana retrasada, o Samuel ([``callsamu``](https://github.com/callsamu)), um
grande amigo meu, me pediu para escrever um script simples para lidar com o
[Timewarrior](https://timewarrior.net). A ideia era algo simples: um programa
com uma lista de opções que rode constantemente em outro terminal, como um
cronômetro à parte mesmo.  
De primeira, ele sugeriu que eu usasse o Korn Shell 93 para fazer o script, até
porque existem funções nele que facilitam a vida de quem quer escrever um
programa interativo, como o próprio ``select`` (o tema deste artigo).  
No entanto, por se tratar de algo simples, eu decidi fazer em [Bourne shell
original](https://heirloom.sourceforge.net/sh.html) (nem sequer no POSIX) e
implementar o que o Korn Shell teria de forma embutida em nome da diversão.  
Esse artigo não se trata desse script em si, então vamos direto ao que me
levou a precisar usar o ``select`` nesse código.  

Em um certo momento, eu queria que fosse possível listar os IDs de cada um
desses cronômetros numa lista a fim de selecioná-los para alguma ação em
específica, afinal, o timewarrior faz uso desses IDs para a esmagadora
maioria das operações.  
Existem várias abordagens possíveis, como, por exemplo, pedir para o usuário
inserir o ID em específico manualmente após rodar o ``timew summary`` e, caso
não estivesse na lista de IDs, jogar uma mensagem de erro e repetir o pedido. 
No entanto, algo como o ``select`` ainda seria uma menor opção, tanto por
reduzir a possibilidade de erro quanto por ser mais rápido de usar, tendo apenas
de digitar o índice numérico da lista.  
Cá vai um trecho do manual do Korn Shell 88, onde o ``select`` apareceu
primeiro --- algo ligeiramente engraçado, pois o shell POSIX, que tem o o 88 
como a maior, digamos, inspiração, não inclui essa função, o que talvez seja
compreensível a fim de simplicidade e de não incluir uma das _killer features_
(que não é algo absolutamente insubstituível) do 88 no padrão geral:

> select _identificador_  [  in _palavras_  . . .  ] ;do _lista_ ;done
>   O comando ``select`` exibe na saída de erro padrão (_file descriptor_ nº 2),
>   o conjunto de **palavras**, cada uma numerada pelo seu índice. Se
>   "in **palavras** ..." vier a ser omitido, então os parâmetros posicionais
>   ($@) serão utilizados em seu lugar.
>   O prompt PS3 é, então, exibido e uma linha é lida da entrada padrão.
>   Se essa linha consistir do número de índice de uma das palavras listadas,
>   então o valor da variável **identificador** será definido para a palavra
>   correspondente. Se a linha estiver vazia, a lista com o conjunto de
>   **palavras** e seus respectivos índices é impressa novamente. Caso nenhuma
>   dessas condições acima se aplique e a linha contenha um valor válido de
>   índice e nem esteja vazia, a variável **identificador** é definida como
>   nula, uma _string_ vazia. O conteúdo da linha que fora lida é salvo na
>   variável ``REPLY``. A **lista** é executada para cada seleção até que o
>   ``select`` seja quebrado (``break``), o que é equivalente a um laço
>   ``while``, ou até que um caractere de fim de arquivo --- ``EOF`` ou
>   Ctrl + D/^D --- seja encontrado.
>   Caso a variável ``REPLY`` seja nula --- ou seja, um índice inválido foi
>   informado ---, o conjunto de **palavras** com seus respectivos índices é
>   impresso novamente, com o prompt PS3 a pedir uma nova resposta --- o que é,
>   na prática, o comportamento do caso em que a linha esteja vazia, mencionado
>   anteriormente.

Traduzido e adaptado de [KSH88(1), do saudoso site
``research.att.com``](https://web.archive.org/web/20130522134302/http://www2.research.att.com/sw/download/man/man1/ksh88.html).

Bem, é simples. É uma função que associa elementos a um índice (algo que o
próprio shell já faz com elementos posicionais, ``$1``, ``$2``, etc) e lê o
número do índice para retornar seu elemento correspondente.  
O que vai mudar de fato aqui é que não temos como criar uma nova palavra-chave
e copiar toda a funcionalidade um-para-um. Não temos macros aqui, shell não é C.  
Por esse motivo, estaremos a fazer um demake da palavra-chave enquanto função,
não um port compatível de forma intercambiável.  
Irei pela abordagem de ter apenas os elementos (palavras, como visto no trecho
do manual acima) enquanto argumentos da função e exportar o conteúdo da
linha/índice e o elemento selecionado nas variáveis ``REPLY`` e ``SELECTED``,
respectivamente.

Antes disso, precisamos saber como acessaremos o elemento posicional tendo seu
número passado em outra variável.  
No Korn Shell 93 --- e no Bash, e outros shells que tenham suporte a arrays ---,
nós podemos ir por uma via preguiçosa: duplicar o array dos elementos
posicionais em outro e acessar pelo índice.  
Cá vai um código extremamente básico para demonstrar:

```sh
function indice_elemento {
    elems=("$@")
    print -v elems
    printf >&2 'Digite um índice: '
    read i
    print -v elems[i]
}
```
```console
S145% indice_elemento a b c d e
(
        a
        b
        c
        d
        e
)
Digite um índice: 1
b
S145% : Lembrete: nós contamos do zero aqui.
```

No entanto, não temos como declarar arrays no Bourne shell e nem no POSIX, então
essa possibilidade preguiçosa já cai por terra. O que resta é que usemos algo
análogo a um ponteiro em C --- ou, já que estamos numa linguagem interpretada,
uma analogia a [variáveis
variáveis](https://www.php.net/manual/en/language.variables.variable.php) (sim,
o nome é esse mesmo)
do PHP e aos [``nameref``s do Korn Shell
93](https://web.archive.org/web/20130520185752/http://www2.research.att.com/sw/download/man/man1/ksh.html)
seria mais justa --- para que consigamos acessar o valor da variável cujo
identificador está contido em outra variável.  
Em 2024, implementei um código para isso no
[herbiec](https://github.com/takusuman/herbiec):

```sh
function eval_per_identifier {
	[[ $verbose ]] && set -x
	identifier=$1

	eval echo -n \$\{"$identifier"\}
	unset identifier
}
```

Dentro do contexto do herbiec (e do conhecimento que eu tinha na época), essa
função acabava por ser "eficiente" e cumpria o trabalho para a maioria das
coisas. Na época eu também não tinha ideia da existência de ``nameref``s nativos
do Korn Shell 93.  
Todavia, para esse caso, não precisamos de uma função dedicada a isso, apenas
escrever uma declaração com o valor de ``\$`` (o caractere "$" puro) ao lado do
número de índice/identificador, com tudo isso sendo precedido pelo ``eval``, que
executa a linha de código em definitivo:

```sh
eval selecionado="\$$indice"
```

Sim, eu sei que o ``eval`` não é a coisa mais segura do mundo, mas não temos
outra alternativa na prática que seja portátil.  
Escrevi uma função para demonstrar como isso funciona: 

```sh
teste_namerefoide() {
    read i
    eval elem="\$$i"  
    echo Selecionado $elem
}
```

```console
S145% teste_namerefoide Hoje é um belo dia
+ teste_namerefoide Hoje é um belo dia
+ read i
4
+ eval elem='$4'
+ elem=belo
+ echo Selecionado belo
Selecionado belo
```

Feito isso, agora é só correr para o abraço e implementar uma função que liste
os elementos e leia o índice:

```sh
select_in() {
    prompt3="${PS3:-#? }"
    REPLY=0
    SELECTED=""
    index=0
    for elem do
        index=`expr $index + 1`
        printf >&2 '%d) %s\n' \
            "$index" "$elem"
    done
    printf >&2 '%s' "$prompt3"
    read -r REPLY </dev/tty
    if [ `expr "x$REPLY" : 'x[0-9]*$'` -gt 0 ] &&
        [ $REPLY -ge 1 ] && [ $REPLY -le $# ]; then
        eval SELECTED=\"\$$REPLY\"
    fi
    export REPLY SELECTED
}
```

Resolvi aplicar algumas boas-práticas aqui: primeiramente, estamos lendo a
entrada do ``/dev/tty`` ao invés da entrada padrão a fim de evitar eventuais
problemas com redirecionamentos; além disso, também utilizei a opção ``-r``
(de "raw", "cru") pois não esperamos ter de tratar nada na string de índice,
afinal é só um inteiro.  
Também fiz questão de usar aspas duplas no ``eval``, só para garantir. 
Ah, e claro, tratei a entrada duas vezes: primeiro, verificamos se a entrada é
puramente numérica, afinal, estamos a lidar com índices e, em seguida,
verificamos se o índice é válido.  
E bem, cá vai uma curiosidade (e um spoiler de um futuro JdP que aborde o
[Heirloom NG](https://heirloom-ng.pindorama.net.br)): o Bourne shell não suporta
expressões aritméticas nativamente, logo se torna necessário usar uma
calculadora externa para tal, sendo o ``expr`` a mais comum.

Para usar essa função é simples, sendo apenas questão de inseri-la na condição
de um ``while`` **sem** um subshell (afinal precisamos das variáveis exportadas
``REPLY`` e ``SELECTED``):

```sh
while select_in insira os elementos cá; do
    : [...]
done
```

E cá vai um código de exemplo, usando todas as funcionalidades que
implementamos. No espírito do Gramado in Concert, achei que seria interessante
usar a famigerada escala de Dó como exemplo:

```sh
PS3='Qual nota? '
while select_in Dó Ré Mi Fá Sol Lá Si; do
	case "x$SELECTED" in
		'x') : "SELECT" está vazio.
			if [ -z "$REPLY" ]; then
				echo 'Você não selecionou nada?'
			else
				echo "Índice inválido: $REPLY"
				: Sai da seleção.
				break 
			fi ;;
		x'Dó'|x'Fá'|x'Sol') echo "Nota $SELECTED, ou você quis dizer clave?" ;;
		*) echo Nota $SELECTED ;;
	esac
done
```

E bem, talvez uma pergunta que possa ser-me feita é qual a licença para
reutilizar esse código. Tudo nesse blog está na [CC-BY
4.0](https://creativecommons.org/licenses/by/4.0/deed.pt-br), que permite
redistribuição franca desde que haja atribuição de créditos.  
Por mim, faça o que tu queres, só credites me também.  

Finalizo esse artigo já de volta à Serra, depois de ter feito uma parte no
ônibus e, claro, a grande maioria no litoral.
Agradeço por ter lido até aqui. Até a próxima.
