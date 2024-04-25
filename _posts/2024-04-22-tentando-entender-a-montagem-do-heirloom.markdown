---
layout: post
title: "Tentando entender o sistema de montagem do Heirloom"
---

Já faz um certo tempo que postei algo aqui com os artigos ainda não terminados,
mas enfim. Resolvi começar esse blog comentando sobre algo que eu venho
trabalhando há um tempo --- desde 2021, o que já seria um "caso sério"? Só que
ao invés de noites em claro por estar empapuçado de amor, são noites em claro
resolvendo _bugs_ como transbordamento de dados, falhas de segmentação e afins
---, esse que é uma bifurcação do Heirloom Toolchest para a nova geração,
também conhecido como "[Heirloom New Generation](http://heirloom-ng.pindorama.dob.jp)".  
A história está completa no website, então não me estenderei tanto no que se
tange à origem do projeto Heirloom Toolchest original e o porquê dessa
bifurcação.  

## _"Se o projeto é, agora, seu, qual a dificuldade em entender o sistema de montagem?"_

Em suma, desde quando comecei até hoje, eu e toda a equipe praticamente
trabalhamos apenas na parte em C do projeto, [fazendo pouca ou nenhuma alteração
no sistema de montagem em si](https://github.com/Projeto-Pindorama/heirloom-ng/commits/20240220-fix/build/mk.config)
--- que é escrito puramente em Makefile (usando do "dialeto", ou melhor, extensões
do GNU) e shell, na boa e velha maneira de se fazer um sistema de montagem ---
afinal, não havia um porquê de ir mais a fundo e entender além do que já era
descrito no README do projeto original desde a primeira versão [desde pelo
menos 2004](https://github.com/ryanwoodsmall/heirloom-project/blob/2ec07de0c363f8dec1cabbe02ed7b97a4358e59a/heirloom/README#L75C56-L83C17):

> The first thing to understand is the build system. This is actually quite
> simple: Every directory contains a file named Makefile.mk that includes the
> directory-specific make instructions. To generate the real Makefile,
> configuration settings are prepended to the directory-specific file.
> You have to edit these configuration settings before you start compiling;
> they are located in the file build/mk.config which is also in make syntax.
> Follow the descriptions in this file and select appropriate values for
> your system.

Em suma, isto basicamente nos diz que os ``Makefile``s para cada programa são
gerados a partir da matriz ``Makefile.mk``, essa específica de cada programa, e
somados (ou contatenados, chame como preferir) ao arquivo de configuração
``mk.config``, presente na pasta ``build/``.  
Sim, a documentação do projeto resume-se basicamente a esse README e não vai
mais a fundo do que isso, apenas o necessário para que você, caso quisesse,
conseguisse mexer em algo antes de compilar (já que não usamos ``./configure``s
nem nada do tipo) e/ou não estivesse numa plataforma que usasse o RPM.

## _"Por que mexer justo agora, 20 anos depois?"_

Entretanto, depois de muita insistência de algumas pessoas que acompanham o
projeto, eu resolvi que seria bom começar a implementar uma suíte de testes ---
até porque, mesmo que o Heirloom NG continuasse sendo um mero conjunto de
_patches_ feitos especificamente para que funcionasse como userspace do
[Copacabana](http://copacabana.pindorama.dob.jp) e nada mais nem nada menos,
continuaria sendo bom ter certeza de que os programas estão rodando
corretamente e se os _patches_ aplicados realmente ajeitaram algo
ou se foram placebo por simplesmente criar um binário não-funcional.  
Então temos um problema: como criar um novo "alvo" do Make para testes? Como
chamá-los? Como saber se não estaremos a resolver tudo com uma "foice e um
martelo" e deixando a precisão e os padrões do projeto de lado?  
Simples _ma non troppo_: alquimia. Vulgo estudar o sistema de montagem
quebrando-o em partes e analisando o que for possível em todas as frentes,
fazendo anotações e corrigindo quando necessário.

## Destilação: analisando o produto e o subproduto do ``gmake``

Primeiramente, vamos analisar quais arquivos compõem o sistema de montagem em
si. Numa árvore de código-fonte limpa, essa é a saída que deveremos ter:

```console
S145% cd /usr/home//luiz/projetos/heirloom-toolchest/
S145% find . -type f \( -name '*.mk' -o -name 'mk.*' -o -name '[M,m]akefile' \) -print
# [...]
./build/mk.head
./build/mk.tail
./build/Makefile.mk
./build/mk.config
# [...]
./Makefile.mk
./makefile
```

Além dos arquivos que são específicos para cada programa e biblioteca --- e que
eu cortei da saída, afinal são literalmente mais que cem ---, os ``Makefile.mk``,
temos basicamente os arquivos dentro de ``build/``, um ``Makefile.mk`` e um
arquivo que passa praticamente batido, ``makefile``.

O movimento natural a partir disso, ignorando a parte de configuração
customizada do ``build/mk.config``, é executar o GNU Make. Logo, faremos-o, mas
com a partir do ``strace``(1) e iremos parar antes de começarmos a compilar algo
de fato.

```console
S145% pwd                                                                                    
/usr/home/luiz/projetos/heirloom-toolchest
S145% strace gmake
```

Certo, é muita coisa na tela, melhor passar para um arquivo e analisar de lá.

```console
S145% strace gmake &> '/tmp/strace make Heirloom.txt'
^CS145% pg /tmp/strace\ make\ Heirloom.txt
```

As primeiras linhas podem ser ignoradas por nós, afinal elas tratam de
procedimentos internos do sistema para carregar bibliotecas dinâmicas, além de
configurar o conjunto de sinais e ações que devem ser tomadas caso algum seja
detectado (``sigaction(2)`` e seus amigos). O que nos interessa é ver qual
arquivo Makefile é aberto, afinal, após a primeira compilação, o Heirloom gera
um outro arquivo chamado ``Makefile`` (com "M" maiúsculo) na raiz da pasta,
então, o que está acontecendo?

```console
^CS145% pg /tmp/strace\ make\ Heirloom.txt
# [...]
newfstatat(AT_FDCWD, ".", {st_mode=S_IFDIR|0775, st_size=1458, ...}, 0) = 0
openat(AT_FDCWD, ".", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
fstat(3, {st_mode=S_IFDIR|0775, st_size=1458, ...}) = 0
brk(0x5c5a91380000)                     = 0x5c5a91380000
getdents64(3, 0x5c5a913577d0 /* 150 entries */, 32768) = 4208
getdents64(3, 0x5c5a913577d0 /* 0 entries */, 32768) = 0
close(3)                                = 0
openat(AT_FDCWD, "makefile", O_RDONLY)  = 3
fcntl(3, F_GETFD)                       = 0
fcntl(3, F_SETFD, FD_CLOEXEC)           = 0
fstat(3, {st_mode=S_IFREG|0644, st_size=4164, ...}) = 0
read(3, "SHELL = /bin/sh\r\n\r\nSUBDIRS = bui"..., 4096) = 4096
read(3, "igraphs \\\r\n\t\t\t-Wuninitialized -W"..., 4096) = 68
read(3, "", 4096)                       = 0
close(3)                                = 0
newfstatat(AT_FDCWD, "RCS", 0x7ffd276bf100, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "SCCS", 0x7ffd276bf100, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "makefile", {st_mode=S_IFREG|0644, st_size=4164, ...}, 0) = 0                                                                                    
clock_gettime(CLOCK_REALTIME, {tv_sec=1713877589, tv_nsec=474742533}) = 0
newfstatat(AT_FDCWD, "dummy", 0x7ffd276bcec0, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "makefiles", 0x7ffd276bcec0, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "Makefile", 0x7ffd276bcd50, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "Makefile.mk", {st_mode=S_IFREG|0644, st_size=3438, ...}, 0) = 0
fstat(1, {st_mode=S_IFREG|0644, st_size=13507, ...}) = 0
:█ 
```

Perceberam isso? Olhem com cuidado:

Primeiro, abrimos a pasta atual, representada como ``.``, para um descritor de
arquivo, que então é informado à função ``fstat``(2) junto com o endereço de
uma ``struct`` que conterá informações sobre a pasta  atual e também a
``getdents``(2), essa que, de fato, vai nos dar uma lista dos arquivos na pasta
informada em forma de um _array_ de ``struct``s do tipo ``dirent``.  
Sim, isso soa confuso, mas não se preocupem, iremos olhar o código diretamente
agora ao invés das chamadas de sistema que são feitas, afinal, muitas delas
estão escondidas por funções-porcelana.

Podemos ver qual parte do código de fato faz isso ao procurarmos não por uma
chamada da função ``getdents`` ou ``fstat``, mas sim onde a ``struct`` é
declarada. Por uma questão de formalidade, iremos estudar o código do GNU Make
3.81, versão que foi contemporânea do último lançamento formal do Heirloom
Toolchest de 15 de Julho de 2007.  
Para isso, basta que baixemos o arquivo do servidor HTTPS do GNU e extrairmos-o.

```console
S145% curl -Lo- https://ftp.gnu.org/gnu/make/make-3.81.tar.gz \
        | gzip -cd - \
        | tar -xvf - -C /usr/src
```

Então, podemos realizar a busca utilizando o ``find``(1), ``xargs``(1)
e ``grep``(1):

```console
S145% pwd
/usr/src/make-3.81
S145% find . -type f -name '*.c' -print | xargs grep -n 'readdir'
./dir.c: 678:      ENULLLOOP (d, readdir (dir->dirstream));
./dir.c: 1202:  gl->gl_readdir = read_dirstream;
./read.c: 223:  /* all lower case since readdir() (the vms version) 'lowercasifies' */
./vmsfunctions.c: 78:readdir (DIR *dir)
./glob/glob.c: 250:# define readdir(str) __readdir (str)
./glob/glob.c: 1317:                                  ? (*pglob->gl_readdir) (stream)
./glob/glob.c: 1318:                                  : readdir ((DIR *) stream));
./w32/compat/dirent.c: 109:readdir(DIR* pDir)
./w32/compat/dirent.c: 131:     /* bump count for next call to readdir() or telldir() */
./w32/compat/dirent.c: 160:     /* reset members which control readdir() */
./w32/compat/dirent.c: 181:     /* return number of times readdir() called */
./w32/compat/dirent.c: 199:     for (--nPosition; nPosition && readdir(pDir); nPosition--);
```

Curioso, o único arquivo que não menciona a função diretamente é o ``read.c``...

```console
S145% sed 9q read.c 
/* Reading and parsing of makefiles for GNU Make.
Copyright (C) 1988, 1989, 1990, 1991, 1992, 1993, 1994, 1995, 1996, 1997,
1998, 1999, 2000, 2001, 2002, 2003, 2004, 2005, 2006 Free Software
Foundation, Inc.
This file is part of GNU Make.

GNU Make is free software; you can redistribute it and/or modify it under the
terms of the GNU General Public License as published by the Free Software
Foundation; either version 2, or (at your option) any later version.
```

_Ha, touché!_ Achamos o arquivo que lê os ``Makefile``s dentro do próprio GNU
Make.  
Vejamos, aquele comentário refere-se a uma questão do VMS, mais especificamente
o fato de que esse sistema tem problemas em lidar com maiúsculas e minúsculas no
nome dos arquivos. Isso não nos interessa agora, mas é uma pista de que há algum
tipo de lista de arquivos que o comando ``gmake``/``make`` precisa procurar
para ler e executar por padrão.  

**Linhas 219 até 234, [arquivo ``read.c``](https://github.com/mirror/make/blob/3.81/read.c):**

```c
  /* If there were no -f switches, try the default names.  */

  if (num_makefiles == 0)
    {
      static char *default_makefiles[] =
#ifdef VMS
        /* all lower case since readdir() (the vms version) 'lowercasifies' */
        { "makefile.vms", "gnumakefile.", "makefile.", 0 };
#else
#ifdef _AMIGA
        { "GNUmakefile", "Makefile", "SMakefile", 0 };
#else /* !Amiga && !VMS */
        { "GNUmakefile", "makefile", "Makefile", 0 };
#endif /* AMIGA */
#endif /* VMS */
      register char **p = default_makefiles;
      while (*p != 0 && !file_exists_p (*p))
        ++p;
    /* [...] Mais 2904 linhas... */
```

Podemos ver que há a declaração de um _array_ chamado ``*default_makefiles[]``.  
Um nome autodescritivo: em ordem de prioridade, tem uma lista de arquivos que
devem ser lidos por padrão pelo programa, como tínhamos proposto antes enquanto
hipótese. Limpando toda a "crosta" de compatibilidade feita por ``#ifdef``s e
focando apenas em sistemas UNIX-compatíveis, temos isso:  

**Linha 229, arquivo ``read.c`` (adaptado):**

```c
static char *default_makefiles[] = { "GNUmakefile", "makefile", "Makefile", 0 };
```

_Touché, encore._ Era a prova que precisávamos para saber que o arquivo
"``makefile``" na raiz do Heirloom era executado primeiro e que o
"``Makefile``" gerado jamais seria executado se não fosse explicitado.

Poderíamos simplesmente ter lido a [página da especificação POSIX no site da
OpenGroup](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/make.html#tag_20_76_13)
em coisa de 10 minutos ao invés de investir pelo menos algumas horas em todo
esse processo? Claro, mas o processo "alquímico" nos mostra mais sobre como as
coisas funcionam.

Logo, volta a dúvida que, parafraseando Dante Alighieri, tenciona-me a cabeça
na última semana desde o _pull-request_
[nº 49](https://github.com/Projeto-Pindorama/heirloom-ng/pull/49): para que
serve o "``Makefile``" gerado? E o que o "``makefile``" de fato faz para
gerá-lo? Pois bem, como dito anteriormente, qualquer arquivo com a extensão
``.mk`` terá o conteúdo dos arquivos da pasta ``build/`` pré-anexados antes do
seu conteúdo na síntese final, o que pode ser visto além das palavras aqui:  

**Linhas 23 até 34, [arquivo ``makefile``](https://github.com/Projeto-Pindorama/heirloom-ng/blob/master/makefile):**

```makefile
.DEFAULT:
        + for i in $(SUBDIRS) ;\
        do      \
                (cd "$$i" && $(MAKE) $@) || exit ; \
        done
        $(MAKE) -f Makefile $@

.SUFFIXES: .mk
.mk:
        cat build/mk.head build/mk.config $< build/mk.tail >$@

makefiles: Makefile $(SUBDIRS:=/Makefile)
```

Segundo a documentação da OpenGroup, ``.SUFFIXES`` contém uma lista de sufixos
que devem ser utilizados pelas "regras" --- que são, basicamente, quando
declara-se um alvo --- e sua ordem define como serão de fato usados. Que honra
seja feita, a documentação específica do GNU é melhor para que isso seja
compreendido, além do código em si.
Basicamente, isso faz com que o alvo, que é um arquivo, tenha seu sufixo ---
desde que na lista ``.SUFFIXES`` --- removido e isso é passado para outra
variável, no caso ``$*``.  

**Linhas 628 até 657, [arquivo ``file.c``](https://github.com/mirror/make/blob/3.81/file.c):**

```c
void
snap_deps (void)
{
  struct file *f;
  struct file *f2;
  struct dep *d;
  struct file **file_slot_0;
  struct file **file_slot;
  struct file **file_end;

  /* Perform second expansion and enter each dependency
     name as a file. */

  /* Expand .SUFFIXES first; it's dependencies are used for
     $$* calculation. */
  for (f = lookup_file (".SUFFIXES"); f != 0; f = f->prev)
    expand_deps (f);

  /* We must use hash_dump (), because within this loop
     we might add new files to the table, possibly causing
     an in-situ table expansion.  */
  file_slot_0 = (struct file **) hash_dump (&files, 0, 0);
  file_end = file_slot_0 + files.ht_fill;
  for (file_slot = file_slot_0; file_slot < file_end; file_slot++)
    for (f = *file_slot; f != 0; f = f->prev)
      {
        if (strcmp (f->name, ".SUFFIXES") != 0)
          expand_deps (f);
      }
  free (file_slot_0);
```

Ele contatena o nome do alvo (representado por ``$@``), nomes esses que
foram declarados na regra ``makefiles``, com o sufixo de dependência,
resultando no nome do arquivo de dependência (representado por ``$<``),
e isso pode ser visto aqui, onde nós primeiro passamos pelos alvos e guardamos
os nomes dos alvos a serem gerados numa ``struct`` chamada ``*deps``, dentro da
``struct`` ``f``, essa que é atribuída por meio do macro
``record_waiting_files()``:

**Linhas 472 até 486, [arquivo ``read.c``](https://github.com/mirror/make/blob/3.81/read.c):**

```c
#define record_waiting_files()						      \
  do									      \
    { 									      \
      if (filenames != 0)						      \
        {                                                                     \
	  fi.lineno = tgts_started;                                           \
	  record_files (filenames, pattern, pattern_percent, deps,            \
                        cmds_started, commands, commands_idx, two_colon,      \
                        &fi);                                                 \
        }                                                                     \
      filenames = 0;							      \
      commands_idx = 0;							      \
      no_targets = 0;                                                         \
      if (pattern) { free(pattern); pattern = 0; }                            \
    } while (0)
```

Esse macro não aparece pelo gdb, mas sim a função que o dito chama,
``record_files()``, essa sim que irá guardar os nomes.  

**Linha 34, [arquivo ``makefile``](https://github.com/Projeto-Pindorama/heirloom-ng/blob/master/makefile):**

```makefile
makefiles: Makefile $(SUBDIRS:=/Makefile)
```

**_Backtrace_ do GDB:**

```c
record_files (filenames=<optimized out>, pattern=<optimized out>, pattern_percent=<optimized out>, deps=<optimized out>, cmds_started=<optimized out>, 
    commands=<optimized out>, commands_idx=<optimized out>, two_colon=<optimized out>, flocp=<optimized out>) at read.c:1971
1971              if (f->double_colon)
(gdb) 
1977              if (cmds != 0 && cmds == f->cmds)
(gdb) 
1995              f->is_target = 1;
(gdb) 
1998              if (f == default_file && this == 0 && cmds == 0)
(gdb) 
2005              if (f == suffix_file && this == 0)
(gdb) print f->name
$3 = 0x5555555932d0 "makefiles"
(gdb) print f
$4 = (struct file *) 0x555555594a40
(gdb) ptype f
type = struct file {
    char *name;
    char *hname;
    char *vpath;
    struct dep *deps;
    struct commands *cmds;
    int command_flags;
    char *stem;
    struct dep *also_make;
    uintmax_t last_mtime;
    uintmax_t mtime_before_update;
    struct file *prev;
    struct file *last;
    struct file *renamed;
    struct variable_set_list *variables;
    struct variable_set_list *pat_variables;
    struct file *parent;
    struct file *double_colon;
    short update_status;
    enum cmd_state command_state : 2;
    unsigned int precious : 1;
    unsigned int low_resolution_time : 1;
    unsigned int tried_implicit : 1;
    unsigned int updating : 1;
    unsigned int updated : 1;
    unsigned int is_target : 1;
    unsigned int cmd_target : 1;
    unsigned int phony : 1;
    unsigned int intermediate : 1;
    unsigned int secondary : 1;
    unsigned int dontcare : 1;
    unsigned int ignore_vpath : 1;
    unsigned int pat_searched : 1;
    unsigned int considered : 1;
}
```

Note-se o uso da função ``expand_deps()``, que procura por arquivos dentro da
pasta que estejam dentro da lista de alvos utilizando a ``lookup_file()``
Esses nomes de arquivos ficam guardados dentro de uma ``struct`` do tipo ``dep``
chamada ``d1``, num formado de lista encadeada. O Make, então, verifica se o
alvo já existe e, se não, ele guarda o nome do alvo numa outra lista chamada
``files`` em forma de _hash_ e, posteriormente, executa a regra do sufixo em cada
um para que sejam criados.  
Pode-se ver isso ao executar o programa ``gmake`` utilizando o ``gdb`` e marcando
pontos de interrupção na execução das funções ``snap_deps()``, ``expand_deps()``,
``lookup_file()`` e ``enter_file()``.  

**_Backtrace_ do GDB:**

```c++
expand_deps (f=f@entry=0x555555594a40) at file.c:584
584               d1->name = 0;
(gdb) 
577           for (d1 = new; d1 != 0; d1 = d1->next)
(gdb) 
579               d1->file = lookup_file (d1->name);
(gdb) 

Breakpoint 3.1, lookup_file (name=0x555555591510 "chroot/Makefile") at file.c:77
77      {
(gdb) 
84        assert (*name != '\0');
(gdb) 

Breakpoint 3.2, lookup_file (name=0x555555591510 "chroot/Makefile") at file.c:105
105       while (name[0] == '.' && name[1] == '/' && name[2] != '\0')
(gdb) 
125       file_key.hname = name;
(gdb) 
126       f = (struct file *) hash_find_item (&files, &file_key);
(gdb) 
lookup_file (name=<optimized out>) at file.c:131
131       return f;
(gdb) 
expand_deps (f=f@entry=0x555555594a40) at file.c:580
580               if (d1->file == 0)
(gdb) 
581                 d1->file = enter_file (d1->name);
(gdb) 

Breakpoint 4, enter_file (name=0x555555591510 "chroot/Makefile") at file.c:136
136     {
(gdb) 
145       assert (*name != '\0');
(gdb) 
167       file_key.hname = name;
(gdb) 
168       file_slot = (struct file **) hash_find_slot (&files, &file_key);
(gdb) 
170       if (! HASH_VACANT (f) && !f->double_colon)
(gdb) 
179       new = (struct file *) xmalloc (sizeof (struct file));
(gdb) 
180       bzero ((char *) new, sizeof (struct file));
(gdb) 
181       new->name = new->hname = name;
(gdb) 
186           new->last = new;
(gdb) 
187           hash_insert_at (&files, new, file_slot);
(gdb) 
```

A contatenação em si é feita com uma função chamada ``convert_to_pattern()``,
como pode ser visto no ``gdb`` após o término da execução da função
``snap_deps()``, que é executada antes da ``install_default_implicit_rules()``
para que os sufixos definidos no Makefile tenham prioridade sob os padrões.

**_Backtrace_ do GDB:**

```c++
main (argc=<optimized out>, argv=<optimized out>, envp=<optimized out>) at main.c:1763
1763      convert_to_pattern ();
(gdb) 
1770      install_default_implicit_rules ();
(gdb) 
1774      count_implicit_rule_limits ();
(gdb) 
1778      build_vpath_lists ();
(gdb) 
1784      if (old_files != 0)
(gdb) 
1794      if (!restarts && new_files != 0)
(gdb) 
1804      remote_setup ();
(gdb) 
1806      if (read_makefiles != 0)
(gdb) 
1814          int orig_db_level = db_level;
(gdb) 
1817          if (! ISDB (DB_MAKEFILES))
(gdb) 
1818            db_level = DB_NONE;
(gdb) 
1828            while (d != 0)
(gdb) 
1831                if (f->double_colon)
(gdb) 
1863                    makefile_mtimes = (FILE_TIMESTAMP *)
(gdb) 
1866                    makefile_mtimes[mm_idx++] = file_mtime_no_search (d->file);
(gdb) 
1868                    d = d->next;
(gdb) 
1828            while (d != 0)
(gdb) 
1874          define_makeflags (1, 1);
(gdb) 
1876          rebuilding_makefiles = 1;
(gdb) 
1877          status = update_goal_chain (read_makefiles);
(gdb)
```

_"Tá, mas por que ele não faz isso com outras regras além do ``makefiles``? Tem a
``ou8``, ``ps2diet``, ``world``..."_ Pois então, é "tentativa e erro", o Make
verifica se o arquivo com o sufixo existe ou não antes de executar a regra do
sufixo.

**_Backtrace_ do GDB:**

```c++
2146        if (goals == 0)
(gdb) 
2148            if (**default_goal_name != '\0')
(gdb) 
2150                if (default_goal_file == 0 ||
(gdb) 
2178                goals = alloc_dep ();
(gdb) 
2179                goals->file = default_goal_file;
(gdb) 
2196        DB (DB_BASIC, (_("Updating goal targets....\n")));
(gdb) 
2198        switch (update_goal_chain (goals))
(gdb) 

Breakpoint 3.1, lookup_file (name=name@entry=0x5555555a03e0 "dummy.mk") at file.c:77
77      {
(gdb) 
84        assert (*name != '\0');
(gdb) 

Breakpoint 3.2, lookup_file (name=0x5555555a03e0 "dummy.mk") at file.c:105
105       while (name[0] == '.' && name[1] == '/' && name[2] != '\0')
(gdb) 
125       file_key.hname = name;
(gdb) 
126       f = (struct file *) hash_find_item (&files, &file_key);
(gdb) 
lookup_file (name=name@entry=0x5555555a03e0 "dummy.mk") at file.c:131
131       return f;
(gdb) 
pattern_search (file=file@entry=0x555555592fc0, archive=archive@entry=0, depth=depth@entry=1, recursions=recursions@entry=0) at implicit.c:706
706                   vname = name;
(gdb) 
707                   if (vpath_search (&vname, (FILE_TIMESTAMP *) 0))
(gdb) 
723                   if (intermed_ok)
(gdb) 
766               if (failed)
(gdb) 
771                   free_idep_chain (deps);
(gdb) 
772                   deps = 0;
(gdb) 
781           if (i < nrules)
(gdb) 
433       for (intermed_ok = 0; intermed_ok == !!intermed_ok; ++intermed_ok)
(gdb) 
439           for (i = 0; i < nrules; i++)
(gdb) 
452               if (rule == 0)
(gdb) 
457               if (intermed_ok && rule->terminal)
(gdb) 
462               rule->in_use = 1;
(gdb) 
466               stem = filename
(gdb) 
468               stemlen = namelen - rule->lens[matches[i]] + 1;
(gdb) 
469               check_lastslash = checked_lastslash[i];
(gdb) 
470               if (check_lastslash)
(gdb) 
476               DBS (DB_IMPLICIT, (_("Trying pattern rule with stem `%.*s'.\n"),
(gdb) 
479               strncpy (stem_str, stem, stemlen);
(gdb) 
480               stem_str[stemlen] = '\0';
(gdb) 
484               file->stem = stem_str;
(gdb) 
488               for (dep = rule->deps; dep != 0; dep = dep->next)
(gdb) 
503                   p = get_next_word (dep->name, &len);
(gdb) 
510                       if (p == 0)
(gdb) 
515                       for (p2 = p; p2 < p + len && *p2 != '%'; ++p2)
(gdb) 
518                       if (dep->need_2nd_expansion)
(gdb) 
566                            if (p2 < p + len)
(gdb) 
568                               register unsigned int i = p2 - p;
(gdb) 
569                               bcopy (p, depname, i);
(gdb) 
570                               bcopy (stem_str, depname + i, stemlen);
(gdb) 
571                               bcopy (p2 + 1, depname + i + stemlen, len - i - 1);
(gdb) 
572                               depname[len + stemlen - 1] = '\0';
(gdb) 
574                               if (check_lastslash)
(gdb) 
585                            p2 = depname;
(gdb) 
594                           for (; *id_ptr; id_ptr = &(*id_ptr)->next)
(gdb) 
597                           *id_ptr = (struct idep *)
(gdb) 
608                           if (order_only || add_dir || had_stem)
(gdb) 
610                               unsigned long l = lastslash - filename + 1;
(gdb) 
612                               for (d = *id_ptr; d != 0; d = d->next)
(gdb) 
614                                   if (order_only)
(gdb) 
617                                   if (add_dir)
(gdb) 
629                                   if (had_stem)
(gdb) 
630                                     d->had_stem = 1;
(gdb) 
612                               for (d = *id_ptr; d != 0; d = d->next)
(gdb) 
634                           if (!order_only && *p2)
(gdb) 
645                       p = get_next_word (p, &len);
(gdb) 
510                       if (p == 0)
(gdb) 
488               for (dep = rule->deps; dep != 0; dep = dep->next)
(gdb) 
651               file->stem = 0;
(gdb) 
656               for (d = deps; d != 0; d = d->next)
(gdb) 
658                   char *name = d->name;
(gdb) 
660                   if (file_impossible_p (name))
(gdb) 
676                   DBS (DB_IMPLICIT,
(gdb) 
685                   for (expl_d = file->deps; expl_d != 0; expl_d = expl_d->next)
(gdb) 
686                     if (streq (dep_name (expl_d), name))
(gdb) 
685                   for (expl_d = file->deps; expl_d != 0; expl_d = expl_d->next)
(gdb) 
686                     if (streq (dep_name (expl_d), name))
(gdb) 
685                   for (expl_d = file->deps; expl_d != 0; expl_d = expl_d->next)
(gdb) 
699                   if (((f = lookup_file (name)) != 0 && f->is_target)
(gdb) 

Breakpoint 3.1, lookup_file (name=name@entry=0x5555555946c0 "dummy.mk") at file.c:77
77      {
(gdb) 
84        assert (*name != '\0');
(gdb) 

Breakpoint 3.2, lookup_file (name=0x5555555946c0 "dummy.mk") at file.c:105
105       while (name[0] == '.' && name[1] == '/' && name[2] != '\0')
(gdb) 
125       file_key.hname = name;
(gdb) 
126       f = (struct file *) hash_find_item (&files, &file_key);
(gdb) 
lookup_file (name=name@entry=0x5555555946c0 "dummy.mk") at file.c:131
131       return f;
(gdb) 
pattern_search (file=file@entry=0x555555592fc0, archive=archive@entry=0, depth=depth@entry=1, recursions=recursions@entry=0) at implicit.c:706
706                   vname = name;
(gdb) 
707                   if (vpath_search (&vname, (FILE_TIMESTAMP *) 0))
(gdb) 
723                   if (intermed_ok)
(gdb) 
725                       if (intermediate_file == 0)
(gdb) 
726                         intermediate_file
(gdb) 
729                       DBS (DB_IMPLICIT,
(gdb) 
733                       bzero ((char *) intermediate_file, sizeof (struct file));
(gdb) 
734                       intermediate_file->name = name;
(gdb) 
735                       if (pattern_search (intermediate_file,
(gdb) 
752                       if (intermediate_file->variables)
(gdb) 
753                         free_variable_set (intermediate_file->variables);
(gdb) 
754                       file_impossible (name);
(gdb) 
766               if (failed)
(gdb) 
771                   free_idep_chain (deps);
(gdb) 
772                   deps = 0;
(gdb) 
781           if (i < nrules)
(gdb) 
433       for (intermed_ok = 0; intermed_ok == !!intermed_ok; ++intermed_ok)
(gdb) 
789       if (rule == 0)
(gdb) 
978       free_idep_chain (deps);
(gdb) 
979       free (tryrules);
(gdb) 
981       return rule != 0;
(gdb) 
```

Executando com o gdb, vemos que, quando o arquivo existe, "``failed``" é 
redefinido como falso e o arquivo é "atualizado" utilizando a função
``remake_file()``, mas apenas se estiver na lista de dependências (da
qual falamos antes). Em resumo, quando tudo dá certo, essa é a saída que
deveremos ter:

**_Backtrace_ do GDB:**

```c++
750       DBF (DB_BASIC, _("Must remake target `%s'.\n"));
(gdb) 
754       if (!streq(file->name, file->hname))
(gdb) 
761       remake_file (file);
(gdb) 
make: Entering directory `/usr/home/luiz/projetos/heirloom-toolchest'
cat build/mk.head build/mk.config Makefile.mk build/mk.tail >Makefile
[Detaching after vfork from child process 16777]
```

Isso também apareceu pelo ``strace``, mas de forma menos clara. Podemos ver que,
em vários momentos, chamadas do ``fstat``(2) são feitas para se obter
informações sobre a existência do arquivo e data de modificação:

```console
S145% pg /tmp/strace\ make\ Heirloom.txt
# [...]
newfstatat(AT_FDCWD, "RCS", 0x7ffe8ba615e0, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "SCCS", 0x7ffe8ba615e0, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "makefile", {st_mode=S_IFREG|0644, st_size=4164, ...}, 0) = 0
clock_gettime(CLOCK_REALTIME, {tv_sec=1713909508, tv_nsec=329059299}) = 0
newfstatat(AT_FDCWD, "dummy", 0x7ffe8ba5f4a0, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "makefiles", 0x7ffe8ba5f4a0, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "Makefile", 0x7ffe8ba5f3a0, 0) = -1 ENOENT (No such file or directory)
newfstatat(AT_FDCWD, "Makefile.mk", {st_mode=S_IFREG|0644, st_size=3438, ...}, 0) = 0
fstat(1, {st_mode=S_IFREG|0644, st_size=7226, ...}) = 0
write(1, "cat build/mk.head build/mk.confi"..., 70cat build/mk.head build/mk.config Makefile.mk build/mk.tail >Makefile
) = 70
pipe2([3, 4], 0)                        = 0
close(4)                                = 0
:█
```
  
Entretanto, caso eu não tivesse feito esse registro utilizando o ``gdb`` e o
``strace``, eu faria uma boa aposta que é pelo fato de, além deles não terem
nenhum correspondente com o sufixo como, por exemplo, "``world.mk``", eles
também não têm nenhuma dependência listada, ao contrário do ``makefiles``.
Eu já teria acertado apenas pela primeira, de qualquer forma, mesmo a segunda
não sendo o motivo.
 
Se isso tudo ainda lhe é um bocado confuso, creio que agora fica um pouco
mais claro quando executa-se o programa ``gmake`` com a opção ``--trace``:

```console
S145% gmake --trace &> '/tmp/trace make Heirloom.txt'
S145% pg '/tmp/trace make Heirloom.txt'
makefile:31: update target 'Makefile' due to: target does not exist
cat build/mk.head build/mk.config Makefile.mk build/mk.tail >Makefile
makefile:31: update target 'build/Makefile' due to: target does not exist
cat build/mk.head build/mk.config build/Makefile.mk build/mk.tail >build/Makefile
makefile:31: update target 'libwchar/Makefile' due to: target does not exist
cat build/mk.head build/mk.config libwchar/Makefile.mk build/mk.tail >libwchar/Makefile
makefile:31: update target 'libcommon/Makefile' due to: target does not exist
cat build/mk.head build/mk.config libcommon/Makefile.mk build/mk.tail >libcommon/Makefile
makefile:31: update target 'libuxre/Makefile' due to: target does not exist
cat build/mk.head build/mk.config libuxre/Makefile.mk build/mk.tail >libuxre/Makefile
makefile:31: update target '_install/Makefile' due to: target does not exist
cat build/mk.head build/mk.config _install/Makefile.mk build/mk.tail >_install/Makefile
makefile:31: update target 'banner/Makefile' due to: target does not exist
cat build/mk.head build/mk.config banner/Makefile.mk build/mk.tail >banner/Makefile
makefile:31: update target 'basename/Makefile' due to: target does not exist
cat build/mk.head build/mk.config basename/Makefile.mk build/mk.tail >basename/Makefile
makefile:31: update target 'bc/Makefile' due to: target does not exist
cat build/mk.head build/mk.config bc/Makefile.mk build/mk.tail >bc/Makefile
makefile:31: update target 'bdiff/Makefile' due to: target does not exist
cat build/mk.head build/mk.config bdiff/Makefile.mk build/mk.tail >bdiff/Makefile
makefile:31: update target 'bfs/Makefile' due to: target does not exist
cat build/mk.head build/mk.config bfs/Makefile.mk build/mk.tail >bfs/Makefile
makefile:31: update target 'cal/Makefile' due to: target does not exist
cat build/mk.head build/mk.config cal/Makefile.mk build/mk.tail >cal/Makefile
makefile:31: update target 'calendar/Makefile' due to: target does not exist
cat build/mk.head build/mk.config calendar/Makefile.mk build/mk.tail >calendar/Makefile
makefile:31: update target 'cat/Makefile' due to: target does not exist
cat build/mk.head build/mk.config cat/Makefile.mk build/mk.tail >cat/Makefile
:█
```

Veem? É claro que o programa nos diz "``makefile:31: update target 'Makefile'
due to: target does not exist``" antes de criá-lo pelo ``remake_file()``.

Agora que já sabemos que para adicionar algo novo no projeto como, por exemplo,
uma suite de testes, é melhor criar um arquivo independente com instruções
para aquela pasta e só adicionar em ``SUBDIRS`` para aquele ``Makefile.mk``
possa virar um ``Makefile`` definitivo e... ``$(MAKE)``? ``.DEFAULT``?

**Linhas 23 até 28, [arquivo ``makefile``](https://github.com/Projeto-Pindorama/heirloom-ng/blob/master/makefile):**

```makefile
.DEFAULT:
        + for i in $(SUBDIRS) ;\
        do      \
                (cd "$$i" && $(MAKE) $@) || exit ; \
        done
        $(MAKE) -f Makefile $@
```

Pois bem, presumo que essa parte será a mais simples de explicar até agora.  
Basicamente, o ``.DEFAULT`` é um alvo especial, assim como o ``.SUFFIXES``
e outros cujos nomes são precedidos por um ponto, que é usado para definir
comandos que serão executados pelo Make caso não hajam outras regras definidas
para se montar um determinado alvo. É literalmente uma "regra padrão", mais um
caso de nome autodescritivo.  

**Linhas 477 até 481 e 1583 até 1586, [arquivo ``main.c``](https://github.com/mirror/make/blob/3.81/main.c):**

```c
/* Pointer to structure for the file .DEFAULT
   whose commands are used for any file that has none of its own.
   This is zero if the makefiles do not define .DEFAULT.  */

struct file *default_file;
```
```c
  /* Define the default variables.  */
  define_default_variables ();

  default_file = enter_file (".DEFAULT");
```

Nesse caso, o ``.DEFAULT`` executa um laço _for_ em todas as pastas que estão
em ``SUBDIRS`` --- inclusive, a confusão mais comum é pensar que o
``Makefile.mk`` era transformado em ``Makefile`` por meio desse laço, o que se
mostrou não ser verdade agora há pouco ---, esse que entra em cada pasta e
executa o ``$(MAKE)`` e, em seguida, ao fim do laço,
``$(MAKE) -f Makefile $@``. A partir daqui, vamos por partes para explicar o
que é o "``Makefile``" e o que é o ``$(MAKE)``.  
Antes de tudo, lembremos que, pela ordem de prioridade, o arquivo ``makefile``
sempre será executado antes de qualquer outro, então o ``Makefile`` precisaria
ser explicitado que o Make o executasse e, claro, o ``makefile`` já faz isso por
si só, como vimos.  

Certo, mas o que é o ``$(MAKE)``? Em resumo, é uma forma do Make "se chamar"
recursivamente, mas com os mesmos parâmetros.
Não precisaria se ir muito longe para se ver como funciona, apenas limpar
a árvore de código-fonte do Heirloom depois de uma primeira compilação:

```console
S145% gmake mrproper
rm -f .foo .FOO Makefile
for i in build libwchar libcommon libuxre _install banner basename bc bdiff bfs cal calendar cat chmod chown chroot cksum cmp col comm copy cp cpio csplit cut date dc dd deroff diff diff3 dircmp dirname df du echo ed env expand expr factor file find fmt fmtmsg fold getconf getopt grep groups hd head hostname id join kill line listusers ln logins logname ls mail man mesg mkdir mkfifo mknod more mvdir nawk news nice nl nohup oawk od paste pathchk pg pgrep pr printenv printf priocntl ps psrinfo pwd random renice rm rmdir sdiff sed setpgrp shl sleep sort spell split stty su sum sync tabs tail tapecntl tar tcopy tee test time touch tr true tsort tty ul uname uniq units users wc what who whoami whodo xargs yes ;\
do      \
        (cd "$i" && gmake mrproper) || exit ; \
done
gmake[1]: Entering directory '/usr/home/luiz/projetos/heirloom-toolchest/build'
rm -f core log *~
rm -f Makefile maninst crossln genintro
gmake[1]: Leaving directory '/usr/home/luiz/projetos/heirloom-toolchest/build'
gmake[1]: Entering directory '/usr/home/luiz/projetos/heirloom-toolchest/libwchar'
# [...]
```

Achou que eu não ia buscar como isso é feito pelo próprio GNU Make no
código-fonte? Achou errado. Vamos lá de novo.

```console
S145% find /usr/src/make-3.81/ -type f -name '*.c' -print | xargs grep -n '$(MAKE)'           
/usr/src/make-3.81/commands.c: 355:          && (strstr (p, "$(MAKE)") != 0 || strstr (p, "${MAKE}") != 0))
/usr/src/make-3.81/main.c: 1128:     done before $(MAKE) is figured out so its definitions will not be
```

O que temos aqui... O arquivo ``commands.c``, que define comandos, variáveis e
"macros" internos do GNU Make e o que parece ser um comentário no arquivo
``main.c``. Vamos analisar o ``commands.c`` primeiro, porque tem umas coisas
interessantes a se notar.  

**Linhas 353 até 356, [arquivo ``commands.c``](https://github.com/mirror/make/blob/3.81/commands.c):**

```c
      /* If no explicit '+' was given, look for MAKE variable references.  */
      if (!(flags & COMMANDS_RECURSE)
          && (strstr (p, "$(MAKE)") != 0 || strstr (p, "${MAKE}") != 0))
        flags |= COMMANDS_RECURSE;
```

Bem, primeiramente se verifica se o Make já não estaria sendo executado
recursivamente ao se verificar se ``COMMANDS_RECURSE`` já não está presente no
valor de ``flags`` e, então, se procuraria por "``${MAKE}``"  ou ``$(MAKE)`` ---
que é mais comumente usado --- na _string_ ``p`` utilizando a função
``strstr()``. Para quem não está acostumado, essa função é o equivalente de se
usar as expressões regulares embutidas de KornShell 93 (ou do GNU Broken-Again)
para encontrar uma _string_ dentro d'outra, a exemplo:

```sh
if [[ "$p" =~ (.*\$[\(,\{]MAKE[\),\}].*) ]]; then
```

Isso é mais uma curiosidade e não seria muito relevante de se comentar a fundo;
entretanto, é de se notar que o Make apenas aumenta o valor de um inteiro para
dizer que se trata de uma recursão e nada mais, o que é interessante porque nos
dá motivo para ver onde esse inteiro é usado no código do programa e como ele
atua. A menção do "$(MAKE)" no arquivo ``main.c`` apenas nos diz que o programa
deve ler as variáveis de ambiente antes de chamar sua recursão para que as
próximas definições não venham do ambiente... Confuso, mas tranquilo porque não
é o que queremos saber por ora.  
Bem, onde mais essa ``COMMANDS_RECURSE`` é mencionada? Além do ``commands.c``,
também a vemos em ``job.c`` e ``remake.c``.  
Algo que é digno de alguma curiosidade é que a maioria das verificações por esse
valor são negativas, ou seja, "faça algo apenas se ``COMMANDS_RECURSE`` **não**
esteja definida". O que me chamou a atenção é que esse algo é, de fato, a
"limpeza" da variável ``argv[]``. Quem programa em C, Nawk, Perl, PHP, Python,
JavaScript, enfim, qualquer linguagem que tenha bebido muito da fonte de C,
saberá que essa variável contém os parâmetros que são passados para o programa.  
Em resumo, quando o Make não executa recursivamente, os parâmetros (alvos e
afins) passados a ele são limpos da memória.

**Linhas 1032 até 1079, [arquivo ``job.c``](https://github.com/mirror/make/blob/3.81/job.c):**

```c
  /* If -q was given, say that updating `failed' if there was any text on the
     command line, or `succeeded' otherwise.  The exit status of 1 tells the
     user that -q is saying `something to do'; the exit status for a random
     error is 2.  */
  if (argv != 0 && question_flag && !(flags & COMMANDS_RECURSE))
    {
#ifndef VMS
      free (argv[0]);
      free ((char *) argv);
#endif
      child->file->update_status = 1;
      notice_finished_file (child->file);
      return;
    }

  if (touch_flag && !(flags & COMMANDS_RECURSE))
    {
      /* Go on to the next command.  It might be the recursive one.
	 We construct ARGV only to find the end of the command line.  */
#ifndef VMS
      if (argv)
        {
          free (argv[0]);
          free ((char *) argv);
        }
#endif
      argv = 0;
    }

  if (argv == 0)
    {
    next_command:
#ifdef __MSDOS__
      execute_by_shell = 0;   /* in case construct_command_argv sets it */
#endif
      /* This line has no commands.  Go to the next.  */
      if (job_next_command (child))
	start_job_command (child);
      else
	{
	  /* No more commands.  Make sure we're "running"; we might not be if
             (e.g.) all commands were skipped due to -n.  */
          set_command_state (child->file, cs_running);
	  child->file->update_status = 0;
	  notice_finished_file (child->file);
	}
      return;
    }
```

Isso ainda se repete mais para frente, mas é uma função exclusiva do EMX, uma
extensão para o MS-DOS e o IBM OS/2 que mantém os descritores de arquivos dos
"trabalhos" do Make abertos caso seja chamado recursivamente, então acredito que
não caiba comentar muito além aqui.

Finalmente, conseguiremos chegar onde queríamos: para que serve o bendito
``Makefile`` gerado a partir do ``Makefile.mk``? E porque ele é chamado apenas
no final do laço _for_ que compila todos os programas e bibliotecas do Heirloom?  
Quando analisamos o arquivo ``Makefile.mk``, vemos que ele contém regras
específicas para criar a estrutura de pastas onde o Heirloom será instalado
--- essa que pode ser alterada em ``build/mk.config``, como já falamos
anteriormente ---, a ``directories``, e para fazer ligações no sistema de
arquivos, a ``links``, duas regras que são "escondidas" por uma
"regra-porcelana" chamada ``install``. Entretanto, como o ``makefile`` é o
primeiro arquivo a ser chamado, ele executa o ``Makefile`` dessa maneira:

**Linhas 35 até 41, [arquivo ``makefile``](https://github.com/Projeto-Pindorama/heirloom-ng/blob/master/makefile):**

```makefile
install:
	$(MAKE) -f Makefile directories
	+ for i in $(SUBDIRS) ;\
	do	\
		(cd "$$i" && $(MAKE) $@) || exit ; \
	done
	$(MAKE) -f Makefile links
```

Perceba-se que cada programa ainda tem sua própria regra para ser instalado,
afinal, como pode-se ver no próprio
[site do Heirloom](http://heirloom-ng.pindorama.dob.jp/#what), alguns têm
variantes específicas para cada especificação, logo precisariam ser instalados
em outro diretório além do padrão informado no ``build/mk.config`` como
``DEFBIN``.  
Primeiro, cria-se os diretórios, então instala-se cada programa a partir de sua
própria regra e, por fim, façam-se as ligações no sistema de arquivos.

Por padrão, caso não seja executado o alvo ``install``, esse arquivo meramente
gera o ``intro.1``, que é uma página de manual para o pacote Heirloom em si, ao
passar o arquivo de dependência ``intro.1.in`` para o _script_ em shell
[``genintro.sh``](https://github.com/Projeto-Pindorama/heirloom-ng/blob/master/build/genintro.sh).

**Linhas 1 até 4, [arquivo ``Makefile.mk``](https://github.com/Projeto-Pindorama/heirloom-ng/blob/master/Makefile.mk):**

```makefile
all: intro.1

intro.1: intro.1.in
	$(SHELL) build/genintro intro.1.in >intro.1
```

Ou seja, esse arquivo não faz nada, absolutamente nada de vital além de instalar
o pacote; o que é até justificável, considerando que o arquivo contém diversas
listas de comandos e adicionar tudo em um arquivo apenas seria deselegante e,
além de tudo, atrapalharia quem quisesse estudar o sistema de montagem por si só
ao aumentar o tamanho do ``makefile`` em pelo menos 90 linhas.

Bem, fico por aqui. Esse artigo me tomou dois dias de pesquisa e quebração de
cuca para entender o que estava acontecendo em diversos momentos, mas meu único
arrependimento foi não ter conseguido lançar ontem, Dia de São Jorge.  
O próximo passo vai ser implementar a suíte de testes, espero trazer algo novo
até lá.
