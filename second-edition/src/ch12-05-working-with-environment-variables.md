## Trabalhando com Variáveis de Ambiente

Melhoraremos `minigrep` adicionando um recurso extra: uma opção para
pesquisa insensível às letras maiúsculas ou minúsculas, que o usuário poderá ativar através de
variável de ambiente. Poderíamos fazer deste recurso uma opção de linha de comando e exigir que
os usuários entram cada vez que eles querem que ele se aplique, mas, em vez disso, usaremos um
variável de ambiente. Isso permite que nossos usuários estabeleçam a variável de ambiente
uma vez e todas as suas buscas são insensíveis às maiúsculas e minúsculas naquela sessão do terminal.

### Escrevendo um Teste de Falha para a Função `search` insensível a Maiúsculas e Minúsculas

Queremos adicionar uma nova função `search_case_insensitive` que chamaremos quando a variável 
de ambiente estiver ativada. Seguiremos com o processo TDD, então o primeiro passo é novamente 
escrever um teste de falha. Vamos adicionar um novo teste para a nova função `search_case_insensitive` 
e renomear nosso antigo teste de `one_result` para `case_sensitive` de forma a esclarecer as 
diferenças entre os dois testes, conforme mostrado na Listagem 12-20:

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

<span class="caption">Listagem 12-20: Adicionando um novo teste de falha para a
função insensível à maiúsculas e minúsculas que estamos prestes a adicionar</span>

Note que também editamos o `contents` do antigo teste. Adicionamos uma nova linha com o 
texto `“Duct tape”` usando um D maiúsculo que não deve corresponder à consulta “duct” quando 
procuramos de forma sensível à maiúsculas e minúsculas. Alterando o teste antigo desta forma, 
ajuda a garantir que não quebramos acidentalmente a diferenciação de maiúsculas e minúsculas
na funcionalidade de pesquisa que já implementamos. Este teste deve passar agora e deve continuar 
a passar enquanto trabalhamos na pesquisa insensível à maiúsculas e minúsculas.

O novo teste para a pesquisa insensível usa “rUsT” para sua consulta. Na função `search_case_insensitive` 
que estamos prestes a adicionar, a consulta “rUsT” deve combinar a linha que contém “Rust:” 
com um R maiúsculo e também a linha “Trust me.”, embora ambos tenham uma caixa (maiúsculas e 
minúsculas) diferente da consulta. Este é o nosso teste de falha, e ele não compilará porque 
ainda não definimos a função `search_case_insensitive`. Sinta-se livre para adicionar uma 
implementação que sempre retorna um vetor vazio, semelhante à forma como fizemos para a função 
`search` na Listagem 12-16 para ver a compilação e o teste falhar.

### Implementando a Função `search_case_insensitive`

A função `search_case_insensitive`, mostrada na Listagem 12-21, será quase o mesmo que a função 
`search`. A única diferença é que vamos forçar minúsculas para `query` e para cada `line`, 
qualquer que seja o caso dos argumentos de entrada, eles serão sempre minúsculos quando verificamos 
se a linha contém a consulta:

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

<span class="caption">Listagem 12-21: Definindo a função `search_case_insensitive`
para forçar caixa baixa na consulta antes de compará-las</span>

Primeiro, caixa baixa na string `query` e a armazenamos em uma variável sombreada com
o mesmo nome. Chamar `to_lowercase` na consulta é necessário, portanto, não importa
se a consulta do usuário é “rust”, “RUST”, “Rust”, ou “rUsT”, trataremos a
consulta como se fosse “rust” sendo insensível ao caso.

Note que `query` é agora uma `String` ao invés de um fatia de string, porque chamar 
`to_lowercase` cria novos dados em vez de referenciar dados existentes. Suponha que
a consulta é “rUsT”, por exemplo: essa fatia de string não contém minúsculas
“u” ou “t” para nós usarmos, então temos que alocar uma nova `String` contendo
“rust”. Quando passamos `query` como um argumento para o método `contains` agora, nós
precisamos adicionar um ampersand (&) porque a assinatura de `contains` é definida para
uma fatia de string.

Em seguida, adicionamos uma chamada a `to_lowercase` em cada `line` antes de verificarmos se
contém `query` para passar para caixa baixa em todos os caracteres. Agora que convertemos `line`
e `query` para letras minúsculas, encontraremos correspondências, não importa qual seja o caso da
consulta.

Vamos ver se esta implementação passa nos testes:

```text
running 2 tests
test test::case_insensitive ... ok
test test::case_sensitive ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Ótimo! Eles passaram. Agora, vamos chamar a nova função `search_case_insensitive`
da função `run`. Primeiro, adicionaremos uma opção de configuração ao
`Config` struct para alternar entre pesquisa sensível a maiúsculas e minúsculas.
Adicionar esse campo causará erros no compilador, já que não estamos inicializando
o campo em nenhum lugar:

<span class="filename">Arquivo: src/lib.rs</span>

```rust
pub struct Config {
    pub query: String,
    pub filename: String,
    pub case_sensitive: bool,
}
```

Note que nós adicionamos o campo `case_sensitive` que contém um Booleano. Em seguida nós
precisamos da função `run` para verificar o valor do campo `case_sensitive` e usá-la
para decidir se devemos chamar a função `search` ou a 
função `search_case_insensitive`, conforme mostrado na Listagem 12-22. Note que isso ainda
não irá compilar ainda:

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

<span class="caption">Listagem 12-22: Chamando  `search` ou
`search_case_insensitive` baseado no valor em `config.case_sensitive`</span>

Finalmente, precisamos verificar a variável de ambiente. As funções para
trabalhar com variáveis de ambiente estão no módulo `env` na biblioteca padrão
, por isso queremos trazer esse módulo para o escopo com uma linha `use std::env;`
no topo de *src/lib.rs*. Então vamos usar o método `var` do módulo `env`
para verificar uma variável de ambiente chamada `CASE_INSENSITIVE`, conforme
na Listagem 12-23:

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

<span class="caption">Listagem 12-23: Checando por uma variável de ambiente chamada
`CASE_INSENSITIVE`</span>

Aqui, criamos uma nova variável `case_sensitive`. Para definir seu valor, chamamos a
função `env::var` e passamos o nome da variável de ambiente `CASE_INSENSITIVE`
. O método `env::var` retorna um `Result` que será o sucesso
variante `Ok` que contém o valor da variável de ambiente se a
variável de ambiente está definida. Ele retornará a variante `Err` se a
variável de ambiente não está definida.

Estamos usando o método `is_err` no `Result` para verificar se é um erro
e, portanto, não definido, o que significa que *deveria* fazer uma pesquisa sensível a maiúsculas e minúsculas. Se 
a variável de ambiente `CASE_INSENSITIVE` está configurada para qualquer coisa,`is_err` irá
retornar false e realizará uma pesquisa sem distinção entre maiúsculas e minúsculas. Nós não nos importamos com
o *valor* da variável de ambiente, apenas se está definido ou não,
estamos verificando `is_err` em vez de `unwrap`, `expect` ou qualquer um dos outros
métodos que vimos em `Result`.

Nós passamos o valor na variável `case_sensitive` para a instância `Config`
na função `run` pode ler esse valor e decidir se deve chamar `search` ou
`search_case_insensitive` conforme implementamos na Listagem 12-22.

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

Se você estiver usando o PowerShell, precisará definir a variável de ambiente e
executar o programa em dois comandos em vez de um:

```text
$ $env.CASE_INSENSITIVE=1
$ cargo run to poem.txt
```

Excelente, também temos linhas contendo “To”! Nosso programa `minigrep` agora pode fazer
busca insensível a maiúsculas e minúsculas controlada por uma variável de ambiente. Agora você 
sabe como gerenciar as opções definidas usando argumentos de linha de comando ou variáveis de ambiente!

Alguns programas permitem argumentos *and* variáveis de ambiente para a mesma
configuração. Nesses casos, os programas decidem que um ou outro tenham
precedência. Para outro exercício por conta própria, tente controlar o caso
insensibilidade através de um argumento de linha de comando ou uma variável de ambiente
. Decida se o argumento da linha de comando ou a variável de ambiente
deve ter precedência se o programa for executado com um conjunto para diferenciação de maiúsculas e minúsculas
ou um conjunto para maiúsculas e minúsculas insensível.

O módulo `std::env` contém muitos mais recursos úteis para lidar com
variáveis de ambiente: confira sua documentação para ver o que está disponível.
