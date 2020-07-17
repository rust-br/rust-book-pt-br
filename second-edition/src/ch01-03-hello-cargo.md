## Hello, Cargo!
## Olá, Cargo!

Cargo is Rust’s build system and package manager. Most Rustaceans will use this
tool to manage their Rust projects because Cargo takes care of a lot of tasks
for you, such as building your code, downloading the libraries your code
depends on, and building those libraries. (We call libraries your code needs
*dependencies*.)

Cargo é o gerente de sistemas e pacotes Rust. A maioria dos *Rustaceans* não usa 
essa ferramenta para gerenciar seus projetos Rust porque o Cargo cuida de muitas 
tarefas para você, como criar seu código, fazer o download das bibliotecas das 
quais seu código depende e criar essas bibliotecas. (Chamamos de bibliotecas que 
seu código precisa de *dependências*.)

The simplest Rust programs, like the one we’ve written so far, don’t have any
dependencies, so if we had built the Hello World project with Cargo, it would
only be using the part of Cargo that takes care of building your code. As you
write more complex Rust programs, you’ll want to add dependencies, and if you
start the project off using Cargo, that will be a lot easier to do.

Os programas Rust mais simples, como o que escrevemos até agora, não têm dependências; 
portanto, se tivéssemos construído o projeto Hello World com o Cargo, ele usaria apenas 
a parte do Cargo que cuida da criação do seu código. Ao escrever programas Rust mais 
complexos, você deseja adicionar dependências e, se você iniciar o projeto usando 
Cargo, isso será muito mais fácil.

As the vast majority of Rust projects use Cargo, the rest of this book will
assume that you’re using Cargo too. Cargo comes installed with Rust itself, if
you used the official installers as covered in the “Installation” section. If
you installed Rust through some other means, you can check if you have Cargo
installed by entering the following into your terminal:

Como a grande maioria dos projetos Rust usa Cargo, o restante deste livro pressupõe que 
você também esteja usando Cargo. Cargo vem instalado com o próprio Rust, se você usou 
os instaladores oficiais, conforme descrito na seção “Instalação”. Se você instalou 
Rust por outros meios, poderá verificar se possui o Cargo instalado inserindo o seguinte 
em seu terminal:

```text
$ cargo --version
```

If you see a version number, great! If you see an error like `command not
found`, then you should look at the documentation for your method of
installation to determine how to install Cargo separately.

Se você vir um número de versão, ótimo! Se você vir um erro como `command not
found`, consulte a documentação do seu método de instalação para determinar como instalar 
o Cargo separadamente.

### Creating a Project with Cargo
### Criando Projetos com Cargo

Let’s create a new project using Cargo and look at how it differs from our
original Hello World project. Navigate back to your *projects* directory (or
wherever you decided to put your code) and then on any operating system run:

Vamos criar um novo projeto usando Cargo e ver como ele difere do nosso projeto original 
Hello World. Navegue de volta para o diretório *projects* (ou onde quer que você tenha 
decidido colocar seu código) e, em seguida, em qualquer sistema operacional:

```text
$ cargo new hello_cargo --bin
$ cd hello_cargo
```

<!-- Below -- so we always have to start a cargo project with the --bin option
if we want it to be something we can execute and not just a library, is that
right? It might be worth laying that out -->
<!-- As of Rust 1.21.0 (the version we're using for the book), yes, you must
always specify `--bin`. In a version of Rust in the near future (1.25 or 1.26),
binary crates will become the default kind of crate that `cargo new` makes, so
you won't have to specify `--bin` (but you can if you want and the behavior
will be the same). We'd rather not go into any more detail than we have here
because of this change; I think "The `--bin` argument to passed to `cargo new`
makes an executable application (often just called a *binary*), as opposed to a
library." lays this out enough. /Carol -->

This creates a new binary executable called `hello_cargo`. The `--bin` argument
to passed to `cargo new` makes an executable application (often just called a
*binary*), as opposed to a library. We’ve given `hello_cargo` as the name for
our project, and Cargo creates its files in a directory of the same name.

Isso cria um novo executável binário chamado `hello_cargo`. O argumento `--bin` 
transmitido para `cargo new` cria um aplicativo executável (geralmente chamado 
apenas de *binário*), em oposição a uma biblioteca. Atribuímos `hello_cargo` como o 
nome do nosso projeto e o Cargo cria seus arquivos em um diretório com o mesmo nome.

Go into the *hello_cargo* directory and list the files, and you should see that
Cargo has generated two files and one directory for us: a *Cargo.toml* and a
*src* directory with a *main.rs* file inside. It has also initialized a new git
repository, along with a *.gitignore* file.

Vá para o diretório *hello_cargo* e liste os arquivos, e você verá que Cargo gerou 
dois arquivos e um diretório para nós: um diretório *Cargo.toml* e *src* com um 
arquivo *main.rs* dentro. Também inicializou um novo repositório git, junto com 
um arquivo *.gitignore*.

> Note: Git is a common version control system. You can change `cargo new` to
> use a different version control system, or no version control system, by
> using the `--vcs` flag. Run `cargo new --help` to see the available options.

> Nota: Git é um sistema de controle de versão comum. Você pode alterar `cargo new` 
> para usar um sistema de controle de versão diferente, ou nenhum sistema de controle 
> de versão, usando o sinalizador `--vcs`. Execute `cargo new --help` para ver as 
> opções disponíveis.

Open up *Cargo.toml* in your text editor of choice. It should look similar to
the code in Listing 1-2:

Abra *Cargo.toml* no seu editor de texto de sua escolha. Deve ser semelhante ao código 
na Listagem 1-2:

<span class="filename">Filename: Cargo.toml</span>

<span class="filename">Nome do arquivo: Cargo.toml</span>

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
```

<span class="caption">Listing 1-2: Contents of *Cargo.toml* generated by `cargo
new`</span>

<span class="caption">Listagem 1-2: Conteúdo de *Cargo.toml* gerado por `cargo new`</span>

This file is in the [*TOML*][toml]<!-- ignore --> (Tom’s Obvious, Minimal
Language) format, which is what Cargo uses as its configuration format.

Este arquivo está no formato [*TOML*][toml] <!-- ignore --> (Tom Óbvia, Linguagem Mínima), que 
é o que o Cargo usa como formato de configuração.

[toml]: https://github.com/toml-lang/toml

The first line, `[package]`, is a section heading that indicates that the
following statements are configuring a package. As we add more information to
this file, we’ll add other sections.


A primeira linha, `[package]`, é um cabeçalho de seção que indica que as seguintes instruções 
estão configurando um pacote. À medida que adicionamos mais informações a este arquivo, 
adicionaremos outras seções.

The next three lines set the configuration information Cargo needs in order to
know that it should compile your program: the name, the version, and who wrote
it. Cargo gets your name and email information from your environment, so if
that’s not correct, go ahead and fix that and save the file.

As próximas três linhas definem as informações de configuração que Cargo precisa 
para saber que ele deve compilar seu programa: o nome, a versão e quem o escreveu. 
Cargo obtém seu nome e informações de e-mail do seu ambiente; portanto, se isso não estiver 
correto, prossiga, corrija-o e salve o arquivo.

The last line, `[dependencies]`, is the start of a section for you to list any
of your project’s dependencies. In Rust, packages of code are referred to as
*crates*. We won’t need any other crates for this project, but we will in the
first project in Chapter 2, so we’ll use this dependencies section then.

A última linha, `[dependencies]`, é o início de uma seção para você listar qualquer uma 
das dependências do seu projeto. Em Rust, pacotes de código são referidos como *crates*. 
Não precisaremos de outras crates para este projeto, mas precisaremos no primeiro projeto 
do capítulo 2, portanto, usaremos essa seção de dependências.

Now open up *src/main.rs* and take a look:

Agora abra *src/main.rs* e olhe:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo has generated a “Hello World!” for you, just like the one we wrote in
Listing 1-1! So far, the differences between our previous project and the
project generated by Cargo are that with Cargo our code goes in the *src*
directory, and we have a *Cargo.toml* configuration file in the top directory.

Cargo gerou um “Hello World!” para você, exatamente como o que escrevemos na 
Lista 1-1! Até agora, as diferenças entre o projeto anterior e o projeto gerado 
pelo Cargo são que, com Cargo, nosso código entra no diretório *src* e temos um 
arquivo de configuração *Cargo.toml* no diretório superior.

Cargo expects your source files to live inside the *src* directory so that the
top-level project directory is just for READMEs, license information,
configuration files, and anything else not related to your code. In this way,
using Cargo helps you keep your projects nice and tidy. There’s a place for
everything, and everything is in its place.

Cargo espera que seus arquivos source morem dentro do diretório *src*, para que 
o diretório de projeto de nível superior seja apenas para READMEs, informações de 
licença, arquivos de configuração e qualquer outra coisa não relacionada ao seu código. 
Dessa forma, o uso do Cargo ajuda a manter seus projetos organizados. Há um lugar 
para tudo, e tudo está em seu lugar.

If you started a project that doesn’t use Cargo, as we did with our project in
the *hello_world* directory, you can convert it to a project that does use
Cargo by moving the project code into the *src* directory and creating an
appropriate *Cargo.toml*.

Se você iniciou um projeto que não usa Cargo, como fizemos com nosso projeto no 
diretório *hello_world*, você pode convertê-lo em um projeto que usa Cargo movendo 
o código do projeto para o diretório *src* e criando um apropriado *Cargo.toml*.

### Building and Running a Cargo Project
### Construindo e Executando um projeto Cargo

Now let’s look at what’s different about building and running your Hello World
program through Cargo! From your project directory, build your project by
entering the following commands:

Agora, vamos ver o que há de diferente na criação e execução do seu programa Hello 
World através do Cargo! No diretório do projeto, construa seu projeto digitando os 
seguintes comandos:

```text
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

This creates an executable file in *target/debug/hello_cargo* (or
*target\\debug\\hello_cargo.exe* on Windows), which you can run with this
command:

Isso cria um arquivo executável em *target/debug/hello_cargo* (ou 
*target\\debug\\hello_cargo.exe* no Windows), que você pode executar com 
este comando:

```text
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

Bam! If all goes well, `Hello, world!` should print to the terminal once more.
Running `cargo build` for the first time also causes Cargo to create a new file
at the top level called *Cargo.lock*, which is used to keep track of the exact
versions of dependencies in your project. This project doesn’t have
dependencies, so the file is a bit sparse. You won’t ever need to touch this
file yourself; Cargo will manage its contents for you.

Bam! Se tudo correr bem, `Hello, world!` deve ser impresso no terminal mais uma 
vez. A execução do `cargo build` pela primeira vez também faz com que o Cargo crie 
um novo arquivo no nível superior chamado *Cargo.lock*, que é usado para acompanhar 
as versões exatas das dependências do seu projeto. Este projeto não tem dependências, 
portanto o arquivo é um pouco esparso. Você nunca precisará tocar nesse arquivo; 
Cargo gerenciará seu conteúdo para você.

We just built a project with `cargo build` and ran it with
`./target/debug/hello_cargo`, but we can also use `cargo run` to compile and
then run all in one go:

Nós apenas construímos um projeto com `cargo build` e o executamos com 
`./target/debug/hello_cargo`, mas também podemos usar o `cargo run` para compilar e
então executar tudo de uma só vez:

```text
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Notice that this time, we didn’t see the output telling us that Cargo was
compiling `hello_cargo`. Cargo figured out that the files haven’t changed, so
it just ran the binary. If you had modified your source code, Cargo would have
rebuilt the project before running it, and you would have seen output like this:

Observe que, desta vez, não vimos a saída nos dizendo que Cargo estava compilando 
`hello_cargo`. Cargo descobriu que os arquivos não foram alterados; portanto, apenas 
executou o binário. Se você tivesse modificado seu código-fonte, Cargo reconstruiria 
o projeto antes de executá-lo e você teria visto resultados como este:

```text
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Finally, there’s `cargo check`. This command will quickly check your code to
make sure that it compiles, but not bother producing an executable:

Finalmente, há `cargo check`. Este comando verificará rapidamente seu código para 
garantir que ele seja compilado, mas não se incomode em produzir um executável:

```text
$ cargo check
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Why would you not want an executable? `cargo check` is often much faster than
`cargo build`, because it skips the entire step of producing the executable. If
you’re checking your work throughout the process of writing the code, using
`cargo check` will speed things up! As such, many Rustaceans run `cargo check`
periodically as they write their program to make sure that it compiles, and
then run `cargo build` once they’re ready to give it a spin themselves.

Por que você não gostaria de um executável? O `cargo check` geralmente é muito mais 
rápido que o `cargo build`, porque pula toda a etapa de produção do executável. Se você 
estiver verificando seu trabalho durante todo o processo de escrever o código, o uso 
de `cargo check` acelerará as coisas! Como tal, muitos *Rustaceans* executam `cargo check` 
periodicamente enquanto escrevem seu programa para garantir que ele seja compilado e, 
em seguida, executam `cargo build` quando estiverem prontos para rodar.

So to recap, using Cargo:

- We can build a project using `cargo build` or `cargo check`
- We can build and run the project in one step with `cargo run`
- Instead of the result of the build being put in the same directory as our
  code, Cargo will put it in the *target/debug* directory.

Então, para recapitular, usando Cargo:

- Podemos construir um projeto usando `cargo build` ou `cargo check`
- Podemos construir e executar o projeto em uma única etapa com `cargo run`
- Em vez de o resultado da compilação ser colocado no mesmo diretório que o nosso código, 
  Cargo o colocará no diretório *target/debug*.

A final advantage of using Cargo is that the commands are the same no matter
what operating system you’re on, so at this point we will no longer be
providing specific instructions for Linux and Mac versus Windows.

Uma vantagem final do uso do Cargo é que os comandos são os mesmos, independentemente do 
sistema operacional em que você esteja; portanto, neste momento, não forneceremos mais 
instruções específicas para Linux e Mac versus Windows.

### Building for Release
### Criando para Liberação

When your project is finally ready for release, you can use `cargo build
--release` to compile your project with optimizations. This will create an
executable in *target/release* instead of *target/debug*. These optimizations
make your Rust code run faster, but turning them on makes your program take
longer to compile. This is why there are two different profiles: one for
development when you want to be able to rebuild quickly and often, and one for
building the final program you’ll give to a user that won’t be rebuilt
repeatedly and that will run as fast as possible. If you’re benchmarking the
running time of your code, be sure to run `cargo build --release` and benchmark
with the executable in *target/release*.

Quando seu projeto estiver finalmente pronto para o lançamento, você poderá usar o 
`cargo build --release` para compilar seu projeto com otimizações. Isso criará um 
executável em *target/release* em vez de *target/debug*. Essas otimizações tornam seu 
código Rust mais rápido, mas ativá-los leva mais tempo para compilar o programa. É por 
isso que existem dois perfis diferentes: um para desenvolvimento, quando você deseja 
reconstruir de forma rápida e frequente, e outro para a criação do programa final, 
que você fornecerá a um usuário que não será reconstruído repetidamente e que será 
executado como o mais rápido possível. Se você estiver comparando o tempo de execução 
do seu código, lembre-se de executar `cargo build --release` e faça a comparação com 
o executável em *target/release*.


### Cargo as Convention
### Cargo como Convenção

With simple projects, Cargo doesn’t provide a whole lot of value over just
using `rustc`, but it will prove its worth as you continue. With complex
projects composed of multiple crates, it’s much easier to let Cargo coordinate
the build.

Em projetos simples, Cargo não fornece muito valor ao usar apenas `rustc`, mas 
provará seu valor à medida que você continua. Com projetos complexos compostos 
por várias *crates*, é muito mais fácil deixar Cargo coordenar a construção.

Even though the `hello_cargo` project is simple, it now uses much of the real
tooling you’ll use for the rest of your Rust career. In fact, to work on any
existing projects you can use the following commands to check out the code
using Git, change into the project directory, and build:

Embora o projeto `hello_cargo` seja simples, agora ele usa grande parte das ferramentas 
reais que você usará para o resto de sua carreira em Rust. De fato, para trabalhar em 
qualquer projeto existente, você pode usar os seguintes comandos para verificar o código 
usando o Git, mudar para o diretório do projeto e criar:

```text
$ git clone someurl.com/someproject
$ cd someproject
$ cargo build
```

If you want to look at Cargo in more detail, check out [its documentation].

Se você deseja examinar Cargo com mais detalhes, consulte [its documentation] (em inglês).

[its documentation]: https://doc.rust-lang.org/cargo/

<!--Below -- I`m not sure this is the place for this conversation, it seems too
deep into the weeds for the "getting started" chapter. I know we discussed
Nightly Rust as an appendix previously, but honestly I think this is more
suited somewhere online, perhaps in the extended docs. I like the idea of
finishing the chapter here, on this practical note, and I think at this point
readers will want to get stuck in anyway and may skip this and never come back
because it's buried at the end of a chapter that's not really related to it. If
it's online somewhere separate they can come to it when they're ready. What do
you think?-->
<!-- Ok, I can see that. /Carol -->

## Summary
## Resumo

You’re already off to a great start on your Rust journey! In this chapter,
you’ve:

* Installed the latest stable version of Rust
* Written a “Hello, world!” program using both `rustc` directly and using
  the conventions of `cargo`

Você já começou bem a sua jornada Rust! Neste capítulo, você:

* Instalou a versão estável mais recente de Rust
* Escreveu um programa “Hello, world!” usando diretamente o `rustc` e as 
  convenções do `cargo`

This is a great time to build a more substantial program, to get used to
reading and writing Rust code. In the next chapter, we’ll build a guessing game
program. If you’d rather start by learning about how common programming
concepts work in Rust, see Chapter 3.

Este é um ótimo momento para criar um programa mais substancial, para se acostumar 
a ler e escrever código Rust. No próximo capítulo, criaremos um programa de jogos 
de adivinhação. Se você preferir começar a aprender sobre como os conceitos comuns 
de programação funcionam em Rust, consulte o Capítulo 3.