# Writing Automated Tests
# Escrevendo Testes Automatizados

In his 1972 essay “The Humble Programmer,” Edsger W. Dijkstra said that
“Program testing can be a very effective way to show the presence of bugs, but
it is hopelessly inadequate for showing their absence.” That doesn’t mean we
shouldn’t try to test as much as we can!

Em seu ensaio de 1972, "The Humble Programmer", Edsger W. Dijkstra disse que "O 
teste de programas podem ser uma maneira muito eficaz de mostrar a presença de bugs, 
mas é irremediavelmente inadequado para mostrar sua ausência". Isso não significa que 
não devemos tentar testar o máximo que pudermos!

Correctness in our programs is the extent to which our code does what we intend
it to do. Rust is designed with a high degree of concern about the correctness
of programs, but correctness is complex and not easy to prove. Rust’s type
system shoulders a huge part of this burden, but the type system cannot catch
every kind of incorrectness. As such, Rust includes support for writing
automated software tests within the language.

A correção em nossos programas é a extensão em que nosso código faz o que pretendemos 
fazer. Rust é projetado com um alto grau de preocupação com a correção dos programas, 
mas a correção é complexa e não é fácil de provar. O sistema de tipos Rust suporta uma 
grande parte desse fardo, mas o sistema de tipos não pode pegar todo tipo de erro. Como tal, 
Rust inclui suporte para escrever testes de software automatizados na linguagem.

As an example, say we write a function called `add_two` that adds 2 to whatever
number is passed to it. This function’s signature accepts an integer as a
parameter and returns an integer as a result. When we implement and compile
that function, Rust does all the type checking and borrow checking that you’ve
learned so far to ensure that, for instance, we aren’t passing a `String` value
or an invalid reference to this function. But Rust *can’t* check that this
function will do precisely what we intend, which is return the parameter plus 2
rather than, say, the parameter plus 10 or the parameter minus 50! That’s where
tests come in.

Como exemplo, digamos que escrevemos uma função chamada `add_two` que adiciona 2 ao 
número que for passado para ela. A assinatura desta função aceita um número inteiro como 
parâmetro e retorna um número inteiro como resultado. Quando implementamos e compilamos essa 
função, Rust faz toda a verificação de tipo e de borrow que você aprendeu até agora para 
garantir que, por exemplo, não passemos um valor de `String` ou uma referência inválida à essa 
função. Mas Rust *não pode* verificar se essa função fará exatamente o que pretendemos, que é 
retornar o parâmetro mais 2 em vez de, digamos, o parâmetro mais 10 ou o parâmetro menos 50! É 
aí que entram os testes.

We can write tests that assert, for example, that when we pass `3` to the
`add_two` function, the returned value is `5`. We can run these tests whenever
we make changes to our code to make sure any existing correct behavior has not
changed.

Podemos escrever testes que afirmam, por exemplo, que quando passamos `3` para a 
função `add_two`, o valor retornado é `5`. Podemos executar esses testes sempre que 
fizermos alterações em nosso código para garantir que qualquer comportamento correto 
existente não seja alterado.

Testing is a complex skill: although we can’t cover every detail about how to
write good tests in one chapter, we’ll discuss the mechanics of Rust’s testing
facilities. We’ll talk about the annotations and macros available to you when
writing your tests, the default behavior and options provided for running your
tests, and how to organize tests into unit tests and integration tests.

O teste é uma habilidade complexa: embora não possamos cobrir todos os detalhes sobre 
como escrever  bons testes em um capítulo, discutiremos a mecânica de facilidade de teste 
em Rust. Falaremos sobre as anotações e macros disponíveis para você ao escrever seus testes, 
o comportamento padrão e as opções fornecidas para executar seus testes e como organizar os 
testes em testes de unidade e testes de integração.