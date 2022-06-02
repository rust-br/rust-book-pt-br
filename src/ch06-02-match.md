## Operador `match` de Controle de Fluxo

O Rust tem um excelente operador de controle de fluxo chamado `match`, que nos
permite comparar um valor com uma série de padrões e executar um código com
base no padrão que casar. Padrões podem ser compostos de valores literais,
variáveis, caracteres-curinga e várias outras coisas. O Capítulo 18 aborda
todos os tipos de padrões e o que eles fazem. A grande utilidade do `match` vem
da expressividade dos padrões e das análises feitas pelo compilador, tendo
certeza de que todos os casos possíveis estão sendo tratados.

Imagine que expressão `match` funciona como uma máquina de contar moedas: as
moedas passam por um canal que possui furos de vários tamanhos, e cada moeda
cai no primeiro furo em que ela couber. Da mesma forma, os valores passam por
cada padrão de um `match`, e logo no primeiro padrão que o valor "se encaixar",
o bloco de código que estiver associado a ele será executado.

Aproveitando que acabamos de falar sobre moedas, vamos usá-las como exemplo de
utilização do `match`! Podemos escrever uma função que recebe uma moeda
qualquer dos Estados Unidos e, assim como uma máquina, determina qual moeda ela
é e retorna seu valor em _cents_, como mostra a Listagem 6-3:

> **Nota do tradutor:** diferentemente do que acontece na maioria dos países,
> as moedas dos Estados Unidos possuem nomes: as de 1 _cent_ são chamadas de
> _Penny_; as de 5 _cents_, de _Nickel_; as de 10 _cents_, de _Dime_; e as de 25
> _cents_, de _Quarter_.

```rust
enum Moeda {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn valor_em_cents(moeda: Moeda) -> u32 {
    match moeda {
        Moeda::Penny => 1,
        Moeda::Nickel => 5,
        Moeda::Dime => 10,
        Moeda::Quarter => 25,
    }
}
```

<span class="caption">Listagem 6-3: Uma enum e uma expressão `match` em que os
padrões comparados são as variantes da enum.</span>

Vamos analisar o `match` da função `valor_em_cents`. Primeiro, usamos a
palavra-chave `match` seguida de uma expressão, que neste caso é o valor
`moeda`. É parecido a uma expressão utilizada com `if`, mas tem uma grande
diferença: com `if`, a expressão precisa retornar um valor _booleano_. Aqui,
pode ser de qualquer tipo. O tipo da variável `moeda`, neste exemplo, é a enum
`Moeda`, que definimos na Listagem 6-3.

Em seguida vêm os braços do `match`. Um braço é composto por duas partes: um
padrão e algum código. O primeiro braço deste exemplo tem um padrão, que é o
valor `Moeda::Penny`, e o operador `=>` que separa o padrão do código associado
a ele. O código, nesse caso, é apenas o valor `1`. Os braços são separados uns
dos outros com uma vírgula.

Quando o `match` é executado, ele compara o valor resultante com o padrão de
cada braço, na ordem. Se o valor casar com um determinado padrão, o código
associado a esse padrão será executado. Se o valor não se encaixa nesse padrão,
a execução passa para o próximo braço, bem parecido com a máquina de contar
moedas. Podemos ter tantos braços quanto precisamos. No caso da Listagem 6-3,
nosso `match` tem quatro braços.

O código associado a cada braço é uma expressão, e o seu valor resultante, no
braço que combinar, é o que será retornado pela expressão `match`.

Tipicamente não se usa chaves se o braço do `match` for curto, como é o caso na
Listagem 6-3, em que cada braço retorna apenas um valor. Se você quiser
executar mais de uma linha de código em um braço, você pode usar chaves para
delimitá-las. Por exemplo, o código seguinte vai escrever na tela "Moeda da
sorte!" sempre que o método for chamado com uma `Moeda::Penny`, mas ainda vai
retornar o último valor do bloco, `1`:

```rust
# enum Moeda {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter,
# }
#
fn valor_em_cents(moeda: Moeda) -> u32 {
    match moeda {
        Moeda::Penny => {
            println!("Moeda da sorte!");
            1
        },
        Moeda::Nickel => 5,
        Moeda::Dime => 10,
        Moeda::Quarter => 25,
    }
}
```

### Padrões Atrelados a Valores

Outra característica útil dos braços do `match` é que eles podem ser atrelados
a partes dos valores que se encaixam no padrão. É assim que podemos extrair
valores dentro de uma variante de uma enum.

Por exemplo, vamos alterar uma das nossas variantes, inserindo dados dentro
dela. De 1999 até 2008, os Estados Unidos cunhou _quarters_ com um _design_
diferente para cada um dos 50 estados em um dos lados da moeda. Nenhuma outra
moeda tinha essa diferença no _design_, apenas os _quarters_. Podemos adicionar
essa informação à nossa `enum` alterando a variante `Quarter` para incluir o
valor `Estado`, como é feito na Listagem 6-4:

```rust
#[derive(Debug)] // Para podermos ver qual é o estado com mais facilidade
enum Estado {
    Alabama,
    Alaska,
    // ... etc
}

enum Moeda {
    Penny,
    Nickel,
    Dime,
    Quarter(Estado),
}
```

<span class="caption">Listagem 6-4: Enum `Moeda`, cuja variante `Quarter`
também guarda o valor `Estado`.</span>

Vamos imaginar que um amigo nosso está tentando colecionar todas os _quarters_
dos 50 estados. Enquanto separamos nosso troco por tipo de moeda, vamos também
dizer o nome do estado associado a cada _quarter_. Se for um dos que o nosso
amigo ainda não tem, ele pode colocá-lo na sua coleção.

Na expressão `match` desse código, vamos adicionar uma variável chamada
`estado` ao padrão que casa com os valores da variante `Moeda::Quarter`. Quando
uma `Moeda::Quarter` é testada, a variável `estado` vai ser atrelada ao valor
do estado daquele _quarter_. Assim vamos poder usar o `estado` no código do
braço, desse jeito:

```rust
# #[derive(Debug)]
# enum Estado {
#    Alabama,
#    Alaska,
# }
#
# enum Moeda {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(Estado),
# }
#
fn valor_em_cents(moeda: Moeda) -> u32 {
    match moeda {
        Moeda::Penny => 1,
        Moeda::Nickel => 5,
        Moeda::Dime => 10,
        Moeda::Quarter(estado) => {
            println!("Quarter do estado {:?}!", estado);
            25
        },
    }
}
```

Se executarmos `valor_em_cents(Moeda::Quarter(Estado::Alaska))`, `moeda` seria
`Moeda::Quarter(Estado::Alaska)`. Quando comparamos esse valor em cada um dos
braços do `match`, nenhum deles vai casar enquanto não chegar em
`Moeda::Quarter(estado)`. Nesse ponto, `estado` vai estar atrelado ao valor
`Estado::Alaska`. Podemos, então, usar esse valor na expressão `println!`,
obtendo o estado contido na variante `Quarter` da enum `Moeda`.

### Usando `match` com `Option<T>`

Na seção anterior, queríamos obter o valor `T` contido em um `Some` quando era
o caso em uma `Option<T>`. Também podemos manipular uma `Option<T> usando
`match`, assim como fizemos com a enum `Moeda`! Em vez de comparar moedas,
vamos comparar as variantes de `Option<T>`, mas a forma de trabalhar com a
expressão `match` continua a mesma.

Digamos que queremos escrever uma função que recebe um `Option<i32>`, e se
houver um valor embutido nele, soma um a esse valor. Se não houver um valor,
a função deve retornar `None`, e nem tentar executar nenhuma operação.

Essa função é bem fácil de implementar, graças ao `match`, e vai ficar conforme
visto na Listagem 6-5:

```rust
fn mais_um(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let cinco = Some(5);
let seis = mais_um(cinco);
let nenhum = mais_um(None);
```

<span class="caption">Listagem 6-5: Uma função que usa um `match` para tratar
uma `Option<i32>`.</span>

#### Casando `Some(T)`

Vamos examinar a primeira execução de `mais_um` em mais detalhes. Quando
chamamos `mais_um(cinco)`, a variável `x` no corpo da função `mais_um` vai ter
o valor `Some(5)`. Então comparamos ele a cada braço do `match`.

```rust,ignore
None => None,
```

O valor `Some(5)` não casa com o padrão `None`, então seguimos para o
próximo braço.

```rust,ignore
Some(i) => Some(i + 1),
```

`Some(5) casa com `Some(i)`? Sim, casa! Temos a mesma variante. O `i`
está atrelado ao valor contido em `Some`, então `i` passa a ter o valor `5`. O
código desse braço é executado, então somamos um ao valor de `i` e criamos um
novo `Some` contendo nosso total de `6`.

#### Casando `None`

Agora vamos considerar a segunda chamada da função `mais_um` na Listagem 6-5,
em que `x` é `None`. Nós entramos no `match` e comparamos com o primeiro braço.

```rust,ignore
None => None,
```

Confere! Não há nenhum valor para somar, então o programa para e retorna o
valor `None` do lado direito do `=>`. Como o primeiro braço já casou, nenhum
dos demais será testado.

A combinação de enums e a expressão `match` é útil em diversas situações. Você
verá muito esse padrão em Rust: fazer o `match` de uma enum, associar uma
variável ao valor embutido, e então executar um código baseado nele. Pode
parecer complicado no começo, mas uma vez que você se acostuma, você vai querer
que tivesse isso em todas as linguagens. É, sistematicamente, um favorito dos
usuários.

### _Matches_ São Exaustivos

Há outro aspecto do `match` que precisamos discutir. Considere essa versão da
nossa função `mais_um`:

```rust,ignore
fn mais_um(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```

Nós não tratamos o caso `None`, logo vai ocorrer um _bug_ no nosso código. Por
sorte, é um _bug_ que o Rust sabe detectar. Se tentarmos compilar esse código,
vamos ter esse erro:

```text
error[E0004]: non-exhaustive patterns: `None` not covered
 -->
  |
6 |         match x {
  |               ^ pattern `None` not covered
```

O Rust sabe que nós não cobrimos todos os casos possíveis, e sabe até de qual
padrão nos esquecemos! _Matches_ em Rust são *exaustivos*: precisamos extinguir
até a última possibilidade pra que o nosso código seja válido. Especialmente no
caso de uma `Option<T>`, em que o Rust não nos deixa esquecer de tratar
explicitamente o caso `None`. Ele nos impede de assumir que temos um valor
válido quando possivelmente temos um valor nulo, e portanto, cometer o erro de
um bilhão de dólares que vimos mais cedo.

### The `_` Placeholder
### O _Placeholder_ `_`

O Rust também tem um padrão que podemos usar em situações em que não queremos
listar todos os valores possíveis. Por exemplo, um `u8` pode ter valores
válidos de 0 a 255. Se nos importamos apenas com os valores 1, 3, 5 e 7, não
queremos ser obrigados a listar o 0, 2, 4, 6, 8, 9, e assim por diante até 255.
Felizmente, nem precisamos: em vez disso, podemos usar o padrão especial `_`.

```rust
let algum_valor_u8 = 0u8;
match algum_valor_u8 {
    1 => println!("um"),
    3 => println!("três"),
    5 => println!("cinco"),
    7 => println!("sete"),
    _ => (),
}
```

O padrão `_` casa com qualquer valor. Colocando ele depois dos demais
braços, o `_` vai casar com todos os casos possíveis que não foram
especificados antes dele. O `()` é só o valor-unidade, pra que nada aconteça no
caso `_`. Como resultado, podemos dizer que não queremos fazer nada com os
possíveis valores que não listamos antes do _placeholder_ `_`.

Contudo, a expressão `match` pode ser um tanto verbosa em uma situação em que
queremos apenas lidar com *um* dos casos. Pra essa situação, o Rust oferece o
`if let`.
