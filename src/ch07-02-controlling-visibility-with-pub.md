## Controlando a Visibilidade com `pub`

Resolvemos as mensagens de erro mostradas na Listagem 7-5 movendo o código de `network` e
`network::server` para  os arquivos *src/network/mod.rs* e
*src/network/server.rs*, respectivamente. Nesse ponto, `cargo build` era
capaz de construir nosso projeto, mas ainda recebemos mensagens de *warning* sobre as
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

Então, por que estamos recebendo esses warnings(avisos)? Afinal, estamos construindo uma biblioteca
com funções que se destinam a ser usadas pelos nossos *usuários*, não necessariamente por
nós dentro de nosso próprio projeto, por isso não deveria importar que essas funções `connect`
não sejam utilizadas. O ponto de criá-las é que elas serão usadas por
outro projeto, não o nosso.

Para entender por que esse programa invoca esses warnings(avisos), vamos tentar usar a
biblioteca `connect` de outro projeto, chamando-a externamente. Para fazer isso,
vamos criar um crate binário no mesmo diretório que a nossa crate de biblioteca
fazendo um arquivo *src/main.rs* conter esse código:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

Usamos o comando `extern crate` para trazer o crate de biblioteca `communicator`
para o escopo. Nosso pacote agora contém *duas* crates. Cargo trata *src/main.rs*
como um arquivo raiz de um crate binário, que é separado do crate de biblioteca existente
cujo arquivo raiz é *src/lib.rs*. Esse padrão é bastante comum para
projetos executáveis: a maioria das funcionalidades está em uma crate de biblioteca e a crate binária
usa essa crate de biblioteca. Como resultado, outros programas também podem usar a
crate de biblioteca, e é uma boa separação de responsabilidades.

Do ponto de vista de uma crate fora da biblioteca `communicator` 
todos os módulos que criamos estão dentro de um módulo que tem o mesmo
nome como da crate, `communicator`. Chamamos o módulo de nível superior de um
crate de *módulo raiz*.

Observe também que, mesmo que estejamos usando uma crate externa dentro de um submódulo do nosso
projeto, o `extern crate` deve entrar em nosso módulo raiz (então em *src/main.rs*
ou *src/lib.rs*). Então, em nossos submódulos, podemos consultar itens de crates externas
como se os itens fossem módulos de nível superior.

Agora, nossa crate binária apenas chama a função `connect` da nossa biblioteca do
módulo `client`. No entanto, invocar agora `cargo build` nos dará um erro
após os *warnings*:

```text
error[E0603]: module `client` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Ah ha! Este erro nos diz que o módulo `client` é privado, que é o
cerne das advertências. É também a primeira vez em que nos encontramos com os conceitos de
*público* e *privado* no contexto do Rust. O estado padrão de todos os códigos em
Rust é privado: ninguém mais tem permissão para usar o código. Se você não usar uma
função privada dentro do seu programa, como ele é o único código
permitido a usar essa função, Rust irá avisá-lo de que a função 
não foi utilizada.

Depois de especificar que uma função como `client::connect` é pública, não só
será permitida a nossa chamada para essa função a partir de nossa crate binária, mas o
warning(aviso) de que a função não é utilizada irá desaparecer. Marcar uma função como pública
permite ao Rust saber que a função será usada por código fora do nosso programa.
Rust considera que agora é possível que a 
função esteja "sendo usada". Assim, quando uma função é marcada como pública, Rust não
exige que seja usada em nosso programa e deixará de avisar que a função
não é utilizada.

### Fazendo uma Função Pública

Para dizer ao Rust que torne pública uma função, adicionamos a palavra-chave `pub` ao início
da declaração. Nos focaremos em corrigir o *warning* que indica
`client::connect` não foi utilizado por enquanto, assim como o erro `` module `client` is
private `` (`` módulo `client` é privado ``) do nosso crate binário. Modifique *src/lib.rs* para tornar
o módulo `client` público, assim:

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
pub mod client;

mod network;
```

A palavra-chave `pub` é colocada logo antes do `mod`. Vamos tentar fazer o build novamente:

```text
error[E0603]: function `connect` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Opa! Temos um erro diferente! Sim, mensagens diferentes de erro 
são motivo para comemorar. O novo erro mostra que que a função `connect` é privada 
(function `connect` is private), então vamos editar *src/client.rs* para torná-la pública também:

<span class="filename">Arquivo: src/client.rs</span>

```rust
pub fn connect() {
}
```

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

O código compila, e o warning(aviso) sobre `client::connect` não estar em uso 
se foi!

Os avisos de código não utilizados nem sempre indicam que um item no seu código precisa
se tornar público: se você *não* quiser que essas funções façam parte de sua 
API pública, *warnings* de código não utilizado podem alertá-lo de que esses códigos não são mais necessários,
e que podem ser excluídos com segurança. Eles também podem estar alertando você para um bug, caso você tivesse apenas
acidentalmente removido todos os lugares dentro da sua biblioteca onde esta função é
chamada.

Mas neste caso, nós *queremos* que as outras duas funções façam parte da nossa
API pública da crate, então vamos marcá-las como `pub` também para se livrar dos
*warnings* remanescentes. Modifique *src/network/mod.rs* dessa forma:

<span class="filename">Arquivo: src/network/mod.rs</span>

```rust,ignore
pub fn connect() {
}

mod server;
```

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

Hmmm, ainda estamos recebendo um *warning* de função não utilizada, embora
`network::connect` esteja configurado para `pub`. A razão é que a função é pública
dentro do módulo, mas o módulo `network` na qual a função reside não é
público. Estamos trabalhando a partir do interior da biblioteca desta vez, enquanto que
com `client::connect` trabalhamos de fora. Precisamos mudar
*src/lib.rs* para tornar `network` pública também, assim:

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;
```

Agora, quando compilamos, esse aviso desapareceu:

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

Apenas um warning(aviso) permanece. Tente consertar isso por conta própria!

### Regras de Privacidade

No geral, estas são as regras para a visibilidade do item:

1. Se um item for público, ele pode ser acessado através de qualquer um dos seus módulos pais.
2. Se um item é privado, ele só pode ser acessado por seu módulo pai imediato e
   qualquer um dos módulos filhos do pai.

### Exemplos de Privacidade

Vejamos mais alguns exemplos de privacidade para obter alguma prática. Crie um novo
projeto da biblioteca e digite o código da Listagem 7-6 no seu novo projeto
*src/lib.rs*:

<span class="filename">Arquivo: src/lib.rs</span>

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

<span class="caption">Lista 7-6: Exemplos de funções públicas e privadas,
alguns dos quais estão incorretos</span>

Antes de tentar compilar este código, tente um palpite sobre quais linhas na
função `try_me` terá erros. Em seguida, tente compilar o código para ver se
você estava certo e leia sobre a discussão dos erros!

#### Olhando para os Erros

A função `try_me` está no módulo raiz do nosso projeto. O módulo chamado
`outermost` é privado, mas a segunda regra de privacidade afirma que a função `try_me`
pode acessar o módulo `outermost` porque `outermost` está no
módulo atual (raiz), bem como `try_me`.

A chamada para `outermost::middle_function` funcionará porque `middle_function` é
pública e `try_me` está acessando `middle_function` através do seu módulo pai
`outermost`. Determinamos no parágrafo anterior que este módulo é
acessível.

A chamada para `outermost::middle_secret_function` causará um erro de compilação.
`middle_secret_function` é privado, então a segunda regra se aplica. O módulo raiz
não é o módulo atual de `middle_secret_function` (` outermost`
é), nem é um módulo filho do módulo atual de `middle_secret_function`.

O módulo denominado `inside` é privado e não tem módulos filhos, portanto, ele só pode
ser acessado pelo seu módulo atual `outermost`. Isso significa que a função `try_me`
não tem permissão de chamar `outermost::inside::inner_function` ou
`outermost::inside::secret_function`.

#### Reparando os Erros

Aqui estão algumas sugestões para alterar o código na tentativa de corrigir os
erros. Antes de tentar cada uma, tente adivinhar se ela irá consertar os
erros e, em seguida, compile o código para ver se você está certo ou não, usando as
regras de privacidade para entender o porquê.

* E se o módulo `inside` fosse público?
* E se `outermost` fosse público e `inside` fosse privado?
* E se, no corpo de `inner_function`, você chamasse o
  `::outermost::middle_secret_function()`? (Os dois dois pontos no início significam
   que queremos consultar os módulos a partir do módulo raiz.)

Sinta-se livre para projetar mais experimentos e experimentá-los!

Em seguida, vamos falar sobre trazer itens ao escopo com a palavra-chave `use`.