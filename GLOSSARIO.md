# Glossário de tradução (proposta)

Este arquivo propõe um vocabulário compartilhado para a tradução pt-BR.
Ele não substitui o [CONTRIBUTING.md](CONTRIBUTING.md): só fecha o que a
comunidade já pratica no texto e o que ainda está em discussão.

É uma **proposta**. Nada aqui está decidido até a comunidade revisar.
Discussão principal: issue de padronização
([#41](https://github.com/rust-br/rust-book-pt-br/issues/41)).

## Princípios

1. Não traduzir termos técnicos consagrados nem blocos de código.
2. Estrangeirismo na prosa vai em itálico (`_ownership_`).
3. Sentido figurado vai entre aspas duplas.
4. Na dúvida, abrir issue no formato `Como traduzir a palavra/termo "x"?`.

## Manter em inglês

| Termo | Notas | Issue |
|---|---|---|
| *ownership* | Não usar posse/propriedade até haver acordo | [#54](https://github.com/rust-br/rust-book-pt-br/issues/54) |
| *borrowing*, *borrow* | Já aparece no título do cap. 4 | |
| *borrow checker* | | [#59](https://github.com/rust-br/rust-book-pt-br/issues/59) |
| *lifetime* | O título do cap. 10 mistura “tempos de vida”; na prosa, preferir *lifetime* | [#45](https://github.com/rust-br/rust-book-pt-br/issues/45) (fechada) |
| *trait* | Masculino | |
| *crate* | Masculino | [#67](https://github.com/rust-br/rust-book-pt-br/issues/67) (fechada) |
| *move* | | [#58](https://github.com/rust-br/rust-book-pt-br/issues/58) |
| *drop* | | [#56](https://github.com/rust-br/rust-book-pt-br/issues/56) (fechada) |
| *unsafe* | | |
| *smart pointer* | O cap. 15 já usa “ponteiros inteligentes” no título, com o termo inglês ao lado | [#51](https://github.com/rust-br/rust-book-pt-br/issues/51) |
| *Box*, *Rc*, *RefCell* | Não flexionar nem traduzir o nome do tipo | [#53](https://github.com/rust-br/rust-book-pt-br/issues/53) |

## Traduzir

| Inglês | pt-BR | Notas |
|---|---|---|
| Summary | Resumo | Fecho de capítulo |
| Filename | Arquivo | Antes de um bloco de código |
| Listing | Listagem | |
| Warning | Aviso | Evitar misturar com _warning_ |
| slice | fatia | Já usado no cap. 4; o título ainda usa *_Slices_* |
| error handling | tratamento de erros | |
| pattern matching | casamento de padrões | Já usado no cap. 6 |
| Getting Started | Começando | |
| Guessing Game | Jogo de Adivinhação | |

## Gênero

Palavras em inglês não têm gênero. Para manter o texto consistente:

- femininas: *enum* (enumeração), *struct* (estrutura)
- masculinos: *crate*, *trait*, *Rust*, *Cargo*

Exemplos: “uma *enum*”, “a *struct*”, “um *crate*”, “o *trait*”.

## Código e listings

- Não traduzir blocos ` ```rust `, identificadores, APIs, nomes de crates nem comandos (`cargo test`).
- Não traduzir a saída do compilador.
- Comentários dentro dos listings: só traduzir se o capítulo vizinho já fizer isso; senão, manter o inglês.

## Como alterar este arquivo

1. Abra ou comente uma issue de termo.
2. Envie um PR pequeno só com a linha do glossário e, se preciso, um exemplo.
3. Depois que o termo for aceito, a revisão dos capítulos já traduzidos pode ser feita à parte.
