## Recoverable Errors with `Result`
## Erros recuperáveis com `Result`

Most errors aren’t serious enough to require the program to stop entirely.
Sometimes, when a function fails, it’s for a reason that we can easily
interpret and respond to. For example, if we try to open a file and that
operation fails because the file doesn’t exist, we might want to create the
file instead of terminating the process.

A maior parte dos erros não são sérios o suficiente para precisar que o
programa pare totalmente. Às vezes, quando uma função falha, é por uma
razão que nós podemos fácilmente interpretar e responder. Por exemplo,
se tentamos abrir um arquivo e essa operação falhar porque o arquivo não
existe, nós podemos querer criar o arquivo ao invés de terminar o processo.

Recall in Chapter 2 in the on “[Handling Potential Failure with the `Result`
Type][handle_failure]<!-- ignore -->” section that the `Result` enum is defined
as having two variants, `Ok` and `Err`, as follows:

Lembre do Capítulo 2, na seção “[Tratando Potenciais Falhas com o Tipo Result][handle_failure]
<!-- ignore -->”  que o enum `Result` enum é definido como tendo duas variantes,
`Ok` e `Err`, como mostrado:

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-the-result-type

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

The `T` and `E` are generic type parameters: we’ll discuss generics in more
detail in Chapter 10. What you need to know right now is that `T` represents
the type of the value that will be returned in a success case within the `Ok`
variant, and `E` represents the type of the error that will be returned in a
failure case within the `Err` variant. Because `Result` has these generic type
parameters, we can use the `Result` type and the functions that the standard
library has defined on it in many different situations where the successful
value and error value we want to return may differ.

O `T` e `E` são parâmetros de tipos genéricos: nós os discutiremos em mais
detalhe no Capítulo 10. O que você precisa saber agora é que `T` representa
o tipo do valor que vai ser retornado dentro da variante `Ok` em caso de sucesso,
e `E` representa o tipo de erro que será retornado dentro da variante `Err`
em caso de falha. Por `Result` ter esses parâmetros de tipo genéricos, nós
podemos usar o tipo `Result` e as funções qua a biblioteca padrão definiu sobre
ele em diversas situações em que o valor de sucesso e o valor de erro que 
queremos retornar possam divergir.

Let’s call a function that returns a `Result` value because the function could
fail: in Listing 9-3 we try to open a file:

Vamos chamar uma função que retorna um valor `Result` porque a função poderia
falhar: na Lista 9-3 tentamos abrir um arquivo:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```

<span class="caption">Listing 9-3: Opening a file</span>
<span class="caption">Lista 9-3: Abrindo um arquivo</span>

How do we know `File::open` returns a `Result`? We could look at the standard
library API documentation, or we could ask the compiler! If we give `f` a type
annotation of a type that we know the return type of the function is *not* and
then we try to compile the code, the compiler will tell us that the types don’t
match. The error message will then tell us what the type of `f` *is*. Let’s try
it: we know that the return type of `File::open` isn’t of type `u32`, so let’s
change the `let f` statement to this:

Como que sabemos que `File::open` retorna um `Result`? Poderíamos olhar na documentação
da API da biblioteca padrão, ou poderíamos perguntar para o compilador! Se damos à `f`
uma anotação de tipo que sabemos *não* ser o tipo retornado pela função e tentamos
compilar o código, o compilador nos dirá que os tipos não se casam. A mensagem de erro
vai então nos dizer qual é *de fato* o tipo de `f`. Vamos tentar isso: nós sabemos que
o tipo retornado por `File::open` não é `u32`, então vamos mudar a declaração
`let f` para isso:

```rust,ignore
let f: u32 = File::open("hello.txt");
```

Attempting to compile now gives us the following output:

Tentar compilar agora nos dá a seguinte saída:

```text
error[E0308]: mismatched types
 --> src/main.rs:4:18
  |
4 |     let f: u32 = File::open("hello.txt");
  |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
`std::result::Result`
  |
  = note: expected type `u32`
  = note:    found type `std::result::Result<std::fs::File, std::io::Error>`
```

This tells us the return type of the `File::open` function is a `Result<T, E>`.
The generic parameter `T` has been filled in here with the type of the success
value, `std::fs::File`, which is a file handle. The type of `E` used in the
error value is `std::io::Error`.

Isso nos diz que o valor de retorno de `File::open` é um `Result<T, E>`.
O parâmetro genérico `T` foi preenchido aqui com o tipo do valor de sucesso,
`std::fs::File`, que é um *handle* de arquivo. O tipo de `E` usado no valor
de erro é `std::io::Error`.

This return type means the call to `File::open` might succeed and return to us
a file handle that we can read from or write to. The function call also might
fail: for example, the file might not exist or we might not have permission to
access the file. The `File::open` function needs to have a way to tell us
whether it succeeded or failed and at the same time give us either the file
handle or error information. This information is exactly what the `Result` enum
conveys.

Esse tipo de retorno significa que a chamada a `File::open` pode dar certo
e retornar para nós um *handle* de arquivo que podemos usar pra ler ou escrever
nele. Essa chamada de função pode também falhar: por exemplo, o arquivo pode não
existir ou talvez não tenhamos permissão para acessar o arquivo. A função `File::open`
precisa ter uma maneira de nos dizer se ela teve sucesso ou falhou e ao mesmo tempo
nos dar ou o *handle* de arquivo ou informação sobre o erro. Essa informação é 
exatamente o que o enum `Result` comunica.

In the case where `File::open` succeeds, the value we will have in the variable
`f` will be an instance of `Ok` that contains a file handle. In the case where
it fails, the value in `f` will be an instance of `Err` that contains more
information about the kind of error that happened.

No caso em que `File::open` tem sucesso, o valor na variável `f` será uma instância
de `Ok` que contém um *handle* de arquivo. No caso em que ela falha, o valor em `f`
será ua instância de `Err` que contém mais informação sobre o tipo de erro que
aconteceu.

We need to add to the code in Listing 9-3 to take different actions depending
on the value `File::open` returned. Listing 9-4 shows one way to handle the
`Result` using a basic tool: the `match` expression that we discussed in
Chapter 6.

Devemos fazer com que o código na Lista 9-3 faça diferentes ações dependendo
do valor retornado por `File::open`. A Lista 9-4 mostra uma maneira de lidar
com o `Result` usando uma ferramenta básica: a expressão `match` que discutimos
no Capítulo 6.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
}
```
<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Houve um problema ao abrir o arquivo: {:?}", error)
        },
    };
}
```

<span class="caption">Listing 9-4: Using a `match` expression to handle the
`Result` variants we might have</span>

<span class="caption">Lista 9-4: Usando uma expressão `match` para tratar as
variantes de `Result` que podemos encontrar</span>

Note that, like the `Option` enum, the `Result` enum and its variants have been
imported in the prelude, so we don’t need to specify `Result::` before the `Ok`
and `Err` variants in the `match` arms.

Note que, como no enum `Option`, o enum `Result` e suas variantes foram importadas
no prelúdio, então não precisamos especificar `Result::` antes das variantes `Ok` 
e `Err` nas linhas de `match`.

Here we tell Rust that when the result is `Ok`, return the inner `file` value
out of the `Ok` variant, and we then assign that file handle value to the
variable `f`. After the `match`, we can then use the file handle for reading or
writing.

Aqui dizemos ao Rust que quando o resultado é `Ok` ele deve retornar o valor
interno `file` de dentro da variante `Ok` e nós então podemos atribuir este
valor de *handle* de arquivo à variável `f`. Depois do `match`, nós podemos então
usar o *handle* de arquivo para ler ou escrever.

The other arm of the `match` handles the case where we get an `Err` value from
`File::open`. In this example, we’ve chosen to call the `panic!` macro. If
there’s no file named *hello.txt* in our current directory and we run this
code, we’ll see the following output from the `panic!` macro:

A outra linha de `match` trata do caso em que recebemos um valor de `Err` de
`File::open`. Nesse exemplo, nós escolhemos chamar a macro `panic!`. Se não
há nenhum arquivo chamado *hello.txt* no diretório atual e rodarmos esse código,
veremos a seguinte saída da macro `panic!`:

```text
thread 'main' panicked at 'There was a problem opening the file: Error { repr:
Os { code: 2, message: "No such file or directory" } }', src/main.rs:9:12
```

As usual, this output tells us exactly what has gone wrong.

Como sempre, essa saída nos diz exatamente o que aconteceu de errado.

### Matching on Different Errors

### Dando `match` em Diferentes Erros

The code in Listing 9-4 will `panic!` no matter the reason that `File::open`
failed. What we want to do instead is take different actions for different
failure reasons: if `File::open` failed because the file doesn’t exist, we want
to create the file and return the handle to the new file. If `File::open`
failed for any other reason, for example because we didn’t have permission to
open the file, we still want the code to `panic!` in the same way as it did in
Listing 9-4. Look at Listing 9-5, which adds another arm to the `match`:

O código na Lista 9-4 chamará `panic!` não importa a razão pra `File::open`
ter falhado. O que queremos fazer ao invés é tomar diferentes ações para diferentes
motivos de falha: se `File::open` falhou porque o arquivo não existe, nós 
queremos criar um arquivo e retornar o *handle* para o novo arquivo. Se `File::open`
falhou por qualquer outra razão, por exemplo porque não temos a permissão para
abrir o arquivo, nós ainda queremos chamar `panic!` da mesma maneira que fizemos
na Lista 9-4. Veja a Lista 9-5, que adiciona outra linha ao `match`:

<span class="filename">Filename: src/main.rs</span>

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!(
                        "Tried to create file but there was a problem: {:?}",
                        e
                    )
                },
            }
        },
        Err(error) => {
            panic!(
                "There was a problem opening the file: {:?}",
                error
            )
        },
    };
}
```

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!(
                        "Tentou criar um arquivo e houve um problema: {:?}",
                        e
                    )
                },
            }
        },
        Err(error) => {
            panic!(
                "Houve um problema ao abrir o arquivo: {:?}",
                error
            )
        },
    };
}
```
<span class="caption">Listing 9-5: Handling different kinds of errors in
different ways</span>


<span class="caption">Lista 9-5: Tratando diferentes tipos de erros de diversas
maneiras</span>

The type of the value that `File::open` returns inside the `Err` variant is
`io::Error`, which is a struct provided by the standard library. This struct
has a method `kind` that we can call to get an `io::ErrorKind` value.
`io::ErrorKind` is an enum provided by the standard library that has variants
representing the different kinds of errors that might result from an `io`
operation. The variant we want to use is `ErrorKind::NotFound`, which indicates
the file we’re trying to open doesn’t exist yet.

O tipo do valor que `File::open` retorna dentro da variante `Err` é `io::Error`,
que é uma struct fornecida pela biblioteca padrão. Essa struct tem o método
`kind` que podemos chamar para receber um valor de `io::ErrorKind`. `io::ErrorKind`
é um enum fornecido pela biblioteca padrão que tem variantes representanto diversos
tipos de erros que podem ocorrer em uma operação de `io`. A variante que queremos 
usar é `ErrorKind::NotFound`, que indica que o arquivo que queremos abrir não existe
ainda.

The condition `if error.kind() == ErrorKind::NotFound` is called a *match
guard*: it’s an extra condition on a `match` arm that further refines the arm’s
pattern. This condition must be true for that arm’s code to be run; otherwise,
the pattern matching will move on to consider the next arm in the `match`. The
`ref` in the pattern is needed so `error` is not moved into the guard condition
but is merely referenced by it. The reason `ref` is used to take a reference in
a pattern instead of `&` will be covered in detail in Chapter 18. In short, in
the context of a pattern, `&` matches a reference and gives us its value, but
`ref` matches a value and gives us a reference to it.

A condição `if error.kind() == ErrorKind::NotFound` é chamada de um *match guard*:
é uma condição extra dentro de uma linha de `match` que posteriormente refina
o padrão da linha. Essa condição deve ser verdadeira para o código da linha ser
executado; caso contrário a análise de padrões vai continuar considerando as 
próximas linhas no `match`. O `ref` no padrão é necessário para que o `error`
não seja movido para a condição do *guard*, mas meramente referenciado por ele.
A razão de `ref` ser utilizado ao invés de `&` para pegar uma referência vai ser
discutida em detalhe no Capítulo 18. Resumindo, no contexto de um padrão, `&` 
corresponde uma referência e nos dá seu valor, enquanto `ref` corresponde um valor
e nos dá uma referência a ele.

The condition we want to check in the match guard is whether the value returned
by `error.kind()` is the `NotFound` variant of the `ErrorKind` enum. If it is,
we try to create the file with `File::create`. However, because `File::create`
could also fail, we need to add an inner `match` statement as well. When the
file can’t be opened, a different error message will be printed. The last arm
of the outer `match` stays the same so the program panics on any error besides
the missing file error.

A condição que queremos checar no *match guard* é se o valor retornado pelo
`error.kind()` é a variante `NotFound` do enum `ErrorKind`. Se é, queremos
tentar criar um arquivo com `File::create`. No entanto, como `File::create`
pode também falhar, precisamos adicionar um `match` interno também. Quando
o arquivo não pode ser aberto, outro tipo de mensagem de erro será mostrada.
A última linha do `match` externo continua a mesma de forma que o programa
entre em pânico pra qualquer erro além do de arquivo ausente.

### Shortcuts for Panic on Error: `unwrap` and `expect`
### Atalhos para Pânico em erro: `unwrap` e `expect`

Using `match` works well enough, but it can be a bit verbose and doesn’t always
communicate intent well. The `Result<T, E>` type has many helper methods
defined on it to do various tasks. One of those methods, called `unwrap`, is a
shortcut method that is implemented just like the `match` statement we wrote in
Listing 9-4. If the `Result` value is the `Ok` variant, `unwrap` will return
the value inside the `Ok`. If the `Result` is the `Err` variant, `unwrap` will
call the `panic!` macro for us. Here is an example of `unwrap` in action:

Usar `match` funciona bem o suficiente, mas pode ser um pouco verboso e nem
sempre comunica a intenção bem. O tipo `Result<T, E>` tem vários métodos 
auxiliares definidos para fazer diversas tarefas. Um desses métodos, chamado
`unwrap`, é um método de atalho que é implementado justamente como o `match` que
escrevemos na Lista 9-4. Se o valor de `Result` for da variante `Ok`, `unwrap`
vai retornar o valor dentro de `Ok`. Se o `Result` for da variante `Err`, `unwrap`
vai chamar a macro `panic!`. Aqui um exemplo de `unwrap` em ação:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

If we run this code without a *hello.txt* file, we’ll see an error message from
the `panic!` call that the `unwrap` method makes:

Se rodarmos esse código sem um arquivo *hello.txt*, veremos uma mensagem de err
da chamada de `panic!` que o método `unwrap` faz:

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
repr: Os { code: 2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```

Another method, `expect`, which is similar to `unwrap`, lets us also choose the
`panic!` error message. Using `expect` instead of `unwrap` and providing good
error messages can convey your intent and make tracking down the source of a
panic easier. The syntax of `expect` looks like this:

Outro método, `expect`, que é semelhante a `unwrap`, nos deixa também escolher
a mensagem de erro do `panic!`. Usar `expect` ao invés de `unwrap` e fornecer
boas mensagens de erros podem transmitir sua intenção e tornar a procura pela
fonte de pânico mais fácil. A sintaxe de `expect` se parece com isso:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Falhou ao abrir hello.txt");
}
```

We use `expect` in the same way as `unwrap`: to return the file handle or call
the `panic!` macro. The error message used by `expect` in its call to `panic!`
will be the parameter that we pass to `expect`, rather than the default
`panic!` message that `unwrap` uses. Here’s what it looks like:

Nós usamos `expect` da mesma maneira que `unwrap`: para retornar o *handle* de arquivo
ou chamar a macro de `panic!`. A mensagem de erro usada por `expect` na sua chamada
de `panic!` será o parâmtero que passamos para `expect` ao invés da mensagem padrão
que o `unwrap` usa. Aqui é como ela se parece:

```text
thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```

```text
thread 'main' panicked at 'Falhou ao abrir hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }',
/stable-dist-rustc/build/src/libcore/result.rs:868
```
Because this error message starts with the text we specified, `Failed to open
hello.txt`, it will be easier to find where in the code this error message is
coming from. If we use `unwrap` in multiple places, it can take more time to
figure out exactly which `unwrap` is causing the panic because all `unwrap`
calls that panic print the same message.

Como essa mensagem de erro começa com o texto que especificamos, `Falhou ao abrir
hello.txt`, será mais fácil de encontrar aonde no código essa mensagem de erro
se originou. Se usamos `unwrap` em diversos lugares, pode tomar mais tempo encontrar
exatamente qual dos `unwrap` está causando o pânico, dado que todas as chamadas
a `unwrap` chamam o print de pânico com a mesma mensagem


### Propagating Errors
### Propagando Erros

When you’re writing a function whose implementation calls something that might
fail, instead of handling the error within this function, you can return the
error to the calling code so that it can decide what to do. This is known as
*propagating* the error and gives more control to the calling code where there
might be more information or logic that dictates how the error should be
handled than what you have available in the context of your code.

Quando você está escrevendo uma função cuja implementação chama algo que pode 
falhar, ao invés de tratar o erro dentro dessa função, você pode retornar o
erro ao código que a chamou de forma que ele possa decidir o que fazer. Isso é
conhecido como *propagar* o erro e dá mais controle ao código que chamou sua
função, que pode ter mais informação ou lógica que dite como o erro deve ser
tratado do que você tem disponível no contexto do seu código.

For example, Listing 9-6 shows a function that reads a username from a file. If
the file doesn’t exist or can’t be read, this function will return those errors
to the code that called this function:

Por exemplo, a Lista 9-6 mostra uma função que lê um nome de usuário de uma rquivo
Se o arquivo não existe ou não pode ser lido, essa função vai retornar esses erros
ao código que chamou essa função:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```
<span class="caption">Listing 9-6: A function that returns errors to the
calling code using `match`</span>


<span class="caption">Lista 9-6: Uma função que retorna erros ao código que a chamou
usando `match`</span>

Let’s look at the return type of the function first: `Result<String,
io::Error>`. This means the function is returning a value of the type
`Result<T, E>` where the generic parameter `T` has been filled in with the
concrete type `String`, and the generic type `E` has been filled in with the
concrete type `io::Error`. If this function succeeds without any problems, the
code that calls this function will receive an `Ok` value that holds a
`String`—the username that this function read from the file. If this function
encounters any problems, the code that calls this function will receive an
`Err` value that holds an instance of `io::Error` that contains more
information about what the problems were. We chose `io::Error` as the return
type of this function because that happens to be the type of the error value
returned from both of the operations we’re calling in this function’s body that
might fail: the `File::open` function and the `read_to_string` method.

Vamos olhar primeiro ao tipo retornado pela função: `Result<String, io::Error>`.
Isso significa que a função está retornando um valor do tipo `Result<T, E>` onde
o parâmetro genérico `T` foi preenchido pelo tipo concreto `String` e o tipo genérico
`E` foi preenchido pelo tipo concreto `io::Error`. Se essa função tem sucesso sem 
nenhum problema, o código que chama essa função vai receber um valor `Ok` que contém
uma `String`- o nome de usuário que essa função leu do arquivo. Se essa função
encontra qualquer problema, o código que a chama receberá um valor de `Err`
que contém uma instância de `io::Error` que contém mais informação
sobre o que foi o problema. Escolhemos `io::Error` como o tipo de retorno
dessa função porque este veio a ser o tipo de valor de erro retornado das 
duas operações que estamos chamando no corpo dessa função que podem falhar:
a função `File::open` e o método `read_to_string`.

The body of the function starts by calling the `File::open` function. Then we
handle the `Result` value returned with a `match` similar to the `match` in
Listing 9-4, only instead of calling `panic!` in the `Err` case, we return
early from this function and pass the error value from `File::open` back to the
calling code as this function’s error value. If `File::open` succeeds, we store
the file handle in the variable `f` and continue.

O corpo da função começa chamando a função `File::open`. Nós então lidamos
com o valor de `Result` retornado com um `match` semelhante ao `match` na Lista 9-4,
só que ao invésde chamar `panic!` no caso de `Err`, retornamos mais cedo dessa função
e passamos o valor de erro de `File::open` de volta ao código que a chamou, como o
valor de erro da nossa função. Se `File::open` tem sucesso, nós guardamos o *handle* de
arquivo na variável `f` e continuamos.

Then we create a new `String` in variable `s` and call the `read_to_string`
method on the file handle in `f` to read the contents of the file into `s`. The
`read_to_string` method also returns a `Result` because it might fail, even
though `File::open` succeeded. So we need another `match` to handle that
`Result`: if `read_to_string` succeeds, then our function has succeeded, and we
return the username from the file that’s now in `s` wrapped in an `Ok`. If
`read_to_string` fails, we return the error value in the same way that we
returned the error value in the `match` that handled the return value of
`File::open`. However, we don’t need to explicitly say `return`, because this
is the last expression in the function.

Então, criamos uma nova `String` na variável `s` e chamaos o método `read_to_string`
no *handle* de arquivo `f` para escrever os conteúdos do arquivo em `s`. O método
`read_to_string` também retorna um `Result` porque ele pode falhar, mesmo que
`File::open` teve sucesso. Então precisamos de outro `match` para tratar esse
`Result`: se `read_to_string` teve sucesso, então nossa função teve sucesso, e nós
retornamos o nome de usuário do arquivo que está agora em `s` embrulhado em um `Ok`.
Se `read_to_string` falhou, retornamos o valor de erro da mesma maneira que retornamos
o valor de erro no `match` que tratou o valor de retorno de `File::open`.
No entanto, não precisamos explicitamente escrever `return`, porque essa já é a 
última expressão na função.

The code that calls this code will then handle getting either an `Ok` value
that contains a username or an `Err` value that contains an `io::Error`. We
don’t know what the calling code will do with those values. If the calling code
gets an `Err` value, it could call `panic!` and crash the program, use a
default username, or look up the username from somewhere other than a file, for
example. We don’t have enough information on what the calling code is actually
trying to do, so we propagate all the success or error information upwards for
it to handle appropriately.

O código que chama esse código vai então tratar receber ou um valor `Ok` que
contém um nome de usuário ou um valor de `Err` que contém um `io::Error`. Nós
não sabemos que o código que chamou nossa função fará com esses valores. Se o 
código que chamou recebe um valor de `Err`, ele poderia chamar `panic!` e causar
um crash, usar um nome de usuário padrão, ou procurar o nome de usuário em outro
lugar que não um arquivo, poe exemplo. Nós não temos informação o suficiente sobre
o que o código que chamou está de fato tentando fazer, então propagamos toda a 
informação de sucesso ou erro para cima para que ele a trate apropriadamente.

This pattern of propagating errors is so common in Rust that Rust provides the
question mark operator `?` to make this easier.

Esse padrão de propagação de erros é tão comum em Rust que Rust disponibiliza
o operador de interrogação `?` para tornar isso mais fácil

#### A Shortcut for Propagating Errors: `?`
#### Um atalho para propagar erros: `?`

Listing 9-7 shows an implementation of `read_username_from_file` that has the
same functionality as it had in Listing 9-6, but this implementation uses the
question mark operator:

A Lista 9-7 mostra uma implementação de `read_username_from_file` que tem a 
mesma funcionalidade que tinha na Lista 9-6, mas esta implementação usa o operador
de interrogação:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```
<span class="caption">Listing 9-7: A function that returns errors to the
calling code using `?`</span>

<span class="caption">Lista 9-7: Uma função que retorna erros para o código
que a chamou usando `?`</span>

The `?` placed after a `Result` value is defined to work in almost the same way
as the `match` expressions we defined to handle the `Result` values in Listing
9-6. If the value of the `Result` is an `Ok`, the value inside the `Ok` will
get returned from this expression and the program will continue. If the value
is an `Err`, the value inside the `Err` will be returned from the whole
function as if we had used the `return` keyword so the error value gets
propagated to the calling code.

O `?` colocado após um valor de `Result` é definido para funcionar em quase
a mesma maneira que as expressões `match` que definimos para tratar o valor
de `Result` na Lista 9-6. Se o valor de `Result` é um `Ok`, o valor dentro do
`Ok` vai ser retornado dessa expressão e o programa vai continuar. Se o valor
é de `Err`, o valor dentro do `Err` vai ser retornado da função inteira como se
tivéssemos usado a *keyword* `return` de modo que o valor de erro é propagado
ao código que chamou a função.

The one difference between the `match` expression from Listing 9-6 and what the
question mark operator does is that when using the question mark operator,
error values go through the `from` function defined in the `From` trait in the
standard library. Many error types implement the `from` function to convert an
error of one type into an error of another type. When used by the question mark
operator, the call to the `from` function converts the error type that the
question mark operator gets into the error type defined in the return type of
the current function that we’re using `?` in. This is useful when parts of a
function might fail for many different reasons, but the function returns one
error type that represents all the ways the function might fail. As long as
each error type implements the `from` function to define how to convert itself
to the returned error type, the question mark operator takes care of the
conversion automatically.

A única diferença entre a expressão `match` da Lista 9-6 e o que o operador
de interrogação faz é que quando usamos o operador de interrogação, os valores
de erro passam pela função `from` definida no *trait* `From` na biblioteca
padrão. Vários tipos de erro implementa a função `from` para converter um
erro de um tipo em um erro de outro tipo. Quando usado pelo operador de 
interrogação, a chamada à função `from` converte o tipo de erro que o
operador recebe no tipo de erro definido no tipo de retorno da função em 
que estamos usando `?`. Isso é útil quando partes de uma função podem falhar
por várias razões diferentes, mas a função retorna um tipo de erro que
representa todas as maneiras que a função pode falhar. Enquanto cada
tipo de erro implementar a função `from` para definir como se converter
ao tipo de erro retornado, o operador de interrogação lida com a conversão
automaticamente.

In the context of Listing 9-7, the `?` at the end of the `File::open` call will
return the value inside an `Ok` to the variable `f`. If an error occurs, `?`
will return early out of the whole function and give any `Err` value to the
calling code. The same thing applies to the `?` at the end of the
`read_to_string` call.

No contexto da Lista 9-7, o `?` no final da chamada de `File::open` vai
retornar o valor dentro do `Ok` à variável `f`. Se um erro ocorrer, `?`
vai retornar mais cedo a função inteira e dar um valor de `Err` ao código
que a chamou. O mesmo se aplica ao `?` ao final da chamada de `read_to_string`.

The `?` eliminates a lot of boilerplate and makes this function’s
implementation simpler. We could even shorten this code further by chaining
method calls immediately after the `?` as shown in Listing 9-8:

O `?` elimina um monte de excesso e torna a implementação dessa 
função mais simple. Poderíamos até encurtar ainda mais esse código 
ao encadear chamadas de método imediatamente depois do `?`, como mostrado
na Lista 9-8:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```
<span class="caption">Listing 9-8: Chaining method calls after the question
mark operator</span>

<span class="caption">Lista 9-8: Encadeando chamadas de método após o operador
de interrogação</span>

We’ve moved the creation of the new `String` in `s` to the beginning of the
function; that part hasn’t changed. Instead of creating a variable `f`, we’ve
chained the call to `read_to_string` directly onto the result of
`File::open("hello.txt")?`. We still have a `?` at the end of the
`read_to_string` call, and we still return an `Ok` value containing the
username in `s` when both `File::open` and `read_to_string` succeed rather than
returning errors. The functionality is again the same as in Listing 9-6 and
Listing 9-7; this is just a different, more ergonomic way to write it.

Nós movemos a criação da nova `String` em `s` para o começo da função;
essa parte não mudou. Ao invés de criar uma variável `f`, nós encadeamos
a chamada para `read_to_string` diretamente ao resultado de 
`File::open("hello.txt")?`. Nós ainda temos um `?` ao fim da chamada a 
`read_to_string`, e ainda retornamos um valor de `Ok` contendo o nome de usuário
em `s` quando tanto `File::open` e `read_to_string` tiveram sucesso ao invés
de retornarem erros. Essa funcionalidade é novamente a mesma da Lista 9-6 e 
Lista 9-7; essa é só uma maneira diferente, mais ergonômica de escrevê-la.

#### `?` Can Only Be Used in Functions That Return Result
#### `?` Somente Pode Ser Usado em Funções Que Retornam Result

The `?` can only be used in functions that have a return type of `Result`,
because it is defined to work in the same way as the `match` expression we
defined in Listing 9-6. The part of the `match` that requires a return type of
`Result` is `return Err(e)`, so the return type of the function must be a
`Result` to be compatible with this `return`.

O `?` só pode ser usado em funções que tem um tipo de retorno de `Result`,
porque está definido a funcionar da mesma maneira que a expressão `match` que
definimos na Lista 9-6. A parte do `match` que requer um tipo de retorno de
`Result` é `return Err(e)`, então o tipo de retorno da função deve ser
um `Result` para ser compatível com esse `return`.

Let’s look at what happens if we use `?` in the `main` function, which you’ll
recall has a return type of `()`:

Vamos ver o que ocorre quando usamos `?` na função `main`, que como vimos tem
um tipo de retorno de `()`:

```rust,ignore
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
```

When we compile this code, we get the following error message:

Quando compilamos esse código recebemos a seguinte mensagem de erro:

```text
error[E0277]: the `?` operator can only be used in a function that returns
`Result` (or another type that implements `std::ops::Try`)
 --> src/main.rs:4:13
  |
4 |     let f = File::open("hello.txt")?;
  |             ------------------------
  |             |
  |             cannot use the `?` operator in a function that returns `()`
  |             in this macro invocation
  |
  = help: the trait `std::ops::Try` is not implemented for `()`
  = note: required by `std::ops::Try::from_error`
```

This error points out that we’re only allowed to use the question mark operator
in a function that returns `Result`. In functions that don’t return `Result`,
when you call other functions that return `Result`, you’ll need to use a
`match` or one of the `Result` methods to handle it instead of using `?` to
potentially propagate the error to the calling code.

Esse erro aponta que só podemos usar o operador de interrogação em funções
que retornam `Result`. EM funções que não retornam `Result`, quando você chama
outras funções que retornam `Result`, você deve usar um `match` ou um dos métodos
de `Result` para tratá-lo ao invés de usar `?` para potencialmente
propagar o erro ao código que a chamou.

Now that we’ve discussed the details of calling `panic!` or returning `Result`,
let’s return to the topic of how to decide which is appropriate to use in which
cases.

Agora que discutimos os detalhes de chamar `panic!` ou retornar `Result`, vamos
retornar ao tópico de como decidir qual é apropriado para utilizar em quais 
casos.
