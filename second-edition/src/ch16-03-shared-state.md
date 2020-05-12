## Concorrência de Estado Compartilhado

A passagem de mensagens é uma ótima maneira de lidar com a concorrência, mas não 
é a única. Considere esta parte do slogan da documentação da linguagem 
Go novamente: "comunique-se compartilhando memória".

Como seria a comunicação compartilhando memória? Além disso, por que os entusiastas 
da transmissão de mensagens não o usariam e, em vez disso, fariam o contrário?

De certa forma, os canais em qualquer linguagem de programação são semelhantes à ownership única, 
porque depois de transferir um valor para um canal, você não deve mais usá-lo. A concorrência 
de memória compartilhada é como ownership múltipla: vários segmentos podem acessar o mesmo 
local de memória ao mesmo tempo. Como você viu no Capítulo 15, onde ponteiros inteligentes 
tornaram possível a ownsership múltipla, a ownership múltipla pode adicionar complexidade 
adicional porque esses proprietários diferentes precisam ser gerenciados. O sistema de 
tipos e as regras de ownership Rust ajudam muito a corrigir esse gerenciamento. Por exemplo, 
vejamos as mutexes, uma das primitivas de simultaneidade mais comuns para memória compartilhada.

### Mutexes Permitem Acesso a Dados de Uma Thread por Vez

Um *mutex* é uma abreviação de “exclusão mútua“ (mutual exclusion), permite apenas 
que uma thread acesse alguns dados a qualquer momento. Para acessar os dados em um mutex, 
uma thread deve primeiro sinalizar que deseja acessar, solicitando a aquisição do 
bloqueio (*lock*) do mutex. O bloqueio é uma estrutura de dados que faz parte do mutex que 
rastreia quem atualmente tem acesso exclusivo aos dados. Portanto, descrevemos o mutex 
como *quem guarda* os dados que ele mantém por meio do sistema de bloqueio.

Os mutexes têm a reputação de serem difíceis de usar, porque você precisa se lembrar de duas regras:

1. Você deve tentar adquirir o bloqueio antes de usar os dados.
2. Ao concluir os dados que o mutex protege, você deve desbloquear o mutex 
   para que outras threads possam adquirir o bloqueio.

Para uma metáfora do mundo real de um mutex, imagine um painel de discussão em uma 
conferência com apenas um microfone. Antes que um membro do painel possa falar, ele precisa 
perguntar ou sinalizar que deseja usar o microfone. Quando eles pegam o microfone, eles podem 
conversar pelo tempo que quiserem e, em seguida, entregam o microfone ao próximo membro do 
painel que pede para falar. Se um membro do painel esquecer de entregar o microfone quando 
terminar, ninguém mais poderá falar. Se o gerenciamento do microfone compartilhado der errado, 
o painel não funcionará conforme o planejado!

O gerenciamento de mutexes pode ser incrivelmente difícil de acertar, e é por 
isso que tantas pessoas estão entusiasmadas com os canais. No entanto, graças ao 
sistema de tipos e às regras de ownership Rust, não podemos bloquear e desbloquear incorretamente.

#### A API do `Mutex<T>`

Como um exemplo de como usar um mutex, vamos começar usando um mutex em 
um contexto de thread única, como mostra a Listagem 16-12:

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

<span class="caption">Listagem 16-12: Explorando a API do `Mutex<T>` em um 
contexto de thread única para manter a simplicidade</span>

Como em muitos tipos, criamos `Mutex<T>` usando a função associada `new`. 
Para acessar os dados dentro do mutex, usamos o método `lock` para adquirir 
o bloqueio. Essa chamada bloqueará a thread atual e, portanto, não realizará trabalho 
até que seja a nossa vez de ter o bloqueio.

A chamada para `lock` falharia se outra thread estivesse segurando a 
trava entrasse em pânico. Nesse caso, ninguém jamais conseguiria trancar, 
por isso escolhemos `unwrap` e ter essa thread em pânico se estivermos nessa situação.

Depois de adquirirmos o bloqueio, podemos tratar o valor de retorno, chamado `num` 
nesse caso, como uma referência mutável aos dados internos. O sistema de tipos garante 
que adquirimos um bloqueio antes de usar o valor em `m`:` Mutex<i32>` não é um` i32`, 
portanto *devemos* adquirir o bloqueio para poder usar o valor `i32`. Nós não 
podemos esquecer; o sistema de tipos não nos permitirá acessar o `i32` interno de 
outra forma.

Como você pode suspeitar, o `Mutex<T>` é um ponteiro inteligente. Mais precisamente, 
a chamada para `lock` *retorna* um ponteiro inteligente chamado `MutexGuard`. Este ponteiro 
inteligente implementa `Deref` que aponta para nossos dados internos; o ponteiro inteligente 
também possui uma implementação `Drop` que libera o bloqueio automaticamente quando um `MutexGuard` 
sai do escopo, o que acontece no final do escopo interno da Listagem 16-12. Como resultado, não 
corremos o risco de esquecer de liberar o bloqueio e impedir que o mutex seja usado por outras 
threads, porque o desbloqueio ocorre automaticamente.

Depois de soltar a trava (bloqueio), podemos imprimir o valor mutex e ver que conseguimos alterar 
o `i32` interno para o valor 6.

#### Compartilhando um `Mutex<T>` Entre Várias Threads

Agora, vamos tentar compartilhar um valor entre várias threads usando o `Mutex<T>`. 
Subimos 10 threads e cada uma delas incrementará um valor do contador em 1, para que o 
contador passe de 0 a 10. Observe que os próximos exemplos terão erros de compilador e nós 
usaremos para aprender mais sobre como usar `Mutex<T>` e como Rust nos ajuda a usá-lo corretamente. 
A Listagem 16-13 tem nosso exemplo inicial:

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

<span class="caption">Listagem 16-13: Dez threads, cada uma incrementa o 
contador protegido por um `Mutex<T>`</span>

Estamos criando uma variável `counter` para armazenar um `i32` dentro de um `Mutex<T>`, como 
fizemos na Listagem 16-12. Em seguida, criamos 10 threads mapeando um intervalo de números. Usamos 
`thread::spawn` e damos a todas as threads o mesmo fechamento (closure), aquele que move o contador 
para a thread, adquire um bloqueio no `Mutex<T> ` chamando o método `lock` e, em seguida, adiciona 
1 ao valor no mutex. Quando uma thread termina o fechamento (closure), o `num` sai do escopo e libera a trava 
para que outra thread possa adquiri-lo.

Na thread principal, coletamos todas as alças de junção (join handles), como fizemos na Listagem 16-2, 
e então chamamos `join` em cada uma para garantir que todas as threads terminem. Nesse ponto, 
a thread principal adquirirá o bloqueio e imprimirá o resultado deste programa.

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

A mensagem de erro informa que o valor do `counter` é movido para a closure e, 
em seguida, é capturado quando chamamos `lock`. Essa descrição parece o que queríamos, 
mas não é permitido!

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

Aha! A primeira mensagem de erro indica que o `counter` é movido para a closure da thread 
associada ao `handle`. Esse move está nos impedindo de capturar o `counter`  quando 
tentamos chamar o `lock` e armazenar o resultado em `num2` na segunda thread! Portanto, 
Rust está nos dizendo que não podemos mudar a propriedade do `counter` para vários segmentos. 
Isso foi difícil de ver anteriormente, porque nossas threads estavam em um loop e o Rust não 
pode apontar para threads diferentes em diferentes iterações do loop. Vamos corrigir o erro 
do compilador com um método de multiple-ownership que discutimos no capítulo 15.

#### Múltiplas Ownership com Multiplas Threads

No Capítulo 15, atribuímos um valor a vários owners (proprietários) usando o ponteiro 
inteligente `Rc<T>` para criar um valor contado por referência. Vamos fazer o mesmo aqui 
e ver o que acontece. Vamos agrupar o `Mutex<T>` em `Rc<T>` na Listagem 16-14 e clonar o 
`Rc<T>` antes de mover a ownership para a thread. Agora que vimos os erros, 
também voltaremos a usar o loop `for` e manteremos a palavra-chave `move` com a closure:

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

<span class="caption">Lista 16-14: Tentativa de usar `Rc<T>` para permitir 
que várias threads possuam o `Mutex<T>`</span>

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

Uau, essa mensagem de erro é muita prolixa! Aqui estão algumas partes importantes para 
focar: o primeiro erro diz que `` `std::rc::Rc<std::sync::Mutex<i32>>` cannot be sent 
between threads safely``  (`` `std::rc::Rc<std::sync::Mutex<i32>>` não pode ser 
enviado com segurança entre as threads ``) . A razão para isso está na próxima parte 
importante a ser focada, na mensagem de erro. A mensagem de erro destilada diz `` the
trait bound `Send` is not satisfied ``. Falaremos sobre o `Send` na próxima seção: é uma 
das traits que garante que os tipos que usamos com threads sejam usados em situações concorrentes.

Infelizmente, `Rc<T>` não é seguro para compartilhar threads. Quando `Rc<T>` gerencia 
a contagem de referência, ela é adicionada à contagem de cada chamada para `clonar` e 
subtrai da contagem quando cada clone é descartado. Mas ele não usa nenhuma primitiva 
de concorrência para garantir que as alterações na contagem não sejam interrompidas 
por outra thread. Isso pode levar a contagens erradas - erros sutis que, por sua vez, 
podem levar a vazamentos de memória ou perda de valor antes que terminemos. O que precisamos 
é de um tipo exatamente como `Rc<T>`, mas que faça alterações na referência de contagem de 
maneira segura em threads.

#### Referência Atômica Contando Com `Arc<T>`

Felizmente, `Arc<T>` *é* um tipo como `Rc<T>` que é seguro para uso em
situações concorrentes. O ‘a’ significa *atômico*, o que significa que é um 
tipo *atômico de contagem*. Atomics é um tipo adicional de concorrência 
primitiva que não abordaremos em detalhes aqui: consulte a documentação da biblioteca 
padrão para `std::sync::atomic` para obter mais detalhes. Neste ponto, você só 
precisa saber que os atômicos funcionam como tipos primitivos, mas são seguros para 
compartilhar através de threads.

Você pode então se perguntar por que todos os tipos primitivos não são atômicos e 
por que os tipos da biblioteca padrão não são implementados para usar o `Arc<T>` por padrão. 
O motivo é que a segurança da thread vem com uma penalidade de desempenho que você 
só deseja pagar quando realmente precisa. Se você está apenas executando operações 
com valores em uma única thread, seu código pode ser executado mais rapidamente 
se não precisar impor as garantias que a atomicidade fornece.

Vamos voltar ao nosso exemplo: `Arc<T>` e `Rc<T>` têm a mesma API, portanto, 
corrigimos nosso programa alterando a linha `use`, a chamada para `new` e a chamada 
para `clone`. O código da Lista 16-15 finalmente compilará e executará:

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

<span class="caption">Listagem 16-15: Usando um `Arc<T>` para agrupar o `Mutex<T>` 
para poder compartilhar o ownership através de várias threads</span>

Este código imprimirá o seguinte:

```text
Result: 10
```

Conseguimos! Contamos de 0 a 10, o que pode não parecer muito impressionante, 
mas nos ensinou muito sobre o `Mutex<T>` e a segurança em threads. Você também pode 
usar a estrutura deste programa para realizar operações mais complicadas do que apenas 
incrementar um contador. Usando essa estratégia, você pode dividir um cálculo em partes 
independentes, dividi-las entre threads e, em seguida, usar o `Mutex<T>` para ter cada
thread atualiza o resultado final com sua parte.

### Semelhanças Entre `RefCell<T>` / `Rc<T>` e `Mutex<T>` / `Arc<T>`

Você deve ter notado que o `counter` é imutável, mas podemos obter uma referência 
mutável ao valor dentro dele; isso significa que `Mutex<T>` fornece mutabilidade 
interior, como a família `Cell` faz. Do mesmo modo que usamos `RefCell<T>` no 
Capítulo 15 para nos permitir mudar o conteúdo dentro de um `Rc<T>`, usamos o 
`Mutex<T>` para mudar o conteúdo dentro de um `Arc<T>`.

Outro detalhe a ser observado é que Rust não pode nos proteger de todos os tipos 
de erros lógicos quando usamos o `Mutex<T>`. Lembre-se no Capítulo 15 que o uso de `Rc<T>` 
corria o risco de criar ciclos de referência, onde dois valores de `Rc<T>` se referem um 
ao outro, causando vazamento de memória. Da mesma forma, o `Mutex<T>` vem com o risco de 
criar *deadlocks*. Isso ocorre quando uma operação precisa bloquear dois recursos e duas threads 
adquiriram um dos bloqueios, fazendo com que esperem uma pela outra para sempre. Se você estiver 
interessado em deadlocks, tente criar um programa Rust com deadlocks; depois pesquise estratégias 
de mitigação de deadlocks para mutex em qualquer linguagem e implementa-os em Rust. A documentação 
da API da biblioteca padrão para `Mutex<T>` e `MutexGuard` oferece informações úteis.

Concluiremos este capítulo falando sobre as traits `Send` e `Sync`, 
e como podemos usá-las com tipos personalizados.