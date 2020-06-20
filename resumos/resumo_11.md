# Introdução ao Jasmine
Nesta sessão, vamos escrever testes para uma variedade da funcionalidade sincrona e assincrona, vamos perver e escrever o código necessario para os testes e vamos confirmar a passagem de suite de testes. Infelizmente o JS não tem funções de teste definidas por padrão, então usaremos uma biblioteca que fornecerá esta funcionalidade. Usaremos o [Jasmine](https://jasmine.github.io/), de versão 2.2. Acesse o link e clone o repo usando o seguinte comando:

```
git clone https://github.com/udacity/ud549

```

# Explorando o SpecRunner
Analisando o arquivo SpecRunner, vemos que é um arquivo normal. logo no inicio podemos ver os arquivos da biblioteca Jasmine, no centro uma mensagem "incluir arquivos fonte aqui", no fim outra mensagem ("Arquivos spec aqui"). Os arquivos fonte são os arquivos do aplicativo, e os specs são os testes.

Analisando o PlayerSpec.js, podemos ver algumas linhas familiares, como:

```
expect(player.currentlyPlayingSong).toEqual(song);

```

Esse codigo é bem parecido com o teste que escrevemos na sessão anterior, mas esta linha, e as outras, estão empacotadas em funções pais.

Vejamos a função pai, `it`:

```
it("should be able to play a Song", function() {})

```

Ela diz que deve tocar uma musica. Se subirmos, vemos outra:

```
describe("Player", function() {})

```

Quando abrimos o SpecRunner, podemos ver esses termos na pagina.

# Identificação de suítes e especificações
Comparando o codigo em "PlayerSpec" e a pagina "SpecRunner", podemos confirmar que as chamadas 'describe' têm a cor preta e as chamadas 'it' têm a cor verde.

# Introdução a suítes e especificações
Como vimos na pagina SpecRunner, usamos o describe e o it para criar um destaque, ou seja, eles servem para organizar as informações.

Usamos o it para identificar uma especificação (ou um "spec"). Um "spec" é um container para um teste, uma forma de identificar o recurso que estamos testando, como "it('Deve ser capaz de tocar uma musica')". Se as expectativas de um spec retornarem como verdadeiras, então o spec passará. Se retornar falso, o spec falhará. Penso no 'it' como uma definição de limite em relação ao teste.

O 'describe' identifica o suíte, que é um grupo de specs. No caso do PlayerSpec, há um suite chamado 'Player', e informa que tudo que esta no bloco está relacionado ao player.

Não existe forma certa ou errada de organizar os testes, tudo depende de suas preferencias. O 'describe' é uma ferramenta organizacional, que tem efeito de recuo no HTML.

# Elaboração de um teste
Agora que sabemos como organizar testes, vamos escrever um, mas desta vez analisando cada um das partes que o compõem. No exemplo será:

```
expect(add(0.1, 0.2)).toBe(0.3);

```

Cada teste começa chamando expect. Esse é o ponto de partida de todo teste, é o que inicia o processo. A função expect aceita um valor unico chamado de "atual", que é o que testaremos. No exemplo acima, o "atual" é `add(0.1, 0.2)`. Precisamos informar ao framework de teste que tipo de comparação usaremos contra o atual. O metodo de comparação se chama "matcher", que é o metodo encadeado depois de chamarmos expect. O Jasmine inclui muitas funções matcher, e podemos adicionar as nossas. Neste exemplo, o metodo é o `toBe()`, que equivale a uma comparação de igualdade. Por fim, passamos o valor desejado para o matcher, neste caso, o valor 0.3, que é o que esperamos da soma de 0.1 com 0.2.

Pense neste teste da seguinte maneira:

```
add(0.1, 0.2) === 0.3;

```

Se a expressão retornar verdadeiro, o teste passará. Caso contrario, falhará. Tambem podemos negar um teste adionando a palavra `not` antes do matcher:

```
expect(add(0.1, 0.2)).not.toBe(0.1);

```

Neste exemplo, o teste seria validado como verdadeiro, pois a soma de 0.1 com 0.2 não é igual a 0.1.

# Vários testes por especificação
Considere o seguinte spec:

```
it("should indicate that the song is currently paused", function() {
    expect(true).toBe(true);
    expect(false).not.toBe(false);
});

```

Esse spec contém dois testes, sendo que apenas um deles passará. Para que o spec passe, os dois testes precisam ser verdadeiros, porém, como isso não ocorre, este spec falhará.

# Introdução a vermelho-verde-refatorar
Na sessão anterior, tinhamos uma função `add`, na qual adicionamos os testes, identificamos os testes, identificamos erros e refatoramos o codigo. Essa abordagem funciona, mas o real poder do teste aparece quando o escrevemos primeiro. Chamamos isso de "ciclo vermelho/verde/refatore". Isso porque escreveremos o teste primeiro, e todos falham, pois não existe um codigo. Depois escreveremos o teste para passar. Após finalizar, podemos refatorar o código para adiocionar novos recursos. Vejamos o ciclo em ação criando um aplicativo de agenda de endereços.

# Elaboração do AddressBookSpec.js
Vamos começar criando dois novos arquivos, o primeiro "AddressBookSpec.js", dentro da pasta `spec/`, e o segundo "AddressBook.js", dentro da pasta `src/`. Tambem precisaremos atualizar o SpecRunner.html para executar os dois arquivos. O Player e as entradas de musicas foram comentadas.

Pensando no codigo da agenda de endereços, qual funcionalidade seria util? Poder adiocar um contato seria util, certo? Então vamos começar a partir daí:

```
describe('Address Book', function() {
	it("poder adicionar um novo contato", function() {
		var addressBook = new AddressBook(),
		thisContact = new Contact();

		addressBook.addContact(thisContact);

	    expect(addressBook.getContact(0)).toBe(thisContact);
	});
});

```

Analisando o codigo, vemos que logo no inicio a suíte chamado "Address Book", e nele teremos um spec chamado `it("poder adicionar um novo contato")`. Fizemos uma abordagem orientada ao objeto, instanciando um objeto `new AddressBook()` no spec.

Para adicionar um contato, precisarei de um metodo `addContact` na agenda de endereços. Passamos um novo objeto para este metodo, instanciado a partir de `Contact`, o `thisContact`.

Na ultima linha, testamos se `thisContact` foi adionado a agenda. Esperamos que, ao obtermos o primeiro contato da agenda de endereços, isso seria o mesmo que `thisContact`. Com isso, tambem precisarei de um metodo `getContact` na agenda de endereços que aceite um indice inteiro.

OBS: Lembre-se que estamos prevendo como nosso codigo será, por isso, não se atente aos objetos instanciados, pois as funções construtoras ainda serão definidas, assim como os metodos.

# Elaboração de nossa implementação
Se executarmos o "SpecRunner.html", falharemos em um dos testes. Alem da mensagem de "describe", veremos que "AdressBook" não foi definido. Chegamos na parte vermelha do ciclo de vida da refatoração. Neste momento precisaremos escrever a implementação da agenda, no arquivo `src/AddressBook.js`

# Iteração em nossa implementação
Agora que o teste falhou, vamos iterar a implementação para torna-lo verde. Precisamos editar o arquivo `src/AddressBook.js` e escrever a implementação nele. Sabendo que, assim como mostrado na pagina SpecRunner, AddressBook não esta definido, podemos ir ao arquivo citado e corrigir o erro. O que temos que fazer é definir a função construtora:

```
function AddressBook(){

}

```

Voltando a pagina, veremos que Contact tambem não foi definido. Fazemos o mesmo, porem, para uma melhor organização, colocamos em um arquivo diferente, em `src/Contact.js`:

```
function Contact(){

}

```

Lembre-se de atualizar o arquivo "Contact.js" no SpecRunner.

# Completando nossa implementação
Continuaremos nesse processo de recarregar a pagina, identifcar o erro e conserta-lo. Vamos aos erros e suas soluções:

1. `addressBook.addContact is not a function`

Para este erro, devemos adiocionar o metodo `addContact`, mas antes definir um array para os contatos:

```
function AddressBook(contact){
	this.contacts = [];
}

```


```
AddressBook.prototype.addContact = function(contact){
		this.contacts.push(contact)
}

```

Apos adicionar novo metodo ao prototipo, definimos sua funcionalidade, que é acrescentar o contato passado para o array `contacts`.

2. `addressBook.getContact is not a function`

Assim como no anterior, devemos adicionar um novo metodo, nesse caso `getContact`:

```
AddressBook.prototype.getContact = function(index){
		return this.contacts[index]
}

```

Esse metodo retorna o contato da posição solicitada.

# Outra especificação
Outro recurso util de uma agenda de endereços seria deletar contatos. Vamos fazer o spec deste recurso:

```
it("deve ser capaz de deletar um contato", function() {
		var addressBook = new AddressBook(),
		thisContact = new Contact();

		addressBook.addContact(thisContact);
	    addressBook.deleteContact(0);

	    expect(addressBook.deleteContact(0)).not.toBe(thisContact);
});

```

Assim como antes, definimos um novo spec, com o metodo it, instanciamos as variaveis "addressBook" e "thisContact" e adicionamos o contato à agenda para podermos deletá-lo. Nossa expectativa é de que se obtermos o primeiro contato da agenda, esperamso que o objeto não seja definido. Se deletarmos, ele não deve existir. Como ainda não escrevemos o metodo deleteContact(), então sabemos que haverá uma falha. O metodo ficará da seguinte forma:

```
AddressBook.prototype.deleteContact = function(index){
	this.contacts.splice(index, 1)
}

```

O metodo `splice()` remove/adiciona itens. O primeiro parametro é o indice do array a ser removido e o segundo a quantidade de itens a ser removido partir deste indice. Ainda há um terceiro, que é serve para adicionar o novo item.

Retornando a pagina, veremos que tudo esta correto.

# Remoção do código redundante
Ao observar o arquivo AddressBookSpec.js, identificamos alguns codigos redundantes. Configuramos as variaveis AdressBook e Contact em cada um dos specs:

```
it("deve ser capaz de adicionar um novo contato", function() {
		var addressBook = new AddressBook(),
		thisContact = new Contact();
});

it("deve ser capaz de deletar um contato", function() {
		var addressBook = new AddressBook(),
		thisContact = new Contact();
});

```

Por um lado, isso é bom, pois os testes ficam claros, mas por outro lado, fazemos isso manualmente. Felizmente, o Jasmine possibilita definir uma função que deve ser executada antes de cada teste. Ela é chamada de "função beforeEach". Vamos refatorar os specs para evitar essa repetição:

```
describe('Agenda de endereços', function() {
	var addressBook,
		thisContact;

	beforeEach(function() {
			addressBook = new AddressBook();
			thisContact = new Contact();
		})

	it("deve ser capaz de adicionar um novo contato", function() {
		addressBook.addContact(thisContact);

	    expect(addressBook.getContact(0)).toBe(thisContact);
	});

	it("deve ser capaz de deletar um contato", function() {
		addressBook.addContact(thisContact);
	    addressBook.deleteContact(0);

	    expect(addressBook.getContact(0)).not.toBeDefined();
	});
});

```

Note que as declarações das variaveis foram movidas para o suite. Lembre-se que no JS, escopos internos tem acesso as variaves declaradas em escopos externos. Todos os specs terão acesso a essas variaveis. Dentro da função `beforeEach()` foram instanciados os novos objetos.

A linha de codigo `addressBook.addContact(thisContact);` tambem se repete, assim tambem poderiamos colocá-la na função, porém não temos certeza de que esta linha de codigo será uma funcionalidade necessaria dos specs futuros que escreveremos dentro deste suíte. Futuramente ela pode aparecer dentro da função `beforeEach()`, mas por hora não.

# Como testar código assíncrono
Testar funções assincronas é um pouco diferente, pois precisamos informar ao framework de teste que a função assincrona terminou. Imagine que um aplicativo precise fazer uma chamada de API para um servidor a fim de obter uma lista de contatos iniciais para abastecer o aplicativo (veja a imagem "ciclo-de-informacoes.png"). Escreveremos um novo suite de teste para resolver essa funcionalidade. Faremos uma falsa funcionalidade assincrona. Neste caso precisaremos escrever a implementação primeiro.

Normalmente, teriamos um metodo assincrono do forncedor da API, como o metodo push do [Firebase](https://firebase.google.com/?hl=pt-br)

Escreveremos uma função `getInitialContacts` e, para torna-la assincrona, usaremos `setTimeout`:

```
function AddressBook(contact){
	this.contacts = [];
	this.initialComplete = false;
}

AddressBook.prototype.getInitialContacts = function(cb){
	var self = this;

	setTimeout( function() {
		self.initialComplete = true;
		if (cb) {
			return cb ();
		}
	}, 3)
}

```

Igonaremos a parte, pois só são boilerplates que fazem a função agir (como self, que pega a referencia do objeto, e cb, que funciona como `done()`, visto anteriomente no Gulp). A parte importante é o `self.initialComplete = true` dentro do callback. É isso que a falsa chamada de API fará quando uma função assincrona terminar. Veja que inicialmente, "initialComplete" é falso dentro do construtor.

# Elaboração de um teste assíncrono
Escreveremos um novo suite de teste chamado de "Agenda de Endereçoes Assincrona":

```
describe("Agenda de Endereçoes Assincrona", function() {
	it("deve ser capaz de deletar um contato", function() {
		addressBook = new AddressBook();

		addressBook.getInitialContacts();

	    expect(addressBook.initialComplete).toBe(true);
	});
})

```

Nele temos um novo spec, que pega os contatos inicias. Instanciamos a variavel addressBook novamente, pois estamos em outro escopo, e chamamos o metodo `getInitialContacts`. Lembre-se que esta é uma função assincrona. Depois de chamá-la, temos uma expectativa, que `addressBook.initialComplete` seja verdadeiro

# Correção de nosso teste assíncrono
Se observarmos o SpecRunner, veremos que há tres specs, sendo que um falhou. A mensagem da pagina diz "esperava-se que falso fosse verdadeiro". O valor inicial da varievel é falso, e queremos que ela seja verdadeira. A razão da falha é porque o teste ou expectativa está sendo executado antes que a função assincrona possa completar a tarefa. Voce pensar em colcar a expectativa ou teste dentro da função callback, em AddressBook.js, os chamando apenas após o tempo definido, mas isso não funcionaria, pois a expectativa será executada no escopo do aplicativo e não no escopo do framework do teste. Podemos resolver este problema usando "beforeEach", como antes, e tambem podemos usar uma função especial chamada "done", que sinaliza para o framework quando uma função assincrona termina, e "permite" que possamos prosseguir e executar os testes:

```
describe("Agenda de Endereçoes Assincrona", function() {
	var addressBook = new AddressBook();

	beforeEach(function(done) {
		addressBook.getInitialContacts(function() {
			done();
		})
	})

	it("deve ser capaz de deletar um contato", function(done) {
		addressBook.getInitialContacts();

	    expect(addressBook.initialComplete).toBe(true);
	    done();
	});
});

```

Apos refotorar, movemos a variavel addressBook para o escopo do nivel do suite, adicionamos a função "beforeEach" e passamos "done" para o callback. Chamamos addressBook.getInitialContacts, como antes, mas, no callback, chamamos a função done. Isso sinalizará para o framework que a função assincrona terminou de fazer o que era necessario e que podemos continuar testando. A unica coisa que devemos fazer é sinalizar para o framework qual dos testes depende da execução assincrona. Usamos done para fazer isso. Passamos done para a função dentro do expect e chamamos done depois do teste.

Voltando ao navegador, vemos que todos os testes passam.

## OBS

[OPCIONAL] Se você estiver curioso para ver o que acontece em cada combinação de transmitir vs. chamar done, um de nossos alunos tentou todas elas para ver o que aconteceria. Os resultados nesse thread estão realçados e vão além das demonstrações do Jasmine 2.1.2. Observe o caso em que, se você transmite a função done como um argumento, também deve chamá-la (caso contrário, as coisas não funcionam!). E esperamos que a equipe do Jasmine aceite uma requisição pull para atualizar essas informações para todos os alunos (consulte [thread](https://discussions.udacity.com/t/async-tests-why-the-second-done-call/40751/4) se tiver curiosidade).

Observe, também, que, se você está testando código assíncrono, não precisa transmitir e chamar done.