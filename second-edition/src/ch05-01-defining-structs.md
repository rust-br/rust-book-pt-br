## Defining and Instantiating Structs
## Definindo e Instanciando Structs 

Structs are similar to tuples, which were discussed in Chapter 3. Like tuples,
the pieces of a struct can be different types. Unlike tuples, we name each piece
of data so it’s clear what the values mean. As a result of these names, structs 
are more flexible than tuples: we don’t have to rely on the order of the data 
to specify or access the values of an instance.

Structs são semelhantes às tuples, que foram discutidas no Capítulo 3. Como 
tuples, as peças de uma struct podem ser de tipos diferentes. Ao contrário das 
tuples, nomeie cada dado de modo que seja claro o que cada um significa. Como 
resultado destes nomes, structs são mais flexíveis que tuples: nós não temos de
saber a ordem dos dados para especificar ou aceder aos valores de uma instância.

To define a struct, we enter the keyword `struct` and name the entire struct. 
A struct’s name should describe the significance of the pieces of data being 
grouped together. Then, inside curly braces, we define the names and types of 
the pieces of data, which we call *fields*. For example, Listing 5-1 shows a 
struct to store information about a user account:

Para definir um struct, digite a palavra-chave 'struct' e o nome todo o struct.
O nome do struct deve descrever o significado dos dados agrupados. Em seguida,
dentro das chaves, vamos definir os nomes e tipos dos dados, o que chamamos de 
*campos*. Por exemplo, a lista 5-1 mostra uma struct para armazenar informações
sobre a conta de um usuário:

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

<span class="caption">Listing 5-1: A `User` struct definition</span>
<span class="caption">Lista 5-1: Definição de struct `User`</span>

To use a struct after we’ve defined it, we create an *instance* of that struct 
by specifying concrete values for each of the fields. We create an instance by 
stating the name of the struct, and then add curly braces containing 
`key: value` pairs where the keys are the names of the fields and the values 
are the data we want to store in those fields. We don’t have to specify the 
fields in the same order in which we declared them in the struct. In other 
words, the struct definition is like a general template for the type, and 
instances fill in that template with particular data to create values of the 
type. For example, we can declare a particular user as shown in Listing 5-2:

Para usar uma struct depois a definirmos, criamos uma *instância* dessa struct,
especificando valores para cada um dos campos. Estamos a criar uma instância, 
indicando o nome da struct e depois entre chavetas, adicionamos chaves contendo
pares ‘chave:valor’ onde as chaves são os nomes dos campos e os valores são os 
dados que deseja armazenar nesses campos. Nós não temos para especificar os 
campos na mesma ordem em que os temos declarado na struct. Em outras palavras,
a definição da struct é como um modelo geral para o tipo, e as instâncias 
preenchem esse modelo com os dados específicos, para criar valores desse tipo.
Por exemplo, podemos declarar um usuário específico como mostrado 
na lista 5-2:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let user1 = User {
    email: String::from("alguem@exemplo.com"),
    username: String::from("algumnome123"),
    active: true,
    sign_in_count: 1,
};
```

<span class="caption">Listing 5-2: Creating an instance of the `User`
struct</span>
<span class="caption">Lista 5-2: Criar uma instancia da struct `User`</span>


To get a specific value from a struct, we can use dot notation. If we wanted 
just this user’s email address, we can use `user1.email` wherever we want to 
use this value. To change a value in a struct, if the instance is mutable, we 
can use the dot notation and assign into a particular field. Listing 5-3 shows
how to change the value in the `email` field of a mutable `User` instance:

Para obter um valor específico de uma struct, podemos utilizar a notação de 
ponto. Se quiséssemos apenas esse endereço de e-mail do usuário, podemos usar
"user1.email" sempre que queremos usar este valor. Para alterar um valor em uma
struct, se a instância é mutável, podemos usar a notação de ponto e atribuir 
a um campo específico. Lista 5-3 mostra como alterar o valor do campo "e-mail"
de uma instância de "User" mutável:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let mut user1 = User {
    email: String::from("alguem@exemplo.com"),
    username: String::from("algumnome123"),
    active: true,
    sign_in_count: 1,
};

user1.email = String::from("outroemail@exemplo.com");
```

<span class="caption">Listing 5-3: Changing the value in the `email` field 
of a `User` instance</span>

<span class="caption">Lista 5-3: Mudando o valo no campo `email` da 
instância de `User`</span>


### Field Init Shorthand when Variables Have the Same Name as Fields
### Abreviatura do Init dos campos quando as variáveis têm o mesmo nome dos Campos


If you have variables with the same names as struct fields, you can use *field
init shorthand*. This can make functions that create new instances of structs 
more concise. First, let’s look at the more verbose way to initialize a struct 
instance. The function named `build_user` shown here in Listing 5-4 has 
parameters named `email` and `username`. The function creates and returns 
a `User` instance:

Se você tiver as variáveis com os mesmos nomes dos campos da struct, você pode 
usar o campo *init shorthand*. Isto pode fazer com que as funções que criam 
novas instâncias de structs mais concisos. Em primeiro lugar, vejamos o modo 
mais detalhado para inicializar uma instância de uma struct. A função chamada
"build_user" mostrada aqui na Lista 5-4 tem parâmetros chamados 'e-mail' e 
'username’ (Nome de usuário). A função cria e retorna uma instância do "User":

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">Listing 5-4: A `build_user` function that takes an email
and username and returns a `User` instance</span>

<span class="caption">Lista 5-4: Uma função "build_user" que recebe um endereço
de correio electrónico e o nome do usuário e retorna uma instância de 'user'
</span>



Because the parameter names `email` and `username` are the same as the `User` 
struct’s field names `email` and `username`, we can write `build_user` without
the repetition of `email` and `username` as shown in Listing 5-5. This version
of `build_user` behaves the same way as the one in Listing 5-4. The field init 
syntax can make cases like this shorter to write, especially when structs have 
many fields.

Porque os nomes dos parâmetros 'e-mail' e 'username' são os mesmos que os nomes
de campo do 'e-mail' e 'Nome de usuário' da struct "User", podemos escrever 
"build_user" sem a repetição de "e-mail' e 'username' como mostrado na listagem
5-5. Esta versão de "build_user" comporta-se da mesma maneira como na Lista 5-4.
A sintaxe do campo init pode fazer casos como esse mais curtos para escrever, 
especialmente quando structs têm muitos campos.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">Listing 5-5: A `build_user` function that uses field init
syntax since the `email` and `username` parameters have the same name as struct
fields</span>

<span class="caption">Lista 5-5: Uma função "build_user" que usa a sintaxe 
campo init porque os parâmetros 'e-mail' e 'username' têm o mesmo nome dos 
campos da struct</span>

### Creating Instances From Other Instances With Struct Update Syntax
### A Criação de instâncias de outras instâncias com Sintaxe de Atualização
### da Struct (Struct Update Syntax)

It’s often useful to create a new instance from an old instance, using most of 
the old instance’s values but changing some. Listing 5-6 shows an example of 
creating a new `User` instance in `user2` by setting the values of `email` and
`username` but using the same values for the rest of the fields from the 
`user1` instance we created in Listing 5-2:

É frequentemente útil criar uma nova instância a partir de uma antiga instância, 
usando a maioria dos valores da antiga instância mas mudando alguns. A Lista 5-6
mostra um exemplo da criação de uma nova instância do "user1" em "user2" através
da definição dos valores de 'e-mail' e 'username' mas usando os mesmos valores 
para o resto dos campos do exemplo "user1" que criamos na Lista 5-2:

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("alguem@exemplo.com"),
#     username: String::from("algumnome123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("outro@exemplo.com"),
    username: String::from("outronome567"),
    active: user1.active,
    sign_in_count: user1.sign_in_count,
};
```

<span class="caption">Listing 5-6: Creating a new `User` instance, `user2`, and
setting some fields to the values of the same fields from `user1`</span>

<span class="caption">Lista 5-6: Criação de uma nova instância do "User", 
"user2", e a definição de alguns campos para os valores dos mesmos campos
do "user1"</span>


The *struct update syntax* achieves the same effect as the code in Listing 5-6 
using less code. The struct update syntax uses `..` to specify that the 
remaining fields not set explicitly should have the same value as the fields in
the given instance. The code in Listing 5-7 also creates an instance in `user2`
that has a different value for `email` and `username` but has the same values 
for the `active` and `sign_in_count` fields that `user1` has:

A *struct update syntax* (Sintaxe de Atualização da Struct) alcança o mesmo 
efeito que o código na Lista 5-6 usando menos código. A sintaxe de atualização
struct usa '.' para especificar que os campos restantes não explicitamente 
configurado deve ter o mesmo valor que os campos na determinada instância. 
O código na listagem 5-7 também cria uma instância no "user2", que tem um valor
diferente de 'e-mail' e 'Nome de usuário' mas tem os mesmos valores para a 
"ACTIVO" e "ign_no_count" campos que "user1":


```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("someone@example.com"),
#     username: String::from("someusername123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

<span class="caption">Listing 5-7: Using struct update syntax to set a new
`email` and `username` values for a `User` instance but use the rest of the
values from the fields of the instance in the `user1` variable</span>

<span class="caption"> Lista 5-7: Usando struct update sintax para definir
valores de `email` e `username` de uma nova instância do `User` mas usar o 
resto dos valores dos campos da instância da variável `user1`</span>

### Tuple Structs without Named Fields to Create Different Types
### Tuple Structs sem Campos Nomeados para Criar Tipos Diferentes

We can also define structs that look similar to tuples, called *tuple structs*,
that have the added meaning the struct name provides, but don’t have names
associated with their fields, just the types of the fields. The definition of a
tuple struct still starts with the `struct` keyword and the struct name, which
are followed by the types in the tuple. For example, here are definitions and
usages of tuple structs named `Color` and `Point`:

Podemos também definir structs que parecem semelhantes a tuples, chamadas 
*tuple structs*, que têm o significado que o nome struct fornece, mas não 
têm os nomes associados com os seus campos, apenas os tipos dos campos. 
A definição de uma tuple struct, ainda começa com a palavra-chave `struct` e o
nome da struct, que são seguidos pelos tipos na tuple. Por exemplo, aqui estão
as definições e usos da tuple structs chamado `Color` e `Point`:

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

Note that the `black` and `origin` values are different types, since they’re
instances of different tuple structs. Each struct we define is its own type,
even though the fields within the struct have the same types. Otherwise, tuple
struct instances behave like tuples, which we covered in Chapter 3.

Note que os valores "black" e "origin" são diferentes tipos, uma vez que eles
são de diferentes instâncias tuple structs. Cada struct que definimos é o seu 
próprio tipo, embora os campos dentro do struct tenham os mesmos tipos. Caso 
contrário, as tuple struct comportam-se como instâncias de tuples, que 
discutimos no Capítulo 3.

### Unit-Like Structs without Any Fields
### Estruturas Unit-Like sem quaisquer campos

We can also define structs that don’t have any fields! These are called
*unit-like structs* since they behave similarly to `()`, the unit type.
Unit-like structs can be useful in situations such as when you need to
implement a trait on some type, but you don’t have any data that you want to
store in the type itself. We’ll be discussing traits in Chapter 10.

Podemos também definir structs que não têm quaisquer campos! Estes são chamados
de *unit-like structs* (Unidades como Structs) porque eles se comportam da mesma
forma que '()', o tipo unidade. Unit-like structs podem ser úteis em situações,
como quando você precisa implementar um trait de algum tipo, mas você não tem 
quaisquer dados que você deseja armazenar no tipo em si. Traits será discutido 
no Capítulo 10.


### Ownership of Struct Data
### A Propriedade de Dados da Struct

In the `User` struct definition in Listing 5-1, we used the owned `String`
type rather than the `&str` string slice type. This is a deliberate choice
because we want instances of this struct to own all of its data and for that
data to be valid for as long as the entire struct is valid.

Na definição de struct 'User', na Listagem 5-1, utilizamos a propriedade tipo 
'String' em vez de '&str’, uma ‘fatia’ tipo string. Esta é uma escolha 
deliberada, porque queremos que instâncias deste struct possuam todos os seus 
dados e para que os dados sejam válidos por todo o tempo que o struct é válido.

It’s possible for structs to store references to data owned by something else,
but to do so requires the use of *lifetimes*, a Rust feature that is discussed
in Chapter 10. Lifetimes ensure that the data referenced by a struct is valid
for as long as the struct is. Let’s say you try to store a reference in a
struct without specifying lifetimes, like this:

É possível para structs armazenar referências a dados que são propriedade de 
algo diferente, mas para isso requer o uso de *lifetimes*, uma característica 
de Rust que é discutida no Capítulo 10. Lifetimes garantem que os dados 
referenciados por um struct são válidos enquanto struct existir. Vamos dizer 
que você tenta armazenar uma referência em um struct sem especificar lifetimes,
como este:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
struct User {
   username: &str,
    email: &str,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: "someone@example.com",
        username: "someusername123",
        active: true,
        sign_in_count: 1,
    };
}
```

The compiler will complain that it needs lifetime specifiers:
O compilador irá reclamar que é preciso especificar lifetimes:

```text
error[E0106]: missing lifetime specifier
 -->
  |
2 |     username: &str,
  |               ^ expected lifetime parameter

 error[E0106]: missing lifetime specifier
 -->
  |
3 |     email: &str,
  |            ^ expected lifetime parameter
```

 We’ll discuss how to fix these errors so you can store references in structs
 in Chapter 10, but for now, we’ll fix errors like these using owned types like
 `String` instead of references like `&str`.

Vamos discutir como corrigir estes erros, assim você pode armazenar referências
em structs no Capítulo 10, mas por agora, vamos corrigir erros como estes 
usando tipos de propriedade, utilizando `String` em vez de referências 
como `&str`.
