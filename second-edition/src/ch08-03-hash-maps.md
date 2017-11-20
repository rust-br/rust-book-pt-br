## Hash Maps

The last of our common collections is the *hash map*. The type `HashMap<K, V>`
stores a mapping of keys of type `K` to values of type `V`. It does this via a
*hashing function*, which determines how it places these keys and values into
memory. Many different programming languages support this kind of data
structure, but often with a different name: hash, map, object, hash table, or
associative array, just to name a few.

A última das nossas coleções comuns é o *hash map*. O tipo `HashMap <K, V>`
armazena um mapeamento de chaves do tipo `K` para valores do tipo` V`. Ele faz isso através de um
*hashing function*, que determina como ele coloca essas chaves e valores em
memória. Muitas linguagens de programação diferentes suportam este tipo de 
estrutura de dados, mas muitas vezes com um nome diferente: hash, map, object, hash table ou
associative array, apenas para citar alguns.

Hash maps are useful for when you want to be able to look up data not by an
index, as you can with vectors, but by using a key that can be of any type. For
example, in a game, you could keep track of each team’s score in a hash map
where each key is a team’s name and the values are each team’s score. Given a
team name, you can retrieve their score.

Os Hash maps são úteis para quando você deseja poder procurar dados sem uso de
índice, como você pode com vetores, mas usando uma chave que pode ser de qualquer tipo. Por 
exemplo, em um jogo, você poderia acompanhar a pontuação de cada equipe em um hash map
onde cada chave é o nome de uma equipe e os valores são cada pontuação da equipe. Dado um
nome da equipe, você pode recuperar sua pontuação.

We’ll go over the basic API of hash maps in this chapter, but there are many
more goodies hiding in the functions defined on `HashMap` by the standard
library. As always, check the standard library documentation for more
information.

Examinaremos a API básica dos hash map neste capítulo, mas há muitos
mais coisas escondidas nas funções definidas no `HashMap` pela biblioteca
padrão. Como sempre, verifique a documentação da biblioteca padrão para mais
informação. 

### Creating a New Hash Map
### Criando um novo Hash Map

We can create an empty `HashMap` with `new`, and add elements with `insert`.
Here we’re keeping track of the scores of two teams whose names are Blue and
Yellow. The Blue team will start with 10 points and the Yellow team starts with
50:

Podemos criar um `HashMap` vazio com `new`, e adicionar elementos com `insert`.
Aqui, estamos acompanhando as pontuações de duas equipes cujos nomes são Blue e
Yellow. A equipe blue começará com 10 pontos e a equipe yellow começa com
50:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

Note that we need to first `use` the `HashMap` from the collections portion of
the standard library. Of our three common collections, this one is the least
often used, so it’s not included in the features imported automatically in the
prelude. Hash maps also have less support from the standard library; there’s no
built-in macro to construct them, for example.

Observe que precisamos primeiro `use` o `HashMap` da parte de coleções da
biblioteca padrão. De nossas três coleções comuns, esta é a de menor
frequencia de uso, por isso não está inclusa nos recursos importados automaticamente no
prelúdio. Os Hash maps também têm menos suporte da biblioteca padrão; não há
macro embutida para construí-los, por exemplo.

Just like vectors, hash maps store their data on the heap. This `HashMap` has
keys of type `String` and values of type `i32`. Like vectors, hash maps are
homogeneous: all of the keys must have the same type, and all of the values
must have the same type.

Assim como os vetores, os mapas hash armazenam seus dados no heap. Este `HashMap` tem
chaves do tipo `String` e valores do tipo `i32`. Como vetores, os hash maps são
homogêneos: todas as chaves devem ter o mesmo tipo e todos os valores
devem ter o mesmo tipo.

Another way of constructing a hash map is by using the `collect` method on a
vector of tuples, where each tuple consists of a key and its value. The
`collect` method gathers up data into a number of collection types, including
`HashMap`. For example, if we had the team names and initial scores in two
separate vectors, we can use the `zip` method to create a vector of tuples
where “Blue” is paired with 10, and so forth. Then we can use the `collect`
method to turn that vector of tuples into a `HashMap`:

Outra maneira de construir um hash map é usando o método `collect` em um
vetor de tuplas, onde cada tupla consiste de uma chave e seu valor. O 
método `collect` reúne dados em vários tipos de coleção, incluindo
`HashMap`. Por exemplo, se tivéssemos os nomes das equipes e as pontuações iniciais em dois
vetores separados, podemos usar o método `zip` para criar um vetor de tuplas
onde “Blue” é emparelhado com 10, e assim por diante. Então podemos usar o método `collect`
para transformar esse vetor de tuplas em um `HashMap`:

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

The type annotation `HashMap<_, _>` is needed here because it’s possible to
`collect` into many different data structures, and Rust doesn’t know which you
want unless you specify. For the type parameters for the key and value types,
however, we use underscores and Rust can infer the types that the hash map
contains based on the types of the data in the vector.

A anotação de tipo `HashMap <_, _>` é necessária aqui porque é possível
`collect` em muitas estruturas de dados diferentes, e Rust não sabe qual você
deseja, a menos que você especifique. Para os parâmetros de tipo, para os tipos de chave e valor,
no entanto, usamos underscores e Rust pode inferir os tipos que o hash map
contém com base nos tipos de dados no vetor.

### Hash Maps and Ownership
### Hash Maps and Ownership

For types that implement the `Copy` trait, like `i32`, the values are copied
into the hash map. For owned values like `String`, the values will be moved and
the hash map will be the owner of those values:

Para os tipos que implementam a `Copy` trait, como `i32`, os valores são copiados
no hash map. Para valores owned como `String`, os valores serão movidos e
o hash map será o owner desses valores:

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are invalid at this point
```

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name e field_value são inválidos neste ponto
```


We would not be able to use the bindings `field_name` and `field_value` after
they have been moved into the hash map with the call to `insert`.

Não poderíamos usar as ligações `field_name` e` field_value` depois
que foram transferidos para o hash map com a chamada para `insert`.

If we insert references to values into the hash map, the values themselves will
not be moved into the hash map. The values that the references point to must be
valid for at least as long as the hash map is valid, though. We will talk more
about these issues in the Lifetimes section of Chapter 10.

Se inserimos referências a valores no hash map, os próprios valores
não serão movido para o hash map. Os valores que as referências apontam devem ser
válido pelo menos enquanto o hash map seja válido, no entanto. Falaremos mais
sobre esses problemas na seção Lifetimes do Capítulo 10. 

### Accessing Values in a Hash Map
### Acessando valores em um Hash Map

We can get a value out of the hash map by providing its key to the `get` method:

Podemos obter um valor do hash map fornecendo a chave para o método `get`:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

Here, `score` will have the value that’s associated with the Blue team, and the
result will be `Some(&10)`. The result is wrapped in `Some` because `get`
returns an `Option<&V>`; if there’s no value for that key in the hash map, `get`
will return `None`. The program will need to handle the `Option` in one of the
ways that we covered in Chapter 6.

Aqui, `score` terá o valor que está associado à equipe Blue, e o
resultado será `Some(&10)`. O resultado está envolvido em `Some` porque `get`
retorna `Option<&V>`; se não houver valor para essa chave no hash map, `get`
retornará `None`. O programa precisará lidar com `Option` em uma das
formas que abordamos no Capítulo 6.

We can iterate over each key/value pair in a hash map in a similar manner as we
do with vectors, using a `for` loop:

Podemos iterar sobre cada par chave/valor em um hash map de uma maneira similar à que 
fazemos com vetores, usando um loop `for`:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

This will print each pair, in an arbitrary order:

Isso imprimirá cada par, em uma ordem arbitrária:

```text
Yellow: 50
Blue: 10
```

### Updating a Hash Map
### Atualizando um Hash Map

While the number of keys and values is growable, each individual key can only
have one value associated with it at a time. When we want to change the data in
a hash map, we have to decide how to handle the case when a key already has a
value assigned. We could choose to replace the old value with the new value,
completely disregarding the old value. We could choose to keep the old value
and ignore the new value, and only add the new value if the key *doesn’t*
already have a value. Or we could combine the old value and the new value.
Let’s look at how to do each of these!

Embora o número de chaves e valores sejam crescentes, cada chave individual pode apenas
tem um valor associado a ele por vez. Quando queremos mudar os dados em
um hash map, temos que decidir como lidar com o caso quando uma chave já possui uma
valor atribuído. Poderíamos optar por substituir o valor antigo pelo novo valor,
desconsiderando completamente o valor antigo. Poderíamos escolher manter o valor antigo
e ignorar o novo valor, e apenas adicione o novo valor se a chave ainda *não*
tem um valor. Ou podemos combinar o valor antigo ao valor novo.
Vejamos como fazer cada um desses!

#### Overwriting a Value
#### Sobrescrevendo um Valor

If we insert a key and a value into a hash map, then insert that same key with
a different value, the value associated with that key will be replaced. Even
though this following code calls `insert` twice, the hash map will only contain
one key/value pair because we’re inserting the value for the Blue team’s key
both times:

Se inserimos uma chave e um valor em um hash map, então  se inserir essa mesma chave com
um valor diferente, o valor associado a essa chave será substituído. Eembora 
o seguinte código chame `insert` duas vezes, o hash map só conterá
um par de chave/valor porque inserimos o valor da chave da equipe Blue
ambas as vezes:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

This will print `{"Blue": 25}`. The original value of 10 has been overwritten.

Isso imprimirá `{" Blue ": 25}`. O valor original de 10 foi substituído.

#### Only Insert If the Key Has No Value
#### Insira Apenas se a Chave Não Possui Valor

It’s common to want to check if a particular key has a value and, if it does
not, insert a value for it. Hash maps have a special API for this, called
`entry`, that takes the key we want to check as an argument. The return value
of the `entry` function is an enum, `Entry`, that represents a value that might
or might not exist. Let’s say that we want to check if the key for the Yellow
team has a value associated with it. If it doesn’t, we want to insert the value
50, and the same for the Blue team. With the entry API, the code for this looks
like:

É comum querer verificar se uma determinada chave tem um valor e, se 
não tiver, inserir um valor para ela. Os Hash maps possuem uma API especial para isso, chamada
`entry`, que leva a chave que queremos verificar como um argumento. O valor de retorno
da função `entry` é um enum, `Entry`, que representa um valor que pode
ou não existir. Digamos que queremos verificar se a chave para o time Yellow
tem um valor associado a ela. Se não tiver, queremos inserir o valor
50, e o mesmo para a equipe Blue. Com a API de entrada, o código irá parecer 
com:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

The `or_insert` method on `Entry` returns the value for the corresponding
`Entry` key if it exists, and if not, inserts its argument as the new value for
this key and returns the modified `Entry`. This is much cleaner than writing
the logic ourselves, and in addition, plays more nicely with the borrow checker.

O método `or_insert` em `Entry` retorna o valor para o `Entry` correspondente
se a chave existir, e se não, insere seu argumento como o novo valor para
esta chave e retorna a `Entry` modificada. Isso é muito mais limpo do que escrever
a lógica por nós mesmos e, além disso, trabalha-se de forma maislimpa com o borrow checker.

This code will print `{"Yellow": 50, "Blue": 10}`. The first call to `entry`
will insert the key for the Yellow team with the value 50, since the Yellow
team doesn’t have a value already. The second call to `entry` will not change
the hash map since the Blue team already has the value 10.

Este código imprimirá `{"Yellow": 50, "Blue": 10}`. A primeira chamada para `entry`
irá inserir a chave para a equipe Yellow com o valor 50, uma vez que o time Yellow
já não possua um valor. A segunda chamada para `entry` não vai mudar
o hash map pois o time Blue já possui o valor 10.

#### Update a Value Based on the Old Value
#### Atualize um Valor com Base no Valor Antigo

Another common use case for hash maps is to look up a key’s value then update
it, based on the old value. For instance, if we wanted to count how many times
each word appeared in some text, we could use a hash map with the words as keys
and increment the value to keep track of how many times we’ve seen that word.
If this is the first time we’ve seen a word, we’ll first insert the value `0`.

Outro caso de uso comum para hash maps é procurar o valor de uma chave e, em seguida, atualiza-la
, com base no valor antigo. Por exemplo, se quisermos contar quantas vezes
cada palavra apareceu em algum texto, podemos usar um hash map com as palavras como chaves
e incrementar o valor para acompanhar quantas vezes vimos essa palavra.
Se esta é a primeira vez que vimos uma palavra, primeiro inseriremos o valor `0`.

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

This will print `{"world": 2, "hello": 1, "wonderful": 1}`. The `or_insert`
method actually returns a mutable reference (`&mut V`) to the value for this
key. Here we store that mutable reference in the `count` variable, so in order
to assign to that value we must first dereference `count` using the asterisk
(`*`). The mutable reference goes out of scope at the end of the `for` loop, so
all of these changes are safe and allowed by the borrowing rules.

Isso imprimirá `{"world": 2, "hello": 1, "wonderful": 1}`. O método `or_insert`
na verdade retorna uma referência mutável (`&mutV`) para o valor desta
chave. Aqui nós armazenamos essa referência mutável na variável `count`, então, 
para poder atribuir esse valor, devemos primeiro desreferenciar `count` usando o asterisco
(`*`). A referência mutável fica fora do escopo no final do loop `for`, então
todas essas mudanças são seguras e permitidas pelas regras de borrow.

### Hashing Function
### Funções Hashing

By default, `HashMap` uses a cryptographically secure hashing function that can
provide resistance to Denial of Service (DoS) attacks. This is not the fastest
hashing algorithm out there, but the tradeoff for better security that comes
with the drop in performance is worth it. If you profile your code and find
that the default hash function is too slow for your purposes, you can switch to
another function by specifying a different *hasher*. A hasher is a type that
implements the `BuildHasher` trait. We’ll be talking about traits and how to
implement them in Chapter 10. You don’t necessarily have to implement your own
hasher from scratch; crates.io has libraries that others have shared that
provide hashers implementing many common hashing algorithms.

Por padrão, `HashMap` usa uma função de hashing criptográficamente segura que pode
fornecer resistência aos ataques de Negação de Serviço (DoS). Este não é o algoritmo 
mais rápido de hashing por aí, mas a compensação por uma melhor segurança que vem
com a queda na performance vale a pena. Se você testar a velocidade do seu código e encontrar
que a função de hash padrão é muito lenta para seus propósitos, você pode mudar para
outra função especificando um *hasher* diferente. Um hasher é um tipo que
implementa a trait `BuildHasher`. Vamos falar sobre traits e como
implementá-los no Capítulo 10. Você não precisa necessariamente implementar o seu próprio
hasher do zero; crates.io tem bibliotecas de hashers de uso comum que 
outras pessoas compartilharam lá.

## Summary
## Summary

Vectors, strings, and hash maps will take you far in programs where you need to
store, access, and modify data. Here are some exercises you should now be
equipped to solve:

Vetores, strings e hash maps irão levá-lo longe em programas onde você precisa
armazenar, acessar e modificar dados. Aqui estão alguns exercícios que você deve estar
capacitado para resolver:

* Given a list of integers, use a vector and return the mean (average), median
  (when sorted, the value in the middle position), and mode (the value that
  occurs most often; a hash map will be helpful here) of the list.
* Convert strings to Pig Latin, where the first consonant of each word is moved
  to the end of the word with an added “ay”, so “first” becomes “irst-fay”.
  Words that start with a vowel get “hay” added to the end instead (“apple”
  becomes “apple-hay”). Remember about UTF-8 encoding!
* Using a hash map and vectors, create a text interface to allow a user to add
  employee names to a department in the company. For example, “Add Sally to
  Engineering” or “Add Amir to Sales”. Then let the user retrieve a list of all
  people in a department or all people in the company by department, sorted
  alphabetically.

* Dada uma lista de inteiros, use um vetor e retorne a média, a mediana
   (quando classificado, o valor na posição do meio) e modo (o valor que
   ocorre com mais frequência; um hash map será útil aqui) da lista.
* Converta strings para Pig Latin, onde a primeira consoante de cada palavra é movida
   para o final da palavra adicionado um "ay" , então “first” se torna “irst-fay”.
   Palavras que começam com uma vogal recebem “hay” adicionado ao final (“apple”
   torna-se “apple-hay”). Lembre-se sobre a codificação UTF-8!
* Usando um hash map e vetores, crie uma interface de texto para permitir que um usuário adicione
   nomes de funcionários para um departamento da empresa. Por exemplo, “Add Sally to
   Engineering” ou “Add Amir to Sales”. Em seguida, deixe o usuário recuperar uma lista de todas
   as pessoas de um departamento ou todas as pessoas na empresa por departamento, ordenadas
   alfabeticamente.

The standard library API documentation describes methods these types have that
will be helpful for these exercises!

A documentação da API da biblioteca padrão descreve métodos que esses tipos possuem
que será útil para esses exercícios!

We’re getting into more complex programs where operations can fail, which means
it’s a perfect time to go over error handling next!

Estamos entrando em programas mais complexos onde as operações podem falhar, o que significa
que é um momento perfeito para passar pelo tratamento de erros em seguida!
