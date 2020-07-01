## Melhorando Nosso Projeto de E/S

Com esse novo conhecimento sobre iteradores, podemos melhorar o projeto de 
E/S (I/O) no Capítulo 12 usando iteradores para tornar os locais no código mais 
claros e concisos. Vejamos como os iteradores podem melhorar nossa implementação 
da função `Config::new` e da função `search`.

### Removendo um `clone` Usando um Iterador

Na Listagem 12-6, adicionamos código que utilizou uma fatia (slice) dos valores `String` 
e criamos uma instância da estrutura `Config` indexando a fatia e clonando os valores, 
permitindo que a estrutura `Config` possua esses valores. Na Listagen 13-24, 
reproduzimos a implementação da função `Config::new` como na Listagem 12-23:

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

<span class="caption">Listagem 13-24: Reprodução da função `Config::new` 
da Listagem 12-23</span>

Na época, dissemos que não nos preocupávamos com as chamadas ineficientes do 
`clone` porque as removeríamos no futuro. Bem, essa hora é agora!

Precisávamos de `clone` aqui porque temos uma fatia com elementos `String` no 
parâmetro `args`, mas a função `new` não tem ownership `args`. Para retornar 
ownership de uma instância `Config`, tivemos que clonar os valores dos campos 
`query` e `filename` de `Config` para que a instância `Config` possa ter ownership 
sobre seus valores.

Com nosso novo conhecimento sobre iteradores, podemos alterar a função `new` 
para assumir ownership de um iterador como argumento, em vez de pegar uma 
fatia borrowed (emprestada). Usaremos a funcionalidade do iterador em vez do 
código que verifica o comprimento da fatia e os índices em locais específicos. 
Isso esclarecerá o que a função `Config::new` está fazendo, porque o iterador 
acessará os valores.

Uma vez que o `Config::new` possui ownership do iterador e para de usar as operações 
de indexação que borrow, podemos mover os valores `String` do iterador para `Config`, 
em vez de chamar `clone` e fazer uma nova alocação.

#### Usando o Iterador Retornado Diretamente

Abra o arquivo *src/main.rs*  do seu projeto de E/S, que deve se parecer 
com o seguinte:

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

Alteraremos o início da função `main` que tínhamos na Listagem 12-24 no código 
da Listagem 13-25. Isso não será compilado até que atualizemos o `Config::new` 
também.

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

<span class="caption">Lista 13-25: Passando o valor de retorno de `env::args` para 
`Config::new`</span>

A função `env::args` retorna um iterador! Em vez de coletar os valores do 
iterador em um vetor e, em seguida, passar uma fatia para `Config::new`, 
agora estamos passando ownership do iterador retornado de `env::args` para 
`Config::new` diretamente.

Em seguida, precisamos atualizar a definição de `Config::new`. No arquivo *src/lib.rs* 
do seu projeto de E/S, alteremos a assinatura do `Config::new` para parecer com a 
Listagem 13-26. Isso ainda não será compilado porque precisamos atualizar o corpo 
da função.

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        // --snip--
```

<span class="caption">Listagem 13-26: Atualizando a assinatura de `Config::new` 
para esperar um iterador</span>

A documentação da biblioteca padrão para a função `env::args` mostra que o tipo de 
iterador retornado é `std::env::Args`. Atualizamos a assinatura da função `Config::new` 
para que o parâmetro `args` tenha o tipo `std::env::Args` em vez de `& [String]`. Como 
estamos assumindo o ownership de `args` e alterando (mutanting) o `args` ao iterá-lo, 
podemos adicionar a palavra-chave `mut` à especificação do parâmetro `args` para 
torná-lo mutável.

#### Usando Métodos de Traits `Iterator` em vez de Indexação

Em seguida, corrigiremos o corpo de `Config::new`. A documentação da biblioteca padrão 
também menciona que `std::env::Args` implementa trait `Iterator`, para que possamos 
chamar o método `next`! A Lista 13-27 atualiza o código da Lista 12-23 para usar 
o método `next`:

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

<span class="caption">Lista 13-27: Alterando o corpo de `Config::new` para usar 
os métodos do iterador</span>

Lembre-se de que o primeiro valor, no valor retornado de `env::args`, é o nome do 
programa. Queremos ignorar isso e chegar ao próximo valor, então primeiro chamamos 
`next` e não fazemos nada com o valor de retorno. Segundo, chamamos `next` para obter 
o valor que queremos colocar no campo `query` de `Config`. Se `next` retornar um `Some`, 
usamos um `match` para extrair o valor. Se retornar `None`, significa que não foram 
fornecidos argumentos suficientes e retornamos cedo com um valor `Err`. Fazemos o 
mesmo para o valor de `filename`.

### Tornando o Código mais Claro com Adaptadores de Interadores

Também podemos tirar proveito dos iteradores na função `search` em nosso projeto 
de E/S, que é reproduzido aqui na Listagem 13-28, como na Listagem 12-19:

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

<span class="caption">Listagem 13-28: A implementação da função `search` 
da Listagem 12-19</span>

Podemos escrever esse código de maneira mais concisa usando métodos de adaptador 
de iterador. Fazer isso também evita a existência de um vetor intermediário mutável 
de `results`. O estilo de programação funcional prefere minimizar a quantidade de 
estado mutável para tornar o código mais claro. A remoção do estado mutável pode 
permitir um aprimoramento futuro para fazer a pesquisa acontecer em paralelo, porque 
não precisaríamos gerenciar o acesso simultâneo ao vetor `results`. A Listagem 13-29 
mostra essa mudança:

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents.lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

<span class="caption">Lista 13-29: Usando métodos do adaptador iterador na 
implementação da função `search`</span>

Lembre-se de que o objetivo da função `search` é retornar todas as linhas em
`contents` que contém a `query`. Semelhante ao exemplo `filter` na Listagem 13-19, 
esse código usa o adaptador `filter` para manter apenas as linhas para as quais 
`line.contains(query)` retorna `true`. Em seguida, coletamos as linhas correspondentes 
em outro vetor com `collect`. Muito mais simples! Sinta-se à vontade para fazer a 
mesma alteração para usar os métodos do iterador na função `search_case_insensitive`.

A próxima pergunta lógica é qual estilo você deve escolher em seu próprio código 
e por quê: a implementação original na Listagem 13-28 ou a versão usando iteradores 
na Listagem 13-29. A maioria dos programadores Rust prefere usar o estilo do iterador. 
É um pouco mais difícil entender o que aconteceu no início, mas depois de ter uma idéia 
dos vários adaptadores de iteradores e o que eles fazem, os iteradores podem ser mais 
fáceis de entender. Em vez de mexer com os vários bits do loop e criar novos vetores, 
o código se concentra no objetivo de alto nível do loop. Isso abstrai parte do código 
comum, para que seja mais fácil ver os conceitos únicos desse código, como a condição 
de filtragem que cada elemento no iterador deve passar.

Mas as duas implementações são realmente equivalentes? A suposição intuitiva pode ser que 
o loop de mais baixo nível seja mais rápido. Vamos falar sobre desempenho.