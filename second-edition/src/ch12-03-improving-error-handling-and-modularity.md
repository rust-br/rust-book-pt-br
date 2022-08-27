## Refatoração para Melhorar a Modularidade e o Tratamento de Erros

Para melhorar o nosso programa, repararemos quatro problemas que têm a ver com a
estrutura do programa e como ele está tratando possíveis erros.

Primeiro, a nossa função `main` agora executa duas tarefas: analisa argumentos e
abre arquivos. Para uma função tão pequena, este não é um grande problema. No entanto, se
continuamos a desenvolver o nosso programa dentro de `main`, o número de tarefas separadas que
a função `main` manipula aumentarão. Com uma função ganhando responsabilidades,
torna-se mais difícil de raciocinar, mais difícil de testar e mais difícil de mudar
sem quebrar uma das suas partes. É melhor separar a funcionalidade para que cada
função seja responsável por uma tarefa.

Esta questão também se liga ao segundo problema: embora `query` e `filename`
sejam variáveis de configuração para o nosso programa, variáveis como `f` e `contents`
são usadas para executar a lógica do programa. Quanto maior o `main` se torna, mais
variáveis precisamos trazer no escopo; quanto mais variáveis temos no escopo,
mais difícil será acompanhar o objetivo de cada uma. É melhor agrupar
as variáveis de configuração em uma estrutura para tornar claro seu objetivo.

O terceiro problema é que usamos `expect` para imprimir uma mensagem de erro, ao
abrir um arquivo, falha, mas a mensagem de erro apenas imprime `file not found`.
Abrir um arquivo pode falhar de várias maneiras, além do arquivo faltando: como
exemplo, o arquivo pode existir, mas talvez não possamos ter permissão para abri-lo.
Agora, se estivermos nessa situação, imprimiríamos a mensagem de erro `file not found`
que daria ao usuário a informação errada!

O quarto problema, usamos `expect` repetidamente para lidar com diferentes erros, e se o usuário
executa o nosso programa sem especificar argumentos suficientes, eles terão erros `index out
of bounds` do Rust, que não explica claramente o problema. Seria
melhor se todo o código de tratamento de erros estiver em um só lugar para futuros mantenedores
terem apenas um lugar para consultar, no código, se a lógica de tratamento de erros precisar de
mudança. Ter todo o código de tratamento de erros em um só lugar também assegurará que
estamos imprimindo mensagens que serão significativas para nossos usuários finais.

Vamos abordar esses quatro problemas refatorando nosso projeto.

### Separação de Responsabilidades para Projetos Binários

O problema organizacional da atribuição de responsabilidade por múltiplas tarefas para
a função `main` é comum a muitos projetos binários. Como resultado, a comunidade Rust
desenvolveu um tipo de processo de orientação para dividir as
responsabilidades de um programa binário quando `main` começa a ficar grande. O processo tem
as seguintes etapas:

* Divida seu programa em um *main.rs* e um *lib.rs*, e mova a lógica
do seu programa para *lib.rs*.
* Enquanto sua lógica de análise de linha de comando é pequena, ela pode permanecer em *main.rs*.
* Quando a lógica de análise de linha de comando começa a ficar complicada, extraia
de *main.rs* e mova para *lib.rs*.
* As responsabilidades que permanecem na função `main` depois desse processo
deve estar limitado a:

  * Chamar a lógica de análise de linha de comando com os valores do argumento
  * Ajustar qualquer outra configuração
  * Chamando uma função `run` em *lib.rs*
  * Manipulação do erro se `run` retornar um erro  

Esse padrão é sobre separar responsabilidades: *main.rs* lida com a execução do
programa e *lib.rs* lida com toda a lógica da tarefa em questão. Porque nós
não podemos testar diretamente a função `main`, esta estrutura nos permite testar toda
lógica do programa, movendo-a para funções em *lib.rs*. O único código que
permanece em *main.rs* será pequeno o suficiente para verificar se está correto com 
uma leitura rápida. Vamos retrabalhar o nosso programa seguindo este processo.

#### Extraindo o Parseador de Argumento

Vamos extrair a funcionalidade de análise de argumentos de `main` para *src/lib.rs*. 
A listagem 12-5 mostra o novo início do `main` que chama uma nova
função `parse_config`, que iremos definir em *src/main.rs* por enquanto.

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let (query, filename) = parse_config(&args);

    // --snip--
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let filename = &args[2];

    (query, filename)
}
```

<span class="caption">Listagem 12-5: Extraindo uma função `parse_config` de
`main`</span>

Ainda estamos coletando os argumentos da linha de comando em um vetor, mas em vez de
atribuir o valor do argumento no índice `1` para a variável `query` e o
valor do argumento no índice `2` para a variável `filename` dentro da função `main`
, passamos todo o vetor para a função `parse_config`. A
função `parse_config` mantém a lógica que determina qual argumento
vai em qual variável e passa os valores de volta para `main`. Ainda criamos
as variáveis `query` e `filename` no `main`, mas `main` não tem mais a
responsabilidade de determinar como os argumentos e as variáveis da linha de comando
correspondem.

Essa retrabalho pode parecer um exagero para o nosso pequeno programa, mas estamos refatorando
em pequenos passos incrementais. Depois de fazer essa alteração, execute o programa novamente para
verificar se a análise do argumento ainda funciona. É bom verificar seu progresso
constantemente, porque isso irá ajudá-lo a identificar a causa dos problemas quando eles
ocorrerem.

#### Agrupando Valores de Configuração

Podemos dar outro pequeno passo para melhorar ainda mais a função `parse_config`.
No momento, estamos retornando uma tupla, mas depois quebramos imediatamente a tupla
em partes individuais novamente. Este é um sinal de que talvez não tenhamos
a abstração certa ainda.

Outro indicador que mostra que há espaço para melhoria é a parte `config`
de `parse_config`, o que implica que os dois valores que retornamos estão relacionados e
ambos são parte de um valor de configuração. Atualmente, não estamos transmitindo esse
significado na estrutura dos dados, que não sejam o agrupamento dos dois valores em um
tupla: podemos colocar os dois valores em uma estrutura e dar a cada uma das estruturas
um nome significativo. Isso facilitará os futuros mantenedores
deste código para entender como os diferentes valores se relacionam entre si e
qual é o propósito deles.

> Nota: algumas pessoas chamam este anti-padrão de usar valores primitivos quando um
> tipo complexo seria mais apropriado *primitive obsession* (obsessão primitiva).

A Listagem 12-6 mostra a adição de uma estrutura chamada `Config` definida para ter
campos chamados `query` e `filename`. Também mudamos a função `parse_config`
para retornar uma instância da estrutura `Config` e atualizamos `main` para usar
os campos struct em vez de ter variáveis separadas:

<span class="filename">Arquivo: src/main.rs</span>

```rust,should_panic
# use std::env;
# use std::fs::File;
#
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = parse_config(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    let mut f = File::open(config.filename).expect("file not found");

    // --snip--
}

struct Config {
    query: String,
    filename: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let filename = args[2].clone();

    Config { query, filename }
}
```

<span class="caption">Listagem 12-6: Refatorando `parse_config` para retornar uma
instância de uma struct `Config`</span>

A assinatura do `parse_config` agora indica que ele retorna um valor `Config`.
No corpo de `parse_config`, onde costumávamos retornar trechos de strings com
referência a valores `String` em `args`, agora definimos `Config` para conter valores
`String` owned. A variável `args` em `main` é o owner do argumento de valores e está 
apenas permitindo que a função `parse_config` os empreste (borrow), o que significa
que violaremos as regras de borrow do Rust se o `Config` tentasse se apropriar (ownership) dos
valores em `args`.

Podemos gerenciar os dados `String` de várias maneiras diferentes, mas o
mais fácil, embora ineficiente, o caminho é chamar o método `clone` nos
valores. Isso fará uma cópia completa dos dados para a instância `Config` 
, que leva mais tempo e memória do que armazenar uma referência à string de 
dados. No entanto, a clonagem dos dados também torna nosso código muito direto
porque não precisamos administrar as vidas das referências; nessa circunstância, 
desistir de um pouco de desempenho para ganhar simplicidade é uma coisa que vale 
a pena a troca.

> ### Os Prós e Contras de Usar `clone`
>
> Existe uma tendência entre muitos Rustaceos para evitar o uso de `clone` para consertar
> problemas de ownership devido ao seu custo de tempo de execução. No Capítulo 13, 
> você aprenderá como usar métodos mais eficientes neste tipo de situação. Mas por agora,
> é bom copiar algumas strings para continuar a fazer progresso porque iremos
> fazer essas cópias apenas uma vez, e nosso nome de arquivo e seqüência de consulta são muito
> pequenos. É melhor ter um programa de trabalho que seja um pouco ineficiente do que
> tentar hiper-optimizar o código na sua primeira passagem. À medida que você se torna mais experiente
> com Rust, será mais fácil começar com a solução mais eficiente, mas para
> agora, é perfeitamente aceitável chamar `clone`.

Atualizamos `main` para que ele coloque a instância de `Config` retornada por
`parse_config` em uma variável chamada `config`, e atualizamos o código
anteriormente usado para as variáveis separadas `query` e `filename` para que ele agora
,em vez disso, use os campos na estrutura `Config`.

Agora, nosso código transmite mais claramente que `query` e `filename` estão relacionados, e
seu objetivo é configurar como o programa funcionará. Qualquer código que use
esses valores sabem encontrá-los na instância `config` nos campos nomeados
para esse propósito.

#### Criando um Construtor para `Config`

Até agora, nós extraímos a lógica responsável por analisar os argumentos da linha de 
comando de `main` e colocá-los na função `parse_config`, o que nos ajudou a ver que os 
valores `query` e `filename` estavam relacionados e essa relação deve ser transmitida 
em nosso código. Nós então adicionamos uma estrutura `Config` para nomear o propósito 
relacionado de `query` e `filename`, e para poder retornar os nomes dos valores como 
nomes de campos struct a partir da função `parse_config`.

Então, agora que a finalidade da função `parse_config` é criar uma instância `Config`
, podemos alterar `parse_config` de ser uma função simples para um função denominada 
`new` que está associada à estrutura `Config`. Fazendo essa mudança tornará o código 
mais idiomático: podemos criar instâncias de tipos na biblioteca padrão, como `String`, 
chamando `String::new`, e mudando `parse_config` para uma função `new` associada a 
`Config`, iremos ser capazes de criar instâncias de `Config` chamando `Config::new`. 
Listagem 12-7 mostra as mudanças que precisamos fazer:

<span class="filename">Arquivo: src/main.rs</span>

```rust,should_panic
# use std::env;
#
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args);

    // --snip--
}

# struct Config {
#     query: String,
#     filename: String,
# }
#
// --snip--

impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let filename = args[2].clone();

        Config { query, filename }
    }
}
```

<span class="caption">Listagem 12-7: Alterar `parse_config` para
`Config::new`</span>

Atualizamos `main` onde estávamos chamando `parse_config` para, em vez disso, chamar
`Config::new`. Alteramos o nome de `parse_config` para `new` e movemos para
dentro de um bloco `impl`, que associa a função `new` a `Config`. Experimente
compilar este código novamente para garantir que ele funciona.

### Consertando o Tratamento de Erros

Agora vamos trabalhar em consertar o nosso tratamento de erros. Lembre-se de que tentar acessar
os valores no vetor `args` no índice `1` ou no índice `2` causará pânico no programa se 
o vetor contiver menos de três itens. Tente executar o programa sem argumentos; Isso parecerá assim:

```text
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep`
thread 'main' panicked at 'index out of bounds: the len is 1
but the index is 1', src/main.rs:29:21
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

A linha `index out of bounds: the len is 1 but the index is 1` é uma mensagem de erro
destinada aos programadores. Isso não ajudará os usuários finais a entender o que
aconteceu e o que eles deveriam fazer a respeito disso. Vamos consertar isso agora.

#### Melhorando a Mensagem de Erro

Na Listagem 12-8, adicionamos uma verificação na função `new` que verificará que o
pedaço é longo o suficiente antes de acessar os índices `1` e `2`. Se o pedaço não for
suficientemente longo, o programa gera um pânico e exibe uma mensagem de erro melhor do que a
mensagem `index out of bounds`:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
// --snip--
fn new(args: &[String]) -> Config {
    if args.len() < 3 {
        panic!("not enough arguments");
    }
    // --snip--
```

<span class="caption">Listagem 12-8: Adicionando uma verificação para o número de
argumentos</span>

Este código é semelhante à função `Guess::new` que escrevemos na Listagem 9-9 onde
chamamos `panic!` quando o argumento `value` estava fora do alcance válido de
valores. Em vez de verificar uma variedade de valores aqui, estamos checando que o
comprimento de `args` é pelo menos `3` e o resto da função pode operar sob
o pressuposto de que essa condição foi cumprida. Se `args` tiver menos de três
itens, essa condição será verdadeira, e chamamos a macro `panic!` para terminar o
programa imediatamente.

Com estas poucas linhas de código adicionais em `new`, vamos executar o programa sem nenhum
argumento novamente para ver como o erro parece agora:

```text
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep`
thread 'main' panicked at 'not enough arguments', src/main.rs:30:12
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Este resultado é melhor: agora temos uma mensagem de erro razoável. No entanto, nós também
temos informações estranhas que não queremos dar aos nossos usuários. Talvez usando
a técnica que usamos na Lista 9-9 não é a melhor para usar aqui: uma chamada para
`panic!` é mais apropriado para um problema de programação e não um problema de uso
, conforme discutido no Capítulo 9. Em vez disso, podemos usar outra técnica que você
aprendeu no Capítulo 9 - retornando um `Result` que indica sucesso
ou um erro.

#### Retornando um `Result` de um `new` Em vez de Chamar `panic!`

Em vez disso, podemos retornar um valor `Result` que conterá uma instância `Config` em
caso bem-sucedido e descreverá o problema no caso de erro. Quando `Config::new` está se 
comunicando com `main`, podemos usar o tipo `Result` para sinalizar que não houve problema. 
Então podemos mudar `main` para converter uma variante `Err` em um erro mais prático para 
os nossos usuários sem os demais textos sobre `thread 'main'` e `RUST_BACKTRACE` que uma 
chamada para `panic!` causa.

A Listagem 12-9 mostra as mudanças que precisamos fazer para o valor de retorno de
`Config::new` e o corpo da função necessária para retornar um `Result`. Note
que isso não compilará até que atualizemos `main` também, o que faremos na
próxima listagem:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        Ok(Config { query, filename })
    }
}
```

<span class="caption">Listagem 12-9: Retornando um `Result` de `Config::new`</span>

Nossa função `new` agora retorna um` Result` com uma instância `Config` no caso de sucesso 
e um `&'static str` no caso de erro. Lembre-se da seção “The Static Lifetime” no capítulo 10 
que `& 'static str` é o tipo de string literal, que é o nosso tipo de mensagem de erro por enquanto.

Fizemos duas mudanças no corpo da função `new`: em vez de chamar `panic!` quando o 
usuário não passa argumentos suficientes, agora devolvemos um valor `Err`
, e nós wrapped (embalamos) o valor de retorno `Config` em um `Ok`. Estas alterações
fazem com que a função esteja conforme a sua nova assinatura de tipo.

Retornar um valor `Err` de `Config::new` permite que a função `main` lide com o valor `Result` 
retornado da função `new` e saia do processo de forma mais limpa no caso de erro.

#### Chamando `Config::new` e Manipulação de Erros

Para lidar com o caso de erro e imprimir uma mensagem amigável, precisamos atualizar
`main` para lidar com o `Result` sendo retornado por `Config::new`, conforme mostrado na
Listagem 12-10. Também assumiremos a responsabilidade de sair da linha de comando
com um código de erro diferente de zero do `panic!` e implementá-lo manualmente.
O status de saída diferente de zero, é uma convenção para sinalizar o processo que chamou nosso
programa que, o programa saiu com um estado de erro.

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
```

<span class="caption">Listagem 12-10: Se ao criar um `Config` falha, saimos com um 
código de erro</span>

Nesta lista, usamos um método que não abordamos antes:
`unwrap_or_else`, que está definido em `Result <T, E>` pela biblioteca padrão.
Usar `unwrap_or_else` nos permite definir algum erro personalizado, não-`panic!` de
manipulação. Se o `Result` for um valor `Ok`, o comportamento deste método é semelhante
a `unwrap`: ele retorna o valor interno `Ok`. No entanto, se o valor é um valor `Err`, 
este método chama o código na *closure*, que é uma função anônima que definimos e 
passamos como um argumento para `unwrap_or_else`. Nós entraremos em detalhes sobre 
closures no Capítulo 13. Por enquanto, você precisa apenas saber que `unwrap_or_else` 
passará o valor interno do `Err`, que neste caso é a string estática `not enough arguments` 
que adicionamos na Listagem 12-9, para o nosso closure no argumento `err` que aparece 
entre os pipes verticais. O código no closure pode então usar o valor `err` quando ele é executado.

Adicionamos uma nova linha de `use` para importar `process` da biblioteca padrão.
O código na closure que será executado no caso de erro são apenas duas linhas: nós
imprimos o valor de `err` e depois chamamos `process::exit`. A função `process::exit`
interromperá o programa imediatamente e retornará o número que foi passado como o código 
de status de saída. Isso é semelhante ao manuseio baseado no `panic!` que usamos na Listagem 
12-8, mas já não obtemos todos os resultados extras. Vamos tentar isto:

```text
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48 secs
     Running `target/debug/minigrep`
Problem parsing arguments: not enough arguments
```

Ótimo! Este resultado é muito mais amigável para os nossos usuários.

### Extraindo a Lógica do `main`

Agora que terminamos de refatorar a análise da configuração, voltemos a lógica do programa. 
Como afirmamos em “Separação de Responsabilidades para Projetos Binários”, vamos extrair 
uma função chamada `run` que irá armazenar toda a lógica atualmente na função `main` que 
não está envolvida com a configuração ou manipulação de erros. Quando terminarmos, `main` 
será conciso e fácil de verificar por inspeção, e poderemos fazer testes para todas as
outras lógicas.

Listagem 12-11 mostra a função extraída `run`. Por enquanto, estamos apenas fazendo
a pequena melhoria incremental da extração da função. Ainda estamos definindo a função 
em *src/main.rs*:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    run(config);
}

fn run(config: Config) {
    let mut f = File::open(config.filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");

    println!("With text:\n{}", contents);
}

// --snip--
```

<span class="caption">Listagem 12-11: Extraindo uma função `run` contendo o
resto da lógica do programa</span>

A função `run` agora contém toda a lógica restante de `main`, começando por ler o arquivo. 
A função `run` pega a instância `Config` como um argumento.

#### Retornando Erros da Função `run`

Com a lógica do programa restante separada na função `run`, podemos melhorar o tratamento 
de erros, como fizemos com `Config::new` na Listagem 12-9. Em vez de permitir que o programa 
entre em pânico ao chamar `expect`, a função `run` retornará um `Result<T, E>` quando 
algo der errado. Isso nos permitirá consolidar ainda mais na lógica principal 
a manipulação de erros em uma maneira fácil de usar. A Listagem 12-12 mostra as mudanças 
que precisamos fazer para a assinatura e corpo de `run`:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
use std::error::Error;

// --snip--

fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    println!("With text:\n{}", contents);

    Ok(())
}
```

<span class="caption">Listagem 12-12: Alterar a função `run` para retornar
`Result`</span>

Nós fizemos três mudanças significativas aqui. Primeiro, mudamos o tipo de retorno
da função `run` para `Result<(), Box<Error>>`. Esta função anteriormente
devolveu o tipo de unidade, `()`, e nós mantemos isso como o valor retornado `Ok` no
caso.

Para o tipo de erro, usamos o *trait object* `Box<Error>` (e nós trouxemos
`std::error::Error` no escopo com uma instrução `use` na parte superior). Vamos cobrir
objetos trait no Capítulo 17. Por enquanto, apenas saiba que `Box<Error>` significa que
a função retornará um tipo que implemente o trait `Error`, mas não temos que especificar 
qual tipo em particular o valor de retorno será. Isso nos dá flexibilidade para retornar 
valores de erro que podem ser de diferentes tipos em diferentes casos de erro.

Em segundo lugar, removemos as chamadas para `expect` em favor de `?`, como falamos sobre
isso no Capítulo 9. Ao invés de `panic!` em um erro, `?` retornará o valor do erro
a partir da função atual para que o chamador lide com ele.

Em terceiro lugar, a função `run` agora retorna um valor `Ok` no caso de sucesso. Nós
declaramos o tipo de sucesso da função `run` como `()` na assinatura, que significa que 
precisamos wrap (envolver) o valor do tipo de unidade no valor `Ok`. Esta sintaxe `Ok(())`
pode parecer um pouco estranha no início, mas usar `()` como este é o maneira idiomática 
de indicar que chamamos `run` para seus efeitos colaterais somente; ele não retorna o 
valor que precisamos.

Quando você executa este código, ele compilará, mas exibirá um aviso:

```text
warning: unused `std::result::Result` which must be used
  --> src/main.rs:18:5
   |
18 |     run(config);
   |     ^^^^^^^^^^^^
= note: #[warn(unused_must_use)] on by default
```

Rust nos diz que nosso código ignorou o valor `Result` e o valor de `Result` pode indicar 
que ocorreu um erro. Mas não estamos checando para ver se ocorreu ou não o erro, e o 
compilador nos lembra que provavelmente queríamos tratar algum código de erros aqui! 
Vamos corrigir esse problema agora.

#### Manipulação de Erros Retornados de `run` em `main`

Verificamos erros e lidaremos com eles usando uma técnica semelhante à nossa manipulação 
de erros com `Config::new` na Listagem 12-10, mas com umas diferenças:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    if let Err(e) = run(config) {
        println!("Application error: {}", e);

        process::exit(1);
    }
}
```

Usamos `if let` em vez de `unwrap_or_else` para verificar se `run` retorna um valor `Err` 
e chama `process::exit(1)` se o fizer. A função `run` não retorna um valor que queremos 
`unwrap` da mesma forma que `Config::new` retorna a instância `Config`. Porque `run` 
retorna `()` no caso de sucesso, nós só nos preocupamos em detectar um erro, por 
isso não precisamos de `unwrap_or_else` para devolver o valor unwrapped porque seria 
apenas `()`.

Os corpos das funções `if let` e `unwrap_or_else` são os mesmos em ambos os casos: 
imprimimos o erro e saímos.

### Dividindo o Código em uma Crate de Biblioteca

O nosso projeto `minigrep` parece estar bem até agora! Agora vamos dividir o *src/main.rs* 
e colocar algum código no arquivo *src/lib.rs* para que possamos testá-lo em um arquivo 
*src/main.rs* com menos responsabilidades.

Vamos mover todo o código que não é da função `main` de  *src/main.rs* para
*src/lib.rs*:

* A definição de função `run`
* As instruções relevantes `use`
* A definição de `Config`
* A definição da função `Config::new`

O conteúdo de *src/lib.rs* deve ter as assinaturas mostradas na Listagem 12-13
(omitimos o corpo das funções por brevidade). Observe que isso não irá
compilar até modificar o *src/main.rs* na listagem depois desta:

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
use std::error::Error;
use std::fs::File;
use std::io::prelude::*;

pub struct Config {
    pub query: String,
    pub filename: String,
}

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        // --snip--
    }
}

pub fn run(config: Config) -> Result<(), Box<Error>> {
    // --snip--
}
```

<span class="caption">Listagem 12-13: movendo `Config` e `run` para
*src/lib.rs*</span>

Nós fizemos um uso liberal do `pub` aqui: no `Config`, seus campos e seu método `new`
, e na função `run`. Agora temos uma crate de biblioteca que tem uma
API pública que podemos testar!

Agora, precisamos trazer o código que nós movemos para *src/lib.rs* no escopo da
crate binária em *src/main.rs*, conforme mostrado na Listagem 12-14:

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
extern crate minigrep;

use std::env;
use std::process;

use minigrep::Config;

fn main() {
    // --snip--
    if let Err(e) = minigrep::run(config) {
        // --snip--
    }
}
```

<span class="caption">Listagem 12-14: Trazendo a crate `minigrep` para o
escopo de *src/main.rs*</span>

Para colocar a crate de biblioteca na crate binária, usamos `extern crate
minigrep`. Em seguida, adicionaremos uma linha `use minigrep::Config` para trazer para o 
escopo o tipo `Config`, e iremos prefixar a função `run` com o nome da nossa crate. Agora
todas as funcionalidades devem estar conectadas e devem funcionar. Execute o programa com
`cargo run` e verifique se tudo funciona corretamente.

Ufa! Isso foi trabalhoso, mas nós nos preparamos para o sucesso no
futuro. Agora é muito mais fácil lidar com erros, e nós fizemos o código mais
modular. Quase todo o nosso trabalho será feito em *src/lib.rs* a partir daqui.

Vamos aproveitar desta nova recém-descoberta modularidade para fazer algo que seria
difícil com o código antigo, mas é fácil com o novo código: nós iremos
escreva alguns testes!