# STATUS ATUAL

A segunda edição de "The Rust Programming Language" ("A linguagem de
programação Rust") está cada vez mais perto da impressão!

Isso significa que não podemos fazer grandes mudanças nos capítulos
que estão na coluna "Frozen" ("congelada") ou nas colunas a direita
dela [no board do nosso Projeto][proj]. [Issues][issues] ou
[Pull Requests][pulls] que sejam de capítulos congelados são
bem-vindos, mas serão fechados até começarmos a trabalhar na terceira
edição. Agradecemos!

[proj]: https://github.com/rust-lang/book/projects/1
[issues]: https://github.com/rust-lang/book/issues/
[pulls]: https://github.com/rust-lang/book/pulls/

# The Rust Programming Language

[![Build Status](https://travis-ci.org/rust-lang/book.svg?branch=master)](https://travis-ci.org/rust-lang/book)

Este repositório contém duas edições de “The Rust Programming
Language”; nós recomendamos começar com a segunda edição.

A segunda edição é uma revisão que será impressa por No Starch Press,
disponível por volta de maio de 2018. Vá a [No Starch][nostarch] para
ver as informações mais atualizadas sobre data de lançamento e fazer um
pedido.

[nostarch]: https://nostarch.com/rust

Você pode ler o livro de graça online! Veja as últimas revisões
[oficial], [beta], ou [recém desenvolvida] do livro de Rust (em
inglês). Esteja ciente que problemas nestas versões podem já ter sido
resolvidos neste repositório.

[oficial]: https://doc.rust-lang.org/stable/book/second-edition/
[beta]: https://doc.rust-lang.org/beta/book/second-edition/
[recém desenvolvida]: https://doc.rust-lang.org/nightly/book/second-edition/

[A primeira edição ainda pode ser lida aqui (versão em inglês)][first].

[first]: https://doc.rust-lang.org/book/


## Requirements

Compilar o livro requer [mdBook], de preferência a mesma versão que
[rust-lang/rust usa neste arquivo][rust-mdbook]. Para baixar:

[mdBook]: https://github.com/azerupi/mdBook
[rust-mdbook]: https://github.com/rust-lang/rust/blob/master/src/tools/rustbook/Cargo.toml

```bash
$ cargo install mdbook --vers [version-num]
```

## Building

Para compilar o livro, primeiro entre (`cd`) na pasta `first-edition`
ou `second-edition`, dependendo de qual edição do livro você quer
compilar. Então execute:

```bash
$ mdbook build
```

O resultado vai estar na subpasta `book`. Para verificar, abra no seu
browser.

_Firefox:_
```bash
$ firefox book/index.html                       # Linux
$ open -a "Firefox" book/index.html             # OS X
$ Start-Process "firefox.exe" .\book\index.html # Windows (PowerShell)
$ start firefox.exe .\book\index.html           # Windows (Cmd)
```

_Chrome:_
```bash
$ google-chrome book/index.html                 # Linux
$ open -a "Google Chrome" book/index.html       # OS X
$ Start-Process "chrome.exe" .\book\index.html  # Windows (PowerShell)
$ start chrome.exe .\book\index.html            # Windows (Cmd)
```

Para rodar os testes:

```bash
$ mdbook test
```

## Contribuir

Nos adoraríamos sua ajuda! Por favor, leia [CONTRIBUTING.md][contrib]
para aprender sobre tipos de contribuições que estamos buscando.

[contrib]: https://github.com/rust-lang/book/blob/master/CONTRIBUTING.md

### Translations

Nós amamos mais ainda ajudas na tradução da segunda edição do livro!
Veja a label [Translations] para se juntar aos esforços atualmente em
andamento. Abra uma nova issue para começar a trabalhar em uma nova
linguagem! Estamos aguardando [suporte de mdbook] para múltiplas línguas
antes de fazer merge de qualquer coisa, mas sinta-se a vontade para
começar! Os capítulos na [coluna "Frozen"] do projeto não terão maiores
mudanças, então, se começar com esses, você não terá retrabalho :)

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[suporte de mdbook]: https://github.com/azerupi/mdBook/issues/5
[coluna congelada]: https://github.com/rust-lang/book/projects/1

## No Starch

Como a segunda edição do livro vai ser publicada por No Starch, nós
primeiro interagimos aqui, depois enviados o texto a No Starch. Então
fazem a edição, e nós editamos aqui.

Existe uma pasta, *nostarch*, que corresponde ao texto no sistema da
No Starch.

Quando começamos a trabalhar com a No Starch em um arquivo de word, nós
também colocamos os arquivos no repositório na pasta *nostarch/odt*.
Para extrair o texto do arquivo de word em markdown, para enviar
mudanças para o livro online:

1. Abra o arquivo word no LibreOffice
1. Aceite todas as mudanças encontradas
1. Salve como Microsoft Word 2007-2013 XML (.docx) na pasta *tmp*
1. Execute `./doc-to-md.sh`
1. Verifique as mudanças no arquivo markdown na pasta *nostarch* e
   copie essas mudanças para a pasta *src* em seu devido lugar.

## Graphviz dot

Isto é basicamente para referências para Carol, pois ela precisa sempre
verificar.

A gente usa [Graphviz](http://graphviz.org/) para alguns diagramas no
livro. A origem desses arquivos é a pasta `dot`. Para transformar um
arquivo `dot`, por exemplo, `dot/trpl04-01.dot`, em `svg`, execute:

```bash
$ dot dot/trpl04-01.dot -Tsvg > src/img/trpl04-01.svg
```

No SVG gerado, remova atributos de largura e altura da tag `svg` e
coloque um atributo `viewBox` com `0.00 0.00 1000.00 1000.00` ou outro
valor que não corte a imagem.

## Verificação Ortográfica

Para procurar erros de ortografia no código fonte, você pode usar o
script `spellcheck.sh`. Precisa de um dicionário de palavras válidas,
que existe em `dictionary.txt`. Se o script der um falso erro (digamos,
você usa a palavra `BTreeMap`, que o script considera inválida), você
precisa adicionar a palavra ao arquivo `dictionary.txt` (mantenha a
ordem já usada no arquivo).
