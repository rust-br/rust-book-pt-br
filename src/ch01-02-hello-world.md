<!--
## Hello, World!

Now that you’ve installed Rust, let’s write your first Rust program. It’s
traditional when learning a new language to write a little program that prints
the text `Hello, world!` to the screen, so we’ll do the same here!

> Note: This book assumes basic familiarity with the command line. Rust makes
> no specific demands about your editing or tooling or where your code lives, so
> if you prefer to use an integrated development environment (IDE) instead of
> the command line, feel free to use your favorite IDE. Many IDEs now have some
> degree of Rust support; check the IDE’s documentation for details. Recently,
> the Rust team has been focusing on enabling great IDE support, and progress
> has been made rapidly on that front!
-->


## Olá, Mundo!

Agora que Rust já está instalado, vamos escrever seu primeiro programa Rust.
Quando aprendemos uma nova linguagem de programação, é tradicional escrever um
pequeno programa que imprime "Olá, mundo!" (_“Hello, world!”_) na tela, e vamos
fazer o mesmo aqui.

> Nota: Este livro assume que o leitor tem um pouco de familiaridade com a
> linha de comando. Rust não faz nenhuma demanda a respeito do seu editor ou
> ferramentas ou sobre onde seu código mora, então se você preferir usar uma
> IDE ao invés da linha de comando, sinta-se livre para usar sua IDE favorita.
> Muitas IDEs têm agora algum nível de suporte a Rust. Cheque  a documentação
> da IDE para detalhes. Recentemente o time Rust tem focado em habilitar um
> ótimo suporte a IDEs, e progresso tem sido rapidamente feito nesse front!

<!--
### Creating a Project Directory

You’ll start by making a directory to store your Rust code. It doesn’t matter
to Rust where your code lives, but for the exercises and projects in this book,
we suggest making a *projects* directory in your home directory and keeping all
your projects there.
-->

### Criando um Diretório de Projeto

Você irá começar criando um diretório para guardar seu código Rust. Não importa
para Rust onde seu código vive, mas para os exercícios e projetos neste livro,
nós sugerimos criar um diretório _projects_ no seu diretório principal e manter
todo o seus projetos lá.

<!--
Open a terminal and enter the following commands to make a *projects* directory
and a directory for the “Hello, world!” project within the *projects* directory.

For Linux, macOS, and PowerShell on Windows, enter this:
-->

Abra um terminal e digite os seguintes comandos para criar um diretório
_projects_ e um diretório para o projeto "Hello, world!" dentro do diretório
_projects_.

Para Linux, macOS, e PowerShell no Windows, digite o seguinte:

```text
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

<!--
For Windows CMD, enter this:

-->

No CMD do Windows, digite o seguinte:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

<!--
### Writing and Running a Rust Program

Next, make a new source file and call it *main.rs*. Rust files always end with
the *.rs* extension. If you’re using more than one word in your filename, use
an underscore to separate them. For example, use *hello_world.rs* rather than
*helloworld.rs*.

Now open the *main.rs* file you just created and enter the code in Listing 1-1.
-->

### Escrevendo e Executando um Programa Rust

Crie um novo arquivo *main.rs*. Arquivos Rust sempre terminam com a extensão
*.rs*. Se o nome do seu arquivo tem mais de uma palavra, use um _underscore_
para separá-los. Por exemplo, você deve nomear o seu arquivo *hello_world.rs*
em vez de *helloworld.rs*.

Agora abra o arquivo *main.rs* que você acabou de criar e digite o código na
Listagem 1-1:


<span class="filename">Filename: main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

<!--
<span class="caption">Listing 1-1: A program that prints `Hello, world!`</span>

Save the file and go back to your terminal window. On Linux or macOS, enter
the following commands to compile and run the file:
-->

<span class="caption">Listagem 1-1: Um programa que imprime `Hello, world!`</span>

Salve o arquivo e volte ao seu terminal. No Linux ou macOS, digite os seguintes
comandos para compilar e rodar o arquivo:

```text
$ rustc main.rs
$ ./main
Hello, world!
```

<!--
On Windows, enter the command `.\main.exe` instead of `./main`:

-->

No Windows, digite o comando `.\main.exe` ao invés de `./main`:

```powershell
> rustc main.rs
> .\main.exe
Hello, world!
```

<!--
Regardless of your operating system, the string `Hello, world!` should print to
the terminal. If you don’t see this output, refer back to the
[“Troubleshooting”][troubleshooting] part of the Installation
section for ways to get help.

If `Hello, world!` did print, congratulations! You’ve officially written a Rust
program. That makes you a Rust programmer—welcome!
-->

Independente do seu sistema operacional, você deverá ver a mensagem
`Hello, world!` no seu terminal. Se você não ver essa mensagem, volte à parte
[“Troubleshooting”][troubleshooting] da seção "Instalação" sobre formas de
obter ajuda. 

Se `Hello, world!` apareceu, parabéns! Você oficialmente escreveu um programa
Rust. Isso faz de você um programador Rust! Seja bem-vindo!

<!--
### Anatomy of a Rust Program

Let’s review in detail what just happened in your “Hello, world!” program.
Here’s the first piece of the puzzle:

```rust
fn main() {

}
```

These lines define a function in Rust. The `main` function is special: it is
always the first code that runs in every executable Rust program. The first
line declares a function named `main` that has no parameters and returns
nothing. If there were parameters, they would go inside the parentheses, `()`.
-->

### Anatomia de um Programa em Rust

Agora vamos ver o que aconteceu com o seu programa "Hello, world!" em detalhes.
Aqui está a primeira peça do quebra-cabeça:

```rust
fn main() {

}
```

Estas linhas definem uma função em Rust. A função `main` é especial: ela é
sempre o primeiro código a ser executado em todo programa executável Rust.
A primeira linha declara uma função chamada `main` que não tem nenhum parâmetro
e não retorna nada. Se houvessem parâmetros, eles seriam inseridos dentro dos
parênteses, `()`.

<!-- 
Also, note that the function body is wrapped in curly brackets, `{}`. Rust
requires these around all function bodies. It’s good style to place the opening
curly bracket on the same line as the function declaration, adding one space in
between.

At the time of this writing, an automatic formatter tool called `rustfmt` is
under development. If you want to stick to a standard style across Rust
projects, `rustfmt` will format your code in a particular style. The Rust team
plans to eventually include this tool with the standard Rust distribution, like
`rustc`. So depending on when you read this book, it might already be installed
on your computer! Check the online documentation for more details.

Inside the `main` function is the following code:
-->

Também, note que o corpo da função é envolvido em chaves, `{}`. Rust as requer
ao redor de todos os corpos de função. É boa prática colocar a chave de
abertura na mesma linha que a declaração da função, adicionando um espaço entre
elas.

No momento desta escrita, uma ferramenta de formatação automática chamada
`rustfmt` está sendo desenvolvida. Se você quiser se ater a um estilo padrão
através de projetos Rust, `rustfmt` irá formatar seu código num estilo
particular. O time Rust planeja eventualmente incluir esta ferramenta com a
distribuição padrão de Rust, assim como o `rustc`. Então dependendo de quando
você ler este livro, ela pode já estar instalada no seu computador! Cheque a
documentação online para mais detalhes.

Dentro da função `main` está o seguinte código:

```rust
    println!("Hello, world!");
```

<!--
This line does all the work in this little program: it prints text to the
screen. There are four important details to notice here. First, Rust style is
to indent with four spaces, not a tab.
-->

Esta linha faz todo o trabalho neste pequeno programa: ela imprime texto na
tela. Há quatro detalhes importantes a se notar aqui. Primeiro, o estilo Rust
é indentar com quatro espaços, não com uma tabulação (tab).

<!--
Second, `println!` calls a Rust macro. If it called a function instead, it
would be entered as `println` (without the `!`). We’ll discuss Rust macros in
more detail in Chapter 19. For now, you just need to know that using a `!`
means that you’re calling a macro instead of a normal function.
-->

Em segundo, `println!` chama uma macro Rust. Se ele chamasse uma função ao
invés disso, ele seria escrito como `println` (sem a `!`). Nós iremos discutir
macros Rust em mais detalhes no capítulo 19. Por agora, você só precisa saber
que usar uma `!` significa que você está chamando uma macro ao invés de uma
função normal.

<!--
Third, you see the `"Hello, world!"` string. We pass this string as an argument
to `println!`, and the string is printed to the screen.
-->

Em terceiro, você vê a string `"Hello, world!"`. Nós passamos essa string como
um argumento para o `println!`, e a string é impressa na tela.

<!--
Fourth, we end the line with a semicolon (`;`), which indicates that this
expression is over and the next one is ready to begin. Most lines of Rust code
end with a semicolon.
-->

Em quarto, nós terminamos a linha com um ponto e vírgula (`;`), que indica que
essa expressão terminou e a próxima está pronta para começar. A maioria das
linhas de Rust terminam com um ponto e vírgula.

<!--
### Compiling and Running Are Separate Steps

You’ve just run a newly created program, so let’s examine each step in the
process.

Before running a Rust program, you must compile it using the Rust compiler by
entering the `rustc` command and passing it the name of your source file, like
this:
-->

### Compilação e Execução São Etapas Diferentes

Você acabou de rodar um programa récem-criado, então vamos analisar cada etapa
no processo.

Antes de rodar um programa Rust, você deve compilá-lo usando o compilador Rust
ao digitar o comando `rustc` e passar a ele o nome do arquivo fonte, desta
forma:

```text
$ rustc main.rs
```

<!--
If you have a C or C++ background, you’ll notice that this is similar to `gcc`
or `clang`. After compiling successfully, Rust outputs a binary executable.

On Linux, macOS, and PowerShell on Windows, you can see the executable by
entering the `ls` command in your shell. On Linux and macOS, you’ll see two
files. With PowerShell on Windows, you’ll see the same three files that you
would see using CMD.

-->

Se você tem experiência em C ou C++, vocẽ irá notar que isso é similar ao `gcc`
ou ao `clang`. Depois de compilar com sucesso, Rust produz um executável
binário.

Em Linux, macOS, e PowerShell no Windows, você pode ver o executável ao entrar
o comando `ls` no seu shell. Em Linux e macOS, você irá ver dois arquivos. No
PowerShell no Windows, você irá ver os mesmos três arquivos que você veria
usando CMD.

```text
$ ls
main  main.rs
```

<!--
With CMD on Windows, you would enter the following:
-->

No CMD no Windows, você digitaria o seguinte:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

<!--
This shows the source code file with the *.rs* extension, the executable file
(*main.exe* on Windows, but *main* on all other platforms), and, when using
Windows, a file containing debugging information with the *.pdb* extension.
From here, you run the *main* or *main.exe* file, like this:
-->

Isto mostra o código fonte com a extensão _.rs_, o arquivo executável (
_main.exe_ no Windows, mas _main_ em todas as outras plataformas), e, usando
Windows, um arquivo contendo informação de debugging com a extensão _.pdb_.
Daqui, você roda os arquivos _main_ ou _main.exe_, da seguinte forma:

```text
$ ./main # or .\main.exe on Windows
```

<!--
If *main.rs* was your “Hello, world!” program, this line would print `Hello,
world!` to your terminal.
-->

Se _main.rs_ era o seu programa "Hello, world!", esta linha irá imprimir
`Hello, world!` no seu terminal.

<!-- 
If you’re more familiar with a dynamic language, such as Ruby, Python, or
JavaScript, you might not be used to compiling and running a program as
separate steps. Rust is an *ahead-of-time compiled* language, meaning you can
compile a program and give the executable to someone else, and they can run it
even without having Rust installed. If you give someone a *.rb*, *.py*, or
*.js* file, they need to have a Ruby, Python, or JavaScript implementation
installed (respectively). But in those languages, you only need one command to
compile and run your program. Everything is a trade-off in language design.
-->

Se você é mais familiar com uma linguagem dinâmica, como Ruby, Python, ou
Javascript, você pode não estar acostumado a compilar e rodar um programa como
etapas separadas. Rust é uma linguagem compilada _ahead of time_ 
(antecipadamente), o que significa que você pode compilar um programa e dar o
executável para outra pessoa, e ela pode rodar o programa mesmos sem ter Rust
instalada. Se você der a alguém um arquivo _.rb_, _.py_, ou _.js_, ela irá
precisar ter uma implementação de Ruvy, Python, ou Javascript instaladas
respectivamente. Mas nessas linguagens, você apenas precisa de um comando
para compilar e rodar seu programa. Tudo é um trade-off no projeto de
linguagens.

<!--
Just compiling with `rustc` is fine for simple programs, but as your project
grows, you’ll want to manage all the options and make it easy to share your
code. Next, we’ll introduce you to the Cargo tool, which will help you write
real-world Rust programs.
-->

Apenas compilando com `rustc` é ok para programas simples, mas quando seu
projeto crescer, você pode querer gerenciar todas as opções e tornar fácil
compartilhar seu código. Em seguida, nós iremos introduzir à ferramenta Cargo,
que irá ajudá-lo a escrever programas Rust reais.

[troubleshooting]: ch01-01-installation.html#troubleshooting
