---
layout: post
title:  "Jornal dos Pindoramas #5: Noite quente de verão e sem luz: como ler valores com espaço para um array no Bash (ou ksh93)"
date:   2026-01-11 20:09:00 +0300
categories: jdp copacabana
---

**Note for non-lusophones: If you can't read any Portuguese, [click here](https://takusuman-github-io.translate.goog/blog/jdp/copacabana/2026/01/11/como-ler-espacos-num-array-bash.html?_x_tr_sl=pt&_x_tr_tl=en&_x_tr_hl=pt-BR&_x_tr_pto=wapp).**

Caso eu ainda não tenha dito: sim, estou de férias! E em pleno litoral,
olha só.  
E, como diria algum jornalista/escritor/intelectual cujo nome não me lembro
(e cuja frase em questão eu li no Twitter), férias são uma desculpa muito
cara para poder se ler um livro. No meu (ou melhor, nosso) caso, são uma
ótima desculpa para continuar trabalhando --- e também ler um livro, por que
não?  
A questão é que, apesar de ser um ótimo mar, não é um mar de rosas: descobri
que no litoral gaúcho a falta de luz consegue ser mais recorrente e duradoura
do que em Gramado durante o verão (o que eu confesso ter pensado não ser
possível), e o, digamos, "suporte técnico" aqui, da CEEE Equatorial, não é tão
responsivo quanto o da Serra Gaúcha pelo o que me disseram, então escrevo,
nesse exato momento, em breu total (e sem um dicionário por perto para
verificar minha escrita numa parte ou outra). 
Dito tudo isso, podemos finalmente começar o artigo.

Eu resolvi voltar a polir a Mitzune para usar no sistema de montagem do
Copacabana --- até porque eu a destinei para isso ainda lá em 2021, mas nunca
tinha chegado no estágio da automação até agora. Caso você ainda não saiba, a
Mitzune é um script de automação para o comando ``chroot``, mas não vou entrar
em muitos detalhes para não alongar a conversa. Inclusive, se você for tolerante
a mau áudio e problemas de codec, eu fiz [um vídeo (também lá em 2021) que
mostrava o funcionamento prático da Mitzune como
containerizador](https://www.youtube.com/watch?v=DjnR-UoEJ90).  
Como todo projeto relativamente antigo que fiz, tem falhas. O código em questão
até que é um bom exemplo de como escrever shell, modéstia à parte, mas muita
coisa foi feita no espírito do _hack_, ou seja, sem considerar pormenores que
não tomariam cinco ou seis linhas de código para resolver e que poderiam causar
vários problemas em qualquer ambiente Linux mais heterodoxo (ou caso alguém use
espaços em nome de arquivos).  
Dentre esses pormenores, tem o arquivo que funciona como um "proto-banco de dados"
(sendo de enorme pretensão minha ao chamá-lo assim), que guarda os ``chroot``s que
a Mitzune tem acesso e suas respectivas configurações, o ``prefixes``. Esse
arquivo, originalmente, tem nove colunas, todas separadas por... espaços. Sim,
espaços, afinal foi o mais simples de implementar na época:

```sh
function show_prefix_info {
	# [...]
    prefix_info=($(grep "$prefixName" "$mitzune_prefix/prefixes"))
	prefix_partition=$(df -H "${prefix_info[2]}" | awk 'FNR==2 {print $1}')
```

A questão é que os tempos mudam e, apesar de não ser _essencial_ para o sistema
de montagem do Copacabana, eu creio que seja uma boa hora para começar a se
consertar isso --- e também porque eu andei a implementar umas poucas coisas
novas nesse projeto para se adequar às necessidades do Copacbana.  
Como novo caractere para se delimitar os elementos, escolhi o ``|``, pois é a
escolha mais sã nesse caso. Além disso, também vou testar no terminal antes de
mexer direto no código da Mitzune para poupar tempo.

A primeira coisa que alguém pensaria é em usar o sed:

```sh
sed 's/|/" "/g; s/\(.*\)/"\1"/'
```

Simples, só substituir as barras verticais por aspas e depois fechar tudo com
aspas no começo e no fim. Ou seja, uma entrada hipotética assim:

```console
a|b c|d|e|f g h|i|j k|l|m n o p
```

Ficaria primeiro assim (``s/|/" "/g``):

```console
a" "b c" "d" "e" "f g h" "i" "j k" "l" "m n o p
```

E, por fim, assim (``s/\(.*\)/"\1"/``):

```console
"a" "b c" "d" "e" "f g h" "i" "j k" "l" "m n o p"
```

Missão cumprida!

Ou será que não?

```console
S145% t=($(echo 'a|b c|d|e|f g h|i|j k|l|m n o p' | sed 's/|/" "/g; s/\(.*\)/"\1"/'))
+ sed 's/|/" "/g; s/\(.*\)/"\1"/'
+ echo 'a|b c|d|e|f g h|i|j k|l|m n o p'
+ t=( '"a"' '"b' 'c"' '"d"' '"e"' '"f' g 'h"' '"i"' '"j' 'k"' '"l"' '"m' n o 'p"' )
S145% print -v t
+ print -v t
(
        '"a"'
        '"b'
        'c"'
        '"d"'
        '"e"'
        '"f'
        g
        'h"'
        '"i"'
        '"j'
        'k"'
        '"l"'
        '"m'
        n
        o
        'p"'
)
```

Tudo que fizemos foi por água abaixo quando o shell avalia os elementos e
coloca-os no array.  
"Pois trate de mudar o tipo de aspas!"

```console
S145% t=($(echo 'a|b c|d|e|f g h|i|j k|l|m n o p' | sed 's/|/'\'' '\''/g; s/\(.*\)/'\''\1'\''/'))
+ echo 'a|b c|d|e|f g h|i|j k|l|m n o p'
+ sed $'s/|/\' \'/g; s/\\(.*\\)/\'\\1\'/'
+ t=( $'\'a\'' $'\'b' $'c\'' $'\'d\'' $'\'e\'' $'\'f' g $'h\'' $'\'i\'' $'\'j' $'k\'' $'\'l\'' $'\'m' n o $'p\'' )
S145% print -v t
+ print -v t
(
        $'\'a\''
        $'\'b'
        $'c\''
        $'\'d\''
        $'\'e\''
        $'\'f'
        g
        $'h\''
        $'\'i\''
        $'\'j'
        $'k\''
        $'\'l\''
        $'\'m'
        n
        o
        $'p\''
)
```

E sim, o mesmo ainda ocorre se usar ``" \' \' "`` ao invés de ``' '\'' '\'' '``
no sed.  
E sim², também vai acontecer se você tentar fazer o trabalho do sed de forma
manual. E você talvez se pergunte como daria para se fazer esse trabalho do sed
de forma manual, e é assim:

```sh
sed 's/|/\
/g' | while read l; do
            printf '"%s"\n' "$l"
    done
```

Ainda usa o sed para substituir os separadores por quebras de linha, mas é
"manual", ora pois. Enfim, dá o mesmo resultado que acima, e é pior de ler
dentro de um subshell dentro de uma declaração de um array.  
Um iniciante empolgado e um tanto pretensioso (não no mau-sentido) possivelmente
tentaria isso.

Então voltamos à estaca zero: como podemos lidar com esses separadores e ler os
elementos corretamente?  
Simples, fazer como se faria numa linguagem normal. Ao invés de jogar os
elementos dentro de um subshell dentro de uma declaração de array, podemos
adicionar elemento por elemento no array:
 
```sh
sed 's/|/\     
/g' | for ((elem=0;; elem++)); do
        if read l; then
            t[$elem]="$l"
        else
            break
        fi
    done
```

Sim, ainda usamos o sed para substituir os separadores por quebras de linha e
então passar para o laço _``for``_, para então ser lido linha por linha pelo
``read`` e adicionado no array:

```console
+ echo 'a|b c|d|e|f g h|i|j k|l|m n o p'
+ ((elem=0))
+ ((1))
+ read l
+ sed $'s/|/\\\n/g'
+ t[0]=a
+ ((elem++))
+ ((1))
+ read l
+ t[1]='b c'
+ ((elem++))
+ ((1))
+ read l
+ t[2]=d
+ ((elem++))
+ ((1))
+ read l
+ t[3]=e
+ ((elem++))
+ ((1))
+ read l
+ t[4]='f g h'
+ ((elem++))
+ ((1))
+ read l
+ t[5]=i
+ ((elem++))
+ ((1))
+ read l
+ t[6]='j k'
+ ((elem++))
+ ((1))
+ read l
+ t[7]=l
+ ((elem++))
+ ((1))
+ read l
+ t[8]='m n o p'
+ ((elem++))
+ ((1))
+ read l
+ break
S145% print -v t
+ print -v t
(
        a
        'b c'
        d
        e
        'f g h'
        i
        'j k'
        l
        'm n o p'
)
```

Funciona, ficou limpo e simples. Até já daria para parar por aqui.  
Eu implementaria assim, assim como creio que a maioria dos programadores também
faria assim. Mas ainda dá para melhorar.  
Sim, dá para trocar a quebra de linha literal (vulgo "POSIX") no sed pela quebra
de linha que teríamos no printf (uma extensão originada no KornShell 93
conhecida como [_ANSI C strings_](https://www.linuxjournal.com/article/1273)),
ficando, assim, ``$'s/|/\\\n/g'``; mas e se desse para não usar o sed?  
Agora é a hora que o pessoal do C, Go (e a maioria das linguagens, francamente)
arregalará os olhos:

```sh
s='a|b c|d|e|f g h|i|j k|l|m n o p'
elem=0
for ((c=0; c<${#s}; c++)); do
    curchar="${s:$c:1}"
    if [[ "$curchar" == '|' ]]; then
        ((elem+= 1))
        continue
    fi
    t[$elem]+="$curchar"
done
```

Sim, talvez seja um bocado mais lento e é argumentavelmente mais frágil do que
a implementação acima --- afinal estamos, agora, incrementando caracteres no 
elemento do array ao invés de fazer uma nova associação ---, mas é puramente em
shell, sem chamar nada de fora. É lógica pura.

Mas bem, quando disse que é "mais frágil", é porque, se você rodaresse código
de novo sem dar um ``unset`` no array criado, teremos a mesma informação
duplicada:

```console
S145% print -v t
(
        aa
        'b cb c'
        dd
        ee
        'f g hf g h'
        ii
        'j kj k'
        ll
        'm n o pm n o p'
)
```

Para isso da duplicação, podemos simplesmente inicializar o próximo elemento do
array com uma string vazia:

```sh
# [...]
    if [[ "$curchar" == '|' ]]; then
        ((elem+= 1))
        t[$elem]=""
        continue
    fi
# [...]
```

Mas isso levanta o problema de que o primeiro elemento nunca será limpo, então
apenas o primeiro elemento se torna duplicado enquanto os outro seguem
corretos:

```console
S145% print -v t
(
        aa
        'b c'
        d
        e
        'f g h'
        i
        'j k'
        l
        'm n o p'
)
```

A solução em si? Bem, infelizmente não podemos fazer a inicialização do primeiro
elemento na declaração do laço, pois o shell trata tudo dentro das parênteses
duplas como uma expressão matemática, logo não temos como trabalhar com strings
aqui:

```console
+ ((t[0]="", c=0))
/usr/bin/ksh: elem=0, t[0]="", c=0: arithmetic syntax error
```

Mas nem tudo está perdido. Temos diversas soluções para isso, mas a primeira que
me veio à cabeça foi:

```sh
for ((c=0; c<${#s}; c++)); do
    curchar="${s:$c:1}"
    [[ "$curchar" == '|' ]] && ((elem+= 1))
    [[ (($c == 0)) || "$curchar" == '|' ]] && t[$elem]=""
    [[ "$curchar" == '|' ]] && continue
    t[$elem]+="$curchar"
done 
```

Não é absolutamente lindo, mas funciona bem.
Tá, para melhorar visualmente até daria para escrever assim:

```sh
for ((c=0; c<${#s}; c++)); do
    curchar="${s:$c:1}"
    (($c == 0)) && t[$elem]=""
    if [[ "$curchar" == '|' ]]; then
        ((elem+= 1))
        t[$elem]=""
        continue
    fi
    t[$elem]+="$curchar"
done 
```

Também temos de zerar o inteiro que guia a posição no array (no caso sendo o
``$elem``), mas isso creio que dê para fazer na própria declaração do laço
``for``:

```sh
for ((elem=0, c=0; c<${#s}; c++)); do
# [...]
```

Caso contrário, mesmo dando ``unset`` no array criado, teremos a informação
posicionada à frente do índice que desejamos:

```console
S145% print -v t
(
        [8]=a
        [9]='b c'
        [10]=d
        [11]=e
        [12]='f g h'
        [13]=i
        [14]='j k'
        [15]=l
        [16]='m n o p'
)
```

O código corrigido para ambos esses problemas seria assim:

```sh
for ((elem=0, c=0; c<${#s}; c++)); do
    curchar="${s:$c:1}"
    (($c == 0)) && t[$elem]=""
    if [[ "$curchar" == '|' ]]; then
        ((elem+= 1))
        t[$elem]=""
        continue
    fi
    t[$elem]+="$curchar"
done 
```

Funciona, é portável, ficou bonitinho, mas não é tão rápido...  
A pergunta de milhões: tem como fazer isso de forma mais rápida, portável (ao
menos entre o Broken-Again Shell e o KornShell 93) e em uma linha?  
Sim, tem!

```sh
IFS='|' read -r -A t <<< "$s"
```

Exatamente: só o ``read``.  
O problema é quando a gente chega no Bash:

```sh
bash: read: -A: invalid option
read: usage: read [-ers] [-a array] [-d delim] [-i text] [-n nchars] [-N nchars] [-p prompt] [-t timeout] [-u fd] [name ...]
```

A solução é tão besta quanto o problema: usar ``-a`` ao invés do ``-A``.  
A opção ``-a`` se tornou sinônimo para a ``-A`` do Bash no ksh2020, mas não fui
atrás da versão exata, [apenas da nota no arquivo ``NEWS`` do repositório atual
do KornShell
93](https://github.com/ksh93/ksh/blob/0f6866b6bebc26d003acb6f467d0694a8aea5177/NEWS#L1592-L1598). 
Isso pode ser resolvido de duas formas: usando uma condição para verificar se
estamos no KornShell ou no Bash ou simplesmente usando a opção ``-a`` e
ignorando versões antigas do KornShell:

```sh
if [[ -n ${.sh.version} ]]; then
    IFS='|' read -r -A t <<< "$s"
else
    IFS='|' read -r -A t <<< "$s"
fi
```

E bem, o quão mais rápido isso é? Não sei, vejamos juntos.  
Para fazer o teste, usarei o [método do prof.º Júlio Cezar
Neves](https://youtu.be/lm-B2qecsCI?si=pDyNP1xgw6wbTXWg&t=737), que é
rodar a mesma coisa 200 vezes com o ``time``:

```console
S145% cat teste1.ksh                                                      
s='a|b c|d|e|f g h|i|j k|l|m n o p'
for ((i=0; i<200; i++)); do
        IFS='|' read -r -A t <<< "$s"
done
```

```console
localhost% cat teste2.ksh
s='a|b c|d|e|f g h|i|j k|l|m n o p'
for ((i=0; i<200; i++)); do
        for ((elem=0, c=0; c<${#s}; c++)); do
            curchar="${s:$c:1}"
            (($c == 0)) && t[$elem]=""
            if [[ "$curchar" == '|' ]]; then
                ((elem+= 1))
                t[$elem]=""
                continue
            fi
            t[$elem]+="$curchar"
        done
done
```

Cá os resultados:

```console
S145% time ksh teste1.ksh   

real    0m00.03s
user    0m00.01s
sys     0m00.01s
```

```console
S145% time ksh teste2.ksh 

real    0m00.10s
user    0m00.09s
sys     0m00.00s
```

Em suma, a nossa implementação "manual", puramente algorítmica, foi **três
vezes mais lenta** do que o ``read``.

Para fechar com chave de ouro (e ser justo), também vou comparar com a nossa
primeira implementação usando o sed, que achei ser mais rápida que a nossa
"manual":

```console
S145% cat teste3.ksh
for ((i=0; i<200; i++)); do
        printf 'a|b c|d|e|f g h|i|j k|l|m n o p' |
                sed $'s/|/\\\n/g' |
                for ((elem=0;; elem++)); do
                        if read l; then
                            t[$elem]="$l"
                        else
                            break
                        fi
                done
done
```

E o resultado é que eu estava redondamente errado:

```
S145% time ksh teste3.ksh

real    0m01.30s
user    0m00.90s
sys     0m00.66s
```

Isso foi bom para vermos que nem sempre chamar um programa "de fora" do shell
vai ser mais rápido do que fazer uma operação interna, mesmo que seja num SSD.  
Ela não foi minimamente mais lenta, ela foi **13 vezes mais lenta** do que a
"manual", essa que já era lenta comparada com o ``read``. Numa comparação dessa
implementação diretamente com o ``read``, ela é cerca de **43 vezes mais
lenta**.  
Organizando tudo isso numa tabela, só pelo frufru, temos:

| tempo                            | real     | user     | sys      |
|----------------------------------|----------|----------|----------|
| ``read``  +  ``IFS='\|'``        | 0m00.03s | 0m00.01s | 0m00.01s |
| Manual (caractere por caractere) | 0m00.10s | 0m00.09s | 0m00.00s |
| ``sed`` + ``for`` + ``read``     | 0m01.30s | 0m00.90s | 0m00.66s |


Não sei por que organizei tudo isso numa tabela sendo que eu acabei de chegar na
conclusão, mas tudo bem. Talvez tenha sido para deixar esse artigo um pouco mais
"cientificóide".

Talvez uma forma decente de terminar esse artigo seja mostrando como isso fica
no código da Mitzune.

```sh
function show_prefix_info {
        # Transforms the line containing the $prefixName information
        # in an array.
        grep "$prefixName" "$mitzune_prefix/prefixes" |
                IFS='|' read -r -a prefix_info
```

É, ficou bom. Talvez eu ainda precise mexer na expressão do grep para "pegar"
estritamente apenas o prefixo passado, e não similares (ex.: ``copacabana`` e
``copacabana-0.4``), mas isso é coisa pouca.

À altura em que escrevo, a luz já voltou e não está mais tão calor. De fato, já
termino de escrever hoje, dia 11, às 8h.  
Preciso otimizar não apenas código, mas também minha escrita. :^)

E bem, depois de todo o artigo, talvez você ainda esteja se perguntando por que
tem um artigo técnico no meio do JdP sendo que até agora só foi usado para
resumir semanas (ou meses) de progresso no projeto Pindorama e se este artigo
não deveria estar em qualquer outra parte do blog. A resposta é que decidi
passar a escrever esse tipo de artigo como parte do JdP sempre que se tratar de
algo do Pindorama --- ou seja, quase todos os novos artigos técnicos serão
parte do JdP, presumo. 

Agradeço pela leitura e até a próxima. Falou!
