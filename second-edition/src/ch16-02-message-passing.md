## Message Passing to Transfer Data Between Threads
## Transferir Dados Entre Threads Passando Mensagens

One increasingly popular approach to ensuring safe concurrency is *message
passing*, where threads or actors communicate by sending each other messages
containing data. Here’s the idea in a slogan from the Go language documentation:

Uma abordagem cada vez mais popular para garantir a concorrência segura é a 
*message passing* (passagem de mensagens), na qual threads ou atores se comunicam enviando mensagens 
contendo dados. Aqui está a idéia em um slogan da documentação da linguagem Go:

> Do not communicate by sharing memory; instead, share memory by
> communicating.
>
> --[Effective Go](http://golang.org/doc/effective_go.html)

> Não se comunique compartilhando memória; em vez disso, compartilhe memória
> comunicando.
>
> --[Effective Go](http://golang.org/doc/effective_go.html)

One major tool Rust has for accomplishing message sending concurrency is the
*channel*, a programming concept that Rust’s standard library provides an
implementation of. You can imagine a channel in programming like a channel of
water, such as a stream or a river. If you put something like a rubber duck or
a boat into a stream, it will travel downstream to the end of the river.

Uma ferramenta importante que Rust possui para realizar o envio de mensagens 
concorrentes é o *canal* (channel), um conceito de programação que a biblioteca 
padrão do Rust fornece uma implementação. Você pode imaginar um canal na programação 
como um canal de água, como um córrego ou um rio. Se você colocar algo como um pato 
de borracha ou um barco em um riacho, ele viajará rio abaixo até o final do rio.

A channel in programming has two halves: a transmitter and a receiver. The
transmitter half is the upstream location where we put rubber ducks into the
river, and the receiver half is where the rubber duck ends up downstream. One
part of our code calls methods on the transmitter with the data we want to
send, and another part checks the receiving end for arriving messages. A
channel is said to be *closed* if either the transmitter or receiver half is
dropped.

Um canal na programação tem duas metades: um transmissor e um receptor. 
A metade do transmissor é o local a montante, onde colocamos os patos de borracha 
no rio, e a metade do receptor é o local onde o pato de borracha termina a jusante. 
Uma parte do nosso código chama métodos no transmissor com os dados que queremos enviar, 
e outra parte verifica o recebimento de mensagens. Diz-se que um canal está *fechado* (closed) se 
a metade do transmissor ou do receptor cair.

Here, we’ll work up to a program that has one thread to generate values and
send them down a channel, and another thread that will receive the values and
print them out. We’ll be sending simple values between threads using a channel
to illustrate the feature. Once you’re familiar with the technique, you could
use channels to implement a chat system or a system where many threads perform
parts of a calculation and send the parts to one thread that aggregates the
results.

Aqui, trabalharemos em um programa que tenha uma thread para gerar valores e enviá-los 
por um canal, e outra thread que receberá os valores e os imprimirá. Enviaremos valores 
simples entre threads usando um canal para ilustrar o recurso. Quando você estiver familiarizado 
com a técnica, poderá usar canais para implementar um sistema de bate-papo (chat) ou um sistema em 
que muitas threads realizam partes de um cálculo e enviam as partes para um thread que agrega os
resultados.

First, in Listing 16-6, we’ll create a channel but not do anything with it.
Note that this won’t compile yet because Rust can’t tell what type of values we
want to send over the channel:

Primeiro, na Listagem 16-6, criaremos um canal, mas não faremos nada com ele.
Observe que isso ainda não será compilado porque Rust não pode dizer que tipo de valores 
queremos enviar pelo canal:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
#     tx.send(()).unwrap();
}
```

<span class="caption">Listing 16-6: Creating a channel and assigning the two
halves to `tx` and `rx`</span>

<span class = "caption">Listagem 16-6: Criando um canal e atribuindo as 
duas metades para `tx` e` rx`</span>

We create a new channel using the `mpsc::channel` function; `mpsc` stands for
*multiple producer, single consumer*. In short, the way Rust’s standard library
implements channels means a channel can have multiple *sending* ends that
produce values but only one *receiving* end that consumes those values. Imagine
multiple rivers and streams flowing together into one big river: everything
sent down any of the streams will end up in one river at the end. We’ll start
with a single producer for now, but we’ll add multiple producers when we get
this example working.

Criamos um novo canal usando a função `mpsc::channel`; `mpsc` significa 
*multiple producer, single consumer* (produtor múltiplo, consumidor único). Em resumo, 
a maneira como a biblioteca padrão Rust implementa canais significa que um canal pode 
ter várias extremidades *de envio* que produzem valores, mas apenas uma extremidade 
*de recebimento* que consome esses valores. Imagine vários rios e riachos fluindo juntos 
para um grande rio: tudo o que for enviado por qualquer um dos riachos terminará em um 
rio no final. Vamos começar com um único produtor por enquanto, mas adicionaremos vários 
produtores quando este exemplo funcionar.

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

The `mpsc::channel` function returns a tuple, the first element of which is the
sending end and the second element is the receiving end. The abbreviations `tx`
and `rx` are traditionally used in many fields for *transmitter* and *receiver*
respectively, so we name our variables as such to indicate each end. We’re
using a `let` statement with a pattern that destructures the tuples; we’ll
discuss the use of patterns in `let` statements and destructuring in
Chapter 18. Using a `let` statement this way is a convenient approach to
extract the pieces of the tuple returned by `mpsc::channel`.

A função `mpsc::channel` retorna uma tupla, cujo primeiro elemento é o terminal de envio 
e o segundo elemento é o terminal de recebimento. As abreviaturas `tx` e `rx` são tradicionalmente 
usadas em muitos campos para *trasmissor* e *receptor* respectivamente, portanto, 
nomeamos nossas variáveis como tal para indicar cada extremidade. Estamos usando uma declaração 
`let` com um padrão que destrói as tuplas; discutiremos o uso de padrões nas instruções `let` e 
a destruição no Capítulo 18. Usar uma declaração` let` dessa maneira é uma abordagem 
conveniente para extrair as partes da tupla retornadas pelo `mpsc::channel`.

Let’s move the transmitting end into a spawned thread and have it send one
string so the spawned thread is communicating with the main thread, as shown in
Listing 16-7. This is like putting a rubber duck in the river upstream or
sending a chat message from one thread to another:

Vamos mover a extremidade de transmissão para uma thread gerada (spawn) e enviar uma 
string para que a thread gerada se comunique com a thread principal, conforme 
mostrado na Listagem 16-7. É como colocar um pato de borracha no rio a montante 
ou enviar uma mensagem de bate-papo de uma thread a outra:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
```

<span class="caption">Listing 16-7: Moving `tx` to a spawned thread and sending
“hi”</span>

<span class="caption">Listagem 16-7: Movendo `tx` para uma thread gerada e enviando 
"hi" (oi)</span>

Again, we’re using `thread::spawn` to create a new thread and then using `move`
to move `tx` into the closure so the spawned thread owns `tx`. The spawned
thread needs to own the transmitting end of the channel to be able to send
messages through the channel.

Novamente, estamos usando `thread::spawn` para criar uma nova thread e, em 
seguida, usando `move` para mover `tx` para closure, de modo que a thread gerada 
possua `tx`. A thread gerada precisa possuir a extremidade de transmissão do 
canal para poder enviar mensagens através do canal.

The transmitting end has a `send` method that takes the value we want to send.
The `send` method returns a `Result<T, E>` type, so if the receiving end has
already been dropped and there’s nowhere to send a value, the send operation
will return an error. In this example, we’re calling `unwrap` to panic in case
of an error. But in a real application, we would handle it properly: return to
Chapter 9 to review strategies for proper error handling.

O fim da transmissão possui um método `send` que assume o valor que queremos enviar. 
O método `send` retorna um tipo `Result <T, E>`; portanto, se o fim do recebimento já 
tiver sido descartado e não houver lugar para enviar um valor, a operação de envio 
retornará um erro. Neste exemplo, chamamos `unwrap` para entrar em pânico (panic) 
em caso de erro. Mas em uma aplicação real, nós a trataríamos adequadamente: retorne 
ao Capítulo 9 para revisar estratégias para o tratamento adequado de erros.

In Listing 16-8, we’ll get the value from the receiving end of the channel in
the main thread. This is like retrieving the rubber duck from the water at the
end of the river or like getting a chat message:

Na Listagem 16-8, obteremos o valor da extremidade receptora do canal na 
thread principal. É como recuperar o pato de borracha da água no final do rio 
ou receber uma mensagem de bate-papo:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

<span class="caption">Listing 16-8: Receiving the value “hi” in the main thread
and printing it</span>

<span class="caption">Listagem 16-8: Recebendo o valor “hi” (oi) na thread principal
e imprimi-lo</span>

The receiving end of a channel has two useful methods: `recv` and `try_recv`.
We’re using `recv`, short for *receive*, which will block the main thread’s
execution and wait until a value is sent down the channel. Once a value is
sent, `recv` will return it in a `Result<T, E>`. When the sending end of the
channel closes, `recv` will return an error to signal that no more values will
be coming.

A extremidade receptora de um canal possui dois métodos úteis: `recv` e `try_recv`. 
Estamos usando `recv`, abreviação de *receive* (receber), que bloqueará a execução da 
thread principal e aguardará até que um valor seja enviado pelo canal. Depois que um 
valor é enviado, `recv` retornará em um `Result<T, E>`. Quando o final do envio do canal 
fechar, `recv` retornará um erro para sinalizar que não haverá mais valores.

The `try_recv` method doesn’t block, but will instead return a `Result<T, E>`
immediately: an `Ok` value holding a message if one is available and an `Err`
value if there aren’t any messages this time. Using `try_recv` is useful if
this thread has other work to do while waiting for messages: we could write a
loop that calls `try_recv` every so often, handles a message if one is
available, and otherwise does other work for a little while until checking
again.

O método `try_recv` não bloqueia, mas, em vez disso, retorna um `Result<T, E>` 
imediatamente: um valor `Ok` contendo uma mensagem, se uma estiver disponível, e 
um valor `Err` se não houver nenhuma mensagem desta vez. O uso de `try_recv` é útil
se essa thread tiver outro trabalho a ser feito enquanto aguarda mensagens: poderíamos 
escrever um loop que chame `try_recv` de vez em quando, manipulará uma mensagem se 
houver uma disponível e, caso não houver mensagens funcionará por um tempo até verificar novamente.

We’ve used `recv` in this example for simplicity; we don’t have any other work
for the main thread to do other than wait for messages, so blocking the main
thread is appropriate.

Usamos `recv` neste exemplo para simplificar; não temos outro trabalho para a thread 
principal, a não ser aguardar mensagens, portanto, o bloqueio da thread principal é apropriado.

When we run the code in Listing 16-8, we’ll see the value printed from the main
thread:

Quando executamos o código na Listagem 16-8, veremos o valor impresso na thread principal:

```text
Got: hi
```

Perfect!

Perfeito!

### Channels and Ownership Transference
### Canais e Transferência de Ownership

The ownership rules play a vital role in message sending because they help us
write safe, concurrent code. Preventing errors in concurrent programming is the
advantage we get by making the trade-off of having to think about ownership
throughout our Rust programs. Let’s do an experiment to show how channels and
ownership work together to prevent problems: we’ll try to use a `val` value in
the spawned thread *after* we’ve sent it down the channel. Try compiling the
code in Listing 16-9:

As regras de ownership desempenham um papel vital no envio de mensagens, porque nos 
ajudam a escrever código simultâneo e seguro. Evitar erros na programação simultânea é a 
vantagem que obtemos ao compensar ter que pensar em ownership em todos os nossos programas 
Rust. Vamos fazer um experimento para mostrar como os canais e ownership trabalham em conjunto 
para evitar problemas: tentaremos usar um valor `val` no segmento gerado *depois* o enviaremos
pelo canal. Tente compilar o código na Listagem 16-9:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust,ignore
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val);
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

<span class="caption">Listing 16-9: Attempting to use `val` after we’ve sent it
down the channel</span>

<span class="caption">Listagem 16-9: Tentativa de usar `val` depois que o enviamos pelo canal</span>

Here, we try to print `val` after we’ve sent it down the channel via `tx.send`.
Allowing this would be a bad idea: once the value has been sent to another
thread, that thread could modify or drop it before we try to use the value
again. Potentially, the other thread's modifications could cause errors or
unexpected results due to inconsistent or nonexistent data. However, Rust gives
us an error if we try to compile the code in Listing 16-9:

Aqui, tentamos imprimir `val` depois que o enviamos pelo canal via `tx.send`. 
Permitir isso seria uma péssima idéia: depois que o valor for enviado para outra thread, 
essa thread poderá modificá-lo ou descartá-lo antes de tentarmos usá-lo novamente. Potencialmente, 
as modificações da outra thread podem causar erros ou resultados inesperados devido a dados 
inconsistentes ou inexistentes. No entanto, Rust nos dá um erro se tentarmos compilar o código na Listagem 16-9:

```text
error[E0382]: use of moved value: `val`
  --> src/main.rs:10:31
   |
9  |         tx.send(val).unwrap();
   |                 --- value moved here
10 |         println!("val is {}", val);
   |                               ^^^ value used here after move
   |
   = note: move occurs because `val` has type `std::string::String`, which does
not implement the `Copy` trait
```

Our concurrency mistake has caused a compile time error. The `send` function
takes ownership of its parameter, and when the value is moved, the receiver
takes ownership of it. This stops us from accidentally using the value again
after sending it; the ownership system checks that everything is okay.

Nosso erro de concorrência causou um erro de tempo de compilação. 
A função `send` assume ownership de seu parâmetro e, quando o valor é movido, 
o receptor assume a ownership. Isso nos impede de usar o valor acidentalmente novamente 
após enviá-lo; o sistema de ownership verifica se está tudo bem.

### Sending Multiple Values and Seeing the Receiver Waiting
### Enviando Vários Valores e Vendo o Receptor Aguardando

The code in Listing 16-8 compiled and ran, but it didn’t clearly show us that
two separate threads were talking to each other over the channel. In Listing
16-10 we’ve made some modifications that will prove the code in Listing 16-8 is
running concurrently: the spawned thread will now send multiple messages and
pause for a second between each message:

O código da Listagem 16-8 compilou e executou, mas não nos mostrou claramente que 
duas threads separadas estavam conversando pelo canal. Nas Listagens 16-10, fizemos 
algumas modificações que provam que o código da Listagem 16-8 está sendo executado 
simultaneamente: a thread gerada agora enviará várias mensagens e fará uma pausa 
por um segundo entre cada mensagem:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

<span class="caption">Listing 16-10: Sending multiple messages and pausing
between each one</span>

<span class="caption">Listagem 16-10: Enviando várias mensagens e 
pausando entre cada uma delas</span>

This time, the spawned thread has a vector of strings that we want to send to
the main thread. We iterate over them, sending each individually, and pause
between each by calling the `thread::sleep` function with a `Duration` value of
one second.

Desta vez, a thread gerada (spawn) possui um vetor de strings que queremos enviar para a 
thread principal. Nós iteramos sobre eles, enviando cada um individualmente e pausamos 
entre eles chamando a função `thread::sleep` com um valor de `Duration` de um segundo.

In the main thread, we’re not calling the `recv` function explicitly anymore:
instead, we’re treating `rx` as an iterator. For each value received, we’re
printing it. When the channel is closed, iteration will end.

Na thread principal, não estamos mais chamando explicitamente a função `recv`: 
em vez disso, estamos tratando `rx` como um iterador. Para cada valor recebido, 
estamos imprimindo. Quando o canal é fechado, a iteração termina.

When running the code in Listing 16-10, you should see the following output
with a one second pause in between each line:

Ao executar o código na Lista 16-10, você deverá ver a seguinte saída com uma 
pausa de um segundo entre cada linha:

```text
Got: hi
Got: from
Got: the
Got: thread
```

Because we don’t have any code that pauses or delays in the `for` loop in the
main thread, we can tell that the main thread is waiting to receive values from
the spawned thread.

Como não temos nenhum código que pausa ou atrasa o loop `for` na thread principal, 
podemos dizer que a thread principal está aguardando para receber valores da thread gerada (spawn).

### Creating Multiple Producers by Cloning the Transmitter
### Criando Vários Produtores ao Clonar o Transmissor

Earlier we mentioned that `mpsc` was an acronym for *multiple* *producer,
single consumer*. Let’s put `mpsc` to use and expand the code in Listing 16-10
to create multiple threads that all send values to the same receiver. We can do
so by cloning the transmitting half of the channel, as shown in Listing 16-11:

Mencionamos anteriormente que `mpsc` era um acrônimo para *multiple producer,
single consumer* (multiplo produtor, consumidor único). Vamos colocar o `mpsc` para 
usar e expandir o código na Listagem 16-10 para criar várias threads que todas enviam 
valores para o mesmo receptor. Podemos fazer isso clonando a parte transmissora do canal, 
conforme mostrado na Listagem 16-11:

<span class="filename">Filename: src/main.rs</span>

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
# use std::thread;
# use std::sync::mpsc;
# use std::time::Duration;
#
# fn main() {
// --snip--

let (tx, rx) = mpsc::channel();

let tx1 = mpsc::Sender::clone(&tx);
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx {
    println!("Got: {}", received);
}

// --snip--
# }
```

<span class="caption">Listing 16-11: Sending multiple messages from multiple
producers</span>

<span class="caption">Listagem 16-11: Enviando várias mensagens de vários 
produtores</span>

This time, before we create the first spawned thread, we call `clone` on the
sending end of the channel. This will give us a new sending handle we can pass
to the first spawned thread. We pass the original sending end of the channel to
a second spawned thread. This gives us two threads, each sending different
messages to the receiving end of the channel.

Desta vez, antes de criarmos o primeira thread gerada (spawn), chamamos `clone` no final do 
canal de envio. Isso nos dará uma novo handle de envio que podemos passar para a primeira 
thread gerada. Passamos o final de envio original do canal para um segundo segmento gerado. 
Isso nos dá duas threads, cada uma enviando mensagens diferentes para a extremidade receptora do canal.

When you run the code, you’ll *probably* see output like this:

Quando você executa o código, *provavelmente* verá uma saída como esta:

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

You might see the values in another order; it depends on your system. This is
what makes concurrency interesting as well as difficult. If you experiment with
`thread::sleep`, giving it various values in the different threads, each run
will be more non-deterministic and create different output each time.

Você pode ver os valores em outra ordem; isso depende do seu sistema. É isso que torna 
a concorrência interessante e difícil. Se você experimentar o `thread::sleep`, 
fornecendo vários valores nas diferentes threads, cada execução será não determinística 
e criará uma saída diferente a cada vez.

Now that we’ve looked at how channels work, let’s look at a different method of
concurrency.

Agora que vimos como os canais funcionam, vamos ver um método diferente de concorrência.
