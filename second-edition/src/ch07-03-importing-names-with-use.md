## Referring to Names in Different Modules
## Referindo-se a Nomes em Módulos Diferentes

We’ve covered how to call functions defined within a module using the module
name as part of the call, as in the call to the `nested_modules` function shown
here in Listing 7-7:

Cobrimos como chamar funções definidas dentro de um módulo usando o nome do módulo
como parte da chamada, como na chamada para a função `nested_modules` mostrada
aqui na Listagem 7-7:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

fn main() {
    a::series::of::nested_modules();
}
```

<span class="caption">Listing 7-7: Calling a function by fully specifying its
enclosing module’s path</span>

<span class="caption">Listagem 7-7: Chamando uma função especificando completamente 
o caminho do módulo que a cerca</span>


As you can see, referring to the fully qualified name can get quite lengthy.
Fortunately, Rust has a keyword to make these calls more concise.

Como você pode ver, referir-se ao nome totalmente qualificado pode ficar bastante longo.
Felizmente, Rust tem uma palavra-chave para tornar estas chamadas mais concisas.

### Bringing Names into Scope with the `use` Keyword
### Trazendo Nomes no Escopo com a Palavra-Chave `use`

Rust’s `use` keyword shortens lengthy function calls by bringing the modules of
the function you want to call into scope. Here’s an example of bringing the
`a::series::of` module into a binary crate’s root scope:

A palavra-chave `use` de Rust encurta as chamadas de função longas, trazendo os módulos e
a função que deseja chamar para o escopo. Aqui está um exemplo de trazer o
módulo `a::series::of` no escopo da raiz de uma crate binária:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of;

fn main() {
    of::nested_modules();
}
```

The line `use a::series::of;` means that rather than using the full
`a::series::of` path wherever we want to refer to the `of` module, we can use
`of`.

A linha `use a::series::of;` significa que, ao invés de usar todo o caminho
`a::series::of` onde queremos referir-se ao módulo `of`, podemos usar
`of`.

The `use` keyword brings only what we’ve specified into scope: it does not
bring children of modules into scope. That’s why we still have to use
`of::nested_modules` when we want to call the `nested_modules` function.

A palavra-chave `use` traz apenas o que especificamos no escopo: ela não
levar os filhos dos módulos ao escopo. É por isso que ainda temos que usar
`of::nested_modules` quando queremos chamar a função `nested_modules`.

We could have chosen to bring the function into scope by instead specifying the
function in the `use` as follows:

Poderíamos ter escolhido trazer a função para o escopo, em vez de especificar a função
no `use` da seguinte forma:

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of::nested_modules;

fn main() {
    nested_modules();
}
```

Doing so allows us to exclude all the modules and reference the function
directly.

Isso nos permite excluir todos os módulos e fazer referência à função
diretamente.

Because enums also form a sort of namespace like modules, we can bring an
enum’s variants into scope with `use` as well. For any kind of `use` statement,
if you’re bringing multiple items from one namespace into scope, you can list
them using curly brackets and commas in the last position, like so:


Porque enums também formam uma espécie de namespace como módulos, podemos trazer
as variáveis de enum para o escopo com `use` também. Para qualquer tipo de declaração de `use`
se você estiver trazendo vários itens de um namespace para o escopo, você pode listá-los
usando chaves e vírgulas na última posição, assim:

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::{Red, Yellow};

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = TrafficLight::Green;
}
```

We’re still specifying the `TrafficLight` namespace for the `Green` variant
because we didn’t include `Green` in the `use` statement.

Nós ainda estamos especificando o namespace `TrafficLight` para a variante `Green`
porque não incluímos `Green` na declaração `use`.

### Bringing All Names into Scope with a Glob
### Trazendo Todos os Nomes para o Escopo com um Glob

To bring all the items in a namespace into scope at once, we can use the `*`  syntax, which is called the *glob operator*. This example brings all the variants of an enum into scope without having to list each specifically:

Para trazer todos os itens de um namespace para o escopo ao mesmo tempo, podemos usar a sintaxe `*`, que é chamada de *operador glob*. Este exemplo traz todas as variantes de um enum ao escopo sem ter que listar cada uma especificamente:

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::*;

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = Green;
}
```

The `*` will bring into scope all the visible items in the `TrafficLight`
namespace. You should use globs sparingly: they are convenient, but this might
also pull in more items than you expected and cause naming conflicts.

O `*` trará para o escopo todos os itens visíveis no namespace `TrafficLight`. 
Você deve usar globs com moderação: eles são convenientes, mas isso pode
também trazer mais itens do que se esperava e causar conflitos de nomeação.

### Using `super` to Access a Parent Module
### Usando `super` para Acessar um Módulo Pai

As we saw at the beginning of this chapter, when you create a library crate,
Cargo makes a `tests` module for you. Let’s go into more detail about that now.
In your `communicator` project, open *src/lib.rs*:

Como vimos no início deste capítulo, quando você cria uma crate de biblioteca,
o Cargo faz um módulo de `tests` para você. Vamos ver isso em mais detalhes agora.
No seu projeto `communicator`, abra *src/lib.rs*:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;

#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Chapter 11 explains more about testing, but parts of this example should make
sense now: we have a module named `tests` that lives next to our other modules
and contains one function named `it_works`. Even though there are special
annotations, the `tests` module is just another module! So our module hierarchy
looks like this:

O Capítulo 11 explica mais sobre testes, mas algumas partes deste exemplo devem fazer
sentido agora: temos um módulo chamado `tests` que vive ao lado de nossos outros módulos
e contém uma função chamada `it_works`. Embora existam anotações especiais, 
o módulo `tests` é apenas outro módulo! Então nossa hierarquia de módulos
se parece com isso:

```text
communicator
 ├── client
 ├── network
 |   └── client
 └── tests
```

Tests are for exercising the code within our library, so let’s try to call our
`client::connect` function from this `it_works` function, even though we won’t
be checking any functionality right now. This won't work yet:

Os testes são para o exercício do código dentro da nossa biblioteca, então vamos tentar chamar nossa
A função `client :: connect` desta função` it_works`, mesmo que não possamos
esteja a verificar qualquer funcionalidade agora. Isso ainda não funcionará:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        client::connect();
    }
}
```

Run the tests by invoking the `cargo test` command:

Execute os testes invocando o comando `cargo test`:

```text
$ cargo test
   Compiling communicator v0.1.0 (file:///projects/communicator)
error[E0433]: failed to resolve. Use of undeclared type or module `client`
 --> src/lib.rs:9:9
  |
9 |         client::connect();
  |         ^^^^^^ Use of undeclared type or module `client`
```

The compilation failed, but why? We don’t need to place `communicator::` in
front of the function like we did in *src/main.rs* because we are definitely
within the `communicator` library crate here. The reason is that paths are
always relative to the current module, which here is `tests`. The only
exception is in a `use` statement, where paths are relative to the crate root
by default. Our `tests` module needs the `client` module in its scope!

A compilação falhou, mas por quê? Não precisamos colocar `communicator ::` em
frente da função como fizemos em *src/main.rs* porque estamos definitivamente
dentro da crate de biblioteca `communicator` aqui. A razão é que os caminhos são
sempre relativos ao módulo atual, que aqui é `tests`. A única
exceção está em uma instrução `use`, onde os caminhos são relativos à crate raiz
por padrão. Nosso módulo `tests` precisa do módulo `client` no seu escopo!

So how do we get back up one module in the module hierarchy to call the
`client::connect` function in the `tests` module? In the `tests` module, we can
either use leading colons to let Rust know that we want to start from the root
and list the whole path, like this:

Então, como podemos retroceder de um módulo na hierarquia de módulo para chamar o
função `client::connect` no módulo `tests`? No módulo `tests`, temos a opção de:
usar dois pontos para deixar Rust saber que queremos começar a partir da raiz
e listar todo o caminho, assim:

```rust,ignore
::client::connect();
```

Or, we can use `super` to move up one module in the hierarchy from our current
module, like this:

Ou, podemos usar `super` para mover para cima um módulo na hierarquia de nosso 
módulo atual, assim:

```rust,ignore
super::client::connect();
```

These two options don’t look that different in this example, but if you’re
deeper in a module hierarchy, starting from the root every time would make your
code lengthy. In those cases, using `super` to get from the current module to
sibling modules is a good shortcut. Plus, if you’ve specified the path from the
root in many places in your code and then you rearrange your modules by moving
a subtree to another place, you’d end up needing to update the path in several
places, which would be tedious.

Essas duas opções não parecem tão diferentes neste exemplo, mas se você for
mais a fundo em uma hierarquia de módulos, começando a partir da raiz, isso faria seu
código longo. Nesses casos, usando `super` para partir do módulo atual para
módulos irmãos é um bom atalho. Além disso, se você especificou o caminho da
raiz em muitos lugares do seu código e depois reorganizando seus módulos movendo
uma sub-árvore para outro lugar, você acabaria precisando atualizar o caminho em vários
lugares, o que seria tedioso.

It would also be annoying to have to type `super::` in each test, but you’ve
already seen the tool for that solution: `use`! The `super::` functionality
changes the path you give to `use` so it is relative to the parent module
instead of to the root module.

Também seria irritante ter que digitar `super ::` em cada teste, mas você
já viu a ferramenta para essa solução: `use`! A funcionalidade `super ::`
altera o caminho que você dá para `use`, por isso é relativo ao módulo pai
em vez do módulo raiz.

For these reasons, in the `tests` module especially, `use super::something` is
usually the best solution. So now our test looks like this:

Por estas razões, especialmente no módulo `tests`, `use super::something` é
geralmente a melhor solução. Então, agora nosso teste parece assim:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    use super::client;

    #[test]
    fn it_works() {
        client::connect();
    }
}
```

When we run `cargo test` again, the test will pass and the first part of the
test result output will be the following:

Quando executarmos novamente  `cargo test`, o teste passará e a primeira parte do
resultado do teste será o seguinte:

```text
$ cargo test
   Compiling communicator v0.1.0 (file:///projects/communicator)
     Running target/debug/communicator-92007ddb5330fa5a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

## Summary
## Sumário

Now you know some new techniques for organizing your code! Use these techniques
to group related functionality together, keep files from becoming too long, and
present a tidy public API to your library users.

Agora você conhece algumas técnicas novas para organizar o seu código! Use estas técnicas
para agrupar as funcionalidades relacionadas, evitar que os arquivos tornem-se muito longos, e
apresentar uma API pública arrumada para os usuários da sua biblioteca.

Next, we’ll look at some collection data structures in the standard library
that you can use in your nice, neat code!

Em seguida, analisaremos algumas estruturas de dados de coleções na biblioteca padrão
que você pode usar em seu código, legal!

