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

Isso se chama **tripla**. caso queira saber mais sobre [Grafos entre aqui](http://pt.wikipedia.org/wiki/Teoria_dos_grafos).

##Por que usar?

Um banco de dados de grafos facilita nossa vida quando precisamos conectar entidades do nosso sistema e nesse caso o Levelgraph é uma ótima escolha já que podemos utilizá-lo tanto no navegador como no servidor e é feito com Node.js por isso facilmente integrável.

Por exemplo quem usa MongoDb sabe que é muito custoso separar em várias coleções pois ele não possui JOIN, o cenário ideal seria deixar a modelagem real no MongoDB e usar o Levelgraph para criar a rede de conexões das entidades e que ele seja usado para esse tipo de busca.

Pegando nosso exemplo acima, vamos ver algumas buscas que podemos fazer em um banco de grafos:

> Busque todas pessoas que estudam Javascript
> Busque todas pessoas que estudam Javascript ensinado por Suissa
> Busque todas pessoas que ensinam Javascript

Isso apenas com uma conexão, imagine com diversas.

##Como usar?

Vamos começar instalando ele localmente na nossa aplicação.

```
npm install levelgraph --save
```

Ou para usar no navegador você pode baixar o [levelgraph.min.js](https://github.com/mcollina/levelgraph/blob/master/build/levelgraph.min.js).

Para iniciar uma database é bem simples:

```
var levelgraph = require("levelgraph"); // não precisa no navegador
var db = levelgraph("yourdb");
```

###Inserindo

Inserir uma conexão(tripla) é super simples, para isso usaremos a função `put`:

```
db.put(tripla, callback);
```

Como vemos no exemplo abaixo:

```
var triple = { sujeito: "a", predicado: "b", objeto: "c" };

db.put(triple, function(err) {
  if(err) console.log('ERROR: ', err);

  console.log('INSERIU');
  db.get({ sujeito: "a" }, function(err, list) {
    console.log(list);
  });
});
```

E quando rodamos no terminal recebemos a seguinte saída:

![log do objeto inserido](https://cldup.com/Wx8e7yJ8q0.png)

####Propriedades

Podemos adicionar mais propriedades na tripla dessa forma:

```
var triple =
  { sujeito: 'a'
  , predicado: 'b'
  , objeto: 'c'
  , novaPropriedade: 'teste'
  };

db.put(triple, function(err) {
  if(err) console.log('ERROR: ', err);

  console.log('INSERIU');
  db.get({ sujeito: 'a' }, function(err, list) {
    console.log(list);
  });
});

```

![](https://cldup.com/to9uAItuB0.png)

Além disso podemos criar objetos no sujeito, predicado e/ou objeto, como vemos abaixo:

```
var triple =
  { sujeito: {name: 'Suissa'}
  , predicado: 'ensina'
  , objeto: {linguagem: 'Javascript'}
  };

db.put(triple, function(err) {
  if(err) console.log('ERROR: ', err);

  console.log('INSERIU');
  db.get({ sujeito: {name: 'Suissa'} }, function(err, list) {
    console.log(list);
  });
});
```

![](https://cldup.com/EeNY1xA7v0.png)

###Consultando

Bom como vimos no exemplo anterior usamos a função `get`:

```
db.get({ subject: "a" }, function(err, list) {
  // Faz algo
});
```

Vamos inserir várias triplas e depois listá-las:

```
var triples =
  [ { subject: "Suissa"
    , predicate: "ensina"
    , object: "Javascript"
    }
  , { subject: "Odin"
    , predicate: "estuda"
    , object: "Javascript"
    }
  , { subject: "Thor"
    , predicate: "estuda"
    , object: "Javascript"
    }
  , { subject: "Loki"
    , predicate: "estuda"
    , object: "Javascript"
    }
  , { subject: "Heimdall"
    , predicate: "estuda"
    , object: "PHP"
    }
  , { subject: "Frey"
    , predicate: "estuda"
    , object: "PHP"
    }
  , { subject: "Galvão"
    , predicate: "ensina"
    , object: "PHP"
    }
  ]
  ;

db.put(triples, function(err) {
  if(err) console.log('ERROR: ', err);
  console.log('INSERIU');
  db.get({}, function(err, list) {
    console.log(list);
  });
});
```

![](https://cldup.com/QeaXDKYqSV.png)

Vamos consultar quais são nossos professores:

```
db.get({predicate: "ensina"}, function(err, list) {
  console.log(list);
});
```

![Listagem dos professores](https://cldup.com/jUQd-yys3R.png)

E quais são nossos alunos:

```
db.get({predicate: "estuda"}, function(err, list) {
  console.log(list);
});

```

![Listagem dos alunos](https://cldup.com/T7CfIhSDaN.png)

Legal mas e como que eu posso buscar todos que estudam Javascript?

Tão simples quanto a busca anterior, só precisamos passar mais um atributo no objeto da query:

```
db.get({predicate: "estuda", object: "Javascript"}, function(err, list) {
  console.log(list);
});
```

![Listagem dos alunos que estudam Javascript](https://cldup.com/qw2_kPgVju.png)



###Streams

Ele também suporta streams, vamos ver um exemplo de como buscar os amigos entre uma pessoa e outra:

```
db.put([{
  subject: "Suissa",
  predicate: "segue",
  object: "Galvão"
}, {
  subject: "Galvão",
  predicate: "segue",
  object: "Suissa"
}, {
  subject: "Galvão",
  predicate: "segue",
  object: "Eminetto"
}, {
  subject: "Piaz",
  predicate: "segue",
  object: "Suissa"
}, {
  subject: "Piaz",
  predicate: "segue",
  object: "Eminetto"
}, {
  subject: "Eminetto",
  predicate: "segue",
  object: "Duodraco"
}], function () {

  var stream = db.searchStream([{
    subject: "Suissa",
    predicate: "friend",
    object: db.v("x")
  }, {
    subject: db.v("x"),
    predicate: "friend",
    object: db.v("y")
  }, {
    subject: db.v("y"),
    predicate: "friend",
    object: "Duodraco"
  }]);

  stream.on("data", function(data) {
    console.log('Caminho percorrido: ', data);
  });
});
```

Para entendermos quem segue quem fiz esse exemplo visual:

![](https://cldup.com/VstqbL8F0Q.png)

Essa busca está procurando qual o caminho que preciso percorrer entre quem **Suissa** segue até chegar em quem segue o **Duodraco**, qualuqer semelhança com aquela feature *Pessoas que você possa conhecer* não é mera coincidência.

Rodando essa busca no terminal temos:

![](https://cldup.com/EW6eY_Gt7K.png)

###Caminhando

Essa API de caminhada é baseada no [Gremlin](http://markorodriguez.com/2011/06/15/graph-pattern-matching-with-gremlin-1-1/).

Para termos maior facilidade nas buscas que retornam um caminho podemos usar as funções `archIn` e `archOut`.

Re-usando aquele exemplo do ensina/estuda Javascript, para sabermos todas pessoas que estudam Javascript basta fazermos:

```
db.nav("Javascript")
  .archIn("estuda")
  .solutions(function(err, results) {
  // prints:
  // [ { x0: 'Loki' }, { x0: 'Odin' }, { x0: 'Thor' } ]
  console.log(results);
});
```

Mas vamos voltar ao nosso exemplo dos seguidores, a busca mais simples é achar as pessoas que seguem o **Suissa**:

```
db.nav("Suissa")
  .archIn("segue")
  .solutions(function(err, results) {
  // prints:
  // [ { x0: 'Galvão' }, { x0: 'Piaz' } ]
  console.log(results);
});
```

Então a função `archIn` é a responsável pela predicado que chega na entidade setada em `db.nav()`.

Logo a função `archOut` ´a responsável pelo predicado que sai da entidade, então vamos buscar quem o **Suissa** segue:

```
db.nav("Suissa")
  .archOut("segue")
  .solutions(function(err, results) {
  // prints:
  // [ { x0: 'Galvão' } ]
  console.log(results);
});
```

E se quisermos saber os seguidores dos seguidores do **Suissa**?

Muito fácil, como visto abaixo:

```
db.nav("Suissa")
  .archOut("segue")
  .archOut("segue")
  .solutions(function(err, results) {
  // prints:
  // [ { x0: 'Galvão', x1: 'Eminetto' },
  // { x0: 'Galvão', x1: 'Suissa' } ]
  console.log(results);
});
```

Só precisamos adicionar outra chamada para `.archOut("segue")` e assim sucessivamente para aumentar o grau da busca.

Ai você se pergunta: **o que que acontece se eu adicionar um archIn ali no final?**

Ótima pergunta! Vamos ver então:

```
db.nav("Suissa")
  .archOut("segue")
  .archOut("segue")
  .archIn("segue")
  .solutions(function(err, results) {
  // prints:
  // [ { x0: 'Galvão', x1: 'Eminetto', x2: 'Galvão' },
  // { x0: 'Galvão', x1: 'Eminetto', x2: 'Piaz' },
  // { x0: 'Galvão', x1: 'Suissa', x2: 'Galvão' },
  // { x0: 'Galvão', x1: 'Suissa', x2: 'Piaz' } ]
  console.log(results);
});
```

Nesse caso nossa busca é o seguinte: 

- inicie em Suissa
- busque um objeto que ele se conecte via "segue" // Galvão
- busque um objeto que ele se conecte via "segue" // Eminetto
- busque um objeto que ele receba conexão via "segue" // Galvão, Piaz

Destrinchando oresultado um pouco mais:

```
{ x0: 'Galvão', x1: 'Eminetto', x2: 'Galvão' }
```

> Suissa segue Galvão, Galvão segue Eminetto, Galvão é seguido por Suissa


```
{ x0: 'Galvão', x1: 'Eminetto', x2: 'Piaz' }
```

> Suissa segue Galvão, Galvão segue Eminetto, Eminetto é seguido por Piaz


```
{ x0: 'Galvão', x1: 'Suissa', x2: 'Galvão' }
```

> Suissa segue Galvão, Galvão segue Suissa, Suissa é seguido por Galvão


```
{ x0: 'Galvão', x1: 'Suissa', x2: 'Piaz' }
```

> Suissa segue Galvão, Galvão segue Suissa, Suissa é seguido por Piaz

Creio que deu para notar que ele vai caminhando e aplicando a busca, como vimos na última busca que é um `archIn` nosso *"caminhador"* para na entidada achada no último `archOut` e executa o `archIn` por isso achando por quem essa entidade **é seguido**.










