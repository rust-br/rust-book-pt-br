## Extensible Concurrency with the `Sync` and `Send` Traits
## Concorrência Extensível com as Traits `Sync` e `Send`

Interestingly, the Rust language has *very* few concurrency features. Almost
every concurrency feature we’ve talked about so far in this chapter has been
part of the standard library, not the language. Our options for handling
concurrency are not limited to the language or the standard library; we can
write our own concurrency features or use those written by others.

Curiosamente, a linguagem Rust possui *muito* pouco recursos de concorrência. Quase 
todos os recursos de concorrência de que falamos até agora neste capítulo fazem parte 
da biblioteca padrão, não da linguagem. Nossas opções para lidar com a concorrência não 
se limitam a linguagem ou à biblioteca padrão; podemos escrever nossos próprios recursos 
de concorrência ou usar aqueles escritos por outras pessoas.

However, two concurrency concepts are embedded in the language: the
`std::marker` traits `Sync` and `Send`.

No entanto, dois conceitos de concorrência estão incorporados na linguagem: 
as `std::marker` traits `Sync` e `Send`.

### Allowing Transference of Ownership Between Threads with `Send`
### Permitindo Transferência de Ownership entre Threads com `Send`

The `Send` marker trait indicates that ownership of the type implementing
`Send` can be transferred between threads. Almost every Rust type is `Send`,
but there are some exceptions, including `Rc<T>`: this cannot be `Send` because
if we cloned an `Rc<T>` value and tried to transfer ownership of the clone to
another thread, both threads might update the reference count at the same time.
For this reason, `Rc<T>` is implemented for use in single-threaded situations
where you don’t want to pay the thread-safe performance penalty.

A trait do marcador `Send` indica que a ownership do tipo que implementa o `Send` 
pode ser transferida entre as threads. Quase todo tipo em Rust é `Send`, mas 
existem algumas exceções, incluindo `Rc<T>`: isso não pode ser `Send` porque 
se clonamos um valor `Rc<T>` e tentamos transferir a ownership do clone para 
outra thread, as duas threads podem atualizar a contagem de referência ao mesmo tempo. 
Por esse motivo, `Rc<T>` é implementado para uso em situações de thread única
onde você não deseja pagar pela penalidade de desempenho para threads seguras.

Therefore, Rust’s type system and trait bounds ensure that we can never
accidentally send an `Rc<T>` value across threads unsafely. When we tried to do
this in Listing 16-14, we got the error `the trait Send is not implemented for
Rc<Mutex<i32>>`. When we switched to `Arc<T>`, which is `Send`, the code
compiled.

Portanto, o sistema de tipos e os limites de trait em Rust garantem que nunca 
possamos enviar acidentalmente um valor `Rc<T>` através de threads de maneira 
insegura. Quando tentamos fazer isso na Lista 16-14, obtivemos o erro `the trait Send is not implemented for
Rc<Mutex<i32>>`. Quando mudamos para `Arc<T>`, que é `Send`, o código compilou.

Any type composed entirely of `Send` types is automatically marked as `Send` as
well. Almost all primitive types are `Send`, aside from raw pointers, which
we’ll discuss in Chapter 19.

Qualquer tipo composto inteiramente de tipos `Send` também é automaticamente marcado 
como `Send`. Quase todos os tipos primitivos são `Send`, além de ponteiros brutos (raw), 
que discutiremos no capítulo 19.

### Allowing Access from Multiple Threads with `Sync`
### Permitindo Acesso de Várias Threads com `Sync`

The `Sync` marker trait indicates that it is safe for the type implementing
`Sync` to be referenced from multiple threads. In other words, any type `T` is
`Sync` if `&T` (a reference to `T`) is `Send`, meaning the reference can be
sent safely to another thread. Similar to `Send`, primitive types are `Sync`
and types composed entirely of types that are `Sync` are also `Sync`.

O marcador `Sync` da trait indica que é seguro que o tipo que implementa `Sync` 
seja referenciado a partir de várias threads. Em outras palavras, qualquer tipo 
`T` é `Sync` se `&T` (uma referência à `T`) for `Send`, o que significa que a 
referência pode ser enviada com segurança para outra thread. Semelhante ao `Send`, 
os tipos primitivos são `Sync` e os tipos compostos inteiramente de tipos que são 
`Sync` também são `Sync`.

The smart pointer `Rc<T>` is also not `Sync` for the same reasons that it’s not
`Send`. The `RefCell<T>` type (which we talked about in Chapter 15) and the
family of related `Cell<T>` types are not `Sync`. The implementation of borrow
checking that `RefCell<T>` does at runtime is not thread-safe. The smart
pointer `Mutex<T>` is `Sync` and can be used to share access with multiple
threads, as you saw in the “Sharing a `Mutex<T>` Between Multiple Threads”
section.

O ponteiro inteligente `Rc<T>` também não é `Sync` pelos mesmos motivos que não 
é `Send`. O tipo `RefCell<T>` (sobre o qual falamos no Capítulo 15) e a família de 
tipos relacionados `Cell<T>` não são `Sync`. A implementação do borrow checking 
que `RefCell<T>` faz no tempo de execução não é segura para threads. O ponteiro 
inteligente `Mutex<T>` é `Sync` e pode ser usado para compartilhar o acesso com 
várias threads, como você viu na seção “Compartilhando um `Mutex<T>` Entre Várias 
Threads".

### Implementing `Send` and `Sync` Manually Is Unsafe
### Implementando `Send` e `Sync` Manualmente Não é Seguro

Because types that are made up of `Send` and `Sync` traits are automatically
also `Send` and `Sync`, we don’t have to implement those traits manually. As
marker traits, they don’t even have any methods to implement. They’re just
useful for enforcing invariants related to concurrency.

Como os tipos que são compostos pelas traits `Send` e `Sync` também são 
automaticamente `Send` e `Sync`, nós não precisamos implementar essas traits 
manualmente. Como traits de marcação, eles nem sequer têm métodos para implementar. 
Eles são úteis apenas para impor invariantes relacionados à concorrência.

Manually implementing these traits involves implementing unsafe Rust code.
We’ll talk about using unsafe Rust code in Chapter 19; for now, the important
information is that building new concurrent types not made up of `Send` and
`Sync` parts requires careful thought to uphold the safety guarantees.
[The Rustonomicon] has more information about these guarantees and how to
uphold them.

A implementação manual dessas traits envolve a implementação de código Rust inseguro. 
Falaremos sobre o uso de código Rust inseguro no Capítulo 19; por enquanto, a informação 
importante é que a construção de novos tipos simultâneos não compostos por partes `Send` 
e `Sync` requer reflexão cuidadosa para manter as garantias de segurança. 
[The Rustonomicon] tem mais informações sobre essas garantias e como defendê-las.

[The Rustonomicon]: https://doc.rust-lang.org/stable/nomicon/

## Summary
## Resumo

This isn’t the last you’ll see of concurrency in this book: the project in
Chapter 20 will use the concepts examined in this chapter in a more realistic
situation than the smaller examples discussed here.

Esta não é a última vez que você verá concorrência neste livro: o projeto no 
capítulo 20 usará os conceitos examinados neste capítulo em uma situação mais 
realista do que os exemplos menores discutidos aqui.

As mentioned earlier, because very little of how Rust handles concurrency is
part of the language, many concurrency solutions are implemented as crates.
These evolve more quickly than the standard library, so be sure to search
online for the current, state-of-the-art crates to use in multithreaded
situations.

Como mencionado anteriormente, como muito pouco de como Rust lida com a 
concorrência faz parte da linguagem, muitas soluções de concorrência são 
implementadas como crates. Elas evoluem mais rapidamente do que a biblioteca padrão; 
portanto, pesquise on-line crates atuais e de última geração para usar em 
situações multithread.

The Rust standard library provides channels for message passing and smart
pointer types, such as `Mutex<T>` and `Arc<T>`, that are safe to use in
concurrent contexts. The type system and the borrow checker ensure that the
code using these solutions won’t end up with data races or invalid references.
Once we get our code to compile, we can rest assured that it will happily run
on multiple threads without the kinds of hard-to-track-down bugs common in
other languages. Concurrent programming is no longer a concept to be afraid of:
go forth and make your programs concurrent, fearlessly!

A biblioteca padrão do Rust fornece canais para passagem de mensagens e tipos 
de ponteiros inteligentes, como `Mutex<T>` e `Arc<T>`, que são seguros para uso em 
contextos de concorrência. O sistema de tipos e o borrow checker garantem que o código 
usando essas soluções não acabe com corridas de dados ou referências inválidas. Uma vez 
que compilamos nosso código, podemos ter certeza de que ele será felizmente executado 
em várias threads sem os tipos de erros difíceis de rastrear comuns em outras linguagens. 
A programação concorrente não é mais um conceito a temer: vá em frente e torne seus 
programas concorrentes, sem medo!

Next, we’ll talk about idiomatic ways to model problems and structure solutions
as your Rust programs get bigger. In addition, we’ll discuss how Rust’s idioms
relate to those you might be familiar with from object-oriented programming.

Em seguida, falaremos sobre maneiras idiomáticas de modelar problemas e estruturar 
soluções à medida que seus programas Rust aumentam. Além disso, discutiremos como os 
idiomas de Rust se relacionam com aqueles com os quais você pode estar familiarizado 
com a programação orientada a objetos.