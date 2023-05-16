## Entrar em `panic!` ou Não Entrar em `panic!`

Então como você decide quando entrar em `panic!` e quando você deveria retornar
um `Result`? Quando o código entra em pânico, não há maneira de se recuperar. Você
poderia chamar `panic!` para qualquer situação de erro, tendo uma maneira de se recuperar
ou não, mas então você estaria decidindo no lugar do código que chama seu código
que a situação é irrecuperável. Quando você decide retornar um valor de `Result`,
você lhe dá opções em vez de tomar a decisão por ele. O código
que chama seu código pode tentar se recuperar de uma maneira que é apropriada para
a situação, ou ele pode decidir que um valor de `Err` nesse caso é irrecuperável,
chamando `panic!` e transformando seu erro recuperável em um irrecuperável.
Portanto, retornar `Result` é uma boa escolha padrão quando você está definindo
uma função que pode falhar.

Em algumas situações é mais apropriado escrever código que entra em pânico em vez
de retornar um `Result`, mas eles são menos comuns. Vamos explorar porque é apropriado
entrar em pânico em alguns exemplos, protótipos de código e testes; depois situações
em que você como humano pode saber que um método não vai falhar, mas que o compilador não
tem como saber; e concluir com algumas diretrizes sobre como decidir entrar ou
não em pânico em código de biblioteca.

### Exemplos, Protótipos, e Testes São Todos Lugares em que É Perfeitamente Ok Entrar em Pânico

Quando você está escrevendo um exemplo para ilustrar algum conceito, ter código
de tratamento de erro robusto junto do exemplo pode torná-lo menos claro. Em exemplos,
é compreensível que uma chamada a um método como `unwrap` que poderia chamar `panic!`
apenas substitua a maneira como você trataria erros na sua aplicação,
que pode ser diferente baseado no que o resto do seu código está fazendo.

De forma semelhante, os métodos `unwrap` e `expect` são bem úteis ao fazer
protótipos, antes de você estar pronto para decidir como tratar erros. Eles deixam
marcadores claros no seu código para quando você estiver pronto para tornar
seu programa mais robusto.

Se uma chamada de método falha em um teste, queremos que o teste inteiro falhe,
mesmo se esse método não é a funcionalidade sendo testada. Como `panic!` é o modo
que um teste é marcado como falha, chamar `unwrap` ou `expect` é exatamente o que
deveria acontecer.


### Casos em que Você Tem Mais Informação Que o Compilador

Seria também apropriado chamar `unwrap` quando você tem outra lógica que
assegura que o `Result` vai ter um valor `Ok`, mas essa lógica não é algo
que o compilador entenda. Você ainda vai ter um valor de `Result` que precisa 
lidar: seja qual for a operação que você está chamando, ela ainda tem uma possibilidade
de falhar em geral, mesmo que seja logicamente impossível que isso ocorra nessa 
situação particular. Se você consegue assegurar ao inspecionar manualmente o código que
você nunca terá uma variante `Err`, é perfeitamente aceitável chamar `unwrap`.
Aqui temos um exemplo:

```rust
use std::net::IpAddr;

let home = "127.0.0.1".parse::<IpAddr>().unwrap();
```

Nós estamos criando uma instância `IpAddr` ao analisar uma string *hardcoded*. Nós
podemos ver que `127.0.0.1` é um endereço de IP válido, então é aceitável usar 
`unwrap` aqui. No entanto, ter uma string válida *hardcoded* não muda o tipo retornado
pelo método `parse`: ainda teremos um valor de `Result`, e o compilador ainda 
vai nos fazer tratar o `Result` como se a variante `Err` fosse uma
possibilidade, porque o compilador não é inteligente o bastante para ver que essa string
é sempre um endereço IP válido. Se a string de endereço IP viesse de um usuário ao invés
de ser *hardcoded* no programa, e portanto, de fato tivesse uma possibilidade de falha, nós
definitivamente iríamos querer tratar o `Result` de uma forma mais robusta.


### Diretrizes para Tratamento de Erro

É aconselhável fazer que seu código entre em `panic!` quando é possível que
ele entre em um mau estado. Nesse contexto, mau estado é quando
alguma hipótese, garantia, contrato ou invariante foi quebrada, tal como
valores inválidos, valores contraditórios, ou valores faltando que são passados
a seu código - além de um ou mais dos seguintes:

* O mau estado não é algo que é *esperado* que aconteça ocasionalmente.
* Seu código após certo ponto precisa confiar que ele não está nesse mau estado.
* Não há uma forma boa de codificar essa informação nos tipos que você usa.

Se alguém chama seu código e passa valores que não fazem sentido, a melhor escolha
talvez seja entrar em `panic!` e alertar a pessoa usando sua biblioteca do bug no
código dela para que ela possa consertá-la durante o desenvolvimento. Similarmente,
`panic!` é em geral apropriado se você está chamando código externo que está fora
do seu controle e ele retorna um estado inválido que você não tem como consertar.

Quando se chega a um mau estado, mas isso é esperado que aconteça não importa
quão bem você escreva seu código, ainda é mais apropriado retornar um `Result`
a fazer uma chamada a `panic!`. Um exemplo disso é um *parser* recebendo dados
malformados ou uma requisição HTTP retornando um status que indique que você atingiu
um limite de taxa. Nesses casos, você deveria indicar que falha é uma possibilidade
esperada ao retornar um `Result` para propagar esses estados ruins para cima,
de forma que o código que chamou seu código pode decidir como tratar o problema.
Entrar em `panic!` não seria a melhor maneira de lidar com esses casos.

Quando seu código realiza operações em valores, ele deveria verificar que os valores
são válidos primeiro, e entrar em `panic!` caso não sejam. Isso é 
em boa parte por razões de segurança: tentar operar em dados inválidos pode expor seu
código a vulnerabilidades. Essa é a principal razão para a biblioteca padrão entrar em
`panic!` se você tentar um acesso de memória fora dos limites: tentar acessar memória 
que não pertence à estrutura de dados atual é um problema de segurança comum. Funções 
frequentemente tem *contratos*: seu comportamento somente é garantido se os inputs cumprem
requerimentos específicos. Entrar em pânico quando o contrato é violado faz sentido 
porque uma violação de contrato sempre indica um bug da parte do chamador, e não é o tipo 
de erro que você quer que seja tratado explicitamente. De fato, 
não há nenhuma maneira razoável para o código chamador se recuperar: os *programadores*
que precisam consertar o código. Contratos para uma função, especialmente quando uma
violação leva a pânico, devem ser explicados na documentação da API da função.

No entanto, ter várias checagens de erro em todas suas funções pode ser verboso
e irritante. Felizmente, você pode usar o sistema de tipos do Rust (e portanto a
checagem que o compilador faz) para fazer várias dessas checagens para você. Se
sua função tem um tipo particular como parâmetro, você pode continuar com a lógica
do seu código sabendo que o compilador já assegurou que você tem um valor válido.
Por exemplo, se você tem um tipo em vez de uma `Option`, seu programa espera
ter *algo* ao invés de *nada*. Seu código não precisa tratar dois casos para
as variantes `Some` e `None`: ele vai somente ter um caso para definitivamente ter
um valor. Um código que tente passar nada para sua função não vai nem compilar,
então sua função não precisa checar esse caso em tempo de execução. Outro exemplo é usar
um tipo de inteiro sem sinal como `u32`, que assegura que o parâmetro nunca é
negativo.


### Criando Tipos Customizados para Validação

Vamos dar um passo além na ideia de usar o sistema de tipos de Rust para assegurar que temos
um valor válido e ver como criar um tipo customizado para validação.
Lembre do jogo de adivinhação no Capítulo 2 onde nosso código pedia ao usuário 
para adivinhar um número entre 1 e 100. Nós nunca validamos que o chute do usuário
fosse entre esses números antes de compará-lo com o número secreto; nós somente 
validamos que o chute era positivo. Nesse caso, as consequências não foram tão
drásticas: nosso output de "Muito alto" ou "Muito baixo" ainda estariam corretos. Seria
uma melhoria útil guiar o usuário para chutes válidos, e ter um comportamento distinto
quando um usuário chuta um número fora do limite e quando um usuário digita letras, por exemplo.

Uma maneira de fazer isso seria interpretar o chute como um `i32` em vez de
somente um `u32` para permitir números potencialmente negativos, e então adicionar
uma checagem se o número está dentro dos limites, conforme a seguir:

```rust,ignore
loop {
    // snip

    let palpite: i32 = match palpite.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    if palpite < 1 || palpite > 100 {
        println!("O número secreto vai estar entre 1 e 100.");
        continue;
    }

    match palpite.cmp(&numero_secreto) {
    // snip
}
```

A expressão `if` checa se nosso valor está fora dos limites, informa o usuário
sobre o problema, e chama `continue` para começar a próxima iteração do loop
e pedir por outro chute. Depois da expressão `if` podemos proceder com as 
comparações entre `palpite` e o número secreto sabendo que `palpite` está 
entre 1 e 100.

No entanto, essa não é a solução ideal: se fosse absolutamente crítico que o
programa somente operasse em valores entre 1 e 100, e ele tivesse várias funções
com esse requisito, seria tedioso (e potencialmente impactante na performance)
ter uma checagem dessa em cada função.

Em vez disso, podemos fazer um novo tipo e colocar as validações em uma função
para criar uma instância do tipo em vez de repetir as validações em todo lugar.
Dessa maneira, é seguro para funções usarem o novo tipo nas suas assinaturas e 
confidentemente usar os valores que recebem. A Listagem 9-9  mostra uma maneira de 
definir um tipo `Palpite` que vai somente criar uma instância de `Palpite` se a função
`new` receber um valor entre 1 e 100:

```rust
pub struct Palpite {
    valor: u32,
}

impl Palpite {
    pub fn new(valor: u32) -> Palpite {
        if valor < 1 || valor > 100 {
            panic!("Valor de chute deve ser entre 1 e 100, recebi {}.", valor);
        }

        Palpite {
            valor
        }
    }

    pub fn valor(&self) -> u32 {
        self.valor
    }
}
```

<span class="caption">Listagem 9-9: Um tipo `Palpite` que somente funciona com valores
entre 1 e 100.</span>

Primeiro, definimos uma struct chamada `Palpite` que tem um campo chamado `valor`
que guarda um `u32`. Isso é onde o número vai ser guardado.

Então nós implementamos uma função associada chamada `new` em `Palpite` que cria
instâncias de valores `Palpite`. A função `new` é definida a ter um parâmetro
chamado `valor` de tipo `u32` e retornar um `Palpite`. O código no corpo da função
`new` testa para ter certeza que `valor` está entre 1 e 100. Se `valor` não passa
nesse teste, fazemos uma chamada a `panic!`, que vai alertar ao programador que
está escrevendo o código chamando a função que ele tem um bug que precisa ser 
corrigido, porque criar um `Palpite` com um `valor` fora desses limites violaria
o contrato em que `Palpite::new` se baseia. As condições em que `Palpite::new` pode 
entrar em pânico devem ser discutidas na sua documentação da API voltada ao público;
no Capítulo 14 nós cobriremos convenções de documentação indicando a possibilidade de um `panic!`
na documentação de API. Se `valor` de fato passa no
teste, criamos um novo `Palpite` com o campo `valor` preenchido com o parâmetro
`valor` e retornamos o `Palpite`.

Em seguida, implementamos um método chamado `valor` que pega `self` emprestado, não
tem nenhum outro parâmetro, e retorna um `u32`. Esse é o tipo de método às vezes
chamado de *getter*, pois seu propósito é pegar um dado de um dos campos e o retornar.
Esse método público é necessário porque o campo `valor` da struct `Palpite` é privado.
É importante que o campo `valor` seja privado para que código usando a struct `Palpite`
não tenha permissão de definir o valor de `valor` diretamente: código de fora do módulo
*deve* usar a função `Palpite::new` para criar uma instância de `Palpite`, o que certifica
que não há maneira de um `Palpite` ter um `valor` que não foi checado pelas condições
definidas na função `Palpite::new`.

Uma função que tem um parâmetro ou retorna somente números entre 1 e 100 pode
então declarar na sua assinatura que ela recebe ou retorna um `Palpite` em vez
de um `u32` e não precisaria fazer nenhuma checagem adicional no seu corpo.

## Resumo

As ferramentas de tratamento de erros de Rust são feitas para te ajudar a escrever
código mais robusto. A macro `panic!` sinaliza que seu programa está num estado que
não consegue lidar e deixa você parar o processo ao invés de tentar prosseguir com
valores inválidos ou incorretos. O enum `Result` usa o sistema de tipos de Rust para 
indicar que operações podem falhar de uma maneira que seu código pode se recuperar. 
Você pode usar `Result` para dizer ao código que chama seu código que ele precisa
tratar potenciais sucessos ou falhas também. Usar `panic!` e `Result` nas situações
apropriadas fará seu código mais confiável em face aos problemas inevitáveis.

Agora que você viu as maneiras úteis em que a biblioteca padrão usa genéricos com
os enums `Option` e `Result`, nós falaremos como genéricos funcionam e como você
pode usá-los em seu código no próximo capítulo.
