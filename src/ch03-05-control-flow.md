## Controle de fluxo

Decidir se deve ou não executar algum código, dependendo se uma condição é verdadeira
e decidir executar algum código repetidamente enquanto uma condição é verdadeira,
são blocos de construção básicos na maioria das linguagens de programação. As construções
mais comuns que permitem controlar o fluxo de execução do código Rust são as expressões `if` e
laços de repetição.

### Expressão `if`

Uma expressão `if` permite ramificar seu código dependendo das condições. Você
fornece uma condição e, em seguida, estado, "Se esta condição for cumprida, execute este bloco
de código. Se a condição não for atendida, não execute este bloco de código. ”

Crie um novo projeto chamado *branches* no seu diretório *projects* para explorar
a expressão `if`. No arquivo *src/main.rs*, digite o seguinte:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let numero = 3;

    if numero < 5 {
        println!("condição era verdadeira");
    } else {
        println!("condição era falsa");
    }
}
```

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

Todas as expressões `if` começam com a palavra-chave `if`, que é seguida por uma
condição. Neste caso, a condição verifica se a variável
`number` tem um valor menor que 5. O bloco de código que queremos executar se o
condição é verdadeira é colocada imediatamente após a condição dentro de chaves.
Blocos de código associados às condições em expressões `if` são
às vezes chamado de *divisões*, assim como as expressões `de combinação` que nós
discutimos na seção “Comparando o Palpite ao Número Secreto” de
Capítulo 2.

Opcionalmente, também podemos incluir uma expressão `else`, que escolhemos
fazer aqui, para dar ao programa um bloco de código alternativo a ser executado, caso a
condição seja avaliada como falsa. Se você não fornecer uma expressão `else` e a
condição for falsa, o programa simplesmente ignorará o bloco `if` e passará para o
próximo bit de código.

Tente executar este código; você deve ver a seguinte saída:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
condição era verdadeira
```

Vamos tentar alterar o valor de `numero` para um valor que torne a condição
`false` para ver o que acontece:

```rust,ignore
let numero = 7;
```

Execute o programa novamente e observe a saída:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
condição era falsa
```

Também é importante notar que a condição neste código *deve* ser um `bool`. E se
a condição não é um `bool`, nós vamos receber um erro. Por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let numero = 3;

    if numero {
        println!("número era 3");
    }
}
```

A condição `if` é avaliada para um valor de `3` desta vez, e Rust lança um
erro:

```text
error[E0308]: mismatched types
 --> src/main.rs:4:8
  |
4 |     if numero {
  |        ^^^^^^ expected bool, found integral variable
  |
  = note: expected type `bool`
             found type `{integer}`
```

O erro indica que Rust esperava um `bool`, mas obteve um inteiro. Ao contrário de
linguagens como Ruby e JavaScript, o Rust não tentará automaticamente
converter tipos não-booleanos em um booleano. Você deve explicitar e sempre fornecer
`if` com um booleano como sua condição. Se quisermos que o bloco de código `if` seja executado
somente quando um número não é igual a `0`, por exemplo, podemos mudar o `if`
para o seguinte:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let numero = 3;

    if numero != 0 {
        println!("número era algo diferente de zero");
    }
}
```

A execução deste código irá imprimir `número era algo diferente de zero`.

#### Gerenciando Múltiplas Condições com `else if`

Você pode ter várias condições combinando `if` e` else` em um `else if`.
Por exemplo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let numero = 6;

    if numero % 4 == 0 {
        println!("número é divisível por 4");
    } else if numero % 3 == 0 {
        println!("número é divisível por 3");
    } else if numero % 2 == 0 {
        println!("número é divisível por 2");
    } else {
        println!("número não é divisível por 4, 3 ou 2");
    }
}
```

Este programa tem quatro caminhos possíveis. Depois de executá-lo, você deve
ver a seguinte saída:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
número é divisível por 3
```

Quando este programa é executado, ele verifica cada expressão `if` por sua vez e executa
o primeiro corpo para o qual a condição é verdadeira. Note que mesmo que 6 seja
divisível por 2, nós não vemos a saída `o número é divisível por 2`, nem vemos o
texto `número não é divisível por 4, 3 ou 2` do bloco `else`.
Isso ocorre porque o Rust só executa o bloco para a primeira condição verdadeira e,
depois de encontrar um, não verifica o restante.

Usar muitas expressões `else if` pode confundir seu código, portanto, se você tiver
mais de uma, convém refatorar seu código. O Capítulo 6 descreve uma poderosa
construção de ramificação em Rust chamada `match` para esses casos.

#### Usando `if` em uma declaração `let`

Pelo fato de `if` ser uma expressão, podemos usá-la do lado direito de uma declaração `let`,
como na Listagem 3-2:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let condicao = true;
    let numero = if condicao {
        5
    } else {
        6
    };

    println!("O valor do número é: {}", numero);
}
```

<span class = "caption"> Listagem 3-2: Atribuindo o resultado de uma expressão `if`
para uma variável </span>

A variável `numero` será ligada a um valor baseado no resultado da expressão `if`.
Execute este código para ver o que acontece:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/branches`
O valor do número é: 5
```

Lembre-se de que os blocos de código são avaliados até a última expressão, e os
números por si mesmos também são expressões. Neste caso, o valor de
toda a expressão `if` depende de qual bloco de código é executado. Isso significa que
os valores que têm o potencial de serem resultados de cada braço do `if` e que devem ser
do mesmo tipo; na Listagem 3-2, os resultados do braço `if` e do `else`
eram inteiros `i32`. Se os tipos forem incompatíveis, como no exemplo a
seguir, receberemos um erro:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let condicao = true;

    let numero = if condicao {
        5
    } else {
        "seis"
    };

    println!("O valor do número é: {}", numero);
}
```

Quando tentamos executar esse código, recebemos um erro. Os braços `if` e` else` possuem
valores de tipos que são incompatíveis, e Rust indica exatamente onde encontrar o
problema no programa:

```text
error[E0308]: if and else have incompatible types
 --> src/main.rs:4:18
  |
4 |       let numero = if condicao {
  |  __________________^
5 | |         5
6 | |     } else {
7 | |         "seis"
8 | |     };
  | |_____^ expected integral variable, found &str
  |
  = note: expected type `{integer}`
             found type `&str`
```

A expressão no bloco `if` é avaliada como um inteiro, e a expressão no bloco` else` é
avaliada como uma string. Isso não funcionará porque as variáveis precisam ter
um único tipo. Rust precisa saber em tempo de compilação qual é o tipo da variável `numero`,
definitivamente, para que possa verificar em tempo de compilação que seu tipo é
válido em todos os lugares em que usamos `numero`. Rust não seria capaz de fazer isso se o tipo
de `numero` fosse determinado apenas em tempo de execução; o compilador seria mais complexo e
faria menos garantias sobre o código se tivesse que manter o controle de
vários tipos hipotéticos para qualquer variável.

### Repetição com laços de repetição

Geralmente, é útil executar um bloco de código mais de uma vez. Para essa tarefa,
o Rust fornece vários *loops*. Um loop percorre o código dentro do corpo do loop
até o final e, em seguida, inicia imediatamente no início. Para
experimentar loops, vamos criar um novo projeto chamado *loops*.

O Rust possui três tipos de loops: `loop`,` while` e `for`. Vamos tentar cada um.

#### Código de Repetição com `loop`

A palavra-chave `loop` diz ao Rust para executar um bloco de código várias vezes
para sempre ou até que você diga explicitamente para parar.

Como exemplo, altere o arquivo *src/main.rs* do diretório *loops* para algo
como isso:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    loop {
        println!("novamente!");
    }
}
```

Quando executamos este programa, veremos ``novamente!` impresso repetidamente
até que paremos o programa manualmente. A maioria dos terminais suporta um atalho de teclado,
<span class = "keystroke"> ctrl-c </span>, para parar um programa que está preso em um
loop contínuo. De uma chance:

```text
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29 secs
     Running `target/debug/loops`
novamente!
novamente!
novamente!
novamente!
^Cnovamente!
```

O símbolo `^C` representa onde você pressionou <span class ="keystroke"> ctrl-c
</span>. Você pode ou não ver a palavra `novamente!` Impressa depois do `^C`,
dependendo de onde o código estava no loop quando recebeu o sinal de parada.

Felizmente, o Rust oferece outra maneira mais confiável de sair de um loop.
Você pode colocar a palavra-chave `break` dentro do loop para dizer ao programa quando
parar de executar o loop. Lembre-se que fizemos isso no jogo de adivinhação no
“Quitting After a Guess Correct” do Capítulo 2 para sair do
programa quando o usuário ganhou o jogo, adivinhando o número correto.

#### Loops condicionais com `while`

Geralmente, é útil para um programa avaliar uma condição dentro de um loop. Enquanto
a condição é verdadeira, o loop é executado. Quando a condição deixa de ser verdadeira,
o programa chama o `break`, parando o loop. Esse tipo de loop pode ser implementado
usando uma combinação de `loop`,` if`, `else` e` break`; você poderia tentar isso
agora em um programa, se você quiser.

No entanto, esse padrão é tão comum que o Rust possui uma construção de linguagem integrada
para isso, chamado de loop `while`. A Listagem 3-3 usa `while`: o programa faz o loop
três vezes, a contagem decrescente de cada vez e, depois do ciclo, imprime
outra mensagem e sai.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let mut numero = 3;

    while numero != 0 {
        println!("{}!", numero);

        numero = numero - 1;
    }

    println!("LIFTOFF!!!");
}
```

<span class = "caption"> Listagem 3-3: Usando um loop `while` para executar o código enquanto
condição for verdadeira </span>

Essa construção elimina muito o aninhamento que seria necessário se você usasse
`loop`,` if`, `else` e` break`, e é mais claro. Enquanto a condição for
verdadeira, o código é executado; caso contrário, sai do loop.

#### Looping através de uma coleção com `for`

Você poderia usar a construção `while` para fazer um loop sobre os elementos de uma coleção,
como uma matriz. Por exemplo, vamos ver a Listagem 3-4:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut indice = 0;

    while indice < 5 {
        println!("O valor é: {}", a[indice]);

        indice = indice + 1;
    }
}
```

<span class = "caption"> Listagem 3-4: percorrendo cada elemento de uma coleção
usando um loop `while` </span>

Aqui, o código conta através dos elementos na matriz. Começa no índice
`0` e, em seguida, faz um loop até atingir o índice final na matriz (isto é,
quando `indice <5` não é mais verdadeiro). Executando este código irá imprimir todos os elementos
na matriz:

```text
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
     Running `target/debug/loops`
O valor é: 10
O valor é: 20
O valor é: 30
O valor é: 40
O valor é: 50
```

Todos os cinco valores de matriz aparecem no terminal, conforme esperado. Embora `indice`
vai chegar a um valor de `5` em algum momento, o loop para de executar antes de tentar
para buscar um sexto valor da matriz.

Mas essa abordagem é propensa a erros; poderíamos fazer o programa entrar em pânico se o
o comprimento do índice estivesse incorreto. Também é lento, porque o compilador adiciona código de tempo de execução
para executar a verificação condicional em cada elemento em cada iteração
através do loop.

Como uma alternativa mais concisa, você pode usar um laço `for` e executar algum código
para cada item de uma coleção. Um laço `for` parece com este código na Listagem 3-5:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for elemento in a.iter() {
        println!("O valor é: {}", elemento);
    }
}
```

<span class = "caption"> Listagem 3-5: percorrendo cada elemento de uma coleção
usando um laço `for` </span>

Quando executamos esse código, veremos a mesma saída da listagem 3-4. Mais
importante, agora aumentamos a segurança do código e eliminamos a
chance de erros que podem resultar de ir além do final da matriz ou não
indo longe o suficiente e faltando alguns itens.

Por exemplo, no código da Listagem 3-4, se você removeu um item do array `a`,
mas esqueceu de atualizar a condição para `while indice <4`, o código causaria um
pânico. Usando o loop `for`, você não precisa se lembrar de alterar qualquer outro
código se você alterou o número de valores na matriz.

A segurança e a concisão dos loops `for` fazem deles o loop mais comumente usado
em Rust. Mesmo em situações em que você deseja executar algum código
certo número de vezes, como no exemplo da contagem regressiva que usou um loop `while`
da Listagem 3-3, a maioria dos Rustaceans usaria um loop `for`. A maneira de fazer isso
seria usar um `Range`, que é um tipo fornecido pela biblioteca padrão
que gera todos os números em sequência a partir de um número e terminando
antes de outro número.

Veja como seria a contagem regressiva usando um loop `for` e outro método,
que nós ainda não falamos, `rev`, para reverter o intervalo:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    for numero in (1..4).rev() {
        println!("{}!", numero);
    }
    println!("LIFTOFF!!!");
}
```

Este código é um pouco melhor, não é?

## Resumo

Você conseguiu! Esse foi um capítulo considerável: você aprendeu sobre variáveis, tipos
de dados escalares e compostos, funções, comentários, expressões `if` e loops! E se
você quer praticar com os conceitos discutidos neste capítulo, tente construir
programas para fazer o seguinte:

* Converta temperaturas entre Fahrenheit e Celsius.
* Gerar o n-ésimo número de Fibonacci.
* Imprima a letra da canção de natal "Os Doze Dias de Natal"
aproveitando a repetição na música.

Quando você estiver pronto para seguir em frente, falaremos sobre um conceito em Rust que *não*
comumente existente em outras linguagens de programação: propriedade.
