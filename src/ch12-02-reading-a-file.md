## Lendo um Arquivo

Agora vamos adicionar funcionalidades para ler o arquivo que é especificado no
argumento `filename` da linha de comando. Primeiro, precisamos de um arquivo de amostra para testá-lo:
usaremos um arquivo com uma pequena quantidade de texto em várias linhas com algumas palavras repetidas. 
Listagem 12-3 tem um poema de Emily Dickinson que funcionará bem! Crie um arquivo chamado
*poem.txt* no diretório raiz do seu projeto e entre com o poema “I’m Nobody!
Who are you?”

<span class="filename">Arquivo: poem.txt</span>

```text
I’m nobody! Who are you?
Are you nobody, too?
Then there’s a pair of us — don’t tell!
They’d banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

<span class="caption">Listagem 12-3: Um poema de Emily Dickinson fará um bom
caso de teste.</span>

Com o texto no lugar, edite *src/main.rs* e adicione o código para abrir o arquivo, como
mostrado na Listagem 12-4:

<span class="filename">Arquivo: src/main.rs</span>

```rust,should_panic
use std::env;
use std::fs::File;
use std::io::prelude::*;

fn main() {
#     let args: Vec<String> = env::args().collect();
#
#     let query = &args[1];
#     let filename = &args[2];
#
#     println!("Searching for {}", query);
    // --snip--
    println!("In file {}", filename);

    let mut f = File::open(filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");

    println!("With text:\n{}", contents);
}
```

<span class="caption">Listagem 12-4: Leitura do conteúdo do arquivo especificado
pelo segundo argumento</span>

Primeiro, adicionamos mais instruções `use` para trazer partes relevantes da
biblioteca padrão: precisamos de `std::fs::File` para lidar com arquivos, e
`std::io::prelude::*` contém vários traits úteis para fazer E/S, incluindo
arquivo de E/S. Da mesma forma que Rust tem um prelúdio geral que traz certos
tipos e funções no escopo automaticamente, o módulo `std::io` possui o seu próprio
prelúdio de tipos e funções comuns que você precisará ao trabalhar com E/S. Ao contrário
do prelúdio padrão, devemos adicionar explicitamente uma instrução `use` para o prelúdio
de `std::io`.

Em `main`, adicionamos três declarações: primeiro, recebemos um identificador mutável para o
arquivo chamando a função `File::open` e transmitindo o valor da
variável `filename`. Em segundo lugar, criamos uma variável chamada `contents` e configuramos
para uma `String` mutável e vazia. Isso manterá o conteúdo do arquivo depois que nós
lê-lo. Terceiro, chamamos `read_to_string` no nosso arquivo e passamos um
referência mutável para `contents` como argumento.

Após essas linhas, adicionamos novamente uma declaração temporária `println!` que
imprime o valor do `contents` depois que o arquivo é lido, para que possamos verificar que
o programa está funcionando até o momento.

Vamos executar este código com qualquer string como o primeiro argumento da linha de comando (porque
ainda não implementamos a parte de pesquisa) e o arquivo *poem.txt* como o
segundo argumento:

```text
$ cargo run the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I’m nobody! Who are you?
Are you nobody, too?
Then there’s a pair of us — don’t tell!
They’d banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

Ótimo! O código lê e, em seguida, imprime o conteúdo do arquivo. Mas o código tem
algumas falhas. A função `main` tem múltiplas responsabilidades: geralmente,
as funções são mais claras e fáceis de manter se cada função for responsável
por apenas uma idéia. O outro problema é que também não estamos lidando com erros,
como poderíam ser. O programa ainda é pequeno, então essas falhas não são um grande problema,
mas à medida que o programa cresce, será mais difícil consertá-los de forma elegante. É uma boa
pratica começar a refatoração no início do desenvolvimento de um programa, porque são
muito mais fáceis de refatorar quantidades menores de código. Vamos fazer isso depois.