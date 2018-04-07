## Working with Environment Variables
## Trabalhando com Variáveis de Ambiente

We’ll improve `minigrep` by adding an extra feature: an option for
case-insensitive searching that the user can turn on via an environment
variable. We could make this feature a command line option and require that
users enter it each time they want it to apply, but instead we’ll use an
environment variable. Doing so allows our users to set the environment variable
once and have all their searches be case insensitive in that terminal session.

Melhoraremos `minigrep` adicionando um recurso extra: uma opção para
pesquisa insensível às letras maiúsculas ou minúsculas, que o usuário poderá ativar através de
variável de ambiente. Poderíamos fazer deste recurso uma opção de linha de comando e exigir que
os usuários entram cada vez que eles querem que ele se aplique, mas, em vez disso, usaremos um
variável de ambiente. Isso permite que nossos usuários estabeleçam a variável de ambiente
uma vez e todas as suas buscas são insensíveis às maiúsculas e minúsculas naquela sessão do terminal.

### Writing a Failing Test for the Case-Insensitive `search` Function
### Escrevendo um Teste de Falha para a Função `search` insensível a Maiúsculas e Minúsculas

We want to add a new `search_case_insensitive` function that we’ll call when
the environment variable is on. We’ll continue to follow the TDD process, so
the first step is again to write a failing test. We’ll add a new test for the
new `search_case_insensitive` function and rename our old test from
`one_result` to `case_sensitive` to clarify the differences between the two
tests, as shown in Listing 12-20:

Queremos adicionar uma nova função `search_case_insensitive` que chamaremos quando
a variável de ambiente estiver ativada. Seguiremos com o processo TDD, então
o primeiro passo é novamente escrever um teste de falha. Vamos adicionar um novo teste para a
nova função `search_case_insensitive` e renomear nosso antigo teste de
`one_result` para `case_sensitive` de forma a esclarecer as diferenças entre os dois
testes, conforme mostrado na Listagem 12-20:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn case_sensitive() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Duct tape.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }

    #[test]
    fn case_insensitive() {
        let query = "rUsT";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Trust me.";

        assert_eq!(
            vec!["Rust:", "Trust me."],
            search_case_insensitive(query, contents)
        );
    }
}
```

<span class="caption">Listing 12-20: Adding a new failing test for the
case-insensitive function we’re about to add</span>

<span class="caption">Listagem 12-20: Adicionando um novo teste de falha para a
função insensível à maiúsculas e minúsculas que estamos prestes a adicionar</span>


Note that we’ve edited the old test’s `contents` too. We’ve added a new line
with the text `“Duct tape”` using a capital D that shouldn’t match the query
“duct” when we’re searching in a case-sensitive manner. Changing the old test
in this way helps ensure that we don’t accidentally break the case-sensitive
search functionality that we’ve already implemented. This test should pass now
and should continue to pass as we work on the case-insensitive search.

Note que também editamos o `contents` do antigo teste. Adicionamos uma nova linha
com o texto `“Duct tape”` usando um D maiúsculo que não deve corresponder à consulta
“duct” quando procuramos de forma sensível à maiúsculas e minúsculas. Alterando o teste antigo
desta forma, ajuda a garantir que não quebramos acidentalmente a diferenciação de maiúsculas e minúsculas
na funcionalidade de pesquisa que já implementamos. Este teste deve passar agora
e deve continuar a passar enquanto trabalhamos na pesquisa insensível à maiúsculas e minúsculas.

The new test for the case-*insensitive* search uses “rUsT” as its query. In the
`search_case_insensitive` function we’re about to add, the query “rUsT” should
match the line containing “Rust:” with a capital R and also the line “Trust
me.” even though both have different casing than the query. This is our failing
test, and it will fail to compile because we haven’t yet defined the
`search_case_insensitive` function. Feel free to add a skeleton implementation
that always returns an empty vector, similar to the way we did for the `search`
function in Listing 12-16 to see the test compile and fail.

O novo teste para a pesquisa insensível usa “rUsT” para sua consulta. Na função
`search_case_insensitive` que estamos prestes a adicionar, a consulta “rUsT” deve
combinar a linha que contém “Rust:” com um R maiúsculo e também a linha “Trust
me.”, embora ambos tenham uma caixa (maiúsculas e minúsculas) diferente da consulta. Este é o nosso teste de
falha, e ele não compilará porque ainda não definimos a
função `search_case_insensitive`. Sinta-se livre para adicionar uma implementação
que sempre retorna um vetor vazio, semelhante à forma como fizemos para a função `search`
na Listagem 12-16 para ver a compilação e o teste falhar.

### Implementing the `search_case_insensitive` Function
### Implementando a Função `search_case_insensitive`

The `search_case_insensitive` function, shown in Listing 12-21, will be almost
the same as the `search` function. The only difference is that we’ll lowercase
the `query` and each `line` so whatever the case of the input arguments,
they’ll be the same case when we check whether the line contains the query:

A função `search_case_insensitive`, mostrada na Listagem 12-21, será quase
o mesmo que a função `search`. A única diferença é que vamos forçar minúsculas
para `query` e para cada `line`, qualquer que seja o caso dos argumentos de entrada,
eles serão sempre minúsculos quando verificamos se a linha contém a consulta:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}
```

<span class="caption">Listing 12-21: Defining the `search_case_insensitive`
function to lowercase the query and the line before comparing them</span>

<span class="caption">Listagem 12-21: Definindo a função `search_case_insensitive`
para forçar caixa baixa na consulta antes de compará-las</span>

First, we lowercase the `query` string and store it in a shadowed variable with
the same name. Calling `to_lowercase` on the query is necessary so no matter
whether the user’s query is “rust”, “RUST”, “Rust”, or “rUsT”, we’ll treat the
query as if it was “rust” and be insensitive to the case.

Primeiro, caixa baixa na string `query` e a armazenamos em uma variável sombreada com
o mesmo nome. Chamar `to_lowercase` na consulta é necessário, portanto, não importa
se a consulta do usuário é “rust”, “RUST”, “Rust”, ou “rUsT”, trataremos a
consulta como se fosse “rust” sendo insensível ao caso.

Note that `query` is now a `String` rather than a string slice, because calling
`to_lowercase` creates new data rather than referencing existing data. Say the
query is “rUsT”, as an example: that string slice doesn’t contain a lowercase
“u” or “t” for us to use, so we have to allocate a new `String` containing
“rust”. When we pass `query` as an argument to the `contains` method now, we
need to add an ampersand because the signature of `contains` is defined to take
a string slice.

Note que `query` é agora uma `String` ao invés de um fatia de string, porque chamar 
`to_lowercase` cria novos dados em vez de referenciar dados existentes. Suponha que
a consulta é “rUsT”, por exemplo: essa fatia de string não contém minúsculas
“u” ou “t” para nós usarmos, então temos que alocar uma nova `String` contendo
“rust”. Quando passamos `query` como um argumento para o método `contains` agora, nós
precisamos adicionar um ampersand (&) porque a assinatura de `contains` é definida para
uma fatia de string.

Next, we add a call to `to_lowercase` on each `line` before we check whether it
contains `query` to lowercase all characters. Now that we’ve converted `line`
and `query` to lowercase, we’ll find matches no matter what the case of the
query is.

Em seguida, adicionamos uma chamada a `to_lowercase` em cada `line` antes de verificarmos se
contém `query` para passar para caixa baixa em todos os caracteres. Agora que convertemos `line`
e `query` para letras minúsculas, encontraremos correspondências, não importa qual seja o caso da
consulta.

Let’s see if this implementation passes the tests:

Vamos ver se esta implementação passa nos testes:

```text
running 2 tests
test test::case_insensitive ... ok
test test::case_sensitive ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Great! They passed. Now, let’s call the new `search_case_insensitive` function
from the `run` function. First, we’ll add a configuration option to the
`Config` struct to switch between case-sensitive and case-insensitive search.
Adding this field will cause compiler errors since we aren’t initializing this
field anywhere yet:

Ótimo! Eles passaram. Agora, vamos chamar a nova função `search_case_insensitive`
da função `run`. Primeiro, adicionaremos uma opção de configuração ao
`Config` struct para alternar entre pesquisa sensível a maiúsculas e minúsculas.
Adicionar esse campo causará erros no compilador, já que não estamos inicializando
o campo em nenhum lugar:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
pub struct Config {
    pub query: String,
    pub filename: String,
    pub case_sensitive: bool,
}
```

Note that we added the `case_sensitive` field that holds a Boolean. Next, we
need the `run` function to check the `case_sensitive` field’s value and use
that to decide whether to call the `search` function or the
`search_case_insensitive` function, as shown in Listing 12-22. Note this still
won’t compile yet:

Note que nós adicionamos o campo `case_sensitive` que contém um Booleano. Em seguida nós
precisamos da função `run` para verificar o valor do campo `case_sensitive` e usá-la
para decidir se devemos chamar a função `search` ou a
função `search_case_insensitive`, conforme mostrado na Listagem 12-22. Note que isso ainda
não irá compilar ainda:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# use std::error::Error;
# use std::fs::File;
# use std::io::prelude::*;
#
# fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#      vec![]
# }
#
# fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#      vec![]
# }
#
# struct Config {
#     query: String,
#     filename: String,
#     case_sensitive: bool,
# }
#
pub fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    let results = if config.case_sensitive {
        search(&config.query, &contents)
    } else {
        search_case_insensitive(&config.query, &contents)
    };

    for line in results {
        println!("{}", line);
    }

    Ok(())
}
```

<span class="caption">Listing 12-22: Calling either `search` or
`search_case_insensitive` based on the value in `config.case_sensitive`</span>

<span class="caption">Listagem 12-22: Chamando  `search` ou
`search_case_insensitive` baseado no valor em `config.case_sensitive`</span>


Finally, we need to check for the environment variable. The functions for
working with environment variables are in the `env` module in the standard
library, so we want to bring that module into scope with a `use std::env;` line
at the top of *src/lib.rs*. Then we’ll use the `var` method from the `env`
module to check for an environment variable named `CASE_INSENSITIVE`, as shown
in Listing 12-23:

Finalmente, precisamos verificar a variável de ambiente. As funções para
trabalhar com variáveis de ambiente estão no módulo `env` na biblioteca padrão
, por isso queremos trazer esse módulo para o escopo com uma linha `use std::env;`
no topo de *src/lib.rs*. Então vamos usar o método `var` do módulo `env`
para verificar uma variável de ambiente chamada `CASE_INSENSITIVE`, conforme
na Listagem 12-23:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
use std::env;
# struct Config {
#     query: String,
#     filename: String,
#     case_sensitive: bool,
# }

// --snip--

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

<span class="caption">Listing 12-23: Checking for an environment variable named
`CASE_INSENSITIVE`</span>

<span class="caption">Listagem 12-23: Checando por uma variável de ambiente chamada
`CASE_INSENSITIVE`</span>


Here, we create a new variable `case_sensitive`. To set its value, we call the
`env::var` function and pass it the name of the `CASE_INSENSITIVE` environment
variable. The `env::var` method returns a `Result` that will be the successful
`Ok` variant that contains the value of the environment variable if the
environment variable is set. It will return the `Err` variant if the
environment variable is not set.

Aqui, criamos uma nova variável `case_sensitive`. Para definir seu valor, chamamos a
função `env::var` e passamos o nome da variável de ambiente `CASE_INSENSITIVE`
. O método `env::var` retorna um `Result` que será o sucesso
variante `Ok` que contém o valor da variável de ambiente se a
variável de ambiente está definida. Ele retornará a variante `Err` se a
variável de ambiente não está definida.

We’re using the `is_err` method on the `Result` to check whether it’s an error
and therefore unset, which means it *should* do a case-sensitive search. If the
`CASE_INSENSITIVE` environment variable is set to anything, `is_err` will
return false and will perform a case-insensitive search. We don’t care about
the *value* of the environment variable, just whether it’s set or unset, so
we’re checking `is_err` rather than `unwrap`, `expect`, or any of the other
methods we’ve seen on `Result`.

Estamos usando o método `is_err` no `Result` para verificar se é um erro
e, portanto, não definido, o que significa que *deveria* fazer uma pesquisa sensível a maiúsculas e minúsculas. Se 
a variável de ambiente `CASE_INSENSITIVE` está configurada para qualquer coisa,`is_err` irá
retornar false e realizará uma pesquisa sem distinção entre maiúsculas e minúsculas. Nós não nos importamos com
o *valor* da variável de ambiente, apenas se está definido ou não,
estamos verificando `is_err` em vez de `unwrap`, `expect` ou qualquer um dos outros
métodos que vimos em `Result`.

We pass the value in the `case_sensitive` variable to the `Config` instance so
the `run` function can read that value and decide whether to call `search` or
`search_case_insensitive` as we implemented in Listing 12-22.

Nós passamos o valor na variável `case_sensitive` para a instância `Config`
na função `run` pode ler esse valor e decidir se deve chamar `search` ou
`search_case_insensitive` conforme implementamos na Listagem 12-22.

Let’s give it a try! First, we’ll run our program without the environment
variable set and with the query “to”, which should match any line that contains
the word “to” in all lowercase:

Vamos tentar! Primeiro, executaremos nosso programa sem o conjunto de variáveis 
de ambiente e com a consulta “to”, que deve corresponder a qualquer linha que contenha
a palavra “to” em todas as letras minúsculas:


```text
$ cargo run to poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep to poem.txt`
Are you nobody, too?
How dreary to be somebody!
```

Looks like that still works! Now, let’s run the program with `CASE_INSENSITIVE`
set to `1` but with the same query “to”; we should get lines that contain “to”
that might have uppercase letters:

Parece que isso ainda funciona! Agora, vamos executar o programa com `CASE_INSENSITIVE`
definido como `1` mas com a mesma consulta “to”; devemos pegar linhas que contenham “to”
que possam ter letras maiúsculas:

```text
$ CASE_INSENSITIVE=1 cargo run to poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep to poem.txt`
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

If you’re using PowerShell, you will need to set the environment variable and
run the program in two commands rather than one:

Se você estiver usando o PowerShell, precisará definir a variável de ambiente e
executar o programa em dois comandos em vez de um:

```text
$ $env.CASE_INSENSITIVE=1
$ cargo run to poem.txt
```

Excellent, we also got lines containing “To”! Our `minigrep` program can now do
case-insensitive searching controlled by an environment variable. Now you know
how to manage options set using either command line arguments or environment
variables!

Excelente, também temos linhas contendo “To”! Nosso programa `minigrep` agora pode fazer
busca insensível a maiúsculas e minúsculas controlada por uma variável de ambiente. Agora você 
sabe como gerenciar as opções definidas usando argumentos de linha de comando ou variáveis de ambiente!

Some programs allow arguments *and* environment variables for the same
configuration. In those cases, the programs decide that one or the other takes
precedence. For another exercise on your own, try controlling case
insensitivity through either a command line argument or an environment
variable. Decide whether the command line argument or the environment variable
should take precedence if the program is run with one set to case sensitive and
one set to case insensitive.

Alguns programas permitem argumentos *and* variáveis de ambiente para a mesma
configuração. Nesses casos, os programas decidem que um ou outro tenham
precedência. Para outro exercício por conta própria, tente controlar o caso
insensibilidade através de um argumento de linha de comando ou uma variável de ambiente
. Decida se o argumento da linha de comando ou a variável de ambiente
deve ter precedência se o programa for executado com um conjunto para diferenciação de maiúsculas e minúsculas
ou um conjunto para maiúsculas e minúsculas insensível.

The `std::env` module contains many more useful features for dealing with
environment variables: check out its documentation to see what is available.

O módulo `std::env` contém muitos mais recursos úteis para lidar com
variáveis de ambiente: confira sua documentação para ver o que está disponível.
