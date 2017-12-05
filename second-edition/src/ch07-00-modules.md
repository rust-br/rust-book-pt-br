# Using Modules to Reuse and Organize Code
# Usando Módulos para Reutilizar e Organizar Código

When you start writing programs in Rust, your code might live solely in the
`main` function. As your code grows, you’ll eventually move functionality into
other functions for reuse and better organization. By splitting your code into
smaller chunks, each chunk is easier to understand on its own. But what happens
if you have too many functions? Rust has a module system that enables the reuse
of code in an organized fashion.

Quando você começa a escrever programas em Rust, seu código pode viver apenas no
função `main`. À medida que seu código cresce, você acabará por mover a funcionalidade para
outras funções para reutilização e melhor organização. Ao dividir seu código em
pequenos pedaços, cada pedaço é mais fácil de entender por conta própria. Mas o que acontece
se você tem muitas funções? Rust possui um sistema de módulos que permite a reutilização
de código de forma organizada.

In the same way that you extract lines of code into a function, you can extract
functions (and other code, like structs and enums) into different modules. A
*module* is a namespace that contains definitions of functions or types, and
you can choose whether those definitions are visible outside their module
(public) or not (private). Here’s an overview of how modules work:

Da mesma forma que você extrai linhas de código em uma função, você pode extrair
funções (e outros códigos, como structs e enums) em diferentes módulos. Um
*module* é um namespace que contém definições de funções ou tipos, e
você pode escolher se essas definições são visíveis fora de seu módulo
(public) ou não (private). Aqui está uma visão geral de como os módulos funcionam:

* The `mod` keyword declares a new module. Code within the module appears
  either immediately following this declaration within curly brackets or in
  another file.
* By default, functions, types, constants, and modules are private. The `pub`
  keyword makes an item public and therefore visible outside its namespace.
* The `use` keyword brings modules, or the definitions inside modules, into
  scope so it’s easier to refer to them.

* A palavra-chave `mod` declara um novo módulo. O código dentro do módulo aparece
   imediatamente após esta declaração dentro de colchetes ou em
   outro arquivo.
* Por padrão, as funções, tipos, constantes e módulos são privados. A palavra-chave `pub`
    torna um item público e, portanto, é visível fora do seu namespace.
* A palavra-chave `use` traz módulos, ou as definições dentro dos módulos, ao
   escopo, por isso é mais fácil se referir a eles.

We’ll look at each of these parts to see how they fit into the whole.

Examinaremos cada uma dessas partes para ver como elas se encaixam no todo.
