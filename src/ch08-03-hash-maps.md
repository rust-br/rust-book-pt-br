## Hash Maps

A última das nossas coleções comuns é o *hash map*. O tipo `HashMap <K, V>`
armazena um mapeamento de chaves do tipo `K` para valores do tipo` V`. Ele faz isso através de um
*hashing function*, que determina como ele coloca essas chaves e valores em
memória. Muitas linguagens de programação diferentes suportam este tipo de 
estrutura de dados, mas muitas vezes com um nome diferente: hash, map, object, hash table ou
associative array, apenas para citar alguns.

Os Hash maps são úteis para quando você deseja poder procurar dados sem uso de
índice, como você pode com vetores, mas usando uma chave que pode ser de qualquer tipo. Por 
exemplo, em um jogo, você poderia acompanhar a pontuação de cada equipe em um hash map
onde cada chave é o nome de uma equipe e os valores são cada pontuação da equipe. Dado um
nome da equipe, você pode recuperar sua pontuação.

Examinaremos a API básica dos hash map neste capítulo, mas há muitos
mais coisas escondidas nas funções definidas no `HashMap` pela biblioteca
padrão. Como sempre, verifique a documentação da biblioteca padrão para mais
informação. 

### Criando um novo Hash Map

Podemos criar um `HashMap` vazio com `new`, e adicionar elementos com `insert`.
Aqui, estamos acompanhando as pontuações de duas equipes cujos nomes são Blue e
Yellow. A equipe blue começará com 10 pontos e a equipe yellow começa com
50:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

Observe que precisamos primeiro `use` o `HashMap` da parte de coleções da
biblioteca padrão. De nossas três coleções comuns, esta é a de menor
frequencia de uso, por isso não está inclusa nos recursos importados automaticamente no
prelúdio. Os Hash maps também têm menos suporte da biblioteca padrão; não há
macro embutida para construí-los, por exemplo.

Assim como os vetores, os mapas hash armazenam seus dados no heap. Este `HashMap` tem
chaves do tipo `String` e valores do tipo `i32`. Como vetores, os hash maps são
homogêneos: todas as chaves devem ter o mesmo tipo e todos os valores
devem ter o mesmo tipo.

Outra maneira de construir um hash map é usando o método `collect` em um
vetor de tuplas, onde cada tupla consiste de uma chave e seu valor. O 
método `collect` reúne dados em vários tipos de coleção, incluindo
`HashMap`. Por exemplo, se tivéssemos os nomes das equipes e as pontuações iniciais em dois
vetores separados, podemos usar o método `zip` para criar um vetor de tuplas
onde “Blue” é emparelhado com 10, e assim por diante. Então podemos usar o método `collect`
para transformar esse vetor de tuplas em um `HashMap`:

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

A anotação de tipo `HashMap <_, _>` é necessária aqui porque é possível
`collect` em muitas estruturas de dados diferentes, e Rust não sabe qual você
deseja, a menos que você especifique. Para os parâmetros de tipo, para os tipos de chave e valor,
no entanto, usamos underscores e Rust pode inferir os tipos que o hash map
contém com base nos tipos de dados no vetor.

### Hash Maps e Ownership

Para os tipos que implementam a `Copy` trait, como `i32`, os valores são copiados
no hash map. Para valores owned como `String`, os valores serão movidos e
o hash map será o owner desses valores:


```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name e field_value são inválidos neste ponto
```


Não poderíamos usar as ligações `field_name` e` field_value` depois
que foram transferidos para o hash map com a chamada para `insert`.

Se inserimos referências a valores no hash map, os próprios valores
não serão movido para o hash map. Os valores que as referências apontam devem ser
válidos pelo menos enquanto o hash map seja válido, no entanto. Falaremos mais
sobre esses problemas na seção Lifetimes do Capítulo 10. 

### Acessando Valores em um Hash Map

Podemos obter um valor do hash map fornecendo a chave para o método `get`:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

Aqui, `score` terá o valor que está associado à equipe Blue, e o
resultado será `Some(&10)`. O resultado está envolvido em `Some` porque `get`
retorna `Option<&V>`; se não houver valor para essa chave no hash map, `get`
retornará `None`. O programa precisará lidar com `Option` em uma das
formas que abordamos no Capítulo 6.

Podemos iterar sobre cada par chave/valor em um hash map de uma maneira similar à que 
fazemos com vetores, usando um loop `for`:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

Isso imprimirá cada par, em uma ordem arbitrária:

```text
Yellow: 50
Blue: 10
```

### Atualizando um Hash Map

Embora o número de chaves e valores sejam crescentes, cada chave individual pode apenas
tem um valor associado a ele por vez. Quando queremos mudar os dados em
um hash map, temos que decidir como lidar com o caso quando uma chave já possui um
valor atribuído. Poderíamos optar por substituir o valor antigo pelo novo valor,
desconsiderando completamente o valor antigo. Poderíamos escolher manter o valor antigo
e ignorar o novo valor, e apenas adicionar o novo valor se a chave ainda *não*
tem um valor. Ou podemos combinar o valor antigo ao valor novo.
Vejamos como fazer cada um desses!

#### Sobrescrevendo um Valor

Se inserimos uma chave e um valor em um hash map, então  se inserir essa mesma chave com
um valor diferente, o valor associado a essa chave será substituído. Eembora 
o seguinte código chame `insert` duas vezes, o hash map só conterá
um par de chave/valor porque inserimos o valor da chave da equipe Blue
ambas as vezes:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

Isso imprimirá `{"Blue": 25}`. O valor original de 10 foi substituído.

#### Insira Apenas se a Chave Não Possui Valor

É comum querer verificar se uma determinada chave tem um valor e, se 
não tiver, inserir um valor para ela. Os Hash maps possuem uma API especial para isso, chamada
`entry`, que leva a chave que queremos verificar como um argumento. O valor de retorno
da função `entry` é um enum, `Entry`, que representa um valor que pode
ou não existir. Digamos que queremos verificar se a chave para o time Yellow
tem um valor associado a ela. Se não tiver, queremos inserir o valor
50, e o mesmo para a equipe Blue. Com a API de entrada, o código irá parecer 
com:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

O método `or_insert` em `Entry` retorna o valor para o `Entry` correspondente
se a chave existir, e se não, insere seu argumento como o novo valor para
esta chave e retorna a `Entry` modificada. Isso é muito mais limpo do que escrever
a lógica por nós mesmos e, além disso, trabalha-se de forma mais limpa com o borrow checker.

Este código imprimirá `{"Yellow": 50, "Blue": 10}`. A primeira chamada para `entry`
irá inserir a chave para a equipe Yellow com o valor 50, uma vez que o time Yellow
já não possua um valor. A segunda chamada para `entry` não vai mudar
o hash map pois o time Blue já possui o valor 10.

#### Atualize um Valor com Base no Valor Antigo

Outro caso de uso comum para hash maps é procurar o valor de uma chave e, em seguida, atualiza-la
, com base no valor antigo. Por exemplo, se quisermos contar quantas vezes
cada palavra apareceu em algum texto, podemos usar um hash map com as palavras como chaves
e incrementar o valor para acompanhar quantas vezes vimos essa palavra.
Se esta é a primeira vez que vimos uma palavra, primeiro inseriremos o valor `0`.

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

Isso imprimirá `{"world": 2, "hello": 1, "wonderful": 1}`. O método `or_insert`
na verdade retorna uma referência mutável (`& mutV`) para o valor desta
chave. Aqui nós armazenamos essa referência mutável na variável `count`, então, 
para poder atribuir esse valor, devemos primeiro desreferenciar `count` usando o asterisco
(`*`). A referência mutável fica fora do escopo no final do loop `for`, então
todas essas mudanças são seguras e permitidas pelas regras de borrow.

### Funções Hashing

Por padrão, `HashMap` usa uma função de hashing criptográficamente segura que pode
fornecer resistência aos ataques de Negação de Serviço (DoS). Este não é o algoritmo 
mais rápido de hashing por aí, mas a compensação por uma melhor segurança que vem
com a queda na performance vale a pena. Se você testar a velocidade do seu código e encontrar
que a função de hash padrão é muito lenta para seus propósitos, você pode mudar para
outra função especificando um *hasher* diferente. Um hasher é um tipo que
implementa a trait `BuildHasher`. Vamos falar sobre traits e como
implementá-los no Capítulo 10. Você não precisa necessariamente implementar o seu próprio
hasher do zero; crates.io tem bibliotecas de hashers de uso comum que 
outras pessoas compartilharam lá.

## Sumário

Vetores, strings e hash maps irão levá-lo longe em programas onde você precisa
armazenar, acessar e modificar dados. Aqui estão alguns exercícios que você deve estar
capacitado para resolver:


* Dada uma lista de inteiros, use um vetor e retorne a média, a mediana
   (quando classificado, o valor na posição do meio) e modo (o valor que
   ocorre com mais frequência; um hash map será útil aqui) da lista.
* Converta strings para Pig Latin, onde a primeira consoante de cada palavra é movida
   para o final da palavra adicionado um "ay" , então “first” se torna “irst-fay”.
   Palavras que começam com uma vogal recebem “hay” adicionado ao final (“apple”
   torna-se “apple-hay”). Lembre-se sobre a codificação UTF-8!
* Usando um hash map e vetores, crie uma interface de texto para permitir que um usuário adicione
   nomes de funcionários para um departamento da empresa. Por exemplo, “Add Sally to
   Engineering” ou “Add Amir to Sales”. Em seguida, deixe o usuário recuperar uma lista de todas
   as pessoas de um departamento ou todas as pessoas na empresa por departamento, ordenadas
   alfabeticamente.

A documentação da API da biblioteca padrão descreve métodos que esses tipos possuem
que será útil para esses exercícios!

Estamos entrando em programas mais complexos onde as operações podem falhar, o que significa
que é um momento perfeito para passar pelo tratamento de erros em seguida!
