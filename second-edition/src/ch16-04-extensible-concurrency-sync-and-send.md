## Concorrência Extensível com as Traits `Sync` e `Send`

Curiosamente, a linguagem Rust possui *muito* pouco recursos de concorrência. Quase 
todos os recursos de concorrência de que falamos até agora neste capítulo fazem parte 
da biblioteca padrão, não da linguagem. Nossas opções para lidar com a concorrência não 
se limitam a linguagem ou à biblioteca padrão; podemos escrever nossos próprios recursos 
de concorrência ou usar aqueles escritos por outras pessoas.

No entanto, dois conceitos de concorrência estão incorporados na linguagem: 
as `std::marker` traits `Sync` e `Send`.

### Permitindo Transferência de Ownership entre Threads com `Send`

A trait do marcador `Send` indica que a ownership do tipo que implementa o `Send` 
pode ser transferida entre as threads. Quase todo tipo em Rust é `Send`, mas 
existem algumas exceções, incluindo `Rc<T>`: isso não pode ser `Send` porque 
se clonamos um valor `Rc<T>` e tentamos transferir a ownership do clone para 
outra thread, as duas threads podem atualizar a contagem de referência ao mesmo tempo. 
Por esse motivo, `Rc<T>` é implementado para uso em situações de thread única
onde você não deseja pagar pela penalidade de desempenho para threads seguras.

Portanto, o sistema de tipos e os limites de trait em Rust garantem que nunca 
possamos enviar acidentalmente um valor `Rc<T>` através de threads de maneira 
insegura. Quando tentamos fazer isso na Lista 16-14, obtivemos o erro `the trait Send is not implemented for
Rc<Mutex<i32>>`. Quando mudamos para `Arc<T>`, que é `Send`, o código compilou.

Qualquer tipo composto inteiramente de tipos `Send` também é automaticamente marcado 
como `Send`. Quase todos os tipos primitivos são `Send`, além de ponteiros brutos (raw), 
que discutiremos no capítulo 19.

### Permitindo Acesso de Várias Threads com `Sync`

O marcador `Sync` da trait indica que é seguro que o tipo que implementa `Sync` 
seja referenciado a partir de várias threads. Em outras palavras, qualquer tipo 
`T` é `Sync` se `&T` (uma referência à `T`) for `Send`, o que significa que a 
referência pode ser enviada com segurança para outra thread. Semelhante ao `Send`, 
os tipos primitivos são `Sync` e os tipos compostos inteiramente de tipos que são 
`Sync` também são `Sync`.

O ponteiro inteligente `Rc<T>` também não é `Sync` pelos mesmos motivos que não 
é `Send`. O tipo `RefCell<T>` (sobre o qual falamos no Capítulo 15) e a família de 
tipos relacionados `Cell<T>` não são `Sync`. A implementação do borrow checking 
que `RefCell<T>` faz no tempo de execução não é segura para threads. O ponteiro 
inteligente `Mutex<T>` é `Sync` e pode ser usado para compartilhar o acesso com 
várias threads, como você viu na seção “Compartilhando um `Mutex<T>` Entre Várias 
Threads".

### Implementando `Send` e `Sync` Manualmente Não é Seguro

Como os tipos que são compostos pelas traits `Send` e `Sync` também são 
automaticamente `Send` e `Sync`, nós não precisamos implementar essas traits 
manualmente. Como traits de marcação, eles nem sequer têm métodos para implementar. 
Eles são úteis apenas para impor invariantes relacionados à concorrência.

A implementação manual dessas traits envolve a implementação de código Rust inseguro. 
Falaremos sobre o uso de código Rust inseguro no Capítulo 19; por enquanto, a informação 
importante é que a construção de novos tipos simultâneos não compostos por partes `Send` 
e `Sync` requer reflexão cuidadosa para manter as garantias de segurança. 
[The Rustonomicon] tem mais informações sobre essas garantias e como defendê-las.

[The Rustonomicon]: https://doc.rust-lang.org/stable/nomicon/

## Resumo

Esta não é a última vez que você verá concorrência neste livro: o projeto no 
capítulo 20 usará os conceitos examinados neste capítulo em uma situação mais 
realista do que os exemplos menores discutidos aqui.

Como mencionado anteriormente, como muito pouco de como Rust lida com a 
concorrência faz parte da linguagem, muitas soluções de concorrência são 
implementadas como crates. Elas evoluem mais rapidamente do que a biblioteca padrão; 
portanto, pesquise on-line crates atuais e de última geração para usar em 
situações multithread.

A biblioteca padrão do Rust fornece canais para passagem de mensagens e tipos 
de ponteiros inteligentes, como `Mutex<T>` e `Arc<T>`, que são seguros para uso em 
contextos de concorrência. O sistema de tipos e o borrow checker garantem que o código 
usando essas soluções não acabe com corridas de dados ou referências inválidas. Uma vez 
que compilamos nosso código, podemos ter certeza de que ele será felizmente executado 
em várias threads sem os tipos de erros difíceis de rastrear comuns em outras linguagens. 
A programação concorrente não é mais um conceito a temer: vá em frente e torne seus 
programas concorrentes, sem medo!

Em seguida, falaremos sobre maneiras idiomáticas de modelar problemas e estruturar 
soluções à medida que seus programas Rust aumentam. Além disso, discutiremos como os 
idiomas de Rust se relacionam com aqueles com os quais você pode estar familiarizado 
com a programação orientada a objetos.