# Introdução
Quando usamos o JS no navegador para renderizar a IU, às vezes as pessoas chamam de aplicativo de pagina unica. O que isso quer dizer? Um site comum funciona assim:

* Quando o usuario o visita, o navegador solicita uma pagina do servidor do site, o servidor gera o HTML e envia. Quando o usuario navega, o navegador solicita uma nova pagina do servidor, o servidor envia o novo HTML para o navegador e o usuario ve a nova pagina. Todo vez que o usuario navega, esse ciclo se repete.

Quando as pessoas dizem	"aplicativo de pagina unica", não querem dizer que há só uma tela no aplicativo, e sim que o  navegador não precisa voltar ao servidor para ter novas paginas. O JS lida com a transição entre as paginas. Então só há uma pagina inicial enviada do servidor. É por isso que se chama aplicativo de pagina unica. React Router é uma ferramenta que nos permite usar o React para criar um aplicativo de pagina unica. Vamos usa-lo para adicionar roteadores ao app.

## Aplicações de página única
Aplicações de página única (ou single-page applications, SPA) podem funcionar de diferentes maneiras. Uma maneira como uma aplicação single-page pode ser carregada é baixando todo o conteúdo do site de uma única vez. Desse modo, enquanto você navega pela página, todo o conteúdo já está disponível para o navegador e não haverá a necessidade de atualizar a janela. Outra maneira em que aplicações single-page funcionam é fazendo o download de tudo o que for necessário para renderizar a página que o usuário requisitou e, então, quando o usuário navegar para uma nova página, requisições JavaScript assíncronas são feitas apenas para o novo conteúdo solicitado.

Outro fator-chave para uma boa aplicação single-page é o controle do conteúdo das páginas através da URL. Aplicações single-page são altamente interativas, e usuários desejam poder retornar para um determinado estado utilizando apenas a URL. Por que isso é importante? "Favoritabilidade" (tenho certeza de que isso não é uma palavra...ainda)! Quando você adiciona um site aos favoritos, esse favorito é apenas a URL que, por sua vez, não grava o state da página. Isso está intimamente ligado ao conceito de aplicações com arquitetura RESTful (saiba mais [aqui](https://becode.com.br/o-que-e-api-rest-e-restful/) e [aqui](https://pt.stackoverflow.com/questions/45783/o-que-%C3%A9-rest-e-restful)).

Você notou que nenhuma das ações realizadas no aplicativo altera a URL da página? Precisamos criar aplicações React que ofereçam páginas "favoritáveis"!

## React Router
O React Router transforma projetos React em aplicações single-page e faz isso fornecendo alguns componentes especializados, que gerenciam a criação de links e a URL da aplicação, fornecem transições para a navegação entre diferentes URLs e muito mais.

De acordo com o site do React Router:

*React Router é uma coleção de componentes de navegação que compõem, de forma declarativa, sua aplicação.

Se você estiver interessado, fique à vontade de conferir o site em https://reacttraining.com/.

Na próxima sessão, vamos renderizar dinamicamente o conteúdo da página com base em um valor do objeto `this.state`. Usaremos este exemplo básico como uma ideia de como o React Router funciona, controlando o que está sendo visualizado por meio dos estados. Em seguida, vamos mudar para o uso do React Router. Vamos caminhar com você durante a instalação do React Router, adicionando a biblioteca ao projeto e juntando as peças do quebra-cabeça para que a biblioteca possa gerenciar seus links e URLs.

# Renderizando páginas dinamicamente
Vamos criar um formulário que nos permita criar novos contatos e salvá-los em nosso servidor. Lembre que o React favorece a composição de componentes, portanto, nós queremos criar nosso novo pedaço de UI como um componente separado e usar composição para inclui-lo em um outro componente.

Nós não queremos que o formulário apareça o tempo todo, então, vamos começar exibindo o formulário apenas se uma configuração específica estiver habilitada. Vamos guardar essa configuração no this.state. Fazendo dessa forma, teremos uma ideia de como o React Router funciona.

## Pensando na logica
Do que serve um aplicativo de contatos se não podemos criar novos contatos? Vamos adicionar uma nova tela para adicionar novas pessoas.  A primeira coisa a fazer é criar um novo arquivo. Vamos chamá-lo de `src/CreateContact.js`. Precisamos importar o React no `Component`. As telas no React são componentes.

```
import React, { Component } from 'react'

```

Vamos adicionar a classe do componente. `CreateContact extends Compoenent`.

```
class CreateContact extends Compoenent {
	render() {
		return (
			<div>Create Contacts!</div>
		)
	}
}

```

Temos um metode `render` simples. Vamos exportar o padrão `CreateContact`:

```
export default CreateContact

```

Quando importamos no aplicativo, vai funionar. Vamos fazer isso em `src/Appjs`.

```
import CreateContact from './CreateContact'

```

Vamos renderizar para garantir que esta tudo certo.

```
class App extends Component {
	//...
	render() {
		return (
			<div>
			    <ListContacts onDeleteContact={this.removeContact} contacts={this.state.contacts} />
			    <CreateContact/>
			</div>
		)
	}
}

```

Precisamos salvar. Veremos as duas telas/componentes juntas na pagina. A ideia não é renderizar todas telas do aplicativo ao mesmo tempo, e sim escolher que tela queremos mostrar. Podemos usar o React Router agora, mas vamos ver como fazer com componente `state` primeiro. Já sabemos usar o componente estado. Vai nos ajudar a entender como o React Router raciocina se implementarmos sozinhos antes. Precisamos do estado para decidir que tela mostrar. Vamos chamar de `screen`.

```
class App extends Component {
	state = {
		screen: 'list',
		contacts = []
	}
	//...
}

```

Podemos ter uma pagina quando `screen` for `list`, e outra quando for `create`. Temos o estado para trocar entre `list` e `create`, assim podemos tomar decisões no metodo `render` sobre o que mostrar. Escrevemos:

```
class App extends Component {
	//...
	render() {
		return (
			<div>
			{this.state.screen === 'list' && (
    			<ListContacts
    				onDeleteContact={this.removeContact}
    				contacts={this.state.contacts}
    			/>
			)}
			{this.state.screen === 'create' && (
			    <CreateContact/>
			)}
			</div>
		)
	}
}

```

Se for a tela que queremos ver, podemos inclui-la na expressão. Se for verdadeiro, vamos ver uma lista. Fizemos a mesma coisa para tela `create`. Nosso estado atual é `list`. Ao salvar o arquivo e atualizar, veremos somente o componente/tela `ListContacts`. Podemos editar o estado manualmente para `create` e ver como ficaria.

Realizamos várias mudanças importantes nesse capitulo. Criamos o componente CreateContact, que será responsável pelo formulário que criará os novos contatos. Seguindo a linha geral do React, de favorecer a composição de componentes, isolamos o novo trecho de código em um componente e o adicionamos ao método `render()` no componente `App`.

Em uma tentativa de recriar de forma extremamente simples a maneira de funcionamento do React Router, adicionamos uma propriedade `screen` para o `this.state` e usamos essa propriedade para controlar o que deve ser exibido na página. Se `this.state.screen` é `list`, então mostraremos a lista de todos os contatos. Se `this.state.screen` é `create`, então mostraremos o componente `CreateContact`.

## Sintaxe de avaliação de curto-circuito (Short-Circuit)
Neste vídeo, e quando criamos as validações do que deveria ser exibido nele, utilizamos uma sintaxe um tanto quanto estranha:

```
{this.state.screen === 'list' && (
  <ListContacts
    contacts={this.state.contacts}
    onDeleteContact={this.removeContact}
  />
)}

```

e

```
{this.state.screen === 'create' && (
  <CreateContact />
)}

```

Isso pode ficar um pouco confuso tanto com o código JSX para um componente como com o código para executar uma expressão. Mas, na verdade, trata-se apenas da expressão lógica `&&`:

```
expressão && expressão

```

O que estamos fazendo aqui é uma técnica em JavaScript chamada avaliação de curto-circuito (short-circuit). Se a primeira expressão for validada como `true`, a segunda expressão é executada. No entanto, se a primeira expressão for validada como `false`, a segunda expressão não será executada. Utilizaremos este método como uma guarda para primeiro verificar o valor de `this.state.screen` antes de exibir o componente correto.

Para um conteúdo mais detalhado sobre o tema, dê uma olhada na página [the short-circuit evaluation info](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Short-circuit_evaluation) on MDN.

## Adicionando um botão
No momento, temos que modificar o estado manualmente para fazer com que o aplicativo exiba telas diferentes. Queremos que o usuário seja capaz de controlar isso na própria aplicação, então vamos adicionar um botão!

Queremos que o usuario faça a transição em vez de fixar o codigo. Vamos adicionar um botão perto do campo de buscar contatos e adicionar um click handler que muda o estado e a tela. Isso acontece no arquivo `src/ListContacts.js`. Vamos até o metodo `render`. Vamos colocar logo abaixo do input.

```
return (
	<div className='list-contacts'>
		<div className='list-contacts-top'>
			<input
				className='search-contacts'
				type='text'
				placeholder='Search contacts'
				value={this.state.query}
				onChange={(event) => this.updateQuery(event.target.value)}
			/>
			<a
				href="#create"
				onClick={() => ()}
				className="add-contact"
			>Add Contact</a>
	</div>
	//...
)

```

Vamos colocar uma função vazia por enquanto porque não sabemos o que vamos fazer ainda. Agora temos um botão. Quando clicamos não sabemos o que fazer. Queremos mudar o estado do aplicativo. Estamos no componente de lista de contatos. Não tem o estado da tela. Temos que voltar para `src/App.js`. Para modificar o estado, precisamos passar uma função para o componente de lista de contatos. Já fizemos isso no `onDeleteContact`. Quando o contato é removido, chamamos `this.removeContact`, que chama o `setState`. Vamos fazer a mesma coisa. Vamos chamar de `onNavigate`. Podemos passar uma função já criada, ou declarar a função. Queremos que o `setState` e mudar a tela para `create`.

```
class App extends Component {
	//...
	render() {
		return (
			<div>
			{this.state.screen === 'list' && (
    			<ListContacts
    				onDeleteContact={this.removeContact}
    				contacts={this.state.contacts}
    				onNavigate={() => {
						this.setState({ screen: 'create'})
    				}}
    			/>
			)}
			{this.state.screen === 'create' && (
			    <CreateContact/>
			)}
			</div>
		)
	}
}

```

Agora que passamos a função como `onNavigate`, precisamos voltar à `ListContacts.js`. No click handler `onClick` que esta vazio, passamos `this.props.onNavigate`. Quando o botão for clicado, a função é chamada e ele troca o estado.

```
return (
	<div className='list-contacts'>
		<div className='list-contacts-top'>
			<input
				className='search-contacts'
				type='text'
				placeholder='Search contacts'
				value={this.state.query}
				onChange={(event) => this.updateQuery(event.target.value)}
			/>
			<a
				href="#create"
				onClick={() => (this.props.onNavigate)}
				className="add-contact"
			>Add Contact</a>
	</div>
	//...
)

```

Quando o estado mudar, em vez de vermos o `ListContacts`, vamos ver o `CreateContact`. O que deveria acontecer quando clicamos no botão de voltar? Não voltaremos para a lista de contatos.

Por isso temos o React Router. Ele mantem a IU e o URL em sincronia. As expectativas do usuario em releção a telas, links e URLs ficam intactas.

## Recapitulando roteamento dinâmico
No código que adicionamos a esta sessão, utilizamos o state para controlar o conteúd{
  "_from": "prepend-http@^1.0.1",
  "_id": "prepend-http@1.0.4",
  "_inBundle": false,
  "_integrity": "sha1-1PRWKwzjaW5BrFLQ4ALlemNdxtw=",
  "_location": "/prepend-http",
  "_phantomChildren": {},
  "_requested": {
    "type": "range",
    "registry": true,
    "raw": "prepend-http@^1.0.1",
    "name": "prepend-http",
    "escapedName": "prepend-http",
    "rawSpec": "^1.0.1",
    "saveSpec": null,
    "fetchSpec": "^1.0.1"
  },
  "_requiredBy": [
    "/url-parse-lax"
  ],
  "_resolved": "https://registry.npmjs.org/prepend-http/-/prepend-http-1.0.4.tgz",
  "_shasum": "d4f4562b0ce3696e41ac52d0e002e57a635dc6dc",
  "_spec": "prepend-http@^1.0.1",
  "_where": "C:\\Users\\Marco\\Documents\\Udacity\\Web Front Avançado\\3 - Construindo com React\\_13 - Angular\\quizes\\FEF-Quiz-Angular-Up-and-Running-master\\node_modules\\url-parse-lax",
  "author": {
    "name": "Sindre Sorhus",
    "email": "sindresorhus@gmail.com",
    "url": "sindresorhus.com"
  },
  "bugs": {
    "url": "https://github.com/sindresorhus/prepend-http/issues"
  },
  "bundleDependencies": false,
  "deprecated": false,
  "description": "Prepend `http://` to humanized URLs like todomvc.com and localhost",
  "devDependencies": {
    "ava": "*",
    "xo": "*"
  },
  "engines": {
    "node": ">=0.10.0"
  },
  "files": [
    "index.js"
  ],
  "homepage": "https://github.com/sindresorhus/prepend-http#readme",
  "keywords": [
    "prepend",
    "protocol",
    "scheme",
    "url",
    "uri",
    "http",
    "https",
    "humanized"
  ],
  "license": "MIT",
  "name": "prepend-http",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/sindresorhus/prepend-http.git"
  },
  "scripts": {
    "test": "xo && ava"
  },
  "version": "1.0.4"
}
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    {
  "_from": "tough-cookie@~2.4.3",
  "_id": "tough-cookie@2.4.3",
  "_inBundle": false,
  "_integrity": "sha512-Q5srk/4vDM54WJsJio3XNn6K2sCG+CQ8G5Wz6bZhRZoAe/+TxjWB/GlFAnYEbkYVlON9FMk/fE3h2RLpPXo4lQ==",
  "_location": "/phantomjs-prebuilt/tough-cookie",
  "_phantomChildren": {},
  "_requested": {
    "type": "range",
    "registry": true,
    "raw": "tough-cookie@~2.4.3",
    "name": "tough-cookie",
    "escapedName": "tough-cookie",
    "rawSpec": "~2.4.3",
    "saveSpec": null,
    "fetchSpec": "~2.4.3"
  },
  "_requiredBy": [
    "/phantomjs-prebuilt/request"
  ],
  "_resolved": "https://registry.npmjs.org/tough-cookie/-/tough-cookie-2.4.3.tgz",
  "_shasum": "53f36da3f47783b0925afa06ff9f3b165280f781",
  "_spec": "tough-cookie@~2.4.3",
  "_where": "C:\\Users\\Marco\\Documents\\Udacity\\Web Front Avançado\\3 - Construindo com React\\_13 - Angular\\quizes\\FEF-Quiz-Angular-Up-and-Running-master\\node_modules\\phantomjs-prebuilt\\node_modules\\request",
  "author": {
    "name": "Jeremy Stashewsky",
    "email": "jstash@gmail.com"
  },
  "bugs": {
    "url": "https://github.com/salesforce/tough-cookie/issues"
  },
  "bundleDependencies": false,
  "contributors": [
    {
      "name": "Alexander Savin"
    },
    {
      "name": "Ian Livingstone"
    },
    {
      "name": "Ivan Nikulin"
    },
    {
      "name": "Lalit Kapoor"
    },
    {
      "name": "Sam Thompson"
    },
    {
      "name": "Sebastian Mayr"
    }
  ],
  "dependencies": {
    "psl": "^1.1.24",
    "punycode": "^1.4.1"
  },
  "deprecated": false,
  "description": "RFC6265 Cookies and Cookie Jar for node.js",
  "devDependencies": {
    "async": "^1.4.2",
    "nyc": "^11.6.0",
    "string.prototype.repeat": "^0.2.0",
    "vows": "^0.8.1"
  },
  "engines": {
    "node": ">=0.8"
  },
  "files": [
    "lib"
  ],
  "homepage": "https://github.com/salesforce/tough-cookie",
  "keywords": [
    "HTTP",
    "cookie",
    "cookies",
    "set-cookie",
    "cookiejar",
    "jar",
    "RFC6265",
    "RFC2965"
  ],
  "license": "BSD-3-Clause",
  "main": "./lib/cookie",
  "name": "tough-cookie",
  "repository": {
    "type": "git",
    "url": "git://github.com/salesforce/tough-cookie.git"
  },
  "scripts": {
    "cover": "nyc --reporter=lcov --reporter=html vows test/*_test.js",
    "test": "vows test/*_test.js"
  },
  "version": "2.4.3"
}
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                {
  "_from": "plur@^1.0.0",
  "_id": "plur@1.0.0",
  "_inBundle": false,
  "_integrity": "sha1-24XGgU9eXlo7Se/CjWBP7GKXUVY=",
  "_location": "/plur",
  "_phantomChildren": {},
  "_requested": {
    "type": "range",
    "registry": true,
    "raw": "plur@^1.0.0",
    "name": "plur",
    "escapedName": "plur",
    "rawSpec": "^1.0.0",
    "saveSpec": null,
    "fetchSpec": "^1.0.0"
  },
  "_requiredBy": [
    "/pretty-ms"
  ],
  "_resolved": "https://registry.npmjs.org/plur/-/plur-1.0.0.tgz",
  "_shasum": "db85c6814f5e5e5a3b49efc28d604fec62975156",
  "_spec": "plur@^1.0.0",
  "_where": "C:\\Users\\Marco\\Documents\\Udacity\\Web Front Avançado\\3 - Construindo com React\\_13 - Angular\\quizes\\FEF-Quiz-Angular-Up-and-Running-master\\node_modules\\pretty-ms",
  "author": {
    "name": "Sindre Sorhus",
    "email": "sindresorhus@gmail.com",
    "url": "sindresorhus.com"
  },
  "bugs": {
    "url": "https://github.com/sindresorhus/plur/issues"
  },
  "bundleDependencies": false,
  "deprecated": false,
  "description": "Naively pluralize a word",
  "devDependencies": {
    "ava": "0.0.4"
  },
  "engines": {
    "node": ">=0.10.0"
  },
  "files": [
    "index.js"
  ],
  "homepage": "https://github.com/sindresorhus/plur#readme",
  "keywords": [
    "plur",
    "plural",
    "plurals",
    "pluralize",
    "singular",
    "count",
    "word",
    "string",
    "str",
    "naive",
    "simple"
  ],
  "license": "MIT",
  "name": "plur",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/sindresorhus/plur.git"
  },
  "scripts": {
    "test": "node test.js"
  },
  "version": "1.0.0"
}
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            {
  "_from": "tough-cookie@^0.12.1",
  "_id": "tough-cookie@0.12.1",
  "_inBundle": false,
  "_integrity": "sha1-giDH4hq9WxPZaAQlS9WoHr8sfWI=",
  "_location": "/insight/tough-cookie",
  "_phantomChildren": {},
  "_requested": {
    "type": "range",
    "registry": true,
    "raw": "tough-cookie@^0.12.1",
    "name": "tough-cookie",
    "escapedName": "tough-cookie",
    "rawSpec": "^0.12.1",
    "saveSpec": null,
    "fetchSpec": "^0.12.1"
  },
  "_requiredBy": [
    "/insight"
  ],
  "_resolved": "https://registry.npmjs.org/tough-cookie/-/tough-cookie-0.12.1.tgz",
  "_shasum": "8220c7e21abd5b13d96804254bd5a81ebf2c7d62",
  "_spec": "tough-cookie@^0.12.1",
  "_where": "C:\\Users\\Marco\\Documents\\Udacity\\Web Front Avançado\\3 - Construindo com React\\_13 - Angular\\quizes\\FEF-Quiz-Angular-Up-and-Running-master\\node_modules\\insight",
  "author": {
    "name": "GoInstant Inc., a salesforce.com company"
  },
  "bugs": {
    "url": "https://github.com/goinstant/tough-cookie/issues"
  },
  "bundleDependencies": false,
  "dependencies": {
    "punycode": ">=0.2.0"
  },
  "deprecated": "ReDoS vulnerability parsing Set-Cookie https://nodesecurity.io/advisories/130",
  "description": "RFC6265 Cookies and Cookie Jar for node.js",
  "devDependencies": {
    "async": ">=0.1.12",
    "vows": "0.7.0"
  },
  "engines": {
    "node": ">=0.4.12"
  },
  "homepage": "https://github.com/goinstant/tough-cookie",
  "keywords": [
    "HTTP",
    "cookie",
    "cookies",
    "set-cookie",
    "cookiejar",
    "jar",
    "RFC6265",
    "RFC2965"
  ],
  "license": "MIT",
  "main": "./lib/cookie",
  "name": "tough-cookie",
  "repository": {
    "type": "git",
    "url": "git://github.com/goinstant/tough-cookie.git"
  },
  "scripts": {
    "test": "vows test.js"
  },
  "version": "0.12.1"
}
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         ze](https://www.npmjs.com/package/form-serialize) para exportar essa informação na forma de um objeto JavaScript para o aplicativo utilizar.

```
npm install --save form-serialize

```

O que acontece quando enviamos o formulario? Vamos preencher os campos e enviar o formulario. A pagina recarrega e na URL temos um parametro `query`, com o avatar, URL, o nome e o email. A maneira como os formularios funcionam no navegador é usando as propriedades `name` dos campos input e serializando os valores dentro deles no URL. Queremos fazer a mesma coisa, mas com o JS. Vamos serializar o formulario e avisar ao aplicativo que queremos criar um contato. O aplicativo vai salvar o contato e adiciona-lo ao estado. Assim o JS simplesmente vai chamar. Em de o navegador dominar o formulario, vamos controlar nós mesmos.

Vamos criar um handler chamado `handleSubmit` e chama-lo no formulario.

```
class CreateContact extends Compoenent {
	handleSubmit = (e) => {
		e.preventDefault()
		const values = serializeForm(e.target, {hash: true})
		if (this.props.onCreateContact) {
			this.props.onCreateContact(values)
		}
	}
	render() {
		return (
			<div>
				<Link className="close-create-contact" to="/">Close<Link>
				<form onSubmit={this.handleSubmit} className="create-contact-form">
					<imageInput
						className="create-contact-avatar-input"
						name="avatarURL"
						maxHeight={64}
					/>
					<div className="create-contact-details">
						<input type="text" name="name" placeholder="Name"/>
						<input type="text" name="email" placeholder="Email"/>
					</div>
				</form>
			</div>
		)
	}
}

```

Usamos o `preventDefault` para que o navegador não envie mais o formulario e atualize a pagina. Serializamos usando a biblioteca `form-serialize`. Antes importamos:

```
import serializeForm from 'form-serialize'

```

Ele faz a mesma coisa que o navegador faz com o URL, mas, em vez de serializar numa string e recarregar a pagina, vamos serializar num objeto. Dizemos que os valores são `serializeForm` e incluimos o evento `target`. É o formulario em si. O `serializeForm` vai "entrar" em todos os inputs dentro do furmulario, olhar `name`, extrair os valores e criar um objeto. Quando colocamos `hash: true`, vamos ter o objeto. O trabalho está feito. Ele vai chamar quem renderizou. Vamos usar `this.props.onCreateContact()`, passando os valores, isto é, `values`.

## Atualizando o servidor com o novo contato
Agora, nosso formulário de contato está pronto. Além disso, estamos serializando nossos dados e os passando para o componente pai. Tudo o que precisamos fazer para ter um aplicativo completamente funcional é salvar o contato no servidor.

Vamos voltar para o componente (`src/App.js`) do aplicativo e passar a  a propriedade `onCrateContact` para criar o contato. Estamos dando o componente para o React Router e ele cria o componente para nós na hora de renderizar. Mas, quando listamos os contatos, no primeiro `Route`, usamos a `render` prop. Assim ficamos no comando e podemos passar propriedades para o componente. Vamos mudar o `Route` debaixo para usar o `render` tambem.

```
render() {
		return (
			<div className='app'>
				<Route exact path="/" render={() => (
	    			<ListContacts
	    				onDeleteContact={this.removeContact}
	    				contacts={this.state.contacts}
	    			/>
				)}/>

			<Route path="/create" render={() => (
				<CreateContact/>
			)}/>
			</div>
		)
	}

```

Nós estamos no camando de renderizar o componente. Agora podemos passar a propriedade `onCreateContact`, passando o contato (`contact`).

```
render() {
		return (
			<div className='app'>
				<Route exact path="/" render={() => (
	    			<ListContacts
	    				onDeleteContact={this.removeContact}
	    				contacts={this.state.contacts}
	    			/>
				)}/>

			<Route path="/create" render={() => (
				<CreateContact
					onCreateContact={(contact)}
				/>
			)}/>
			</div>
		)
	}

```

Vamos chamar de `createContact` e passar para cima. Vamos colocar o metodo acima do `render`. Vamos para `ContactsAPI`, chamar `create` e passar o contato (`contact`). Recebemos de volta do servidor. Vamos incluir no nosso estado para poder adiciona-lo a lista. Vamos retornar um objeto com uma chave chamada `contacts`. Vamos usar o `state.contacts` atual e vamos concatenar com o novo contato.

```
createContact(contact) {
	ContactsAPI.create(contact).then(contact => {
			this.setState(state => ({
				contacts: state.contacts.contact([ contact ])
			}))
		})
}
render() {
		return (
			<div className='app'>
				<Route exact path="/" render={() => (
	    			<ListContacts
	    				onDeleteContact={this.removeContact}
	    				contacts={this.state.contacts}
	    			/>
				)}/>

			<Route path="/create" render={() => (
				<CreateContact
					onCreateContact={(contact) => {
						this.createContact(contact)
					}}
				/>
			)}/>
			</div>
		)
	}

```

O `concat` retorna um novo array, assim teremos uma nova pessoa na lista. Temos mais uma coisa a fazer depois de criarmos o contato. Vamos voltar para o URL anterior. Queremos listar os contatos de novo assim que adicionarmos um novo contato. Vamos trazer a propriedade do React Router chamado `history`. Vamos usar em `onCreateContact` o `history.push("/")`. Vamos criar o contato e voltar para a lista.

```
createContact(contact) {
	ContactsAPI.create(contact).then(contact => {
			this.setState(state => ({
				contacts: state.contacts.contact([ contact ])
			}))
		})
}
render() {
		return (
			<div className='app'>
				<Route exact path="/" render={() => (
	    			<ListContacts
	    				onDeleteContact={this.removeContact}
	    				contacts={this.state.contacts}
	    			/>
				)}/>

			<Route path="/create" render={({ histoty }) => (
				<CreateContact
					onCreateContact={(contact) => {
						this.createContact(contact)
						history.push("/")
					}}
				/>
			)}/>
			</div>
		)
	}

```

# Para aprender mais
Caso você esteja interessado em aprender mais sobre o React Router, recomendamos estes dois recursos. Primeiro, o [Construa seu própro React Router v4](https://tylermcginnis.com/build-your-own-react-router-v4/) guiar você para que possa implementar sua própria miniversão do React Router, trazendo um entendimento melhor dos detalhes de implementação. Em seguida, o treinamento do React Training tem um [curso gratuito](https://reacttraining.com/online/react-router) inteiramente dedicado ao React Router.

# Conclusão do curso

## Continue aprendendo
Excelente trabalho! Você aprendeu a construir aplicações em React, mas sempre há mais para se aprender! Dê uma olhada nestes recursos para ampliar ainda mais seus conhecimentos:

* [Documentação do React](https://facebook.github.io/react/docs/hello-world.html)
* [Blog do Tyler](https://tylermcginnis.com/)

## Comunidades de desenvolvedores React para participar
Interaja com outros desenvolvedores, faça parte de comunidades e esteja sempre por dentro de tudo que acontece no mundo de React.

[React Brasil](https://react-brasil-slack.herokuapp.com/), comunidade brasileira (app: Slack)
[Reactiflux](https://www.reactiflux.com/), comunidade internacional (app: Discord)

## Pessoas para seguir
Sejam postagens em blogs ou desenvolvedores para seguir no Twitter, grande parte do que se pode aproveitar de uma nova tecnologia pode ser encontrado utilizando os recursos existentes na comunidade. Portanto, gostaríamos de compartilhar com você nossos recursos favoritos da comunidade React, que foram muito úteis nos últimos anos. Esperamos que estes recursos também sejam úteis pra você.

* [Dan Abramov](https://twitter.com/dan_abramov)
* [Sebastian Markbåge](https://twitter.com/sebmarkbage)
* [Henry Zhu](https://twitter.com/left_pad)
* [Preethi Kasireddy](https://twitter.com/iam_preethi)
* [Merrick Christensen](https://twitter.com/iammerrick)
* [Christopher Chedeau](https://twitter.com/vjeux)
* [React](https://twitter.com/reactjs)
* [Tyler McGinnis](https://twitter.com/tylermcginnis33)

## Postagens // Dictionary of default apps to install into new profiles.  Each entry maps
// the extension's id to its crx's path (relative to this file) and the version
// of the extension contained in the crx.
//
// When an extension is added or removed here, the two lists 'default_apps_list'
// and 'default_apps_list_linux_dest' in build/common.gypi must also be updated.
{
  "blpcfgokakmgnkcojhhkbfbldkacnbeo" : {
    "external_crx": "youtube.crx",
    "external_version": "4.2.5",
    "is_bookmark_app": true
  },
  "pjkljhegncpnkpknbcohdijeoejaedia" : {
    "external_crx": "gmail.crx",
    "external_version": "7"
  },
  "apdfllckaahabafndbhieahigkjlhalf" : {
    "external_crx": "drive.crx",
    "external_vers