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
Ter que navegar ou fechar a aba para autualizar o SW é um pouco chato durante o desenvolvimento. Mas, felizmente, há formas de facilitar isso. Se fizermos uma mudança no `/wittr/public/js/sw/index.js`, voltar para pagina e atualizaá-la, veremos que a aba `waiting` ficou disponivel. Ele esta em espera porque a aba `active` ainda esta usando a versão "antiga". Mas, ao inves de fechá-la ou navegar para outra origem, atualizamos a pagina segurando `shift`. Isso atualiza a pagina, mas ignora o SW. Isso faz parte das especificações do SW, e deve funcionar nos navegadores que suportam o SW. É uma maneira rapida de testar alterações não relacionadas ao SW, como pequenas alterações de CSS, e, como esta a�      �k��ȑ(���
�;V7a<���ّf�;���<�{s 	vc\ l�����oe=�z�E�M�(豚 �2�*+3++�����}<�F�8څ�e��>����|�WI�Q���%��n���0%�i����$��az��y�
�QG���<ٓ.��U/=ۨ��}�����9w�`��>M����f�ܭ��|�8�8J�ut�n'��w�m�ϑ��p�1�u�l3��D}�l����7��H��7�!��-��4v����4|z��l�&�<@S��`�`"�tt}���F�q��/����\[����}w�W��ߐ����{���ܐ��Ǯ㼺����H0���Mnt���}��w��ǯ�k��%���k����ԁ���Z����,y���T��w���Oi�������݁�W�>wM6˕���t���'���UZ���h4�d֤��X45/����i��B,5�����>E��q�0�����(���:�v��>��㧇(G�>X�H0|J��$6q�閽yW<y���p�l���Oiɺ�v� �w�?�A~Ȏ�h7��݉���>�",��]=�z�]��$yȒB(�h��!�_���mpf~>��9��6�c�>���U�z��9�y�3��v�(�a�[�b
&�D�4����C�U��Az�n�C���
�Y��.�S��M������I3�b�u��d���l�N��u��s��0�O˵g�pk	]Zޑv��la�.��-�k���>��&��/�n���1��L�9�޳�:�O^���xɹ�թ:�W�v��po���N�6��
ED�i�bD4"<��J��؊xB�kO���pfH/B�'�����ݽ��`�
J��NJ���͈ܘ~��U�$5n�vjl�c6���L���C��n��Ua@A�4v��|Xn�\��0�p�Z��	�3F�cA6�:��
��W&=�2�i�IO�Lz�e��a�x2ڭg�P �Q�%F>��h��+�@���������}�����j�w��c�d
~ݰ1J�;����6�_?{H>���|ֆ��.|D�$q
¹�W%�y������
�G\��O�p�.���KZ��K>펔Ń��ZQ���:<��qն��۾�#�
��(��WD:Q�qr ΕT� -O�T�NDd0��bFA����5�߬U�?��-&2���Z*�s��U=���9Lg�Qd�'����� ����h>���%#F��5�戬��=��ܿ���QJ�I&a�G';��v+[�"��3.b\���y��ȣ�Z=��hx��(�'� ��~��e�Z��5ɂ8�%Z�	7����MMjX�j�X�w@BP˰�E�i&H$#��kE���6�.M���(�:c�SGٻe3���8:#�Hc0o�D��?�#�Ȱg"��x|�LT�0@{��Չ���m�A#�f��Ȥ��5bR
��a@�!XHiᓦ���?��g�!���9z"j�T����PJ�U���(ڭ��苈i���)���_��p�<�$E�P�^:����	�ω��6j��84���?��z��d��@>���Ly��ȴrjU�*`�-:��3cP�#:���:RGgZ�`:���]E�*�cj�x{�@PݠGo)_�M�Q��}��ß`���%ۈj琈�ё�c GC.�J,�T4�`�����da	ΙiƠ��_�^A���A�m�	6�Z�x1S�8o	<�W�Ųc�[�t9�"2��.���EX���J��z&V����m����v�}{��G�=�L�E	���*�������a;���9�4�!P���+vIU�[����\z���s�r1��<~B�:����#�^Հ�a�<�F���f\���g�c��+s�ox5�X��/��Rwk�$~i��j|������q�;Rz�G�&�+q
�M]�� U���D��h�Ϣ��J˚b�u�}���j?��݋�7,�
oƯ�ī�6I=1����9|��-��tڛ�s���Rx7�p|4<���7�y��T���#g\�dοh���;�������3+�]cJ�Zh�u�%g� H�eVl�
���� E�j� L�W(Q�Fҹ���B�z�	;�k{��1G�cE�T���Uu�0��5Ձ���ͼ�W�
�����v]�Y�A�@����[⫷�Wo���_�%�zK���D�OQ۟�8\���
1yo��G�C�E��.�������vPg�8H�ع�P"݈b���w�$u-{<Z���`�d+��*l�����m���yq�9@Rh�pX�v�$d�4�cљC���N��N'��m8F��O���+�x��(�C��4$iE���k�Jt�q@��C	=YG��X�A���?���G�_�9�V!����Eљ_��s,�s�u�:�������>x�o���;�"�Y?��Y��M��exʔG��;t�}C� C� �tE�X��QvX"��%]$�<��R����1ɍ����Y��
׼`����\=�?�D��!gZr�K;ӽwM��-=V����~���g�Snx��V��������?�P4�_U��ѫA�����]>��1պmQ��m�P�"��f�ᶋ靑�{�֩�4�6�ٌ/��M������ �������䄡I+0�6�W�` �?G��Mtpi톇/+��^1R�̚��X,��ps�E�P�Qq�M��F�"y
'?�ϹiK�9]��&7�?��$�٥�漳C���'�`��'�w 2U(��+�O�g{h�wq���S5䂾���l�B�ݨ�o��x����"|e<��:�`���G�n����M����/��j�?��i'�mH�x�.�c1e5�q�/��l�����`q�
�"��1��_�^�0!t�vI�b���ԡ�E��-���ĳ^44Y�[´�J��9����������M�0`/��pl0��N䲛�G����A���u�+�E��uԲ�Ok���1�_�=L����/���4��͚�i�I�t�M��-:����eq!��o���KA���@��h5Mҥ�!܂�a\|�ƫԸ�[�S޲rY�Q���ܳ���ϿQ��T%���9��쫵"a!ƌ�bs�q����*��;�T���N5��	�1'�8�4�2k��5�~�Y��V�J�	
��&>��Y����b �΍��L�۟]�������t�+ǝ-VK0�ҿ<{1uC���]�_�X���Z�<����^�?�a�;
􏖱��3.����87�%�-�Ko�x�ǳ��qaH�/:$�M1$�����s���}��n�v������D�̛�$J#&���%U�uT��7o�{���&\L��:s8r�u[�oE��8���	���pS�f��ߤ�ޏ�v�����?�X��H8!�^��^�t1��½��Q�L��d�~��-:��K*�dFDm��JlS��H$o:=h+��2؇:�E��d	�Z9p5^�����S��+_7܃#������f�&[���M��f�4b�*�V�7Zw�.X�X��(� ���t���!|�6�&��=���W��ץ���na l�o37���u�h���G�}M{�3GW�˰�˟l�S�B�E�!HCrKb6j��$rxV:��j��QL�˲),8&7��P>�-cde���\Eb
��+����D7�Ǧ:.ӑ��$�<�}��Խ�C�4����;zX�Q�I�7ED&��Mǅֶ)�u�|ȋ�����ˈb4�S#OV��e��/J���b�b+`�$�\rפQu�f��t����i"��H=�<EZ��)�j��2�d^���eԑ��C'o�V�ӟ���%@	�b:e�N�r�O����p��AI���b|�O%8(c�A�3�o!��E�������Ը�=�s:�s�b��'����,�_���)+e�tF)[��n����zO�S��M��>��]��M;�Q�h?�;	�3x�%��	̳�|�r䋵f��uN舻���V���.�@n!ٮ$�\�r-�wkd�|Rk]��3�����y�Ŭd��Jt�ǥ�_ۚm]�	wg���g�#�7]��U��_c���t�Ł�~�g)j��Q#:���cO���܃��,JC�ơ�z�A�Ƶ����3��c|�}HM�#�A78��$H�/&�]L��[��^�@�.-:�=�:���iD�և(���ta� ���h˛�I�z�ө؞��6��="�t�ح�/2G�q���qA[�%��yH�)'�`8�!��>q�_sh��q����i�gU�B&�S9�8�j�X���R]+�*�5����#$K7Ie9z.�Z�L���!YqGc��y%Z���[���2q�a�O���xTh�z�ٙ�DW��p����^�.�g��q�;�ZO��M=�籬�JUBV	�>���#�oΈY�>g��R��-���O�H]���\D4��ңU���H��&5�����p�2��
���f��FU��9��l�&�1et�F��� 1��yL$3N�=� c �a��{/A:�#o�(�R
����§H���L6�=_�,	t+���&��+a��p�����-'ށ̏Z�=A�喟�1.i*��L��8�o��O�:���2r�VN��{�ȳ���x�Ra���oL���=����+�q%Þ ���|�9�*�:I-��O��F�|�J*7Tj��IK�ʙ_�IJVaǂ� �i��h��s��TZ59��!9��U�۪�Z�3.9 /�{�f�5Q�A6�ަ��!1�C�'ٷ���~���A��?"Y�6�r�A�8탦=n�iV��c�1��"�AQ�SRNx�#:������%C��U�K~�)�Zn1-���8H�����d��{�.mX�5iiK�i�k �55.��-�9Tb��lQ5,~��-�R��~���I���^���q�{;Zv�So�?��j�3��rvݕӦ�Wn�o������Boն��jfY!g'|f1�����[ݹ�{�*�4G�Z��)�
O14Q+�b���ԛGl�r�=�sb�'��#&�By9o�:Uq���J�J�0m��L�T�r|&��Q�S��4O�J���SqM/8����>�90�Z�XN��V�as���3�F��QżCbZBc�c���[J�g.�"��z��ZnY���K�V:�Rtj;(�b�(�ЕX,w����ǩ�ҕt\36�k��§�=76��X:�|��E6����!�K&�_X�%_Mw��Γ��n��Ʋ�� �h��<��0-3�О炔�i����I�x�_>@+�
oW��V� FQF���pL9ȼ[.-P<���d7v�MYD4�O��"��7��o�uX�.~�4wV�[[��a���v}<q��=.Ɖ�~E���=c5����	*0��HA�����k����/�e��1V�yB®�Jy^��0F�$҂d��X���ۀ��9�����)���=����86����A������5E}��39��+Sc��5N���^������6��1��'������bC��K!�oa�6��"��s�[��Z�2Q�]�WgK4���P�y�H5�O8��	�ۣEӸw��gn͜����4�^���՞tM��P�dR�>�I�$���d�#n�^�,���l�qR�L��2ܳ�<���jTg3�b��=L��U�8'����߇����C/��q'_)��;#���̷��d��?�u?���~����B�?�����$/*����m-bM= 6+@}�2�~T=J�����D@*MZ��O4 :$;o~ ~�d�O`�K\͉��W
�-���Z�K�;n���w�T�Z���Ѧ,[[I�lZ}f,�e�,_���Χ�}s�*8������X���y�wl�Y'�Ö�{��i��>���_���#�J�E8�=ۥg�-u9����i΁����/�
��1❡���p��qY ���|YG^p�β�$�>L�v����9w�u��AA~����')�c�k�T�*]y��F��]
J/��W�{���,�L��������C|�ĸ��O�>:�$"��r约&��)����!�N��('ʯ/(M���/w]�t芄ߑ�?#V����H�����L��N��4�b�'���D���/云�1�n��:�� i���,��)���@��7�=8h����x����Ѭ 2C�Ǐoe������D��۲�{�U�p�$+S��RQ�w-a7�V^�%��Huȓɬ겋�jF�]�gqv3`����Z�ǀ� nNhÔO(��
��w@����5%���%�7,{_C!r�,|y1y�;�y�;K��b�IҊw]�jedsZ.��5�*��C^�;|*z[_���=ڟ
�G�03�BT0RG�1�BU{�a�=�H�&|4'��|�W˅L�F�F���x��&zWW��a������c�;�k��p��EB��x��Ґߋ��_��8�m�BZ��Ҕ�6��/�>�-����Y�DN2+�%�;�?&�2�o�������jg)��-7"�3IZ�^��$�X���o[A�A�k���{L��6u/M�!����(��Q�O��ϸ^��_��:C��*�m�X)W
إ�[�~S�b�mכ�3g���}��5�`���Ѹ_�s�C,Y:E���{�뚑��bO��|��I�8�ܰ��w-^/�Tr�`��]�V��.W�Rj� B�>����E_��y�"zt�;���j{����*��Q;>u�9oH�?��c~�a[ZH��<�S.���^��ذƺ��V�ȇ�
I����k��Z$��\*�!����������7)���-�p}JqG�z��o�9�Z����9r��ے���!��|eH��o�᳈���
���K3�>�8c�G_�۬�qVvG��I����xV�N���Yi[�"�JH�eMiR'�1e�;m�v{�?.�Ŏ#�V��r-R>�6��	8
�1�,�_���u�E"��ا@]�,DN����U�>�Wr[q�B��N��M\Qe},@��'���@�1��,�?(��x-�A�}�
-aθ�x��LV�����3s}�sT��=�ԟ\>9���P��5Z8�Hc��<ǞsH�6H��C�(�faBy��L���b�6��>�����ʪ:�2M�"��NR����� �hr�Pu�9qn��wh�W�r>a���w���ho]���������x]Q�讹�tE�R�G�wHì|ȼ�,���s9��6�֗.2�,/��d�f�&�D�G��W�:yPWJ�� 4�����7;��Ε��4;qd�KI˥��\,g�YA������?5��ڴ��ц��g`�.&��di���a��.kL�D;ĵ�����vɷ�"'<�/��1�T�Q�#R�1%w~jf6I���NpV).GNY¢5�C���܄�Ob���78)h����C ZE�¤
$��/�A7,@�D�*�1\"j梢p�D3�f-��@?�ilh�c�k�hr��b�F�n?�I�O��ϥs1��\yM�M�kD���5;��.54���|�V�a�^J�k�}��=���ѩ�7{�vЖsI3s�=���WTg5\�+:�p,�ns�#!m��M��R0~�2������=q�C�Sc t����DϪ��)}ɴ5rp"R|������̵5�(��&��2�}�׎zK8�ۯt�N�����R�NC�\V)w�P~�r{u�j2=T�܅��{��X ���Zv'9����zL�������\[hP�D��/��Dي8{1)>�Ȁ���>S��Xa#�2;,{���%ĭ$G� �Wp,bj�;��b��M�g$K�����5����W.4n����*�?~eβ#�\�jS��:�vn��C�Q��+� ���w�f/������S����ɧ���Jr����jg��<���!i6_��)���p(��1L@�z�ũ����:���a�|1�n�&OM���u��M�j?'#י��R�!5��ִG�0�����0�a��ׄ�uiE�4؆���гR1��P��61z�=�>�����I��7j���s�����j���@b���y���[A2�$̊~�8:_H��)b���}��a'���#&��{�p��I�8���ˢ�b<�[{������^������19$j�&E\�(GD��P� Y���dꛚ�+D�S�Ud��a�ȼh��Uߔ0�,����X�'k�b2�:�!��p
�=�|8���^�=qHw����	��.�cHTK�{��N��>T�f����w��b��.w�,���z��N�;���&�4����KkW��8��kkr�4O@�vL��I_}a��+�����֣;�4R��~ ��HY�+=w�I�z0����o���.!���eNN��
������8yd�b%[���@���qt'K�o�+�Rߵ%q!�!p�񑏯�ԂEcZ�Du�ק!h;c� V�3��7�|�=�9��AR��U7��V�Фh}�c*��i��pB6ֆ���,>:G�����B'5И^:np��j� �7�#�#��>Ĳ6J?`���5?�+���]�e�T��~84�'�,h���ޕL�gM4�<h^4$h2� 4�(d�m�?�����h#������p��[������܍���]����WTi9�jVd���k&���7��G��b֞��P��a�0��;��Nq5�%:3F��Iz�֓P��	=Z��$���K�h��j� 4}_o�P�݅���T����]=/��U�$r$l�%�N�~缒�hW�5�UY�N�����޾E���cdR�ytBO��bU��U���
�}B���=���Zs�0�x�]p�F'�Yۃ�� �
�ib/$_�82m��A��/�8̶b���3�O�v���ǯ����ۨ��%%�������xt2ͬנ�Dې�#�+l������Z�A�؇��8*.ݼ��K�g�V�[G�.I4ws~� s�ÇW1�}nt��I�N8���߱"�k�x�U+�ƛJ��\k��0��	� =\HAPU�J����t�;�Q� �G�[��I��{}M�+�5��Ԍ'�EQ�Y�D��Ҏ���_zw�E"/G��D
]����h0~O�X5|�X�����hB��3s�uF��(Be��~��]�+���\���@í0:>=d��@f��� ��*�ʼ%�N�`��bo�r|{Ŭ4��=�v����4��䓈<���*�c	xĻ��e6�q�mle��&�X��XJәq�x,�6~L<`V^i\�\؏)�ag�&H�Z_�3-�����6̊7��5��96쒉)Lʅ�v�:�d�!C��?z/ղ�_�xbg��`��u��͟�er��_�h�ɞƝ�D�A��:+�5���rC(	�b���r�BJ%�L�I?%���3�^,��C��2���8� ����a�E�5����e�l�WS�l~�T/�����0.��-{<y��GfO�}�Wq��F� �a�
�e��)�'Tp^q)�8,{vu5*>�h�ʍ�p.r#r�WoÅs�ېC�&$���M��a���!��.��xܫ����2ɈE�a\$!=�*�ن��j���'�}��VſQ@����.o��N��>��F\�������_ޖ+/H���=1i��M��v�eKR��@I[�G��|�^��в�7��$.�֞t���+���H��$�L�P�*��q5]��g�˲Im!sG�T�<#Z��2H�����E��I}け�2�0y!��І_����N6A�u�9ʲ�Ez5s����FZ'���`<m���f��x�s��n�^��!���t+[�;x[���ClZ�g)�2@F��凃���R΁��3,�.|ڧ��-i�ѫq��X��ϳ3x�O ��HI:�!����L|�څȝ�ݭw���$�SL�펈e��(�0k>?{���PH��	����";�~��gS��x8M�lh�޿�8���;�a����8����g��#c���3�:��2\�*�qN���M텐PN!X���X`@�ڳ�Z.���#��a�Rx���}K7�0ɶA����Jڪ��o,��XJWN���^�կ�Iq���ѹ�"��NL�t�IuzM=|9+f'x��u_����%�|RdP�ԩ��e_��b��
n�4���E����A_$�
�_ ����]&�q�sY��^�lJ�NwT���H��^T?�iG��,� y����C�G��4
���q� O�=�Nx��vZT���{n�0��..��z�}|a�7���`O~kr[z�+�WZ���;�3���0�H�Q�2Sq�
�A}��sb�5i�]���Q{��a��{��k�o��M],�1;��i��D�I���}��u��^A��腱�E�	��Dw��Χ���dhEQw>���Nʟy��,
��Y�c����9�.ZN��Q�7�_��k�.�S:��F��΋��D(1���LMA٣ p�����K-�P��\��+�"�Vy)�w�1)�d78L�/�.d�S�����U�)�U,8��7	J~n������{�)�����>/e�-L�M��>�����?��A&�#M���0���⪸�; C��%��]�C��nB&Ϳ\��9x�C���<�C0V��c�_^���K@�����R�S-w6����+�>�˜V{����|d���$�Y��o�ZI���.�g���i��x��8/BS֋��ZvVDX9�e!V�CZ-�,�Ln)�d�������5U�Qx�q���'';� 5��\O˹�������\��1�Qf������\��Ī���ӏa�����!NP�:YpCY.�_ ���P,��q�wҩPw�	f__ڄ�9��M)68�<q�ǯ��vae���G�+e]p84a�|^�C���ZQ��l��"e�����`���е��-`�e@���(~�ŷp����C���I�GnՁ��⃠j'q�]m&�+�D�]�fm��i�v���bX�=��aF�����=},A�ubi9��ICǺ*��"���[�}m����}\�4TC���H��˭�(C\	$چ�'Ԗu,n�;�9א1K�lP-�h"�s�Eδ#\��,�;aER;�g��F��!�G�����:��O(Q�ù�����y�"���ee�/`	��F%� ��>����ad��
���j�x�1��	6���wF'+L�$���w��-õ~�Q#���Y; O�����8/+��`�/y*��4�8'JcZ���iJ����f&U����Ȋ�c��;5�.���Y�a>]�S_h��Sw���U|��z�r���&h*n��&����+>��;{1��8W�t6�O"��"�?s�y]�}��Ю�mF^}�W�7���K����/�r�A��3��*��u#�������P#����M_Z��w��ڛ50�q��pn���4n�Q�}�M����8�G�"ߦ�QrF�s�嶴�Βw@�8�������[�Ӓ��|%u1�.�I��/<
+x�4�d�)�t�G6���*�_v^Wq����H鎥�������2�/k΅��[ɭ��'X@�g���O�r��'�_�]����߇�vG��d��a�}X�Z��·e*]FB�d�-��߉�C�a
'�B�en��(�M���zya�?�M7���bk�T�C�� @���-�f莌e
p�MJ^m�_s�
�K��DM�
�k�N]�
�k�~�Sa� Gv�^#�.�} -r]ng�6�k ᧟��ӆ��1u�<�KM?BL�w%F쿼�\����a�o�|��>�Z@l_A#�����2H?|
���|���r�5$��~"���>�ߢ-Ґ�`7xꠡ�l�c��f����
;�Xԇh�w�(���u�;Ϩu��>x��
�{���k��;�]��k/#��$�2��9��|3r�#5<\�"^�`��c��댗�}T������Y�he4^��y���do.W��/&���6딃ʖ�3D2p5Ϛl���N�8<[�$��;.�:��'t(�?g@��y�>���5��:~u�8#�b��,���C��R��WD����j�r�*�XC��(S[)�fΫ"�b|?#�<J��|��]ɘ�Ź�iK�PC���3�r�Zr���Sް��-�aJ;�q *L
yY�.��O����[�gE�h6�ǃ�����7��mDŐ?�o_k��Q�h�x�/iF�>��-$�d�ǐv蜞_�]����sw|���Q���$�~��u�h��}@
�=\�Q}e�x�w���>G��X�?�!Tb��V�l�I�qM�SG���-(	���0�V �+R��'<�I 7%S ��j����r\��Gٟ��3X��AvVǗ���SR�a�	Q� Ccḧ�����0F��6������s���vRx�r9�gN�'FH8&R���i
�>x��j}�8�=�|���k���0�q�C�C��Xp�?
�tF�~����jH�A���Sf��0���� �ɐ��c�A
��][�c��.z�X�N��2-�99c���-�)�\^j��1b�)~ � ���	�c�_N�|���S�&R�E��3%T/�{8��ˢi>��@u���
(��r9�z��v�����8�rM&�v`
6�w�&ʳ��W�%G�.p+m�݀#�n��ƍ:�KF�.7��W�f����U:�:�0e��W��AO3�<��䜗a`�r�vy�9Z����x<+���]x�ǄC�$C����kaJPA93&���ؼn¸��uq�<����"=�>��q�BΝ���N��c���ޑvbt��hi':GZ9�Xݨ�9��-M �zcs+G����9"b��Ђ3e-�}7���5s\���'Jܽ2��2j� C�s��m<Q������|�?����~��A�l0
��� h`HWH$���5;�J��o�QkR�N!Btn^�@��]�D�> � 	cQCתJW�	��_`겷x&��'svC;ˆ]��y<�t%Q+���93u��<�G�j9�	'�Umž�3�R�� ����sMx�����[b��؄1>�7�*=FY��b�{��f�eV��U�\ՒG�#ۇ���B5O3����yv��k�}�S̿��������G�.P�!�_G4��U�˞d���?'?i8"�"'�&�]�kk���!�����c��Y�y�6��J3|-?�L�.�mu�a]�V�I��k�A
!�ՙi&���Y��@��/�Y�e>?�W4T�ʴ7��NIP��1|��H%���S۔W�*�ڑ��)oȪ���	�(�YU�!̺���?�uT\<�4�u0Z�n���o>b�[G���P�Jxji��9`O��j��V���a��Ȯ�8m�7��:ZlDᙃp�ɩN�.&�oN�=n�4@���F�N�%Qa<o����|�GW��4�d6�% �al��=� UE�iʹ��%'aE�g.��:�K�s�t�ȁ��!K�m� p��@.��7� 5�|R�EHH� ����۲o8�UUɻ�i
}�B�P3+֚"U��"T�l�qh,D�K��َ���Q�QK�m�L���;��h �MJo}[�������`�i.k�S�i�,ހ�y�͎�B+�T���a��j5{�N�wh�r'�'T�F�{�֡�];�E'��Na`a���a����[u�[G��NFI��먕�6���29Ȣ�y֙F3����AC�	�V;SCjպ '�
 5d����I�B�|�u����J�j�8�$[��L�`��7/4|ˎ����%�@�����^}���.~½�=Z߇�EL�/6o��Ӵ��o�Qt�Zpj� �Rp�t �D�}� !�z��n��U�C'� +?I���'*S���8��XC��.�_���F���P͗�v�)1ݦ!qH_��9F���wA�Z�̥�~�����>)�m���"8i��(r���ײ��p��4#�f�u�������]��`|f���ȞMR�2��<P��}�3}f�
5F6�Tt�0�-kfaԎ��Tp�q��5�3���T��9R���Oh\���W%7�.s�x��#��4n�����ή�c�L�p�в� D?�C'��O��N*'�E6s�q���@/|K��=0G,��,��C �1N���F��(^�_K�0v=g�z14(��{��y�e��jh���X\W�����6�Yw��un�!�8���K�Vx�3�^�ٍOX�%
7D=����|G4](���S�~�	�F���>�|��\e1����D��vD�i��5�FW!�$�(�H��T���}��X.y1���=�X��墦��ʦS� K�~��\;��6�ݩO�6]��q����Ym���Ť���)�?$9�;�]��~�����(;�LZcg,DJpt=H�~�D����X�3�s>��1�����i?�cJ���+��V�|�M���{ ���q�H���X�'ǭ.w�������b��zv�0��1ȃ��M���u��d���-ڭ1����Կ�E�Ƌ�Z,�h��ӆ�?C��-"i���J!(B��(�rc{Y�4������t�twp�~��L�C.�h��]�4�TVD`W��@��F
_����o�J�;`qcJ
L��k�5q��>Js����E���R�fN(�����4��Q3%2��Ӗ#�0!z��Z�2oO�(�O����4�����_�"|��/���3��CU�N�P"�;���p`����ʘ��Y�\P	�X@�^j �~�T����8��h�8�Q��A��t��O�3��H4�n`��I�W���*&���O���3���:hD�d��R�1иU�P��n��&��CNAI>�$�����J�h�%�ٰpG��I�7��H�y�E���i������>�Gݟ��Nњ��<��F���h�&{��QXh�iGsb����[� �^�G��U3�iia�_�t�B]p���:�^M�S~b�	�`~�tK�8%־���3���g3sS�+m��ׁ����ߟaiY��#(����^���C���'�Sj-�� ��n���}h�Dӵ����6� ��Z��l�fe�2����`�&j��E������c'�����ٔ���[8�A�����6���������1�-)r9a5c:�iVq�<1���2����
0�6�A$�V�-ڡiL�3�w���j���>��C�&],�!'�O��qO>&A�U���CY$����I��f�W�������Up�(��r	LG���U��9�Y�d|��	�a��Sb�~���+�Q��F�A�~��b@d(�9��ᒘ��)��`���EmQeץ�ˀ�~����;�-�"j���`+��t����3� ;���}W�ژ2&��K''Q���*pE�BKj��H	�#��ᓱyJ�T�E��q��`i3� 1�O���_'�ln쮨7Ee�����!{�*��$Fr����@�j
�GѨW���\������cǈ���TC����؈c��} _���l�	.����z,).ƅw��l\��������>��!���~�����	��!!}� ��38�~q$���3��kt�T�[c�&����&�]5e3k����Sh2t{�|d٦��0r̸��9��6fU3�ZxejB�#>-��HF.Cx���,�%��T�("81y��!����(��pʔNÎ{��|u@H�=}���6"h�t?�	���B]ٛ�N�p�B�ZĿA� *R�w@��(��FR�E����Ɠ�hSE,nE��X0�+�`u�c��%>-�~+�km���Մf����N�>�z��=�l꣇|O$�D�X$��� �ߪ��gr��ȩ�*!pɪ=�/��>�܉��pd���Ƴ�f��%7z�]m���!���'�+��.5�Z�L�O�&�����G�Y<k��w�\׍��%ػ��i��1xJ���(�P7�.I֚t�q�_u��&���"{s}v��qN�XxfĲ�W���4����[�Tm�3<�,��o���$pj��{a��Bt�]W��d�A��x��&ن���N��,��?pޓ�?̮E��^����@M�7�f:B�(�*k����Cc��#%�vN8��`U� ������c팙�ꠦL���1����+{&sҸZe�bԸ�FɮÛ�\���E��K�1�m0$�B��[����θ���(H4嶒x�'� ����\щ�w��y�2�C�0��i��႑۹���fl����������u��K�a�q>�O�SDp˰�+����U��p�X`�=���q�B�猝�W�ļ+��Y��+��7��\)l�6�O�����*�%>-/�9r�`YẢMzgg�o�r޼ �v剞�`
ѱ�`K��syЈ����X`x=T�S��Q�#���\���/b�� ����1ћ-)�騙��NcB�z�\*���%Y�	T?0��M�~�1Ρ��9�D(�'7��ޫv 6�;�Z��l�}A�*�}�o7Q��D�-|��Q箺��zs�{]���\��$�	�0��ĥ�8�j�^��xMG��#�ڍ�k4�i�@�ϟ� o�F@����-p1�3�rV7�OK�"�~�9�S.� O��n��n?n:�q��x�fC�ۍf�x4�6Y���e��b���I�i	GV��i�eS~L^Q�M��'Y���l9g��|�,E��O����nS��M�M�ǥ� 7�w��^���&o�׍ē��O�f�H^P�Z��ƍ�A��0⥎���}0aF2�Xep7�VX)��7�7p�S�Ɠ�f1m��:H?�95W�W�>M���0@t�gr�|o֦;�5�/ȎI*�*d��t����h6^��F�qSmM���w��l1���|6�Кd��4��;t���.��������v������V��S��H���x�����B�Z]��Ѯ{q�t��ڮ�u��\�V�Sf��\P���a����7cݺ�eҪ��m�M�!+�rs	uN\nB�e*\q�Rqx�;�δL>�25�m�j�	�ӿ�|i��R�BB�јG�>X�WM�Tv�I<J���V�倁|h�~vC{��z��J2����ɧ#�"�3,�Z���p0s�� �"��r��?���e�q��'�>���b�U��tM=%4�V�7�(9�JA%����׺��R��k0��U�:�l9��M�a��t��)��]6��HJ�{��i/ʅ)t�)ٸj�=(z��_zA�}-��h�p����m�=y�1ٺܚS��ܢ.�rY؈;N��-��ו�'O1]&�pz� �K��j�ZyR�O��Ɓ81͔+yc`:���k!&V[�a�7c�+$!��藖L��H��������T�-�*VZT./NRŢYL�!3NJs8��&�Z�u5nn�q�R~����Ƴ��i�z�<�u�sW�r�5u�y�\�ʪAJv2}=�ʡT�`�ޠ~����B���g<§8O)��T	��l�MV�X�F�YX�S9+d�6�])9�5�X��,�J"���n��ύ���97n�8g�ו4'�$�gZZ%ʽЖ���RN�M%�))�|䴒a�T� q+R��r'�B.���1Ab!�a���L��I�v�Bju3�!���X���sV�d�C�D�6!I,�00�r�n0̍�x> �{=��x@����f�������u�$"���7$%���!��dY�IG��lHB�@C��pt�$�O�ސ��O�+�0��\2mH��
�6$\�ǲmH�9��ېp�|�� �C��C���A� �nH��Uɼ!�Ԋ�дeR-fC�I�XM\&S�34q�"��W5K��`��K�̓�}������Twe�9�,�A�ϝ��d�b`�Zd0��6	aoh�2	��Ф��R�`O��p@�&���L�����e��TX/��,��i8uY8 ��*Y8 \�&�p@�Ld�pp׆�(���5�V���Ġ݁	km��������	�@�T8\U����'Wh��,�Aã�p@�FY8�dk�d5 \�������nZ4���5�"�k`�D������ta�LX���o`ʺ7�6PY8 `�����uoI�����dtXM\&��M\vIeက�g$��`0�F�p�~"��������*Y8 \�l �p@�]g14]���Єe�l���,������ ?T�H\}_8 `�����������u���+�+ZH.���L:&��l�D� ��,�a�Y8 \�&�p@�����Lvwh�2�zS�o�D6��oվ36Y�g�oE���� 4��q&��&�70q}ԟ���������dذ��'退b���ذ��/退+�C6�����N]�[4oh�MV�����̴ӡ	,��:��o.����pgN�B6��z�S�^���vg��[B[6�Ʃh�������5����&�6p���ĵ5]m�CS��tj�&���rk<8}�\j����l;��L����)�p�`�q@�&�Vop
�[@4����LlT4ؠ�P�8 `������*l�B�h�AYN\��=wp��\-�\}�8$�j�!�L'����3�WCSخڜ:$�j{ꐐ�gt�?؍�Τ�� �� �Xh�p�+}m�lPXp�p���E�81(����kob���Ե�95��YD��t^�M`{�yu68��Ϋ��)����(�� �T�l6�����WTlP�hpex ��2V@����T4�2*cH���7CB69�zC�Wj:)����̘�jh�L����),�95�pY�D【M�Uop
�+_ �]pf�n86�]T4ؠuQ�8 `������hp�hpu�␐�]S��l�^3�8 d�)w:8��L���),��B��B�1��衊G=PS�DL[C���pe�Ɛ�+#6�l�PD4	�2fcH�&�N]&�N^է�!!W�5�P}�8$���!!���:CS���Q�8$dcp{���Y��lP��h��,@E　�
]
���Š���bp⪎��'�)�����#���j�ꐐM����)�&Ed1�ǡ�J�ϴ�EC�5� ���}�
ow�'��R�W+�q�(���;�� G:�������
��Q���j%늸�
�Ѳh��#`�$v�ᛲ��eYE�� #����d̧�P��*a{��g���eE�&�+]7�X[��4���r7xH�ͼ�,��l9fJoo�<�v(*+ cj����x�O�CR�q��X����"W�w�GJ�\ϱg��z1�@Y��ʎ�$XCuط�6{���f1�*�P�����K��u;�R�ʱC���S�
�Q���Jd0"ej���Z��tTr��?��ٟ�G��Oi�MF�4�F��/�:D;>�턬��W3�s>��cR&��~Q�WH�~�P͸�?SAy�2�����8}���q�U�e��\W)�04	R�/ �/�\[���=(���`��i]*����⣤{�1U�U��˪�b�s�V-�7t�R������6�?�,�,��7���Iț�5�V�q�vɧ@�@���h16�>�^(�]?#1���D_^ub,�:іW��ʫN�O��.��������Q`%����V ��`��^o��#Ve|���>6f.�}�3����ޣ*+�>'�nga`M�mI�A����Έd\��݉��O٤�h<��Px/�@(�f�uW�������K:
c[	��:�ڬ�$B�2��"	���
�gv��;w�Z����so�]�c��Z�:��8��h�O�<�����A�c/�:/�0~0�A:�����_����`;זC��7�A�]�y��7��¿��}�;7��1"�W�����	7�[-��� ����(C�_;"Z��oJI��3a�G�����OY�0-��
�B�`�mğ6Q�C��]�;��9|�l�+߃���]��O�����Ja5'�&h�ɬIK�4hj^rC�g;H��S�������[JtrgɣƔ�7]�NtNڧ���a�������!�_���mpf$�2��(������
V�<�]83�w���.��>r�:�O�=��pu��<�ÀN��E�����|�}@�#���Vp#>!�,�p}C�m6�����m����qsJ���$�v��h�1LR��8.��r����%�"%��TK;B�1�?��nį04�!�=�A���[A�A�V)�n�-����#GJ��:z��*����)�[ĝ~����!0�iDg~�ϱT2̑���k�*s���ٿ����Z��f�tcgh롉��f.�S�<b����,A�H���� @���D8EK�H�3x�w�����'b��)��E�`��y�F�!��z(��v�Cδd��95�S�<9-Tג	�c���yّ�wl�i���?��:&��Uu{>.�����?��W�q}�j�=t}r���`L�n[��q��>��}��(�_��bzg��^�u*�2�����|�s������� ������Y$"Eߤ�dޫ`0 ��Ì%D�����>�o�&��@�Ԁ.������U��vQ�T f
���HX$O���9#m�5���c��������8�tܜwv轗^�$,Ѣ���D�
e�m�	�lM��}D���
HL�gс%�o��wE3]x��"�Q�ǜC4B'���B��ժ�#wL*AM�c\e������rX��ېv�d]��4b�jt�ܧ0γ�vF;���칰�1��#�xB��t���w9A_�v!Z&���g�hh�F��i�[�4\�s���7������a���鷄c���R��}�|��&����C�),t�
k����۩?�]J���k���;�&�ùY�<M>i��Bt�)���E��p~A�,.��Vnv)��G���V�$]��-���#
�3Y���ܢ��`ΧS���ܳ)x2�b�����*A�G���Z���B�C�]˗���c�re�G&�Ɯ���<ˬ�']J2�ܪR�6A��ć�4��5�ܱ\LDѹ�ݛ�{c��k�<�Y�h:���D븳�j	&Z��g/�nh7�ba���k�s_=Wk��v�c����g�?
􏖱��3.����87�%�-�Ko�x�ǳ��qaH�/:$�M1$�����s���}��n�v�9O������E��DiĄQ3��*��������*�3�y�u�p����V�َ����@+l0	7��a���M���(o��>n�~��ӊ�(��r�EO�eL#\� �;���δ[H&�g;ܢ���bLfD�Vz��6�xR��)��i�z�X��E��d	�Z9p5^�����S��+_7܃#����7i�e}\���k�O#֯o�}�u�Q���E펢0���LW]��h3k�P�C�]u�~]���V�6s3ຨYgH`q�.��a����0g��~!�a��J'�ԡPc�Ґܒ��Zc�Tş���]PC�������Kv(�c+ĥ�����l�W�XI��n6�		�v��f'�%�=��0uo��s��!�������Z׈ZN�)"2a(�o:.԰�Ma��c@^���_�A�9��y�
G.��}Q�{ݙ%�@b!�*�k⛐���s�Qp�7�S��h�"�L�i��H�E{��h�yѣ>�QG���-`�~XN�b0� %d�锡:u�=>�J
���;0�s�-C
�|�/l"��V|`�L��uz�L��$-�:��e
.����W��B��g�1~&8�*OglF\�T�?����	���l=�i����q<�O�b7Y���`�*vU�7U�HD���4S�$������'0ϲ�ˑ/֚U�S�9�#�>ok*���],,$��B�]I���k�Z��ȸ��ֺF7�g�1\��n�<��bV2��[%���RtW��mͶ.넻3c���3�ԛ.|���p��qe ��J�������Ҹ����ƞ���W�SI�8TS�c�[�Ƶ����3GS�>�>��Q�c%�f'Y���4��$��)�u�A���+�ܥE��'Y��9[�����v���#6P�G[ޔO��۝N�����)@|�����n�}�9
��U�n#n�dŌz��CZLI�)pZ'�L����C;u��'�'O+<�b��0�����y.I��05���V�)T�jJeuGH�n�:�r�\����@c{B�4qGc��y%Z���[���2q�a�O���xTh�z�ٙ�DW��p����^�.�g��q�;�ZO��M=�籬�J-'����|lcG�ߜ�4}�$��2s5Zyo�쑺(c�z{-�e�ңU���H��&�����_�G����w���m3kk����# �+[�͡�\ڀI�1��8��d@I߀A���{i��{�R��p��J(^6�U3�H�|yĲ$Э���)0tq�¾�'R���)�]Pu��'{���|�������Y��I�Ʋ�ʩAt�4b�y6�g�XD�K��/j�11�R�6��=D]	�+�� (u��K}�*�:I-��O��F�|�IU��JM@e�9'-+g~'Y(1X}������C�>@�y�Si�$֣H���U���g\r@^F����k��� ��M�%ZgP@+ɓ��xy�f?FY�� ���,B�=�p�2[�+]{�Z��4����cزRe��-pA��6����gN����Sr����=M�SĢ�E�_�4A���p}]?�QWw�Ш7O#�Ie3fH�Ch <��ݍ�=c���p�zDSF~�n���X>V��(�.`���}T��b�(p�{n���5�o��c�	�(�s�
��p�_��⹵1u��rf8q��q�ɧ�[���A��k��ݤ���/0fJ���=hR�%���9���1���&�
G��$��8�/oǡ1V�Gm ��b�[��FPy�5vN�A��K�Dl,��]�#Hz'9�"���"��a+@�׊���}�?K��$[՗��p��F��(^���aV�^X���m4�U���F-kcᦄi�C�$z��aw,�B�����Т�Ի�ѷ�K�Er^�/+�u�^����{�wA�\
V2M���Q;q�H��fJd͗4:C��Qk,����e�����>�D�A��!��,�+i@u#)1s�R�}Qc����jff��:�16��]�>k3
��+��+��.� ��k���U��#�!F�ӿ_��n�8�&�b���pד�Oo��"g��/n�o����Q~�C&��x�H	B�Hx/;,ɻ�*
�&�Ǖ�[rB�כ#����ܩ34�^�d�����==���1p�Ho��&I�"�)��\����4��.�2�t�%!/+ȌMd���J*�f������&H^]�"$]��b�X����AZ�O�Ԉu�_A߱&�^N�S�z�j�.���p�_�"���@=��p��������B��g��4d�5@gD/���L�g�Hs��� {ʄIv}IKW��K�k=v��<�Ӊ����]��!��?4�/�R.�K��Ό�KA���+{:�{S�܀�H\"OxY�.��<��$�yu)��&|Kɹ~)��\N��(~AS�?o���\6���m���P��AX�λm^2ӹd��i�}��Rhk ���3��_�p�.٠���ax���/���_�T���]�Ⱦ
;�����gb_v��a�E���%�0�ޖ��ޡ��O�C�i:g���n�i����O'���B���H�Z#�k���5ҿ�H�Z#�ej�o��*>,?���!����m���C!5� ����T��>ڸ�w��0c�~���Gp���T�'��J�X=`������0ę��W�ճ��Ǭ~�*~��aV[6�� ?�=ф�ͺ*~�?�Դ��,����<k� As�!C��aOѤ	���=*���`E��~	�i�$��gTb�U�R��^+�sY��)� �㔠:��n֐
��<I�(��Zj��OdjJ;���1B�kR��l>-�o�W�D�<>��v�u�lو�V���ay��07g6v��BOD�>�Yk��i���`��������4q>C���/sjr�%��l|pv��*�<�̨>ɀ_�SL�L_�'���i�iA��N����ۑ�~^��O�j�گ��ȩ4���ڡ�G�W����{o���`��;��,���\JNW�*U]X��k��`�bR>Y�V�+S�aN��EL銥���=$���(Rs�H!^P;���=z" �-J�����Xw� �އ�����5d��]��y.�r3��(��\����e�4q�a�\zd9�g^�5���^�.׭O�;nͥ���v�!�Tٓ��ecT`e��Nu���ŝ$���.����4!��Pi���J��?@��qY)4bE�d`���I,X��s�R�uC"m�Eej�����qS�MP/�f� ��5ܔ���h�m'u�U!U�:�&i.�5�=���b!oNc^�R�մ�Js,H�q9_y���.�����#T�`�a�Ӻ�0M��r�4\W\��9�Ci��8ܪ��.`��r]���+o�"w�ju��� {���|d�$I�g�k�n;�]�;�z�B�1.�C���v�D��	ht���- ����r4�����-
��! ��fQͫÒ,�t��5X��;�_B���BeQ=P�.���s�b"s�f��z�P���R+�k�N:�{k�����ؘ��!�߂.L��?ژ���"���^0��er�V�^�):�?}@�
�� �͎v�C@F|K��L+7��6�m!�b�@�/I�ڜ���_�m��a��ὣ�Uh��8U;�ggsIa.�]�0ߙ!�-|����m�N��CD��L����ݦp�9�����_\���ɝ�|� ��/|U��hִ�WW���Z_]���j}u����2�Zq ��D�����]�:��z�t��N�D��Լ�l�N3����V��n~8'j��K��ǟs�2�]��0,���ڟ爇2� 6�LH	ʋp��	���Z�Y�t�(>��H~��Xv?�u�
b�s^�)緦�u;�	F���4�1\i��\�2����!E�+���I0x��)��i� b�9ɀ��͞'�0�n�A��m�d -�k'@jcq�!�eV�ZJ}	_9[�M�suӞ�,����(Q��V#�1*wW\Pj�����i������_��<�T����VБǳ�2�7î%�h��bb��~���zƷ�d[s=�ג�;�
2� �iG4.��٭�T�i��� x���"��!��{��:��.�'`sXGv�j?w��q��t
�RA>.H��b�� �*�6��'}R��c�,H�UB9sZ�A[��V�*��5�\c!Y���L�K�.lt��b���-��<��-d�AB6�ϰN�j;3x�I������|vwU�6�0��In���Y����ϑ�z	Y�ҥ�M�C0����K�����xݝ(�ԒW|.z ��6ӄE	������h���@�􂫺Ar��G�7NLL_/Y�^�|�d�z������%�]����fs���Y���X�]C1�.f�tL|��H/,9�}�؋ +հ��z��Ua]�z��Di��J�L�.�y[vƢ��3i��r���[J�HF(�O�/M=T�=�.1�
�z�t��}U{5�����(��t5�B��x�ӼY���	����!��d�O�-��O�A��l=$�heg2]p����YB�,%f��Q.�l������=�M&�( �p��:�?�͸ç�
'r+ǥ����>�!�	�ױ˴�^��;��$�C֨{��Q�Zl�Eb>���7��#��Fh�t+�J�e��9�;��[���;�^3�4܆�e��2���-��h)X���ڌ1P ?I��ܐ��FXQ��*����&�r P�/�-`��R���9����P+�cb�x�?lI<���{���9��캘��^�$8l?A�#�--�Ty�TM�8�F|Xh�BI(n�ts�OO�ɘM�>:u�s^�JR2�fJj�b�$n�Ŝ.7TӶd�٘���E���#j��ˣ���(DÞ�t9�
V�r�D(�5ȁ2u��e��3�A���4\�|�4S}�V�ȱhm����+Q�q���=�c�T�]��%&R��Td�����h��Ydn�ڞڭ��?�p�j^���:�N���yR��t���F'h�M����3`(T�`n���g��cw���ѮTt
ͲM�����|��H=e���#~�Q���k����~�~��0w��:T����b �ٽ��)����v�lX�Q�<�ݜ�����s6*4������+m�pd�R/�k�N�ڽ�w�
?�t�g�t]���H@!�07��_�A��i4�^t�1feF����8�Ρ��a�ؐz�t��@s�N�۸�[�'r)�a��}�b�a�2���;OC)��Z��#�DN��3�g�WϜ��9_=s�z漌gN푂rO�cm
!�m�V�(e�̩TT��w�ӱ�!C���/jਥ/��Mz�l���=�eG�nؒ
bt뤲0F�.5^aL�c�xz&�f��~�Y�k�Mr4���l�~̾���7��o-��WB@�-��J��:-/~����'OW�:��P�c\:�����`���m��2���"�L�$۝� �Gw�+�����ɯ�I>���\�f��>b�r�T τ۶���N	�m=���LS������C���;��G�#^�Jγ��1��=��o��	�i:���!% ���X���,�V?ϔ���n}�&-5Gưe*�!���F�Q�ȍm�%O Y��ȶ����o4KxBw���{���I�I�'j�@��rn<Og���[�g�f�t�7IZ�8vxW�22��
)��-:�?��3���!�&�y2�"ɢK�"؝������v;	=�Ia岉���j���B+���~�?��'-�^��:l���z�7�Xw�0�d|�I�!,��T��F�U(�B�o�ϼox��B��g��H���v|w=AƗ��M^�0��]�@��^'����v�E.��s���w�|�}0:����w�sY��&;]g'3.��`��Rè:�����>��Y�_�`�%�Lپ ����^h���'���hz�ɧ�h��9D��=�Zpn�q,��\��p�e�m��Tj,�A��"��o��a�2��S�>���=�����۔�+�W:��WA�}r�9�(����x*����.b�)��YV|f�y�w��o�D+�+N8��w�ӽ����1F<�k+��N���Ϯطn.�DʖQ���
k���,��R:���AR�%U�,l��^Ǐ�,����ܲmw��t�G�P�F �������u'�a
��2a`�����, '�cO��r}��89}W�8���
��p�e>\�Ykqy_LGw#��O��q�;��lPÜBNV}��o�lu�ҡ\���G��}z��s�������2f�������k�#��D����0ڠ����M1?��~Q;X���:�n��"��R!�p�8Q�>�EJ�8��Fc�G|QR�_�~&��O�15��'5��"�ȴt�^BB�S�u_�a��F������-"�U����K��3����~����,�Ͽv�C�J��Ͱ�?���q�z���2�ް�L�0��0Yk+�7�ֺƦ�&�?���>��)e8�qF��{69~x��vQ�z���[I!���p�?�=�G��c+wKhR�j8��c+ǕVJ���#�v����!ga�as�~�N[���0P�lz����ks����]5^ݲz�T�R�(����S�5����Mx�t�Čˤ�n&�S�5[�ܾ���
b+�!���%�s�m�{�S��"��>���{i�\Z	�( �Û�Ix��Xկ��_cU�ƪ~�U���2����:L>�P�6!���"�����z��o��n��6�"���&� ���<%B�r��sM���mԉ%��_��0&�� -/�s6�$<oܮ�����n�m[�k4e��:5��ȁ#)�¤�X��� N~y�13u��e~E{�=%�I���#m��毡\����>��;���ן�|�)�מ�|�)�Ƿƌ*|.+ιt�d��k�	����W�/��F�v���k��^6�luG�)��|�{��(�\���D���	������9v�S��$��F��;�mHV�し�/��'����&*�1��Ԁ�w&�4GQ[��Έ�ԩ���Ma��g�y{<&Ĺ�@��+c�RgFm���2��{���Sv���2��KD���1�d���l/�F�u�� w���Ժ�� +�6���p2�S���?m����Z�����&��ўi�0�au�"(o��}�x�̈����Dts�)��J�ur�<f��9��Q
_�I�+^\���N��x/k��UJ˘e����Z2���W �XW��A�~2��W��X�L�C���"�'pX�F�}��-,���v{�j��h�����0�F^C�c��:z�m�%|X��\�s9E��H� �h�����!� ճ�GrƽAk^'�8��<�o�C�����\L�E��3�h�����3���w4У�'ޣs78���n�����E) T�g��v�.��:��C�ƞ�ݧ^�Fv]�(G�����˚)$;-ݭ��|�^��8}��Iz�e�B/[�l�����$6$�>���ϐt�Q������7��f��$^�$��Ŷ��ޘo��d��p��E�-޵�_��/���F��e���45�ti��r	bf�'|��CIu0mH����sGfC�U��VQ�&|���-i ���6m���x��GM㥆��������(Y8��O>�W,�6ϐ>{�˜�br�[b��������p�p:�����E�/�"i�3�/e���"k7̤����WE�Q,�`:��枯Wf�s# ���9l�,ze�ϳ�bϳ#UA=u�E��Bz:Q������PB�?<�1�C�$u�I�ф׼Ϥlt�Z�Z����hR���.��; :�㟣9�9�я��s}ɿ���+��|oEW��BL�2�U�d�L}�v���{�"6��R����@�`��P���@�����C�W<a���!�k�I��i���G���s)��MO����N8��p��m����7��7-t�i,Ѩ��'_P�/�̸m�9����������r��^4f��T.�-� x΍��+-�����£q�Iy�w��t�:{6�3�+�8��e,#�mZDĶ؍��VT����b�P���u�X�p�,)OWl�g�|nl����M���v�a�~z�tv��0�I� �p���Ū��U�OS��ŭl���-�i[w]�B��qZ3s�(v:wH��26�<�]ʬV+}{���lC!Kv%�M��1����˄^�b[cü_I��-1�u�Ô�����0�p"ߖT�_�h������Uo�X�Q����wl�t=cn.R����$�4��                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               75083402ef6053b0f2f20\8d5f293465684920da2a9f9896507528031be0ed="ZDU5OOW/pO8yJeZwuh4w+H378tszNTg4XcC72dA4jSyILaFFAply9mU1OTg3MzkwMzAnBzZkMDkyOUUwMjWZ6LN4O7cr4OGUBqwOEJzk4BiaEW3JbtRpt9Zw1bQiTjA4MjlrsDI1ODo5ORJiNWHQA1gezbHegVEbkxflFIs+J5CPA93VDVsebTYl5uTjcig4OTlPiXE+CMeokGpvGwsaJkTReTg1M7Oja2IjfWDd5ZokuReVlo0W3oKZVFRlIYAtg2oSwQhdGifSgS3CM1cpNs6+fg+iCi8ba4I+WaXRpwDsH2DSQ+I="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\30683721429016e04f928979196e7ed68ebec927="NTQ0NOS4qes1IeUluE409Hv2o981NDQ0UM29j9Ftj3yOf6EXB5si9jQ0NDQ2NDQ0NDQkUjQ0NDU0NBQ0NDTycGoUxI6LM0UgNonjtwIh/4IHKolo1jKt9vScJwRo8DQ0NDQ6tDQ0NDY0NBQ0NDST02rU5nvWRjx2RH4flf9xzpu9OhjEtR32F7ZUgYG+PiQ0NDSkz4rjHgT/dsWKmyF2OtIFdDQ0NP6FuFeDZQQow975N9tenfIG+Uk+v6xh6Ib7n+X7ik5R8P4I4KMTAHTtqeYZevPVAfaAiMwmjxqdGMGEZNmC9SA="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\5a9a553ff777b5a94cb6ab4a1240ab9f72e2e1af="MjI1YenuqOlgdrQo6U8393b19o01NTFhUJjo3Ydoj3rbefdFBM1w8jMyNWE7YjU2YWN1X2U1NzY5N0FmNDVTknqupSx01xILjdLxLhQEBad1Z1eUKZfdyJp6XfPeUzc3OTdv5jQ1MWM0YUFmYjHWvOV4JkeCN87lFbKqvK01XHIqI2CpKhSXQT3cjWMZRCFhNGH1Mi9Yhm6C9eZE+pQRJ5ehdWE5Ym3Bg/pJMB7Hd/jO17gaqF5GYXrnxG1dyNAJnDIsI+s2QqHLxNjqNo8yoaEyx7+efvgJIWrNtCNQsgaOmv18owc="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\5819e0ff462f24a09e2de1c68b110d6ca3a30e35="OTk0NbG0pOs0dOkouE9h+Hb2ooo5OTQ1BcGwj9A4g3GOfvQbCpsjozg5NDVjODk0NWEoXzQ1YTk5NBVhODlCMIOx39/twHMRPFJSJXdCEs1vshUGUWRSFZDCSL5Nk2E4OTQ74Tg5NDdhOBk0NWE7bEtSxqbFyC4bIKguQDjgKHjGv1oJVKNtGHCTqFkYByQ1YTh7X5UMoL5j5mEQ/UoIKmnbdDVhODgSSCEUWHlJ5hx+PsWOToZC99v1HVTdLzQ7RrZ7WcYwotMMdprb5dgstT01PFSmZbhl+St79lSYqOO5HQ4HFSw="
49223e53a50797f62e14849eeec1b0603ca3e49e\10d9c3658624cdbae4d75083402ef6053b0f2f20\01f950429ce03eb4e4ddaba89be0dd623a0e63d9="MmIwZLW//+9lcOJzvB5l8y3y844yYjBkAcrri4E8iCqKL/AQUZ9ypzNiMGRnM2IwZGUjBDBkZTJiMERlM2JYtzAAuQR4lt/PwzxUxqCE43R+2Dwr/HmM8QyWGiBIAWUzYjBq5TNiMGZlM0IwZGWH5tSIspapDjX8LDXT7tSKcdzlZnVMBOhCD7O+cK/ZEiBkZTM41ijexFIuZiC7QPkkvzAxcGRlM9PBZCIh9lTdzFYICG3TEVQ6k9VlzoYxWv9CZ+Rhv2fXgBGo/zLRuk9f2uVDkahqdPgEVHKpI/tZk8zHwEspztE="
05ebb2318ac97cba3ac943542c9763a1e48659d2="MzIyMuK+r+0zJ+Mjvkgy8n3wpdkzMjIyVsu7iddriXqIeacRAZ0k8DIyMjIwMjIyMjIiVDIyMjMyMhIyMjLi/LS8DYTCAWxaKOT0gjfP4NT9oALbB+ZsdDDWk00OOTIyMjI8sjIyMjAyMhIyMjJ+M6jhP3dGG/M2H2u2Iv3Fu58sggo+/dgqN3h+x4ZxD3IyMjIeH3A7VO8IWD6g5aC6yOBPITwRUF8lvBWshC8ZHnA6Noat1unNn/z9Xzqz13cFDo75RTzWp12vXhHEsb4Sh9HXcjIyMouAafaIn7VNpN3b+BMhpfHAmm8cjcsbpDnJMdoW4mVV+xdtu5xWK0j8C9RH/DVBf0i9qVF6F1zcG1qhgUrTKfM="
ec13c5a13d1b3a554e820bcf33cd64d112575b65="ZTMyN7Loru02d7Uivk1ipHzwoIllMzI3Bp26idI733uIfPdHAJ0hoGQzMjdgZDMyN2J0VTI3YmUzMhdiZDOvHPVyb+6pZQ+1mC1x1ZeygjWIg6Das0B0xTd7hr5+r2JkMzI54mQzMjViZBMyN2IrtS0oPnh1zLA0csgY5gFgDPyblzai7ytfuqLtDehhbyI3YmQ/n5QPBqEPbSTILVPvTupWcjdiZM5VYPPPzhhSioVge6w8CMXU5tNMMVptCq5/l5ZgMwhLlxfB1l/qpgXU775adMXv/uG2TxY5tBUu5y7o8jQSfaY="
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           �  ���                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        �#��I�No�ɂ��=p:q���5e��B�ְ�W�OJ6�� C�Nq2#��;pi����qs������x��m��4�v/��4��6���BQ`�N��$V��_,��L5,H��	���N��l2g�;c3='�2�@$��`g�cT�\���'ߛ@^��pB؈���/��+�ed������X���ͅ)g��Gծ� jn�
ey'}�R:���,Yʏ��p�txJ������BQ-?7��Jٟ�|7{�����f����-f���-���r�U^�7p��{z����¨����l�"=�.�q��lB��V�H( ���	�pk�a��[�|��w���`�}c1�՛��ua��&�ON2+�w^Ŕ�l���! g����o�MGc:;�i౽������~PşH�^����}��]H�[�j?	�B��`�LI�kE����k��H!���ϭ� �5+�~���3�0�8>Q[�0t�76��&�%c���f�*Ɵ�>0�읪�=�=��]	��q   BTComm format 0  (@  U���g͟�ѹˊ+�C�@��ɺP��ٳ���2V����L�|�PY�8_ݼ2�EdA�fP�����9�����X��	/�o	�C㑑|*�d��9OA���ޣ�<�����������&w����PQ�U�|p��ׄ�,?E�N�,z�Ќ^,f���ؚ���q}���0��|<+�WN0Gl{kv�Yڕ�j�HC ��t�$6Z.[�0Eoـ��6'a��hd3�H�av��t��:�U��4�@�c�~YT׊bQy
����3�`�g�E@���[Zt*t̄T�>��Y4��Lb	n�4
�=x5~"F	�����G�/l3��_~A�������mED,	���Rl���A3qW���cv��*��~��&u,`� �n�&h����y��>w��CZ{>)wq|���.ڻ�y�R/=ј\S���OjIcx�H�(�^�1��M��\*��oY71"��>���� ��O��<�A�08��L��u_>,d�Jz��E�f	�	���-L����O��V���>M0��<�Babu�V�웽u��}&u�	]�h"�a��T�b&���&���*�{�#,g~��>E[��qb
�9���U� +�)��Sp���Z��vv�z���l��3]��������f�Y��q�}ƒK����躼��pW:%�_��ୁ�>T��L��w��~(��[�`|�@�F1>u�`�%�g���f��l<W�p�/fH�()w�I�A���C^�s�ŖO`��k9�-!��U��:�WW�%6v��#���CXX���2�^1fĿ[���E��6-���r�%�s���)%Tl��1X���GRE���ӹ��GŮJ��~-��d{��L�7��p��0*�j9�"s���5�D�̺B��Zt���9'JE8�X5.g��v֟����d �>Á4G/�1����gg���sRUۑmp�2����s��i�����b��{ݪ�~z��i5�ov4n�h��Y'T8�*O)Ű��t��ng���RL���UvL_�m�j)c�A���~�3�x���G�*Kv�4t��9�n���J$�a��.iB�;�ǷC� �Pۢ�6�H�aF�ΟDǰ#�>D�
U���/�]�Q-��)v�I��i�Y�'��w�w�tp�]"Y�y��	?,co�xڳ@����Z��v�IZ۪qNb��TK�����L���m��]�>��/K��Ɩ�~丆R�`d92aX�x����GH=W��*覶��QJڃ,��Fݬ֓V8��0y����~�@�(��d�Τ~¦)ӛ��8���t��	�KG*�/zA�OJF�3�����W5�j=V���-���;��Z�Ǆ�L�@��˗��B��Y�V��yhn���W/����;z���ҩO�`�	���o���F̱a4�K)�Y��a5|F���Wg�$�S`޶�����s��_̜�̸4�dݣdw�t:{W Y���2[V%	�p�L��P�bН��&y�\S>��������|m*?��Mt@8Wۚ6�)@��S�v��S���N`����ո��O���_Cd�
M	m7��0�����G�	 "��5�3�-8ҫ	1]�/1��c%�����n���դ����eD'&��z�e��i��BWp�zJ���7�Q��74�.�e���ϼoK����2��'O�p�}�sV��_EP���̨��k��K@��fp�,ٗ3��C��-<�����X�O�_�S>����z�N�RnhD.L�~=��E�r���	�c}Ғ��S<�V��3��2��z
���f�W�P�1��v�RE��kKVJ^;�_]�>�u�� B�`��|�ށ���N��`"FH��N��R	���<� S��j9 ����ҖiH8?\���J?�f�4��"��Г��Fl����]�uU��Q���=e!� �CV����l*���&��@I�=�����;ֈ��j���p��w�u/��l��	:��m�Xai�4w���*�-hV0��+X���]|�(�����Jx<n:����F�p��6��Z}�5'�+�ރ���lm�N��!� >���"���o��/�xY�Y�2��#L�~�d�N�/}�48Y0t�p����pȱX� 2�	ۆ�����Z*� ժ�LT�Mɮ�w�L�ƖEN�~�T&i>�Z�@��;�z��$��7/�nq/������Un�;�^��9OU�Y�U�����C��BPMVָ��Ђ�L$�I��˗��S �%�c9%�:%��мN���
!q���>"�Og�z�F������8�U���W.��YϕyԐPw�����.$�/"��4�\��,=�n�6rYĉM��a>�Trw��ɻ����6�Ip���\�rx\iԆZ��0����%J�o�����QB���I����J�EN�Ƣ͛��T�E�$�twt��Ϟ\�t�U���L����^��T�l�aĥN1�}��#�����c�'�0���+*
iDp'Z��n��q���ba�M:�!x��-�?�5��>�� ��e&�P��J��,��Z�����Ơ�4"
W��LX�]�`�O��H�	�I�����+�p1�	���I�5��e7�W�6�:���TLQ�uC�NM]�֬9�S��[?���洶�` (w#oD��<$�c��ZcE���z\�P>�_q5'�*��L7�0x�����ך�u��Z<d1@�K���@�+yM��(�^���X�z|��fN�Fg%��=��?�s�+RGH��}����s�(D�X����(�{7��f]��۴�O!��j0��c��h�e��*V��-̭Xa��h�{oT.^�_��`B�w���/[ؙ�bQ�'���{+�	�Aő��T���Q����a2J^!����9�Xe�܋[���3�.ˀT�!)xz�$���$�2�B�~�cQ�� m/y��v7�W�]�J|���#�c������@�Kٚ�4��M��)&�w˰����b7^}s�����G<�_X6ى��ē���Qw����*�v��+;S&S�'�����H�W��3�����zN�q�
x@(ǦHK����O-��ց
v�t���.4�nq��¬�'6qOH:��G���v"(�QS&:��ۊb�e=�>w��j��T����r���A�Hf��`*�kҨ	� �RS�L�i�ޭ}:p`�W���j=-�wV��1���]�%M�K���!��f2��c�b3uބ�[:S��u��+I_2L����; ��8s��T��C��y���b���qF�yv�l+��f;r�gS
��h4a'D���3��O,d[�j+����"��Y��5����]�Pz�76���Y�"D�E��Ц!��#l2̀Of��B��e�
�&��>��)����U�sJ�y��縇L���힗�[tx	��W}�r2b�E��)��8��BP|O���Z��8��хif��s[���Dݎ�!�)~����X�4�T܆l�aA�'1<~�y&�"v]�r�����3\�w�B�u�l^G��o��֛����~a�bm�>�Y:�M7���Aq