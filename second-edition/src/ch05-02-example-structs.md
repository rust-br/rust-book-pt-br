## An Example Program Using Structs
## Um Exemplode um Programa que usa Structs

To understand when we might want to use structs, let’s write a program that
calculates the area of a rectangle. We’ll start with single variables, and then
refactor the program until we’re using structs instead.

Para entender quando podemos querer usar structs, vamos escrever um programa 
que calcula a área de um retângulo. Vamos começar com as variáveis individuais
e em seguida, refazer o programa até usar structs em vez das variáveis.

Let’s make a new binary project with Cargo called *rectangles* that will take
the length and width of a rectangle specified in pixels and will calculate the
area of the rectangle. Listing 5-8 shows a short program with one way of doing
just that in our project’s *src/main.rs*:

Vamos fazer um novo projeto binário com Cargo, chamado *retângulos* que terá
o comprimento e a largura do retângulo especificados em pixels e irá calcular a 
área do retângulo. A Lista 5-8 mostra um programa curto com uma maneira de 
fazer isso no nosso projeto *src/main.rs*:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let length1 = 50;
    let width1 = 30;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(length1, width1)
    );
}

fn area(length: u32, width: u32) -> u32 {
    length * width
}
```

<span class="caption">Listing 5-8: Calculating the area of a rectangle
specified by its length and width in separate variables</span>

<span class="caption">Lista 5-8: Calcular a área de um retângulo 
especificado pelo seu comprimento e largura em variáveis separadas</span>

Now, run this program using `cargo run`:

Agora, executamos este programa usando `cargo run`:

```text
The area of the rectangle is 1500 square pixels.
```

### Refactoring with Tuples
### Refazendo com Tuples

Even though Listing 5-8 works and figures out the area of the rectangle by
calling the `area` function with each dimension, we can do better. The length
and the width are related to each other because together they describe one
rectangle.

Embora a lista 5-8 funcione e descubra a área do retângulo chamando a função
`area` com cada dimensão, nós podemos fazer melhor. O comprimento e a largura 
estão relacionados uns aos outros porque juntos eles descrevem um retângulo.

The issue with this method is evident in the signature of `area`:

O problema com este método é evidente na assinatura de `area`:

```rust,ignore
fn area(length: u32, width: u32) -> u32 {
```

The `area` function is supposed to calculate the area of one rectangle, but the
function we wrote has two parameters. The parameters are related, but that’s
not expressed anywhere in our program. It would be more readable and more
manageable to group length and width together. We’ve already discussed one way
we might do that in the Grouping Values into Tuples section of Chapter 3 on
page XX: by using tuples. Listing 5-9 shows another version of our program that
uses tuples:

A função `area` supostamente calcula a área de um retângulo, mas a função
que escrevemos tem dois parâmetros. Os parâmetros estão relacionados, mas isso
não é indicado em qualquer lugar no nosso programa. Seria mais legível e 
mais gerenciável agrupar comprimento e largura. Já discutimos uma maneira de
podermos fazer isso no Agrupamento de Valores em Tuples, na seção do capítulo 3
na página XX: através do uso de tuples. Lista 5-9 mostra outra versão do 
nosso programa que usa tuples:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let rect1 = (50, 30);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

<span class="caption">Listing 5-8: Specifying the length and width of the
rectangle with a tuple</span>

<span class="caption">Lista 5-8: Especificando o comprimento e a largura de um 
retangulo através de uma Tuple.</span>

In one way, this program is better. Tuples let us add a bit of structure, and
we’re now passing just one argument. But in another way this version is less
clear: tuples don’t name their elements, so our calculation has become more
confusing because we have to index into the parts of the tuple.

Em uma maneira, este programa é melhor. Tuples dexa-nos adicionar um pouco 
de estrutura, e agora estamos passando apenas um argumento. De outra forma, 
mas esta versão é menos claro: tuplas não nome seus elementos, de modo que 
nosso cálculo se tornou mais confusa porque temos de índice para as peças da 
tupla.

It doesn’t matter if we mix up length and width for the area calculation, but
if we want to draw the rectangle on the screen, it would matter! We would have
to keep in mind that `length` is the tuple index `0` and `width` is the tuple
index `1`. If someone else worked on this code, they would have to figure this
out and keep it in mind as well. It would be easy to forget or mix up these
values and cause errors, because we haven’t conveyed the meaning of our data in
our code.

Não importa, para o cálculo da área, trocar-se comprimento e largura, mas 
se queremos desenhar o retângulo na tela, já importa! Temos de ter em 
mente que `comprimento` é a tuple índice `0` e `largura` é o tuple índice `1`.
Se alguém trabalhar com este código, terá de descobrir isso e mantê-lo em 
mente. Seria fácil esquecer ou misturar estes valores e causar erros, porque
não se transmitiu o significado dos nossos dados no nosso código.

### Refactoring with Structs: Adding More Meaning
### Reprogramação com Structs: Adicionando mais significado

We use structs to add meaning by labeling the data. We can transform the tuple
we’re using into a data type with a name for the whole as well as names for the
parts, as shown in Listing 5-10:

Usamos structs para dar significado aos dados usando rotulos. Podemos 
transformar a tuple que estamos usando, em um tipo de dados, com um nome
para o conjunto, bem como os nomes das partes, como mostra a Lista 5-10:

<span class="filename">Filename: src/main.rs</span>

```rust
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.length * rectangle.width
}
```

<span class="caption">Listing 5-10: Defining a `Rectangle` struct</span>
<span class="caption">Lista 5-10: Definindo um `Rectangulo` struct</span>

Here we’ve defined a struct and named it `Rectangle`. Inside the `{}` we
defined the fields as `length` and `width`, both of which have type `u32`. Then
in `main` we create a particular instance of a `Rectangle` that has a length of
50 and a width of 30.

Aqui temos definido um struct denominado `Rectangle`. Dentro do `{}` 
definimos os campos como `comprimento` e `largura`, ambas do tipo `u32`. 
Em `main`, criamos uma instância específica de um `Rectangle` que tem um 
comprimento de 50 e largura de 30.

Our `area` function is now defined with one parameter, which we’ve named
`rectangle`, whose type is an immutable borrow of a struct `Rectangle`
instance. As mentioned in Chapter 4, we want to borrow the struct rather than
take ownership of it. This way, `main` retains its ownership and can continue
using `rect1`, which is the reason we use the `&` in the function signature and
where we call the function.

A nossa função `área` agora é definida com um parâmetro, que chamamos 
`rectangle`, cujo tipo é um empréstimo de uma instância da struct imutável
`Rectangle`. Como mencionado no capítulo 4, queremos usar a struct, em vez de
tomar posse dela. Desta forma, `main` mantém-se a sua proprietaria e pode 
continuar a usar o `rect1`, que é a razão para usar o '&' na assinatura 
da função e onde chamamos a função.

The `area` function accesses the `length` and `width` fields of the `Rectangle`
instance. Our function signature for `area` now indicates exactly what we mean:
calculate the area of a `Rectangle` using its `length` and `width` fields. This
conveys that the length and width are related to each other, and gives
descriptive names to the values rather than using the tuple index values of `0`
and `1`—a win for clarity.

A função `área` acessa os campos `comprimento` e `largura` da instância 
`Rectangle`. Nossa função assinatura para `área` agora indica exatamente o que 
queremos dizer: calcular a área de um retângulo "' usando o seu "Comprimento"
e "Largura" campos. Transmite o que o comprimento e a largura são relacionados
uns aos outros, e dá nomes descritivos para os valores em vez de usar a tupla 
de valores de índice de `0` e `1`-uma vitória para maior clareza.

### Adding Useful Functionality with Derived Traits
### Adicionando Funcionalidade Util com Traits Derivadas

It would be helpful to be able to print out an instance of the `Rectangle`
while we’re debugging our program in order to see the values for all its
fields. Listing 5-11 uses the `println!` macro as we have been in earlier
chapters:

Seria útil para ser capaz de imprimir uma instância do `Rectangle` enquanto
estamos depurando o nosso programa, a fim de consultar os valores para todos
os seus campos. Lista 5-11 usa a macro 'Println!' como temos usado nos 
capítulos anteriores:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!("rect1 is {}", rect1);
}
```

<span class="caption">Listing 5-11: Attempting to print a `Rectangle`
instance</span>

<span class="caption">Lista 5-11: Tentativa de impressão de uma instância
de `Rectangle`</span>

When we run this code, we get an error with this core message:

Quando executamos este código, obtemos um erro com esta mensagem interna:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Display` is not satisfied
```

The `println!` macro can do many kinds of formatting, and by default, `{}`
tells `println!` to use formatting known as `Display`: output intended for
direct end user consumption. The primitive types we’ve seen so far implement
`Display` by default, because there’s only one way you’d want to show a `1` or
any other primitive type to a user. But with structs, the way `println!` should
format the output is less clear because there are more display possibilities:
do you want commas or not? Do you want to print the curly braces? Should all
the fields be shown? Due to this ambiguity, Rust doesn’t try to guess what we
want and structs don’t have a provided implementation of `Display`.

A macro 'Println!' pode fazer muitos tipos de formatação, e por padrão, 
`{}` diz a `println!`, para utilizar a formatação conhecida como `Display`:
saída destinada para consumo do utilizador final. Os tipos primitivos vimos 
até agora implementar `Display` por padrão, porque só há uma maneira que você 
deseja mostrar um `1` ou qualquer outro tipo primitivo para um usuário. Mas com
Structs, a forma `Println!` deve formatar a saída é menos clara, pois existem 
mais possibilidades de exibição: você quer vírgulas ou não? Deseja imprimir o 
chaves? Todos os campos devem ser mostrado? Devido a esta ambiguidade, Rust 
não tente adivinhar o que queremos e structs não tem fornecido uma 
implementação de `Display`.

If we continue reading the errors, we’ll find this helpful note:

Se continuarmos a ler os erros, encontramos esta nota explicativa:

```text
note: `Rectangle` cannot be formatted with the default formatter; try using
`:?` instead if you are using a format string
```
Avisa que `Rectangle` não pode ser formatado com o formato por defeito;
Devemos tentar usar `:?` ao invés, se estivermos a usar um formato de string

Let’s try it! The `println!` macro call will now look like `println!("rect1 is`
`{:?}", rect1);`. Putting the specifier `:?` inside the `{}` tells `println!` we
want to use an output format called `Debug`. `Debug` is a trait that enables us
to print out our struct in a way that is useful for developers so we can see
its value while we’re debugging our code.

Vamos tentar! A chamada da macro `Println!` agora vai ficar como 
`println!("rect1 is {:?}", rect1);`. Colocando o especificador `:?` dentro 
do `{}` diz à `println!` que nós queremos usar um formato de saída chamado
`Debug`. `Debug` é uma trait (característica) que nos permite imprimir nossos struct
em uma maneira que é útil para os desenvolvedores para que possamos ver o seu 
valor enquanto estamos a depurar do nosso código.

Run the code with this change. Drat! We still get an error:
Executamos o código com esta mudança. Pô! Nós ainda obtemos um erro:

```text
error: the trait bound `Rectangle: std::fmt::Debug` is not satisfied
```

But again, the compiler gives us a helpful note:
Mas, novamente, o compilador dá-nos uma nota útil:

```text
note: `Rectangle` cannot be formatted using `:?`; if it is defined in your
crate, add `#[derive(Debug)]` or manually implement it
```
`nota: 'Rectangle' não pode ser formatado usando ':?'; se estiver definido no`
`crate, adicionamos '#[derive(Debug)]' ou adicionamos manualmente.`


Rust *does* include functionality to print out debugging information, but we
have to explicitly opt-in to make that functionality available for our struct.
To do that, we add the annotation `#[derive(Debug)]` just before the struct
definition, as shown in Listing 5-12:

Rust *não* inclui funcionalidades para imprimir informações de depuração,
mas temos de inseri-la explicitamente para tornar essa funcionalidade 
disponível para nossa struct. Para isso, adicionamos a anotação 
`#[derivam(Debug)]` pouco antes da definição da struct, como mostrado 
na Lista 5-12:

<span class="filename">Filename: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    length: u32,
    width: u32,
}

fn main() {
    let rect1 = Rectangle { length: 50, width: 30 };

    println!("rect1 is {:?}", rect1);
}
```

<span class="caption">Listing 5-12: Adding the annotation to derive the `Debug`
trait and printing the `Rectangle` instance using debug formatting</span>

<span class="caption">Lista 5-12: Adicionando a anotação para derivar `debug`
e imprimir a instância `Rectangle`usando a formatação debug</span>

Now when we run the program, we won’t get any errors and we’ll see the
following output:

Agora, quando executamos o programa, não teremos quaisquer erros e vamos ver
a seguinte informação:

```text
rect1 is Rectangle { length: 50, width: 30 }
```

Nice! It’s not the prettiest output, but it shows the values of all the fields
for this instance, which would definitely help during debugging. When we have
larger structs, it’s useful to have output that’s a bit easier to read; in
those cases, we can use `{:#?}` instead of `{:?}` in the `println!` string.
When we use the `{:#?}` style in the example, the output will look like this:

Boa! Não é o mais bonito, mas mostra os valores de todos os campos para essa 
instância, que irá ajudar durante a depuração. Quando temos structs maiores, 
é útil ter a informação um pouco mais fácil de ler; nesses casos, podemos usar
`{:#?}` ao invés de `{:?}` na frase `println!`. Quando utilizamos o `{:#?}` no
exemplo, vamos ver a informação como esta:

```text
rect1 is Rectangle {
    length: 50,
    width: 30
}
```

Rust has provided a number of traits for us to use with the `derive` annotation
that can add useful behavior to our custom types. Those traits and their
behaviors are listed in Appendix C. We’ll cover how to implement these traits
with custom behavior as well as how to create your own traits in Chapter 10.

Rust forneceu um número de traits para usarmos com a notação `derive` que pode 
adicionar um comportamento útil aos nossos tipos personalizados. Esses 
traits e seus comportamentos estão listadas no Apêndice C. Abordaremos como 
implementar estas características com comportamento personalizado, bem como 
a forma de criar as suas próprias características no Capítulo 10.

Our `area` function is very specific: it only computes the area of rectangles.
It would be helpful to tie this behavior more closely to our `Rectangle`
struct, because it won’t work with any other type. Let’s look at how we can
continue to refactor this code by turning the `area` function into an `area`
*method* defined on our `Rectangle` type.

A nossa função `area` é muito específica: ela apenas calcula a área de 
retângulos. Seria útil amarrar este comportamento à nossa struc `Rectangle`,
porque não vai funcionar com qualquer outro tipo. Vejamos como podemos 
continuar a refazer esse código rodando a `area` em um *método* `area` definido
no nosso `Rectangle`.