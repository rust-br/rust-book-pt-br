# Introdução

Bem-vindo ao “A Linguagem de Programação Rust”, um livro introdutório sobre Rust.

Rust é uma linguagem de programação que ajuda a escrever software mais rápido e confiável. 
A ergonomia de alto nível e o controle de baixo nível estão frequentemente em desacordo 
no design da linguagem de programação; Rust desafia isso. Ao equilibrar uma poderosa 
capacidade técnica e uma ótima experiência de desenvolvedor, Rust oferece a opção de 
controlar detalhes de baixo nível (como o uso de memória) sem todo o incômodo 
tradicionalmente associado a esse controle.

## Para Quem Rust Serve

Rust é excelente para muitas pessoas por várias razões. Vamos discutir alguns dos grupos 
mais importantes.

### Times de Desenvolvedores

Rust está provando ser uma ferramenta produtiva para colaborar entre grandes equipes 
de desenvolvedores com níveis variados de conhecimento de programação de sistemas. 
O código de baixo nível é propenso a uma variedade de erros sutis, que na maioria 
das outras linguagens só podem ser detectados por meio de testes extensivos e revisão 
cuidadosa do código por desenvolvedores experientes. Em Rust, o compilador desempenha 
um papel de guardião, recusando-se a compilar código com esses tipos de erros - incluindo 
erros de concorrência. Ao trabalhar junto com o compilador, a equipe pode dedicar mais 
tempo à lógica do programa, em vez de procurar bugs.

A Rust também traz ferramentas de desenvolvedor contemporâneas para o mundo da 
programação de sistemas:

* Cargo, o gerenciador de dependências incluso e ferramenta de compilação, torna a adição, 
  compilação e gerenciamento de dependências indolor e consistente em todo o ecossistema Rust.
* O Rustfmt garante um estilo de codificação consistente entre os desenvolvedores.
* O Rust Language Server (RLS) possibilita a integração de IDEs para preenchimento de código 
  e mensagens de erro em linha.  

Usando essas e outras ferramentas no ecossistema Rust, os desenvolvedores podem ser produtivos 
enquanto escrevem código de sistema.

### Estudantes

Rust é para estudantes e pessoas interessadas em aprender sobre os conceitos de 
sistemas. Muitas pessoas aprenderam sobre tópicos como desenvolvimento de sistemas 
operacionais através de Rust. A comunidade fica feliz em responder às perguntas dos 
alunos. Por meio de esforços como este livro, as equipes do Rust desejam tornar 
os conceitos de sistemas mais acessíveis a mais pessoas, especialmente aquelas que 
estão começando a programar.

### Empresas

Rust é usado em produção por centenas de empresas, grandes e pequenas, para uma 
variedade de tarefas, como ferramentas de linha de comando, serviços na Web, 
ferramentas DevOps, dispositivos embarcados, análise e transcodificação de 
áudio e vídeo, criptomoedas, bioinformática, motores de busca, internet das coisas, 
aprendizado de máquina e até partes importantes do navegador Firefox.

### Desenvolvedores de Código Aberto

Rust é para pessoas que desejam criar a linguagem de programação Rust, a comunidade, 
as ferramentas de desenvolvedor e as bibliotecas Rust. Gostaríamos que você 
contribuísse para a linguagem Rust.

### Pessoas que Valorizam a Velocidade e a Estabilidade

Por velocidade, entendemos a velocidade dos programas que Rust permite criar e a 
velocidade com que Rust permite que você os escreva. As verificações do compilador 
Rust garantem estabilidade por meio de adições e refatoração de recursos, em oposição 
ao código legado frágil (quebrável) em linguagens sem essas verificações, que os desenvolvedores 
têm medo de modificar. Ao buscar abstrações de custo zero, recursos de nível superior 
que se compilam para código de baixo  nível, tão rápido quanto o código escrito manualmente, 
Rust se esforça para tornar o código seguro bem como um código rápido.

Esta não é uma lista completa de tudo que a linguagem Rust espera apoiar, mas esses 
são alguns dos maiores interessados. No geral, a maior ambição de Rust é aceitar trocas 
aceitas pelos programadores há décadas e eliminar a dicotomia. Segurança *e* produtividade. 
Velocidade *e* ergonomia. Experimente Rust e veja se as opções funcionam para você.

## Para Quem Serve esse Livro

Este livro pressupõe que você tenha escrito código em outra linguagem de programação, 
mas não faz nenhuma suposição sobre qual. Tentamos tornar o material amplamente acessível 
para aqueles de uma ampla variedade de contextos de programação. Não passamos muito tempo 
conversando sobre o que *é* programação ou como pensar sobre programação; alguém novato 
em programação seria melhor atendido lendo um livro especificamente fornecendo uma 
introdução à programação.

## Como Usar esse Livro

Este livro geralmente supõe que você o esteja lendo de frente para trás, ou seja, os 
capítulos posteriores se baseiam nos conceitos dos capítulos anteriores, e os capítulos 
anteriores podem não se aprofundar nos detalhes de um tópico, revisando o tópico em um 
capítulo posterior.

Existem dois tipos de capítulos neste livro: capítulos conceituais e capítulos de 
projetos. Nos capítulos conceituais, você aprenderá sobre um aspecto de Rust. Nos 
capítulos de projeto, criaremos pequenos programas juntos, aplicando o que aprendemos 
até agora. Os capítulos 2, 12 e 20 são capítulos de projetos; o resto são capítulos conceituais.

Além disso, o Capítulo 2 é uma introdução prática ao Rust como linguagem. Abordaremos 
conceitos de alto nível e os capítulos posteriores serão detalhados. Se você é o tipo 
de pessoa que gosta de sujar as mãos imediatamente, o Capítulo 2 é ótimo para isso. Se 
você é realmente esse tipo de pessoa, pode até pular o Capítulo 3, que abrange recursos 
muito semelhantes a outras linguagens de programação, e vá direto ao Capítulo 4 para 
aprender sobre o sistema de ownership (propriedade) Rust. Por outro lado, se você é 
particularmente aluno meticuloso que prefere aprender todos os detalhes antes de passar 
para o próximo, pule o Capítulo 2 e vá direto para o Capítulo 3.

O Capítulo 5 discute estruturas e métodos, e o Capítulo 6 aborda enumerações, expressões 
`match` e a construção de fluxo de controle `if let`. Estruturas e enums são as maneiras 
de criar tipos personalizados no Rust.

No Capítulo 7, você aprenderá sobre o sistema de módulos e a privacidade do Rust para 
organizar seu código e sua API pública. O capítulo 8 discute algumas estruturas comuns de 
dados de coleta fornecidas pela biblioteca padrão: vetores, seqüências de caracteres e mapas 
de hash. O Capítulo 9 trata da filosofia e das técnicas de manipulação de erros em Rust.

O capítulo 10 analisa *generics* (genéricos), *traits* (características) e *lifetimes* 
(tempo de vida; vida útil), que permitem definir o código que se aplica a vários tipos. 
O capítulo 11 é sobre testes, o que ainda é necessário, mesmo com as garantias de segurança 
Rust para garantir que a lógica do seu programa esteja correta. No Capítulo 12, construiremos um 
subconjunto da funcionalidade da ferramenta de linha de comando `grep` que pesquisa texto nos 
arquivos e usaremos muitos dos conceitos discutidos nos capítulos anteriores.

O Capítulo 13 explora *closures* (fechamentos) e iteradores: recursos Rust provenientes 
de linguagens de programação funcionais. No capítulo 14, exploraremos mais sobre o Cargo 
e falaremos sobre as práticas recomendadas para compartilhar suas bibliotecas com outras 
pessoas. O capítulo 15 discute os *smart pointers* (ponteiros inteligentes) fornecidos 
pela biblioteca padrão e as *traits* que permitem sua funcionalidade.

No Capítulo 16, abordaremos diferentes modelos de programação concorrente e como 
Rust ajuda você a programar, sem medo, usando várias threads. O Capítulo 17 analisa como 
a linguagem Rust se comparam aos princípios de Programação Orientada a Objetos com os quais 
você deve estar familiarizado.

O capítulo 18 é uma referência sobre *patterns* (padrões) e *pattern matching* (correspondência 
de padrões), que são maneiras poderosas de expressar idéias nos programas Rust. O Capítulo 19 
é um monte de tópicos avançados nos quais você pode estar interessado, incluindo *unsafe Rust* 
(Rust inseguro) e mais sobre *lifetimes*, *traits*, tipos, funções e *closures* (fechamentos).

No capítulo 20, concluiremos um projeto em que implementaremos um servidor web multithread 
de baixo nível!

Finalmente, existem alguns apêndices. Eles contêm informações úteis sobre a linguagem em 
um formato mais parecido como uma referência.

No final, não há uma maneira errada de ler o livro: se você quiser pular, vá em frente! 
Você pode ter que voltar atrás se achar as coisas confusas. Faça o que funciona para você.

Uma parte importante do processo de aprendizado Rust é aprender a ler as mensagens de erro 
fornecidas pelo compilador. Como tal, mostraremos muito código que não é compilado, e a 
mensagem de erro que o compilador mostrará nessa situação. Dessa forma, se você escolher um 
exemplo aleatório, ele pode não ser compilado! Leia o texto ao redor para garantir que você 
não tenha escolhido um dos exemplos em andamento.

## Contribuindo Para o Livro

Este livro é de código aberto. Se você encontrar um erro, não hesite em registrar um problema 
ou enviar um *pull request* (solicitação de recebimento) [pt_br on GitHub]. Por favor, consulte 
[CONTRIBUTING.md] para mais detalhes.

Para contribuições na língua inglêsa veja [on GitHub] e [CONTRIBUTING-en-us.md], respectivamente.

[on GitHub]: https://github.com/rust-lang/book
[CONTRIBUTING-en-us.md]: https://github.com/rust-lang/book/blob/master/CONTRIBUTING.md

[pt_br on GitHub]: https://github.com/rust-br/rust-book-pt-br
[CONTRIBUTING.md]: https://github.com/rust-br/rust-book-pt-br/blob/master/CONTRIBUTING.md