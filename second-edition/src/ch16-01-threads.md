## Using Threads to Run Code Simultaneously

## Usando Threads para Executar Código Simultaneamente

In most current operating systems, an executed program’s code is run in a
*process*, and the operating system manages multiple processes at once. Within
your program, you can also have independent parts that run simultaneously. The
feature that runs these independent parts is called *threads*.

Na maioria dos sistemas operacionais atuais, o código de um programa é executado em um
*processo*, e o sistema operacional gerencia vários processos ao mesmo tempo. Dentro do seu 
programa, você também pode ter partes independentes que são executadas simultaneamente. O 
recurso que executa essas partes independentes é chamado *threads*.

Splitting the computation in your program into multiple threads can improve
performance because the program does multiple tasks at the same time, but it
also adds complexity. Because threads can run simultaneously, there’s no
inherent guarantee about the order in which parts of your code on different
threads will run. This can lead to problems, such as:

Dividir a computação em seus programas em várias threads pode melhorar o desempenho 
porque o programa executa várias tarefas ao mesmo tempo, mas também adiciona complexidade. 
Como as threads podem ser executadas simultaneamente, não há garantia inerente sobre a ordem 
em que partes do seu código em diferentes threads serão executadas. Isso pode levar a problemas, como:

* Race conditions, where threads are accessing data or resources in an
  inconsistent order
* Deadlocks, where two threads are waiting for each other to finish using a
  resource the other thread has, preventing both threads from continuing
* Bugs that only happen in certain situations and are hard to reproduce and fix
  reliably

* Condições de corrida, em que as threads acessam dados ou recursos em uma ordem inconsistente
* Deadlocks, em que duas threads aguardam o término do uso de um recurso que a outra thread possui, 
impedindo que ambas as threads continuem
* Erros que só acontecem em determinadas situações e são difíceis de reproduzir e corrigir de forma confiável

Rust attempts to mitigate the negative effects of using threads. Programming in
a multithreaded context still takes careful thought and requires a code
structure that is different from programs that run in a single thread.

Rust tenta atenuar os efeitos negativos do uso de threads. Programação em um contexto 
multithread ainda requer cuidadosa reflexão e requer um código de estrutura diferente dos 
programas executados em uma única thread.

Programming languages implement threads in a few different ways. Many operating
systems provide an API for creating new threads. This model where a language
calls the operating system APIs to create threads is sometimes called *1:1*,
one operating system thread per one language thread.

Linguagens de programação implementam threads de maneiras diferentes. Muitos sistemas operacionais
fornecem uma API para criar novas threads. Este modelo onde uma linguagem chama as APIs do 
sistema operacional para criar threads às vezes é chamado *1:1*, uma thread do sistema operacional 
por uma thread de linguagem.

Many programming languages provide their own special implementation of threads.
Programming language-provided threads are known as *green* threads, and
languages that use these green threads will execute them in the context of a
different number of operating system threads. For this reason, the green
threaded model is called the *M:N* model: `M` green threads per `N` operating
system threads, where `M` and `N` are not necessarily the same number.

Muitas linguagens de programação fornecem sua própria implementação especial de threads.
As threads fornecidas pela linguagem de programação são conhecidas como threads *verdes* (green) 
e as linguagens que usam esses threads verdes irão executá-las num contexto diferente das 
threads do sistema operacional. Por esse motivo, o modelo verde é chamado *M:N*: `M` threads 
verdes por `N` threads do sistema operacional, onde `M` e `N` não são necessariamente o mesmo número.

Each model has its own advantages and trade-offs, and the trade-off most
important to Rust is runtime support. Runtime is a confusing term and can have
different meanings in different contexts.

Cada modelo tem suas próprias vantagens e trade-offs (compensações), e o trade-off mais importante 
para Rust é o suporte ao runtime (tempo de execução). Runtime é um termo confuso e pode ter 
significados diferentes em diferentes contextos.

In this context, by *runtime* we mean code that is included by the language in
every binary. This code can be large or small depending on the language, but
every non-assembly language will have some amount of runtime code. For that
reason, colloquially when people say a language has “no runtime,” they often
mean “small runtime.” Smaller runtimes have fewer features but have the
advantage of resulting in smaller binaries, which make it easier to combine the
language with other languages in more contexts. Although many languages are
okay with increasing the runtime size in exchange for more features, Rust needs
to have nearly no runtime and cannot compromise on being able to call into C to
maintain performance.

Nesse contexto, por *runtime*, queremos dizer código incluído pela linguagem em cada 
binário. Esse código pode ser grande ou pequeno, dependendo da linguagem, mas toda 
linguagem que não seja assembly terá alguma quantidade de código de runtime. Por essa razão, 
coloquialmente quando as pessoas dizem que uma linguagem "não runtime", geralmente
significa "runtime pequeno". Os runtimes menores têm menos recursos, mas têm a
vantagem de resultar em binários menores, o que facilita a combinação da linguagem com 
outras linguagens em mais contextos. Embora muitas linguagens sejam ok com o aumento do 
tamanho do runtime em troca de mais recursos, Rust necessita, praticamente, não ter runtime 
e não pode comprometer a capacidade de chamar C para manter o desempenho.

The green threading M:N model requires a larger language runtime to manage
threads. As such, the Rust standard library only provides an implementation of
1:1 threading. Because Rust is such a low-level language, there are crates that
implement M:N threading if you would rather trade overhead for aspects such as
more control over which threads run when and lower costs of context switching,
for example.

O modelo M:N requer um tempo maior de runtime da linguagem para gerenciar threads. Como tal, 
a biblioteca padrão Rust fornece apenas uma implementação de threading 1:1. Como Rust é uma linguagem 
de baixo nível, existem crates (caixas) que implementam threading M:N, se você preferir 
trocar custos indiretos por aspectos, como maior controle sobre quais threads são executados 
e custos mais baixos de alternância de contexto, por exemplo.

Now that we’ve defined threads in Rust, let’s explore how to use the
thread-related API provided by the standard library.

Agora que definimos threads em Rust, vamos explorar como usar a API relacionada a thread 
fornecida pela biblioteca padrão.

### Creating a New Thread with `spawn`

### Criando uma Nova Thread com `spawn` 

To create a new thread, we call the `thread::spawn` (gerada/criada) function and pass it a
closure (we talked about closures in Chapter 13) containing the code we want to
run in the new thread. The example in Listing 16-1 prints some text from a main
thread and other text from a new thread:

Para criar uma nova thread, chamamos a função `thread::spawn` e passamos para uma closure 
(falamos sobre closure no Capítulo 13) contendo o código que queremos executar na nova thread. 
O exemplo na Listagem 16-1 imprime texto da thread principal e texto da nova thread:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

<span class="caption">Listing 16-1: Creating a new thread to print one thing
while the main thread prints something else</span>

<span class="caption">Listagem 16-1: Criando uma nova thread para imprimir uma coisa
enquanto a thread principal imprime outra coisa</span>

Note that with this function, the new thread will be stopped when the main
thread ends, whether or not it has finished running. The output from this
program might be a little different every time, but it will look similar to the
following:

Observe que, com esta função, a nova thread será parada quando a thread principal terminar, 
independentemente de ter terminado ou não a execução. A saída deste programa pode ser um 
pouco diferente a cada vez, mas será semelhante à seguinte:

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

The calls to `thread::sleep` force a thread to stop its execution for a short
duration, which allows a different thread to run. The threads will probably
take turns, but that isn’t guaranteed: it depends on how your operating system
schedules the threads. In this run, the main thread printed first, even though
the print statement from the spawned thread appears first in the code. And even
though we told the spawned thread to print until `i` is 9, it only got to 5
before the main thread shut down.

As chamadas para `thread::sleep` forçam a thread a interromper sua execução por um 
curto período, o que permite que uma thread diferente seja executada. As threads 
provavelmente se revezarão, mas isso não é garantido: depende de como o sistema 
operacional agenda as threads. Nesta execução, a thread principal imprimiu primeiro, 
mesmo que a instrução de impressão da thread spawned apareça primeiro no código. E 
mesmo que disséssemos que a linha gerada imprimisse até que `i` seja 9, ela só chegou 
a 5, no momento que a thread principal foi desligada.

If you run this code and only see output from the main thread, or don’t see any
overlap, try increasing the numbers in the ranges to create more opportunities
for the operating system to switch between the threads.

Se você executar esse código e vir apenas a saída da thread principal ou não houver 
sobreposição, tente aumentar os números nos intervalos para criar mais oportunidades 
para o sistema operacional alternar entre as threads.

### Waiting for All Threads to Finish Using `join` Handles

### Aguardando a Conclusão de Todas as Threads Usando Identificadores `join`


The code in Listing 16-1 not only stops the spawned thread prematurely most of
the time due to the main thread ending, but there is no guarantee that the
spawned thread will get to run at all. The reason is that there is no guarantee
on the order in which threads run!

O código da Listagem 16-1 interrompe a thread spawned prematuramente devido ao 
término da thread principal, mas não há garantia de que a thread spawned possa 
ao menos executar. O motivo é que não há garantia na ordem em que as threads 
são executadas!

We can fix the problem of the spawned thread not getting to run, or not getting
to run completely, by saving the return value of `thread::spawn` in a variable.
The return type of `thread::spawn` is `JoinHandle`. A `JoinHandle` is an owned
value that, when we call the `join` method on it, will wait for its thread to
finish. Listing 16-2 shows how to use the `JoinHandle` of the thread we created
in Listing 16-1 and call `join` to make sure the spawned thread finishes before
`main` exits:

Podemos corrigir o problema da thread spawned não funcionar, ou não funcionar completamente, 
salvando o valor de retorno de `thread::spawn` em uma variável. O tipo de retorno de 
`thread::spawn` é `JoinHandle`. Um `JoinHandle` é um valor de propriedade que, quando chamamos 
o método `join`, aguardará seu término. A Listagem 16-2 mostra como usar o `JoinHandle` da 
thread que criamos na Listagem 16-1 e chame `join` para garantir que o thread spawned 
termine antes do fim da execução do `main`:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

<span class="caption">Listing 16-2: Saving a `JoinHandle` from `thread::spawn`
to guarantee the thread is run to completion</span>

<span class="caption">Listagem 16-2: Salvando um `JoinHandle` da `thread::spawn`
para garantir que a thread seja executada até a conclusão</span>

Calling `join` on the handle blocks the thread currently running until the
thread represented by the handle terminates. *Blocking* a thread means that
thread is prevented from performing work or exiting. Because we’ve put the call
to `join` after the main thread’s `for` loop, running Listing 16-2 should
produce output similar to this:

Chamar `join` no identificador bloqueia a thread atualmente em execução até que a
thread representado pelo identificador termine. *Blocking* (Bloquear) uma thread significa 
que que essa fica impedida de executar ou de encerrar. Como colocamos a chamada para `join` 
após o loop `for` da thread principal, a execução da Listagem 16-2 deve produzir uma 
saída semelhante a esta:

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

The two threads continue alternating, but the main thread waits because of the
call to `handle.join()` and does not end until the spawned thread is finished.

As duas threads continuam alternando, mas a thread principal espera por causa da 
chamada para `handle.join()` e não termina até que o thread spawned seja finalizada.

But let’s see what happens when we instead move `handle.join()` before the
`for` loop in `main`, like this:

Mas vamos ver o que acontece quando movemos `handle.join()` antes do 
loop `for` em `main`, assim:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

The main thread will wait for the spawned thread to finish and then run its
`for` loop, so the output won’t be interleaved anymore, as shown here:

A thread principal aguardará o final da thread spawned e executará seu loop `for`, 
para que a saída não seja mais intercalada, como mostrado aqui:

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Thinking about such a small detail as where to call `join` can affect whether
or not your threads run at the same time.

Pense bem, como um pequeno detalhe, onde chamar `join` pode afetar a execução 
ou não  de suas threads executadas ao mesmo tempo.

### Using `move` Closures with Threads

### Usando Closure `move` com Threads

The `move` closure, which we mentioned briefly in Chapter 13, is often used
alongside `thread::spawn` because it allows us to use data from one thread in
another thread.

O closure `move`, que mencionamos brevemente no Capítulo 13, é freqüentemente usado 
junto com `thread::spawn` porque nos permite usar dados de uma thread em outra thread.

In Chapter 13, we said that “If we want to force the closure to take ownership
of the values it uses in the environment, we can use the `move` keyword before
the parameter list. This technique is mostly useful when passing a closure to a
new thread to move the data so it’s owned by the new thread.”

No capítulo 13, dissemos que “se queremos forçar a closure a ter ownership dos valores que ela 
usa no ambiente, podemos usar a palavra-chave `move` antes da lista de parâmetros. Essa técnica 
é útil principalmente ao passar dados de uma closure para uma thread e a thread ter ownership sob 
esses dados".

Now that we’re creating new threads, we’ll talk about capturing values in
closures.

Agora que estamos criando novas threads, falaremos sobre a captura de valores 
em closures.

Notice in Listing 16-1 that the closure we pass to `thread::spawn` takes no
arguments: we’re not using any data from the main thread in the spawned
thread’s code. To do so, the spawned thread’s closure must capture the values
it needs. Listing 16-3 shows an attempt to create a vector in the main thread
and use it in the spawned thread. However, this won’t yet work, as you’ll see
in a moment:

Observe na Listagem 16-1 que a closure que passamos para `thread::spawn` não exige 
argumentos: não estamos usando dados da thread principal no código da thread spawned. 
Para fazer isso, a closure da thread spawned deve capturar os valores necessários. A 
Listagem 16-3 mostra uma tentativa de criar um vetor na thread principal e usá-la na 
thread spawned. No entanto, isso ainda não funcionará, como você verá em um momento:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

<span class="caption">Listing 16-3: Attempting to use a vector created by the
main thread in another thread</span>

<span class="caption">Lista 16-3: Tentativa de usar um vetor criado pela
thread principal em outra thread</span>

The closure uses `v`, so it will capture `v` and make it part of the closure’s
environment. Because `thread::spawn` runs this closure in a new thread, we
should be able to access `v` inside that new thread. But when we compile this
example, we get the following error:

A closure usa `v`, então ela capturará `v` e fará com que ele faça parte do
ambiente da closure. Como a `thread::spawn` executa essa closure em uma nova thread, nós
devemos poder acessar `v` dentro dessa nova thread. Mas quando compilamos o 
exemplo, obtemos o seguinte erro:

```text
error[E0373]: closure may outlive the current function, but it borrows `v`,
which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value `v`
7 |         println!("Here's a vector: {:?}", v);
  |                                           - `v` is borrowed here
  |
help: to force the closure to take ownership of `v` (and any other referenced
variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ^^^^^^^
```

Rust *infers* how to capture `v`, and because `println!` only needs a reference
to `v`, the closure tries to borrow `v`. However, there’s a problem: Rust can’t
tell how long the spawned thread will run, so it doesn’t know if the reference
to `v` will always be valid.

Rust *infere* como capturar `v` e porque `println!` apenas precisa de uma referência
para `v`, a closure tenta emprestar `v`. No entanto, há um problema: Rust não pode
dizer quanto tempo a thread gerada será executada, então ele não sabe por quanto 
tempo a referência para `v` será válida.

Listing 16-4 provides a scenario that’s more likely to have a reference to `v`
that won’t be valid:

A Listagem 16-4 fornece um cenário com maior probabilidade de ter uma referência para `v`
que não será válida:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}
```

<span class="caption">Listing 16-4: A thread with a closure that attempts to
capture a reference to `v` from a main thread that drops `v`</span>

<span class="caption">Listagem 16-4: Uma thread com uma closure que tenta
capturar uma referência ao `v` de um thread principal que drops (descarte) o `v`</span>

If we were allowed to run this code, there’s a possibility the spawned thread
will be immediately put in the background without running at all. The spawned
thread has a reference to `v` inside, but the main thread immediately drops
`v`, using the `drop` function we discussed in Chapter 15. Then, when the
spawned thread starts to execute, `v` is no longer valid, so a reference to it
is also invalid. Oh no!

Se pudéssemos executar esse código, existe a possibilidade de a thread spawned ser 
imediatamente colocada em segundo plano sem ser executada. A thread spawned tem dentro uma 
referência ao `v`, mas a thread principal drops (descarta) o `v` imediatamente, usando 
a função `drop` que discutimos no Capítulo 15. Então, quando a thread spawned começa a 
ser executada, `v` não é mais válido, portanto, uma referência a ela também é inválida. Ah não!

To fix the compiler error in Listing 16-3, we can use the error message’s
advice:

Para corrigir o erro do compilador na Listagem 16-3, podemos usar os conselhos 
da mensagem de erro:

```text
help: to force the closure to take ownership of `v` (and any other referenced
variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ^^^^^^^
```

By adding the `move` keyword before the closure, we force the closure to take
ownership of the values it’s using rather than allowing Rust to infer that it
should borrow the values. The modification to Listing 16-3 shown in Listing
16-5 will compile and run as we intend:

Ao adicionar a palavra-chave `move` antes da closure, forçamos a closure a ter o
ownership dos valores que está usando, em vez de permitir que Rust deduza que deve 
emprestar os valores. A modificação na Listagem 16-3 mostrada na Listagem
16-5 serão compilados e executados conforme pretendemos:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

<span class="caption">Listing 16-5: Using the `move` keyword to force a closure
to take ownership of the values it uses</span>

<span class="caption">Listagem 16-5: Usando a palavra-chave `move` para forçar uma closure
para ter ownership dos valores que usa</span>

What would happen to the code in Listing 16-4 where the main thread called
`drop` if we use a `move` closure? Would `move` fix that case? Unfortunately,
no; we would get a different error because what Listing 16-4 is trying to do
isn’t allowed for a different reason. If we add `move` to the closure, we would
move `v` into the closure’s environment, and we could no longer call `drop` on
it in the main thread. We would get this compiler error instead:

O que aconteceria com o código na Listagem 16-4 em que a thread principal chamado
`drop` se usarmos uma closure `move`? O `move` corrige esse caso? Infelizmente,
não; receberíamos um erro diferente porque o que a Listagem 16-4 está tentando fazer
não é permitido por um motivo diferente. Se adicionarmos `move` a closure, faríamos
mover `v` para o ambiente da closure e não poderíamos mais chamar `drop`
na linha principal. Em vez disso, obteríamos esse erro do compilador:

```text
error[E0382]: use of moved value: `v`
  --> src/main.rs:10:10
   |
6  |     let handle = thread::spawn(move || {
   |                                ------- value moved (into closure) here
...
10 |     drop(v); // oh no!
   |          ^ value used here after move
   |
   = note: move occurs because `v` has type `std::vec::Vec<i32>`, which does
   not implement the `Copy` trait
```

Rust’s ownership rules have saved us again! We got an error from the code in
Listing 16-3 because Rust was being conservative and only borrowing `v` for the
thread, which meant the main thread could theoretically invalidate the spawned
thread’s reference. By telling Rust to move ownership of `v` to the spawned
thread, we’re guaranteeing Rust that the main thread won’t use `v` anymore. If
we change Listing 16-4 in the same way, we’re then violating the ownership
rules when we try to use `v` in the main thread. The `move` keyword overrides
Rust’s conservative default of borrowing; it doesn’t let us violate the
ownership rules.

As regras de ownership Rust nos salvaram de novo! Ocorreu um erro no código da Listagem 
16-3 porque Rust estava sendo conservador e apenas emprestou `v` para a thread, 
o que significava que a thread principal teoricamente poderia invalidar a referência 
da thread spawned. Ao dizer para Rust em mudar a ownership de `v` para a thread spawned, 
estamos garantindo ao Rust que a thread principal não use mais o `v`. Se alterarmos 
a Listagem 16-4 da mesma maneira, violaremos as regras de ownership ao tentarmos usar `v`
na thread principal. A palavra-chave `move` substitui o padrão conservador de 
ownership Rust; não nos deixa violar regras de ownership.

With a basic understanding of threads and the thread API, let’s look at what we
can *do* with threads.

Com um entendimento básico das threads e da API de thread, vejamos o que podemos 
*fazer*  com as threads.