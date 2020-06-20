# Recupere e exiba o primeiro planeta
Nesta sessão continuaremos com o projeto "Exoplanet Explorer". Tambem usaremos diferentes tecnicas para criar correntes de trabalhos assincronos. Esses trabalho é raramente isolado e, desta forma, muitas ações assicronas dependem umas das outras. Este é o serviço dos promises. Em vez de criar uma "piramide amaldiçoada" ("pyramid of doom" - problema comum gerado a partir de um programa com muitos níveis de recuo aninhado para controlar o acesso a uma função.), promises simplificam a junção de varias ações assincronas. Elas funcionam até mesmo com ações programaticas.

Para este quiz, antes de tudo, mudamos para o branch `first-thumb-start` e naveguamos para `app/scripts/app.js`.

```
git checkout first-thumb-start

```
O codigo que adicionamos exibe uma thumbnail do primeiro planeta:

```
function createPlanetThumb(data) {
    var pT = document.createElement('planet-thumb');
    for (var d in data) {
      pT[d] = data[d];
    }
    home.appendChild(pT);
}

window.addEventListener('WebComponentsReady', function() {
    home = document.querySelector('section[data-route="home"]');

    getJSON('../data/earth-like-results.json')
    .then(function(response) {
      addSearchHeader(response.query);
      console.log(response)
      return getJSON(response.results[0])
    })
    .catch(function() {
      throw Error('Search Request Error')
    })
    .then(function(planetData) {
      createPlanetThumb(planetData)
    })
    .catch(function(e) {
      addSearchHeader('unknown')
      console.log(e)
    })
});

```

A primeira coisa que mudamos, comparado a sessão anterior, foi implementar a ultima linha do primeiro `.then`, isto é, agora retornamos a resposta do segundo `getJSON` atraves de `return getJSON(response.results[0])`. Ele pega a URL do primeiro planeta. Ao retorna-lo, ele é passado ao proximo `.then` (`.then(function(planetData) {createPlanetThumb(planetData)})`). Ao receber dados do pleneta (`planetData`), ele cria uma miniatura atraves de `createPlanetThumb(planetData)`. Alem disso, temos dois `.catch`. Um para erros nos resultados de busca, outro para qualquer outro erro.

Antes de prosseguirmos, vejamos uma sintaxe diferente.

```
function createPlanetThumb(data) {
    var pT = document.createElement('planet-thumb');
    for (var d in data) {
      pT[d] = data[d];
    }
    home.appendChild(pT);
}

window.addEventListener('WebComponentsReady', function() {
    home = document.querySelector('section[data-route="home"]');

    getJSON('../data/earth-like-results.json')
    .then(function(response) {
      addSearchHeader(response.query);
      console.log(response)
      return getJSON(response.results[0])
    })
    .catch(function() {
      throw Error('Search Request Error')
    })
    .then(createPlanetThumb) // Alternado
    .catch(function(e) {
      addSearchHeader('unknown')
      console.log(e)
    })
});

```

Em vez de uma função anonima (`.then(function(planetData) {createPlanetThumb(planetData)})`), podemos usar a função `.then(createPlanetThumb)`, que receberá o mesmo argumento que a função anônima recebia, mesmo sem especicarmos isso.

O resultado disso esta ilustrado na imagem "exoplanet-thumb.png".

# Estratégias de gerenciamento de erros
Até agora, o tratamamento de erros apareceu como `.catches`, como no exemplo abaixo, mas há outras formas:

```
get('example.json')
.then(resolveFunc)
.catch(rejectFunc);

```

Uma das outras formas seria:

```
get('example.json')
.then(resolveFunc)
.catch(undefined, rejectFunc);

```

Esses dois pedaços de código são equivalentes. `.catch` é apenas uma abreviação de `.then(undefined)` seguido de uma função "rejection". Perceba como o `.then` da segundo exemplo recebe dois argumentos. A função completa para `.then` é, na verdade, esta:

```
get('example.json').then(resolveFunc, rejectFunc);

```

Nesta forma, se qualquer promise prévia for rejeitada, a função `rejectFunc` é chamada. Se ela for resolvida, a função `resolveFunc` é chamada. Se não houver uma função "resolve" (`get('example.json').then(rejectFunc)`) e promise antes de deste `.then` for resolvida, então este `.then` é pulado e o proximo `.then` é chamado.

Em todos os casos, assim que uma promise é rejeitada, o mecanismo do JS pula para a proxima função "reject" na cadeia, seja ela uma `.catch`, ou uma `.then`. Isso significa que um erro em das promises destacadas na imagem "gerenciamento-de-erros.png" é pego pelo primeiro `.catch`. Ambos os metodos, `.catch` e `.then` com duas callbacks (destacados na imagem "gerenciamento-de-erros-2.png") funcionam igualmente bem. Porém, é recomendado que usemos `.catch` quando pudermos, porque `.catch` é mais facil de ler e de escrever do que uma segunda callback `.then`, que pode ser dificil de ser notada. Dito isso, há uma enorme diferença na ordem de execução entre `.catch` e da ultima callback da imagem.

Saiba que não podemos chamar as funções "resolve" e "reject" se elas forem parte do mesmo `.then` (ideia ilustrada na imagem "chamando-promises.png"). Apenas uma ou a outra, ou nenhuma, será chamada. Se algo der errado com a função "resolve", vamos precisar de outro `.catch` ou outro `.then` mais abaixo para pega-lo. Mas, se tivermos um `.then`, depois um `.catch`, cada um com suas proprias funções "resolve" ou "reject", ambas podem ser possivelmente chamadas.

Por fim, devemos ressaltar: é importante notar que não é necessariamente verdade que passar um valor para "resolve" significa que a promise foi bem sucedida.

# Thenables encadeadas
Existem diferenças sutis entre as estratégias e isso pode nos prejudicar se não formos cuidadosos. No exemplo da imagem "thens-e-catchs.png" vemos difentes cenarios de um longa cadeia. Normalmente não misturamos diferentes sintaxes como na imagem, mas devemos pensar em diferentes situações. No exemplo da imagem, `async()` retorna uma promise assim como `recovery()`. A ideia é que o metodo recovery alinhe a corrente e continue resolvendo se algo der errado.

Ainda alisando a mesma imagem, vamos ao "porquê" da resposta:

1. Se houver um erro no `async()`, a função reject será chamada (`.then(undefined, function(e) { console.log('1') return recovery() })`). O log será igual a 1. Se o recovery funcionar, a proxima função result, do proximo `.then`, será chamada ((`.then(function() { console.log('3') return async(urls[1]) })`) e o log será igual a 3.

2. Se um erro ocorrer em `urls = data.urls`, a proxima função será chamada (`.then(undefined, function(e) { console.log('1') return recovery() })`). O log será igual a 1. A função `recovery()` acontece e as coisas voltam ao rumo, e então a proxima função result, do proximo `.then`, será chamada ((`.then(function() { console.log('3') return async(urls[1]) })`) e o log será igual a 3. Depois disso não há mais conexões , então acabou.

3. Se houver um erro na função `recovery()` do unico `.catch`, algo interessante ocorrerá, porque a função `recovery()` só será chamada caso ha um outro erro, nesse caso nada será exibido no log.

4. Para o penultimo `.then`, se algo der errado com a função `async(urls[1])`, o log será igual a 3, e a proxima função reject será chamada. Neste caso, o numero 4 deverá aparecer na tela.

Isso pode ser dificil, mas se algo der errado, a proxima função reject será chamada.

# Séries x requisições paralelas
Quando precisamos efetuar um trabalho assincrono, o trabalho pode não estar isolado. Nós deveremos efetuar ações assincronas multiplas. Neste momento estamos no estagio "Chain" do curso, no qual criaremos cadeias de promises.

Existem duas estrategias para desempenhar ações assincronas multiplas. Há ações em **serie** e em **paralelo**. Ações em serie ocorrem uma após as outras, como se os tres gatos da imagem "req-serie-paralelo.png" esperassem pela sua vez no foguete. E as ações em paralelo ocorrem de forma simultanea, como se cada um dos gatos tivessem seu proprio foguete. O codigo assuncrono é sempre em série, e o assincrono pode ser em série ou em paralelo. De qualquer forma, em serie ou em paralelo, nenhum deles é melhor, cada um possui seu proposito.

No questionario (imagem "antigo-quiz.png") no qual obtivemos a lista JSON e requisitamos palnetas JSON individuais, executamos requisiçoes em série, pois um dependia do outro. Mas se tivermos que requisitar muitos planetas JSON, faremos uma requisição programatica. Tambem podemos requisitar em paralelo, pois isso reduzirá o tempo de carregamento dos dados.

No exemplo da imagem "problema-assincrono.png", há um problema. Ele parece repetir os URLs da consulta de planeta, mas algo inesperado acontece: As miniaturas são criadas de forma aleatória. Lembr-se, não podemos prever a ordem na qual a requisição retornará, então não sabemos quando as promises `getJSON(url)` se resolverão. Elas podem se resolver em ordem diferente da que foram criadas. Nesse caso, as miniaturas serão criadas de forma aleatoria. Isso não é necessariamente uma falha ou bug, mas leva a questão: Como fazemos para as thumbs aparecerem na ordem correta?

# Promises com .forEach
Neste quiz, resolveremos o problema anterior (imagem "antigo-quiz.png"). Para isso, criaremos uma cadeia ou sequencia de promises:

```
getJSON('../data/earth-like-results.json')
    .then(function(response) {
      var sequence = Promise.resolve();

      response.results.forEach(function(url) {
        sequence = sequence.then(function() {
          return getJSON(url)
        })
        .then(createPlanetThumb);
      });
})
.catch(function(e) {
    console.log(e)
});

```

Estamos iterando os URLs e adicionando dois `.then` para cada uma das URLs:

```
response.results.forEach(function(url) {
    sequence = sequence.then(function() {
        return getJSON(url)
    })
    .then(createPlanetThumb);
});

```

O primeiro obtem os dados JSON, e o segundo, uma miniatura do planeta. Para cada repetição do array dos URLs de dados do planeta, a sequencia fica mais longa em dois `.then` por vez. Cada `.then` aguardará o proximo antes de resolver e antes de ser executado.

A boa noticia é que	as miniaturas aparecem na ordem correta, a ruim é que isso acontece em serie (como é possivel observar na imagem "thumb-serie.png"). Podemos ver que cada requisição espera a outra terminar.

Para que ocorresse de forma paralela o código deveria ser:

```
getJSON('../data/earth-like-results.json')
    .then(function(response) {
      var sequence = Promise.resolve();

      response.results.forEach(function(url) {
        sequence.then(function() { // alterado
          return getJSON(url)
        })
        .then(createPlanetThumb);
      });
})
.catch(function(e) {
    console.log(e)
});

```

Desta vez não estamos adicionando à sequencia, em vez disso, adicionamos dois `.then`, que é logo sobrescrito pela proxima repetição/interação do loop. Felizmente, os dois `.thens` ficam unidos um a outro e continuam executando. O resultado disso no painel network esta ilustrado na imagem "thumb-paralelo.png". Elas estão em paralelo, mas não ha garantia quanto a ordem. Na verdade, para muitos aplicativos, isso funciona. Podemo lidar com a ordem de outra maneira no front-end, mas se a ordem resolvida pelos promises for importante, este código se tornar fonte de problemas futuros. Navegadores podem requisitar fontes simultaneas, então faz sentido rodar este codigo em paralelo. Mas se quisermos fazer isso, devemos fazer onde fique obvio que a requisição acontece em paralelo.

**OBS**: As mudanças foram feitas no branch `foreach-start`.

# Promises com .map
O `.map()` é metodo array que aceita uma função e retorna um array. Esse array será o resultado da execução da função contra todo o elemento dele. Por exemplo, veja a imagem "map-1.png". Note que isso começa com o primeiro elemento, cria o array e passa o elemento para a função e a executa imediatamente. Então `.map` se muda para o proximo elemento (como na imagem "map-2.png"). Ele está no array esta tambem executando. Ele vai para o proximo e faz isso até que o novo array seja criado.

Neste caso, o arry que queremos iterar é o dos `URLs` e queremos chamar getJSON e `createPlanetThumb` para cada um deles. Neste quiz, iremos para o branch `map-start`. O codigo ficará da seguinte forma:

```
getJSON('../data/earth-like-results.json')
  .then(function(response) {
    response.results.map(function(url) {
    getJSON(url).then(createPlanetThumb);
  });
});

```

Mapeamos uma função anonima em cada URL e no array do resultados. A função pega o URL, obtém os dados do planeta e cria a miniatura dele. Não precisamos criar uma sequencia aqui. Mas a ordem não é garantida ao usar `.map`. Se isso for importante, precisariamos adicionar outra logica.

# Todas as promises
Há mais um metodo promise que devemos conhecer, que é `.all`. Ele pega um array de promises, o executa e retorna um array de valores na mesma ordem dos promises originais.

```
Promise.all(arrayOfPromisses)
.then(function(arrayOfValues) {
  ...
})

```

`.all` falha rapido, ele rejeitará assim que o primeiro promise rejeitar, sem esperar pelos outros promises. Então, se um dos promises rejeitar, todo o `.all` rejeitará. Mas quando o promise se resolver, o proximo da cadeia recebe o array de valores.

Neste quiz, iremos refatorar o codigo atual (do quiz anterior) usando `.all`. O codigo fica da seguinte forma:

```
getJSON('../data/earth-like-results.json')
    .then(function(response) {

      addSearchHeader(response.query);

      return Promise.all(response.results.map(getJSON));
    })
    .then(function(planetData) {
      planetData.forEach(function(planet) {
        createPlanetThumb(planet);
      });
    })
    .catch(function(error) {
      console.log(error);
    });

```

`.map` retorna um array, que passa todos os URLs para `getJSON()`. Como foram mapeados, esta função `getJSON()` roda imediatamente em todo URL. Quando todos os promises estiverem resolvidos, o link seguinte da cadeia roda, nesta caso `.then` recebe um `planetData`. Para cada planeta, criamos uma miniatura.

Os planetas estão novamente na ordem correta, e as requisições estão em paralelo. Como `.all` garante a ordem do array resultante, as miniaturas aparecem na ordem correta,.