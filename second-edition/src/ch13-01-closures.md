## Closures: Funções Anônimas que Podem Capturar seu Ambiente

Os closures (fechamentos) de Rust são funções anônimas que você pode salvar em uma variável 
ou transmitir como argumentos para outras funções. Você pode criar closure em 
um único local e depois chamá-lo para avaliá-lo em um contexto diferente. Diferentemente 
das funções, os closures podem capturar valores do escopo em que são chamados. 
Vamos demonstrar como esses recursos de closure permitem a reutilização de 
código e a personalização do comportamento.

### Criando uma Abstração de Comportamento com Closures

Vamos trabalhar em um exemplo de situação em que é útil armazenar um closure para 
ser executado posteriormente. Ao longo do caminho, falaremos sobre a sintaxe de closures, 
inferência de tipos e _traits_.

Considere esta situação hipotética: trabalhamos em uma startup que está criando um 
aplicativo para gerar planos personalizados de treinos (exercícios). O back-end é escrito em Rust, 
e o algoritmo que gera o plano de treinos leva em consideração muitos fatores, como 
idade do usuário do aplicativo, índice de massa corporal, preferências de treinos, 
treinos recentes e um número de intensidade que eles especificam. O algoritmo real 
usado não é importante neste exemplo; o importante é que esse cálculo leve alguns segundos. 
Queremos chamar esse algoritmo apenas quando precisarmos e apenas uma vez, para não fazer o 
usuário esperar mais do que o necessário.

Simularemos a chamada desse algoritmo hipotético com a função
`simulated_expensive_calculation` mostrado na Listagem 13-1, que imprimirá 
`calculating slowly...` (calculando lentamente...), aguardará dois segundos e retornará qualquer 
número que tenhamos passado:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn simulated_expensive_calculation(intensity: u32) -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    intensity
}
```

<span class="caption">Listagem 13-1: Uma função para substituir um cálculo hipotético 
que leva cerca de 2 segundos para ser executado</span>

A seguir, a função `main`, que contém as partes do aplicativo de treino importantes 
para este exemplo. Esta função representa o código que o aplicativo chamará 
quando um usuário solicitar um plano de treino. Como a interação com o front-end do 
aplicativo não é relevante para o uso de closure, codificaremos os valores que 
representam as entradas do nosso programa e imprimiremos as saídas.

As entradas necessárias são estas:

* Um número de intensidade do usuário, especificado quando ele solicita um 
treino para indicar se deseja um treino de baixa intensidade ou um treino de alta intensidade
* Um número aleatório que irá gerar alguma variedade nos planos de treino

A saída será o plano de treino recomendado. A Listagem 13-2 mostra a função `main` 
que usaremos:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let simulated_user_specified_value = 10;
    let simulated_random_number = 7;

    generate_workout(
        simulated_user_specified_value,
        simulated_random_number
    );
}
# fn generate_workout(intensity: u32, random_number: u32) {}
```

<span class="caption">Listagem 13-2: Uma função `main` com valores codificados 
para simular a entrada do usuário e a geração aleatória de números</span>

Codificamos a variável `simulated_user_specified_value` como 10 e a variável 
`simulated_random_number` como 7 por uma questão de simplicidade; em um programa 
real, obteríamos o número de intensidade da interface do aplicativo e usaríamos 
a crate `rand` para gerar um número aleatório, como fizemos no exemplo do jogo 
de adivinhação no capítulo 2. A função `main` chama uma função `generate_workout` 
com os valores de entrada simulados.

Agora que temos o contexto, vamos ao algoritmo. A função
`generate_workout` na Listagem 13-3 contém a lógica de negócios do
aplicativo com o qual estamos mais preocupados neste exemplo. O restante 
das alterações de código neste exemplo será feito para esta função.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
# fn simulated_expensive_calculation(num: u32) -> u32 {
#     println!("calculating slowly...");
#     thread::sleep(Duration::from_secs(2));
#     num
# }
#
fn generate_workout(intensity: u32, random_number: u32) {
    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            simulated_expensive_calculation(intensity)
        );
        println!(
            "Next, do {} situps!",
            simulated_expensive_calculation(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                simulated_expensive_calculation(intensity)
            );
        }
    }
}
```

<span class="caption">Listagem 13-3: A lógica de negócios que imprime os planos 
de treino com base nas entradas e chama a função `simulated_expensive_calculation`</span>

O código na Listagem 13-3 possui várias chamadas para a função de _cálculo lento_. 
O primeiro bloco `if` chama `simulated_expensive_calculation` duas vezes, o `if` 
dentro do `else` externo não é chamado, e o código dentro do segundo caso `else` 
o chama uma vez.

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

O comportamento desejado da função `generate_workout` é primeiro verificar se 
o usuário deseja um treino de baixa intensidade (indicado por um número menor 
que 25) ou um treino de alta intensidade (um número igual ou superior a 25).

Os planos de treino de baixa intensidade recomendam várias flexões e abdominais 
com base no algoritmo complexo que estamos simulando.

Se o usuário deseja um treino de alta intensidade, há uma lógica adicional: 
se o valor do número aleatório gerado pelo aplicativo for 3, o aplicativo 
recomendará uma pausa e hidratação. Caso contrário, o usuário terá vários 
minutos de execução com base no algoritmo complexo.

Esse código funciona da maneira que a empresa (negócio) deseja agora, mas digamos que a 
equipe de ciência de dados decida que precisamos fazer algumas alterações no futuro 
na forma como chamamos a função `simulated_expensive_calculation`. Para simplificar 
a atualização quando essas alterações ocorrerem, queremos refatorar esse código para 
que ele chame a função `simulated_expensive_calculation` apenas uma vez. Também queremos 
reduzir o local em que estamos chamando a função desnecessariamente duas vezes, sem adicionar 
outras chamadas a essa função no processo. Ou seja, não queremos chamá-lo se o resultado 
não for necessário e ainda queremos chamá-lo apenas uma vez.

#### Refatoração Usando Funções

Poderíamos reestruturar o programa de treinos de várias maneiras. Primeiro, 
tentaremos extrair a chamada duplicada para a função `simulated_expensive_calculation` 
em uma variável, conforme mostrado na Listagem 13-4:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
# fn simulated_expensive_calculation(num: u32) -> u32 {
#     println!("calculating slowly...");
#     thread::sleep(Duration::from_secs(2));
#     num
# }
#
fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_result =
        simulated_expensive_calculation(intensity);

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_result
        );
        println!(
            "Next, do {} situps!",
            expensive_result
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_result
            );
        }
    }
}
```

<span class="caption">Listagem 13-4: Extraindo as chamadas para
`simulated_expensive_calculation` em um único local e armazenando o resultado na 
variável `expensive_result`</span>

Esta mudança unifica todas as chamadas para `simulated_expensive_calculation` e 
resolve o problema do primeiro bloco `if` chamando desnecessariamente a função duas 
vezes. Infelizmente, agora estamos chamando essa função e aguardando o resultado em 
todos os casos, o que inclui o bloco interno `if` que não usa o valor do resultado.

Queremos definir o código em um local do nosso programa, mas apenas *executar* 
o código onde realmente precisamos do resultado. Este é um caso de uso para closures!

#### Refatoração com Closures para Código da Loja

Em vez de sempre chamar a função `simulated_expensive_calculation` antes dos blocos `if`, 
podemos definir um closure e armazenar o *closure* em uma variável em vez de armazenar 
o resultado da chamada de função, como mostra a Listagem 13-5. Na verdade, podemos mover 
todo o corpo de `simulated_expensive_calculation` dentro do closure que estamos 
apresentando aqui:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
let expensive_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
# expensive_closure(5);
```

<span class="caption">Listagem 13-5: Definindo um closure e armazenando-o 
na variável `expensive_closure`</span>

A definição de closure vem após o `=` para atribuí-lo à variável `expensive_closure`. 
Para definir um closure, começamos com um par de tubos (pipes) verticais (`|`), dentro 
dos quais especificamos os parâmetros para closure; essa sintaxe foi escolhida devido à 
sua semelhança com as definições de closure em Smalltalk e Ruby. Esse closure possui um 
parâmetro chamado `num`: se tivéssemos mais de um parâmetro, os separaríamos com vírgulas, 
como `|param1, param2|`.

Após os parâmetros, colocamos colchetes que seguram o corpo do closure; eles são 
opcionais se o corpo do closure for uma expressão única. O final do closure, depois 
dos colchetes, precisa de um ponto e vírgula para concluir a declaração `let`. 
O valor retornado da última linha no corpo do closure (`num`) será o valor retornado 
do closure quando for chamado, porque essa linha não termina em ponto e vírgula; 
assim como nos corpos funcionais.

Observe que esta declaração `let` significa que `expensive_closure` contém a *definição* 
de uma função anônima, não o *valor resultante* de chamar a função anônima. Lembre-se de 
que estamos usando um closure porque queremos definir o código para chamar em um ponto, 
armazenar esse código e chamá-lo em um momento posterior; o código que queremos chamar 
agora está armazenado em `expensive_closure`.

Com o closure definido, podemos alterar o código nos blocos `if` para chamar o closure 
para executar o código e obter o valor resultante. Chamamos o closure como fazemos com 
uma função: especificamos o nome da variável que mantém a definição de closure e a 
seguimos com parênteses contendo os valores do argumento que queremos usar, 
como mostra a Listagem 13-6:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_closure = |num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_closure(intensity)
        );
        println!(
            "Next, do {} situps!",
            expensive_closure(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_closure(intensity)
            );
        }
    }
}
```

<span class="caption">Listagem 13-6: Chamando o `expensive_closure` 
que definimos</span>

Agora, o cálculo caro (expensive; lento) é chamado em apenas um lugar, e estamos 
apenas executando esse código onde precisamos dos resultados.

No entanto, reintroduzimos um dos problemas da Lista 13-3: ainda estamos chamando 
closure duas vezes no primeiro bloco `if`, que chamará o código caro duas vezes 
e fará com que o usuário espere o dobro do tempo necessário. Podemos resolver esse 
problema criando uma variável local para esse bloco `if` para conter o resultado 
de chamar o closure, mas os closures nos fornecem outra solução. Falaremos sobre 
essa solução daqui a pouco. Mas primeiro vamos falar sobre por que não há anotações 
de tipo na definição de closure e as traits envolvidas nos closures.

### Inferência e Anotação de Tipo de Closure

Os closures não exigem que você anote os tipos dos parâmetros ou o valor de retorno, 
como as funções `fn`. As anotações de tipo são necessárias nas funções porque fazem 
parte de uma interface explícita exposta aos seus usuários. Definir rigidamente essa 
interface é importante para garantir que todos concordem com os tipos de valores que 
uma função usa e retorna. Mas os closures não são usados em uma interface exposta como 
esta: eles são armazenados em variáveis e usados sem nomeá-los e expô-los aos usuários 
de nossa biblioteca.

Os closures são geralmente curtos e relevantes apenas dentro de um contexto estreito 
(específico) e não em qualquer cenário arbitrário. Nesses contextos limitados, o compilador 
é capaz de inferir com segurança os tipos dos parâmetros e o tipo de retorno, semelhante 
à forma como é capaz de inferir os tipos da maioria das variáveis.

Fazer com que os programadores anotem os tipos nessas pequenas funções anônimas 
seria irritante e amplamente redundante com as informações que o compilador já 
tem disponível.

Assim como nas variáveis, podemos adicionar anotações de tipo se quisermos aumentar 
explicitação e clareza ao custo de ser mais detalhado do que o estritamente 
necessário. Anotar os tipos para o closure que definimos na Lista 13-5 seria 
semelhante à definição mostrada na Lista 13-7:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
let expensive_closure = |num: u32| -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

<span class="caption">Listagem 13-7: Incluindo anotações de tipo opcional 
do parâmetro e retornando tipos de valor no closure</span>

Com as anotações de tipo adicionadas, a sintaxe dos closures se parece mais com 
a sintaxe das funções. A seguir, é apresentada uma comparação vertical da sintaxe 
para a definição de uma função que adiciona 1 ao seu parâmetro e um closure que 
tem o mesmo comportamento. Adicionamos alguns espaços para alinhar as partes relevantes. 
Isso ilustra como a sintaxe de closure é semelhante à sintaxe de função, exceto 
pelo uso de pipes e a quantidade de sintaxe opcional:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

A primeira linha mostra uma definição de função e a segunda linha mostra uma definição 
de closure totalmente anotada. A terceira linha remove as anotações de tipo da definição 
na closure e a quarta linha remove os colchetes, que são opcionais porque o corpo do 
closure possui apenas uma expressão. Todas essas são definições válidas que produzirão 
o mesmo comportamento quando forem chamadas.

As definições de closure terão um tipo concreto inferido para cada um de seus parâmetros 
e para seu valor de retorno. Por exemplo, a Listagem 13-8 mostra a definição de um closure 
curto que apenas retorna o valor que recebe como parâmetro. Esse closure não é muito útil, 
exceto para os fins deste exemplo. Observe que não adicionamos anotações de tipo à definição: 
se tentarmos chamar o closure duas vezes, usando uma `String` como argumento na primeira 
vez e um `u32` na segunda vez, obteremos um erro .

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```

<span class="caption">Listing 13-8: Attempting to call a closure whose types
are inferred with two different types</span>

The compiler gives us this error:

```text
error[E0308]: mismatched types
 --> src/main.rs
  |
  | let n = example_closure(5);
  |                         ^ expected struct `std::string::String`, found
  integral variable
  |
  = note: expected type `std::string::String`
             found type `{integer}`
```

A primeira vez que chamamos `example_closure` com o valor `String`, o compilador 
deduz o tipo de `x` e o tipo de retorno do closure para `String`. Esses tipos 
são bloqueados no closure em `example_closure`, e obtemos um erro de tipo se 
tentarmos usar um tipo diferente com o mesmo closure.

### Armazenando Closures Usando Parâmetros Genéricos e as Traits `Fn`

Vamos voltar ao nosso aplicativo de geração de treinos. Na Listagem 13-6, 
nosso código ainda estava chamando o closure de cálculo caro mais vezes do 
que o necessário. Uma opção para resolver esse problema é salvar o resultado 
do closure caro em uma variável para reutilização e usar a variável em cada 
local em que precisamos do resultado, em vez de chamar o closure novamente. 
No entanto, esse método pode resultar em muitos códigos repetidos.

Felizmente, outra solução está disponível para nós. Podemos criar uma estrutura 
que reterá o closure e o valor resultante de chamar o closure. A estrutura executará 
o closure apenas se precisarmos do valor resultante e armazenará em cache o valor 
resultante, para que o restante do nosso código não seja responsável por salvar e 
reutilizar o resultado. Você pode conhecer esse padrão como *memoization* (memorização) 
ou *lazy evaluation* (avaliação lenta).

Para criar uma estrutura que mantenha um closure, precisamos especificar o tipo 
do closure, porque uma definição de estrutura precisa conhecer os tipos de cada 
um de seus campos. Cada instância do closure tem seu próprio tipo anônimo: ou seja, 
mesmo que dois closures tenham a mesma assinatura, seus tipos ainda são considerados 
diferentes. Para definir estruturas, enumerações ou parâmetros de função que usam 
closures, usamos limites (bounds) genéricos e de traits, conforme discutimos no Capítulo 10.

As traits `Fn` são fornecidas pela biblioteca padrão. Todos os closures implementam 
pelo menos uma das traits: `Fn`,` FnMut` ou `FnOnce`. Discutiremos a diferença entre 
essas traits na seção "Capturando o Ambiente com Closures"; Neste exemplo, podemos 
usar a trait `Fn`.

Nós adicionamos tipos ao trait `Fn` vinculado, para representar os tipos dos 
parâmetros e retornamos valores que closures devem ter para corresponder 
a esse trait vinculado. Nesse caso, nosso closure possui um parâmetro do tipo 
`u32` e retorna um `u32`, portanto a trait que especificamos é `Fn(u32) -> u32`.

A Listagem 13-9 mostra a definição da estrutura do `Cacher` que contém um closure 
e um valor de resultado opcional:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}
```

<span class="caption">Listagem 13-9: Definindo uma estrutura `Cacher` que mantém 
um closure em `calculation` e um resultado opcional em `value`</span>

A estrutura `Cacher` possui um campo `calculation` do tipo genérico `T`. Os limites 
da trait em `T` especificam que é um closure usando trait `Fn`. Qualquer 
closure que queremos armazenar no campo `calculation` deve ter um parâmetro `u32` 
(especificado dentro dos parênteses após `Fn`) e deve retornar um `u32` 
(especificado após o `->`).

> Nota: As funções implementam todas as três traits `Fn` também. Se o que queremos 
> fazer não exige a captura de um valor do ambiente, podemos usar uma função em 
> vez de um closure, onde precisamos de algo que implemente uma trait `Fn`.

O campo `value` é do tipo `Option<u32>`. Antes de executarmos o closure, o `value` 
será `None`. Quando o código que usa `Cacher` pede o *result* (resultado) do closure, o 
`Cacher` executará o closure naquele momento e armazenará o resultado dentro 
de um variante `Some` no campo `value`. Então, se o código solicitar o resultado 
do closure novamente, em vez de executar o closure novamente, o `Cacher` retornará 
o resultado mantido na variante `Some`.

A lógica em torno do campo `value` que acabamos de descrever é definida 
na Listagem 13-10:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
# struct Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     calculation: T,
#     value: Option<u32>,
# }
#
impl<T> Cacher<T>
    where T: Fn(u32) -> u32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}
```

<span class="caption">Lista 13-10: A lógica de armazenamento em cache do `Cacher`</span>

Queremos que o `Cacher` gerencie os valores dos campos da estrutura, em vez de 
permitir que o código de chamada potencialmente altere os valores nesses campos 
diretamente, que esses campos sejam privados.

A função `Cacher::new` usa um parâmetro genérico `T`, que definimos como tendo o 
mesmo trait vinculado à estrutura do `Cacher`. Então `Cacher::new` retorna uma 
instância do `Cacher` que contém o closure especificado no campo `calculation` e 
um valor `None` no campo `value`, porque ainda não o executamos.

Quando o código de chamada precisa do resultado da avaliação do closure, em vez 
de chamar diretamente o closure, ele chamará o método `value`. Este método verifica 
se já temos um valor resultante em `self.value` em um `Some`; se tivermos, ele 
retornará o valor dentro de `Some` sem executar o closure novamente.

Se `self.value` for `None`, o código chama o closure armazenado em
`self.calculation`, salva o resultado em `self.value` para uso futuro, e
retorna o valor também.

A Listagem 13-11 mostra como podemos usar essa estrutura do `Cacher` na 
função `generate_workout` da Listagem 13-6:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
# struct Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     calculation: T,
#     value: Option<u32>,
# }
#
# impl<T> Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     fn new(calculation: T) -> Cacher<T> {
#         Cacher {
#             calculation,
#             value: None,
#         }
#     }
#
#     fn value(&mut self, arg: u32) -> u32 {
#         match self.value {
#             Some(v) => v,
#             None => {
#                 let v = (self.calculation)(arg);
#                 self.value = Some(v);
#                 v
#             },
#         }
#     }
# }
#
fn generate_workout(intensity: u32, random_number: u32) {
    let mut expensive_result = Cacher::new(|num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    });

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_result.value(intensity)
        );
        println!(
            "Next, do {} situps!",
            expensive_result.value(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_result.value(intensity)
            );
        }
    }
}
```

<span class="caption">Lista 13-11: Usando o `Cacher` na função `generate_workout` 
para abstrair a lógica de armazenamento em cache</span>

Em vez de salvar o closure em uma variável diretamente, salvamos uma nova instância 
do `Cacher` que mantém o closure. Então, em cada lugar que queremos o resultado, 
chamamos o método `value` na instância do `Cacher`. Podemos chamar o método `value` 
quantas vezes quisermos, ou não chamá-lo nunca, e o cálculo caro será executado 
no máximo uma vez.

Tente executar este programa com a função `main` da Lista 13-2. Mude os valores 
nas variáveis `simulated_user_specified_value` e `simulated_random_number` para 
verificar que em todos os casos nos vários blocos `if` e `else`, 
`calculating slowly...` apareça apenas uma vez e somente quando necessário. O `Cacher` 
cuida da lógica necessária para garantir que não estamos chamando o cálculo caro mais 
do que precisamos, para que `generate_workout` possa se concentrar na lógica de negócios.

### Limitações da Implementação do `Cacher`

O armazenamento em cache de valores é um comportamento geralmente útil que podemos 
usar em outras partes do nosso código com closures diferentes. No entanto, existem 
dois problemas com a implementação atual do `Cacher` que dificultariam sua 
reutilização em diferentes contextos.

O primeiro problema é que uma instância do `Cacher` assume que sempre obterá o 
mesmo valor para o parâmetro `arg` no método `value`. Ou seja, este teste do 
`Cacher` falhará:

```rust,ignore
#[test]
fn call_with_different_values() {
    let mut c = Cacher::new(|a| a);

    let v1 = c.value(1);
    let v2 = c.value(2);

    assert_eq!(v2, 2);
}
```

Este teste cria uma nova instância do `Cacher` com um closure que retorna o valor 
passado para ele. Chamamos o método `value` nesta instância do `Cacher` com um 
valor `arg` de 1 e, em seguida, um valor `arg` de 2, e esperamos que a chamada 
para `value` com o valor `arg` de 2 deva retornar 2)

Execute este teste com a implementação do `Cacher` na Lista 13-9 e na Lista 13-10, 
e o teste falhará no `assert_eq!` com esta mensagem:

```text
thread 'call_with_different_values' panicked at 'assertion failed: `(left == right)`
  left: `1`,
 right: `2`', src/main.rs
```

O problema é que, na primeira vez em que chamamos `c.value` com 1, a instância do 
`Cacher` salvou `Some(1)` em `self.value`. Depois disso, não importa o que passamos 
para o método `value`, ele sempre retornará 1.

Tente modificar o `Cacher` para conter um mapa de hash em vez de um único valor. 
As chaves do mapa de hash serão os valores `arg` que são passados, e os valores 
do mapa de hash serão o resultado de chamar o closure dessa chave. Em vez de 
verificar se o `self.value` possui diretamente o valor `Some` ou `None`, a função
`value` procurará o `arg` no mapa de hash e retornará o valor, se estiver presente. 
Se não estiver presente, o `Cacher` chamará o closure e salvará o valor resultante 
no mapa de hash associado ao seu valor `arg`.

O segundo problema com a implementação atual do `Cacher` é que ele aceita apenas 
closures que pegam um parâmetro do tipo `u32` e retornam um `u32`. Podemos querer 
armazenar em cache os resultados de closures que pegam uma fatia de string e retornam 
valores `usize`, por exemplo. Para corrigir esse problema, tente introduzir parâmetros 
mais genéricos para aumentar a flexibilidade da funcionalidade do `Cacher`.

### Capturando o Ambiente com Closures

No exemplo do gerador de treinos, usamos apenas closures como funções anônimas 
inline. No entanto, os closures têm um recurso adicional que as funções não têm: 
eles podem capturar seu ambiente e acessar variáveis do escopo em que estão definidos.

A Lista 13-12 tem um exemplo de um closure armazenado na variável `equal_to_x` 
que usa a variável `x` do ambiente circundante do closure:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x;

    let y = 4;

    assert!(equal_to_x(y));
}
```

<span class="caption">Listagem 13-12: Exemplo de closure que se refere a uma 
variável em seu escopo anexo (acerca)</span>

Aqui, embora `x` não seja um dos parâmetros de `equal_to_x`, o closure `equal_to_x` 
pode usar a variável `x` que é definida no mesmo escopo em que `equal_to_x` é definido.

Não podemos fazer o mesmo com funções; se tentarmos o exemplo a seguir, 
nosso código não será compilado:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let x = 4;

    fn equal_to_x(z: i32) -> bool { z == x }

    let y = 4;

    assert!(equal_to_x(y));
}
```

Recebemos o error:

```text
error[E0434]: can't capture dynamic environment in a fn item; use the || { ...
} closure form instead
 --> src/main.rs
  |
4 |     fn equal_to_x(z: i32) -> bool { z == x }
  |                                          ^
```

O compilador até nos lembra que isso só funciona com closures!

Quando um closure captura um valor de seu ambiente, ele usa memória para armazenar 
os valores para uso no corpo do closure. Esse uso de memória é uma sobrecarga que 
não queremos pagar nos casos mais comuns em que queremos executar código que não 
captura seu ambiente. Como as funções nunca podem capturar seu ambiente, a definição 
e o uso de funções nunca terão essa sobrecarga.

Os closures podem capturar valores de seu ambiente de três maneiras, que mapeiam 
diretamente as três maneiras pelas quais uma função pode assumir um parâmetro: 
ownership, tomar borrow mutuamente e tomar borrow imutáveis. Estes são 
codificados nas três traits `Fn` da seguinte maneira:

* `FnOnce` consome as variáveis que captura do seu escopo anexo, conhecido como *environment*
(ambiente) do closure. Para consumir as variáveis capturadas, o closure deve ter ownership 
dessas variáveis e movê-las para o closure quando é definido. A parte `Once` do nome 
representa o fato de que o closure não pode ter ownership das mesmas variáveis mais de uma vez, 
para que ele possa ser chamado apenas uma vez.
* `FnMut` pode mudar o ambiente porque _borrows_ valores mutuamente.
* `Fn` borrows valores imutáveis do ambiente.

Quando você cria um closure, Rust infere qual trait usar com base em 
como o closure usa os valores do ambiente. Todos os closures implementam o `FnOnce` 
porque todos podem ser chamados pelo menos uma vez. Closures que não movem as 
variáveis capturadas também implementam `FnMut`, e closures que não precisam 
de acesso mutável às variáveis capturadas também implementam` Fn`. Na Listagem 13-12, o
closure `equal_to_x` empresta imutável `x` (para que `equal_to_x` tenha a 
trait `Fn`) porque o corpo do closure precisa apenas ler o valor em `x`.

Se você deseja forçar o closure a ter ownership dos valores que ele usa no ambiente, 
pode usar a palavra-chave `move` antes da lista de parâmetros. Essa técnica é útil 
principalmente ao passar um closure para uma nova thread para mover os dados, para 
que sejam de ownership da nova thread.

Teremos mais exemplos de closure `move` no capítulo 16 quando falarmos sobre 
concorrência. Por enquanto, aqui está o código da Lista 13-12 com a palavra-chave 
`move` adicionada à definição de closure e usando vetores em vez de números 
inteiros, porque os números inteiros podem ser copiados em vez de movidos; 
observe que esse código ainda não será compilado.

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let x = vec![1, 2, 3];

    let equal_to_x = move |z| z == x;

    println!("can't use x here: {:?}", x);

    let y = vec![1, 2, 3];

    assert!(equal_to_x(y));
}
```

Podemos receber os seguintes erros:

```text
error[E0382]: use of moved value: `x`
 --> src/main.rs:6:40
  |
4 |     let equal_to_x = move |z| z == x;
  |                      -------- value moved (into closure) here
5 |
6 |     println!("can't use x here: {:?}", x);
  |                                        ^ value used here after move
  |
  = note: move occurs because `x` has type `std::vec::Vec<i32>`, which does not
  implement the `Copy` trait
```

O valor `x` é movido para o closure quando o closure é definido, porque adicionamos 
a palavra-chave `move`. O closure tem o ownership de `x`, e `main` não pode mais 
usar `x` na declaração `println! `. A remoção de `println!` irá corrigir este exemplo.

Na maioria das vezes, ao especificar um dos limites de trait `Fn`, você pode 
começar com `Fn` e o compilador informará se você precisa de `FnMut` ou `FnOnce` 
com base no que acontece no corpo do closure.

Para ilustrar situações em que closures capazes de capturar seu ambiente são úteis 
como parâmetros de função, vamos para o próximo tópico: iteradores.
