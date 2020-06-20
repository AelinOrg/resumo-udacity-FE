# VisÃ£o geral do service worker
O Service Worker Ã© um simples arquivo JS que fica entre nÃ³s e o pedido de rede. Ã‰ um tipo de web Worker, ou seja, funciona separado da pagina. NÃ£o Ã© visivel ao usuario, nÃ£o acessa o DOM, mas controla a pagina. Mais especificamente, intercepta os pedidos feitos pelo navegador. A partir daÃ­, fazemos o que quizermos. Enviar um pedido para a rede, como sempre, ou pular a rede, recorrer a algum cache, criar uma resposta personalizada, ou qualuer combinaÃ§Ã£o disso tudo.

NÃ³s o registramos assim:

```
navigator.serviceWoker.register('/sw.js')

```

Fornecemos a localizaÃ§Ã£o do nosso script Service Worker. Ele retorna uma promise para que possamos receber retornos de sucesso ou falha.

```
navigator.serviceWoker.register('/sw.js').then(function(reg) {
	console.log('Oi!');
}).catch(function(erro) {
	console.log('Boo!');
});

```

Se chamarmos "register" com o "serviceWorker" jÃ¡ registrado, nÃ£o hÃ¡ problema. O navegador nÃ£o registra de novo. Apenas retorna uma promessa do atual registro.

Tambem podemos fornecer um escopo.

```
navigator.serviceWoker.register('/sw.js', {
	scope: '/my-app/'
})

```

O Service Worker controla as paginas cujos URLs comecem com o escopo (no caso `/my-app/`) e ignora as que nÃ£o. Por exemplo:

1. `/my-app/`
2. `/my-app/hello/world/`
3. `/`
4. `/another-app/`
5. `/my-app`

Neste exemplo, o SW controlarÃ¡ apenas o 1Âº e o 2Âº. Nenhum outro com URL mais raso. Note que o ultimo caso carece da barra final, entÃ£o conta como URL mais raso. Pode ser ter divresos SWs com escopos distintos, o que vem a calhar, por exemplo, em paginas no GitHub, em ue diversos projetos vÃªm de uma mesmas origem.

https://marco.github.io/svgong
https://marco.github.io/trained
https://marco.github.io/serviceready

Escopos permitem ter um SW por projeto. O escopo-padrÃ£o Ã© determinado pela localizaÃ§Ã£o do script do SW. Ã‰ basicamente, o caminho para o script. Normalmente nÃ£o precisa definir o escopo. Basta por o script no lugar certo.

SW URL		   | Default scope
--------------------------
/foo/sw.js 	   | /foo/
/foo/bar/sw.js | /foo/bar/
/sw.js 		   | /

Dentro do SW nÃ³s monitoramos eventos determinados. Como em outros eventos JS, podemos reagir a eles ou atÃ© barrar o padrÃ£o.

```
self.addEventListener('install', function(event) {
	//...
})

self.addEventListener('activate', function(event) {
	//...
})

self.addEventListener('fetch', function(event) {
	//...
})

```

Veremos cada um dos eventos mais a frente.

Atualmente, a maioria dos navegadores (Chrome, Firefox, Samsung Internet, Safari, Edge)oferecem suporte ao SW. Como o SW Ã© um pro-aprimoramento progressivo, podemos usa-lo em navegadores que o suportam sem prejuizo aos usuarios de navegadores antigos, pois eles apenas nÃ£o terÃ£o os beneficios. Ademais, emeprega-lo hoje para melhorar a UX (User experience). Para usar o SW de modo seguro e nÃ£o intrusivo, basta condicionar sua execuÃ§Ã£o Ã  existencia de suporte, isto Ã©, verificar se o navegador suporta, se sim o SW Ã© executado:

```
if(navigator.serviceWorker) {
	navigator.serviceWorker.register('/sw.js');
}

```

Se o navegador nÃ£o suporta-lo, `navigator.serviceWorker` serÃ¡ `undefined`, ou valor falho, e navegadores pularÃ£o tudo dentro da instruÃ§Ã£o `if` evitando chamar funÃ§Ãµes indefinidas.

# Escopo

1. `/`
2. `/sw.js`
3. `/foo`
4. `/foo/`
5. `/foo/bar/index.html`
6. `/foo/bar`
7. `/foo.htm`

Dado o seguinte codigo `navigator.serviceWorker.register('/sw.js', {scope: '/foo/'});` para os caminhos acima, podemos confimar que apenas o 4Âº, 5Âº e 6Âº serÃ£o controlados.

# Adicionando um service worker ao projeto
Vamos adicionar um SW ao Wittr e mexer com os pedidos. Para facilitar, o projeto jÃ¡ contem um script em `/wittr/public/js/sw/index.js`, o qual estÃ¡ vazio. Se adicionarmos um `console.log('OI!')`, o resultado serÃ¡ servido em `sw.js`, no diretorio raiz do servidor, `https://localhost:8888/sw.js`. LÃ¡ veremos, alÃ©m do nosso log, codigo extra da saida do plugin Babel.

Conforma falamos antes, o SW recebe eventos. Vamos adicionar um listener para um deles: "fetch".

```
self.addEventListener('fetch', function(event) {
	//...
})

```

Quando alguem navega dentro do escopo do SW, ele o controla. Os pedidos de HTML pela rede vÃ£o ao SW e disparam um evento fetch. Mas nÃ£o Ã© sÃ³ isso. Acontece um fetch por pedido disparado na pagina. CSS, JS, imagens... Um evento fetch para cada, mesmo que lancem o pedido de outra origem. E podemos inspeciona-los com JS.

No nosso evento fetch, vamos registrar `event.request`.

```
self.addEventListener('fetch', function(event) {
	console.log(event.request)
})

```

# Registrando um service worker
O codigo no nosso SW ainda nÃ£o faz nada, porque ele nÃ£o foi registrado. Para comeÃ§ar vamos rodar alguns comandos:

```
git reset --hard

```

e

```
git checkout task-register-sw

```

Uma vez no branch correto, devemos abrir `/wittr/public/js/sw/index.js`. Veremos o seguinte scripts do SW:

```
self.addEventListener('fetch', function(event) {
  console.log(event.request);
});

```

Ele contem o `eventListener` de fetch visto anteriormente. E se, no navegador, acessarmos `https://localhost:8888/sw.js`, veremos o valor de saida do codigo, alem de um pouco mais que o Babel adicionou. Queremos registrar este SW assim que o nosso app iniciar. EntÃ£o vamos acessar `/wittr/public/js/main/IndexController.js`. A construtora do `IndexController` cuida da configuraÃ§Ã£o do nosso app, configurando o `._openSocket()` para as atualizaÃ§Ãµes em tempo real. Este serÃ¡ o codigo personalizado do nosso app. Neste caso, estÃ¡ configurando nossas visualizaÃ§Ãµes e se preparando para receber o que importa.

AlÃ©m disso, o JS nÃ£o possui metodos privados, mas Ã© comum iniciar mÃ©todos com underline se eles forem chamados apenas por outros metodos deste objeto. Note que no final ele chama `._registerServiceWorker`. Esta Ã© implementaÃ§Ã£o:

```
IndexController.prototype._registerServiceWorker = function() {
  // TODO: register service worker
};

```

Esta atualmente vazio, mas iremos preencher. O codigo serÃ¡:

```
IndexController.prototype._registerServiceWorker = function() {
  // TODO: register service worker
  if (!navigator.serviceWorker) return;

  navigator.serviceWorker.register('/sw.js').then(function() {
    console.log('Registro concluido');
  }).catch(function(){
    console.log('O registro falhou');
  });
};

```

Chamamos `navigator.serviceWorker.register` passando a URL do script (`/sw.js/`). NÃ£o precisamos passar o escopo, porque o padrÃ£o esta correto. Mas somente isso causaria erro em navegadores que nÃ£o suportam service worker. Poderiamos juntar tudo isso em declaraÃ§Ã£o `if`, mas em vez disso, vamos somente retornar se o SW nÃ£o for suportado. E como `.register` retorna uma promise, registramos uma mensagem de sucesso, e, em caso de erro, vamos registrar uma mensagem de falha.

Depois de tudo pronto, voltando ao navegador, se atualizarmos a pagina, veremos no console a mensagem de sucesso, ou erro. Porem ainda nÃ£o recebemos a mensagem do log `console.log(event.request);`. Mas atualizando uma segunda vez, veremos o resultados finalmente aparecendo. Temos uma mensagem para cada requisiÃ§Ã£o feita pela pagina, todo JS, CSS, imagens, incluindo requisiÃ§Ãµes que vÃ£o atÃ© as origens.

O escopo do Service Worker restringe as paginas que ele controla, mas ele intercepta quase todas as requisiÃ§Ãµes feitas pelas paginas controladas, independente da URL. E nÃ£o Ã© sÃ³ isso. Como veremos em breve, podemos mudar os cabeÃ§alhos ou responder com algo diferente. Isso Ã© muito poderoso. Por causa disso, SW sÃ£o limitados a HTTPS, a forma segura do HTTP.

Voltando novamente ao caminho que as requisiÃ§Ãµes percorrem pela rede, como ja vimos, vamos destrinchar essa relaÃ§Ã£o com HTTPS. Quando navegamos por HTTP nÃ£o criptografado, qualquer um dos processos no meio do caminho, podem remover, modificar, ou ate mesmo adicionar conteudo. Isso Ã© ruim. Podemos requisitar uma noticia de uma fonte confiavel, mas, sem criptografia, o que podemos acabar recebendo pode ser muito diferente daquilo que o jornalista escreveu. Scripts maliciosos podem apreender dados inseridos por nÃ³s, alterar bancos de dados, ler cookies, tudo isso sem o nosso consentimento. Mas SWs vivem mais do que paginas, entÃ£o eles podem ser usados para prolongar um ataque. Ã‰ inaceitavel permitir que um intermediario malicioso controle o nosso SW. E Ã© por isso que ele sÃ³ roda em HTTPS. Felizmente, esta regra nÃ£o inclui "localhosts", e Ã© por isso que as coisas funcionam bem no nosso servidor local.

Precisamos de duas atualizaÃ§Ãµes para ver as mensagens no console, e isso Ã©, na verdade, um padrÃ£o. O SW tem um ciclo de vida diferente para paginas. Por exemplo, se, em `/wittr/public/js/sw/index.js`, se alterarmos o log para outra coisa, como `console.log('Oi')`, voltar na pagina e atualizar, nÃ£o veremos nenhua mudanÃ§a. Isso ocorre devido ao ciclo de vida do SW.

# O ciclo de vida dos service workers
Anteriormente, vimos duas singularidades. Quando criamos o nosso SW, precisamos de duas atualizaÃ§Ãµes para vermos o resultado. Depois, quando alteramos o SW, nÃ£o obtivemos a alteraÃ§Ã£o. O ciclo de vida do SW Ã© uma das partes mais complexas.

Com a nossa pagina jÃ¡ aberta, adicionamos um codigo para registrar o nosso SW. Deposi atualizamos. Atualizar gera uma nova janela do cliente, entÃ£o vai para a rede, obtemos a nossa resposta, e a antiga janela do cliente vai embora. Pode parecer que nÃ£o hÃ¡ sobreposiÃ§Ã£o entre a pagina antiga e a nova quando nÃ³s atualizamos, mas hÃ¡. Por exemplo, se a respota voltasse indicando que o navegador deveria salvar o recurso no disco via dialogo de download, a janela antiga teria permanecido. Mas, neste caso, a resposta foi uma pagina, entÃ£o nos livramos da antiga. A partir da nossa paginam sairam requisiÃ§Ãµes por CSS, imagens, mas tambem pelo nosso novo JS, que registrou o SW. NÃ£o vimos mensagens de requisiÃ§Ãµes quando atualizamos a pagina pela primeira vez, porque o SW sÃ³ controla paginas quando sÃ£o carregadas, e esta pagina carregou antes do SW existir. Isso significa que requisiÃ§Ãµes adicionais que essa pagina fizer vÃ£o ignorar o SW. Mas, entÃ£o, atualizando a pagina, criamos uma nova janela do cliente. Como o nosso SW ja estava funcionando, mas nÃ£o estava "conectado" com a pagina, ele passou a controla-la. Por isso, a requisiÃ§Ã£o foi para o SW, assim como todos os subrecursos. Isso explica porque precisamos de duas atualizaÃ§Ãµes para ver as mensagens de requisiÃ§Ãµes.

Quanto ao fato de alterarmos o que estamos registrando, mas ainda continuarmos vedo requisiÃ§Ãµes sendo registradas, temos outra explicaÃ§Ã£o. Se uma pagina carregar via SW, ele vai procurar por uma atualizaÃ§Ã£o do SW no segundo plano. Se ele encontrar alguma mudanÃ§a, como recursos e bytes identicos, ele se torna uma nova versÃ£o. Mas ele nÃ£o assume o controle. Ele espera. Ele nÃ£o assume atÃ© que todas as paginas que usem a versÃ£o atual desapareÃ§am. Isso assegura que sÃ³ haja uma versÃ£o do nosso site sendo executado por vez, como em apps nativos. Infelizmente, uma atualizaÃ§Ã£o nÃ£o deixa a nova versÃ£o assumir. Isso se deve a sopreposiÃ§Ã£o de janelas do cliente. NÃ£o hÃ¡ um momento em que o SW ativo nÃ£o esteja em uso. Para isso acontecer, esta pagina precisa ser fechada (ou navegar para qualquer outra pagina e retornar), ou navegar para uma copia que nÃ£o esteja controlada pelo SW. Quando isso acontece, o SW novo assume e os futuros carregamentos da pagina aconteÃ§am por ele.

Esse processo de atualizaÃ§Ã£o pode parecer complicado a principio, mas, na verdade, Ã© o mesmo processo que navegadores como Chrome usam. O chrome baixa a atualizaÃ§Ã£o no segundo plano, mas ela nÃ£o assume atÃ© o navegador ser fechado e aberto de novo. Ele avisa, atualmente, que a atuaziÃ§Ã£o pronta mostrando o icone de download. Mais a frente, nÃ³s tambem vamos notificar os usuarios sobre atualizaÃ§Ãµes do Wittr.

Quando o navegador volta a recorrer	ao SW por atualizaÃ§Ãµes, ele passa pelo cache do navegador, como todas as requisoÃ§Ãµes. Por isso, Ã© recomendado fortemente manter o tempo de cache do nosso SW curto, como zero. Assim como o Wittr esta configurado. Como precuaÃ§Ã£o de seguranÃ§a, se configurarmos o script do nosso SW para um cache de mais de um dia, o navegador vai ignorar isso e configurar o cache para 24 horas. Isso nÃ£o significa que o nosso SW vai parar depois de 24 horas, significa que as buscas por atualizaÃ§Ãµes vÃ£o ignorar o cache do navegador se o SW dele tiver mais de um dia. Mais a frente, veremos algumas ferramentas que mostram em qual estado o SW estÃ¡.

# Usando as ferramentas do desenvolvedor para service workers
Para usar as ferramentas e recursos mais recentes, vamos usar o [Chrome Canary](https://www.google.com/chrome/browser/canary.html).

# Ferramentas do desenvolvedor para service workers
Com o Canary aberto, vamos ver as ferramentas mais detalhadamente. O console jÃ¡ nos Ã© familiar. Todo codigo rodado nele irÃ¡ rodar de encontro aos documentos(`documents`). Mas o SW fica forÃ¡ dos documentos, entÃ£o se digitarmos `self.registration` no console, que sÃ³ existe no SW, obteremos `undefined` como resposta. Contudo, podemos alterar o contexto do console usando o menu drop-down, o qual estÃ¡ por padrÃ£o em `top`. Podemos selecionar diferentes frames, workers, e atÃ© o SW. Selecionando este ultimo, podemos digitar o comando novamente, o qual fucionarÃ¡ desta vez.

Tambem podemos usar o `debugger` com o SW. Indo na aba `Sources`, abrindo o arquivo de extensÃ£o `.sw` e o depurando como qualquer JS. Mas diferente das ferramentas padrÃµes do JS, o SW tem seu proprio painel na aba `Resources`. O item de lixeira nos permite desregistrar o SW, o que Ã© util se nos refizermos o `fetch` do SW do zero. As abas dentro deste painel, nos dÃ£o uma ideia sobre o ciclo de vida do SW. Podemos ver que hÃ¡ um SW ativo, mas nÃ£o hÃ¡ uma versÃ£o nova em espera (ou talvez haja). Sabemos disso se aba `waiting` estiver acizentada.

## Quiz - Colocando o SW em espera
Ja vimos o ciclo de vida do SW e as ferramentas para observa-lo. Faremos uma agora uma alteraÃ§Ã£o no SW. O colocaremos em estado de espera e, depois, permitiremos que ele fique ativo. Para isso, antes, execute os seguintes comandos:

```
git reset --hard

```

e

```
git checkout log-requests

```

O primeiro comando elimina qualquer alteraÃ§Ã£o local que tenhanmos feito. O	 segundo altera o branc, deixando o projeto em um estado em que ele registra um SW e mensagens de requisiÃ§Ãµes no console.

Tudo que deveremos fazer Ã© alterar o script do SW em `/wittr/public/js/sw/index.js`. Podemos fazer qualquer tipo de mudanÃ§a, pois queremos apenas que a aba `waiting` entre em funcionamento. Colocaremos um comentario antes do request:

```
self.addEventListener('fetch', function(event) {
  console.log('oi', event.request);
});

```

Feito isso, atualizamos a pagina para obter o novo SW. Na aba `Resources`, veremos o SW novo em espera. Depois disso, testamos, na pagina de configuraÃ§Ãµes: `localhost:8889`, usando o codigo "sw-waiting".

Para ativar o novo SW devemos fechar todas as janelas que estejasm usando o SW atual, ou naveagar com essas janelas para uma pagina que nÃ£o esteja no escopo do SW. Depois disso, o SW novo deve ficar ativo, sem work em espera. Para confirmar, digitamos "sw-active" na pagina de configuraÃ§Ã£o.

## Atualizando
Ter que navegar ou fechar a aba para autualizar o SW Ã© um pouco chato durante o desenvolvimento. Mas, felizmente, hÃ¡ formas de facilitar isso. Se fizermos uma mudanÃ§a no `/wittr/public/js/sw/index.js`, voltar para pagina e atualizaÃ¡-la, veremos que a aba `waiting` ficou disponivel. Ele esta em espera porque a aba `active` ainda esta usando a versÃ£o "antiga". Mas, ao inves de fechÃ¡-la ou navegar para outra origem, atualizamos a pagina segurando `shift`. Isso atualiza a pagina, mas ignora o SW. Isso faz parte das especificaÃ§Ãµes do SW, e deve funcionar nos navegadores que suportam o SW. Ã‰ uma maneira rapida de testar alteraÃ§Ãµes nÃ£o relacionadas ao SW, como pequenas alteraÃ§Ãµes de CSS, e, como esta a‹      í½k“ÛÈ‘(úıü
¬;V7a<øìÏÙ‘fÇ;£õ<{s 	vc\ l©ÍèûÛoe=€zâE€Mù(è±š ª2«*+3++öòçÉî¸²}<İF»8Ú…£eœ¬>Şåáç|´WIäQ²»İ%»ğn™¤ë0%¯i–¤·û$Úåaz÷¦y´
âQG÷»Û<Ù“.È×U/=Û¨·İ}ø†À¥½9wË`õñ>M»õíÕfáÜ­’õ|å8…8JƒutÈn'ûÏw£mòÏ‘æñ§pù1Êu¿l3İÓD}ølï’£İÇ7ùîH‡·7Á!Îÿ-Úî“4vù³ıç4|z“ lß&»<@S–¹`ü`"ƒtt}£¡¿FÓq“Ş/ƒ×Îş\[ùîßÀ}wWä‰ëßÿÙÿÚ{ô±ïÜÿáÇ®ã¼ºæç¿€H0¸‰ÃMntüÇ£}ƒçw”¡Ç¯­kş·%†¤ğk™€°Ô£ÃşZ‹ùù§,y˜ˆÏTÈ¡w°ÏöOi²‰âğÚïÂİßWŞ>wM6Ë•ïÁátå®àÓ'ÉÓÎUZ¥°šh4õdÖ¤¥X45/¹¡ñ³¤iòé¸B,5¸ııïï>Eëüqç‡0ºÈÑôÛ(‹şŞ:ÏvÇÑ>‹²ã§‡(GÙ>X…H0|JÊø$6qòé–½yW<yˆÖëp÷l¯¶ùOiÉºà¤vœ ”w÷?çA~ÈÛh7¢ğİ‰ƒØö>É", ]=†z¹]†›$yRÌ‘B(§h¨Ö!_Ûö£mpf~> 9’Ù6ˆcû>Úü¯U°zÿä9îÂ™y¾3¾¶vÉ(÷a[b
&÷D—4¾‡°¦CÀU¢·AzínƒC”ƒ
–Yòğ.¥S–MöâßÑÄîI3¢bÒuàÃd³»ÿl¡N¢µuå®ás·Ö0·OËµg“pk	]ZŞ‘vµÙlaì.áÃ-¼kûá–â>¢ƒ&Ïõ/õn¡™Ş1³ÙLø9ÚŞ³Õ:â©O^±é¹xÉ¹Õ©:³WÉv‹¦poâÀçN™6î
EDÒi˜bD4"<ŸÈJÎÇØŠxB‰kO˜ãÓpfH/B¬'ƒíèÙûİ½†Ê`Í
J£åNJŠ‚¿ÍˆÜ˜~Á¢Uû$5n‹vjlœc6Ë¼¹L ìÕC¯ÿnÑæ‡Ua@A…4v‹ÿ|Xn£\”ø0åpµZİñ	Ë3FšcA6ˆ:İñˆ
Ÿ§W&=£2éi•IO§Lzæe±×aæx2Ú­gÙP ±Q±%F>ÚûhÏÌ+ú@”„ôØÀ‹†ÕÀ}·ºø©jĞw³Ñc‚d
~İ°1Jö;òÆ²Ã6‡_?{H>ı„·|Ö†ú.|D¦$q
Â¹ÜW%Õy­¸ÚíÊ
ÒG\ÚØOşpØ.³¶‹KZÑ¾K>í”Åƒü©ZQ¥¡‰:< qÕ¶ºúÛ¾²#¯
§Ç(üôWD:Q˜qr Î•Tæº -O‘TæNDd0±òbFA“ÉÄâ5Ãß¬U–?Åá-&2öˆ¬Z*şsàÿU=Œ®¦9LgÚQdß'éöˆô­ ¿Åêíh>³‘„%#F¿Âš¯5²æˆ¬¯µ=ı˜Ü¿³ú¨QJáI&aÂG';‰v+[Ç"êİ3.b\êyôÔÈ£§Z=Õñhx¨š(ª'  ‘ı~ìŞe÷ZÑÇ5É‚8ü%Z‡	7œ¾ÅMMjXÊjøXšw@BPË°ÙEài&H$#ş¤kEƒ‚î6ÿ.Mö˜(Š:cĞSGÙ»e3‹š¤8:#êµHc0oæµD«ü?„#¿È°g"ˆìx|³LTœ0@{¼®Õ‰ê•Òßm°A#äfíˆÈ¤ĞÜ5bR
Ûéa@Ö!XHiá“¦˜áŒÏ?èËgØ!°¼ô9z"jôT¥¤ PJ¸U¿ûç(Ú­ÃÏè‹ˆiˆÀ)ÛáÄ_ºpò<¯$EáPÂ^:ÒÆÁê	£Ï‰Âã6jÖñ84‡«ğ?“µzŞdÃ‚@>÷‰í°Ly‡ÇÈ´rjUå*`ğ-:èÀ3cPö#:ËÇÇ:RGgZ˜`:Ë©ó”]Eö*©cjÍx{ƒ@Pİ Go)_†M¡Q×ı}°ŒÃŸ`æğÌ%ÛˆjçˆÆÑ‘Œc GC.J,ò¨T4í`¸üÇÍèda	Î™iÆ £_Å^A³ŸäAÌm¾	6îZÒx1Sª8o	<–WüÅ²cø[¦t9˜"2‡­.ş›¨EX´ëòJ»°z&V åôÕm¢í½³êvö}{ÿ´GŠ=ŒL˜E	ÿÅô*ÃÁÃÜï©Ña;Š9ª4!Põ¬€+vIUª[Ü¿­ƒ\zãØÛs„r1ªº<~BÄ:ûâíñ’#ø^Õ€ˆaŞ<‰F‰ªÏf\„ÓÂgÊcÆş+s“ox5ŠX›ô/ÃÍRwk©$~iÃój|ö”¸ª°’q¬;Rz™G&Ş+q
ÕM]·Õ UñöD›ÚhŒÏ¢èÿJËšb¶u¹}‰ÿÖj?ïğ•İ‹µ7,ş
oÆ¯ÂÄ«î6I=1íº†Ë9|ÄÉ-×ßtÚ›¡sôõŞRx7÷p|4<”Œˆ7y ªT·âå#g\ğdÎ¿hÂæÁ;ÍŞÄÁı‘§3+Â]cJÅZh¸uÇ%gÔ HƒeVlí
¡³ŠÃ Eªjş LW(QšFÒ¹®ŞB¿z“	;«k{ÒÁ1GÒcEÔT´çÄUu»0ËÃ5ÕèÀñÍ¼ŠWõ
€ù«Šv]ÆYÕAÕ@ÿ‡ıÕ[â«·ÄWo‰¯Ş_½%¾zK¼„·D¶OQÛŸÃ8\åïÌ
1yo”áGùCİEİã.‹ğßÚë×ÃvPgà8H”Ø¹ÖP"İˆb‰±ÌwÙ$u-{<ZÁø`„d+èë*l³’ÈğÚmŸñ¸šyqĞ9@Rh÷pXvÃ$d«4‰cÑ™CÁ’èNô–N'øŸm8FğO¨«ñ+Íxò(Cù»4$iE‘‚îkïJtîq@›©C	=YGÿXÅA–ıŸ?åÁéG‡_¿9ÄV!ôô£†EÑ™_éÇs,•säu„:òÚÀ½ÊÂë>xöoìõú;¢"²Y?İØYˆMĞÌexÊ”Gìû;t¸}C– C¤ ÆtEéX ÌQvX"œ¢%]$é<Á»R÷ëˆÿå1ÉğÂú¢Y°è
×¼`£ùÊË\=”?ŞD»ı!gZrëK;Ó½wMã-=V”éÇÍ~»½ògğ¹SnxµâVÕìù¸Ğ¦“ò?üP4¬_UÆõÑ«AğöĞõÉ]>«‚1ÕºmQ¾ÇmúPŞ"ô±f£á¶‹é‘ú{ÕÖ©Ê4è6ŠÙŒ/ÛëMÓÚºÄû‡ ¤¼÷°ßÙä„¡I+0É6¼WÁ` ø?G´£Mtpií†‡/+šğ^1Rã‘Ìš‹·X,ÀpsØEùP¶QqMïôFÂ"y
'?„Ï¹iK®9]Ãõ&7ì?üü$ÆÙ¥ãæ¼³Cï½ôª'ù`‰ı'¤w 2U(ƒ™+Og{hšwqú·‰S5ä‚¾ÿ¬ôl¢BÁİ¨èo¦ë‚x­˜¨ï"|e<æ¢:á`»Óæ¸Gõnƒî»’M§œ¡ß/ıüj›?ö¦i'‡mH»x².Çc1e5ºqó§/‚ól¡ÑÎê`q¡
—"°âª1’¿_Â^ì0!tÚvIºbé—óÔ¡¯E»‡-“ñşÄ³^44Y£[Â´éµJ®Ñ9ğãı›äóû‡¿éMŞ0`/ş–pl0ØğNä²›ŞGÁ¯ºÜA§°Ğu¬+¬E¬˜uÔ²ÁOkô©¸1Ç_É=L†µ°˜/‰ŞÁ4‘ÎÍšçiòIót¢Mùí-:¥†óÂeq!¸èo÷‚³KAĞèË@ú€h5MÒ¥¯!Ü‚ïa\|øÆ«Ô¸™[´SŞ²rY¯Q†íâÜ³éÂì–Ï¿QÑöT%ˆñÈ9…çì«µ"a!ÆŒ¡bsÄqËÛ¬æ*ÈÂ;ÙT¥ïñN5ôî	­1'©8ê4Ï2kè•Á5ø~’YèäV•J·	
üğ&>„¤Yö¬æåb ŠÎïŞLÜÛŸ]‹çñÍÒ‘ªtÊ+Ç-VK0ÑÒ¿<{1uC»±Û]”_›Xœûê¹Z«<µû»­^Æ?§a¸;
ô–±¼ä»3.±·¸™87š%-—KoÖx‰Ç³ÙÆqaHô/:$ÒM1$úµÍŸÚsÍŸØ}İnİv½ÄËæD¦Ì›ô$J#&Œšñ%UåuT›7o¬{ªØÏ&\L¡Ë:s8ræu[×oEí8ØİĞ	±Â“pS¾fíŞß¤íŞòvïïã–ï·ì?­XŒ’H8!ç^ôô^Æt1ÂõÂ½ãéšQêL»…dÂ~¶Ã-:ùK*ÆdFDm¥÷JlSËH$o:=h+ô–2Ø‡:íEÓûd	ŸZ9p5^ÀçæÊàSÇÜ+_7Üƒ#ÅôÆÁ¬òf“&[ÖÇõM¼fı4bı*ğVŞ7Zwå.X¿XÔî(Ú øèÌtõ×å!|Œ6³&ÎÕ=ÔßõW·ï×¥»na l…o37®‹šu¶h°äè°Gˆ}M{˜3GW¿Ë°ğËŸlàS‡BEş!HCrKb6j§$rxV:°»jˆÂQLàË²),8&7–ìP>Ì-cdeğŠ…\Eb
¶Ê+Ü¬¤ÅD7ÈÇ¦:.Ó‘Àì$·<¤}¦ÂÔ½‰Cº4ÄöØÚ;zXëQËIƒ7ED&åöMÇ…Ö¶)ÌuÀ|È‹øüú«èœËˆb4§S#OVáÈe•¶/JÔÚğb¶b+`¸$˜\r×¤QuäfªìtÜıği"š§H=Ó<EZ…æ)ÒjÑ»2Úd^ô¨ÏeÔ‘ú£C'o¦V†ÓŸµÌÆ%@	™b:e¨NİrO¦’Âàp™˜AI²ïûb| O%8(cÖAè™3­o!±ÿEğÌáíòáÔ¸†=Ús:s¹bøÔ'±ÊÓ›,Õ_¸èá)+e£tF)[¸nš¡¯¹zOáS¯ØMÖğ¹¡>µŠ]ÕëM;ÒQìh?Í;	ø3x‡%øâ	Ì³ì|érä‹µf•ôuNèˆ»ÁÛVËÂÁ.‹@n!Ù®$ü\êµr-‹wkdÜ|Rk]£Í3ê‰®éùÜyïÅ¬dªï·JtÏÇ¥è®_Ûšm]Ö	wgÆ­¹g”#¨7]øøU¬¡_cû¾ˆt³Å’~©g)jçÚQ#:çøácOƒİÚÜƒëñ,JCªÆ¡šzÄAÆµ®Ÿª×3ÆÀc|}HMğ#‰A78Éä$H¦/&ñ]L¡®[•^‰@å.-: =É:ÅàÍiD´Ö‡(âÚôta¤ ÂŠ÷hË›òIõz»Ó©Øùÿ6ˆ="·tÙØ­±/2GqºÊÚqA[ì%³yH‹)'Ë`8­!÷…>q_sh§®qõ„şäi…gU‘B&÷S9½8©j¶X˜šÁR]+ù*Å5¥²º‰#$K7Ie9z.ÂZ×L ±½!YqGc»×y%Zˆ±•[ûªë2q¿aÕO¹¾ÅxThğz‰Ù™ùDW¸“pº¾™”^².˜gÓÑqê«;—ZO±¥M=—ç±¬ºJUBV	>¶†±#ïoÎˆYš>g’ıR™¹-‹¼·OöH]Œ±Ú\D4ìœÒ£U‡÷Hü…&5ƒ™¿€p‡2úï
é¥ÿÚfÖÖFUõ‰9»lí&„1et¢FÀƒÒ 1æ¹yL$3N =° c èaÂÖ{/A:Ò#o÷(¥R
­Ë¢ŠÂ§H¥ÂÇL6’=_±,	t+·ƒğ&¹¤+aÙóp«‹“ö-'ŞÌZ·=AÕå–Ÿì1.i*ÎøL´Í8¸o§àOÍ:—ó2rıVN¢ó§{÷È³‰Š¬xãRaÁş‹oLŒ§Ô=©¿‡¨+áq%Ã ¥ÎĞ|ñ9š*¦:I-ÓõO¼óFŞ|¶J*7TjšÉIKÃÊ™_ÅIJVaÇ‚‡ î¨iğĞhĞsŞëTZ59±™!9°‡UøÛªÕZà3.9 /£{Øfö5QÔA6›Ş¦ÿ­!1ÌC’'Ù·ñò°Í~Œ²»A¢á?"Y„6årÆAó8íƒ¦=n­iVÃücû1ÎÏ"öAQæSRNxƒ#:õ¡ÿˆèÏ%CşUöK~¶)ıZn1-ûåÌ8H½ûĞÑÛdÿ„{ü.mX®5iiKæiök ç55.Š¿-å9Tb­úlQ5,~‡‘-şRÊÆ~Á»™I™ˆ^òôóq‘{;Zv†So³?Ôj…3 Çrvİ•Ó¦æWnĞoòÑÜéÕÔBoÕ¶ãÃjfY!g'|f1¼¦œ©›[İ¹—{­*û4GÇZÄ±)á
O14Q+“bÓéÔÔ›Gl»rî=˜sbœ'¤«#&öBy9oæ:Uq¬ÏJšJÓ0må¯LğTÕr|&ÖQ¢S´‹4O¦J­ÁáSqM/8˜òùÓ>ü90ÿZòXN¢–V’as„Œô3ÚF»¼QÅ¼CbZBc¶c²‹[Jµg.©"—zúóZnYö…ãK­V:ôRtj;(¿b‰(¢Ğ•X,w¢·‘°Ç©ğÒ•t\36ÿkçÂ§®=76ÈÈX:“|ûÚE6õ†Ù!ÎK&‰_X¼%_Mw‡ØÎ“án„éÆ²œ™ é­hëö<‰Ã0-3ÜĞç‚”çiŞò´ÜÔIÆxÀ_>@+¹
oWˆÏV’ FQF§øÿpL9È¼[.-P<›„òd7v¹MYD4ØO‰ç"¶Ÿ7ı¿oÃuXÉ.~½4wV°[[¯Ëa¸ØÈv}<qªØ=.Æ‰Ó~E¿½‡=c5ñ£ ç	*0®HAçúãÅé£k†á®õ£/¾e¸Ş1V·yBÂ®ªJy^ºê0FÍ$Ò‚d©­XœÚïÛ€¸9¡“ŸĞñ)…éé=½îéğ86ŒµªúAæå‘ô¼Å5E}Ûğ39±+ScüÃ5N„Éé•^©Œø¨‡6˜ 1‘¬'­¢­Äô¤bC•½K!‘oaÃ6·Ñ"ğèsˆ[¢ZÛ2Q¨]­WgK4ÙPÿy…H5ŒO8¯â	èÛ£EÓ¸wÂ¥€gnÍœ¡¤”™4Á^ûÕtMöPüdRÆ>ìIò¶$¯ëÂd#nã^Ğ,óœˆlµqRúL€2Ü³<·¬“jTg3b×î=L¸ØUõ8'¢£Áß‡éßÚÊC/õôq'_)ŸŒ;#èûïÌ·ÖÜd÷ˆ?½u?âË~”û†İB§?¥Éú€­$/*ŸµØm-bM= 6+@}Á2°~T=J²ÆÀú“D@*MZÀïO4 :$;o~ ~ÜdÒO`¨K\Í‰†“W
à-˜‚ëZëKñ…;n÷´Äw¬TÒZœ‹ÉÑ¦,[[IlZ}f,ƒeÃ,_ûÇÎ§ç}s÷*8ûıš ëX²…×yşwl°Y'«Ã–º{öi¸‰>¿®·_™µ‡#óJéE8=Û¥gÍ-u9İÙÑÕiÎ“ëó©ó/˜
¬Ø1â¡œ„¥p´óqY ™”¹|YG^pËÎ²¬$ä>LŞv‹ñÇğ9w¾uíˆÅAA~ÍğÑï')¹càk°T¶*]y¿êFãÔ]
J/¥W{š®¿,èLÔüˆŞ×ÖÕÃC|›Ä¸¨İO¸>:$"’³rçº¦&×ğ)¸¨š!åNù­('Ê¯/(M™€§/w]“tèŠ„ß‘§?#VëëÃéH¶ªö½áLÁÌN»4¹bğ'øŠáDÀ®¡/äº‘Ø1Ìn¤¤:²ï˜ i¹²Ö,°¡)‹¹°@­¬7Ñ=8hèÔù¹x¿ÀÖÁÑ¬ 2CßÇoeŞç”ÉøïÖD¤éÛ²·{ÄUæpÌ$+Sã×RQŒw-a7ÍV^³%µÇHuÈ“É¬ê²‹»jF‰]gqv3`ˆ­¦ÇZ»Ç€Ö nNhÃ”O(¸İ
²½w@ê²×5%ÚØŞ%˜7,{_C!r†,|y1yÅ;¸yõ;K©bØIÒŠw]›jedsZ.“¡5Ï*ÔûC^¦;|*z[_‹“ª=ÚŸ
ÕGß03üBT0RGñ™1©BU{·a›=¯HÍ&|4'æÖ|ŒWË…L‘F›Fà‘ÔxõÊ&zWWßÈa®Óü¶ÿœcÀ;ÒkÏ¯p¬±EBáxÜ¸Òß‹¸®_‰8¨m×BZŞÒ”«6†°/û>²-†ÏÒÔYòDN2+ï%¼;?&í2ão†çíæ³ÍÌÈjg)ïç-7"îª3IZ³^şŠ$©X´£­o[Aò²Aßk–ÕÇ{LâÃ6u/Mâ¬!úÆÇÇ(‹–QåO·øÏ¸^áÄ_ÿ‚:Cı²*¯mÖX)W
Ø¥õ[~S¤bğm×› 3g‘}×ì5­`û™²Ñ¸_ısíC,Y:Eí­ä†{Óëš‘“‹bOâÏ|üIÒ8™Ü°ÿì‰w-^/§Trœ`úÀ]²V°Ê.W²Rj¹ B²>Ÿ‹õE_Êóy¸"zt¥;©ÊÄj{²òåê*´Q;>u±9oHØ?¥ác~úa[ZHšğ<ÛS.ù¦ó^÷ÂØ°Æº½€VùÈ‡ü
I³¬úÓkƒ¾Z$“„\*Ô!¶ á¾„­¦±Èİ7)€ÖÛ-øp}JqGúz›£o´9úZ›£¯³9rÙğÛ’Œµ¯!âë|eHİÕoÔá³ˆùÄ×
”ÙK3Ş>Õ8cµG_òÛ¬¶qVvGìˆç¯I€ıáúxVÏNºø‹Yi[‘"£JH³eMiR'Ë1e¸;mœv{š?.ƒÅ#ğV‹Är-R>ä6°Ø	8
ˆ1Î,Ğ_¸ÁuµE"š‰Ø§@]î˜,DN¢à°ÃUø>¸Wr[q™B…åN½’M\Qe},@²ç'€²á@ü1ÌĞ,ß?(x-èAÀ}ô
-aÎ¸êx¾œLV­”†ïŸ3s}á¼sT—ò=³ÔŸ\>9¥™ŸPˆí5Z8ñHcĞô<ÇsHù6HîøC(…faByÆáL‚ğÊbİ6€>ıùüğ‡Êª:ƒ2MÒ"×ÑNRê•ÓàÈÉ ¶hrãPu¡9qnÈ÷whWær>a‰§Åw‚³çho]«áïõÅõƒåéx]Qè®¹òtEïR¿G‹wHÃ¬|È¼À,ÍÃés9¡•6ÇÖ—.2Š,/Íådñf“&æD©GëÁWã:yPWJÆİ 4±ßËÀâˆ7;úƒÎ•Ìí4;qd¢KIË¥³µ\,gËYA­áÑâÆ?5¡ÌÚ´¤“Ñ†ö¢g`ü.&Œì¶diÚÍ×a¿‰.kLŒD;Äµ‚¸õˆ®vÉ·ë"'<äŸ/ö¿1ñ‰TİQï#R÷1%w~jf6Ià“²NpV).GNYÂ¢5›C÷¨ûÜ„ObéÅñ78)hÆ¿ÿïC ZEÂ¤
$õ‡/ÏA7,@§Dœ*Å1\"jæ¢¢pøD3äµf-‚é@?çilhçcêk°hr“§b‘Fán?½I“O‡†Ï¥s1¡é€\yMİM«kDñÓ®5;£†.54™›Ü|ìVña©^J¶kÏ}Åü=—³‘Ñ©¾7{¦vĞ–sI3sª=ëŠİWTg5\µ+:ˆp,¸ns¯#!mïíM°ûR0~à2¯ õÄò=qÙCúSc tšÜ—‰DÏªÊÕ)}É´5rp"R|¶‹¤±‘²Ìµ5ñ(ÿ€&ñÃ2ß}€×zK8ãÛ¯tùNáû ÏşRNCï\V)wP~Ör{uâj2=TÜ…úš{ùá±X âê©ËZv'9…·æÍzL¢š²ºÒø\[hP›D”¬/ı§DÙŠ8{1)>ÇÈ€õŠ®>SÕéXa#¡2;,{èú–%Ä­$GÈ ÅWp,bj;éábîÂM”g$KÕİØüƒ5ø“ĞòW.4n¾¯æ«*ã?~eÎ²#å\•jSê¯ò:÷vn ¾C÷QÓ+¢ Üşîw•f/ñß‘õSÇôà§×É§ØÈæJrœºÙjg¼ë<—ÒÑ!i6_„¾)ñú‚p(êŠ1L@‘zûÅ©ˆ‰Éë:ƒ„Èaœ|1‡nÎ&OMŒš¢uõÌM‚j?'#×™ôªR˜!5òúÖ´G0ÃøûÚ×0õa‰¼×„òuiE›4Ø†¬çÅĞ³R1ªÅPƒš61zÊ=³>ÚÄÇà¯ïIºš7jÀü…sÏã’ÔİjÁ¦“@bõ¤Öy¯ª‡[A2ø$ÌŠ~ù8:_H™ì)bÿ²}”æa'éæÛ#&œÿ{½pÖâIÅ8£ÖéË¢b<¹[{ÿ°ÿı í^›‚”œëô19$j¦&E\û(GDñÏPˆ YóùÙdê›šÏ+D€SíŸUdëîaÈ¼h¦ªUß”0Á,Ÿü…Xš'k½b2¯:–!Í¯p
ä=ò¸|8Ÿ¹µ^Ç=qHwù» §	Ôæ.ácHTKÓ{¶†N±Ô>Tëf¦¶Œ²wúİbäÒ.wì,òÇ´z£ºNã;ååè¸&å4™ÊÖáKkWŠé8‘×kkrŸ4O@©vL°İI_}a¼Ï+¶–×á½¥Ö£;Ÿ4Rı”~ øHY+=w˜I–z0¹¿ÃoãÚó—¹.!î¼eNN˜‰
¸®¦²8ydÍb%[¯ª@ÀÉøqt'KìoÒ+âRßµ%q!Ä!p©ñ‘¯ŒÔ‚EcZŠDuÚ×§!h;c¬ VŸ3Åü7ù|à=á9äëARŞĞU7¸ñVĞ¤h}Óc*ñĞiš¼pB6Ö†‰Ëƒ,>:GÖó¾†»ûâB'5Ğ˜^:npëÙjù û7…#‡#­â¹>Ä²6J?`©®Ë5?È+’œş]˜eT“£~84¸'ğ,h’½ó‰Ş•LôgM4Ü<h^4$h2¥ 4×(dm™?Š´Ì¥ƒh#¾®÷¹ªp¹ú[²«ÊÈÎë¸Üœ”¡]®¬¶ñWTi9ïjVdõ»¨k&Œ ø7àçGŞİbÖù¸PæÕa³0ûÎ;ˆëNq5 %:3FËğIzÖÖ“P§Ü	=ZÁÑ$Û÷½KòhÃÄj» 4}_oóPÜİ…ŞàTªÉÙ]=/‹ÄUë$r$lÍ%¾Nì˜~ç¼’øhWñ5UY„N¿À¥÷·Ş¾E’²¦cdR¹ytBO€ßbU¶¼Uµ³ø
‡}Bçÿæ=ĞÆÜZsÛ0çx©]p±F'ñYÛƒËá ¸
ßib/$_œ82mÃİA¼Ê/Â8Ì¶bÆÊ¥3ƒOØv“‡Ç¯ñ‡––­Û¨äÇ%%š³ˆø÷ñxt2Í¬× ŸDÛî#¸+lëÚ‡™İèZ´A÷Ø‡Œµ8*.İ¼‰¨KÄgàV­[Gû.I4ws~Ú s™Ã‡W1}ntòÂœ¦IÂN8áÌÑß±"ñ…kôx€U+‹Æ›J°Ô\köË0 Ñ	È =\HAPUÄJ¹ª…t›;»Qæ ÓG¸[ïÀIšŞ{}M¾+ì5õÔŒ'çEQ«Y·D°ÁÒªİñ_zwôE"/GÁ¼D
]øô»¹h0~OšX5|óXàÀÁ±hBö…3s«uF£¤(BeóÄ~¼î]á+¶† \Ïá¼Ã@Ã­0:>=d†@f…Æé Şÿ*¹Ê¼%¢Nó`‰ÿbor|{Å¬4ø»=óv†¡æğ4ÏÓä“ˆ<¤æé*Œc	xÄ»–Ÿe6¨qmleûÚ&¥X¸ÂXJÓ™q‘x,ñŒ6~L<`V^i\È\Ø)ïagÁ&H£Z_3-ïåÇÊÔ6ÌŠ7üÌ5ô96ì’‰)LÊ…ßvò­:ëdö!C®İ?z/Õ²Ê_‘xbgÀó`şuĞÍŸ‚erÈá¯_ÑhôÉÆÍDçAœ:+Ğ5ÊöğrC(	½bŞå¯r¸BJ%ıL§I?%âÕô3^,ı”C¨¡2†‹¤8˜ ºÆÃèaåEñ5Í÷¬»eıl¸WSıl~±T/Œ¢†ğÉ0.Ÿğ-{<yßÑGfOö}­Wq´ÏFŸ ´a¢
Áeƒª)ğ'Tp^q)í•8,{vu5*>×hò’Ê¸p.r#r˜WoÃ…s±ÛCÍ&$ƒ¸ÈMøÛa»¯£!×ñ.’ˆxÜ«©ˆŒà2ÉˆEÑa\$!=¦*ÉÙ†ÓñŒj‚´ğ'Ø}™VÅ¿Q@Ô †Ö.o²§Në°ñ>ñ„F\©³ÿÛìºÒø_Ş–+/HöÛâ=1iŸùM‹ÿvíœeKRšÆ@I[ÉG à|Ü^­ÎĞ²±7ßÛ$.ŠÖt—Ôú+€²åHïŞ$ÜL¿Pè*ü”q5]ÑægÇË²Im!sG®TÉ<#Zôñ2H¹•ã…òE«I}ã‘ãŸ2Ö0y!¯ÇĞ†_´ş”ÇN6Aà´uÚ9Ê²«Ez5s×îúâFZ'¾××`<m±ÕçfÇîx¯sçnù^å¤¡!ÅÏõt+[ò;x[±ÙClZ¸g)…2@Fıíå‡ƒõãöRÎ–²3,ë.|Ú§‡ï„-i³Ñ«q¸ÆX³‚Ï³3xöO ¸ÒHI:¢!ÿ€”ãL|Ú…ÈŠİ­w¨×$»SL«íˆe‡ë(‡0k>?{“¯şPHëî	“ó¨âı";¬~Ş®gSÚÅx8Mşlh¬Ş¿È8ÑùÏ;’aı±š—8ı™gÆ#c¯‡‹3à:ô‰£ı2\Â*ÈqNª£Mí…PN!X†ãX`@ÈÚ³åZ.—†Œ#‹…aùRx–Ñ}K7ì¡0É¶Aõ¹‘ÆJÚª±œo,„XJWNàéÚ^¸Õ¯ÊIqÿ®‰Ñ¹¤"éŠÍNL…tõIuzM=|9+f'xèï’u_Şê©Øé%ı|RdP½Ô©ƒe_îÜbô¸
n—4›ÍE“¨‚A_$
Ø_ ‘ªøÉ]&à£q¡sY„Ä^ìlJ±NwT«…HŸÏ^T?ÕiG£Œ,Á yëú¶ïC¦GëÛ4
âÆÎq Oï=‰Nx§ÈvZT›·Ú{n®0õå..ÍÆz­}|aÈ7•ĞÎ`O~kr[z§+õWZìã;ä3éö°0ÁHÙQ½2Sq°
–A}éÑsb«5iñ™]ùØßQ{çûa±·{™Äk¤o¼ØM],±1;½Èi¨ÙDæIçÑş}ŠÆu„Ğ^A’Àè…±¥E	¾ûDwãæÎ§›õ¦dhEQw>‡®ïNÊŸyş¢,
åñYâ©c·®ÌÙ9ı.ZN‰ŞQ£7_œ›k˜.’S:»óFµšÎ‹§ôD(1ÂØèLMAÙ£ p¡èñÒúK-ÎPŒË\§š+³"ÕVy)³w—1)èd78L«/ª.dùSŞâğöˆUÛ)âU,8‡•7	J~nËô·™½’{¸)®Œø«ß>/eï²-L£M®ß>„«¿À?£«A&œ#M“ªŞ0½‘¾âª¸; Cşñ†%ı†]ŠC¦¤nB&Í¿\Ÿñ9xˆCâÇà<‰C0VÌÀcò_^âè¥ÔK@¿•ƒ¬RÿS-w6öÇŞØ+õ>ôËœV{ñäø°|dáÁç$®Y§å¡oµZI·¾.g¬‹ãiëâxºº8/BSÖ‹’¦ZvVDX9¯e!VCZ-®,íLn)†dêßÑßÇÔ5UÌQxÂqø¬¾'';Ø 5¦Û\OË¹¦äÎÍõ´Á\ëß1”Qf˜Ìş”ªò\ŸÕÄªŸë‡äÓa–Õù¾!NPç:YpCY.ë_ ²ï€¼P,ÁúqıwÒ©Pw‘	f__Ú„ó§9£ıM)68×<qÇ¯îèvae¬òÉG³+e]p84aÓ|^ÎCˆŸ¥ZQ¹øl¦µ"eÙñ¤àıÎ`å÷‘Ğµ¡-`¨e@°İÛ(~ºÅ·pğîàCˆŒI™GnÕùÒâƒ j'qŒ]m&ğ¹+ÒDÏ]ÌfmËi¼v–¬ĞbX´=ÚÕaF‘ÏÔÓîˆ=},A«ubi9œáICÇº*ŸŠ"ŒŠÎ[‘}m¡¼Óò}\Ä4TCòñH‰ûË­ù(C\	$Ú†Ô'Ô–u,nÅ;æ³9×1KµlP-âh"ÒséEÎ´#\°»,÷;aER;§g‰¡F»ı!ÿGş´ÿÈÿ:°ÇO(Â›QœÃ¹óîô“áy"õåÒeeİ/`	—ÄF%Î ïò>¼õùÄad‡Õ
Øäjx³1®º	6îúÌwF'+LÓ$­ÔÆw¾˜-Ãµ~çQ#Êü’Y; Oşùíñï£8/+™à`õ/y*¢”4õ8'JcZ ô²iJ™½Áÿf&Uù÷®ÈŠ£c©á;5É.…™ÃYÕa>]ñ‘S_h†áSwŸ‚U|ƒÔzçr ÿß&h*n´¿&†˜ËÅ+>¥€;{1åŞ8WÔt6ãO"½ç"ï?sœy]ì}’îĞ®ßmF^}ŞW’7‘KŒ‹À™/rÈAÖì3Âé*‹Áu#–Øúù·ıÏP#«¬›éM_Zï”¿¯ªwãäÚ›50Äq†¾pn‚»³4nQÌ}ˆMµåá•Ë8ÌG‹"ß¦˜QrFÎsÊå¶´—Î’w@œ8ì÷òô ı³[ÎÓ’“ë|%u1.şI‘Ã/<
+x±4µd³)çtÂG6´‰Ó*Š_v^Wq’… ÊHé¥ƒ»öšîÕ·2Î/kÎ…²Ì[É­ÙÊ'X@ûgˆ‘ÑO”r¯Å'ò_¹]’ÿ˜Üß‡ëvGÑÏdª÷a™}X¦Z–©Î‡e*]FBşdÍ-¦ïß‰›C²a
'‹BenÀ„(ÍMüáËzya”?ÂM7˜˜âbkªTäC“¦ @†¾š-ªfèŒe
pöMJ^mà_s­
·K±¶DMÑ
·ká¶N]õ
·k·~ÇSa– Gvà­^#ÿ.ì} -r]ng¸6ík á§ŸèÁÓ†±Ü1u´<ƒKM?BL®w%Fì¿¼ñ\¢¡®§a²oº|€Î>…Z@l_A#³®ˆóË2H?|
òÕÃ|ÍÙÙr5$ Í~"öòò>÷ß¢-Òó`7xê ¡Çlâc›f¡ÿë‹Æ
;¶XÔ‡h½wÂ(ŞÃçuÖ;Ï¨u¾à>xÂ–óÏ
¤{íóŸk€å;£]°Åk/#ãã$ß2…Ÿ9‚ò|3rØ#5<\Ç"^¹`‹cøüëŒ—ä†}TŠå÷äÜYè•he4^—Ìyè€æàdo.Wƒï/&ÿ¯6ë”ƒÊ–ƒ3D2p5Ïšl‹ƒ‹NÀ8<[â$çË;.à:ø›'t(¿?g@‰€yş>¸ÿÎ5ÔÊ:~u§8#b÷Õ,Ì‚ÚCÑéRšºWD¨–ÛjÊrŒ*ÄXCÍã(S[)øfÎ«"Æb|?#ª<J|­ã]É˜‡Å¹…iKãPCŠÏæ3©rôZr¼˜‘SŞ°¾­-ğaJ;Ÿq *L
yYÎ.ÚOèñœéø[ gEºh6ÙÇƒ÷ÿ“ÜÔ7ç´ÓmDÅ?½o_k©…QÂhÎx…/iFğ>ŞÛ-$ÄdÖÇvèœ_â]œ£»åsw|æ†ÏQï÷ê$·~¬uôhÿ–}@
Ó=\Q}e±x¥wŸàò>G²X˜?¥!TbøV«lïI±qM±SG¹²ç-(	¿•0ÔV ­+RÌì'<šI 7%S àÌjôàûç¾r\…ò”GÙŸÏë3XÈİAvVÇ—Õóƒ‘ĞSRøa¨	QÑ CchÌˆÍÚÇÿœ0F‹ı6Š“ûä¨úıs¡ùùvRx…r9ïgN¥'FH8&R òìi
é>xÂĞj}§8Ã=¾| ãëk´ş0€qüC§CšÖXpó?
İtF‘~®ş‚¸jH„A•–ıSf¾Ğ0ŸÉâ± ˆÉ»¿c§A
¸–][çcáÛ.zÕX­NÈò2-â99c‚Àß-¸)ç®\^júÄ1bç)~ ş ³º’	Ñc•_NÇ|úŠ»Sô&RşEËí3%T/È{8ÕìË¢i>¶É@uš­‰
(ï•är9ÌzŸév¦ø¶ì8ÈrM&Úv`
6¿wá&Ê³¾¼W¥%GŞ.p+mã¡İ€#å·nƒÔÆ:ÃKFå.7Ë‹W‰f¢Şü—U:Ï:€0eø‹W²çAO3Â<¼»äœ—a`³rùvyš9ZñìŞx<+²£ì]x”Ç„Cû$C±…ä¦ÄkaJPA93&çşäØ¼nÂ¸•¯uq¯<––»"=Ø>§q÷BÎ¦¨ÚNéï”c‹‡á‰Ş‘vbt¤hi':GZ9«Xİ¨Æ9ãå-M ¤zcs+GÒê¦×9"b†’Ğ‚3e-¸}7¼ó›¸Ø5s\“í“é'JÜ½2à«ß2j³ Cÿs˜üm<QŒÍõşøÕ|³?êİåÇ~·ÖAì‹l0
ğ¢Õ h`HWH$ú‡‡5;ºJãçoíQkR‹N!Btn^«@Šş]ëDÖ> Ö 	cQC×ªJW˜	…ë_`ê²·x&Êë'svC;Ë†]Àèy<å£t%Q+»±Ö93uç´È<¬G¬j9î	'ÁUmÅ¾Ç3›Røã¿ ÿ÷µsMx”ùÇÌü[büÉØ„1>´7Á*=FY´Œbğ{§—f„eVıUı\Õ’Gû#Û‡ØÂ‹B5O3õ¡ú–yvıõkç}ÌSÌ¿¡çò…ª›ˆ GÑ.PË!é_G4…áUŠËdÜØô?'?i8"Œ"'Ê&°]ñkk§™û!ÜâİıÏc¾ÓYµy¤6ê²êJ3|-?ëLì.muêa]¯V´IÁ‰k‡A
!ˆÕ™i&²¤ÕY½ô@Áö/ÙYòe>?å’W4TÊ´7ÌôNIP®ü1|¤»H%Ëı§SÛ”W´*œÚ‘¬Î)oÈªôÂ	Ó(™YUÓ!Ìº„ôØ?¥uT\<õ4Ôu0ZënµÁšo>bÍ[G—²PÔJxji¸Ÿ9`OàÂ’ŞjÇÍV”µèaèøÈ®8m›7«õ:ZlDá™ƒpŸÉ©Nğ¡¸.&¯oNÆ=n¼4@š¦óFĞN¹%Qa<oÎí»¨Ü|íGW·ó4Ãd6×% ëalö=ä UE©iÊ¹¸Ñ%'aEòg.¼¤:KsøtãÈøò!KØm´ p‹ø@.Ì7† 5â|R¼EHH÷ £ğ¬®ê·Û²o8ìUUÉ»“i
}¦B—P3+Öš"UŠŞ"Tşl²qh,Då¯KûñÙÂ…ÕÃüQ«QKÁmñLµšœ;‡h ëMJo}[Á‘ãÁú²Ş`âi.kŒS°iã,Ş€—yÍ”B+ŠT¦“ç…a»j5{§N¯whõr'Š'TµFïœªÏ{Ö¡á©];ÅE'ˆƒNa`aùıùa™«…Õ[uÂˆ²[GÅĞNFIî‰Ãë¨•ÁÂ…6öº“29È¢¸yÖ™F3ÈöóÏACâ	ÃV;SCjÕº '°
 5d¨ŒÈıI”B“|îu˜¡òJ™j÷8û$[ñËL…`îÆ7/4|Ë¶÷úï%ç¡@¢°Öûí^}ÎüË.~Â½ü=Zß‡ùEL’/6oØ£Ó´ÉÙoÂQt§Zpjõ §Rpªt çD€}ù !€zıËnœñµU´C'™ +?Iéá¦¤'*Sœ–8›”XC¯.­_ÎáÆFÈÉàPÍ—§vˆ)1İ¦!qH_³ë9FÀ˜´wA“ZÌ¥×~ÚÁÄÇı>)¯mùĞÅ"8iŞô(r“½ù×²óòp›á4#Ôf°uá€ô›†üáª]¯Å`|fàÖÈMRÍ2Á”<P–’} 3}f¿
5F6ïTt«0¤-kfaÔòôTpÁqÕà5š3ƒŠÔT¸–9Rò¬“Oh\¸¨W%7Ñ.s«xä®á#†š4nÕêíæÓÎ®úcóLõp«Ğ²µ D?–C'“ˆO—¯N*'æ¨E6sÎq‚·á@/|K†=0G,‚¼,‡ÿC Ş1N½±ËF«‡(^¿_K±0v=gŒz14(˜ø{ÂÃy—eìÑjh¦½ÔX\W¾ÄöÎ½6¼Ywãunå!¾8‚ŠâKåVx»3Ç^úÙOXı%
7D=€|GÂƒ4](ìÂüS’~¤	äFàïá>İ| \e1ßøÉéDÒıvDÌi™ö5FW!¥$ç(ÔHÿÆT–ÆÛ}Õç€X.y1°»Í=ÍXõìå¢¦†‹Ê¦S– K¾~õÔ\;»6Šİ©Oô6]¢ÛqÓğĞYmöØÀÅ¤ìÊü)°?$9Ù;Ù]„Î~´ûÃäú(;ÖLZcg,DJpt=HÅ~ÓD…é¸şX‰3æs>Íñ1›¯±±¶i?öcJ¿’+±ÀVê|õMÀˆœ{ ‰˜…q¡H–÷­X•'Ç­.w­À¹¾çËğb‹¾zv»0şö1Èƒ„üMó¶ëÇuËûdˆƒô-Ú­1¿‚ÆÔ¿¸E„Æ‹¥Z,øhª¦Ó†È?Cô¾-"i´‘ıJ!(Bª›(¼rc{Yï4¢Á”§šÆt¢twpª~çŞLÜC.İh°¢]Ò4ùTVD`Wó•Â@ìî¿F
_…„†èoöJˆ;`qcJ
Lş’k×5qİÙ>Jsƒ¢Í÷E´ìÛRßfN(³óÀ’ö4ŸæQ3%2§ïÓ–#º0!z¾¹Z¼2oO°(ûO¬·ÇÚ4­š¹¢¥_Ä"|öÉ/ÍİÆ3ÇÏCUŸNîP"Ğ;İğˆp`óòÉÊ˜øãY•\P	ïX@‘^j ù~—T‰½ïÏ8ìêhü8ÈQ˜ A†Ùt•ğ¾OÈ3„H4‹n`®«IíWœ¼ı*&§ôÖO£±„3‰˜â:hDŸd‡¯Rê1Ğ¸UªPšÊnÑ§&…ğCNAI>Å$‹™…¾J‰hŞ%œÙ°pG’ìIĞ7»©Höy´Eó‚íçiñù‡‰¯>—GİŸˆæNÑš† <ùîF¸¾ôh&{¨â¢QXhÿiGsb™‚ø«[î µ^ÁG¼ÄU3Öiiaè_Âtì‚B]pÅË‡:‡^M§S~b¼	Ÿ`~ôtKÄ8%Ö¾ˆ¾ä3–™g3sS¤+mª×¤ô“ÌßŸaiYş‡#(ÎçãÒ^Ùä–ñC’Á°'’Sj-÷€ ‚nƒŠÄ}hÂDÓµ¡§Ÿ«6ˆ î‹áZìØlÜfeº2ìÛÑÎ`Ô&jõè³Eˆ—ÚÀ¨¡c'ßÈû‡³Ù”õŠè[8íA¹³÷°Á6›‚ù‹ŞóäÖ1ì-)r9a5c:¯iVqå­<1€°ğ·2µ«ÉÆ
0²6†A$ÖVá-Ú¡iLÂ3üwëÅåjúš>İöC¦&],Ç!'ïO›qO>&AÍUƒÅÍCY$›«ôÑIô£f£Wù’™¿Œ‚UpÀ(€år	LG—şÎU®’9ÛYÓd|î§	a°Sbê“~‡û+¤Qü”FAÎ~¹b@d(Å9´–á’˜ñ§)ÜÛ`“ãòEmQe×¥÷Ë€Ä~ßØÓë;¢-óš"j×äà`+‚Ût‚Èö¶3ÿ ;óÈÕ}WÃÚ˜2&÷¯K''Q—š*pEBKj÷H	á#§Äá“±yJ½TâEãÕq¼†`i3æ 1†OÕí¸È_'Älnì®¨7Ee”¢äØŸ!{ç*üÏ$Fråïâ@²j
ÅGÑ¨WâÀç\êô¶¶ ïcÇˆüàTCø“²øØˆcïØ} _î—âñ–lË	.Êö£—z,).Æ…w¤œl\ŸœŠ˜œòäş>ÿ÷!€üª~Çó‡ùµ¤	ŒË!!}’ Í”38ü~q$œìğ3âüktèT€[c&´°ìÿ&ü]5e3k¹¾ì¡Sh2t{„|dÙ¦¿€0rÌ¸«ú9»Ü6fU3 ZxejBé›#>-êúHF.Cx±Ğì,Ì%«©Tš("81yıÖ!ù†ñ(â­ÛpÊ”NÃ{¯|u@H¢=}Çìí6"h•t?Ü	©Ñ†B]Ù›ìNpŸBºZÄ¿A§ *Rªw@£®(ËúFR›EƒÖıÆ“ÓhSE,nEŸ¶X0Õ+î`u¿câ¨§%>-š~+à kmÃÂÕ„f…±şN¿>íz¡¢=Ólê£‡|O$ÑDX$€¼î †ßª›¥grı¡È©Ï*!pÉª=î˜/ ¨>ÑÜ‰¸‹pdÑËÏÆ³fñòµ%7zĞ]m‚Çœ!¦ƒµ'¶+œ.5¨Zæ±LÖOû&—¬¼óGÂY<kûÆwî\×’˜%Ø»äÏi´ş1xJ¹ı(P7±.IÖštÌqî_uŠ˜&æ»ö¼"{s}v§şqNËXxfÄ²ıWı­ƒ4õÁâÎ[…Tmç‰3<,øĞošŞï$pjŒŸ{a¢‰Btñ]WÎÕdıA·ƒxÿÜ&Ù†÷ÈN»§,î±?pŞ“»?Ì®EŸ•^‘ õ§@M§7šf:B÷(ó*k¢”»¡Cc»#%úvN8ë§`U† ŸŠ²ƒæcíŒ™Ãê ¦L¦ÁÃ1‡ÅÎÀ+{&sÒ¸Ze€bÔ¸¨FÉ®Ã›²\Û·ÂEªÓKû1ˆm0$Bªé”[ÜßìİÎ¸ÁùÁ(H4å¶’xï'û Œ¾‚\Ñ‰Áw§³yâ2³C•0¿¥i¯óá‚‘Û¹àåØflšæâÜ×Ñ„ğ„Áu½KÖaœq>ÅOÌSDpË°”+‚»—ÒUÑpæX`¿=÷ğœq”B˜çŒÏWÄ¼+šŞYäã+öû7ş\)lä¹6O‚ı˜’À*Ó%>-/†9r€`YAÌ‰MzggâoŠrŞ¼ Îvå‰Ï`
Ñ±`KÔÛsyĞˆı¥ŒôX`x=TµSíQ¶# ¥Ğ\Ãğó/b‚ı »Ñş¬1Ñ›-)é¨™•şNcB¦z¢\*’‰¦%Yñ	T?0Œ M·~“1Î¡…Î9³D(©'7ğÑŞ«v 6;òZ‚ålÒ}Aí*“}­o7QšåDŸ-|¸ßQç®º¤Üzsê°{]¹øœ\™ù$•	Œ0ÀÄ¥ã‹8Âj£^ÓÑxMGãÈ#ñÚÄk4’iÕ@àÏŸó oÆF@Ê×õä-p13¡rV7á¾OKÛ"ª~Ó9÷S.ó O¼ßnâın?n:šqãÑx®fCŒÛfÜx4ö6Yñ»÷‡e˜Ùbºñ÷IÂi	GV¼Úiğ²eS~L^QM›ô'Yˆ¶¬l9gå‹|Ö,E˜£O’îËÑnSÑãMÉMÇ¥÷ 7•w†›^—İğº&o¯×Ä“¼íOßfóH^PÒZòÔÆ¸A¯¼0â¥˜†Š}0aF2¼Xep7ùVX)Ú´7Ë7pêSĞÆ“Éf1mÙÛ:H?ş95WçWş>MûûÏ0@tÂ—à³grÉ|oÖ¦;â5¥/ÈI*­*dÈßtµª¹h6^ßà¾F¸qSmM‹ÒÁwâ¥Él1›Ïæ|6¡Ğšd¡˜4íı;tà‘».İãšöŞëßÂÕv•ÿÖéÓÈV—çŠS¼çH‚¦ÃxŸìßÁ‹BïZ]Ÿè›Ñ®{qûtÚÊÚ®ªuÌî\®Vå‚Sf“æ\P‚ıóa©‰¤á7cİºîeÒªœ‡m—M™!+êrs	uN\nBËe*\q¸Rqxö;Î´L>Â25±m–jŒ	î¿Ó¿|iÄæRïBBÑ˜Gê>XWMßTvóI<J¸¨ê²V’å€|hˆ~vC{ÇÅz”‚J2îèëÏÉ§#÷"ş3,İZ¨´p0sĞÔ ­"”Ÿr’ô‰?¸’ÃeqéÉ'Ğ>ïÖ×bŠU»tM=%4ÏVˆ7é(9äJA%å÷¤ê×ºŸÄRÿÏk0íÂU™:½l9µÍMëaÌë…tâì)Õù]6ÖêHJ‡{½i/Ê…)tã)Ù¸jö=(zÌİ_zA¨}-ƒ„hïp§’÷€mè=yÑ1ÙºÜšSÍÁÜ¢.ÉrYØˆ;NÈş-Øí×•Ï'O1]&†pz‹ şKœÁjÆZyRšO¹“Æ81Í”+yc`:”›‡k!&V[ÿa‡7cÊ+$!ïáè—–L¢ê«H‰Ç­ÅÑöÉæTï-Ç*VZT./NRÅ¢YLÖ!3NJs8ìÚ&ÇZ¼u5nn¦qÃR~‹½ÏñÆ³ììi·zŸ<Ñu¬sWèrä5uyØ\íºÊªAJv2}=âÊ¡T¬`³Ş ~Ûîš¡B­°g<Â§8O)üœT	ŠÆl˜MVüX’FÈYXñS9+dÑ6ß])9Ÿ5©XåÂ,úJ"–ºòºn¨äÏ êÒ97n×8gó‰×•4'$»gZZ%Ê½Ğ–½ŒéRN‘M%á))÷|ä´’a±Tì q+R±r'‰B.¾÷ä1Ab!ŒaÙ’œL˜ÛIÑvƒBju3§!¾¦®XˆËÌsVŞdâ½CÌD°6!I,ã00çr¨n0Ìæx> Ì{=Ìéx@˜‘æÜf¬‡¹’†¶ºu‡$"“÷’7$%¦‹É!ÉÈdYšIG™èlHBÊ@CÒÁpt†$¤O Ş„ôOƒ+–0ÃÀ\2mH¸Ë
¹6$\ƒÇ²mH¸9åÛp²|î× ÏCÓÕC•¬°A™ ònHÀ¿UÉ¼!›ÔŠéĞ´eR-fC—I½XM\&S¨34qÔ"‡¬W5Kˆ`„¾KíÍƒÁ}ª’…ÂÕëTweĞ9ˆ,®AöÏádğb`ºZd0•…6	aohÊ2	áñĞ¤õ±RØ`O ²p@À&é¿š¸LŞÑÎĞÄeşTX/ı‘,Ïƒi8uY8 ÜÇ*Y8 \ƒ&²p@¸Ldápp×†ó(‘…Â5œVÓÕÚÄ İ	kmâ“ŞÀ”µ®´…	Ø@ÒT8\UÊÂ›œ'WhĞğ¨,°AÃ£²p@ÀFY8ÓdkÇd5 \ƒ¹ÈÂáànZ4‘…Â5È"„k`ÑD×À¡ÓÕÆtaåLX÷¦Ëo`Êº7Ü6PY8 `£ËÀÀ´uoI³‰ëŞdtXM\&‘äM\vIeá€€õg$çÎ`0Fë€p§~"„»¯’…Âıï*Y8 \ƒl ²p@¸]g14]™ìîĞ„eºlğ†¦,ƒ¥ƒÊÂá ?TÛH\}_8 `“¢µ˜¸»‰ÊÂ›œu¼‰+Ò+ZH.»£ŒL:&«álşD× û‰,®aÿY8 \ƒ&²p@¸¶±š®LvwhÂ2ùzSÖoÆD6“ÖoÕ¾36Y…g×oEšÊÂá 4¥÷q&®&ã¬70q}ÔŸ„ÁÔì’ò£éêødØ°•¨'é€€b‰º’Ø°•¨/é€€+‡C6–ƒ—é°äN]¦[4ohòŠMVÚñĞô›Ì´Ó¡	,®¾:²‰o.†¦°ØpgN¥B6¹³zƒS˜^ÅÑèvg›Ô[B[6œÆ©h°Á‹‡ŠÆø5ÃŞ˜&6p®ÅĞÄµ5]m¹CS×Ötjó†&¯­érk<8}™\j§ƒ˜Él;œÂL¾­‹Á)Ìp¦`¢q@È&ïVop
Ó[@4»ÂÜLlT4Ø ‚PÑ8 `ƒ‘ŠÆ´*lĞB¨h°AYN\¦»=wpêú\-„\}­8$äjÓ!ïL'åÙĞ¶3WCSØ®Úœ:$äj{êõgt˜?ØæÎ¤ıĞ ×á ®Xhşp€+}m†lPXpşp€¬‹Eç81(¸‹¡‰kob™îĞÔµ¯95ÙÿYDèÙt^M`{Óyu68…™Î«‹Á)ÌÀ±‹(ıá ×T„l6¨Î»àÜWTlPì©hpexâ€€ÿ»2V@À¦„ôT4¸2*cHÀÕî7CB69ÁzC“Wj:)‡¦¯Ì˜øjhËL·œ³¡),«95ÙpYÀDã€MçUop
Ó+_ ƒ]pfÕn86¨]T4Ø uQÑ8 `ÃŠŠÆî¨hpµhpuÄâ«]S‡„l°^3Ñ8 d“)w:8™L¹³¡),¯B®B®1¨Ùè¡ŠG=PS¢DL[C®Œ×peÀÆ€+#6†lPD4	¸2fcHÀ&åN]&åN^Õ§Æ!!Wß5ùP}×8$äêˆş!!›²:CSØÁ QÑ8$dcp{ •YÀ‡lP»¨h°Á,@Eã€€«
]
¸ªÚÅ €ÊÀbpâªíò'“)×š¼ª#„üÏjƒêMÁ”³¡)ìŸ&Ed1…Ç¡ªJàÏ´EC¨5ø Åù²}°
owÉ'ô¦RšW+†qí³(µ›¾;Íò G:”¶İåı—Ñ
 ¶Qÿ¨ã¥j%ëŠ¸ö
İÑ²hƒ•#`ì$v÷á›²²…eYEËÅ #„µá®òdÌ§şPÀŸ*a{øÒg§«˜eEÖ&Î+]7üX[À4¸á§r7xHøÍ¼¬,êÊl9fJooé<ıv(*+ cjÓÀÆxĞOÈCRÏqË”Xš® €"WwŞGJë£\Ï±g°¤z1«@YîáÊã$XCuØ·Á6{“—•f1í³*°PŒ²¨ˆKÂòu;§R©Ê±C‹Ÿ—S½
öQÄèõJd0"ej©¾±Z­îtTrÈŞ?»ÙŸ£G„ÔOi²MFû4ÜF‡í/Ñ:D;>Øí„¬­·W3¨s>åêcR&àû~QğWHõ~¼PÍ¸ü?SAy¾2êÈÇçô†8}À¿ïq…UáeàÌ\W)Ê04	Rø/ Ê/Ë\[ö”ò=(İ¿Û`“£i]*¼ıİï¤â£¤{‡1U‡UæõËª³b•s®V-•7t­RÚ¬Š£ùî6†?´,©,³Ä7´ÚçIÈ›±5òœV„qvÉ§@Ú@¸»ğh16º>Ş^(Ù]?#1©¼êD_^ub,¯:Ñ–WèÊ«NºO­ß.»‡òñÏÿ¾×Q`%»øÉÊV ş¬`·¶^oƒÏ#Ve|¼Ø¾>6f.”}à3‹°ÿÉŞ£*+«>'é°nga`M˜mI”A·ûä´íÎˆd\¿‰İ‰¼‰OÙ¤åh<ßPx/@(üfÌuW´şö’¼©İK:
c[	ÿ½:¤Ú¬û$BŒ2•÷"	Òùº
á¥gvúê´;w’ZÈäšÚsoÊ]òc´ûZŞ:Ü‡8ÿ·h»OÒ<ØåÏöŸA„c/ä:/Ó0~0‘A:º‡¾ÑĞ_£é¸áÊ`;×–C¾û7ğAß]çyâú7äöÂ¿¶Æ}ì;7äø1"»W×ü¼±á	7°[-€ÿ ûòÏï(C_;"Z×üoJIá×3a©G‡ıµóóOYò0-¨
Bï`ŸmÄŸ6Q¾Cûâ]¸;ˆ 9|îšl–+ßƒÂéÊ]Á§O’§«´Ja5'Ğ&hêÉ¬IK±4hj^rCãg;HÓäS¡öşş÷ª¢[JtrgÉ£Æ”©7]ÊNtNÚ§¨íÏa®ò÷¼çÊí!_Ûö£mpf$ï2üâ(°÷»ûÿµ
VáŸ<Ç]83ÏwÆ×Ö.¥á>rŠ:ˆO†=şÛpu€Ï< Ã€N“¦E—„‘ÎË|—}@Ç#ËŞVp#>!Ù,ãp}C…m6Š¶÷ğÚmŸñ¸øqsJ‡¥$…v‡åh·1LRü’8.çÂr ª²„%Ñ"%œªTK;B„1Â?¡®nÄ¯04ã!ü=ÊAµ‘¿[A³A’V)è¾n¥-¨Õåà#GJèÉ:züÇ*²ìÿü)–[Ä~ıæƒ!0ŒiDg~¤Ï±T2Ì‘×êÈk÷*s¬ûàÙ¿±×ëïˆZˆÈfıtcghë¡‰ù€f.ÃS¦<bßÁÖò†,A†HŒéŠÒ° @™£ì°D8EKºHÒ3x‚w¥î×ÿË'bá)„õE³`Ñ®yÁFó•!–¹z(¼‰vûCÎ´dŞ95êSå<9-T×’	­c¼¥ŠyÙ‘şwl‚iöÛí•?ƒÿ:&½¸Uu{>.´éä†ü?”ëW•q}ôj¼=t}r—Ïê†`Lµn[”ïq›ƒ>”·}¬Ù(Æ_¸íbzg¤ş^µu*ƒ2ºâ¶‰|¾sŒÓÚºÄû‡ ¤¼÷€ÇY$"Eß¤˜dŞ«`0 üŸÃŒ%D›˜èàÒ>ÚoÎ&¼×@ŒÔ€.³æâí‹£U²İvQşT f
²éŞHX$Oáá‡ğ9#mÉ5§‹‚c¸Şä†ı‡ŸŸÄ8»tÜœwvè½—^õ$,Ñ¢ÿ„ôD¦
eémğ	òlM³Â}Dÿ¶ò„
HL¥gÑ%ÈoñßwE3]x©ï"ĞQÍÇœC4B'´¹òB›ãÕªÂ#wL*AMËc\e±û†ëßrXà©Ûvñd]Ç4bÊjtãÜ§0Î³…vF;«ƒ…ì¹°âª1’·#ÃxBè´í’tÄÒÅw9A_‹v!Z&âı‰g½hh²F·„iÓ[•4\£sàÇû7Éç÷Òš¼aş–…é·„cƒÁ†R¾}ó|µ§&ô«®·CĞ),të
k«òÖóÛ©?±]J¾¸‹k–Ñ;˜&’Ã¹Yó<M>i®Bt¢)£½³E§Ôp~A¸,.­½Vnv)Šó¨G¹ä ˆVÓ$]úÂ-øŞÈ#
â3Y¥ÆÍÜ¢ò–`Î§S†íâÜ³)x2ÔbşŠ¶§*AŒGÎÙÍZÕËÂBŒCÕ]Ë—¸ÚÛcÌre—G&´Æœ¤â¨Ó<Ë¬¡']J2ÜªRé6AŞÄ‡4Ë‚5ÂÜ±\LDÑ¹ñİ›‰{cû³kñ<¾Yºh:êßÌDë¸³Åj	&Zú—g/æ¯nh7Öba»‹òk‹s_=Wk•§vßc·ÕËøg¸?
ô–±¼ä»3.±·¸™87š%-—KoÖx‰Ç³ÙÆqaHô/:$ÒM1$úµÍŸÚsÍŸØ}İnİv9O†ÂÔÿŒÅEƒDiÄ„Q3¾¤* ¼ŠãæÅë*ö3ÁyuæpäÌë¶îVäÙƒİı@+l0	7åûaÖîıMÚîı(o÷ş>nù~ËşÓŠÅ(‰„rîEOïeL#\¿ Ü;®¥Î´[H&ìg;Ü¢“¹¤bLfDÔVz¯Ä6åxR½)üÍizËXò„šíEÓûd	ŸZ9p5^ÀçæÊàSÇÜ+_7Üƒ#Åı‡ïµ7i²e}\ßäÉkÖO#Ö¯oå}£u×Qî‚õ‹Eí¢0€ÎLW]ÂÇh3kâŒPİCı]uû~]¸ëÂVø6s3àº¨YgH`qØ.³Ñaûšö0g®~!–aÌÜJ'øÔ¡Pc‘ÒÜ’˜Zc’TÅŸ¼âÖ]PC”…ıÇïñ“Kv(ìc+Ä¥‰à¹ŠÄl•W¸XI‹‰n6		Êv™f'¹%Ï=Ÿ©0uo‚®s… !¶°ÇÖŞÑÃZ×ˆZN¼)"2a(·o:.Ô°¶Ma®æc@^Äç×_õA£9½˜y²
G.«´}Q¢{İ™%á@b!™*¹kâ››©²sĞQp÷7ÂS¤‰h"õLóiš§H¨E{ìÊh“yÑ£>—QGê¼-`˜~XNÖb0— %dŠé”¡:uË=>™J
ƒÃÅ;0ƒs´-C
®|ß/l"ØÿV|`–LŠ˜uzæLëÛ$-¯:à‹e
.æò‹ïW²ÇBû¢g“1~&8Ä*OglF\°Tá¢?„§¬”	ŒÒ¥l=âºi†¾æêq<…O½b7YÃç†ú`Ô*vU¯7UìHD±£ı4Sì$àÏà–à‹'0Ï²ó¥Ë‘/ÖšUÒSÔ9¡#î>ok*ƒÿ‚],,$ÜB²]Iø¹ÔkåZïÖÈ¸ù¤ÖºF7šgÔ1\³øn‚<÷bV2Õ÷[%ºçãRtW‚¯mÍ¶.ë„»3cÖÜ3ÊÔ›.|üâãp¤˜qe ÒÍJú¥¥¨›Ò¸ÌùÌùÆ»µ¹W‹SIÕ8TSc‡[ëÆµ®Ÿª×3GSì>Ï>¤ÁQõc%±f'YƒœÉ4ãÅ$¾‹)Ôu‹A£²Ñ+¨Ü¥E´¡'Y§¼9[ÑúğâôvŸ¦§#6P¼G[Ş”Oª×ÛNÅöÌÿ·)@|ì¹¥ËÆn}‘9
ŒÓUÖn#n‹dÅŒzœÍCZLIÉ)pZ'ŸL¥Óüò˜C;u«'ô'O+<«bˆ’0¹¸ŸÊéy.I·¸05ƒ¤ºVò)TŠjJeuGH–n’:Êrô\„µ®™@c{B€4qGc»×y%Zˆ±•[ûªë2q¿aÕO¹¾ÅxThğz‰Ù™ùDW¸“pº¾™”^².˜gÓÑqê«;—ZO±¥M=—ç±¬ºJ-'™¿ßÀ|lcGŞßœ³4}Î$û¥2s5ZyoŸì‘º(c§z{-ˆeœÒ£U‡÷Hü…&şş™ŒÜ_ÀG¸Ãıw…ôÒm3kk£ªú„# é+[»Í¡Ö\Ú€Iê1‘Ì8öd@Iß€A¶öØ{iÄñ‘·{”R‰¨púã†J(^6ŞU3ÙHö|yÄ²$Ğ­ÜÂ›–à)0tq²Â¾±'R¾¡±)Ú]Pu¹å'{¬´¸|‰¶÷íü©YçòI²Æ²ßÊ©Atş4bïy6¡g XDK…û/j¼11R÷6üß=D]	+ö´ (u†æK}Ÿ*¦:I-ÓõO¼óFŞ|ÆIUİæ†JM@e°9'-+g~'Y(1X}„‚¸£¦ÁC£>@Ïy¯SiÕ$Ö£HÊì¶U«µÀg\r@^F÷°Íìk¢¨ó ë½Mÿ%ZgP@+É“ìÛxyØf?FYİ Ñğ‘,B›=†p¸2[Û+]{ÜZ“¼4¦ùÇöcØ²Re³-pAæù6¶³Ã¾gN£õÁSrÈ½«Ç=M—SÄ¢£EÏ_Û4AÀ»p}]?ïQWw¿Ğ¨7O#ëIe3fHòCh <ö°İîƒ=cÂÏòp‹zDSF~Ìn§ ìX>V™(òŒ.`¡®}TÛb¾(pë{n­«‡5æo“çcø	í(Ùsò
‹®p_ÈŞâ¹µ1uĞËrf8qŠq§É§ì[ õóA¶îkÿÆİ¤×äÔ/0fJòªóÉ=hRç%—¹×9¯œ¢1¢‹&Ò
Gá¿Ş$ŸË8‰/oÇ¡1VÎGm şÇbÙ[…ÉFPyş5vN‹AƒõKÛDl,ø…]Å#Hz'9Ã"üâ«"÷®a+@Ü×Š³ò¿ø}ó?KÎÔ$[Õ—¸şp»ËF«‡(^¿ŞıaVê^X‘ÖÄm4¡U»Œ±F-kcá¦„i–Cƒ$z–äaw,äB³Á ÁéĞ¢»Ô»ñÑ·ëKEr^Ğ/+óuè^ı–°{ÄwAÜ\
V2M¼°‚Q;qúHª²fJdÍ—4:CœØQk,¹•ã®e…ä…Â• >òDëAÌõ!‡¿,›+i@u#)1sŒRÂ}QcÀù¦©jff³³:ô16¤²]è>k3
öî+£á+ªê.¬ Ş¼kµƒåUïØ#!FĞÓ¿_€n8ü&âbŞïÁp×“½Oo´ú"g¥¸/nàoş¸¡ÏQ~éC&ÎÂxÔH	BÿHx/;,É»É*
â&ØÇ•†[rBû×›#ÌğïƒíÜ©34Â^ªd†´³õ¯==°»¾1pÚHo—á&IÃ"¡)¶¾\øˆ™‡4ûö.Ì2¨tô%!/+ÈŒMd™×ÈJ*ªfºƒÙù’&H^]ë"$]××bî€X—ÁˆAZğOŒÔˆuŠ_Aß±&Ü^NûS½z¿jõ.ïÈÜpŠ_ì"»ã@=’†p´÷­«˜ØóÿBÚÂgıÂ4dÓ5@gD/‹ÀÅL½gÉHsúíç {Ê„Iv}IKWëK®k=vú‰<³Ó‰×ÉéŸ]†º!íÎ?4÷/ÎR.§K´ÎŒçKAÿÂ¢+{:»{S¯Ü€¸H\"OxYÌ.ûä<ˆ‹$‘yu)¤§&|KÉ¹~)¤ğ¨»\Nòâ(~ASÚ?o¹ìÑ\6õôÎmÿ×ùPæÉAXşÎ»m^2Ó¹d¿Èi‚}ãúRhk æ¤«…3¡Ï_ÉpÆ.Ù ÎìçaxßüÏ/Šå½º_èTŸ…í]èÈ¾
;ëãÜâ¤êgb_v–§a°EÃÈŞ%ë0ÎŞ–“ŸŞ¡ßğOå«Cßi:g¾ÑônğiºñæåO'”›“B¿¿ÖHÿZ#ıkô¯5Ò¿ÖHÿZ#ıej¤o‚İ*>,?ìéê!ÈÂğë‡m²âìC!5Å ÓúĞÚTº>Ú¸ŸwïË0c~¢€¾GpŞÁÏTÎ'¯ÊJX=`²é¹À©ã´¸0Ä™¤üWÏÕ³£¦Ç¬~ß*~…§aV[6¦é ?ˆ=Ñ„ÄÍº*~Å?ÜÔ´Šƒ,ÿ°î£<kØ Asû!CºÇaOÑ¤	«•Ù=*ºìÑ`E®‰~	£iˆ$şşgTbùUÇR¹^+ÚsY¡¼)Î îã” :±”nÖ
ûá<IÍ(³İZj¦‰OdjJ;‹Æè’1BäkR™l>-æoÈWçDë<>ğv÷u¿lÙˆV¯ÿøayˆã07g6vÌëBODî>òYk‹ÄiÜƒÛ`ƒ¨³õ¿û†4q>C¥Ù/sjr•%ñÓl|pv£‡*ª<àÌ¨>É€_”SL“L_'ëğ¾ÌiüiA†ßN¡ê¹ÁÛ‘~^ŒO™j Ú¯òõÈ©4¦¥íÚ¡ÊGÿW°‹ˆ§{oÚßÂ`ÍÅ;àÈ,üÄï\JNW“*U]XÛékØÇ`übR>YœVš+SÓaNÁšELéŠ¥¯¤™=$Š½ƒ(RsÉH!^P;—…†=z" Î-Jˆùœ¾ÂXwœ ËŞ‡éÇì³È5d®õ]Èòy.…r3ÁÀ(Á¤\ª”–«e°4qáa§\zd9g^ù5ÉÂô^.×­OÜ;nÍ¥ÓÏà³vö!šTÙ“±íecT`eéêNuíóäÅ$ğÈç.–™Ôï4!•ò˜–Pi£JÈÀ?@àqY)4bEÁd`¢í†ØI,Xá˜sR†uC"mŠEej”¡¶´ÙqSñMP/»f¾ ¿ê5Ü”â÷®hÓm'uåU!UÓ:Õ&i.ğ5¹=ÂÅb!oNc^à¢RÙÕ´ªJs,HŠq9_yËöö.ÉßÆÑê#T¿`a‘Óº§0M“”r®4\W\Ñî9ëCiè«Ó8Üªéâ.`•‹r]½Û+oŸ"wüjuÇÏÁ {”¡|dŞ$IÎgk»n;Æ]Õ;÷z­B‰1.€C¾Àùv¤DÏú	htğØì˜- š«Õæ¨r4ÆÕà„­-
âä! ‡ûfQÍ«Ã’,œtøª5X¡ƒ;ê_BĞÌõBeQ=Pá.š–ôs¤b"s©f–±zPû²±R+™kœN:™{kû…¡ÔÊØ˜†¦!¥ß‚.LÉĞ?Ú˜ë«•"âùå^0ú er”Vª^Ä):ÿ?}@ 
°¢ áŠÍvôŠC@F|K´„L+7¬´6m!Œbú@Ù/IàÚœ²†_İm”Ça“á½£ÖUhÉ¶8U;€ggsIa.‰]¨0ß™!è-|š¾øımîNàšCD£ëL›ÚÂÉİ¦p‡9†ê­ûĞ_\ˆêõÉÊ|ª ÷Õ/|U‚¡hÖ´ÁWW­¯®Z_]µ¾ºj}uÕúêªõ2®Zq ÜDé‹ê¹Áß]µ:¯šz´t¿NŠD±çÔ¼ò†l†N3Üù¯¹Vİ§n~8'jÙÎKª†ÇŸsá2»]§´0,û‡ÚŸçˆ‡2é 6ĞLH	Ê‹p°Å	«íì°Z…Y†tâ(>¤…H~üÂ‡¼Xv?Éu˜
b¹s^İ)ç·¦u;¢	F† 4Ç1\iÙĞ\È2¼œ†ù!Eû+ù€›I0xò×)÷âiË b¿9É€¨òÍ'Í0Şn¦AáämÓd -ík'@jcqë!eVÛZJ}	_9[·M¥suÓ«,ãØ(QüàV#©1*wW\Pj°ğ¼¨iÿµ–Å×è_¦º<‰T…­…ÈVĞ‘Ç³ç‹2È7Ã®%·hÖébb˜~®üêzÆ·˜d[s=È×’;á
2ƒ ­iG4.îó´Ù­ìšT¯i‚’ø xò ÄÌ"…”!ª {íâ:‚§.Ã'`sXGvÚj?wàÀqÈÂt
ŸRA>.Hª¡bª× Æ*Ø6ªŞ'}R‘¹có,H—UB9sZâA[ÏïVù*‹¼5Õ\c!YÁ×ÊLñKú.ltâÕb²šô-ºâ<«ñ-dæAB6ìˆÏ°N®j;3x³Ií¢İşëë|vwU›60¼íIn©ÿûYÍãÌüÏ‘Öz	Y–Ò¥ŸMáC0‚¬ç’K«ğ¼‰Âxİ(ÈÔ’W|.z ÿÍ6Ó„E	¨ôÛß–½h·Ãı@æô‚«ºArÈÅGö7NLL_/Y¾^²|½dùzÉòõ’åë%Ë]²¨±fsú²Y›ËìXé‰]C1ê.fštL|ÒãH/,9õ}ÊØ‹ +Õ°Ûùz ²Ua]­zºİDi–“JØLí.£y[vÆ¢éÑ3i½¼r½Èñ¹®[J®HF(İO’/M=Tì–=—.1ğ
ÄzŞtí‚ş}U{5¿–§Ùâ(‡”t5­B¤xÖÓ¼Y…±¹	…‹½ƒ!’«dâOî-ƒÖOÃAº±l=$Äheg2]p¢ÀñYB¿,%fâáQ.ßl°¯Šº¶§=±M&á( ™p§ô:“?ÒÍ¸Ã§Ç
'r+Ç¥¿Ãüš>Ø!Ò	â×±Ë´‰^’†;¾İ$«CÖ¨{òêQZl¯Eb>Œ ø7£İ#£ëFhÕt+¼Jûe‚9ë;îØ[ğÒ”;Ù^3Ä4Ü†Ûe¨Ø2µÄè-àÃh)XÁ‡‘ÚŒ1P ?I««Ü‹•FXQËê*ÈÉè¿&Õr P°/‚-`ÆÁR¼Çõ9““ âP+®cb¬x‰?lI<çğ{Ÿ¼É9ìñìº˜®ñ^Ç$8l?Aó#ø--˜TyÁTM•8œF|XhÒBI(n¿tsÇOOÇÉ˜M—>:ués^œJR2òfJjÔb®$nÅœ.7TÓ¶dÓÙ˜åüÜEšš£#j–®Ë£º†„(DÃĞt9‹
VÒr“D(õ5È2uƒÀe–Á3¥A©™…4\û|õ4S}äVÈ±hmÙ÷£²+QšqòÅÌ=ÖcÀT»]ê†%&R·öTdµÕÂÉÇh­°YdnãÚÚ­’Ú?æpÙj^øÕß:í»NÍÔÃyRİÖt¦­ƒF'h·Mıµü¸3`(Tñ‘¼`në­§gØßcw¤÷ÒÑ®Tt
Í²M»Ó™|êH=eüä#~öQôèµkû·ìÃ~ó~ğ×0wë·É:T‚êûšb ¶Ù½¤·)‚•¿ÛvõlX–Q<İœåçÏÚàs6*4 ‰Ñª‘Ó+m–pdáR/‡k¹NïÚ½†w‰
?§tÜgİt]—âÙH@!Î07áÄ_ºA÷¹i4^tõ1feF—…óŞ8ßÎ¡¸—aÄØz¬t¸è@sN€Û¸Æ[Œ'r)µaóü}­b aù2°¼;OC)ı™ZèŸë#‡DNñÕ3ç«gÎWÏœ¯9_=s¾zæ¼ŒgNí‘‚rO”cm
!ƒm¢V€(eìÌ©TTÅÔwáÓ±†!CŒ®¥/jà¨¥/ŠŠMz±lºğä=İeG³nØ’
btë¤²0FÇ.5^aLïcşxz&‹f˜~©Y¬k·Mr4íı·l„~Ì¾ÃÎà7íáo-ÛüWB@¹-İûJ¼ğ :-/~ûÿæú'OW—:P¯c\:‰ù·ø`§¦¶mÚÇ2ßı”"šLŸ$ÛÏ ÎGwŞ+Óú¸¦òÉ¯üI>ş¿è\Îfå•>b—rÌT Ï„Û¶ëáîN	ùm=…¼ûLSÖôıøîî“C—Ğ×;ºíGá#^ÖJÎ³àì1ª=ÕÎo …	ïi:«Åë!% °ñËXĞ·©,€V?Ï”²ñ‘¿æn}ğ½&-5GÆ°e*Û!”¡FÁQÙÈm¯%O Y×ÇÈ¶èé´¿o4KxBw–ı{ş®ŞI×I„'jî@ğ–ärn<Ogşë©[âgÔfštº7IZ8vxWú22†ã
)ëé-:¡?Üı3»¤!ô&Íy2·"É¢Kø"Ø¸‡Œ¢Ûv;	=ÑIaå²‰¡»ñ¢j‰ö§B+Ñşš~Ğ?‚Ü'-¤^ÍĞ:lŸÊŞzÜ7õXwÚ0îd|‘I¤!,ŒßTàœFçU(ôB÷o’Ï¼ox™£Bö‰gñæ H÷àÍv|w=AÆ—ŞäM^õ0øÛ]ş@âÏ^'ëõµ„vÖE.—‚s’ÅŞwâ|Û}0:„û‹±w‚sY¸®&;]g'3.ÚË`‹ÇRÃ¨:÷‹êùÓ>üœY“_ÿ`ã%íLÙ¾ ‚şƒ^hÒí©'¯‘hzÖÉ§Ûh—…9D‰ =ãZpn¹q,‡\…øp²eÃmúŸTj,éAÛû"Óıo“…aå2É×SØ>œşƒ=öš‚‚ŠÛ”É+ÅW:ª¥WA³}r²9ƒ(àÇĞéx*ÃæÀá.bô)İîYV|f¢y“wóÀoD+œ+N8‘•w°Ó½‚­1F<µk+Ÿ¨N¶äĞÏ®Ø·n.€DÊ–Q³¶Ò
k´Áš,°“R:İÚ‘AR¢%U×,l°ö^Çô,’©ó²Ü²mwâútêG»P·F Šõîƒ¥áÀ§´u'úa
ñÆ2a`ÿş£ä, 'µcOÚñr}™­89}Wá8·¹±
×ïpše>\¢Ykqy_LGw#¤ØO“´q¶;ŸÃlPÃœBNV}¥ƒo¾luÒ¡\éİÛG·}z•Œsø¡ü¹‹œ2f¢©îæı«Ùk¼#ˆ·D×âÓò0Ú ¿¥M1?ƒé¹~Q;X¯ğ:¢n¿İ"öÇR!ºp’8Q >ÿEJÅ8ºFcÎG|QR“_å~&ºOŞ15ƒè'5†Ğ"ä È´t¬^BBSÙu_²aöŠF¥‚éØ-"ÌUøÄëKìÚ3ĞÜñ£~¡™Ó,“Ï¿v¦CŞJÒÕÍ°³?ØìÏqäzÄèö2äŞ°íLŠ0µ£0Yk+®7µÖºÆ¦À&¦?Ãùğ>Ÿô)e8qF˜º{69~x±ˆvQŸzõœâ[I!·ø¯pİ?í•=÷Gƒ½c+wKhRŠj8×ûc+Ç•VJÍŞŞ#vŸû‡ï!gaÃasÍ~ÑN[ëì•İ0Pîlz¶»ÖÀksû¨Î§]5^İ²zşT¦R¾(£éı·Sõ5ÍûÑìMxtÓÄŒË¤Çn&æSµ5[Ü¾ìı¡
b+ï—!ÊÙÈ%äšsØmÁ{œS…±"½‘>®ò{iÄ\Z	ƒ( ‚Ã›‡Ixâ×XÕ¯±ª_cU¿Æª~Uı«ú2±ªÑ:L>°P‚6!«¸¥"÷ÌÅäŒªzÕÖoÅÍnšµ6Å"¢…&À û’Ã<%B·r˜ãsM·ŠÎmÔ‰%¾†_‘½0&¤° -/€s6ƒ$<oÜ®£ÆñÀ¦n‹m[ük4eçÉ:5–²È#)ûÂ¤ıX…çä N~yğš13u£¦e~E{=%ıI§ÍË#mï‰æ¯¡\»ïÇä>¹îŒ¤¾;ÉùÈ×Ÿ¢|ã)Ê×¢|İ)ÊÇ·ÆŒ*|.+Î¹tÚd™Ùkç	«àØÔW“/ÿÖFšv…ÖÖkÿÄ^6ÀluG½)Ò|±{îä(Ğ\¼°Dš¼ä	ÿ§±¶´Ò9v„SŞÃ$ÿ”F«°;ÛmHVåã—ğ/Èæ'™¶èÇ&*á1‡¯Ô€°w&å4GQ[Ä™Îˆ¿Ô©‹ú›MaæÇgŸy{<&Ä¹×@Ì+c˜RgFmìÛÖ2‹{º£ğSvö©â¡2¢KD«¨1Öd¡ˆ¬l/ßFëuÜå wÒô­Ôºî” +´6Ÿ˜¦p2ÇSÈóÅ?mø‰ûïZ¶üéìó&ÂÕÑiâ0åauü"(oÿœ}úx¨ÌˆÄÍİÌDts)³¢JurŞ<f¡ğ9ü½Q
_IÑ+^\“›¿NÕÖx/kõUJË˜eõõÜZ2Åôì›W ËXWá³A½~2ÆÛWÚŞXĞL¡C­»"¤'pXéFå}Ø˜-,ë× v{êjõ‡hŠƒóÏ0‚F^C§cşÍ:zìmè%|XØš\òs9E…¾HÑ Ãh®ûŞ§!Ü Õ³í‰GrÆ½Ak^'›8Àö<¡oÙCôõ›‹š\LìE‡Ş3Üh³ÆÔÁõ3¸Ôéw4Ğ£õ'Ş£s78ÓÙé·nûÎèï¿E) T®gËív¡.¬Â:ù´Cë½Æ“İ§^èFv]û(G¤óÏğËš)$;-İ­ñÈ|©^’»8}¤èIzßeˆB/[¸lüù°ÉŞ$6$í>÷ÖÏtƒQ’±²µÕÔ7ï”â¶fú•$^„$°óÅ¶³ĞŞ˜oåìd é¥pÂÀEŠ-ŞµŞ_Á§/ü¼ñFÖÛeµ¹Î45ìti×ßr	bfÁ'|ÁµCIu0mHŸÅÀ…sGfCé¹UÂÁVQ­&|òîı-i ÙúÔ6mğ¯é¸Ïx¤ÓGMã¥†»¶û—ä›(Y8ÅÑO>§W,û6Ï>{³Ëœ†brÃ[b†Àı¥½†¦pì€p:ú·‚µEó/«"iÅ3ç/e®¨ğ"k7Ì¤ô¢û–WEİQ,û`:ğÔæ¯Wfªs# ªœ‹9lŠ,zeÚÏ³­bÏ³#UA=uñ°E¤‘Bz:Qœ†ŠÈPB£?<‡1ÎCß$uóIèÑ„×¼Ï¤lt¥Zã®Zš·‡¡hR¶Š.¨£; :†ãŸ£9ü9êÑÕÛs}É¿…»Ü+Ãê|oEW«BL¯2·Uëd—L}üv¾”Û{ğ"6çûR¬©½Ñ@ã`½ÀP’€õ@‚ı îCäW<a–íÍ!îÂk¦IÎÚi»¢¥Gè¨îÕs)ıòMO«ÔÃÀN8©àpåİm¥—•7ù˜7-t¼i,Ñ¨ÔÇ'_Pá/šÌ¸mÔ9®¦íÒ¸ÅÔÙçÿrÜÔ^4f¦ÚT.†-‰ xÎğ+-Ì‰“ñ˜½ÀÂ£q´Iyãw‡ƒtò:{6«3‚+Ş8ªe,#mZDÄ¶ØıãVTï€Åûáb“P¸ççuôXépÿ,)OWl’gø|nl²ÕøÁMöÏv§a°~z“tvÜ0òIö ğp¶¦•Åª¥ UƒOSó©Å­l‚­…-¢i[w]ÓBÊäqZ3sÜ(v:wH­î•26ş<Ï]Ê¬V+}{¼æÿlC!Kv%¤M’À1…®ÖóË„^Ÿb[cÃ¼_Iïà-1‰uœÃ”ÄÍÀœã0øp"ß–T¸_àh¸¾°ééäUoÄX²Q°©€àwlÜt=cn.R¡øÿ¾$¸4şÖ                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               75083402ef6053b0f2f20\8d5f293465684920da2a9f9896507528031be0ed="ZDU5OOW/pO8yJeZwuh4w+H378tszNTg4XcC72dA4jSyILaFFAply9mU1OTg3MzkwMzAnBzZkMDkyOUUwMjWZ6LN4O7cr4OGUBqwOEJzk4BiaEW3JbtRpt9Zw1bQiTjA4MjlrsDI1ODo5ORJiNWHQA1gezbHegVEbkxflFIs+J5CPA93VDVsebTYl5uTjcig4OTlPiXE+CMeokGpvGwsaJkTReTg1M7Oja2IjfWDd5ZokuReVlo0W3oKZVFRlIYAtg2oSwQhdGifSgS3CM1cpNs6+fg+iCi8ba4I+WaXRpwDsH2DSQ+I="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\30683721429016e04f928979196e7ed68ebec927="NTQ0NOS4qes1IeUluE409Hv2o981NDQ0UM29j9Ftj3yOf6EXB5si9jQ0NDQ2NDQ0NDQkUjQ0NDU0NBQ0NDTycGoUxI6LM0UgNonjtwIh/4IHKolo1jKt9vScJwRo8DQ0NDQ6tDQ0NDY0NBQ0NDST02rU5nvWRjx2RH4flf9xzpu9OhjEtR32F7ZUgYG+PiQ0NDSkz4rjHgT/dsWKmyF2OtIFdDQ0NP6FuFeDZQQow975N9tenfIG+Uk+v6xh6Ib7n+X7ik5R8P4I4KMTAHTtqeYZevPVAfaAiMwmjxqdGMGEZNmC9SA="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\5a9a553ff777b5a94cb6ab4a1240ab9f72e2e1af="MjI1YenuqOlgdrQo6U8393b19o01NTFhUJjo3Ydoj3rbefdFBM1w8jMyNWE7YjU2YWN1X2U1NzY5N0FmNDVTknqupSx01xILjdLxLhQEBad1Z1eUKZfdyJp6XfPeUzc3OTdv5jQ1MWM0YUFmYjHWvOV4JkeCN87lFbKqvK01XHIqI2CpKhSXQT3cjWMZRCFhNGH1Mi9Yhm6C9eZE+pQRJ5ehdWE5Ym3Bg/pJMB7Hd/jO17gaqF5GYXrnxG1dyNAJnDIsI+s2QqHLxNjqNo8yoaEyx7+efvgJIWrNtCNQsgaOmv18owc="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\5819e0ff462f24a09e2de1c68b110d6ca3a30e35="OTk0NbG0pOs0dOkouE9h+Hb2ooo5OTQ1BcGwj9A4g3GOfvQbCpsjozg5NDVjODk0NWEoXzQ1YTk5NBVhODlCMIOx39/twHMRPFJSJXdCEs1vshUGUWRSFZDCSL5Nk2E4OTQ74Tg5NDdhOBk0NWE7bEtSxqbFyC4bIKguQDjgKHjGv1oJVKNtGHCTqFkYByQ1YTh7X5UMoL5j5mEQ/UoIKmnbdDVhODgSSCEUWHlJ5hx+PsWOToZC99v1HVTdLzQ7RrZ7WcYwotMMdprb5dgstT01PFSmZbhl+St79lSYqOO5HQ4HFSw="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\01f950429ce03eb4e4ddaba89be0dd623a0e63d9="MmIwZLW//+9lcOJzvB5l8y3y844yYjBkAcrri4E8iCqKL/AQUZ9ypzNiMGRnM2IwZGUjBDBkZTJiMERlM2JYtzAAuQR4lt/PwzxUxqCE43R+2Dwr/HmM8QyWGiBIAWUzYjBq5TNiMGZlM0IwZGWH5tSIspapDjX8LDXT7tSKcdzlZnVMBOhCD7O+cK/ZEiBkZTM41ijexFIuZiC7QPkkvzAxcGRlM9PBZCIh9lTdzFYICG3TEVQ6k9VlzoYxWv9CZ+Rhv2fXgBGo/zLRuk9f2uVDkahqdPgEVHKpI/tZk8zHwEspztE="
05ebb2318ac97cba3ac943542c9763a1e48659d2="MzIyMuK+r+0zJ+Mjvkgy8n3wpdkzMjIyVsu7iddriXqIeacRAZ0k8DIyMjIwMjIyMjIiVDIyMjMyMhIyMjLi/LS8DYTCAWxaKOT0gjfP4NT9oALbB+ZsdDDWk00OOTIyMjI8sjIyMjAyMhIyMjJ+M6jhP3dGG/M2H2u2Iv3Fu58sggo+/dgqN3h+x4ZxD3IyMjIeH3A7VO8IWD6g5aC6yOBPITwRUF8lvBWshC8ZHnA6Noat1unNn/z9Xzqz13cFDo75RTzWp12vXhHEsb4Sh9HXcjIyMouAafaIn7VNpN3b+BMhpfHAmm8cjcsbpDnJMdoW4mVV+xdtu5xWK0j8C9RH/DVBf0i9qVF6F1zcG1qhgUrTKfM="
ec13c5a13d1b3a554e820bcf33cd64d112575b65="ZTMyN7Loru02d7Uivk1ipHzwoIllMzI3Bp26idI733uIfPdHAJ0hoGQzMjdgZDMyN2J0VTI3YmUzMhdiZDOvHPVyb+6pZQ+1mC1x1ZeygjWIg6Das0B0xTd7hr5+r2JkMzI54mQzMjViZBMyN2IrtS0oPnh1zLA0csgY5gFgDPyblzai7ytfuqLtDehhbyI3YmQ/n5QPBqEPbSTILVPvTupWcjdiZM5VYPPPzhhSioVge6w8CMXU5tNMMVptCq5/l5ZgMwhLlxfB1l/qpgXU775adMXv/uG2TxY5tBUu5y7o8jQSfaY="
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           ˜  øïß                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        ó‚#ÙäIŸNo•É‚Àï=p:qœé¹ı5e•BÍÖ°×W¨OJ6® CÛNq2#£¼;pi“Ÿ×qsŠûÍşë£íx³m•ä4‡v/ôÄ4ºĞ6…êÃBQ`öNÈÅ$Vş®_,á•L5,Hˆ	¹¸°N…›l2g”;c3='¹2«@$Ãîš`g°cTÃ\ÄÁ¹'ß›@^ïâ¾pBØˆ°¶¡/…î+¾ed‚‹ïÂÎX«‹‡Í…)gÛñGÕ®ÿ jnğ
ey'}¸R:Ÿ´ñ»,YÊç‚Èp¸txJ‚„‚…¿«BQ-?7ˆìJÙŸÕ|7{·Š±©ÿf¨¢Ùù-f¯¡‰-’ƒrÀU^Ê7pˆñ{z»¯ ˆÂ¨ùğl“"=Ç.èqÊÓlB…¶VîH( àöµ	™pka±­[û|¥Êw—Éñ`}c1‘Õ›ªÂua”ß&–ON2+¼w^Å”©l¸ÈÔ! g°¼³«oıMGc:;Öià±½“°²–Ğ~PÅŸHí^œ‘ª²}¹¥]H…[şj?	ÒB¥Ì`ıLIkE±ˆ®ËkÊÿH!­·Ï­Â ˆ5+Ä~ËÜ¢3­0ò8>Q[û0tÔ76°«&Œ%cã«ûfè¹*ÆŸë>Â0Šìª¬=Ö=şÅ]	“©q   BTComm format 0  (@  Uû¸ÈgÍŸ¹Ñ¹ËŠ+÷Cß@‡‚ÉºPÃé…Ù³ÔÇÕ2V¤È©¡LŸ|¢PYã©8_İ¼2ûEdA¸fPŠà‚ò’9…É¢ âX™ñ	/èo	ñCã‘‘|*ğdª9OAÍØ×Ş£ı<–¥ü°¯ºÓÚË†ç&w—äİPQìUØ|pşÎ×„Ş,?E€Nµ,z ĞŒ^,fø…´ØšÎùßq}ö­Ş0‚³|<+ÄWN0Gl{kvúYÚ•jÄHC ştÔ$6Z.[’0EoÙ€­æ6'aÒÓhd3îH’avà¾òt†Ÿ:æUÛÍ4³@îcş~YT×ŠbQy
¹÷Ë3†`õgÍE@§Š¾[Zt*tÌ„TÌ>ŒÉY4ÈéLb	nã4
Â=x5~"F	ååâå…G¯/l3Œ¶_~A§‹Œø®ªmED,	¢û¨Rlš¿ƒA3qW§‰cvÈÇ*ö~™‹&u,`µ „nğ¬&h Ëãíyğ‘>wôÚCZ{>)wq|øµí.Ú»ÆyÓR/=Ñ˜\S¼ûâOjIcxèHØ(¤^óµ1ÀÅMì”\*·ÕoY71"½°>£³µ™ ıOÉà´<ÎAî´08ÈÆLĞáu_>,dâJz¶ÀEœf	å	ÒÃÜ-LÖëèOÀÌVÒñÀ>M0†Ì<üBabuªV ì›½u³è}&u…	]î•h"ºa²ùTŸb&éÒÖ&í¼êÜ*Ë{Û#,g~ìÜ>E[—ğqb
ä9°»‘Uë +¤)’¯SpãıÁZ¾ävvŠzæÿ÷lş3]Ëäèç™§´«ôf‰Y€é±q£}Æ’KŠÙ÷­èº¼ªã€pW:%®_†ìà­¶>TŸèL÷õw†ç~(âõ[İ`|Ö@F1>u÷`ó%©gâõ«füål<WëpÊ/fH™()wñIêA™³àC^Ês‡Å–O`¹k9Ò-!ŠôU…¿:¨WWÚ%6vÃİ#êôóCXX´»ú2¾^1fÄ¿[‹íÙE††6-’àìr %¯sŞıÑ)%Tl»Ü1X–¾ŞGREÆòüÓ¹•ƒGÅ®JÅõ~-åÏd{˜õLü7Ìıp¯Å0*òj9ş"sàóà5ŒD€ÌºB³æZtÿÇò9'JE8…X5.gï…ívÖŸ«†Ìöd ø>Ã4G/£1Ş‰îÛgg¥’«sRUÛ‘mpî2ìæ¾çs›‘iöº«öùb‘­{İªˆ~z‚ãƒi5é’ov4nh½ûY'T8™*O)Å°µ¨t‘¶ngö¦RL®ÇæUvL_ä°mÁj)cé¬A  Ø~›3Šxª„”Gá*Kv¤4t¦Ë9Ïn¸ƒ¹J$¨a¶¤.iB²;óÇ·C ‰PÛ¢‡6ÕHùaF¼ÎŸDÇ°#ê>D’
UøŞ/×]ë¹Q-ê)vIö¦iÉY†'ØßwÁwŞtp€]"Y¿y½Ê	?,co˜xÚ³@†•°ÕZƒ‡vïIZÛªqNbÇÕTKå¸Æü¥ÚLæĞømŠı]€>›•/KÃÉÆ–˜~ä¸†Rú`d92aX©x¿ßœğGH=W½ä*è¦¶ÓÈQJÚƒ,ÌŞFİ¬Ö“V8®œ0y“·¶Ï~¤@–(òÉdÎ¤~Â¦)Ó›‹¶8ã€¬t·Ê	ôKG*¯/zAûOJFî½Ÿ°3™„“ş€W5Ëj=V‘µî¦-¸Ãø;œ—ZôÇ„ŒLß@Ÿ¦Ë—ÿÏBËå´YƒV¬ƒyhnÄÇ²W/”áæß;zÂÛ–Ò©Oœ`à¤	ô‹‰oFÌ±a4ĞK)öYÇñœa5|FõİÙWg„$ˆS`Ş¶«¶¢sÊÍ_ÌœÃÌ¸4Ådİ£dwıt:{W Y°×£2[V%	ùpÙLõàPÛbĞàä£&yğ\S>Î»¤˜ÜŞÖ|m*?«çMt@8WÛš6¥)@–”SÓvSíöÀN`˜¯ÇŸÕ¸èøO±£ó_Cd¯
M	m7òÂ0Ôğˆ»ùGñ	 "¼Ì5­3º-8Ò«	1]/1¹Ïc%ÑÿÉÅÉnÌøœÕ¤¼š–¸eD'&¤ôzâe½êiìç¸BWpğzJ ®ô7ÙQöº74¢.ƒe‘©œÏ¼oK‘ÖĞô2ôÚ'OÕpË}½sVª—_EP’ıƒÌ¨¤ÁkšåK@õÙfp¥,Ù—3áÍCÿ -<¬ÏëéÃX³Oï¹_õS>¶´ÒîšzòNüRnhD.Lç®~=î½ÛEÕr÷øÊ	àc}Ò’…ÛS<ÖV”Õ3şí2ú´z
±èÒf™WöPõ1»Ÿv”RE·ÜkKVJ^;ª_]¶>§u•ãƒ B³`ˆÒ|ôŞ§©ÄNøĞ`"FHÌËNÓãR	¹ë¢Í<¡ S§àj9 †Õæ÷Ò–iH8?\ÖÄÒJ?Îfä4”¡"’¶Ğ“ÈÿFlæ×çê]÷uUáûQï÷ç=e!í ¿CVô÷ÕÙl*…Ğö&âÙ@I”=¸ºµôì;Öˆù¿jõ‘´p„æwâu/´Él£¨	:ú…mXaiî4w™ş¦*ı-hV0™+Xõ¿É]|®(Èş“—œJx<n:‹Èá÷Fåp¢6Ï¿Z}…5'û+çŞƒ¥€«lmÄN¥¼!ê >»Ó"ŠÔÈoÊ§/©xYî•YÔ2ö½#L×~¢d¹N»/}ğ48Y0tøp‰¢÷…pÈ±Xñ 2Ä	Û†¬’˜‹†Z*ª ÕªœLT«MÉ®Â”¨wÿLïÆ–ENö~ÛT&i>÷Zå@¢³;Ÿz­„$×Ë7/ò¡µnq/ñé–¡êå©Un´;®^´È9OUÍYôU·ô¬À¾Cµ³BPMVÖ¸âŞĞ‚âL$ñIƒêË—šS Ó%ºc9%Û:%ù¢Ğ¼NçÅô
!q·é¬æ¡>"öOg“zëF¨­¶ÀŞÛ8íU­©W.Ÿ¦YÏ•yÔPw¿ö»ƒ.$’/"áÍ4·\³Œ,=«n¥6rYÄ‰M›§a>˜TrwŠíÉ»¬ ª–6ÔIpê¸º\ğrx\iÔ†ZÄó0£ô»æ%Joıˆ‡úQBÎÜëI·ÁÑîJÎENÆ¢Í›Ö½TšEı$Ôtwtš¥Ï\…t±U£¸ÄL»Ü¤é^ß®Tğlï²aÄ¥N1‚}§£#¡Ì·ÑÒcë'š0ü ’+*
iDp'Z»ºnÂ—š©qƒó¶baM:ó!xù‹-©?ú5š²>ç—è ïÈe&ÜPûºJ‚É,¡ªZîì¦·Æ ó4"
W¾ÃLXÕ]æ`ÈOäéH	ğIÂÁááƒÇ+€p1…	‹¼§IÎ5½Áe7åW¾6:ªúTLQœuCÊNM]Ö¬9üSş[?§Úêæ´¶·` (w#oDÑŞ<$÷cÿãZcE§Šïz\ÇP>—_q5'ú*ÕÃL7€0xò¶ÔÔÅÍ×šõu•ÇZ<d1@úK×ÓÊ@ï+yMôî(ë£^Ú„‘X¼z|¾fNîFg%ëÎ=×ç?âsî+RGH±³}¡ÿæêsÂ(D¦X¨ÕĞÛ(Í{7¾ëf]û§Û´ñO!ıÒj0•çcß¶høeª¾*VÁÔ-Ì­XaïÍh’{oT.^»_…¾`BÛwºÿ¡/[Ø™ôbQ–'‡šÊ{+Ú	ĞAÅ‘ìÉTÜü¯Q‡„‘ôa2J^!°‘½¬9»Xe…Ü‹[’ò3Ğ.Ë€T !)xzÓ$¬¶²$…2™Bô~ËcQúÎ m/yÇÔv7ÓWœ]J|‰€²#äcä‹íÁ©Î@ËKÙšÈ4şÅMƒ’)&¤wË°úˆ‰b7^}s‰Ïø¢‡G<­_X6Ù‰Û×Ä“Ò‰ùQw¬¾ÒÇ*ËvÄì+;S&S'—€Ü˜ˆHíWãœ™3”‹«£êzNâq
x@(Ç¦HK¡ÿ¯O-Š´Ö
vÆt·à›.4ànqšáÂ¬È'6qOH:öİG‡ìÄv"(QS&:­ÊÛŠb°e=>wê‹åjµïTĞô¿ûr¦£òA¿HfÉ‡`*†kÒ¨	§ ÈRSßLÌiÀŞ­}:p`®WÓãÛj=-ÖwVéÅ1ë²ï]º%M¯K§ö£!¢ìf2Õÿc˜b3uŞ„†[:SÀÍuŠ+I_2LÕúõá‚; ¤8sûT÷¶C y¨¥şb®ëÿqFßyvÔl+çÇf;rå°gS
„ò±¡h4a'DÌÒå§3®ºO,d[Èj+ÂÕê"üÉYï5–†é]®Pz¸76º¿†YĞ"DüEŒÃĞ¦!ı#l2Í€Of£ÌBñÃeÕ
©&”à>«)´ººİUãsJ«yµÓç¸‡Lÿø„í—ı[tx	ÛáW}Şr2bÍEŒà)Ë8‘ÇBP|Oé©ÌÉZÜÎ8ÚÑ…ifçs[·ßşDİß!¯)~˜Äà»X 4‰TÜ†lÂaA˜'1<~èy&¼"v]¡rÁÒ€£œ3\äw™Bİu£l^G†¼oÖ›ËÇÿ¹~aÒbmü>‡Y:ÈM7ÿÁÎAq