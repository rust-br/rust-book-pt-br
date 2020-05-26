## Comparing Performance: Loops vs. Iterators
## Comparando o Desempenho: Loops x Iteradores

To determine whether to use loops or iterators, you need to know which version
of our `search` functions is faster: the version with an explicit `for` loop or
the version with iterators.

Para determinar se deseja usar loops ou iteradores, você precisa saber qual versão 
de nossas funções `search` é mais rápida: a versão com um loop explícito `for` ou 
a versão com iteradores.

We ran a benchmark by loading the entire contents of *The Adventures of
Sherlock Holmes* by Sir Arthur Conan Doyle into a `String` and looking for the
word *the* in the contents. Here are the results of the benchmark on the
version of `search` using the `for` loop and the version using iterators:

Executamos uma referência carregando o conteúdo inteiro do *The Adventures of 
Sherlock Holmes*, de Sir Arthur Conan Doyle, em um `String` e procurando a palavra 
*the* no conteúdo. Aqui estão os resultados do benchmark na versão do `search` 
usando o loop `for` e a versão usando os iteradores:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

The iterator version was slightly faster! We won’t explain the benchmark code
here, because the point is not to prove that the two versions are equivalent
but to get a general sense of how these two implementations compare
performance-wise.

A versão do iterador foi um pouco mais rápida! Não explicaremos o código de 
referência aqui, porque o objetivo não é provar que as duas versões são equivalentes, 
mas ter uma noção geral de como essas duas implementações se comparam em termos de 
desempenho.

For a more comprehensive benchmark, you should check using various texts of
various sizes as the `contents`, different words and words of different lengths
as the `query`, and all kinds of other variations. The point is this:
iterators, although a high-level abstraction, get compiled down to roughly the
same code as if you’d written the lower-level code yourself. Iterators are one
of Rust’s *zero-cost abstractions*, by which we mean using the abstraction
imposes no additional runtime overhead. This is analogous to how Bjarne
Stroustrup, the original designer and implementor of C++, defines
*zero-overhead* in “Foundations of C++” (2012):

Para um benchmark mais abrangente, você deve verificar o uso de vários textos de 
vários tamanhos como o `contents`, diferentes palavras, e palavras de diferentes 
comprimentos como a `query` e todos os tipos de outras variações. O ponto é o seguinte: 
os iteradores, embora sejam uma abstração de alto nível, são compilados para 
aproximadamente o mesmo código como se você tivesse escrito o código de baixo nível. 
Iteradores são uma das *zero-cost abstractions* (abstrações de custo zero) de Rust, 
com o que queremos dizer usando a abstração não impõe sobrecarga de tempo de execução 
adicional. Isso é análogo ao modo como Bjarne Stroustrup, o designer e implementador 
original do C ++, define *zero-overhead* em "Foundations of C ++" (2012):

> In general, C++ implementations obey the zero-overhead principle: What you
> don’t use, you don’t pay for. And further: What you do use, you couldn’t hand
> code any better.

> Em geral, as implementações de C++ obedecem ao princípio de zero sobrecarga: 
> o que você não usa, não paga. E mais: o que você usa, não poderia entregar código
> melhor.

As another example, the following code is taken from an audio decoder. The
decoding algorithm uses the linear prediction mathematical operation to
estimate future values based on a linear function of the previous samples. This
code uses an iterator chain to do some math on three variables in scope: a
`buffer` slice of data, an array of 12 `coefficients`, and an amount by which
to shift data in `qlp_shift`. We’ve declared the variables within this example
but not given them any values; although this code doesn’t have much meaning
outside of its context, it’s still a concise, real-world example of how Rust
translates high-level ideas to low-level code.

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

To calculate the value of `prediction`, this code iterates through each of the
12 values in `coefficients` and uses the `zip` method to pair the coefficient
values with the previous 12 values in `buffer`. Then, for each pair, we
multiply the values together, sum all the results, and shift the bits in the
sum `qlp_shift` bits to the right.

Para calcular o valor de `prediction`, este código repete cada um dos 12 valores 
em `coefficients` e usa o método `zip` para emparelhar os valores do coeficiente 
com os 12 valores anteriores em `buffer`. Então, para cada par, multiplicamos os 
valores juntos, somamos todos os resultados e alteramos os bits na soma `qlp_shift` 
bits para a direita.

Calculations in applications like audio decoders often prioritize performance
most highly. Here, we’re creating an iterator, using two adaptors, and then
consuming the value. What assembly code would this Rust code compile to? Well,
as of this writing, it compiles down to the same assembly you’d write by hand.
There’s no loop at all corresponding to the iteration over the values in
`coefficients`: Rust knows that there are 12 iterations, so it “unrolls” the
loop. *Unrolling* is an optimization that removes the overhead of the loop
controlling code and instead generates repetitive code for each iteration of
the loop.

Cálculos em aplicativos como decodificadores de áudio geralmente priorizam mais o 
desempenho. Aqui, estamos criando um iterador, usando dois adaptadores e consumindo 
o valor. Em qual código _assembly_ , esse código Rust, teria gerado (compilado)? 
Bem, no momento da redação deste documento, ele é compilado no mesmo assembly que 
você escreveria à mão. Não existe um loop que corresponda à iteração sobre os valores 
em `coefficients`: Rust sabe que existem 12 iterações; portanto,"unrolls" (desenrola) 
o loop. *Unrolling* é uma otimização que remove a sobrecarga do código de controle do 
loop e gera um código repetitivo para cada iteração do loop.

All of the coefficients get stored in registers, which means accessing the
values is very fast. There are no bounds checks on the array access at runtime.
All these optimizations that Rust is able to apply make the resulting code
extremely efficient. Now that you know this, you can use iterators and closures
without fear! They make code seem like it’s higher level but don’t impose a
runtime performance penalty for doing so.

Todos os coeficientes são armazenados nos registradores, o que significa que o 
acesso aos valores é muito rápido. Não há verificações de limites no acesso à matriz 
em tempo de execução. Todas essas otimizações que Rust pode aplicar tornam o código 
resultante extremamente eficiente. Agora que você sabe disso, pode usar iteradores e 
closures sem medo! Eles fazem o código parecer um nível mais alto, mas não impõem uma 
penalidade de desempenho em tempo de execução por isso.

## Summary
## Resumo

Closures and iterators are Rust features inspired by functional programming
language ideas. They contribute to Rust’s capability to clearly express
high-level ideas at low-level performance. The implementations of closures and
iterators are such that runtime performance is not affected. This is part of
Rust’s goal to strive to provide zero-cost abstractions.

Closures e iteradores são recursos Rust inspirados em idéias da linguagem de programação 
funcionais. Elas contribuem para a capacidade de Rust em expressar claramente idéias 
de alto nível com desempenho de baixo nível. As implementações de closures e iteradores 
são tais que o desempenho do tempo de execução não é afetado. Isso faz parte do objetivo 
do Rust de fornecer abstrações de custo zero.

Now that we’ve improved the expressiveness of our I/O project, let’s look at
some more features of `cargo` that will help us share the project with the
world.

Agora que melhoramos a expressividade do nosso projeto de E/S, vejamos mais alguns recursos 
de `cargo` que nos ajudarão a compartilhar o projeto com o mundo.