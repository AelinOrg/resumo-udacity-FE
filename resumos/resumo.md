# Visão geral do service worker
O Service Worker é um simples arquivo JS que fica entre nós e o pedido de rede. É um tipo de web Worker, ou seja, funciona separado da pagina. Não é visivel ao usuario, não acessa o DOM, mas controla a pagina. Mais especificamente, intercepta os pedidos feitos pelo navegador. A partir daí, fazemos o que quizermos. Enviar um pedido para a rede, como sempre, ou pular a rede, recorrer a algum cache, criar uma resposta personalizada, ou qualuer combinação disso tudo.

Nós o registramos assim:

```
navigator.serviceWoker.register('/sw.js')

```

Fornecemos a localização do nosso script Service Worker. Ele retorna uma promise para que possamos receber retornos de sucesso ou falha.

```
navigator.serviceWoker.register('/sw.js').then(function(reg) {
	console.log('Oi!');
}).catch(function(erro) {
	console.log('Boo!');
});

```

Se chamarmos "register" com o "serviceWorker" já registrado, não há problema. O navegador não registra de novo. Apenas retorna uma promessa do atual registro.

Tambem podemos fornecer um escopo.

```
navigator.serviceWoker.register('/sw.js', {
	scope: '/my-app/'
})

```

O Service Worker controla as paginas cujos URLs comecem com o escopo (no caso `/my-app/`) e ignora as que não. Por exemplo:

1. `/my-app/`
2. `/my-app/hello/world/`
3. `/`
4. `/another-app/`
5. `/my-app`

Neste exemplo, o SW controlará apenas o 1º e o 2º. Nenhum outro com URL mais raso. Note que o ultimo caso carece da barra final, então conta como URL mais raso. Pode ser ter divresos SWs com escopos distintos, o que vem a calhar, por exemplo, em paginas no GitHub, em ue diversos projetos vêm de uma mesmas origem.

https://marco.github.io/svgong
https://marco.github.io/trained
https://marco.github.io/serviceready

Escopos permitem ter um SW por projeto. O escopo-padrão é determinado pela localização do script do SW. É basicamente, o caminho para o script. Normalmente não precisa definir o escopo. Basta por o script no lugar certo.

SW URL		   | Default scope
--------------------------
/foo/sw.js 	   | /foo/
/foo/bar/sw.js | /foo/bar/
/sw.js 		   | /

Dentro do SW nós monitoramos eventos determinados. Como em outros eventos JS, podemos reagir a eles ou até barrar o padrão.

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

Atualmente, a maioria dos navegadores (Chrome, Firefox, Samsung Internet, Safari, Edge)oferecem suporte ao SW. Como o SW é um pro-aprimoramento progressivo, podemos usa-lo em navegadores que o suportam sem prejuizo aos usuarios de navegadores antigos, pois eles apenas não terão os beneficios. Ademais, emeprega-lo hoje para melhorar a UX (User experience). Para usar o SW de modo seguro e não intrusivo, basta condicionar sua execução à existencia de suporte, isto é, verificar se o navegador suporta, se sim o SW é executado:

```
if(navigator.serviceWorker) {
	navigator.serviceWorker.register('/sw.js');
}

```

Se o navegador não suporta-lo, `navigator.serviceWorker` será `undefined`, ou valor falho, e navegadores pularão tudo dentro da instrução `if` evitando chamar funções indefinidas.

# Escopo

1. `/`
2. `/sw.js`
3. `/foo`
4. `/foo/`
5. `/foo/bar/index.html`
6. `/foo/bar`
7. `/foo.htm`

Dado o seguinte codigo `navigator.serviceWorker.register('/sw.js', {scope: '/foo/'});` para os caminhos acima, podemos confimar que apenas o 4º, 5º e 6º serão controlados.

# Adicionando um service worker ao projeto
Vamos adicionar um SW ao Wittr e mexer com os pedidos. Para facilitar, o projeto já contem um script em `/wittr/public/js/sw/index.js`, o qual está vazio. Se adicionarmos um `console.log('OI!')`, o resultado será servido em `sw.js`, no diretorio raiz do servidor, `https://localhost:8888/sw.js`. Lá veremos, além do nosso log, codigo extra da saida do plugin Babel.

Conforma falamos antes, o SW recebe eventos. Vamos adicionar um listener para um deles: "fetch".

```
self.addEventListener('fetch', function(event) {
	//...
})

```

Quando alguem navega dentro do escopo do SW, ele o controla. Os pedidos de HTML pela rede vão ao SW e disparam um evento fetch. Mas não é só isso. Acontece um fetch por pedido disparado na pagina. CSS, JS, imagens... Um evento fetch para cada, mesmo que lancem o pedido de outra origem. E podemos inspeciona-los com JS.

No nosso evento fetch, vamos registrar `event.request`.

```
self.addEventListener('fetch', function(event) {
	console.log(event.request)
})

```

# Registrando um service worker
O codigo no nosso SW ainda não faz nada, porque ele não foi registrado. Para começar vamos rodar alguns comandos:

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

Ele contem o `eventListener` de fetch visto anteriormente. E se, no navegador, acessarmos `https://localhost:8888/sw.js`, veremos o valor de saida do codigo, alem de um pouco mais que o Babel adicionou. Queremos registrar este SW assim que o nosso app iniciar. Então vamos acessar `/wittr/public/js/main/IndexController.js`. A construtora do `IndexController` cuida da configuração do nosso app, configurando o `._openSocket()` para as atualizações em tempo real. Este será o codigo personalizado do nosso app. Neste caso, está configurando nossas visualizações e se preparando para receber o que importa.

Além disso, o JS não possui metodos privados, mas é comum iniciar métodos com underline se eles forem chamados apenas por outros metodos deste objeto. Note que no final ele chama `._registerServiceWorker`. Esta é implementação:

```
IndexController.prototype._registerServiceWorker = function() {
  // TODO: register service worker
};

```

Esta atualmente vazio, mas iremos preencher. O codigo será:

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

Chamamos `navigator.serviceWorker.register` passando a URL do script (`/sw.js/`). Não precisamos passar o escopo, porque o padrão esta correto. Mas somente isso causaria erro em navegadores que não suportam service worker. Poderiamos juntar tudo isso em declaração `if`, mas em vez disso, vamos somente retornar se o SW não for suportado. E como `.register` retorna uma promise, registramos uma mensagem de sucesso, e, em caso de erro, vamos registrar uma mensagem de falha.

Depois de tudo pronto, voltando ao navegador, se atualizarmos a pagina, veremos no console a mensagem de sucesso, ou erro. Porem ainda não recebemos a mensagem do log `console.log(event.request);`. Mas atualizando uma segunda vez, veremos o resultados finalmente aparecendo. Temos uma mensagem para cada requisição feita pela pagina, todo JS, CSS, imagens, incluindo requisições que vão até as origens.

O escopo do Service Worker restringe as paginas que ele controla, mas ele intercepta quase todas as requisições feitas pelas paginas controladas, independente da URL. E não é só isso. Como veremos em breve, podemos mudar os cabeçalhos ou responder com algo diferente. Isso é muito poderoso. Por causa disso, SW são limitados a HTTPS, a forma segura do HTTP.

Voltando novamente ao caminho que as requisições percorrem pela rede, como ja vimos, vamos destrinchar essa relação com HTTPS. Quando navegamos por HTTP não criptografado, qualquer um dos processos no meio do caminho, podem remover, modificar, ou ate mesmo adicionar conteudo. Isso é ruim. Podemos requisitar uma noticia de uma fonte confiavel, mas, sem criptografia, o que podemos acabar recebendo pode ser muito diferente daquilo que o jornalista escreveu. Scripts maliciosos podem apreender dados inseridos por nós, alterar bancos de dados, ler cookies, tudo isso sem o nosso consentimento. Mas SWs vivem mais do que paginas, então eles podem ser usados para prolongar um ataque. É inaceitavel permitir que um intermediario malicioso controle o nosso SW. E é por isso que ele só roda em HTTPS. Felizmente, esta regra não inclui "localhosts", e é por isso que as coisas funcionam bem no nosso servidor local.

Precisamos de duas atualizações para ver as mensagens no console, e isso é, na verdade, um padrão. O SW tem um ciclo de vida diferente para paginas. Por exemplo, se, em `/wittr/public/js/sw/index.js`, se alterarmos o log para outra coisa, como `console.log('Oi')`, voltar na pagina e atualizar, não veremos nenhua mudança. Isso ocorre devido ao ciclo de vida do SW.

# O ciclo de vida dos service workers
Anteriormente, vimos duas singularidades. Quando criamos o nosso SW, precisamos de duas atualizações para vermos o resultado. Depois, quando alteramos o SW, não obtivemos a alteração. O ciclo de vida do SW é uma das partes mais complexas.

Com a nossa pagina já aberta, adicionamos um codigo para registrar o nosso SW. Deposi atualizamos. Atualizar gera uma nova janela do cliente, então vai para a rede, obtemos a nossa resposta, e a antiga janela do cliente vai embora. Pode parecer que não há sobreposição entre a pagina antiga e a nova quando nós atualizamos, mas há. Por exemplo, se a respota voltasse indicando que o navegador deveria salvar o recurso no disco via dialogo de download, a janela antiga teria permanecido. Mas, neste caso, a resposta foi uma pagina, então nos livramos da antiga. A partir da nossa paginam sairam requisições por CSS, imagens, mas tambem pelo nosso novo JS, que registrou o SW. Não vimos mensagens de requisições quando atualizamos a pagina pela primeira vez, porque o SW só controla paginas quando são carregadas, e esta pagina carregou antes do SW existir. Isso significa que requisições adicionais que essa pagina fizer vão ignorar o SW. Mas, então, atualizando a pagina, criamos uma nova janela do cliente. Como o nosso SW ja estava funcionando, mas não estava "conectado" com a pagina, ele passou a controla-la. Por isso, a requisição foi para o SW, assim como todos os subrecursos. Isso explica porque precisamos de duas atualizações para ver as mensagens de requisições.

Quanto ao fato de alterarmos o que estamos registrando, mas ainda continuarmos vedo requisições sendo registradas, temos outra explicação. Se uma pagina carregar via SW, ele vai procurar por uma atualização do SW no segundo plano. Se ele encontrar alguma mudança, como recursos e bytes identicos, ele se torna uma nova versão. Mas ele não assume o controle. Ele espera. Ele não assume até que todas as paginas que usem a versão atual desapareçam. Isso assegura que só haja uma versão do nosso site sendo executado por vez, como em apps nativos. Infelizmente, uma atualização não deixa a nova versão assumir. Isso se deve a sopreposição de janelas do cliente. Não há um momento em que o SW ativo não esteja em uso. Para isso acontecer, esta pagina precisa ser fechada (ou navegar para qualquer outra pagina e retornar), ou navegar para uma copia que não esteja controlada pelo SW. Quando isso acontece, o SW novo assume e os futuros carregamentos da pagina aconteçam por ele.

Esse processo de atualização pode parecer complicado a principio, mas, na verdade, é o mesmo processo que navegadores como Chrome usam. O chrome baixa a atualização no segundo plano, mas ela não assume até o navegador ser fechado e aberto de novo. Ele avisa, atualmente, que a atuazição pronta mostrando o icone de download. Mais a frente, nós tambem vamos notificar os usuarios sobre atualizações do Wittr.

Quando o navegador volta a recorrer	ao SW por atualizações, ele passa pelo cache do navegador, como todas as requisoções. Por isso, é recomendado fortemente manter o tempo de cache do nosso SW curto, como zero. Assim como o Wittr esta configurado. Como precuação de segurança, se configurarmos o script do nosso SW para um cache de mais de um dia, o navegador vai ignorar isso e configurar o cache para 24 horas. Isso não significa que o nosso SW vai parar depois de 24 horas, significa que as buscas por atualizações vão ignorar o cache do navegador se o SW dele tiver mais de um dia. Mais a frente, veremos algumas ferramentas que mostram em qual estado o SW está.

# Usando as ferramentas do desenvolvedor para service workers
Para usar as ferramentas e recursos mais recentes, vamos usar o [Chrome Canary](https://www.google.com/chrome/browser/canary.html).

# Ferramentas do desenvolvedor para service workers
Com o Canary aberto, vamos ver as ferramentas mais detalhadamente. O console já nos é familiar. Todo codigo rodado nele irá rodar de encontro aos documentos(`documents`). Mas o SW fica forá dos documentos, então se digitarmos `self.registration` no console, que só existe no SW, obteremos `undefined` como resposta. Contudo, podemos alterar o contexto do console usando o menu drop-down, o qual está por padrão em `top`. Podemos selecionar diferentes frames, workers, e até o SW. Selecionando este ultimo, podemos digitar o comando novamente, o qual fucionará desta vez.

Tambem podemos usar o `debugger` com o SW. Indo na aba `Sources`, abrindo o arquivo de extensão `.sw` e o depurando como qualquer JS. Mas diferente das ferramentas padrões do JS, o SW tem seu proprio painel na aba `Resources`. O item de lixeira nos permite desregistrar o SW, o que é util se nos refizermos o `fetch` do SW do zero. As abas dentro deste painel, nos dão uma ideia sobre o ciclo de vida do SW. Podemos ver que há um SW ativo, mas não há uma versão nova em espera (ou talvez haja). Sabemos disso se aba `waiting` estiver acizentada.

## Quiz - Colocando o SW em espera
Ja vimos o ciclo de vida do SW e as ferramentas para observa-lo. Faremos uma agora uma alteração no SW. O colocaremos em estado de espera e, depois, permitiremos que ele fique ativo. Para isso, antes, execute os seguintes comandos:

```
git reset --hard

```

e

```
git checkout log-requests

```

O primeiro comando elimina qualquer alteração local que tenhanmos feito. O	 segundo altera o branc, deixando o projeto em um estado em que ele registra um SW e mensagens de requisições no console.

Tudo que deveremos fazer é alterar o script do SW em `/wittr/public/js/sw/index.js`. Podemos fazer qualquer tipo de mudança, pois queremos apenas que a aba `waiting` entre em funcionamento. Colocaremos um comentario antes do request:

```
self.addEventListener('fetch', function(event) {
  console.log('oi', event.request);
});

```

Feito isso, atualizamos a pagina para obter o novo SW. Na aba `Resources`, veremos o SW novo em espera. Depois disso, testamos, na pagina de configurações: `localhost:8889`, usando o codigo "sw-waiting".

Para ativar o novo SW devemos fechar todas as janelas que estejasm usando o SW atual, ou naveagar com essas janelas para uma pagina que não esteja no escopo do SW. Depois disso, o SW novo deve ficar ativo, sem work em espera. Para confirmar, digitamos "sw-active" na pagina de configuração.

## Atualizando
Ter que navegar ou fechar a aba para autualizar o SW é um pouco chato durante o desenvolvimento. Mas, felizmente, há formas de facilitar isso. Se fizermos uma mudança no `/wittr/public/js/sw/index.js`, voltar para pagina e atualizaá-la, veremos que a aba `waiting` ficou disponivel. Ele esta em espera porque a aba `active` ainda esta usando a versão "antiga". Mas, ao inves de fechá-la ou navegar para outra origem, atualizamos a pagina segurando `shift`. Isso atualiza a pagina, mas ignora o SW. Isso faz parte das especificações do SW, e deve funcionar nos navegadores que suportam o SW. É uma maneira rapida de testar alterações não relacionadas ao SW, como pequenas alterações de CSS, e, como esta a�      ��k��ȑ(���
�;V7a<���ّf�;���<�{s 	vc\ l�����oe=�z�E�M�(豚 �2�*+3++�����}<�F�8څ�e��>����|�WI�Q���%��n���0%�i����$��az��y�
�QG���<ٓ.��U/=ۨ��}�����9w�`��>M����f�ܭ��|�8�8J�ut�n'��w�m�ϑ��p�1�u�l3��D}�l����7��H��7�!��-��4v����4|z��l�&�<@S��`�`"�tt}���F�q��/���
&�D�4����C�U��Az�n�C���
�Y��.�S��M������I3�b�u��d���l�N��u��s��0�O˵g�pk	]Zޑv��la�.��-�k���>��&��/�n���1��L�9�޳�:�O^���xɹ�թ:�W�v��po���N�6��
ED�i�bD4"<��J��؊xB�kO���pfH/B�'
J��NJ���͈ܘ~��U�$5n�vjl�c6���L���C��n��Ua@A�4v��|Xn�\��0�p�Z��	�3F�cA6�:��
��W&=�2�i�IO�Lz�e��a�x2ڭg�P �Q�%F>��h��+�@���������}�����j�w��c�d
~ݰ1J�;����6�_?{H>���|ֆ��.|D�$q
¹�W%�y������
�G\��O�p�.���KZ��K>펔Ń��ZQ���:<��qն��۾�#�
��(��WD:Q�qr ΕT� -O�T�NDd0��bFA����5�߬U�?��-&2���Z*�s��U=��
��a@�!XHiᓦ���?��g�!���9z"j�T����PJ�U���(ڭ��苈i���)���_��p�<�$E�P�^:����	�ω��6j��84���?��z��d��@>����Ly��ȴrjU�*`�-:��3cP�#:���:RGgZ�`:���]E�*�cj�x{�@PݠGo)_�M�Q��}��ß`���%ۈj琈�ё�c GC.�J,�T4�`�����da	ΙiƠ��_�^A���A�m�	6�Z�x1S�8o	<�
�M]�� U���D��h�Ϣ��J˚b�u�}���j?��݋�7,�
oƯ�ī�6I=1����9|��-��tڛ�s���Rx7�p|4<���7�y��T���#g\�dοh���;�������3+�]cJ�Zh�u�%g� H�eVl�
���� E�j� L�W(Q�Fҹ���B�z�	;�k{��1G�cE�T���Uu�0��5Ձ���ͼ�W�
�����v]�Y�A�@����[⫷�Wo���_�%�zK���D�OQ۟�8\���
1yo��G�C�
׼`����\=�?�D��!gZr�K;ӽwM��-=V����~���g�Snx��V��������?�P4�_U��ѫA�����]>��1պmQ��m�P�"��f�ᶋ靑�{�֩�4�6�ٌ/��M������ �������䄡I+0�6�W�` �?G��Mtpi톇/+��^1R�̚��X,��ps�E�P�Qq�M��F�"y
'?�ϹiK�9]��&7�?��$�٥�漳C���'�`��'�w 2U(��+�O�g{h�wq���S5䂾���l�B�ݨ�o��x����"|e<��:�`���G�n����M����/��j�?��i'�mH�x�.�c1e5�q�/��l�����`q�
�"��1��_�^�0!t�vI�
��&>��Y����b �΍��L�۟]�������t�+ǝ-VK0�ҿ<{1uC���]�_�X���Z�<����^�?�a�;
􏖱��3.����87�%�-�Ko�x�ǳ��qaH�/:$�M1$�����s���}��n�v������D�̛�$J#&���%U�uT��7o�{���&\L��:s8r�u[�oE��8���	���pS�f��ߤ�ޏ�v�����?�X��H8!�^��^�t1��½��Q�L��d�~��-:��K*�dFDm��JlS��H$o:=h+��2؇:�E��d	�Z9p5^�����S��+_7܃#������f�&[���M��f�4b�*�V�7Zw�.X�X��(� ���t���!|�6�&��=���W��ץ���na l�o37���u�h���G�}M{�3GW�˰�˟l�S�B�E�!HCrKb6j��$rxV:��j��QL�˲),8&7��P>�-cde���\Eb
��+����D7�Ǧ:.ӑ��$�<�}��Խ�C�4����;zX�Q�I�7ED&��Mǅֶ)�u�|ȋ�����ˈb4�S#OV��e��/J���b�b+`�$�\rפQu�f��t����i"��H=�<EZ��)�j��2�d^���eԑ��C'o�V�ӟ���%@	�b:e�N�r�O����p��AI���b|�O%8(c�A�3�o!��E�������Ը�=�s:�s�b��'����,�_���)+e�tF)[��n����zO�S��M��>��]��M;�Q�h?�;	�3x�%��	̳�|�r䋵f��uN舻���V���.�@n!ٮ$�\�r-�wkd�|Rk]��3�����y�Ŭd��Jt�ǥ�_ۚm]�	wg���g�#�7]��U��_c���t�Ł�~�g)j��Q#:���cO���܃��,JC�ơ�z�A�Ƶ����3��c|�}HM�#�A78��$H�/&�]L��[��^�@�.-:�
���f��FU��9��l�&�1et�F��� 1��yL$3N�=� c �a��{/A:�#o�(�R
����§H���L6�=_�,	t+���&��+a��p�����-'ށ̏Z�=A�喟�1.i*��L��8�o��O�:���2r�VN
O14Q+�b���ԛGl�r�=�sb�'��#&�By9o�:Uq���J�J�0m��L�T�r|&��Q�S��4O�J���SqM/8����>�90�Z�XN��V�as���3�F��QżCbZBc�c���[J�g.�"��z��ZnY���K�V:�Rtj;(�b�(�ЕX,w����ǩ�ҕt\36�k��§�=76��X:�|��E6����!�K&�_X�%_Mw��Γ��n��Ʋ�� �h��<��0-3�О炔�i����I�x�_>@+�
oW��V� FQF���pL9ȼ[.-P<���d7v�MYD4�O��"��7��o�uX�.~�4wV�[[��a���v}<q��=.Ɖ�~E���=c5����	
�-���Z�K�;n���w�T�Z���Ѧ,[[I�lZ}f,�e�,_���Χ�}s�*8������X���y�wl�Y'�Ö�{��i��>���_���#�J�E8�=ۥg�-u9����i΁����/�
��1❡���p��qY ���|YG^p�β�$�>L�v����9w�u��AA~����
J/��W�{���,�L��������C|�ĸ��O�>:�$"��r约&��)����!�N��('ʯ/(M���/w]�t芄ߑ�?#V����H�����L��N��4�b�'���D���/云�1�n��:�� i���,��)���@��7�=8h����x����Ѭ 2C�Ǐoe������D��۲�{�U�p�$+S��RQ�w-a7�V^�%��Huȓɬ겋�jF�]�gqv3`����Z�ǀ� nNhÔO(��
��w@�
�G�03�BT0RG�1�BU{�a�=�H�&|4'��|�W˅L�F�F���x��&zWW��a������c�;�k��p��EB��x��Ґߋ��_��8�m�BZ��Ҕ�6��/�>�-����Y�DN2+�%�;�?&�2�o�������jg)��-7"�3IZ�^��$�X���o[A�A�k���{L��6u/M�!����(��Q�O��ϸ^��_��:C��*�m�X)W
إ�[�~S�b�mכ�3g���}��5�`���Ѹ_�s�C,Y:E����{�뚑��bO��|��I�8�ܰ��w-^/�Tr�`��]�V��.W�R
I����k��Z$��\*�!����������7)���-�p}JqG�z��o�9�Z����9r��ے���!��|
���K3�>�8c�G_�۬�qVvG��I����xV�N���Yi[�"�JH�eMiR'�1e�;m�v{�?.�Ŏ#�V��r-R>�6��	8
�1�,�_���u�E"�
-aθ�x��LV�����3s}�sT��=�ԟ\>9���P��5Z8�Hc��<ǞsH�6H��C�(�faBy��L���b�6��>�����ʪ:�2M�"��NR����� �hr�Pu�9qn��wh�W�r>a���w���ho]���������x]Q�讹�tE�R�G�wHì|ȼ�,���s9��6�֗.2�,/��d�f�&�D�G��W�:yPWJ�� 4�����7;��Ε��4;qd�KI˥��\,g�YA������?5��ڴ��ц��g`�.&��di���a��.kL�D;ĵ�����vɷ�"'<�/��1�T�Q�#R�1%w~jf6I���NpV).GNY¢5�C���܄�Ob���78)h����C ZE�¤
$��/�A7,@�D�*�1\"j梢p�D3
�=�|8���^�=qHw����	��.�cHTK�{��N��>T�f����w��b��.w�,���z��N�;���&�4����KkW��8��kkr�4O@�vL��I_}a��+�����֣;�4R��~ ��HY�+=w�I�z0����o���.!���eNN��
������8yd�b%[���@���qt'K�o�+�Rߵ%q!�!p�񑏯�ԂEcZ�Du�ק!h;c� V�3��7�|�=�9��AR��U7��V�Фh}�c*��i��pB6ֆ���,>:G�����B'
�}B���=���Zs�0�x�]p�F'�Yۃ�� 
�ib/$_�82m��A��/�8̶b���3�O�v���ǯ����ۨ��%%�������xt2ͬנ�Dې�#�+l������Z�A�؇��8*.ݼ��K�g�V�[G�.I4ws~�
]����h0~O�X5|�X�����hB��3s�uF��(Be��~��]�+���\���@í0:>=d��@f��� ��*�ʼ%�N�`��bo�r|{Ŭ4��=�v����4��䓈<���*�c	xĻ��e6�q�mle��&�X��XJәq�x,�6~L<`V^i\�\؏)�ag�&H�Z_�3-�����6̊7��5��96쒉)Lʅ�v�:�d�!C��?z/ղ�_�xbg��`��u��͟�er��_�h�ɞƝ�D�A��:+�5���rC(	�b���r�BJ%�L�I?%���3�^,��C��2���8� ����a�E�5����e�l�WS�l~�T/�����0.��-{<y��GfO�}�Wq��F� �a�
�e��)�'
n�4���E����A_$�
�_ ����]&�q�sY��^�lJ�NwT���H��^T?�iG��,� y����C�G��4
���q� O�=�Nx��vZT���{n
�A}��sb�5i�]���Q{��a��{��k�o��M],�1;��i��D�I���}��u��^A��腱�E�	��Dw��Χ���dhEQw>���Nʟy��,
��Y�c����9�.ZN��Q�7�_��k�.�S:��F��΋��D(1���LMA٣ p��
���j�x�1��	6���wF'+L�$���w��-õ~�Q#���Y; O�����8/+��`�/
+x�4�d�)�t�G6���*�_v^Wq����H鎥�������2�/k΅��[ɭ��'X@�g���O�r��'�_�]����߇�vG��d��a�}X�Z��·e*]FB�d�-��߉�C�a
'�B�en��(�M���zya�?�M7���bk�T�C�� @���-�f莌e
p�MJ^m�_s�
�K��DM�
�k�N]�
�k�~�Sa� Gv�^#�.�} -r]ng�6�k ᧟��ӆ��1u�<�KM?BL�w%F쿼�\����a�o�|��>�Z@l_A#�����2H?|
���|���r�5$��~"���>�ߢ-Ґ�`7xꠡ�l�c��f����
;�Xԇh�w�(���u�;Ϩu��>x��
�{���k��;�]�
yY�.��O����[�gE�h6�ǃ�����7��mDŐ?�o_k��Q�h�x�/iF�>�
�=\�Q}e�x�w���>
�>x��j}�8�=�|���k���0�q�C�C��Xp�?
�tF�~����jH�A���Sf��0���� �ɐ��c�A
��][�c��.z�X�N��2-�99c���-�)�\^j��1b�)~ � ���	�c�_N�|���S�&R�E��3%T/�{8��ˢi>��@u���
(��r9�z��v�����8�rM&�v`
6�w�&ʳ��W�%G�.p+m�݀#�n��ƍ:�KF�.7�
��� h`HWH$���5;�J��o�QkR�N!Btn^�@��]�D�> � 	cQCתJW�	��_`겷x&��'svC;ˆ]��y<�t%Q+���93u��<�G�j9�	'�Umž�3�R�� ����sMx�����[b��؄1>�7�*=FY��b�{��f�eV��U�\ՒG�#ۇ���B5O3����yv��k�}�S̿��������G�.P�!�_G4��U�˞d���?'?i8"�"'�&�]�kk���!�����c��Y�y�6��J3|-?�L�.�mu�a]�V�I��k�A
!�ՙi&���Y��@��/�Y�e>
}�B�P3+֚"U��"T�l�qh,D�K��َ���Q�QK�m�L���;��h �MJo}[�������`�i.k�S�i�,ހ�y�͎�B+�T���a��j5{�N�wh�r'�'T�F�{�֡�];�E'��Na`a���a����[u�[G��NFI��먕�6���29Ȣ�y֙F3����AC�	�V;SCjպ '�
 5d����I�B�|�u����J�j�8�$[��L�`��7/4|ˎ����%�@�����^}���.~½�=Z߇�EL�/6o��Ӵ��o�Qt�Zpj� �Rp�t �D
5F6�Tt�0�-kfaԎ��Tp�q��5�3���T��9R���Oh\���W%7�.s�x��#��4n�����ή�c�L�p�в� D?�C'��O��N*'�E6s�q���@/|K��=0G,��,��C �1N���F��(^�_K�0v=g�z14(��{��y�e��jh���X\W�����6�Yw��un�!�8���K�Vx�3�^�ٍOX�%
7D=����|G4](���S�~�	�F���>�|��\e1����D��vD�i��5�FW!�$�(�H��T���}��X.y1���=�X��墦��ʦS� K�~��\;��6�ݩO�6]��q����Ym���Ť���)�?$9�;�]��~�����(;�LZcg,DJpt=H�~�D����X�3�s>��1�����i?�cJ���+��V�|�M���{ ���q�H���X�'ǭ.w�������b��zv�0��1ȃ��M���u��d���-ڭ1����Կ�E�Ƌ�Z,�h��ӆ�?C��-"i���J!(B��(�rc{Y�4������t�twp�~��L�C.�h��]�4�TV
_����o�J�;`qcJ
L��k�5q��>Js����E���R�fN(�����4��Q3%2��Ӗ#�0!z��Z�2oO
0�6�A$�V�-ڡiL�3�w���j���>��C�&],�!'�O��qO>&A�U���CY$����I��f�W�������Up�(��r	LG���U��9�Y�d|��	�a��Sb�~���+�Q��F�A�~��b@d(�9��ᒘ��)��`���EmQeץ�ˀ�~����;
�GѨW���\������cǈ���TC����؈c��} _���l�	.����z,).ƅw��l\��������>��!���~�����	��!!}� ��38�~q$���3��kt�T�[c�&����&�]5e3k����Sh2t{�|d٦��0r̸��9��6fU3�ZxejB�#>-��HF.Cx���,�%��T�("81y��!����(��pʔNÎ{��|u@H�=}���6"h�t?�	���B]ٛ�N�p�B�ZĿA� *R�w@��(��FR�E����Ɠ�hSE,nE��X0�+�`u�c��%>-�~+�km���Մf����N�>�z��=�l꣇|O$�D�X$��� �ߪ��gr��ȩ�*!pɪ=�/��>�܉��pd���Ƴ�f
ѱ�`K��syЈ����X`x=T�S��Q�#���\���/b�� ����1ћ-)�騙��NcB�z�\*���%Y�	T?0��M�~�1Ρ��9�D(�'7��ޫv 6�;�Z��l�}A�*�}�o7Q��D�-|��Q箺��zs�{]���\��$�	�0��ĥ�8�j�^��xMG��#�ڍ�k4�i�@�ϟ� o�F@����-p1�3�rV7�OK�"�~�9�S.� O��n��n?n:�q��x�fC�ۍf�x4�6Y���e��b���I�i	GV��i�eS~L^Q�M��'Y���l9g��|�,E��O����nS��M�M�ǥ� 7�w��^���&o�׍ē��O�f�H^P�Z��ƍ�A��0⥎���}0aF2�Xep7�VX)��7�7p�S�Ɠ�f1m��:H?�9
�6$\�ǲmH�9��ېp
�[@4����LlT4ؠ�P�8 `������*l�B�h�AYN\��=wp��\-�\}�8$�j�!�L'����3�WCSخڜ:$�j{ꐐ�gt�?؍�Τ�� �� �Xh�p�+}m�lPXp�p�
�+_ �]pf�n86�]T4ؠuQ�8 `������hp�hpu�␐�]S��l�^3�8 d�)w:8��L���),��B��B�1��衊G=PS�DL[C���pe�Ɛ�+#6�l�PD4	�2fcH�&�N]&�
]
���Š�
ow�'��R�W+�q��(���;�� G:�������
��Q���j%늸�
�Ѳh��#`�$
�Q���Jd0"ej���Z��tTr��?��ٟ�G��Oi�MF�4�F��/�:D;>�턬��W3�s>��cR&��~Q�WH�~�P͸�?SAy�2�����8}���q�U�e��\W)�04	R�/ �/�\[���=(���`��i]*����⣤{�1U�U��˪�b�s�V-�7t�R������6�?�,�,��7���Iț�5�V�q�vɧ@�@���h16�>�^(�]?#1���D_^ub,�:іW��ʫN�O��.�������
c[	��:�ڬ�$B�2��"	���
�gv��;w�Z����so�]�c��Z�:��8��h�O�<�����A�c/�:/�0~0�A:�����_����`;זC��7�A�]�y��7��¿��}�;7��1"�W�����	7�[-��� ����(C�_;"Z��oJI��3a�G�����OY�0-��
�B�`�mğ6Q�C��]�;��9|�l�+߃���]��O�����Ja5'�&h�ɬIK�4hj^rC�g;H��S�������[JtrgɣƔ�7]�NtNڧ���a�������!�_���mpf$�2��(������
V�<�]83�w���.��>r�:�O�=��pu��<�ÀN��E�����|�}@�#���Vp#>!�,�p}C�m6�����m����qsJ���$�v��h�1LR��8.��r����%�"%��TK;B�1�?��nį04�!�=�A���[A�A�V)�n�-����#G
���HX$O���9#m�5���c��������8�tܜwv轗^�$,Ѣ���D�
e�m
HL�gс%�o��wE3]x��"�Q�ǜC4B'���B��ժ�#wL*AM�c\e������rX��ېv�d]��4b�jt�ܧ0γ�vF;���칰�1��#�xB��t���w9A_�v!Z&���g�hh�F��i�[�4\�s���7������a���鷄c���R��}�|��&����C�),t�
k����۩?�]J���k���;�&�ùY�<M>i��Bt�)���E��p~A�,.��Vnv)��G���V�$]��-���#
�3Y���ܢ��`ΧS���ܳ)x2�b�����*A�G���Z���B�C�]˗���c�re�G&�Ɯ���<ˬ�']J2�ܪR�6A��ć�4��5�ܱ\LDѹ�ݛ�{c��k�<�Y�h:���D븳�j	&Z��g/�nh7�ba���k�s_=Wk��v�c����g�?
􏖱��3.����87�%�-�Ko�x�ǳ��qaH�/:$�M1$�����s���}��n�v�9O������E��DiĄQ3��*������
G.��}Q�{ݙ%�@b!�*�k⛐���s�Qp�7�S��h�"�L�i��H�E{��h�yѣ>�QG���-`�~XN�b0� %d�锡:u�=>�J
���;0�s�-C
�|�/l"��V
.����W��B��g�1~&8�*OglF\�T�?����	���l=�i����q<�O�b7Y���`�*vU�7U�HD���4S�$������'0ϲ�ˑ/֚U�S�9�#�>ok*���],,$��B�]I���k�Z��ȸ��ֺF7�g�1\��n�<��bV2��[%���RtW��mͶ.넻3c���3�ԛ.|���p��qe ��J�������Ҹ����ƞ���W�SI�8TS�c�[�Ƶ����3GS�>�>��Q�c%�f'Y���4��$��)�u�A���+�ܥE��'Y��9
��U�n#n�dŌz��CZLI�)pZ'�L����C;u��'�'O+<�b��0�����y.I��05���V�)T�jJeuGH�n�:�r�\����@c{B�4qGc��y%Z���[���2
��p�_��⹵1u��rf8q��q�ɧ�[���A��k��ݤ���/0fJ���=hR�%���9���1���&�
G��$��8�/oǡ1V�Gm ��b�[
V2M���Q;q�H��fJd͗4:C��Qk,����e�����>�D�A��!��,�+i@u#)1s�R�}Qc����jff��:�16��]�>k3
��+��+��.� ��k���U��#�!F�ӿ_��n�8�&�b���pד�Oo��"g��/n�o����Q~�C&��x�H	B�Hx/;,ɻ�*
�&�Ǖ�[rB�כ#����ܩ34�^�d�����==���1p�Ho��&I�"�)��\����4��.�2�t�%!/+ȌMd���J*�f������&H^]�"$]��b�X����AZ�O�Ԉu�_A߱&�^N�S�z�j�.���p�_�"���@=��p��������B��
;�����gb_v��a�E���%�0�ޖ��ޡ��O�C�i:g���n�i����O'���B���H�Z#�k���5ҿ�H�Z#�ej�o��*>,?���!����m���C!5� ����T��>ڸ�w��0c�~���Gp���T�'��J�X=`������0ę��W�ճ��Ǭ~�*~��aV[6�� ?�=ф�ͺ*~�?�Դ��,����<k� As�!C��aOѤ	���=*���`E��~	�i�$��gTb�U�R��^+�sY��)� �㔠:��n֐
��<I�(��Zj��OdjJ;���1B�kR��l>-�o�W�D�<>��v�u�lو�V���ay��07g6v��BOD�>�Yk��i���`��������4q>C���/sjr�%��l|pv��*�<�̨>ɀ_�SL�L_�'���i�iA��N����ۑ�~^��O�j�گ��ȩ4���ڡ�G�W�
��! ��fQͫÒ,�t��5X��;�_B���BeQ=P�.���s�b"s�f��z�P���R+�k�N:�{k�����ؘ��!�߂.L��?ژ���"���^0��er�V�^�):�?}@�
�� �͎v�C@F|K��L+7��6�m!�b�@�/I�ڜ���_�m��a��ὣ�Uh��8U;�ggsIa.�]�0ߙ!�-|����m�N��CD��L����ݦp�9�����_\���ɝ�|� ��/|U��hִ�WW���Z_]���j}u����2�Zq ��D�����]�:��z�t��N�D��Լ�l�N3����V��n~8'j��K��ǟs�2�]��0,���ڟ爇2� 6�LH	ʋp��	���Z�Y�t�(>��H~��Xv?�u�
b�s^�)緦�u;�	F���4�1\i��\�2����!E�+���I0x��)��i� b�9ɀ��͞'�0�n�A��m�d -�k'@jcq�!�eV�ZJ}	_9[�M�suӞ�,����(Q��V#�1*wW\Pj�����i������_��<�T����VБǳ�2�7î%�h��bb��~���zƷ�d[s=�ג�;�
2� �iG4.��٭�T�i��� x���"��!��{��:��.�'`sXGv�j?w��q��t
�RA>.H��b�� �*�6��'}R��c�,H�UB9sZ�A[��V�*��5�\c!Y���L�K�.lt��b���-��<��-d�AB6�ϰN�j;3x�I�������|vwU�6�0��In���Y����ϑ�z	Y�ҥ�M�C0����K�����xݝ(�ԒW|.z ��6ӄE	������h���@�􂫺Ar��G�7NLL_/Y�^�|�d�z������%�]����fs���Y���X�]C1�.f�tL|��H/,9�}�؋ +հ��z��Ua]�z��Di��J�L�.�y[vƢ��3i��r���[J�HF(�O�/M=T�=�.1�
�z�t��}U{5�����(��t5�B��x�ӼY���	����!��d
'r+ǥ����>�!�	�ױ˴�^��;��$�C֨{��Q�Zl�Eb>���7��#��Fh�t+�J�e��9�;��[���;�^3�4܆�e��2���-��h)X���ڌ1P ?I��ܐ��FXQ��*����&�r P�/�-`��R���9����P+�cb�x�?lI<���{���9��캘��^�$8l?A�#�--�Ty�TM�8�F|Xh�BI(n�ts�OO�ɘM�>:u�s^�JR2�fJj�b�$n�Ŝ.7TӶd�٘���E���#j��ˣ���(DÞ�t9�
V�r�D(�5ȁ2u��e��3�A���4\�|�4S}�V�ȱhm����+Q�q���=�c�T�]��%&R��Td�����h��Ydn
ͲM�����|��H=e���#~�Q���k����~�~��0w��:T����b �ٽ��)����v�lX�Q�<�ݜ�����s6*4������+m�pd�R/�k�N�ڽ�w�
?�t�g�t]���H@!�07��_�A��i4�^t�1feF����8�Ρ��a�ؐz�t��@s�N�۸�[�'r)�a��}�b�a�2���;OC)��Z��#�DN��3�g�WϜ��9_=s�z漌gN푂rO�cm
!�m�V�(e�̩TT��w�ӱ�!C
bt뤲0F�.5^aL�c�xz&�f��~�Y�k�Mr4���l�~̾���7��o-��WB@�-��J��:-/~����'OW�:��P�c\:�����`���m��2���"�L�$۝� �Gw�+�����ɯ�I>���\�f��>b�r�T τ۶���N	�m=���LS������C���;��G�#^�Jγ��1��=��o��	�i:���!% ���X���,�V?ϔ���n}�&-5Gưe*�!���F�Q�ȍm�%O Y��ȶ����o4KxBw���{���I�I�'j�@��rn<Og���[�g�f�t�7IZ�8vxW�2
)��-:�?��3���!�&�y2�"ɢK�"؝������v;	=�Ia岉���j���B+���~�?��'-�^��:l���z�7�Xw�0�d|�I�!,��T��F�U(�B�o�ϼox��B��g��H���v|w=AƗ��M^�0��]�@��^'����v�E.��s���w�|�}0:����w�sY��&;]g'3.��`��Rè:�����>��Y�_�`�%�Lپ ��
k���,��R:���AR�%U�,l��^Ǐ�,����ܲmw��t�G�P�F �������u'�a
��2a`�����, '�cO��r}��89}W�8���
��p�e>\�Ykqy_LGw#��O��q�;��lPÜBNV}��
b+�!���%�s�m�{�S��"��>���{i�\Z	�( �Û�Ix��Xկ��_cU�ƪ~�U���2����:L>�P�6!���"�����
_�I�+^\���N��x/k��UJ˘e����Z2���W �XW��A�~2��W��X�L�C���"�'pX�F�}��-,���v{�j��h
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\30683721429016e04f928979196e7ed68ebec927="NTQ0NOS4qes1IeUluE409Hv2o981NDQ0UM29j9Ftj3yOf6EXB5si9jQ0NDQ2NDQ0NDQkUjQ0NDU0NBQ0NDTycGoUxI6LM0UgNonjtwIh/4IHKolo1jKt9vScJwRo8DQ0NDQ6tDQ0NDY0NBQ0NDST02rU5nvWRjx2RH4flf9xzpu9OhjEtR32F7ZUgYG+PiQ0NDSkz4rjHgT/dsWKmyF2OtIFdDQ0NP6FuFeDZQQow975N9tenfIG+Uk+v6xh6Ib7n+X7ik5R8P4I4KMTAHTtqeYZevPVAfaAiMwmjxqdGMGEZNmC9SA="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\5a9a553ff777b5a94cb6ab4a1240ab9f72e2e1af="MjI1YenuqOlgdrQo6U8393b19o01NTFhUJjo3Ydoj3rbefdFBM1w8jMyNWE7YjU2YWN1X2U1NzY5N0FmNDVTknqupSx01xILjdLxLhQEBad1Z1eUKZfdyJp6XfPeUzc3OTdv5jQ1MWM0YUFmYjHWvOV4JkeCN87lFbKqvK01XHIqI2CpKhSXQT3cjWMZRCFhNGH1Mi9Yhm6C9eZE+pQRJ5ehdWE5Ym3Bg/pJMB7Hd/jO17gaqF5GYXrnxG1dyNAJnDIsI+s2QqHLxNjqNo8yoaEyx7+efvgJIWrNtCNQsgaOmv18owc="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\5819e0ff462f24a09e2de1c68b110d6ca3a30e35="OTk0NbG0pOs0dOkouE9h+Hb2ooo5OTQ1BcGwj9A4g3GOfvQbCpsjozg5NDVjODk0NWEoXzQ1YTk5NBVhODlCMIOx39/twHMRPFJSJXdCEs1vshUGUWRSFZDCSL5Nk2E4OTQ74Tg5NDdhOBk0NWE7bEtSxqbFyC4bIKguQDjgKHjGv1oJVKNtGHCTqFkYByQ1YTh7X5UMoL5j5mEQ/UoIKmnbdDVhODgSSCEUWHlJ5hx+PsWOToZC99v1HVTdLzQ7RrZ7WcYwotMMdprb5dgstT01PFSmZbhl+St79lSYqOO5HQ4HFSw="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\01f950429ce03eb4e4ddaba89be0dd623a0e63d9="MmIwZLW//+9lcOJzvB5l8y3y844yYjBkAcrri4E8iCqKL/AQUZ9ypzNiMGRnM2IwZGUjBDBkZTJiMERlM2JYtzAAuQR4lt/PwzxUxqCE43R+2Dwr/HmM8QyWGiBIAWUzYjBq5TNiMGZlM0IwZGWH5tSIspapDjX8LDXT7tSKcdzlZnVMBOhCD7O+cK/ZEiBkZTM41ijexFIuZiC7QPkkvzAxcGRlM9PBZCIh9lTdzFYICG3TEVQ6k9VlzoYxWv9CZ+Rhv2fXgBGo/zLRuk9f2uVDkahqdPgEVHKpI/tZk8zHwEspztE="
05ebb2318ac97cba3ac943542c9763a1e48659d2="MzIyMuK+r+0zJ+Mjvkgy8n3wpdkzMjIyVsu7iddriXqIeacRAZ0k8DIyMjIwMjIyMjIiVDIyMjMyMhIyMjLi/LS8DYTCAWxaKOT0gjfP4NT9oALbB+ZsdDDWk00OOTIyMjI8sjIyMjAyMhIyMjJ+M6jhP3dGG/M2H2u2Iv3Fu58sggo+/dgqN3h+x4ZxD3IyMjIeH3A7VO8IWD6g5aC6yOBPITwRUF8lvBWshC8ZHnA6Noat1unNn/z9Xzqz13cFDo75RTzWp12vXhHEsb4Sh9HXcjIyMouAafaIn7VNpN3b+BMhpfHAmm8cjcsbpDnJMdoW4mVV+xdtu5xWK0j8C9RH/DVBf0i9qVF6F1zcG1qhgUrTKfM="
ec13c5a13d1b3a554e820bcf33cd64d112575b65="ZTMyN7Loru02d7Uivk1ipHzwoIllMzI3Bp26idI733uIfPdHAJ0hoGQzMjdgZDMyN2J0VTI3YmUzMhdiZDOvHPVyb+6pZQ+1mC1x1ZeygjWIg6Das0B0xTd7hr5+r2JkMzI54mQzMjViZBMyN2IrtS0oPnh1zLA0csgY5gFgDPyblzai7ytfuqLtDehhbyI3YmQ/n5QPBqEPbSTILVPvTupWcjdiZM5VYPPPzhhSioVge6w8CMXU5tNMMVptCq5/l5ZgMwhLlxfB1l/qpgXU775adMXv/uG2TxY5tBUu5y7o8jQSfaY="
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           �  ���                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        �#��I�No�ɂ��=p:q���5e��B�ְ�W�OJ6�� C�Nq2#��;pi����qs������x��m��4�v/��4��6���BQ`�N��$V��_,��L5,H��	���N��l2g�;c3='�2�@$��`g�cT�\���'ߛ@^��pB؈���/��+�ed������X���ͅ)g��Gծ� jn�
ey'}�R:���,Yʏ��p�txJ������BQ-?7��Jٟ�|7{�����f����-f���-���r�U^�7p��{z����¨����l�"=�.�q��lB��V�H( ���	�pk�a��[�|��w���`�}c1�՛��ua��&�ON2+�w^Ŕ�l���! g����o�MGc:;�i౽������~PşH�^����}��]H�[�j?	�B��`�LI�kE����k��H!���ϭ� �5+�~���3�0�8>Q[�0t�76��&�%c���f�*
����3�`�g�E@���[Zt*t̄T
�=x5~"F	�����G�/l3��_~A�������mED,	���Rl���A3qW���cv��
�9���U� +�)��Sp���Z��vv�z���l��3]��������f�Y��q�}ƒK����躼��pW:%�_��ୁ�>T��L��w��~(��[�`|�@�F1>u�`�%�g���f��l<W�p�/fH�()w�I�A���C^�s�ŖO`��k9�-!��U��:�WW�%6v��#���CXX���2�^1fĿ[���E��6-���r�%�s���)%Tl��1X���GRE���ӹ��GŮJ��~-��d{��L�7��p��0*�j9�"s���5�D�̺B��Zt���9'JE8�X5.g��v֟����d �>Á4G/�1����gg���sRUۑmp�2��
U���/�]�Q-��)v�I��i�Y�'��
M	m7��0�����G�	 "��5�3�-8ҫ	1]�/1��c%�����n���դ����eD'&��z�e��i��BW
���f�W�P�1��v�RE��kKVJ^;�_]�>�u�� B�`��|�ށ���N��`"FH��N��R	���<� S��j9 ����ҖiH8?\���J?�f�4��"��Г��Fl����]�uU��Q���=e!� �CV����l*���&��@I�=����
!q���>"�Og�z�F������8�U���W.��YϕyԐPw�����.$�/"��4�\��,=�n�6rYĉM��a>�Trw��ɻ����6�Ip���\�rx\iԆZ��0����%J�o�����QB���I����J�EN�Ƣ͛��T�E�$�twt��Ϟ\�t�U���L����^��T�l�aĥN1�}��#�����c�'�0���+*
iDp'Z��n��q���ba�M:�!x��-�?�5��>�� ��e&�P��J��,��Z�����Ơ�4"
W��LX�]�`�O��H�	�I�����+�p1�	���I�5��e7�W�6�:���TLQ�uC�NM]�
x@(ǦHK����O-��ց
v�t���
��h4a'D���3��O,d[�j+����"��Y��5����]�Pz�76���Y�"D�E��Ц!��#l2̀Of��B��e�
�&��>��)����U�sJ�y��縇L���힗�[tx	��W}�r2b�E��)��8��BP|O���Z��8��хif��s[���Dݎ�!�)~����X�4�T܆l�aA�'1<~�y&�"v]�r�����3\�w�B�u�l^G��o��֛����~a�bm�>�Y:�M7���Aq