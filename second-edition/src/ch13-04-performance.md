## Comparando o Desempenho: Loops x Iteradores

Para determinar se deseja usar loops ou iteradores, você precisa saber qual versão 
de nossas funções `search` é mais rápida: a versão com um loop explícito `for` ou 
a versão com iteradores.

Executamos uma referência carregando o conteúdo inteiro do *The Adventures of 
Sherlock Holmes*, de Sir Arthur Conan Doyle, em um `String` e procurando a palavra 
*the* no conteúdo. Aqui estão os resultados do benchmark na versão do `search` 
usando o loop `for` e a versão usando os iteradores:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

A versão do iterador foi um pouco mais rápida! Não explicaremos o código de 
referência aqui, porque o objetivo não é provar que as duas versões são equivalentes, 
mas ter uma noção geral de como essas duas implementações se comparam em termos de 
desempenho.

Para um benchmark mais abrangente, você deve verificar o uso de vários textos de 
vários tamanhos como o `contents`, diferentes palavras, e palavras de diferentes 
comprimentos como a `query` e todos os tipos de outras variações. O ponto é o seguinte: 
os iteradores, embora sejam uma abstração de alto nível, são compilados para 
aproximadamente o mesmo código como se você tivesse escrito o código de baixo nível. 
Iteradores são uma das *zero-cost abstractions* (abstrações de custo zero) de Rust, 
com o que queremos dizer usando a abstração não impõe sobrecarga de tempo de execução 
adicional. Isso é análogo ao modo como Bjarne Stroustrup, o designer e implementador 
original do C ++, define *zero-overhead* em "Foundations of C ++" (2012):

> Em geral, as implementações de C++ obedecem ao princípio de zero sobrecarga: 
> o que você não usa, não paga. E mais: o que você usa, não poderia entregar código
> melhor.

Como outro exemplo, o código a seguir é obtido de um decodificador de áudio. 
O algoritmo de decodificação usa a operação matemática de previsão linear para 
estimar valores futuros com base em uma função linear das amostras anteriores. 
Este código usa uma cadeia de iteradores para fazer algumas contas em três 
variáveis no escopo: uma fatia de dados `buffer`, uma matriz de 12 `coefficients` 
(coeficientes) e uma quantidade pela qual mudar os dados em `qlp_shift`. Declaramos 
as variáveis neste exemplo, mas não atribuímos nenhum valor a elas; embora esse código 
não tenha muito significado fora de seu contexto, ainda é um exemplo conciso e real 
do mundo de como Rust traduz idéias de alto nível em código de baixo nível.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

Para calcular o valor de `prediction`, este código repete cada um dos 12 valores 
em `coefficients` e usa o método `zip` para emparelhar os valores do coeficiente 
com os 12 valores anteriores em `buffer`. Então, para cada par, multiplicamos os 
valores juntos, somamos todos os resultados e alteramos os bits na soma `qlp_shift` 
bits para a direita.

Cálculos em aplicativos como decodificadores de áudio geralmente priorizam mais o 
desempenho. Aqui, estamos criando um iterador, usando dois adaptadores e consumindo 
o valor. Em qual código _assembly_ , esse código Rust, teria gerado (compilado)? 
Bem, no momento da redação deste documento, ele é compilado no mesmo assembly que 
você escreveria à mão. Não existe um loop que corresponda à iteração sobre os valores 
em `coefficients`: Rust sabe que existem 12 iterações; portanto,"unrolls" (desenrola) 
o loop. *Unrolling* é uma otimização que remove a sobrecarga do código de controle do 
loop e gera um código repetitivo para cada iteração do loop.

Todos os coeficientes são armazenados nos registradores, o que significa que o 
acesso aos valores é muito rápido. Não há verificações de limites no acesso à matriz 
em tempo de execução. Todas essas otimizações que Rust pode aplicar tornam o código 
resultante extremamente eficiente. Agora que você sabe disso, pode usar iteradores e 
closures sem medo! Eles fazem o código parecer um nível mais alto, mas não impõem uma 
penalidade de desempenho em tempo de execução por isso.

## Resumo

Closures e iteradores são recursos Rust inspirados em idéias da linguagem de programação 
funcionais. Elas contribuem para a capacidade de Rust em expressar claramente idéias 
de alto nível com desempenho de baixo nível. As implementações de closures e iteradores 
são tais que o desempenho do tempo de execução não é afetado. Isso faz parte do objetivo 
do Rust de fornecer abstrações de custo zero.

Agora que melhoramos a expressividade do nosso projeto de E/S, vejamos mais alguns recursos 
de `cargo` que nos ajudarão a compartilhar o projeto com o mundo.