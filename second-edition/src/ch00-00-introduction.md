# Introduction
# Introdução

Welcome to “The Rust Programming Language,” an introductory book about Rust.

Bem-vindo ao “A Linguagem de Programação Rust”, um livro introdutório sobre Rust.

Rust is a programming language that helps you write faster, more reliable
software. High-level ergonomics and low-level control are often at odds with
each other in programming language design; Rust stands to challenge that.
Through balancing powerful technical capacity and a great developer experience,
Rust gives you the option to control low-level details (such as memory usage)
without all the hassle traditionally associated with such control.

Rust é uma linguagem de programação que ajuda a escrever software mais rápido e confiável. 
A ergonomia de alto nível e o controle de baixo nível estão frequentemente em desacordo 
no design da linguagem de programação; Rust desafia isso. Ao equilibrar uma poderosa 
capacidade técnica e uma ótima experiência de desenvolvedor, Rust oferece a opção de 
controlar detalhes de baixo nível (como o uso de memória) sem todo o incômodo 
tradicionalmente associado a esse controle.

## Who Rust is For
## Para Quem Rust Serve

Rust is great for many people for a variety of reasons. Let’s discuss a few of
the most important groups.

Rust é excelente para muitas pessoas por várias razões. Vamos discutir alguns dos grupos 
mais importantes.

### Teams of Developers
### Times de Desenvolvedores

Rust is proving to be a productive tool for collaborating among large teams of
developers with varying levels of systems programming knowledge. Low-level code
is prone to a variety of subtle bugs, which in most other languages can only be
caught through extensive testing and careful code review by experienced
developers. In Rust, the compiler plays a gatekeeper role by refusing to
compile code with these kinds of bugs--including concurrency bugs. By working
alongside the compiler, the team can spend more time focusing on the logic of
the program rather than chasing down bugs.

Rust está provando ser uma ferramenta produtiva para colaborar entre grandes equipes 
de desenvolvedores com níveis variados de conhecimento de programação de sistemas. 
O código de baixo nível é propenso a uma variedade de erros sutis, que na maioria 
das outras linguagens só podem ser detectados por meio de testes extensivos e revisão 
cuidadosa do código por desenvolvedores experientes. Em Rust, o compilador desempenha 
um papel de guardião, recusando-se a compilar código com esses tipos de erros - incluindo 
erros de concorrência. Ao trabalhar junto com o compilador, a equipe pode dedicar mais 
tempo à lógica do programa, em vez de procurar bugs.

Rust also brings contemporary developer tools to the systems programming world:

* Cargo, the included dependency manager and build tool, makes adding,
  compiling, and managing dependencies painless and consistent across the Rust
  ecosystem.
* Rustfmt ensures a consistent coding style across developers.
* The Rust Language Server powers IDE integration for code completion and
  inline error messages.

A Rust também traz ferramentas de desenvolvedor contemporâneas para o mundo da 
programação de sistemas:

* Cargo, o gerenciador de dependências incluso e ferramenta de compilação, torna a adição, 
  compilação e gerenciamento de dependências indolor e consistente em todo o ecossistema Rust.
* O Rustfmt garante um estilo de codificação consistente entre os desenvolvedores.
* O Rust Language Server (RLS) possibilita a integração de IDEs para preenchimento de código 
  e mensagens de erro em linha.  

By using these and other tools in the Rust ecosystem, developers can be
productive while writing systems-level code.

Usando essas e outras ferramentas no ecossistema Rust, os desenvolvedores podem ser produtivos 
enquanto escrevem código de sistema.

### Students
### Estudantes

Rust is for students and people who are interested in learning about systems
concepts. Many people have learned about topics like operating systems
development through Rust. The community is happy to answer student questions.
Through efforts such as this book, the Rust teams want to make systems concepts
more accessible to more people, especially those getting started with
programming.

Rust é para estudantes e pessoas interessadas em aprender sobre os conceitos de 
sistemas. Muitas pessoas aprenderam sobre tópicos como desenvolvimento de sistemas 
operacionais através de Rust. A comunidade fica feliz em responder às perguntas dos 
alunos. Por meio de esforços como este livro, as equipes do Rust desejam tornar 
os conceitos de sistemas mais acessíveis a mais pessoas, especialmente aquelas que 
estão começando a programar.

### Companies
### Empresas

Rust is used in production by hundreds of companies, large and small, for a
variety of tasks, such as command line tools, web services, DevOps tooling,
embedded devices, audio and video analysis and transcoding, cryptocurrencies,
bioinformatics, search engines, internet of things applications, machine
learning, and even major parts of the Firefox web browser.

Rust é usado em produção por centenas de empresas, grandes e pequenas, para uma 
variedade de tarefas, como ferramentas de linha de comando, serviços na Web, 
ferramentas DevOps, dispositivos embarcados, análise e transcodificação de 
áudio e vídeo, criptomoedas, bioinformática, motores de busca, internet das coisas, 
aprendizado de máquina e até partes importantes do navegador Firefox.

### Open Source Developers
### Desenvolvedores de Código Aberto

Rust is for people who want to build the Rust programming language, community,
developer tools, and libraries. We’d love for you to contribute to the Rust
language.

Rust é para pessoas que desejam criar a linguagem de programação Rust, a comunidade, 
as ferramentas de desenvolvedor e as bibliotecas Rust. Gostaríamos que você 
contribuísse para a linguagem Rust.

### People Who Value Speed and Stability
### Pessoas que Valorizam a Velocidade e a Estabilidade

By speed, we mean both the speed of the programs that Rust lets you create and
the speed at which Rust lets you write them. The Rust compiler’s checks ensure
stability through feature additions and refactoring, as opposed to brittle
legacy code in languages without these checks that developers are afraid to
modify. By striving for zero-cost abstractions, higher level features that
compile to lower level code as fast as code written manually, Rust endeavors to
make safe code be fast code as well.

Por velocidade, entendemos a velocidade dos programas que Rust permite criar e a 
velocidade com que Rust permite que você os escreva. As verificações do compilador 
Rust garantem estabilidade por meio de adições e refatoração de recursos, em oposição 
ao código legado frágil (quebrável) em linguagens sem essas verificações, que os desenvolvedores 
têm medo de modificar. Ao buscar abstrações de custo zero, recursos de nível superior 
que se compilam para código de baixo  nível, tão rápido quanto o código escrito manualmente, 
Rust se esforça para tornar o código seguro bem como um código rápido.

This isn’t a complete list of everyone the Rust language hopes to support, but
these are some of the biggest stakeholders. Overall, Rust’s greatest ambition
is to take trade-offs that have been accepted by programmers for decades and
eliminate the dichotomy. Safety *and* productivity. Speed *and* ergonomics.
Give Rust a try, and see if its choices work for you.

Esta não é uma lista completa de tudo que a linguagem Rust espera apoiar, mas esses 
são alguns dos maiores interessados. No geral, a maior ambição de Rust é aceitar trocas 
aceitas pelos programadores há décadas e eliminar a dicotomia. Segurança *e* produtividade. 
Velocidade *e* ergonomia. Experimente Rust e veja se as opções funcionam para você.

## Who This Book is For
## Para Quem Serve esse Livro

This book assumes that you’ve written code in some other programming language,
but doesn’t make any assumptions about which one. We’ve tried to make the
material broadly accessible to those from a wide variety of programming
backgrounds. We don’t spend a lot of time talking about what programming *is*
or how to think about it; someone new to programming entirely would be better
served by reading a book specifically providing an introduction to programming.

Este livro pressupõe que você tenha escrito código em outra linguagem de programação, 
mas não faz nenhuma suposição sobre qual. Tentamos tornar o material amplamente acessível 
para aqueles de uma ampla variedade de contextos de programação. Não passamos muito tempo 
conversando sobre o que *é* programação ou como pensar sobre programação; alguém novato 
em programação seria melhor atendido lendo um livro especificamente fornecendo uma 
introdução à programação.

## How to Use This Book
## Como Usar esse Livro

This book generally assumes that you’re reading it front-to-back, that is,
later chapters build on top of concepts in earlier chapters, and earlier
chapters may not dig into details on a topic, revisiting the topic in a later
chapter.

Este livro geralmente supõe que você o esteja lendo de frente para trás, ou seja, os 
capítulos posteriores se baseiam nos conceitos dos capítulos anteriores, e os capítulos 
anteriores podem não se aprofundar nos detalhes de um tópico, revisando o tópico em um 
capítulo posterior.

There are two kinds of chapters in this book: concept chapters, and project
chapters. In concept chapters, you’ll learn about an aspect of Rust. In the
project chapters, we’ll build small programs together, applying what we’ve
learned so far. Chapters 2, 12, and 20 are project chapters; the rest are
concept chapters.

Existem dois tipos de capítulos neste livro: capítulos conceituais e capítulos de 
projetos. Nos capítulos conceituais, você aprenderá sobre um aspecto de Rust. Nos 
capítulos de projeto, criaremos pequenos programas juntos, aplicando o que aprendemos 
até agora. Os capítulos 2, 12 e 20 são capítulos de projetos; o resto são capítulos conceituais.

Additionally, Chapter 2 is a hands-on introduction to Rust as a language. We’ll
cover concepts at a high level, and later chapters will go into them in detail.
If you’re the kind of person who likes to get their hands dirty right away,
Chapter 2 is great for that. If you’re *really* that kind of person, you may
even wish to skip over Chapter 3, which covers features that are very similar
to other programming languages, and go straight to Chapter 4 to learn about
Rust’s ownership system. By contrast, if you’re a particularly meticulous
learner who prefers to learn every detail before moving onto the next, you may
want to skip Chapter 2 and go straight to Chapter 3.

Além disso, o Capítulo 2 é uma introdução prática ao Rust como linguagem. Abordaremos 
conceitos de alto nível e os capítulos posteriores serão detalhados. Se você é o tipo 
de pessoa que gosta de sujar as mãos imediatamente, o Capítulo 2 é ótimo para isso. Se 
você é realmente esse tipo de pessoa, pode até pular o Capítulo 3, que abrange recursos 
muito semelhantes a outras linguagens de programação, e vá direto ao Capítulo 4 para 
aprender sobre o sistema de ownership (propriedade) Rust. Por outro lado, se você é 
particularmente aluno meticuloso que prefere aprender todos os detalhes antes de passar 
para o próximo, pule o Capítulo 2 e vá direto para o Capítulo 3.

Chapter 5 discusses structs and methods, and Chapter 6 covers enums, `match`
expressions, and the `if let` control flow construct. Structs and enums are the
ways to make custom types in Rust.

O Capítulo 5 discute estruturas e métodos, e o Capítulo 6 aborda enumerações, expressões 
`match` e a construção de fluxo de controle `if let`. Estruturas e enums são as maneiras 
de criar tipos personalizados no Rust.

In Chapter 7, you'll learn about Rust's module system and privacy for
organizing your code and its public API. Chapter 8 discusses some common
collection data structures provided by the standard library: vectors, strings,
and hash maps. Chapter 9 is all about Rust's error handling philosophy and
techniques.

No Capítulo 7, você aprenderá sobre o sistema de módulos e a privacidade do Rust para 
organizar seu código e sua API pública. O capítulo 8 discute algumas estruturas comuns de 
dados de coleta fornecidas pela biblioteca padrão: vetores, seqüências de caracteres e mapas 
de hash. O Capítulo 9 trata da filosofia e das técnicas de manipulação de erros em Rust.

Chapter 10 digs into generics, traits, and lifetimes, which give you the power
to define code that applies to multiple types. Chapter 11 is all about testing,
which is still necessary even with Rust's safety guarantees to ensure your
program's logic is correct. In Chapter 12, we'll build a subset of the
functionality of the `grep` command line tool that searches for text within
files and we'll use many of the concepts we discussed in the previous chapters.

O capítulo 10 analisa *generics* (genéricos), *traits* (características) e *lifetimes* 
(tempo de vida; vida útil), que permitem definir o código que se aplica a vários tipos. 
O capítulo 11 é sobre testes, o que ainda é necessário, mesmo com as garantias de segurança 
Rust para garantir que a lógica do seu programa esteja correta. No Capítulo 12, construiremos um 
subconjunto da funcionalidade da ferramenta de linha de comando `grep` que pesquisa texto nos 
arquivos e usaremos muitos dos conceitos discutidos nos capítulos anteriores.

Chapter 13 explores closures and iterators: features of Rust that come from
functional programming languages. In Chapter 14, we'll explore more about Cargo
and talk about best practices for sharing your libraries with others. Chapter
15 discusses smart pointers provided by the standard library and the traits
that enable their functionality.

O Capítulo 13 explora *closures* (fechamentos) e iteradores: recursos Rust provenientes 
de linguagens de programação funcionais. No capítulo 14, exploraremos mais sobre o Cargo 
e falaremos sobre as práticas recomendadas para compartilhar suas bibliotecas com outras 
pessoas. O capítulo 15 discute os *smart pointers* (ponteiros inteligentes) fornecidos 
pela biblioteca padrão e as *traits* que permitem sua funcionalidade.

In Chapter 16, we'll go through different models of concurrent programming and
how Rust helps you to program using multiple threads fearlessly. Chapter 17
looks at how Rust idioms compare to Object Oriented Programming principles you
may be familiar with.

No Capítulo 16, abordaremos diferentes modelos de programação concorrente e como 
Rust ajuda você a programar, sem medo, usando várias threads. O Capítulo 17 analisa como 
a linguagem Rust se comparam aos princípios de Programação Orientada a Objetos com os quais 
você deve estar familiarizado.

Chapter 18 is a reference on patterns and pattern matching, which are powerful
ways of expressing ideas throughout Rust programs. Chapter 19 is a smorgasbord
of advanced topics that you might be interested in, including unsafe Rust and
more about lifetimes, traits, types, functions, and closures.

O capítulo 18 é uma referência sobre *patterns* (padrões) e *pattern matching* (correspondência 
de padrões), que são maneiras poderosas de expressar idéias nos programas Rust. O Capítulo 19 
é um monte de tópicos avançados nos quais você pode estar interessado, incluindo *unsafe Rust* 
(Rust inseguro) e mais sobre *lifetimes*, *traits*, tipos, funções e *closures* (fechamentos).

In Chapter 20, we'll finish up with a project where we'll implement a low-level
multithreaded web server!

No capítulo 20, concluiremos um projeto em que implementaremos um servidor web multithread 
de baixo nível!

Finally, there are some appendices. These contain useful information about the
language in a more reference-like format.

Finalmente, existem alguns apêndices. Eles contêm informações úteis sobre a linguagem em 
um formato mais parecido como uma referência.

In the end, there’s no wrong way to read a book: if you want to skip ahead, go
for it! You may have to jump back if you find things confusing. Do whatever
works for you.

No final, não há uma maneira errada de ler o livro: se você quiser pular, vá em frente! 
Você pode ter que voltar atrás se achar as coisas confusas. Faça o que funciona para você.

An important part of the process of learning Rust is learning how to read the
error messages that the compiler gives you. As such, we’ll be showing a lot of
code that doesn’t compile, and the error message the compiler will show you in
that situation. As such, if you pick a random example, it may not compile!
Please read the surrounding text to make sure that you didn’t happen to pick
one of the in-progress examples.

Uma parte importante do processo de aprendizado Rust é aprender a ler as mensagens de erro 
fornecidas pelo compilador. Como tal, mostraremos muito código que não é compilado, e a 
mensagem de erro que o compilador mostrará nessa situação. Dessa forma, se você escolher um 
exemplo aleatório, ele pode não ser compilado! Leia o texto ao redor para garantir que você 
não tenha escolhido um dos exemplos em andamento.

## Contributing to the Book
## Contribuindo Para o Livro

This book is open source. If you find an error, please don’t hesitate to file
an issue or send a pull request [on GitHub]. Please see [CONTRIBUTING.md] for
more details.

Este livro é de código aberto. Se você encontrar um erro, não hesite em registrar um problema 
ou enviar um *pull request* (solicitação de recebimento) [pt_br on GitHub]. Por favor, consulte 
[CONTRIBUTING.md] para mais detalhes.

Para contribuições na língua inglêsa veja [on GitHub] e [CONTRIBUTING-en-us.md], respectivamente.

[on GitHub]: https://github.com/rust-lang/book
[CONTRIBUTING-en-us.md]: https://github.com/rust-lang/book/blob/master/CONTRIBUTING.md

[pt_br on GitHub]: https://github.com/rust-br/rust-book-pt-br
[CONTRIBUTING.md]: https://github.com/rust-br/rust-book-pt-br/blob/master/CONTRIBUTING.md
