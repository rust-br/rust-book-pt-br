## Validating References with Lifetimes

Quandos falamos sobre referências no Capítulo 4, nós deixamos de fora um detalhe
importante: toda referência em Rust tem um _lifetime_, que é o escopo no qual
aquela referência é válida. A maior parte das vezes tempos de vida são implícitos e
inferidos, assim como a maior parte do tempo tipos são inferidos. Similarmente
quando temos que anotar tipos porque múltiplos tipos são possíveis, há casos em
que os tempos de vida das referências poderiam estar relacionados de alguns modos
diferentes, então Rust precisa que nós anotemos as relações usando parâmetros
genéricos de tempo de vida para que ele tenha certeza que as referênciais reais
usadas em tempo de execução serão definitivamente válidas.

Sim, é um pouco incomum, e será diferente de ferramentas que você usou em 
outras linguagens de programação. Tempos de vida são, de alguns jeitos, a 
característica mais distinta de Rust. 

Tempos de vida são um tópico grande que não poderão ser cobertos inteiramente 
nesse capítulo, então nós vamos cobrir algumas formas comuns que você pode 
encontrar a sintaxe de tempo de vida nesse capítulo para que você se 
familiarize com os conceitos. O Capítulo 19 conterá informações mais avançadas
sobre tudo que tempos de vida podem fazer.

### Tempos de Vida Previnem Referências Soltas

O principal alvo de lifetimes é prevenir referências soltas, quais fazem com
que o programa referencie dados quais nós não estamos querendo referenciar.
Considere o programa na Listagem 10-18, com um escopo exterior e um interior.
O escopo exterior declara uma variável chamada `r` com nenhum valor inicial, e
o escopo interior declara uma variável chamada `x` com o valor inicial de 5.
Dentro do escopo interior, nós tentamos estabelecer o valor de `r` como uma 
referência para `x`. Então, o escopo interior acaba, e nós tentamos imprimir o
valor de `r`:

```rust,ignore
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

<span class="caption">Listagem 10-18: Uma tentativa de usar uma refência cujo
valor saiu de escopo</span>

> #### Variáveis Não Inicializadas Não Podem Ser Usadas
>
> Os próximos exemplos declaram vaŕiáveis sem darem a elas um valor inicial, 
> então o nome da variável existe no escopo exterior. Isso pode parecer um 
> conflito com Rust não ter null. No entanto, se tentarmos usar uma variável
> antes de atribuir um valor a ela, nós teremos um erro em tempo de compilação.
> Tente!

Quando compilarmos esse código, nós teremos um erro:

```text
error: `x` does not live long enough
   |
6  |         r = &x;
   |              - borrow occurs here
7  |     }
   |     ^ `x` dropped here while still borrowed
...
10 | }
   | - borrowed value needs to live until here
```

A variável `x` não "vive o suficiente". Por que não? Bem, `x` vai sair de 
escopo quando passarmos pela chaves na linha 7, terminando o escopo interior.
Mas `r` é válida para o escopo exterior; seu escopo é maior e dizemos que ela
"vive mais tempo". Se Rust permitisse que esse código funcionasse, `r` estaria
fazendo uma referência à memória que foi desalocada quando `x` saiu de escopo,
e qualquer coisa que tentássemos fazer com `r` não funcionaria corretamente.
Então como o Rust determina que esse código não deve ser permitido?

#### O Verificador de Empréstimos

A parte do compilador chamada de *verificador de empréstimos* compara escopos
para determinar que todos os empréstimos são válidos. A Listagem 10-19 mostra o
mesmo exemplo da Listagem 10-18 com anotações mostrando os tempos de vida das
variáveis.


```rust,ignore
{
    let r;         // -------+-- 'a
                   //        |
    {              //        |
        let x = 5; // -+-----+-- 'b
        r = &x;    //  |     |
    }              // -+     |
                   //        |
    println!("r: {}", r); // |
                   //        |
                   // -------+
}
```

<span class="caption">Listagem 10-19: Anotações de tempos de vida de `r` e `x`,
chamadas de `a` e `b` respectivamente</span>

Nós anotamos o tempo de vida de `r` com `a` e o tempo de vida de `x` com `b`.
Como você pode ver, o bloco interior de `'b` é bem menor que o bloco de tempo
de vida do exterior `'a'`. Em tempo de compilação, o Rust compara o tamanho dos
dois tempos de vida e vê que `r` tem um tempo de vida de `'a`, mas que ele se
refere a um objeto com um tempo de vida `'b`. O programa é rejeitado porque o 
tempo de vida de `'b` é mais curto que o tempo de vida de `'a`: o sujeito da
referência não vive tanto quanto a referência.

Vamos olhar para o exemplo na Listagem 10-20 que não tenta fazer uma referência
solta e compila sem nenhum erro:

```rust
{
    let x = 5;            // -----+-- 'b
                          //      |
    let r = &x;           // --+--+-- 'a
                          //   |  |
    println!("r: {}", r); //   |  |
                          // --+  |
}                         // -----+
```

<span class="caption">Listagem 10-20: Uma referência válida porque os dados têm
um tempo de vida maior do que o da referência</span>

Aqui, `x` tem o tempo de vida de `'b`, que nesse caso tem um tempo de vida 
maior que o de `'a`. Isso quer dizer que `r` pode referenciar `x`: o Rust sabe
que a referência em `r` será sempre válida enquanto `x` for válido.

Agora que mostramos onde os tempos de vida de referências estão em um exemplo
concreto e discutimos como Rust analisa tempos de vida para garantir que
referências sempre serão válidas, vamos falar sobre tempos de vidas genéricos
de parâmetros e retornar valores no contexto das funções.

### Tempos de Vida Génericos em Funções

Vamos escrever uma função que retornará a mais longa de dois cortes de string. 
Nós queremos ser capazes de chamar essa função passando para ela dois cortes 
de strings, e queremos que retorne uma string. O código na Listagem 10-21
deve imprimir `A string mais longa é abcd` uma vez que tivermos implementado a
função `maior`:

<span class="filename">Nome do Arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let resultado = maior(string1.as_str(), string2);
    println!("A string mais longa é {}", resultado);
}
```

<span class="caption">Listagem 10-21: Uma função `main` que chama pela função
`maior` para achar a mais longa entre duas strings</span>

Note que queremos que a função pegue cortes de string (que são referências, 
como falamos no Capítulo 4) já que não queremos que a função `maior` tome posse
de seus argumentos. Nós queremos que uma função seja capaz de aceitar cortes de
uma `String` (que é o tipo de variável `string1`) assim como literais de string
(que é o que a variável `strin2` contém).

Recorra à seção do Capítulo 4 "Cortes de Strings como Parâmetros" para mais 
discussões sobre porque esses são os argumentos que queremos.

Se tentarmos implementar a função `maior` como mostrado na Listagem 10-22 ela
não vai compilar:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn maior(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

<span class="caption">Listagem 10-22: Uma implementação da função `maior` que
retorna o mais longo de dois cortes de string, mas ele não compila ainda</span>

Ao invés disso recebemos o seguinte erro que fala sobre tempos de vida:

```text
error[E0106]: missing lifetime specifier
   |
1  | fn maior(x: &str, y: &str) -> &str {
   |                                 ^ expected lifetime parameter
   |
   = help: this function's return type contains a borrowed value, but the
   signature does not say whether it is borrowed from `x` or `y`
```

O texto de ajuda está nos dizendo que o tipo de retorno precisa de um parâmetro
de tempo de vida genérico nele porque o Rust não pode dizer se a referência que
está sendo retornada se refere a `x` ou `y`. Atualmente, nós também não 
sabemos, já que o bloco `if` no corpo dessa função retorna uma referência para 
`x` e o bloco `else` retorna uma referência para `y`!

Enquanto estamos definindo essa função, não sabemos os valores concretos que 
serão passados para essa função, então não sabemos se o caso `if` ou o caso
`else` será executado. Nós também não sabemos os tempos de vida concretos das
referências que serão passadas, então não podemos olhar para esses escopos como
fizemos nas Listagem 10-19 e 10-20 afim de determinar que a referência que
retornaremos sempre será válida. O verificador de empréstimos não consegue 
determinar isso também porque não sabe como os tempos de vida de `x` e `y` se
relacionam com o tempo de vida do valor de retorno. Nós vamos adicionar 
parâmetros genéricos de tempo de vida que definirão a relação entre as 
referências para que o verificador de empréstimos possa fazer sua análise.

### Sintaxe de Anotação de Tempo de Vida

Anotações de tempo de vida não mudam quanto tempo qualquer uma das referências
envolvidas viverão. Do mesmo modo que funções podem aceitar qualquer tipo de 
assinatura que especifica um parâmetro de tipo genérico, funções podem aceitar
referências com qualquer tempo de vida quando a assinatura especificar um 
parâmetro genérico de tempo de vida. O que anotações de tempo de vida fazem é
relacionar os tempos de vida de múltiplas referências uns com os outros.

Anotações de tempo de vida tem uma sintaxe levemente incomum: os nomes dos
parâmetros de tempos de vida precisam começar com uma apóstrofe `'`. Os nomes 
dos parâmetros dos tempos de vida são usualmente todos em caixa baixa, e como 
tipos genéricos, seu nome usualmente são bem curtos. `'a` é o nome que a maior
parte das pessoas usam por padrão. Parâmetros de anotações de tempos de vida 
vão depois do `&` de uma referência, e um espaço separa a anotação de tempo de 
vida do tipo da referência.

Aqui vão alguns exemplos: nós temos uma referência para um `i32` sem um 
parâmetro tempo de vida, uma referência para um `i32` que tem um parâmetro de
tempo de vida chamado `'a`:

```rust,ignore
&i32        // uma referência
&'a i32     // uma referência com um tempo de vida explícito
&'a mut i32 // uma referência mutável com um tempo de vida explícito
```

Uma anotação de tempo de vida por si só não tem muito significado: anotações de
tempos de vida dizem ao Rust como os parâmetros genéricos de tempos de vida de
múltiplas referências se relacionam uns com os outros. Se tivermos uma função
com o parâmetro `primeiro` que é uma referência para um `i32` que tem um tempo
de vida de `'a`, e a função tem outro parâmetro chamado `segundo` que é outra
referência para um `i32` que também possui um tempo de vida `'a`, essas duas
anotações de tempo de vida com o mesmo nome indicam que as referências 
`primeiro` e `segundo` precisam ambas viver tanto quanto o mesmo tempo de vida
genérico.

### Anotações de Tempo de Vida em Assinaturas de Funções

Vamos olhar para anotações de tempo de vida no contexto da função `maior` que
estamos trabalhando. Assim como parâmetros de tipos genéricos, parâmetros de 
tempos de vida genéricos precisam ser declarados dentro de colchetes angulares
entre o nome da função e a lista de parâmetros. A limitanção que queremos 
dar ao Rust é que para as referências nos parâmetros e o valor de retorno devem
ter o mesmo tempo de vida, o qual nomearemos `'a` e adicionaremos para cada uma
das referências como mostrado na Listagem 10-23:

<span class="filename">Nome do Arquivo: src/main.rs</span>

```rust
fn maior<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

<span class="caption">Listagem 10-23: A definição da função `maior` especifica
todas as referências na assinatura como tendo o mesmo tempo de vida, `'a`</span>

Isso compilará e produzirá o resultado que queremos quando usada com a função
`main` na Listagem 10-21.

A assinatura de função agora diz que para algum tempo de vida `'a`, a função
receberá dois parâmetros, ambos serão cortes de string que vivem pelo menos
tanto quanto o tempo de vida `'a`. A função retornará um corte de string que 
também vai durar tanto quanto o tempo de vida `'a`. Esse é o contrato que 
estamos dizendo ao Rust que queremos garantir.

Especificando os parâmetros de tempo de vida nessa assinatura de função, não
estamos modificando os tempos de vida de quaisquer valores passados ou 
retornados, mas estamos dizendo que quaisquer valores que não concordem com
esse contrato devem ser rejeitados pelo verificador de empréstimos. Essa função
não sabe (ou não precisa saber) exatamente quanto tempo `x` e `y` vão viver,
apenas precisa saber que existe algum escopo que pode ser substituído por `'a`
que irá satisfazer essa assinatura.

Quando estiver anotando tempos de vidas em funções, as anotações vão na 
assinatura da função, e não no código no corpo da função. Isso acontece porque
o Rust consegue analisar o código dentro da função sem nenhuma ajuda, mas 
quando uma função tem referências para ou de códigos de fora daquela função,
os tempos de vida dos argumentos ou os valores de retorno poderão ser 
diferentes cada vez que a função é chamada. Isso seria incrivelmente custoso e
frequentemente impossível para o Rust descobrir. Nesse caso, precisamos anotar
os tempos de vida nós mesmos.

Quando referências concretas são passadas para `maior`, o tempo de vida 
concreto que é substituído por `'a` é a parte do escopo de `x` que sobrepõe o
escopo de `y`. Já que escopos sempre se aninham, outra maneira de dizer isso é
que o tempo de vida genérico `'a` terá um tempo de vida concreto igual ao menor
dos tempos de vida de `x` e `y`. Porque nós anotamos a referência retornada com
o mesmo parâmetro `'a`, a referência retornada será portanto garantida de ser
válida tanto quanto for o tempo de vida mais curto de `x` e `y`.

Vamos ver como isso restringe o uso da função `maior` passando referências que
tem diferentes tempos de vida concretos. A Listagem 10-25 é um exemplo direto
que deve corresponder suas intuições de qualquer linguagem: `string1` é válida
até o final do escopo exterior, `string2` e a`string2` é válida até o final do 
escopo interior. Com o verificador de empréstimos aprovando esse código; ele 
vai compilar e imprimir 
`A string mais longa é`:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
# fn maior<'a>(x: &'a str, y: &'a str) -> &'a str {
#     if x.len() > y.len() {
#         x
#     } else {
#         y
#     }
# }
#
fn main() {
    let string1 = String::from("a string longa é longa");

    {
        let string2 = String::from("xyz");
        let resultado = maior(string1.as_str(), string2.as_str());
        println!("A string mais longa é {}", resultado);
    }
}
```

<span class="caption">Listagem 10-24: Usando a função `maior` com referências
para valores de `String` que tem tempos de vida concretos diferentes</span>

Em seguida, vamos tentar um exemplo que vai mostrar que o tempo de vida da 
referência em `resultado` precisa ser o menor dos tempos de vida dos dois 
argumentos. Nós vamos mover a declaração da variável `resultado` para fora do
escopo interior, mas deixar a atribuição do valor para a variável `resultado`
dentro do escopo com `string2`. Em seguida, vamos mover o `println!` que usa o
`resultado` fora do escopo interior, depois que ele terminou. O código na 
Listagem 10-25 não compilará:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let string1 = String::from("a string longa é longa");
    let resultado;
    {
        let string2 = String::from("xyz");
        resultado = longest(string1.as_str(), string2.as_str());
    }
    println!("A string mais longa é {}", resultado);
}
```

<span class="caption">Listagem 10-25: A tentativa de usar `resultado` depois 
que `string2` saiu de escopo não compilará</span>

Se tentarmos compilar isso, receberemos esse erro:

```text
error: `string2` does not live long enough
   |
6  |         resultadod = longest(string1.as_str(), string2.as_str());
   |                                            ------- borrow occurs here
7  |     }
   |     ^ `string2` dropped here while still borrowed
8  |     println!("The longest string is {}", result);
9  | }
   | - borrowed value needs to live until here
```

O erro está dizendo que para `resultado` ser válido para `println!`, a 
`string2` teria que ser válida até o final do escopo exterior. Rust sabe disso
porque nós anotamos os tempos de vida dos parâmetros da função e retornamos
valores com o mesmo parâmetro do tempo de vida, `'a`.

Nós podemos olhar para esse código como humanos e ver que a `string1` é mais
longa, e portanto `resultado` conterá a referência para a `string1`. Porque a
`string1` não saiu de escopo ainda, a referência para `string1` ainda será 
válida para o `println!`. No entanto, o que dissemos ao Rust com os parâmetros
de tempo de vida é que o tempo de vida da referência retornado pela função 
`maior` é o mesmo que o menor dos tempos de vida das referências passadas. 
Portanto, o verificador de empréstimos não permite o código da Listagem 10-25
como possível já que tem um referência inválida.

Tente fazer mais alguns experimentos que variam os valores e os tempos de vidas
das referências passadas para a função `maior` e como a referência retornada é
usada. Crie hipóteses sobre seus experimentos se eles vão passar pelo 
verificador de empréstimos ou não antes de você compilar, e então cheque para
ver se você está certo!

### Pensando em Termos de Tempos de Vida

O modo exato de especificar parâmetros de tempos de vida depende do que sua 
função está fazendo. Por exemplo, se mudaramos a implementação da função 
`maior` para sempre retornar o primeiro argumento ao invés do corte de string
mais longo, não precisaríamos especificar um tempo de vida no parâmetro `y`.
Este código compila:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

Nesse exemplo, especificamos o tempo de vida do parâmetro `'a` para o parâmetro
`x` e o tipo de retorno, mas, não para o parâmetro `y`, já que o tempo de vida 
de `y` não tem qualquer relação com o tempo de vida `x` ou o valor retornado.

Quando retornarmos uma referência de um valor em uma função, o parâmetro de tempo de 
vida para o tipo de retorno precisa combinar o parâmetro do tempo de vida de um
dos argumentos. Se a referência retornada *não* refere a nenhum dos argumentos,
a única outra possibilidade é que refira a um valor criado dentro da função, o
que seria uma referência solta já que o valor sairá de escopo no fim da função.
Considere essa tentativa da função `maior` que não compilará:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn maior<'a>(x: &str, y: &str) -> &'a str {
    let resultado = String::from("string muito longa");
    resultado.as_str()
}
```

Mesmo especificando um parâmetro de tempo de vida `'a` para o tipo de retorno,
essa implementação falha em compilar porque o valor de retorno do tempo de vida
não é relacionado com o tempo de vida dos parâmetros de forma alguma. Esta é a
mensagem de erro que recebemos:

```text
error: `resultado` does not live long enough
  |
3 |     resultado.as_str()
  |     ^^^^^^ does not live long enough
4 | }
  | - borrowed value only lives until here
  |
note: borrowed value must be valid for the lifetime 'a as defined on the block
at 1:44...
  |
1 | fn maior<'a>(x: &str, y: &str) -> &'a str {
  |                                             ^
```

O problema é que `resultado` sairá de escopo e será limpo no final da função 
`maior`, e estamos tentando retornar uma referência para `resultado` da função.
Não há nenhum modo que possamos especificar parâmetros de tempo de vida que
mudariam uma referência solta, e o Rust não nos deixará criar uma referência 
solta. Nesse caso, a melhor solução seria retornar um tipo de dado com posse
ao invés de uma referência de modo que a função chamadora é então responsável
por limpar o valor.

Em última análise, a sintaxe de tempo de vida é sobre conectar tempos de vida
de vários argumentos e retornar valores de funções. Uma vez que estão 
conectados, o Rust tem informação o suficiente para permitir operações seguras
de memória e não permitir operações que criariam ponteiros soltos ou outro tipo
de violação à segurança da memória.

### Anotações de Tempo de Vida em Definições de Struct

Até agora, nós só definimos structs para conter tipos com posse. É possível 
para structs manter referências, mas precisamos adicionar anotações de tempo de
vida em todas as referências na definição do struct. A Listagem 10-26 tem a 
struct chamada `ExcertoImportante` que contém um corte de string:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
struct ExcertoImportante<'a> {
    parte: &'a str,
}

fn main() {
    let romance = String::from("Chame-me Ishmael. Há alguns anos...");
    let primeira_sentenca = romance.split('.')
        .next()
        .expect("Não pôde achar um '.'");
    let i = ExcertoImportante { parte: primeira_sentenca };
}
```

<span class="caption">Listagem 10-26: Um struct que contém uma referência,
então sua definição precisa de uma anotação de tempo de vida</span>

Esse struct tem um campo, `parte`, que contém um corte de string, que é uma
referência. Assim como tipos genéricos de dados, temos que declarar o nome do
parâmetro genérico de tempo de vida dentro de colchetes angulares depois do
nome do struct para que possamos usar o parâmetro de tempo de vida no corpo da
definição do struct.

A função `main` cria uma instância da struct `ExcertoImportante` que contém uma
referência pra a primeira sentença da `String` com posse da variável `romance`.

### Elisão de Tempo de Vida 

Nessa seção, nós aprendemos que toda referência tem um tempo de vida, e nós
precisamos especificar os parâmetros dos tempos de vida para funções ou 
estruturas que usam referências. No entanto, no Capítulo 4 nós tínhamos a 
função na seção "Cortes de Strings", mostradas novamente na Listagem 10-27, que
compilam sem anotações de tempo de vida:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
fn primeira_palavra(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

<span class="caption">Listagem 10-27: Uma função definida no Capítulo 4 que
compila sem anotações de tempo de vida, mesmo o parâmetro e o tipo de retorno
sendo referências</span>

A razão pela qual essa função compila sem anotações de tempo de vida é 
histórica: em versões mais antigas de pre Rust-1.0, isso não teria compilado.
Toda referência precisava de um tempo de vida explícito. Naquele tempo, a 
assinatura da função teria sido escrita da seguinte forma:

```rust,ignore
fn primeira_palavra<'a>(s: &'a str) -> &'a str {
```

Depois de escrever muito código em Rust, o time de Rust descobriu que os 
programadores de Rust estavam digitando as mesmas anotações de tempo de vida
de novo e de novo. Essas situações eram previsíveis e seguiam alguns padrões
determinísticos. O time de Rust programou esses padrões no compilador de código 
de Rust para que o verificador de empréstimos possa inferir os tempos de vida
dessas situações sem forçar o programador adicionar essas anotações 
explicitamente.

Nós mencionamos essa parte da história de Rust porque é inteiramente possível
que mais padrões determinísticos surgirão e serão adicionado ao compilador. No
futuro, até menos anotações de tempo de vida serão necessárias.

Os padrões programados nas análises de referência de Rust são chamados de 
*regras de elisão de tempo de vida*. Essas não são regras para o programador
seguir; as regras são um conjunto de casos particular que o compilador irá
considerar, e se seu código se encaixa nesses casos, você não precisa escrever
os tempos de vida explicitamente.

As regras de elisão não fornecem total inferência:  se o Rust aplicar as regras 
de forma determinística ainda podem haver ambiguidades como quais tempos 
de vida as referências restantes deveriam ter. Nesse caso, o compilador dará um
erro que pode ser solucionado adicionando anotações de tempo de vida que 
correspondem com as suas intenções para como as referências se relacionam umas
com as outras.

Primeiro, algumas definições: Tempos de vida em parâmetros de funções ou 
métodos são chamados *tempos de vida de entrada*, e tempos de vida em valores
de retorno são chamados de *tempos de vida de saída*.

Agora, as regras que o compilador usa para descobrir quais referências de 
tempos de vidas têm quando não há anotações explícitas. A primeira regra se 
aplica a tempos de vida de entrada, e a segunda regra se aplica a tempos de 
vida de saída. Se o compilador chega no fim das três regras e ainda há 
referências que ele não consegue descobrir tempos de vida, o compilador irá
parar com um erro.

1. Cada parâmetro que é uma referência tem seu próprio parâmetro de tempo de
  vida. Em outras palavras, uma função com um parâmetro tem um parâmetro de 
  tempo de vida: `fn foo<'a>(x: &'a i32)`, uma função com dois argumentos 
  recebe dois parâmetros de tempo de vida separados: 
  `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`, e assim por diante.

2. Se há exatamente uma entrada de parâmetro de tempo de vida, aquele tempo de
  vida é atribuído para todos os parâmetros de saída do tempo de vida:
  `fn foo<'a>(x: &'a i32) -> &'a i32`.

3. Se há múltiplas entradas de parâmetros de tempo de vida, mas uma delas é
  `&self` ou `&mut self` porque é um método, então o tempo de vida de `self` é
  atribuído para todos os parâmetro de tempo de vida de saída. Isso melhora a
  escrita de métodos

Vamos fingir que somos o compilador e aplicamos essas regras para descobrir 
quais os tempos de vida das referências na assinatura da função 
`primeira_palavra` na Listagem 10-27. A assinatura começa sem nenhum tempo de 
vida associado com as referências:

```rust,ignore
fn primeira_palavra(s: &str) -> &str {
```

Então nós (como o compilador) aplicamos a primeira regra, que diz que cada 
parâmetro tem seu próprio tempo de vida. Nós vamos chamá-lo de `'a` como é 
usual, então agora a assinatura é:

```rust,ignore
fn primeira_palavra<'a>(s: &'a str) -> &str {
```

À segunda regra, que se aplica porque existe apenas um tempo de vida. A
segunda regra diz que o tempo de vida de um parâmetro de entrada é atribuído
a um tempo de vida de saída, então agora a assinatura é:

```rust,ignore
fn primeira_palavra<'a>(s: &'a str) -> &'a str {
```

Agora todas as referências nessa assinatura de função possuem tempos de vida, e
o compilador pode continuar sua análise sem precisar que o programador anote os
tempos de vida na assinatura dessa função.

Vamos fazer outro exemplo, dessa vez com a função `maior` que não tinha 
parâmetros de tempo de vida quando começamos a trabalhar com ela na Listagem 
10-22:

```rust,ignore
fn maior(x: &str, y: &str) -> &str {
```

Fingindo que somos o compilador novamente, vamos aplicar a primeira regra: cada
parâmetro tem seu próprio tempo de vida. Dessa vez temos dois parâmetros, então
temos dois tempos de vida:

```rust,ignore
fn maior<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Olhando para a segunda regra, ela não se aplica já que há mais de uma entrada 
de tempo de vida. Olhando para a terceira regra, ela também não se aplica 
porque isso é uma função e não um método, então nenhum dos parâmetros são 
`self`. Então, acabaram as regras, mas não descobrimos qual é o tempo de vida 
do tipo de retorno. É por isso que recebemos um erro quando tentamos
compilar o código da Listagem 10-22: o compilador usou as regras de elisão de
tempo de vida que sabia, mas ainda sim não conseguiu descobrir todos os tempos
de vida das referências na assinatura.

Porque a terceira regra só se aplica em assinaturas de métodos, vamos olhar
tempos de vida nesse contexto agora, e ver porque a terceira regra significa 
que não temos que anotar tempos de vida em assinaturas de métodos muito 
frequentemente.

### Anotações de Tempo de Vida em Definições de Métodos

Quando implementamos métodos em uma struct com tempos de vida, a sintaxe é
novamente a mesma da de parâmetros de tipos genéricos que mostramos na Listagem
10-11: o lugar que parâmetros de tempos de vida são declarados e usados depende
se o parâmetro de tempo de vida é relacionado aos campos do struct ou aos
argumentos dos métodos e dos valores de retorno.

Nomes de tempos de vida para campos de estruturas sempre precisam ser 
declarados após a palavra-chave `impl` e então usadas após o nome da struct,
já que esses tempos de vida são partes do tipo da struct.

Em assinaturas de métodos dentro do bloco `impl`, as referências podem estar
amarradas às referências de tempo de vida nos campos de struct, ou elas podem
ser independentes. Além disso, as regras de elisão de tempo de vida 
constantemente fazem com que anotações não sejam necessárias em assinaturas de
métodos. Vamos ver alguns exemplos usando a struct chamada `ExcertoImportante`
que definimos na Listagem 10-26.

Primeiro, aqui há um método chamado `level`. O único parâmetro é uma referência
para `self`, e o valor de retorno é apenas um `i32`, não uma referência para
nada:

```rust
# struct ExcertoImportante<'a> {
#     part: &'a str,
# }
#
impl<'a> ExcertoImportante<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

A declaração do parâmetro de tempo de vida depois de `impl` e uso depois do 
tipo de nome é obrigatório, mas nós não necessariamente precisamos de anotar o
tempo de vida da referência `self` por causa da primeira regra da elisão.

Aqui vai um exemplo onde a terceira regra da elisão de tempo de vida se aplica:

```rust
# struct ExcertoImportante<'a> {
#     part: &'a str,
# }
#
impl<'a> ExcertoImportante<'a> {
    fn anuncio_e_parte_de_retorno(&self, anuncio: &str) -> &str {
        println!("Atenção por favor: {}", anuncio);
        self.part
    }
}
```

Há dois tempos de vida de entrada, então o Rust aplica a primeira regra de 
elisão de tempos de vida e dá ambos ao `&self` e ao `anuncio` seus próprios
tempos de vida. Então, porque um dos parâmetros é `self`, o tipo de retorno
tem o tempo de vida de `&self` e todos os tempos de vida foram contabilizados.

### O Tempo de Vida Estático

Há *um* tipo especial de tempo de vida que precisamos discutir: `'static`. O
tempo de vida `static` é a duração completa do programa. Todos os literais de
string têm um tempo de vida `static`, o qual podemos escolher anotar como o
seguinte:

```rust
let s: &'static str = "Eu tenho um tempo de vida estático.";
```

O texto dessa string é guardado diretamente no binário do seu programa e o
binário do seu programa está sempre disponível. Logo, o tempo de vida de todas
as literais de string é `'static`.

Você pode ver sugestões de usar o tempo de vida `'static` em uma mensagem de 
ajuda de erro, mas antes de especificar `'static` como o tempo de vida para uma
referência, pense sobre se a referência que você tem é uma que vive todo o 
tempo de vida do seu programa ou não (ou mesmo se você quer que ele viva tanto,
se poderia). Na maior parte do tempo, o problema no código é uma tentativa de 
criar uma referência solta ou uma incompatibilidade dos tempos de vida 
disponíveis, e a solução é consertar esses problemas, não especificar um tempo
de vida `'static`.


### Parâmetros de Tipos Genéricos, Limites de Trais e Tempos de Vida Juntos

Vamos rapidamente olhar para a sintaxe de especificar parâmetros de tipos
genéricos, limites de traits e tempos de vida todos em uma função!

```rust
use std::fmt::Display;

fn maior_com_um_anuncio<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Anúncio! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Essa é a função `maior` da Listagem 10-23 que retorna a maior de dois cortes de
string, mas com um argumento extra chamado `ann`. O tipo de `ann` é o tipo 
genérico `T`, que pode ser preenchido por qualquer tipo que implemente o trait
`Display` como está especificado na cláusula `where`. Esse argumento extra será
impresso antes da função comparar os comprimentos dos cortes de string, que é 
porque o trait de `Display` possui um limite. Porque tempos de vida são um tipo
genérico, a declaração de ambos os parâmetros de tempo de vida `'a` e o tipo
genérico `T` vão na mesma lista com chaves angulares depois do nome da função.

## Sumário

Nós cobrimos várias coisas nesse capítulo! Agora que você sabe sobre parâmetros
de tipos genéricos, traits e limites de traits, e parâmetros genéricos de tempo
de vida, você está pronto para escrever código que não é duplicado mas pode ser
usado em muitas situações. Parâmetros de tipos genéricos significam que o 
código pode ser aplicado a diferentes tipos. Traits e limites de traits 
garantem que mesmo que os tipos sejam genéricos, esses tipos terão o 
comportamento que o código precisa. Relações entre tempos de vida de 
referências especificadas por anotações de tempo de vida garantem que esse 
código flexível não terá referências soltas. E tudo isso acontece em tempo de
compilação para que a performace em tempo de execução não seja afetada!

Acredite ou não, há ainda mais para aprender nessas áreas: Capítulo 17 
discutirá objetos de trait, que são outro modo de usar traits. O Capítulo 19
vai cobrir cenários mais complexos envolvendo anotações de tempo de vida. O
Capítulo 20 vai tratar de alguns tipos avançados de características do sistema.
Em seguida, porém, vamos falar sobre como escrever testes em Rust para que
possamos ter certeza que nosso código usando todas essas características está
funcionando do jeito que queremos!