# A Linguagem de Programação Rust

[🇧🇷A Linguagem de Programação Rust](title-page.md)
[🇧🇷Prefácio](foreword.md)
[Introdução](ch00-00-introduction.md)

## Começando

- [🇧🇷 Introdução](ch01-00-getting-started.md)
    - [🇧🇷 Instalação](ch01-01-installation.md)
    - [🇧🇷 Olá, Mundo!](ch01-02-hello-world.md)
    - [Olá, Cargo!](ch01-03-hello-cargo.md)

- [🇧🇷 Jogo de Adivinhação](ch02-00-guessing-game-tutorial.md)

- [🇧🇷 Conceitos Comuns de Programação](ch03-00-common-programming-concepts.md)
    - [🇧🇷 Variáveis e Mutabilidade](ch03-01-variables-and-mutability.md)
    - [🇧🇷 Tipos de Dados](ch03-02-data-types.md)
    - [🇧🇷 Funções](ch03-03-how-functions-work.md)
    - [🇧🇷 Comentários](ch03-04-comments.md)
    - [🇧🇷 Controle de Fluxo](ch03-05-control-flow.md)

- [🇧🇷 Entendendo _Ownership_](ch04-00-understanding-ownership.md)
    - [🇧🇷 O Que É Ownership?](ch04-01-what-is-ownership.md)
    - [🇧🇷 Referências e _Borrowing_](ch04-02-references-and-borrowing.md)
    - [🇧🇷 _Slices_](ch04-03-slices.md)

- [🇧🇷 Structs](ch05-00-structs.md)
    - [Definindo e Instanciando Structs](ch05-01-defining-structs.md)
    - [Um Exemplo do Uso de Structs](ch05-02-example-structs.md)
    - [Sintaxe de Métodos](ch05-03-method-syntax.md)

- [🇧🇷 Enums e Casamento de Padrões](ch06-00-enums.md)
    - [🇧🇷 Definindo uma Enum](ch06-01-defining-an-enum.md)
    - [🇧🇷 Operador `match`](ch06-02-match.md)
    - [🇧🇷 Controle de Fluxo Conciso com `if let`](ch06-03-if-let.md)

## Alfabetização Rust Básica

- [🇧🇷 Módulos](ch07-00-modules.md)
    - [OLD🇧🇷 `mod` e o Sistema de Arquivos](ch07-01-mod-and-the-filesystem.md)
    - [OLD🇧🇷 Controlando a Visibilidade com `pub`](ch07-02-controlling-visibility-with-pub.md)
    - [OLD🇧🇷 Importando nomes com `use`](ch07-03-importing-names-with-use.md)

    - [Packages and Crates](ch07-01-packages-and-crates.md)
    - [Defining Modules to Control Scope and Privacy](ch07-02-defining-modules-to-control-scope-and-privacy.md)
    - [Paths for Referring to an Item in the Module Tree](ch07-03-paths-for-referring-to-an-item-in-the-module-tree.md)
    - [Bringing Paths Into Scope with the `use` Keyword](ch07-04-bringing-paths-into-scope-with-the-use-keyword.md)
    - [Separating Modules into Different Files](ch07-05-separating-modules-into-different-files.md)


- [Fundamental Collections](ch08-00-fundamental-collections.md)
    - [🇧🇷 Vetores](ch08-01-vectors.md)
    - [🇧🇷 Strings](ch08-02-strings.md)
    - [🇧🇷 Hash Maps](ch08-03-hash-maps.md)

- [🇧🇷 Tratamento de Erros](ch09-00-error-handling.md)
    - [🇧🇷 Erros Irrecuperáveis com `panic!`](ch09-01-unrecoverable-errors-with-panic.md)
    - [🇧🇷 Erros recuperáveis com `Result`](ch09-02-recoverable-errors-with-result.md)
    - [🇧🇷 Entrar em `panic!` ou Não Entrar em `panic!`](ch09-03-to-panic-or-not-to-panic.md)

- [🇧🇷 Tipos Genéricos, _Traits_, e Tempos de vida (_Lifetimes_)](ch10-00-generics.md)
    - [🇧🇷 Tipos Genéricos de Dados](ch10-01-syntax.md)
    - [🇧🇷 Traits: Definindo Comportamento Compartilhado](ch10-02-traits.md)
    - [🇧🇷 Validating References with Lifetimes](ch10-03-lifetime-syntax.md)

- [Writing Automated Tests](ch11-00-testing.md)
    - [How to Write Tests](ch11-01-writing-tests.md)
    - [Controlling How Tests Are Run](ch11-02-running-tests.md)
    - [Test Organization](ch11-03-test-organization.md)

- [An I/O Project: Building a Command Line Program](ch12-00-an-io-project.md)
    - [Accepting Command Line Arguments](ch12-01-accepting-command-line-arguments.md)
    - [Reading a File](ch12-02-reading-a-file.md)
    - [Refactoring to Improve Modularity and Error Handling](ch12-03-improving-error-handling-and-modularity.md)
    - [Developing the Library’s Functionality with Test Driven Development](ch12-04-testing-the-librarys-functionality.md)
    - [Working with Environment Variables](ch12-05-working-with-environment-variables.md)
    - [Writing Error Messages to Standard Error Instead of Standard Output](ch12-06-writing-to-stderr-instead-of-stdout.md)

## Thinking in Rust

- [Functional Language Features: Iterators and Closures](ch13-00-functional-features.md)
    - [Closures: Anonymous Functions that Can Capture Their Environment](ch13-01-closures.md)
    - [Processing a Series of Items with Iterators](ch13-02-iterators.md)
    - [Improving Our I/O Project](ch13-03-improving-our-io-project.md)
    - [Comparing Performance: Loops vs. Iterators](ch13-04-performance.md)

- [More about Cargo and Crates.io](ch14-00-more-about-cargo.md)
    - [Customizing Builds with Release Profiles](ch14-01-release-profiles.md)
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

- [Concorrência sem Medo](ch16-00-concurrency.md)
    - [Usando Threads para Executar Código Simultaneamente](ch16-01-threads.md)
    - [Usando Troca de Mensagens para Transferir Dados entre Threads](ch16-02-message-passing.md)
    - [Concorrência com Estado Compartilhado](ch16-03-shared-state.md)
    - [Concorrência Extensível com as Trais `Sync` e `Send`](ch16-04-extensible-concurrency-sync-and-send.md)

- [🇧🇷 Rust é uma Linguagem Orientada a Objetos?](ch17-00-oop.md)
    - [🇧🇷 O que Significa Orientado a Objetos?](ch17-01-what-is-oo.md)
    - [🇧🇷 Usando Objetos Trait que Permitem Valores de Tipos Diferentes](ch17-02-trait-objects.md)
    - [🇧🇷 Implementando um Padrão de Projeto Orientado a Objetos](ch17-03-oo-design-patterns.md)

## Tópicos Avançados

- [Padrões e Casamento (Pattern Matching)](ch18-00-patterns.md)
    - [Todos Lugares Onde Padrões Podem Ser Usados](ch18-01-all-the-places-for-patterns.md)
    - [Refutabilidade: Se um Padrão Pode ou Não Falhar no Match](ch18-02-refutability.md)
    - [Sintaxe dos Padrões](ch18-03-pattern-syntax.md)

- [Funcionalidades Avançadas](ch19-00-advanced-features.md)
    - [Rust Inseguro](ch19-01-unsafe-rust.md)
    - [Traits Avançados](ch19-03-advanced-traits.md)
    - [Tipos Avançados](ch19-04-advanced-types.md)
    - [Funções e Closures Avançadas](ch19-05-advanced-functions-and-closures.md)
    - [Macros](ch19-06-macros.md)

- [Projeto Final: Construindo um Servidor Web Multi Thread](ch20-00-final-project-a-web-server.md)
    - [Criando um Servidor Web de uma Thread](ch20-01-single-threaded.md)
    - [Transformando Nosso Servidor de Uma Thread em um Multi Thread](ch20-02-multithreaded.md)
    - [Desligamento e Limpeza sem Problemas](ch20-03-graceful-shutdown-and-cleanup.md)

- [🇧🇷Apêndices](appendix-00.md)
    - [A - Palavras chave](appendix-01-keywords.md)
    - [B - Operadores e Símbolos](appendix-02-operators.md)
    - [C - Traits Deriváveis](appendix-03-derivable-traits.md)
    - [D - Ferramentas Úteis para Desenvolvimento](appendix-04-useful-development-tools.md)
    - [E - Edições](appendix-05-editions.md)
    - [🇧🇷 F - Traduções do Livro](appendix-06-translation.md)
    - [G - Como Rust é Feito e “Nightly Rust”](appendix-07-nightly-rust.md)
