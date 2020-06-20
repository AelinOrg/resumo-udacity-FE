# Como vou realizar esse projeto?

* Faça o curso de testes em JavaScript
* Baixe os recursos necessários ao projeto.
* Analise a funcionalidade da aplicação em seu navegador.
* Explore o HTML (./index.html), CSS (./css/style.css) e JavaScript (./js/app.js) da aplicação para entender como ela funciona.
* Explore o arquivo de especificações do Jasmine, em ./jasmine/spec/feedreader.js, e analise sua documentação.
* Edite a variável allFeeds, em ./js/app.js, para fazer o teste fornecido falhar e ver como o Jasmine visualiza essa falha em sua aplicação.
* Retorne a variável allFeeds para um estado de passagem.
* Elabore um teste que faça loop em cada feed no objeto allFeeds e garanta que ele tenha uma URL definida e não esteja vazia.
* Elabore um teste que faça loop em cada feed no objeto allFeeds, e garanta que ele tenha um nome definido e não esteja vazio.
* Elabore uma nova suíte de testes, denominada "O menu".
* Elabore um teste que garanta que o elemento de menu permaneça oculto por padrão. Você precisará analisar o HTML e o CSS para determinar como ocultaremos/mostraremos o elemento de menu.
* Elabore um teste que garanta que o menu altere a visibilidade quando alguém clica no item de menu. Esse teste deve ter duas expectativas: o menu deve ser exibido quando alguém clica nele e ocultado quando mais um clique é feito.
* Elabore uma suíte de testes denominada "Entradas iniciais".
* Elabore um teste que garanta que, quando a função loadFeed for chamada e concluir seu trabalho, exista pelo menos um elemento .entry no contêiner .feed.
* Elabore uma nova suíte de testes, denominada "Nova seleção de feed".
* Elabore um teste que garanta que, quando um novo feed for carregado pela função loadFeed, o conteúdo realmente mude.
* Nenhum teste deve depender dos resultados de outro.
* Callbacks devem ser usados para garantir que feeds sejam carregados antes de serem testados.
* Implemente a resolução de erros para variáveis não definidas e o acesso de array fora do limite.
* Ao terminar - todos os seus testes devem ser aprovados.