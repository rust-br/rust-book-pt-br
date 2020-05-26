## Improving Our I/O Project
## Melhorando Nosso Projeto de E/S

With this new knowledge about iterators, we can improve the I/O project in
Chapter 12 by using iterators to make places in the code clearer and more
concise. Let’s look at how iterators can improve our implementation of the
`Config::new` function and the `search` function.

Com esse novo conhecimento sobre iteradores, podemos melhorar o projeto de 
E/S (I/O) no Capítulo 12 usando iteradores para tornar os locais no código mais 
claros e concisos. Vejamos como os iteradores podem melhorar nossa implementação 
da função `Config::new` e da função `search`.

### Removing a `clone` Using an Iterator
### Removendo um `clone` Usando um Iterador

In Listing 12-6, we added code that took a slice of `String` values and created
an instance of the `Config` struct by indexing into the slice and cloning the
values, allowing the `Config` struct to own those values. In Listing 13-24,
we’ve reproduced the implementation of the `Config::new` function as it was in
Listing 12-23:

Na Listagem 12-6, adicionamos código que utilizou uma fatia (slice) dos valores `String` 
e criamos uma instância da estrutura `Config` indexando a fatia e clonando os valores, 
permitindo que a estrutura `Config` possua esses valores. Na Listagen 13-24, 
reproduzimos a implementação da função `Config::new` como na Listagem 12-23:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

<span class="caption">Listing 13-24: Reproduction of the `Config::new` function
from Listing 12-23</span>

<span class="caption">Listagem 13-24: Reprodução da função `Config::new` 
da Listagem 12-23</span>

At the time, we said not to worry about the inefficient `clone` calls because
we would remove them in the future. Well, that time is now!

Na época, dissemos que não nos preocupávamos com as chamadas ineficientes do 
`clone` porque as removeríamos no futuro. Bem, essa hora é agora!

We needed `clone` here because we have a slice with `String` elements in the
parameter `args`, but the `new` function doesn’t own `args`. To return
ownership of a `Config` instance, we had to clone the values from the `query`
and `filename` fields of `Config` so the `Config` instance can own its values.

Precisávamos de `clone` aqui porque temos uma fatia com elementos `String` no 
parâmetro `args`, mas a função `new` não tem ownership `args`. Para retornar 
ownership de uma instância `Config`, tivemos que clonar os valores dos campos 
`query` e `filename` de `Config` para que a instância `Config` possa ter ownership 
sobre seus valores.

With our new knowledge about iterators, we can change the `new` function to
take ownership of an iterator as its argument instead of borrowing a slice.
We’ll use the iterator functionality instead of the code that checks the length
of the slice and indexes into specific locations. This will clarify what the
`Config::new` function is doing because the iterator will access the values.

Com nosso novo conhecimento sobre iteradores, podemos alterar a função `new` 
para assumir ownership de um iterador como argumento, em vez de pegar uma 
fatia borrowed (emprestada). Usaremos a funcionalidade do iterador em vez do 
código que verifica o comprimento da fatia e os índices em locais específicos. 
Isso esclarecerá o que a função `Config::new` está fazendo, porque o iterador 
acessará os valores.

Once `Config::new` takes ownership of the iterator and stops using indexing
operations that borrow, we can move the `String` values from the iterator into
`Config` rather than calling `clone` and making a new allocation.

Uma vez que o `Config::new` possui ownership do iterador e para de usar as operações 
de indexação que borrow, podemos mover os valores `String` do iterador para `Config`, 
em vez de chamar `clone` e fazer uma nova alocação.

#### Using the Returned Iterator Directly
#### Usando o Iterador Retornado Diretamente

Open your I/O project’s *src/main.rs* file, which should look like this:

Abra o arquivo *src/main.rs  do seu projeto de E/S, que deve se parecer 
com o seguinte:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
}
```

We’ll change the start of the `main` function that we had in Listing 12-24 at
to the code in Listing 13-25. This won’t compile until we update `Config::new`
as well.

Alteraremos o início da função `main` que tínhamos na Listagem 12-24 no código 
da Listagem 13-25. Isso não será compilado até que atualizemos o `Config::new` 
também.

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let config = Config::new(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
}
```

<span class="caption">Listing 13-25: Passing the return value of `env::args` to
`Config::new`</span>

<span class="caption">Lista 13-25: Passando o valor de retorno de `env::args` para 
`Config::new`</span>

The `env::args` function returns an iterator! Rather than collecting the
iterator values into a vector and then passing a slice to `Config::new`, now
we’re passing ownership of the iterator returned from `env::args` to
`Config::new` directly.

A função `env::args` retorna um iterador! Em vez de coletar os valores do 
iterador em um vetor e, em seguida, passar uma fatia para `Config::new`, 
agora estamos passando ownership do iterador retornado de `env::args` para 
`Config::new` diretamente.

Next, we need to update the definition of `Config::new`. In your I/O project’s
*src/lib.rs* file, let’s change the signature of `Config::new` to look like
Listing 13-26. This still won’t compile because we need to update the function
body.

Em seguida, precisamos atualizar a definição de `Config::new`. No arquivo *src/lib.rs* 
do seu projeto de E/S, alteremos a assinatura do `Config::new` para parecer com a 
Listagem 13-26. Isso ainda não será compilado porque precisamos atualizar o corpo 
da função.

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        // --snip--
```

<span class="caption">Listing 13-26: Updating the signature of `Config::new` to
expect an iterator</span>

<span class="caption">Listagem 13-26: Atualizando a assinatura de `Config::new` 
para esperar um iterador</span>

The standard library documentation for the `env::args` function shows that the
type of the iterator it returns is `std::env::Args`. We’ve updated the
signature of the `Config::new` function so the parameter `args` has the type
`std::env::Args` instead of `&[String]`. Because we’re taking ownership of
`args` and we’ll be mutating `args` by iterating over it, we can add the `mut`
keyword into the specification of the `args` parameter to make it mutable.

A documentação da biblioteca padrão para a função `env::args` mostra que o tipo de 
iterador retornado é `std::env::Args`. Atualizamos a assinatura da função `Config::new` 
para que o parâmetro `args` tenha o tipo `std::env::Args` em vez de `& [String]`. Como 
estamos assumindo o ownership de `args` e alterando (mutanting) o `args` ao iterá-lo, 
podemos adicionar a palavra-chave `mut` à especificação do parâmetro `args` para 
torná-lo mutável.

#### Using `Iterator` Trait Methods Instead of Indexing
#### Usando Métodos de Traits `Iterator` em vez de Indexação

Next, we’ll fix the body of `Config::new`. The standard library documentation
also mentions that `std::env::Args` implements the `Iterator` trait, so we know
we can call the `next` method on it! Listing 13-27 updates the code from
Listing 12-23 to use the `next` method:

Em seguida, corrigiremos o corpo de `Config::new`. A documentação da biblioteca padrão 
também menciona que `std::env::Args` implementa trait `Iterator`, para que possamos 
chamar o método `next`! A Lista 13-27 atualiza o código da Lista 12-23 para usar 
o método `next`:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
# use std::env;
#
# struct Config {
#     query: String,
#     filename: String,
#     case_sensitive: bool,
# }
#
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let filename = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file name"),
        };

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

<span class="caption">Listing 13-27: Changing the body of `Config::new` to use
iterator methods</span>

<span class="caption">Lista 13-27: Alterando o corpo de `Config::new` para usar 
os métodos do iterador</span>

Remember that the first value in the return value of `env::args` is the name of
the program. We want to ignore that and get to the next value, so first we call
`next` and do nothing with the return value. Second, we call `next` to get the
value we want to put in the `query` field of `Config`. If `next` returns a
`Some`, we use a `match` to extract the value. If it returns `None`, it means
not enough arguments were given and we return early with an `Err` value. We do
the same thing for the `filename` value.

Lembre-se de que o primeiro valor, no valor retornado de `env::args`, é o nome do 
programa. Queremos ignorar isso e chegar ao próximo valor, então primeiro chamamos 
`next` e não fazemos nada com o valor de retorno. Segundo, chamamos `next` para obter 
o valor que queremos colocar no campo `query` de `Config`. Se `next` retornar um `Some`, 
usamos um `match` para extrair o valor. Se retornar `None`, significa que não foram 
fornecidos argumentos suficientes e retornamos cedo com um valor `Err`. Fazemos o 
mesmo para o valor de `filename`.

### Making Code Clearer with Iterator Adaptors
### Tornando o Código mais Claro com Adaptadores de Interadores

We can also take advantage of iterators in the `search` function in our I/O
project, which is reproduced here in Listing 13-28 as it was in Listing 12-19:

Também podemos tirar proveito dos iteradores na função `search` em nosso projeto 
de E/S, que é reproduzido aqui na Listagem 13-28, como na Listagem 12-19:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

<span class="caption">Listing 13-28: The implementation of the `search`
function from Listing 12-19</span>

<span class="caption">Listagem 13-28: A implementação da função `search` 
da Listagem 12-19</span>

We can write this code in a more concise way using iterator adaptor methods.
Doing so also lets us avoid having a mutable intermediate `results` vector. The
functional programming style prefers to minimize the amount of mutable state to
make code clearer. Removing the mutable state might enable a future enhancement
to make searching happen in parallel, because we wouldn’t have to manage
concurrent access to the `results` vector. Listing 13-29 shows this change:

Podemos escrever esse código de maneira mais concisa usando métodos de adaptador 
de iterador. Fazer isso também evita a existência de um vetor intermediário mutável 
de `results`. O estilo de programação funcional prefere minimizar a quantidade de 
estado mutável para tornar o código mais claro. A remoção do estado mutável pode 
permitir um aprimoramento futuro para fazer a pesquisa acontecer em paralelo, porque 
não precisaríamos gerenciar o acesso simultâneo ao vetor `results`. A Listagem 13-29 
mostra essa mudança:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents.lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

<span class="caption">Listing 13-29: Using iterator adaptor methods in the
implementation of the `search` function</span>

<span class="caption">Lista 13-29: Usando métodos do adaptador iterador na 
implementação da função `search`</span>

Recall that the purpose of the `search` function is to return all lines in
`contents` that contain the `query`. Similar to the `filter` example in Listing
13-19, this code uses the `filter` adaptor to keep only the lines that
`line.contains(query)` returns `true` for. We then collect the matching lines
into another vector with `collect`. Much simpler! Feel free to make the same
change to use iterator methods in the `search_case_insensitive` function as
well.

Lembre-se de que o objetivo da função `search` é retornar todas as linhas em
`contents` que contém a `query`. Semelhante ao exemplo `filter` na Listagem 13-19, 
esse código usa o adaptador `filter` para manter apenas as linhas para as quais 
`line.contains(query)` retorna `true`. Em seguida, coletamos as linhas correspondentes 
em outro vetor com `collect`. Muito mais simples! Sinta-se à vontade para fazer a 
mesma alteração para usar os métodos do iterador na função `search_case_insensitive`.

The next logical question is which style you should choose in your own code and
why: the original implementation in Listing 13-28 or the version using
iterators in Listing 13-29. Most Rust programmers prefer to use the iterator
style. It’s a bit tougher to get the hang of at first, but once you get a feel
for the various iterator adaptors and what they do, iterators can be easier to
understand. Instead of fiddling with the various bits of looping and building
new vectors, the code focuses on the high-level objective of the loop. This
abstracts away some of the commonplace code so it’s easier to see the concepts
that are unique to this code, such as the filtering condition each element in
the iterator must pass.

A próxima pergunta lógica é qual estilo você deve escolher em seu próprio código 
e por quê: a implementação original na Listagem 13-28 ou a versão usando iteradores 
na Listagem 13-29. A maioria dos programadores Rust prefere usar o estilo do iterador. 
É um pouco mais difícil entender o que aconteceu no início, mas depois de ter uma idéia 
dos vários adaptadores de iteradores e o que eles fazem, os iteradores podem ser mais 
fáceis de entender. Em vez de mexer com os vários bits do loop e criar novos vetores, 
o código se concentra no objetivo de alto nível do loop. Isso abstrai parte do código 
comum, para que seja mais fácil ver os conceitos únicos desse código, como a condição 
de filtragem que cada elemento no iterador deve passar.

But are the two implementations truly equivalent? The intuitive assumption
might be that the more low-level loop will be faster. Let’s talk about
performance.

Mas as duas implementações são realmente equivalentes? A suposição intuitiva pode ser que 
o loop de mais baixo nível seja mais rápido. Vamos falar sobre desempenho.