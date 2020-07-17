## Installation
## Instalação

The first step is to install Rust. We’ll download Rust through `rustup`, a
command line tool for managing Rust versions and associated tools. You’ll need
an internet connection for the download.

O primeiro passo é instalar Rust. Vamos fazer o download de Rust através do `rustup`, uma
ferramenta de linha de comando para gerenciar versões Rust e ferramentas associadas. 
Você precisará de uma conexão com a Internet para o download.

> Note: If you prefer not to use `rustup` for some reason, please see [the Rust
> installation page](https://www.rust-lang.org/tools/install) for other options.

> Nota: Se você preferir não usar o `rustup` por algum motivo, consulte [a página de 
> instalação de Rust](https://www.rust-lang.org/pt-BR/tools/install) para outras opções.

The following steps install the latest stable version of the Rust compiler.
Rust’s stability guarantees ensure that all the examples in the book that
compile will continue to compile with newer Rust versions. The output might
differ slightly between versions, because Rust often improves error messages
and warnings. In other words, any newer, stable version of Rust you install
using these steps should work as expected with the content of this book.

As etapas a seguir instalam a versão estável mais recente do compilador Rust. As garantias 
de estabilidade de Rust garantem que todos os exemplos do livro que compilam continuem 
sendo compilados com as versões mais recentes de Rust. A saída pode diferir ligeiramente 
entre as versões, porque Rust geralmente melhora as mensagens de erro e os avisos. Em outras 
palavras, qualquer versão mais recente e estável de Rust instalada usando essas etapas deve 
funcionar conforme o esperado com o conteúdo deste livro.

> ### Command Line Notation
>
> In this chapter and throughout the book, we’ll show some commands used in the
> terminal. Lines that you should enter in a terminal all start with `$`. You
> don’t need to type in the `$` character; it indicates the start of each
> command. Lines that don’t start with `$` typically show the output of the
> previous command. Additionally, PowerShell-specific examples will use `>`
> rather than `$`.

> ### Notação de Linha de Comando
>
> Neste capítulo e ao longo do livro, mostraremos alguns comandos usados no terminal. As 
> linhas que você deve inserir em um terminal começam com `$`. Você não precisa digitar o caractere 
> `$`; indica o início de cada comando. As linhas que não começam com `$` normalmente mostram a 
> saída do comando anterior. Além disso, exemplos específicos em PowerShell usarão `>` em vez de `$`.

### Installing `rustup` on Linux or macOS
### Instalando `rustup` no Linux ou macOS

If you’re using Linux or macOS, open a terminal and enter the following command:

Se você estiver usando Linux ou macOS, abra um terminal e digite o seguinte comando:

```text
$ curl https://sh.rustup.rs -sSf | sh
```

The command downloads a script and starts the installation of the `rustup`
tool, which installs the latest stable version of Rust. You might be prompted
for your password. If the install is successful, the following line will appear:

O comando baixa um script e inicia a instalação da ferramenta `rustup`, que instala a 
versão estável mais recente de Rust. Você pode ser solicitado a fornecer sua senha. Se a 
instalação for bem-sucedida, a seguinte linha aparecerá:

```text
Rust is installed now. Great!
```

If you prefer, feel free to download the script and inspect it before running
it.

Se preferir, faça o download do script e inspecione-o antes de executá-lo.

The installation script automatically adds Rust to your system PATH after your
next login. If you want to start using Rust right away instead of restarting
your terminal, run the following command in your shell to add Rust to your
system PATH manually:

O script de instalação adiciona Rust automaticamente ao PATH do sistema após seu próximo 
login. Se você deseja começar a usar Rust imediatamente, em vez de reiniciar o terminal, 
execute o seguinte comando no shell para adicionar Rust ao PATH do sistema manualmente:

```text
$ source $HOME/.cargo/env
```

Alternatively, you can add the following line to your _~/.bash_profile_:

Como alternativa, você pode adicionar a seguinte linha ao seu _~/.bash_profile_:

```text
$ export PATH="$HOME/.cargo/bin:$PATH"
```

Additionally, you’ll need a linker of some kind. It’s likely one is already
installed, but when you try to compile a Rust program and get errors indicating
that a linker could not execute, that means a linker isn’t installed on your
system and you’ll need to install one manually. C compilers usually come with
the correct linker. Check your platform’s documentation for how to install a C
compiler. Also, some common Rust packages depend on C code and will need a C
compiler. Therefore, it might be worth installing one now.

Além disso, você precisará de um *linker* de algum tipo. Provavelmente já está
instalado, mas quando você tenta compilar um programa Rust e obtem erros, indicando 
que um linker não pôde executar, isso significa que um linker não está instalado no 
seu sistema e você precisará instalá-lo manualmente. Os compiladores C geralmente vêm 
com o linker correto. Verifique a documentação da sua plataforma para saber como instalar 
um compilador C. Além disso, alguns pacotes Rust comuns dependem do código C e precisarão 
de um compilador C. Portanto, pode valer a pena instalar um agora.

### Installing `rustup` on Windows
### Instalando `rustup` no Windows

On Windows, go to [https://www.rust-lang.org/tools/install][install] and follow
the instructions for installing Rust. At some point in the installation, you’ll
receive a message explaining that you’ll also need the C++ build tools for
Visual Studio 2013 or later. The easiest way to acquire the build tools is to
install [Build Tools for Visual Studio 2019][visualstudio]. The tools are in
the Other Tools and Frameworks section.

No Windows, vá para [https://www.rust-lang.org/pt-BR/tools/install][install-br] e siga as instruções 
para instalar Rust. Em algum momento da instalação, você receberá uma mensagem explicando 
que também precisará das ferramentas de *build* do C ++ para o Visual Studio 2013 ou posterior. 
A maneira mais fácil de adquirir as ferramentas de build é instalar [Ferramentas Integradas do Visual 
Studio 2019][visualstudio-br]. ~~As ferramentas estão na seção: Outras Ferramentas e Estruturas.~~

[install]: https://www.rust-lang.org/tools/install
[visualstudio]: https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2019

[install-br]: https://www.rust-lang.org/pt-BR/tools/install
[visualstudio-br]: https://visualstudio.microsoft.com/pt-br/downloads/

The rest of this book uses commands that work in both _cmd.exe_ and PowerShell.
If there are specific differences, we’ll explain which to use.

O restante deste livro usa comandos que funcionam no _cmd.exe_ e no PowerShell. Se houver 
diferenças específicas, explicaremos qual usar.

### Updating and Uninstalling
### Atualização e Desinstalação

After you’ve installed Rust via `rustup`, updating to the latest version is
easy. From your shell, run the following update script:

Depois de instalar o Rust via `rustup`, é fácil atualizar para a versão mais recente. No 
seu shell, execute o seguinte script de atualização:

```text
$ rustup update
```

To uninstall Rust and `rustup`, run the following uninstall script from your
shell:

Para desinstalar o Rust e o `rustup`, execute o seguinte script de desinstalação do seu shell:

```text
$ rustup self uninstall
```

### Troubleshooting
### Solução de Problemas

To check whether you have Rust installed correctly, open a shell and enter this
line:

Para verificar se você possui Rust instalado corretamente, abra um shell e digite esta linha:

```text
$ rustc --version
```

You should see the version number, commit hash, and commit date for the latest
stable version that has been released in the following format:

Você deverá ver o número da versão, *commit* hash, e *commit* da data da versão estável 
mais recente lançada no seguinte formato:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

If you see this information, you have installed Rust successfully! If you don’t
see this information and you’re on Windows, check that Rust is in your `%PATH%`
system variable. If that’s all correct and Rust still isn’t working, there are
a number of places you can get help. The easiest is the #beginners channel on
[the official Rust Discord][discord]. There, you can chat with other Rustaceans
(a silly nickname we call ourselves) who can help you out. Other great
resources include [the Users forum][users] and [Stack Overflow][stackoverflow].

Se você visualizar essas informações, instalou Rust com sucesso! Se você não vir essas 
informações e estiver no Windows, verifique se Rust está na sua variável de sistema 
`%PATH%`. Se tudo estiver correto e Rust ainda não estiver funcionando, há vários 
lugares onde você pode obter ajuda. O mais fácil é o canal #beginners em 
[the official Rust Discord][discord]. Lá, você pode conversar com outros *Rustaceans* 
(um apelido bobo que chamamos de nós mesmos) que podem ajudá-lo. Outros ótimos recursos 
incluem [the Users forum][users] e [Stack Overflow][stackoverflow].

[discord]: https://discord.gg/rust-lang
[users]: https://users.rust-lang.org/
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust

### Local Documentation
### Documentação Local

The installer also includes a copy of the documentation locally, so you can
read it offline. Run `rustup doc` to open the local documentation in your
browser.

O instalador também inclui uma cópia da documentação localmente, para que você possa lê-la 
offline. Execute `rustup doc` para abrir a documentação local no seu navegador.

Any time a type or function is provided by the standard library and you’re not
sure what it does or how to use it, use the application programming interface
(API) documentation to find out!

Sempre que um tipo ou função for fornecida pela biblioteca padrão e você não tiver certeza 
do que esta faz ou como usá-la, use a documentação da interface de programação de aplicativos 
(API) para descobrir!