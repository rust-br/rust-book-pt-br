## Referências e _Borrowing_

O problema de usar tuplas, que vimos no fim da seção anterior, é que precisamos
retornar a `String`, de forma que ainda possamos usá-la após a chamada à função
`calcula_tamanho`, para dentro da qual a `String` foi movida.

Aqui está uma forma de como você poderia definir e usar uma função
`calcula_tamanho` que recebe uma *referência* para um objeto como parâmetro, em
vez de pegar este valor para si:

<span class="filename">Arquivo: src/main.rs</span>

```rust
fn main() {
    let s1 = String::from("texto");

    let tamanho = calcula_tamanho(&s1);

    println!("O tamanho de '{}' é {}.", s1, tamanho);
}

fn calcula_tamanho(s: &String) -> usize {
    s.len()
}
```

Primeiro, repare que todo aquele código usando uma tupla na declaração da
variável e no retorno da função já se foi. Segundo, note que passamos `&s1` para
`calcula_tamanho`, e na sua definição, temos `&String` em vez de apenas
`String`.

Esses `&` são *referências*, e eles permitem que você se refira a algum valor
sem tomar posse dele. A Figura 4-5 mostra um diagrama.

<img alt="&String s apontando para String s1" src="img/trpl04-05.svg" class="center" />

<span class="caption">Figura 4-5: `&String s` apontando para `String s1`</span>

> Nota: O oposto de referenciar usando `&` é *derreferenciar*, feito por meio do
> operador derreferenciador, `*`. Veremos alguns usos do operador
> derreferenciador no Capítulo 8 e vamos discutir detalhes da derreferenciação
> no Capítulo 15.

Vamos olhar mais de perto esta chamada de função:

```rust
# fn calcula_tamanho(s: &String) -> usize {
#     s.len()
# }
let s1 = String::from("texto");

let tamanho = calcula_tamanho(&s1);
```

A sintaxe `&s1` nos permite criar uma referência que *se refere* ao valor `s1`,
mas não o possui. Como ela não o possui, o valor a que ela aponta não será
destruído quando a referência sair de escopo.

Da mesma forma, a assinatura da função usa `&` para indicar que o tipo do
parâmetro `s` é uma referência. Vamos adicionar algumas anotações para explicar:

```rust
fn calcula_tamanho(s: &String) -> usize { // s é uma referência para uma String
    s.len()
} // Aqui, s sai de escopo. Mas como ela não possui o valor a que se refere,
  // nada acontece.
```

O escopo no qual a variável `s` é válida é o mesmo escopo de qualquer parâmetro
de função, mas não destruímos o valor apontado pela referência quando ela sai de
escopo, pois ela não tem posse dele. Funções que têm referências como
parâmetros, em vez dos próprios valores, não precisam retornar os valores para
devolver a posse deles, já que nunca tiveram esta posse.

Colocar referências como parâmetros de funções é chamado de *borrowing* (do
inglês, empréstimo). Assim como na vida real, se uma pessoa possui alguma coisa,
você pode pegar emprestado dela. Quando você termina de usar, você deve
devolver.

E o que acontece se tentarmos modificar alguma coisa que pegamos emprestado?
Tente rodar o código da Listagem 4-4. Alerta de spoiler: não funciona!

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let s = String::from("texto");

    modifica(&s);
}

fn modifica(uma_string: &String) {
    uma_string.push_str(" longo");
}
```

<span class="caption">Listagem 4-4: Tentativa de modificar um valor emprestado</span>

Aqui está o erro:

```text
error[E0596]: cannot borrow immutable borrowed content `*uma_string` as mutable
 --> main.rs:8:5
  |
7 | fn modifica(uma_string: &String) {
  |                         ------- use `&mut String` here to make mutable
8 |     uma_string.push_str(" longo");
  |     ^^^^^^^^^^ cannot borrow as mutable
```

Assim como as variáveis são imutáveis por padrão, referências também são. Não
temos permissão para modificar algo para o qual temos uma referência.

### Referências Mutáveis

Podemos corrigir o erro no código da Listagem 4-4 com um pequeno ajuste:

<span class="filename">Arquivo: src/main.rs</span>

```rust
fn main() {
    let mut s = String::from("texto");

    modifica(&mut s);
}

fn modifica(uma_string: &mut String) {
    uma_string.push_str(" longo");
}
```

Primeiro, temos que fazer com que `s` seja `mut`. Depois, temos que criar uma
referência mutável com `&mut s` e aceitar uma referência mutável com
`uma_string: &mut String`.

Mas referências mutáveis possuem uma grande restrição: você só pode ter uma
referência mutável para um determinado dado em um determinado escopo. Este
código vai falhar:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
let mut s = String::from("texto");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);
```

Aqui está o erro:

```text
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> main.rs:5:19
  |
4 |     let r1 = &mut s;
  |                   - first mutable borrow occurs here
5 |     let r2 = &mut s;
  |                   ^ second mutable borrow occurs here
6 | }
  | - first borrow ends here
```

Esta restrição permite a mutação, mas de uma forma bem controlada. Isto é algo
com que novos Rustáceos passam trabalho, porque a maioria das linguagens de
programação permitem modificar um valor quando você quiser. O benefício de ter
esta restrição é que o Rust previne _data races_ em tempo de compilação. 

Um *data race* é parecido com uma condição de corrida, e acontece quando esses
três fatores ocorrem:

1. Dois ou mais ponteiros acessam o mesmo dado ao mesmo tempo.
1. Ao menos um dos ponteiros é usado para escrever sobre o dado.
1. Não há nenhum mecanismo sendo usado para sincronizar o acesso ao dado.

Data races causam comportamento indefinido e pode ser difíceis de diagnosticar
e corrigir quando você está tentando rastreá-los em tempo de execução. Rust
previne este problema de acontecer porque não vai nem deixar compilar um código
com data races!

Como sempre, podemos usar chaves (`{}`) para criar um novo escopo, permitindo
múltiplas referências mutáveis, mas não *simultâneas*:

```rust
let mut s = String::from("texto");

{
    let r1 = &mut s;

} // aqui r1 sai de escopo, então já podemos criar uma nova referência sem
  // problema nenhum.

let r2 = &mut s;
```

Existe uma regra parecida para combinar referências mutáveis e imutáveis. Este
código resulta em erro:

```rust,ignore
let mut s = String::from("texto");

let r1 = &s; // sem problema
let r2 = &s; // sem problema
let r3 = &mut s; // PROBLEMA GRANDE

println!("{}, {}, and {}", r1, r2, r3);
```

Aqui está o erro:

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as
immutable
 --> main.rs:6:19
  |
4 |     let r1 = &s; // sem problema
  |               - immutable borrow occurs here
5 |     let r2 = &s; // sem problema
6 |     let r3 = &mut s; // PROBLEMA GRANDE
  |                   ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

Eita! Nós *também* não podemos ter uma referência mutável enquanto temos uma
imutável. Usuários de uma referência imutável não esperam que os valores mudem
de repente! Porém, múltiplas referências imutáveis são permitidas, pois ninguém
que esteja apenas lendo os dados será capaz de afetar a leitura que está sendo
feita em outra parte do código.

Observe que o escopo de uma referência começa de onde é inserida e continua até
a última vez que a referência foi usada. Por exemplo, este código será compilado
porque o último uso das referências imutáveis, ocorrem antes da referência mutável
ser inserida:

```rust,ignore
let mut s = String::from("texto");

let r1 = &s; // sem problema
let r2 = &s; // sem problema
println!("{} and {}", r1, r2);
// r1 e r2 não serão mais utilizadas a partir deste ponto

let r3 = &mut s; // sem problema
println!("{}", r3);
```
O escopo das referências imutáveis `r1` e `r2` terminam após o `println!` onde eles
são utilizados pela última vez, que é antes da referência mutável `r3` ser criada.
Esses escopos não se sobrepõem, portanto, este código é permitido.

Mesmo que esses erros sejam frustrantes às vezes, lembre-se que é o compilador
do Rust apontando um bug potencial antecipadamente (em tempo de compilação,
em vez de execução), e mostrando exatamente onde está o problema, em vez de você
ter que investigar por que algumas vezes os seus dados não são aquilo que você
esperava que fosse.

### Referências Soltas

Em linguagens com ponteiros, é fácil criar erroneamente um *ponteiro solto*, um
ponteiro que referencia um local na memória que pode ter sido dado para outra
parte do programa, basta liberar alguma memória e preservar um ponteiro para
ela. Por outro lado, em Rust, o compilador garante que nenhuma referência será
uma referência solta: se temos uma referência para algum dado, o compilador vai
se certificar que esse dado não vai sair de escopo antes da referência.

Vamos tentar criar uma referência solta, que o Rust vai impedir com um erro em
tempo de compilação:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let referencia_para_o_nada = soltar();
}

fn soltar() -> &String {
    let s = String::from("texto");

    &s
}
```

Aqui está o erro:

```text
error[E0106]: missing lifetime specifier
 --> main.rs:5:16
  |
5 | fn soltar() -> &String {
  |                ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is
  no value for it to be borrowed from
  = help: consider giving it a 'static lifetime
```

Esta mensagem de erro se refere a uma característica que não abordamos ainda:
*lifetimes*. Vamos discutir lifetimes em detalhe no Capítulo 10. Mas, se você
desconsiderar a parte sobre lifetimes, a mensagem mostra a razão deste código
ser um problema:

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from.
```

> Tradução: o tipo de retorno desta função contém um valor emprestado, mas não
> há nenhum valor que se possa pegar emprestado.

Vamos dar uma olhada mais de perto no que está acontecendo, exatamente, em cada
estágio da nossa função `soltar`:

```rust,ignore
fn soltar() -> &String { // soltar retorna uma referência a uma String

    let s = String::from("texto"); // s é uma nova String

    &s // retornamos uma referência a uma String, s
} // Aqui, s sai de escopo e é destruída. Sua memória é devolvida.
  // Perigo!
```

Como `s` é criada dentro da função `soltar`, quando o código desta função
termina, `s` é desalocada. Mas nós tentamos retornar uma referência para ela.
Isto significa que esta referência apontaria para uma `String` inválida! Isso
não é bom. Rust não vai nos deixar fazer isso.

A solução aqui é retornar a `String` diretamente:

```rust
fn nao_soltar() -> String {
    let s = String::from("texto");

    s
}
```

Isto funciona sem nenhum problema. A `String` é movida para fora, e nada é
desalocado.

### As Regras de Referências

Vamos recapitular o que discutimos sobre referências:

1. Em um dado momento, você pode ter *um ou outro*, mas não os dois:
  * Uma referência mutável.
  * Qualquer número de referências imutáveis.
2. Referências devem ser válidas sempre.

Em seguida, vamos ver um tipo diferente de referências: _slices_.
