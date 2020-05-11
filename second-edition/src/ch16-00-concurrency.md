# Fearless Concurrency
# Concorrência sem medo

Handling concurrent programming safely and efficiently is another of Rust’s
major goals. *Concurrent programming*, where different parts of a program
execute independently, and *parallel programming*, where different parts of a
program execute at the same time, are becoming increasingly important as more
computers take advantage of their multiple processors. Historically,
programming in these contexts has been difficult and error prone: Rust hopes to
change that.

Lidar com a programação concorrente com segurança e eficiência é outro dos
objetivos principais em Rust. *Programação concorrente*, onde diferentes partes de um programa
executam independentemente e *programação paralela*, onde diferentes partes de um
programa executam ao mesmo tempo, estão se tornando cada vez mais importante
os computadores aproveitarem seus múltiplos processadores. Historicamente,
a programação nesses contextos tem sido difícil e propensa a erros: Rust espera
mudar isso.

Initially, the Rust team thought that ensuring memory safety and preventing
concurrency problems were two separate challenges to be solved with different
methods. Over time, the team discovered that the ownership and type systems are
a powerful set of tools to help manage memory safety *and* concurrency
problems! By leveraging ownership and type checking, many concurrency errors
are *compile time* errors in Rust rather than runtime errors. Therefore, rather
than you spending lots of time trying to reproduce the exact circumstances
under which a runtime concurrency bug occurs, incorrect code will refuse to
compile and present an error explaining the problem. As a result, you can fix
your code while you’re working on it rather than potentially after it has been
shipped to production. We’ve nicknamed this aspect of Rust *fearless*
*concurrency*. Fearless concurrency allows you to write code that is free of
subtle bugs and is easy to refactor without introducing new bugs.

Inicialmente, a equipe Rust achou que garantir a segurança da memória e impedir
problemas de concorrência eram dois desafios separados a serem resolvidos com diferentes
métodos. Com o tempo, a equipe descobriu que os sistemas de ownership e tipo são
um conjunto poderoso de ferramentas para ajudar a gerenciar a segurança da memória *e* problemas
de concorrência! Ao alavancar a ownership (posse) e a verificação de tipo, muitos erros de concorrência
são erros de *tempo de compilação* em Rust em vez de erros de tempo de execução. Portanto, 
ao invés de você gastar muito tempo tentando reproduzir as circunstâncias exatas
sob o qual ocorre um erro de concorrência em tempo de execução, o código incorreto se recusará a
compilar e apresentará um erro ao explicar o problema. Como resultado, você pode corrigir
seu código enquanto você trabalha nele, e não potencialmente depois que ele foi
enviado para produção. Apelidamos esse aspecto de Rust *concorrência sem medo*.
A concorrência sem medo permite que você escreva um código livre de erros sutis e 
é fácil refatorar sem introduzir novos erros.

> Note: For simplicity’s sake, we’ll refer to many of the problems as
> concurrent rather than being more precise by saying concurrent and/or
> parallel. If this book was specifically about concurrency and/or parallelism,
> we’d be more. specific. For this chapter, please mentally substitute
> concurrent and/or parallel whenever we use concurrent.

> Nota: por uma questão de simplicidade, nos referiremos a muitos dos problemas como
> concorrente, em vez de ser mais preciso, dizendo concorrente e/ou paralelo. Se este 
> livro fosse especificamente sobre concorrência e/ou paralelismo, seríamos mais 
> específicos. Neste capítulo, substitua mentalmente concorrente e/ou paralelo 
> sempre que usarmos concorrente.

Many languages are dogmatic about the solutions they offer for handling
concurrent problems. For example, Erlang has elegant functionality for message
passing concurrency but has only obscure ways to share state between threads.
Supporting only a subset of possible solutions is a reasonable strategy for
higher-level languages, because a higher-level language promises benefits from
giving up some control to gain abstractions. However, lower-level languages are
expected to provide the solution with the best performance in any given
situation and have fewer abstractions over the hardware. Therefore, Rust offers
a variety of tools for modeling problems in whatever way is appropriate for
your situation and requirements.

Muitas linguagens são dogmáticos sobre as soluções que elas oferecem para lidar com 
problemas de concorrência. Por exemplo, Erlang possui uma funcionalidade elegante para 
passagem de mensagens concorrentes, mas possui apenas maneiras obscuras de compartilhar 
estado entre threads. O suporte a apenas um subconjunto de soluções possíveis é uma estratégia 
razoável para linguagens de alto nível, porque uma linguagem de alto nível promete benefícios
desistindo de algum controle para obter abstrações. No entanto, espera-se que as linguagens de 
baixo nível forneçam à solução o melhor desempenho em qualquer situação e tenham menos abstrações 
sobre o hardware. Portanto, Rust oferece uma variedade de ferramentas para modelar problemas da 
maneira que for apropriada para sua situação e requisitos.

Here are the topics we’ll cover in this chapter:

Aqui estão os tópicos que abordaremos neste capítulo:

* How to create threads to run multiple pieces of code at the same time
* *Message passing* concurrency, where channels send messages between threads
* *Shared state* concurrency, where multiple threads have access to some piece
  of data
* The `Sync` and `Send` traits, which extend Rust’s concurrency guarantees to
  user-defined types as well as types provided by the standard library

  

* Como criar threads para executar várias partes do código ao mesmo tempo
* *Passagem de mensagem* concorrente, onde os canais enviam mensagens entre threads
* *Estado compartilhado* (Shared state concorrente, em que várias threads têm acesso a alguma parte
   dos dados  
* As características `Sync` e `Send`, que estendem as garantias de concorrência do Rust a tipos definidos pelo usuário, bem como aos tipos fornecidos pela biblioteca padrão

