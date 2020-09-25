<!-- # Appendix G - How Rust is Made and “Nightly Rust” -->
# Apêndice G - Como é feito o Rust e o "Rust Nightly"

Este apêndice é sobre como o Rust é feito e como isso afeta você como um
desenvolvedor Rust. Mencionamos que a saída deste livro foi gerada pelo Rust
estável (*Stable*) na versão 1.21.0, mas todos os exemplos que compilam devem
continuar sendo compilados em qualquer versão estável do Rust mais recente.
Esta seção explica como garantimos que isso seja verdade!

<!-- ### Stability Without Stagnation -->
### Estabilidade sem estagnação

Como linguagem, o Rust se preopcupa muito com a estabilidade do seu código.
Queremos que o Rust seja uma base sólida sobre a qual você possa construir,
e se as coisas estivessem mudando constantemente, isso seria impossível. Ao
mesmo tempo, se não pudermos experimentar novos recursos, poderemos descobrir
falhas importantes somente após o lançamento, quando não podemos mais mudar as
coisas.

Nossa solução para esse problema é o que chamamos de "estabilidade sem
estagnação", e somos guiados pelo seguinte princípio: você nunca deve ter medo
de atualizar para uma nova versão do Rust. Cada atualização deve ser indolor,
mas também deve trazer novos recursos, menos bugs e tempos de compilação mais
rápidos.

<!-- ### Choo, Choo! Release Channels and Riding the Trains -->
### Tchu, Tchu! Canais de _Release_ e Passeios de Trem

O desenvolvimento do Rust opera em um "*train scheduler*" (Horário de trem).
Isto é, todo o desenvolvimento é feito na _branch_ `master` do repositório do
Rust. As versões seguem um modelo de trem de liberação de 
software (_train model_) que têm sido usado pela Cisco IOS e outros projetos de software. Existem três canais de _release_ para o Rust.

<!-- 
* Nightly
* Beta
* Stable
-->

* Nightly
* Beta
* Stable (Estável)

A maioria dos desenvolvedores Rust usa principalmente o canal estável
(_Stable_), mas aqueles que desejam usar novos recursos experimentais
podem usar o _Nightly_ ou o _Beta_.

Aqui está um exemplo de como o processo de desenvolvimento e lançamento
(_release_) funciona: vamos supor que a equipe do Rust esteja trabalhando no
lançamento do Rust 1.5. Esse lançamento ocorreu em dezembro de 2015, mas nos
fornecerá números de versão realistas. Um novo recurso foi adicionado ao Rust:
um novo commit é feito na _branch_ `master`.
Todas as noites, uma nova versão _Nightly_ do Rust é produzida. Todo dia é um
dia de lançamento e esses lançamentos são criados automaticamente por nossa
infraestrutura de lançamento. Assim, com o passar do tempo, nossos lançamentos
ficam assim, uma vez por noite:

```text
nightly: * - - * - - *
```

A cada seis semanas, chega a hora de preparar uma nova _release_!
A _branch_ `beta` do repositório do Rust é ramificada a partir da _branch_
`master` usada pelo _Nightly_. Agora existem duas _releases_.

```text
nightly: * - - * - - *
                     |
beta:                *
```

A maioria dos usuários do Rust não usa ativamente as versões beta, mas faz
testes com versões beta no sistema de IC (integração contínua) para ajudar o
Rust a descobrir possíveis regressões. Enquanto isso, ainda há uma _release_
todas as noites:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Agora digamos que uma regressão seja encontrada. Ainda bem que tivemos algum
tempo para testar a versão beta antes da regressão se tornar uma versão estável!
A correção é aplicada à _branch_ `master`, de modo que todas as noites é
corrigida e, em seguida, a correção é portada para a _branch_ `beta`, e uma nova
versão beta é produzida:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Seis semanas depois da criação da primeira versão beta, é hora de uma versão
estável! A _branch_ `stable` é produzida a partir da _branch_` beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Viva! Rust 1.5 está feito! No entanto, esquecemos uma coisa: como as seis
semanas se passaram, também precisamos de uma nova versão beta da *próxima*
versão do Rust, 1.6.
Então, depois que a _branch_ `stable` é criada a partir da `beta`,
a próxima versão da `beta` é criada a partir da `nightly` novamente:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Isso é chamado de "_train model_" (modelo de trem) porque a cada seis semanas,
uma _release_ "sai da estação", mas ainda precisa percorrer o canal beta antes
de chegar como uma _release_ estável.

O Rust é lançando a cada seis semanas, como um relógio.
Se você souber a data de um lançamento do Rust, poderá saber a data do próximo:
seis semanas depois. Um aspecto interessante de ter lançamentos agendados a cada
seis semanas é que o próximo trem estará chegando em breve. Se um recurso falhar
em uma versão específica, não há necessidade de se preocupar: outra está
acontecendo em pouco tempo! Isso ajuda a reduzir a pressão para ocultar recursos
possivelmente "não polidos" perto do prazo de lançamento.

Graças a esse processo, você sempre pode verificar a próxima versão do Rust e
verificar por si mesmo que é fácil fazer uma atualização para a mesma: se uma
versão beta não funcionar conforme o esperado, você pode reportar à equipe e ter
isso corrigido antes do próximo lançamento estável!
A quebra de uma versão beta é relativamente rara, mas o `rustc` ainda é um
software, e bugs existem.

<!-- ### Unstable Features -->
### Recursos instáveis

Há mais um problema neste modelo de lançamento: recursos instáveis.
O Rust usa uma técnica chamada "sinalizadores de recursos" para determinar quais
recursos estão ativados em uma determinada _release_. Se um novo recurso estiver
em desenvolvimento ativo, ele pousará na _branch_ `master` e, logo, no
_Nightly_, mas atrás de um sinalizador de recurso.
Se você, como usuário, deseja experimentar o recurso de trabalho em andamento,
pode, mas deve estar usando uma versão _Nightly_ do Rust e anotar seu
código-fonte com o sinalizador apropriado para ativar.

Se você estiver usando uma versçao beta ou estável do Rust, você não pode usar
qualquer sinalizador de recurso. Essa é a chave que nos permite usar de forma
prática os novos recursos antes de declará-los estáveis para sempre. Aqueles que
desejam optar pelo que há de mais moderno podem fazê-lo, e aqueles que desejam
uma experiência sólida podem se manter estáveis sabendo que seu código não será
quebrado. Estabilidade sem estagnação.

Este livro contém informações apenas sobre recursos estáveis, pois os recursos
em desenvolvimento ainda estão sendo alterados e certamente serão diferentes
entre quando este livro foi escrito e quando eles forem ativados em compilações
estáveis. Você pode encontrar documentação on-line para recursos do exclusivos
do _Nightly_ (_nightly-only_).

<!-- ### Rustup and the Role of Rust Nightly -->
### O Rustup e o papel do _Rust Nightly_

O Rustup facilita a troca entre os diferentes canais de _release_ do Rust,
global ou por projeto. Para instalar o _Rust Nightly_, por exemplo:

```text
$ rustup install nightly
```

Você pode ver todos os _toolchains_ (versões do Rust e componentes associados)
instalados com o `rustup` também.
Veja um exemplo nos computadores de seus autores:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Como pode ver, o _toolchain_ estável (_Stable_) é o padrão. A maioria dos
usuários do Rust usa o estável na maioria das vezes. Você pode querer usar o
estável na mioria das vezes, mas usará o _Nightly_ em um projeto específico
por se preocupar com um recurso de ponta.
Para fazer isso, você pode usar `rustup override` no diretório desse projeto
para definir que o _toolchain_ do _Nightly_ deve ser usado quando o `rustup` for
usado nesse diretório.

```text
$ cd ~/projects/needs-nightly
$ rustup override add nightly
```

Agora, toda que chamar `rustc` ou `cargo` dentro de `~/projects/needs-nightly`,
o `rustup` irá garantir que você esteja usando o _Rust Nightly_ ao invés do
padrão _Rust Stable_. Isso é útil quando você tem muitos projetos em Rust.

<!-- ### The RFC Process and Teams -->
### O Processo de RFC e Equipes

Então, como você aprende sobre esses novos recursos? O modelo de desenvolvimento
da Rust segue um processo de solicitação de comentários (RFC). Se você deseja
uma melhoria no Rust, pode escrever uma proposta, chamada RFC (Request For
Comments).

Qualquer um pode escrever RFCs para melhorar o Rust, e as propostas são
revisadas e discutidas pela equipe do Rust, que é composta por muitas subequipes
de tópicos. Há uma lista completa das equipes
[no site do Rust](https://www.rust-lang.org/en-US/team.html),
que inclui equipes para cada área do projeto: design de linguagem, implementação
do compilador, infraestrutura, documentação e muito mais. A equipe apropriada
lê a proposta e os comentários, escreve alguns comentários próprios e,
eventualmente, há um consenso para aceitar ou rejeitar o recurso.

Se o recurso for aceito, uma _Issue_ será aberta no repositório do Rust e alguém
poderá implementá-lo. A pessoa que a implementa pode muito bem não ser a pessoa
que propôs o recurso em primeiro lugar! Quando a implementação está pronta, ela
chega à _branch_ `master` atrás de um sinalizador de recurso, conforme
discutimos na seção "Recursos instáveis".

Depois de algum tempo, assim que os desenvolvedores do Rust que usam versões
_Nightly_ puderem experimentar o novo recurso, os membros da equipe discutirão o
recurso, como ele funciona no _Nightly_ e decidirão se ele deve se tornar parte
do Rust estável ou não. Se a decisão for sim, o portão do recurso será removido
e o recurso agora será considerado estável! Ele entra no próximo trem para uma
nova versão estável do Rust.
