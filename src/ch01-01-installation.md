## Instalação

O primeiro passo é instalar Rust. Vamos fazer o download de Rust através do `rustup`, uma ferramenta de linha de comando para gerenciar versões Rust e ferramentas associadas. 
Você precisará de uma conexão com a Internet para o download.

> Nota: Se você preferir não usar o `rustup` por algum motivo, consulte [a página de instalação de Rust](https://www.rust-lang.org/pt-BR/tools/install) para outras opções.

As etapas a seguir instalam a versão estável mais recente do compilador Rust. As garantias de estabilidade de Rust garantem que todos os exemplos do livro que compilam continuem sendo compilados com as versões mais recentes de Rust. A saída pode diferir ligeiramente entre as versões, porque Rust geralmente melhora as mensagens de erro e os avisos. Em outras palavras, qualquer versão mais recente e estável de Rust instalada usando essas etapas deve funcionar conforme o esperado com o conteúdo deste livro.

> ### Notação de Linha de Comando
>
> Neste capítulo e ao longo do livro, mostraremos alguns comandos usados no terminal. As linhas que você deve inserir em um terminal começam com `$`. Você não precisa digitar o caractere `$`; indica o início de cada comando. As linhas que não começam com `$` normalmente mostram a saída do comando anterior. Além disso, exemplos específicos em PowerShell usarão `>` em vez de `$`.

### Instalando `rustup` no Linux ou macOS

Se você estiver usando Linux ou macOS, abra um terminal e digite o seguinte comando:

```text
$ curl https://sh.rustup.rs -sSf | sh
```

O comando baixa um script e inicia a instalação da ferramenta `rustup`, que instala a versão estável mais recente de Rust. Você pode ser solicitado a fornecer sua senha. Se a instalação for bem-sucedida, a seguinte linha aparecerá:

```text
Rust is installed now. Great!
```

Se preferir, faça o download do script e inspecione-o antes de executá-lo.

O script de instalação adiciona Rust automaticamente ao PATH do sistema após seu próximo login. Se você deseja começar a usar Rust imediatamente, em vez de reiniciar o terminal, execute o seguinte comando no shell para adicionar Rust ao PATH do sistema manualmente:

```text
$ source $HOME/.cargo/env
```

Como alternativa, você pode adicionar a seguinte linha ao seu _~/.bash_profile_:

```text
$ export PATH="$HOME/.cargo/bin:$PATH"
```

Além disso, você precisará de um *linker* de algum tipo. Provavelmente já está instalado, mas quando você tenta compilar um programa Rust e obtem erros, indicando que um linker não pôde executar, isso significa que um linker não está instalado no seu sistema e você precisará instalá-lo manualmente. Os compiladores C geralmente vêm com o linker correto. Verifique a documentação da sua plataforma para saber como instalar 
um compilador C. Além disso, alguns pacotes Rust comuns dependem do código C e precisarão de um compilador C. Portanto, pode valer a pena instalar um agora.

### Instalando `rustup` no Windows

No Windows, vá para [https://www.rust-lang.org/pt-BR/tools/install][install-br] e siga as instruções para instalar Rust. Em algum momento da instalação, você receberá uma mensagem explicando que também precisará das ferramentas de *build* do C ++ para o Visual Studio 2013 ou posterior. 
A maneira mais fácil de adquirir as ferramentas de build é instalar [Ferramentas Integradas do Visual Studio 2019][visualstudio-br]. <!--Diretório mudou: As ferramentas estão na seção: Outras Ferramentas e Estruturas. -->

[install]: https://www.rust-lang.org/tools/install
[visualstudio]: https://www.visualstudio.com/downloads/#build-tools-for-visual-studio-2019

[install-br]: https://www.rust-lang.org/pt-BR/tools/install
[visualstudio-br]: https://visualstudio.microsoft.com/pt-br/downloads/

O restante deste livro usa comandos que funcionam no _cmd.exe_ e no PowerShell. Se houver diferenças específicas, explicaremos qual usar.

### Atualização e Desinstalação

Depois de instalar o Rust via `rustup`, é fácil atualizar para a versão mais recente. No seu shell, execute o seguinte script de atualização:

```text
$ rustup update
```

Para desinstalar o Rust e o `rustup`, execute o seguinte script de desinstalação do seu shell:

```text
$ rustup self uninstall
```

### Solução de Problemas

Para verificar se você possui Rust instalado corretamente, abra um shell e digite esta linha:

```text
$ rustc --version
```

Você deverá ver o número da versão, *commit* hash, e *commit* da data da versão estável mais recente lançada no seguinte formato:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Se você visualizar essas informações, instalou Rust com sucesso! Se você não vir essas informações e estiver no Windows, verifique se Rust está na sua variável de sistema `%PATH%`. Se tudo estiver correto e Rust ainda não estiver funcionando, há vários lugares onde você pode obter ajuda. O mais fácil é o canal #beginners em [the official Rust Discord][discord]. Lá, você pode conversar com outros *Rustáceos* (um apelido bobo que chamamos a nós mesmos) que podem ajudá-lo. Outros ótimos recursos 
incluem o canal no Telegram [Rust Brasil][telegram], além do [the Users forum][users] e [Stack Overflow][stackoverflow].

[discord]: https://discord.gg/rust-lang
[users]: https://users.rust-lang.org/
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust
[telegram]: https://t.me/rustlangbr

### Documentação Local

O instalador também inclui uma cópia da documentação localmente, para que você possa lê-la offline. Execute `rustup doc` para abrir a documentação local no seu navegador.

Sempre que um tipo ou função for fornecida pela biblioteca padrão e você não tiver certeza do que esta faz ou como usá-la, use a documentação da interface de programação de aplicativos (API) para descobrir!
