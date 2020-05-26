## Closures: Anonymous Functions that Can Capture Their Environment
## Closures: Funções Anônimas que Podem Capturar seu Ambiente

Rust’s closures are anonymous functions you can save in a variable or pass as
arguments to other functions. You can create the closure in one place and then
call the closure to evaluate it in a different context. Unlike functions,
closures can capture values from the scope in which they’re called. We’ll
demonstrate how these closure features allow for code reuse and behavior
customization.

Os closures (fechamentos) de Rust são funções anônimas que você pode salvar em uma variável 
ou transmitir como argumentos para outras funções. Você pode criar closure em 
um único local e depois chamá-lo para avaliá-lo em um contexto diferente. Diferentemente 
das funções, os closures podem capturar valores do escopo em que são chamados. 
Vamos demonstrar como esses recursos de closure permitem a reutilização de 
código e a personalização do comportamento.

### Creating an Abstraction of Behavior with Closures
### Criando uma Abstração de Comportamento com Closures

Let’s work on an example of a situation in which it’s useful to store a closure
to be executed later. Along the way, we’ll talk about the syntax of closures,
type inference, and traits.

Vamos trabalhar em um exemplo de situação em que é útil armazenar um closure para 
ser executado posteriormente. Ao longo do caminho, falaremos sobre a sintaxe de closures, 
inferência de tipos e _traits_.

Consider this hypothetical situation: we work at a startup that’s making an app
to generate custom exercise workout plans. The backend is written in Rust, and
the algorithm that generates the workout plan takes into account many factors,
such as the app user’s age, body mass index, exercise preferences, recent
workouts, and an intensity number they specify. The actual algorithm used isn’t
important in this example; what’s important is that this calculation takes a
few seconds. We want to call this algorithm only when we need to and only call
it once so we don’t make the user wait more than necessary.

Considere esta situação hipotética: trabalhamos em uma startup que está criando um 
aplicativo para gerar planos personalizados de treinos (exercícios). O back-end é escrito em Rust, 
e o algoritmo que gera o plano de treinos leva em consideração muitos fatores, como 
idade do usuário do aplicativo, índice de massa corporal, preferências de treinos, 
treinos recentes e um número de intensidade que eles especificam. O algoritmo real 
usado não é importante neste exemplo; o importante é que esse cálculo leve alguns segundos. 
Queremos chamar esse algoritmo apenas quando precisarmos e apenas uma vez, para não fazer o 
usuário esperar mais do que o necessário.

We’ll simulate calling this hypothetical algorithm with the function
`simulated_expensive_calculation` shown in Listing 13-1, which will print
`calculating slowly...`, wait for two seconds, and then return whatever number
we passed in:

Simularemos a chamada desse algoritmo hipotético com a função
`simulated_expensive_calculation` mostrado na Listagem 13-1, que imprimirá 
`calculating slowly...` (calculando lentamente...), aguardará dois segundos e retornará qualquer 
número que tenhamos passado:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-1: A function to stand in for a hypothetical
calculation that takes about 2 seconds to run</span>

<span class="caption">Listagem 13-1: Uma função para substituir um cálculo hipotético 
que leva cerca de 2 segundos para ser executado</span>

Next is the `main` function, which contains the parts of the workout app
important for this example. This function represents the code that the app will
call when a user asks for a workout plan. Because the interaction with the
app’s frontend isn’t relevant to the use of closures, we’ll hardcode values
representing inputs to our program and print the outputs.

A seguir, a função `main`, que contém as partes do aplicativo de treino importantes 
para este exemplo. Esta função representa o código que o aplicativo chamará 
quando um usuário solicitar um plano de treino. Como a interação com o front-end do 
aplicativo não é relevante para o uso de closure, codificaremos os valores que 
representam as entradas do nosso programa e imprimiremos as saídas.

The required inputs are these:

* An intensity number from the user, which is specified when they request
  a workout to indicate whether they want a low-intensity workout or a
  high-intensity workout
* A random number that will generate some variety in the workout plans

As entradas necessárias são estas:

* Um número de intensidade do usuário, especificado quando ele solicita um 
treino para indicar se deseja um treino de baixa intensidade ou um treino de alta intensidade
* Um número aleatório que irá gerar alguma variedade nos planos de treino

The output will be the recommended workout plan. Listing 13-2 shows the `main`
function we’ll use:

A saída será o plano de treino recomendado. A Listagem 13-2 mostra a função `main` 
que usaremos:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-2: A `main` function with hardcoded values to
simulate user input and random number generation</span>

<span class="caption">Listagem 13-2: Uma função `main` com valores codificados 
para simular a entrada do usuário e a geração aleatória de números</span>

We’ve hardcoded the variable `simulated_user_specified_value` as 10 and the
variable `simulated_random_number` as 7 for simplicity’s sake; in an actual
program, we’d get the intensity number from the app frontend, and we’d use the
`rand` crate to generate a random number, as we did in the Guessing Game
example in Chapter 2. The `main` function calls a `generate_workout` function
with the simulated input values.

Codificamos a variável `simulated_user_specified_value` como 10 e a variável 
`simulated_random_number` como 7 por uma questão de simplicidade; em um programa 
real, obteríamos o número de intensidade da interface do aplicativo e usaríamos 
a crate `rand` para gerar um número aleatório, como fizemos no exemplo do jogo 
de adivinhação no capítulo 2. A função `main` chama uma função `generate_workout` 
com os valores de entrada simulados.

Now that we have the context, let’s get to the algorithm. The function
`generate_workout` in Listing 13-3 contains the business logic of the
app that we’re most concerned with in this example. The rest of the code
changes in this example will be made to this function.

Agora que temos o contexto, vamos ao algoritmo. A função
`generate_workout` na Listagem 13-3 contém a lógica de negócios do
aplicativo com o qual estamos mais preocupados neste exemplo. O restante 
das alterações de código neste exemplo será feito para esta função.

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-3: The business logic that prints the workout
plans based on the inputs and calls to the `simulated_expensive_calculation`
function</span>

<span class="caption">Listagem 13-3: A lógica de negócios que imprime os planos 
de treino com base nas entradas e chama a função `simulated_expensive_calculation`</span>

The code in Listing 13-3 has multiple calls to the slow calculation function.
The first `if` block calls `simulated_expensive_calculation` twice, the `if`
inside the outer `else` doesn’t call it at all, and the code inside the
second `else` case calls it once.

O código na Listagem 13-3 possui várias chamadas para a função de _cálculo lento_. 
O primeiro bloco `if` chama `simulated_expensive_calculation` duas vezes, o `if` 
dentro do `else` externo não é chamado, e o código dentro do segundo caso `else` 
o chama uma vez.

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

The desired behavior of the `generate_workout` function is to first check
whether the user wants a low-intensity workout (indicated by a number less
than 25) or a high-intensity workout (a number of 25 or greater).

O comportamento desejado da função `generate_workout` é primeiro verificar se 
o usuário deseja um treino de baixa intensidade (indicado por um número menor 
que 25) ou um treino de alta intensidade (um número igual ou superior a 25).

Low-intensity workout plans will recommend a number of push-ups and sit-ups
based on the complex algorithm we’re simulating.

Os planos de treino de baixa intensidade recomendam várias flexões e abdominais 
com base no algoritmo complexo que estamos simulando.

If the user wants a high-intensity workout, there’s some additional logic: if
the value of the random number generated by the app happens to be 3, the app
will recommend a break and hydration. If not, the user will get a number of
minutes of running based on the complex algorithm.

Se o usuário deseja um treino de alta intensidade, há uma lógica adicional: 
se o valor do número aleatório gerado pelo aplicativo for 3, o aplicativo 
recomendará uma pausa e hidratação. Caso contrário, o usuário terá vários 
minutos de execução com base no algoritmo complexo.

This code works the way the business wants it to now, but let’s say the data
science team decides that we need to make some changes to the way we call the
`simulated_expensive_calculation` function in the future. To simplify the
update when those changes happen, we want to refactor this code so it calls the
`simulated_expensive_calculation` function only once. We also want to cut the
place where we’re currently unnecessarily calling the function twice without
adding any other calls to that function in the process. That is, we don’t want
to call it if the result isn’t needed, and we still want to call it only once.

Esse código funciona da maneira que a empresa (negócio) deseja agora, mas digamos que a 
equipe de ciência de dados decida que precisamos fazer algumas alterações no futuro 
na forma como chamamos a função `simulated_expensive_calculation`. Para simplificar 
a atualização quando essas alterações ocorrerem, queremos refatorar esse código para 
que ele chame a função `simulated_expensive_calculation` apenas uma vez. Também queremos 
reduzir o local em que estamos chamando a função desnecessariamente duas vezes, sem adicionar 
outras chamadas a essa função no processo. Ou seja, não queremos chamá-lo se o resultado 
não for necessário e ainda queremos chamá-lo apenas uma vez.

#### Refactoring Using Functions
#### Refatoração Usando Funções

We could restructure the workout program in many ways. First, we’ll try
extracting the duplicated call to the `simulated_expensive_calculation`
function into a variable, as shown in Listing 13-4:

Poderíamos reestruturar o programa de treinos de várias maneiras. Primeiro, 
tentaremos extrair a chamada duplicada para a função `simulated_expensive_calculation` 
em uma variável, conforme mostrado na Listagem 13-4:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-4: Extracting the calls to
`simulated_expensive_calculation` to one place and storing the result in the
`expensive_result` variable</span>

<span class="caption">Listagem 13-4: Extraindo as chamadas para
`simulated_expensive_calculation` em um único local e armazenando o resultado na 
variável `expensive_result`</span>

This change unifies all the calls to `simulated_expensive_calculation` and
solves the problem of the first `if` block unnecessarily calling the function
twice. Unfortunately, we’re now calling this function and waiting for the
result in all cases, which includes the inner `if` block that doesn’t use the
result value at all.

Esta mudança unifica todas as chamadas para `simulated_expensive_calculation` e 
resolve o problema do primeiro bloco `if` chamando desnecessariamente a função duas 
vezes. Infelizmente, agora estamos chamando essa função e aguardando o resultado em 
todos os casos, o que inclui o bloco interno `if` que não usa o valor do resultado.

We want to define code in one place in our program, but only *execute* that
code where we actually need the result. This is a use case for closures!

Queremos definir o código em um local do nosso programa, mas apenas *executar* 
o código onde realmente precisamos do resultado. Este é um caso de uso para closures!

#### Refactoring with Closures to Store Code
#### Refatoração com Closures para Código da Loja

Instead of always calling the `simulated_expensive_calculation` function before
the `if` blocks, we can define a closure and store the *closure* in a variable
rather than storing the result of the function call, as shown in Listing 13-5.
We can actually move the whole body of `simulated_expensive_calculation` within
the closure we’re introducing here:

Em vez de sempre chamar a função `simulated_expensive_calculation` antes dos blocos `if`, 
podemos definir um closure e armazenar o *closure* em uma variável em vez de armazenar 
o resultado da chamada de função, como mostra a Listagem 13-5. Na verdade, podemos mover 
todo o corpo de `simulated_expensive_calculation` dentro do closure que estamos 
apresentando aqui:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-5: Defining a closure and storing it in the
`expensive_closure` variable</span>

<span class="caption">Listagem 13-5: Definindo um closure e armazenando-o 
na variável `expensive_closure`</span>

The closure definition comes after the `=` to assign it to the variable
`expensive_closure`. To define a closure, we start with a pair of vertical
pipes (`|`), inside which we specify the parameters to the closure; this syntax
was chosen because of its similarity to closure definitions in Smalltalk and
Ruby. This closure has one parameter named `num`: if we had more than one
parameter, we would separate them with commas, like `|param1, param2|`.

A definição de closure vem após o `=` para atribuí-lo à variável `expensive_closure`. 
Para definir um closure, começamos com um par de tubos (pipes) verticais (`|`), dentro 
dos quais especificamos os parâmetros para closure; essa sintaxe foi escolhida devido à 
sua semelhança com as definições de closure em Smalltalk e Ruby. Esse closure possui um 
parâmetro chamado `num`: se tivéssemos mais de um parâmetro, os separaríamos com vírgulas, 
como `|param1, param2|`.

After the parameters, we place curly brackets that hold the body of the
closure—these are optional if the closure body is a single expression. The end
of the closure, after the curly brackets, needs a semicolon to complete the
`let` statement. The value returned from the last line in the closure body
(`num`) will be the value returned from the closure when it’s called, because
that line doesn’t end in a semicolon; just like in function bodies.

Após os parâmetros, colocamos colchetes que seguram o corpo do closure; eles são 
opcionais se o corpo do closure for uma expressão única. O final do closure, depois 
dos colchetes, precisa de um ponto e vírgula para concluir a declaração `let`. 
O valor retornado da última linha no corpo do closure (`num`) será o valor retornado 
do closure quando for chamado, porque essa linha não termina em ponto e vírgula; 
assim como nos corpos funcionais.

Note that this `let` statement means `expensive_closure` contains the
*definition* of an anonymous function, not the *resulting value* of calling the
anonymous function. Recall that we’re using a closure because we want to define
the code to call at one point, store that code, and call it at a later point;
the code we want to call is now stored in `expensive_closure`.

Observe que esta declaração `let` significa que `expensive_closure` contém a *definição* 
de uma função anônima, não o *valor resultante* de chamar a função anônima. Lembre-se de 
que estamos usando um closure porque queremos definir o código para chamar em um ponto, 
armazenar esse código e chamá-lo em um momento posterior; o código que queremos chamar 
agora está armazenado em `expensive_closure`.

With the closure defined, we can change the code in the `if` blocks to call the
closure to execute the code and get the resulting value. We call a closure like
we do a function: we specify the variable name that holds the closure
definition and follow it with parentheses containing the argument values we
want to use, as shown in Listing 13-6:

Com o closure definido, podemos alterar o código nos blocos `if` para chamar o closure 
para executar o código e obter o valor resultante. Chamamos o closure como fazemos com 
uma função: especificamos o nome da variável que mantém a definição de closure e a 
seguimos com parênteses contendo os valores do argumento que queremos usar, 
como mostra a Listagem 13-6:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-6: Calling the `expensive_closure` we’ve
defined</span>

<span class="caption">Listagem 13-6: Chamando o `expensive_closure` 
que definimos</span>

Now the expensive calculation is called in only one place, and we’re only
executing that code where we need the results.

Agora, o cálculo caro (expensive; lento) é chamado em apenas um lugar, e estamos 
apenas executando esse código onde precisamos dos resultados.

However, we’ve reintroduced one of the problems from Listing 13-3: we’re still
calling the closure twice in the first `if` block, which will call the
expensive code twice and make the user wait twice as long as they need to. We
could fix this problem by creating a variable local to that `if` block to hold
the result of calling the closure, but closures provide us with another
solution. We’ll talk about that solution in a bit. But first let’s talk about
why there aren’t type annotations in the closure definition and the traits
involved with closures.

No entanto, reintroduzimos um dos problemas da Lista 13-3: ainda estamos chamando 
closure duas vezes no primeiro bloco `if`, que chamará o código caro duas vezes 
e fará com que o usuário espere o dobro do tempo necessário. Podemos resolver esse 
problema criando uma variável local para esse bloco `if` para conter o resultado 
de chamar o closure, mas os closures nos fornecem outra solução. Falaremos sobre 
essa solução daqui a pouco. Mas primeiro vamos falar sobre por que não há anotações 
de tipo na definição de closure e as traits envolvidas nos closures.

### Closure Type Inference and Annotation
### Inferência e Anotação de Tipo de Closure

Closures don’t require you to annotate the types of the parameters or the
return value like `fn` functions do. Type annotations are required on functions
because they’re part of an explicit interface exposed to your users. Defining
this interface rigidly is important for ensuring that everyone agrees on what
types of values a function uses and returns. But closures aren’t used in an
exposed interface like this: they’re stored in variables and used without
naming them and exposing them to users of our library.

Os closures não exigem que você anote os tipos dos parâmetros ou o valor de retorno, 
como as funções `fn`. As anotações de tipo são necessárias nas funções porque fazem 
parte de uma interface explícita exposta aos seus usuários. Definir rigidamente essa 
interface é importante para garantir que todos concordem com os tipos de valores que 
uma função usa e retorna. Mas os closures não são usados em uma interface exposta como 
esta: eles são armazenados em variáveis e usados sem nomeá-los e expô-los aos usuários 
de nossa biblioteca.

Closures are usually short and relevant only within a narrow context rather
than in any arbitrary scenario. Within these limited contexts, the compiler is
reliably able to infer the types of the parameters and the return type, similar
to how it’s able to infer the types of most variables.

Os closures são geralmente curtos e relevantes apenas dentro de um contexto estreito 
(específico) e não em qualquer cenário arbitrário. Nesses contextos limitados, o compilador 
é capaz de inferir com segurança os tipos dos parâmetros e o tipo de retorno, semelhante 
à forma como é capaz de inferir os tipos da maioria das variáveis.

Making programmers annotate the types in these small, anonymous functions would
be annoying and largely redundant with the information the compiler already has
available.

Fazer com que os programadores anotem os tipos nessas pequenas funções anônimas 
seria irritante e amplamente redundante com as informações que o compilador já 
tem disponível.

As with variables, we can add type annotations if we want to increase
explicitness and clarity at the cost of being more verbose than is strictly
necessary. Annotating the types for the closure we defined in Listing 13-5
would look like the definition shown in Listing 13-7:

Assim como nas variáveis, podemos adicionar anotações de tipo se quisermos aumentar 
explicitação e clareza ao custo de ser mais detalhado do que o estritamente 
necessário. Anotar os tipos para o closure que definimos na Lista 13-5 seria 
semelhante à definição mostrada na Lista 13-7:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-7: Adding optional type annotations of the
parameter and return value types in the closure</span>

<span class="caption">Listagem 13-7: Incluindo anotações de tipo opcional 
do parâmetro e retornando tipos de valor no closure</span>

With type annotations added, the syntax of closures looks more similar to the
syntax of functions. The following is a vertical comparison of the syntax for
the definition of a function that adds 1 to its parameter and a closure that
has the same behavior. We’ve added some spaces to line up the relevant parts.
This illustrates how closure syntax is similar to function syntax except for
the use of pipes and the amount of syntax that is optional:

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

The first line shows a function definition, and the second line shows a fully
annotated closure definition. The third line removes the type annotations from
the closure definition, and the fourth line removes the brackets, which are
optional because the closure body has only one expression. These are all valid
definitions that will produce the same behavior when they’re called.

A primeira linha mostra uma definição de função e a segunda linha mostra uma definição 
de closure totalmente anotada. A terceira linha remove as anotações de tipo da definição 
na closure e a quarta linha remove os colchetes, que são opcionais porque o corpo do 
closure possui apenas uma expressão. Todas essas são definições válidas que produzirão 
o mesmo comportamento quando forem chamadas.

Closure definitions will have one concrete type inferred for each of their
parameters and for their return value. For instance, Listing 13-8 shows the
definition of a short closure that just returns the value it receives as a
parameter. This closure isn’t very useful except for the purposes of this
example. Note that we haven’t added any type annotations to the definition: if
we then try to call the closure twice, using a `String` as an argument the
first time and a `u32` the second time, we’ll get an error.

As definições de closure terão um tipo concreto inferido para cada um de seus parâmetros 
e para seu valor de retorno. Por exemplo, a Listagem 13-8 mostra a definição de um closure 
curto que apenas retorna o valor que recebe como parâmetro. Esse closure não é muito útil, 
exceto para os fins deste exemplo. Observe que não adicionamos anotações de tipo à definição: 
se tentarmos chamar o closure duas vezes, usando uma `String` como argumento na primeira 
vez e um `u32` na segunda vez, obteremos um erro .

<span class="filename">Filename: src/main.rs</span>

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

The first time we call `example_closure` with the `String` value, the compiler
infers the type of `x` and the return type of the closure to be `String`. Those
types are then locked in to the closure in `example_closure`, and we get a type
error if we try to use a different type with the same closure.

A primeira vez que chamamos `example_closure` com o valor `String`, o compilador 
deduz o tipo de `x` e o tipo de retorno do closure para `String`. Esses tipos 
são bloqueados no closure em `example_closure`, e obtemos um erro de tipo se 
tentarmos usar um tipo diferente com o mesmo closure.

### Storing Closures Using Generic Parameters and the `Fn` Traits
### Armazenando Closures Usando Parâmetros Genéricos e as Traits `Fn`

Let’s return to our workout generation app. In Listing 13-6, our code was still
calling the expensive calculation closure more times than it needed to. One
option to solve this issue is to save the result of the expensive closure in a
variable for reuse and use the variable in each place we need the result,
instead of calling the closure again. However, this method could result in a
lot of repeated code.

Vamos voltar ao nosso aplicativo de geração de treinos. Na Listagem 13-6, 
nosso código ainda estava chamando o closure de cálculo caro mais vezes do 
que o necessário. Uma opção para resolver esse problema é salvar o resultado 
do closure caro em uma variável para reutilização e usar a variável em cada 
local em que precisamos do resultado, em vez de chamar o closure novamente. 
No entanto, esse método pode resultar em muitos códigos repetidos.

Fortunately, another solution is available to us. We can create a struct that
will hold the closure and the resulting value of calling the closure. The
struct will execute the closure only if we need the resulting value, and it
will cache the resulting value so the rest of our code doesn’t have to be
responsible for saving and reusing the result. You may know this pattern as
*memoization* or *lazy evaluation*.

Felizmente, outra solução está disponível para nós. Podemos criar uma estrutura 
que reterá o closure e o valor resultante de chamar o closure. A estrutura executará 
o closure apenas se precisarmos do valor resultante e armazenará em cache o valor 
resultante, para que o restante do nosso código não seja responsável por salvar e 
reutilizar o resultado. Você pode conhecer esse padrão como *memoization* (memorização) 
ou *lazy evaluation* (avaliação lenta).

To make a struct that holds a closure, we need to specify the type of the
closure, because a struct definition needs to know the types of each of its
fields. Each closure instance has its own unique anonymous type: that is, even
if two closures have the same signature, their types are still considered
different. To define structs, enums, or function parameters that use closures,
we use generics and trait bounds, as we discussed in Chapter 10.

Para criar uma estrutura que mantenha um closure, precisamos especificar o tipo 
do closure, porque uma definição de estrutura precisa conhecer os tipos de cada 
um de seus campos. Cada instância do closure tem seu próprio tipo anônimo: ou seja, 
mesmo que dois closures tenham a mesma assinatura, seus tipos ainda são considerados 
diferentes. Para definir estruturas, enumerações ou parâmetros de função que usam 
closures, usamos limites (bounds) genéricos e de traits, conforme discutimos no Capítulo 10.

The `Fn` traits are provided by the standard library. All closures implement at
least one of the traits: `Fn`, `FnMut`, or `FnOnce`. We’ll discuss the
difference between these traits in the “Capturing the Environment with
Closures” section; in this example, we can use the `Fn` trait.

As traits `Fn` são fornecidas pela biblioteca padrão. Todos os closures implementam 
pelo menos uma das traits: `Fn`,` FnMut` ou `FnOnce`. Discutiremos a diferença entre 
essas traits na seção "Capturando o Ambiente com Closures"; Neste exemplo, podemos 
usar a trait `Fn`.

We add types to the `Fn` trait bound to represent the types of the parameters
and return values the closures must have to match this trait bound. In this
case, our closure has a parameter of type `u32` and returns a `u32`, so the
trait bound we specify is `Fn(u32) -> u32`.

Nós adicionamos tipos ao trait `Fn` vinculado, para representar os tipos dos 
parâmetros e retornamos valores que closures devem ter para corresponder 
a esse trait vinculado. Nesse caso, nosso closure possui um parâmetro do tipo 
`u32` e retorna um `u32`, portanto a trait que especificamos é `Fn(u32) -> u32`.

Listing 13-9 shows the definition of the `Cacher` struct that holds a closure
and an optional result value:

A Listagem 13-9 mostra a definição da estrutura do `Cacher` que contém um closure 
e um valor de resultado opcional:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}
```

<span class="caption">Listing 13-9: Defining a `Cacher` struct that holds a
closure in `calculation` and an optional result in `value`</span>

<span class="caption">Listagem 13-9: Definindo uma estrutura `Cacher` que mantém 
um closure em `calculation` e um resultado opcional em `value`</span>

The `Cacher` struct has a `calculation` field of the generic type `T`. The
trait bounds on `T` specify that it’s a closure by using the `Fn` trait. Any
closure we want to store in the `calculation` field must have one `u32`
parameter (specified within the parentheses after `Fn`) and must return a
`u32` (specified after the `->`).

A estrutura `Cacher` possui um campo `calculation` do tipo genérico `T`. Os limites 
da trait em `T` especificam que é um closure usando trait `Fn`. Qualquer 
closure que queremos armazenar no campo `calculation` deve ter um parâmetro `u32` 
(especificado dentro dos parênteses após `Fn`) e deve retornar um `u32` 
(especificado após o `->`).

> Note: Functions implement all three of the `Fn` traits too. If what we want
> to do doesn’t require capturing a value from the environment, we can use a
> function rather than a closure where we need something that implements an `Fn`
> trait.

> Nota: As funções implementam todas as três traits `Fn` também. Se o que queremos 
> fazer não exige a captura de um valor do ambiente, podemos usar uma função em 
> vez de um closure, onde precisamos de algo que implemente uma trait `Fn`.

The `value` field is of type `Option<u32>`. Before we execute the closure,
`value` will be `None`. When code using a `Cacher` asks for the *result* of the
closure, the `Cacher` will execute the closure at that time and store the
result within a `Some` variant in the `value` field. Then if the code asks for
the result of the closure again, instead of executing the closure again, the
`Cacher` will return the result held in the `Some` variant.

O campo `value` é do tipo `Option<u32>`. Antes de executarmos o closure, o `value` 
será `None`. Quando o código que usa `Cacher` pede o *result* (resultado) do closure, o 
`Cacher` executará o closure naquele momento e armazenará o resultado dentro 
de um variante `Some` no campo `value`. Então, se o código solicitar o resultado 
do closure novamente, em vez de executar o closure novamente, o `Cacher` retornará 
o resultado mantido na variante `Some`.

The logic around the `value` field we’ve just described is defined in Listing
13-10:

A lógica em torno do campo `value` que acabamos de descrever é definida 
na Listagem 13-10:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-10: The caching logic of `Cacher`</span>

<span class="caption">Lista 13-10: A lógica de armazenamento em cache do `Cacher`</span>

We want `Cacher` to manage the struct fields’ values rather than letting the
calling code potentially change the values in these fields directly, so these
fields are private.

Queremos que o `Cacher` gerencie os valores dos campos da estrutura, em vez de 
permitir que o código de chamada potencialmente altere os valores nesses campos 
diretamente, que esses campos sejam privados.

The `Cacher::new` function takes a generic parameter `T`, which we’ve defined
as having the same trait bound as the `Cacher` struct. Then `Cacher::new`
returns a `Cacher` instance that holds the closure specified in the
`calculation` field and a `None` value in the `value` field, because we haven’t
executed the closure yet.

A função `Cacher::new` usa um parâmetro genérico `T`, que definimos como tendo o 
mesmo trait vinculado à estrutura do `Cacher`. Então `Cacher::new` retorna uma 
instância do `Cacher` que contém o closure especificado no campo `calculation` e 
um valor `None` no campo `value`, porque ainda não o executamos.

When the calling code needs the result of evaluating the closure, instead of
calling the closure directly, it will call the `value` method. This method
checks whether we already have a resulting value in `self.value` in a `Some`;
if we do, it returns the value within the `Some` without executing the closure
again.

Quando o código de chamada precisa do resultado da avaliação do closure, em vez 
de chamar diretamente o closure, ele chamará o método `value`. Este método verifica 
se já temos um valor resultante em `self.value` em um `Some`; se tivermos, ele 
retornará o valor dentro de `Some` sem executar o closure novamente.

If `self.value` is `None`, the code calls the closure stored in
`self.calculation`, saves the result in `self.value` for future use, and
returns the value as well.

Se `self.value` for `None`, o código chama o closure armazenado em
`self.calculation`, salva o resultado em `self.value` para uso futuro, e
retorna o valor também.

Listing 13-11 shows how we can use this `Cacher` struct in the function
`generate_workout` from Listing 13-6:

A Listagem 13-11 mostra como podemos usar essa estrutura do `Cacher` na 
função `generate_workout` da Listagem 13-6:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 13-11: Using `Cacher` in the `generate_workout`
function to abstract away the caching logic</span>

<span class="caption">Lista 13-11: Usando o `Cacher` na função `generate_workout` 
para abstrair a lógica de armazenamento em cache</span>

Instead of saving the closure in a variable directly, we save a new instance of
`Cacher` that holds the closure. Then, in each place we want the result, we
call the `value` method on the `Cacher` instance. We can call the `value`
method as many times as we want, or not call it at all, and the expensive
calculation will be run a maximum of once.

Em vez de salvar o closure em uma variável diretamente, salvamos uma nova instância 
do `Cacher` que mantém o closure. Então, em cada lugar que queremos o resultado, 
chamamos o método `value` na instância do `Cacher`. Podemos chamar o método `value` 
quantas vezes quisermos, ou não chamá-lo nunca, e o cálculo caro será executado 
no máximo uma vez.

Try running this program with the `main` function from Listing 13-2. Change the
values in the `simulated_user_specified_value` and `simulated_random_number`
variables to verify that in all the cases in the various `if` and `else`
blocks, `calculating slowly...` appears only once and only when needed. The
`Cacher` takes care of the logic necessary to ensure we aren’t calling the
expensive calculation more than we need to so `generate_workout` can focus on
the business logic.

Tente executar este programa com a função `main` da Lista 13-2. Mude os valores 
nas variáveis `simulated_user_specified_value` e `simulated_random_number` para 
verificar que em todos os casos nos vários blocos `if` e `else`, 
`calculating slowly...` apareça apenas uma vez e somente quando necessário. O `Cacher` 
cuida da lógica necessária para garantir que não estamos chamando o cálculo caro mais 
do que precisamos, para que `generate_workout` possa se concentrar na lógica de negócios.

### Limitations of the `Cacher` Implementation
### Limitações da Implementação do `Cacher`

Caching values is a generally useful behavior that we might want to use in
other parts of our code with different closures. However, there are two
problems with the current implementation of `Cacher` that would make reusing it
in different contexts difficult.

O armazenamento em cache de valores é um comportamento geralmente útil que podemos 
usar em outras partes do nosso código com closures diferentes. No entanto, existem 
dois problemas com a implementação atual do `Cacher` que dificultariam sua 
reutilização em diferentes contextos.

The first problem is that a `Cacher` instance assumes it will always get the
same value for the parameter `arg` to the `value` method. That is, this test of
`Cacher` will fail:

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

This test creates a new `Cacher` instance with a closure that returns the value
passed into it. We call the `value` method on this `Cacher` instance with an
`arg` value of 1 and then an `arg` value of 2, and we expect the call to
`value` with the `arg` value of 2 should return 2.

Este teste cria uma nova instância do `Cacher` com um closure que retorna o valor 
passado para ele. Chamamos o método `value` nesta instância do `Cacher` com um 
valor `arg` de 1 e, em seguida, um valor `arg` de 2, e esperamos que a chamada 
para `value` com o valor `arg` de 2 deva retornar 2)

Run this test with the `Cacher` implementation in Listing 13-9 and Listing
13-10, and the test will fail on the `assert_eq!` with this message:

Execute este teste com a implementação do `Cacher` na Lista 13-9 e na Lista 13-10, 
e o teste falhará no `assert_eq!` com esta mensagem:

```text
thread 'call_with_different_values' panicked at 'assertion failed: `(left == right)`
  left: `1`,
 right: `2`', src/main.rs
```

The problem is that the first time we called `c.value` with 1, the `Cacher`
instance saved `Some(1)` in `self.value`. Thereafter, no matter what we pass in
to the `value` method, it will always return 1.

O problema é que, na primeira vez em que chamamos `c.value` com 1, a instância do 
`Cacher` salvou `Some(1)` em `self.value`. Depois disso, não importa o que passamos 
para o método `value`, ele sempre retornará 1.

Try modifying `Cacher` to hold a hash map rather than a single value. The keys
of the hash map will be the `arg` values that are passed in, and the values of
the hash map will be the result of calling the closure on that key. Instead of
looking at whether `self.value` directly has a `Some` or a `None` value, the
`value` function will look up the `arg` in the hash map and return the value if
it’s present. If it’s not present, the `Cacher` will call the closure and save
the resulting value in the hash map associated with its `arg` value.

Tente modificar o `Cacher` para conter um mapa de hash em vez de um único valor. 
As chaves do mapa de hash serão os valores `arg` que são passados, e os valores 
do mapa de hash serão o resultado de chamar o closure dessa chave. Em vez de 
verificar se o `self.value` possui diretamente o valor `Some` ou `None`, a função
`value` procurará o `arg` no mapa de hash e retornará o valor, se estiver presente. 
Se não estiver presente, o `Cacher` chamará o closure e salvará o valor resultante 
no mapa de hash associado ao seu valor `arg`.

The second problem with the current `Cacher` implementation is that it only
accepts closures that take one parameter of type `u32` and return a `u32`. We
might want to cache the results of closures that take a string slice and return
`usize` values, for example. To fix this issue, try introducing more generic
parameters to increase the flexibility of the `Cacher` functionality.

O segundo problema com a implementação atual do `Cacher` é que ele aceita apenas 
closures que pegam um parâmetro do tipo `u32` e retornam um `u32`. Podemos querer 
armazenar em cache os resultados de closures que pegam uma fatia de string e retornam 
valores `usize`, por exemplo. Para corrigir esse problema, tente introduzir parâmetros 
mais genéricos para aumentar a flexibilidade da funcionalidade do `Cacher`.

### Capturing the Environment with Closures
### Capturando o Ambiente com Closures

In the workout generator example, we only used closures as inline anonymous
functions. However, closures have an additional capability that functions don’t
have: they can capture their environment and access variables from the scope in
which they’re defined.

No exemplo do gerador de treinos, usamos apenas closures como funções anônimas 
inline. No entanto, os closures têm um recurso adicional que as funções não têm: 
eles podem capturar seu ambiente e acessar variáveis do escopo em que estão definidos.

Listing 13-12 has an example of a closure stored in the `equal_to_x` variable
that uses the `x` variable from the closure’s surrounding environment:

A Lista 13-12 tem um exemplo de um closure armazenado na variável `equal_to_x` 
que usa a variável `x` do ambiente circundante do closure:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x;

    let y = 4;

    assert!(equal_to_x(y));
}
```

<span class="caption">Listing 13-12: Example of a closure that refers to a
variable in its enclosing scope</span>

<span class="caption">Listagem 13-12: Exemplo de closure que se refere a uma 
variável em seu escopo anexo (acerca)</span>

Here, even though `x` is not one of the parameters of `equal_to_x`, the
`equal_to_x` closure is allowed to use the `x` variable that’s defined in the
same scope that `equal_to_x` is defined in.

Aqui, embora `x` não seja um dos parâmetros de `equal_to_x`, o closure `equal_to_x` 
pode usar a variável `x` que é definida no mesmo escopo em que `equal_to_x` é definido.

We can’t do the same with functions; if we try with the following example, our
code won’t compile:

Não podemos fazer o mesmo com funções; se tentarmos o exemplo a seguir, 
nosso código não será compilado:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let x = 4;

    fn equal_to_x(z: i32) -> bool { z == x }

    let y = 4;

    assert!(equal_to_x(y));
}
```

We get an error:

Recebemos o error:

```text
error[E0434]: can't capture dynamic environment in a fn item; use the || { ...
} closure form instead
 --> src/main.rs
  |
4 |     fn equal_to_x(z: i32) -> bool { z == x }
  |                                          ^
```

The compiler even reminds us that this only works with closures!

O compilador até nos lembra que isso só funciona com closures!

When a closure captures a value from its environment, it uses memory to store
the values for use in the closure body. This use of memory is overhead that we
don’t want to pay in more common cases where we want to execute code that
doesn’t capture its environment. Because functions are never allowed to capture
their environment, defining and using functions will never incur this overhead.

Quando um closure captura um valor de seu ambiente, ele usa memória para armazenar 
os valores para uso no corpo do closure. Esse uso de memória é uma sobrecarga que 
não queremos pagar nos casos mais comuns em que queremos executar código que não 
captura seu ambiente. Como as funções nunca podem capturar seu ambiente, a definição 
e o uso de funções nunca terão essa sobrecarga.

Closures can capture values from their environment in three ways, which
directly map to the three ways a function can take a parameter: taking
ownership, borrowing mutably, and borrowing immutably. These are encoded in the
three `Fn` traits as follows:

Os closures podem capturar valores de seu ambiente de três maneiras, que mapeiam 
diretamente as três maneiras pelas quais uma função pode assumir um parâmetro: 
ownership, tomar borrow mutuamente e tomar borrow imutáveis. Estes são 
codificados nas três traits `Fn` da seguinte maneira:

* `FnOnce` consumes the variables it captures from its enclosing scope, known
  as the closure’s *environment*. To consume the captured variables, the
  closure must take ownership of these variables and move them into the closure
  when it is defined. The `Once` part of the name represents the fact that the
  closure can’t take ownership of the same variables more than once, so it can
  be called only once.
* `FnMut` can change the environment because it mutably borrows values.
* `Fn` borrows values from the environment immutably.

* `FnOnce` consome as variáveis que captura do seu escopo anexo, conhecido como *environment*
(ambiente) do closure. Para consumir as variáveis capturadas, o closure deve ter ownership 
dessas variáveis e movê-las para o closure quando é definido. A parte `Once` do nome 
representa o fato de que o closure não pode ter ownership das mesmas variáveis mais de uma vez, 
para que ele possa ser chamado apenas uma vez.
* `FnMut` pode mudar o ambiente porque _borrows_ valores mutuamente.
* `Fn` borrows valores imutáveis do ambiente.

When you create a closure, Rust infers which trait to use based on how the
closure uses the values from the environment. All closures implement `FnOnce`
because they can all be called at least once. Closures that don’t move the
captured variables also implement `FnMut`, and closures that don’t need mutable
access to the captured variables also implement `Fn`. In Listing 13-12, the
`equal_to_x` closure borrows `x` immutably (so `equal_to_x` has the `Fn` trait)
because the body of the closure only needs to read the value in `x`.

Quando você cria um closure, Rust infere qual trait usar com base em 
como o closure usa os valores do ambiente. Todos os closures implementam o `FnOnce` 
porque todos podem ser chamados pelo menos uma vez. Closures que não movem as 
variáveis capturadas também implementam `FnMut`, e closures que não precisam 
de acesso mutável às variáveis capturadas também implementam` Fn`. Na Listagem 13-12, o
closure `equal_to_x` empresta imutável `x` (para que `equal_to_x` tenha a 
trait `Fn`) porque o corpo do closure precisa apenas ler o valor em `x`.

If you want to force the closure to take ownership of the values it uses in the
environment, you can use the `move` keyword before the parameter list. This
technique is mostly useful when passing a closure to a new thread to move the
data so it’s owned by the new thread.

Se você deseja forçar o closure a ter ownership dos valores que ele usa no ambiente, 
pode usar a palavra-chave `move` antes da lista de parâmetros. Essa técnica é útil 
principalmente ao passar um closure para uma nova thread para mover os dados, para 
que sejam de ownership da nova thread.

We’ll have more examples of `move` closures in Chapter 16 when we talk about
concurrency. For now, here’s the code from Listing 13-12 with the `move`
keyword added to the closure definition and using vectors instead of integers,
because integers can be copied rather than moved; note that this code will not
yet compile.

Teremos mais exemplos de closure `move` no capítulo 16 quando falarmos sobre 
concorrência. Por enquanto, aqui está o código da Lista 13-12 com a palavra-chave 
`move` adicionada à definição de closure e usando vetores em vez de números 
inteiros, porque os números inteiros podem ser copiados em vez de movidos; 
observe que esse código ainda não será compilado.

<span class="filename">Filename: src/main.rs</span>

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

We receive the following error:

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

The `x` value is moved into the closure when the closure is defined, because we
added the `move` keyword. The closure then has ownership of `x`, and `main`
isn’t allowed to use `x` anymore in the `println!` statement. Removing
`println!` will fix this example.

O valor `x` é movido para o closure quando o closure é definido, porque adicionamos 
a palavra-chave `move`. O closure tem o ownership de `x`, e `main` não pode mais 
usar `x` na declaração `println! `. A remoção de `println!` irá corrigir este exemplo.

Most of the time when specifying one of the `Fn` trait bounds, you can start
with `Fn` and the compiler will tell you if you need `FnMut` or `FnOnce` based
on what happens in the closure body.

Na maioria das vezes, ao especificar um dos limites de trait `Fn`, você pode 
começar com `Fn` e o compilador informará se você precisa de `FnMut` ou `FnOnce` 
com base no que acontece no corpo do closure.

To illustrate situations where closures that can capture their environment are
useful as function parameters, let’s move on to our next topic: iterators.

Para ilustrar situações em que closures capazes de capturar seu ambiente são úteis 
como parâmetros de função, vamos para o próximo tópico: iteradores.
