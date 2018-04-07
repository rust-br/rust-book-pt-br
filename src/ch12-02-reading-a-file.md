## Reading a File
## Lendo um Arquivo

Now we’ll add functionality to read the file that is specified in the
`filename` command line argument. First, we need a sample file to test it with:
the best kind of file to use to make sure `minigrep` is working is one with a
small amount of text over multiple lines with some repeated words. Listing 12-3
has an Emily Dickinson poem that will work well! Create a file called
*poem.txt* at the root level of your project, and enter the poem “I’m Nobody!
Who are you?”

Agora vamos adicionar funcionalidades para ler o arquivo que é especificado no
argumento `filename` da linha de comando. Primeiro, precisamos de um arquivo de amostra para testá-lo:
o melhor tipo de arquivo a ser usado para garantir que o `minigrep` esteja funcionando é um ,com uma
pequena quantidade de texto, em várias linhas com algumas palavras repetidas. Listagem 12-3
tem um poema de Emily Dickinson que funcionará bem! Crie um arquivo chamado
*poem.txt* no diretório raiz do seu projeto e entre com o poema “I’m Nobody!
Who are you?”

<span class="filename">Filename: poem.txt</span>

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

<span class="caption">Listing 12-3: A poem by Emily Dickinson will make a good
test case.</span>

<span class="caption">Listagem 12-3: Um poema de Emily Dickinson fará um bom
caso de teste.</span>

With the text in place, edit *src/main.rs* and add code to open the file, as
shown in Listing 12-4:

Com o texto no lugar, edite *src/main.rs* e adicione o código para abrir o arquivo, como
mostrado na Listagem 12-4:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 12-4: Reading the contents of the file specified
by the second argument</span>

<span class="caption">Listagem 12-4: Leitura do conteúdo do arquivo especificado
pelo segundo argumento</span>

First, we add some more `use` statements to bring in relevant parts of the
standard library: we need `std::fs::File` to handle files, and
`std::io::prelude::*` contains various useful traits for doing I/O, including
file I/O. In the same way that Rust has a general prelude that brings certain
types and functions into scope automatically, the `std::io` module has its own
prelude of common types and functions you’ll need when working with I/O. Unlike
the default prelude, we must explicitly add a `use` statement for the prelude
from `std::io`.

Primeiro, adicionamos mais instruções `use` para trazer partes relevantes da
biblioteca padrão: precisamos de `std::fs::File` para lidar com arquivos, e
`std::io::prelude::*` contém vários traits úteis para fazer E/S, incluindo
arquivo de E/S. Da mesma forma que Rust tem um prelúdio geral que traz certos
tipos e funções no escopo automaticamente, o módulo `std::io` possui o seu próprio
prelúdio de tipos e funções comuns que você precisará ao trabalhar com E/S. Ao contrário
do prelúdio padrão, devemos adicionar explicitamente uma instrução `use` para o prelúdio
de `std::io`.

In `main`, we’ve added three statements: first, we get a mutable handle to the
file by calling the `File::open` function and passing it the value of the
`filename` variable. Second, we create a variable called `contents` and set it
to a mutable, empty `String`. This will hold the content of the file after we
read it in. Third, we call `read_to_string` on our file handle and pass a
mutable reference to `contents` as an argument.

Em `main`, adicionamos três declarações: primeiro, recebemos um identificador mutável para o
arquivo chamando a função `File::open` e transmitindo o valor da
variável `filename`. Em segundo lugar, criamos uma variável chamada `contents` e configuramos
para uma `String` mutável e vazia. Isso manterá o conteúdo do arquivo depois que nós
lê-lo. Terceiro, chamamos `read_to_string` no nosso arquivo e passamos um
referência mutável para `contents` como argumento.

After those lines, we’ve again added a temporary `println!` statement that
prints the value of `contents` after the file is read, so we can check that the
program is working so far.

Após essas linhas, adicionamos novamente uma declaração temporária `println!` que
imprime o valor do `contents` depois que o arquivo é lido, para que possamos verificar que o
o programa está funcionando até o momento.

Let’s run this code with any string as the first command line argument (because
we haven’t implemented the searching part yet) and the *poem.txt* file as the
second argument:

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

Great! The code read and then printed the content of the file. But the code has
a few flaws. The `main` function has multiple responsibilities: generally,
functions are clearer and easier to maintain if each function is responsible
for only one idea. The other problem is that we’re not handling errors as well
as we could be. The program is still small so these flaws aren’t a big problem,
but as the program grows, it will be harder to fix them cleanly. It’s good
practice to begin refactoring early on when developing a program, because it’s
much easier to refactor smaller amounts of code. We’ll do that next.

Ótimo! O código lê e, em seguida, imprime o conteúdo do arquivo. Mas o código tem
algumas falhas. A função `main` tem múltiplas responsabilidades: geralmente,
as funções são mais claras e fáceis de manter se cada função for responsável
por apenas uma idéia. O outro problema é que também não estamos lidando com erros,
como poderíam ser. O programa ainda é pequeno, então essas falhas não são um grande problema,
mas à medida que o programa cresce, será mais difícil consertá-los de forma elegante. É uma boa
pratica começar a refatoração no início do desenvolvimento de um programa, porque são
muito mais fáceis de refatorar quantidades menores de código. Vamos fazer isso depois.
