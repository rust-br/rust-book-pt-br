<!--
## Installation

The first step is to install Rust. We’ll download Rust through `rustup`, a
command line tool for managing Rust versions and associated tools. You’ll need
an internet connection for the download.

> Note: If you prefer not to use `rustup` for some reason, please see [the Rust
> installation page](https://www.rust-lang.org/tools/install) for other options.

The following steps install the latest stable version of the Rust compiler.
Rust’s stability guarantees ensure that all the examples in the book that
compile will continue to compile with newer Rust versions. The output might
differ slightly between versions, because Rust often improves error messages
and warnings. In other words, any newer, stable version of Rust you install
using these steps should work as expected with the content of this book.
-->

## Instalação

O primeiro passo é instalar Rust. Nós vamos baixar Rust através do `rustup`,
uma ferramenta de linha de comando para gerenciar versões de Rust e ferramentas
associadas. Você vai precisar de uma conexão com a internet para executar o
download.

> Nota: se você prefere não usar o `rustup` por algum motivo, por favor veja o
> [a página de instalação de Rust](https://www.rust-lang.org/tools/install)
> para outras opções.

Os próximos passos irão instalar a última versão estável do compilador de Rust.
A garantias de estabilidade de Rust certificam-se de que todos os exemplos
neste livro que compilam vao continuar a compilar com novas versões de Rust.
As saídas podem variar um pouco de versão a versão, porque Rust frequentemente
melhora mensagens de erro e alertas. Em outras palavras, qualquer versão nova,
estável, de Rust qu você instalar usando esses passos devem funcionar como o
esperado com o conteúdo deste livro.

<!--
> ### Command Line Notation
>
> In this chapter and throughout the book, we’ll show some commands used in the
> terminal. Lines that you should enter in a terminal all start with `$`. You
> don’t need to type in the `$` character; it indicates the start of each
> command. Lines that don’t start with `$` typically show the output of the
> previous command. Additionally, PowerShell-specific examples will use `>`
> rather than `$`.
-->

> ### Notação de Linha de Comando
>
> Neste capítulo e ao longo do livro, nós iremos mostrar alguns comandos usados
> no terminal. Linhas que você deve digitar num terminal todas começam com `$`.
> Você não precisa digitar o caráctere `$`. Ele indica o início de cada comando.
> Linhas que não comecem com `$` tipicamente mostram a saída do comando anterior.
> Adicionalmente, exemplos específicos de PowerShell irão utilizar `>` ao invés
> de `$`.

<!--
### Installing `rustup` on Linux or macOS

If you’re using Linux or macOS, open a terminal and enter the following command:
-->

### Instalando `rustup` em Linux ou Mac

Se você utiliza Linux ou Mac, abra um terminal e digite o seguinte comando:

```text
$ curl https://sh.rustup.rs -sSf | sh
```

<!--
The command downloads a script and starts the installation of the `rustup`
tool, which installs the latest stable version of Rust. You might be prompted
for your password. If the install is successful, the following line will appear:
-->

Este comando baixa um script e inicia a instalação da ferramenta `rustup`, a
qual instala a última versão estável de Rust. Talvez seja solicitado que você
digite sua senha. Se a instalação for bem-sucedida, a seguinte linha irá
aparecer:

```text
Rust is installed now. Great!
```

<!--
If you prefer, feel free to download the script and inspect it before running
it.

The installation script automatically adds Rust to your system PATH after your
next login. If you want to start using Rust right away instead of restarting
your terminal, run the following command in your shell to add Rust to your
system PATH manually:
-->

Se preferir, sinta-se livre para baixar o script e inspecioná-lo antes de
executar.

O script de instalação adiciona automaticamente Rust à variável PATH do
seu ambiente após o seu próximo login. Se você quiser usar Rust imediatamente
ao invés de reiniciar o seu terminal, execute o seguinte comando no seu shell
para adicionar Rust à PATH do seu sistema manualmente:

```text
$ source $HOME/.cargo/env
```

<!--
Alternatively, you can add the following line to your *~/.bash_profile*:
-->

Outra opção é adicionar a linha abaixo ao seu `~/.bash_profile`:

```text
$ export PATH="$HOME/.cargo/bin:$PATH"
```

<!--
Additionally, you’ll need a linker of some kind. It’s likely one is already
installed, but when you try to compile a Rust program and get errors indicating
that a linker could not execute, that means a linker isn’t installed on your
system and you’ll need to install one manually. C compilers usually come with
the correct linker. Check your platform’s documentation for how to install a C
compiler. Also, some common Rust packages depend on C code and will need a C
compiler. Therefore, it might be worth installing one now.
-->

<!-- TODO traduzir o trecho acima -->

### Instalando no Windows

<!--
### Installing `rustup` on Windows

On Windows, go to [https://www.rust-lang.org/tools/install][install] and follow
the instructions for installing Rust. At some point in the installation, you’ll
receive a message explaining that you’ll also need the C++ build tools for
Visual Studio 2013 or later. The easiest way to acquire the build tools is to
install [Build Tools for Visual Studio 2019][visualstudio]. The tools are in
the Other Tools and Frameworks section.
-->

No Windows, vá até [https://www.rust-lang.org/tools/install][install] e siga
as instruções para instalar Rust. Em algum ponto da instalação, você receberá
uma mensagem explicando que você também irá precisar de ferramentas de
compilação de C++ para o Visual Studio 2013 ou superior. A forma mais fácil de
adquiri-las é instalando as [Ferramentas de Compilação para o Visual 
Studio 2019][visualstudio]. As ferramentas estão na seção Other Tools and
Frameworks.

[install]: https://www.rust-lang.org/tools/install
[visualstudio]: https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2019

<!--
The rest of this book uses commands that work in both *cmd.exe* and PowerShell.
If there are specific differences, we’ll explain which to use.

### Updating and Uninstalling

After you’ve installed Rust via `rustup`, updating to the latest version is
easy. From your shell, run the following update script:
-->

O restante deste livro utiliza comandos que funcionam tanto no *cmd.exe* quanto
no PowerShell. Se existirem diferenças específicas, nós iremos apontar qual 
utilizar.

### Atualizando e desinstalando

Após instalar Rust pelo `rustup`, atualizar para a última versão é fácil.
Do seu seu shell, rode o seguinte script de atualização:

```text
$ rustup update
```

<!--
To uninstall Rust and `rustup`, run the following uninstall script from your
shell:
-->

Para desinstalar Rust e o `rustup`, rode o seguinte script de desinstalação do
seu shell:

```text
$ rustup self uninstall
```

<!--
### Troubleshooting

To check whether you have Rust installed correctly, open a shell and enter this
line:
-->

### Solução de Problemas

Para checar se você tem Rust instalado corretamente, abra um terminal e digite
esta linha:

```text
$ rustc --version
```

<!--
You should see the version number, commit hash, and commit date for the latest
stable version that has been released in the following format:
-->

Você deverá ver o número, o hash e a data do commit da última versão estável
lançada no seguinte formato:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

<!--
If you see this information, you have installed Rust successfully! If you don’t
see this information and you’re on Windows, check that Rust is in your `%PATH%`
system variable. If that’s all correct and Rust still isn’t working, there are
a number of places you can get help. The easiest is the #beginners channel on
[the official Rust Discord][discord]. There, you can chat with other Rustaceans
(a silly nickname we call ourselves) who can help you out. Other great
resources include [the Users forum][users] and [Stack Overflow][stackoverflow].
-->

Se aparecer a mensagem acima, você instalou Rust com sucesso! Se você não ver
esta informação e você estiver no Windows, cheque que Rust está na sua variável
de ambiente `%PATH%`. Se tudo isso estiver correto e Rust ainda não funcionar,
existem vários locais em que você pode pedir ajuda. O mais fácil é o canal
#beginners no [Discord oficial de Rust][discord]. Lá, você pode falar com
outros Rustáceos (um apelido bobo que damos a nós mesmos) que podem te ajudar.
Outros ótimos recursos incluem [o fórum de usários][users] e o [Stack
Overflow][stackoverflow].

[discord]: https://discord.gg/rust-lang
[users]: https://users.rust-lang.org/
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust

<!--
### Local Documentation

The installation of Rust also includes a copy of the documentation locally, so
you can read it offline. Run `rustup doc` to open the local documentation in
your browser.

Any time a type or function is provided by the standard library and you’re not
sure what it does or how to use it, use the application programming interface
(API) documentation to find out!
-->
### Documentação local

O instalador também inclui uma cópia local da documentação para que você possa
acessá-la offline. Execute o comando `rustup doc` para abrir a documentação no
seu navegador.

Sempre que um tipo ou for fornecido pela biblioteca padrão e você não tiver
certeza do que ele faz ou de como usá-lo, use a documentação da API para
descobrir!
