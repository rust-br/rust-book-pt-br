## Shared State Concurrency
## Concorrência de Estado Compartilhado

Message passing is a fine way of handling concurrency, but it’s not the only
one. Consider this part of the slogan from the Go language documentation again:
“communicate by sharing memory.”

A passagem de mensagens é uma ótima maneira de lidar com a concorrência, mas não 
é a única. Considere esta parte do slogan da documentação da linguagem 
Go novamente: "comunique-se compartilhando memória".

What would communicating by sharing memory look like? In addition, why would
message passing enthusiasts not use it and do the opposite instead?

Como seria a comunicação compartilhando memória? Além disso, por que os entusiastas 
da transmissão de mensagens não o usariam e, em vez disso, fariam o contrário?

In a way, channels in any programming language are similar to single ownership,
because once you transfer a value down a channel, you should no longer use that
value. Shared memory concurrency is like multiple ownership: multiple threads
can access the same memory location at the same time. As you saw in Chapter 15
where smart pointers made multiple ownership possible, multiple ownership can
add additional complexity because these different owners need managing. Rust’s
type system and ownership rules greatly assist in getting this management
correct. For an example, let’s look at mutexes, one of the more common
concurrency primitives for shared memory.

De certa forma, os canais em qualquer linguagem de programação são semelhantes à ownership única, 
porque depois de transferir um valor para um canal, você não deve mais usá-lo. A concorrência 
de memória compartilhada é como ownership múltipla: vários segmentos podem acessar o mesmo 
local de memória ao mesmo tempo. Como você viu no Capítulo 15, onde ponteiros inteligentes 
tornaram possível a ownsership múltipla, a ownership múltipla pode adicionar complexidade 
adicional porque esses proprietários diferentes precisam ser gerenciados. O sistema de 
tipos e as regras de ownership Rust ajudam muito a corrigir esse gerenciamento. Por exemplo, 
vejamos as mutexes, uma das primitivas de simultaneidade mais comuns para memória compartilhada.

### Mutexes Allow Access to Data from One Thread at a Time
### Mutexes Permitem Acesso a Dados de Uma Thread por Vez

A *mutex* is an abbreviation for “mutual exclusion,” as in, it only allows one
thread to access some data at any given time. To access the data in a mutex, a
thread must first signal that it wants access by asking to acquire the mutex’s
*lock*. The lock is a data structure that is part of the mutex that keeps track
of who currently has exclusive access to the data. Therefore, we describe the
mutex as *guarding* the data it holds via the locking system.

Um *mutex* é uma abreviação de “exclusão mútua“ (mutual exclusion), permite apenas 
que uma thread acesse alguns dados a qualquer momento. Para acessar os dados em um mutex, 
uma thread deve primeiro sinalizar que deseja acessar, solicitando a aquisição do 
bloqueio (*lock*) do mutex. O bloqueio é uma estrutura de dados que faz parte do mutex que 
rastreia quem atualmente tem acesso exclusivo aos dados. Portanto, descrevemos o mutex 
como *quem guarda* os dados que ele mantém por meio do sistema de bloqueio.

Mutexes have a reputation for being difficult to use because you have to
remember two rules:

1. You must attempt to acquire the lock before using the data.
2. When you’re done with the data that the mutex guards, you must unlock the
   data so other threads can acquire the lock.

Os mutexes têm a reputação de serem difíceis de usar, porque você precisa se lembrar de duas regras:

1. Você deve tentar adquirir o bloqueio antes de usar os dados.
2. Ao concluir os dados que o mutex protege, você deve desbloquear o mutex 
   para que outras threads possam adquirir o bloqueio.

For a real-world metaphor of a mutex, imagine a panel discussion at a
conference with only one microphone. Before a panelist can speak, they have to
ask or signal that they want to use the microphone. When they get the
microphone, they can talk for as long as they want to and then hand the
microphone to the next panelist who requests to speak. If a panelist forgets to
hand the microphone off when they’re finished with it, no one else is able to
speak. If management of the shared microphone goes wrong, the panel wouldn’t
work as planned!

Para uma metáfora do mundo real de um mutex, imagine um painel de discussão em uma 
conferência com apenas um microfone. Antes que um membro do painel possa falar, ele precisa 
perguntar ou sinalizar que deseja usar o microfone. Quando eles pegam o microfone, eles podem 
conversar pelo tempo que quiserem e, em seguida, entregam o microfone ao próximo membro do 
painel que pede para falar. Se um membro do painel esquecer de entregar o microfone quando 
terminar, ninguém mais poderá falar. Se o gerenciamento do microfone compartilhado der errado, 
o painel não funcionará conforme o planejado!

Management of mutexes can be incredibly tricky to get right, which is why so
many people are enthusiastic about channels. However, thanks to Rust’s type
system and ownership rules, we can’t get locking and unlocking wrong.

O gerenciamento de mutexes pode ser incrivelmente difícil de acertar, e é por 
isso que tantas pessoas estão entusiasmadas com os canais. No entanto, graças ao 
sistema de tipos e às regras de ownership Rust, não podemos bloquear e desbloquear incorretamente.

#### The API of `Mutex<T>`
#### A API do `Mutex<T>`

As an example of how to use a mutex, let’s start by using a mutex in a
single-threaded context, as shown in Listing 16-12:

Como um exemplo de como usar um mutex, vamos começar usando um mutex em 
um contexto de thread única, como mostra a Listagem 16-12:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

<span class="caption">Listing 16-12: Exploring the API of `Mutex<T>` in a
single-threaded context for simplicity</span>

<span class="caption">Listagem 16-12: Explorando a API do `Mutex<T>` em um 
contexto de thread única para manter a simplicidade</span>

As with many types, we create a `Mutex<T>` using the associated function `new`.
To access the data inside the mutex, we use the `lock` method to acquire the
lock. This call will block the current thread so it can’t do any work until
it’s our turn to have the lock.

Como em muitos tipos, criamos `Mutex<T>` usando a função associada `new`. 
Para acessar os dados dentro do mutex, usamos o método `lock` para adquirir 
o bloqueio. Essa chamada bloqueará a thread atual e, portanto, não realizará trabalho 
até que seja a nossa vez de ter o bloqueio.

The call to `lock` would fail if another thread holding the lock panicked. In
that case, no one would ever be able to get the lock, so we’ve chosen to
`unwrap` and have this thread panic if we’re in that situation.

A chamada para `lock` falharia se outra thread estivesse segurando a 
trava entrasse em pânico. Nesse caso, ninguém jamais conseguiria trancar, 
por isso escolhemos `unwrap` e ter essa thread em pânico se estivermos nessa situação.

After we’ve acquired the lock, we can treat the return value, named `num` in
this case, as a mutable reference to the data inside. The type system ensures
that we acquire a lock before using the value in `m`: `Mutex<i32>` is not an
`i32`, so we *must* acquire the lock to be able to use the `i32` value. We
can’t forget; the type system won’t let us access the inner `i32` otherwise.

Depois de adquirirmos o bloqueio, podemos tratar o valor de retorno, chamado `num` 
nesse caso, como uma referência mutável aos dados internos. O sistema de tipos garante 
que adquirimos um bloqueio antes de usar o valor em `m`:` Mutex<i32>` não é um` i32`, 
portanto *devemos* adquirir o bloqueio para poder usar o valor `i32`. Nós não 
podemos esquecer; o sistema de tipos não nos permitirá acessar o `i32` interno de 
outra forma.

As you might suspect, `Mutex<T>` is a smart pointer. More accurately, the call
to `lock` *returns* a smart pointer called `MutexGuard`. This smart pointer
implements `Deref` to point at our inner data; the smart pointer also has a
`Drop` implementation that releases the lock automatically when a `MutexGuard`
goes out of scope, which happens at the end of the inner scope in Listing
16-12. As a result, we don’t risk forgetting to release the lock and blocking
the mutex from being used by other threads because the lock release happens
automatically.

Como você pode suspeitar, o `Mutex<T>` é um ponteiro inteligente. Mais precisamente, 
a chamada para `lock` *retorna* um ponteiro inteligente chamado `MutexGuard`. Este ponteiro 
inteligente implementa `Deref` que aponta para nossos dados internos; o ponteiro inteligente 
também possui uma implementação `Drop` que libera o bloqueio automaticamente quando um `MutexGuard` 
sai do escopo, o que acontece no final do escopo interno da Listagem 16-12. Como resultado, não 
corremos o risco de esquecer de liberar o bloqueio e impedir que o mutex seja usado por outras 
threads, porque o desbloqueio ocorre automaticamente.

After dropping the lock, we can print the mutex value and see that we were able
to change the inner `i32` to 6.

Depois de soltar a trava (bloqueio), podemos imprimir o valor mutex e ver que conseguimos alterar 
o `i32` interno para o valor 6.

#### Sharing a `Mutex<T>` Between Multiple Threads
#### Compartilhando um `Mutex<T>` Entre Várias Threads

Now, let’s try to share a value between multiple threads using `Mutex<T>`.
We’ll spin up 10 threads and have them each increment a counter value by 1, so
the counter goes from 0 to 10. Note that the next few examples will have
compiler errors, and we’ll use those errors to learn more about using
`Mutex<T>` and how Rust helps us use it correctly. Listing 16-13 has our
starting example:

Agora, vamos tentar compartilhar um valor entre várias threads usando o `Mutex<T>`. 
Subimos 10 threads e cada uma delas incrementará um valor do contador em 1, para que o 
contador passe de 0 a 10. Observe que os próximos exemplos terão erros de compilador e nós 
usaremos para aprender mais sobre como usar `Mutex<T>` e como Rust nos ajuda a usá-lo corretamente. 
A Listagem 16-13 tem nosso exemplo inicial:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

<span class="caption">Listing 16-13: Ten threads each increment a counter
guarded by a `Mutex<T>`</span>

<span class="caption">Listagem 16-13: Dez threads, cada uma incrementa o 
contador protegido por um `Mutex<T>`</span>

We’re creating a `counter` variable to hold an `i32` inside a `Mutex<T>`, as we
did in Listing 16-12. Next, we’re creating 10 threads by mapping over a range
of numbers. We use `thread::spawn` and give all the threads the same closure,
one that moves the counter into the thread, acquires a lock on the `Mutex<T>`
by calling the `lock` method, and then adds 1 to the value in the mutex. When a
thread finishes running its closure, `num` will go out of scope and release the
lock so another thread can acquire it.

Estamos criando uma variável `counter` para armazenar um `i32` dentro de um `Mutex<T>`, como 
fizemos na Listagem 16-12. Em seguida, criamos 10 threads mapeando um intervalo de números. Usamos 
`thread::spawn` e damos a todas as threads o mesmo fechamento (closure), aquele que move o contador 
para a thread, adquire um bloqueio no `Mutex<T> ` chamando o método `lock` e, em seguida, adiciona 
1 ao valor no mutex. Quando uma thread termina o fechamento (closure), o `num` sai do escopo e libera a trava 
para que outra thread possa adquiri-lo.

In the main thread, we collect all the join handles, as we did in Listing 16-2,
and then call `join` on each to make sure all the threads finish. At that
point, the main thread will acquire the lock and print the result of this
program.

Na thread principal, coletamos todas as alças de junção (join handles), como fizemos na Listagem 16-2, 
e então chamamos `join` em cada uma para garantir que todas as threads terminem. Nesse ponto, 
a thread principal adquirirá o bloqueio e imprimirá o resultado deste programa.

We hinted that this example won’t compile, now let’s find out why!

Sugerimos que este exemplo não compila, agora vamos descobrir o porquê!

```text
error[E0382]: capture of moved value: `counter`
  --> src/main.rs:10:27
   |
9  |         let handle = thread::spawn(move || {
   |                                    ------- value moved (into closure) here
10 |             let mut num = counter.lock().unwrap();
   |                           ^^^^^^^ value captured here after move
   |
   = note: move occurs because `counter` has type `std::sync::Mutex<i32>`,
   which does not implement the `Copy` trait

error[E0382]: use of moved value: `counter`
  --> src/main.rs:21:29
   |
9  |         let handle = thread::spawn(move || {
   |                                    ------- value moved (into closure) here
...
21 |     println!("Result: {}", *counter.lock().unwrap());
   |                             ^^^^^^^ value used here after move
   |
   = note: move occurs because `counter` has type `std::sync::Mutex<i32>`,
   which does not implement the `Copy` trait

error: aborting due to 2 previous errors
```

The error message states that the `counter` value is moved into the closure and
then is captured when we call `lock`. That description sounds like what we
wanted, but it’s not allowed!

A mensagem de erro informa que o valor do `counter` é movido para a closure e, 
em seguida, é capturado quando chamamos `lock`. Essa descrição parece o que queríamos, 
mas não é permitido!

Let’s figure this out by simplifying the program. Instead of making 10 threads
in a `for` loop, let’s just make two threads without a loop and see what
happens. Replace the first `for` loop in Listing 16-13 with this code instead:

Vamos descobrir isso simplificando o programa. Em vez de fazer 10 threads em um 
loop `for`, vamos fazer duas threads sem loop e ver o que acontece. Substitua o 
primeiro loop `for` na Listagem 16-13 por este código:

```rust,ignore
let handle = thread::spawn(move || {
    let mut num = counter.lock().unwrap();

    *num += 1;
});
handles.push(handle);

let handle2 = thread::spawn(move || {
    let mut num2 = counter.lock().unwrap();

    *num2 += 1;
});
handles.push(handle2);
```

We make two threads and change the variable names used with the second thread
to `handle2` and `num2`. When we run the code this time, compiling gives us the
following:

Criamos duas threads e alteramos os nomes das variáveis usadas com a segunda thread 
para `handle2` e `num2`. Quando executamos o código dessa vez, a compilação 
nos fornece o seguinte:

```text
error[E0382]: capture of moved value: `counter`
  --> src/main.rs:16:24
   |
8  |     let handle = thread::spawn(move || {
   |                                ------- value moved (into closure) here
...
16 |         let mut num2 = counter.lock().unwrap();
   |                        ^^^^^^^ value captured here after move
   |
   = note: move occurs because `counter` has type `std::sync::Mutex<i32>`,
   which does not implement the `Copy` trait

error[E0382]: use of moved value: `counter`
  --> src/main.rs:26:29
   |
8  |     let handle = thread::spawn(move || {
   |                                ------- value moved (into closure) here
...
26 |     println!("Result: {}", *counter.lock().unwrap());
   |                             ^^^^^^^ value used here after move
   |
   = note: move occurs because `counter` has type `std::sync::Mutex<i32>`,
   which does not implement the `Copy` trait

error: aborting due to 2 previous errors
```

Aha! The first error message indicates that `counter` is moved into the closure
for the thread associated with `handle`. That move is preventing us from
capturing `counter` when we try to call `lock` on it and store the result in
`num2` in the second thread! So Rust is telling us that we can’t move ownership
of `counter` into multiple threads. This was hard to see earlier because our
threads were in a loop, and Rust can’t point to different threads in different
iterations of the loop. Let’s fix the compiler error with a multiple-ownership
method we discussed in Chapter 15.

Aha! A primeira mensagem de erro indica que o `counter` é movido para a closure da thread 
associada ao `handle`. Esse move está nos impedindo de capturar o `counter`  quando 
tentamos chamar o `lock` e armazenar o resultado em `num2` na segunda thread! Portanto, 
Rust está nos dizendo que não podemos mudar a propriedade do `counter` para vários segmentos. 
Isso foi difícil de ver anteriormente, porque nossas threads estavam em um loop e o Rust não 
pode apontar para threads diferentes em diferentes iterações do loop. Vamos corrigir o erro 
do compilador com um método de multiple-ownership que discutimos no capítulo 15.

#### Multiple Ownership with Multiple Threads
#### Múltiplas Ownership com Multiplas Threads

In Chapter 15, we gave a value multiple owners by using the smart pointer
`Rc<T>` to create a reference-counted value. Let’s do the same here and see
what happens. We’ll wrap the `Mutex<T>` in `Rc<T>` in Listing 16-14 and clone
the `Rc<T>` before moving ownership to the thread. Now that we’ve seen the
errors, we’ll also switch back to using the `for` loop, and we’ll keep the
`move` keyword with the closure:

No Capítulo 15, atribuímos um valor a vários owners (proprietários) usando o ponteiro 
inteligente `Rc<T>` para criar um valor contado por referência. Vamos fazer o mesmo aqui 
e ver o que acontece. Vamos agrupar o `Mutex<T>` em `Rc<T>` na Listagem 16-14 e clonar o 
`Rc<T>` antes de mover a ownership para a thread. Agora que vimos os erros, 
também voltaremos a usar o loop `for` e manteremos a palavra-chave `move` com a closure:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

<span class="caption">Listing 16-14: Attempting to use `Rc<T>` to allow
multiple threads to own the `Mutex<T>`</span>

<span class="caption">Lista 16-14: Tentativa de usar `Rc<T>` para permitir 
que várias threads possuam o `Mutex<T>`</span>

Once again, we compile and get... different errors! The compiler is teaching us
a lot.

Mais uma vez, compilamos e obtemos ... erros diferentes! O compilador está 
nos ensinando muito.

```text
error[E0277]: the trait bound `std::rc::Rc<std::sync::Mutex<i32>>:
std::marker::Send` is not satisfied in `[closure@src/main.rs:11:36:
15:10 counter:std::rc::Rc<std::sync::Mutex<i32>>]`
  --> src/main.rs:11:22
   |
11 |         let handle = thread::spawn(move || {
   |                      ^^^^^^^^^^^^^ `std::rc::Rc<std::sync::Mutex<i32>>`
cannot be sent between threads safely
   |
   = help: within `[closure@src/main.rs:11:36: 15:10
counter:std::rc::Rc<std::sync::Mutex<i32>>]`, the trait `std::marker::Send` is
not implemented for `std::rc::Rc<std::sync::Mutex<i32>>`
   = note: required because it appears within the type
`[closure@src/main.rs:11:36: 15:10 counter:std::rc::Rc<std::sync::Mutex<i32>>]`
   = note: required by `std::thread::spawn`
```

Wow, that error message is very wordy! Here are some important parts to focus
on: the first inline error says `` `std::rc::Rc<std::sync::Mutex<i32>>` cannot 
be sent between threads safely ``. The reason for this is in the next important
part to focus on, the error message. The distilled error message says `` the
trait bound `Send` is not satisfied ``. We’ll talk about `Send` in the next
section: it’s one of the traits that ensures the types we use with threads are
meant for use in concurrent situations.

Uau, essa mensagem de erro é muita prolixa! Aqui estão algumas partes importantes para 
focar: o primeiro erro diz que `` `std::rc::Rc<std::sync::Mutex<i32>>` cannot be sent 
between threads safely``  (`` `std::rc::Rc<std::sync::Mutex<i32>>` não pode ser 
enviado com segurança entre as threads ``) . A razão para isso está na próxima parte 
importante a ser focada, na mensagem de erro. A mensagem de erro destilada diz `` the
trait bound `Send` is not satisfied ``. Falaremos sobre o `Send` na próxima seção: é uma 
das traits que garante que os tipos que usamos com threads sejam usados em situações concorrentes.

Unfortunately, `Rc<T>` is not safe to share across threads. When `Rc<T>`
manages the reference count, it adds to the count for each call to `clone` and
subtracts from the count when each clone is dropped. But it doesn’t use any
concurrency primitives to make sure that changes to the count can’t be
interrupted by another thread. This could lead to wrong counts—subtle bugs that
could in turn lead to memory leaks or a value being dropped before we’re done
with it. What we need is a type exactly like `Rc<T>` but one that makes changes
to the reference count in a thread-safe way.

Infelizmente, `Rc<T>` não é seguro para compartilhar threads. Quando `Rc<T>` gerencia 
a contagem de referência, ela é adicionada à contagem de cada chamada para `clonar` e 
subtrai da contagem quando cada clone é descartado. Mas ele não usa nenhuma primitiva 
de concorrência para garantir que as alterações na contagem não sejam interrompidas 
por outra thread. Isso pode levar a contagens erradas - erros sutis que, por sua vez, 
podem levar a vazamentos de memória ou perda de valor antes que terminemos. O que precisamos 
é de um tipo exatamente como `Rc<T>`, mas que faça alterações na referência de contagem de 
maneira segura em threads.

#### Atomic Reference Counting with `Arc<T>`
#### Referência Atômica Contando Com `Arc<T>`

Fortunately, `Arc<T>` *is* a type like `Rc<T>` that is safe to use in
concurrent situations. The ‘a’ stands for *atomic*, meaning it’s an *atomically
reference counted* type. Atomics are an additional kind of concurrency
primitive that we won’t cover in detail here: see the standard library
documentation for `std::sync::atomic` for more details. At this point, you just
need to know that atomics work like primitive types but are safe to share
across threads.

Felizmente, `Arc<T>` *é* um tipo como `Rc<T>` que é seguro para uso em
situações concorrentes. O ‘a’ significa *atômico*, o que significa que é um 
tipo *atômico de contagem*. Atomics é um tipo adicional de concorrência 
primitiva que não abordaremos em detalhes aqui: consulte a documentação da biblioteca 
padrão para `std::sync::atomic` para obter mais detalhes. Neste ponto, você só 
precisa saber que os atômicos funcionam como tipos primitivos, mas são seguros para 
compartilhar através de threads.

You might then wonder why all primitive types aren’t atomic and why standard
library types aren’t implemented to use `Arc<T>` by default. The reason is that
thread safety comes with a performance penalty that you only want to pay when
you really need to. If you’re just performing operations on values within a
single thread, your code can run faster if it doesn’t have to enforce the
guarantees atomics provide.

Você pode então se perguntar por que todos os tipos primitivos não são atômicos e 
por que os tipos da biblioteca padrão não são implementados para usar o `Arc<T>` por padrão. 
O motivo é que a segurança da thread vem com uma penalidade de desempenho que você 
só deseja pagar quando realmente precisa. Se você está apenas executando operações 
com valores em uma única thread, seu código pode ser executado mais rapidamente 
se não precisar impor as garantias que a atomicidade fornece.

Let’s return to our example: `Arc<T>` and `Rc<T>` have the same API, so we fix
our program by changing the `use` line, the call to `new`, and the call to
`clone`. The code in Listing 16-15 will finally compile and run:

Vamos voltar ao nosso exemplo: `Arc<T>` e `Rc<T>` têm a mesma API, portanto, 
corrigimos nosso programa alterando a linha `use`, a chamada para `new` e a chamada 
para `clone`. O código da Lista 16-15 finalmente compilará e executará:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

<span class="caption">Listing 16-15: Using an `Arc<T>` to wrap the `Mutex<T>`
to be able to share ownership across multiple threads</span>

<span class="caption">Listagem 16-15: Usando um `Arc<T>` para agrupar o `Mutex<T>` 
para poder compartilhar o ownership através de várias threads</span>

This code will print the following:

Este código imprimirá o seguinte:

```text
Result: 10
```

We did it! We counted from 0 to 10, which may not seem very impressive, but it
did teach us a lot about `Mutex<T>` and thread safety. You could also use this
program’s structure to do more complicated operations than just incrementing a
counter. Using this strategy, you can divide a calculation into independent
parts, split those parts across threads, then use a `Mutex<T>` to have each
thread update the final result with its part.

Conseguimos! Contamos de 0 a 10, o que pode não parecer muito impressionante, 
mas nos ensinou muito sobre o `Mutex<T>` e a segurança em threads. Você também pode 
usar a estrutura deste programa para realizar operações mais complicadas do que apenas 
incrementar um contador. Usando essa estratégia, você pode dividir um cálculo em partes 
independentes, dividi-las entre threads e, em seguida, usar o `Mutex<T>` para ter cada
thread atualiza o resultado final com sua parte.

### Similarities Between `RefCell<T>`/`Rc<T>` and `Mutex<T>`/`Arc<T>`
### Semelhanças Entre `RefCell<T>` / `Rc<T>` e `Mutex<T>` / `Arc<T>`

You might have noticed that `counter` is immutable, but we could get a mutable
reference to the value inside it; this means `Mutex<T>` provides interior
mutability, like the `Cell` family does. In the same way we used `RefCell<T>`
in Chapter 15 to allow us to mutate contents inside an `Rc<T>`, we use
`Mutex<T>` to mutate contents inside an `Arc<T>`.

Você deve ter notado que o `counter` é imutável, mas podemos obter uma referência 
mutável ao valor dentro dele; isso significa que `Mutex<T>` fornece mutabilidade 
interior, como a família `Cell` faz. Do mesmo modo que usamos `RefCell<T>` no 
Capítulo 15 para nos permitir mudar o conteúdo dentro de um `Rc<T>`, usamos o 
`Mutex<T>` para mudar o conteúdo dentro de um `Arc<T>`.

Another detail to note is that Rust can’t protect us from all kinds of logic
errors when we use `Mutex<T>`. Recall in Chapter 15 that using `Rc<T>` came
with the risk of creating reference cycles, where two `Rc<T>` values refer to
each other, causing memory leaks. Similarly, `Mutex<T>` comes with the risk of
creating *deadlocks*. These occur when an operation needs to lock two resources
and two threads have each acquired one of the locks, causing them to wait for
each other forever. If you’re interested in deadlocks, try creating a Rust
program that has a deadlock; then research deadlock mitigation strategies for
mutexes in any language and have a go at implementing them in Rust. The
standard library API documentation for `Mutex<T>` and `MutexGuard` offers
useful information.

Outro detalhe a ser observado é que Rust não pode nos proteger de todos os tipos 
de erros lógicos quando usamos o `Mutex<T>`. Lembre-se no Capítulo 15 que o uso de `Rc<T>` 
corria o risco de criar ciclos de referência, onde dois valores de `Rc<T>` se referem um 
ao outro, causando vazamento de memória. Da mesma forma, o `Mutex<T>` vem com o risco de 
criar *deadlocks*. Isso ocorre quando uma operação precisa bloquear dois recursos e duas threads 
adquiriram um dos bloqueios, fazendo com que esperem uma pela outra para sempre. Se você estiver 
interessado em deadlocks, tente criar um programa Rust com deadlocks; depois pesquise estratégias 
de mitigação de deadlocks para mutex em qualquer linguagem e implementa-os em Rust. A documentação 
da API da biblioteca padrão para `Mutex<T>` e `MutexGuard` oferece informações úteis.

We’ll round out this chapter by talking about the `Send` and `Sync` traits, and
how we can use them with custom types.

Concluiremos este capítulo falando sobre as traits `Send` e `Sync`, 
e como podemos usá-las com tipos personalizados.