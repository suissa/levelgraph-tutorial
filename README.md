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

Inserir uma conexão(tripla) é tão simples quanto:

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
var levelgraph = require("levelgraph");

db = levelgraph("graph-teste");
db.put([{
  subject: "matteo",
  predicate: "friend",
  object: "daniele"
}, {
  subject: "daniele",
  predicate: "friend",
  object: "matteo"
}, {
  subject: "daniele",
  predicate: "friend",
  object: "marco"
}, {
  subject: "lucio",
  predicate: "friend",
  object: "matteo"
}, {
  subject: "lucio",
  predicate: "friend",
  object: "marco"
}, {
  subject: "marco",
  predicate: "friend",
  object: "davide"
}], function () {

  var stream = db.searchStream([{
    subject: "matteo",
    predicate: "friend",
    object: db.v("x")
  }, {
    subject: db.v("x"),
    predicate: "friend",
    object: db.v("y")
  }, {
    subject: db.v("y"),
    predicate: "friend",
    object: "davide"
  }]);

  stream.on("data", function(data) {
    console.log(data);
  });
});
```

Nesse caso a ordem de amizades é a seguinte:

matteo -> daniele -> marco -> davide

E a resposta é:

![](https://cldup.com/Bs04jakdGc.png)


















