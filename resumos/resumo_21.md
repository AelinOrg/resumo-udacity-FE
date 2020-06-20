# Introdução a acessibilidade
A expressão acessibilidade Web refere-se a prática inclusiva de fazer websites que possam ser utilizados por todas as pessoas que tenham deficiência ou não. Quando os sites são corretamente concebidos, desenvolvidos e editados, todos os usuários podem ter igual acesso à informação e funcionalidade

Uma boa acessibilidade, ou "a11y", é fundamental para garantir que todos os usuários possam acessar o conteúdo dos seus sites e aplicativos. Dar a devida atenção à acessibilidade no início do processo vai garantir que seu produto final seja melhor lapidado e funcione para mais pessoas.

Ao longo deste curso, vamos colocar você para fazer diversos exercícios práticos para você se familiarizar com assuntos e técnicas de acessibilidade. Veja todos os exercícios no [repositório do projeto no GitHub](https://github.com/udacity/ud891). Então, vamos começar! Clone o repositório para poder consultá-lo quando chegamos nas perguntas sobre nodes.

**Observação**: Costumamos encurtar a palavra "acessibilidade" para "a11y" porque existem 11 letras entre "A" e "Y" na palavra "accessibility" (acessibilidade). Às vezes, você vai ver esse padrão em outros contextos, como i18n para "internationalization" (internacionalização), e l10n para "localization" (localização).

Embora a acessibilidade sirva para incluir pessoas, ela melhora para todos.

# Entendendo a diversidade de usuários

Algumas estatísticas sobre a deficiência nos EUA:

* Cerca de 2% da população tem alguma deficiência visual (ou seja, cegueira ou dificuldade considerável para enxergar, mesmo usando óculos)
* Cerca de 50% da população tem algum erro refrativo clinicamente relevante (uma deficiência visual que, se branda, pode ser corrigida com óculos)
* Cerca de 8% dos homens e 0,5% das mulheres sofrem de alguma forma de deficiência visual relacionada às cores
* Cerca de 2% dos adultos têm deficiência auditiva
* Mais de 4% têm problemas cognitivos (problemas de memória, déficit de atenção ou problemas para tomar decisões)

# Listas de verificação
Como a acessibilidade é algo tão amplo e a aplicação é tão diversa, precisaremos de um mapa para nos guiar, neste caso o [WCAG](https://www.w3.org/TR/WCAG20/) (as Diretrizes de Acessibilidade para o Conteudo da Web). Ele apresenta instrunções e boas praticas desenvolvidas por especilistas em acessibilidade a fim de explicar o significado de acessibilidade de forma metodica. Varios paises impõem o uso das diretrizes nas exigencias legais da web. o WCAG é organizado em torno de quatro principios:

* Perceptivel: Os usuarios podem perceber o conteudo. Se algo foi perceptivel atraves da visão, isso não significa que todos possam percebê-lo.

* Operavel: Usuarios acessam os componentes da UI e navegam pelo conteudo? Algo que requer interação visual não pode ser operado por alguem que não utiliza um mouse ou uma tela de toque.

* Compreensível: Compreensão do conteudo. Eles conseguem compreender a interface e ela é consistente o suficiente para evitar confusão?

* Robusta: Ela é robusta o suficiente para que o conteudo seja consumido por uma grande variedade de agentes usuarios? Ela funciona com tecnologia assistiva?

Juntas elas forma o POUR (POCR).

Enquanto o WCAG fornece caminhos compreensiveis que nos ajudam a ter em mente, ele pode ser demais. Para facilitar o entendimento, a WebAIM resumiu as diretrizes em [checklist](https://webaim.org/standards/wcag/checklist) facil de ser seguido focado especialmente para o conteudo web. O checklist traz um resumo requintado do que precisa ser implementado e apresenta as especificações WCAG, caso precise das definições expandida. O WCAG e o checklist tratam da acessibilidade, mas são aproximações limitadas da verdadeira acessibilidade. O que realmente importa é a experiencia do usuario, e não so conferir alguns itens. Embora alguns guias nos deem um norte para pensar no assunto, existem partes imcompletas e até mesmo ultrapassadas. Com essas ferramentas temos a direção do trabalho.

# Passando pelos aspectos práticos

* Foco do teclado: Construindo coisas que podem ser operadas com o teclado. Isso serve para os usuarios com deficiencias motoras, mas garante que a UI (Interface do Usuario) esteja pronta para as semantica.

* Semantica: É nela que garantimos que nossa interface se expresse de forma robusta funcionando com varias tecnologias assistivas.

* Estilo: Estilo e design visual. Veremos tecnicas para tornar a interface de usuario mais flexivel e usavel.