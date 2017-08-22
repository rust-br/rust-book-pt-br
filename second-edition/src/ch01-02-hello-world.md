## Hello, World!

Now that you have Rust installed, let’s write your first Rust program. It’s
traditional when learning a new language to write a little program to print the
text “Hello, world!” to the screen, and in this section, we’ll follow that
tradition.

Agora que Rust já está instalado, vamos escrever nosso primeiro programa. É tradicional
quando aprendemos uma nova linguagem de programação, escrever um pequeno programa que imprime
“Hello, world!” na tela, e é exatamente isso que iremos fazer nesta seção.

> Note: This book assumes basic familiarity with the command line. Rust itself
> makes no specific demands about your editing, tooling, or where your code
> lives, so if you prefer an IDE to the command line, feel free to use your
> favorite IDE.

> Nota: Este livro assume que o leitor tenha familiaridade com a linha de comando. Rust não exige
> que você use um determinado editor ou IDE, ou seja, você está livre para usar o que bem entender
> para escrever seu código Rust.

### Creating a Project Directory
### Criando um diretório de projeto

First, make a directory to put your Rust code in. Rust doesn’t care where your code
lives, but for this book, we’d suggest making a *projects* directory in your
home directory and keeping all your projects there. Open a terminal and enter
the following commands to make a directory for this particular project:

Primeiramente, crie um diretório para armazenar o seu código Rust. Rust não se importa aonde você vai armazenar o seu código, mas enquanto lê este livro, nós sugerimos criar um diretório chamado *projects* e armazenar todos os seus projetos ali. Abra o seu terminal e digite os seguintes comandos:

Linux e Mac:

```text
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Windows CMD:

```cmd
> mkdir %USERPROFILE%\projects
> cd %USERPROFILE%\projects
> mkdir hello_world
> cd hello_world
```

Windows PowerShell:

```powershell
> mkdir $env:USERPROFILE\projects
> cd $env:USERPROFILE\projects
> mkdir hello_world
> cd hello_world
```

### Writing and Running a Rust Program
### Escrevendo e Executando um Programa em Rust

Next, make a new source file and call it *main.rs*. Rust files always end with
the *.rs* extension. If you’re using more than one word in your filename, use
an underscore to separate them. For example, you’d use *hello_world.rs* rather
than *helloworld.rs*.

Crie um novo arquivo *main.rs*. Arquivos relacionados a linguagem Rust sempre
terminam com a extensão *.rs* . Se você for nomear o seu arquivo com mais de um nome, use
um underscore para separá-los. Por exemplo, você deve nomear o seu arquivo *hello_world.rs* ao
invés de *helloworld.rs*.

Now open the *main.rs* file you just created, and type the following code:

Agora abra o arquivo *main.rs* que você acabou de criar e digite o seguinte código:

<span class="filename">Filename: main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Save the file, and go back to your terminal window. On Linux or OSX, enter the
following commands:

Salve o arquivo e volte ao seu terminal. No Linux ou OSX, digite os
seguintes comandos:

```text
$ rustc main.rs
$ ./main
Hello, world!
```

On Windows, run `.\main.exe` instead of `./main`. Regardless of your
operating system, you should see the string `Hello, world!` print to the
terminal. If you did, then congratulations! You’ve officially written a Rust
program. That makes you a Rust programmer! Welcome!

No Windows, para executar o seu programa você deve digitar `.\main.exe` ao invés de `./main`. Independente do seu
sistema operacional, você deverá ver a mensagem `Hello, world!` no seu terminal. 
Se você chegou até aqui, parabéns! Você escreveu o seu primeiro programa em Rust. 
Isso faz de você um programador Rust!

### Anatomy of a Rust Program
### Anatomia de um Programa escrito em Rust

Now, let’s go over what just happened in your “Hello, world!” program in
detail. Here’s the first piece of the puzzle:

Agora vamos ver o que aconteceu com o seu programa “Hello, world!” em
detalhes. Aqui está a primeira peça do quebra-cabeças:

```rust
fn main() {

}
```

These lines define a *function* in Rust. The `main` function is special: it’s
the first thing that is run for every executable Rust program. The first line
says, “I’m declaring a function named `main` that has no parameters and returns
nothing.” If there were parameters, their names would go inside the
parentheses, `(` and `)`.

Estas linhas definem uma *função* em Rust. A função `main` é especial: é a
primeira coisa que é executada em cada programa escrito em Rust. A primeira linha diz, 
“Estou declarando uma função chamada `main` que não contém nenhum parâmetro e que não retorna nada.” 
Se existissem parâmetros, eles estariam dentro dos parênteses, `(` and `)`.

Also note that the function body is wrapped in curly braces, `{` and `}`. Rust
requires these around all function bodies. It’s considered good style to put
the opening curly brace on the same line as the function declaration, with one
space in between.

Também vale notar que o corpo da função está envolvida por duas chaves, `{` and `}`. Rust
requer essas duas chaves no começo e no fim do corpo de uma função. É considerado uma boa prática
colocar a chave que indica a abertura de uma função na mesma linha da declaração da função, com um espaço
entre elas.

Inside the `main` function:

Dentro da função `main` :

```rust
    println!("Hello, world!");
```

This line does all of the work in this little program: it prints text to the
screen. There are a number of details to notice here. The first is that Rust
style is to indent with four spaces, not a tab.

Esta linha acima executa tudo o que o nosso pequeno programa faz: imprime um texto
na tela. Existem alguns detalhes a se notar aqui. O primeiro é que estilo de identação de Rust
é quatro espaços e não tab.

The second important part is `println!`. This is calling a Rust *macro*,
which is how metaprogramming is done in Rust. If it were calling a function
instead, it would look like this: `println` (without the `!`). We’ll discuss
Rust macros in more detail in Appendix E, but for now you just need to know
that when you see a `!` that means that you’re calling a macro instead of a
normal function.

A segunda parte importante é o `println!`. Este comando está chamando uma *macro* em Rust, que é como a metaprogramação é feita em Rust. Se estivessemos chamando uma função, usaríamos: `println` (sem o `!`). Iremos discutir macros em Rust em maiores detalhes no Apêndice E, mas por agora você só precisa saber que quando usamos um `!` isso significa que estamos chamando uma macro ao invés de uma função.

Next is `"Hello, world!"` which is a *string*. We pass this string as an
argument to `println!`, which prints the string to the screen. Easy enough!

A próxima linha do nosso programa é `"Hello, world!"` que é uma *string*. Nós passamos esta string como um argumento para a macro `println!`, e ela imprime a string na tela.

The line ends with a semicolon (`;`). The `;` indicates that this expression is
over, and the next one is ready to begin. Most lines of Rust code end with a
`;`.

A linha termina com um ponto e vírgula (`;`). O `;` indica esta expressão acabou, e que a próxima esta pronta para começar. A maioria das linhas de código Rust terminam com um `;`.

### Compiling and Running Are Separate Steps
### Compilação e Execução são Diferentes Etapas

In “Writing and Running a Rust Program”, we showed you how to run a newly
created program. We’ll break that process down and examine each step now.

Na seção “Escrevendo e Executando um Programa em Rust”, mostramos como você pode executar um programa que você acabou de criar. A partir de agora, vamos pegar este processo, quebrar em vários passos e examiná-los.

Before running a Rust program, you have to compile it. You can use the Rust
compiler by entering the `rustc` command and passing it the name of your source
file, like this:

Antes de executar qualquer programa em Rust, voce deve compilá-lo. Você deve usar o compilador Rust utilizando o comando `rustc` junto com o nome do seu código fonte conforme o exemplo abaixo:

```text
$ rustc main.rs
```

If you come from a C or C++ background, you’ll notice that this is similar to
`gcc` or `clang`. After compiling successfully, Rust should output a binary
executable, which you can see on Linux or OSX by entering the `ls` command in
your shell as follows:

Se você já programou em C ou C++, irá notar que esta etapa é bem similar ao `gcc` ou `clang`. Após a compilação ser realizada com sucesso, Rust deve gerar como saída um binário executável, o qual poderá conferir com o comando `ls` no seu terminal se estiver em um ambiente Linux ou OSX:

```text
$ ls
main  main.rs
```

On Windows, you’d enter:
No Windows, digite o seguinte:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.rs
```

This shows we have two files: the source code, with the *.rs* extension, and the
executable (*main.exe* on Windows, *main* everywhere else). All that’s left to
do from here is run the *main* or *main.exe* file, like this:

Temos aqui dois arquivos: o código fonte, que termina com a extensão *.rs*, e o arquivo  executável (*main.exe* no Windows, *main* em qualquer outro Sistema Operacional). O que nos resta fazer é executar o arquivo *main* ou *main.exe* :

```text
$ ./main  # or .\main.exe on Windows
```

If *main.rs* were your “Hello, world!” program, this would print `Hello,
world!` to your terminal.

Se o seu arquivo *main.rs* tiver o código que escrevemos acima, vai aparecer no seu terminal a mensagem `Hello,
world!`.

If you come from a dynamic language like Ruby, Python, or JavaScript, you may
not be used to compiling and running a program being separate steps. Rust is an
*ahead-of-time compiled* language, which means that you can compile a program,
give it to someone else, and they can run it even without having Rust
installed. If you give someone a `.rb`, `.py`, or `.js` file, on the other
hand, they need to have a Ruby, Python, or JavaScript implementation installed
(respectively), but you only need one command to both compile and run your
program. Everything is a tradeoff in language design.

Se você programa utilizando uma linguagem dinâmica como Ruby, Python, ou JavaScript, você provavelmente não está acostumado compilar para só depois executar um programa. Rust é uma linguagem *ahead-of-time compiled* , o que significa que você pode compilar um programa, enviá-lo para alguém, e tal pessoa pode executar o seu programa mesmo que não tenha Rust instalado no ambiente dela. Se você enviar a alguém um arquivo `.rb`, `.py`, ou `.js`, a pessoa vai precisar ter um interpretador Ruby, Python, ou JavaScript (respectivamente) instalados no ambiente dela, mas nesse caso (linguagens interpretadads) você só precisa de um único comando para compilar e executar o seu programa. Quando falamos de design de linguagens de programação tudo se trata de um tradeoff (cada escolha uma renúncia).

Just compiling with `rustc` is fine for simple programs, but as your project
grows, you’ll want to be able to manage all of the options your project has
and make it easy to share your code with other people and projects. Next, we’ll
introduce you to a tool called Cargo, which will help you write real-world Rust
programs.

Em se tratando de programas simples, compilar utilizando `rustc` servirá na maioria dos casos, mas conforme o seu projeto cresce, você com certeza vai querer gerenciar todas as opções possíveis para o seu projeto e facilitar compartilhar o seu código com o outros. Por esta razão iremos te apresentar agora uma ferramenta chamada Cargo, que irá te ajudar a escrever programas Rust usados no mundo real.

## Hello, Cargo!

Cargo is Rust’s build system and package manager, and Rustaceans use Cargo to
manage their Rust projects because it makes a lot of tasks easier. For example,
Cargo takes care of building your code, downloading the libraries your code
depends on, and building those libraries. We call libraries your code needs
*dependencies*.

Cargo é o sistema de build e o gerenciador de pacotes (package manager) do ecossistema Rust. Nós Rustaceans usamos Cargo para gerenciar nossos projetos Rust porque ele facilita muito as coisas. Por exemplo, Cargo se encarrega do processo de build do seu código, fazer o download de bibliotecas e dependências e assim por diante. 
*Dependências* são bibliotecas que seu código necessita para executar.

The simplest Rust programs, like the one we’ve written so far, don’t have any
dependencies, so right now, you’d only be using the part of Cargo that can take
care of building your code. As you write more complex Rust programs, you’ll
want to add dependencies, and if you start off using Cargo, that will be a lot
easier to do.

Programas simples em Rust, como o que escrevemos a pouco, não tem nenhuma dependência, então, neste caso, você só vai usar a parte do Cargo que cuida do build do seu código. Conforme escrever programas mais complexos em Rust, você com certeza vai adicionar dependências, e se utilizar Cargo, irá facilitar as coisas. 

As the vast, vast majority of Rust projects use Cargo, we will assume that
you’re using it for the rest of the book. Cargo comes installed with Rust
itself, if you used the official installers as covered in the Installation
chapter. If you installed Rust through some other means, you can check if you
have Cargo installed by typing the following into your terminal:

Como a vasta maioria dos projetos em Rust usam Cargo, iremos assumir o uso dele no restante do livro. Cargo já vem instalado com Rust por padrão, se você utilizou algum dos instaladores oficiais contidos na seção Instalando Rust. 
Se você utilizou outros meios para instalar Rust, poderá verificar sem tem Cargo instalado digitando o seguinte comando no seu terminal:

```text
$ cargo --version
```

If you see a version number, great! If you see an error like `command not
found`, then you should look at the documentation for your method of
installation to determine how to install Cargo separately.

Se aparecer um número de versão, ótimo! Mas, se aparecer um erro como `command not found`, você vai ter que olhar na documentação do método de instalação utilizado, para verificar como instalar Cargo em seu ambiente.

### Creating a Project with Cargo
### Criando um Projeto com Cargo

Let’s create a new project using Cargo and look at how it differs from our
project in `hello_world`. Go back to your projects directory (or wherever you
decided to put your code):

Vamos criar um novo projeto usando Cargo e ver o que o diferencia do nosso primeiro projeto `hello_world`. Volte so seu diretório de projetos (ou aonde quer que você colocou o seu código):

Linux e Mac:

```text
$ cd ~/projects
```

Windows:

```cmd
> cd %USERPROFILE%\projects
```

And then on any operating system run:

E então execute o comando abaixo:

```text
$ cargo new hello_cargo --bin
$ cd hello_cargo
```

We passed the `--bin` argument to `cargo new` because our goal is to make an
executable application, as opposed to a library. Executables are binary
executable files often called just *binaries*. We’ve given `hello_cargo`
as the name for our project, and Cargo creates its files in a directory
of the same name that we can then go into.

Passamos o argumento `--bin` para o comando `cargo new` porque o nosso objetivo é fazer uma aplicação executável, ao invés de uma biblioteca. Executáveis são arquivos binários muitas vezes chamadas apenas de *binários*. Demos o nome `hello_cargo` ao nosso projeto, e Cargo se encarregou de criar os arquivos em um diretório com o mesmo nome.

If we list the files in the *hello_cargo* directory, we can see that Cargo has
generated two files and one directory for us: a *Cargo.toml* and a *src*
directory with a *main.rs* file inside. It has also initialized a new git
repository in the *hello_cargo* directory for us, along with a *.gitignore*
file; you can change this to use a different version control system, or no
version control system, by using the `--vcs` flag.

Se listarmos os arquivos no nosso diretório *hello_cargo*, podemos ver que Cargo gerou dois arquivos e um diretório para nós: um arquivo *Cargo.toml* e um diretório *src* com um arquivo *main.rs* dentro dele. Também automaticamente já inicializou um repositório git no diretório *hello_cargo*, junto com um arquivo *.gitignore*; você pode querer usar um diferente sistema de controle de versão ou nenhum sistema de controle de versão, basta usar a flag `--vcs` para que isso ocorra.

Open up *Cargo.toml* in your text editor of choice. It should look something
like this:

Abra o arquivo *Cargo.toml* no seu editor de texto. Vai ser algo parecido com isso aqui:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
```

This file is in the [*TOML*][toml]<!-- ignore --> (Tom’s Obvious, Minimal
Language) format. TOML is similar to INI but has some extra goodies and is used
as Cargo’s configuration format.

Este arquivo está no formato [*TOML*][toml]<!-- ignore --> (Tom’s Obvious, Minimal Language). TOML é similar a INI com algumas coisinhas a mais e é usado como o formato de configuração do Cargo.

[toml]: https://github.com/toml-lang/toml

The first line, `[package]`, is a section heading that indicates that the
following statements are configuring a package. As we add more information to
this file, we’ll add other sections.

A primeira linha, `[package]`, é um cabeçalho de seção que indica que o que se segue, está configurando um pacote. Ao passo que adicionamos mais informações neste arquivo, vamos adicionando outras seções a ele.

The next three lines set the three bits of configuration that Cargo needs to
see in order to know that it should compile your program: its name, what
version it is, and who wrote it. Cargo gets your name and email information
from your environment. If it’s not correct, go ahead and fix that and save the
file.

As próximas três linhas informam para o Cargo como ele deverá compilar o seu programa: nome, versão, autor do programa. Cargo pega seu nome e e-mail através do seu ambiente. Se não estiver correto, corrija e depois salve o arquivo.

The last line, `[dependencies]`, is the start of a section for you to list any
*crates* (which is what we call packages of Rust code) that your project will
depend on so that Cargo knows to download and compile those too. We won’t need
any other crates for this project, but we will in the guessing game tutorial in
the next chapter.

A última linha, `[dependencies]`, é o começo da seção em que você deve listar quaisquer *crates* (que é como chamamos pacotes de código Rust) que seu projeto irá depender. Assim o Cargo saberá que deve fazer o download e compilar estes pacotes também. Não iremos precisar de nenhum crate (pacote) para este nosso projeto inicial (Hello World), mas para o projeto do próximo capítulo (Guessing Game), iremos utilizar os crates.

Now let’s look at *src/main.rs*:

Agora vamos dar uma olhada em *src/main.rs*:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo has generated a “Hello World!” for you, just like the one we wrote
earlier! So that part is the same. The differences between our previous project
and the project generated by Cargo that we’ve seen so far are:

O Cargo gerou um “Hello World!” para você, igual o que a gente escreveu anteriormente!  As diferenças entre o projeto que geramos e o projeto que o Cargo gerou para nósaté agora foram:

- Our code goes in the *src* directory
- The top level contains a *Cargo.toml* configuration file

- Nosso código fica dentro do diretório *src*
- Na raiz do nosso projeto existe um arquivo de configuração *Cargo.toml*

Cargo expects your source files to live inside the *src* directory so that the
top-level project directory is just for READMEs, license information,
configuration files, and anything else not related to your code. In this way,
using Cargo helps you keep your projects nice and tidy. There’s a place for
everything, and everything is in its place.

Cargo espera que os códigos-fonte fiquem no diretório *src* de maneira que 
o diretório acima fique apenas para READMEs, informações a respeito de licença,
arquivos de configuração, e tudo mais que não for relacionado a código. Assim,
o uso de Cargo te ajuda a manter os seus projetos bem organizados.

If you started a project that doesn’t use Cargo, as we did with our project in
the *hello_world* directory, you can convert it to a project that does use
Cargo by moving your code into the *src* directory and creating an appropriate
*Cargo.toml*.

Se você iniciou um projeto que não usa Cargo, como fizemos no nosso projeto inicial *hello_world*, você pode convertê-lo em um projeto Cargo. Basta mover o seu código para um diretório *src* e criar um arquivo *Cargo.toml*

### Building and Running a Cargo project
### Realizando o Build e Executando um projeto Cargo

Now let’s look at what’s different about building and running your Hello World
program through Cargo! To do so, enter the following commands:

Agora vamos ver qual a diferença de fazermos o build e executarmos o nosso projeto Hello World através do Cargo! Para tal, digite os seguintes comandos:

```text
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
```

This should have created an executable file in *target/debug/hello_cargo* (or
*target\debug\hello_cargo.exe* on Windows), which you can run with this command:

O comando acima deve ter criado um arquivo executável em *target/debug/hello_cargo* (ou *target\debug\hello_cargo.exe* no Windows), que você pode rodar com o seguinte comando:

```text
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

Bam! If all goes well, `Hello, world!` should print to the terminal once more.

Bam! Se tudo ocorrer bem, a mensagem `Hello, world!` deve aparecer no seu terminal.

Running `cargo build` for the first time also causes Cargo to create a new file
at the top level called *Cargo.lock*, which looks like this:

Executar `cargo build` pela primeira vez faz com que Cargo crie um novo arquivo
chamado *Cargo.lock*, que se parece com o seguinte:

<span class="filename">Filename: Cargo.lock</span>

```toml
[root]
name = "hello_cargo"
version = "0.1.0"
```

Cargo uses the *Cargo.lock* to keep track of dependencies in your application.
This project doesn’t have dependencies, so the file is a bit sparse.
Realistically, you won’t ever need to touch this file yourself; just let Cargo
handle it.

Cargo usa o arquivo *Cargo.lock* para rastrear as dependências na sua aplicação.
Este projeto não tem dependências, então o arquivo é um pouco escasso.
Na realidade, você nem precisará mexer neste arquivo; apenas deixe que Cargo
se encarregue dele.

We just built a project with `cargo build` and ran it with
`./target/debug/hello_cargo`, but we can also use `cargo run` to compile
and then run:

Nós acabamos de fazer o build de um projeto utilizando o comando `cargo build` e executá-lo com o comando `./target/debug/hello_cargo`, mas podemos usar também o comando `cargo run` para compilá-lo e depois executá-lo:

```text
$ cargo run
     Running `target/debug/hello_cargo`
Hello, world!
```

Notice that this time, we didn’t see the output telling us that Cargo was
compiling `hello_cargo`. Cargo figured out that the files haven’t changed, so
it just ran the binary. If you had modified your source code, Cargo would have
rebuilt the project before running it, and you would have seen something like
this:

Note que desta vez, não vimos nenhuma mensagem de output (saída) nos dizendo que Cargo estava compilando `hello_cargo`. Cargo entendeu que não teve nenhuma mudança nos arquivos, então ele apenas executou o binário. Se você modificou o seu código fonte, Cargo teve que dar um re-build no projeto antes de executá-lo, e você deve ter observado algo como:

```text
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
     Running `target/debug/hello_cargo`
Hello, world!
```

So a few more differences we’ve now seen:

Então, algumas diferenças que vimos até agora foram:

- Instead of using `rustc`, build a project using `cargo build` (or build and
  run it in one step with `cargo run`)
- Instead of the result of the build being put in the same directory as our
  code, Cargo will put it in the *target/debug* directory.

- Ao invés de usar `rustc`, faça o build do projeto usando `cargo build` (ou faça o build e execute de uma vez só utilizando `cargo run`)
- Ao invés de colocar o resultado do build no mesmo diretório do nosso código, Cargo irá colocar o resultado do build no diretório *target/debug*.

The other advantage of using Cargo is that the commands are the same no matter
what operating system you’re on, so at this point we will no longer be
providing specific instructions for Linux and Mac versus Windows.

A outra vantagem em se usar Cargo é que os comandos são os mesmos, não importa em qual sistema operacional você esteja. Por conta disso, a partir de agora não iremos especificar o tipo de sistema operacional quer seja ele Linux, Mac ou Windows.

### Building for Release
### Do Build à Release

When your project is finally ready for release, you can use `cargo build
--release` to compile your project with optimizations. This will create an
executable in *target/release* instead of *target/debug*. These optimizations
make your Rust code run faster, but turning them on makes your program take
longer to compile. This is why there are two different profiles: one for
development when you want to be able to rebuild quickly and often, and one for
building the final program you’ll give to a user that won’t be rebuilt and
that we want to run as fast as possible. If you’re benchmarking the running
time of your code, be sure to run `cargo build --release` and benchmark with
the executable in *target/release*.

Quando o seu projeto está finalmente pronto para release, você pode usar o comando `cargo build --release` para compilar o seu projeto com otimizações. Isso vai criar um arquivo executável no diretório *target/release* ao invés de *target/debug*. Estas otimizações fazem o seu código Rust executar de maneira mais rápida, mas por outro lado, leva mais tempo para compilar. Por isso existem dois profiles (perfis) diferentes: um para desenvolvimento, quando você deseja dar rebuild rápido e diversas vezes, e um profile (perfil) para fazer o build da versão final do seu programa que você irá utilizar quando quiser rodar o seu código da maneira mais rápida possível. Se você está realizando um processo de benchmarking no tempo de execução do seu código, se certifique de executar o comando `cargo build --release` e realizar o benchmark com o executável que encontra-se no diretório *target/release*.

### Cargo as Convention
### Cargo como Convenção

With simple projects, Cargo doesn’t provide a whole lot of value over just
using `rustc`, but it will prove its worth as you continue. With complex
projects composed of multiple crates, it’s much easier to let Cargo coordinate
the build. With Cargo, you can just run `cargo build`, and it should work the
right way. Even though this project is simple, it now uses much of the real
tooling you’ll use for the rest of your Rust career. In fact, you can get
started with virtually all Rust projects you want to work
on with the following commands:

Em projetos simples, Cargo não oferece muito "valor" por assim dizer, você pode apenas utilizar o comando `rustc`. Mas, quando trabalhamos com projetos mais complexos,
compostos de múltiplos crates (pacotes), é bem mais fácil deixar Cargo coordenar
o processo de build. Com Cargo, você pode apenas executar o comando `cargo build`, e tudo deverá funcionar da maneira correta. Apesar do nosso projeto agora ser simples, ele utiliza muito das ferramentas que você irá usar pelo resto da sua carreira no mundo Rust. De fato, você pode começar todos os seus projetos Rust com os seguintes comandos:

```text
$ git clone someurl.com/someproject
$ cd someproject
$ cargo build
```

> Note: If you want to look at Cargo in more detail, check out the official
[Cargo guide], which covers all of its features.

> Nota: Se você deseja saber mais detalhes a respeito do Cargo, confira o Guia Oficial [Cargo guide].

[Cargo guide]: http://doc.crates.io/guide.html
