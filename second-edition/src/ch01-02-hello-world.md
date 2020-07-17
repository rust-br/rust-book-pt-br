## Hello, World!

Now that you’ve installed Rust, let’s write your first Rust program. It’s
traditional when learning a new language to write a little program that prints
the text `Hello, world!` to the screen, so we’ll do the same here!

Agora que você instalou Rust, vamos escrever seu primeiro programa Rust. Quando 
se aprende uma nova linguagem, é tradicional escrever um pequeno programa que imprime 
o texto `Hello, world!` na tela, para que façamos o mesmo aqui!

> Note: This book assumes basic familiarity with the command line. Rust makes
> no specific demands about your editing or tooling or where your code lives, so
> if you prefer to use an integrated development environment (IDE) instead of
> the command line, feel free to use your favorite IDE. Many IDEs now have some
> degree of Rust support; check the IDE’s documentation for details. Recently,
> the Rust team has been focusing on enabling great IDE support, and progress
> has been made rapidly on that front!

> Nota: Este livro pressupõe familiaridade básica com a linha de comando. Rust não requer 
> exigências específicas sobre a sua edição, ferramentas ou a localização do seu código; 
> portanto, se você preferir usar um ambiente de desenvolvimento integrado (IDE) em vez 
> da linha de comando, fique à vontade para usar o seu IDE favorito. Muitos IDEs agora 
> têm algum grau de apoio ao Rust; consulte a documentação do IDE para obter detalhes. 
> Recentemente, a equipe do Rust tem se concentrado em permitir um ótimo suporte a 
> IDE, e houve progresso rápido nessa frente!

### Creating a Project Directory
### Criando um Diretório de Projeto

You’ll start by making a directory to store your Rust code. It doesn’t matter
to Rust where your code lives, but for the exercises and projects in this book,
we suggest making a *projects* directory in your home directory and keeping all
your projects there.

Você começará criando um diretório para armazenar seu código Rust. Não importa 
para Rust onde seu código mora, mas para os exercícios e projetos deste livro, 
sugerimos criar um diretório *projects* no diretório inicial e manter todos os 
seus projetos lá.

Open a terminal and enter the following commands to make a *projects* directory
and a directory for the Hello, world! project within the *projects* directory.

Abra um terminal e digite os seguintes comandos para criar um diretório *projects* 
e um diretório para o projeto Hello, world! dentro do diretório *projects*.

For Linux, macOS, and PowerShell on Windows, enter this:

Para Linux, macOS e PowerShell no Windows, digite o seguinte:

```text
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

For Windows CMD, enter this:

Para o Windows CMD, digite o seguinte:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

### Writing and Running a Rust Program
### Escrevendo e Executando um Programa Rust

Next, make a new source file and call it *main.rs*. Rust files always end with
the *.rs* extension. If you’re using more than one word in your filename, use
an underscore to separate them. For example, use *hello_world.rs* rather than
*helloworld.rs*.

Em seguida, crie um novo arquivo source e chame-o de *main.rs*. Arquivos Rust sempre 
terminam com a extensão ***.rs*** . Se você estiver usando mais de uma palavra no seu nome 
de arquivo, use um sublinhado para separá-las. Por exemplo, use *hello_world.rs* 
em vez de *helloworld.rs*.

Now open the *main.rs* file you just created and enter the code in Listing 1-1.

Agora abra o arquivo *main.rs* que você acabou de criar e insira o código na Listagem 1-1.

<span class="filename">Filename: main.rs</span>

<span class="filename">Nome do arquivo: main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

<span class="caption">Listing 1-1: A program that prints `Hello, world!`</span>

<span class="caption">Listagem 1-1: Um programa que imprime `Hello, world!`</span>

Save the file and go back to your terminal window. On Linux or macOS, enter
the following commands to compile and run the file:

Salve o arquivo e volte para a janela do seu terminal. No Linux ou macOS, digite 
os seguintes comandos para compilar e executar o arquivo:

```text
$ rustc main.rs
$ ./main
Hello, world!
```

On Windows, enter the command `.\main.exe` instead of `./main`:

No Windows, digite o comando `.\main.exe` em vez de `./main`:

```powershell
> rustc main.rs
> .\main.exe
Hello, world!
```

Regardless of your operating system, the string `Hello, world!` should print to
the terminal. If you don’t see this output, refer back to the
[“Troubleshooting”][troubleshooting]<!-- ignore --> part of the Installation
section for ways to get help.

Independentemente do seu sistema operacional, a string `Hello, world!` deve ser 
impressa no terminal. Se você não vir essa saída, consulte a parte 
[“Solução de Problemas”][troubleshooting]<!-- ignore --> da seção instalação para obter 
maneiras de obter ajuda.

If `Hello, world!` did print, congratulations! You’ve officially written a Rust
program. That makes you a Rust programmer—welcome!

Se `Hello, world!` foi impresso, parabéns! Você escreveu oficialmente um programa Rust. 
Isso faz de você um programador Rust — bem-vindo!

### Anatomy of a Rust Program
### Anatomia de um Programa Rust

Let’s review in detail what just happened in your Hello, world! program.
Here’s the first piece of the puzzle:

Vamos analisar em detalhes o que aconteceu no seu programa Hello, world! . 
Aqui está a primeira peça do quebra-cabeça:

```rust
fn main() {

}
```

These lines define a function in Rust. The `main` function is special: it is
always the first code that runs in every executable Rust program. The first
line declares a function named `main` that has no parameters and returns
nothing. If there were parameters, they would go inside the parentheses, `()`.

Essas linhas definem uma função em Rust. A função `main` é especial: é sempre 
o primeiro código executado em todos os programas Rust executáveis. A primeira 
linha declara uma função chamada `main` que não possui parâmetros e não retorna 
nada. Se houvesse parâmetros, eles entrariam entre parênteses, `()`.

Also, note that the function body is wrapped in curly brackets, `{}`. Rust
requires these around all function bodies. It’s good style to place the opening
curly bracket on the same line as the function declaration, adding one space in
between.

Observe também que o corpo da função está entre colchetes, `{}`. Rust exige isso 
em todos os corpos funcionais. É um bom estilo colocar o colchete de abertura na 
mesma linha da declaração de função, adicionando um espaço no meio.

At the time of this writing, an automatic formatter tool called `rustfmt` is
under development. If you want to stick to a standard style across Rust
projects, `rustfmt` will format your code in a particular style. The Rust team
plans to eventually include this tool with the standard Rust distribution, like
`rustc`. So depending on when you read this book, it might already be installed
on your computer! Check the online documentation for more details.

No momento da redação deste artigo, uma ferramenta formatadora automática chamada 
`rustfmt` está em desenvolvimento. Se você deseja manter um estilo padrão nos projetos 
Rust, o `rustfmt` formatará seu código em um estilo específico. A equipe do Rust planeja 
eventualmente incluir essa ferramenta na distribuição padrão do Rust, como `rustc`. 
Portanto, dependendo de quando você ler este livro, ele poderá já estar instalado no 
seu computador! Consulte a documentação online para mais detalhes.

Inside the `main` function is the following code:

Dentro da função `main` está o seguinte código:

```rust
    println!("Hello, world!");
```

This line does all the work in this little program: it prints text to the
screen. There are four important details to notice here. First, Rust style is
to indent with four spaces, not a tab.

Esta linha faz todo o trabalho neste pequeno programa: imprime texto na tela. Há 
quatro detalhes importantes a serem observados aqui. Primeiro, o estilo Rust é 
recuar com quatro espaços, não uma tabulação.

Second, `println!` calls a Rust macro. If it called a function instead, it
would be entered as `println` (without the `!`). We’ll discuss Rust macros in
more detail in Chapter 19. For now, you just need to know that using a `!`
means that you’re calling a macro instead of a normal function.

Segundo, `println!` chama uma macro Rust. Se fosse chamada uma função, ela seria 
inserida como `println` (sem o`!`). Discutiremos Rust macros com mais detalhes no 
Capítulo 19. Por enquanto, você só precisa saber que usar um `!` significa que você 
está chamando uma macro em vez de uma função normal.

Third, you see the `"Hello, world!"` string. We pass this string as an argument
to `println!`, and the string is printed to the screen.

Terceiro, você vê a string `"Hello, world!"`. Passamos essa string como argumento 
para `println!`, e a string é impressa na tela.

Fourth, we end the line with a semicolon (`;`), which indicates that this
expression is over and the next one is ready to begin. Most lines of Rust code
end with a semicolon.

Quarto, terminamos a linha com um ponto-e-vírgula (`;`), que indica que essa expressão 
acabou e a próxima está pronta para começar. A maioria das linhas do código Rust termina 
com um ponto e vírgula.

### Compiling and Running Are Separate Steps
### Compilar e Executar são Etapas Separadas

You’ve just run a newly created program, so let’s examine each step in the
process.

Você acabou de executar um programa recém-criado, portanto, vamos examinar cada etapa do 
processo.

Before running a Rust program, you must compile it using the Rust compiler by
entering the `rustc` command and passing it the name of your source file, like
this:

Antes de executar um programa Rust, você deve compilá-lo usando o compilador Rust digitando 
o comando `rustc` e passando o nome do seu arquivo source, assim:

```text
$ rustc main.rs
```

If you have a C or C++ background, you’ll notice that this is similar to `gcc`
or `clang`. After compiling successfully, Rust outputs a binary executable.

Se você tem experiência em C ou C ++, notará que isso é semelhante a `gcc` ou 
`clang`. Após compilar com sucesso, Rust gera um executável binário.

On Linux, macOS, and PowerShell on Windows, you can see the executable by
entering the `ls` command in your shell. On Linux and macOS, you’ll see two
files. With PowerShell on Windows, you’ll see the same three files that you
would see using CMD.

No Linux, macOS e PowerShell no Windows, você pode ver o executável digitando o 
comando `ls` no seu shell. No Linux e macOS, você verá dois arquivos. Com o 
PowerShell no Windows, você verá os mesmos três arquivos que usaria no CMD.

```text
$ ls
main  main.rs
```

With CMD on Windows, you would enter the following:

Com o CMD no Windows, você digitaria o seguinte:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

This shows the source code file with the *.rs* extension, the executable file
(*main.exe* on Windows, but *main* on all other platforms), and, when using
Windows, a file containing debugging information with the *.pdb* extension.
From here, you run the *main* or *main.exe* file, like this:

Isso mostra o arquivo de código-fonte com a extensão *.rs*, o arquivo executável 
(*main.exe* no Windows, mas *main* em todas as outras plataformas) e, ao usar o 
Windows, um arquivo contendo informações de depuração com o extensão *.pdb*. A 
partir daqui, você executa o arquivo *main* ou *main.exe*, assim:

```text
$ ./main # or .\main.exe on Windows
```

If *main.rs* was your Hello, world! program, this line would print `Hello,
world!` to your terminal.

Se *main.rs* era seu programa Hello, world!, esta linha imprimirá `Hello, world!` 
no seu terminal.

If you’re more familiar with a dynamic language, such as Ruby, Python, or
JavaScript, you might not be used to compiling and running a program as
separate steps. Rust is an *ahead-of-time compiled* language, meaning you can
compile a program and give the executable to someone else, and they can run it
even without having Rust installed. If you give someone a *.rb*, *.py*, or
*.js* file, they need to have a Ruby, Python, or JavaScript implementation
installed (respectively). But in those languages, you only need one command to
compile and run your program. Everything is a trade-off in language design.

Se você está mais familiarizado com uma linguagem dinâmica, como Ruby, Python ou 
JavaScript, pode não estar acostumado a compilar e executar um programa como etapas 
separadas. Rust é uma linguagem *compilada antecipadamente*, o que significa que você 
pode compilar um programa e fornecer o executável para outra pessoa, e eles podem 
executá-lo mesmo sem Rust instalado. Se você fornecer a alguém um arquivo *.rb*, *.py* 
ou *.js*, eles deverão ter uma implementação Ruby, Python ou JavaScript instalada 
(respectivamente). Mas essas linguagens, você só precisa de um comando para compilar 
e executar seu programa. Tudo é uma troca no design da linguagem.

Just compiling with `rustc` is fine for simple programs, but as your project
grows, you’ll want to manage all the options and make it easy to share your
code. Next, we’ll introduce you to the Cargo tool, which will help you write
real-world Rust programs.

Apenas compilar com `rustc` é bom para programas simples, mas à medida que o seu projeto 
cresce, você deseja gerenciar todas as opções e facilitar o compartilhamento do seu código. 
Em seguida, apresentaremos a ferramenta Cargo, que ajudará você a criar programas Rust 
no mundo real.

[troubleshooting]: ch01-01-installation.html#troubleshooting

[Solução de Problemas]: ch01-01-installation.html#solucao-de-problemas
