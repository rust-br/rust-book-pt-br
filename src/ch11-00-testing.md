# Escrevendo testes automatizados

Em seu ensaio de 1972 "The Humble Programmer", Edsger W. Dijkstra disse que "o teste de programa pode ser uma maneira muito eficaz de mostrar a presença de bugs, mas é irremediavelmente inadequado para mostrar sua ausência". Isso não significa que não devemos tentar testar o máximo que pudermos!

Correção em nossos programas é a extensão em que nosso código faz o que pretendemos que ele faça. Rust é projetado com um alto grau de preocupação com a correção dos programas, mas a correção é complexa e não é fácil de provar. O sistema de tipos de Rust carrega uma grande parte desse fardo, mas o sistema de tipos não pode pegar todo tipo de incorreção. Como tal, Rust inclui suporte para escrever testes de software automatizados dentro da linguagem.

Como exemplo, digamos que escrevemos uma função chamada `adicionar_dois` que adiciona 2 a qualquer número que seja passado a ela. A assinatura desta função aceita um inteiro como parâmetro e retorna um inteiro como resultado. Quando implementamos e compilamos essa função, Rust faz toda a verificação de tipo e empresta a verificação que você aprendeu até agora para garantir que, por exemplo, não estamos passando um valor `String` ou uma referência inválida para essa função. Mas Rust não pode verificar se esta função fará exatamente o que pretendemos, que é retornar o parâmetro mais 2 em vez de, digamos, o parâmetro mais 10 ou o parâmetro menos 50! É aí que entram os testes.

Podemos escrever testes que afirmam, por exemplo, que quando passamos `3` para a função `adicionar_dois`, o valor retornado é `5`. Podemos executar esses testes sempre que fizermos alterações em nosso código para garantir que nenhum comportamento correto existente não tenha sido alterado.

Testar é uma habilidade complexa: embora não possamos cobrir todos os detalhes sobre como escrever bons testes em um capítulo, discutiremos a mecânica das instalações de teste do Rust. Falaremos sobre as anotações e macros disponíveis para você ao escrever seus testes, o comportamento padrão e as opções fornecidas para executar seus testes e como organizar testes em testes de unidade e testes de integração.
