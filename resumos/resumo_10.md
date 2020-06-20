# O que é um pixel?
Os pixels compõem os dados de uma foto. Cada cor imaginavel pode ser representada com a combinação de vermelho, verde e azul (RGB). Dependendo do tipo de imagem, ele pode ter ainda um canal alfa (A) adicional, que informa se a cor é solida ou transparente.

# Filtros e Efeitos
Aplicativos como photoshop e instagram permiter aplicar efeitos nas imagens. Mas por que editariamos uma imagem no navegador? A maioria das fotos só existem no formato digital. A potencia necessaria para processar uma imagem é quase que insignificante. Fazer coisas localmente permite fazer mudanças sem precisar ir e vir no servidor. Podemos deixar uma imagem em preto e branco alterando os valores RBG, inverter cores, etc.

# Dados de Imagem Canvas 2D
A imagem no canvas é representado no JS por um objeto ImageData. Ele contém os valores de:

* `width`;
* `height`; e
* array `data` - contem os valore de vermelho, verde, azul e do alfa de cada pixel.

A propriedade `data` é geralmente representada como `Uint8ClampedArray`, sendo:

* U - significa que o array não esta assinado ou contem somente os valores positivos.
* int8 - indica que armazenaremos numeros de 8bits. Numeos positivos de 8bits vão de 0 a 255.

Se o JS facilitasse para nós, os dados de 1px seriam separados uns dos outros da seguinte forma:

* [[r⁰, g⁰, b⁰, a⁰], [r¹, g¹, b¹, a¹], [r², g², b², a²]... [rⁿ, gⁿ, bⁿ, aⁿ]]

Em vez disso, temos uma lista para parsear ao aplicarmos efeitos:

* [r⁰, g⁰, b⁰, a⁰, r¹, g¹, b¹, a¹, r², g², b², a²... rⁿ, gⁿ, bⁿ, aⁿ]

Podemos obter ou modificar os dados da imagem chamando uma dessas funções:

* `createImageData` - Inicia um objeto `blankImage` que podemos modificar.
* `getImageData` - Obtem dados em um canvas de apoio.
* `putImageData`- Armazena dados em um canvas de apoio.

#  Game Loop / Processamento da Entrada de Usuário
Reproduzir um vídeo em um canvas usando requestAnimationFrame é apenas uma das várias interatividades que você pode fazer.

Para criar aplicativos mais complexos, precisamos pensar mais não apenas sobre o que apresentamos ao usuário na tela, mas também sobre a entrada (teclado, mouse, áudio) que o usuário pode gerar que deveremos processar.

O game loop é uma sequência de eventos executados continuamente enquanto um aplicativo ou jogo é usado. O requestAnimationFrame cuida do trabalho pesado, pois garante que seu aplicativo seja executado com cerca de 60 quadros por segundo durante a exibição ativa do aplicativo.

Supondo-se que já tenhamos criado as funções que planejamos chamar, um game loop completo teria a aparência a seguir.

```
function draw() {
    // request to execute this function at the next earliest convenience
    requestAnimationFrame(draw);
    processInput();
    moveObjectsAndEnemies();
    drawAllTheThings();
}

```

## Processamento de Entradas do Teclado
Embora não seja muito difícil processar os pressionamentos do teclado manualmente, prefiro usar projetos de código aberto que já tenham aperfeiçoado uma biblioteca que atenda ao que desejo fazer. Uma dessas bibliotecas é a [Kibo](https://github.com/marquete/kibo).

A Kibo permite referenciar chaves por seus nomes comuns ('a', '3', 'up') em vez de seus códigos-chave simplificando muito seu código. Você também pode anexar eventos à ação de pressionar ou soltar uma chave, bem como chaves modificadoras ou curingas.

```
var k = new Kibo();
k.down(['up', 'w'], function() {
    // Do something cool on the canvas
});

k.up(['enter', 'q'], function() {
    // Do other stuff.
});

```

## Processamento da entrada de mouse
Assim como outros elementos DOM, o canvas pode aceitar eventos click e mousedown. Temos um pouco de trabalho para descobrir onde exatamente no canvas o usuário clicou. Os eventos do clique do mouse retornam posições clientX e clientY que são globais na janela do navegador. Cada elemento sabe onde está posicionado em relação à posição dos navegadores (0,0) (offsetLeft e offsetTop).

Para adquirir a relação do canvas ao clique, você precisa subtrair os valores offsetLeft e offsetTop de clientX e clientY. Verifique o código de exemplo abaixo.

```
var c = document.querySelector("canvas");

function handleMouseClick(evt) {
        x = evt.clientX - c.offsetLeft;
        y = evt.clientY - c.offsetTop;
        console.log("x,y:"+x+","+y);
}
c.addEventListener("click", handleMouseClick, false);

```