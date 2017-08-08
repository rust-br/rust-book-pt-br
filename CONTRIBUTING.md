# Contribuições
## Por onde começar?
**1.** Faça o _fork_ do projeto.

**2.** [Entenda nosso fluxo](#fluxo).

**3.** [Leia e pratique as boas práticas](#boas-pr%C3%A1ticas).

## Fluxo
Qualquer tipo de ajuda (seja ela grande ou pequena) é bem-vinda. Se encontrar qualquer parte do livro que possa ser melhorada, essa é uma grande oportunidade para participar. Caso não saiba por onde começar:

**1.** Fork o projeto.

**2.** Antes de iniciar o processo de contribuição, **crie uma nova branch** para fazer suas alterações.

Alguns exemplos:

- Para tradução: `git checkout -b translate/chapter-01`
- Para revisões: `git checkout -b review/chapter-01`
- Para erros: `git checkout -b fix/chapter-01-line-31`

**3.** Após realizar as alterações, é hora de fazer um commit com uma mensagem coerente do que foi feito. Exemplo:

```
git add --all
git commit -am ‘Adiciona tradução|revisão|melhoria capítulo X linha/linhas Y’
git push origin translate/chapter-o1
```

**4.** Envie um _Pull Request_ com as alterações feitas, fazendo referência para o `master` do repositório oficial.

**5.** Sua contribuição será analisada pela comunidade. Em alguns casos pediremos algumas alterações antes de dar merge.

Após o merge:

- Delete a branch utilizada:

```
git checkout master
git push origin --delete <branch_name>
git branch -D <branch_name>
```

- Atualize seu repositório com o repositório oficial:

```
git fetch upstream
git rebase upstream/master
git push -f origin master
```

**6.** Quando iniciar uma nova contribuição, inicie repita o processo pelo passo 2.

## Sistema de labels e Project
Para nos ajudar a saber o progresso do projeto e de cada capítulo em específico, contamos com um sistema de labels para informar aos colaboradores quais capítulos estão disponíveis e quais já estão sendo trabalhados por alguém.

Os capítulos disponíveis estarão marcados com as labels `aguardando tradução` ou `aguardando revisão`. Caso tenha interesse de ajudar nesses capítulos, comente na issue dizendo que está começando a trabalhar nele e um dos membros da equipe irá atualizar a label.

Você também pode acompanhar o andamento do projeto por [aqui](https://github.com/rust-br/rust-book-pt-br/projects/1).

Qualquer dúvida não hesite em perguntar!

## Boas práticas
- **Não traduza termos técnicos e blocos de código**
- Antes de enviar sua contribuição, certifique-se de que está enviando apenas um **único** commit que represente o que foi feito. Caso tenha feito vários commits, [esmague-os](http://gitready.com/advanced/2009/02/10/squashing-commits-with-rebase.html) antes de fazer o _Pull Request_.
- Caso tenha qualquer tipo de dúvida, abra uma _Issue_ que faremos o possível para te ajudar.
- Contribua com as discussões.
- Estrangeirismo: Utilizar o formato itálico. Exemplo: _bug_
- Sentido Figurado: Sempre destacar com aspas duplas.
- Marcação para código: Utilizar um apóstrofe (\`) para indicar um pedaço de código no meio de um texto (`let mut guess = String::new();`). Ou três apóstrofe com o nome da linguagem de programação na frente (\`\`\`rust), para indicar um bloco de código:

```rust
fn main() {
    println!("Hello, world!");
}
```

## Dúvidas em tradução de termos, palavras, expressões etc…
Quando estiver em uma situação em que você não sabe exatamente como traduzir uma palavra, termo ou expressão, nós recomendamos que abra uma _Issue_ com um título descritivo como por exemplo: “_Como traduzir a palavra/termo “x”?_” e coloque uma descrição fazendo referência à linha e o capítulo que esteja trabalhando.

:crab: :crab: :crab:
