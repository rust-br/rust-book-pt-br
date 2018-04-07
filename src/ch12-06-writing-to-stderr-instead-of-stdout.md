## Writing Error Messages to Standard Error Instead of Standard Output
## Escrevendo Mensagens de Erro para Erro Padrão em Vez de Saída Padrão

At the moment we’re writing all of our output to the terminal using the
`println!` function. Most terminals provide two kinds of output: *standard
output* (`stdout`) for general information and *standard error* (`stderr`)
for error messages. This distinction enables users to choose to direct the
successful output of a program to a file but still print error messages to the
screen.

No momento, estamos escrevendo toda a nossa saída para o terminal usando a função 
`println!`. A maioria dos terminais fornece dois tipos de saída: *saída padrão* 
(`stdout`) para informações gerais e *erro padrão* (`stderr`) para mensagens 
de erro. Essa distinção permite que os usuários escolham direcionar a saída 
bem-sucedida de um programa para um arquivo, mas ainda imprimir mensagens de erro na tela.

The `println!` function is only capable of printing to standard output, so we
have to use something else to print to standard error.

A função `println!` só é capaz de imprimir na saída padrão, então temos 
que usar outra coisa para imprimir em erro padrão.

### Checking Where Errors Are Written to
### Verificando Onde os Erros são Escritos

First, let’s observe how the content printed by `minigrep` is currently being
written to standard output, including any error messages we want to write to
standard error instead. We’ll do that by redirecting the standard output stream
to a file while also intentionally causing an error. We won’t redirect the
standard error stream, so any content sent to standard error will continue to
display on the screen.

Primeiro, vamos observar como o conteúdo impresso por `minigrep` está sendo 
gravado na saída padrão, incluindo as mensagens de erro que desejamos gravar 
no erro padrão. Faremos isso redirecionando o fluxo de saída padrão para um arquivo e, 
ao mesmo tempo, causando um erro intencionalmente. Não redirecionamos o fluxo de 
erros padrão, portanto, qualquer conteúdo enviado ao erro padrão continuará sendo exibido na tela.

Command line programs are expected to send error messages to the standard error
stream so we can still see error messages on the screen even if we redirect the
standard output stream to a file. Our program is not currently well-behaved:
we’re about to see that it saves the error message output to a file instead!

Espera-se que os programas de linha de comando enviem mensagens de erro para o fluxo erro padrão
, para que ainda possamos ver mensagens de erro na tela, mesmo se redirecionarmos o fluxo 
de saída padrão para um arquivo. Nosso programa não está bem comportado: estamos prestes a ver 
que ele salva a saída da mensagem de erro em um arquivo!

The way to demonstrate this behavior is by running the program with `>` and the
filename, *output.txt*, that we want to redirect the standard output stream to.
We won’t pass any arguments, which should cause an error:

A maneira de demonstrar este comportamento é rodando o programa com `>` e o 
nome do arquivo, *output.txt*, para o qual queremos redirecionar o fluxo de saída padrão.
Não passamos nenhum argumento, o que deve causar um erro:

```text
$ cargo run > output.txt
```

The `>` syntax tells the shell to write the contents of standard output to
*output.txt* instead of the screen. We didn’t see the error message we were
expecting printed on the screen, so that means it must have ended up in the
file. This is what *output.txt* contains:

A sintaxe `>` diz ao shell para gravar o conteúdo da saída padrão para
*output.txt* em vez da tela. Nós não vimos a mensagem de erro que estávamos
esperando impresso na tela, o que significa que deve ter acabado no
arquivo. Isto é o que o *output.txt* contém:

```text
Problem parsing arguments: not enough arguments
```

Yup, our error message is being printed to standard output. It’s much more
useful for error messages like this to be printed to standard error and have
only data from a successful run end up in the file when we redirect standard
output this way. We’ll change that.

Sim, nossa mensagem de erro está sendo impressa na saída padrão. É muito mais 
útil que mensagens de erro como essa sejam impressas no erro padrão e que somente 
os dados de uma execução bem-sucedida acabem no arquivo quando redirecionamos a 
saída padrão dessa maneira. Nós vamos mudar isso.

### Printing Errors to Standard Error
### Imprimindo Erros em Padrão de Erros

We’ll use the code in Listing 12-24 to change how error messages are printed.
Because of the refactoring we did earlier in this chapter, all the code that
prints error messages is in one function, `main`. The standard library provides
the `eprintln!` macro that prints to the standard error stream, so let’s change
the two places we were calling `println!` to print errors to use `eprintln!`
instead:

Usaremos o código da Listagem 12-24 para alterar a forma como as mensagens de erro são impressas.
Por causa da refatoração que fizemos anteriormente neste capítulo, todo o código que 
imprime mensagens de erro está em uma função, `main`. A biblioteca padrão fornece a 
macro `eprintln!` que imprime no fluxo de erro padrão, então vamos alterar os dois 
locais que estávamos chamando `println!` para imprimir erros para usar `eprintln!`:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Arquivo: src/main.rs</span>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {}", e);

        process::exit(1);
    }
}
```

<span class="caption">Listing 12-24: Writing error messages to standard error
instead of standard output using `eprintln!`</span>

<span class="caption">Listagem 12-24: Escrevendo mensagens de erro para o erro padrão
em vez da saída padrão usando o `eprintln!`</span>

After changing `println!` to `eprintln!`, let’s run the program again in the
same way, without any arguments and redirecting standard output with `>`:

Depois de alterar `println!` para `eprintln!`, vamos executar o programa novamente 
da mesma forma, sem argumentos e redirecionando a saída padrão com `>`:

```text
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Now we see the error onscreen and *output.txt* contains nothing, which is the
behavior we expect of command line programs.

Agora vemos o erro na tela e o *output.txt* não contém nada, que é o 
comportamento esperado dos programas de linha de comando.

Let’s run the program again with arguments that don’t cause an error but still
redirect standard output to a file, like so:

Vamos executar o programa novamente com argumentos que não causam erro, mas ainda 
redirecionamos a saída padrão para um arquivo, da seguinte forma:

```text
$ cargo run to poem.txt > output.txt
```

We won’t see any output to the terminal, and *output.txt* will contain our
results:

Não veremos nenhuma saída para o terminal e *output.txt* conterá nossos 
resultados:

<span class="filename">Filename: output.txt</span>

<span class="filename">Arquivo: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

This demonstrates that we’re now using standard output for successful output
and standard error for error output as appropriate.

Isso demonstra que agora estamos usando a saída padrão para saída bem-sucedida e 
erro padrão para saída de erro, apropriadamente.

## Summary
## Resumo

In this chapter, we’ve recapped some of the major concepts you’ve learned so
far and covered how to do common I/O operations in a Rust context. By using
command line arguments, files, environment variables, and the `eprintln!` macro
for printing errors, you’re now prepared to write command line applications. By
using the concepts in previous chapters, your code will be well organized,
store data effectively in the appropriate data structures, handle errors
nicely, and be well tested.

Neste capítulo, recapitulamos alguns dos principais conceitos que você aprendeu até agora 
e abordamos como realizar operações de E/S comuns em um contexto Rust. Usando argumentos 
de linha de comando, arquivos, variáveis de ambiente e a macro `eprintln!` para 
erros de impressão, você está preparado para escrever aplicativos de linha de comando. Usando 
os conceitos dos capítulos anteriores, seu código será bem organizado, armazenará dados de forma 
eficaz nas estruturas de dados apropriadas, tratará erros com precisão e será bem testado.

Next, we’ll explore some Rust features that were influenced by functional
languages: closures and iterators.

Em seguida, exploraremos alguns recursos do Rust que foram influenciados por linguagens 
funcionais: closures e iteradores.
