## Transferir Dados Entre Threads Passando Mensagens

Uma abordagem cada vez mais popular para garantir a concorrência segura é a 
*message passing* (passagem de mensagens), na qual threads ou atores se comunicam enviando mensagens 
contendo dados. Aqui está a idéia em um slogan da documentação da linguagem Go:

> Não se comunique compartilhando memória; em vez disso, compartilhe memória
> comunicando.
>
> --[Effective Go](http://golang.org/doc/effective_go.html)

Uma ferramenta importante que Rust possui para realizar o envio de mensagens 
concorrentes é o *canal* (channel), um conceito de programação que a biblioteca 
padrão do Rust fornece uma implementação. Você pode imaginar um canal na programação 
como um canal de água, como um córrego ou um rio. Se você colocar algo como um pato 
de borracha ou um barco em um riacho, ele viajará rio abaixo até o final do rio.

Um canal na programação tem duas metades: um transmissor e um receptor. 
A metade do transmissor é o local a montante, onde colocamos os patos de borracha 
no rio, e a metade do receptor é o local onde o pato de borracha termina a jusante. 
Uma parte do nosso código chama métodos no transmissor com os dados que queremos enviar, 
e outra parte verifica o recebimento de mensagens. Diz-se que um canal está *fechado* (closed) se 
a metade do transmissor ou do receptor cair.

Aqui, trabalharemos em um programa que tenha uma thread para gerar valores e enviá-los 
por um canal, e outra thread que receberá os valores e os imprimirá. Enviaremos valores 
simples entre threads usando um canal para ilustrar o recurso. Quando você estiver familiarizado 
com a técnica, poderá usar canais para implementar um sistema de bate-papo (chat) ou um sistema em 
que muitas threads realizam partes de um cálculo e enviam as partes para um thread que agrega os
resultados.

Primeiro, na Listagem 16-6, criaremos um canal, mas não faremos nada com ele.
Observe que isso ainda não será compilado porque Rust não pode dizer que tipo de valores 
queremos enviar pelo canal:

<span class="filename">Nome do arquivo: src/main.rs</span>

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
#     tx.send(()).unwrap();
}
```

<span class = "caption">Listagem 16-6: Criando um canal e atribuindo as 
duas metades para `tx` e` rx`</span>

Criamos um novo canal usando a função `mpsc::channel`; `mpsc` significa 
*multiple producer, single consumer* (produtor múltiplo, consumidor único). Em resumo, 
a maneira como a biblioteca padrão Rust implementa canais significa que um canal pode 
ter várias extremidades *de envio* que produzem valores, mas apenas uma extremidade 
*de recebimento* que consome esses valores. Imagine vários rios e riachos fluindo juntos 
para um grande rio: tudo o que for enviado por qualquer um dos riachos terminará em um 
rio no final. Vamos começar com um único produtor por enquanto, mas adicionaremos vários 
produtores quando este exemplo funcionar.

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

A função `mpsc::channel` retorna uma tupla, cujo primeiro elemento é o terminal de envio 
e o segundo elemento é o terminal de recebimento. As abreviaturas `tx` e `rx` são tradicionalmente 
usadas em muitos campos para *trasmissor* e *receptor* respectivamente, portanto, 
nomeamos nossas variáveis como tal para indicar cada extremidade. Estamos usando uma declaração 
`let` com um padrão que destrói as tuplas; discutiremos o uso de padrões nas instruções `let` e 
a destruição no Capítulo 18. Usar uma declaração` let` dessa maneira é uma abordagem 
conveniente para extrair as partes da tupla retornadas pelo `mpsc::channel`.

Vamos mover a extremidade de transmissão para uma thread gerada (spawn) e enviar uma 
string para que a thread gerada se comunique com a thread principal, conforme 
mostrado na Listagem 16-7. É como colocar um pato de borracha no rio a montante 
ou enviar uma mensagem de bate-papo de uma thread a outra:

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

<span class="caption">Listagem 16-7: Movendo `tx` para uma thread gerada e enviando 
"hi" (oi)</span>

Novamente, estamos usando `thread::spawn` para criar uma nova thread e, em 
seguida, usando `move` para mover `tx` para closure, de modo que a thread gerada 
possua `tx`. A thread gerada precisa possuir a extremidade de transmissão do 
canal para poder enviar mensagens através do canal.

O fim da transmissão possui um método `send` que assume o valor que queremos enviar. 
O método `send` retorna um tipo `Result <T, E>`; portanto, se o fim do recebimento já 
tiver sido descartado e não houver lugar para enviar um valor, a operação de envio 
retornará um erro. Neste exemplo, chamamos `unwrap` para entrar em pânico (panic) 
em caso de erro. Mas em uma aplicação real, nós a trataríamos adequadamente: retorne 
ao Capítulo 9 para revisar estratégias para o tratamento adequado de erros.

Na Listagem 16-8, obteremos o valor da extremidade receptora do canal na 
thread principal. É como recuperar o pato de borracha da água no final do rio 
ou receber uma mensagem de bate-papo:

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

<span class="caption">Listagem 16-8: Recebendo o valor “hi” (oi) na thread principal
e imprimi-lo</span>

A extremidade receptora de um canal possui dois métodos úteis: `recv` e `try_recv`. 
Estamos usando `recv`, abreviação de *receive* (receber), que bloqueará a execução da 
thread principal e aguardará até que um valor seja enviado pelo canal. Depois que um 
valor é enviado, `recv` retornará em um `Result<T, E>`. Quando o final do envio do canal 
fechar, `recv` retornará um erro para sinalizar que não haverá mais valores.

O método `try_recv` não bloqueia, mas, em vez disso, retorna um `Result<T, E>` 
imediatamente: um valor `Ok` contendo uma mensagem, se uma estiver disponível, e 
um valor `Err` se não houver nenhuma mensagem desta vez. O uso de `try_recv` é útil
se essa thread tiver outro trabalho a ser feito enquanto aguarda mensagens: poderíamos 
escrever um loop que chame `try_recv` de vez em quando, manipulará uma mensagem se 
houver uma disponível e, caso não houver mensagens funcionará por um tempo até verificar novamente.

Usamos `recv` neste exemplo para simplificar; não temos outro trabalho para a thread 
principal, a não ser aguardar mensagens, portanto, o bloqueio da thread principal é apropriado.

Quando executamos o código na Listagem 16-8, veremos o valor impresso na thread principal:

```text
Got: hi
```

Perfeito!

### Canais e Transferência de Ownership

As regras de ownership desempenham um papel vital no envio de mensagens, porque nos 
ajudam a escrever código simultâneo e seguro. Evitar erros na programação simultânea é a 
vantagem que obtemos ao compensar ter que pensar em ownership em todos os nossos programas 
Rust. Vamos fazer um experimento para mostrar como os canais e ownership trabalham em conjunto 
para evitar problemas: tentaremos usar um valor `val` no segmento gerado *depois* o enviaremos
pelo canal. Tente compilar o código na Listagem 16-9:

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

<span class="caption">Listagem 16-9: Tentativa de usar `val` depois que o enviamos pelo canal</span>

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

Nosso erro de concorrência causou um erro de tempo de compilação. 
A função `send` assume ownership de seu parâmetro e, quando o valor é movido, 
o receptor assume a ownership. Isso nos impede de usar o valor acidentalmente novamente 
após enviá-lo; o sistema de ownership verifica se está tudo bem.

### Enviando Vários Valores e Vendo o Receptor Aguardando

O código da Listagem 16-8 compilou e executou, mas não nos mostrou claramente que 
duas threads separadas estavam conversando pelo canal. Nas Listagens 16-10, fizemos 
algumas modificações que provam que o código da Listagem 16-8 está sendo executado 
simultaneamente: a thread gerada agora enviará várias mensagens e fará uma pausa 
por um segundo entre cada mensagem:

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

<span class="caption">Listagem 16-10: Enviando várias mensagens e 
pausando entre cada uma delas</span>

Desta vez, a thread gerada (spawn) possui um vetor de strings que queremos enviar para a 
thread principal. Nós iteramos sobre eles, enviando cada um individualmente e pausamos 
entre eles chamando a função `thread::sleep` com um valor de `Duration` de um segundo.

Na thread principal, não estamos mais chamando explicitamente a função `recv`: 
em vez disso, estamos tratando `rx` como um iterador. Para cada valor recebido, 
estamos imprimindo. Quando o canal é fechado, a iteração termina.

Ao executar o código na Lista 16-10, você deverá ver a seguinte saída com uma 
pausa de um segundo entre cada linha:

```text
Got: hi
Got: from
Got: the
Got: thread
```

Como não temos nenhum código que pausa ou atrasa o loop `for` na thread principal, 
podemos dizer que a thread principal está aguardando para receber valores da thread gerada (spawn).

### Criando Vários Produtores ao Clonar o Transmissor

Mencionamos anteriormente que `mpsc` era um acrônimo para *multiple producer,
single consumer* (multiplo produtor, consumidor único). Vamos colocar o `mpsc` para 
usar e expandir o código na Listagem 16-10 para criar várias threads que todas enviam 
valores para o mesmo receptor. Podemos fazer isso clonando a parte transmissora do canal, 
conforme mostrado na Listagem 16-11:

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

<span class="caption">Listagem 16-11: Enviando várias mensagens de vários 
produtores</span>

Desta vez, antes de criarmos o primeira thread gerada (spawn), chamamos `clone` no final do 
canal de envio. Isso nos dará uma novo handle de envio que podemos passar para a primeira 
thread gerada. Passamos o final de envio original do canal para um segundo segmento gerado. 
Isso nos dá duas threads, cada uma enviando mensagens diferentes para a extremidade receptora do canal.

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

Você pode ver os valores em outra ordem; isso depende do seu sistema. É isso que torna 
a concorrência interessante e difícil. Se você experimentar o `thread::sleep`, 
fornecendo vários valores nas diferentes threads, cada execução será não determinística 
e criará uma saída diferente a cada vez.

Agora que vimos como os canais funcionam, vamos ver um método diferente de concorrência.