# Funções

Funções são difundidas em códigos em Rust. Você já viu uma das mais
importantes funções da linguagem: a função `main`, que é o
ponto de entrada de diversos programas. Você também já viu a notação `fn`, que permite você
declarar uma nova função.

Códigos em Rust usam, por convenção, o estilo *snake case* para nomes de função e
variável. No snake case, todas as letras são minúsculas e sublinhado (underline) separa as palavras.
Aqui está um programa que contém uma definição de função de exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    println!("Olá, mundo!");

    outra_funcao();
}

fn outra_funcao() {
    println!("Outra função.");
}
```

As definições de funções em Rust iniciam com `fn` e tem um par de parênteses
depois do nome da função. As chaves dizem ao compilador onde o
corpo da função começa e termina.

Podemos chamar qualquer função que tenhamos definido, inserindo seu nome, seguido de um
conjunto de parenteses. Pelo fato da `outra_funcao` ter sido definida no programa, ela pode
ser chamada dentro da função `main`. Note que definimos `outra_funcao`
*depois* da função `main`; poderíamos ter definido antes
também. Rust não se importa onde você definiu suas funções, apenas que elas foram
definidas em algum lugar.

Vamos começar um novo projeto binário, chamado *funcoes* para explorar mais
funções. Coloque o exemplo `outra_funcao` em *src/main.rs* e execute-o. Você
verá a seguinte saída:

```text
$ cargo run
   Compiling funcoes v0.1.0 (file:///projects/funcoes)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28 secs
     Running `target/debug/funcoes`
Olá, mundo!
Outra função.
```

As linhas são executadas na ordem em que aparecem na função `main`.
Primeiro, a mensagem "Olá, mundo!" é exibida, e então
`outra_funcao` é chamada e exibida a mensagem.

### Parâmetros de função

Funções também podem ser definidas tendo *parâmetros*, que são variáveis especiais
que fazem parte da assinatura da função. Quando uma função tem parâmetros, você
pode fornecer tipos específicos para esses parâmetros. Tecnicamente, os
valores definidos são chamados de *argumentos*, mas informalmente, as pessoas tendem
a usar as palavras *parâmetro* e *argumento* para falar tanto de
variáveis da definição da função como os valores passados quando você
chama uma função.

A seguinte versão (reescrita) da `outra_funcao` mostra como os parâmetros
aparecem no Rust:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    outra_funcao(5);
}

fn outra_funcao(x: i32) {
    println!("O valor de x é: {}", x);
}
```

Tente executar este programa; você verá a seguinte saída:

```text
$ cargo run
   Compiling funcoes v0.1.0 (file:///projects/funcoes)
    Finished dev [unoptimized + debuginfo] target(s) in 1.21 secs
     Running `target/debug/funcoes`
O valor de x é: 5
```

A declaração de `outra_funcao` tem um parâmetro chamado `x`. O tipo do
`x` é especificado como `i32`. Quando `5` é passado para a `outra_funcao`, a macro
`println!` coloca `5` onde o par de chaves estava na string
de formato.

Nas assinaturas de função, você *deve* declarar o tipo de cada parâmetro. Essa é
decisão deliberada no design do Rust: exigir anotações de tipo na definição da função,
significa que o compilador quase nunca precisará que as use em outro lugar do código
para especificar o que você quer.

Quando você precisa que uma função tenha vários parâmetros, separe as declarações de parâmetros
com vírgula, como a seguir:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    outra_funcao(5, 6);
}

fn outra_funcao(x: i32, y: i32) {
    println!("O valor de x é: {}", x);
    println!("O valor de y é: {}", y);
}
```

Este exemplo cria uma função com dois parâmetros, ambos com o 
tipo `i32`. Então a função exibe os valores de ambos os parâmetros. Note que os
parâmetros de função não precisam ser do mesmo tipo, isto apenas
aconteceu neste exemplo.

Vamos tentar executar este código. Substitua o programa *src/main.rs*, atualmente em seu projeto *funcoes*
com o exemplo anterior e execute-o usando `cargo
run`:

```text
$ cargo run
   Compiling funcoes v0.1.0 (file:///projects/funcoes)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/funcoes`
O valor de x é: 5
O valor de y é: 6
```

Porque nós chamamos a função com `5` sendo o valor de` x` e `6` é passado
como o valor de `y`, as duas cadeias são impressas com esses valores.

### Corpos de função

Corpos de função são constituídos por uma série de declarações que terminam,
opcionalmente, em uma expressão. Até agora, foram apresentadas apenas funções sem uma expressão final,
mas você viu uma expressão como parte de instruções. Porque Rust é uma
linguagem baseada em expressão, essa é uma importante distinção a ser entendida.
Outras linguagens não têm as mesmas distinções, então, vamos ver o que são
declarações e expressões e como elas afetam o corpo
das funções.

### Declarações e Expressões

Na verdade, já usamos declarações e expressões. *Declarações* são
instruções que executam alguma ação e não retornam um resultado. E *expressões*
retornam um resultado. Vamos ver alguns exemplos.

Criar uma variável e atribuir um valor a ela com a palavra-chave `let` é uma declaração.
Na Listagem 3-1, `let y = 6;` é uma declaração:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let y = 6;
}
```

<span class="caption">Listagem 3-1: A declaração da função `main` contendo uma declaração.</span>

Definições de função também são definições; todo o exemplo
é uma declaração em si.

Definições não retornam valores. Assim sendo, você não pode atribuir uma declaração `let` para
outra variável, como o código a seguir tenta fazer; você receberá um erro:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let x = (let y = 6);
}
```

Quando você rodar esse programa, o erro será o seguinte:

```text
$ cargo run
   Compiling funcoes v0.1.0 (file:///projects/funcoes)
error: expected expression, found statement (`let`)
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^
  |
  = note: variable declaration using `let` is a statement
```

A declaração `let y = 6` não retorna um valor, então não existe nada para o
`x` se ligar. Isso é diferente do que acontece em
outras linguagens, como
C e Ruby, onde a atribuição retorna o valor atribuído. Nestas linguagens, você pode escrever `x = y = 6` e ter ambos, `x` e `y` contendo o valor
`6`; esse não é o caso em Rust.

Expressões avaliam algo e compõem a maior parte do código que
você escreverá em Rust. Considere uma simples operação matemática, como um `5 + 6`, que
é uma expressão que avalia o valor `11`. Expressões podem fazer parte de
declarações: na Listagem 3-1, o `6` na declaração `let y = 6;` é uma
expressão que avalia o valor `6`. A chamada de função é uma
expressão. Chamar uma macro é uma expressão. O bloco que vamos usar para criar
um novo escopo, `{}`, é uma expressão, por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let x = 5;

    let y = {
        let x = 3;
        x + 1
    };

    println!("O valor de y é: {}", y);
}
```

A expressão:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

é um bloco que, nesse exemplo, avalia `4`. Esse valor fica vinculado ao `y`
como parte da declaração `let`. Note o  `x + 1` sem um ponto e vírgula no final, que é diferente da maioria das linhas vistas até agora.
Expressões não terminam com ponto e vírgula.
Se você adicionar um ponto e vírgula ao fim de uma expressão,
você a transforma em uma declaração, que então não retornará um valor.
Tenha isso em mente, enquanto explora os valores e expressões de retorno da função a seguir.

### Funções com valor de retorno

Funções podem retornar valores para o código que os chama. Não nomeamos valores de
retorno, mas declaramos o tipo deles depois de uma seta (`->`). Em Rust, o valor de retorno
da função é sinônimo do valor da expressão
final no bloco do corpo de uma função. Você pode retornar cedo de uma função usando
a palavra-chave `return` e especificando um valor, mas a maioria das funções retorna
a última expressão implicitamente. Veja um exemplo de uma função que retorna um
valor:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn cinco() -> i32 {
    5
}

fn main() {
    let x = cinco();

    println!("O valor de x é: {}", x);
}
```

Não há chamadas de função, macros ou até mesmo declarações `let` na função` cinco`
- apenas o número `5` por si só. Essa é uma função perfeitamente válida em
Rust. Observe que o tipo de retorno da função também é especificado como `-> i32`. Tente
executar este código; a saída deve ficar assim:

```text
$ cargo run
   Compiling funcoes v0.1.0 (file:///projects/funcoes)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/funcoes`
O valor de x é: 5
```

O `5` em` cinco` é o valor de retorno da função, e é por isso que o tipo de retorno
é `i32`. Vamos verificar isso com mais detalhes. Existem dois bits importantes:
primeiro, a linha `let x = cinco ();` mostra que estamos usando o valor de retorno de uma
função para inicializar uma variável. Porque a função `cinco` retorna um` 5`,
essa linha é a mesma que a seguinte:

```rust
let x = 5;
```

Em segundo lugar, a função `cinco` não tem parâmetros e define o tipo de
valor de retorno, mas o corpo da função é um `5` solitário sem ponto e vírgula
porque é uma expressão cujo valor queremos retornar.

Vamos ver outro exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let x = soma_um(5);

    println!("O valor de x é: {}", x);
}

fn soma_um(x: i32) -> i32 {
    x + 1
}
```

A execução deste código irá imprimir `O valor de x é: 6`. Mas se colocarmos um
ponto e vírgula no final da linha que contém `x + 1`, alterando-o de
expressão para uma declaração, receberemos um erro.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let x = soma_um(5);

    println!("O valor de x é: {}", x);
}

fn soma_um(x: i32) -> i32 {
    x + 1;
}
```

Executar este código produz um erro, da seguinte maneira:

```text
error[E0308]: mismatched types
 --> src/main.rs:7:28
  |
7 |   fn soma_um(x: i32) -> i32 {
  |  ____________________________^
8 | |     x + 1;
  | |          - help: consider removing this semicolon
9 | | }
  | |_^ expected i32, found ()
  |
  = note: expected type `i32`
             found type `()`
```

A principal mensagem de erro, "tipos incompatíveis", revela o problema central com este
código. A definição da função `soma_um` diz que retornará uma
`i32`, mas as declarações não avaliam um valor expresso por` () `,
a tupla vazia. Portanto, nada é retornado, o que contradiz a função
definição e resulta em erro. Nesta saída, Rust fornece uma mensagem para
possivelmente ajudar a corrigir este problema: sugere a remoção do ponto e vírgula, que
iria corrigir o erro.
