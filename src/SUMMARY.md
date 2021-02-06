# A Linguagem de Programação Rust

[🇧🇷 A linguagem de programação Rust](title-page.md)
[🇧🇷 Prefácio](foreword.md)
[🇧🇷 Introdução](ch00-00-introduction.md)

## Começando

- [🇧🇷 Começando](ch01-00-getting-started.md)
    - [🇧🇷 Instalação](ch01-01-installation.md)
    - [🇧🇷 Olá, Mundo!](ch01-02-hello-world.md)
    - [🇧🇷 Olá, Cargo!](ch01-03-hello-cargo.md)

- [🇧🇷 Jogo de Adivinhação](ch02-00-guessing-game-tutorial.md)

- [🇧🇷 Conceitos Comuns de Programação](ch03-00-common-programming-concepts.md)
    - [🇧🇷 Variáveis e Mutabilidade](ch03-01-variables-and-mutability.md)
    - [🇧🇷 Tipos de dados](ch03-02-data-types.md)
    - [🇧🇷 Funções](ch03-03-how-functions-work.md)
    - [🇧🇷 Comentários](ch03-04-comments.md)
    - [🇧🇷 Controle de fluxo](ch03-05-control-flow.md)

- [🇧🇷 Entendendo _Ownership_](ch04-00-understanding-ownership.md)
    - [🇧🇷 O Que É Ownership?](ch04-01-what-is-ownership.md)
    - [🇧🇷 Referências e _Borrowing_](ch04-02-references-and-borrowing.md)
    - [🇧🇷 _Slices_](ch04-03-slices.md)

- [🇧🇷 Usando Structs para Estruturar Dados Relacionados](ch05-00-structs.md)
    - [🇧🇷 Definindo e Instanciando Structs](ch05-01-defining-structs.md)
    - [🇧🇷 Um Exemplo de um Programa que usa Structs](ch05-02-example-structs.md)
    - [🇧🇷 Sintaxe do Método](ch05-03-method-syntax.md)

- [🇧🇷 Enums e Casamento de Padrões](ch06-00-enums.md)
    - [🇧🇷 Definindo uma Enum](ch06-01-defining-an-enum.md)
    - [🇧🇷 Operador `match`](ch06-02-match.md)
    - [🇧🇷 Controle de Fluxo Conciso com `if let`](ch06-03-if-let.md)

## Basic Rust Literacy

- [🇧🇷 Módulos](ch07-00-modules.md)
    - [🇧🇷 `mod` e o Sistema de Arquivos](ch07-01-mod-and-the-filesystem.md)
    - [🇧🇷 Controlando a Visibilidade com `pub`](ch07-02-controlling-visibility-with-pub.md)
    - [🇧🇷 Importando nomes com `use`](ch07-03-importing-names-with-use.md)

- [🇧🇷 Coleções comuns](ch08-00-common-collections.md)
    - [🇧🇷 Vetores](ch08-01-vectors.md)
    - [🇧🇷 Strings](ch08-02-strings.md)
    - [🇧🇷 Hash Maps](ch08-03-hash-maps.md)

- [🇧🇷 Tratamento de Erros](ch09-00-error-handling.md)
    - [🇧🇷 Erros Irrecuperáveis com `panic!`](ch09-01-unrecoverable-errors-with-panic.md)
    - [🇧🇷 Erros recuperáveis com `Result`](ch09-02-recoverable-errors-with-result.md)
    - [🇧🇷 Entrar em `panic!` ou Não Entrar em `panic!`](ch09-03-to-panic-or-not-to-panic.md)

- [🇧🇷 Tipos Genéricos, _Traits_, e Tempos de vida (_Lifetimes_)](ch10-00-generics.md)
    - [🇧🇷 Tipos Genéricos de Dados](ch10-01-syntax.md)
    - [🇧🇷 Traits: Definindo Comportamento Compartilhado](ch10-02-traits.md) - NEED DEFAULT METHOD IMPLEMENTATIONS
    - [🇧🇷 Validating References with Lifetimes](ch10-03-lifetime-syntax.md)

- [Testing](ch11-00-testing.md)
    - [Writing tests](ch11-01-writing-tests.md)
    - [Running tests](ch11-02-running-tests.md)
    - [Test Organization](ch11-03-test-organization.md)

- [🇧🇷 Um projeto de E/S](ch12-00-an-io-project.md)
    - [🇧🇷 Aceitando Argumentos em Linha de Comando](ch12-01-accepting-command-line-arguments.md)
    - [🇧🇷 Lendo um Arquivo](ch12-02-reading-a-file.md)
    - [🇧🇷 Refatoração para Melhorar a Modularidade e o Tratamento de Erros](ch12-03-improving-error-handling-and-modularity.md)
    - [🇧🇷 Desenvolvendo a Biblioteca de Funcionalidades](ch12-04-testing-the-librarys-functionality.md)
    - [🇧🇷 Trabalhando com Variáveis de Ambiente](ch12-05-working-with-environment-variables.md)
    - [🇧🇷 Escrevendo Mensagens de Erro para Erro Padrão em Vez de Saída Padrão](ch12-06-writing-to-stderr-instead-of-stdout.md)

## Pensando em Rust

- [Functional Language Features in Rust](ch13-00-functional-features.md)
    - [Closures](ch13-01-closures.md)
    - [Iterators](ch13-02-iterators.md)
    - [Improving our I/O Project](ch13-03-improving-our-io-project.md)
    - [Performance](ch13-04-performance.md)

- [More about Cargo and Crates.io](ch14-00-more-about-cargo.md)
    - [Release Profiles](ch14-01-release-profiles.md)
    - [Publishing a Crate to Crates.io](ch14-02-publishing-to-crates-io.md)
    - [Cargo Workspaces](ch14-03-cargo-workspaces.md)
    - [Installing Binaries from Crates.io with `cargo install`](ch14-04-installing-binaries.md)
    - [Extending Cargo with Custom Commands](ch14-05-extending-cargo.md)

- [🇧🇷 Ponteiros Inteligentes (Smart Pointers)](ch15-00-smart-pointers.md)
    - [🇧🇷 `Box<T>` Aponta para Dados no Heap e Tem Tamanho Conhecido](ch15-01-box.md)
    - [🇧🇷 Tratando Ponteiros Inteligentes como Referências Normais com a Trait `Deref`](ch15-02-deref.md)
    - [🇧🇷 A Trait `Drop` Roda Código durante a Limpeza](ch15-03-drop.md)
    - [🇧🇷 `Rc<T>`, o Ponteiro Inteligente com Contagem de Referências](ch15-04-rc.md)
    - [🇧🇷 `RefCell<T>` e a Pattern de Mutabilidade Interior](ch15-05-interior-mutability.md)
    - [🇧🇷 Ciclos de Referências Podem Vazar Memória](ch15-06-reference-cycles.md)

- [Concurrency](ch16-00-concurrency.md)

- [🇧🇷 Rust é uma linguagem orientada a objetos?](ch17-00-oop.md)
    - [🇧🇷 O que significa orientado a objetos?](ch17-01-oo.md)
    - [🇧🇷 Usando objetos trait que permitem valores de tipos diferentes](ch17-02-trait-objects.md)
    - [🇧🇷 Implementando um padrão de projeto orientado a objetos](ch17-03-oo-design-patterns.md)

## Tópicos avançados

- [Patterns](ch18-00-patterns.md) (perhaps an appendix?)

- [More Lifetimes](ch19-00-more-lifetimes.md) (perhaps merge this into advanced type system features?)

- [Advanced Type System Features](ch20-00-advanced-types.md) (perhaps called "Advanced Traits"?)

- [Appendix](appendix-00.md)
    - [Keywords](appendix-01-keywords.md)
    - [Operators](appendix-02-operators.md)
    - [Derivable Traits](appendix-03-derivable-traits.md)
    - [Nightly Rust](appendix-04-nightly-rust.md)
    - [Macros](appendix-05-macros.md)
    - [🇧🇷 G - Como é feito o Rust e o "Rust Nightly"](appendix-07-nightly-rust.md)
