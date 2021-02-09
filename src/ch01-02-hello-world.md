## Olá, mundo!

Agora que você instalou Rust, vamos escrever seu primeiro programa Rust. Quando se aprende uma nova linguagem, é tradicional escrever um pequeno programa que imprime o texto `Hello, world!` na tela, para que façamos o mesmo aqui!

> Nota: Este livro pressupõe familiaridade básica com a linha de comando. Rust não requer exigências específicas sobre a sua edição, ferramentas ou a localização do seu código; portanto, se você preferir usar um ambiente de desenvolvimento integrado (IDE) em vez da linha de comando, fique à vontade para usar o seu IDE favorito. Muitos IDEs agora têm algum grau de apoio ao Rust; consulte a documentação do IDE para obter detalhes. 
> Recentemente, a equipe do Rust tem se concentrado em permitir um ótimo suporte a IDE, e houve progresso rápido nessa frente!

### Criando um Diretório de Projeto

Você começará criando um diretório para armazenar seu código Rust. Não importa para Rust onde seu código mora, mas para os exercícios e projetos deste livro, sugerimos criar um diretório *projects* no diretório inicial e manter todos os seus projetos lá.

Abra um terminal e digite os seguintes comandos para criar um diretório *projects* e um diretório para o projeto Hello, world! dentro do diretório *projects*.

Para Linux, macOS e PowerShell no Windows, digite o seguinte:

```text
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Para o Windows CMD, digite o seguinte:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

### Escrevendo e Executando um Programa Rust

Em seguida, crie um novo arquivo source e chame-o de *main.rs*. Arquivos Rust sempre terminam com a extensão ***.rs*** . Se você estiver usando mais de uma palavra no seu nome de arquivo, use um sublinhado para separá-las. Por exemplo, use *hello_world.rs* em vez de *helloworld.rs*.

Agora abra o arquivo *main.rs* que você acabou de criar e insira o código na Listagem 1-1.

<span class="filename">Nome do arquivo: main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

<span class="caption">Listagem 1-1: Um programa que imprime `Hello, world!`</span>

Salve o arquivo e volte para a janela do seu terminal. No Linux ou macOS, digite os seguintes comandos para compilar e executar o arquivo:

```text
$ rustc main.rs
$ ./main
Hello, world!
```

No Windows, digite o comando `.\main.exe` em vez de `./main`:

```powershell
> rustc main.rs
> .\main.exe
Hello, world!
```

Independentemente do seu sistema operacional, a string `Hello, world!` deve ser impressa no terminal. Se você não vir essa saída, consulte a parte [“Solução de Problemas”][troubleshooting]<!-- ignore --> da seção instalação para obter maneiras de obter ajuda.

Se `Hello, world!` foi impresso, parabéns! Você escreveu oficialmente um programa Rust. Isso faz de você um programador Rust — bem-vindo!

### Anatomia de um Programa Rust

Vamos analisar em detalhes o que aconteceu no seu programa Hello, world! 
Aqui está a primeira peça do quebra-cabeça:

```rust
fn main() {

}
```

Essas linhas definem uma função em Rust. A função `main` é especial: é sempre o primeiro código executado em todos os programas Rust executáveis. A primeira linha declara uma função chamada `main` que não possui parâmetros e não retorna nada. Se houvesse parâmetros, eles entrariam entre parênteses, `()`.

Observe também que o corpo da função está entre colchetes, `{}`. Rust exige isso em todos os corpos funcionais. É um bom estilo colocar o colchete de abertura na mesma linha da declaração de função, adicionando um espaço no meio.

No momento da redação deste artigo, uma ferramenta formatadora automática chamada `rustfmt` está em desenvolvimento. Se você deseja manter um estilo padrão nos projetos Rust, o `rustfmt` formatará seu código em um estilo específico. A equipe do Rust planeja eventualmente incluir essa ferramenta na distribuição padrão do Rust, como `rustc`. 
Portanto, dependendo de quando você ler este livro, ele poderá já estar instalado no seu computador! Consulte a documentação online para mais detalhes.

Dentro da função `main` está o seguinte código:

```rust
    println!("Hello, world!");
```

Esta linha faz todo o trabalho neste pequeno programa: imprime texto na tela. Há quatro detalhes importantes a serem observados aqui. Primeiro, o estilo Rust é recuar com quatro espaços, não uma tabulação.

Segundo, `println!` chama uma macro Rust. Se fosse chamada uma função, ela seria inserida como `println` (sem o`!`). Discutiremos Rust macros com mais detalhes no Capítulo 19. Por enquanto, você só precisa saber que usar um `!` significa que você está chamando uma macro em vez de uma função normal.

Terceiro, você vê a string `"Hello, world!"`. Passamos essa string como argumento para `println!`, e a string é impressa na tela.

Quarto, terminamos a linha com um ponto-e-vírgula (`;`), que indica que essa expressão acabou e a próxima está pronta para começar. A maioria das linhas do código Rust termina com um ponto e vírgula.

### Compilar e Executar são Etapas Separadas

Você acabou de executar um programa recém-criado, portanto, vamos examinar cada etapa do processo.

Antes de executar um programa Rust, você deve compilá-lo usando o compilador Rust digitando o comando `rustc` e passando o nome do seu arquivo source, assim:

```text
$ rustc main.rs
```

Se você tem experiência em C ou C ++, notará que isso é semelhante a `gcc` ou `clang`. Após compilar com sucesso, Rust gera um executável binário.

No Linux, macOS e PowerShell no Windows, você pode ver o executável digitando o comando `ls` no seu shell. No Linux e macOS, você verá dois arquivos. Com o PowerShell no Windows, você verá os mesmos três arquivos que usaria no CMD.

```text
$ ls
main  main.rs
```

Com o CMD no Windows, você digitaria o seguinte:

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

Isso mostra o arquivo de código-fonte com a extensão *.rs*, o arquivo executável (*main.exe* no Windows, mas *main* em todas as outras plataformas) e, ao usar o Windows, um arquivo contendo informações de depuração com o extensão *.pdb*. A partir daqui, você executa o arquivo *main* ou *main.exe*, assim:

```text
$ ./main # or .\main.exe on Windows
```

Se *main.rs* era seu programa Hello, world!, esta linha imprimirá `Hello, world!` no seu terminal.

Se você está mais familiarizado com uma linguagem dinâmica, como Ruby, Python ou JavaScript, pode não estar acostumado a compilar e executar um programa como etapas separadas. Rust é uma linguagem *compilada antecipadamente*, o que significa que você pode compilar um programa e fornecer o executável para outra pessoa, e eles podem executá-lo mesmo sem Rust instalado. Se você fornecer a alguém um arquivo *.rb*, *.py* 
ou *.js*, eles deverão ter uma implementação Ruby, Python ou JavaScript instalada (respectivamente). Mas essas linguagens, você só precisa de um comando para compilar e executar seu programa. Tudo é uma troca no design da linguagem.

Apenas compilar com `rustc` é bom para programas simples, mas à medida que o seu projeto cresce, você deseja gerenciar todas as opções e facilitar o compartilhamento do seu código. 
Em seguida, apresentaremos a ferramenta Cargo, que ajudará você a criar programas Rust no mundo real.

[troubleshooting]: ch01-01-installation.html#troubleshooting
[Solução de Problemas]: ch01-01-installation.html#solucao-de-problemas
