# Concorrência

Lidar com programação concorrente de forma segura e eficiente é outro grande
objetivo de Rust. *Programação concorrente*, onde diferentes partes de um 
programa são executadas de forma independente, e *programação paralela*,
onde diferentes partes de um programa são executadas no mesmo instante de tempo,
estão se tornando cada vez mais importantes à medida que mais  computadores 
aproveitam de seus múltiplos  processadores. Historicamente, programar neste
contexto tem sido difícil e sucetível a erros: Rust espera mudar este cenário.

Inicialmente, a equipe de desenvolvimento do Rust pensou que garantir a segurança de
memória e prevenir problemas de concorrência eram dois desafios separados com
soluções distintas. Ao longo do tempo, no entanto, a equipe descobriu que os sistemas
de tipo e *ownership* do Rust formam um conjunto poderoso de ferramentas para ajudar a gerenciar
a segurança de memória *e* problemas de concorrência! Aproveitando o *ownership* e a 
verificação de tipo, muitos erros de concorrência são identificados em tempo de compilação
ao invés de execução no Rust. Portanto, em vez de fazer você gastar muito tempo tentando
reproduzir as circunstâncias exatas em que ocorre um bug de concorrência em tempo de execução,
o código incorreto não irá compilar e uma mensagem de erro será apresentada explicando o problema.
Como resultado, você pode corrigir seu código enquanto ainda está trabalhando nele, em vez de
corrigí-lo após ele ter sido enviado para produção. Nós apelidamos este aspecto do Rust
de *fearless concurrency* (concorrência sem medo). A concorrência sem medo no Rust permite que você
escreva um código livre de bugs discretos e fácil de refatorar sem a introdução de novos bugs.

> Observação: Por simplicidade, vamos nos referir a muitos dos problemas como *concorrentes*, em vez de ser
> mais precisos distinguindo-os entre *concorrentes e/ou paralelos*. Se este livro fosse sobre 
> concorrência e/ou paralelismo, seríamos mais específicos. Para este capítulo, substitua mentalmente
> por *concorrente e/ou paralelo* sempre que usarmos a palavra *concorrente*.

Muitas linguagens de programação são dogmáticas quanto às soluções que elas oferecem
para lidar com problemas de concorrência. A linguagem  Erlang, por exemplo, tem  uma
elegante funcionalidade para  troca de mensagens (do inglês, *message-passing*)  de 
forma concorrente, no entanto, possui formas  obscuras para compartilhar estado entre
*threads*. Suportar apenas um subconjunto de soluções possíveis é uma estratégia razoável
para linguagens de alto nível, uma vez que tais linguagens ganham abstrações ao abrir mão de
algum tipo de controle. Em linguagens de baixo nível, no entanto, espera-se uma solução
com a melhor performance  em qualquer situação e menos abstrações sobre o hardware. 
Por este motivo, Rust oferece uma variedade de ferramentas para modelar problemas 
de concorrência da forma apropriada, considerando diferentes situações e requisitos.

Aqui estão os tópicos que cobriremos neste capítulo:

* Como criar *threads* para executar várias partes de código ao mesmo tempo;
* Concorrência em troca de mensagens (*Message-passing*), onde canais (*chanels*) enviam mensagens entre *threads*;
* Concorrência com estado compartilhado (*Shared-state*), onde múltiplas *threads* têm acesso a uma mesma porção de dados;
* As traits `Sync` e `Send`, que extendem as garantias de concorrência do Rust para tipos definidos  pelo usuário, bem como
 tipos fornecidos pela biblioteca padrão.