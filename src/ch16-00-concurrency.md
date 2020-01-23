# Concorrência sem medo

Lidar com a programação concorrente com segurança e eficiência é um dos principais objetivos do Rust. *Programação concorrente*, onde diferentes partes de um programa são executadas independentemente e *programação paralela*, onde diferentes partes de um programa são executadas ao mesmo tempo, estão se tornando cada vez mais importantes à medida que mais computadores tiram vantagem de seus múltiplos processadores. Historicamente, a programação nesses contextos tem sido difícil e propensa a erros: Rust espera mudar isso.

Inicialmente, a equipe do Rust achou que garantir a segurança da memória e evitar problemas de concorrência eram dois desafios separados a serem resolvidos com métodos diferentes. Com o tempo, a equipe descobriu que os sistemas de propriedade e tipo são um poderoso conjunto de ferramentas para ajudar a gerenciar problemas de segurança de memória *e* concorrência! Ao alavancar a propriedade e a verificação de tipo, muitos erros de concorrência são erros de *tempo de compilação* no Rust em vez de erros de tempo de execução. Portanto, em vez de você gastar muito tempo tentando reproduzir as circunstâncias exatas sob as quais ocorre um erro de concorrência de tempo de execução, o código incorreto se recusará a compilar e apresentará um erro explicando o problema. Como resultado, você pode corrigir seu código enquanto trabalha nele, e não potencialmente depois que ele é enviado para produção. Apelidamos esse aspecto do Rust de *concorrência sem medo*. A concorrência sem medo permite que você escreva um código livre de erros sutis e fácil de refatorar sem a introdução de novos erros.

>Nota: por uma questão de simplicidade, nos referimos a muitos dos problemas como concorrente,
>em vez de mais precisos, dizendo concorrente e/ou paralelo. Se este livro fosse especificamente
>sobre concorrência e/ou paralelismo, seríamos mais específicos. Neste capítulo, substitua
>mentalmente concorrente e/ou paralelo sempre que usarmos concorrência.

Muitas linguagens são dogmáticos sobre as soluções que eles oferecem para lidar com problemas concorrentes. Por exemplo, Erlang possui uma funcionalidade elegante para concorrência de *message passing*, mas possui apenas maneiras obscuras de *shared state* entre as threads.
O suporte a apenas um subconjunto de soluções possíveis é uma estratégia razoável para linguagens de alto nível, porque promete benefícios ao abrir mão de algum controle para obter abstrações. No entanto, espera-se que as linguagens de baixo nível forneçam à solução o melhor desempenho em qualquer situação e tenham menos abstrações sobre o hardware. Portanto, o Rust oferece uma variedade de ferramentas para modelar problemas da maneira que for apropriada para sua situação e requisitos.

Aqui estão os tópicos que abordaremos neste capítulo:

* Como criar threads para executar várias partes do código ao mesmo tempo;
* Concorrência de *Message passing*, onde os canais enviam mensagens entre threads;
* Concorrência de *Shared state*, em que vários threads têm acesso a alguns dados;
* Os traits `Sync` e` Send`, que estendem as garantias de concorrência do Rust aos tipos definidos pelo usuário, bem como aos tipos fornecidos pela biblioteca padrão.
