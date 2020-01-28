<!--
## Comments

All programmers strive to make their code easy to understand, but sometimes
extra explanation is warranted. In these cases, programmers leave notes, or
*comments*, in their source code that the compiler will ignore but people
reading the source code may find useful.
-->

## Comentários

Todos os programadores se esforçam para tornar seu código fácil de entender, mas às vezes
explicação extra é garantida. Nestes casos, os programadores deixam notas ou
*comentários*, em seus códigos fonte que o compilador irá ignorar, mas as pessoas que
lerem o código-fonte podem achar útil.

<!--
Here’s a simple comment:

```rust
// hello, world
```
-->

Aqui está um comentário simples:

```rust
// olá, mundo
```

<!--
In Rust, comments must start with two slashes and continue until the end of the
line. For comments that extend beyond a single line, you’ll need to include
`//` on each line, like this:

```rust
// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.
```
-->

Em Rust, os comentários devem começar com duas barras e continuar até o final da
linha. Para comentários que se estendem além de uma única linha, você precisará incluir
`//` em cada linha, assim:

```rust
// Então, estamos fazendo algo complicado aqui, tempo suficiente para que precisemos
// várias linhas de comentários para fazer isso! Ufa! Espero que este comentário
// explique o que está acontecendo.
```

<!--
Comments can also be placed at the end of lines containing code:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```
-->

Comentários também podem ser colocados no final das linhas contendo código:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let numero_da_sorte = 7; // Estou com sorte hoje.
}
```

<!--
But you’ll more often see them used in this format, with the comment on a
separate line above the code it’s annotating:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```
-->

Mas você verá com mais frequência essas palavras nesse formato, com o comentário em uma
linha separada acima do código que está anotando:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    // Estou com sorte hoje.
    let numero_da_sorte = 7;
}
```

<!--
Rust also has another kind of comment, documentation comments, which we’ll
discuss in the “Publishing a Crate to Crates.io” section of Chapter 14.
-->

Rust também tem outro tipo de comentário, comentários de documentação, que discutiremos
no Capítulo 14.
