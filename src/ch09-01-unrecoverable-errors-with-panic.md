## Erros Irrecuperáveis com `panic!`

Às vezes coisas ruins acontecem no seu código, e não há nada que você possa fazer
sobre isso. Nesses casos, Rust tem a macro `panic!`. Quando ela é executada, seu
programa vai imprimir uma mensagem de falha, resolver e limpar a pilha e então
terminar. A situação mais comum em que isso acontece é quando algum tipo de bug
foi detectado, e não é claro para o programador como tratar esse erro.

> ### Resolver a Pilha ou Abortar em Resposta a `panic!`
>
> Por padrão, quando acontece um `panic!`, o programa começa a *resolver* a pilha,
> o que significa que Rust percorre a pilha e limpa os dados de cada função que encontra.
> Mas essa limpeza dá bastante trabalho. A alternativa é imediatamente *abortar*,
> que termina o programa sem fazer a limpeza. A memória que o programa estava usando vai então
> precisar ser limpada pelo sistema operacional. Se no seu projeto você precisa tornar
> o binário final o menor possível, você pode deixar de resolver e sim abortar no *panic*
> adicionando `panic = 'abort'` à seção apropriadada de `[profile]` no seu arquivo
> *Cargo.toml*. Por exemplo, se você quiser abortar no *panic* no modo de *release*, adicione
> isso:
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Vamos tentar chamar `panic!` em um programa simples:

<span class="filename">Arquivo: src/main.rs</span>

```rust,should_panic
fn main() {
    panic!("Quebra tudo");
}
```

Quando você roda o programa, verá algo como isso:

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25 secs
     Running `target/debug/panic`
thread 'main' panicked at 'Quebra tudo', src/main.rs:2
note: Run with `RUST_BACKTRACE=1` for a backtrace.
error: Process didn't exit successfully: `target/debug/panic` (exit code: 101)
```

A chamada a `panic!` causa a mensagem de erro contida nas últimas três linhas.
A primeira linha mostra nossa mensagem de pânico e a posição no código fonte
em que ocorreu o pânico: *src/main.rs:2* indica que é a segunda linha do nosso 
arquivo *src/main.rs*.

Nesse caso, a linha indicada é parte do nosso código, e se formos àquela linha
veremos a chamada à macro `panic!`. Em outros casos, a chamada a `panic!` pode
estar em código que nosso código chama. O nome do arquivo e número de linha
reportado pela mensagem de erro será no código de outra pessoa quando a macro 
`panic!` for chamada, não a linha do nosso código que eventualmente levou a chamada
de `panic!`. Podemos usar o *backtrace* das funções de onde veio a chamada a `panic!`
para entender qual parte de nosso código está causando o problema. Vamos discutir
o que é um *backtrace* em seguida.

### Usando um Backtrace de `panic!`

Vamos ver outro exemplo para ver o que acontece quando uma chamada `panic!` vem de uma
biblioteca por causa de um bug no nosso código em vez de nosso código chamar
a macro diretamente. A Listagem 9-1 tem código que tenta acessar um elemento em
um vetor por meio de um índice:

<span class="filename">Arquivo: src/main.rs</span>

```rust,should_panic
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

<span class="caption">Listagem 9-1: Tentativa de acessar um elemento além do fim
de um vetor, que causará um `panic!`</span>

Aqui, estamos tentando acessar o centésimo elemento (centésimo pois o índice 
começa em zero) de nosso vetor, mas ele só tem três elementos. Nesse caso, Rust
entrará em pânico. Supostamente `[]` devolve um elemento, mas se você passa um
índice inválido, não há elemento que Rust possa retornar que fosse correto.

Outras linguagens, como C, vão tentar te dar exatamente o que você pediu nessa
situação, mesmo que não seja o que você quer: você vai receber o que quer que 
esteja na localização na memória que corresponderia àquele elemento no vetor,
mesmo que a memória não pertença ao vetor. Isso se chama um *buffer overread* e 
pode levar a vulnerabilidades de segurança se um agressor for capaz de manipular
o índice de forma a ler dados guardados depois do array aos quais ele não deveria
ter acesso.

Para proteger seu programa desse tipo de vulnerabilidade, se você tentar ler
um elemento em um índice que não exista, Rust vai parar a execução e se recusar
a continuar. Vamos fazer isso e ver o que acontece:

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is
100', /stable-dist-rustc/build/src/libcollections/vec.rs:1362
note: Run with `RUST_BACKTRACE=1` for a backtrace.
error: Process didn't exit successfully: `target/debug/panic` (exit code: 101)
```

Esse erro aponta para um arquivo que não escrevemos, *libcollections/vec.rs*.
Essa é a implementação de `Vec<T>` na biblioteca padrão. O código que roda
quando usamos `[]` em nosso vetor `v` está em *libcollections/vec.rs*, e é aí
que o `panic!` está de fato acontecendo.


A próxima linha nos diz que podemos definir a variável de ambiente `RUST_BACKTRACE`
para ter um *backtrace* (rastro) do que aconteceu, exatamente, para causar o erro. Um 
*backtrace* é uma lista de todas as funções que foram chamadas para chegar a esse
ponto. *Backtraces* em Rust funcionam como em outras linguagens: a chave para ler
o *backtrace* é começar do topo e ler até você ver os arquivos que você escreveu.
Esse é o ponto em que o problema se originou. As linhas acima das que mencionam seu
código são funções que você chamou; as linhas abaixo são funções que chamaram seu
código. Essas linhas podem incluir código do núcleo do Rust, código das bibliotecas
padrão, ou *crates* que você está usando. Vamos tentar ver um backtrace: a Listagem 9-2
mostra uma saída semelhante a o que você verá:

```text
$ RUST_BACKTRACE=1 cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 100', /stable-dist-rustc/build/src/libcollections/vec.rs:1392
stack backtrace:
   1:     0x560ed90ec04c - std::sys::imp::backtrace::tracing::imp::write::hf33ae72d0baa11ed
                        at /stable-dist-rustc/build/src/libstd/sys/unix/backtrace/tracing/gcc_s.rs:42
   2:     0x560ed90ee03e - std::panicking::default_hook::{{closure}}::h59672b733cc6a455
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:351
   3:     0x560ed90edc44 - std::panicking::default_hook::h1670459d2f3f8843
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:367
   4:     0x560ed90ee41b - std::panicking::rust_panic_with_hook::hcf0ddb069e7abcd7
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:555
   5:     0x560ed90ee2b4 - std::panicking::begin_panic::hd6eb68e27bdf6140
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:517
   6:     0x560ed90ee1d9 - std::panicking::begin_panic_fmt::abcd5965948b877f8
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:501
   7:     0x560ed90ee167 - rust_begin_unwind
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:477
   8:     0x560ed911401d - core::panicking::panic_fmt::hc0f6d7b2c300cdd9
                        at /stable-dist-rustc/build/src/libcore/panicking.rs:69
   9:     0x560ed9113fc8 - core::panicking::panic_bounds_check::h02a4af86d01b3e96
                        at /stable-dist-rustc/build/src/libcore/panicking.rs:56
  10:     0x560ed90e71c5 - <collections::vec::Vec<T> as core::ops::Index<usize>>::index::h98abcd4e2a74c41
                        at /stable-dist-rustc/build/src/libcollections/vec.rs:1392
  11:     0x560ed90e727a - panic::main::h5d6b77c20526bc35
                        at /home/you/projects/panic/src/main.rs:4
  12:     0x560ed90f5d6a - __rust_maybe_catch_panic
                        at /stable-dist-rustc/build/src/libpanic_unwind/lib.rs:98
  13:     0x560ed90ee926 - std::rt::lang_start::hd7c880a37a646e81
                        at /stable-dist-rustc/build/src/libstd/panicking.rs:436
                        at /stable-dist-rustc/build/src/libstd/panic.rs:361
                        at /stable-dist-rustc/build/src/libstd/rt.rs:57
  14:     0x560ed90e7302 - main
  15:     0x7f0d53f16400 - __libc_start_main
  16:     0x560ed90e6659 - _start
  17:                0x0 - <unknown>

```

<span class="caption">Listagem 9-2: O *backtrace* gerado por uma chamada
a `panic!` mostrado quando a variável de ambiente `RUST_BACKTRACE` está definida.</span>

Isso é bastante saída! A saída exata que você recebe pode ser diferente dependendo
do seu sistema operacional e versão de Rust. Para conseguir *backtraces* com essa 
informação, símbolos de debug devem estar ativados. Símbolos de debug estão ativados
por padrão quando usamos cargo build ou cargo run sem a opção de --release, 
como temos aqui.

Na saída da Listagem 9-2, a linha 11 do *backtrace* aponta para a linha no nosso
projeto que está causando o problema: *src/main.rs* na linha 4. Se não 
queremos que nosso programa entre em pânico, a localização apontada pela primeira
linha mencionando um arquivo que escrevemos é onde deveríamos começar a investigar
para entender como chegamos nesse ponto com valores que causaram o pânico. Na lista
9-1 onde nós deliberadamente escrevemos código que causaria pânico para demonstrar
como usar *backtraces*, a forma de consertar o pânico é não requisitar um elemento
no índice 100 de um vetor que contém apenas três itens. Quando seu código entrar
em pânico no futuro, você precisará descobrir quais ações o seu código está tomando,
e com quais valores, que estão causando o pânico, e o que deveria ser feito em vez
disso.

Voltaremos ao `panic!` e veremos mais adiante no capítulo quando usá-lo, ou não,
para lidar com erros. Em seguida, olharemos como se recuperar de um erro usando 
`Result`.
