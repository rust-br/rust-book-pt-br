## Vectors
## Vetores

The first type we’ll look at is `Vec<T>`, also known as a *vector*. Vectors
allow us to store more than one value in a single data structure that puts all
the values next to each other in memory. Vectors can only store values of the
same type. They are useful in situations where you have a list of items, such
as the lines of text in a file or the prices of items in a shopping cart.


O primeiro tipo que iremos ver é `Vec<T>`, também conhecido como *vetor*. Vetores
permitem guardar mais de um valor na mesma estrutura de dados que coloca todos
os valores um ao lado do outro na memória. Vetores só podem guardar valores do
mesmo tipo. Eles são úteis em situações onde há uma lista de itens, como
as linha de texto em um arquivo ou preços de itens em um carrinho de compras.

### Creating a New Vector
### Criando um Novo Vetor

To create a new, empty vector, we can call the `Vec::new` function:

Para criar um novo vetor, vazio, chamamos a função `Vec::new`:

```rust
let v: Vec<i32> = Vec::new();
```

Note that we added a type annotation here. Since we aren’t inserting any values
into this vector, Rust doesn’t know what kind of elements we intend to store.
This is an important point. Vectors are homogeneous: they may store many
values, but those values must all be the same type. Vectors are implemented
using generics, which Chapter 10 will cover how to use in your own types. For
now, all you need to know is that the `Vec` type provided by the standard
library can hold any type, and when a specific `Vec` holds a specific type, the
type goes within angle brackets. We’ve told Rust that the `Vec` in `v` will
hold elements of the `i32` type.


Note que adicionamos uma anotação de tipo aqui. Como não estamos inserindo nenhum valor
no vetor, Rust não sabe o tipo de elementos que irá guardar.
Isto é um ponto importante. Vetores são homogêneos: eles podem guardar muitos 
valores, mas todos esses valores devem ser do mesmo tipo. Vetores são implementados
usando genéricos, onde o capítulo 10 irá cobrir como usar em seus tipos. Por
agora, tudo o que precisa saber é que o tipo `Vec` fornecido pela biblioteca
padrão pode conter qualquer tipo, e quando um `Vec` específico possui um tipo específico, o
tipo vai dentro de `< >`. Falamos para Rust que `Vec` em `v` guardará
elementos do tipo `i32`.

In real code, Rust can infer the type of value we want to store once we insert
values, so you rarely need to do this type annotation. It’s more common to
create a `Vec` that has initial values, and Rust provides the `vec!` macro for
convenience. The macro will create a new `Vec` that holds the values we give
it. This will create a new `Vec<i32>` that holds the values `1`, `2`, and `3`:

No código real, a Rust pode inferir o tipo de valor que queremos armazenar uma vez que inserimos
valores, então você raramente precisa fazer essa anotação de tipo. É mais comum
criar um `Vec` que possui valores iniciais, e o Rust fornece a macro `vec!` por
conveniência. A macro criará um novo `Vec` que contém os valores que damos
. Isso criará um novo `Vec <i32>` que contém os valores `1`,` 2` e `3`:


```rust
let v = vec![1, 2, 3];
```

Because we’ve given initial `i32` values, Rust can infer that the type of `v`
is `Vec<i32>`, and the type annotation isn’t necessary. Let’s look at how to
modify a vector next.

Como nós damos valores iniciais  `i32`, Rust pode inferir que o tipo de `v`
é `Vec <i32>`, e a anotação de tipo não é necessária. Vejamos a seguir como
modificar um vetor.

### Updating a Vector
### Modificando um Vetor

To create a vector then add elements to it, we can use the `push` method:

Para criar um vetor e adicionar elementos a ele, podemos usar o método `push`:

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

As with any variable as we discussed in Chapter 3, if we want to be able to
change its value, we need to make it mutable with the `mut` keyword. The
numbers we place inside are all of type `i32`, and Rust infers this from the
data, so we don’t need the `Vec<i32>` annotation.

Como qualquer variável que discutimos no Capítulo 3, se quisermos poder
alterar seu valor, precisamos fazê-lo mutável com a palavra-chave `mut`. Os 
números que inserimos são todos do tipo `i32`, e Rust infere isso dos
dados, por isso não precisamos da anotação `Vec <i32>`.

### Dropping a Vector Drops its Elements
### Descartando um Vetor descarta seus Elementos

Like any other `struct`, a vector will be freed when it goes out of scope:

Como qualquer outro `struct`, um vetor será liberado quando ele estiver fora do escopo:

```rust
{
    let v = vec![1, 2, 3, 4];

    // do stuff with v

} // <- v goes out of scope and is freed here
```


```rust
{
    let v = vec![1, 2, 3, 4];

    // use as informações em v

} // <- v sai do escopo e é liberado aqui
```


When the vector gets dropped, all of its contents will also be dropped, meaning
those integers it holds will be cleaned up. This may seem like a
straightforward point, but can get a little more complicated once we start to
introduce references to the elements of the vector. Let’s tackle that next!

Quando o vetor é descartado, todos os seus conteúdos também será descartado, o que significa
esses inteiros que ele contém serão limpos. Isso pode parecer um
ponto direto, mas pode ficar um pouco mais complicado quando começamos a
introduzir referências aos elementos do vetor. Vamos abordar isso em seguida!

### Reading Elements of Vectors
### Lendo Elementos do Vetor

Now that you know how to create, update, and destroy vectors, knowing how to
read their contents is a good next step. There are two ways to reference a
value stored in a vector. In the examples, we’ve annotated the types of the
values that are returned from these functions for extra clarity.

Agora que você sabe como criar, atualizar e destruir vetores, o próximo
passo é saber ler o seu conteúdo é um bom passo seguinte. Existem duas maneiras de fazer referência a
valor armazenado em um vetor. Nos exemplos, anotamos os tipos de
valores que são retornados dessas funções para maior clareza.

This example shows both methods of accessing a value in a vector either with
indexing syntax or the `get` method:

Este exemplo mostra os dois métodos de acesso a um valor em um vetor com
sintaxe de indexação ou o método `get`:

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
let third: Option<&i32> = v.get(2);
```

There are a few things to note here. First, that we use the index value of `2`
to get the third element: vectors are indexed by number, starting at zero.
Second, the two different ways to get the third element are: using `&` and
`[]`, which gives us a reference, or using the `get` method with the index
passed as an argument, which gives us an `Option<&T>`.

Há algumas coisas a serem observadas aqui. Primeiro, que usamos o valor do índice de `2`
para obter o terceiro elemento: os vetores são indexados por número, começando em zero.
Em segundo lugar, as duas maneiras diferentes de obter o terceiro elemento são: usando `&` e
`[]`, que nos dá uma referência, ou usando o método `get` com o índice
passado como um argumento, o que nos dá uma `Opção <& T>`.

The reason Rust has two ways to reference an element is so that you can choose
how the program behaves when you try to use an index value that the vector
doesn’t have an element for. As an example, what should a program do if it has
a vector that holds five elements then tries to access an element at index 100
like this:

A razão pela qual Rust tem duas maneiras de fazer referência a um elemento é para que você possa escolher
como o programa se comporta quando você tenta usar um valor de índice que o vetor
não tem um elemento para ele. Por exemplo, o que um programa deve fazer se tiver
um vetor que contém cinco elementos, então tenta acessar um elemento no índice 100
dessa maneira:

```rust,should_panic
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

When you run this, you will find that with the first `[]` method, Rust will
cause a `panic!` when a non-existent element is referenced. This method would
be preferable if you want your program to consider an attempt to access an
element past the end of the vector to be a fatal error that should crash the
program.

Quando você executa isso, você encontrará que com o primeiro método `[]`, Rust irá
causar um `panic!` quando um elemento inexistente é referenciado. Este método seria
preferível se você quiser que seu programa considere uma tentativa de acessar um
elemento, passado o fim do vetor, para ser um erro fatal que, deve travar o
programa.

When the `get` method is passed an index that is outside the array, it will
return `None` without panicking. You would use this if accessing an element
beyond the range of the vector will happen occasionally under normal
circumstances. Your code can then have logic to handle having either
`Some(&element)` or `None`, as we discussed in Chapter 6. For example, the
index could be coming from a person entering a number. If they accidentally
enter a number that’s too large and your program gets a `None` value, you could
tell the user how many items are in the current `Vec` and give them another
chance to enter a valid value. That would be more user-friendly than crashing
the program for a typo!

Quando é passado um índice que está fora da matriz para o método `get`, ele 
retorna `None` sem entrar em pânico. Você usaria isso se acessasse um elemento
além do alcance do vetor ocorrerá ocasionalmente sob 
circunstâncias normais. Seu código pode então ter lógica para lidar tanto com
`Some(&element)` ou `None`, como discutimos no Capítulo 6. Por exemplo, o
O índice pode ser proveniente de uma pessoa que digite um número. Se eles acidentalmente
insira um número que é muito grande e seu programa recebe um valor `None`, você poderia
dizer ao usuário quantos itens estão no atual `Vec` e dar uma nova
chance de inserir um valor válido. Isso seria mais amigável do que quebrar o
programa por um erro de digitação!

#### Invalid References
#### Referências Inválidas

Once the program has a valid reference, the borrow checker will enforce the
ownership and borrowing rules covered in Chapter 4 to ensure this reference and
any other references to the contents of the vector stay valid. Recall the rule
that says we can’t have mutable and immutable references in the same scope.
That rule applies in this example, where we hold an immutable reference to the
first element in a vector and try to add an element to the end:

Uma vez que o programa tenha uma referência válida, o verificador de empréstimo (borrow checker) faz valer
as regras de propriedade e empréstimo abrangidas no Capítulo 4 para garantir essa referência e
quaisquer outras referências aos conteúdos do vetor permaneçam válidas. Lembre-se da regra
que diz que não podemos ter referências mutáveis e imutáveis no mesmo escopo.
Essa regra se aplica neste exemplo, onde mantemos uma referência imutável ao
primeiro elemento em um vetor e tentamos adicionar um elemento ao final:



```rust,ignore
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);
```

Compiling this will give us this error:

Compilar isso nos dará esse erro:

```text
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as
immutable
  |
4 | let first = &v[0];
  |              - immutable borrow occurs here
5 |
6 | v.push(6);
  | ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

This code might look like it should work: why should a reference to the first
element care about what changes about the end of the vector? The reason why
this code isn’t allowed is due to the way vectors work. Adding a new element
onto the end of the vector might require allocating new memory and copying the
old elements over to the new space, in the circumstance that there isn’t enough
room to put all the elements next to each other where the vector was. In that
case, the reference to the first element would be pointing to deallocated
memory. The borrowing rules prevent programs from ending up in that situation.

Este código pode parecer que deveria funcionar: por que uma referência ao primeiro
elemento deveria se preocupar com o que muda sobre o final do vetor? A razão porque
este código não é permitido é devido à forma como os vetores funcionam. Adicionando um novo elemento
no final do vetor pode exigir a atribuição de nova alocação de memória e copiar os
elementos antigos para o novo espaço, na circunstância de não haver espaço suficiente
para colocar todos os elementos próximos um do outro onde o vetor estava. Nesse
caso, a referência ao primeiro elemento apontaria para memória não alocada.
As regras de empréstimo impedem que os programas acabem nessa situação.

> Note: For more on this, see The Nomicon at
*https://doc.rust-lang.org/stable/nomicon/vec.html*.


> Nota: para mais informações, veja o Nomicon em
*https://doc.rust-lang.org/stable/nomicon/vec.html*.


### Using an Enum to Store Multiple Types
### Usando um Enum para Armazenar Vários Tipos

At the beginning of this chapter, we said that vectors can only store values
that are all the same type. This can be inconvenient; there are definitely use
cases for needing to store a list of things of different types. Luckily, the
variants of an enum are all defined under the same enum type, so when we need
to store elements of a different type in a vector, we can define and use an
enum!

No início deste capítulo, dissemos que os vetores só podem armazenar valores
que são todos do mesmo tipo. Isso pode ser inconveniente; definitivamente há casos 
de uso para a necessidade de armazenar uma lista de coisas de diferentes tipos. Felizmente, o
as variantes de um enum são todas definidas sob o mesmo tipo de enum, então, quando precisamos
armazenar elementos de um tipo diferente em um vetor, podemos definir e usar um
enum!

For example, let’s say we want to get values from a row in a spreadsheet, where
some of the columns in the row contain integers, some floating point numbers,
and some strings. We can define an enum whose variants will hold the different
value types, and then all of the enum variants will be considered the same
type, that of the enum. Then we can create a vector that holds that enum and
so, ultimately, holds different types:

Por exemplo, digamos que queremos obter valores de uma linha em uma planilha, onde
algumas das colunas da linha contêm números inteiros, alguns números de ponto flutuante,
e algumas strings. Podemos definir um enum cujas variantes guardarão os diferentes
tipos de valor, e então todas as variantes de enum serão consideradas do mesmos
tipo, o do enum. Então, podemos criar um vetor que contenha esse enum e
então, em última instância, possui diferentes tipos:

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

<span class="caption">Listing 8-1: Defining an enum to be able to hold
different types of data in a vector</span>

<span class="caption">Listagem 8-1: Definindo um enum para poder guardar
diferentes tipos de dados em um vetor</span>


The reason Rust needs to know exactly what types will be in the vector at
compile time is so that it knows exactly how much memory on the heap will be
needed to store each element. A secondary advantage to this is that we can be
explicit about what types are allowed in this vector. If Rust allowed a vector
to hold any type, there would be a chance that one or more of the types would
cause errors with the operations performed on the elements of the vector. Using
an enum plus a `match` means that Rust will ensure at compile time that we
always handle every possible case, as we discussed in Chapter 6.

A razão pela qual Rust precisa saber exatamente quais tipos estarão no vetor em
tempo de compilação é para que ele saiba exatamente a quantidade de memória no heap que será
necessária para armazenar cada elemento. Uma vantagem secundária para isso é que podemos ser
explícitos sobre quais tipos são permitidos neste vetor. Se Rust permitisse um vetor
guardar qualquer tipo, haveria uma chance de que um ou mais dos tipos
causar erros com as operações realizadas nos elementos do vetor. Usando
um enum mais um `match` significa que a Rust garantirá no tempo de compilação que nós
sempre lidaremos com todos os casos possíveis, como discutimos no Capítulo 6.

If you don’t know at the time that you’re writing a program the exhaustive set
of types the program will get at runtime to store in a vector, the enum
technique won’t work. Instead, you can use a trait object, which we’ll cover in
Chapter 17.

Se você não sabe no momento em que você está escrevendo um programa, o conjunto exaustivo
dos tipos que o programa irá precisar no tempo de execução para armazenar em um vetor, a técnica de usar
o enum não funcionará. Em vez disso, você pode usar um objeto trait, que abordaremos no
Capítulo 17.

Now that we’ve gone over some of the most common ways to use vectors, be sure
to take a look at the API documentation for all of the many useful methods
defined on `Vec` by the standard library. For example, in addition to `push`
there’s a `pop` method that will remove and return the last element. Let’s move
on to the next collection type: `String`!

Agora que examinamos algumas das maneiras mais comuns de usar vetores, certifique-se
para dar uma olhada na documentação da API para todos os muitos métodos úteis
definidos no `Vec` pela biblioteca padrão. Por exemplo, além de `push`
existe um método `pop` que irá remover e retornar o último elemento. Vamos mover
para o próximo tipo de coleção: `String`!
