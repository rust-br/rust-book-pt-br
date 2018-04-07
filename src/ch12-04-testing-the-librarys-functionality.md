## Developing the Library’s Functionality with Test Driven Development
## Desenvolvendo a Biblioteca de Funcionalidades com Desenvolvimento Guiado por Testes

Now that we’ve extracted the logic into *src/lib.rs* and left the argument
collecting and error handling in *src/main.rs*, it’s much easier to write tests
for the core functionality of our code. We can call functions directly with
various arguments and check return values without having to call our binary
from the command line. Feel free to write some tests for the functionality in
the `Config::new` and `run` functions on your own.

Agora que extraímos a lógica em *src/lib.rs* e deixamos o argumento de
coleta e tratamento de erros em *src/main.rs*, é muito mais fácil escrever testes
para nosso código da funcionalidade principal. Podemos chamar funções diretamente com
vários argumentos e verificar valores de retorno sem ter que chamar o nosso binário
da linha de comando. Sinta-se livre para escrever alguns testes para 
as funções `Config::new` e `run` por sua conta.

In this section, we’ll add the searching logic to the `minigrep` program by
using the Test Driven Development (TDD) process. This software development
technique follows these steps:

Nesta seção, adicionaremos a lógica de busca ao programa `minigrep`
usando o processo Desenvolvimento Guiado por Testes (Test Driven Development (TDD)). 
Nessa técnica de desenvolvimento de software, segue estas etapas:

1. Write a test that fails, and run it to make sure it fails for the reason you
   expected.
2. Write or modify just enough code to make the new test pass.
3. Refactor the code you just added or changed, and make sure the tests
   continue to pass.
4. Repeat from step 1!

1. Escreva um teste que falha e execute-o, para certificar-se de que ele falha pelo motivo 
    esperado por você.
2. Escreva ou modifique o código apenas o suficiente para fazer passar no teste.
3. Refatore o código que você acabou de adicionar ou alterou e certifique-se de que os testes
    continuam a passar.
4. Repita a partir do passo 1!

This process is just one of many ways to write software, but TDD can help drive
code design as well. Writing the test before you write the code that makes the
test pass helps to maintain high test coverage throughout the process.

Este processo é apenas uma das muitas maneiras de escrever software, mas o TDD pode ajudar a conduzir
design de código também. Escrevendo o teste antes de escrever o código que faz o
teste passar, ajuda a manter uma alta cobertura de teste ao longo do processo.

We’ll test drive the implementation of the functionality that will actually do
the searching for the query string in the file contents and produce a list of
lines that match the query. We’ll add this functionality in a function called
`search`.

Testaremos a implementação da funcionalidade que realmente fará
a busca da string de consulta no conteúdo do arquivo, e produzir uma lista de
linhas que correspondem à consulta. Vamos adicionar essa funcionalidade em uma função chamada
`search`.

### Writing a Failing Test
### Escrevendo um Teste de Falha

Because we don’t need them anymore, let’s remove the `println!` statements from
*src/lib.rs* and *src/main.rs* that we used to check the program’s behavior.
Then, in *src/lib.rs*, we’ll add a `test` module with a test function, as we
did in Chapter 11. The test function specifies the behavior we want the
`search` function to have: it will take a query and the text to search for the
query in, and will return only the lines from the text that contain the query.
Listing 12-15 shows this test, which won't compile yet:

Porque não precisamos mais deles, vamos remover as instruções `println!` de
*src/lib.rs* e *src/main.rs* que costumávamos verificar o comportamento do programa.
Então, em *src/lib.rs*, adicionaremos um módulo `test` com uma função de teste, como nós
fizemos no Capítulo 11. A função de teste especifica o comportamento que queremos
para a função `search` tenha: receberá os parâmetros da consulta e o texto para realizar a
consulta, e retornará apenas as linhas do texto que contém a consulta.
A Listagem 12-15 mostra esse teste, que ainda não compilará:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
# fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#      vec![]
# }
#
#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }
}
```

<span class="caption">Listing 12-15: Creating a failing test for the `search`
function we wish we had</span>

<span class="caption">Listagem 12-15: Criando um teste de falha para a função `search`
que desejamos ter</span>

This test searches for the string “duct.” The text we’re searching is three
lines, only one of which contains “duct.” We assert that the value returned
from the `search` function contains only the line we expect.

Este teste procura a string “duct”. O texto que estamos procurando contém três
linhas, apenas uma das quais contém “duct.” Afirmamos que o valor retornado
a partir da função `search` contém apenas a linha que esperamos.

We aren’t able to run this test and watch it fail because the test doesn’t even
compile: the `search` function doesn’t exist yet! So now we’ll add just enough
code to get the test to compile and run by adding a definition of the `search`
function that always returns an empty vector, as shown in Listing 12-16. Then
the test should compile and fail because an empty vector doesn’t match a vector
containing the line `"safe, fast, productive."`.

Não somos capazes de executar este teste e vê-lo falhar porque o teste nem mesmo
compila: a função `search` ainda não existe! Então, agora vamos adicionar código apenas o suficiente
para obter a compilação do teste, e executar, adicionando uma definição da função `search`
que sempre retorna um vetor vazio, como mostrado na Listagem 12-16. Então
o teste deve compilar e falhar porque um vetor vazio não corresponde a um vetor
contendo a linha `"safe, fast, productive."`.

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    vec![]
}
```

<span class="caption">Listing 12-16: Defining just enough of the `search`
function so our test will compile</span>

<span class="caption">Listagem 12-16: Definindo apenas o suficiente da função `search`
para que nosso teste compile</span>

Notice that we need an explicit lifetime `'a` defined in the signature of
`search` and used with the `contents` argument and the return value. Recall in
Chapter 10 that the lifetime parameters specify which argument lifetime is
connected to the lifetime of the return value. In this case, we indicate that
the returned vector should contain string slices that reference slices of the
argument `contents` (rather than the argument `query`).

Observe que precisamos de uma lifetime explícita `'a` definida na assinatura do
`search` e usada com o argumento `contents` e o valor de retorno. Lembre-se no
Capítulo 10 que os parâmetros de lifetime especificam qual o lifetime do argumento
conectado ao lifetime do valor de retorno. Neste caso, indicamos que
o vetor retornado deve conter pedaços de string que fazem referência a pedaços do
argumento `contents` (em vez do argumento `query`).

In other words, we tell Rust that the data returned by the `search` function
will live as long as the data passed into the `search` function in the
`contents` argument. This is important! The data referenced *by* a slice needs
to be valid for the reference to be valid; if the compiler assumes we’re making
string slices of `query` rather than `contents`, it will do its safety checking
incorrectly.

Em outras palavras, dizemos ao Rust que os dados retornados pela função `search`
irá viver enquanto os dados passarem para a função `search` no
argumento de `contents`. Isso é importante! Os dados referenciados *por* um pedaço precisa
ser válido para que a referência seja válida; se o compilador assume que estamos fazendo
pedaços de string de `query` em vez de `contents`, ele fará sua verificação de segurança
incorretamente.

If we forget the lifetime annotations and try to compile this function, we’ll
get this error:

Se esquecermos as anotações de lifetime e tentarmos compilar esta função, iremos
obter este erro:

```text
error[E0106]: missing lifetime specifier
 --> src/lib.rs:5:51
  |
5 | pub fn search(query: &str, contents: &str) -> Vec<&str> {
  |                                                   ^ expected lifetime
parameter
  |
  = help: this function's return type contains a borrowed value, but the
  signature does not say whether it is borrowed from `query` or `contents`
```

Rust can’t possibly know which of the two arguments we need, so we need to tell
it. Because `contents` is the argument that contains all of our text and we
want to return the parts of that text that match, we know `contents` is the
argument that should be connected to the return value using the lifetime syntax.

Rust não consegue saber qual dos dois argumentos que precisamos, então precisamos informar
isto. Porque `contents` é o argumento que contém todo o nosso texto e nós
queremos retornar as partes desse texto que combinam, sabemos que o `contents` é o
argumento que deve ser conectado ao valor de retorno usando a sintaxe de lifetime.

Other programming languages don’t require you to connect arguments to return
values in the signature, so although this might seem strange, it will get
easier over time. You might want to compare this example with the “Validating
References with Lifetimes” section in Chapter 10.

Outras linguagens de programação não exigem que você conecte argumentos para retornar
valores na assinatura, por isso, embora isso possa parecer estranho, ele ficará
mais fácil ao longo do tempo. Você pode querer comparar este exemplo com a seção “Validando
Referências com Lifetimes” no Capítulo 10.

Now let’s run the test:

Agora vamos executar o teste:

```text
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
--warnings--
    Finished dev [unoptimized + debuginfo] target(s) in 0.43 secs
     Running target/debug/deps/minigrep-abcabcabc

running 1 test
test test::one_result ... FAILED

failures:

---- test::one_result stdout ----
        thread 'test::one_result' panicked at 'assertion failed: `(left ==
right)`
left: `["safe, fast, productive."]`,
right: `[]`)', src/lib.rs:48:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.


failures:
    test::one_result

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

Great, the test fails, exactly as we expected. Let’s get the test to pass!

Ótimo, o teste falha, exatamente como esperávamos. Vamos fazer o teste passar!

### Writing Code to Pass the Test
### Escrevendo Código para Passar no Teste

Currently, our test is failing because we always return an empty vector. To fix
that and implement `search`, our program needs to follow these steps:

Atualmente, nosso teste está falhando porque sempre devolvemos um vetor vazio. Para consertar 
isso é preciso implementar `search`, nosso programa precisa seguir essas etapas:

* Iterate through each line of the contents.
* Check whether the line contains our query string.
* If it does, add it to the list of values we’re returning.
* If it doesn’t, do nothing.
* Return the list of results that match.

* Iterar através de cada linha do conteúdo.
* Verificar se a linha contém nossa string de consulta.
* Se a tiver, adicione-a à lista de valores que estamos retornando.
* Se não, não faça nada.
* Retorna a lista de resultados que correspondem.

Let’s work through each step, starting with iterating through lines.

Vamos trabalhar em cada passo, começando por iterar através de linhas.

#### Iterating Through Lines with the `lines` Method
#### Iterar Através de Linhas com o Método `lines`

Rust has a helpful method to handle line-by-line iteration of strings,
conveniently named `lines`, that works as shown in Listing 12-17. Note this
won’t compile yet:

Rust tem um método útil para lidar com a iteração linha-a-linha de strings,
convenientemente chamado `lines`, que funciona como mostrado na Listagem 12-17. Observe que isso
ainda não compilará:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        // do something with line
    }
}
```

<span class="caption">Listing 12-17: Iterating through each line in `contents`
</span>

<span class="caption">Listagem 12-17: Iterando para cada linha em `contents`
</span>

The `lines` method returns an iterator. We’ll talk about iterators in depth in
Chapter 13, but recall that you saw this way of using an iterator in Listing
3-4, where we used a `for` loop with an iterator to run some code on each item
in a collection.

O método `lines` retorna um iterador. Vamos falar sobre iteradores em profundidade no
Capítulo 13, mas lembre-se de que você viu essa maneira de usar um iterador na Listagem
3-4, onde usamos um loop `for` com um iterador para executar algum código em cada item
de uma coleção.

#### Searching Each Line for the Query
#### Pesquisando Cada Linha para a Consulta

Next, we’ll check whether the current line contains our query string.
Fortunately, strings have a helpful method named `contains` that does this for
us! Add a call to the `contains` method in the `search` function, as shown in
Listing 12-18. Note this still won’t compile yet:

Em seguida, verificamos se a linha atual contém nossa string de consulta.
Felizmente, as strings possuem um método útil chamado `contains` que faz isso para
nós! Adicione uma chamada ao método `contains` na função `search`, conforme mostrado na
Listagem 12-18. Observe que isso ainda não compilará ainda:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        if line.contains(query) {
            // do something with line
        }
    }
}
```

<span class="caption">Listing 12-18: Adding functionality to see whether the
line contains the string in `query`</span>

<span class="caption">Listagem 12-18: Adicionando funcionalidade para ver se
a linha contém a string na `query`</span>


#### Storing Matching Lines
#### Armazenamento de Linhas Correspondentes

We also need a way to store the lines that contain our query string. For that,
we can make a mutable vector before the `for` loop and call the `push` method
to store a `line` in the vector. After the `for` loop, we return the vector, as
shown in Listing 12-19:

Nós também precisamos de uma maneira de armazenar as linhas que contêm nossa string de consulta. Por isso,
podemos fazer um vetor mutável antes do loop `for` e chamar o método `push`
para armazenar uma `line` no vetor. Após o loop `for`, devolvemos o vetor, como
mostrado na Listagem 12-19:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

<span class="caption">Listing 12-19: Storing the lines that match so we can
return them</span>

<span class="caption">Listagem 12-19: Armazenando as linhas que combinam para que possamos
devolvê-las</span>


Now the `search` function should return only the lines that contain `query`,
and our test should pass. Let’s run the test:

Agora, a função `search` deve retornar apenas as linhas que contêm` query`,
e nosso teste deve passar. Vamos executar o teste:

```text
$ cargo test
--snip--
running 1 test
test test::one_result ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Our test passed, so we know it works!

Nosso teste passou, então sabemos que funciona!

At this point, we could consider opportunities for refactoring the
implementation of the search function while keeping the tests passing to
maintain the same functionality. The code in the search function isn’t too bad,
but it doesn’t take advantage of some useful features of iterators. We’ll
return to this example in Chapter 13 where we’ll explore iterators in detail
and look at how to improve it.

Neste ponto, poderíamos considerar oportunidades de refatorar
a implementação da função de pesquisa, mantendo os testes passando para
a mesma funcionalidade. O código na função de pesquisa não é muito ruim,
mas não tira proveito de algumas características úteis dos iteradores. Iremos
voltar para este exemplo no Capítulo 13, onde exploraremos iteradores em detalhes
e veremos como melhorá-lo.

#### Using the `search` Function in the `run` Function
#### Usando a Função `search` na Função` run`

Now that the `search` function is working and tested, we need to call `search`
from our `run` function. We need to pass the `config.query` value and the
`contents` that `run` reads from the file to the `search` function. Then `run`
will print each line returned from `search`:

Agora que a função `search` está funcionando e testada, precisamos chamar `search`
da nossa função `run`. Precisamos passar o valor `config.query` e o 
`contents` que `run` lê do arquivo para a função `search`. Então, `run`
irá imprimir cada linha retornada de `search`:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Arquivo: src/lib.rs</span>

```rust,ignore
pub fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    for line in search(&config.query, &contents) {
        println!("{}", line);
    }

    Ok(())
}
```

We’re still using a `for` loop to return each line from `search` and print it.

Ainda estamos usando um loop `for` para retornar cada linha de `search` e imprimi-lo.

Now the entire program should work! Let’s try it out, first with a word that
should return exactly one line from the Emily Dickinson poem, “frog”:

Agora, todo o programa deve funcionar! Vamos tentar, primeiro, com uma palavra que
deve retornar exatamente uma linha do poema de Emily Dickinson, “frog”:

```text
$ cargo run frog poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.38 secs
     Running `target/debug/minigrep frog poem.txt`
How public, like a frog
```

Cool! Now let’s try a word that will match multiple lines, like “body”:

Legal! Agora vamos tentar uma palavra que combine várias linhas, como “body”:

```text
$ cargo run body poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep body poem.txt`
I’m nobody! Who are you?
Are you nobody, too?
How dreary to be somebody!
```

And finally, let’s make sure that we don’t get any lines when we search for a
word that isn’t anywhere in the poem, such as “monomorphization”:

E, finalmente, vamos nos certificar de que não recebemos nenhuma linha quando buscamos uma
palavra que não está em qualquer lugar no poema, como “monomorphization”:

```text
$ cargo run monomorphization poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep monomorphization poem.txt`
```

Excellent! We’ve built our own mini version of a classic tool and learned a lot
about how to structure applications. We’ve also learned a bit about file input
and output, lifetimes, testing, and command line parsing.

Excelente! Nós construímos nossa própria mini versão de uma ferramenta clássica e aprendemos muito
sobre como estruturar aplicativos. Também aprendemos um pouco sobre a entrada de arquivos
e saída, lifetimes, teste e análise de linha de comando.

To round out this project, we’ll briefly demonstrate how to work with
environment variables and how to print to standard error, both of which are
useful when you’re writing command line programs.

Para completar este projeto, brevemente demonstraremos como trabalhar com
variáveis de ambiente e como imprimir em erro padrão, ambos
úteis quando você está escrevendo programas de linha de comando.
