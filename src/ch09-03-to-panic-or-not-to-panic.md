## To `panic!` or Not to `panic!`

## Entrar em `panic!` ou Não Entrar em `panic!`

So how do you decide when you should `panic!` and when you should return
`Result`? When code panics, there’s no way to recover. You could call `panic!`
for any error situation, whether there’s a possible way to recover or not, but
then you’re making the decision on behalf of the code calling your code that a
situation is unrecoverable. When you choose to return a `Result` value, you
give the calling code options rather than making the decision for it. The
calling code could choose to attempt to recover in a way that’s appropriate for
its situation, or it could decide that an `Err` value in this case is
unrecoverable, so it can call `panic!` and turn your recoverable error into an
unrecoverable one. Therefore, returning `Result` is a good default choice when
you’re defining a function that might fail.

Então como você decide quando entrar em `panic!` e quando você deveria retornar
um `Result`? Quando o código entra em pânico, não há maneira de se recuperar. Você
poderia chamar `panic!` para qualquer situação de erro, tendo uma maneira de se recuperar
ou não, mas então você estaria decidindo no lugar do código que chama seu código
que a situação é irrecuperável. Quando você decide retornar um valor de `Result`,
você lhe dá opções em vez de tomar a decisão por ele. O código
que chama seu código pode tentar se recuperar de uma maneira que é apropriada para
a situação, ou ele pode decidir que um valor de `Err` nesse caso é irrecuperável,
chamando `panic!` e transformando seu erro recuperável em um irrecuperável.
Portanto, retornar `Result` é uma boa escolha padrão quando você está definindo
uma função que pode falhar.

In a few situations it’s more appropriate to write code that panics instead of
returning a `Result`, but they are less common. Let’s explore why it’s
appropriate to panic in examples, prototype code, and tests; then in situations
where you as a human can know a method won’t fail that the compiler can’t
reason about; and conclude with some general guidelines on how to decide
whether to panic in library code.

Em algumas situações é mais apropriado escrever código que entra em pânico ao invés
de retornar um `Result`, mas eles são menos comuns. Vamos explorar porque é apropriado
entrar em pânico em alguns exemplos, protótipos de código e testes; depois situações
em que você como humano pode saber que um método não vai falhar que o compilador não
tem como saber; e concluir com algumas diretrizes sobre como decidir entrar ou
não em pânico em código de biblioteca.

### Examples, Prototype Code, and Tests Are All Places it’s Perfectly Fine to Panic

### Exemplos, Protótipos, e Testes São Todos Lugares em que É Perfeitamente Ok Entrar em Pânico

When you’re writing an example to illustrate some concept, having robust error
handling code in the example as well can make the example less clear. In
examples, it’s understood that a call to a method like `unwrap` that could
`panic!` is meant as a placeholder for the way that you’d want your application
to handle errors, which can differ based on what the rest of your code is doing.

Quando você está escrevendo um exemplo para ilustrar algum conceito, ter código
de tratamento de erro robusto junto do exemplo pode torná-lo menos claro. Em exemplos,
é compreensível que uma chamada a um método como `unwrap` que poderia chamar `panic!`
apenas substitua a maneira como você trataria erros na sua aplicação,
que pode ser diferente baseado no que o resto do seu código está fazendo.

Similarly, the `unwrap` and `expect` methods are very handy when prototyping,
before you’re ready to decide how to handle errors. They leave clear markers in
your code for when you’re ready to make your program more robust.

De forma semelhante, os métodos `unwrap` e `expect` são bem úteis ao fazer
protótipos, antes de você estar pronto para decidir como tratar erros. Eles deixam
marcadores claros no seu código para quando você estiver pronto para tornar
seu programa mais robusto.

If a method call fails in a test, we’d want the whole test to fail, even if
that method isn’t the functionality under test. Because `panic!` is how a test
is marked as a failure, calling `unwrap` or `expect` is exactly what should
happen.

Se uma chamada de método falha em um teste, queremos que o teste inteiro falhe,
mesmo se esse método não é a funcionalidade sendo testada. Como `panic!` é o modo
que um teste é marcado como falha, chamar `unwrap` ou `expect` é exatamente o que
deveria acontecer.

### Cases When You Have More Information Than the Compiler

### Casos em que Você Tem Mais Informação Que o Compilador

It would also be appropriate to call `unwrap` when you have some other logic
that ensures the `Result` will have an `Ok` value, but the logic isn’t
something the compiler understands. You’ll still have a `Result` value that you
need to handle: whatever operation you’re calling still has the possibility of
failing in general, even though it’s logically impossible in your particular
situation. If you can ensure by manually inspecting the code that you’ll never
have an `Err` variant, it’s perfectly acceptable to call `unwrap`. Here’s an
example:

Seria também apropriado chamar `unwrap` quando você tem outra lógica que
assegura que o `Result` vai ter um valor `Ok`, mas essa lógica não é algo
que o compilador entenda. Você ainda vai ter um valor de `Result` que precisa 
lidar: seja qual for a operação que você está chamando, ela ainda tem uma possibilidade
de falhar em geral, mesmo que seja logicamente impossível que isso ocorra nessa 
situação particular. Se você consegue assegurar ao inspecionar manualmente o código que
você nunca tera uma variante `Err`, é perfeitamente aceitável chamar `unwrap`.
Aqui temos um exemplo:

```rust
use std::net::IpAddr;

let home = "127.0.0.1".parse::<IpAddr>().unwrap();
```

We’re creating an `IpAddr` instance by parsing a hardcoded string. We can see
that `127.0.0.1` is a valid IP address, so it’s acceptable to use `unwrap`
here. However, having a hardcoded, valid string doesn’t change the return type
of the `parse` method: we still get a `Result` value, and the compiler will
still make us handle the `Result` as if the `Err` variant is still a
possibility because the compiler isn’t smart enough to see that this string is
always a valid IP address. If the IP address string came from a user rather
than being hardcoded into the program, and therefore *did* have a possibility
of failure, we’d definitely want to handle the `Result` in a more robust way
instead.

Nós estamos criando uma instância `IpAddr` ao analisar uma string *hardcoded*. Nós
podemos ver que `127.0.0.1` é um endereço de IP válido, então é aceitável usar 
`unwrap` aqui. No entanto, ter uma string válida *hardcoded* não muda o tipo retornado
pelo método `parse`: ainda teremos um valor de `Result`, e o compilador ainda 
vai nos fazer tratar o `Result` como se a variante `Err` fosse uma
possibilidade, porque o compilador não é inteligente o bastante para ver que essa string
é sempre um endereço IP válido. Se a string de endereço IP viesse de um usuário ao invés
de ser *hardcoded* no programa, e portanto, de fato tivesse uma possibilidade de falha, nós
definitivamente iríamos querer tratar o `Result` de uma forma mais robusta.

### Guidelines for Error Handling

### Diretrizes para Tratamento de Erro

It’s advisable to have your code `panic!` when it’s possible that your code
could end up in a bad state. In this context, bad state is when some
assumption, guarantee, contract, or invariant has been broken, such as when
invalid values, contradictory values, or missing values are passed to your
code—plus one or more of the following:

É aconselhável fazer que seu código entre em `panic!` quando é possível que
ele entre em um estado mau. Nesse contexto, estado mau é quando
alguma hipótese, garantia, contrato ou invariante foi quebrada, tal como
valores inválidos, valores contraditórios, ou valores faltando que são passados
a seu código - além de um ou mais dos seguintes:

* The bad state is not something that’s *expected* to happen occasionally.
* Your code after this point needs to rely on not being in this bad state.
* There’s not a good way to encode this information in the types you use.

* O mau estado não é algo que é *esperado* que aconteça ocasionalmente.
* Seu código após certo ponto precisa confiar que ele não está nesse mau estado.
* Não há uma forma boa de codificar essa informação nos tipos que você usa.

If someone calls your code and passes in values that don’t make sense, the best
choice might be to `panic!` and alert the person using your library to the bug
in their code so they can fix it during development. Similarly, `panic!` is
often appropriate if you’re calling external code that is out of your control,
and it returns an invalid state that you have no way of fixing.

Se alguém chama seu código e passa valores que não fazem sentido, a melhor escolha
talvez seja entrar em `panic!` e alertar a pessoa usando sua biblioteca do bug no
código dela para que ela possa consertá-la durante o desenvolvimento. Similarmente,
`panic!` é em geral apropriado se você está chamando código externo que está fora
do seu controle e ele retorna um estado inválido que você não tem como consertar.

When a bad state is reached, but it’s expected to happen no matter how well you
write your code, it’s still more appropriate to return a `Result` rather than
making a `panic!` call. Examples of this include a parser being given malformed
data or an HTTP request returning a status that indicates you have hit a rate
limit. In these cases, you should indicate that failure is an expected
possibility by returning a `Result` to propagate these bad states upwards so
the calling code can decide how to handle the problem. To `panic!` wouldn’t be
the best way to handle these cases.

Quando se chega a um estado mau, mas isso é esperado que aconteça não importa
quão bem você escreva seu código, ainda é mais apropriado retornar um `Result`
a fazer uma chamada a `panic!`. Um exemplo disso é um *parser* recebendo dados
malformados ou uma requisição HTTP retornando um status que indique que você atingiu
um limite de taxa. Nesses casos, você deveria indicar que falha é uma possibilidade
esperada ao retornar um `Result` para propagar esses estados ruins para cima,
de forma que o código que chamou seu código pode decidir como tratar o problema.
Entrar em `panic!` não seria a melhor maneira de lidar com esses casos.

When your code performs operations on values, your code should verify the
values are valid first, and `panic!` if the values aren’t valid. This is mostly
for safety reasons: attempting to operate on invalid data can expose your code
to vulnerabilities. This is the main reason the standard library will `panic!`
if you attempt an out-of-bounds memory access: trying to access memory that
doesn’t belong to the current data structure is a common security problem.
Functions often have *contracts*: their behavior is only guaranteed if the
inputs meet particular requirements. Panicking when the contract is violated
makes sense because a contract violation always indicates a caller-side bug,
and it’s not a kind of error you want the calling code to have to explicitly
handle. In fact, there’s no reasonable way for calling code to recover: the
calling *programmers* need to fix the code. Contracts for a function,
especially when a violation will cause a panic, should be explained in the API
documentation for the function.

Quando seu código realiza operações em valores, ele deveria verificar que os valores
são válidos primeiro, e entrar em `panic!` caso não o sejam. Isso é 
em boa parte por razões de segurança: tentar operar em dados inválidos pode expor seu
código a vulnerabilidades. Essa é a principal razão para a biblioteca padrão entrar em
`panic!` se você tentar um acesso de memória fora dos limites: tentar acessar memória 
que não pertence à estrutura de dados atual é um problema de segurança comum. Funções 
frequentemente tem *contratos*: seu comportamento somente é garantido se os inputs cumprem
requerimentos específicos. Entrar em pânico quando o contrato é violado faz sentido 
porque uma violação de contrato sempre indica um bug da parte do chamador, e não é o tipo 
de erro que você quer que seja tratado explicitamente. De fato, 
não há nenhuma maneira razoável para o código chamador se recuperar: os *programadores*
que precisam consertar o código. Contratos para uma função, especialmente quando uma
violação leva a pânico, devem ser explicados na documentação da API da função.

However, having lots of error checks in all of your functions would be verbose
and annoying. Fortunately, you can use Rust’s type system (and thus the type
checking the compiler does) to do many of the checks for you. If your function
has a particular type as a parameter, you can proceed with your code’s logic
knowing that the compiler has already ensured you have a valid value. For
example, if you have a type rather than an `Option`, your program expects to
have *something* rather than *nothing*. Your code then doesn’t have to handle
two cases for the `Some` and `None` variants: it will only have one case for
definitely having a value. Code trying to pass nothing to your function won’t
even compile, so your function doesn’t have to check for that case at runtime.
Another example is using an unsigned integer type like `u32`, which ensures the
parameter is never negative.

No entanto, ter várias checagens de erro em todas suas funções pode ser verboso
e irritante. Felizmente, você pode usar o sistema de tipos do Rust (e portanto a
checagem que o compilador faz) para fazer várias dessas checagens para você. Se
sua função têm um tipo particular como parâmetro, você pode continuar com a lógica
do seu código sabendo que o compilador já assegurou que você tem um valor válido.
Por exemplo, se você tem um tipo ao invés de uma `Option`, seu programa espera
ter *algo* ao invés de *nada*. Seu código não precisa tratar dois casos para
as variantes `Some` e `None`: ele vai somente ter um caso para definitivamente ter
um valor. Um código que tente passar nada para sua função não vai nem compilar,
então sua função não precisa checar esse caso em tempo de execução. Outro exemplo é usar
um tipo de inteiro sem sinal como `u32`, que assegura que o parâmetro nunca é
negativo.

### Creating Custom Types for Validation

### Criando Tipos Customizados para Validação

Let’s take the idea of using Rust’s type system to ensure we have a valid value
one step further and look at creating a custom type for validation. Recall the
guessing game in Chapter 2 where our code asked the user to guess a number
between 1 and 100. We never validated that the user’s guess was between those
numbers before checking it against our secret number; we only validated that
the guess was positive. In this case, the consequences were not very dire: our
output of “Too high” or “Too low” would still be correct. It would be a useful
enhancement to guide the user toward valid guesses and have different behavior
when a user guesses a number that’s out of range versus when a user types, for
example, letters instead.

Vamos dar um passo além na ideia de usar o sistema de tipos de Rust para assegurar que temos
um valor válido um passo adiante e ver como criar um tipo customizado para validação.
Lembre do jogo de adivinhação no Capítulo 2 onde nosso código pedia ao usuário 
para adivinhar um número entre 1 e 100. Nós nunca validamos que o chute do usuário
fosse entre esses números antes de compará-lo com o número secreto; nós somente 
validamos que o chute era positivo. Nesse caso, as consequências não foram tão
drásticas: nosso output de "Muito alto" ou "Muito baixo" ainda estariam corretos. Seria
uma melhoria útil guiar o usuário para chutes válidos, e ter um comportamento distinto
quando um usuário chuta um número fora do limite e quando um usuário ao invés digita, 
por exemplo, letras.

One way to do this would be to parse the guess as an `i32` instead of only a
`u32` to allow potentially negative numbers, and then add a check for the
number being in range, like so:

Uma maneira de fazer isso seria interpretar o chute como um `i32` ao invés de
somente um `u32` para permitir números potenciamente negativos, e então adicionar
uma checagem se o número está dentro dos limites, conforme a seguir:

```rust,ignore
loop {
    // snip

    let guess: i32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    if guess < 1 || guess > 100 {
        println!("The secret number will be between 1 and 100.");
        continue;
    }

    match guess.cmp(&secret_number) {
    // snip
}
```

```rust,ignore
loop {
    // snip

    let palpite: i32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    if palpite < 1 || palpite > 100 {
        println!("O número secreto vai estar entre 1 e 100.");
        continue;
    }

    match guess.cmp(&numero_secreto) {
    // snip
}
```

The `if` expression checks whether our value is out of range, tells the user
about the problem, and calls `continue` to start the next iteration of the loop
and ask for another guess. After the `if` expression, we can proceed with the
comparisons between `guess` and the secret number knowing that `guess` is
between 1 and 100.

A expressão `if` checa se nosso valor está fora dos limites, informa o usuário
sobre o problema, e chama `continue` para começar a próxima iteração do loop
e pedir por outro chute. Depois da expressão `if` podemos proceder com as 
comparações entre `guess` e o número secreto sabendo que `guess` está 
entre 1 e 100.

However, this is not an ideal solution: if it was absolutely critical that the
program only operated on values between 1 and 100, and it had many functions
with this requirement, it would be tedious (and potentially impact performance)
to have a check like this in every function.

No entanto, essa não é a solução ideal: se fosse absolutamente crítico que o
programa somente operasse em valores entre 1 e 100, e ele tivesse várias funções
com esse requisito, seria tedioso (e potencialmente impactante na performance)
ter uma checagem dessa em cada função.

Instead, we can make a new type and put the validations in a function to create
an instance of the type rather than repeating the validations everywhere. That
way, it’s safe for functions to use the new type in their signatures and
confidently use the values they receive. Listing 9-9 shows one way to define a
`Guess` type that will only create an instance of `Guess` if the `new` function
receives a value between 1 and 100:

Ao invés disso, podemos fazer um novo tipo e colocar as validações em uma função
para criar uma instância do tipo ao invés de repetir as validações em todo lugar.
Dessa maneira, é seguro para funções usarem o novo tipo nas suas assinaturas e 
confidentemente usar os valores que recebem. A Listagem 9-9  mostra uma maneira de 
definir um tipo `Guess` que vai somente criar uma instância de `Guess` se a função
`new` receber um valor entre 1 e 100:

```rust
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

    pub fn value(&self) -> u32 {
        self.value
    }
}
```

```rust
pub struct Palpite {
    valor: u32,
}

impl Palpite {
    pub fn new(valor: u32) -> Palpite {
        if valor < 1 || valor > 100 {
            panic!("Valor de chute deve ser entre 1 e 100, recebi {}.", valor);
        }

        Palpite {
            valor
        }
    }

    pub fn valor(&self) -> u32 {
        self.valor
    }
}
```

<span class="caption">Listing 9-9: A `Guess` type that will only continue with
values between 1 and 100</span>

<span class="caption">Listagem 9-9: Um tipo `Guess` que somente funciona com valores
entre 1 e 100</span>

First, we define a struct named `Guess` that has a field named `value` that
holds a `u32`. This is where the number will be stored.

Primeiro, definimos uma struct chamada `Guess` que tem um campo chamado `value`
que guarda um `u32`. Isso é onde o número vai ser guardado.

Then we implement an associated function named `new` on `Guess` that creates
instances of `Guess` values. The `new` function is defined to have one
parameter named `value` of type `u32` and to return a `Guess`. The code in the
body of the `new` function tests `value` to make sure it’s between 1 and 100.
If `value` doesn’t pass this test, we make a `panic!` call, which will alert
the programmer who is writing the calling code that they have a bug they need
to fix, because creating a `Guess` with a `value` outside this range would
violate the contract that `Guess::new` is relying on. The conditions in which
`Guess::new` might panic should be discussed in its public-facing API
documentation; we’ll cover documentation conventions indicating the possibility
of a `panic!` in the API documentation that you create in Chapter 14. If
`value` does pass the test, we create a new `Guess` with its `value` field set
to the `value` parameter and return the `Guess`.

Então nós implementamos uma função associada chamada `new` em `Palpite` que cria
instâncias de valores `Palpite`. A função `new` é definida a ter um parâmetro
chamado `valor` de tipo `u32` e retornar um `Palpite`. O código no corpo da função
`new` testa para ter certeza que `valor` está entre 1 e 100. Se `valor` não passa
nesse teste, fazemos uma chamada a `panic!`, que vai alertar ao programador que
está escrevendo o código chamando a função que ele tem um bug que precisa ser 
corrigido, porque criar um `Palpite` com um `valor` fora desses limites violaria
o contrato em que `Palpite::new` se baseia. As condições em que `Palpite::new` pode 
entrar em pânico devem ser discutidas na sua documentação da API voltada ao público;
no capítulo 14 nós cobriremos convenções de documentação indicando a possibilidade de um `panic!`
na documentação de API. Se `valor` de fato passa no
teste, criamos um novo `Palpite` com o campo `valor` preenchido com o parâmetro
`valor` e retornamos o `Palpite`.

Next, we implement a method named `value` that borrows `self`, doesn’t have any
other parameters, and returns a `u32`. This is a kind of method sometimes
called a *getter*, because its purpose is to get some data from its fields and
return it. This public method is necessary because the `value` field of the
`Guess` struct is private. It’s important that the `value` field is private so
code using the `Guess` struct is not allowed to set `value` directly: code
outside the module *must* use the `Guess::new` function to create an instance
of `Guess`, which ensures there’s no way for a `Guess` to have a `value` that
hasn’t been checked by the conditions in the `Guess::new` function.

Em seguida, implementamos um método chamado `valor` que pega `self` emprestado, não
tem nenhum outro parâmetro, e retorna um `u32`. Esse é o tipo de método às vezes
chamado de *getter*, pois seu propósito é pegar um dado de um dos campos e o retornar.
Esse método público é necessário porque o campo `valor` da struct `Palpite` é privado.
É importante que o campo `valor` seja privado para que código usando a struct `Palpite`
não tenha permissão de definir o valor de `valor` diretamente: código de fora do módulo
*deve* usar a função `Palpite::new` para criar uma instância de `Palpite`, o que certifica
que não há maneira de um `Palpite` ter um `valor` que não foi checado pelas condições
definidas na função `Palpite::new`.

A function that has a parameter or returns only numbers between 1 and 100 could
then declare in its signature that it takes or returns a `Guess` rather than a
`u32` and wouldn’t need to do any additional checks in its body.

Uma função que tem um parâmetro ou retorna somente números entre 1 e 100 pode
então declarar na sua assinatura que ela recebe ou retorna um `Palpite` ao invés
de um `u32` e não precisaria fazer nenhuma checagem adicional no seu corpo.

## Summary

## Resumo

Rust’s error handling features are designed to help you write more robust code.
The `panic!` macro signals that your program is in a state it can’t handle and
lets you tell the process to stop instead of trying to proceed with invalid or
incorrect values. The `Result` enum uses Rust’s type system to indicate that
operations might fail in a way that your code could recover from. You can use
`Result` to tell code that calls your code that it needs to handle potential
success or failure as well. Using `panic!` and `Result` in the appropriate
situations will make your code more reliable in the face of inevitable problems.

As ferramentas de tratamento de erros de Rust são feitas para te ajudar a escrever
código mais robusto. A macro `panic!` sinaliza que seu programa está num estado que
não consegue lidar e deixa você parar o processo ao invés de tentar prosseguir com
valores inválidos ou incorretos. O enum `Result` usa o sistema de tipos de Rust para 
indicar que operações podem falhar de uma maneira que seu código pode se recuperar. 
Você pode usar `Result` para dizer ao código que chama seu código que ele precisa
tratar potenciais sucessos ou falhas também. Usar `panic!` e `Result` nas situações
apropriadas fará seu código mais confiável em face aos problemas inevitáveis.

Now that you’ve seen useful ways that the standard library uses generics with
the `Option` and `Result` enums, we’ll talk about how generics work and how you
can use them in your code in the next chapter.

Agora que você viu as maneiras úteis em que a biblioteca padrão usa genéricos com
os enums `Option` e `Result`, nós falaremos como genéricos funcionam e como você
pode usá-los em seu código no próximo capítulo.



