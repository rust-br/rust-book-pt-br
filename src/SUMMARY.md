# A Linguagem de Programação Rust

[🇧🇷 A Linguagem de Programação Rust](title-page.md)
[🇧🇷 Prefácio](foreword.md)
[Introdução](ch00-00-introduction.md)

## Começando

- [🇧🇷 Iniciando](ch01-00-introduction.md) <!-- TODO mover para ch01-00-getting-started.md -->
    - [🇧🇷 Instalação](ch01-01-installation.md)
    - [🇧🇷 Olá, Mundo!](ch01-02-hello-world.md)
    - [Olá, Cargo](ch01-03-hello-cargo.md)

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
    - [🇧🇷 O Tipo _Slice_](ch04-03-slices.md)

- [🇧🇷 Usando _Structs_ para Estruturar Dados Relacionados](ch05-00-structs.md)
    - [Definindo e Instanciando _Structs_](ch05-01-defining-structs.md)
    - [Um Exemplo de Programa Usando _Structs_](ch05-02-example-structs.md)
    - [Sintaxe de Métodos](ch05-03-method-syntax.md)

- [🇧🇷 Enums e Casamento de Padrões](ch06-00-enums.md)
    - [🇧🇷 Definindo uma Enum](ch06-01-defining-an-enum.md)
    - [🇧🇷 Operador `match`](ch06-02-match.md)
    - [🇧🇷 Controle de Fluxo Conciso com `if let`](ch06-03-if-let.md)

## Basic Rust Literacy

- [🇧🇷 Módulos](ch07-00-modules.md)
    - [🇧🇷 `mod` e o Sistema de Arquivos](ch07-01-mod-and-the-filesystem.md)
    - [🇧🇷 Controlando a Visibilidade com `pub`](ch07-02-controlling-visibility-with-pub.md)
    - [🇧🇷 Importando nomes com `use`](ch07-03-importing-names-with-use.md)
<!-- Novo capítulo de Módulos
- [Managing Growing Projects with Packages, Crates, and Modules](ch07-00-managing-growing-projects-with-packages-crates-and-modules.md)
    - [Packages and Crates](ch07-01-packages-and-crates.md)
    - [Defining Modules to Control Scope and Privacy](ch07-02-defining-modules-to-control-scope-and-privacy.md)
    - [Paths for Referring to an Item in the Module Tree](ch07-03-paths-for-referring-to-an-item-in-the-module-tree.md)
    - [Bringing Paths Into Scope with the `use` Keyword](ch07-04-bringing-paths-into-scope-with-the-use-keyword.md)
    - [Separating Modules into Different Files](ch07-05-separating-modules-into-different-files.md)
-->

- [Coleções Fundamentais](ch08-00-fundamental-collections.md)
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
    - [🇧🇷 Validando Referências com Tempos de Vida](ch10-03-lifetime-syntax.md)

- [Escrevendo Testes Automatizados](ch11-00-testing.md)
    - [Como Escrever Testes](ch11-01-writing-tests.md)
    - [Controlando como Testes são Executados](ch11-02-running-tests.md)
    - [Organização dos Testes](ch11-03-test-organization.md)

- [🇧🇷 Um projeto de E/S: Criando um Programa de Linha de Comando](ch12-00-an-io-project.md)
    - [🇧🇷 Recebendo Argumentos de Linha de Comando](ch12-01-accepting-command-line-arguments.md)
    - [🇧🇷 Lendo um Arquivo](ch12-02-reading-a-file.md)
    - [🇧🇷 Refatoração para Melhorar a Modularidade e o Tratamento de Erros](ch12-03-improving-error-handling-and-modularity.md)
    - [🇧🇷 Desenvolvendo a Biblioteca de Funcionalidades com Desenvolvimento Guiado por Testes](ch12-04-testing-the-librarys-functionality.md)
    - [🇧🇷 Trabalhando com Variáveis de Ambiente](ch12-05-working-with-environment-variables.md)
    - [🇧🇷 Escrevendo para `stderr` ao Invés de `stdout`](ch12-06-writing-to-stderr-instead-of-stdout.md)

## Thinking in Rust

- [Programação Funcional: Iteradores e _Closures_](ch13-00-functional-features.md)
    - [_Closures_: Funções Anônimas que Capturam Seus Ambientes](ch13-01-closures.md)
    - [Processando uma Série de Itens com Iteradores](ch13-02-iterators.md)
    - [Melhorando Nosso Projeto de E/S](ch13-03-improving-our-io-project.md)
    - [Comparando Performance: Loops e Iteradores](ch13-04-performance.md)

- [Mais Sobre o Cargo e o Crates.io](ch14-00-more-about-cargo.md)
    - [Customizando _Builds_ com _Release Profiles_](ch14-01-release-profiles.md)
    - [Publicando uma Crate no Crates.io](ch14-02-publishing-to-crates-io.md)
    - [Workspaces no Cargo](ch14-03-cargo-workspaces.md)
    - [Instalando Binários do Crates.io com o `cargo install`](ch14-04-installing-binaries.md)
    - [Estendendo o Cargo com Comandos Customizados](ch14-05-extending-cargo.md)

- [🇧🇷 Ponteiros Inteligentes (Smart Pointers)](ch15-00-smart-pointers.md)
    - [🇧🇷 `Box<T>` Aponta para Dados no Heap e Tem Tamanho Conhecido](ch15-01-box.md)
    - [🇧🇷 Tratando Ponteiros Inteligentes como Referências Normais com a Trait `Deref`](ch15-02-deref.md)
    - [🇧🇷 A Trait `Drop` Roda Código durante a Limpeza](ch15-03-drop.md)
    - [🇧🇷 `Rc<T>`, o Ponteiro Inteligente com Contagem de Referências](ch15-04-rc.md)
    - [🇧🇷 `RefCell<T>` e a Pattern de Mutabilidade Interior](ch15-05-interior-mutability.md)
    - [🇧🇷 Ciclos de Referências Podem Vazar Memória](ch15-06-reference-cycles.md)

- [Concorrência Sem Medo](ch16-00-concurrency.md)
    - [Usando _Threads_ para Executar Código Simultaneamente](ch16-01-threads.md)
    - [Usando Passagem de Mensagens para Transferir Dados entre _Threads_](ch16-02-message-passing.md)
    - [Concorrência via Estado Compartilhado](ch16-03-shared-state.md)
    - [Concorrência Extensível com as _Traits_ `Sync` e `Send`](ch16-04-extensible-concurrency-sync-and-send.md)

- [🇧🇷 Rust é uma linguagem orientada a objetos?](ch17-00-oop.md)
    - [🇧🇷 O que significa orientado a objetos?](ch17-01-what-is-oo.md)
    - [🇧🇷 Usando objetos trait que permitem valores de tipos diferentes](ch17-02-trait-objects.md)
    - [🇧🇷 Implementando um padrão de projeto orientado a objetos](ch17-03-oo-design-patterns.md)

## Advanced Topics

- [_Patterns_ e _Matching_](ch18-00-patterns.md)
    - [Todos Lugares onde _Patterns_ podem ser usadas](ch18-01-all-the-places-for-patterns.md)
    - [Refutabilidade: se uma _Pattern_ pode Falhar ou Não](ch18-02-refutability.md)
    - [Sintaxe de _Patterns_](ch18-03-pattern-syntax.md)

- [Funcionalidades Avançadas](ch19-00-advanced-features.md)
    - [Rust Inseguro](ch19-01-unsafe-rust.md)
    - [Traits Avançados](ch19-03-advanced-traits.md)
    - [Tipos Avançados](ch19-04-advanced-types.md)
    - [Funções e _Closures_ Avançadas](ch19-05-advanced-functions-and-closures.md)
    - [Macros](ch19-06-macros.md)

- [Projeto Final: Um Servidor Web Multi-Thread](ch20-00-final-project-a-web-server.md)
    - [Um Servidor Web Mono-Thread](ch20-01-single-threaded.md)
    - [Transformando Nosso Servidor Mono-Thread num Servidor Multi-Thread](ch20-02-multithreaded.md)
    - [Graceful Shutdown and Cleanup](ch20-03-graceful-shutdown-and-cleanup.md)

- [Apêndices](appendix-00.md)
    - [A - Palavras Chave](appendix-01-keywords.md)
    - [B - Operadores e Símbolos](appendix-02-operators.md)
    - [C - Traits Deriváveis](appendix-03-derivable-traits.md)
    - [D - Ferramentas Úteis no Desenvolvimento](appendix-04-useful-development-tools.md)
    - [E - Edições](appendix-05-editions.md)
    - [F - Traduções do Livro](appendix-06-translation.md)
    - [G - Como Rust é Feito e o “Rust Nightly”](appendix-07-nightly-rust.md)
