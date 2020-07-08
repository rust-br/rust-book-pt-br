# Escrevendo Testes Automatizados

Em seu ensaio de 1972, "The Humble Programmer", Edsger W. Dijkstra disse que "O 
teste de programas podem ser uma maneira muito eficaz de mostrar a presença de bugs, 
mas é irremediavelmente inadequado para mostrar sua ausência". Isso não significa que 
não devemos tentar testar o máximo que pudermos!

A correção em nossos programas é a extensão em que nosso código faz o que pretendemos 
fazer. Rust é projetado com um alto grau de preocupação com a correção dos programas, 
mas a correção é complexa e não é fácil de provar. O sistema de tipos Rust suporta uma 
grande parte desse fardo, mas o sistema de tipos não pode pegar todo tipo de erro. Como tal, 
Rust inclui suporte para escrever testes de software automatizados na linguagem.

Como exemplo, digamos que escrevemos uma função chamada `add_two` que adiciona 2 ao 
número que for passado para ela. A assinatura desta função aceita um número inteiro como 
parâmetro e retorna um número inteiro como resultado. Quando implementamos e compilamos essa 
função, Rust faz toda a verificação de tipo e de borrow que você aprendeu até agora para 
garantir que, por exemplo, não passemos um valor de `String` ou uma referência inválida à essa 
função. Mas Rust *não pode* verificar se essa função fará exatamente o que pretendemos, que é 
retornar o parâmetro mais 2 em vez de, digamos, o parâmetro mais 10 ou o parâmetro menos 50! É 
aí que entram os testes.

Podemos escrever testes que afirmam, por exemplo, que quando passamos `3` para a 
função `add_two`, o valor retornado é `5`. Podemos executar esses testes sempre que 
fizermos alterações em nosso código para garantir que qualquer comportamento correto 
existente não seja alterado.

O teste é uma habilidade complexa: embora não possamos cobrir todos os detalhes sobre 
como escrever  bons testes em um capítulo, discutiremos a mecânica de facilidade de teste 
em Rust. Falaremos sobre as anotações e macros disponíveis para você ao escrever seus testes, 
o comportamento padrão e as opções fornecidas para executar seus testes e como organizar os 
testes em testes de unidade e testes de integração.