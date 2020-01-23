## Usando threads para executar código simultaneamente

Na maioria dos sistemas operacionais atuais, o código de um programa é executado em um *processo* e
o sistema operacional gerencia vários processos ao mesmo tempo. Dentro do seu programa, você também pode ter
partes independentes que são executadas simultaneamente.
O recurso que executa essas partes independentes é chamada de *threads*.

Dividir o código de seu programa em várias threads pode melhorar o desempenho, pois o programa executa
várias tarefas ao mesmo tempo, mas também adiciona complexidade. Como as threads podem ser executados simultaneamente,
não há garantia inerente sobre a ordem em que partes do seu código em diferentes threads serão executadas. Isso pode
levar a problemas, como:

* Race conditions, em que as threads acessam dados ou recursos em uma ordem inconsistente;
* Deadlocks, onde duas threads estão esperando uma pela outra terminar de usar um recurso que a outra thread já possui,
impedindo que ambas threads continuem;
* Bugs, erros que acontecem apenas em determinadas situações e são difíceis de reproduzir e corrigir de forma confiável.

O Rust tenta atenuar os efeitos negativos do uso de threads. A programação em um contexto multithread ainda requer
cuidadosa reflexão e requer uma estrutura de código diferente dos programas executados em uma única thread.

Linguagens de programação implementam threads de algumas maneiras diferentes. Muitos sistemas operacionais fornecem
uma API para criar novas threads. Esse modelo em que uma linguagem chama as APIs do sistema operacional para criar threads
é chamado às vezes *1:1*, uma thread do sistema operacional por uma thread de linguagem.

Muitas linguagens de programação fornecem sua própria implementação especial de threads.
Os threads fornecidos pela linguagem de programação são conhecidos como threads *verdes*, e as linguagens que usam esses
threads verdes os executarão no contexto de um número diferente de threads do sistema operacional. Por esse motivo, o modelo
com thread verde é chamado de modelo *M:N*: threads verdes são `M` por threads do sistema operacional `N`, em que `M` e `N` não
são necessariamente o mesmo número.

Cada modelo tem suas próprias vantagens e compensações, e a mais importante para o Rust é o suporte ao tempo de execução.
Tempo de execução é um termo confuso e pode ter significados diferentes em contextos diferentes.

Nesse contexto, por *tempo de execução* queremos dizer código incluído na linguagem em todos os binários. Esse código
pode ser grande ou pequeno, dependendo do linguagem, mas toda linguagem que não seja de assembly terá uma certa quantidade
de código de tempo de execução. Por esse motivo, coloquialmente quando as pessoas dizem que uma linguagem “não tem tempo de
execução”, elas geralmente significam “tempo de execução pequeno”. Os tempos de execução menores têm menos recursos, mas
têm a vantagem de resultar em binários menores, o que facilita a combinação da linguagem com outras linguagens. em mais contextos.
Embora muitas linguagens estejam de acordo em aumentar o tamanho do tempo de execução em troca de mais recursos, o Rust precisa
quase não ter tempo de execução e não pode comprometer a possibilidade de chamar o C para manter o desempenho.

O modelo M:N da thread verde requer um tempo de execução de uma linguagem de alto nível para gerenciar threads. Como tal, a biblioteca
padrão Rust fornece apenas uma implementação de segmentação 1:1. Como o Rust é uma linguagem de baixo nível, existem caixas
que implementam o thread M:N se você preferir trocar custos indiretos por aspectos como maior controle sobre quais
threads são executados quando e custos mais baixos de alternância de contexto, por exemplo.

Agora que definimos threads no Rust, vamos explorar como usar a API relacionada a thread fornecida pela biblioteca padrão.

### Criando uma nova thread com `spawn`

Para criar uma nova thread, chamamos a função `thread::spawn` e passamos numa closure (falamos sobre closures no
Capítulo 13) contendo o código que queremos executar no novo thread. O exemplo na `Listagem 16-1` imprime algum texto de um
thread principal e outro texto de um novo thread:

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

<span class="caption">Listagem 16-1: Criando uma nova thread para imprimir uma coisa
enquanto a thread principal imprime outra coisa</span>

Observe que, com esta função, a nova thread será interrompida quando a thread principal terminar,
independentemente de ter terminado ou não a execução. A saída deste programa pode ser um pouco diferente todas as vezes,
mas será semelhante à este abaixo:

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
Ao chamar `thread::sleep`, forçará uma thread a interromper sua execução por um curto período,
o que permite que um thread diferente seja executado. Os encadeamentos provavelmente se revezarão,
mas isso não é garantido: depende de como o sistema operacional agendar as threads. Nesta execução,
a thread principal foi impressa primeiro, mesmo que a instrução de impressão da thread "spawnada" apareça primeiro no código.
E mesmo que tenhamos dito a thread "spawnada" para imprimir até que `i` seja 9, ele só chegou a 5 antes que a thread
principal finalize.

Se você executar esse código e vê apenas a saída da thread principal ou caso não houver sobreposição, tente aumentar
os números nos intervalos para criar mais oportunidades para o sistema operacional alternar entre as threads.

### Aguardando a conclusão de todas as threads usando o `join`

O código na Listagem 16-1 não apenas interrompe a thread "spawnada" prematuramente na maioria das vezes devido ao término
da thread principal, mas não há garantia de que a thread "spawnada" finalize. O motivo é que não há garantia na ordem em
que as threads serão executadas!

Podemos corrigir o problema da thread "spawnada" não funcionar, ou não funcionar completamente, salvando o valor de retorno
da `thread::spawn` em uma variável.
O tipo de retorno de `thread::spawn` é `JoinHandle`. Um `JoinHandle` é um valor de propriedade que, quando chamamos o
método` join`, aguardará seu término. A Listagem 16-2 mostra como usar o `JoinHandle` da thread que criamos na Listagem 16-1
e chame` join` para garantir que a thread "spawnada" termine antes da saída do `main`:

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

<span class="caption">Listagem 16-2: Salvando um `JoinHandle` de `thread::spawn`
para garantir que a thread seja executada até a conclusão</span>

Ao chamar `join`, ele bloqueia a thread atualmente em execução até a thread terminar.
*Bloquear* uma thread significa que a thread é impedida de executar trabalhos ou sair.
Como colocamos a chamada 'join' após o `for` loop da thread principal, a execução da Listagem 16-2 deve
produzir uma saída semelhante a esta:

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
As duas threads continuam alternando, mas a thread principal espera por causa da chamada para `handle.join()`
e não termina até que a thread "spawnada" seja finalizada.

Mas vamos ver o que acontece quando movemos `handle.join()` antes do `for` loop no `main`, assim:

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

A thread principal aguardará o término da thread "spawnada" e, em seguida, executará seu `for` loop,
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
Pensar em um detalhe tão pequeno como chamar `join` pode afetar a execução ou não de suas threads ao mesmo tempo.

### Usando closure `move` com Threads

A closure `move`, que mencionamos brevemente no Capítulo 13, é freqüentemente usado junto com `thread::spawn` porque nos
permite usar dados de uma thread em outra thread.

No capítulo 13, dissemos que “se queremos forçar a closure a se apropriar dos valores que ele usa no ambiente,
podemos usar a palavra-chave `move` antes da lista de parâmetros. Essa técnica é útil principalmente ao passar uma closure
para uma nova thread para mover os dados para que sejam de propriedade da nova thread".

Agora que estamos criando novas threads, falaremos sobre capturar valores em closures.

Observe na Listagem 16-1 que a closure que passamos para `thread::spawn` não exige argumentos: não estamos usando dados
da thread principal no código da thread "spawnada". Para fazer isso, a closure da thread "spawnada" deve capturar os valores
necessários. A Listagem 16-3 mostra uma tentativa de criar um vetor na thread principal e usá-lo na thread "spawnada".
No entanto, isso ainda não funcionará, como você verá em um momento:

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

<span class="caption">Listagem 16-3: Tentando usar um vetor criado pela
thread principal em outra thread</span>

A closure usa `v`, para capturar `v` e torná-lo parte do ambiente da closure. Como a `thread::spawn` executa essa closure
em uma nova thread, devemos poder acessar o `v` dentro dessa nova thread. Mas quando compilamos este exemplo,
obtemos o seguinte erro:

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
Rust *deduz* como capturar `v`, e como  `println!` precisa apenas de uma referência para `v`, a closure tenta
emprestar `v`. No entanto, há um problema: o Rust não sabe por quanto tempo a thread "spawnada" será executada, portanto,
não se sabe se a referência a `v` sempre será válida.

A Listagem 16-4 fornece um cenário com maior probabilidade de ter uma referência a `v` que não será válida:

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

<span class="caption">Listagem 16-4: A thread with a closure that attempts to
capture a reference to `v` from a main thread that drops `v`</span>

Se pudéssemos executar esse código, existe a possibilidade da thread "spawnada" ser imediatamente colocada em segundo plano
sem ser executada. A thread "spawnada" gera uma referência a `v` dentro, mas a thread principal descarta imediatamente `v`,
usando a função `drop` que discutimos no Capítulo 15. Então, quando a thread "spawnada" começa a executar, `v` não é mais
válido, portanto, uma referência a ele também é inválida. Ah não!

Para corrigir o erro do compilador na Listagem 16-3, podemos usar os conselhos da mensagem de erro:

```text
help: to force the closure to take ownership of `v` (and any other referenced
variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ^^^^^^^
```
Ao adicionar a palavra-chave `move` antes da closure, forçamos a closure a se apropriar dos valores que está usando,
em vez de permitir que a Rust deduza que deve emprestar os valores. A modificação da Listagem 16-3 mostrada na Listagem 16-5
será compilada e executada como pretendemos:

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

<span class="caption">Listagem 16-5: Usando a palavra-chave `move` para forçar uma closure
para se apropriar dos valores que usa</span>

O que aconteceria com o código na Listagem 16-4 em que a thread principal chamasse `drop` para usarmos uma
closure `move`? O `move` corrige esse caso? Infelizmente não; receberíamos um erro diferente, porque o que a
Listagem 16-4 está tentando fazer não é permitido por um motivo diferente. Se adicionarmos `move` a closure,
moveríamos `v` para o ambiente da closure e não poderíamos mais chamar `drop` na thread principal.
Em vez disso, obteríamos esse erro do compilador:

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

As regras de propriedade do Rust nos salvaram de novo! Ocorreu um erro no código da Listagem 16-3 porque o Rust
estava sendo conservador e apenas emprestou `v` para a thread, o que significava que a thread principal
teoricamente poderia invalidar a referência da thread "spawnada". Ao dizer ao Rust para mudar a propriedade de `v`
para a thread "spawnada", estamos garantindo ao Rust que a thread principal não use mais o `v`.
Se alterarmos a Listagem 16-4 da mesma maneira, violaremos as regras de propriedade ao tentarmos usar `v`
na thread principal. A palavra-chave `move` substitui o padrão conservador de empréstimos do Rust; não nos permite
violar as regras de propriedade.

Com um entendimento básico das threads e da API da thread, vejamos o que podemos *fazer* com as threads.
