<!-- # Introduction

> Note: This edition of the book is the same as [The Rust Programming
> Language][nsprust] available in print and ebook format from [No Starch
> Press][nsp].

[nsprust]: https://nostarch.com/rust
[nsp]: https://nostarch.com/

Welcome to _The Rust Programming Language_, an introductory book about Rust.
The Rust programming language helps you write faster, more reliable software.
High-level ergonomics and low-level control are often at odds in programming
language design; Rust challenges that conflict. Through balancing powerful
technical capacity and a great developer experience, Rust gives you the option
to control low-level details (such as memory usage) without all the hassle
traditionally associated with such control. -->

# Introdução

> Nota: Esta edição do livro é a mesma que [The Rust Programming
> Language][nsprust] (edição em inglês) disponível nas formas impressa e
> ebook pela [No Starch Press][nsp].

[nsprust]: https://nostarch.com/rust
[nsp]: https://nostarch.com/

Bem-vindo a _A Linguagem de Programação Rust_, um livro introdutório sobre
Rust. A linguagem de programação Rust te ajuda a escrever software mais rápido
e confiável. Ergonômia de alto nível e controle de baixo nível estão
frequentemente em conflito no projeto de linguagens de programação. Rust
desafia esse conflito. Balanceando capacidade técnica poderosa e uma
experiência de desenvolvimento incrível, Rust te dá a opção de controlar
detalhes de baixo nível (como o uso de memória) sem todo o trabalho
tradicionalmente associado com tais controles.

<!-- ## Who Rust Is For

Rust is ideal for many people for a variety of reasons. Let’s look at a few of
the most important groups.

### Teams of Developers

Rust is proving to be a productive tool for collaborating among large teams of
developers with varying levels of systems programming knowledge. Low-level code
is prone to a variety of subtle bugs, which in most other languages can be
caught only through extensive testing and careful code review by experienced
developers. In Rust, the compiler plays a gatekeeper role by refusing to
compile code with these elusive bugs, including concurrency bugs. By working
alongside the compiler, the team can spend their time focusing on the program’s
logic rather than chasing down bugs.

Rust also brings contemporary developer tools to the systems programming world:

- Cargo, the included dependency manager and build tool, makes adding,
  compiling, and managing dependencies painless and consistent across the Rust
  ecosystem.
- Rustfmt ensures a consistent coding style across developers.
- The Rust Language Server powers Integrated Development Environment (IDE)
  integration for code completion and inline error messages.

By using these and other tools in the Rust ecosystem, developers can be
productive while writing systems-level code.
 -->

## Para quem é Rust?

Rust é ideal para muitas pessoas por uma variedade de razões. Vamos dar uma
olhada em alguns dos grupos mais importantes.

### Times de Desenvolvimento

Rust está se provando ser uma ferramenta produtiva para a colaboração entre
times grandes de desenvolvedores com níveis variáveis de conhecimento em
linguagens de programação para sistemas. Código de baixo nível costuma ter uma
variedade de bugs sutis, os quais na maior parte das outras linguagens, podem
ser pegos apenas através de testes intensivos e revisão cuidadosa do código
por pessoas experientes. Em Rust, o compilador exerce um papel de guardião ao
se recusar a compilar código com esses bugs elusivos, incluindo bugs de
concorrência. Ao trabalhar junto do compilador, o time consegue gastar a maior
parte do seu tempo focando na lógica do programa ao invés de procurando bugs.

Rust também traz ferramentas de desenvolvimento contemporâneas para o mundo
da programação de sistemas:

- Cargo, o gestor de dependências e ferramenta de compilação, torna adicionar,
  compilar, e gerenciar dependências indolor e consistente através do ecossistema
  Rust.
- Rustfmt garante um estilo consistente de código entre quem desenvolve.
- O _Rust Language Server_ habilita integrações com Sistemas de Desenvolvimento
  Integrado (IDE) para preenchimento de código (code completion) e mensagens
  de erro embutidas.

Ao usar essas e outras ferramentas do ecossistema Rust, quem desenvolve podem
ser produtivos enquanto escrevem código de sistema.

<!-- ### Students

Rust is for students and those who are interested in learning about systems
concepts. Using Rust, many people have learned about topics like operating
systems development. The community is very welcoming and happy to answer
student questions. Through efforts such as this book, the Rust teams want to
make systems concepts more accessible to more people, especially those new to
programming.

### Companies

Hundreds of companies, large and small, use Rust in production for a variety of
tasks. Those tasks include command line tools, web services, DevOps tooling,
embedded devices, audio and video analysis and transcoding, cryptocurrencies,
bioinformatics, search engines, Internet of Things applications, machine
learning, and even major parts of the Firefox web browser.

### Open Source Developers

Rust is for people who want to build the Rust programming language, community,
developer tools, and libraries. We’d love to have you contribute to the Rust
language. -->

### Estudantes

Rust é para estudantes e aqueles que estejam interessados em aprender sobre
conceitos de sistema. Usando Rust, muitas pessoas aprenderam sobre tópicos
como desenvolvimento de sistemas operacionais. A comunidade é muito receptiva
e contente em responder perguntas de estudantes. Através de esforços como este
livro, os times de Rust querem fazer conceitos de sistemas mais acessíveis
a mais pessoas, especialmente àqueles novos na programação.

### Empresas

Centenas de empresas, pequenas e grandes, usam Rust em produção para uma
variedade de tarefas. Essas tarefas incluem ferramentas de linha de comando,
serviços web, ferramentas DevOps, dispositivos embarcados, análise de áudio e
vídeo e _transcoding_, criptomoedas, bioinformática, motores de busca,
aplicações de Internet das Coisas, aprendizado de máquina, e até partes
importantes do navegador Firefox.

### Desenvolvedores Open-Source

Rust é para pessoas que querem construir a linguagem de programação Rust, a
comunidade, as ferramentes de desenvolvimento, e as bibliotecas. Nós
adoraríamos que você contribuísse para a linguagem Rust.

<!-- ### People Who Value Speed and Stability

Rust is for people who crave speed and stability in a language. By speed, we
mean the speed of the programs that you can create with Rust and the speed at
which Rust lets you write them. The Rust compiler’s checks ensure stability
through feature additions and refactoring. This is in contrast to the brittle
legacy code in languages without these checks, which developers are often
afraid to modify. By striving for zero-cost abstractions, higher-level features
that compile to lower-level code as fast as code written manually, Rust
endeavors to make safe code be fast code as well.

The Rust language hopes to support many other users as well; those mentioned
here are merely some of the biggest stakeholders. Overall, Rust’s greatest
ambition is to eliminate the trade-offs that programmers have accepted for
decades by providing safety _and_ productivity, speed _and_ ergonomics. Give
Rust a try and see if its choices work for you. -->

### Pessoas que Valorizam Velocidade e Estabilidade

By striving for zero-cost abstractions, higher-level features
that compile to lower-level code as fast as code written manually, Rust
endeavors to make safe code be fast code as well.

Rust é para pessoas que desejam velocidade e estabilidade numa linguagem. Por
velocidade, nós queremos dizer a velocidade dos programas que você pode criar
com Rust e a velocidade com que Rust te permite escrevê-los. As checagens do
compilador de Rust garantem a estabilidade ao longo da adição de features e
refatoração. Isto está em contraste com o código legado frágil em linguagens
sem essas checagens, os quais desenvolvedores têm com frequência medo de
modificar. Ao almejar abstrações de custo zero, features de mais alto nível que
compilam para código de nível mais baixo tão rápido quanto código escrito
manualmente, Rust busca fazer de um código seguro um código rápido também.

A linguagem Rust espera suportar muitos outros usuários também. Aqueles
mencionados aqui são meramente alguns dos maiores interessados. No geral,
a maior ambição de Rust é eliminar os trade-offs que programadores aceitaram
por décadas provendo segurança _e_ produtividade, velocidade _e_ ergonomia.
Dê uma chance a Rust e veja se suas escolhas funcionam para você.

<!-- ## Who This Book Is For

This book assumes that you’ve written code in another programming language but
doesn’t make any assumptions about which one. We’ve tried to make the material
broadly accessible to those from a wide variety of programming backgrounds. We
don’t spend a lot of time talking about what programming _is_ or how to think
about it. If you’re entirely new to programming, you would be better served by
reading a book that specifically provides an introduction to programming.
 -->

## Para Quem É Este Livro

Este livro assume que você tenha escrito código em outra linguagem mas não
necessariamente assume sobre qual. Nós tentamos fazer do material amplamente
acessível àqueles de uma grande variedade de passados em programação. Nós
não gastamos muito tempo falando sobre o que a programação _é_ ou como pensar
sobre ela. Se você é inteiramente novo à programação, você seria melhor servido
lendo um livro que especificamente provê uma introdução à programação.

<!-- ## How to Use This Book

In general, this book assumes that you’re reading it in sequence from front to
back. Later chapters build on concepts in earlier chapters, and earlier
chapters might not delve into details on a topic; we typically revisit the
topic in a later chapter.

You’ll find two kinds of chapters in this book: concept chapters and project
chapters. In concept chapters, you’ll learn about an aspect of Rust. In project
chapters, we’ll build small programs together, applying what you’ve learned so
far. Chapters 2, 12, and 20 are project chapters; the rest are concept chapters.

Chapter 1 explains how to install Rust, how to write a Hello, world! program,
and how to use Cargo, Rust’s package manager and build tool. Chapter 2 is a
hands-on introduction to the Rust language. Here we cover concepts at a high
level, and later chapters will provide additional detail. If you want to get
your hands dirty right away, Chapter 2 is the place for that. At first, you
might even want to skip Chapter 3, which covers Rust features similar to those
of other programming languages, and head straight to Chapter 4 to learn about
Rust’s ownership system. However, if you’re a particularly meticulous learner
who prefers to learn every detail before moving on to the next, you might want
to skip Chapter 2 and go straight to Chapter 3, returning to Chapter 2 when
you’d like to work on a project applying the details you’ve learned. -->

## Como Utilizar Este Livro

Em geral, este livro assume que você o está lendo em sequência do começo ao
fim. Capítulos posteriores constroem sobre conceitos de capítulos anteriores,
e capítulos anteriores podem não entrar em detalhes num tópico. Nós
tipicamente revisitamos o tópico num capítulo posterior.

Você irá encontrar dois tipos de capítulos neste livro: capítulos sobre
conceitos e sobre projetos. Em capítulos sobre conceitos, você irá aprender
sobre algum aspecto de Rust. Em capítulos de projetos, nós iremos construir
pequenos programas juntos, aplicando o que você aprendeu até aqui. Os capítulos
2, 12, e 20 são capítulos de projeto; o resto são sobre conceitos.

O capítulo 1 explica como instalar Rust, como escrever um programa
_Hello, world!_, e como usar o Cargo, o gerenciador de dependências e
ferramenta de compilação de Rust. O capítulo 2 é uma introdução mão na massa
à linguagem de programação Rust. Aqui nós cobrimos conceitos num alto nível,
e capítulos posteriores providenciarão detalhes adicionais. Se você quiser
sujar as mãos logo, o capítulo 2 é o lugar para isso. No início, você pode até
pular o capítulo 3, que cobre as features de Rust similares a outras linguagens
de programação, e ir direto ao capítulo 4 para aprender sobre o sistema de
_ownership_ de Rust. No entanto, se você é um aprendiz particularmente
meticuloso, que prefere aprender todos os detalhes antes de ir para o próximo,
você pode querer pular o capítulo 2 e ir direto para o capítulo 3, retornando
ao capítulo 2 quando você quiser trabalhar num projeto aplicando os detalhes
que você aprendeu.

<!-- Chapter 5 discusses structs and methods, and Chapter 6 covers enums, `match`
expressions, and the `if let` control flow construct. You’ll use structs and
enums to make custom types in Rust.

In Chapter 7, you’ll learn about Rust’s module system and about privacy rules
for organizing your code and its public Application Programming Interface
(API). Chapter 8 discusses some common collection data structures that the
standard library provides, such as vectors, strings, and hash maps. Chapter 9
explores Rust’s error-handling philosophy and techniques.

Chapter 10 digs into generics, traits, and lifetimes, which give you the power
to define code that applies to multiple types. Chapter 11 is all about testing,
which even with Rust’s safety guarantees is necessary to ensure your program’s
logic is correct. In Chapter 12, we’ll build our own implementation of a subset
of functionality from the `grep` command line tool that searches for text
within files. For this, we’ll use many of the concepts we discussed in the
previous chapters. -->

O capítulo 5 discute as structs e os métodos, e o capítulo 6 cobre as enums,
expressões `match`, e o construto de controle de fluxo `if let`. Você irá usar
structs e enums para construir tipos customizados em Rust.

No capítulo 7, você irá aprender sobre o sistema de módulos de Rust e sobre
regras de privacidade para organizar seu código e sua Interface de Programação
de Aplicações (API) pública. O capítulo 8 discute algumas estruturas de dados
de coleções comuns que a biblioteca-padrão provê, tais como vetores, strings,
e hash maps. O capítulo 9 explora a filosofia de tratamento de erros de Rust
e suas técnicas.

O capitulo 10 mergulha em programação genérica, traits, e ciclos de vida
(lifetimes), os quais te proporcionam o poder de definir código que se aplique
a múltiplos tipos. O capítulo 11 é todo sobre testes, os quais mesmo com as
garantias de segurança de Rust são necessários para garantir que a lógica do
seu programa esteja correta. No capítulo 12, nós iremos construir nossa própria
implementação de um subconjunto de funcionalidades da ferramenta de linha de
comando`grep`, que busca por texto em arquivos. Para isso, vamos usar muitos
dos conceitos que discutimos nos capítulos anteriores.

<!-- Chapter 13 explores closures and iterators: features of Rust that come from
functional programming languages. In Chapter 14, we’ll examine Cargo in more
depth and talk about best practices for sharing your libraries with others.
Chapter 15 discusses smart pointers that the standard library provides and the
traits that enable their functionality.

In Chapter 16, we’ll walk through different models of concurrent programming
and talk about how Rust helps you to program in multiple threads fearlessly.
Chapter 17 looks at how Rust idioms compare to object-oriented programming
principles you might be familiar with.

Chapter 18 is a reference on patterns and pattern matching, which are powerful
ways of expressing ideas throughout Rust programs. Chapter 19 contains a
smorgasbord of advanced topics of interest, including unsafe Rust, macros, and
more about lifetimes, traits, types, functions, and closures. -->

O capítulo 13 explora as closures e iteradores: features de Rust que vêm de
linguagens de programação funcionais. No capítulo 14, nós vamos examinar o
Cargo em mais profundidade e falar sobre as melhores práticas para compartilhar
suas bibliotecas com outros. O capítulo 15 discute ponteiros inteligentes
(smart pointers) que a biblioteca-padrão provê e as traits que habilitam sua
funcionalidade.

No capítulo 16, nós iremos caminhar pelos diferentes modelos de programação
concorrente e falar sobre como Rust te ajuda a programar em múltiplas threads
sem medo. O capítulo 17 olha para como dialetos de Rust se comparam com os
princípios da programação orientada a objetos com os quais você possa estar
familiarizado.

O capítulo 18 é uma referência sobre padrões e pattern matching, que são formas
poderosas de expressar ideias em programas Rust. O capítulo 19 contém um
apanhado de tópicos avançados de interesse, incluindo Rust inseguro (unsafe),
macros, e mais sobre ciclos de vida, traits, tipos, funções, e closures.

<!-- In Chapter 20, we’ll complete a project in which we’ll implement a low-level
multithreaded web server!

Finally, some appendixes contain useful information about the language in a
more reference-like format. Appendix A covers Rust’s keywords, Appendix B
covers Rust’s operators and symbols, Appendix C covers derivable traits
provided by the standard library, Appendix D covers some useful development
tools, and Appendix E explains Rust editions.

There is no wrong way to read this book: if you want to skip ahead, go for it!
You might have to jump back to earlier chapters if you experience any
confusion. But do whatever works for you. -->

No capítulo 20, nós iremos completar um projeto no qual vamos implementar um
servidor web de baixo nível de múltiplas threads!

Finalmente, alguns apêndices contêm informações úteis sobre a linguagem num
formato mais de referência. O apêndice A cobre as palavras-chave (keywords)
de Rust, o apêndice B sobre os operadores e símbolos de Rust, o apêndice C
cobre traits deriváveis providas pela biblioteca-padrão, o apêndice D cobre
algumas ferramentas de desenvolvimento úteis, e o apêndice E explica as edições
de Rust.

Não há uma forma errada de ler este livro: se você quiser pular à frente, faça
isso! Você pode ter que voltar a capítulos anteriores se você experimentar
alguma confusão. Mas faça o que funcionar para você.

<!-- <span id="ferris"></span>

An important part of the process of learning Rust is learning how to read the
error messages the compiler displays: these will guide you toward working code.
As such, we’ll provide many examples that don’t compile along with the error
message the compiler will show you in each situation. Know that if you enter
and run a random example, it may not compile! Make sure you read the
surrounding text to see whether the example you’re trying to run is meant to
error. Ferris will also help you distinguish code that isn’t meant to work:

| Ferris                                                                  | Meaning                                          |
| ----------------------------------------------------------------------- | ------------------------------------------------ |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain"/>     | This code does not compile!                      |
| <img src="img/ferris/panics.svg" class="ferris-explain"/>               | This code panics!                                |
| <img src="img/ferris/unsafe.svg" class="ferris-explain"/>               | This code block contains unsafe code.            |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain"/> | This code does not produce the desired behavior. |

In most situations, we’ll lead you to the correct version of any code that
doesn’t compile. -->

<span id="ferris"></span>

Uma parte importante do processo de aprender a linguagem Rust é aprender como
ler as mensagens de erro que o compilador mostra: elas irão guiar você para
um código que funcione. Como tal, nós iremos providenciar muitos exemplos que
não compilam junto com a mensagem de erro que o compilador irá mostrar em cada
situação. Saiba que se você digitar e rodar um exemplo qualquer, ele pode não
compilar! Certifique-se de ler o texto ao redor para saber se o exemplo que
você está tentando rodar deve ter algum erro. O Ferris também irá te ajudar a
distinguir código que não deve funcionar:

| Ferris                                                                  | Meaning                                          |
| ----------------------------------------------------------------------- | ------------------------------------------------ |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain"/>     | Este código não compila!                         |
| <img src="img/ferris/panics.svg" class="ferris-explain"/>               | Este código gera panic!                          |
| <img src="img/ferris/unsafe.svg" class="ferris-explain"/>               | Este código contém código inseguro.              |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain"/> | Este código não produz o comportamento desejado. |

Na maior parte das situações, nós iremos te levar à versão correta do código
que não compila.

<!--
## Source Code

The source files from which this book is generated can be found on
[GitHub][book].

[book]: https://github.com/rust-lang/book/tree/master/src
 -->

## Código-fonte

Os arquivos-fonte a partir dos quais este livro é gerado podem ser encontrados
no [Github][book].

[book]: https://github.com/rust-lang/book/tree/master/src
