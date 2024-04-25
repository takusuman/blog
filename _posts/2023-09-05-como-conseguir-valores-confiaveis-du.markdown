---
layout: post
title: "Como conseguir valores confiáveis no du(1)?"
description: "Uma desventura por implementações."
author: "Luiz Antônio"
---

Recentemente, eu estive gerenciando alguns *backups* de DVDs de vídeo na
produtora onde trabalho, pois serão usados no documentário "Escritório
Etílico", com Carlo Mossy, que será lançado em breve e ainda está no
processo de montagem.





Como de costume, estou fazendo pela interface de linha de comando no Linux, e
nisso percebi algo engraçado: o mesmo arquivo tinha dois tamanhos diferentes em
gigabytes.

{% highlight sh %}
nawk '{ printf("%0.2f GB\n", ($1 / ((1024 ** 2) * 2))) }'
{% endhighlight %}
