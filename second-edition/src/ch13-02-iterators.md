## Processing a Series of Items with Iterators
## Processando uma Série de Itens com Interadores

The iterator pattern allows you to perform some task on a sequence of items in
turn. An iterator is responsible for the logic of iterating over each item and
determining when the sequence has finished. When you use iterators, you don’t
have to reimplement that logic yourself.

O padrão (pattern) do iterador permite executar alguma tarefa em uma sequência 
de itens por vez. Um iterador é responsável pela lógica de iterar sobre cada item e 
determinar quando a sequência foi concluída. Quando você usa iteradores, não 
precisa reimplementar essa lógica.

In Rust, iterators are *lazy*, meaning they have no effect until you call
methods that consume the iterator to use it up. For example, the code in
Listing 13-13 creates an iterator over the items in the vector `v1` by calling
the `iter` method defined on `Vec`. This code by itself doesn’t do anything
useful.

Em Rust, os iteradores são *preguiçosos* (lazy), o que significa que não têm 
efeito até você chamar métodos que consomem o iterador para usá-lo. Por exemplo, 
o código na Listagem 13-13 cria um iterador sobre os itens no vetor `v1` chamando 
o método `iter` definido em `Vec`. Esse código por si só não faz nada de útil.

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();
```

<span class="caption">Listing 13-13: Creating an iterator</span>

<span class="caption">Listing 13-13: Criando um interador</span>

Once we’ve created an iterator, we can use it in a variety of ways. In Listing
3-5 in Chapter 3, we used iterators with `for` loops to execute some code on
each item, although we glossed over what the call to `iter` did until now.

Depois de criar um iterador, podemos usá-lo de várias maneiras. Na Listagem 3-5 
do Capítulo 3, usamos iteradores com loops `for` para executar algum código em 
cada item, embora tenhamos encobertado o que a chamada para `iter` fez até agora.

The example in Listing 13-14 separates the creation of the iterator from the
use of the iterator in the `for` loop. The iterator is stored in the `v1_iter`
variable, and no iteration takes place at that time. When the `for` loop is
called using the iterator in `v1_iter`, each element in the iterator is used in
one iteration of the loop, which prints out each value.

O exemplo na Listagem 13-14 separa a criação do iterador do uso do iterador no loop 
`for`. O iterador é armazenado na variável `v1_iter`, e nenhuma iteração ocorre naquele 
momento. Quando o loop `for` é chamado usando o iterador no `v1_iter`, cada elemento no 
iterador é usado em uma iteração do loop, que imprime cada valor.

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

for val in v1_iter {
    println!("Got: {}", val);
}
```

<span class="caption">Listing 13-14: Using an iterator in a `for` loop</span>

<span class="caption">Listagem 13-14: Usando um interador em um loop `for`</span>

In languages that don’t have iterators provided by their standard libraries,
you would likely write this same functionality by starting a variable at index
0, using that variable to index into the vector to get a value, and
incrementing the variable value in a loop until it reached the total number of
items in the vector.

Em linguagens que não possuem iteradores fornecidas por suas bibliotecas padrão, 
você provavelmente escreveria essa mesma funcionalidade iniciando uma variável no 
índice 0, usando essa variável para indexar no vetor para obter um valor e 
incrementando o valor da variável em um loop até atingir o número total de itens 
no vetor.

Iterators handle all that logic for you, cutting down on repetitive code you
could potentially mess up. Iterators give you more flexibility to use the same
logic with many different kinds of sequences, not just data structures you can
index into, like vectors. Let’s examine how iterators do that.

Os iteradores lidam com toda essa lógica para você, reduzindo o código repetitivo 
que você pode acabar estragando. Os iteradores oferecem mais flexibilidade para usar 
a mesma lógica com muitos tipos diferentes de seqüências, não apenas estruturas de 
dados nas quais você pode indexar, como vetores. Vamos examinar como os iteradores 
fazem isso.

### The `Iterator` Trait and the `next` Method
### A Trait `Iterator` e o Método `next`

All iterators implement a trait named `Iterator` that is defined in the
standard library. The definition of the trait looks like this:

Todos os iteradores implementam uma trait chamada `Iterator` que é definida na 
biblioteca padrão. A definição da trait é assim:

```rust
trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

Notice this definition uses some new syntax: `type Item` and `Self::Item`,
which are defining an *associated type* with this trait. We’ll talk about
associated types in depth in Chapter 19. For now, all you need to know is that
this code says implementing the `Iterator` trait requires that you also define
an `Item` type, and this `Item` type is used in the return type of the `next`
method. In other words, the `Item` type will be the type returned from the
iterator.

Observe que esta definição usa uma nova sintaxe: `type Item` e `Self::Item`, 
que estão definindo um *tipo associado* (associated type) com essa trait. 
Falaremos sobre os tipos associados em profundidade no Capítulo 19. Por enquanto, 
tudo o que você precisa saber é que este código diz que a implementação da trait 
`Iterator` exige que você também defina um tipo de `Item` e esse tipo `Item` é usado 
no tipo de retorno do método `next`. Em outras palavras, o tipo `Item` será o 
tipo retornado do iterador.

The `Iterator` trait only requires implementors to define one method: the
`next` method, which returns one item of the iterator at a time wrapped in
`Some` and, when iteration is over, returns `None`.

A trait `Iterator` exige apenas que os implementadores definam um 
método: o método `next`, que retorna um item do iterador por vez, envolvido 
em `Some` e, quando a iteração termina, retorna `None`.

We can call the `next` method on iterators directly; Listing 13-15 demonstrates
what values are returned from repeated calls to `next` on the iterator created
from the vector:

Podemos chamar o método `next` diretamente nos iteradores; A Listagem 13-15 demonstra 
quais valores são retornados de chamadas repetidas para `next` no iterador criado 
a partir do vetor:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

<span class="caption">Listing 13-15: Calling the `next` method on an
iterator</span>

<span class="caption">Listagem 13-15: Chamando o método `next` em um
iterador </span>

Note that we needed to make `v1_iter` mutable: calling the `next` method on an
iterator changes internal state that the iterator uses to keep track of where
it is in the sequence. In other words, this code *consumes*, or uses up, the
iterator. Each call to `next` eats up an item from the iterator. We didn’t need
to make `v1_iter` mutable when we used a `for` loop because the loop took
ownership of `v1_iter` and made it mutable behind the scenes.

Observe que é necessário tornar o `v1_iter` mutável: chamar o método `next` em um 
iterador altera o estado interno que o iterador usa para acompanhar onde está na 
sequência. Em outras palavras, esse código *consome* ou usa o iterador. Cada chamada 
para `next` consome um item do iterador. Não precisamos tornar o `v1_iter` mutável 
quando usamos um loop `for` porque o loop toma ownership do `v1_iter` e o tornou 
mutável nos bastidores.

Also note that the values we get from the calls to `next` are immutable
references to the values in the vector. The `iter` method produces an iterator
over immutable references. If we want to create an iterator that takes
ownership of `v1` and returns owned values, we can call `into_iter` instead of
`iter`. Similarly, if we want to iterate over mutable references, we can call
`iter_mut` instead of `iter`.

Observe também que os valores que obtemos das chamadas para o `next` são referências 
imutáveis aos valores do vetor. O método `iter` produz um iterador sobre referências 
imutáveis. Se queremos criar um iterador que tenha ownership de `v1` e retorna valores 
de ownership, podemos chamar `into_iter` em vez de `iter`. Da mesma forma, se queremos 
iterar sobre referências mutáveis, podemos chamar `iter_mut` em vez de `iter`.

### Methods that Consume the Iterator
### Métodos que Consomem o Interador

The `Iterator` trait has a number of different methods with default
implementations provided by the standard library; you can find out about these
methods by looking in the standard library API documentation for the `Iterator`
trait. Some of these methods call the `next` method in their definition, which
is why you’re required to implement the `next` method when implementing the
`Iterator` trait.

A trait `Iterator` possui vários métodos diferentes, com implementações padrão 
fornecidas pela biblioteca padrão; você pode descobrir sobre esses métodos 
procurando na documentação da API da biblioteca padrão a trait `Iterator`. 
Alguns desses métodos chamam o método `next` em sua definição, e é por isso que 
você precisa implementar o método `next` ao implementar a trait `Iterator`.

Methods that call `next` are called *consuming adaptors*, because calling them
uses up the iterator. One example is the `sum` method, which takes ownership of
the iterator and iterates through the items by repeatedly calling `next`, thus
consuming the iterator. As it iterates through, it adds each item to a running
total and returns the total when iteration is complete. Listing 13-16 has a
test illustrating a use of the `sum` method:

Os métodos que chamam `next` são chamados de *consuming adaptors* (consumidores de 
adaptadores), porque chamá-los consome o iterador. Um exemplo é o método `sum`, 
que assume ownership do iterador e itera pelos itens chamando repetidamente `next`, 
consumindo o iterador. À medida que itera, ele adiciona cada item a um total em 
execução e retorna o total quando a iteração é concluída. A Listagem 13-16 possui 
um teste que ilustra o uso do método `sum`:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    let total: i32 = v1_iter.sum();

    assert_eq!(total, 6);
}
```

<span class="caption">Listing 13-16: Calling the `sum` method to get the total
of all items in the iterator</span>

<span class="caption">Listagem 13-16: Chamando o método `sum` para obter o total 
de todos os itens no iterador</span>

We aren’t allowed to use `v1_iter` after the call to `sum` because `sum` takes
ownership of the iterator we call it on.

Não é permitido o uso de `v1_iter` após a chamada para `sum`, porque `sum` assume 
ownership do iterador em que o chamamos.

### Methods that Produce Other Iterators
### Métodos que Produzem Outros Interadores

Other methods defined on the `Iterator` trait, known as *iterator adaptors*,
allow you to change iterators into different kinds of iterators. You can chain
multiple calls to iterator adaptors to perform complex actions in a readable
way. But because all iterators are lazy, you have to call one of the consuming
adaptor methods to get results from calls to iterator adaptors.

Outros métodos definidos na trait `Iterator`, conhecidos como *iterator adaptors* 
(adaptadores de iterador), permitem alterar os iteradores em diferentes tipos 
de iteradores. Você pode encadear várias chamadas para adaptadores do iterador 
para executar ações complexas de maneira legível. Porém, como todos os iteradores 
são preguiçosos, é necessário chamar um dos métodos do adaptador consumidor para 
obter resultados das chamadas para os adaptadores do iterador.

Listing 13-17 shows an example of calling the iterator adaptor method `map`,
which takes a closure to call on each item to produce a new iterator. The
closure here creates a new iterator in which each item from the vector has been
incremented by 1. However, this code produces a warning:

A Listagem 13-17 mostra um exemplo de chamada do método do adaptador do iterador 
`map`, que exige um closure para chamar cada item para produzir um novo iterador. 
O closure aqui cria um novo iterador no qual cada item do vetor foi incrementado 
em 1. No entanto, esse código produz um aviso:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
let v1: Vec<i32> = vec![1, 2, 3];

v1.iter().map(|x| x + 1);
```

<span class="caption">Listing 13-17: Calling the iterator adaptor `map` to
create a new iterator</span>

<span class="caption">Listagem 13-17: Chamando o adaptador do iterador `map` 
para criar um novo iterador</span>

The warning we get is this:

O aviso que recebemos é este:

```text
warning: unused `std::iter::Map` which must be used: iterator adaptors are lazy
and do nothing unless consumed
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: #[warn(unused_must_use)] on by default
```

The code in Listing 13-17 doesn’t do anything; the closure we’ve specified
never gets called. The warning reminds us why: iterator adaptors are lazy, and
we need to consume the iterator here.

O código na Listagem 13-17 não faz nada; o closure que especificamos nunca é 
chamado. O aviso nos lembra o porquê: os adaptadores do iterador são preguiçosos 
e precisamos consumir o iterador aqui.

To fix this and consume the iterator, we’ll use the `collect` method, which we
used in Chapter 12 with `env::args` in Listing 12-1. This method consumes the
iterator and collects the resulting values into a collection data type.

Para corrigir isso e consumir o iterador, usaremos o método `collect`, que 
usamos no Capítulo 12 com `env::args` na Listagem 12-1. Este método consome 
o iterador e coleta os valores resultantes em um tipo de dados de coleção.

In Listing 13-18, we collect the results of iterating over the iterator that’s
returned from the call to `map` into a vector. This vector will end up
containing each item from the original vector incremented by 1.

Na Lista 13-18, coletamos os resultados da iteração sobre o iterador retornado 
da chamada para `map` em um vetor. Esse vetor acabará contendo cada item do vetor 
original incrementado por 1.

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

<span class="caption">Listing 13-18: Calling the `map` method to create a new
iterator and then calling the `collect` method to consume the new iterator and
create a vector</span>

<span class="caption">Listagem 13-18: Chamando o método `map` para criar um novo 
iterador e, em seguida, chamando o método `collect` para consumir o novo iterador 
e criar um vetor</span>

Because `map` takes a closure, we can specify any operation we want to perform
on each item. This is a great example of how closures let you customize some
behavior while reusing the iteration behavior that the `Iterator` trait
provides.

Como o `map` pega um closure, podemos especificar qualquer operação que deseje 
executar em cada item. Este é um ótimo exemplo de como os closures permitem 
personalizar algum comportamento ao reutilizar o comportamento de iteração 
que o trait `Iterator` fornece.

### Using Closures that Capture Their Environment
### Usando Closures que Capturam seu Ambiente

Now that we’ve introduced iterators, we can demonstrate a common use of
closures that capture their environment by using the `filter` iterator adaptor.
The `filter` method on an iterator takes a closure that takes each item from
the iterator and returns a Boolean. If the closure returns `true`, the value
will be included in the iterator produced by `filter`. If the closure returns
`false`, the value won’t be included in the resulting iterator.

Agora que introduzimos iteradores, podemos demonstrar um uso comum de closures que 
capturam seu ambiente usando o adaptador iterador `filter`. O método `filter` 
em um iterador faz um closure que pega cada item do iterador e retorna um booleano. 
Se o closure retornar `true`, o valor será incluído no iterador produzido pelo `filter`. 
Se o closure retornar `false`, o valor não será incluído no iterador resultante.

In Listing 13-19, we use `filter` with a closure that captures the `shoe_size`
variable from its environment to iterate over a collection of `Shoe` struct
instances. It will return only shoes that are the specified size.

Na Lista 13-19, usamos `filter` com um closure que captura a variável `shoe_size` 
do seu ambiente para iterar sobre uma coleção de instâncias da estrutura do `Shoe`. 
Ele retornará apenas shoes (sapatos) com o tamanho especificado.

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
        .filter(|s| s.size == shoe_size)
        .collect()
}

#[test]
fn filters_by_size() {
    let shoes = vec![
        Shoe { size: 10, style: String::from("sneaker") },
        Shoe { size: 13, style: String::from("sandal") },
        Shoe { size: 10, style: String::from("boot") },
    ];

    let in_my_size = shoes_in_my_size(shoes, 10);

    assert_eq!(
        in_my_size,
        vec![
            Shoe { size: 10, style: String::from("sneaker") },
            Shoe { size: 10, style: String::from("boot") },
        ]
    );
}
```

<span class="caption">Listing 13-19: Using the `filter` method with a closure
that captures `shoe_size`</span>

<span class="caption">Lista 13-19: Usando o método `filter` com um closure 
que captura `shoe_size`</span>

The `shoes_in_my_size` function takes ownership of a vector of shoes and a shoe
size as parameters. It returns a vector containing only shoes of the specified
size.

A função `shoes_in_my_size` assume ownership de um vetor de shoes e um tamanho 
de shoes como parâmetros. Retorna um vetor que contém apenas shoes do tamanho 
especificado.

In the body of `shoes_in_my_size`, we call `into_iter` to create an iterator
that takes ownership of the vector. Then we call `filter` to adapt that
iterator into a new iterator that only contains elements for which the closure
returns `true`.

No corpo de `shoes_in_my_size`, chamamos `into_iter` para criar um iterador 
que pega ownership do vetor. Em seguida, chamamos `filter` para adaptar 
esse iterador a um novo iterador que contém apenas elementos para os quais 
o closure retorna `true`.

The closure captures the `shoe_size` parameter from the environment and
compares the value with each shoe’s size, keeping only shoes of the size
specified. Finally, calling `collect` gathers the values returned by the
adapted iterator into a vector that’s returned by the function.

O closure captura o parâmetro `shoe_size` do ambiente e compara o valor com 
o tamanho de cada shoes, mantendo apenas shoes do tamanho especificado. Por 
fim, chamar `collect` reúne os valores retornados pelo iterador adaptado 
em um vetor retornado pela função.

The test shows that when we call `shoes_in_my_size`, we get back only shoes
that have the same size as the value we specified.

O teste mostra que, quando chamamos `shoes_in_my_size`, recebemos apenas 
shoes que têm o mesmo tamanho que o valor especificado.

### Creating Our Own Iterators with the `Iterator` Trait
### Criando Nossos Próprios Iteradores com a Trait `Iterator`

We’ve shown that you can create an iterator by calling `iter`, `into_iter`, or
`iter_mut` on a vector. You can create iterators from the other collection
types in the standard library, such as hash map. You can also create iterators
that do anything you want by implementing the `Iterator` trait on your own
types. As previously mentioned, the only method you’re required to provide a
definition for is the `next` method. Once you’ve done that, you can use all
other methods that have default implementations provided by the `Iterator`
trait!

Mostramos que você pode criar um iterador chamando `iter`, `into_iter` ou `iter_mut` 
em um vetor. Você pode criar iteradores a partir de outros tipos de coleção na 
biblioteca padrão, como mapa de hash. Você também pode criar iteradores que fazem 
o que quiser, implementando a trait `Iterator` nos seus próprios tipos. Como 
mencionado anteriormente, o único método para o qual você precisa fornecer uma 
definição é o método `next`. Depois de fazer isso, você poderá usar todos os 
outros métodos que possuem implementações padrão fornecidas pela trait `Iterator`!

To demonstrate, let’s create an iterator that will only ever count from 1 to 5.
First, we’ll create a struct to hold some values. Then we’ll make this struct
into an iterator by implementing the `Iterator` trait and using the values in
that implementation.

Para demonstrar, vamos criar um iterador que contará apenas de 1 a 5. Primeiro, 
criaremos uma estrutura para armazenar alguns valores. Em seguida, transformaremos 
essa estrutura em um iterador implementando a trait `Iterator` e usando os 
valores nessa implementação.

Listing 13-20 has the definition of the `Counter` struct and an associated
`new` function to create instances of `Counter`:

A Listagem 13-20 tem a definição da estrutura do `Counter` e uma 
função `new` associada para criar instâncias de `Counter`:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}
```

<span class="caption">Listing 13-20: Defining the `Counter` struct and a `new`
function that creates instances of `Counter` with an initial value of 0 for
`count`</span>

<span class="caption">Listagem 13-20: Definindo a estrutura do `Counter` e uma 
função `new` que cria instâncias do `Counter` com um valor inicial de 0 
para `count`</span>

The `Counter` struct has one field named `count`. This field holds a `u32`
value that will keep track of where we are in the process of iterating from 1
to 5. The `count` field is private because we want the implementation of
`Counter` to manage its value. The `new` function enforces the behavior of
always starting new instances with a value of 0 in the `count` field.

A estrutura `Counter` possui um campo chamado `count`. Este campo possui um 
valor `u32` que acompanhará onde estamos no processo de iteração de 1 a 5. 
O campo `count` é privado porque queremos que a implementação do `Counter` 
gerencie seu valor. A função `new` reforça o comportamento de sempre iniciar 
novas instâncias com um valor 0 no campo `count`.

Next, we’ll implement the `Iterator` trait for our `Counter` type by defining
the body of the `next` method to specify what we want to happen when this
iterator is used, as shown in Listing 13-21:

Em seguida, implementaremos a trait `Iterator` para o nosso tipo `Counter`, 
definindo o corpo do método `next` para especificar o que queremos que aconteça 
quando esse iterador for usado, como mostra a Listagem 13-21:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# struct Counter {
#     count: u32,
# }
#
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

<span class="caption">Listing 13-21: Implementing the `Iterator` trait on our
`Counter` struct</span>

<span class="caption">Listagem 13-21: Implementando a trait `Iterator` em 
nossa estrutura do `Counter`</span>

We set the associated `Item` type for our iterator to `u32`, meaning the
iterator will return `u32` values. Again, don’t worry about associated types
yet, we’ll cover them in Chapter 19.

Definimos o tipo `Item` associado ao nosso iterador como `u32`, o que significa 
que o iterador retornará os valores `u32`. Novamente, não se preocupe com os 
tipos associados ainda, abordaremos eles no capítulo 19.

We want our iterator to add 1 to the current state, so we initialized `count`
to 0 so it would return 1 first. If the value of `count` is less than 6, `next`
will return the current value wrapped in `Some`, but if `count` is 6 or higher,
our iterator will return `None`.

Queremos que nosso iterador adicione 1 ao estado atual; portanto, inicializamos `count` 
em 0, para que retornasse 1 primeiro. Se o valor de `count` for menor que 6, `next` 
retornará o valor atual envolvido em `Some`, mas se `count` for 6 ou superior, 
nosso iterador retornará `None`.

#### Using Our `Counter` Iterator’s `next` Method
#### Usando o Método `next` do Nosso Interador `Counter`

Once we’ve implemented the `Iterator` trait, we have an iterator! Listing 13-22
shows a test demonstrating that we can use the iterator functionality of our
`Counter` struct by calling the `next` method on it directly, just as we did
with the iterator created from a vector in Listing 13-15.

Depois de implementar a trait `Iterator`, temos um iterador! A Listagem 13-22 mostra 
um teste demonstrando que podemos usar a funcionalidade do iterador da nossa 
estrutura `Counter` chamando diretamente o método `next`, exatamente como fizemos 
com o iterador criado a partir de um vetor na Listagem 13-15.

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# struct Counter {
#     count: u32,
# }
#
# impl Iterator for Counter {
#     type Item = u32;
#
#     fn next(&mut self) -> Option<Self::Item> {
#         self.count += 1;
#
#         if self.count < 6 {
#             Some(self.count)
#         } else {
#             None
#         }
#     }
# }
#
#[test]
fn calling_next_directly() {
    let mut counter = Counter::new();

    assert_eq!(counter.next(), Some(1));
    assert_eq!(counter.next(), Some(2));
    assert_eq!(counter.next(), Some(3));
    assert_eq!(counter.next(), Some(4));
    assert_eq!(counter.next(), Some(5));
    assert_eq!(counter.next(), None);
}
```

<span class="caption">Listing 13-22: Testing the functionality of the `next`
method implementation</span>

<span class="caption">Listagem 13-22: Testando a funcionalidade da implementação 
do método `next`</span>

This test creates a new `Counter` instance in the `counter` variable and then
calls `next` repeatedly, verifying that we have implemented the behavior we
want this iterator to have: returning the values from 1 to 5.

Este teste cria uma nova instância `Counter` na variável `counter` e, em seguida, 
chama `next` repetidamente, verificando se implementamos o comportamento que queremos 
que esse iterador tenha: retornando os valores de 1 a 5.

#### Using Other `Iterator` Trait Methods
#### Usando Outros Métodos de Trait `Iterator`

We implemented the `Iterator` trait by defining the `next` method, so we
can now use any `Iterator` trait method’s default implementations as defined in
the standard library, because they all use the `next` method’s functionality.

Implementamos a trait `Iterator` definindo o método `next`, para que agora 
possamos usar implementações padrão de qualquer método de trait `Iterator`, 
conforme definido na biblioteca padrão, porque todos eles usam a funcionalidade 
do método `next`.

For example, if for some reason we wanted to take the values produced by an
instance of `Counter`, pair them with values produced by another `Counter`
instance after skipping the first value, multiply each pair together, keep only
those results that are divisible by 3, and add all the resulting values
together, we could do so, as shown in the test in Listing 13-23:

Por exemplo, se por algum motivo quisermos pegar os valores produzidos por 
uma instância do `Counter`, emparelhe-os com os valores produzidos por outra 
instância do `Counter` depois de pular o primeiro valor, multiplique cada par 
juntos, mantenha apenas os resultados divisível por 3 e adicione todos os valores 
resultantes juntos, poderíamos fazê-lo, como mostra o teste na Listagem 13-23:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# struct Counter {
#     count: u32,
# }
#
# impl Counter {
#     fn new() -> Counter {
#         Counter { count: 0 }
#     }
# }
#
# impl Iterator for Counter {
#     // Our iterator will produce u32s
#     type Item = u32;
#
#     fn next(&mut self) -> Option<Self::Item> {
#         // increment our count. This is why we started at zero.
#         self.count += 1;
#
#         // check to see if we've finished counting or not.
#         if self.count < 6 {
#             Some(self.count)
#         } else {
#             None
#         }
#     }
# }
#
#[test]
fn using_other_iterator_trait_methods() {
    let sum: u32 = Counter::new().zip(Counter::new().skip(1))
                                 .map(|(a, b)| a * b)
                                 .filter(|x| x % 3 == 0)
                                 .sum();
    assert_eq!(18, sum);
}
```

<span class="caption">Listing 13-23: Using a variety of `Iterator` trait
methods on our `Counter` iterator</span>

<span class="caption">Lista 13-23: Usando uma variedade de métodos do trait
`Iterator` no iterador `Counter`</span>

Note that `zip` produces only four pairs; the theoretical fifth pair `(5,
None)` is never produced because `zip` returns `None` when either of its input
iterators return `None`.

Note que o `zip` produz apenas quatro pares; o quinto par `(5,
None)` nunca é produzido porque `zip` retorna `None` quando um de seus iteradores 
de entrada retorna `None`.

All of these method calls are possible because we specified how the `next`
method works, and the standard library provides default implementations for
other methods that call `next`.

Todas essas chamadas de método são possíveis porque especificamos como o 
método `next` funciona, e a biblioteca padrão fornece implementações padrão 
para outros métodos que chamam `next`.