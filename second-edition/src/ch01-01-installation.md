## Instalação

The first step to using Rust is to install it. You’ll need an internet
connection to run the commands in this chapter, as we’ll be downloading Rust
from the internet.

O primeiro passo para se usar a linguagem Rust é instalá-la. Você vai precisar de uma conexão
com a internet para executar os comandos contidos neste capítulo, assim como para fazer o download de Rust.

We’ll be showing off a number of commands using a terminal, and those lines all
start with `$`. You don’t need to type in the `$` character; they are there to indicate
the start of each command. You’ll see many tutorials and examples around the web
that follow this convention: `$` for commands run as a regular user, and `#`
for commands you should be running as an administrator. Lines that don’t start
with `$` are typically showing the output of the previous command.

Iremos mostrar diversos comandos usando o terminal, e todas as linhas que se referem ao terminal
iniciam com `$`. Você não precisa digitar o caractere `$` ; ele serve para indicar o início
de cada comando. Você vai notar em muitos tutorias disponíveis por aí que eles seguem
esta convenção: `$` para comandos que são executados como um usuário normal, e `#`
para comandos que você deve executar como administrador. As linhas que não iniciam com
o caractere `$` mostram o output (saída) do comando anterior.


### Installing on Linux or Mac
### Instalando no Linux ou Mac

If you’re on Linux or a Mac, all you need to do is open a terminal and type
this:

Se você está em um ambiente Linux ou Mac, tudo o que você precisa é abrir seu Terminal e digitar o seguinte comando:

```text
$ curl https://sh.rustup.rs -sSf | sh
```

Este comando vai baixar um script e iniciar a instalação. Talvez um prompt vai aparecer na sua tela requerendo o seu password. Se tudo ocorrer bem, a mensagem abaixo vai aparecer:

```text
Rust is installed now. Great!
```

Mas é claro, que se você não quiser usar o `curl | sh`, você pode fazer o download do script, inspecioná-lo e executá-lo da maneira que achar melhor.

O script de instalação já adiciona automaticamente o Rust ao seu system PATH logo após o seu próximo login.  Mas se você quiser usar Rust imediatamente, execute o seguinte comando no seu terminal:

```text
$ source $HOME/.cargo/env
```

Uma outra opção é adicionar a linha abaixo no seu `~/.bash_profile`:

```text
$ export PATH="$HOME/.cargo/bin:$PATH"
```

### Installing on Windows
### Instalando no Windows

Em um ambiente Windows, vá até o endereço [https://rustup.rs](https://rustup.rs/)<!-- ignore --> e
siga as instruções para realizar o download do arquivo rustup-init.exe. Execute este arquivo e siga
as instruções que aparecerem na sua tela.

O restante dos comandos específicos para um ambiente Windows neste livro, partem da premissa que você está
utilizando o `cmd` como o seu shell. Se você está usando um shell diferente, talvez você poderá usar os
mesmos comandos que um usuário Linux ou de Mac usa. Se algum comando não funcionar, consulte a
documentação referente ao shell que você está utilizando.

### Custom installations
### Instalação customizada

Se por alguma razão você preferir não usar o rustup.rs, por favor consulte [the Rust
installation page](https://www.rust-lang.org/install.html) para outras opções de instalação.

### Updating
### Atualizando o Rust

Se você já tem Rust instalado na sua máquina, atualizar para a última versão é muito fácil.
Do seu shell, execute script de atualização:

```text
$ rustup update
```

### Uninstalling
### Desinstalando

Uninstalling Rust is as easy as installing it. From your shell, run
the uninstall script:

Desinstalar Rust é tão fácil quanto instalá-lo. Do seu shell, execute
o script de desinstalação:

```text
$ rustup self uninstall
```

### Troubleshooting

If you’ve got Rust installed, you can open up a shell, and type this:

Após Rust ser instalado em sua máquina, você pode abrir o seu shell, e digitar a linha abaixo:

```text
$ rustc --version
```

You should see the version number, commit hash, and commit date in a format
similar to this for the latest stable version at the time you install:

Você deverá ver a versão, o hash de commit, e a data do commit em um formato
similar ao seguinte indicando a última versão estável que você acabou de instalar no seu ambiente:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

If you see this, Rust has been installed successfully!
Congrats!

Se aparecer a mensagem acima, Rust foi instalado com sucesso!
Parabéns!

If you don’t and you’re on Windows, check that Rust is in your `%PATH%` system
variable.

Se nao aparecer a mensagem acima e você está em um ambiente Windows, verifique nas suas variáveis de ambiente se Rust aparece no seu`%PATH%`.

If it still isn’t working, there are a number of places where you can get help.
The easiest is [the #rust IRC channel on irc.mozilla.org][irc]<!-- ignore -->,
which you can access through [Mibbit][mibbit]. Go to that address, and you’ll
be chatting with other Rustaceans (a silly nickname we call ourselves) who can
help you out. Other great resources include [the Users forum][users] and
[Stack Overflow][stackoverflow].

Se mesmo assim ainda não funcionar, existem varios lugares aonde você pode pedir ajuda.
A maneira mais fácil é pedir ajuda no [the #rust IRC channel on irc.mozilla.org][irc]<!-- ignore -->,
que você pode acessar via [Mibbit][mibbit]. Você vai falar com vários outros Rustaceans (que é a maneira que chamamos aqueles que programam em Rust) que podem ajudá-lo em suas dúvidas. 
Você também pode buscar ajuda no [the Users forum][users] e no
[Stack Overflow][stackoverflow].

[irc]: irc://irc.mozilla.org/#rust
[mibbit]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[users]: https://users.rust-lang.org/
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust

### Local documentation
### Documentação local

The installer also includes a copy of the documentation locally, so you can
read it offline. Run `rustup doc` to open the local documentation in your
browser.

O instalador também inclui uma cópia da documentação, para que você possa acessá-la offline. Execute o comando `rustup doc` para abrir a documentação local no seu browser.

Any time there’s a type or function provided by the standard library and you’re
not sure what it does, use the API documentation to find out!

Toda vez que se reparar com um tipo ou função fornecido pela biblioteca padrão e você não tiver certeza do que ela faz, use a documentação para descobrir mais a respeito!
