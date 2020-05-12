# Concorrência sem medo

Lidar com a programação concorrente com segurança e eficiência é outro dos
objetivos principais em Rust. *Programação concorrente*, onde diferentes partes de um programa
executam independentemente e *programação paralela*, onde diferentes partes de um
programa executam ao mesmo tempo, estão se tornando cada vez mais importante
os computadores aproveitarem seus múltiplos processadores. Historicamente,
a programação nesses contextos tem sido difícil e propensa a erros: Rust espera
mudar isso.

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

> Nota: por uma questão de simplicidade, nos referiremos a muitos dos problemas como
> concorrente, em vez de ser mais preciso, dizendo concorrente e/ou paralelo. Se este 
> livro fosse especificamente sobre concorrência e/ou paralelismo, seríamos mais 
> específicos. Neste capítulo, substitua mentalmente concorrente e/ou paralelo 
> sempre que usarmos concorrente.

Muitas linguagens são dogmáticos sobre as soluções que elas oferecem para lidar com 
problemas de concorrência. Por exemplo, Erlang possui uma funcionalidade elegante para 
passagem de mensagens concorrentes, mas possui apenas maneiras obscuras de compartilhar 
estado entre threads. O suporte a apenas um subconjunto de soluções possíveis é uma estratégia 
razoável para linguagens de alto nível, porque uma linguagem de alto nível promete benefícios
desistindo de algum controle para obter abstrações. No entanto, espera-se que as linguagens de 
baixo nível forneçam à solução o melhor desempenho em qualquer situação e tenham menos abstrações 
sobre o hardware. Portanto, Rust oferece uma variedade de ferramentas para modelar problemas da 
maneira que for apropriada para sua situação e requisitos.

Aqui estão os tópicos que abordaremos neste capítulo:

* Como criar threads para executar várias partes do código ao mesmo tempo
* *Passagem de mensagem* concorrente, onde os canais enviam mensagens entre threads
* *Estado compartilhado* (Shared state concorrente, em que várias threads têm acesso a alguma parte
   dos dados  
* As características `Sync` e `Send`, que estendem as garantias de concorrência do Rust a tipos definidos pelo usuário, bem como aos tipos fornecidos pela biblioteca padrão

