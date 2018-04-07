## Aceitando Argumentos em Linha de Comando

Vamos criar um novo projeto usando, como sempre, `cargo new`. Chamaremos o nosso projeto
`minigrep` para distingui-lo da ferramenta` grep` que você já pode ter 
no seu sistema.

```text
$ cargo new --bin minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

A primeira tarefa é fazer que `minigrep` aceite seus dois argumentos de linha de comando: o
nome de arquivo e uma string para procurar. Ou seja, queremos ser capazes de administrar o nosso
programa com `cargo run`, uma string para procurar e um caminho para um arquivo onde será feira a
procura, dessa forma:

```text
$ cargo run searchstring example-filename.txt
```

Neste momento, o programa gerado por `cargo new` não pode processar os argumentos que nós
passamos. No entanto, algumas bibliotecas existentes no [Crates.io](https://crates.io/)
que podem  nos ajudar a escrever um programa que aceite argumentos na linha de comando, mas
como você está aprendendo esses conceitos, vamos implementar essa capacidade
nós mesmos.

### Lendo os Valores do Argumento

Para garantir que `minigrep` seja capaz de ler os valores dos argumentos da linha de comando, nós
precisamos de uma função fornecida na biblioteca padrão do Rust, que é
`std::env::args`. Esta função retorna um *iterador* da linha de comando
com os argumentos que foram passados à `minigrep`. Ainda não discutimos iteradores
(nós os cobrimos totalmente no Capítulo 13), mas por enquanto você só precisa saber dois
detalhes sobre iteradores: os iteradores produzem uma série de valores, e podemos chamar
a função `collect` em um iterador para transformá-lo em uma coleção, como um
vetor, contendo todos os elementos que o iterador produz.

Use o código na Listagem 12-1 para permitir que seu programa `minigrep` leia qualquer
argumento da linha de comando passados para ele e depois colete os valores em um vetor:

<span class="filename">Arquivo: src/main.rs</span>

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    println!("{:?}", args);
}
```

<span class="caption">Listagem 12-1: Coletando os argumentos da linha de comando
um vetor e imprimindo-os</span>

Primeiro, trazemos o módulo `std::env` para o escopo com uma declaração `use`, então nós
podemos usar a função `args`. Observe que a função `std::env::args` é
aninhada em dois níveis de módulos. Como discutimos no Capítulo 7, nos casos em que
a função desejada está aninhada em mais de um módulo, é convenção
trazer o módulo pai para o escopo em vez da função. Como resultado, nós
podemos facilmente usar outras funções de `std::env`. Também é menos ambíguo que
adicionar `use std::env::args` e depois chamando a função com apenas `args`
porque `args` pode ser facilmente confundido com uma função definida no
módulo atual.

> ### A Função `args` e Unicode Inválido
>
> Note que `std::env::args` emitirá pânico se algum argumento contiver código
> Unicode inválido. Se o seu programa precisar aceitar argumentos que sejam 
> Unicode inválidos , use `std::env::args_os` em vez disso. Essa função retorna valores `OsString`
> em vez de valores `String`. Nós escolhemos usar `std::env::args` aqui
> por simplicidade, porque os valores de `OsString` diferem por plataforma e são mais
> complexo para trabalhar do que os valores de `String`.

Na primeira linha do `main`, chamamos `env::args`, e usamos `collect` imediatamente
para transformar o iterador em um vetor contendo todos os valores produzidos pelo
iterador. Podemos usar a função `collect` para criar muitos tipos de
coleções, então nós explicitamente anotamos o tipo de `args` para especificar que nós
queremos um vetor de strings. Embora raramente precisemos anotar tipos em
Rust, `collect` é uma função que muitas vezes você precisa anotar, porque Rust
não é capaz de inferir o tipo de coleção que deseja.

Finalmente, imprimimos o vetor usando o formatador de debug, `:?`. Vamos tentar executar
o código sem argumentos e depois com dois argumentos:

```text
$ cargo run
--snip--
["target/debug/minigrep"]

$ cargo run needle haystack
--snip--
["target/debug/minigrep", "needle", "haystack"]
```

Observe que o primeiro valor no vetor é `"target/debug/minigrep"`, que
é o nome do nosso binário. Isso corresponde ao comportamento da lista de argumentos em
C, permitindo que os programas usem o nome pelo qual eles foram invocados em sua execução.
Geralmente, é conveniente ter acesso ao nome do programa, caso desejemos imprimi-lo em mensagens ou 
alterar o comportamento do programa com base no alias da linha de comando que foi usado 
para chamar o programa. Mas, para os fins deste capítulo, vamos ignorá-lo e salvar apenas 
os dois argumentos que precisamos.

### Salvando os Valores do Argumento em Variáveis

Imprimir o valor do vetor de argumentos ilustra que o programa é
capaz de acessar os valores especificados como argumentos da linha de comando. Agora precisamos
salvar os valores dos dois argumentos nas variáveis para que possamos usar esses valores
durante o resto do programa. Fazemos isso na Listagem 12-2:

<span class="filename">Arquivo: src/main.rs</span>

```rust,should_panic
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let filename = &args[2];

    println!("Searching for {}", query);
    println!("In file {}", filename);
}
```

<span class="caption">Listagem 12-2: Criando variáveis para guardar o argumento de consulta
e argumento do nome do arquivo</span>

Como vimos quando imprimimos o vetor, o nome do programa ocupa o primeiro
valor no vetor em `args[0]`, então estamos começando no índice `1`. O primeiro
argumento `minigrep` é a string que estamos procurando, então colocamos uma
referência ao primeiro argumento na variável `query`. O segundo argumento
será o nome do arquivo, então colocamos uma referência ao segundo argumento no
variável `filename`.

Imprimimos temporariamente os valores dessas variáveis para provar que o código funciona
como pretendemos. Vamos executar este programa novamente com os argumentos `test`
e `sample.txt`:

```text
$ cargo run test sample.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep test sample.txt`
Searching for test
In file sample.txt
```

Ótimo, o programa está funcionando! Os valores dos argumentos que precisamos estão sendo
salvos nas variáveis certas. Mais tarde, adicionaremos algum tratamento de erro para lidar
com certas situações errôneas potenciais, como quando o usuário não fornece
argumentos; por enquanto, ignoraremos essa situação, e trabalharemos na adição das 
funcinalidades de leitura dos arquivos.