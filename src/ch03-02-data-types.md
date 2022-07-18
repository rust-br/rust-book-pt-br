# Tipos de dados

Todo valor em Rust é um *tipo de dado*, que informa ao Rust que tipos de
dados estão sendo especificados para que saiba como trabalhar com esses dados. Vamos olhar para
dois subconjuntos de tipos de dados: escalar e composto.

Tenha em mente que Rust é uma linguagem de *tipagem estática*, o que significa
que deve conhecer os tipos de todas as variáveis em tempo de compilação. O compilador
geralmente pode inferir que tipo queremos com base no valor e como o usamos. Nos casos
em que são é possível vários tipos de dados, como quando convertemos uma `String` em um tipo numérico
usando `parse` na seção "Comparando o Adivinha ao Número Secreto" no
Capítulo 2, devemos adicionar uma anotação de tipo, como a seguinte:

```rust
let guess: u32 = "42".parse().expect("Não é um número!");
```

Se não adicionarmos uma anotação de tipo, Rust irá mostrar o seguinte erro,
que significa que o compilador precisa de mais informaçoes para saber qual tipo de dados
queremos usar:

```text
error[E0282]: type annotations needed
 --> src/main.rs:2:9
  |
2 |     let guess = "42".parse().expect("Não é um número!");
  |         ^^^^^
  |         |
  |         cannot infer type for `_`
  |         consider giving `guess` a type
```

Você verá anotações de tipos diferentes para outros tipos de dados.

### Tipos escalares

Um tipo *escalar* representa um valor único. Rust tem quatro tipos escalares primários:
inteiros, números de ponto flutuante, booleanos e caracteres. Você pode reconhecer
esses tipos de outras linguagens de programação. Vamos pular para como eles funcionam no Rust.

### Tipos inteiros

Um *inteiro* é um número sem a parte fracionária. Usamos
um tipo inteiro no Capítulo 2, o tipo `u32`. Esse tipo de
declaração indica que
o valor associado deve ser um inteiro sem sinal (tipos inteiros com sinal começam com `i`, em vez de `u`) que ocupa 32 bits de espaço. Tabela 3-1 mostra
os tipos inteiros internos ao Rust. Cada variante está na
coluna com sinal e sem sinal (por exemplo, `i16`) pode ser usada para declarar um valor do tipo
inteiro.

<span class="caption">Tabela 3-1: Tipos inteiros no Rust</span>

| Tamanho | Signed  | Unsigned |
|---------|---------|----------|
| 8-bit   | `i8`    | `u8`     |
| 16-bit  | `i16`   | `u16`    |
| 32-bit  | `i32`   | `u32`    |
| 64-bit  | `i64`   | `u64`    |
| arch    | `isize` | `usize`  |

Cada variante pode ser com ou sem sinal e ter tamanho explícito.
*Signed* e *unsigned* refere-se à possibilidade do número ser
negativo ou positivo - em outras palavras, se o número precisa de um sinal
com ele (signed) ou se sempre for
positivo pode ser representado sem um sinal (unsigned). É como escrevemos números no papel: Quando
o sinal importa, o número é mostrado com um sinal de mais ou menos; contudo,
quando é seguro assumir que o número é positivo, é mostrado sem sinal.
Números com sinais são armazenados usando a representação complemento de dois (se você não tiver
certeza do que é isso, você pode procurar sobre isso na internet; uma explicação está fora do escopo
deste livro).

Cada variante com sinal pode armazenar números de -(2<sup>n - 1</sup>) até 2<sup>n -
1</sup> - 1 incluso, sendo *n* o número de bits que varia de acordo com o uso. Então, um
`i8` pode armazenar números de -(2<sup>7</sup>) até 2<sup>7</sup> - 1, que é  igual
a -128 até 127. Variantes sem sinal pode armazenar números de 0 até 2<sup>n</sup> - 1,
entao um `u8` pode armazenar números de 0 até 2<sup>8</sup> - 1, que é de 0 até 255.

Além disso, os tipos `isize` e `usize` dependem do computador em que seu programa
está rodando: 64 bits se estiver em uma arquitetura de 64-bit e 32 bits
se sua arquitetura for 32-bit.

Você pode criar inteiros literais em qualquer uma das formas mostrada na Tabela 3-2. Observe
que todos os literais de números, exceto o byte literal, permitem um sufixo de tipo,
como por exemplo, `57u8` e `_` são separadores visuais, tal como `1_000`.

<span class="caption">Tabela 3-2: Inteiros Literais no Rust</span>

| Números literais    | Exemplo       |
|---------------------|---------------|
| Decimal             | `98_222`      |
| Hexadecimal         | `0xff`        |
| Octal               | `0o77`        |
| Binário             | `0b1111_0000` |
| Byte (`u8` apenas)  | `b'A'`        |

Então como você pode saber qual tipo de inteiro usar? Se sentir-se inseguro, as
escolhas padrões do Rust geralmente são boas, e por padrão os inteiros são do tipo `i32`: Esse
tipo geralmente é o mais rápido, até em sistemas de 64-bit. A
principal situação em que você usuaria `isize` ou `usize` é indexar algum tipo de coleção.

#### Tipos de ponto flutuante

Rust também tem dois tipos primitivos para *números de ponto flutuante*, que são
números com casas decimais. Os pontos flutuantes do Rust são
`f32` e `f64`, que têm respectivamente os tamanhos de 32 e 64 bits. O tipo padrão é `f64`
porque nos processadores modernos, a velocidade é quase a mesma que em um `f32`, mas possui
maior precisão.

Esse exemplo mostra números de ponto flutuante em ação:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let x = 2.0; // f64

    let y: f32 = 3.0; // f32
}
```

Números em ponto flutuante são representados de acordo com o padrão IEEE-754. O tipo
`f32` é de precisão simples e `f64` tem precisão dupla.

#### Operações numéricas

Rust suporta operações matemáticas básicas, você pode esperar
todas as seguintes operações para todos os tipos numéricos: adição, subtração, multiplicação, divisão e resto.
O código a seguir mostra como usar cada declaração `let`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    // adição
    let soma = 5 + 10;

    // subtração
    let diferenca = 95.5 - 4.3;

    // multiplicação
    let produto = 4 * 30;

    // divisão
    let quociente = 56.7 / 32.2;

    // resto
    let resto = 43 % 5;
}
```

Cada expressão nessas declarações, usa um operador matemático e computa um único valor,
que então é atribuído à uma variável.
Apêndice B contém uma lista de todos os operadores que o Rust suporta.

#### O tipo booleano

Como em diversas linguagens de programação, o tipo Booleano em Rust possue dois valores
possíveis: `true` e `false`. O tipo Booleano no Rust é especificado usando `bool`.
Por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let t = true;

    let f: bool = false; // com tipo explícito
}
```

A principal utilização de valores Booleanos é através dos condicionais, como um `if`.
Veremos como a expressão `if` funciona em Rust na seção
"Controle de fluxo".

#### O tipo de caractere

Até agora trabalhamos apenas com números, mas Rust também suporta letras. O `char`
é o tipo mais primitivo da linguaguem e o seguinte código
mostra uma forma de utilizá-lo. (Observe que o `char` é
específicado com aspas simples, é o oposto de strings, que usa aspas duplas.)

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let c = 'z';
    let z = 'ℤ';
    let heart_eyed_cat = '😻';
}
```

O tipo `char` representa um valor unicode, o que quer dizer que você pode
armazenar muito mais que apenas ASCII. Letras com acentuação; ideogramas chinês, japonês e
coreano; emoji; e caracteres não visíveis são válidos.
Valores Unicode vão de `U+0000` até `U+D7FF` e `U+E000` até
`U+10FFFF` incluso. Contudo, um "caractere" não é realmente um conceito em Unicode,
então a sua intuição de o que é um "caractere" pode não combinar com o que é um
`char` em Rust. Discutiremos esse tópico em detalhes em "Strings" no Capítulo 8.

### Tipos compostos

*Tipos compostos* podem agrupar vários valores em um único tipo. Rust tem dois
tipos primitivos compostos: tuplas e vetores.

#### O tipo tupla de valores

Uma tupla é de modo geral uma forma de agrupar um certo número de valores
com uma variável do tipo composto.

Criamos uma tupla escrevendo uma lista de valores separados por vírgula
dentro de parênteses. Cada posição da tupla tem um tipo e os tipos dos elementos
da tupla não necessitam serem iguais.
Adicionamos anotações de tipo neste exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

A variável `tup` liga-se a tupla, porque uma tupla é considerada
um único elemento composto. Para pegar os valores da tupla individualmente, podemos usar
a correspondência de padrões para desestruturar o valor de uma tupla, como este:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("O valor do y é: {}", y);
}
```

Esse primeito programa cria uma tupla e vincula ela à variável `tup`. Em seguida,
ele usa um padrão com `let` para tirar `tup` e tranformá-lo em três variáveis
separadas, `x`, `y` e `z`. Isso é chamado de *desestruturação*, porque quebra uma única tupla
em três partes. Finalmente, o programa exibe o valor de `y`,
que é `6.4`.

Além de desestruturar através da correspondência de padrões, podemos
acessar diretamente um elemento da tupla usando um ponto (`.`) como índice
do valor que queremos acessar. Por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let quinhentos = x.0;

    let seis_ponto_quatro = x.1;

    let um = x.2;
}
```

Esse programa cria uma tupla, `x`, e então cria uma variável para cada
elemento usando seus índices. Como ocorre nas maiorias das linguagens, o primeiro
índice em uma tupla é o 0.

#### O tipo matriz

Uma outra maneira de ter uma coleção de vários valores é uma *matriz*. Diferentemente
de uma tupla, todos os elementos de uma matriz devem ser do mesmo tipo.
Matrizes em Rust são diferentes de matrizes de outras linguagens, porque matrizes em Rust são de
tamanhos fixos: uma vez declarado, eles não podem aumentar ou diminuir de tamanho.

Em Rust, os valores que entram numa matriz são escritos em uma lista separados
por vírgulas dentro de colchetes:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

Matrizes são úteis quando você deseja que seus dados sejam alocados em pilha do que
no heap (discutiremos mais sobre pilha e heap no Capítulo 4), ou quando
você quer garantir que sempre terá um número fixo de elementos. Uma matriz não
é tão flexível como um vetor. Um vetor é de tipo semelhante,
fornecido pela biblioteca padrão que *é* permitido diminuir ou aumentar o tamanho.
Se você não tem certeza se deve usar uma matriz ou vetor, você provavlemente usará um
vetor. O Capítulo 8 discute sobre vetores com mais detalhes.

Um exemplo de quando você poderia necessitar usar uma matriz no lugar de um vetor é 
um programa em que você precisa saber o nome dos meses do ano. É improvável
que tal programa deseje adicionar ou remover meses, então você pode usar uma matriz
porque você sabe que sempre conterá 12 itens:

```rust
let meses = ["Janeiro", "Fevereiro", "Março", "Abril", "Maio", "Junho", "Julho",
              "Agosto", "Setembro", "Outubro", "Novembro", "Dezembro"];
```

##### Acessando um elemento da matriz

Uma matriz é um pedaço da memória alocada na pilha. Você pode acessar
os elementos da matriz usando indices, como a seguir:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let primeiro = a[0];
    let segundo = a[1];
}
```

Neste exemplo, a variável chamada `primeiro` irá pegar o valor `1`, porque
é o valor indexado por `[0]` na matriz. A variável chamada `segundo` irá
pegar o valor `2`, do indice `[1]` da matriz.

##### Acesso inválido a elemento da matriz

O que acontece se você tentar acessar um elemento da matriz que está além do fim
da matriz? Digamos que você mude o exemplo para o código a seguir, que será compilado,
mas existe um erro quando for executar:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let a = [1, 2, 3, 4, 5];
    let indice = 10;

    let elemento = a[indice];

    println!("O valor do elemento é: {}", elemento);
}
```

Executando esse código usando `cargo run`, é produzido o seguinte resultado:

```text
$ cargo run
   Compiling arrays v0.1.0 (file:///projects/arrays)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/arrays`
thread '<main>' panicked at 'index out of bounds: the len is 5 but the index is
 10', src/main.rs:6
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

A compilação não produz nenhum erro, mas o programa resulta um
erro *em tempo de execução* e não uma saída com sucesso. Quando você
tenta acessar um elemento usando indexação, o Rust verifica se o índice especificado é menor que o tamaho
da matriz. Se o índice é maior que o tamanho, o Rust vai entrar
em *pânico*, que é o termo usado pelo Rust quando um programa resulta em erro.

Esse é o primeiro exemplo dos pricípios de segurança do Rust em ação. Em várias
linguagens de baixo nível, esse tipo de verificação não é feita e quando você fornece um
índice incorreto, memória inválida pode ser acessada. Rust protege você deste tipo
de erro ao sair imediatamente, em vez de permitir o acesso à memória e
continuando. O Capítulo 9 discute mais sobre o tratamento de erros do Rust.
