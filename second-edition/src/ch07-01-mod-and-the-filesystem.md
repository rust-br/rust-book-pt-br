## `mod` and the Filesystem
## `mod` e o Sistema de Arquivos

We’ll start our module example by making a new project with Cargo, but instead
of creating a binary crate, we’ll make a library crate: a project that other
people can pull into their projects as a dependency. For example, the `rand`
crate discussed in Chapter 2 is a library crate that we used as a dependency in
the guessing game project.

Vamos iniciar o nosso exemplo de módulo fazendo um novo projeto com o Cargo, mas em vez de
criar um crate binário, faremos um crate de biblioteca: um projeto que 
as outras pessoas podem puxar para os seus projetos como uma dependência. Por exemplo, o crate `rand`
discutido no Capítulo 2, é um crate de biblioteca que usamos como uma dependência no
projeto do jogo de adivinhação.

We’ll create a skeleton of a library that provides some general networking
functionality; we’ll concentrate on the organization of the modules and
functions but we won’t worry about what code goes in the function bodies. We’ll
call our library `communicator`. By default, Cargo will create a library unless
another type of project is specified: if we omit the `--bin` option that we’ve
been using in all of the chapters preceding this one, our project will be a
library:

Criaremos um esqueleto de uma biblioteca que fornece algumas funcionalidades gerais
de rede; nos concentraremos na organização dos módulos e funções,
mas não nos preocuparemos com o código que está dentro das funções. Chamaremos
nossa biblioteca de `communicator`. Por padrão, o Cargo criará uma biblioteca, a menos que
outro tipo de projeto seja especificado: se omitimos a opção `--bin`, que temos
usado em todos os capítulos anteriores a este, nosso projeto será um
biblioteca:


```text
$ cargo new communicator
$ cd communicator
```

Notice that Cargo generated *src/lib.rs* instead of *src/main.rs*. Inside
*src/lib.rs* we’ll find the following:

Observe que Cargo gerou *src/lib.rs* em vez de *src/main.rs*. Dentro de
*src/lib.rs* encontraremos o seguinte:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Cargo creates an example test to help us get our library started, rather than
the “Hello, world!” binary that we get when we use the `--bin` option. We’ll
look at the `#[]` and `mod tests` syntax in the “Using `super` to Access a
Parent Module” section later in this chapter, but for now, leave this code at
the bottom of *src/lib.rs*.

Cargo cria um teste de exemplo para nos ajudar a começar nossa biblioteca, em vez de
o binário “Hello, world!” que recebemos quando usamos a opção `--bin`. Olharemos
a sintaxe `#[]` e `mod tests` no “Usando `super` para Acessar um
Módulo Pai” mais adiante neste capítulo, mas por agora, deixe este código
na parte inferior de *src/lib.rs*.

Because we don’t have a *src/main.rs* file, there’s nothing for Cargo to
execute with the `cargo run` command. Therefore, we’ll use the `cargo build`
command to compile our library crate’s code.

Como não temos um arquivo *src/main.rs*, não há nada para ser executado pelo Cargo
com o comando `cargo run`. Portanto, usaremos o comando  `cargo build`
para compilar o código da nossa biblioteca.

We’ll look at different options for organizing your library’s code that will be
suitable in a variety of situations, depending on the intent of the code.

Examinaremos diferentes opções para organizar o código da sua biblioteca que serão
adequados em uma variedade de situações, dependendo da intenção do código.

### Module Definitions
### Definições do Módulo

For our `communicator` networking library, we’ll first define a module named
`network` that contains the definition of a function called `connect`. Every
module definition in Rust starts with the `mod` keyword. Add this code to the
beginning of the *src/lib.rs* file, above the test code:

Para a nossa biblioteca de rede `communicator`, primeiro definiremos um módulo chamado
`network` que contém a definição de uma função chamada` connect`. Cada
definição de módulo em Rust começa com a palavra-chave `mod`. Adicione este código ao
início do arquivo *src/lib.rs*, acima do código de teste:


<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}
```

After the `mod` keyword, we put the name of the module, `network`, and then a
block of code in curly brackets. Everything inside this block is inside the
namespace `network`. In this case, we have a single function, `connect`. If we
wanted to call this function from code outside the `network` module, we
would need to specify the module and use the namespace syntax `::`, like so:
`network::connect()` rather than just `connect()`.

Após a palavra-chave `mod`, colocamos o nome do módulo, `network` e, em seguida, um
bloco de código entre chaves. Tudo dentro deste bloco está dentro do
namespace `network`. Neste caso, temos uma única função, `connect`. Se nós
quisermos chamar essa função do código fora do módulo `network`, nós
precisaremos especificar o módulo e usar a sintaxe do namespace `::`, assim:
`network::connect()` em vez de apenas `connect()`.

We can also have multiple modules, side by side, in the same *src/lib.rs* file.
For example, to also have a `client` module that has a function named `connect`
as well, we can add it as shown in Listing 7-1:

Também podemos ter múltiplos módulos, lado a lado, no mesmo arquivo *src/lib.rs*.
Por exemplo, para ter mais um módulo `client` que possui uma função chamada `connect`
, podemos adicioná-lo como mostrado na Listagem 7-1:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}

mod client {
    fn connect() {
    }
}
```

<span class="caption">Listing 7-1: The `network` module and the `client` module
defined side by side in *src/lib.rs*</span>

<span class = "caption"> Listagem 7-1: O módulo `network` e o módulo `client`
definido lado a lado em *src/lib.rs* </span>

Now we have a `network::connect` function and a `client::connect` function.
These can have completely different functionality, and the function names do
not conflict with each other because they’re in different modules.

Agora, temos uma função `network::connect` e uma função `client::connect`.
Estas podem ter funcionalidades completamente diferentes, e os nomes das funções
não estão em conflito entre si porque estão em módulos diferentes.

In this case, because we’re building a library, the file that serves as the
entry point for building our library is *src/lib.rs*. However, in respect to
creating modules, there’s nothing special about *src/lib.rs*. We could also
create modules in *src/main.rs* for a binary crate in the same way as we’re
creating modules in *src/lib.rs* for the library crate. In fact, we can put
modules inside of modules, which can be useful as your modules grow to keep
related functionality organized together and separate functionality apart. The
choice of how you organize your code depends on how you think about the
relationship between the parts of your code. For instance, the `client` code
and its `connect` function might make more sense to users of our library if
they were inside the `network` namespace instead, as in Listing 7-2:

Nesse caso, como estamos construindo uma biblioteca, o arquivo que serve como
ponto de entrada para construir nossa biblioteca é *src/lib.rs*. No entanto, em relação a
criação de módulos, não há nada de especial sobre *src/lib.rs*. Poderíamos também
criar módulos em *src/main.rs* para uma crate binária da mesma forma que nós
criamos módulos em *src/lib.rs* para a crate da biblioteca. Na verdade, podemos colocar módulos 
dentro de módulos, o que pode ser útil à medida que seus módulos crescem para manter juntas 
funcionalidades relacionadas e separar funcionalidades não relacionadas. A
escolha de como você organiza seu código depende do que você pensa sobre a
relação entre as partes do seu código. Por exemplo, o código `client`
e a função `connect` podem ter mais sentido para os usuários de nossa biblioteca se
eles estivessem dentro do namespace `network`, como na Listagem 7-2:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-2: Moving the `client` module inside the
`network` module</span>

<span class = "caption"> Listagem 7-2: Movendo o módulo `client` para dentro do
módulo `network`</span>

In your *src/lib.rs* file, replace the existing `mod network` and `mod client`
definitions with the ones in Listing 7-2, which have the `client` module as an
inner module of `network`. Now we have the functions `network::connect` and
`network::client::connect`: again, the two functions named `connect` don’t
conflict with each other because they’re in different namespaces.

No seu arquivo *src/lib.rs*, substitua as definições `mod network` e `mod client`
pelas da Listagem 7-2, que possuem o módulo `client` como um
módulo interno da `network`. Agora temos as funções `network::connect` e
`network::client::connect`: novamente, as duas funções denominadas `connect` não conflitam
uma com a outra porque elas estão em diferentes namespaces.

In this way, modules form a hierarchy. The contents of *src/lib.rs* are at the
topmost level, and the submodules are at lower levels. Here’s what the
organization of our example in Listing 7-1 looks like when thought of as a
hierarchy:

Desta forma, os módulos formam uma hierarquia. O conteúdo de *src/lib.rs* está no
nível superior mais alto, e os submódulos estão em níveis mais baixos. Aqui está
a nossa organização quando pensada de forma hierárquica na Listagem 7-1:

```text
communicator
 ├── network
 └── client
```

And here’s the hierarchy corresponding to the example in Listing 7-2:

E aqui está a hierarquia correspondente ao exemplo na Listagem 7-2:

```text
communicator
 └── network
     └── client
```

The hierarchy shows that in Listing 7-2, `client` is a child of the `network`
module rather than a sibling. More complicated projects can have many modules,
and they’ll need to be organized logically in order to keep track of them. What
“logically” means in your project is up to you and depends on how you and your
library’s users think about your project’s domain. Use the techniques shown
here to create side-by-side modules and nested modules in whatever structure
you would like.

Conforme a hierarquia mostrada na Listagem 7-2, `client` é um filho do módulo `network`
em vez de um irmão. Projetos mais complicados podem ter muitos módulos, é necessário 
organizá-los logicamente para mantê-los sob controle. O que "logicamente" significa em 
seu projeto fica a seu critério, e depende do que você e os usuários da sua biblioteca 
pensam sobre o domínio do seu projeto. Use as técnicas mostradas
aqui para criar módulos lado a lado e módulos aninhados em qualquer estrutura que
você queira.

### Moving Modules to Other Files
### Movendo Módulos para Outros Arquivos

Modules form a hierarchical structure, much like another structure in computing
that you’re used to: filesystems! We can use Rust’s module system along with
multiple files to split up Rust projects so not everything lives in
*src/lib.rs* or *src/main.rs*. For this example, let’s start with the code in
Listing 7-3:

Os módulos formam uma estrutura hierárquica, bem parecida com outra estrutura computacional 
que você conhece: sistemas de arquivos! Podemos usar o sistema de módulos do Rust juntamente com
vários arquivos para dividir projetos Rust de forma que nem tudo resida em
*src/lib.rs* ou *src/main.rs*. Para este exemplo, vamos começar com o código em
Listagem 7-3:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
mod client {
    fn connect() {
    }
}

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-3: Three modules, `client`, `network`, and
`network::server`, all defined in *src/lib.rs*</span>

<span class="caption">Listagem 7-3: Três módulos, `client`, `network`, e
`network::server`, todos definidos em *src/lib.rs*</span>

The file *src/lib.rs* has this module hierarchy:

O arquivo *src/lib.rs* possui esta hierarquia de módulos:

```text
communicator
 ├── client
 └── network
     └── server
```

If these modules had many functions, and those functions were becoming lengthy,
it would be difficult to scroll through this file to find the code we wanted to
work with. Because the functions are nested inside one or more `mod` blocks,
the lines of code inside the functions will start getting lengthy as well.
These would be good reasons to separate the `client`, `network`, and `server`
modules from *src/lib.rs* and place them into their own files.

Se esses módulos tivessem muitas funções, e elas estivessem se alongando muito,
seria difícil percorrer esse arquivo para encontrar o código com que queremos
trabalhar. Como as funções estão aninhadas dentro de um ou mais blocos `mod`,
as linhas de código dentro das funções começarão a se alongar também.
Estes seriam bons motivos para separar os módulos `client`, `network`, e `server`
de *src/lib.rs* e colocá-los em seus próprios arquivos.

First, replace the `client` module code with only the declaration of the
`client` module, so that your *src/lib.rs* looks like code shown in Listing 7-4:

Primeiro, substitua o código do módulo `client` por apenas a declaração do
módulo `client`, para que seu *src/lib.rs* se pareça com o código mostrado na Listagem 7-4:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
mod client;

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-4: Extracting the contents of the `client` module but leaving the declaration in *src/lib.rs*</span>

<span class="caption"> Listagem 7-4: Extraindo o conteúdo do módulo `client`, mas deixando a declaração em *src/lib.rs* </span>

We’re still *declaring* the `client` module here, but by replacing the block
with a semicolon, we’re telling Rust to look in another location for the code
defined within the scope of the `client` module. In other words, the line `mod
client;` means:

Ainda estamos *declarando* o módulo `client` aqui, mas ao substituir o bloco
por um ponto e vírgula, estamos dizendo ao Rust para que procure, em outro local, o código
definido no escopo do módulo `client`. Em outras palavras, a linha `mod
client;` significa:

```rust,ignore
mod client {
    // conteúdo de client.rs
}
```

Now we need to create the external file with that module name. Create a
*client.rs* file in your *src/* directory and open it. Then enter the
following, which is the `connect` function in the `client` module that we
removed in the previous step:

Agora precisamos criar o arquivo externo com o nome do módulo. Crie um
arquivo *client.rs* em *src/* e abra-o. Em seguida digite o seguinte,
que é a função `connect` do módulo `client` que foi 
removida na etapa anterior:

<span class="filename">Filename: src/client.rs</span>

<span class="filename">Arquivo: src/client.rs</span>

```rust
fn connect() {
}
```

Note that we don’t need a `mod` declaration in this file because we already
declared the `client` module with `mod` in *src/lib.rs*. This file just
provides the *contents* of the `client` module. If we put a `mod client` here,
we’d be giving the `client` module its own submodule named `client`!

Observe que não precisamos de uma declaração `mod` neste arquivo porque já fizemos
a declaração do módulo `client` com `mod` em *src/lib.rs*. Este arquivo apenas
fornece o *conteúdo* do módulo `client`. Se colocarmos um `mod client` aqui,
nós estaríamos dando ao módulo `client` seu próprio submódulo chamado `client`!

Rust only knows to look in *src/lib.rs* by default. If we want to add more
files to our project, we need to tell Rust in *src/lib.rs* to look in other
files; this is why `mod client` needs to be defined in *src/lib.rs* and can’t
be defined in *src/client.rs*.

Rust só sabe olhar em *src/lib.rs* por padrão. Se quisermos adicionar mais
arquivos para o nosso projeto, precisamos dizer ao Rust em *src/lib.rs* para procurar em outros
arquivos; é por isso que `mod client` precisa ser definido em *src/lib.rs* e não pode
ser definido em *src/client.rs*.

Now the project should compile successfully, although you’ll get a few
warnings. Remember to use `cargo build` instead of `cargo run` because we have
a library crate rather than a binary crate:

Agora, o projeto deve compilar com sucesso, embora você obtenha alguns
warnings (avisos). Lembre-se de usar `cargo build`, em vez de `cargo run`, porque temos
uma crate de biblioteca em vez de uma crate binária:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/lib.rs:4:5
  |
4 | /     fn connect() {
5 | |     }
  | |_____^

warning: function is never used: `connect`
 --> src/lib.rs:8:9
  |
8 | /         fn connect() {
9 | |         }
  | |_________^
```

These warnings tell us that we have functions that are never used. Don’t worry
about these warnings for now; we’ll address them later in this chapter in the
“Controlling Visibility with `pub`” section. The good news is that they’re just
warnings; our project built successfully!

Esses *warnings* nos dizem que temos funções que nunca são usadas. Não se preocupe
com esses *warnings* por enquanto; vamos abordá-los mais adiante neste capítulo, na
seção “Controlando a visibilidade com `pub`”. A boa notícia é que eles são apenas
*warnings*; nosso projeto foi construído com sucesso!

Next, let’s extract the `network` module into its own file using the same
pattern. In *src/lib.rs*, delete the body of the `network` module and add a
semicolon to the declaration, like so:

Em seguida, vamos extrair o módulo `network` em seu próprio arquivo usando o mesmo
procedimento. Em *src/lib.rs*, exclua o corpo do módulo `network` e adicione um
ponto e vírgula para a declaração, assim:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
mod client;

mod network;
```

Then create a new *src/network.rs* file and enter the following:

Em seguida, crie um novo arquivo *src/network.rs* e digite o seguinte:

<span class="filename">Filename: src/network.rs</span>

<span class="filename">Arquivo: src/network.rs</span>

```rust
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

Notice that we still have a `mod` declaration within this module file; this is
because we still want `server` to be a submodule of `network`.

Observe que ainda temos uma declaração `mod` dentro deste arquivo de módulo; isto é
porque ainda queremos que `server` seja um submódulo de `network`.

Run `cargo build` again. Success! We have one more module to extract: `server`.
Because it’s a submodule—that is, a module within a module—our current tactic
of extracting a module into a file named after that module won’t work. We’ll
try anyway so you can see the error. First, change *src/network.rs* to have
`mod server;` instead of the `server` module’s contents:

Execute `cargo build` novamente. Sucesso! Temos mais um módulo para extrair: `server`.
Como ele é um submódulo - ou seja, um módulo dentro de outro - nossa tática atual de 
extrair um módulo para um arquivo com o nome do módulo não funcionará. Iremos
tentar, de qualquer maneira, para que você possa ver o erro. Primeiro, altere o arquivo *src/network.rs* colocando
`mod server;` no lugar do conteúdo do módulo `server`:

<span class="filename">Filename: src/network.rs</span>

<span class="filename">Arquivo: src/network.rs</span>

```rust,ignore
fn connect() {
}

mod server;
```

Then create a *src/server.rs* file and enter the contents of the `server`
module that we extracted:

Em seguida, crie um arquivo *src/server.rs* e insira o conteúdo do módulo `server`
que extraímos:

<span class="filename">Filename: src/server.rs</span>

<span class="filename">Arquivo: src/server.rs</span>

```rust
fn connect() {
}
```

When we try to `cargo build`, we’ll get the error shown in Listing 7-5:

Quando tentamos `cargo build`, obteremos o erro mostrado na Listagem 7-5:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
error: cannot declare a new module at this location
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
  |
note: maybe move this module `src/network.rs` to its own directory via `src/network/mod.rs`
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
note: ... or maybe `use` the module `server` instead of possibly redeclaring it
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
```

<span class="caption">Listing 7-5: Error when trying to extract the `server`
submodule into *src/server.rs*</span>

<span class="caption">Listagem 7-5: Erro ao tentar extrair o submódulo `server`
 em *src/server.rs* </span>

The error says we `cannot declare a new module at this location` and is
pointing to the `mod server;` line in *src/network.rs*. So *src/network.rs* is
different than *src/lib.rs* somehow: keep reading to understand why.

O erro diz que não podemos declarar um novo módulo neste local (`cannot declare a new module at this location`) 
e está apontando para a linha `mod server`; em *src/network.rs*. Então *src/network.rs* é
diferente de *src/lib.rs* de alguma forma: continue lendo para entender o porquê.

The note in the middle of Listing 7-5 is actually very helpful because it
points out something we haven’t yet talked about doing:

A nota no meio da Listagem 7-5 é realmente muito útil, 
porque ela aponta para algo de que não falamos ainda:

```text
note: maybe move this module `network` to its own directory via
`network/mod.rs`
```

Instead of continuing to follow the same file naming pattern we used
previously, we can do what the note suggests:

Em vez de continuar a seguir o mesmo padrão de nomeação de arquivo usado
anteriormente, podemos fazer o que a nota sugere:

1. Make a new *directory* named *network*, the parent module’s name.
2. Move the *src/network.rs* file into the new *network* directory, and
   rename it to *src/network/mod.rs*.
3. Move the submodule file *src/server.rs* into the *network* directory.

1. Crie um novo *diretório* chamado *network*, o nome do módulo pai.
2. Mova o arquivo *src/network.rs* para o novo diretório *network* e
    renomeie para *src/network/mod.rs*.
3. Mova o arquivo de submódulo *src/server.rs* para o diretório *network*.

Here are commands to carry out these steps:

Aqui estão os comandos para executar estas etapas:

```text
$ mkdir src/network
$ mv src/network.rs src/network/mod.rs
$ mv src/server.rs src/network
```

Now when we try to run `cargo build`, compilation will work (we’ll still have
warnings though). Our module layout still looks like this, which is exactly the
same as it did when we had all the code in *src/lib.rs* in Listing 7-3:

Agora, quando tentamos executar `cargo build`, a compilação funcionará (embora ainda teremos
avisos). Nosso layout do módulo ainda parece assim, que é exatamente o
o mesmo que ocorreu quando tínhamos todo o código em *src/lib.rs* na Listagem 7-3:

```text
communicator
 ├── client
 └── network
     └── server
```

The corresponding file layout now looks like this:

O layout do arquivo correspondente agora se parece com isto:

```text
├── src
│   ├── client.rs
│   ├── lib.rs
│   └── network
│       ├── mod.rs
│       └── server.rs
```

So when we wanted to extract the `network::server` module, why did we have to
also change the *src/network.rs* file to the *src/network/mod.rs* file and put
the code for `network::server` in the *network* directory in
*src/network/server.rs* instead of just being able to extract the
`network::server` module into *src/server.rs*? The reason is that Rust wouldn’t
be able to recognize that `server` was supposed to be a submodule of `network`
if the *server.rs* file was in the *src* directory. To clarify Rust’s behavior
here, let’s consider a different example with the following module hierarchy,
where all the definitions are in *src/lib.rs*:

Quando queríamos extrair o módulo `network::server`, por que precisávamos
também mudar o arquivo *src/network.rs* para o arquivo *src/network/mod.rs* e colocar
o código de `network::server` no diretório *network* em
*src/network/server.rs* em vez de apenas extrair o
módulo `network::server` em *src/server.rs*? O motivo é que Rust não
será capaz de reconhecer que `server` deveria ser um submódulo de `network`
se o arquivo *server.rs* estiver no diretório *src*. Para esclarecer o comportamento de Rust
aqui, consideremos um exemplo diferente com a seguinte hierarquia de módulos,
onde todas as definições estão em *src/lib.rs*:

```text
communicator
 ├── client
 └── network
     └── client
```

In this example, we have three modules again: `client`, `network`, and
`network::client`. Following the same steps we did earlier for extracting
modules into files, we would create *src/client.rs* for the `client` module.
For the `network` module, we would create *src/network.rs*. But we wouldn’t be
able to extract the `network::client` module into a *src/client.rs* file
because that already exists for the top-level `client` module! If we could put
the code for *both* the `client` and `network::client` modules in the
*src/client.rs* file, Rust wouldn’t have any way to know whether the code was
for `client` or for `network::client`.

Neste exemplo, temos novamente três módulos : `client`,` network`, e
`network::client`. Seguindo os mesmos passos anteriores para extrair
módulos em arquivos, poderíamos criar *src/client.rs* para o módulo `client`.
Para o módulo `network`, poderíamos criar *src/network.rs*. Mas não seríamos
capazes de extrair o módulo `network::client` para um arquivo *src/client.rs*
porque ele já existe para o módulo `client` de nível superior! Se pudéssemos colocar
o código para *ambos* os módulos `client` e` network::client` no arquivo
*src/client.rs*, Rust não teria nenhuma maneira de saber se o código era
para `client` ou para `network::client`.

Therefore, in order to extract a file for the `network::client` submodule of
the `network` module, we needed to create a directory for the `network` module
instead of a *src/network.rs* file. The code that is in the `network` module
then goes into the *src/network/mod.rs* file, and the submodule
`network::client` can have its own *src/network/client.rs* file. Now the
top-level *src/client.rs* is unambiguously the code that belongs to the
`client` module.

Portanto, para extrair um arquivo para o submódulo `network::client` do
módulo `network`, precisamos criar um diretório para o módulo `network`
em vez de um arquivo *src/network.rs*. O código que está no módulo `network`
entra no arquivo *src/network/mod.rs*, e o submódulo
`network::client` pode ter seu próprio arquivo *src/network/client.rs*. Agora o
o nível superior *src/client.rs* é inequivocamente o código que pertence ao
módulo `client`.

### Rules of Module Filesystems
### Regras dos Módulos e Seus Arquivos

Let’s summarize the rules of modules with regard to files:

Vamos resumir as regras dos módulos em relação aos arquivos:

* If a module named `foo` has no submodules, you should put the declarations
  for `foo` in a file named *foo.rs*.
* If a module named `foo` does have submodules, you should put the declarations
  for `foo` in a file named *foo/mod.rs*.

* Se um módulo chamado `foo` não possui submódulos, você deve colocar as declarações
   para `foo` em um arquivo chamado *foo.rs*.
* Se um módulo chamado `foo` possui submódulos, você deve colocar as declarações
   para `foo` em um arquivo chamado *foo/mod.rs*.

These rules apply recursively, so if a module named `foo` has a submodule named
`bar` and `bar` does not have submodules, you should have the following files
in your *src* directory:

Essas regras aplicam-se de forma recursiva, então, se um módulo chamado `foo` tiver um submódulo chamado
`bar` e` bar` não possui submódulos, você deve ter os seguintes arquivos
no seu diretório *src*:

```text
├── foo
│   ├── bar.rs (contains the declarations in `foo::bar`)
│   └── mod.rs (contains the declarations in `foo`, including `mod bar`)
```

```text
├── foo
│   ├── bar.rs (contém as declarações em `foo::bar`)
│   └── mod.rs (contém as declarações em `foo`, incluindo `mod bar`)
```


The modules should be declared in their parent module’s file using the `mod`
keyword.

Os módulos devem ser declarados no arquivo do módulo pai usando a palavra-chave `mod`.

Next, we’ll talk about the `pub` keyword and get rid of those warnings!

Em seguida, vamos falar sobre a palavra-chave `pub` e nos livrar dessas warnings!
