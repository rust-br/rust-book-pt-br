# Error Handling
# Tratamento de Erros

Rust’s commitment to reliability extends to error handling. Errors are a fact
of life in software, so Rust has a number of features for handling situations
in which something goes wrong. In many cases, Rust requires you to acknowledge
the possibility of an error occurring and take some action before your code
will compile. This requirement makes your program more robust by ensuring that
you’ll discover errors and handle them appropriately before you’ve deployed
your code to production!

O comprometimento de Rust à segurança se extende ao tratamento de erros. Erros
são um fato da vida em software, portanto Rust possui um número de *features*
para lidar com situações em que algo dá errado. Em vários casos, Rust requer que
você reconheça a possibilidade de um erro acontecer e aja preventivamente antes
que seu código compile. Esse requisito torna seu programa mais robusto ao assegurar
que voce irá descobrir erros e lidar com eles apropriadamente antes de mandar seu
código para produção!

Rust groups errors into two major categories: *recoverable* and *unrecoverable*
errors. Recoverable errors are situations in which it’s reasonable to report
the problem to the user and retry the operation, like a file not found error.
Unrecoverable errors are always symptoms of bugs, like trying to access a
location beyond the end of an array.

Rust agrupa erros em duas categorias principais: *recuperáveis* e *irrecuperáveis*.
Erros recuperáveis são situações em que é razoável reportar o problema ao usuário
e tentar a operação novamente, como um erro de arquivo não encontrado. Erros 
irrecuperáveis são sempre sintomas de bugs, como tentar acessar uma localização
além do fim de um *array*

Most languages don’t distinguish between these two kinds of errors and handle
both in the same way using mechanisms like exceptions. Rust doesn’t have
exceptions. Instead, it has the value `Result<T, E>` for recoverable errors and
the `panic!` macro that stops execution when it encounters unrecoverable
errors. This chapter covers calling `panic!` first and then talks about
returning `Result<T, E>` values. Additionally, we’ll explore considerations to
take into account when deciding whether to try to recover from an error or to
stop execution.

A maioria das linguagens não distingue entre esses dois tipos de erros e lida
com ambos da mesma maneira usando mecanismos como *exceptions*. Rust não tem
*exceptions*. Ao invés, ele tem o valor `Result<T, E>` para erros recuperáveis
e a macro `panic!` que para a execução ao encontrar um erro irrecuperável. Esse
capitulo cobre primeiro como chamar `panic!` e depois fala sobre retornar valores
`Result<T, E>`. Adicionalmente, vamos explorar o que se levar em consideração
para decidir entre tentar se recuperar de um erro ou parar execução.

