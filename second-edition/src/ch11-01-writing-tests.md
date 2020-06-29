## How to Write Tests
## Como Escrever Testes

Tests are Rust functions that verify that the non-test code is functioning in
the expected manner. The bodies of test functions typically perform these three
actions:

1. Set up any needed data or state.
2. Run the code you want to test.
3. Assert the results are what you expect.

Os testes são funções de Rust que verificam se o código (non-test) está funcionando 
da maneira esperada. Os corpos das funções de teste geralmente executam essas três ações:

1. Configurar todos os dados ou estados necessários.
2. Executar o código que você deseja testar.
3. Afirmar que os resultados são o que você espera.

Let’s look at the features Rust provides specifically for writing tests that
take these actions, which include the `test` attribute, a few macros, and the
`should_panic` attribute.

Vejamos os recursos que Rust fornece especificamente para escrever testes que 
executam essas ações, que incluem o atributo `test`, algumas macros e o atributo 
`should_panic`.

### The Anatomy of a Test Function
### Anatomia da Função de Teste

At its simplest, a test in Rust is a function that’s annotated with the `test`
attribute. Attributes are metadata about pieces of Rust code; one example is
the `derive` attribute we used with structs in Chapter 5. To change a function
into a test function, add `#[test]` on the line before `fn`. When you run your
tests with the `cargo test` command, Rust builds a test runner binary that runs
the functions annotated with the `test` attribute and reports on whether each
test function passes or fails.

Na sua forma mais simples, um teste em Rust é uma função anotada com o atributo 
`test`. Atributos são metadados sobre partes do código Rust; um exemplo é o atributo 
`derive` que usamos com estruturas no Capítulo 5. Para mudar uma função para uma 
função de teste, adicione `#[test]` na linha antes de `fn`. Quando você executa 
seus testes com o comando `cargo test`, Rust cria um binário executor de teste 
que executa as funções anotadas com o atributo `test` e relata se cada função de 
teste passa ou falha.

In Chapter 7, we saw that when we make a new library project with Cargo, a test
module with a test function in it is automatically generated for us. This
module helps you start writing your tests so you don’t have to look up the
exact structure and syntax of test functions every time you start a new
project. You can add as many additional test functions and as many test modules
as you want!

No capítulo 7, vimos que, quando fazemos um novo projeto de biblioteca com Cargo, 
um módulo de teste com uma função de teste é gerada automaticamente para nós. Este 
módulo ajuda você a começar a escrever seus testes, para que você não precise procurar 
a estrutura exata e a sintaxe das funções de teste toda vez que iniciar um novo projeto. 
Você pode adicionar quantas funções de teste adicionais e quantos módulos de teste desejar!

We’ll explore some aspects of how tests work by experimenting with the template
test generated for us without actually testing any code. Then we’ll write some
real-world tests that call some code that we’ve written and assert that its
behavior is correct.

Vamos explorar alguns aspectos de como os testes funcionam, experimentando o teste de 
modelo gerado para nós sem realmente testar qualquer código. Em seguida, escreveremos 
alguns testes do mundo real que chamam algum código que escrevemos e afirmamos (assert) 
que seu comportamento está correto.

Let’s create a new library project called `adder`:

Vamos criar um novo projeto de biblioteca chamado `adder`:

```text
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

The contents of the *src/lib.rs* file in your `adder` library should look like
Listing 11-1:

O conteúdo do arquivo *src/lib.rs* na sua biblioteca `adder` deve se parecer 
com a Listagem 11-1:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

<span class="caption">Listing 11-1: The test module and function generated
automatically by `cargo new`</span>

<span class="caption">Listagem 11-1: O módulo e a função de teste gerados 
automaticamente pelo `cargo new`</span>

For now, let’s ignore the top two lines and focus on the function to see how it
works. Note the `#[test]` annotation before the `fn` line: this attribute
indicates this is a test function, so the test runner knows to treat this
function as a test. We could also have non-test functions in the `tests` module
to help set up common scenarios or perform common operations, so we need to
indicate which functions are tests by using the `#[test]` attribute.

Por enquanto, vamos ignorar as duas principais linhas e focar na função para ver 
como ela funciona. Observe a anotação `#[test]` antes da linha `fn`: este atributo 
indica que esta é uma função de teste; portanto, o executor de testes sabe tratar 
essa função como um teste. Também poderíamos ter funções que não são de teste no módulo 
`tests` para ajudar a configurar cenários comuns ou executar operações comuns, portanto, 
precisamos indicar quais funções são testes usando o atributo `#[test]`.

The function body uses the `assert_eq!` macro to assert that 2 + 2 equals 4.
This assertion serves as an example of the format for a typical test. Let’s run
it to see that this test passes.

O corpo da função usa a macro `assert_eq!` Para afirmar que 2 + 2 é igual a 4. Esta 
asserção (afirmação) serve como um exemplo do formato para um teste típico. Vamos 
ver se esse teste passa.

The `cargo test` command runs all tests in our project, as shown in Listing
11-2:

O comando `cargo test` executa todos os testes em nosso projeto, como mostra a 
Listagem 11-2:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.22 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

<span class="caption">Listing 11-2: The output from running the automatically
generated test</span>

<span class="caption">Listagem 11-2: A saída da execução do teste gerado 
automaticamente</span>

Cargo compiled and ran the test. After the `Compiling`, `Finished`, and
`Running` lines is the line `running 1 test`. The next line shows the name
of the generated test function, called `it_works`, and the result of running
that test, `ok`. The overall summary of running the tests appears next. The
text `test result: ok.` means that all the tests passed, and the portion that
reads `1 passed; 0 failed` totals the number of tests that passed or failed.

Cargo compilou e executou o teste. Após as linhas `Compiling`, `Finished` e `Running`, 
é a linha `running 1 test`. A próxima linha mostra o nome da função de teste gerada, 
chamada `it_works`, e o resultado da execução desse teste, `ok`. O resumo geral da 
execução dos testes é exibido a seguir. O texto `test result: ok` significa que todos 
os testes passaram e a parte que lê `1 passed; 0 failed` totaliza o número de 
testes que passaram ou falharam.

Because we don’t have any tests we’ve marked as ignored, the summary shows `0
ignored`. We also haven’t filtered the tests being run, so the end of the
summary shows `0 filtered out`. We’ll talk about ignoring and filtering out
tests in the next section, “Controlling How Tests Are Run.”

Como não temos nenhum teste marcado como ignorado, o resumo mostra `0 ignored`. 
Também não filtramos os testes que estão sendo executados; portanto, o final do 
resumo mostra `0 filtered out`. Falaremos sobre ignorar e filtrar testes na próxima 
seção, "Controlando Como os Testes são Executados".

The `0 measured` statistic is for benchmark tests that measure performance.
Benchmark tests are, as of this writing, only available in nightly Rust. See
[the documentation about benchmark tests][bench] to learn more.

A estatística `0 measured` (0 medido) é para testes de benchmark que medem o 
desempenho. Os testes de benchmark estão, até o momento da redação deste documento, 
disponíveis apenas em Rust nightly. Consulte [a documentação sobre testes de benchmark][bench] 
para saber mais.

[bench]: ../../unstable-book/library-features/test.html

The next part of the test output, which starts with `Doc-tests adder`, is for
the results of any documentation tests. We don’t have any documentation tests
yet, but Rust can compile any code examples that appear in our API
documentation. This feature helps us keep our docs and our code in sync! We’ll
discuss how to write documentation tests in the “Documentation Comments”
section of Chapter 14. For now, we’ll ignore the `Doc-tests` output.

A próxima parte da saída de teste, que começa com `Doc-tests adder`, é para os 
resultados de todos os testes de documentação. Ainda não temos testes de documentação, 
mas Rust pode compilar todos os exemplos de código que aparecem na documentação da API. 
Esse recurso nos ajuda a manter nossos documentos e nosso código sincronizados! Discutiremos 
como escrever testes de documentação na seção "Comentários da Documentação" do Capítulo 14. 
Por enquanto, ignoraremos a saída `Doc-tests`.

Let’s change the name of our test to see how that changes the test output.
Change the `it_works` function to a different name, such as `exploration`, like
so:

Vamos mudar o nome do nosso teste para ver como isso altera a saída do teste. Altere 
a função `it_works` para um nome diferente, como `exploration`, da seguinte forma:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
}
```

Then run `cargo test` again. The output now shows `exploration` instead of
`it_works`:

Em seguida, execute o `cargo test` novamente. A saída agora mostra `exploration` 
em vez de` it_works`:

```text
running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Let’s add another test, but this time we’ll make a test that fails! Tests fail
when something in the test function panics. Each test is run in a new thread,
and when the main thread sees that a test thread has died, the test is marked
as failed. We talked about the simplest way to cause a panic in Chapter 9,
which is to call the `panic!` macro. Enter the new test, `another`, so your
*src/lib.rs* file looks like Listing 11-3:

Vamos adicionar outro teste, mas desta vez faremos um teste que falhará! Os testes 
falham quando algo na função de teste entra em pânico. Cada teste é executado em 
um novo thread e, quando o thread principal vê que um thread de teste morreu, o 
teste é marcado como com falha. Falamos sobre a maneira mais simples de causar 
pânico no Capítulo 9, que é a macro `panic!`. Digite o novo teste, `another`, para 
que seu arquivo *src/lib.rs* se pareça com a Listagem 11-3:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```

<span class="caption">Listing 11-3: Adding a second test that will fail because
we call the `panic!` macro</span>

<span class="caption">Listagem 11-3: Adicionando um segundo teste que falhará 
porque chamamos a macro `panic!`</span>

Run the tests again using `cargo test`. The output should look like Listing
11-4, which shows that our `exploration` test passed and `another` failed:

Execute os testes novamente usando o `cargo test`. A saída deve se parecer com 
a Listagem 11-4, que mostra que nosso teste `exploration` passou e `another` falhou:

```text
running 2 tests
test tests::exploration ... ok
test tests::another ... FAILED

failures:

---- tests::another stdout ----
    thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:10:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed
```

<span class="caption">Listing 11-4: Test results when one test passes and one
test fails</span>

<span class="caption">Listagem 11-4: Resultados de teste quando um teste passa e 
outro falha</span>

Instead of `ok`, the line `test tests::another` shows `FAILED`. Two new
sections appear between the individual results and the summary: the first
section displays the detailed reason for each test failure. In this case,
`another` failed because it `panicked at 'Make this test fail'`, which happened
on line 10 in the *src/lib.rs* file. The next section lists just the names of
all the failing tests, which is useful when there are lots of tests and lots of
detailed failing test output. We can use the name of a failing test to run just
that test to more easily debug it; we’ll talk more about ways to run tests in
the “Controlling How Tests Are Run” section.

Em vez de `ok`, a linha `test tests::another` mostra `FAILED` (falha). Duas novas seções 
aparecem entre os resultados individuais e o resumo: a primeira seção exibe o motivo 
detalhado de cada falha no teste. Nesse caso, `another` falhou porque  `panicked at 'Make 
this test fail'` (entrou em pânico em 'Make this test fail'), o que aconteceu na linha 10 
no arquivo *src/lib.rs*. A próxima seção lista apenas os nomes de todos os testes com falha, 
o que é útil quando há muitos testes e muitas saídas detalhadas de teste com falha. 
Podemos usar o nome de um teste com falha para executar apenas esse teste para depurá-lo 
com mais facilidade; falaremos mais sobre maneiras de executar testes na seção 
"Controlando como os Testes são Executados".

The summary line displays at the end: overall, our test result is `FAILED`.
We had one test pass and one test fail.

A linha de resumo é exibida no final: no geral, o resultado do nosso teste é `FAILED` (falha). 
Tivemos teste que passou e uma falha de teste.

Now that you’ve seen what the test results look like in different scenarios,
let’s look at some macros other than `panic!` that are useful in tests.

Agora que você viu como são os resultados dos testes em diferentes cenários, vamos 
ver algumas macros que não sejam `panic!` que são úteis nos testes.

### Checking Results with the `assert!` Macro
### Verificando Resultados com a Macro `assert!`

The `assert!` macro, provided by the standard library, is useful when you want
to ensure that some condition in a test evaluates to `true`. We give the
`assert!` macro an argument that evaluates to a Boolean. If the value is
`true`, `assert!` does nothing and the test passes. If the value is `false`,
the `assert!` macro calls the `panic!` macro, which causes the test to fail.
Using the `assert!` macro helps us check that our code is functioning in the
way we intend.

A macro `assert!`, fornecida pela biblioteca padrão, é útil quando você deseja garantir 
que alguma condição em um teste seja avaliada como `true`. Damos à macro `assert!` um 
argumento que avalia como um booleano. Se o valor for `true`,`assert!` não faz nada e o 
teste passa. Se o valor for `false`, a macro `assert!` chama a macro `panic!`, o que 
causa falha no teste. Usar a macro `assert!` nos ajuda a verificar se nosso código está 
funcionando da maneira que pretendemos.

In Chapter 5, Listing 5-15, we used a `Rectangle` struct and a `can_hold`
method, which are repeated here in Listing 11-5. Let’s put this code in the
*src/lib.rs* file and write some tests for it using the `assert!` macro.

No Capítulo 5, Listagem 5-15, usamos uma estrutura `Rectangle` e um método 
`can_hold`, que são repetidos aqui na Listagem 11-5. Vamos colocar esse código no 
arquivo *src/lib.rs* e escrever alguns testes usando a macro `assert!`

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[derive(Debug)]
pub struct Rectangle {
    length: u32,
    width: u32,
}

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length > other.length && self.width > other.width
    }
}
```

<span class="caption">Listing 11-5: Using the `Rectangle` struct and its
`can_hold` method from Chapter 5</span>

<span class="caption">Listagem 11-5: Usando a estrutura `Rectangle` e seu 
método `can_hold` do Capítulo 5</span>

The `can_hold` method returns a Boolean, which means it’s a perfect use case
for the `assert!` macro. In Listing 11-6, we write a test that exercises the
`can_hold` method by creating a `Rectangle` instance that has a length of 8 and
a width of 7 and asserting that it can hold another `Rectangle` instance that
has a length of 5 and a width of 1:

O método `can_hold` retorna um booleano, o que significa que é um caso de uso 
perfeito para a macro `assert!`. Na Listagem 11-6, escrevemos um teste que exercita 
o método `can_hold` criando uma instância `Rectangle` que possui um comprimento de 
8 e uma largura de 7 e afirmando (asserting) que ele pode conter outra instância 
`Rectangle` que possui um comprimento de 5 e uma largura de 1:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(larger.can_hold(&smaller));
    }
}
```

<span class="caption">Listing 11-6: A test for `can_hold` that checks whether a
larger rectangle can indeed hold a smaller rectangle</span>

<span class="caption">Listagem 11-6: Um teste para `can_hold` que verifica se 
um retângulo maior pode realmente conter um retângulo menor</span>

Note that we’ve added a new line inside the `tests` module: `use super::*;`.
The `tests` module is a regular module that follows the usual visibility rules
we covered in Chapter 7 in the “Privacy Rules” section. Because the `tests`
module is an inner module, we need to bring the code under test in the outer
module into the scope of the inner module. We use a glob here so anything we
define in the outer module is available to this `tests` module.

Observe que adicionamos uma nova linha dentro do módulo `tests`:`use super::*;`. 
O módulo `tests` é um módulo regular que segue as regras de visibilidade usuais 
que abordamos no capítulo 7 na seção "Regras de Privacidade". Como o módulo `tests` 
é um módulo interno, precisamos colocar o código em teste no módulo externo no 
escopo do módulo interno. Nós usamos um _glob_ aqui, então qualquer coisa que 
definimos no módulo externo está disponível para este módulo `tests`.

We’ve named our test `larger_can_hold_smaller`, and we’ve created the two
`Rectangle` instances that we need. Then we called the `assert!` macro and
passed it the result of calling `larger.can_hold(&smaller)`. This expression
is supposed to return `true`, so our test should pass. Let’s find out!

Nomeamos nosso teste como `larger_can_hold_smaller` e criamos as duas instâncias 
de `Rectangle` que precisamos. Então chamamos a macro `assert!` e passamos o 
resultado da chamada `larger.can_hold(&smaller)`. Esta expressão deve retornar 
`true`, portanto nosso teste deve passar. Vamos descobrir!

```text
running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

It does pass! Let’s add another test, this time asserting that a smaller
rectangle cannot hold a larger rectangle:

Passa! Vamos adicionar outro teste, desta vez afirmando que um retângulo menor 
não pode conter um retângulo maior:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(!smaller.can_hold(&larger));
    }
}
```

Because the correct result of the `can_hold` function in this case is `false`,
we need to negate that result before we pass it to the `assert!` macro. As a
result, our test will pass if `can_hold` returns `false`:

Como o resultado correto da função `can_hold` nesse caso é `false`, precisamos 
negar esse resultado antes de passá-lo para a macro `assert!`. Como resultado, 
nosso teste será aprovado se `can_hold` retornar `false`:

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Two tests that pass! Now let’s see what happens to our test results when we
introduce a bug in our code. Let’s change the implementation of the `can_hold`
method by replacing the greater-than sign with a less-than sign when it
compares the lengths:

Dois testes que passam! Agora vamos ver o que acontece com os resultados de nossos 
testes quando introduzimos um bug em nosso código. Vamos alterar a implementação 
do método `can_hold` substituindo o sinal maior que por um sinal menor no momento que 
ele compara os comprimentos:

```rust
# fn main() {}
# #[derive(Debug)]
# pub struct Rectangle {
#     length: u32,
#     width: u32,
# }
// --snip--

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length < other.length && self.width > other.width
    }
}
```

Running the tests now produces the following:

Executando os testes agora produz o seguinte:

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... FAILED

failures:

---- tests::larger_can_hold_smaller stdout ----
    thread 'tests::larger_can_hold_smaller' panicked at 'assertion failed:
    larger.can_hold(&smaller)', src/lib.rs:22:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

Our tests caught the bug! Because `larger.length` is 8 and `smaller.length` is
5, the comparison of the lengths in `can_hold` now returns `false`: 8 is not
less than 5.

Nossos testes pegaram o bug! Como `larger.length` é 8 e `small.length` é 5, a 
comparação dos comprimentos em `can_hold` agora retorna `false`: 8 não é 
menor que 5.

### Testing Equality with the `assert_eq!` and `assert_ne!` Macros
### Testando a Igualdade com as Macros `assert_eq!` e `assert_ne!`

A common way to test functionality is to compare the result of the code under
test to the value you expect the code to return to make sure they’re equal. You
could do this using the `assert!` macro and passing it an expression using the
`==` operator. However, this is such a common test that the standard library
provides a pair of macros—`assert_eq!` and `assert_ne!`—to perform this test
more conveniently. These macros compare two arguments for equality or
inequality, respectively. They’ll also print the two values if the assertion
fails, which makes it easier to see *why* the test failed; conversely, the
`assert!` macro only indicates that it got a `false` value for the `==`
expression, not the values that lead to the `false` value.

Uma maneira comum de testar a funcionalidade é comparar o resultado do código 
sob teste com o valor que você espera que o código retorne para garantir que sejam 
iguais. Você pode fazer isso usando a macro `assert!` e passando uma expressão usando 
o operador `==`. No entanto, esse é um teste tão comum que a biblioteca padrão fornece 
um par de macros —`assert_eq!` e `assert_ne!`— para executar esse teste mais 
convenientemente. Essas macros comparam dois argumentos para igualdade ou desigualdade, 
respectivamente. Eles também imprimirão os dois valores se a asserção falhar, o que 
facilita ver *por que* o teste falhou; por outro lado, a macro `assert!` indica apenas 
que obteve um valor `false` para a expressão `==`, não os valores que levam ao 
valor `false`.

In Listing 11-7, we write a function named `add_two` that adds `2` to its
parameter and returns the result. Then we test this function using the
`assert_eq!` macro.

Na Listagem 11-7, escrevemos uma função chamada `add_two` que adiciona `2` ao seu 
parâmetro e retorna o resultado. Então testamos esta função usando a macro `assert_eq!`.

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

<span class="caption">Listing 11-7: Testing the function `add_two` using the
`assert_eq!` macro</span>

<span class="caption">Listagem 11-7: Testando a função `add_two` usando a 
macro `assert_eq!`</span>

Let’s check that it passes!

Vamos verificar se passa!

```text
running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

The first argument we gave to the `assert_eq!` macro, `4`, is equal to the
result of calling `add_two(2)`. The line for this test is `test
tests::it_adds_two ... ok`, and the `ok` text indicates that our test passed!

O primeiro argumento que fornecemos à macro `assert_eq!`, `4`, é igual ao 
resultado da chamada de `add_two(2)`. A linha para este teste é `test tests::it_adds_two 
... ok`, e o texto `ok` indica que nosso teste passou!

Let’s introduce a bug into our code to see what it looks like when a test that
uses `assert_eq!` fails. Change the implementation of the `add_two` function to
instead add `3`:

Vamos introduzir um bug em nosso código para ver como ele fica quando um teste que 
usa `assert_eq!` falha. Mude a implementação da função `add_two` para adicionar `3`:

```rust
# fn main() {}
pub fn add_two(a: i32) -> i32 {
    a + 3
}
```

Run the tests again:

Execute os testes novamente:

```text
running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
        thread 'tests::it_adds_two' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:11:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

Our test caught the bug! The `it_adds_two` test failed, displaying the message
`` assertion failed: `(left == right)` `` and showing that `left` was `4` and
`right` was `5`. This message is useful and helps us start debugging: it means
the `left` argument to `assert_eq!` was `4` but the `right` argument, where we
had `add_two(2)`, was `5`.

Nosso teste pegou o bug! O teste `it_adds_two` falhou, exibindo a mensagem
`` assertion failed: `(left == right)` `` e mostrando que `left` era `4` e
`right` era `5`. Esta mensagem é útil e nos ajuda a iniciar a depuração: 
significa que o argumento `left` para `assert_eq!` era `4` mas o argumento 
`right`, onde tínhamos `add_two(2)`, era `5`.

Note that in some languages and test frameworks, the parameters to the
functions that assert two values are equal are called `expected` and `actual`,
and the order in which we specify the arguments matters. However, in Rust,
they’re called `left` and `right`, and the order in which we specify the value
we expect and the value that the code under test produces doesn’t matter. We
could write the assertion in this test as `assert_eq!(add_two(2), 4)`, which
would result in a failure message that displays `` assertion failed: `(left ==
right)` `` and that `left` was `5` and `right` was `4`.

Observe que em algumas linguagens e estruturas de teste, os parâmetros para as 
funções que afirmam que dois valores são iguais são chamados de `expected` e
`actual`, e a ordem em que especificamos os argumentos é importante. No entanto, 
em Rust, eles são chamados de `left` e `right`, e a ordem em que especificamos o 
valor que esperamos e o valor que o código em teste produz não importa. Poderíamos 
escrever a asserção neste teste como `assert_eq!(add_two(2), 4)`, o que resultaria 
em uma mensagem de falha que exibia `` assertion failed: `(left == right)` `` onde 
`left` era `5` e `right` era `4`.

The `assert_ne!` macro will pass if the two values we give it are not equal and
fail if they’re equal. This macro is most useful for cases when we’re not sure
what a value *will* be, but we know what the value definitely *won’t* be if our
code is functioning as we intend. For example, if we’re testing a function that
is guaranteed to change its input in some way, but the way in which the input
is changed depends on the day of the week that we run our tests, the best thing
to assert might be that the output of the function is not equal to the input.

A macro `assert_ne!` será aprovada se os dois valores que fornecemos não forem 
iguais e falhará se forem iguais. Essa macro é mais útil para casos em que não 
temos certeza de qual *será* o valor, mas sabemos qual *não* será o valor 
definitivamente se o nosso código estiver funcionando como pretendemos. Por exemplo, 
se estamos testando uma função que é garantida para alterar sua entrada de alguma 
forma, mas a maneira como a entrada é alterada depende do dia da semana em que 
executamos nossos testes, a melhor coisa a afirmar pode ser que a saída da função 
não seja igual à entrada.

Under the surface, the `assert_eq!` and `assert_ne!` macros use the operators
`==` and `!=`, respectively. When the assertions fail, these macros print their
arguments using debug formatting, which means the values being compared must
implement the `PartialEq` and `Debug` traits. All the primitive types and most
of the standard library types implement these traits. For structs and enums
that you define, you’ll need to implement `PartialEq` to assert that values of
those types are equal or not equal. You’ll need to implement `Debug` to print
the values when the assertion fails. Because both traits are derivable traits,
as mentioned in Listing 5-12 in Chapter 5, this is usually as straightforward
as adding the `#[derive(PartialEq, Debug)]` annotation to your struct or enum
definition. See Appendix C, “Derivable Traits,” for more details about these
and other derivable traits.

Sob a superfície, as macros `assert_eq!` e `assert_ne!` usam os operadores `==` e
 `!=`, respectivamente. Quando as asserções falham, essas macros imprimem seus 
 argumentos usando a formatação de depuração, o que significa que os valores 
 comparados devem implementar as traits `PartialEq` e` Debug`. Todos os tipos 
 primitivos e a maioria dos tipos de biblioteca padrão implementam essas traits. 
 Para estruturas e enumerações definidas por você, é necessário implementar o 
 `PartialEq` para afirmar que os valores desses tipos são iguais ou não. Você 
 precisará implementar `Debug` para imprimir os valores quando a asserção falhar. 
 Como ambas as traits são deriváveis, conforme mencionado na Listagem 5-12 no 
 Capítulo 5, isso geralmente é tão simples quanto adicionar a anotação 
 `#[derive(PartialEq, Debug)]` à sua definição de struct ou enum. Consulte o Apêndice 
 C, “Traits Deriváveis”, para obter mais detalhes sobre essas e outras traits deriváveis.

### Adding Custom Failure Messages
### Adicionando Mensagens de Falha Personalizadas

You can also add a custom message to be printed with the failure message as
optional arguments to the `assert!`, `assert_eq!`, and `assert_ne!` macros. Any
arguments specified after the one required argument to `assert!` or the two
required arguments to `assert_eq!` and `assert_ne!` are passed along to the
`format!` macro (discussed in Chapter 8 in the “Concatenation with the `+`
Operator or the `format!` Macro” section), so you can pass a format string that
contains `{}` placeholders and values to go in those placeholders. Custom
messages are useful to document what an assertion means; when a test fails,
you’ll have a better idea of what the problem is with the code.

Você também pode adicionar uma mensagem personalizada para ser impressa com a 
mensagem de falha como argumentos opcionais nas macros `assert!`, `assert_eq!` e 
`assert_ne!`. Quaisquer argumentos especificados após o argumento necessário para 
`assert!` ou os dois argumentos necessários para `assert_eq!` e `assert_ne!` são 
transmitidos para a macro `format!` (Discutida no Capítulo 8 em "Concatenação com o 
`+` operador ou a seção `format!` macro”), para que você possa passar uma string de 
formato que contém espaços reservados `{}` e valores para esses espaços reservados. 
Mensagens personalizadas são úteis para documentar o que significa uma asserção; quando 
um teste falha, você terá uma idéia melhor de qual é o problema com o código.

For example, let’s say we have a function that greets people by name and we
want to test that the name we pass into the function appears in the output:

Por exemplo, digamos que temos uma função que cumprimenta as pessoas pelo nome e 
queremos testar se o nome que passamos na função aparece na saída:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}
```

The requirements for this program haven’t been agreed upon yet, and we’re
pretty sure the `Hello` text at the beginning of the greeting will change. We
decided we don’t want to have to update the test when the requirements change,
so instead of checking for exact equality to the value returned from the
`greeting` function, we’ll just assert that the output contains the text of the
input parameter.

Os requisitos para este programa ainda não foram acordados e temos certeza de que 
o texto `Hello` no início da saudação será alterado. Decidimos que não queremos 
atualizar o teste quando os requisitos forem alterados; portanto, em vez de verificar 
se há igualdade exata com o valor retornado da função `greeting`, apenas afirmaremos 
que a saída contém o texto da entrada parâmetro.

Let’s introduce a bug into this code by changing `greeting` to not include
`name` to see what this test failure looks like:

Vamos introduzir um bug nesse código, alterando `greeting` para não incluir `name` para 
ver como é essa falha no teste:

```rust
# fn main() {}
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}
```

Running this test produces the following:

A execução deste teste produz o seguinte:

```text
running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'assertion failed:
result.contains("Carol")', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::greeting_contains_name
```

This result just indicates that the assertion failed and which line the
assertion is on. A more useful failure message in this case would print the
value we got from the `greeting` function. Let’s change the test function,
giving it a custom failure message made from a format string with a placeholder
filled in with the actual value we got from the `greeting` function:

Esse resultado indica apenas que a asserção falhou e qual linha da asserção está 
ativada. Uma mensagem de falha mais útil nesse caso imprimiria o valor que obtivemos 
da função `greeting`. Vamos alterar a função de teste, fornecendo uma mensagem de 
falha personalizada feita a partir de uma string de formato com um espaço reservado 
preenchido com o valor real que obtivemos da função `greeting`:

```rust,ignore
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```

Now when we run the test, we’ll get a more informative error message:

Agora, quando executarmos o teste, receberemos uma mensagem de erro mais 
informativa:

```text
---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'Greeting did not
contain name, value was `Hello!`', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

We can see the value we actually got in the test output, which would help us
debug what happened instead of what we were expecting to happen.

Podemos ver o valor que realmente obtivemos na saída de teste, o que nos 
ajudaria a depurar o que aconteceu, em vez do que esperávamos que acontecesse.

### Checking for Panics with `should_panic`
### Verificando Pânico com `should_panic`

In addition to checking that our code returns the correct values we expect,
it’s also important to check that our code handles error conditions as we
expect. For example, consider the `Guess` type that we created in Chapter 9,
Listing 9-9. Other code that uses `Guess` depends on the guarantee that `Guess`
instances will contain only values between 1 and 100. We can write a test that
ensures that attempting to create a `Guess` instance with a value outside that
range panics.

Além de verificar se nosso código retorna os valores corretos que esperamos, 
também é importante verificar se nosso código trata as condições de erro conforme 
o esperado. Por exemplo, considere o tipo `Guess` que criamos no Capítulo 9, 
Listagem 9-9. Outro código que usa o `Guess` depende da garantia de que as instâncias 
do `Guess` conterão apenas valores entre 1 e 100. Podemos escrever um teste que 
garanta que a tentativa de criar uma instância do `Guess` com um valor fora intervalo
entre em pânico.

We do this by adding another attribute, `should_panic`, to our test function.
This attribute makes a test pass if the code inside the function panics; the
test will fail if the code inside the function doesn’t panic.

Fazemos isso adicionando outro atributo, `should_panic`, à nossa função de teste. 
Este atributo faz um teste passar se o código dentro da função entrar em pânico; 
o teste falhará se o código dentro da função não entrar em pânico.

Listing 11-8 shows a test that checks that the error conditions of `Guess::new`
happen when we expect them to:

A Listagem 11-8 mostra um teste que verifica se as condições de erro de `Guess::new` 
acontecem quando esperamos delas:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<span class="caption">Listing 11-8: Testing that a condition will cause a
`panic!`</span>

<span class="caption">Listagem 11-8: Testando se uma condição causará um 
`panic!`</span>

We place the `#[should_panic]` attribute after the `#[test]` attribute and
before the test function it applies to. Let’s look at the result when this test
passes:

Colocamos o atributo `#[should_panic]` após o atributo `#[test]` e antes da função 
de teste à qual ele se aplica. Vejamos o resultado quando este teste for aprovado:

```text
running 1 test
test tests::greater_than_100 ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Looks good! Now let’s introduce a bug in our code by removing the condition
that the `new` function will panic if the value is greater than 100:

Parece bom! Agora, vamos introduzir um bug em nosso código, removendo a condição 
de que a função `new` entre em pânico se o valor for maior que 100:

```rust
# fn main() {}
# pub struct Guess {
#     value: u32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1  {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}
```

When we run the test in Listing 11-8, it will fail:

Quando executamos o teste na Listagem 11-8, ele falha:

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

We don’t get a very helpful message in this case, but when we look at the test
function, we see that it’s annotated with `#[should_panic]`. The failure we got
means that the code in the test function did not cause a panic.

Não recebemos uma mensagem muito útil nesse caso, mas, quando analisamos a função 
de teste, vemos que ela é anotada com `#[should_panic]`. A falha obtida significa 
que o código na função de teste não causou pânico.

Tests that use `should_panic` can be imprecise because they only indicate that
the code has caused some panic. A `should_panic` test would pass even if the
test panics for a different reason than the one we were expecting to happen. To
make `should_panic` tests more precise, we can add an optional `expected`
parameter to the `should_panic` attribute. The test harness will make sure that
the failure message contains the provided text. For example, consider the
modified code for `Guess` in Listing 11-9 where the `new` function panics with
different messages depending on whether the value is too small or too large:

Os testes que usam `should_panic` podem ser imprecisos porque indicam apenas que 
o código causou algum pânico. Um teste `should_panic` seria aprovado mesmo que o 
teste entre em pânico por uma razão diferente daquela que esperávamos que acontecesse. 
Para tornar os testes `should_panic` mais precisos, podemos adicionar um parâmetro 
opcional `expected` ao atributo `should_panic`. O mecanismo de teste garantirá que a 
mensagem de falha contenha o texto fornecido. Por exemplo, considere o código 
modificado para `Guess` na Listagem 11-9, onde a função `new` entra em pânico com 
mensagens diferentes, dependendo se o valor é muito pequeno ou muito grande:

<span class="filename">Filename: src/lib.rs</span>

<span class="filename">Nome do arquivo: src/lib.rs</span>

```rust
# fn main() {}
# pub struct Guess {
#     value: u32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 {
            panic!("Guess value must be greater than or equal to 1, got {}.",
                   value);
        } else if value > 100 {
            panic!("Guess value must be less than or equal to 100, got {}.",
                   value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<span class="caption">Listing 11-9: Testing that a condition will cause a
`panic!` with a particular panic message</span>

<span class="caption">Listagem 11-9: Testando se uma condição causará um 
`panic!` com uma mensagem de pânico específica</span>

This test will pass because the value we put in the `should_panic` attribute’s
`expected` parameter is a substring of the message that the `Guess::new`
function panics with. We could have specified the entire panic message that we
expect, which in this case would be `Guess value must be less than or equal to
100, got 200.` What you choose to specify in the expected parameter for
`should_panic` depends on how much of the panic message is unique or dynamic
and how precise you want your test to be. In this case, a substring of the
panic message is enough to ensure that the code in the test function executes
the `else if value > 100` case.

Este teste será aprovado porque o valor que colocamos no parâmetro `expected` do 
atributo `should_panic` é uma substring da mensagem com a qual a função `Guess::new` 
entra em pânico. Poderíamos ter especificado toda a mensagem de pânico que esperamos, 
que nesse caso seria `Guess value must be less than or equal to
100, got 200.` (O valor do palpite deve ser menor ou igual a 100, obteve 200). 
O que você escolhe especificar no parâmetro *expected* para `should_panic` depende 
de como grande parte da mensagem de pânico é única ou dinâmica e quão precisa você 
deseja que seu teste seja. Nesse caso, uma substring da mensagem de pânico é suficiente 
para garantir que o código na função de teste execute o caso `else if value> 100`.

To see what happens when a `should_panic` test with an `expected` message
fails, let’s again introduce a bug into our code by swapping the bodies of the
`if value < 1` and the `else if value > 100` blocks:

Para ver o que acontece quando um teste `should_panic` com uma mensagem `expected` 
falha, vamos introduzir novamente um bug em nosso código trocando os corpos dos 
blocos `if value <1` e `else if value> 100`:

```rust,ignore
if value < 1 {
    panic!("Guess value must be less than or equal to 100, got {}.", value);
} else if value > 100 {
    panic!("Guess value must be greater than or equal to 1, got {}.", value);
}
```

This time when we run the `should_panic` test, it will fail:

Desta vez, quando executarmos o teste `should_panic`, ele falhará:

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

---- tests::greater_than_100 stdout ----
        thread 'tests::greater_than_100' panicked at 'Guess value must be
greater than or equal to 1, got 200.', src/lib.rs:11:12
note: Run with `RUST_BACKTRACE=1` for a backtrace.
note: Panic did not include expected string 'Guess value must be less than or
equal to 100'

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

The failure message indicates that this test did indeed panic as we expected,
but the panic message did not include the expected string `'Guess value must be
less than or equal to 100'`. The panic message that we did get in this case was
`Guess value must be greater than or equal to 1, got 200.` Now we can start
figuring out where our bug is!

A mensagem de falha indica que esse teste realmente entrou em pânico como 
esperávamos, mas a mensagem de pânico não incluiu a sequência esperada 
`'Guess value must be less than or equal to 100'` (O valor do palpite deve ser menor 
ou igual a 100). A mensagem de pânico que recebemos neste caso foi `Guess value must 
be greater than or equal to 1, got 200.` (O palpite deve ser maior ou igual a 1, tem 
200). Agora podemos começar a descobrir onde está o nosso bug!

Now that you know several ways to write tests, let’s look at what is happening
when we run our tests and explore the different options we can use with `cargo
test`.

Agora que você conhece várias maneiras de escrever testes, vejamos o que está 
acontecendo quando executamos nossos testes e exploramos as diferentes opções que 
podemos usar com o `cargo test`.