# Functional Language Features: Iterators and Closures
# Recursos Funcionais da Linguagem: Iteradores e Closures

Rust’s design has taken inspiration from many existing languages and
techniques, and one significant influence is *functional programming*.
Programming in a functional style often includes using functions as values by
passing them in arguments, returning them from other functions, assigning them
to variables for later execution, and so forth.

O design de Rust se inspirou em muitas linguagens e técnicas existentes, e uma 
influência significativa é a *programação funcional*. A programação em um estilo 
funcional geralmente inclui o uso de funções como valores, passando-os em argumentos, 
retornando-os de outras funções, atribuindo-os a variáveis para execução posterior, 
e assim por diante.

In this chapter, we won’t debate the issue of what functional programming is or
isn’t but will instead discuss some features of Rust that are similar to
features in many languages often referred to as functional.

Neste capítulo, não discutiremos a questão do que é ou não a programação funcional, 
mas discutiremos alguns recursos de Rust que são semelhantes aos recursos em muitas 
linguagens frequentemente denominadas funcionais.

More specifically, we’ll cover:

* *Closures*, a function-like construct you can store in a variable
* *Iterators*, a way of processing a series of elements
* How to use these two features to improve the I/O project in Chapter 12
* The performance of these two features (Spoiler alert: they’re faster than you
  might think!)

Mais especificamente, abordaremos:

* *Closures*, uma construção semelhante a uma função que você pode armazenar em uma variável
* *Iteradores*, uma maneira de processar uma série de elementos
* Como usar esses dois recursos para melhorar o projeto de E/S (I/O) no Capítulo 12
* O desempenho desses dois recursos (alerta de spoiler: eles são mais rápidos do que você imagina!)  

Other Rust features, such as pattern matching and enums, which we’ve covered in
other chapters, are influenced by the functional style as well. Mastering
closures and iterators is an important part of writing idiomatic, fast Rust
code, so we’ll devote this entire chapter to them.

Outros recursos de Rust, como pattern matching (correspondência de padrões) e enums (enumerações), 
que abordamos em outros capítulos, também são influenciados pelo estilo funcional. O domínio de 
closures e iteradores é uma parte importante da escrita de código rápido Rust e idiomático; 
portanto, dedicaremos todo esse capítulo a eles.