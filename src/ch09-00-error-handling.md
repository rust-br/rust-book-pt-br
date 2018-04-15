# Tratamento de Erros

O comprometimento de Rust à segurança se extende ao tratamento de erros. Erros
são um fato da vida em software, portanto Rust possui um número de *features*
para lidar com situações em que algo dá errado. Em vários casos, Rust requer que
você reconheça a possibilidade de um erro acontecer e aja preventivamente antes
que seu código compile. Esse requisito torna seu programa mais robusto ao assegurar
que voce irá descobrir erros e lidar com eles apropriadamente antes de mandar seu
código para produção!

Rust agrupa erros em duas categorias principais: *recuperáveis* e *irrecuperáveis*.
Erros recuperáveis são situações em que é razoável reportar o problema ao usuário
e tentar a operação novamente, como um erro de arquivo não encontrado. Erros 
irrecuperáveis são sempre sintomas de bugs, como tentar acessar uma localização
além do fim de um *array*.

A maioria das linguagens não distingue esses dois tipos de erros e lida
com ambos da mesma maneira usando mecanismos como exceções. Rust não tem
exceções. Em vez disso, ele tem o valor `Result<T, E>` para erros recuperáveis
e a macro `panic!` que para a execução ao encontrar um erro irrecuperável. Esse
capítulo cobre primeiro como chamar `panic!` e depois fala sobre retornar valores
`Result<T, E>`. Adicionalmente, vamos explorar o que se levar em consideração
para decidir entre tentar se recuperar de um erro ou parar execução.

