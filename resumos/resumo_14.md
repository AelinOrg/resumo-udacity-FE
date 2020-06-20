# Introdução ao AJAX
O conceito principal do AJAX é simples. Nós requisitamos alguns dados e, sem ficar esperando a requisição do retorno, fazemos outra coisa. Quando a requisição retorna, cuidamos dela. Esse é um resumo do AJAX

AJAX era uma sigla para "JavaScript Assincrono e XML", e ainda veremos assim, mas é um termo errôneo agora. AJAX é o conceito de requisitar dados de forma assincrona, seja num arquivo XML, JS, ou JSON de um API REST. Não importa quais sejam os dados. Nós requisitamos os dados de forma assincrona e lidamos com eles quando voltam. Nesta sessão, aprenderemos a fazer requisições AJAX usando JS. Primeiro vamos fazer com o consagrado objeto XHR. Depois veremos como o jQuery faz requisiões AJAX. Por fim, veremos a maneira nova e aprimorada de fazer requisições assicronas usano Fetch API.

# Demonstração de cliente servidor
Vamos começar explicando o que é uma solicitação. Na pratica podemos imaginar a internet como um monte de pessoas enviando mensagens uma para as outras. Imagine que temos três pessoas, uma delas será o cliente (navegador), outra a internet, e a outra o servidor (computador dedicado a fornecer conteudo aos navegadores). Se o navegador quiser algo do servidor, ele enviará uma requisição GET para ele. Uma requisião GET é uma mensagem que informa ao servidor, quem é o navegador e o que ele quer. Então ele envia a mensagem, a internet transmite para o servidor, que lê a mensagem envia de volta o que o navegador solicitou. Isso é chamado de "response" (resposta). O navegador pode abrir a mensagem e fazer o que quiser com ela.

Para que um site seja aberto, muitas solicitações de dados são realizadas. Na maioria das vezes, a resposta é essencial para que a pagina seja carregada, caso do HTML e CSS. Quando o navegador faz uma solicitação de modo sincrono, ou sem AJAX, ele deve esperar pelas respostas antes de continuar com o carregamento. O AJAX é especial porque permite que esses tipos de solicitação sejam feitos de modo assincrono, o que significa que eles podem acontecer em segundo planos sem bloquear o restante do carregamento da pagina.

Voltando a analogia de envio de mensagem por parte do navegador. Mas agora temos alguns planos em mente quando a resposta chegar. Assim, o browser envia a solicitação e guarda as instruções para mais tarde. O navegador fica livre para outros processos. Quando a resposta chegar, o navegador a abrirá, "olhorá" para as instruções e fara algo. Essas instruções que foram separadas são chamadas de callback (retorno de chamada), ja que as usamos quando  recebemos uma respota de volta.

## Vocabulário

* Requisição GET: Uma requisição de dados na Internet, enviada de um cliente para um servidor.
* Response: Uma resposta do servidor para uma requisição. Enviada do servidor para um cliente. A resposta de uma requisição GET normalmente incluirá dados de que o cliente precisa para carregar o conteúdo da página.

# Definição e exemplos de Ajax
As requisiçoes AJAX permitem a recuperação e exibição sem recarregar a pagina da web. A assincronia AJAX se refere ao fato de que a requisição não impede que outros eventos ocorram. A pagina segue normalmente e só acessa os dados quando o servidor devolve. As requisições AJAX ocorrem de inumeras maneiras e com varios niveis de dificuldade. Alguns pedem uma chave de API, outras usam "OAuth", e algumas não usam nenhuma auntenticação, e os dados são devolvidos por requisições AJAX diferentes. "X" em AJAX representa XML, que era o formato dominante dos dados hierarquicos, mas hoje JSON é bem mais usado. A maioria das requisições AJAX hohe são na verdade AJAJ, ou seja, "JavaScript Assincrono e JSON" (Asynchrnous JavaScript And JSON). Inserido nas respostas AJAX, é muito comum ver HTML, que os sites usam para preencher parte da pagina.

## Uma breve história
Frameworks JavaScript e Single Page Apps são, hoje em dia, as maneiras utilizadas para desenvolvimento, mas vamos entender de onde viemos antes de chegar a este ponto.

Em uma aplicação web renderizada pelo servidor tradicional, o computador cliente realiza uma requisição solicitando uma página web. O servidor cria e retorna a página para o cliente. Por fim, o cliente carrega a nova página e exibe a informação. Se o cliente interagir com a página, submetendo um formulário, por exemplo, um novo ciclo é iniciado. O cliente faz outra requisição, o servidor responde com uma página completamente diferente e o cliente carrega e apresenta ao usuário esta nova página.

Até meados dos anos 2000, essa era, basicamente, a única forma de comunicação via Internet. As informações estavam no servidor e o cliente as solicitavam por meio de requisições, atualizando a página e exibindo seu conteúdo. Esse ciclo se repetiria para cada nova requisição de página.

No fim dos anos 90, a equipe do Microsoft Outlook adicionou o componente XMLHTTP ao Internet Explorer e construiu uma versão web do cliente de e-mail Outlook. Esse código foi atualizado mais tarde, por outros navegadores, para XMLHttpRequest. Isso permitiu que os navegadores fizessem requisições HTTP por meio de JavaScript e atualizassem o conteúdo da página sem carregar uma página inteira do servidor. No lugar do modelo síncrono de esperar a página inteira, a interface do usuário poderia ser atualizada de maneira assíncrona enquanto o usuário continuaria trabalhando, com a maior parte das informações sendo transmitida utilizando o formato XML.

## AJAX
Em 2005, Jesse James Garrett criou o termo AJAX para representar "JavaScript Assíncrono XML". Essa era, essencialmente, a técnica que utilizava XMLHTTPRequest para recuperar dados e modificar a página atual.

O AJAX dominou rapidamente o mundo web, sendo disseminado para muito além do Microsoft Outlook. Aplicações de ponta como Flickr, GMail e Google Maps adotaram o AJAX rapidamente. Em vez de precisar esperar a resposta do servidor com os dados e ter toda a página recarregada, essa nova maneira de se carregar dados de forma assíncrona era incrível.

## Inconsistências nos navegadores
Nem tudo eram flores com o Ajax. Havia muitas implementações diferentes de navegadores incompatíveis, e desenvolvedores eram forçados a programar para um navegador específico, ou escrever códigos complexos para todos eles. Com o passar do tempo, surgiram bibliotecas JavaScript, como jQuery e YUI, para reconciliar essas divergências.

Aplicações AJAX eram ótimas, mas difíceis, e os desenvolvedores tinham dificuldades para criá-las; enquanto os navegadores continuavam evoluindo e as pessoas demandavam aplicativos em mais dispositivos, o código se tornou cada vez mais complexo e confuso. Esse desafio acabou por implicar no surgimento de bibliotecas JavaScript padrão. Bibliotecas JavaScript surgiram para abstrair a complexidade dos diferentes navegadores, tornando o desenvolvimento de aplicações poderosas e complexas gerenciável.

## Exemplo Google
Com o DevTools aberto na aba network e o browser exibindo a homepage do Google, ao digitarmos uma letra, veremos que a pagina será alterada. No DevTools veremos uma requisição do tipo XHR, que significa que foi feita uma requisição assincrona.

# APIs

## Recuperando informações
Vimos os conceitos do Ajax e entendemos que essa é a tecnologia que utilizaremos para adicionar informações ao nosso projeto de maneira assíncrona. Mas, de onde vem essas informações? Como nós temos acesso a elas e como nosso aplicativo saberá trazer esses dados?

Vamos começar utilizando uma API para interagir com várias fontes de dados.

## O que é uma API?
O acronimo "API" significa:

**A**pplication
**P**rogramming
**I**nterface

Ou , em português, interface para a programação de aplicações.

Existem dados disponíveis apenas esperando para serem utilizados. A maior parte das aplicações ricas em dados que você utiliza busca seus dados em sites de terceiros. Elas recuperam esses dados por meio de APIs.

## Algumas APIs

* [APIs do Google](https://developers.google.com/apis-explorer/) - Todos os serviços do Google que você possa imaginar.
* [Banco de dados de APIs gigantes](https://www.programmableweb.com/apis/directory) - Vale muito a pena vasculhar esse site para se inspirar.
* Você sabia que a [Udacity possui uma API](https://www.udacity.com/public-api/v1/courses)? Ela está disponível para qualquer um que queira utilizá-la. Nós queremos que desenvolvedores tenham fácil acesso e possam compartilhar nosso catálogo de cursos.

# Crie uma requisição assíncrona com XHR
Pense na tarefa de assar um bolo. Após misturamos todos os ingredientes e colocar no forno, é preciso que ele fique totalmente pronto antes de realizarmos outros processos, como assar colocar glacê. Requisitamos que o forno transformasse a massa de bolo em bolo. Ao terminar, o timer vai apitar, avisando que podemos pegar o bolo. Mas não precisamos ficar esperando que fique pronto. Podemos fazer outras coisas. Podemos fazer o glacê agora.

Um objeto XHR é fornecido pelo ambiente JS e é usado para requisições AJAX. É igaul a proporção da massa de bolo. É preciso fazer os passos manualmente para configuarar a requisição e enviar. Mas o codigo pode fazer outras coisas. Quando a resposta voltar, estará tudo pronto para os dados. Pensando nessa analogia, vamos analaisar o objeto XHR.

# O objeto XHR
Assim como o document é disponibilizado pelo mecanismo JavaScript, o JavaScript também possui uma maneira de realizarmos requisições HTTP assíncronas. Fazemos isso através de um objeto XMLHttpRequest. Podemos criar esses objetos com a função construtora do XMLHttpRequest disponibilizada.

Uma das melhores formas de aprender é colocando a mão na massa e fazendo tentativas! Então, para seguir em frente, abra as ferramentas do desenvolvedor e, no console, execute o seguinte:

```
const asyncRequestObject = new XMLHttpRequest();

```

O construtor da função possui as letras "XML" em seu nome, mas isso não significa que o objeto é restrito para documentos XML. Lembre-se de que a sigla "AJAX" representa "JavaScript Assíncrono e XML". Uma vez que o formato principal utilizado para troca de dados assíncronos era o XML, é fácil entender de onde a função XMLHttpRequest herdou seu nome!

XMLHttpRequests (normalmente abreviadas para XHR) podem ser utilizadas para requisitar qualquer tipo de arquivo (arquivos de texto, arquivos HTML, arquivos JSON, Imagens, etc.) ou dados de uma API.

**Observação**: *agora, vamos nos aprofundar nos detalhes do objeto XMLHttpRequest. Vamos aprender como criá-lo, quais métodos e propriedades devem ser utilizados e como fazer requisições assíncronas. Para ainda mais detalhes sobre o uso do objeto XHR e para fazer requisições assíncronas com ele, confira estes links:*

* Documentação MDN - https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/open
* Especificação WHATWG - https://xhr.spec.whatwg.org/
* Especificação W3C - https://www.w3.org/TR/XMLHttpRequest/

# Método .open() do XHR
Até aqui, construímos um objeto XHR denominado `asyncRequestObject`. Existem diferentes métodos disponíveis neste objeto, e um dos mais importantes é o método [`open`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/open).

```
asyncRequestObject.open();

```

`.open()` recebe uma série de parâmetros, mas os mais importantes são os dois primeiros: o método HTTP e a URL para envio da requisição

Se quisermos requisitar a página de maneira assíncrona do conhecido site de imagens Unsplash, utilizaríamos uma requisição `GET` e forneceríamos a URL:

```
asyncRequestObject.open('GET', 'https://unsplash.com');

```

## Um pouco enferrujado com os métodos HTTP?
Os dois principais que você usará são:

* `GET` - para recuperar dados
* `POST` - para enviar dados

Para mais informações, confira nosso [curso HTTP & Web Servers](https://classroom.udacity.com/courses/ud303)!

Aviso: por motivos de segurança, você só pode requisitar recursos e informações a uma página caso esteja no mesmo domínio que o site que carregará essas informações. Por exemplo, para fazer uma requisição assíncrona ao google.com, seu navegador precisa estar na página google.com. Isso é conhecido como [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy), que pode parecer extremamente restritivo - e de fato é!

A razão por trás disso é que o JavaScript controla muita informação na página, tendo acesso inclusive aos cookies, de modo que seria possível determinar senhas se rastreássemos as teclas digitadas. No entanto, a web não seria o que é hoje se toda essa informação ficasse restrita em seus servidores. A maneira de contornar este problema é por meio do CORS (Cross-Origin Resource Sharing). [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) é uma tecnologia implementada no servidor que permite que requisições sejam feitas a partir de outras origens.

## QUESTION 1
Vá para a página Google, abra as ferramentas do desenvolvedor e execute o código abaixo no console:

```
const req = new XMLHttpRequest();
req.open('GET', 'https://www.google.com/');

```

O que acontecerá?

**R:** Nada. O método `.open()` do objeto XHR **não envia a requisição!** Ele prepara as configurações necessárias e atribui ao objeto as informações de que ele precisa para quando a requisição for enviada de fato. Vamos agora enviar a requisição de fato!

## QUESTION 2
O método `.open()` de um objeto XHR pode receber diferentes argumentos. Utilize a [documentação](https://developer.mozilla.org/pt-BR/docs/Web/API/XMLHttpRequest/open) para explicar o que o código a seguir faz:

```
const myAsyncRequest = new XMLHttpRequest();
myAsyncRequest.open('GET', 'https://udacity.com/', false);

```

**R:** O JavaScript congela e espera até que a requisição retorne. Passar `false` como terceiro parâmetro para o método open faz com que a requisição XHR seja realizada de forma síncrona, fazendo com que o mecanismo JavaScript pause e espere até que a requisição retorne antes de continuar - essa "pausa e espera" também é chamada de "blocking". Isso é uma péssima ideia e fere completamente o propósito de se ter um comportamento assíncrono. Garanta que você nunca vá configurar seu objeto XHR dessa forma! Em vez disso, ou passe `true` como terceiro parâmetro ou deixe-o em branco.

# Método .send() do XHR
Para enviar a requisição de fato, precisamos utilizar o [método send](https://developer.mozilla.org/pt-BR/docs/Web/API/XMLHttpRequest/send):

```
asyncRequestObject.send();

```

Vamos ver o que acontece com este codigo:

```
const asyncRequestObject = new XHTMLHttpRequest();
asyncRequestObject.open('GET', 'https://unplash.com')
asyncRequestObject.send();

```

A requisição XHR é enviada, mas não vemos nada. Mas algo acontece. Podemos ver a requisição no painel network (lembre-se estar na pagina do site a requisitar). Antes, para gravar o trafego de rede, o DevTools precisa gravar (Sinalização vermelha de rec no canto superior esquerdo). Executando a requisição novamente, ela é enviada. Poderemos ver ela surgindo no painel. Ao selecionar uma requisição, o `Header` aparece. Podemos ver a URL requisitada ("Request URL") e o metodo da requisição ("Request Method"). Há tambem uma visualização (`Preview`) da requisição e o pagamento da resposta (`Response`) contem o HTML da resposta. Apesar de nada acontecer na pagina ou no codigo, a requisição foi enviada.

Não faz muito sentido fazer uma requisição para algo e não fazer nada com essa informação! Por que pediríamos bolo se não fossemos comê-lo? Não faz sentido! Queremos comer o bolo também!

## Tratando a resposta bem-sucedida
Para tratar a resposta bem-sucedida de uma requisição XHR, nós configuramos a propriedade onload no objeto para uma função com essa responsabilidade:

```
function handleSuccess () {
    // in the function, the `this` value is the XHR object
    // this.responseText holds the response from the server

    console.log( this.responseText ); // the HTML of https://unsplash.com/
}

asyncRequestObject.onload = handleSuccess;

```

Como acabamos de ver, se `onload` não estivesse configurado, a requisição retornaria... mas nada aconteceria.

## Tratando erros
Você já deve ter deduzido que a propriedade **onload** é chamada quando a resposta é bem-sucedida. Se algo acontecer com a requisição e ela não puder ser concluída, então precisaremos utilizar a propriedade **onerror**:

```
function handleError () {
    // in the function, the `this` value is the XHR object
    console.log( 'An error occurred 😞' );
}

asyncRequestObject.onerror = handleError;

```

Assim como com o `onload`, caso `onerror` não esteja configurado e um erro ocorra, esse erro falhará silenciosamente e seu código (seu usuário também!) não terá idéia do ocorrido nem conhecerá uma maneira de se recuperar.

# Uma requisição completa
Este é o código completo que construímos para criar um objeto XHR, especificar a informação que deve ser requisitada, configurar os tratamentos de sucesso e de erro e, só então, enviar a requisição:

```
function handleSuccess () {
    console.log( this.responseText ); // the HTML of https://unsplash.com/
}

function handleError () {
    console.log( 'An error occurred 😞' );
}

const asyncRequestObject = new XMLHttpRequest();

asyncRequestObject.open('GET', 'https://unsplash.com');
asyncRequestObject.onload = handleSuccess;
asyncRequestObject.onerror = handleError;

asyncRequestObject.send();

```

## APIs e JSON
Recuperar o HTML de um site é possível, mas, provalvemente, não será muito útil. A informação retorna em um formato muito difícil de analisar e consumir. Seria muito mais fácil se pudéssemos ver apenas a informação que queremos de uma maneira mais estruturada. Se você está pensando que o formato JSON seria uma boa ideia, você está certo, e eu vou dar um pedaço do bolo a você!

Em vez de realizar a requisição para a URL principal do site Unsplash, vamos criar um aplicativo que recupera apenas uma imagem da API Unsplash e artigos relevantes do The New York Times.

Ao fazer uma requisição para uma API que retorna JSON, tudo o que precisamos fazer é converter a resposta JSON em um objeto JavaScript. Podemos fazer isso com [`JSON.parse();`](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse). Vamos explorar a função `onload` para tratar uma resposta JSON:

```
function handleSuccess () {
    const data = JSON.parse( this.responseText ); // convert data from JSON to a JavaScript object
    console.log( data );
}

asyncRequestObject.onload = handleSuccess;

```

# Passo a passo inicial do projeto
O projeto inicial está no GitHub: https://github.com/udacity/course-ajax. Você pode clonar o projeto executando o seguinte comando git no seu terminal:

git clone https://github.com/udacity/course-ajax.git
Com o projeto clonado, veja que há três diferentes pastas:

* `lesson-1-async-w-xhr`
* `lesson-2-async-w-jQuery`
* `lesson-3-async-w-fetch`

Certifique-se de trabalhar nos arquivos da lição correto. Como está é a primeira lição, vamos trabalhar no diretório `lesson-1-async-w-xhr`.

## Crie suas Contas
Para completar estes últimos passos, você precisará de contas registradas nas páginas do Unsplash e do The New York Times.

### Unsplash

* Crie uma conta de desenvolvedor aqui - https://unsplash.com/developers
* Em seguida, crie uma aplicação aqui - https://unsplash.com/oauth/applications
* Isso dará a você uma "Application ID", de que você precisará para fazer requisições

### The New York

* Crie uma conta de desenvolvedor aqui - https://developer.nytimes.com/
* Eles enviarão um e-mail a você, com a chave da API (você precisará dela para fazer as requisições)

## Requisição para Unsplash
Em nosso aplicativo, a variável searchedForText contém o texto no qual estamos interessados, e vamos configurar a propriedade onload para uma função chamada addImage (que é uma função que, por hora, não faz nada). Se atribuirmos temporariamente o valor "hippos" para searchedForText, o código para a requisição XHR ao Unsplah será:

```
function addImage(){}
const searchedForText = 'hippos';
const unsplashRequest = new XMLHttpRequest();

unsplashRequest.open('GET', `https://api.unsplash.com/search/photos?page=1&query=${searchedForText}`);
unsplashRequest.onload = addImage;

unsplashRequest.send()

```

...mas quando tentamos executar este código, um erro ocorre.

## QUESTION
A requisição ao Unsplash não funciona porque é necessário um header HTTP, que será enviado junto. Qual é o método XHR para adicionar um header à requisição? [Veja a documentação](https://developer.mozilla.org/pt-BR/docs/Web/API/XMLHttpRequest) para descobrir!

**R:** `.setRequestHeader()`

# Configurando um header na requisição
O método XHR para incluir um header na requisição é o setRequestHeader. Então, o código completo precisa ser desta forma:

```
const searchedForText = 'hippos';
const unsplashRequest = new XMLHttpRequest();

unsplashRequest.open('GET', `https://api.unsplash.com/search/photos?page=1&query=${searchedForText}`);
unsplashRequest.onload = addImage;
unsplashRequest.setRequestHeader('Authorization', 'Client-ID <your-client-id>');
unsplashRequest.send();

function addImage(){
}

```

Quando a requisição retornar com sucesso, vamos pausar dentro da função para ver o que retornou. Para isso vamos adicionar um depurador na função `addImage()`:

```
const unsplashRequest = new XMLHttpRequest();
const searchedForText = 'hippos';

unsplashRequest.onload = addImage;
unsplashRequest.onerror = function(err) {
	requestError(err, 'image')
}

unsplashRequest.open('GET', `https://api.unsplash.com/search/photos?page=1&query=${searchedForText}`);
unsplashRequest.setRequestHeader('Authorization', 'Client-ID 25772576d598747a2ee9829bb9e6aa28307f58fafaa38d48271a8d4b5402b42e');
unsplashRequest.send();

function addImage(){
	debugger;
}

```

Ao atualizar a pagina, e buscar por algo como "flamingos", a pagina será pausada. O valor é o objeto XHR (`Local > this: XMLHttpRequest`) e a resposta é armazenada no `responseText`. A resposta JSON, no painel `Network`, aba `Response`, exibe todos os textos. Na aba `Preview` vemos um total de 208 respostas para `hippos`. Devemos converte-las de JSON para um objeto JavaScript (usando `JSON.parser()`) e formatar os dados:

```
const unsplashRequest = new XMLHttpRequest();
const searchedForText = 'hippos';

unsplashRequest.onload = addImage;
unsplashRequest.onerror = function(err) {
	requestError(err, 'image')
}

unsplashRequest.open('GET', `https://api.unsplash.com/search/photos?page=1&query=${searchedForText}`);
unsplashRequest.setRequestHeader('Authorization', 'Client-ID 25772576d598747a2ee9829bb9e6aa28307f58fafaa38d48271a8d4b5402b42e'); // Meu ID
unsplashRequest.send();

function addImage(){
	let htmlContent = '';
	const data = JSON.parse(this.responseText);
	const firstImage = data.results[0];

	htmlContent = `<figure>
		<img src="${firstImage.urls.regular}" alt="${searchedForText}">
		<figcaption> ${searchedForText} by ${firstImage.user.name}</figcaption>
	</figure>`

	responseContainer.InsertAdjacentHTML('afterbegin', htmlContent);

}

```

Pegamos a primeira imagem atraves de `const firstImage = data.results[0];`. Tendo a primeira imagem, basta adiciona-la à pagina. O codigo adicionará um elemento de imagem com um ponto de imagem do Unsplash e uma legenda da pessoa que tirou a foto. Ela será adicionada na resposta como primeiro elemento. Saiba que, alguns elementos como `.name`, são da propria API do Unsplash.

Tambem queremos resover o problema de não haver retorno de imagens, isto é, ter certeza de que as imagens foram realmente armazenadas. Fazemos isso como uma condição:

```
const unsplashRequest = new XMLHttpRequest();
const searchedForText = 'hippos';

unsplashRequest.onload = addImage;
unsplashRequest.onerror = function(err) {
	requestError(err, 'image')
}

unsplashRequest.open('GET', `https://api.unsplash.com/search/photos?page=1&query=${searchedForText}`);
unsplashRequest.setRequestHeader('Authorization', 'Client-ID 25772576d598747a2ee9829bb9e6aa28307f58fafaa38d48271a8d4b5402b42e');
unsplashRequest.send();

function addImage() {
	let htmlContent = '';
	const data = JSON.parse(this.responseText);

	if(data && data.results && data.results[0]) {
		const firstImage = data.results[0];
		htmlContent = `<figure>
			<img src="${firstImage.urls.regular}" alt="${searchedForText}">
				<figcaption> ${searchedForText} by ${firstImage.user.name}</figcaption>
			</figure>`;
	} else {
		htmlContent = `<div class="error-no-image">No images available</div>`;
	}

	responseContainer.insertAdjacentHTML('afterbegin', htmlContent);

}

```

Isso vai verificar se os resultados de imagens serão retornados. Mesmo que não sejam, exibiremos a mensagem de falta de imagens.

Como o New York Times não demanda um header específico, não precisaremos fazer nada de especial neste caso. Configuramos a propriedade da função `addArticles` desta forma:

```
function addArticles () {}
const articleRequest = new XMLHttpRequest();
articleRequest.onload = addArticles;
articleRequest.open('GET', `http://api.nytimes.com/svc/search/v2/articlesearch.json?q=${searchedForText}&api-key=1sAuS5hZrwKtxSVzq6KVGV2qxlFU68B9`); // Meu ID
articleRequest.send();

```

Garanta que você preencheu a URL acima com a chave da API que recebeu por e-mail do The New York Times após se cadastrar como desenvolvedor.

A resposta tem um objeto de aninhada. As propriedades dos documentos do objeto contem artigos. Então é preciso converter a resposta de JSON para objeto JS.

```
const articleRequest = new XMLHttpRequest();
articleRequest.onload = addArticles;
articleRequest.open('GET', `http://api.nytimes.com/svc/search/v2/articlesearch.json?q=${searchedForText}&api-key=4eee4a45-c96d-4c50-bd49-7be4c1ec0c04
`); // Meu ID
articleRequest.send();

function addArticles () {
	let htmlContent = '';
	const data = JSON.parse(this.contentText);

	if (data.response && data.response.docs && data.response.docs.lenght > 1) {
		htmlContent = '<ul>' + data.response.docs.map(`<li class ="article">
			<h2><a href="${article.web.url}">${article.headline.main}</a></h2>
			<p>${article.snippet}</p>
		</li>`).join() + </ul>
	} else {
		htmlContent = `<div class="error-no-articles">No articles available</div>`
	}
}

```

Se alguns arquivos forem retornados, a seção mapeará cada artigo e retornará uma lista de itens com o cabeçalho do artigo em um trecho (snippet) do artigo. Por fim, combinamos as listas de itens de artigos a uma lista de tags fora da ordem. Isso será adicionado no final, dentro do container da respota (no projeto). Se não houver artigos, aparecerá: "nenhum arquivo disponivel."


# Recapitulando XHR
Existe uma série de passos que devem ser executados para enviar uma requisição HTTP assíncrona com JavaScript.

## Para enviar uma requisição assíncrona

* crie um objeto XHR com o construtor do `XMLHttpRequest`
* utilize o método `.open()` - configure o método HTTP e a URL do recurso que será requisitado
* configure a propriedade `.onload` - configure esta propriedade para uma função que será executada caso a requisição seja bem-sucedida
* configure a propriedade `.onerror` - configure esta propriedade para uma função que será executada caso a requisição falhe
* utilize o método `.send()` - envie a requisição

## Utilizando a response

* utilize a propriedade `.responseText` - receberá o texto da resposta assíncrona

**Obs.:** *a especificação original do objeto XHR foi criada em 2006. Essa foi a primeira versão da especificação. Não houve grandes alterações na especificação nos últimos anos.

*Em 2012, foi iniciado o trabalho de especificação da versão 2 do XHR. Em 2014, a especificação XHR2 foi fundida à XHR1 para que não houvesse padrões divergentes. Ainda existem referências ao XHR2, mas a especificação XHR incorpora a XHR2 completamente.*

*Confira este artigo no portal HTML5Rocks sobre [novos truques no XHR2](http://www.html5rocks.com/en/tutorials/file/xhr2/) que agora fazem parte da especificação XHR.*

# Codigo Final

```
/* eslint-disable */


(function () {
    const form = document.querySelector('#search-form');
    const searchField = document.querySelector('#search-keyword');
    let searchedForText;
    const responseContainer = document.querySelector('#response-container');

    form.addEventListener('submit', function (e) {
        e.preventDefault();
        responseContainer.innerHTML = '';
        searchedForText = searchField.value;

		const imgRequest = new XMLHttpRequest();
		imgRequest.onload = addImage;
		imgRequest.onerror = function(err) {
			requestError(err, 'image')
		};

		imgRequest.open('GET', `https://api.unsplash.com/search/photos?page=1&query=${searchedForText}`);
		imgRequest.setRequestHeader('Authorization', 'Client-ID 25772576d598747a2ee9829bb9e6aa28307f58fafaa38d48271a8d4b5402b42e');
		imgRequest.send();

		const articleRequest = new XMLHttpRequest();
		articleRequest.onload = addArticles;
		articleRequest.open('GET', `http://api.nytimes.com/svc/search/v2/articlesearch.json?q=${searchedForText}&api-key=1sAuS5hZrwKtxSVzq6KVGV2qxlFU68B9`); // Meu ID
		articleRequest.send();

	});

		function addImage() {
			let htmlContent = '';
			const data = JSON.parse(this.responseText);

			if(data && data.results && data.results[0]) {
				const firstImage = data.results[0];
				htmlContent = `<figure>
					<img src="${firstImage.urls.regular}" alt="${searchedForText}">
					<figcaption> ${searchedForText} by ${firstImage.user.name}</figcaption>
				</figure>`;
				} else {
					htmlContent = `<div class="error-no-image">No images available</div>`;
				}

			responseContainer.insertAdjacentHTML('afterbegin', htmlContent);

		}

		function addArticles() {
			let htmlContent = '';
			const data = JSON.parse(this.responseText);

			if (data.response && data.response.docs && data.response.docs.length > 1) {
				htmlContent = '<ul>' + data.response.docs.map(article => `<li class="article">
						<h2><a href="${article.web_url}">${article.headline.main}</a></h2>
						<p>${article.snippet}</p>
					</li>`
				).join('') + '</ul>'
			} else {
				htmlContent = `<div class="error-no-articles">No articles available</div>`
			}

			responseContainer.insertAdjacentHTML('beforeend', htmlContent);
		}
})();


```

Esse é o codigo ajustado, pronto para uso, basta acessar a pagina `index` e realizar uma busca. Variaveis como `searchedForText`, foram declaradas em escopo mais amplo, pois as duas requisições e funções (`addImage` e `addArticles`) as usarão, então não precisam ser declaradas especificamente para cada função, como estavamos fazendo antes. Alem disso, `searchedForText` agora recebe o valor digitado pelo usuario no campo de busca.

# Conclusão sobre XHR
Vimos como um objeto XHR cria e envia requisições assincronas. São varios os passos para se criar um objeto XHR, lidar com uma requisiçao bem-sucedida e lidar com erros. Foram muitos codigos. Será que temos que escrever o codigo em todas as requisições assincronas? Ao usar objetos XHR, a resposta é "sim". Mas nem sempre é objeto XHR que faz requisições assincronas. É possivel usar uma biblioteca como jQuery para a requisição.