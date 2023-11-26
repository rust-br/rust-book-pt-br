# A Linguagem de Programação Rust

<!-- [![Build Status](https://travis-ci.com/rust-lang/book.svg?branch=master)](https://travis-ci.com/rust-lang/book)

This repository contains the source of "The Rust Programming Language" book. -->

[![Build Status](https://travis-ci.com/rust-lang/book.svg?branch=master)](https://travis-ci.com/rust-lang/book)

Este repositório contém o código-fonte do livro "A Linguagem de Programação Rust".

<!-- [The book is available in dead-tree form from No Starch Press][nostarch]

[nostarch]: https://nostarch.com/rust

You can also read the book for free online. Please see the book as shipped with
the latest [stable], [beta], or [nightly] Rust releases. Be aware that issues
in those versions may have been fixed in this repository already, as those
releases are updated less frequently.

[stable]: https://doc.rust-lang.org/stable/book/
[beta]: https://doc.rust-lang.org/beta/book/
[nightly]: https://doc.rust-lang.org/nightly/book/ -->

[Este livro está disponível na forma de árvore morta pela No Starch Press][nostarch].

[nostarch]: https://nostarch.com/rust

Você também pode ler o livro gratuitamente online. Por favor veja o livro assim como incluído com as versões [estável][stable], [beta] e [nightly] de Rust. Esteja ciente de que problemas nessas versões podem já ter sido corrigidas neste repositório, dado que essas versões são atualizadas menos frequentemente.

[stable]: https://doc.rust-lang.org/stable/book/
[beta]: https://doc.rust-lang.org/beta/book/
[nightly]: https://doc.rust-lang.org/nightly/book/

<!-- ## Requirements

Building the book requires [mdBook], ideally the same 0.3.x version that
rust-lang/rust uses in [this file][rust-mdbook]. To get it:

[mdbook]: https://github.com/rust-lang-nursery/mdBook
[rust-mdbook]: https://github.com/rust-lang/rust/blob/master/src/tools/rustbook/Cargo.toml

```bash
$ cargo install mdbook --vers [version-num]
``` -->

## Requisitos

Construir o livro requer o [mdBook]. Idealmente a mesma versão 0.3.x que rust-lang/rust usa [nesse arquivo][rust-mdbook]. Para obtê-lo:

[mdbook]: https://github.com/rust-lang-nursery/mdBook
[rust-mdbook]: https://github.com/rust-lang/rust/blob/master/src/tools/rustbook/Cargo.toml

```bash
$ cargo install mdbook --vers [version-num]
```

<!-- ## Building

To build the book, type:

```bash
$ mdbook build
```

The output will be in the `book` subdirectory. To check it out, open it in
your web browser.

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

To run the tests:

```bash
$ mdbook test
``` -->

## Construindo

Para construir o livro, entre:

```bash
$ mdbook build
```

O resultado estará na subpasta `book`. Para checá-lo, o abra no seu browser.

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

<!-- ## Contributing

We'd love your help! Please see [CONTRIBUTING.md][contrib] to learn about the
kinds of contributions we're looking for.

[contrib]: https://github.com/rust-lang/book/blob/master/CONTRIBUTING.md -->

## Contribuindo

Nós adoraríamos sua ajuda! Por favor veja o [CONTRIBUTING.md][contrib] para saber mais sobre o tipo de contribuições que nós procuramos.

[contrib]: https://github.com/rust-br/rust-book-pt-br/blob/master/CONTRIBUTING.md

<!-- ### Translations

We'd love help translating the book! See the [Translations] label to join in
efforts that are currently in progress. Open a new issue to start working on
a new language! We're waiting on [mdbook support] for multiple languages
before we merge any in, but feel free to start!

[translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang-nursery/mdBook/issues/5 -->

### Traduções

Nós adoraríamos ajuda na tradução deste livro! Veja os labels [Translating][translations] para se ajuntar aos esforços que estão atualmente em progresso. Abra novas issues para começar a trabalhar num novo idioma! Nós estamos esperando pelo [suporte do mdbook][mdbook support] a múltiplos idiomas, antes de juntá-las a esse repositório, mas sinta-se livre para começar!

[translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang-nursery/mdBook/issues/5

<!-- ## Graphviz dot

We're using [Graphviz](http://graphviz.org/) for some of the diagrams in the
book. The source for those files live in the `dot` directory. To turn a `dot`
file, for example, `dot/trpl04-01.dot` into an `svg`, run:

```bash
$ dot dot/trpl04-01.dot -Tsvg > src/img/trpl04-01.svg
```

In the generated SVG, remove the width and the height attributes from the `svg`
element and set the `viewBox` attribute to `0.00 0.00 1000.00 1000.00` or other
values that don't cut off the image. -->

## Graphviz dot

Nós usamos o [Graphviz](http://graphviz.org/) para alguns dos diagramas no livro. Os arquivos-fonte estão no diretório `dot`. Para transformar um arquivo `dot`, por exemplo o `dot/trpl04-01.dot`, em `svg`, rode:

```bash
$ dot dot/trpl04-01.dot -Tsvg > src/img/trpl04-01.svg
```

No arquivo SVG gerado, remova os atributos _width_ e _height_ do elemento `svg` e dê `0.00 0.00 1000.00 1000.00`, ou outros valores que não cortem a imagem, ao atributo `viewBox`.

<!-- ## Spellchecking

To scan source files for spelling errors, you can use the `spellcheck.sh`
script. It needs a dictionary of valid words, which is provided in
`dictionary.txt`. If the script produces a false positive (say, you used word
`BTreeMap` which the script considers invalid), you need to add this word to
`dictionary.txt` (keep the sorted order for consistency). -->

## Checagem de grafia

Para procurar erros de grafia nos arquivos, você pode usar o _script_ `spellcheck.sh`. Ele precisa de um dicionário de palavras válidas, o qual é provido em `dictionary.txt`. Se o script produzir um falso positivo (por exemplo, se você usou a palavra `BTreeMap`, a qual o script considera inválida), você precisará adicionar essa palavra a `dictionary.txt` (mantenha-o ordenado por consistência).
