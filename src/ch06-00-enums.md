<!--
# Enums and Pattern Matching
-->

# Enums e Casamento de Padrões (_Pattern Matching_)

<!--
In this chapter we’ll look at *enumerations*, also referred to as *enums*.
Enums allow you to define a type by enumerating its possible *variants*. First,
we’ll define and use an enum to show how an enum can encode meaning along with
data. Next, we’ll explore a particularly useful enum, called `Option`, which
expresses that a value can be either something or nothing. Then we’ll look at
how pattern matching in the `match` expression makes it easy to run different
code for different values of an enum. Finally, we’ll cover how the `if let`
construct is another convenient and concise idiom available to you to handle
enums in your code.
-->

Neste capítulo vamos ver *enumerações*, também chamadas de *enums*. Enums
permitem definir um tipo por meio da enumeração de seus possíveis valores.
Primeiro, vamos definir e usar uma enum para mostrar como ela pode atrelar
significado aos nossos dados. Depois, vamos explorar uma enum particularmente
útil, chamada `Option`, que expressa um valor que tanto pode ser algo quanto
pode não ser nada. Em seguida, vamos ver como o casamento de padrões por meio
da expressão `match` facilita a execução de códigos diferentes para diferentes
valores de uma enum. Por fim, vamos abordar o `if let`, outra forma concisa e
conveniente que você pode usar para tratar enums no seu código.

<!--
Enums are a feature in many languages, but their capabilities differ in each
language. Rust’s enums are most similar to *algebraic data types* in functional
languages, such as F#, OCaml, and Haskell.
-->

Enums são ferramentas que aparecem em muitas linguagens, mas suas
características variam de uma para outra. Em Rust, enums são mais parecidas com
os *tipos de dados algébricos* das linguagens de programação funcional como F#,
OCaml e Haskell.
