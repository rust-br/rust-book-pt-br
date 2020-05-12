## Usando Threads para Executar Código Simultaneamente

Na maioria dos sistemas operacionais atuais, o código de um programa é executado em um
*processo*, e o sistema operacional gerencia vários processos ao mesmo tempo. Dentro do seu 
programa, você também pode ter partes independentes que são executadas simultaneamente. O 
recurso que executa essas partes independentes é chamado *threads*.

Dividir a computação em seus programas em várias threads pode melhorar o desempenho 
porque o programa executa várias tarefas ao mesmo tempo, mas também adiciona complexidade. 
Como as threads podem ser executadas simultaneamente, não há garantia inerente sobre a ordem 
em que partes do seu código em diferentes threads serão executadas. Isso pode levar a problemas, como:

* Condições de corrida, em que as threads acessam dados ou recursos em uma ordem inconsistente
* Deadlocks, em que duas threads aguardam o término do uso de um recurso que a outra thread possui, 
impedindo que ambas as threads continuem
* Erros que só acontecem em determinadas situações e são difíceis de reproduzir e corrigir de forma confiável

Rust tenta atenuar os efeitos negativos do uso de threads. Programação em um contexto 
multithread ainda requer cuidadosa reflexão e requer um código de estrutura diferente dos 
programas executados em uma única thread.

Linguagens de programação implementam threads de maneiras diferentes. Muitos sistemas operacionais
fornecem uma API para criar novas threads. Este modelo onde uma linguagem chama as APIs do 
sistema operacional para criar threads às vezes é chamado *1:1*, uma thread do sistema operacional 
por uma thread de linguagem.

Muitas linguagens de programação fornecem sua própria implementação especial de threads.
As threads fornecidas pela linguagem de programação são conhecidas como threads *verdes* (green) 
e as linguagens que usam esses threads verdes irão executá-las num contexto diferente das 
threads do sistema operacional. Por esse motivo, o modelo verde é chamado *M:N*: `M` threads 
verdes por `N` threads do sistema operacional, onde `M` e `N` não são necessariamente o mesmo número.

Cada modelo tem suas próprias vantagens e trade-offs (compensações), e o trade-off mais importante 
para Rust é o suporte ao runtime (tempo de execução). Runtime é um termo confuso e pode ter 
significados diferentes em diferentes contextos.

Nesse contexto, por *runtime*, queremos dizer código incluído pela linguagem em cada 
binário. Esse código pode ser grande ou pequeno, dependendo da linguagem, mas toda 
linguagem que não seja assembly terá alguma quantidade de código de runtime. Por essa razão, 
coloquialmente quando as pessoas dizem que uma linguagem "não runtime", geralmente
significa "runtime pequeno". Os runtimes menores têm menos recursos, mas têm a
vantagem de resultar em binários menores, o que facilita a combinação da linguagem com 
outras linguagens em mais contextos. Embora muitas linguagens sejam ok com o aumento do 
tamanho do runtime em troca de mais recursos, Rust necessita, praticamente, não ter runtime 
e não pode comprometer a capacidade de chamar C para manter o desempenho.

O modelo M:N requer um tempo maior de runtime da linguagem para gerenciar threads. Como tal, 
a biblioteca padrão Rust fornece apenas uma implementação de threading 1:1. Como Rust é uma linguagem 
de baixo nível, existem crates (caixas) que implementam threading M:N, se você preferir 
trocar custos indiretos por aspectos, como maior controle sobre quais threads são executados 
e custos mais baixos de alternância de contexto, por exemplo.

Agora que definimos threads em Rust, vamos explorar como usar a API relacionada a thread 
fornecida pela biblioteca padrão.

### Criando uma Nova Thread com `spawn` 

Para criar uma nova thread, chamamos a função `thread::spawn` e passamos para uma closure 
(falamos sobre closure no Capítulo 13) contendo o código que queremos executar na nova thread. 
O exemplo na Listagem 16-1 imprime texto da thread principal e texto da nova thread:

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

As chamadas para `thread::sleep` forçam a thread a interromper sua execução por um 
curto período, o que permite que uma thread diferente seja executada. As threads 
provavelmente se revezarão, mas isso não é garantido: depende de como o sistema 
operacional agenda as threads. Nesta execução, a thread principal imprimiu primeiro, 
mesmo que a instrução de impressão da thread spawned apareça primeiro no código. E 
mesmo que disséssemos que a linha gerada imprimisse até que `i` seja 9, ela só chegou 
a 5, no momento que a thread principal foi desligada.

Se você executar esse código e vir apenas a saída da thread principal ou não houver 
sobreposição, tente aumentar os números nos intervalos para criar mais oportunidades 
para o sistema operacional alternar entre as threads.

### Aguardando a Conclusão de Todas as Threads Usando Identificadores `join`

O código da Listagem 16-1 interrompe a thread spawned prematuramente devido ao 
término da thread principal, mas não há garantia de que a thread spawned possa 
ao menos executar. O motivo é que não há garantia na ordem em que as threads 
são executadas!

Podemos corrigir o problema da thread spawned não funcionar, ou não funcionar completamente, 
salvando o valor de retorno de `thread::spawn` em uma variável. O tipo de retorno de 
`thread::spawn` é `JoinHandle`. Um `JoinHandle` é um valor de propriedade que, quando chamamos 
o método `join`, aguardará seu término. A Listagem 16-2 mostra como usar o `JoinHandle` da 
thread que criamos na Listagem 16-1 e chame `join` para garantir que o thread spawned 
termine antes do fim da execução do `main`:

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

<span class="caption">Listagem 16-2: Salvando um `JoinHandle` da `thread::spawn`
para garantir que a thread seja executada até a conclusão</span>

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

As duas threads continuam alternando, mas a thread principal espera por causa da 
chamada para `handle.join()` e não termina até que o thread spawned seja finalizada.

Mas vamos ver o que acontece quando movemos `handle.join()` antes do 
loop `for` em `main`, assim:

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

Pense bem, como um pequeno detalhe, onde chamar `join` pode afetar a execução 
ou não de suas threads executadas ao mesmo tempo.

### Usando Closure `move` com Threads

O closure `move`, que mencionamos brevemente no Capítulo 13, é freqüentemente usado 
junto com `thread::spawn` porque nos permite usar dados de uma thread em outra thread.

No capítulo 13, dissemos que “se queremos forçar a closure a ter ownership dos valores que ela 
usa no ambiente, podemos usar a palavra-chave `move` antes da lista de parâmetros. Essa técnica 
é útil principalmente ao passar dados de uma closure para uma thread e a thread ter ownership sob 
esses dados".

Agora que estamos criando novas threads, falaremos sobre a captura de valores 
em closures.

Observe na Listagem 16-1 que a closure que passamos para `thread::spawn` não exige 
argumentos: não estamos usando dados da thread principal no código da thread spawned. 
Para fazer isso, a closure da thread spawned deve capturar os valores necessários. A 
Listagem 16-3 mostra uma tentativa de criar um vetor na thread principal e usá-la na 
thread spawned. No entanto, isso ainda não funcionará, como você verá em um momento:

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

<span class="caption">Lista 16-3: Tentativa de usar um vetor criado pela
thread principal em outra thread</span>

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

Rust *infere* como capturar `v` e porque `println!` apenas precisa de uma referência
para `v`, a closure tenta emprestar `v`. No entanto, há um problema: Rust não pode
dizer quanto tempo a thread gerada será executada, então ele não sabe por quanto 
tempo a referência para `v` será válida.

A Listagem 16-4 fornece um cenário com maior probabilidade de ter uma referência para `v`
que não será válida:

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

<span class="caption">Listagem 16-4: Uma thread com uma closure que tenta
capturar uma referência ao `v` de um thread principal que drops (descarte) o `v`</span>

Se pudéssemos executar esse código, existe a possibilidade de a thread spawned ser 
imediatamente colocada em segundo plano sem ser executada. A thread spawned tem dentro uma 
referência ao `v`, mas a thread principal drops (descarta) o `v` imediatamente, usando 
a função `drop` que discutimos no Capítulo 15. Então, quando a thread spawned começa a 
ser executada, `v` não é mais válido, portanto, uma referência a ela também é inválida. Ah não!

Para corrigir o erro do compilador na Listagem 16-3, podemos usar os conselhos 
da mensagem de erro:

```text
help: to force the closure to take ownership of `v` (and any other referenced
variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ^^^^^^^
```

Ao adicionar a palavra-chave `move` antes da closure, forçamos a closure a ter o
ownership dos valores que está usando, em vez de permitir que Rust deduza que deve 
emprestar os valores. A modificação na Listagem 16-3 mostrada na Listagem
16-5 serão compilados e executados conforme pretendemos:

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
para ter ownership dos valores que usa</span>

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

As regras de ownership Rust nos salvaram de novo! Ocorreu um erro no código da Listagem 
16-3 porque Rust estava sendo conservador e apenas emprestou `v` para a thread, 
o que significava que a thread principal teoricamente poderia invalidar a referência 
da thread spawned. Ao dizer para Rust em mudar a ownership de `v` para a thread spawned, 
estamos garantindo ao Rust que a thread principal não use mais o `v`. Se alterarmos 
a Listagem 16-4 da mesma maneira, violaremos as regras de ownership ao tentarmos usar `v`
na thread principal. A palavra-chave `move` substitui o padrão conservador de 
ownership Rust; não nos deixa violar regras de ownership.

Com um entendimento básico das threads e da API de thread, vejamos o que podemos 
*fazer*  com as threads.