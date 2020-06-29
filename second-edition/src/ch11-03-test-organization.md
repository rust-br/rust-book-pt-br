## Test Organization
## Organização de Teste

As mentioned at the start of the chapter, testing is a complex discipline, and
different people use different terminology and organization. The Rust community
thinks about tests in terms of two main categories: *unit tests* and
*integration tests*. Unit tests are small and more focused, testing one module
in isolation at a time, and can test private interfaces. Integration tests are
entirely external to your library and use your code in the same way any other
external code would, using only the public interface and potentially exercising
multiple modules per test.

Conforme mencionado no início do capítulo, o teste é uma disciplina complexa, e 
pessoas diferentes usam terminologia e organização diferentes. A comunidade Rust 
pensa em testes em termos de duas categorias principais: *testes de unidade* 
(unit tests) e *testes de integração* (integration tests). Os testes de unidade 
são pequenos e mais focados, testando um módulo isoladamente por vez e podem testar 
interfaces privadas. Os testes de integração são totalmente externos à sua biblioteca 
e usam seu código da mesma forma que qualquer outro código externo, usando apenas a 
interface pública e potencialmente exercitando vários módulos por teste.

Writing both kinds of tests is important to ensure that the pieces of your
library are doing what you expect them to separately and together.

Escrever os dois tipos de testes é importante para garantir que as partes da 
sua biblioteca estejam fazendo o que você espera que elas façam separadamente 
e juntas.

### Unit Tests
### Testes Unitários

The purpose of unit tests is to test each unit of code in isolation from the
rest of the code to quickly pinpoint where code is and isn’t working as
expected. You’ll put unit tests in the *src* directory in each file with the
code that they’re testing. The convention is to create a module named `tests`
in each file to contain the test functions and to annotate the module with
`cfg(test)`.

O objetivo dos testes de unidade é testar cada unidade de código isoladamente 
do restante do código para identificar rapidamente onde o código está e não está 
funcionando conforme o esperado. Você colocará testes de unidade no diretório 
*src* em cada arquivo com o código que eles estão testando. A convenção é criar 
um módulo chamado `tests` em cada arquivo para conter as funções de teste e 
anotar o módulo com `cfg(test)`.

#### The Tests Module and `#[cfg(test)]`
#### O Módulo de Testes e `#[cfg(test)]`

The `#[cfg(test)]` annotation on the tests module tells Rust to compile and run
the test code only when you run `cargo test`, not when you run `cargo build`.
This saves compile time when you only want to build the library and saves space
in the resulting compiled artifact because the tests are not included. You’ll
see that because integration tests go in a different directory, they don’t need
the `#[cfg(test)]` annotation. However, because unit tests go in the same files
as the code, you’ll use `#[cfg(test)]` to specify that they shouldn’t be
included in the compiled result.

A anotação `#[cfg(test)]` no módulo de testes diz ao Rust para compilar e executar 
o código de teste somente quando você executa o `cargo test`, e não quando o `cargo build`. 
Isso economiza tempo de compilação quando você deseja apenas construir a biblioteca 
e economiza espaço no artefato compilado resultante porque os testes não estão incluídos. 
Você verá que, como os testes de integração estão em um diretório diferente, eles não 
precisam da anotação `#[cfg(test)]`. No entanto, como os testes de unidade estão nos 
mesmos arquivos que o código, você usará `#[cfg(test)]` para especificar que eles não 
devem ser incluídos no resultado compilado.

Recall that when we generated the new `adder` project in the first section of
this chapter, Cargo generated this code for us:

Lembre-se de que, quando geramos o novo projeto `adder` (somador) na primeira seção deste 
capítulo, Cargo gerou esse código para nós:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

This code is the automatically generated test module. The attribute `cfg`
stands for *configuration* and tells Rust that the following item should only
be included given a certain configuration option. In this case, the
configuration option is `test`, which is provided by Rust for compiling and
running tests. By using the `cfg` attribute, Cargo compiles our test code only
if we actively run the tests with `cargo test`. This includes any helper
functions that might be within this module, in addition to the functions
annotated with `#[test]`.

Este código é o módulo de teste gerado automaticamente. O atributo `cfg` significa 
*configuração* (configuration) e informa ao Rust que o item a seguir deve ser 
incluído apenas com uma determinada opção de configuração. Nesse caso, a opção 
de configuração é `test`, que é fornecida pelo Rust para compilar e executar testes. 
Ao usar o atributo `cfg`, Cargo compila nosso código de teste apenas se executarmos 
ativamente os testes com `cargo test`. Isso inclui quaisquer funções auxiliares que 
possam estar dentro deste módulo, além das funções anotadas com `#[test]`.

#### Testing Private Functions
#### Testando Funções Privadas

There’s debate within the testing community about whether or not private
functions should be tested directly, and other languages make it difficult or
impossible to test private functions. Regardless of which testing ideology you
adhere to, Rust’s privacy rules do allow you to test private functions.
Consider the code in Listing 11-12 with the private function `internal_adder`:

Há um debate na comunidade de testes sobre se as funções privadas devem ou não 
ser testadas diretamente, e outras linguagens tornam difícil ou impossível testar 
as funções privadas. Independentemente de qual ideologia de teste você aderir, 
as regras de privacidade de Rust permitem testar funções privadas. Considere o 
código na Listagem 11-12 com a função privada `internal_adder`:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

<span class="caption">Listing 11-12: Testing a private function</span>

<span class="caption">Listagem 11-12: Testando uma função privada</span>

Note that the `internal_adder` function is not marked as `pub`, but because
tests are just Rust code and the `tests` module is just another module, you can
import and call `internal_adder` in a test just fine. If you don’t think
private functions should be tested, there’s nothing in Rust that will compel
you to do so.

Note que a função `internal_adder` não está marcada como `pub`, mas como os testes 
são apenas código Rust e o módulo `tests` é apenas outro módulo, você pode importar 
e chamar `internal_adder` em um teste. Se você não acha que as funções privadas 
devem ser testadas, não há nada no Rust que o obrigue a fazê-lo.

### Integration Tests
### Testes de Integração

In Rust, integration tests are entirely external to your library. They use your
library in the same way any other code would, which means they can only call
functions that are part of your library’s public API. Their purpose is to test
whether many parts of your library work together correctly. Units of code that
work correctly on their own could have problems when integrated, so test
coverage of the integrated code is important as well. To create integration
tests, you first need a *tests* directory.

Em Rust, os testes de integração são totalmente externos à sua biblioteca. Eles 
usam sua biblioteca da mesma maneira que qualquer outro código usaria, o que 
significa que eles só podem chamar funções que fazem parte da API pública da 
sua biblioteca. O objetivo deles é testar se várias partes da sua biblioteca 
funcionam juntas corretamente. As unidades de código que funcionam corretamente 
por conta própria podem ter problemas quando integradas; portanto, a cobertura 
de teste do código integrado também é importante. Para criar testes de integração, 
primeiro você precisa de um diretório *tests*.

#### The *tests* Directory
#### O Diretório *tests*

We create a *tests* directory at the top level of our project directory, next
to *src*. Cargo knows to look for integration test files in this directory. We
can then make as many test files as we want to in this directory, and Cargo
will compile each of the files as an individual crate.

Criamos um diretório *tests* no nível superior do diretório do nosso projeto, ao 
lado de *src*. O Cargo sabe procurar arquivos de teste de integração neste diretório. 
Podemos então criar quantos arquivos de teste quisermos neste diretório, e Cargo 
compilará cada um deles como uma crate individual.

Let’s create an integration test. With the code in Listing 11-12 still in the
*src/lib.rs* file, make a *tests* directory, create a new file named
*tests/integration_test.rs*, and enter the code in Listing 11-13:

Vamos criar um teste de integração. Com o código na Listagem 11-12 ainda no arquivo 
*src/lib.rs*, faça um diretório *tests*, crie um novo arquivo chamado 
*tests/integration_test.rs* e digite o código na Listagem 11-13:

<span class="filename">Filename: tests/integration_test.rs</span>

<span class="filename">Nome do arquivo: tests/integration_test.rs</span>

```rust,ignore
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

<span class="caption">Listing 11-13: An integration test of a function in the
`adder` crate</span>

<span class="caption">Listagem 11-13: Um teste de integração de uma função 
na crate `adder`</span>

We’ve added `extern crate adder` at the top of the code, which we didn’t need
in the unit tests. The reason is that each test in the `tests` directory is a
separate crate, so we need to import our library into each of them.

Adicionamos `extern crate adder` na parte superior do código, o que não era 
necessário nos testes de unidade. O motivo é que cada teste no diretório `tests` 
é uma crate separada, portanto, precisamos importar nossa biblioteca para 
cada um deles.

We don’t need to annotate any code in *tests/integration_test.rs* with
`#[cfg(test)]`. Cargo treats the `tests` directory specially and compiles files
in this directory only when we run `cargo test`. Run `cargo test` now:

Não precisamos anotar nenhum código em *tests/integration_test.rs* com 
`#[cfg(test)]`. Cargo trata o diretório `tests` especialmente e compila 
arquivos nesse diretório somente quando executamos o `cargo test`. Execute o 
`cargo test` agora:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running target/debug/deps/adder-abcabcabc

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-ce99bcc2479f4607

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

The three sections of output include the unit tests, the integration test, and
the doc tests. The first section for the unit tests is the same as we’ve been
seeing: one line for each unit test (one named `internal` that we added in
Listing 11-12) and then a summary line for the unit tests.

As três seções da saída incluem os testes de unidade, o teste de integração e os 
testes de documentos (doc tests). A primeira seção para os testes de unidade é a 
mesma que vimos: uma linha para cada teste de unidade (uma chamada `internal` que 
adicionamos na Listagem 11-12) e, em seguida, uma linha de resumo para os testes 
de unidade.

The integration tests section starts with the line `Running
target/debug/deps/integration-test-ce99bcc2479f4607` (the hash at the end of
your output will be different). Next, there is a line for each test function in
that integration test and a summary line for the results of the integration
test just before the `Doc-tests adder` section starts.

A seção de testes de integração começa com a linha 
`Running target/debug/deps/integration-test-ce99bcc2479f4607` (o hash no final da sua 
saída será diferente). Em seguida, há uma linha para cada função de teste nesse teste 
de integração e uma linha de resumo para os resultados do teste de integração 
imediatamente antes do início da seção `Doc-tests adder`.

Similarly to how adding more unit test functions adds more result lines to the
unit tests section, adding more test functions to the integration test file
adds more result lines to this integration test file’s section. Each
integration test file has its own section, so if we add more files in the
*tests* directory, there will be more integration test sections.

Da mesma forma que adicionar mais funções de teste de unidade adiciona mais linhas 
de resultado à seção de testes de unidade, adicionar mais funções de teste ao arquivo 
de teste de integração adiciona mais linhas de resultado à seção deste arquivo de 
teste de integração. Cada arquivo de teste de integração possui sua própria seção; 
portanto, se adicionarmos mais arquivos no diretório *tests*, haverá mais seções de 
teste de integração.

We can still run a particular integration test function by specifying the test
function’s name as an argument to `cargo test`. To run all the tests in a
particular integration test file, use the `--test` argument of `cargo test`
followed by the name of the file:

Ainda podemos executar uma função de teste de integração específica, especificando o 
nome da função de teste como argumento para `cargo test`. Para executar todos os testes 
em um arquivo de teste de integração específico, use o argumento `--test` de `cargo test` 
seguido do nome do arquivo:

```text
$ cargo test --test integration_test
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/integration_test-952a27e0126bb565

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

This command runs only the tests in the *tests/integration_test.rs* file.

Esse comando executa apenas o teste no arquivo  *tests/integration_test.rs*.

#### Submodules in Integration Tests
#### Sub-Módulos em Testes de Integração

As you add more integration tests, you might want to make more than one file in
the *tests* directory to help organize them; for example, you can group the
test functions by the functionality they’re testing. As mentioned earlier, each
file in the *tests* directory is compiled as its own separate crate.

À medida que você adiciona mais testes de integração, convém criar mais de um 
arquivo no diretório *tests* para ajudar a organizá-los; por exemplo, você pode 
agrupar as funções de teste pela funcionalidade que eles estão testando. Como 
mencionado anteriormente, cada arquivo no diretório *tests* é compilado como 
seu próprio crate separado.

Treating each integration test file as its own crate is useful to create
separate scopes that are more like the way end users will be using your crate.
However, this means files in the *tests* directory don’t share the same
behavior as files in *src* do, as you learned in Chapter 7 regarding how to
separate code into modules and files.

Tratar cada arquivo de teste de integração como seu próprio crate é útil para criar 
escopos separados, mais parecidos com o modo como os usuários finais usarão seu crate. 
No entanto, isso significa que os arquivos no diretório *tests* não compartilham o 
mesmo comportamento que os arquivos em *src*, como você aprendeu no Capítulo 7 sobre 
como separar o código em módulos e arquivos.

The different behavior of files in the *tests* directory is most noticeable
when you have a set of helper functions that would be useful in multiple
integration test files and you try to follow the steps in the “Moving Modules
to Other Files” section of Chapter 7 to extract them into a common module. For
example, if we create *tests/common.rs* and place a function named `setup` in
it, we can add some code to `setup` that we want to call from multiple test
functions in multiple test files:

O comportamento diferente dos arquivos no diretório *tests* é mais perceptível 
quando você tem um conjunto de funções auxiliares que seriam úteis em vários 
arquivos de teste de integração e tenta seguir as etapas na seção “Movendo 
Módulos para Outros Arquivos” do capítulo 7 para extraí-los em um módulo comum. 
Por exemplo, se criarmos *tests/common.rs* e colocarmos uma função chamada `setup` 
nela, poderemos adicionar algum código à `setup` que queremos chamar de várias 
funções de teste em vários arquivos de teste:

<span class="filename">Filename: tests/common.rs</span>

<span class="filename">Nome do arquivo: tests/common.rs</span>

```rust
pub fn setup() {
    // setup code specific to your library's tests would go here
}
```

When we run the tests again, we’ll see a new section in the test output for the
*common.rs* file, even though this file doesn’t contain any test functions nor
did we call the `setup` function from anywhere:

```text
running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/common-b8b07b6f1be2db70

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-d993c68b431d39df

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Having `common` appear in the test results with `running 0 tests` displayed for
it is not what we wanted. We just wanted to share some code with the other
integration test files.

Ter `common` aparecendo nos resultados do teste com `running 0 tests` (executando 
0 testes) exibido, pois não é o que queríamos. Nós apenas queríamos compartilhar 
algum código com os outros arquivos de teste de integração.

To avoid having `common` appear in the test output, instead of creating
*tests/common.rs*, we’ll create *tests/common/mod.rs*. In the “Rules of Module
Filesystems” section of Chapter 7, we used the naming convention
*module_name/mod.rs* for files of modules that have submodules. We don’t have
submodules for `common` here, but naming the file this way tells Rust not to
treat the `common` module as an integration test file. When we move the `setup`
function code into *tests/common/mod.rs* and delete the *tests/common.rs* file,
the section in the test output will no longer appear. Files in subdirectories
of the *tests* directory don’t get compiled as separate crates or have sections
in the test output.

Para evitar que `common` apareça na saída do teste, em vez de criar *tests/common.rs*, 
criaremos *tests/common/mod.rs*. Na seção “Regras dos Sistemas de Arquivos do Módulo” 
do Capítulo 7, usamos a convenção de nomenclatura *module_name/mod.rs* para arquivos 
de módulos que possuem submódulos. Não temos submódulos para `common` aqui, mas nomear 
o arquivo dessa maneira indica ao Rust para não trate o módulo `common` como um arquivo 
de teste de integração. Quando movermos o código de função `setup` para *tests/common/mod.rs* 
e excluirmos o arquivo *tests/common.rs*, a seção na saída de teste não aparecerá mais. Os 
arquivos nos subdiretórios do diretório *tests* não são compilados como crates separadas ou 
possuem seções na saída do teste.

After we’ve created *tests/common/mod.rs*, we can use it from any of the
integration test files as a module. Here’s an example of calling the `setup`
function from the `it_adds_two` test in *tests/integration_test.rs*:

Depois de criarmos *tests/common/mod.rs*, podemos usá-lo a partir de qualquer arquivo de 
teste de integração como módulo. Aqui está um exemplo de como chamar a função `setup` 
do teste `it_adds_two` em *tests/integration_test.rs*:

<span class="filename">Filename: tests/integration_test.rs</span>

<span class="filename">Nome do arquivo: tests/integration_test.rs</span>

```rust,ignore
extern crate adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

Note that the `mod common;` declaration is the same as the module declarations
we demonstrated in Listing 7-4. Then in the test function, we can call the
`common::setup()` function.

Observe que a declaração `mod common;` é igual às declarações do módulo que 
demonstramos na Listagem 7-4. Então, na função de teste, podemos chamar a 
função `common::setup()`.

#### Integration Tests for Binary Crates
#### Testes de Integração para Crates Binárias

If our project is a binary crate that only contains a *src/main.rs* file and
doesn’t have a *src/lib.rs* file, we can’t create integration tests in the
*tests* directory and use `extern crate` to import functions defined in the
*src/main.rs* file. Only library crates expose functions that other crates can
call and use; binary crates are meant to be run on their own.

Se nosso projeto é uma crate binária que contém apenas um arquivo *src/main.rs* e 
não possui um arquivo *src/lib.rs*, não podemos criar testes de integração no 
diretório *tests* e usar `extern crate` para importar funções definidas no arquivo 
*src/main.rs*. Somente crates de biblioteca expõem funções que outras crates podem 
chamar e usar; crates binárias são feitas para serem executadas por conta própria.

This is one of the reasons Rust projects that provide a binary have a
straightforward *src/main.rs* file that calls logic that lives in the
*src/lib.rs* file. Using that structure, integration tests *can* test the
library crate by using `extern crate` to exercise the important functionality.
If the important functionality works, the small amount of code in the
*src/main.rs* file will work as well, and that small amount of code doesn’t
need to be tested.

Esse é um dos motivos pelos quais os projetos Rust que fornecem um binário têm um 
arquivo *src/main.rs* simples que chama a lógica que vive no arquivo *src/lib.rs*. 
Usando essa estrutura, os testes de integração *podem* testar a biblioteca usando 
`extern crate` para exercitar a importante funcionalidade. Se a funcionalidade 
importante funcionar, a pequena quantidade de código no arquivo *src/main.rs* 
também funcionará, e essa pequena quantidade de código não precisará ser testada.

## Summary
## Resumo

Rust’s testing features provide a way to specify how code should function to
ensure it continues to work as you expect, even as you make changes. Unit tests
exercise different parts of a library separately and can test private
implementation details. Integration tests check that many parts of the library
work together correctly, and they use the library’s public API to test the code
in the same way external code will use it. Even though Rust’s type system and
ownership rules help prevent some kinds of bugs, tests are still important to
reduce logic bugs having to do with how your code is expected to behave.

Os recursos de teste em Rust fornecem uma maneira de especificar como o código deve 
funcionar para garantir que continue funcionando conforme o esperado, mesmo quando 
você faz alterações. Os testes de unidade exercitam partes diferentes de uma 
biblioteca separadamente e podem testar os detalhes da implementação privada. Os 
testes de integração verificam se muitas partes da biblioteca funcionam juntas corretamente 
e usam a API pública da biblioteca para testar o código da mesma forma que o código 
externo o usará. Mesmo que o sistema de tipos de Rust e as regras de ownership ajudam 
a evitar alguns tipos de bugs, os testes ainda são importantes para reduzir os bugs 
lógicos relacionados ao comportamento do seu código.

Let’s combine the knowledge you learned in this chapter and in previous
chapters to work on a project!

Vamos combinar o conhecimento que você aprendeu neste capítulo e nos capítulos anteriores 
para trabalhar em um projeto!