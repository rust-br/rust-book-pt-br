## Controlling Visibility with `pub`
## Controlando a Visibilidade com `pub`

We resolved the error messages shown in Listing 7-5 by moving the `network` and
`network::server` code into the *src/network/mod.rs* and
*src/network/server.rs* files, respectively. At that point, `cargo build` was
able to build our project, but we still get warning messages about the
`client::connect`, `network::connect`, and `network::server::connect` functions
not being used:

Resolvemos as mensagens de erro mostradas na Listagem 7-5 movendo o código de `network` e
`network::server` para  os arquivos *src/network/mod.rs* e
*src/network/server.rs*, respectivamente. Nesse ponto, `cargo build` era
capaz de construir nosso projeto, mas ainda recebemos mensagens de warning(viso) sobre as
funções `client::connect`, `network::connect`, e `network::server::connect`
não estarem em uso:

```text
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

So why are we receiving these warnings? After all, we’re building a library
with functions that are intended to be used by our *users*, not necessarily by
us within our own project, so it shouldn’t matter that these `connect`
functions go unused. The point of creating them is that they will be used by
another project, not our own.

Então, por que estamos recebendo esses warnings(avisos)? Afinal, estamos construindo uma biblioteca
com funções que se destinam a ser usadas pelos nossos *usuários*, não necessariamente por
nós dentro de nosso próprio projeto, por isso não deveria importar que essas funções `connect`
não sejam utilizadas. O ponto de criá-las é que elas serão usadas por
outro projeto, não o nosso.

To understand why this program invokes these warnings, let’s try using the
`connect` library from another project, calling it externally. To do that,
we’ll create a binary crate in the same directory as our library crate by
making a *src/main.rs* file containing this code:

Para entender por que esse programa invoca esses warnings(avisos), vamos tentar usar a
biblioteca `connect` de outro projeto, chamando-a externamente. Para fazer isso,
vamos criar um crate binário no mesmo diretório que a nossa crate de biblioteca
fazendo um arquivo *src/main.rs* conter esse código:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

We use the `extern crate` command to bring the `communicator` library crate
into scope. Our package now contains *two* crates. Cargo treats *src/main.rs*
as the root file of a binary crate, which is separate from the existing library
crate whose root file is *src/lib.rs*. This pattern is quite common for
executable projects: most functionality is in a library crate, and the binary
crate uses that library crate. As a result, other programs can also use the
library crate, and it’s a nice separation of concerns.

Usamos o comando `extern crate` para trazer o crate de biblioteca `communicator`
para o escopo. Nosso pacote agora contém *duas* crates. Cargo trata *src/main.rs*
como um arquivo raiz de um crate binário, que é separada das crates binárias de biblioteca existente
cuja arquivo raiz é *src/lib.rs*. Esse padrão é bastante comum para
projetos executáveis: a maioria das funcionalidades está em uma crate de biblioteca e a crate binária
usa essa crate de biblioteca. Como resultado, outros programas também podem usar a
crate de biblioteca, e é uma boa separação de responsabilidades.

From the point of view of a crate outside the `communicator` library looking
in, all the modules we’ve been creating are within a module that has the same
name as the crate, `communicator`. We call the top-level module of a crate the
*root module*.

Do ponto de vista de uma crate fora da biblioteca `communicator` 
todos os módulos que criamos estão dentro de um módulo que tem o mesmo
nome como da crate, `communicator`. Chamamos o módulo de nível superior de um
*módulo raiz*.

Also note that even if we’re using an external crate within a submodule of our
project, the `extern crate` should go in our root module (so in *src/main.rs*
or *src/lib.rs*). Then, in our submodules, we can refer to items from external
crates as if the items are top-level modules.

Observe também que, mesmo que estejamos usando uma crate externa dentro de um submódulo do nosso
projeto, o `extern crate` deve entrar em nosso módulo raiz (então em *src/main.rs*
ou *src/lib.rs*). Então, em nossos submódulos, podemos consultar itens de crates externas
como se os itens fossem módulos de nível superior.

Right now, our binary crate just calls our library’s `connect` function from
the `client` module. However, invoking `cargo build` will now give us an error
after the warnings:

Agora, nossa crate binária apenas chama a função `connect` da nossa biblioteca do
módulo `client`. No entanto, invocar agora `cargo build`  nos dará um erro
após os warnings(avisos):

```text
error[E0603]: module `client` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Ah ha! This error tells us that the `client` module is private, which is the
crux of the warnings. It’s also the first time we’ve run into the concepts of
*public* and *private* in the context of Rust. The default state of all code in
Rust is private: no one else is allowed to use the code. If you don’t use a
private function within your program, because your program is the only code
allowed to use that function, Rust will warn you that the function has gone
unused.

Ah ha! Este erro nos diz que o módulo `client` é privado, que é o
cerne das advertências. É também a primeira vez em que nos encontramos com os conceitos de
*público* e *privado* no contexto do Rust. O estado padrão de todos os códigos em
Rust é privado: ninguém mais tem permissão para usar o código. Se você não usar um
função privada dentro do seu programa, porque seu programa é o único código
permitido usar essa função, Rust irá avisá-lo de que a função 
não foi utilizada.

After we specify that a function like `client::connect` is public, not only
will our call to that function from our binary crate be allowed, but the
warning that the function is unused will go away. Marking a function as public
lets Rust know that the function will be used by code outside of our program.
Rust considers the theoretical external usage that’s now possible as the
function “being used.” Thus, when a function is marked public, Rust will not
require that it be used in our program and will stop warning that the function
is unused.

Depois de especificar que uma função como `client::connect` é pública, não só
será permitida a nossa chamada para essa função a partir de nossa crate binária, mas o
warning(aviso) de que a função não é utilizada irá desaparecer. Marcar uma função como pública
permite ao Rust saber que a função será usada por código fora do nosso programa.
Rust considera que agora é possível que a 
função "sendo usada". Assim, quando uma função é marcada como pública, Rust não
exige que seja usada em nosso programa e deixará de avisar que a função
não é utilizada.

### Making a Function Public
### Fazendo uma Função Pública

To tell Rust to make a function public, we add the `pub` keyword to the start
of the declaration. We’ll focus on fixing the warning that indicates
`client::connect` has gone unused for now, as well as the `` module `client` is
private `` error from our binary crate. Modify *src/lib.rs* to make the
`client` module public, like so:

Para dizer a Rust para tornar pública uma função, adicionamos a palavra-chave `pub` ao início
da declaração. Nos focaremos em corrigir o warning(aviso) que indica
`client::connect` não foi utilizado por enquanto, assim como o erro `` module `client` is
private `` (`` módulo `client` é privado ``) do nosso crate binário. Modifique *src/lib.rs* para fazer o
módulo `client`, assim:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
pub mod client;

mod network;
```

The `pub` keyword is placed right before `mod`. Let’s try building again:

A palavra-chave `pub` é colocada logo antes do `mod`. Vamos tentar construir(build) novamente:

```text
error[E0603]: function `connect` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Hooray! We have a different error! Yes, different error messages are a cause
for celebration. The new error shows `` function `connect` is private ``, so
let’s edit *src/client.rs* to make `client::connect` public too:

Opa! Temos um erro diferente! Sim, diferentes mensagens de erro são uma causa
para a celebração. O novo erro mostra que `` function `connect` is private ``(`` função `connect` é privada ``), então
vamos editar *src/client.rs* para torná-la públicao `client::connect` também:

<span class="filename">Filename: src/client.rs</span>

<span class="filename">Nome do arquivo: src/client.rs</span>

```rust
pub fn connect() {
}
```

Now run `cargo build` again:

Agora execute `cargo build` novamente:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

The code compiled, and the warning about `client::connect` not being used is
gone!

O código compila, e o warning(aviso) sobre `client::connect` não estar em uso 
se foi!

Unused code warnings don’t always indicate that an item in your code needs to
be made public: if you *didn’t* want these functions to be part of your public
API, unused code warnings could be alerting you to code you no longer need that
you can safely delete. They could also be alerting you to a bug if you had just
accidentally removed all places within your library where this function is
called.

Os avisos de código não utilizados nem sempre indicam que um item no seu código precisa
se tornar público: se você *não* quiser que essas funções façam parte de sua 
API pública, warnings(avisos) de código não utilizados podem alertá-lo que esse códigos não utilizados podem
ser excluidos com segurança. Eles também podem estar alertando você para um bug se você tivesse apenas
acidentalmente removido todos os lugares dentro da sua biblioteca onde esta função é
chamada.

But in this case, we *do* want the other two functions to be part of our
crate’s public API, so let’s mark them as `pub` as well to get rid of the
remaining warnings. Modify *src/network/mod.rs* to look like the following:

Mas neste caso, nós *queremos* que as outras duas funções façam parte da nossa
API pública da crate, então vamos marcá-los como `pub` também para se livrar do
warnings(avisos) remanescentes. Modifique *src/network/mod.rs* para se parecer com o seguinte:

<span class="filename">Filename: src/network/mod.rs</span>

<span class="filename">Nome do arquivo: src/network/mod.rs</span>

```rust,ignore
pub fn connect() {
}

mod server;
```

Then compile the code:

Em seguida, compile o código:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / pub fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

Hmmm, we’re still getting an unused function warning, even though
`network::connect` is set to `pub`. The reason is that the function is public
within the module, but the `network` module that the function resides in is not
public. We’re working from the interior of the library out this time, whereas
with `client::connect` we worked from the outside in. We need to change
*src/lib.rs* to make `network` public too, like so:

Hmmm, ainda estamos recebendo uns warnings(avisos) de função não utilizadas, embora
`network::connect` esteja configurado para `pub`. A razão é que a função é pública
dentro do módulo, mas o módulo `network` na qual a função reside não é
público. Estamos trabalhando a partir do interior da biblioteca desta vez, enquanto que
com `client::connect` trabalhamos de fora. Precisamos mudar
*src/lib.rs* para tornar `network` pública também, assim:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;
```

Now when we compile, that warning is gone:

```text
warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default
```

Only one warning is left! Try to fix this one on your own!

Apenas um warning(aviso) permanece. Tente consertar isso por conta própria!

### Privacy Rules
### Regras de Privacidade

Overall, these are the rules for item visibility:

No geral, estas são as regras para a visibilidade do item:

1. If an item is public, it can be accessed through any of its parent modules.
2. If an item is private, it can be accessed only by its immediate parent
   module and any of the parent’s child modules.

1. Se um item for público, ele pode ser acessado através de qualquer um dos seus módulos parentes.
2. Se um item é privado, ele só pode ser acessado por seu mṕdulo pai imediato
   qualquer um dos módulos filho do pai.

### Privacy Examples
### Exemplos de Privacidade

Let’s look at a few more privacy examples to get some practice. Create a new
library project and enter the code in Listing 7-6 into your new project’s
*src/lib.rs*:

Vejamos mais alguns exemplos de privacidade para obter alguma prática. Crie um novo
projeto da biblioteca e digite o código da Listagem 7-6 no seu novo projeto
*src/lib.rs*:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
mod outermost {
    pub fn middle_function() {}

    fn middle_secret_function() {}

    mod inside {
        pub fn inner_function() {}

        fn secret_function() {}
    }
}

fn try_me() {
    outermost::middle_function();
    outermost::middle_secret_function();
    outermost::inside::inner_function();
    outermost::inside::secret_function();
}
```

<span class="caption">Listing 7-6: Examples of private and public functions,
some of which are incorrect</span>

<span class="caption">Lista 7-6: exemplos de funções públicas e privadas,
alguns dos quais estão incorretos</span>


Before you try to compile this code, make a guess about which lines in the
`try_me` function will have errors. Then, try compiling the code to see whether
you were right, and read on for the discussion of the errors!

Antes de tentar compilar este código, tente um palpite sobre quais linhas na
função `try_me` terá erros. Em seguida, tente compilar o código para ver se
você estava certo e leia sobre a discussão dos erros!

#### Looking at the Errors
#### Olhando para os Erros

The `try_me` function is in the root module of our project. The module named
`outermost` is private, but the second privacy rule states that the `try_me`
function is allowed to access the `outermost` module because `outermost` is in
the current (root) module, as is `try_me`.

A função `try_me` está no módulo raiz do nosso projeto. O módulo chamado
`outermost` é privado, mas a segunda regra de privacidade afirma que a função `try_me`
pode acessar o módulo `outermost` porque `outermost` está no
módulo atual (raiz), bem como `try_me`.

The call to `outermost::middle_function` will work because `middle_function` is
public, and `try_me` is accessing `middle_function` through its parent module
`outermost`. We determined in the previous paragraph that this module is
accessible.

A chamada para `outermost::middle_function` funcionará porque `middle_function` é
pública e `try_me` está acessando `middle_function` através do seu módulo pai
`outermost`. Determinamos no parágrafo anterior que este módulo é
acessível.

The call to `outermost::middle_secret_function` will cause a compilation error.
`middle_secret_function` is private, so the second rule applies. The root
module is neither the current module of `middle_secret_function` (`outermost`
is), nor is it a child module of the current module of `middle_secret_function`.

A chamada para `outermost::middle_secret_function` causará um erro de compilação.
`middle_secret_function` é privado, então a segunda regra se aplica. O módulo raiz
não é o módulo atual de `middle_secret_function` (` outermost`
é), nem é um módulo filho do módulo atual de `middle_secret_function`.

The module named `inside` is private and has no child modules, so it can only
be accessed by its current module `outermost`. That means the `try_me` function
is not allowed to call `outermost::inside::inner_function` or
`outermost::inside::secret_function`.

O módulo denominado `inside` é privado e não tem módulos filho, portanto, ele só pode
ser acessado pelo seu módulo atual `outermost`. Isso significa que a função `try_me`
não tem permissão de chamar `outermost::inside::inner_function` ou
`outermost::inside::secret_function`.

#### Fixing the Errors
#### Reparando os Erros

Here are some suggestions for changing the code in an attempt to fix the
errors. Before you try each one, make a guess as to whether it will fix the
errors, and then compile the code to see whether or not you’re right, using the
privacy rules to understand why.

Aqui estão algumas sugestões para alterar o código na tentativa de corrigir os
erros. Antes de tentar cada um, tente adivinhar se ele irá consertar o
erros e, em seguida, compile o código para ver se você está certo ou não, usando as
regras de privacidade para entender o porquê.

* What if the `inside` module was public?
* What if `outermost` was public and `inside` was private?
* What if, in the body of `inner_function`, you called
  `::outermost::middle_secret_function()`? (The two colons at the beginning mean
  that we want to refer to the modules starting from the root module.)

* E se o módulo `inside` fosse público?
* E se `outermost` fosse pública e `inside` fosse privado?
* E se, no corpo de `inner_function`, você chamasse
  `::outermost::middle_secret_function()`? (Os dois dois pontos no início significam
   que queremos consultar os módulos a partir do módulo raiz.)

Feel free to design more experiments and try them out!

Sinta-se livre para projetar mais experimentos e experimentá-los!

Next, let’s talk about bringing items into scope with the `use` keyword.

Em seguida, vamos falar sobre trazer itens ao escopo com a palavra-chave `use`.