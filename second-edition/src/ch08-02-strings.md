## Strings

We’ve already talked about strings a bunch in Chapter 4, but let’s take a more
in-depth look at them now. Strings are an area that new Rustaceans commonly get
stuck on. This is due to a combination of three things: Rust’s propensity for
making sure to expose possible errors, strings being a more complicated data
structure than many programmers give them credit for, and UTF-8. These things
combine in a way that can seem difficult when coming from other languages.

Nós já conversamos sobre as strings no capítulo 4, mas vamos dar uma olhada mais
em profundidade agora. As strings são uma área que os novos Rustáceos geralmente tem maior
dificuldade. Isto é devido a uma combinação de três coisas: a propensão de Rust de
certificar-se de expor possíveis erros, as strings são estruturas de dados mais complicadas
que muitos programadores lhes dão crédito, e UTF-8. Essas coisas
combina de tal forma que parecem difícil quando se vem de outras linguagens.

The reason strings are in the collections chapter is that strings are
implemented as a collection of bytes plus some methods to provide useful
functionality when those bytes are interpreted as text. In this section, we’ll
talk about the operations on `String` that every collection type has, like
creating, updating, and reading. We’ll also discuss the ways in which `String`
is different than the other collections, namely how indexing into a `String` is
complicated by the differences between how people and computers interpret
`String` data.

A razão pela qual as strings estão no capítulo de coleções é que as strings são
implementadas como uma coleção de bytes mais alguns métodos para fornecer informações úteis e
funcionalidade quando esses bytes são interpretados como texto. Nesta seção, iremos
falar sobre as operações em `String` que todo tipo de coleção tem, como
criar, atualizar e ler. Também discutiremos as formas em que `String`
é diferente das outras coleções, a saber, como a indexação em um `String` é
complicada pelas diferenças entre como as pessoas e os computadores interpretam
dados `String`.

### What is a String?
### O que é String?

Before we can dig into those aspects, we need to talk about what exactly we
mean by the term *string*. Rust actually only has one string type in the core
language itself: `str`, the string slice, which is usually seen in its borrowed
form, `&str`. We talked about *string slices* in Chapter 4: these are a
reference to some UTF-8 encoded string data stored elsewhere. String literals,
for example, are stored in the binary output of the program, and are therefore
string slices.

Antes de podermos explorar esses aspectos, precisamos falar sobre o que exatamente
significa o termo *string*. Rust realmente só tem um tipo de string no núcleo
da própria linguagem: `str`, a fatia de string, que geralmente é vista na forma emprestada
, `&str`. Nós falamos sobre *fatias de strings* no Capítulo 4: estas são uma
referência a alguns dados de string codificados em UTF-8 armazenados em outro lugar. Literais de strings,
por exemplo, são armazenados na saída binária do programa e, portanto, são
fatias de string.

The type called `String` is provided in Rust’s standard library rather than
coded into the core language, and is a growable, mutable, owned, UTF-8 encoded
string type. When Rustaceans talk about “strings” in Rust, they usually mean
both the `String` and the string slice `&str` types, not just one of those.
This section is largely about `String`, but both these types are used heavily
in Rust’s standard library. Both `String` and string slices are UTF-8 encoded.

O tipo chamado `String` é fornecido na biblioteca padrão do Rust, em vez de
codificado no núleo da linguagem, e é um extensível, mutável, `owned`, tipo string
codificado UTF-8. Quando Rustáceos falam sobre “strings” em Rust, geralmente significa
tanto os tipos `String` quanto os tipos de string`&str`, normalmente ambos.
Esta seção, é em grande parte sobre `String`, mas ambos esses tipos são usados em grande parte
na biblioteca padrão da Rust. Tanto o `String` como as fatias de string são codificadas em UTF-8.

Rust’s standard library also includes a number of other string types, such as
`OsString`, `OsStr`, `CString`, and `CStr`. Library crates may provide even
more options for storing string data. Similar to the `*String`/`*Str` naming,
they often provide an owned and borrowed variant, just like `String`/`&str`.
These string types may store different encodings or be represented in memory in
a different way, for example. We won’t be talking about these other string
types in this chapter; see their API documentation for more about how to use
them and when each is appropriate.

### Creating a New String
### Criando uma Nova String

Many of the same operations available with `Vec` are available with `String` as
well, starting with the `new` function to create a string, like so:

Muitas das mesmas operações disponíveis com `Vec` também estão disponíveis em `String`,
começando com a função `new` para criar uma string, assim:

```rust
let mut s = String::new();
```

This creates a new empty string called `s` that we can then load data into.

Isso cria uma nova string vazia chamada `s` na qual podemos carregar dados.

Often, we’ll have some initial data that we’d like to start the string off
with. For that, we use the `to_string` method, which is available on any type
that implements the `Display` trait, which string literals do:

Muitas vezes, teremos alguns dados iniciais que gostaríamos de já colocar na string. 
Para isso, usamos o método `to_string`, que está disponível em qualquer tipo
que implementa a trait `Display`, como as strings literais:

```rust
let data = "initial contents";

let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();
```

```rust
let data = "initial contents";

let s = data.to_string();

// o método também funciona em literais diretamente
let s = "initial contents".to_string();
```


This creates a string containing `initial contents`.

Isso cria uma string contendo `initial contents`.

We can also use the function `String::from` to create a `String` from a string
literal. This is equivalent to using `to_string`:

Também podemos usar a função `String :: from` para criar uma` String` de uma string
literal. Isso equivale a usar `to_string`:

```rust
let s = String::from("initial contents");
```

Because strings are used for so many things, there are many different generic
APIs that can be used for strings, so there are a lot of options. Some of them
can feel redundant, but they all have their place! In this case, `String::from`
and `.to_string` end up doing the exact same thing, so which you choose is a
matter of style.

Como as strings são usadas para tantas coisas, existem várias APIs genéricas diferentes
que podem ser usadas para strings, então há muitas opções. Algumas delas
podem parecer redundantes, mas todas têm seu lugar! Nesse caso, `String :: from`
e `.to_string` acabam fazendo exatamente o mesmo, então a que você escolher é uma
questão de estilo.

Remember that strings are UTF-8 encoded, so we can include any properly encoded
data in them:

Lembre-se de que as string são codificadas em UTF-8, para que possamos incluir qualquer dados apropriadamente codificados
neles:

```rust
let hello = "السلام عليكم";
let hello = "Dobrý den";
let hello = "Hello";
let hello = "שָׁלוֹם";
let hello = "नमस्ते";
let hello = "こんにちは";
let hello = "안녕하세요";
let hello = "你好";
let hello = "Olá";
let hello = "Здравствуйте";
let hello = "Hola";
```

### Updating a String
### Atualizando uma String

A `String` can grow in size and its contents can change just like the contents
of a `Vec`, by pushing more data into it. In addition, `String` has
concatenation operations implemented with the `+` operator for convenience.

Uma `String` pode crescer em tamanho e seu conteúdo pode mudar assim como o conteúdo
de um `Vec`, empurrando mais dados para ela. Além disso, `String` tem
operações de concatenação implementadas com o operador `+` por conveniência.

#### Appending to a String with Push
#### Anexando a uma String com Push

We can grow a `String` by using the `push_str` method to append a string slice:

Podemos criar uma `String` usando o método `push_str` para adicionar uma  seqüência de caracteres:

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

`s` will contain “foobar” after these two lines. The `push_str` method takes a
string slice because we don’t necessarily want to take ownership of the
parameter. For example, it would be unfortunate if we weren’t able to use `s2`
after appending its contents to `s1`:

`s` conterá “foobar“ após essas duas linhas. O método `push_str` leva um
fatia de string porque não necessariamente queremos ownership do
parâmetro. Por exemplo, seria lamentável se não pudéssemos usar `s2`
depois de atualizar o seu conteúdo a `s1`:

```rust
let mut s1 = String::from("foo");
let s2 = String::from("bar");
s1.push_str(&s2);
```

The `push` method is defined to have a single character as a parameter and add
it to the `String`:

O método `push` é definido para ter um único caractere como parâmetro e adicionar
à `String`:

```rust
let mut s = String::from("lo");
s.push('l');
```

After this, `s` will contain “lol”.

Após isso, `s` irá conter “lol”.

#### Concatenation with the + Operator or the `format!` Macro
#### Concatenação com o Operador + ou a macro `format!`

Often, we’ll want to combine two existing strings together. One way is to use
the `+` operator like this:

Muitas vezes, queremos combinar duas strings existentes. Uma maneira é usar
o operador `+` dessa forma:

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // Note that s1 has been moved here and can no longer be used
```

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // Note que s1 foi movido aqui e não pode ser mais usado
```

After this code the String `s3` will contain `Hello, world!`. The reason that
`s1` is no longer valid after the addition and the reason that we used a
reference to `s2` has to do with the signature of the method that gets called
when we use the `+` operator. The `+` operator uses the `add` method, whose
signature looks something like this:

Após este código, a String `s3` conterá `Hello, world!`. O motivo que
`s1` não é mais válido após a adição e o motivo que usamos uma
referência a `s2` tem a ver com a assinatura do método que é chamado
quando usamos o operador `+`. O operador `+` usa o método `add`, cuja
assinatura parece algo assim:

```rust,ignore
fn add(self, s: &str) -> String {
```

This isn’t the exact signature that’s in the standard library; there `add` is
defined using generics. Here, we’re looking at the signature of `add` with
concrete types substituted for the generic ones, which is what happens when we
call this method with `String` values. We’ll be discussing generics in
Chapter 10. This signature gives us the clues we need to understand the tricky
bits of the `+` operator.

Esta não é a assinatura exata que está na biblioteca padrão; lá o `add` é
definido usando genéricos. Aqui, estamos olhando a assinatura do `add` com
tipos de concreto substituídos pelos genéricos, o que acontece quando nós
chamamos esse método com valores `String`. Vamos discutir genéricos no
Capítulo 10. Esta assinatura nos dá as pistas que precisamos para entender o complicado
operador `+`.

First of all, `s2` has an `&`, meaning that we are adding a *reference* of the
second string to the first string. This is because of the `s` parameter in the
`add` function: we can only add a `&str` to a `String`, we can’t add two
`String` values together. But wait - the type of `&s2` is `&String`, not
`&str`, as specified in the second parameter to `add`. Why does our example
compile? We are able to use `&s2` in the call to `add` because a `&String`
argument can be *coerced* into a `&str` - when the `add` function is called,
Rust uses something called a *deref coercion*, which you could think of here as
turning `&s2` into `&s2[..]` for use in the `add` function. We’ll discuss deref
coercion in more depth in Chapter 15. Because `add` does not take ownership of
the parameter, `s2` will still be a valid `String` after this operation.

Antes de tudo, `s2` tem um `&`, o que significa que estamos adicionando uma *referência* da
segunda string para a primeira string. Isso é devido ao parâmetro `s` na
função `add`: só podemos adicionar um `&str` à `String`, não podemos adicionar dois
valores `String` juntos. Mas espere - o tipo de `&s2` é `&String`, não
`&str`, conforme especificado no segundo parâmetro para` add`. Por que nosso exemplo
compila? Podemos usar `&s2` na chamada para `add` porque um `&String`
o argumento pode ser *coerced* em um `&str` - quando a função` add` é chamada,
Rust usa algo chamado de *deref coercion*, o que você poderia pensar aqui como
virando `&s2` para`&s2[..]` para uso na função `add`. Vamos discutir deref 
coercion  em maior profundidade no Capítulo 15. Como o `add` não se apropria
o parâmetro `s2` ainda será uma `String` válida após essa operação.

Second, we can see in the signature that `add` takes ownership of `self`,
because `self` does *not* have an `&`. This means `s1` in the above example
will be moved into the `add` call and no longer be valid after that. So while
`let s3 = s1 + &s2;` looks like it will copy both strings and create a new one,
this statement actually takes ownership of `s1`, appends a copy of the contents
of `s2`, then returns ownership of the result. In other words, it looks like
it’s making a lot of copies, but isn’t: the implementation is more efficient
than copying.

Em segundo lugar, podemos ver na assinatura que `add` toma posse de` self`,
porque `self` *não* tem `&`. Isso significa `s1` no exemplo acima
será transferido para a chamada `add` e não será mais válido depois disso. Por enquanto
`let s3 = s1 + &s2;` parece que irá copiar ambas as strings e criar uma nova,
esta declaração realmente adere a `s1`, acrescenta uma cópia do conteúdo
de `s2`, então retorna ownership do resultado. Em outras palavras, parece
estar fazendo muitas cópias, mas não é: a implementação é mais eficiente
do que copiar.

If we need to concatenate multiple strings, the behavior of `+` gets unwieldy:

Se precisarmos concatenar várias strings, o comportamento de `+` fica complicado:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

`s` will be “tic-tac-toe” at this point. With all of the `+` and `"`
characters, it gets hard to see what’s going on. For more complicated string
combining, we can use the `format!` macro:

`s` será “tic-tac-toe” neste momento. Com todos os `+` e `"`, 
fica difícil ver o que está acontecendo. Para strings mais complicadas
, podemos usar o macro `format!`:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

This code will also set `s` to “tic-tac-toe”. The `format!` macro works in the
same way as `println!`, but instead of printing the output to the screen, it
returns a `String` with the contents. This version is much easier to read, and
also does not take ownership of any of its parameters.

Este código também definirá `s` para “tic-tac-toe”. A macro `format!` funciona
do mesmo modo que `println!`, mas em vez de imprimir a saída para a tela, ela
retorna uma `String` com o conteúdo. Esta versão é muito mais fácil de ler, e
também não incide ownership em nenhum dos seus parâmetros.

### Indexing into Strings
### Indexação em Strings

In many other languages, accessing individual characters in a string by
referencing them by index is a valid and common operation. In Rust, however, if
we try to access parts of a `String` using indexing syntax, we’ll get an error.
That is, this code:

Em muitas outras linguagens, acessar caracteres individuais em uma string por
referenciando por índice é uma operação válida e comum. Em Rust, no entanto, se
nós tentamos acessar partes de uma `String` usando sintaxe de indexação, vamos ter um erro.
Ou seja, este código:

```rust,ignore
let s1 = String::from("hello");
let h = s1[0];
```

will result in this error:

resultará neste erro:

```text
error: the trait bound `std::string::String: std::ops::Index<_>` is not
satisfied [--explain E0277]
  |>
  |>     let h = s1[0];
  |>             ^^^^^
note: the type `std::string::String` cannot be indexed by `_`
```

The error and the note tell the story: Rust strings don’t support indexing. So
the follow-up question is, why not? In order to answer that, we have to talk a
bit about how Rust stores strings in memory.

O erro e a nota contam a história: as strings em Rust não suportam a indexação. Assim
a próxima pergunta é, por que não? Para responder a isso, temos que conversar um
pouco sobre como o Rust armazena strings na memória.

#### Internal Representation
#### Representação Interna

A `String` is a wrapper over a `Vec<u8>`. Let’s take a look at some of our
properly-encoded UTF-8 example strings from before. First, this one:

Uma `String` é um invólucro sobre um `Vec <u8>`. Vejamos alguns dos nossos
exemplos UTF-8, codificadas corretamente, de strings vistas anteriormente. Primeiro, este:

```rust
let len = String::from("Hola").len();
```

In this case, `len` will be four, which means the `Vec` storing the string
“Hola” is four bytes long: each of these letters takes one byte when encoded in
UTF-8. What about this example, though?

Neste caso, `len` terá valor de quatro, o que significa que o` Vec` armazena a string
”Hola” tem quatro bytes de comprimento: cada uma dessas letras leva um byte quando codificado em
UTF-8. E o que acontece para esse exemplo?

```rust
let len = String::from("Здравствуйте").len();
```

A person asked how long the string is might say 12. However, Rust’s answer
is 24. This is the number of bytes that it takes to encode “Здравствуйте” in
UTF-8, since each Unicode scalar value takes two bytes of storage. Therefore,
an index into the string’s bytes will not always correlate to a valid Unicode
scalar value.

Uma pessoa que pergunte pelo comprimento da string pode dizer que ela deva ter 12.No entanto, a resposta de Rust
é 24. Este é o número de bytes que é necessário para codificar “Здравствуйте“ em
UTF-8, uma vez que cada valor escalar Unicode leva dois bytes de armazenamento. Assim sendo,
um índice nos bytes da string nem sempre se correlaciona com um  valor escalar Unicode válido.

To demonstrate, consider this invalid Rust code:

Para demonstrar, considere este código inválido do Rust:

```rust,ignore
let hello = "Здравствуйте";
let answer = &hello[0];
```

What should the value of `answer` be? Should it be `З`, the first letter? When
encoded in UTF-8, the first byte of `З` is `208`, and the second is `151`, so
`answer` should in fact be `208`, but `208` is not a valid character on its
own. Returning `208` is likely not what a person would want if they asked for
the first letter of this string, but that’s the only data that Rust has at byte
index 0. Returning the byte value is probably not what people want, even with
only Latin letters: `&"hello"[0]` would return `104`, not `h`. To avoid
returning an unexpected value and causing bugs that might not be discovered
immediately, Rust chooses to not compile this code at all and prevent
misunderstandings earlier.

Qual deve ser o valor da `answer`? Seria `З`, a primeira letra? Quando
codificado em UTF-8, o primeiro byte de `З` é `208`, e o segundo é `151`, então
a `answer` deve, na verdade, ser `208`, mas `208` não é um caractere válido em
si. Retornar `208` provavelmente não é o que uma pessoa gostaria se eles pedissem
a primeira letra desta string, mas esse é o único dado que Rust tem no byte
de índice 0. O retorno do valor do byte provavelmente não é o que as pessoas querem, mesmo com
caracteres contendo acentuação: `&"hello"[0]` retornaria `104`, não` h`. Para evitar o
retornando um valor inesperado e causando erros que podem não ser descobertos
imediatamente, Rust escolhe não compilar este código e previne
mal-entendidos anteriormente.

#### Bytes and Scalar Values and Grapheme Clusters! Oh my!
#### Bytes e Valores Escalares e Clusters de Grafemas! Nossa!

This leads to another point about UTF-8: there are really three relevant ways
to look at strings, from Rust’s perspective: as bytes, scalar values, and
grapheme clusters (the closest thing to what people would call *letters*).

Isso leva a outro ponto sobre UTF-8: existem realmente três maneiras relevantes
de olhar para as strings, da perspectiva do Rust: como bytes, valores escalares e
clusters de grafemas (a coisa mais próxima do que as pessoas chamariam *letras*).

If we look at the Hindi word “नमस्ते” written in the Devanagari script, it is
ultimately stored as a `Vec` of `u8` values that looks like this:

Se olharmos para a palavra Hindi “नमस्ते” escrita na escrita Devanagari, é
em última instância, armazenada como um `Vec` de valores `u8` que se parece com isto:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

That’s 18 bytes, and is how computers ultimately store this data. If we look at
them as Unicode scalar values, which are what Rust’s `char` type is, those
bytes look like this:

Isso é 18 bytes, e é como os computadores de fato armazenam esses dados. Se olharmos para
eles como valores escalares Unicode, que são o tipo `char` de Rust, aqueles
bytes se parecem com isto:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

There are six `char` values here, but the fourth and sixth are not letters,
they’re diacritics that don’t make sense on their own. Finally, if we look at
them as grapheme clusters, we’d get what a person would call the four letters
that make up this word:

Existem seis valores `char` aqui, mas o quarto e o sexto não são letras,
Eles são diacríticos que não fazem sentido por conta própria. Finalmente, se olharmos para
eles como clusters de grafemas, teríamos o que uma pessoa chamaria as quatro letras
que compõem esta palavra:

```text
["न", "म", "स्", "ते"]
```

Rust provides different ways of interpreting the raw string data that computers
store so that each program can choose the interpretation it needs, no matter
what human language the data is in.

Rust fornece diferentes maneiras de interpretar os dados de uma string bruta que os computadores
armazenem para que cada programa possa escolher a interpretação que necessite, não importa
em que idioma humano os dados estão.

A final reason Rust does not allow you to index into a `String` to get a
character is that indexing operations are expected to always take constant time
(O(1)). It isn’t possible to guarantee that performance with a `String`,
though, since Rust would have to walk through the contents from the beginning
to the index to determine how many valid characters there were.

Uma razão final do Rust não permitir que você indexe uma `String` para obter um
caracter é que as operações de indexação sempre esperam um tempo constante
(O(1)). Não é possível garantir que o desempenho com uma `String`,
entretanto, já que o Rust teria que percorrer todo o conteúdo desde o início
até o índice para determinar quantos caracteres válidos havia.

### Slicing Strings
### Fatiando Strings

Because it’s not clear what the return type of string indexing should be, and
it is often a bad idea to index into a string, Rust dissuades you from doing so
by asking you to be more specific if you really need it. The way you can be
more specific than indexing using `[]` with a single number is using `[]` with
a range to create a string slice containing particular bytes:

Porque não está claro qual seria o tipo de retorno da indexação de string, e
muitas vezes é uma má idéia indexar uma string, Rust dissuade-o de fazê-lo
pedindo que você seja mais específico se você realmente precisar disso. Do jeito que você pode ser
mais específico que a indexação usando `[]` com um único número é usando `[]` com
um intervalo para criar uma fatia de string contendo bytes específicos:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Here, `s` will be a `&str` that contains the first four bytes of the string.
Earlier, we mentioned that each of these characters was two bytes, so that
means that `s` will be “Зд”.

Aqui, `s` será um `&str` que contém os primeiros quatro bytes da string.
Mais cedo, mencionamos que cada um desses personagens era de dois bytes, de modo que
significa que `s` será “Зд”.


What would happen if we did `&hello[0..1]`? The answer: it will panic at
runtime, in the same way that accessing an invalid index in a vector does:

O que aconteceria se fizéssemos `&hello[0..1]`? A resposta: entrará em pânico
em tempo de execução, da mesma maneira que acessar um índice inválido em um vetor:

```text
thread 'main' panicked at 'index 0 and/or 1 in `Здравствуйте` do not lie on
character boundary', ../src/libcore/str/mod.rs:1694
```

You should use this with caution, since it can cause your program to crash.

Você deve usar isso com cautela, pois isso pode fazer com que seu programa falhe.

### Methods for Iterating Over Strings
### Métodos para interagir Sobre Strings

Luckily, there are other ways we can access elements in a String.

Felizmente, existem outras maneiras de acessar elementos em um String.

If we need to perform operations on individual Unicode scalar values, the best
way to do so is to use the `chars` method. Calling `chars` on “नमस्ते”
separates out and returns six values of type `char`, and you can iterate over
the result in order to access each element:


Se precisarmos realizar operações em valores escalares Unicode individuais, a melhor
maneira de fazer isso é usar o método `chars`. Chamando `chars` em “नमस्ते”
é separado e retorna seis valores do tipo `char`, e você pode interar
no resultado para acessar cada elemento:

```rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

This code will print:

Este código irá imprimir:

```text
न
म
स
्
त
े
```

The `bytes` method returns each raw byte, which might be appropriate for your
domain:

O método `bytes` retorna cada byte bruto, que pode ser apropriado para o seu
domínio:

```rust
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

This code will print the 18 bytes that make up this `String`, starting with:

Este código imprimirá os 18 bytes que compõem esse `String`, começando por:

```text
224
164
168
224
// ... etc
```

But make sure to remember that valid Unicode scalar values may be made up of
more than one byte.

Mas lembre-se de que os valores escalares Unicode válidos podem ser constituídos por
mais de um byte.

Getting grapheme clusters from strings is complex, so this functionality is not
provided by the standard library. There are crates available on crates.io if
this is the functionality you need.

Obter clusters de grafemas de strings é complexo, então esta funcionalidade não é
fornecida pela biblioteca padrão. Existem crates disponíveis em crates.io se
Esta é a funcionalidade que você precisa.

### Strings are Not so Simple
### As Strings Não são tão Simples

To summarize, strings are complicated. Different programming languages make
different choices about how to present this complexity to the programmer. Rust
has chosen to make the correct handling of `String` data the default behavior
for all Rust programs, which does mean programmers have to put more thought
into handling UTF-8 data upfront. This tradeoff exposes more of the complexity
of strings than other programming languages do, but this will prevent you from
having to handle errors involving non-ASCII characters later in your
development lifecycle.

Para resumir, as strings são complicadas. Diferentes linguagens de programação fazem
escolhas diferentes sobre como apresentar essa complexidade ao programador. Rust
optou por fazer o tratamento correto dos dados `String` o comportamento padrão
para todos os programas Rust, o que significa que os programadores devem pensar mais
no gerenciamento de dados UTF-8 antecipadamente. Este tradeoff expõe mais da complexidade
de strings do que outras linguagens de programação, mas isso irá impedi-lo de
ter que lidar com erros envolvendo caracteres não-ASCII mais tarde em seu
ciclo de desenvolvimento.

Let’s switch to something a bit less complex: hash maps!

Vamos mudar para algo um pouco menos complexo: hash maps!
