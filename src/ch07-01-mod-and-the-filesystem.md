## `mod` e o Sistema de Arquivos

Vamos iniciar o nosso exemplo de módulo fazendo um novo projeto com o Cargo, mas em vez de
criar um crate binário, faremos um crate de biblioteca: um projeto que 
as outras pessoas podem puxar para os seus projetos como uma dependência. Por exemplo, o crate `rand`
discutido no Capítulo 2, é um crate de biblioteca que usamos como uma dependência no
projeto do jogo de adivinhação.

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

Observe que Cargo gerou *src/lib.rs* em vez de *src/main.rs*. Dentro de
*src/lib.rs* encontraremos o seguinte:

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

Cargo cria um teste de exemplo para nos ajudar a começar nossa biblioteca, em vez de
o binário “Hello, world!” que recebemos quando usamos a opção `--bin`. Olharemos
a sintaxe `#[]` e `mod tests` no “Usando `super` para Acessar um
Módulo Pai” mais adiante neste capítulo, mas por agora, deixe este código
na parte inferior de *src/lib.rs*.

Como não temos um arquivo *src/main.rs*, não há nada para ser executado pelo Cargo
com o comando `cargo run`. Portanto, usaremos o comando  `cargo build`
para compilar o código da nossa biblioteca.

Examinaremos diferentes opções para organizar o código da sua biblioteca que serão
adequados em uma variedade de situações, dependendo da intenção do código.

### Definições do Módulo

Para a nossa biblioteca de rede `communicator`, primeiro definiremos um módulo chamado
`network` que contém a definição de uma função chamada` connect`. Cada
definição de módulo em Rust começa com a palavra-chave `mod`. Adicione este código ao
início do arquivo *src/lib.rs*, acima do código de teste:


<span class="filename">Arquivo: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}
```

Após a palavra-chave `mod`, colocamos o nome do módulo, `network` e, em seguida, um
bloco de código entre chaves. Tudo dentro deste bloco está dentro do
namespace `network`. Neste caso, temos uma única função, `connect`. Se nós
quisermos chamar essa função do código fora do módulo `network`, nós
precisaremos especificar o módulo e usar a sintaxe do namespace `::`, assim:
`network::connect()` em vez de apenas `connect()`.

Também podemos ter múltiplos módulos, lado a lado, no mesmo arquivo *src/lib.rs*.
Por exemplo, para ter mais um módulo `client` que possui uma função chamada `connect`
, podemos adicioná-lo como mostrado na Listagem 7-1:

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

<span class = "caption"> Listagem 7-1: O módulo `network` e o módulo `client`
definidos lado a lado em *src/lib.rs* </span>

Agora, temos uma função `network::connect` e uma função `client::connect`.
Estas podem ter funcionalidades completamente diferentes, e os nomes das funções
não estão em conflito entre si porque estão em módulos diferentes.

Nesse caso, como estamos construindo uma biblioteca, o arquivo que serve como
ponto de entrada para construir nossa biblioteca é *src/lib.rs*. No entanto, em relação a
criação de módulos, não há nada de especial sobre *src/lib.rs*. Poderíamos também
criar módulos em *src/main.rs* para um crate binário da mesma forma que nós
criamos módulos em *src/lib.rs* para o crate de biblioteca. Na verdade, podemos colocar módulos 
dentro de módulos, o que pode ser útil à medida que seus módulos crescem para manter juntas 
funcionalidades relacionadas e separar funcionalidades não relacionadas. A
escolha de como você organiza seu código depende do que você pensa sobre a
relação entre as partes do seu código. Por exemplo, o código `client`
e a função `connect` podem ter mais sentido para os usuários de nossa biblioteca se
eles estivessem dentro do namespace `network`, como na Listagem 7-2:

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

<span class = "caption"> Listagem 7-2: Movendo o módulo `client` para dentro do
módulo `network`</span>

No seu arquivo *src/lib.rs*, substitua as definições `mod network` e `mod client`
pelas da Listagem 7-2, que possuem o módulo `client` como um
módulo interno da `network`. Agora temos as funções `network::connect` e
`network::client::connect`: novamente, as duas funções denominadas `connect` não conflitam
uma com a outra porque elas estão em diferentes namespaces.

Desta forma, os módulos formam uma hierarquia. O conteúdo de *src/lib.rs* está no
nível superior mais alto, e os submódulos estão em níveis mais baixos. Aqui está
a nossa organização quando pensada de forma hierárquica na Listagem 7-1:

```text
communicator
 ├── network
 └── client
```

E aqui está a hierarquia correspondente ao exemplo na Listagem 7-2:

```text
communicator
 └── network
     └── client
```

Conforme a hierarquia mostrada na Listagem 7-2, `client` é um filho do módulo `network`
em vez de um irmão. Projetos mais complicados podem ter muitos módulos, é necessário 
organizá-los logicamente para mantê-los sob controle. O que "logicamente" significa em 
seu projeto fica a seu critério, e depende do que você e os usuários da sua biblioteca 
pensam sobre o domínio do seu projeto. Use as técnicas mostradas
aqui para criar módulos lado a lado e módulos aninhados em qualquer estrutura que
você queira.

### Movendo Módulos para Outros Arquivos

Os módulos formam uma estrutura hierárquica, bem parecida com outra estrutura computacional 
que você conhece: sistemas de arquivos! Podemos usar o sistema de módulos do Rust juntamente com
vários arquivos para dividir projetos Rust de forma que nem tudo resida em
*src/lib.rs* ou *src/main.rs*. Para este exemplo, vamos começar com o código em
Listagem 7-3:

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

<span class="caption">Listagem 7-3: Três módulos, `client`, `network`, e
`network::server`, todos definidos em *src/lib.rs*</span>

O arquivo *src/lib.rs* possui esta hierarquia de módulos:

```text
communicator
 ├── client
 └── network
     └── server
```

Se esses módulos tivessem muitas funções, e elas estivessem se alongando muito,
seria difícil percorrer esse arquivo para encontrar o código com que queremos
trabalhar. Como as funções estão aninhadas dentro de um ou mais blocos `mod`,
as linhas de código dentro das funções começarão a se alongar também.
Estes seriam bons motivos para separar os módulos `client`, `network`, e `server`
de *src/lib.rs* e colocá-los em seus próprios arquivos.

Primeiro, substitua o código do módulo `client` por apenas a declaração do
módulo `client`, para que seu *src/lib.rs* se pareça com o código mostrado na Listagem 7-4:

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

<span class="caption"> Listagem 7-4: Extraindo o conteúdo do módulo `client`, mas deixando a declaração em *src/lib.rs* </span>

Ainda estamos *declarando* o módulo `client` aqui, mas ao substituir o bloco
por um ponto e vírgula, estamos dizendo ao Rust para que procure, em outro local, o código
definido no escopo do módulo `client`. Em outras palavras, a linha `mod
client;` significa:

```rust,ignore
mod client {
    // conteúdo de client.rs
}
```

Agora precisamos criar o arquivo externo com o nome do módulo. Crie um
arquivo *client.rs* em *src/* e abra-o. Em seguida digite o seguinte,
que é a função `connect` do módulo `client` que foi 
removida na etapa anterior:

<span class="filename">Arquivo: src/client.rs</span>

```rust
fn connect() {
}
```

Observe que não precisamos de uma declaração `mod` neste arquivo porque já fizemos
a declaração do módulo `client` com `mod` em *src/lib.rs*. Este arquivo apenas
fornece o *conteúdo* do módulo `client`. Se colocarmos um `mod client` aqui,
nós estaríamos dando ao módulo `client` seu próprio submódulo chamado `client`!

Rust só sabe olhar em *src/lib.rs* por padrão. Se quisermos adicionar mais
arquivos para o nosso projeto, precisamos dizer ao Rust em *src/lib.rs* para procurar em outros
arquivos; é por isso que `mod client` precisa ser definido em *src/lib.rs* e não pode
ser definido em *src/client.rs*.

Agora, o projeto deve compilar com sucesso, embora você obtenha alguns
warnings (avisos). Lembre-se de usar `cargo build`, em vez de `cargo run`, porque temos
um crate de biblioteca em vez de um crate binário:

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

Esses *warnings* nos dizem que temos funções que nunca são usadas. Não se preocupe
com esses *warnings* por enquanto; vamos abordá-los mais adiante neste capítulo, na
seção “Controlando a visibilidade com `pub`”. A boa notícia é que eles são apenas
*warnings*; nosso projeto foi construído com sucesso!

Em seguida, vamos extrair o módulo `network` em seu próprio arquivo usando o mesmo
procedimento. Em *src/lib.rs*, exclua o corpo do módulo `network` e adicione um
ponto e vírgula à declaração, assim:

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
mod client;

mod network;
```

Em seguida, crie um novo arquivo *src/network.rs* e digite o seguinte:

<span class="filename">Arquivo: src/network.rs</span>

```rust
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

Observe que ainda temos uma declaração `mod` dentro deste arquivo de módulo; isto é
porque ainda queremos que `server` seja um submódulo de `network`.

Execute `cargo build` novamente. Sucesso! Temos mais um módulo para extrair: `server`.
Como ele é um submódulo - ou seja, um módulo dentro de outro - nossa tática atual de 
extrair um módulo para um arquivo com o nome do módulo não funcionará. Iremos
tentar, de qualquer maneira, para que você possa ver o erro. Primeiro, altere o arquivo *src/network.rs* colocando
`mod server;` no lugar do conteúdo do módulo `server`:

<span class="filename">Arquivo: src/network.rs</span>

```rust,ignore
fn connect() {
}

mod server;
```

Em seguida, crie um arquivo *src/server.rs* e insira o conteúdo do módulo `server`
que extraímos:

<span class="filename">Arquivo: src/server.rs</span>

```rust
fn connect() {
}
```

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

<span class="caption">Listagem 7-5: Erro ao tentar extrair o submódulo `server`
 em *src/server.rs* </span>

O erro diz que não podemos declarar um novo módulo neste local (`cannot declare a new module at this location`) 
e está apontando para a linha `mod server`; em *src/network.rs*. Então *src/network.rs* é
diferente de *src/lib.rs* de alguma forma: continue lendo para entender o porquê.

A nota no meio da Listagem 7-5 é realmente muito útil, 
porque ela aponta para algo de que não falamos ainda:

```text
note: maybe move this module `network` to its own directory via
`network/mod.rs`
```

(**Tradução:** talvez mover este módulo network para o seu próprio diretório via `network/mod.rs`)

Em vez de continuar a seguir o mesmo padrão de nomeação de arquivo usado
anteriormente, podemos fazer o que a nota sugere:

1. Crie um novo *diretório* chamado *network*, o nome do módulo pai.
2. Mova o arquivo *src/network.rs* para o novo diretório *network* e
    renomeie para *src/network/mod.rs*.
3. Mova o arquivo de submódulo *src/server.rs* para o diretório *network*.

Aqui estão os comandos para executar estas etapas:

```text
$ mkdir src/network
$ mv src/network.rs src/network/mod.rs
$ mv src/server.rs src/network
```

Agora, quando tentamos executar `cargo build`, a compilação funcionará (embora ainda teremos
avisos). O layout dos nossos módulos ainda é exatamente o 
mesmo de quando tínhamos todo o código em *src/lib.rs* na Listagem 7-3:

```text
communicator
 ├── client
 └── network
     └── server
```

O layout dos arquivos correspondentes agora ficou assim:

```text
├── src
│   ├── client.rs
│   ├── lib.rs
│   └── network
│       ├── mod.rs
│       └── server.rs
```

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

Neste exemplo, temos novamente três módulos : `client`,` network`, e
`network::client`. Seguindo os mesmos passos anteriores para extrair
módulos em arquivos, poderíamos criar *src/client.rs* para o módulo `client`.
Para o módulo `network`, poderíamos criar *src/network.rs*. Mas não seríamos
capazes de extrair o módulo `network::client` para um arquivo *src/client.rs*
porque ele já existe para o módulo `client` de nível superior! Se pudéssemos colocar
o código para *ambos* os módulos `client` e` network::client` no arquivo
*src/client.rs*, Rust não teria nenhuma maneira de saber se o código era
para `client` ou para `network::client`.

Portanto, para extrair um arquivo para o submódulo `network::client` do
módulo `network`, precisamos criar um diretório para o módulo `network`
em vez de um arquivo *src/network.rs*. O código que está no módulo `network`
entra no arquivo *src/network/mod.rs*, e o submódulo
`network::client` pode ter seu próprio arquivo *src/network/client.rs*. Agora o
o nível superior *src/client.rs* é inequivocamente o código que pertence ao
módulo `client`.

### Regras dos Módulos e Seus Arquivos

Vamos resumir as regras dos módulos em relação aos arquivos:

* Se um módulo chamado `foo` não possui submódulos, você deve colocar as declarações
   para `foo` em um arquivo chamado *foo.rs*.
* Se um módulo chamado `foo` possui submódulos, você deve colocar as declarações
   para `foo` em um arquivo chamado *foo/mod.rs*.

Essas regras se aplicam de forma recursiva, então, se um módulo chamado `foo` tiver um submódulo chamado
`bar` e` bar` não possui submódulos, você deve ter os seguintes arquivos
no seu diretório *src*:


```text
├── foo
│   ├── bar.rs (contém as declarações em `foo::bar`)
│   └── mod.rs (contém as declarações em `foo`, incluindo `mod bar`)
```

Os módulos devem ser declarados no arquivo do módulo pai usando a palavra-chave `mod`.

Em seguida, vamos falar sobre a palavra-chave `pub` e nos livrar dessas warnings!
