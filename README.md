#Levelgraph

##O que é?

É um banco de dados de grafos diferentemente dos outros bancos de grafos ele feito em cima do rapidíssimo [LevelDB](http://code.google.com/p/leveldb/), você pode usar ele dentro da sua aplicação Node.js ou no navegador!

![](https://cldup.com/wvRSezLXzR.png)

A imagem a cima é um exemplo de um grafo onde temos:

- A: sujeito
- B: objeto
- C: predicado

**A** é a entidade de onde parte uma conexão.
**B** é a entidade que recebe a conexão.
**C** é a conexão em si.

> Suissa(A) ensina(C) Javscript(B).
> João(A) estuda(C) Javascript(B).

Caso queira saber mais sobre [Grafos entre aqui](http://pt.wikipedia.org/wiki/Teoria_dos_grafos).

##Por que usar?

Um banco de dados de grafos facilita nossa vida quando precisamos conectar entidades do nosso sistema e nesse caso o Levelgraph é uma ótima escolha já que podemos utilizá-lo tanto no navegador como no servidor e é feito com Node.js por isso facilmente integrável.

Por exemplo quem usa MongoDb sabe que é muito custoso separar em várias coleções pois ele não possui JOIN, o cenário ideal seria deixar a modelagem real no MongoDB e usar o Levelgraph para criar a rede de conexões das entidades e que ele seja usado para esse tipo de busca.

Pegando nosso exemplo acima, vamos ver algumas buscas que podemos fazer em um banco de grafos:

> Busque todas pessoas que estudam Javascript
> Busque todas pessoas que estudam Javascript ensinado por Suissa
> Busque todas pessoas que ensinam Javascript

Isso apenas com uma conexão imagine com diversas

##Como usar?


