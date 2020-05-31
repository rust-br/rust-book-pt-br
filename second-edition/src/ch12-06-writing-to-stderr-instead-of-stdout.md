## Escrevendo Mensagens de Erro para Erro Padrão em Vez de Saída Padrão

No momento, estamos escrevendo toda a nossa saída para o terminal usando a função 
`println!`. A maioria dos terminais fornece dois tipos de saída: *saída padrão* 
(`stdout`) para informações gerais e *erro padrão* (`stderr`) para mensagens 
de erro. Essa distinção permite que os usuários escolham direcionar a saída 
bem-sucedida de um programa para um arquivo, mas ainda imprimir mensagens de erro na tela.

A função `println!` só é capaz de imprimir na saída padrão, então temos 
que usar outra coisa para imprimir em erro padrão.

### Verificando Onde os Erros são Escritos

Primeiro, vamos observar como o conteúdo impresso por `minigrep` está sendo 
gravado na saída padrão, incluindo as mensagens de erro que desejamos gravar 
no erro padrão. Faremos isso redirecionando o fluxo de saída padrão para um arquivo e, 
ao mesmo tempo, causando um erro intencionalmente. Não redirecionamos o fluxo de 
erros padrão, portanto, qualquer conteúdo enviado ao erro padrão continuará sendo exibido na tela.

Espera-se que os programas de linha de comando enviem mensagens de erro para o fluxo erro padrão
, para que ainda possamos ver mensagens de erro na tela, mesmo se redirecionarmos o fluxo 
de saída padrão para um arquivo. Nosso programa não está bem comportado: estamos prestes a ver 
que ele salva a saída da mensagem de erro em um arquivo!

A maneira de demonstrar este comportamento é rodando o programa com `>` e o 
nome do arquivo, *output.txt*, para o qual queremos redirecionar o fluxo de saída padrão.
Não passamos nenhum argumento, o que deve causar um erro:

```text
$ cargo run > output.txt
```

A sintaxe `>` diz ao shell para gravar o conteúdo da saída padrão para
*output.txt* em vez da tela. Nós não vimos a mensagem de erro que estávamos
esperando impresso na tela, o que significa que deve ter acabado no
arquivo. Isto é o que o *output.txt* contém:

```text
Problem parsing arguments: not enough arguments
```

Sim, nossa mensagem de erro está sendo impressa na saída padrão. É muito mais 
útil que mensagens de erro como essa sejam impressas no erro padrão e que somente 
os dados de uma execução bem-sucedida acabem no arquivo quando redirecionamos a 
saída padrão dessa maneira. Nós vamos mudar isso.

### Imprimindo Erros em Padrão de Erros

Usaremos o código da Listagem 12-24 para alterar a forma como as mensagens de erro são impressas.
Por causa da refatoração que fizemos anteriormente neste capítulo, todo o código que 
imprime mensagens de erro está em uma função, `main`. A biblioteca padrão fornece a 
macro `eprintln!` que imprime no fluxo de erro padrão, então vamos alterar os dois 
locais que estávamos chamando `println!` para imprimir erros para usar `eprintln!`:

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

<span class="caption">Listagem 12-24: Escrevendo mensagens de erro para o erro padrão
em vez da saída padrão usando o `eprintln!`</span>

Depois de alterar `println!` para `eprintln!`, vamos executar o programa novamente 
da mesma forma, sem argumentos e redirecionando a saída padrão com `>`:

```text
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Agora vemos o erro na tela e o *output.txt* não contém nada, que é o 
comportamento esperado dos programas de linha de comando.

Vamos executar o programa novamente com argumentos que não causam erro, mas ainda 
redirecionamos a saída padrão para um arquivo, da seguinte forma:

```text
$ cargo run to poem.txt > output.txt
```

Não veremos nenhuma saída para o terminal e *output.txt* conterá nossos 
resultados:

<span class="filename">Arquivo: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Isso demonstra que agora estamos usando a saída padrão para saída bem-sucedida e 
erro padrão para saída de erro, apropriadamente.

## Resumo

Neste capítulo, recapitulamos alguns dos principais conceitos que você aprendeu até agora 
e abordamos como realizar operações de E/S comuns em um contexto Rust. Usando argumentos 
de linha de comando, arquivos, variáveis de ambiente e a macro `eprintln!` para 
erros de impressão, você está preparado para escrever aplicativos de linha de comando. Usando 
os conceitos dos capítulos anteriores, seu código será bem organizado, armazenará dados de forma 
eficaz nas estruturas de dados apropriadas, tratará erros com precisão e será bem testado.

Em seguida, exploraremos alguns recursos do Rust que foram influenciados por linguagens 
funcionais: closures e iteradores.
