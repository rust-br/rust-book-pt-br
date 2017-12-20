## Referindo-se a Nomes em Módulos Diferentes

Vimos como chamar funções definidas dentro de um módulo usando o nome do módulo
como parte da chamada, como na chamada para a função `nested_modules` mostrada
aqui na Listagem 7-7:

<span class="filename">Arquivo: src/main.rs</span>

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

<span class="caption">Listagem 7-7: Chamando uma função especificando completamente 
o caminho do módulo que a cerca</span>


Como você pode ver, referir-se ao nome totalmente qualificado pode ficar bastante longo.
Felizmente, Rust tem uma palavra-chave para tornar estas chamadas mais concisas.

### Trazendo Nomes no Escopo com a Palavra-Chave `use`

A palavra-chave `use` de Rust encurta as chamadas de função longas, trazendo os módulos e
a função que deseja chamar para o escopo. Aqui está um exemplo de como trazer o
módulo `a::series::of` para dentro do escopo raiz de um crate binário:

<span class="filename">Arquivo: src/main.rs</span>

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

A linha `use a::series::of;` significa que, em vez de usar todo o caminho
`a::series::of` sempre que quisermos nos referir ao módulo `of`, podemos usar
`of`.

A palavra-chave `use` traz apenas o que especificamos no escopo: ela não
leva os filhos dos módulos ao escopo. É por isso que ainda temos que usar
`of::nested_modules` quando queremos chamar a função `nested_modules`.

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

Isso nos permite excluir todos os módulos e fazer referência à função
diretamente.

Como as enums também formam uma espécie de *namespace*, assim como os módulos, podemos trazer
as variantes de uma enum para o escopo com `use` também. Para qualquer tipo de declaração de `use`
se você estiver trazendo vários itens de um *namespace* para o escopo, você pode listá-los
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

Nós ainda estamos especificando o *namespace* `TrafficLight` para a variante `Green`
porque não incluímos `Green` na declaração `use`.

### Trazendo Todos os Nomes para o Escopo com um Glob

Para trazer todos os itens de um *namespace* para o escopo ao mesmo tempo, podemos usar a sintaxe `*`, que é chamada de *operador glob*. Este exemplo traz todas as variantes de uma enum ao escopo sem ter que listar cada uma especificamente:

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

O `*` trará para o escopo todos os itens visíveis no *namespace* `TrafficLight`. 
Você deve usar globs com moderação: eles são convenientes, mas isso pode
também trazer mais itens do que se esperava e causar conflitos de nomeação.

### Usando `super` para Acessar um Módulo Pai

Como vimos no início deste capítulo, quando você cria um crate de biblioteca,
o Cargo faz um módulo `tests` para você. Vamos ver isso em mais detalhes agora.
No seu projeto `communicator`, abra *src/lib.rs*:

<span class="filename">Arquivo: src/lib.rs</span>

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

O Capítulo 11 explica mais sobre testes, mas algumas partes deste exemplo devem fazer
sentido agora: temos um módulo chamado `tests` que se situa ao lado de nossos outros módulos
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

Os testes servem para exercitar o código dentro da nossa biblioteca, então vamos tentar chamar nossa
função `client :: connect` a partir da função` it_works`, mesmo que não verefiquemos nenhuma 
funcionalidade agora. Isso ainda não funcionará:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        client::connect();
    }
}
```

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

A compilação falhou, mas por quê? Não precisamos colocar `communicator ::` em
frente da função como fizemos em *src/main.rs* porque estamos definitivamente
dentro do crate da biblioteca `communicator` aqui. A razão é que os caminhos são
sempre relativos ao módulo atual, que aqui é `tests`. A única
exceção está em uma instrução `use`, onde os caminhos são relativos à crate raiz
por padrão. Nosso módulo `tests` precisa do módulo `client` no seu escopo!

Então, como podemos retroceder um módulo na hierarquia para chamar a
função `client::connect` no módulo `tests`? No módulo `tests`, temos a opção de
usar `::` na frente para indicar ao Rust que queremos começar a partir da raiz
e listar todo o caminho, assim:

```rust,ignore
::client::connect();
```

Ou, podemos usar `super` para voltar um módulo na hierarquia a partir de nosso 
módulo atual, assim:

```rust,ignore
super::client::connect();
```

Essas duas opções não parecem tão diferentes neste exemplo, mas se você estiver
mais fundo em uma hierarquia de módulos, começar sempre a partir da raiz tornaria
seu código muito longo. Nesses casos, usar `super` para ir do módulo atual aos
módulos irmãos é um bom atalho. Além disso, se você especificou o caminho a partir da
raiz em muitos lugares do seu código e depois vai reorganizar seus módulos movendo
uma sub-árvore para outro lugar, você acabaria precisando atualizar o caminho em vários
lugares, o que seria tedioso.

Também seria chato ter que digitar `super ::` em cada teste, mas você
já viu a ferramenta para essa solução: `use`! A funcionalidade `super ::`
altera o caminho que você dá para `use`, tornando-o relativo ao módulo pai
em vez do módulo raiz.

Por estas razões, especialmente no módulo `tests`, `use super::alguma_coisa` é
geralmente a melhor solução. Então, agora nosso teste ficou assim:

<span class="filename">Arquivo: src/lib.rs</span>

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

Quando executarmos novamente `cargo test`, o teste passará e a primeira parte do
resultado do teste será o seguinte:

```text
$ cargo test
   Compiling communicator v0.1.0 (file:///projects/communicator)
     Running target/debug/communicator-92007ddb5330fa5a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

## Resumo

Agora você conhece algumas técnicas novas para organizar o seu código! Use estas técnicas
para agrupar as funcionalidades relacionadas, evitar que os arquivos tornem-se muito longos, e
apresentar uma API pública arrumada para os usuários da sua biblioteca.

Em seguida, analisaremos algumas estruturas de dados de coleções na biblioteca padrão
que você pode usar em seu código limpo e elegante!

